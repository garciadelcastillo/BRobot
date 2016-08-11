# BRobot Walkthrough

This section will introduce you to the basics of using BRobot to control mechanical actuators, and walk you through different examples that cover most of the aspects that make this library easy, powerful and fun to work with. 


## Disclaimer

__Working with robots is dangerous.__ Robotic actuators are very powerful machines, but for the most part extremely unaware of their environment; if it collides with something, including yourself, it will not detect it and try to keep going, posing a threat to itself and the operators surrounding it. This is particularly relevant when running in 'automatic' mode, where several security measures are bypassed for the sake of performance.

When using robots in a real-time interactive environment, plase make sure:
- You have been __adequately trained__ to use that particular machine,
- you are in __good physical and mental condition__,
- you are operating the robot under the __utmost security measures__,
- you are following the facility's and facility staff's __security protocols__,
- and the robot has the __appropriate guarding__ in place, including, but not reduced to, e-stops, physical barriers, light courtains, etc. 

__BRobot is in a very early stage of development.__ You are using this software at your own risk, no warranties are provided herewith, and unexpected results/bugs may arise during its use. Always test and simulate your applications thoroughly before running them in a real device. The author/s shall not be liable for any injuries, damages or losses consequence of using this software in any way.


## Introduction

BRobot is a .NET library for real-time manipulation and control of mechanical actuators. It exposes a human-relatable high-level API of abstract actions, which are managed and streamed to connected devices in real time. BRobot is particularly suited for building interactive applications that make the most of sensing and acting on the environment through physical devices.

As of version 0.1.0, BRobot can only interface with ABB robotic arms, and this will be the main focus of the examples provided in this walkthrough. More devices will hopefully come soon, such as Kuka robots, Universal Robots, Arduinos, 3D printers, drones, etc... If you are interested in helping with this, please check the Contribute section.


## Setup

Setting a BRobot project up with extremely easy:

- First, clone this repo to your local machine and open it in Visual Studio. The project comes with the core library, as well as many app examples. 

- If you have not installed RobotStudio in your machine, make sure you do so by [reading this guide](https://github.com/garciadelcastillo/BRobot/blob/master/docs/Setting_up_RobotStudio.md). As specified in the guide, create a station with your choice of robot manipulator, and make sure it is started and running in auto mode. 

- To make sure everything is working, run the 'EXAMPLE_ConnectionCheck' project. If you see a long debug dump with a lot of information from this station, it means connection is live and we can start working!


## Hello World!

At this point, you should have the BRobot project correctly installed in your platform and connected to a virtual ABB robot. The fun is about to start!

Let's run a .NET interactive shell (REPL) with the referenced BRobot assembly. In Visual Studio, go to the Solution Explorer, right click on the BRobot project and choose "Initialize Interactive with Project:"

![](https://github.com/garciadelcastillo/BRobot/blob/master/docs/VS_REPL_01.png)

A C# interactive window should pop up looking like this:

![](https://github.com/garciadelcastillo/BRobot/blob/master/docs/VS_REPL_02.png)

You can think of this as a command line window that has BRobot loaded in it, and through which you can start taking to the robot. 

First thing we need to do is to instantiate a new Robot object to start working with. In the C# interactive, type this:

```csharp
Robot bot = new Robot();
```

`bot` is our new Robot object, which we will use to interface with the virtual device and through which we will issue all instructions. 

There is some setup we need to do before moving on: specify which interaction mode we will be using ([more details later](#modes)), connecting to the controller and starting it. We will start working in [stream](#stream) mode. In the C# interactive, type:

```csharp
bot.Mode("stream");
```

Now, connect to the controller by typing:

```csharp
bot.Connect();
```

If everything went well, you should see a bunch of logs describing the state of the controller. To start running stream mode, type:

```csharp
bot.Start();
```

A log message saying `EXECUTION STATUS CHANGED: Running` is the sign that this worked. Also, if you go to the RAPID tab in RobotStudio, you will observe that the Output window now displays 'Program started', and that there is a big stop button now on the top ribbon bar. The robot is now connected in [stream](#stream) mode, and listening to your instructions! Well done! :)

At this point, we can start issuing actions to the virtual robot, and they will be executed as soon as they get priority. Since the robot is idle right now, anything we send will be executed immediately. Let's move the robot somewhere in the positive XYZ octant. The measures I will be using in this example are suited for a small IRB120, please scale them accordingly if you are simulating a bigger device:

```csharp
bot.MoveTo(300, 100, 200);
```

`MoveTo` is an movement action in absolute global coordinates. You should see the virtual robot start moving slowly to this location in global coordinates. By default, the robot moves at 20 mm/s. We can set the speed for new instructions a bit higher:

```csharp
bot.Speed(100);
```

And issue a new instruction so that the robot moves 200 mm in the positive Z direction:

```csharp
bot.Move(0, 0, 200);
```

`Move` is a relative translation action based on the current location of the robot. You will notice that actions are not immediate, but they happen as soon as all the previously issued ones are complete. The robot was slowly moving to [300, 100, 200], and it wasn't until it reached this point that it increased speed and started moving up. This is what we call _priority-based instruction_. 

Finally, let's move the robot back to a 'home' position by directly setting the rotation angles of the joints. Since this is not a linear movement, we want to be cautious and slow the robot down a little:

```csharp
bot.Speed(50);
bot.Joints(0, 0, 0, 0, 90, 0);
```

As soon as these actions get priority, the robot will start moving back to its initial rest position. 

Before we close the C# interactive window, it is very important to properly disconnect from the controller. Otherwise, memory and subscriptions may not be properly handled, and it may impede our ability to successfuly connect later on:

```csharp
bot.Disconnect();
```

Now you are good to close the C# REPL and RobotStudio if you want ;)


## A simple .NET app

The example above is taking advantage of the C# interactive window capabilities to perform live control of a robotic device. However, the exact same functionality can be achieved inside any .NET app, including console applications, WPF or any other platform that supports .NET (Dynamo, Grasshopper, Unity, etc.). 

For example, the above code can be wrapped in a simple console application. If you create a new console application project in Visual Studio, and add BRobot as a reference, you can get the same result with this code:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using BRobot;

namespace ConsoleApplication1
{
    class Program
    {
        static void Main(string[] args)
        {
            // Instantiate a new Robot object
            Robot bot = new Robot();

            // Set it to work in live streaming mode
            bot.Mode("stream");

            // Connect to the (virtual) controller and start listening to actions
            bot.Connect();
            bot.Start();

            // The robot is listening now! We can start issuing actions!
            // Let's display a message on the FlexPendant
            bot.Message("Let's start rocking!");

            // Now let's move it somewhere in absolute world coordinates
            bot.MoveTo(300, 100, 200);

            // Let's set a higher speed for the next movement
            bot.Speed(100);

            // Let's issue a new order to move 200 mm in the Z direction
            bot.Move(0, 0, 200);

            // And finally, let's bring it back to a 'home' position
            bot.JointsTo(0, 0, 0, 0, 90, 0);

            // Let's halt the program here and let the robot work
            // before exiting the program
            Console.WriteLine("Press any key to EXIT...");
            Console.ReadKey();

            // Remember to disconnect from the controller before leaving!
            bot.Disconnect();
        }
    }
}

```


## The Action Model





## Modes



## Examples




