# **Finding Lane Lines on the Road** 

This is the first project of Term 1 of Udacity'sSelf-driving Nanodegree program. The main objective of the project is to develop a video processing pipeline that annotates input dashcam video with left and right lane lines.

Some concrete requirements for the project are:
* The output video is an annotated version of the input video.
* In a rough sense, the left and right lane lines are accurately annotated throughout almost all of the video. Annotations can be segmented or solid lines
* Visually, the left and right lane lines are accurately annotated by solid lines throughout most of the video.
* Describe the limitations of the pipeline and discuss possible ways to improve it. 


[image1]: ./examples/grayscale.jpg "Grayscale"

---

## Reflection
The task is deceptively challenging and required considerable trial-and-error, tweaking and background study. 

* __Computer vision/OpenCV__:Basics of Computer vision concepts like bitwise operations, colorspaces, Coordinate systems, Canny-edge detection etc. are essential to enable robust implementation. Understanding OpenCV's bit-level image operations are essential. OpenCV's official docmentation for Python is not very comprehensive. Hence, a lot of trial and error and debugging is necessary.

* __Programming in Python__: Intermediate to advanced knowledge of Python is a necessity for efficient implementation. Constructs like lambdas, list comprehensions, map/filter, function callbacks etc. come in very handy. However, I did not strictly follow the "Pythonic" way of implementation.

* __Media Processing__: The project uses a third party media processing library (MoviePy) that takes time to get familiar with, especially for a Python newbie.

Overall, the Project gives a good introduction into the world of Computer Vision basics, both theory and practice. It is a good stepping stone to more advanced concepts in the field.

## Pipeline description

The video processing pipeline consists of the following steps:

## 1. Detecting yellow and white regions in the input image

