## Advanced Lane Finding Project

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

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

[//]: # (Image References)

[distorted]: ./camera_cal/calibration1.jpg "Distorted"
[undistorted]: ./output_images/undistored_calibration1.jpg "Undistorted"

[distortedroad]: ./test_images/test1.jpg "Distorted Road Image"
[undistortedroad]: output_images/undistored_test1.jpg "Undistorted Road Image"

[test_prior_binarycombo]: ./test_images/test6.jpg "Binary Example"
[binarycombo]: ./output_images/threshold_binary_test6.jpg "After combined thresholded colr and gradient Example"

[pespectivetransformed]: ./output_images/threshold_binary_birds_eye_transformed_test6.jpg "Perspective Transformed"

[slidingwindowpolyfit]: ./output_images/threshold_binary_birds_eye_transformed_poly_fit_test6.jpg "Sliding window polynomial fit"

[radiusplotted]: ./output_images/lane_plotted_test6.jpg "Radius plotted"

[videooutput]: ./output_images/project_video_output.mp4 "Video Annotated"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `./AdvancedLaneFinding.ipynb`.  Refer to the functions `calibrate_camera(debug)` and `def cal_undistort(img_p, objpoints_p, imgpoints_p)`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

##### Distorted Image
![distorted][distorted]

##### Undistorted Image
![undistorted][undistorted] 


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images.

I have used the previously calculated `imgpoints` and `objpoints` to calibrate camera using `cv2.calibrateCamera` function, which returns distortion coefficients. These distortion coefficients can be used to undistort the image using `cv2.undistort` which gives the undistorted image below.

| Distorted Road Image | Undistorted Road Image |
|---|---|
|![distortedroad][distortedroad]|![undistortedroad][undistortedroad]|


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (`cal_threshold_binary_image(img, grad_thresh, color_thresh)` in `./AdvancedLaneFinding.ipynb`).  Here's an example of my output for this step. Here I have stacked both gradient thresholded binary image and thresholded s color channel.

| Original Image | After binary thresholding |
|---|---|
|![test_prior_binarycombo][test_prior_binarycombo]|![binarycombo][binarycombo]|


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `undistort_and_birds_eye_transform(img)`, which appears `./AdvancedLaneFinding.ipynb`. The `undistort_and_birds_eye_transform(image)` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```python
    src = np.float32([[568, 468], [715, 468], [1040, 680], [270, 680]])
    dst = np.float32([[200, 0], [1000, 0], [1000, 680], [200, 680]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

| Binary thresholded image | Perspective transformed |
|---|---|
|![binarycombo][binarycombo]|![pespectivetransformed][pespectivetransformed]|

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I created functions like `find_lane_pixels` and `fit_polynomial(birds_eye_image)` in the file  `./AdvancedLaneFinding.ipynb`. The `find_lane_pixels` uses image histogram to get the area where there is high concentration of pixels (which means high probability of lanes) and sliding window technique to find all the points that represent lanes. The `fit_polynomial` funtion tries to fit a 2nd degree poynomial. When the same polynomial is used to plot lanes there is a perfect alignment over the visible lanes.

![slidingwindowpolyfit][slidingwindowpolyfit]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I used the polynomial function derived using lane points identified in the above steps. There is also an optmized function is created named `search_around_poly_real` that leverages previously found lane points to search around for lane pixels rather than repeating the sliding window histogram based search. 

I created function named `measure_curvature_real()` that uses the points dervied using polynomial and using the formula as per https://www.intmath.com/applications-differentiation/8-radius-curvature.php to calculate radius.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I calculated left curve radius and right curve radius using above steps and averaged to get the curvature of the road. I Then plotted the data calculated on the original image. 

Offset of the camera from the center is calculated using below formula

```python
    xm_per_pix = 3.7/700 # meters per pixel in x dimension

    lane_center = (left_fitx[-1] + right_fitx[-1])/2.0
    camera_center = img_shape[1]/2.0
    offset_center = (lane_center - camera_center) * xm_per_pix
```
![radiusplotted][radiusplotted]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

In summary the process is for every image in a video frame
1. Undistort the image which is introduced by the camera
2. Then identify lanes in the image using gradient thresholding techniques using sobel operator.
3. Perform a perspective transformation to get birds eye view of the identified lanes
4. Try to identify points that image and fit a second degree polynomial
5. Using the polynomial derive all the points that falls on the lane
6. Using these points for left lane and right lane, calculate radius and offset
7. Create a polyfill overlay using the lane points identified in the image.

Challenges
1. Quite slow to perform video annotation
2. Could not understand why the annotation logic I have did not work on the challenge video. 