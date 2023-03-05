# C# OpenTK - Programming with OpenGL

I have long wondered how to make my own graphics without having to use Unity or Blender or whatever other game engine. Of course, my aim is not complex graphics. I mostly use graphics programming to create applets or to make some simple simulations. I have heard of OpenGL a while back when I was trying out C++. A lot of people have said that it's a very steep learning curve learning OpenGL. However, I have time... so let's go!

**Disclaimer**: I am not an expert and I will be writing down what I learned in a "notes" style, so follow my tutorials at your own discretion. I am also quite new at C# in general, so take everything I say with a handful of salt. Check out footnotes for references and additional comments.

I will be documenting my learning, and I will try to convey my knowledge into simpler terms. It will be a bit reductive to talk about certain things regarding graphics in simple terms, and it might even upset some people because of the imprecisions. But hey, this is meant to be a beginner-friendly tutorial and it's also supposed to be comprehensible. I've always said that people have a tendency to teach things wrong. There needs to be a balance between making sure a student doesn't get the wrong ideas and limit themselves to beginner level content, and making sure they also aren't overwhelmed by everything that is possible. Anyways, I'd say we can start with the first things!

## Creating a Window

First of all, I will assume that you have Visual Studio/Visual Studio Code or any C# IDE installed. I will also assume that you know in general how to set up a project file. For this we will be creating a C# console project: You can either create one in Visual Studio simply, or use the terminal/shell and run the following:

(Skip if you already created your project folder)
```shell
mkdir <ProjectFolder>
```
and
```shell
cd <ProjectFolder>
dotnet new console
```
Then, install OpenTK:
```shell
dotnet add package OpenTK --version 4.7.7
```
(Check for latest stable version [here](https://www.nuget.org/packages/OpenTK/))

Now, we can start programming. Open the `Program.cs` file and it should look like this:

```CS
using System;

namespace <ProjectFolder>{
    class Program 
    {
        static void Main(string[] args)
        {
            // Delete whatever is in here for now.
        }
    }
}
```

**Remark**: Wherever I wrote `<ProjectFolder>` you should put your project folder name.

Now create a new file and call it whatever you want. I called it `Game.cs`. Inside of it, we declare the OpenTK namespace:

```CS
using System;
using OpenTK;
using OpenTK.Graphics.OpenGL4;
using OpenTK.Windowing.Common;
using OpenTK.Windowing.Desktop;
using OpenTK.Mathematics;

namespace <ProjectFolder>
{
    public class Game
    {

    }
}
```

The `Game` class will be the main class that we use to render anything in our window. This class should extend an OpenTK class called `GameWindow`. The name is pretty self-explanatory, it deals with anything that goes on in a window, such as rendering objects. So now, extend the class and write the first function:

```CS
using System;
using OpenTK;
using OpenTK.Graphics.OpenGL4;
using OpenTK.Windowing.Common;
using OpenTK.Windowing.Desktop;
using OpenTK.Mathematics;

namespace <ProjectFolder>
{
    public class Game : GameWindow
    {
        public Game() : base(GameWindowSettings.Default, NativeWindowSettings.Default) 
        { 
            // Leave empty for now.
        }
    }
}
```

The `base` keyword is used to call the constructor of `GameWindow`. So we initialise the game window using default settings [^1]. Now, for the most exciting part of this section... Creating a window! To create a window you need to define two other functions:

```CS
using System;
using OpenTK;
using OpenTK.Graphics.OpenGL4;
using OpenTK.Windowing.Common;
using OpenTK.Windowing.Desktop;
using OpenTK.Mathematics;

namespace <ProjectFolder>
{
    public class Game : GameWindow
    {
        public Game() : base(GameWindowSettings.Default, NativeWindowSettings.Default) 
        { 
            this.CenterWindow(new Vector2i(400, 400));
        }

        protected override void OnUpdateFrame(FrameEventArgs args)
        {
            base.OnUpdateFrame(args);
        }

        protected override void OnRenderFrame(FrameEventArgs args) 
        {
            GL.ClearColor(new Color4(0.3f, 0.4f, 0.5f, 1.0f));
            GL.Clear(ClearBufferMask.ColorBufferBit);

            this.Context.SwapBuffers();
            base.OnRenderFrame(args);
        }

    }
}
```
**Note**: I mix up my jargon a lot. When I say "function" in the context of a class, I obviously mean "method". It is pretty much the same.

Ok, this is a lot more code. We have two new methods that we are over-riding from the `GameWindow` class. One is `OnUpdateFrame` and the other `OnRenderFrame`. It is pretty straightforward actually [^2][^3]. Both functions are called for every frame (i.e. each time the window refreshes). However, one is exclusively for updating, such as updating values of a function or certain contents in the world of a game you're making. `OnRenderFrame`, on the other hand, is called to render graphics, such as setting background color or drawing a shape. I'm pretty sure they virtually do the same thing.

In simple terms:
- `base.OnUpdateFrame(args)` is just calling the base class' `OnUpdateFrame` method. Nothing interesting..
- `GL` is a class from the `OpenTK.Graphics.OpenGL4` namespace. It's the most important class we have and it's the one that has everything to do with graphics.
  - `GL.ClearColor` is a method that sets the color of the color buffer [^4]. In this case, this function sets the background color.
  - `GL.Clear` is the method that applies the color we specified.
- `this.Context.SwapBuffers` is a method that swaps front and back buffers, which shows the rendered scene to the user.
- `this.CenterWindow` is pretty self-explanatory, and it has one overload which lets you define the size of the window as a 2D vector of integers, which in this case we can define as a `Vector2i` (from the `OpenTK.Mathematics` namespace) where the "i" stands for "integer".

Now if we run this code, a window with a blue background will pop up:

<div align="center">
<img src="https://user-images.githubusercontent.com/79821802/222937363-62b0fae1-374d-4f13-bef5-c54d2a7cfe9d.png" width=400/>
</div>

and we are done with the simple part! Next up, drawing a triangle (a.k.a. the "Hello World" of graphics programming).

### References
- [Two-Bit Coding](https://www.youtube.com/@two-bitcoding8018)

[^1]: Not sure what it means yet.
[^2]: Check out a more detailed response on [StackOverflow](https://stackoverflow.com/a/23552542/18637675).
[^3]: More details on the methods of `GameWindow` in the [documentation](https://opentk.net/api/OpenTK.Windowing.Desktop.GameWindow.html#methods).
[^4]: I'm not sure I know what it means yet, but [here's an explanation](http://what-when-how.com/opengl-programming-guide/buffers-and-their-uses-the-framebuffer-opengl-programming/).
