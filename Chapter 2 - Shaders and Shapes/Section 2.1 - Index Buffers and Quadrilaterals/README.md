## Index Buffers

[:point_right: Jump to Section 2.2](https://github.com/williamchenjun/CS-OpenTK/blob/main/Chapter%202%20-%20Shaders%20and%20Shapes/Section%202.2%20-%20Coordinate%20Transformation/README.md)<br>
[:point_left: Go back to Chapter 2](https://github.com/williamchenjun/CS-OpenTK/blob/main/Chapter%202%20-%20Shaders%20and%20Shapes/README.md)<br>
[:point_left: Go back to home](https://github.com/williamchenjun/CS-OpenTK)

So, we have successfully created a triangle. We only used a vertex buffer, and if you followed the additional step, we also used a color buffer. One thing you might already know is that every rendered surface 2D or 3D on the screen is made out of meshes. A mesh is a set of nodes (vertices) and edges (lines). In Mathematics it's called a fully connected and complete graph. Albeit undirected, that is, the edges don't carry additional information such as a direction. The most minimal polygon that we can create with a set of nodes and edges is a triangle. That is why most of the meshes are made out of triangles. It makes our surfaces more flexible to work with and if you partition a surface enough you can get a very smooth surface topology. There is a whole branch of partial differential equations that deals with numerical approaches to solving them, and those methods include also using triangular meshes[^1].

Anyways, enough talk about meshes and maths. What I am getting at is that in OpenGL everything is made out of a triangular mesh. That means that if we want to define a polygon with more than 3 sides, it's going to be made of 2, 3, ..., $N$ triangles.

**Example**:

<div align="center">
<img src="https://user-images.githubusercontent.com/79821802/222978762-8f4c89ef-053b-45d7-aaf3-c34e6f4eb921.png" width=300/><br>
<span>
<sup><sub>
<b>Figure 1</b>: Polygons made up of non-overlapping triangles.
</sub></sup>
</span>
</div><br>

In OpenTK we can define shapes with more than 3 vertices by using an index buffer. An index buffer assigns an index to our vertices, so that we can reassign some of them to be reused to make a second triangle or more.

<div align="center">
<img width="600" alt="triangular_mesh" src="https://user-images.githubusercontent.com/79821802/222980841-704feb47-bddc-4e7f-88a4-91ea8b340e0f.png"><br>
<span>
<sup><sub>
<b>Figure 2</b>: Triangles in a quadrilateral.
</sub></sup>
</span>
</div><br>

### Create an index buffer

Now, let's move on to actually applying this knowledge to create a rectangle. As you may have guessed, it's just as easy as attaching some colors. We define our buffer in the same way we did before. First define an array of unsigned integers that will be our indices:

```CS
uint[] indices = new uint[]
{
    0, 1, 2,
    2, 1, 3
};
```
(I will be going clockwise), and make sure that you add another vertex to the `vertices` array

```CS
float[] vertices = new float[]
{
    -0.7f, -0.5f, 0.0f,  // bottom left
    -0.7f,  0.5f, 0.0f,  // top left
     0.7f, -0.5f, 0.0f,  // bottom right
     0.7f,  0.5f, 0.0f   // top right
};
```

I changed the `x` coordinate as well so we have a rectangle instead of a square. You may want to add a new color to the `colors` array as well, but it may be ok not to do that as well.

Afterwards, we define the buffer under all the other of our buffers (but **before you define the VAO**)

```CS
this.indexBufferHandle = GL.GenBuffer();
GL.BindBuffer(BufferTarget.ElementArrayBuffer, this.indexBufferHandle);
GL.BufferData(BufferTarget.ElementArrayBuffer, indices.Length * sizeof(uint), indices, BufferUsageHint.StaticDraw);
GL.BindBuffer(BufferTarget.ElementArrayBuffer, 0);
```

This time around our buffer target is the element array buffer. But for the rest it's just the same. Then, **after defining the VAO**, bind this buffer to it

```CS
GL.BindBuffer(BufferTarget.ElementArrayBuffer, this.indexBufferHandle);
```

We don't need to to point it to any data in this case and we don't need to enable anything in the shader files. The last step is to just remember to delete the buffer in `OnUnload`

```CS
GL.DeleteBuffer(this.indexBufferHandle);
```

and in `OnRenderFrame` instead of `GL.DrawArray` use `GL.DrawElements`

```CS
GL.DrawElements(PrimitiveType.Triangles, 6, DrawElementsType.UnsignedInt, 0);
```

That's it! If you compile the code, you should be able to see a rectangle

<div align="center">
<img src="https://user-images.githubusercontent.com/79821802/222981683-3d865e68-ec11-4d2b-a8b9-6b4e26e52dfc.png" width=300/><br>
<span>
<sup><sub>
<b>Figure 3</b>: Rendered rectangle defined by two triangles.
</sub></sup>
</span>
</div>

[:point_right: Jump to Section 2.2](https://github.com/williamchenjun/CS-OpenTK/blob/main/Chapter%202%20-%20Shaders%20and%20Shapes/Section%202.2%20-%20Coordinate%20Transformation/README.md)<br>
[:point_left: Go back to Chapter 2](https://github.com/williamchenjun/CS-OpenTK/blob/main/Chapter%202%20-%20Shaders%20and%20Shapes/README.md)<br>
[:point_left: Go back to home](https://github.com/williamchenjun/CS-OpenTK)

### References
- [OpenTK](https://opentk.net/learn/chapter1/3-element-buffer-objects.html)

[^1]: You can look into the [Finite Element Method](https://en.wikipedia.org/wiki/Finite_element_method) (FEM) for more information. There are more ways than one to solve PDEs numerically of course.
