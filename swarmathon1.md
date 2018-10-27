lorem ipsum i dont remember the rest
  
Contents:  
1. [Part 1: Press A to go forwards](#p1)  
2. [Part 2](#p2)  

### <a name="p1"></a>Press A to go forwards
Preface:
Okay, so I wanted to do something really simple- make pressing 'A' on my controller move the robot forwards. First, I just searched the entire repo for the word 'xbox'. The readme noted
http://wiki.ros.org/joystick_drivers
From there, there are four links to the different kinds of joystick drivers, and I just picked the first one because all the others told me they were things that aren't xbox controllers (ps3, wii, and spacenav).
http://wiki.ros.org/joy

This tells me that the info I get is in joy.buttons and joy.axes, and that the index for 'A' is 0.

So I went to the first place I saw inputs being worked with (rover_gui_plugin.cpp), and threw something in.

First, just added:
```
if (joy_msg->buttons[0])
{
emit joystickDriveForwardUpdate(joy_msg->axes[right_stick_y_axis]);
}
```
But it did nothing.

```
right_stick_y_axis
```
to
```
0.5
```

And it still did nothing. Am I building right?

```
if (joy_msg->axes[right_stick_y_axis] <= -0.1)
{
emit joystickDriveForwardUpdate(-joy_msg->axes[right_stick_y_axis]);
//emit joystickDriveBackwardUpdate(-joy_msg->axes[right_stick_y_axis]);
}
```

ran catkin build

ok, nothing's happening

```
void RoverGUIPlugin::joyEventHandler(const sensor_msgs::Joy::ConstPtr& joy_msg)
{
	return;
```

Sanity check.

Okay, that actually did disable manual control. The catkin messages on build could've told me that, but I wanted to make sure.

```
emit joystickDriveForwardUpdate(-joy_msg->axes[right_stick_y_axis]);
```

to

```
emit joystickDriveForwardUpdate(0.5);
```

... Nothing.

Tried putting the return back in, yup this stops all movement. What am I misunderstanding?

Moved button check below

```
if (abs(joy_msg->axes[right_stick_y_axis]) < 0.1)
{
emit joystickDriveForwardUpdate(0);
emit joystickDriveBackwardUpdate(0);
}
```

OH, OOPS.
Changed

```
joy_msg->axes[0.5]
```
to

```
0.5
```
No wonder.

That didn't solve anything though- note that even commenting out the emits didn't actually stop movement.

Commenting out
```
joystick_publisher.publish(joy_msg);
```

Did as expected.
```
emit joystickDriveForwardUpdate(0.5); 
```

Using the fix earlier. Still no.
Side note: iteration time is about 30s. ~15 for build, ~10-15 for sim setup

```
emit sendInfoLogMessage("Potato");
```

In the function right after checking for the publisher. It's not constant, but it happens. Hm.

Tried in the forward as well. At this point, I'm convinced that those messages don't actually do anything.

Okay, commenting them all out and keeping the "potato", the update is called every time there's controller input.

```
joy_msg->axes[right_stick_y_axis]
```

Nope, it's read-only, so that's not how it's changing.

```
emit sendInfoLogMessage(to_string(joy_msg->axes[left_stick_y_axis]));
```

Doesn't work- we actually work with QString and not std::string
The function is QString::number(double)

Worked for the left, but I actually care about the right right now.

Okay, taking a step back here. Press A to potato actually does work, I was just in the wrong tab (make sure to be in the Info Log)

Now, since those messages clearly have no effect on movement, there's only one other place where the joy axes are even looked at in all the code, which is ROSAdapter.cpp, so I threw this in after linear was declared:
```
if(message->buttons[0])
{
linear = max_motor_cmd;
}
```

And that did it- pressing A now let me move forwards.

Main takeaways:
-everything uses QString, not std string
-"emit sendInfoLogMessage()" prints to console
-make sure to switch to the info log tab if you're printing to it

I still don't know
-What are those joy messages for, then? Positional calculation clearly uses something else, since it's so jittery even with 0 input (plus, that'd be dumb to only consider internal forces anyway)


### <a name="p2"></a>Part 2: more things  
zzz2  
