## Advanced Lane Finding Project


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

[chess_undistorted]: ./output_images/calibration1_undistorted.png "Undistorted"
[test_image]: ./test_images/test3.jpg "Road Transformed"
[chess_undistorted]: ./output_images/calibration1_undistorted.png "Undistorted"
[binary]: ./output_images/test3_binary.png "Binary Example"
[src_points]: ./output_images/transform_road_src_lines.png "Src Points"
[warp_example]: ./output_images/transform_road_and_invert.png "Warp Example"
[poly_fit]: ./output_images/test3_poly_fit.png "Fit Visual"
[output]: ./output_images/test3_result.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cell of the IPython notebook "Advanced-Lane-Lines.ipynb".

In the first block, I define two functions, one for undistorting a single image and another for undistorting a list of images. The second function can plot and save the results, if the flags are set when calling it.

The second block contains the actual calibration of the camera. Just like in the lessons, I'm using "object points" to represent the 3D chessboard coordinates in the real world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the three chessboard images that were not used for the calibration, as the specified number of corners could not be detected. Here is an example result: 

![alt text][chess_undistorted]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to this test image:
![alt text][test_image]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds implemented in the function `colorGradient()` in the 8th cell of the notebook to generate a binary image. I achieved the best result by performing a bitwise AND of the red color channel with the bitwise OR result of my other channels. This is the result for my test image:  

![alt text][binary]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `transform_perspective()`, and another function to invert the transformation called `invert_perspective_transform()`, which appear in the fifth cell of my notebook. The `transform_perspective()` function takes as inputs an image (`img`), as well as a boolean flag (`plot`) which will, if set, result to an image with the connected source points on the road being plotted and saved:
![alt text][src_points]

I chose the hardcode the source and destination points in the following manner:

```python
yHighSrc = 456
yLowSrc = 678
x1Src = 275
x2Src = 587
x3Src = 699
x4Src = 1046

src = np.float32([[x2Src,yHighSrc],[x3Src,yHighSrc],[x4Src,yLowSrc],[x1Src,yLowSrc]])

yHighDst = 0
yLowDst = img.shape[0]
xLowDst = np.int32((x1Src + x2Src)/2)
xHighDst = np.int32((x3Src + x4Src)/2)

dst = np.float32([[xLowDst,yHighDst],[xHighDst,yHighDst],[xHighDst,yLowDst],[xLowDst,yLowDst]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 587, 456      | 431, 0        | 
| 699, 456      | 872, 0        |
| 1046, 678     | 872, 720      |
| 275, 678      | 431, 720      |

I verified that my perspective transform was working as expected by transforming and back transforming a road image, verifying that the lines appear parallel in the warped image and looked like the original lines in the back transformed image.

![alt text][warp_example]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the function `fit_polynomial()` fit my lane lines with a 2nd order polynomial. In the first run, it uses the function `find_lane_pixels()` on a binary warped image, where a sliding window algortihm is used to find the lane pixels. Afterwards the numpy polyfit function is used inside `handle_detection_result()` to find the polynomial coefficents. The result is is visualized in this graphic:
![alt text][poly_fit]

In the subsequent video frames, the function `search_around_poly()` may be used to find the lane pixels, depending on whether or not the lines were detected in the previous iteration. To determine this fact, I check the quality of the fit parameters by looking at their differences to the previous frame's parameters and applying a threshold to them. This is implemented in the Line class's method `determine_detected()`. If both left and right lines were detected, I update the line parameters by calling the class method `updateProperties()`inside the function `handle_detection_result()`. In doing this conditionally, I reject outliers, where the fit parameters vary significantly from the prevoius frame.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The curvature of one line is calculated in the Line class's method `update_curvature_real()`. It transforms the x and y pixel values of the last line detection to the metric system, fit's a polynomial to them and measures the curvature at the bottom of the line with the formula given in the corresponding module. The curvature radius is appended to a list and averaged over the last n = 5 iterations, to reduce the jitter. For display, the curvature results of the left and right lines are averaged, which is implemented in the function `average_curvature_text()`.

I determine the position of the vehicle with respect to center in the function `offset_text()`. First I use the function `find_midpoint_and_bases()` to find the midpoint and line bases in a binary warped road image. These are converted to metric units and saved in the left and right line objects by calling their method's `update_line_base_pos()`. The offset is calculated by subtracting the image's midpoint from the center between the line bases:
```python
offset = (leftLine.line_base_pos + rightLine.line_base_pos)/2 - midpoint*leftLine.xm_per_pix
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I created a mask with colored lane lines and a highlighted lane area inside the function `fit_polynomial()`, which I transformed back to fit the original image scale inside my `pipeline()` function by using `invert_perspective_transform()`. Finally I overlayed this mask on the original image by using Open CV's function `addWeighted()`. The curves used to create the mask are filtered with a weighted average over the past n= 5 iterations inside the Line class, where the most recent fit gets the highest weight applied, to reduce jitter. Here is an example of my result on the test image:

![alt text][output]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
