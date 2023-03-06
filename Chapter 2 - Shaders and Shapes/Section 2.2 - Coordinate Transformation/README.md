## Convert normal coordinates to NDC

As mentioned before, the coordinate system that OpenGL uses is called the normalized device coordinate system. It range fom -1 to 1 in every direction. So it's basically a percentage of how left right up down you want your object to be. However, that can be incovenient when programming because it doesn't really mean much. Where as being able to define how many "pixels"/fragments wide a shape is might be better.

There isn't a lot of programming involved to do this. Instead of writing a function to convert float to NDC, we directly write the conversion formula in the vertex shader code. First of all, let's take a look at what we mean by this conversion

<div align="center">
<img width="500" alt="ndc" src="https://user-images.githubusercontent.com/79821802/222994940-1536fccd-45ed-4846-957a-5d4507e23488.png"><br>
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
