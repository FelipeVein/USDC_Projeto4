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

[image1]: ./imagens_para_writeup/undistorted.JPG "Undistorted"
[image2]: ./imagens_para_writeup/undistorted_road.JPG "Road Transformed"
[image3]: ./imagens_para_writeup/threshold.JPG "Binary Example"
[image4]: ./imagens_para_writeup/birdeye0.JPG "Warp Example"
[image5]: ./imagens_para_writeup/polyfit.JPG "Fit Visual"
[image6]: ./output_images/test3.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell in the IPython notebook located in "./Final_Code.ipynb"

I first assumed that the chessboard is fixed on the x-y plane at z=0, which are the coordinates of the chessboard corners in the world, such that the object points are the same for each calibration image. Using OpenCV functions `findChessboardCorners` and `calibrateCamera` I was able to find the x-y pixle position of the chessboard corners and calibrate the image. The result is as follows:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of a x-axis gradient threshold and H-L-S threshholds. (`thresholdImage` function, in the second code cell in  `Final_Code.ipynb` or in the fifth code cell in `example.ipynb`)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The function `warpImage(img, toBird = True)` does the perspective transform. It can do the camera to bird transform or bird to camera transform. The code can be found in the second code cell in `Final_Code.ipynb` or in the fourth code cell in `example.ipynb`.


```python
    src = np.float32([[0.14*img.shape[1],img.shape[0]],[0.86*img.shape[1],img.shape[0]],[0.55*img.shape[1],0.64*img.shape[0]],[0.45*img.shape[1],0.64*img.shape[0]]])
    dst = np.float32([[0.14*img.shape[1],img.shape[0]],[0.86*img.shape[1],img.shape[0]],[0.86*img.shape[1],0],[0.14*img.shape[1],0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 0.14*img.shape[1], img.shape[0]      | 0.14*img.shape[1], img.shape[0]        | 
| 0.86*img.shape[1], img.shape[0]      | 0.86*img.shape[1], img.shape[0]      |
| 0.55*img.shape[1], 0.64*img.shape[0]     | 0.86*img.shape[1], 0      |
| 0.45*img.shape[1], 0      | 0.14*img.shape[1], 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To fit their positions with a polynomial, I used numpy's `polyfit` function.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To do that, I wrote the `calculate_radius_and_center` function. 

To calculate the vehicle's position with respect to center was simple: I took the diffence between the center of the fitted curves at the nearest point to the car and the center of the image. Then, I multiplied the result by a factor. The result is in meters.

```python
center = binary_warped.shape[1]/2 -  (asd+asd2)/2

mppx = 3.7 / (0.86*binary_warped.shape[1] -  0.14*binary_warped.shape[1])# 3.7/ (0.86*binary_warped.shape[1] -  0.14*binary_warped.shape[1])  is meters per pixel in x axis

center = mppx * center
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented these steps in the `draw_lane` and `draw_data` functions. The result is shown bellow:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).


Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline does not work properly when there are different lighting conditions, shadows, discoloration. 

Set the threshold parameters to the project video wasn't difficult, but with the same parameters the pipeline falls apart in the other two videos.

I tried a lot of different thresholds, but none of them worked well on the Harder Challenge Video. Some of them worked on the Challenge Video, but they weren't good enough. What approach should I take to make it work?

