## OpenTK - Shaders

Shaders must be one of the most confusing topics for me, especially because I am not familiar with Unity, Blender, or any other game engine, and I have absolutely no experience in graphics or game development. The following highlights the main things that I understood about shaders.

In simple terms, a shader is code that runs through your GPU which instructs it to render certain pixels a certain way. So shaders contain the math necessary to color or move objects the way they do, otherwise, we wouldn't see anything. This is why it is crucial to get a good grasp about shaders before jumping into graphics programming. There are various shaders which are used depending on the desired output, style, behaviour. However, it doesn't necessarily have to be complex if you don't want to. For example, vertex shaders are used to render vertices into pixels onto your screen.

### Vertex Shaders

Let's consider vertex shaders. By passing an array of vertices to the shader code as input[^1], you can think of it like the shader will loop over each vertex passed. This way you can apply some properties on each of the vertices. An example usage of this shader is to render movement, such as water, plants, or even just a rotating cube.

<div align="center">
<img src="https://media.tenor.com/kALaOTg9xiYAAAAC/cube-square.gif" height=200/><br>
<span>
<sup><sub>
<b>Figure 1</b>: A GIF of a rotating cube.
</sub></sup>
</span>
</div><br>

Now, a vertex shader will let you specify certain rules for the vertices, however, the rendering of the shape will happen due to a process called rasterization (Figure 2). I am still not very sure what this process pertains, but basically it's a (background) process where the program will figure out where to color the pixel depending on where you defined the vertices. We are going to see this when we start rendering a triangle.

### Fragment Shader

Together with the vertex shader, we usually have a fragment shader. After the rasterization process is done, we have available a new array of data. We call each "pixel" defined to color our triangle a **fragment**. The reason is merely because a fragment doesn't correspond precisely to a pixel on our monitor, but rather a few pixels. So you're free to just approximate the terminology and say that a fragment is a pixel. That's also why they sometimes call it the "pixel shader". In the fragment shader, you basically set the color of each fragment or how each fragment is going to appear.

**Examples**:
<div align="center">
<img src="https://www.computerhope.com/jargon/r/rasterize.jpg" height=300/>
<br>
<span>
<sup><sub>
<b>Figure 2</b>: An example of rasterization happening after a vertex shader is defined.
</sub></sup>
</span>
<br><br>
<img src="https://user-images.githubusercontent.com/79821802/222942908-e8d73aba-b1ef-498f-ba0d-d5bb7050c467.png" width=400/><br>
<span>
<sup><sub>
<b>Figure 3</b>: Process from the defining the vertices to generating a triangle.
</sub></sup>
</span>
</div><br>

In OpenTK, vertex and fragment shaders are written in GLSL, which stands for OpenGL Shading Language. As mentioned above, we will have to write our vertex and fragment shader codes separately. You can either write them in your main file, or in separate files and import them as string. I will share the full shader code and then comment on it below:

```CS
using System;
using System.IO;
using System.Text;
using System.Collections.Generic;
using OpenTK.Graphics.OpenGL4;
using OpenTK.Mathematics;

namespace Test
{
    public class MyShader
    {
        public int ProgramHandle;
        public MyShader(string vert, string frag)
        {
           // Start by reading the shader codes.
            string VertexShaderCode = File.ReadAllText(vert);
            string FragShaderCode = File.ReadAllText(frag);

            int vertexShaderHandle = GL.CreateShader(ShaderType.VertexShader);
            GL.ShaderSource(vertexShaderHandle, VertexShaderCode);

            int fragmentShaderHandle = GL.CreateShader(ShaderType.FragmentShader);
            GL.ShaderSource(fragmentShaderHandle, FragShaderCode);

            GL.CompileShader(vertexShaderHandle);
            GL.CompileShader(fragmentShaderHandle);

            this.ProgramHandle = GL.CreateProgram();
            GL.AttachShader(this.ProgramHandle, vertexShaderHandle);
            GL.AttachShader(this.ProgramHandle, fragmentShaderHandle);

            GL.LinkProgram(this.ProgramHandle);

            // We don't need these anymore.
            GL.DetachShader(this.ProgramHandle, vertexShaderHandle);
            GL.DetachShader(this.ProgramHandle, fragmentShaderHandle);

            GL.DeleteShader(vertexShaderHandle);
            GL.DeleteShader(fragmentShaderHandle);
        }

        public void Use()
        {
            GL.UseProgram(this.ProgramHandle);
        }
    }
}
```

