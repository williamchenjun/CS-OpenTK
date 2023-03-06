## Convert normal coordinates to NDC

As mentioned before, the coordinate system that OpenGL uses is called the normalized device coordinate system. It ranges fom -1 to 1 in every direction. So it's basically a percentage of how left right up down you want your object to be. However, that can be incovenient when programming because it doesn't really mean much. Whereas, being able to define how many "pixels"/fragments wide a shape is might be better.

There isn't a lot of programming involved to do this. Instead of writing a function to convert float to NDC, we directly write the conversion formula in the vertex shader code. First of all, let's take a look at what we mean by this conversion

<div align="center">
<img width="500" alt="ndc" src="https://user-images.githubusercontent.com/79821802/222996442-6d172212-e350-4138-a60c-7ea3a509d745.png"><br>
<span>
  <sup><sub>
    <b>Figure 1</b>: Indicative representation of the coordinate transformation.
  </sub></sup>
</span>
</div>

The coordinate conversion is actually pretty simple. Given $(x,y)$ coordinates in the normal plane, we can convert them to NDC in the following way

```math
\begin{align*}
(n_x, n_y) : \begin{cases}
n_x := \dfrac{x}{\text{viewport width}}\cdot 2 - 1\\[1em]
n_y := \dfrac{y}{\text{viewport height}}\cdot 2 - 1
\end{cases}
\end{align*}
```

A rough explanation is that, dividing x and y respectively by the width and height normalizes them (i.e. puts them in the range 0 to 1). However, the NDC space is from -1 to 1, we double it to extend the range, but we subtract 1 to bring it to the correct side.

**Example**: 

<div align="center">
<img src="https://user-images.githubusercontent.com/79821802/222996306-7793cf3c-06cd-4f7e-b8a3-07ff6519b7ac.png" width=400/><br>
<span>
  <sup><sub>
    <b>Figure 2</b>: The viewport here is (160, 160).
  </sub></sup>
</span><br><br>
<img src="https://user-images.githubusercontent.com/79821802/222996311-78493690-3991-4e51-a463-b91b8c733f43.png" width=400/><br>
<span>
  <sup><sub>
    <b>Figure 3</b>: The final transformed coordinate is (0.25, 0.25).
  </sub></sup>
</span>
</div><br>

### Changing the shader code

In the vertex shader code, we need to instruct the GPU to transform the coordinates so that the fragment shader can render the shape in the correct place.

In `shader.vert`

```GLSL
#version 410

uniform vec2 ViewportDims;
layout(location = 0) in vec3 aPosition;
layout(location = 1) in vec4 aColor;

out vec4 vColor;

void main(void){
    float nx = aPosition.x / ViewportDims.x * 2.0f - 1.0f;
    float ny = -aPosition.y / ViewportDims.y * 2.0f + 1.0f;

    gl_Position = vec4(nx, ny, 0.0f, 1.0f);

    vColor = aColor;
}
```

Let's go through all the new lines:
- The `uniform` keyword [^1] is used to specify a parameter in the vertex shader that allows communication between the program and the shader without the use of a buffer or VAO. They remain constant during the rendering process.
  - `vec2` is a 2D vector.
  - Thus, we defined a variable that will accept input from outside the shader code scope, and it will be a 2D vector that will be our viewport size.
- The `float nx = aPosition.x / ViewportDims.x * 2.0f - 1.0f` is the formula we specified earlier, and the same thing for `ny`.

**Warning**: Remember that `1f` is not a valid input. Floats are defined either as `1.0f`, `1.0` or using a `float` constructor [^2] (e.g. `as_float = float(x)` where `x` is some integer).

This is all the code needed to make the coordinate transformation. However, we now need to find a way to pass the viewport size to the shader. We do that in the shader code `Shader.cs`. At the bottom of the shader class constructor add the following lines:

```CS
int[] viewport = new int[4];
GL.GetInteger(GetPName.Viewport, viewport);
GL.UseProgram(ProgramHandle);
int viewport_unif = GL.GetUniformLocation(ProgramHandle, "ViewportDims");
GL.Uniform2(viewport_unif, (float)viewport[2], (float)viewport[3]);
GL.UseProgram(0);
```

- The `GL.GetInteger` method can fetch us any variable relating to the program. In this case, we ask the viewport size information. This will return a 4D array containing the anchor points and the width and height `(x, y, w, h)`. So, in this case, it would return `(0, 0, 800, 800)`. The second argument of the method indicates the output variable (i.e. where the data should be stored).
- `GL.UseProgram(ProgramHandle)` just indicates that we want to communicate with the shader code.
- Then, we find the location of the `uniform` variable that we defined in `shader.vert` by using `GL.GetUniformLocation` and specifying the `uniform` variable name.
- The `GL.Uniform2` method is the way we pass the data to the variable in the shader code. You might notice that there are various `Uniform` methods. That's because you can pass any type of data to the shader, from N-dimensional arrays to matrices and more. `Uniform2` is a 2D vector with float values.
- Lastly, we unbind the `ProgramHandle`.

That's all! Now, if you compile the code, you probably will not see anything. That's because you need to actually change your rectangle vertices to normal coordinates. For example:

```CS
float x = 160f;
float y = 300f;
float w = 500f;
float h = 200f;

float[] vertices = new float[]
{
    x,      y,       0.0f,
    x,      y+h,     0.0f,
    x+w,    y,       0.0f,
    x+w,    y+h,     0.0f
};
```

My viewport size is `(800, 800)`. So, if we printed out the transformed coordinates, it would be

```CS
float[] transformed_vertices = new float[]
{
    -0.6f,   -0.25f,  0.0f,
    -0.6f,    0.25f,  0.0f,
    0.65f,   -0.25f,  0.0f,
    0.65f,    0.25f,  0.0f
};
```

(Although the coordinates rotate one step clockwise, but I wrote it how we would write it normally)

You can use [this desmos graph](https://www.desmos.com/calculator/rgbom4dfqw) I created to see how the output would look like. Just change the values of `v, a, b, w, h` where `v : viewport` (add a new viewport variable if your viewport is rectangular), `a : x coordinate`, `b : y coordinate`, `w : width of rectangle`, `h : height of rectangle`.

<div align="center">
  <img src="https://user-images.githubusercontent.com/79821802/223028430-1d254949-be71-4131-b767-3229fe99c8f8.png" width=400/><br>
  <span>
  <sup><sub>
    <b>Figure 4</b>: Rendered rectangle after coordinate transformation.
  </sub></sup>
</span>
</div><br>



### Reference
- [Two-bit Coding](https://www.youtube.com/@two-bitcoding8018)

[^1]: Read more about the `uniform` keyword [here](https://www.khronos.org/opengl/wiki/Uniform_(GLSL)).
[^2]: Read more about GLSL data types [here](https://www.khronos.org/opengl/wiki/Data_Type_(GLSL)).