The first step in the pipeline is to isolate the white and yellow regions in the input RGB image. This allows the lane lines (which are white and/or yellow in color) to be clearly identified in the input image. The RGB colorspace is not ideal for detecting yellow lines, particularly in low-light/shadowed images. This is because the RGB colorspace is highly sensitive to changes in brightness: hence in images with shadows or bright sunlight, the range for Yellow lanes in the RGB model will vary a lot. For a more in-depth description of this, refer to this [paper](https://www.researchgate.net/publication/220777443_An_Adaptive_Method_for_Lane_Marking_Detection_Based_on_HSI_Color_Model)

Instead we employ the HSL colorspace for detecting both yellow and white regions in the input image. HSL stands for Hue, Saturation and Lightness. The main advantage of HSL (over RGB) is that it makes it easy to select a color quickly and furthermore, it is easier to specify varying levels of lightness and saturation for a particular color. The HSL colorspace is shown below.

The Hue wheel on the left runs from 0-360 degrees and is essentially a color-picker. The distance from the middle of the color wheel is called the ‘Saturation’, or how much of a given hue is present. Looking closely at the colorwheel, it is apparent that the color becomes more pronounced and more vivid as one travels from the center of the circle to the edge. The Lightness value of an HSL color is in a third dimension, which actually makes the HSL system a cylinder, as shown in the right image. Both S and L components are specified as a percentage from 0-100%.

HSL Colorwheel              |  HSL Cylinder
:--------------------------:|:-------------------------:
![HSL Colorwheel](/desc_images/hue-wheel-300x300.jpg)    |     ![HSL Cylinder](/desc_images/hsl-cylinder-300x228.jpg)

It is easy to see that the HSL ranges for white color is:
* White Min: [0, 200, 0]
* White Max: [255, 255, 255]

And the corresponding ranges for yellow color is:
* Yellow Min: [10, 0, 100]
* Yellow Max: [40, 255, 255]

NOTE: The range for the respective HSL ranges are scaled to an 8-bit representation (0-255).

The detected images are shown below:

Original images             |  Yellow and White regions
:--------------------------:|:-------------------------:
![Original_im1](/desc_images/orig_solidWhiteCurve.jpg)  |  ![HSL_yellow_white1](/desc_images/wy_solidWhiteCurve.jpg)
![HSL Original_im2](/desc_images/orig_solidYellowCurve2.jpg)   |   ![HSL Cylinder](/desc_images/wy_solidYellowCurve2.jpg)


## 2. Applying Gaussian blurring

Gaussian blurring blurs sharp gradients and noise in the input image making it easier to detect pronounced edges later in the pipeline. We use a kernel of size 13 in order to ensure robust edge detection. Higher order kernels require increased computation time, which is an important factor for real-time video processing.

Yellow/White images            |  Gaussian blur with a 13x13 kernel 
:--------------------------:|:-------------------------:
![Yellow_white_im1](/desc_images/wy_solidWhiteCurve.jpg)  |  ![Blur_Yellow_white_im1](/desc_images/blur_solidWhiteCurve.jpg)
![Yellow_white_im2](/desc_images/wy_solidYellowCurve2.jpg)   |   ![Blur Yellow_white_im2](/desc_images/blur_solidYellowCurve2.jpg)

## 3. Grayscaling the image

Next, we apply grayscaling on the blurred images, which makes it easier (and faster) to apply the Canny edge-detection algorithm in the following step:

Grayscaled image 1           |    Grayscaled image 2
:--------------------------:|:-------------------------:
![Grayscaled Image 1](/desc_images/gray_solidWhiteCurve.jpg)  |  ![Grayscaled Image 2](/desc_images/gray_solidWhiteCurve.jpg)

## 4. Applying Canny edge detection

To the grayscaled images, we apply the well-known Canny edge detection algorithm with low and high thresholds set according to the pixel intensities in the input image. First, we compute the median single-channel pixel intensity in the input grayscale image: v. The low and high thresholds are respectively v\/3 and 2\*v/3.

Grayscaled images           |    Canny edge detection
:--------------------------:|:-------------------------:
![Grayscaled Image 1](/desc_images/gray_solidWhiteCurve.jpg)  |  ![Canny Image 1](/desc_images/canny_solidWhiteCurve.jpg)
![Grayscaled Image 2](/desc_images/gray_solidYellowCurve2.jpg)  |  ![Canny Image 2](/desc_images/canny_solidYellowCurve2.jpg)

## 5. Masking a region of interest (RoI) to only select potential lane lines

To isolate only the lane lines, we apply a RoI mask that is tailored to the dimensions of the input image:

RoI for isolating lanes     |  Masked lane lines in RoI
:--------------------------:|:-------------------------:
![RoI Image 1](/desc_images/roi_m_solidWhiteCurve.jpg)  |  ![RoI post Image 1](/desc_images/roi_solidWhiteCurve.jpg)
![RoI Image 2](/desc_images/roi_m_solidYellowCurve2.jpg)  |   ![RoI post Image 2](/desc_images/roi_solidYellowCurve2.jpg)

## 6. Applying the Probabilistic Hough Transform for detecting lines

We next apply OpenCV's Probablistic Hough Transform for detecting lane lines in the RoI. The algorithm is implemented by the HoughLinesP function with the following parameters: 

```python
rho = 1 # distance resolution in pixels of the Hough grid
threshold = 20    # minimum number of votes (intersections in Hough grid cell)
min_line_len = 20  # minimum number of pixels making up a line
max_line_gap = 300  # maximum gap in pixels between connectable line segments
```
The Hough Transform outputs a set of line segments ('Hough lines') detected in the input image. The line segments are specified by the x,y coordinates of their endpoints in a list, like so: [x1, y1, x2, y2]. A list consisting of the detected Hough lines is output by the Hough Transform, like so:
```
[[  0 539 959 539]], [[  0 537 959 537]], [[  0 538 959 538]], [[521 342 958 536]], [[524 342 957 535]], [[ 11 536 448 342]], [[  1 536 436 342]], [[  8 536 445 342]], [[517 342 954 536]], [[  2 536 439 342]], [[511 342 948 536]], [[  6 536 442 342]], [[514 342 951 536]], [[527 342 805 466]], [[232 431 432 342]], [[ 15 536 452 342]], [[529 342 686 412]], [[487 342 946 536]], [[558 354 874 536]], [[446 342 551 351]], [[454 342 539 348]], [[291 462 438 343]], [[282 460 425 344]], [[589 368 893 536]], [[520 342 956 536]], [[508 342 930 529]], [[424 344 549 350]], [[102 490 434 342]], [[475 342 540 346]], [[426 345 496 348]], [[281 462 429 342]], [[405 353 498 342]]
```

## 7. Extrapolating the lane lines to the end and start of the road

The Hough lines are fed to a function that draws the final lane lines on the original image. 

This consists of several steps:
* **Group left and right lane lines based on their slope:** Left lane lines have negative slope; right lane lines have positive slope. This fact can be exploited to separate the Hough lines into left lane and right lane candidates.

* **Best-fit line:** For each lane, we compute the best fit line based on a first-order linear regression. Numpy's [`polyfit`](https://docs.scipy.org/doc/numpy/reference/generated/numpy.polyfit.html) function is used for this purpose:

        ```
                z_right = np.polyfit(x_right,y_right,1)
        ```

Here x_right and y_right contain the X coordinates and Y coordinates respectively of the Hough Lines corresponding to the Right lane. consists of a tuple `(m, b)` which describes the parameters of the best-fit line: `y = mx + b`. This is the line to be extrapolated to the start and end of the lane. we identify the coordinates of the endpoints of the extrapolated lane line as shown by the yellow points in the right figure below. Note that the coordinate system for vertices differs from the coordinate system for the image which are read as matrices by OpenCV and follow a row-major indexing.

Coordinate System     |  Lane line extrapolation end-points
:--------------------------:|:-------------------------:
![Coordinate System](/desc_images/coordinate_system.png)  |  ![Lane line extrapolation](/desc_images/line-segments-extrapolation.jpg)

* **Memory filter for fixing spurious lane lines:** Sometimes frames contain lane lines that are incorrectly detected i.e. they have a wrong slope and/or a wrong y-intercept. Hence there is a need to incorporate a memory in the lane annotation that averages the slope `m` and the y-intercept `b` of the past N frames for both left and right lanes. This moving average of the slope is compared with the current best-fit line's slope and if the latter deviates by more than a per-determined threshold, a new best-fit line is computed based on the moving average slope and y-intercept. This minimizes the presence of spurious lane lines.

*If no line is detected by the Hough filter on either the left or right lane, the respective lane is left blank. This occurs only for a handful of frames in the test videos.*

## 8. Superimposing the detected lane lines on the original image

The extrapolated lane lines are superimposed on the original image using alpha-blending:

Final result 1    |  Final result 2
:--------------------------:|:-------------------------:
![Coordinate System](/test_images_output/solidWhiteCurve.jpg)  |  ![Lane line extrapolation](/test_images_output/solidYellowCurve2.jpg)

## Final results (Videos)

[![](http://img.youtube.com/vi/BP8TPvVJabA/0.jpg)](http://www.youtube.com/watch?v=BP8TPvVJabA "")

[![](http://img.youtube.com/vi/IY2XCq3Uq44/0.jpg)](http://www.youtube.com/watch?v=IY2XCq3Uq44 "")

[![](http://img.youtube.com/vi/IeJD3yWnUV0/0.jpg)](http://www.youtube.com/watch?v=IeJD3yWnUV0 "")


## Limitations of the pipeline

* The pipeline is optimized to work on video of resolution 960 x 540 pixels. Other resolutions and particularly different aspect ratios may need tweaking to achieve optimum results.
* The camera orientation cannot be changed: If the camera is placed on the left or right of center, the RoI needs to be adapted accordingly.
* Optimum parameters of the Hough tranform have not been explored. Hence there are a few frames where no Hough lines are detected, leading to no lane annotations.
* The lane detection does not work for curved lanes.
* The top of the lane is currently hard-coded based on the RoI. It does not adapt according to the horizon level, leading to suboptimal extrapolation of the top of the lane.
* ~~Spurious lane lines are not eliminated: If a line with a very different slope as before is detected, it will not be filtered out as long as it is in the RoI.~~


## Possible improvements to the implementation

* The pipeline can be generalized to work with any camera placement and any video resolution by detecting the orientation and adjusting the RoI accordingly.
* The detection rate of white dashed lines, although very high, can be further improved.
* Polyfitting the detected Hough lines could use higher-order regression to accomodate curving lanes.
* The extrapolation to the top of the lane can be dynamic based on the detected horizon level.