This is the bare minimum code. You could add more to it, but let's just start from here:
- We start by defining a shader class and a constructor. The constructor takes two string arguments `vert` and `frag`. These will be our two text files containing GLSL code for our vertex and fragment shaders.
- Define two local variables called `VertexShaderCode` and `FragShaderCode`, or whatever you choose. They will be strings: `File.ReadAllText` reads the files we pass and return the content as a string.
- We now define two variables to act as a temporary shader handle: `vertexShaderHandle` and `fragmentShaderHandle`. The `GL.CreateShader` method is pretty straightforward. They initialise a shader of the specified type. In this case, `vertexShaderHandler` is of type `ShaderType.VertexShader` and `fragmentShaderHandle` is of type `ShaderType.FragmentShader`.
- We need to compile both of them using `GL.CompileShader` to acutally have our shaders.
- Once we have compiled our two separate shaders, we need to link them together as explained earlier, so that after the vertex shader is done, the fragment shader can actually output color and render the shape on our screen. So, define a global variable `ProgramHandle` or whatever you want to name it.
- Set the `ProgramHandle` to `GL.CreateProgram`. This creates a program that can link together shaders.
- Indeed, we use `GL.AttachShader` to append the shaders to the program.
- Finally, use `GL.LinkShaders` to merge them into one shader.
- The last steps are done to remove the unnecessary shaders. Since we have merged everything into one program, we don't need the individual shaders anymore. So use `GL.DetachShader` and `GL.DeleteShader` to detach them from the program and delete them.
- Lastly, define a method `Use` to call the shader later on.

That's it! We have our first shader. There is still a bit to write in the `Game.cs` file:

```CS
using System;
using OpenTK;
using OpenTK.Graphics.OpenGL4;
using OpenTK.Windowing.Common;
using OpenTK.Windowing.Desktop;
using OpenTK.Windowing.GraphicsLibraryFramework;
using OpenTK.Mathematics;

namespace Test
{
    public class Game : GameWindow
    {
        // Where we store the vertices on the GPU.
        private int vertexBufferHandle;
        private int vertexArrayHandle;
        private MyShader shader;
        public Game() : base(GameWindowSettings.Default, NativeWindowSettings.Default) 
        { 
            this.CenterWindow(new Vector2i(400, 400));
        }

        protected override void OnLoad()
        {
            base.OnLoad();

            GL.ClearColor(new Color4(0.3f, 0.4f, 0.5f, 1.0f));

            float[] vertices = new float[]
            {
                -0.5f, 0.0f, 0.0f,
                0.0f, 0.5f, 0.0f,
                0.5f, 0.0f, 0.0f
            };
            
            this.vertexBufferHandle = GL.GenBuffer();
            GL.BindBuffer(BufferTarget.ArrayBuffer, this.vertexBufferHandle);
            // a float is 4 bytes, so in total we have 36 bytes of data.
            GL.BufferData(BufferTarget.ArrayBuffer, vertices.Length * sizeof(float), vertices, BufferUsageHint.StaticDraw);
            GL.BindBuffer(BufferTarget.ArrayBuffer, 0);

            // Everything written after this will be stored in the vertex array.
            this.vertexArrayHandle = GL.GenVertexArray();
            GL.BindVertexArray(this.vertexArrayHandle);
            // Defines the vertex attributes.
            GL.BindBuffer(BufferTarget.ArrayBuffer, this.vertexArrayHandle);
            GL.VertexAttribPointer(0, 3, VertexAttribPointerType.Float, false, 3 * sizeof(float), 0);
            GL.EnableVertexAttribArray(0);

            GL.BindVertexArray(0);

            shader = new MyShader("Shader/shader.vert", "Shader/shader.frag");
        }

        protected override void OnUnload()
        {
            base.OnUnload();
            GL.BindBuffer(BufferTarget.ArrayBuffer, 0);
            GL.DeleteBuffer(this.vertexBufferHandle);

            GL.UseProgram(0);
            GL.DeleteProgram(shader.ProgramHandle);
        }

        protected override void OnRenderFrame(FrameEventArgs args)
        {
            base.OnRenderFrame(args);
            GL.Clear(ClearBufferMask.ColorBufferBit);
            this.shader.Use();
            GL.BindVertexArray(this.vertexArrayHandle);
            GL.DrawArrays(PrimitiveType.Triangles, 0, 3);

            this.Context.SwapBuffers();
            
        }

        protected override void OnUpdateFrame(FrameEventArgs args)
        {
            base.OnUpdateFrame(args);
        }

        protected override void OnResize(ResizeEventArgs e)
        {
            base.OnResize(e);

            GL.Viewport(0, 0, Size.X, Size.Y);
        }

    }
}
```
Let me try describing everything to you:
- We start by defining 3 global variables: `vertexBufferHandle`, `vertexArrayHandle` and our `shader`. The first variable represents a vertex buffer object (VBO). It is basically what carries our vertex data to the GPU. The second variable represents a vertex array object (VAO) and it helps us define how the vertices are actually structured, since there is no inherent way for the engine to know anything about our vertices.
- In `OnLoad` we have the `GL.ClearColor` method to define the background color. Then, we have a float array made up of 9 elements, representing 3 vertices.
- As I said in the first point, `vertexBufferHandle` is what carries information to the GPU. We instantiate a buffer using `GL.GenBuffer` and we bind the buffer using `GL.BindBuffer`. The type is `BufferType.ArrayBuffer`. Then, we need to define what data the buffer is going to carry using `GL.BufferData`: The first argument is the **type** of the buffer, the second argument is the **size** of the data in bytes, the third argument is the data itself, and the last argument is how we want to use our data. The type `BufferUsageHint.StaticDraw` indicates that our data is not going to change at all over time. There's also other types you can explore.
- Finally, we unbind the buffer by writing `GL.BindBuffer(BufferTarget.ArrayBuffer, 0)`.
- The next part defines the vertex array object (VBA). We start by instantiating the VAO with `GL.GenVertexArray`. Now everything that we write after this point, will point back to the VAO.
- We start by binding the VAO to the engine using `GL.BindVertexArray`.
- We do much of the same thing here as above (although it is not the same thing). We bind the buffer with `GL.BindBuffer`. Then, we define our vertices attributes using `GL.VertexAttribPointer`: The first argument represents the index of the input data in the shader (read below), the second argument is the size of the array, the third argument is the data type, the fourth argument indicates whether you want the data to be normalised (to normalised device coordinates[^2]), the fifth argument is the stride in bytes (that is how big each vertex is. In this case 12 bytes because we consider it an array of arrays of 3 floats each, so each individual vertex is 12 bytes), lastly, the sixth argument is by how much you want to offset the data (in this case 0 because we want all of it).
- We enable the VAO by writing `GL.EnableVertexAttribArray` and then we unbind it with `GL.BindVertexArray(0)`.
- Finally, we import our shader by instantiating the class. We pass the path to our two shader files `shader.frag` and `shader.vert` (read below).

