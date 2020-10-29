**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
*
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

I started by Preparing Objpoints and ImagePoints as below:

* Grayscale the image
* Find Chessboard Corners. It returns two values ret,corners. ret stores whether the corners were returned or not
* If the corners were found, append corners to image points.
* I have also drawn the chessboard corners to visualize the corners

The code for this step is contained under sections "Camera Calibration" , "Calculate Undistortion Parameters" and "Undistort Images" of the IPython notebook located in "./AdvancedLaneFinding.ipynb".  

With this step we will be able to get image points and object points which will be required to calculate the camera calibration and distortion coefficients.

We call the calibrateCamera function which returns us a bunch of parameters, but the ones we are interested are the camera matrix (mtx) and distortion coefficient (dist).

We then use the distortion coefficient to undistort our image. 


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the camera matrix and distortion coefficient calculated above we can undistort our test images.

![Undistorted Images][./writeupImages/Camera_calibration.JPG]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

(Under sections "Exploring different Color Spaces", "Exploring Sobel" and "Combine Sobel and Color Space" of the IPython notebook located in "./AdvancedLaneFinding.ipynb".)

1) I tried a various number of colorspaces as below to get a good binary image in all lighting conditions:
* HLS
* HSV
* LAB
* YUV
* YCrCb

I defined a function "ExtractChannel" to extract particular channel from color space.

Input given:
* image - the warped image from which we need to extract
* colorspace - the cv2 colorspace. Ex- cv2.COLOR_RGB2HSV
* threshold - the threshold value of pixels which need to be selected in order to get the binary image. [min_threshold, max_threshold]
* channel - the channel we need to extract from the image

Output: Binary Image with the required channel and threshold values applied

Here's an example of my output for this step.
![Exploring Color Spcaes][./writeupImages/Color_spaces.JPG]


2) I defined a common function "Sobel" to apply sobel.

Input:
* warpedimage - the original warped image
* threshold - the threshold that is to be applied to select the pixel values
* sobelType - the direction where we need to take the gradient. values- x- for x gradient , y- for y gradient, xy for absolute and dir for direction
* kernelSize - the size of the kernel

Output: Binary Image with the required thresholds, sobelType and kernelSize

Here's an example of my output for this step.
![Exploring Sobel][./writeupImages/Sobel.JPG]


3) Combining Color Space and Sobel:

I decided to use Saturation channel of HLS because it works sort of well under all conditions. But it was not able to generate lines for dotted white lines. I observed that the Lightness channel HLS works well in all the conditions except the case when the image is too bright. I decided to use and of both Saturation and Lightness Channel. But I was not even happy with that as some faint edges were still not detected so I decided to use another luminance channel, Y channel from YUV colorspace. I could see clear vertical edges using the x gradient, so I decided to use X gradient only.

Final Combination:
Channel 1 = Saturation Channel and Lightness Channel from HLS and Y channel from YUV
Final Combination = Channel 1 or Sobel Gradient in X direction

Here's an example of my output for this step.
![Combining Color space and Sobel][./writeupImages/combined.JPG]



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is under section "Region of Interest and Warping" of the IPython notebook located in "./AdvancedLaneFinding.ipynb"

In this step, I first defined a Region Of Interest (ROI) i.e. a Trapezoid with four points:
Left Bottom Corner defined as "left"
Right Bottom Corner defined as "right"
Left Upper Corner defined as "apex_left"
Right Upper Corner defined as "apex_right"

After defining the ROI, the next step is to warp the image, to see the image from bird's eye perspective. To do this we need to calculate a Matrix with the source and destination points. The destination points were selected appropriately so as to see a good bird's eye perspective. The selection of these points were based on hit an trial mechanism only.

Once we get the Matrix we will that along with Image to CV2 warpPerspective function to get the final warped image.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Here's an example of my output for this step.
![ROI and Warping][./writeupImages/ROIandWarping.JPG]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The first step is to create a Histogram of lower half of the image. With this way we are able to find out a distinction between the left lane pixels and right lane pixels. The code for Histogram is under section "Plotting Histogram" of the IPython notebook located in "./AdvancedLaneFinding.ipynb"

Here's an example of my output for this step.
![Histogram][./writeupImages/Histogram.JPG]

The next step is to initiate a Sliding Window Search in the left and right parts which we got from the histogram. Steps are:
1. The left and right base points are calculated from the histogram
2. Calculate the position of all non zero x and non zero y pixels.
3. Start iterating over the windows where we start from points calculate in point 1.
4. Identify the non zero pixels in the window we just defined
5. Collect all the indices in the list and decide the center of next window using these points
6. Seperate the points to left and right positions
7. Fit a second degree polynomial using np.polyfit and point calculate in step 6.

The code for my perspective transform is under sections "Finding Lane Lines with Sliding Window" , "Draw Sliding Window for visualisation" and "Finding Lane Line from prior frame for smooth visualization" of the IPython notebook located in "./AdvancedLaneFinding.ipynb".

Here's an example of my output for this step.
![Sliding Window][./writeupImages/SlidingWIndow.JPG]
![Smooth Visualiztion][./writeupImages/SmoothVisualization.JPG]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for my perspective transform is under section "Calculate Radius of Curvature and Distance" of the IPython notebook located in "./AdvancedLaneFinding.ipynb".

To calculate Radius of Curvature:
1. First we define values to convert pixels to meters
2. Plot the left and right lines
3. Calculate the curvature from left and right lanes seperately
4. Return mean of values calculated in step 3.

For Position of Car w.r.t centre: We know that the center of image is the center of the car. To calculate the deviation from the center, we can observe the pixel positions in the left lane and the right lane. We take the mean of the left bottom most point of the left lane and right bottom most point of the right lane and then subtract it from the center of the car to get the deviation from the center.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for my perspective transform is under section "Unwarp the Warped Image and show plotted Lanes" of the IPython notebook located in "./AdvancedLaneFinding.ipynb".

Steps:
1. Recast the x and y point to give as input in cv2.fillPoly. These are the same points we got from fitting the lines.
2. Calculate the Minv which is Inverse Matrix. This is done by passing the reverse points this time to getPerspectiveTransform function
3. Draw the sidelines from the points selected in step 1 onto a blank warped image
4. Unwarp the image using cv2.warpPerspective.
5. Combine the original image with the image we got from step 4 to plot the lane lines.

Here's an example of my output for this step.
![Plot Lines][./writeupImages/unWarpandPlotLines.JPG]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The first problem is to find the correct source and destination points. It is a hit and trial approach and even 5 pixels up and down can make a big impact. 

The second problem is to analyse various combinations of color channels and find the final combination to work well in almost all conditions. It was again by hit and trial I figured out bad frames and checked my pipleline and made changes to and/or operators and thresholds.

The next challenge and the biggest problem is to stop flickering of lane lines on concrete surface or when the car comes out from the shadow. 

My pipeline is likely to failwhen left lane line to center is of different color and from center to right lane is of different color as in the challenge video. Also in case of a mountain terrain, it is quite likely to fail.

To make it more robust and stop the flickering of lane lines, we can average out the points from the previous frames to have a smooth transition per frame.
