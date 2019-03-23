## Writeup Template

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

[image1]: ./output_images/undistort_output.png "Undistorted"
[image2]: ./output_images/undistorted_straight_lines1.jpg "Road Transformed"
[image3]: ./output_images/thresholded_straight_lines1.jpg "Binary Example"
[image4]: ./output_images/warped_straight_lines1.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines_straight_lines1.jpg "Fit Visual"
[image6]: ./output_images/output_straight_lines1.jpg "Output"
[video1]: ./test_videos_output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third code cell of the IPython notebook located in "Advanced Lane Finding.ipynb" in function named `configure_calibration`

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I added the dist configuration to a pickle file for optimization which I check for existence every time befory applying this distortion correction to the test image using the `cv2.undistort()` function (check the implementation in `undistort_image` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at in the third code cell of the IPython notebook located in "Advanced Lane Finding.ipynb" in function named `combined_thresholds`).I've implemented three other functions `abs_sobel_thresh`,`dir_threshold` & `hls_select` to get the thresholded binary using sobel parameter in x direction,using atan2 for sobel direction getting all lines that has from 40 to 90 degree & using the S & L output from hls to get to control the saturation and lightness which helped in removing shadows Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 3rd code cell of the IPython notebook).  The `birdeye()` function takes as inputs an image (`img`), as well as corners (`corners`) from `undistort_image` function  and inverse (`inverse`) boolean to check whether to switch to birdeye view or reset to the standard view.  I chose the hardcode the source and destination points in the following manner:

```python
    src = np.float32([(250, 680), (1050, 680), (600, 470), (730, 470)])

    dst = np.float32([(280, 720), (1000, 720), (280, 0), (1000, 0)])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 250, 680      | 280, 720      | 
| 1050, 680     | 1000, 720     |
| 600, 470      | 280, 0        |
| 730, 470      | 1000, 0       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I created a histogram, split the histogram for the two lines, the peak of the two halves of the histogram will be the starting point for the left and right lines got nonzero pixels (activated pixels), then I made a number of windows with height equals to image height for evey window I checked whether any activated pixels falls into if so append these pixels to our lists left_lane_inds and right_lane_inds & I draw a rectangle on these pixels, After that I use polyfit function to fit a polynomial throw these points implementation can be found in function `find_lane_pixels` & `fit_polynomial` in the 3rd code cell of the IPython notebook

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Implementation can be found in `measure_curvature_pixels` function in the 3rd code cell of the IPython notebook

I used the math equation to compute radius of curvature value 

` python
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
`
I multipied it be ym_per_pix to get the value in meters other than pixels, I also changed the parameter for polyfit in function `fit_polynomial` check the python code

`python
    left_fit_cr = np.polyfit(lefty * ym_per_pix, leftx * xm_per_pix, 2)
    right_fit_cr = np.polyfit(righty * ym_per_pix, rightx * xm_per_pix, 2)
`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Implementation for the pipeline in `process_image` function

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

* When thresholding I faced a problem with shadows which affected my lane line detection greatly I negated its effects by using the l output from hls color space & minimized the region for wraping

* Some frames didn't have any right line detected which made the polyfit function to fail as there was no lines detected, I had to include more sobel& direction thresholding to make sure that every frame has detected lane lines

The pipline may fail on tunnels & u-turns, It will also fail if lane lines weren't accurately drown in any place

Maybe more object detection & localization could help in making the pipeline more robust