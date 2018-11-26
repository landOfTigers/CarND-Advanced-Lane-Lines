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

[image3]: ./examples/binary_combo_example.jpg "Binary Example"

[src_points]: ./output_images/transform_road_src_lines.png "Src Points"
[warp_example]: ./output_images/transform_road_and_invert.png "Warp Example"

[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
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

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][test_image]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

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

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
