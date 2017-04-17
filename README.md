
# Advanced Lane Finding Project
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

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

[image1]: ./examples/board_undist.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image2-1]: ./examples/test1_undist.jpg "Road Transformed"
[image3]: ./examples/orig_to_thresh.png "Binary Example"
[image4]: ./examples/warped_test.png "Warp Test"
[image4-1]: ./examples/orig_to_warped.png "Warp Example"
[image5]: ./examples/conv_boxes.png "Fit Visual"
[image5-1]: ./examples/conv_lanes.png "Fit Visual"
[image6]: ./examples/lanes_plus_curv.png "Output"
[video1]: ./output_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
### Here I consider the rubric points and describe how I addressed each point in my implementation, located in the `Advanced_Lane_Lines_v1.ipynb` file.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in code cells #2-3.

1. I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  
- Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  
- `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

2. I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.
To demonstrate this step, we can apply the distortion correction to one of the test images like this:

| Original    | Undistorted  | 
|:-----------:|:-------------:|
| ![][image2] | ![][image2-1] |


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in code cells #24-25).  

Here's an example:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for the perspective transform appears in code cells #26-27.

The warping of images requires inputs for a source (`src`) and destination (`dst`) points.  The source and destination points are hardcoded in the following manner:

```
src = np.float32([
        [img_size[0]*(.5-mid_width/2), img_size[1]*height_pct],  # top left
        [img_size[0]*(.5+mid_width/2), img_size[1]*height_pct],  # top right
        [img_size[0]*(.5+bot_width/2), img_size[1]*bottom_trim], # bottom right
        [img_size[0]*(.5-bot_width/2.15), img_size[1]*bottom_trim]  # bottom left
    ])
offset = img_size[0]*.25
dst = np.float32([
        [offset, 0],                            # top left
        [(img_size[0] - offset), 0],            # top right
        [(img_size[0] - offset), img_size[1]],  # bottom right
        [offset, img_size[1]],                  # bottom left
    ])
```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 588, 453      | 320, 0        | 
| 691, 453      | 960, 0        |
| 1036, 673     | 960, 720      |
| 270, 673      | 320, 720      |


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]
![alt text][image4-1]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I used search windows with convolution to detect lane lines on the image and fit the centers of the window detections with a 2nd order polynomial in code cells #31-33:

| Sliding Boxes   | Polynomial Fit | 
|:--------------:|:--------------:|
| ![][image5]    | ![][image5-1]  |

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated for an example image in code cells #35-36.

```
# Now our radius of curvature is in meters
print('{:.2f} m,  {:.2f} m'.format(left_curverad, right_curverad))
```
```
920.16 m,  627.40 m
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The final result is plotted back onto the road, as seen in code cells #42 & 45.

Here is an example of a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's my [video output result](./output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The basic approach was taken from the classroom and listed above in the "goals / steps" of the project.

This included undistorting, gradient & color thresholding, warping, lane detection, and measuring the curvature of radius & offset from lane center.

After tinkering with various thresholding combinations to generate the warped binary image, the settings from the [Project Q&A](https://www.youtube.com/watch?v=vWY8YUayf9Q) were used. 

The pipeline might fail with different lighting/weather conditions or roads with sharper curves or faded lane lines.

The pipeline could be improved by performing the sliding window search differently for already-detected lines, or smoothing the lane line annotations further.  