You have two options to import the shader codes: either write two seperate files such as `shader.frag` and `shader.vert` and import them, or you can write the string in the `Game` class itself (you will need to change the shader class defined above). The `.frag` and `.vert` extentions are not actual extentions. They're just used to differentiate them. They're just files containing strings.

Create a folder in your parent project folder called `Shader`. In it, create two files `shader.frag` and `shader.vert`.

In `shader.vert` write:

```
#version 330 core

layout(location = 0) in vec3 aPosition;

void main(void){
    gl_Position = vec4(aPosition, 1.0);
}
```

The first line is mandatory and it indicates which version of the engine to use. The `layout(location=0)` is the index value we defined as out first argument for the `VertexAttribPointer`. The `in` keyword means "input" and `vec3` means a 3D vector. So the input data we expect is a 3D array. The GLSL must have a main function which specifies the position of the vertices in `gl_Position` as a 4D array.

In `shader.frag` write:
```
#version 330

out vec4 outputColor;

void main()
{
    outputColor = vec4(1.0, 1.0, 0.0, 1.0);
}
```

Similarly, we have the version. Then, we use the keyword `out` to indicate that it should output a color (which is what we see on the screen), and in the main function we specify the color of each fragment/pixel (which will be the color of our triangle).

Now back in `Game` you can initialise the shader with 

```CS
shader = new MyShader("Shader/shader.vert", "Shader/shader.frag");
```

You can ignore `OnUnload` for now. If you want to know more read [this](https://github.com/opentk/LearnOpenTK/blob/master/Chapter1/2-HelloTriangle/Window.cs) (scroll to the bottom). 

In the `OnRenderFrame` method, we do as we did before to create a window and blue background. We start by setting the background color with `GL.Clear(ClearBufferMask.ColorBufferBit)` (define the color in `OnLoad` with `GL.ClearColor`). Then, we specify that we want to use our shader by using the method `Use` from our shader. Afterwards, we bind the VAO and draw our triangles using `GL.DrawArrays`. The first argument of the latter method represents the type of shape we want, the other arguments are the start index and number of vertices we have. Lastly, we render the scene by using `this.Context.SwapBuffers`.

You can specify the following method to resize your window:
```CS
protected override void OnResize(ResizeEventArgs e)
{
    base.OnResize(e);

    GL.Viewport(0, 0, Size.X, Size.Y);
}
```

and that's it! Now if you compile the file, you should see a yellow triangle on a blue background in a square window.

<div align="center">
  <img src="https://user-images.githubusercontent.com/79821802/222949143-d7d3072a-63a8-444b-8270-183be84886b1.png" width=300/><br>
  <sup><sub><b>Figure 4</b>: The resulting window.</sub></sup>
</div>

That is all I have learnt for now about shaders. This is barely touching the surface and it's already pretty incomprehensible. I hope I can learn more about it and maybe with practice, it will become clearer over time.

### References

- [Shader Basics, Blending & Textures • Shaders for Game Devs](https://www.youtube.com/watch?v=kfM-yu0iQBk)
- [LearnOpenTK](https://github.com/opentk/LearnOpenTK/blob/master/Common/Shader.cs)
- [Two-Bit Coding](https://www.youtube.com/@two-bitcoding8018)

[^1]: Read more about about types of shader data [here](https://github.com/opentk/LearnOpenTK/blob/master/Chapter1/2-HelloTriangle/Shaders/shader.vert)
[^2]: You can read more [here](https://learnopengl.com/Getting-started/Coordinate-Systems#:~:text=OpenGL%20expects%20all%20the%20vertices,range%20will%20not%20be%20visible.) about NBC. But basically coordinates are normalised, that is, the range of x, y, z axes are in the range (-1,1) where (0,0) is the center of the screen.
