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

Here I will consider the [rubric](https://review.udacity.com/#!/rubrics/571/view) points individually, and describe how I addressed each point in my implementation.

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function (see in `undistorted_correction()` of [cell 3](https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/Advanced%20Lane%20Finding.ipynb)).  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

<img width="750" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_corners_1.png">

Undistortion of an input image: 

<img width="750" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/chess_undist_1.png">

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like these ones below:

<img width="750" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/undist_5.png">

First, I use the original image, object points, and image points arrays as the inputs/arguments of `undistorted_correction` method. I convert this image into grayscale using OpenCV library, `cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)`. Then, I use `cv2.calibrateCamera()` to compute the camera calibration. The output of `cv2.calibrateCamera()` is used to feed into `cv2.undistort()` to convert the image to the undistorted images as illustrated above.

#### 2. Describe how and identify to use color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image in `combined_gradient_thresholds()` function (see in [cell 3](https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/Advanced%20Lane%20Finding.ipynb)). Here's an example of my outputs for this step.

From the original image, I can use different color and gradient thresholds such as sobelx, sobely, saturation of HLS (using `cv2.COLOR_RGB2HLS`), value of HSV, saturation of HSV (using `cv2.COLOR_RGB2HSV`), b of LAB (using `cv2.COLOR_RGB2Lab`). The `lightness_binary` is the combination of saturation and lightness dimensions in HLS (using `cv2.COLOR_RGB2HLS`). 

<img width="750" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/color_gradient_thresholds_org.png">
<img width="750" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/color_gradient_thresholds_org_1.png">

I have played with many combinations and I found the best combination that works for me as below.
```python
combined_binary[(saturation_HSV_binary==1) | (sobelx_binary==1) | (sobely_binary==1) & (value_HSV_binary==1) | (lightness_binary==1)]=1
```

<img width="650" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/color_gradient_thresholds_org_combine.png">


#### 3. Describe and identify how to performe a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes methods `get_perspective_transform()` and `bird_eye_perspective()` in [cell 3](https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/Advanced%20Lane%20Finding.ipynb).

The `get_perspective_transform()` method takes as inputs an image (`img`), source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

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

The `bird_eye_perspective()` method also takes the image, src, and dst. This method first undistorts the image using `undistorted_correction()` method, then it uses the output to put to `get_perspective_transform()` method. The outputs are `M`, `Minv`, `warp`, and `undist`. 

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/img3_warped.png">

<!-- I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. -->

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/img3.png">

<img width="450" src="https://github.com/ttungl/SDC-term1-Advanced-Lane-Finding/blob/master/output_images/bird_eye_warped_binary.png">

#### 4. Describe how to identify lane-line pixels and fit their positions with a polynomial.

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
