---
layout: post
title: "Detecting lines on technical drawings"
description: ""
category: 
tags: [Programming]
image: /assets/images/line-detection/6x6.png
nocover: true
---
{% include JB/setup %}

Over the past few months I've been working on a program to convert images of technical drawings into digital formats.

In other words, the program will convert an image like this.

![](/assets/images/line-detection/input.png)

Into a digital format that can be used in a CAD program.

![Kicad](/assets/images/line-detection/kicad.png)

One of the basic necessities of such a program is that it needs to detect lines.  I've been wanting to write a little about this.

# Line Detection with OpenCV

Initially I thought about using some of the powerful OpenCV methods for detecting lines.  OpenCV edge detection and Hough transforms
are commonly used to detect lines.  They would be great for detecting lines like the traffic lines in the following image.

[<img src="/assets/images/line-detection/car_lines.jpeg">](https://medium.com/@galen.ballew/opencv-lanedetection-419361364fc0)

I tried using this method for extracting lines from my drawing images but I don't think it works well enough.

Initially, it seems to work well.

![](/assets/images/line-detection/opencv_method.png)

<!--Blue lines representing the detected lines are drawn over the image.-->

But on closer inspection, it has some issues.

Here it is missing some lines that I wish it detected.

![](/assets/images/line-detection/missing_lines.png)

Here it detected some lines that I wish it didn't.

![](/assets/images/line-detection/ocr_lines.png)

I tried experimenting with the various parameters for edge detection and the
Hough-transform line detection.  I was able to make it better but the
amount of missing lines and false positives are always significant.

I could try various amounts of post-processing and probably still make it work for my application,
but I don't think it is a good solution.  I think are much easier methods that can take advantage 
of the properties of my input images.  

For one, most technical drawing images are all just black lines and shapes
with a white background.  This separates them from most camera captured images, which are more suited for OpenCV's prowess.


# Better line detection

Let's just focus on horizontal and vertical lines.

Remember an image is just an array of numbers.  In my case, it is a black and
white 2D image.  With all the pixels basically being either 255 (white) or 0
(black).

Here is a sample image.  There are two lines and
one happens to be twice as wide as the other.  I want to detect exactly two lines
accurately.

![](/assets/images/line-detection/sample_lines.png)


<!--255 255 255 255 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255 255 255 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255 255 255 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255   0 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255   0   0   0 255 255 255 255 255 255 255-->
<!--255 255   0   0   0   0   0   0   0   0   0   0   0   0   0   0-->
<!--255 255   0   0 255 255   0   0   0 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255   0 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->
<!--255 255   0   0 255 255 255 255 255 255 255 255 255 255 255 255-->


How would you do it?

.

.

.

.

I took a simple signal processing approach.  Sum the rows and columns and look for spikes.

First invert the image.

![](/assets/images/line-detection/invert.png)

  <!--0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0   0   0   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0 255   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0 255 255 255   0   0   0   0   0   0   0-->
  <!--0   0 255 255 255 255 255 255 255 255 255 255 255 255 255 255-->
  <!--0   0 255 255   0   0 255 255 255   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0 255   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->
  <!--0   0 255 255   0   0   0   0   0   0   0   0   0   0   0   0-->

<!--(the actual image has a lot more pixels)-->

Sum the columns.

![](/assets/images/line-detection/sum_cols.png)

Sum the rows.

![](/assets/images/line-detection/sum_rows.png)

Detecting the lines then becomes as simple as setting a threshold.  

I wouldn't stop there though.
Input images can get pretty busy with a lot of overlapping lines and shapes.  It might not be clear what 
threshold to use.  Check out the row summation of a more realistic input image.

![](/assets/images/line-detection/input.png)

![](/assets/images/line-detection/input_row_sum.png)

It is really easy to detect the larger lines but the smaller lines won't get picked up by a threshold because of the "noise floor."

So as an extra step the data should be run through a high-pass filter to remove the low frequency "noise."

```python
def butter_highpass(cutoff, fs, order=5):
    nyq = 0.5 * fs
    normal_cutoff = cutoff / nyq
    b, a = signal.butter(order, normal_cutoff, btype='high', analog=False)
    return b, a

def butter_highpass_filter(data, cutoff, fs, order=5):
    b, a = butter_highpass(cutoff, fs, order=order)
    y = signal.filtfilt(b, a, data)
    return y


# im is input image in numpy array

y1 = np.sum(im == 0,axis=0)             # row sum
y1f = butter_highpass_filter(y1,1,25)   # put through high pass filter


plotly.offline.plot([Scatter(y=y1)])    # I'm using Plotly to output graphs
plotly.offline.plot([Scatter(y=y1f)])
```

![](/assets/images/line-detection/highpass.png)

The highpass data is overlaid in orange.  This is a safer data-set to use for threshold detection.

You might be thinking that the lines aren't really detected all the way.  We just figured out one dimension (X or Y)
for each line.  We don't know where each line starts or how long each line is.

Here are the so called detected horizontal and vertical lines.

![](/assets/images/line-detection/horz.png)

![](/assets/images/line-detection/verz.png)

But figuring out where the lines start and end is simply a matter of tracing each red line and looking for black pixels.

# Detecting non-vertical, non-horizontal lines

For lines with slopes that aren't 0 or infinity, my previous method won't work.  I still don't like using
OpenCV edge detection and Hough transform for this.

I could do something like rotate the image 45 degrees and detect lines with slope of 1.  That might be enough for the types of images 
I'm working with.

Maybe there's a better way.  If you think of something, let me know and I'll owe you a beer.

{% include JB/mytwitter %}
