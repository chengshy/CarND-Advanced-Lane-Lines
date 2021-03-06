##Report
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

[image1]: ./report_image/calib_img.png "calib_img"
[image2]: ./report_image/undist_ckb.png "undist checkboard"
[image3]: ./report_image/undist_test.png "undist test image"
[image4]: ./report_image/sobel_dir.png "sobel direction"
[image5]: ./report_image/sobel_mag.png "sobel magnitude"
[image6]: ./report_image/sobel_x.png "sobel x direction"
[image7]: ./report_image/s_threshold.png "threshold in saturation"
[image8]: ./report_image/perspective_transform.png "perspective transform"
[image9]: ./report_image/kernel.png "convolution kernel"
[image10]: ./report_image/window_fitting.png "window fitting"
[image11]: ./report_image/lane_reprojection.png "lane visual"
[image12]: ./report_image/combined_threshold.png "combined threshold"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/advanced_lane_detection.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. Here is the image showing the checkerboard detection

![alt text][image1]

I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image2]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell block 5 in advanced_lane_detection.ipynb).

Here is the sobel direction threshold binary mask

![alt text][image4]

Here is the sobel magnitude threshold binary mask

![alt text][image5]

Here is the sobel x direction threshold binary mask

![alt text][image6]

Here is the HLS S channel threshold binary mask

![alt text][image7]

Here is the combined threshold binary mask

![alt text][image12]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 450      | 300, 0        | 
| 200, 720      | 300, 720      |
| 1100, 720     | 980, 720      |
| 685, 450      | 980, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image8]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For the first image, I calculated the histogram of the bottom half image. Then partition the image into several horizontal slices. Here is the kernel I used to do the convolution:

![alt text][image9]

But sliding the windoe left right to find the maximum convolution result. Based on the sliding windows we found, we can group the pixels to left and right lanes. Using polyfit to find the left and right lane polynomial fit.

![alt text][image10]

This ploynomial fit will be passed to next frame to find the base x pixel position for next line fit.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in cell block No.14 in my code in `advanced_lane_detection.ipynb`. One pixel in x corresponds to 3.7/640 meters and one pixel in y corresponds to 30/720 meters. 
The curvature calculation is done by [these formulas](http://www.intmath.com/applications-differentiation/8-radius-curvature.php)
I calculated left and right curvature of the lane and used the harmonic mean of them to represent the lane curvature.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The No.13 cellblock in advanced_lane_detection.ipynb shows how we reproject lane back onto the road.

![alt text][image11]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's [my video result](./project_video.mp4)

The pipline is in the cellblock No.14 - 15. A low-pass filted is applied to the left and right fit to smooth the lane detection.

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

This is a very basic implementation of the lane detection. Some factors can causing the detection fail:
1. Perspective transform error. Sometimes, when the car bumps because of the uneven road, the preset perspective transform will be not accurate.

2. The image thresholding is the key part of this project. Sometimes, the cracks on the road will be detected as the false lane. And saturation thresholding is not very robust when lighting condition changes. More machine learning based segmentation can be applied.

3. The low pass filter can be implemented better, this naive implementation has some latency.
