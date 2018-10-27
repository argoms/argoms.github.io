
Contents:  
[Part 0: Methodology](#p0)  
[Part 1: Press A to go forwards](#p1)  

### <a name="p0"></a>Methodology  
**Quirks and Conventional Wisdom**  
To those of you with more practical experience, a lot of what I'm saying is probably going to sound really cliche. It really is, but remember that something like half the people here haven't completed CS3. Plus, I don't want to focus on the common wisdom part of it. To me, it's very important to define where you can let yourself go a bit- it's super easy to define how the ideal worker functions, but trying to live up to that can easily make you disillusioned and fail to follow *any* rules.  
  
For example, I'm really weird about certain things. My coding speed goes down by like, a factor of 5-10 when I'm around other people; I *really* need to just be alone and think in order to work through problems. I understand the importance of meetings and communication, but my optimal work environment is extremely introverted. Put me next to a few other people at desks, and I'm mentally drained just by sitting there, let alone actually working. I've had the idea of 'you should do the bulk of your work while surrounded by other people' pushed onto me by lots of people in higher positions (professors, project managers etc.), and I can say that it just doesn't work for me- even if the people I'm surrounded by are interesting, skilled, and highly motivated individuals.  
  
Another issue I have is that I'm horrifically bad at small details without feedback- once I'm looking at something, I can analyze what's wrong without problems. However, if I had to [...]. For example, I lost points on my last physics midterm for writing "72.1283 = 75" in my work. Because of the way I eyeball and adjust (more on that later), I'm *really* bad at noticing these things before I can see the results.  
  
**Solving problems: My system in a nutshell**  
-Eyeball it  
	---Like, a dozen lines of code max. Exact amount depends on your familiarity with the system- starting out, I modified like, 1-2 lines at a time.  
	-----Note that it's not really eyeballing it if you actually know exactly what you want and how to achieve it. I wouldn't call it eyeballing to write a simple data structure, like a linked list, in one go.
	---Did it compile? If not, tweak it until it does if you think you know why it didn't compile.  
	-----Now it compiles. Did it work? It's a miracle.    
	
-Observe how it failed  
---Comment out your eyeballed code  
  
-Start removing bits in the *existing* code to see how that affects functionality  
  
-Use existing code as sanity checks  
	---E.g. checking a new button input? use existing messenging  
	
-Sometimes, do everything again  
	---You can make stupid mistakes while following instructions- if you're really confused, try repeating what you just did with a fresh copy    
	  
Note that a critical aspect of this approach is that it *expects* failure.  
  
**Where to start?**
   
Yeah, but what if you can't even conceptualize the larger problem? Ignore it completely- just look at the system, and find *anything* that you can latch onto.  
  
For example, I had no idea where to start, until I realized that you could control the rovers with xbox controllers. From there, that let me search the repository for 'xbox'. That got me to a very limited selection of pages, which told me that the API called it joy. Then I could start looking for joy etc.  
  
Here's the part where I might've lost some people. When I'm looking at small sections of the code, I can more or less tell what they're meant to do by the function names and logic. That part really just comes down to experience, I guess.   
  
Note that in this process, I'm explaining my learning syle. I've found that I learn better this way- reading code examples where I know the result, and changing things.   
  
As far as I've seen, there aren't really any comprehensive tutorials for swarmathon-specific stuff once you get past setup, and ros documentation in general is very barebones. Step by step tutorials are super useful, but you won't really have any here.  
   
### <a name="p1"></a>Press A to go forwards  
Okay, so I wanted to do something really simple- make pressing 'A' on my controller move the robot forwards. First, I just searched the entire repo for the word 'xbox'. The readme noted that joystick drivers were used for this:  
http://wiki.ros.org/joystick_drivers  
  
From there, there are four links to the different kinds of joystick drivers, and I just picked the first one because all the others told me they were things that aren't xbox controllers (ps3, wii, and spacenav).  
http://wiki.ros.org/joy  
  
**This tells me that the info I get is in joy.buttons and joy.axes, and that the index for 'A' is 0.**   
  
I went to the first place I saw inputs being worked with (rover_gui_plugin.cpp), and threw something in.  
  
First, I took the blind shot. The code inside the if statement is copied directly from a part above it, so all I theoretically should be testing is the button press conditional.
```
if (joy_msg->buttons[0])
{
emit joystickDriveForwardUpdate(joy_msg->axes[right_stick_y_axis]);
}
```
This did nothing. I then noticed my first mistake- the forward update message is sending a float according to how far forwards the right stick is tilted- if I'm pressing the button instead of tilting the stick, the stick's tilt will be 0, and I'd just be telling it to move forwards at a speed of 0 (not moving). The comments say that stick input goes from -1 to 1, so I just put a random positive number in (0.5):  

```
right_stick_y_axis
```
to  
```
0.5
```

And it still did nothing. Am I building right? My first concern was that things weren't being built, so I ran catkin clean before building again.  

>This is a point where I'm going to talk about doing stuff that really makes you look like an idiot in retrospect. When you're confused, you clearly don't have a full understanding of the system that you're working with. Therefore, you shouldn't assume that all your preconceptions about the system were correct. At this point, you should be willing to entertain *anything* that even vaguely sounds like it may have something to do with your issue.  
>   
>This is programming. Your absolute worst case scenario is discarding your changes and pulling the most recent stable version off git. It's okay to try stupid things, because you'll feel even stupider if you discounted them for being illogical.  
  
That being said, in this case, I was completely wrong about the issue (more on that later). However, I did learn a bit about the build process: you'll have to execute this (from the swarmlabs installation guide):  
```  
if ! grep -q "source /opt/ros/kinetic/setup.bash" ~/.bashrc
then 
  echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
fi
source ~/.bashrc
```  
again if you catkin clean, or you'll have missing packages and fail to compile.  
  
Next, I tried inverting one of the existing inputs to make going backwards also make you go forwards- maybe there was something wrong with button inputs? This comes back to the methodology I outlined before: once your eyeballed shot in the dark misses, work with existing stuff to figure out what doesn't.    

```
if (joy_msg->axes[right_stick_y_axis] <= -0.1)
{
emit joystickDriveForwardUpdate(-joy_msg->axes[right_stick_y_axis]);
//emit joystickDriveBackwardUpdate(-joy_msg->axes[right_stick_y_axis]);
}
```  
  
I built this, but nothing changed: backwards was still backwards. Now, I was beginning to think that joy_msg wasn't really doing what I thought it did. To test this, I just disabled the function completely by putting 'return' right after the message checks that it exists.
  
```
void RoverGUIPlugin::joyEventHandler(const sensor_msgs::Joy::ConstPtr& joy_msg)
{
	return;
```
  
Okay, that actually did disable manual control. The catkin messages on build could've told me that, but I wanted to make sure. Next, I wanted to *really* be sure that these DriveForwardUpdate messages weren't doing what I thought they did:  

```
emit joystickDriveForwardUpdate(-joy_msg->axes[right_stick_y_axis]);
```

to  

```
emit joystickDriveForwardUpdate(0.5);
```

... Nothing. Movement still worked as before.  
  
Tried putting the return back in, yup this stops all movement. What am I misunderstanding?  
  
I moved the button check (written at the very start) below a section of code that looks like it zeroes movement if there's no joystick input- it's possible that this was why the button wasn't working before.   
>Again, looking like an idiot in retrospect: even modifying the movement values on stuff that *should've* worked had no effect. Therefore, messages don't actually directly influence movement; all of this is barking up the wrong tree.  
  

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
I was checking axis 0.5 (probably turns into a 0, since int conversion just truncates anything past the decimal), instead of actually saying 0.5.  
  
That didn't solve anything though- note that even commenting out the emits didn't actually stop movement. I changed it more for completeness' sake than anything, really.  
  
Still, the fact that stopping the function right after the intial if statement stopped all movement told me that I was at least in a section of code that had something to do with movement. To make sure my understanding of ROS was correct in this case, I commented out  
```
joystick_publisher.publish(joy_msg);
```
  
This stopped the thing from moving, so that worked as expected.  
  
Now for another dumb sanity check: I just threw this in without any if statement: theoretically, I'd just be moving forwards as soon as I switched to manual control.  
```
emit joystickDriveForwardUpdate(0.5); 
```
Still no.  
  
> Side note: iteration time is about 30s. ~15 for build, ~10-15 for sim setup. I'd be more careful about what I built if I spent more time waiting for a compile than planning. This was the other way round- I could compile stupid stuff almost as a way to metaphorically fidget with my pen while thiking.  
   
I wanted to see if I could get any output. I found that emit sendInfoLogMessage() was used a few times, so I placed one such statement at the top of the code block:  
```
emit sendInfoLogMessage("Potato");
```

Right after checking for the publisher. It's not constant, but it happens- it took a bit of testing, and I found that it pretty much logged this every time I gave a controller input. I put the same statement within the first bit of code which I wrote to test whether 'A' was being pressed, but that seemingly did nothing.  
  
Now for another round of 'doing stupid stuff': I tried changing the joy values directly because I was out of ideas.    
```
joy_msg->axes[right_stick_y_axis] = (things)  
```
  
Nope, it's read-only, so that's not how it's changing. Next, I wanted to see if I could at least print the joystick states:  
  
```
emit sendInfoLogMessage(to_string(joy_msg->axes[left_stick_y_axis]));
```  

Didn't compile. **Turns out we actually work with QString and not std::string.** The function to convert double to string is
```
QString::number(double)  
```

Worked for the left joystick just fine.  
  
Okay, taking a step back here. **Press A to potato actually does work, I was just in the wrong tab** (make sure to be in the Info Log).  
  
Now, since those messages clearly have no effect on movement, there's only one other place where the joy axes are even looked at in all the code, which is ROSAdapter.cpp, so I threw this in after linear was declared:
```
if(message->buttons[0])
{
linear = max_motor_cmd;
}
```

And that did it- pressing A now let me move forwards.

**Main takeaways:**  
-everything uses QString, not std string  
-"emit sendInfoLogMessage()" prints to console  
-make sure to switch to the info log tab if you're printing to it  
-not all messages emitted have to have receivers
  
I still don't know  
-What are those joy messages for, then? I'd initially assumed that they sent to some precompiled stuff, hence why I couldn't find any reference to them in the codebase- this doesn't actually make sense in retrospect, but the alternative was thinking that they literally did nothing (which seems to be the case). My guess is that they're for other things to hook onto.


### <a name="p2"></a>Part 2: (unfinished)  
will update later
