### SDC-term1
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
    
    Tung Thanh Le
    ttungl at gmail dot com
   
**Advanced Lane Finding Project**
---
This is my [video result](https://youtu.be/IGK2Hxb-p24).

<img src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/alf.gif" height="303" width="550">

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Here I will consider the [rubric](https://review.udacity.com/#!/rubrics/571/view) points individually, and describe how I addressed each point in my implementation.

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function (see in `undistorted_correction()` of [block 3](https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/Advanced%20Lane%20Finding.ipynb)).  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_corners_1.png">
<!-- <img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_corners_2.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_corners_3.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_corners_4.png"> -->

Undistortion of an input image: 
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_undist_1.png">
<!-- <img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_undist_2.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_undist_3.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_undist_4.png"> -->

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like these ones below:
<!-- 
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/undist_1.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/undist_2.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/undist_3.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/undist_4.png"> -->
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/undist_5.png">

First, I use the original image, object points, and image points arrays as the inputs/arguments of `undistorted_correction` method. I convert this image into grayscale using `cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)`. Then, I use `cv2.calibrateCamera()` to compute the camera calibration. The output of `cv2.calibrateCamera()` is used to feed into `cv2.undistort()` to convert the image to the undistorted images as illustrated above.


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/color_gradient_thresholds_org_1.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/color_gradient_thresholds_org_combine.png">

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/color_gradient_thresholds_org.png">


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
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
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/img3_warped.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/img3.png">

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/bird_eye_warped_binary.png">

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/fillPolywindow1.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/fillPolywindow2.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/histogram_bird_eye_warped_binary.png">

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/unwarped_back_original.png">
<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/warped_green_surface.png">

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

This is my [video result](https://youtu.be/IGK2Hxb-p24).

---
### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
