# **Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[grayscale]: ./pipeline/grayscale.jpg "Grayscale"
[gaussian]: ./pipeline/gaussian.jpg "Gaussian Blur"
[canny]: ./pipeline/canny.jpg "Canny Edge Detection"
[clipped]: ./pipeline/clipped.jpg "Region of Interest"
[lines]: ./pipeline/lines.jpg "Hough Lines"
[extrapolated]: ./pipeline/extrapolated.jpg "Extrapolated lines"
[result]: ./pipeline/result.jpg "Final result"

---

## Pipeline description

My pipeline consisted of 7 steps. It goes like this:

### 1. Convert image to grayscale

Convert the image to grayscale for further processing. Everyting is simpler with one channel!

![grayscale]

### 2. Apply Gaussian filter

The goal of this operation is reducing the influence of random noise in the image. This in turn decreases number of false-positives in the next step.

![gaussian]

### 3. Find edges with Canny

Perform a Canny edge detection to... well, find edges in the image. This step took some parameter tweaking to balance the number of edges detected. Too few means we lose information about the lane lines, too many means we get too much noise and the information is there, but is not extractable.

![canny]

### 4. Limit the edges to pre-defined region of interest

One trick I used to discard unwanted lines was defining a triangular region of interest in the lower part of the picture. The result is a clipped image, as shown below.

![clipped]

### 5. Detect lines using Hough transform

In this step, I used Hough transform to find lines from binary edge image. I also modified the `hough_lines` function to return the list of lines for further processing (instead of just an image with the lines already drawn) and `draw_lines` not to take input image and to always start drawing from scratch instead. This way I could more easily reuse it later.

![lines]

### 6. Extrapolate the lines

First, the lines are sorted based on their angle. Certain range of angles is interpreted as the left line, and certain as the right. What's important is that the ranges **are not complementary**. It means that lines with certain angles are not taken into account at all. The reason behind it is simple - with camera facing front of the car, all *interesting* lines are sloped. If a line is close to vertical or horizontal, it must come for example from a passing car or a shadow.

After carefully aggregating the lines into two sets, they are approximated using linear regression and drawn from bottom to slightly-under-the-middle of the image.

![extrapolated]

### 7. Overlay the result on the initial image

At the end of the pipeline we combine the output of the previous step with the initial image to achieve the following final effect:

![result]

## Potential shortcomings

Even though the pipeline works fine in a typical scenario, it might not handle some edge cases well enough. This is especially visible in the last (*optional challenge*) video, when the lines wander randomly on the screen.

In other videos, even though the lines' positions are correctly estimated, they still appear a bit shaky. The reason for this is twofold:
- at some times, an edge that does not belong to lane lines manages to sneak through the pipeline, causing the result to be a bit off,
- at other times, even if the lines are correctly detected, the result still may be skewed due to insufficient spread of points thrown into linear regression. A lane line has certain width and if we consider only a small area of points on both sides, but not on its full length, the fitted line might be in the wrong direction.

## Possible improvements

There are a lot of ways the pipeline could be improved. The simplest one is to play a bit more with the parameters to do edge and line detection as good as possible. However, in my opinion improvement in this area could only be a slight one.

Another way is to reduce the region of interest. At the moment it is chosen very conservatively as a triangle with nodes at two lower corners and the middle of the image. Narrowing this region could remove some of the unwanted artifacts.

The third idea is to enhance the extrapolation algorithm. First of all, other method than linear regression could be used. For example, one could calculate slope of the result line as a weighted average of its components, where weight would be the (squared) lenght of the section. A Kalman filter could also be used to smooth out the lines' movement between frames, resulting in a more stable image.
