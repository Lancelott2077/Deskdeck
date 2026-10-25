# This markdown file acts as my learning log

Notes per milestone: what I predicted, what actually happened, and what
I took away from it.

## M0 — Repo setup
We created a github repo for our project and created .gitignore file, this file is created to ensure that the build 
output doesnt get commited as the build output changes frequently and is not necessarily needed by other devs,

the difference between master and main?
we have created a remote main branch using git init -b main, why main cause this commands sets the main branch as 
intial branch as master is initial branch for git and not github, so to avoid creating 2 diff branches. we have 
used the command and then we are pushing everything to the repo createat github online

## M1 — GPIO: LED and buttons

we use HAL functions which we referenced from hal include folder and wrote the functions, we have referred the main.h 
and other includes files to check for function prototypes.
=======
while opening the cubeide, i have come across the launch workspace pop-up i thought of this to be the same folder as 
my project and then tried importing my project but it failed to do so, then i realised that the terms workspace and 
project folder where fundamentally 2 diff things , the ide asks for workspace to create metadata and it must always be
noted that the workspace and project folder must be in two different folders.

## M2 — Timer interrupts and debouncing
Here's a draft for your log. Reword it in your own voice before saving — notes you've rephrased yourself stick better than pasted ones.

---

## M2 — Button debounce and timer interrupts

What bouncing is ?
A push button is two metal contacts that spring together when pressed. They don't meet cleanly — they physically bounce apart and back several times over about 1 to 20 milliseconds before settling. So one press produces a signal that flips between 1 and 0 a few times before it stays at 0. The chip reads every one of those flips as real, because they are real.

The second, separate problem:
Even if the button were perfect, there's another issue. The main loop runs about a million times a second, and a press lasts around 100 milliseconds. So code that checks "is the button down right now" sees "yes" tens of thousands of times for a single press. I want to know the moment the button *changes* from up to down, not that it is currently down.

Why D1 worked but D2 didn't?
D1 said: if the button is down, turn the LED on; otherwise turn it off. Doing that ten thousand times does the same thing as doing it once, because writing "on" to an LED that's already on changes nothing. The answer only depends on what the pin reads right now.

D2 said: if the button is down, flip the LED. Flipping depends on what it was before. Do it an even number of times and nothing changed; an odd number and it flipped. Since I can't control how many thousands of times it ran during my press, the result was random.

So D1 wasn't better written — it was just the one case where the flaw didn't show.

Why not just use HAL_Delay?
A delay would hide both problems, but it stops the CPU completely. Nothing else runs while it waits. My finished device has to watch buttons, count down a timer, and redraw a screen at the same time, and a device that goes deaf for 20 milliseconds every press can't do that.

How the timer fixes it?
The chip has a hardware counter that runs on its own, separate from my code. I set it to raise an interrupt every 1 millisecond. When it fires, the CPU pauses, runs a short function of mine, and goes back to exactly what it was doing. The main loop never waits for anything.

Important: the timer doesn't time the button. It just ticks every 1 ms, forever, knowing nothing about buttons. My code does the counting.

On each tick, for each button, I read the pin once. If it reads the same as last time, I add one to a counter. If it reads differently, I reset the counter to zero. When the counter reaches 20, the reading has been steady for 20 milliseconds and I accept it as the real state. Any bounce resets the count, so flicker can never get through.

Then, if the newly accepted state is "down" and the previous accepted state was "up", that's one press, and I set a flag.

The pattern:
The interrupt notices things and sets flags. The main loop reads the flags and does the actual work. Keeping the interrupt short matters, because while it runs, nothing else can.

The volatile keyword:
A variable written inside the interrupt and read in the main loop has to be marked `volatile`. Without it the compiler assumes nothing in the loop changes that variable, keeps a copy in a register, and never looks at memory again — so the main loop never notices the interrupt changed it. The symptom looks exactly like the interrupt not firing at all.


## M3 — UART

## M4 — USB HID

## M5 — Pomodoro state machine

## M6 — Java companion app and screenshot macro
