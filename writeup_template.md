## Writeup

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

[image1]: ./examples/undistort_output.png "input and Undistorted output"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/thresholding_Final.png "Thresholding Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/Unwarped_straight_lines.png "Unwarp example"
[image6s]: ./test_images/test6.jpg  "test image 6"
[image6d]: ./examples/test6image_output.png "Example Output with test image 6"
[video1]: ./marked_project_video.mp4 "Output Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.
You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the jupyter notebook located in "AdvancedLaneFinding.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image  'camera_cal/calibration1.jpg' using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding functions are implemented in code cell 4 of the notebook).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a class called 'PerspectiveTransformer', which appears in code cell 6 of the notebook.
The class implements two methods 'Warp' and 'Wnwarp' to perform the perspective transform from camera view to 'bird's eye'/top view and vice-versa. 

Followig piece of code shows the source and destination points used for the transformation.
These points were selected using 'test_images/straight_lines1.jpg' as reference image. The objective is to transform the converging straight lines in camera view to parallel straight lines in top view.

```python
        # Points     [(left_bot ), (left top ), (right top), (right bot)]
        self.s_pts = [(230,  700), (595,  450), (685,  450), (1080, 700)]
        self.d_pts = [(300,  700), (300,    0), (1000,   0), (1000, 700)]
        src = np.float32(self.s_pts)
        dst = np.float32(self.d_pts)
        self.M = cv2.getPerspectiveTransform(src, dst)        
        self.Minv = cv2.getPerspectiveTransform(dst, src)
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

I also verified that my inverse perspective transform was working as expected by drawing the warped image back in to camera view.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I mostly used the method explained in the lessons to implement the classify the non-zero pixels in the warped image to left and right lines of the lane.
The code for this is in code cells 8 and 9 of the notebook.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I reused the code/logic explained in the lessons.
This is implemented in function "get_roc_in_meters" in code cell 9 of the notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in function 'process_frame()' in code cell 9 of the notebook.
Here is an example of my result on a test image 6:

![alt text][image6s]
![alt text][image6d]

---

### Pipeline (video)

Here's a [link to my video result](./marked_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Problems faced:

Color thresholding:
I initially implemented the pipeline according the information presented in the lessons. I used S channel (from HLS color space) for color thresholding.
But the program was unable to locate lines when there was shadows or the road color changed from black/brown to gray/white.
I then dumped the images from that portion of the project video and ran the threshold functions to see if the lane lines were extracted properly. Turns out the problem was that S channel was not reliable when there are shadows.
I then switched to extracting the "yellow" color and "white" color as these are the common colors of the road lines.
As shown in the examples, this gives very good results under different lighting conditions.


Curvature calcuations:
The radius of curvature calculation seems to be bit off. I got the radius of curvature of ranging from 300m to 30000ms.
While higher values can be explained due to almost straight lines, I am unabe to explain the values less than 587 m (absolute minimum as per US road standards). So there is probably some bug in the code that calculates the RoC in meters but I was unable to root cause it.


Where the pipeline is likely to fail:
Pipeline will most likely fail in case of roads with multiple lane lines (like double yellow lines) or road edges with contrasting color as the windowing algorithm used for locating lines will likely classify the pixels belonging to road edge as part of right line.

What could you do to make it more robust?
1. Thresholding (specially gradiant) can be better tuned to highlight the lane lines (and mask out the road edges and other disturbances).
2. locate_lines() function can be improved to use adaptive window sizes/margins.
3. Class Lane/Line can keep track of the polynomial fit over time to "smooth" out any sudeen changes.
3. if sanity check for only one line fails, the good line can be used as reference to search. (by adding "possible" width of the lane to the x coordinates of the good line).



