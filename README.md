# USB-Throttle-Quadrant
Designing Open-Source Flight Sim Hardware
___

I'm an avid flight simulator enthusiast. For as long as I can remember, I've been flying planes on the PC... everything from single-engine Cessnas, rotorcraft, fighter aircraft, and the largest of passenger airliners. Unfortunately, my options for *controlling* these aircraft have always been marginal at best.

My first joystick was a Microsoft Sidewinder Standard joystick from 1995. Coupled with Microsoft Flight Simulator 98, it proved to be loads of fun for 5-year-old me.

![Sidewinder Standard](https://i.imgur.com/wMMwUwJ.png)

Unfortunately, at the time, I couldn't figure out how to make the throttle wheels on either side of the joystick work, so I had to resort to adjusting engine power with the keyboard function keys. Needless to say this made landing difficult (or impossible for a 5-year-old) and there was virtually no hope that any aircraft I piloted would ever come down in one piece.

My next joystick was a Logitech Extreme 3D Pro. This thing was amazing compared to the Sidewinder! It was USB, had what seemed like infinite buttons, and... a working throttle!

![Extreme 3D Pro](https://i.imgur.com/AIJzeve.png)

I loved this thing so much. Coupled with Microsoft Flight Simulator 2000, landing was not only possible... it was sometimes survivable! Buzzing around Chicago in a British Airways Concorde was so much fun. I could control my view, put on the brakes, even lower the landing gear without even lifting my hand from the stick! But it just wasn't enough.

After upgrading to X-Plane 9, I realized there was a whole world of realism and types of aircraft out there for me to explore. Starting engines, lowering flaps, deploying speedbrakes, transitioning to forward flight in VTOL aircraft, all required levels of control I just couldn't get with my trusty old logitech joystick. Enter, the Thrustmaster T. Flight Stick X

![T Flight](https://i.imgur.com/Selp5oA.png)

This is actually my current joystick, as of writing this. The ergonomics are far superior to the Logitech, allowing much more precise approaches. I can trim my aircraft very easily with the 6-way hat switch, and just as easily dump my trim with the other buttons on the control stick. Flaps, speedbrakes, reverse thrust, engine start, and view are all mapped to the buttons on the base. The throttle works, don't get me wrong... but there's an annoying clicky detent when you cross between the red and white zones that really inhibits smooth movement. Additionally, with only one throttle lever,  I can't use variable thrust in multi-engine aircraft. This is annoying when making tight turns in a King Air C90 or Boeing 737. I needed... more.

![Throttle Quadrant](https://i.imgur.com/5UoaR2E.png)

But $60 seems a bit steep for 3 wimpy throttle axis! I want something meatier, like what you find in an airliner.

![Expensive Throttle Quadrant](https://i.imgur.com/T1TQZ73.png)

Holy cow! I wanted something that you would find ***in*** an airliner, not something that costs as much ***as*** an airliner!

I should just make my own at that point.

And, so, I decided I would! I had an idea in my head of what I wanted. Two independent throttles, with spring-loaded reverse-thrust that would engage when you pulled them back past idle. That way, when you "hold" the aircraft in reverse to slow down, you could return the engines to idle by just letting go. Simple!

Spending some time at the computer led me to come up with this design:

[Animation](https://i.imgur.com/kygk9N8.mp4)

Which I was able to almost entirely 3D print, save for the skateboard bearings, screws, and potentiometers.

![First Rev](https://i.imgur.com/3raPrWm.jpg)

Unfortunately, 3D printing isn't perfect, and neither was my design. The screws that were supposed to follow the "tracks" on either side kept coming out, and very quickly wore out the plastic from the friction.

![Worn out](https://i.imgur.com/8Q9QJcO.jpg)

I needed a much more robust design if I wanted this to feel anywhere close to usable. I started looking at different springs I could use that offered more resistance than my tiny extension springs.

Eventually, I came across this avenue on Amazon:

![RC Shocks](https://i.imgur.com/9Hl68d3.png)

RC Car shock absorbers! They're the perfect size, look like little gas cylinders, and have bearings built in on either end! 33mm of travel is plenty for my needs, so I decided to pull the trigger and see if they'd work.

After they arrived, I set to work designing an extremely accurate 3D representation of them to use in my design process.

![3D Shocks](https://i.imgur.com/OjDqCYl.jpg)

Next, I modified my first throttle design to incorporate these shocks. I ended up going with this:

[Animation](https://i.imgur.com/Ubx0nK9.mp4)

For this project, I decided to forego designing a custom PCB and to try using what I had lying around instead. A Universal PCB was perfect for mounting the potentiometers to, and a Blue Pill was perfect for the main brain.

![PCB](https://i.imgur.com/h7oVXXh.jpg)

![Interior PCB](https://i.imgur.com/82xGpqp.jpg)

The cool part was that the cylinders and the secondary bearings for the  "arm" were all contained within the throttle's housing, so I was able to retain the single-bearing look from the outside.

![Interior 1](https://i.imgur.com/BbB49ws.jpg)

![Interior 2](https://i.imgur.com/ujlzheo.jpg)

![Exterior 1](https://i.imgur.com/ADpr3Fi.jpg)

Pretty clean look from the outside, if you ask me!

After putting it together, I started writing the firmware for the STM32F103C8T6. By now, I've gotten pretty familiar with USB descriptors and the STM32's USB peripheral. Specifying 4 joystick axis, however, was more difficult than I figured it should be. Initially, I defined four distinct axis of the "throttle" class (two for throttle, two for reverse thrust) compliant with the USB Human Interface Device Descriptor spec. But when I plugged it in to my PC, Windows only recognized 3 of the 4 throttle axis! I double-checked my descriptor, and sure enough, everything looked fine. Windows was just ignoring one of my throttles. My immediate question was "okay, which one is it ignoring?" So I re-classified the 4 throttles to different things like "accelerator, clutch, brake, and steering". Interestingly enough, the "brake" axis was missing when I plugged it back in to my PC! "Why is Windows just totally ignoring the third axis?!" After playing around some more, I discovered that it wasn't always the third axis it ignored. As far as I could tell, there was no rhyme or reason as to which of the 4 axis Windows chose to ignore. It appears to me that Windows was only capable of recognizing up to 3 axis that are part of a "simulation controls" usage page. It's possible that there's a fundamental flaw in how I was describing these axis to Windows, but I was able to confirm that using 4 (mostly) arbitrary generic joystick axis names like "Z, X (rot), Y (rot), and Z (rot)" appeared in Windows just fine. Ultimately, how these axis are presented to the operating system is of little consequence; I just cared about assigning all 4 of them to the appropriate functions in X-Plane.

Something to note at this point is how I was able to use two potentimeters to control 4 separate joystick axis. On each lever, there are 3 defined "detents": Maximum Thrust, Idle Thrust, and Maximum Reverse Thrust. Since Idle Thrust is always a known number of degrees "behind" Maximum Thrust, using Maximum Thrust as a reference point I could determine in my firmware when I was going to feel the Idle Thrust position detent as I pulled the levers back. The moment I pulled the levers back *beyond* idle thrust, then the firmware could increase the Reverse Thrust amount for that particular lever. Maximum Reverse Thrust is always a known number of degrees "beyond" Idle Thrust, so it was very straightforward to determine the value for that axis.

Ultimately, I ended up with a pretty great homebuilt peripheral for the bulk of my flight simulator aircraft.

Here's a video of me using the finished unit to land a Boeing 737-800 at Boston Logan International Airport:

[Video](https://i.imgur.com/aLXppK2.mp4)
