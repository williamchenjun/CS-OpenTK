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

In OpenTK, vertex and fragment shaders are written in GLSL, which stands for OpenGL Shading Language. Let us start, writing a shader class:

```CS
namespace <ProjectFolder>
{
    public class Shader
    {
        
    }
}
```

As mentioned above, we will have to write our vertex and fragment shader files separately. They will have the extension `.vert` and `.frag` and will be written in GLSL. These files are going to be imported into our shader class and compiled at runtime. Thus, we write code to do just that:

```CS

```

### References

- [Shader Basics, Blending & Textures • Shaders for Game Devs](https://www.youtube.com/watch?v=kfM-yu0iQBk)
- [LearnOpenTK](https://github.com/opentk/LearnOpenTK/blob/master/Common/Shader.cs)

[^1]: Read more about about types of shader data [here](https://github.com/opentk/LearnOpenTK/blob/master/Chapter1/2-HelloTriangle/Shaders/shader.vert)
