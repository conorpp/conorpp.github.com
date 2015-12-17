---
layout: post
title: "How to accelerate a program using hardware"
description: ""
category: 
tags: [Design]
image: /assets/images/codesign/fpga.jpg
---
{% include JB/setup %}

The Challenge
==============

One of my recent final projects in school was to take a given software program and make it 
as fast as possible while still being functionally correct.  It was graded
based on speed and how it ranked against each solution in the class.

Here I will show you how to incrementally find bottlenecks in software and replace them with hardware components.

The Specification
==================

For those not familiar with what an field programmable gate array (FPGA) is,
it's a chip made up of various programmable memories, logic cells, look up tables, 
and interconnect that can be configured to be functionally equivalent to near any digital
design that will fit on it.  

I could make a MIPS processor connected to a custom accelerated HDMI controller.  Then I could reprogram it to be a simple counter circuit.  This is quite useful 
for digital design.

For the challenge to be fair, it had to run on an [Altera Cyclone IV FPGA](https://www.altera.com/products/fpga/cyclone-series/cyclone-iv/overview.tablet.html).
It had to be timed with a 50 MHz timer (also configured onto the FPGA) to get a consistent benchmark.
And it had to be functionally correct.  That's it.

The Program
===========

We had to implement a program that could draw 50 randomly generated circles on a shared memory.
We were given a reference C design that ran on a RISC 32 bit processor.  It completes in **14,892,988 cycles**.

Here is the reference function for drawing *one* circle.  Don't try to understand the functionally - try to find areas where the processor
would spend most of its time (bottlenecks).

```c
// This is called 50 times
void plotcircle(unsigned cx, unsigned cy, unsigned r) {
    int x, y;
    int xp;
    x   = r;
    y   = 0;
    xp  = 1 - (int) r;
    while (x >= y) {
        setpixel(cx + x, cy + y);
        setpixel(cx + x, cy - y);
        setpixel(cx - x, cy + y);
        setpixel(cx - x, cy - y);
        setpixel(cx + y, cy + x);
        setpixel(cx + y, cy - x);
        setpixel(cx - y, cy + x);
        setpixel(cx - y, cy - x);
        y   = y + 1;
        if (xp < 0)
            xp += (2*y + 1);
        else {
            x = x - 1;
            xp += 2*(y-x) + 1;
        }
    }
}

```
This reference implements the [Bresenham circle drawing algorithm](http://web.engr.oregonstate.edu/~sllu/bcircle.pdf) [PDF].

Since we can accelerate using hardware, here is diagram of the reference architecture.  

![](/assets/images/codesign/reference.svg)

A simple processor runs the C code and connects to program memory (for .bss, .text, stack, etc.)
and a separate memory for drawing the circles.  It runs on a 50 MHz clock and is timed by a 50 MHz timer.

Accelerating
============

Do you have any ideas?  Here is what I thought of.

### setpixel(...)

My first thought that a bottleneck would be in the `setpixel(...)` function.  It's called *eight* times for each iteration of the loop.
And don't forget to multiply that by 50 because there's 50 circles to plot.  So perhaps by accelerating this small part of the design we could get a large speed up.

So let's see how we can call this function only once per iteration of the loop.  We can write a small coprocessor in verilog to do these 8 writes really fast and in parallel
with this calculation:

```c
y   = y + 1;
if (xp < 0)
xp += (2*y + 1);
else {
    x = x - 1;
    xp += 2*(y-x) + 1;
}
```

Since the coprocessor is in hardware, it can easily do the `cx+x`, `cy+y`, `cy - y`, `...` calculations simultaneously and fast enough to get 8 writes in 8 clock cycles.

Now here is the new architecture:

![](/assets/images/codesign/i1.svg)

The coprocessor sits between the memory and the main processor transparently and acts just like a memory as far as the main processor can tell.  But the coprocessor is doing 8 writes to the pixel memory for each write
from the main processor to achieve the acceleration.  If you're interested, here is [semi-correct verilog](https://github.com/conorpp/codesign-challenge-2015/blob/master/setpixel-coprocessor/verilog/plot_circle.v) I pulled from my git history.

Here is the new C code to interface with the coprocessor:

```c
#define SET_RADIUS(cx,cy,r) ((*(volatile unsigned *) PIX_MEM) = ((r<<18)|(cy<<9)|cx)
#define SET_PIXELS(x,y) ((*(volatile unsigned *) PIX_MEM+4) = (y<<9)|x)

// called 50 times
void plotcircle_hw(int circle, unsigned cx, unsigned cy, unsigned r) {
    int x, y;
    int xp;
    SET_RADIUS(cx,cy,r);
    x   = r;
    y   = 0;
    xp  = 1 - (int) r;
    int i = 0;
    while (x >= y) {
        SET_PIXELS(x,y);    // acceleration
        y   = y + 1;
        if (xp < 0)
            xp += (2*y + 1);
        else {
            x = x - 1;
            xp += 2*(y-x) + 1;
        }
    }
}
```

The coprocessor is mapped to the location defined by `PIX_MEM` so that is where it writes to.  The design is now doing only 1 write (`SET_PIXELS`) and 16 less arithmetic calculations per iteration of the loop.
After running the new design, I found there is about a **40x speedup** over the reference.  Awesome.  Now how can we make it faster?

### Drawing Circles in Parallel

Despite the 40x times speedup, it is still drawing 50 circles sequentially.  If it could draw all of them at the same time, then it could ideally expect a *50 * 40 = 2000x* speedup right?  But this
can be difficult because it's writing to a *shared* memory.  It can't do 50 writes at the same time to one memory.

As far as I know, 50 port memories cannot synthesize for the FPGA.  But we can write to 
50 *different* memories at the same time!  It's space intensive, but we are designing purely for speed.  

The idea here is we can make an individual memory for each circle to be plotted on.
And when reads occur, the coprocessor will read from *all* memories simultaneously and OR them together.

That way, the main processor still thinks it's a regular memory with 50 circles plotted on it.

Here is the new architecture (and [coprocessor verilog](https://github.com/conorpp/codesign-challenge-2015/blob/master/circle-modules-in-parallel/verilog/plot_circle.v) for those that are interested):

![](/assets/images/codesign/i2.svg)

Note that this design also implements the rest of the circle drawing algorithm in hardware.  All it needs is the center point and radius to start drawing a circle.
Here is the little new C code:

```c
#define SET_RADIUS(c,x,y,r) (*(volatile unsigned *) PIX_MEM = ((c<<26)|(r<<18)|(y<<9)|x))

// called 50 times
void plotcircle_hw(int circle_index, unsigned cx, unsigned cy, unsigned r) {
        SET_RADIUS(circle_index,cx,cy,r);
}
```
It writes to the memory mapped coprocessor the circle index (1 to 50), center point, and radius.

Now what is the speedup?  

Unfortunately this does not yield the ideal extra 50x speedup.  First off, there wasn't enough room on the board for 50 individual memories - there was only room for 12.

Second, after calling `plotcircle_hw(...)` 50 times, we need a small delay.  This is to wait for the hardware processors finish plotting.  

After testing for correctness, the design achieves a **600x total speedup**!  That's pretty sweet, but there's still a lot more we can do!  

### Read from program memory in hardware

Let's take a look at the current system for the bottleneck.  What is the main processor doing?  It's calling `SET_RADIUS(...)` 50 times.  That's 50 writes each with
an assortment of shift's and or's to put the arguments into a word.  

```c
#define SET_RADIUS(c,x,y,r) (*(volatile unsigned *) PIX_MEM = ((c<<26)|(r<<18)|(y<<9)|x))
```

We can do this in hardware instead.  Let's add a dual port on the program memory to give the coprocessor direct access to it.  Also, since we're delaying for the hardware modules 
to finish, let's increase their clock inputs to 100 MHz!

![](/assets/images/codesign/i3.svg)
[Coprocessor verilog](https://github.com/conorpp/codesign-challenge-2015/blob/master/final-design/codesign_challenge/verilog/plot_circle.v)

Now here is the minimal C code:

```c
#define PIX_BUSY (*(volatile unsigned *)PIX_MEM+4)
#define PIX_WSTART (*(volatile unsigned *)PIX_MEM)

// called once
void plotallcircles_hw() {
    PIX_WSTART = (unsigned)stack_address_of_global_circle_data_array; 
    while(PIX_BUSY);
}
```

It writes the stack address of where the generated circle coordinates and radii are located.  Now the coprocessor will do the reads on its own.   The C program reads from `PIX_BUSY` which is a control signal to indicate when all circles finish plotting.
Also note that function is only called *once* instead of 50 times.

The speedup was now about **5000x more than the reference design**.

In the reference, it did 8 writes and about 20-24 arithmetic instructions for each iteration of a loop that varied between 0-202 iterations.  It did all of that 50 times.

Now in the fully accelerated design, it just does one write and polls the hardware for when it's done.  The 5000x speed up is expected.


Conclusion
===========

Hardware acceleration is an iterative process and warrants a good understanding of the overall architecture and its bottlenecks.
Plus it's not always required to get as much speed as possible.  The final design here would be expensive to implement in practice.  It could
be acceptable to stop at any of the iterations above for many projects.

This was a fun project and improving the performance of a program is a satisfying experience.  If you got any ideas for how to make it even faster be sure to let me know.

{% include JB/mytwitter %}
