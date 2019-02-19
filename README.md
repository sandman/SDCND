# **Finding Lane Lines on the Road** 

This is the first project of Term 1 of Udacity'sSelf-driving Nanodegree program. The main objective of the project is to develop a video processing pipeline that annotates input dashcam video with left and right lane lines.

Some concrete requirements for the project are:
* The output video is an annotated version of the input video.
* In a rough sense, the left and right lane lines are accurately annotated throughout almost all of the video. Annotations can be segmented or solid lines
* Visually, the left and right lane lines are accurately annotated by solid lines throughout most of the video.
* Describe the limitations of the pipeline and discuss possible ways to improve it. 



[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Pipeline description

The video processing pipeline consists of the following steps:
1. Detecting yellow and white regions in the input image
1. Applying Gaussian blurring
1. Grayscaling the image
1. Applying Canny edge detection
1. Masking a region of interest (RoI) to only select potential lane lines
1. Applying the Probabilistic Hough Transform for detecting lines
1. Extrapolating the lane lines to the end and start of the road
1. Superimposing the detected lane lines on the original image

![alt text][image1]


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...
