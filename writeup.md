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

[image1]: ./output_images/calibration1_undist.jpg "Undistorted Chessboard"
[image2]: ./output_images/pipeline_breakdown/undist_test1.jpg "Road Undistorted"
[image3]: ./output_images/pipeline_breakdown/binary_test1.jpg "Binary Example"
[image4]: ./output_images/pipeline_breakdown/binary_warped_test1.jpg "Warp Example"
[image5]: ./output_images/pipeline_breakdown/lines_test1.jpg "Lines Fit"
[image6]: ./output_images/pipeline_breakdown/final_test1.jpg "Output"
[image7]: ./output_images/pipeline_breakdown/lines_unwarped_test1.jpg "Lines Unwarped"
[image8]: ./test_images/test1.jpg "Original Image"
[video1]: ./project_video.mp4 "Initial Video"
[video2]: ./test_videos_output/output_project_video.mp4 "Final Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the fourth code cell of the IPython notebook located in "./P4.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objpoints` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

The original image is here:

![alt text][image8]

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images. Here I used the distortion matrix from the chessboard images to undistort the test image.

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The function 'process_image' includes all of the color transforms, gradient calculations and other methods used to create a thresholded binary image. I used a combination of color and gradient thresholds to generate a binary image.  Here's an example of my output for this step. I used the sobel operation in the x direction, magnitude and direction of the sobel operation, and thresholds on the L and S color channels of an HLS image.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform(img)`.  The function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```python
offset = 300
src = np.float32([[595, 450],                               # Left    Top
				  [685, 450],                               # Right   Top
				  [1125, img_size[1]],                      # Right   Bottom
				  [180, img_size[1]]])                      # Left    Bottom
dst = np.float32([[offset, 0],                              # Left    Top
				  [img_size[0]-offset, 0],                  # Right   Top
				  [img_size[0]-offset, img_size[1]],        # Right   Bottom
				  [offset, img_size[1]]])                   # Left    Bottom
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595,  450     | 300, 0        | 
| 685,  450     | 980, 0        |
| 1125, 720     | 980, 720      |
| 180,  720     | 300, 720      |

I verified that my perspective transform was working as expected by verifying that the straight lines were created when straight_lines1.jpg was distorted.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I created a class to track the detected lines in video files. On the first frame (or for still images) this class will using a sliding window search to find pixels from the warped binary image and then will fit a 2nd order polynomial to all of the pixels. In subsequent video frames, the algorithm will search in the area near the existing best fit line. I also average over multiple video frames for the fit algorithm.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines 140 through 156 in my code in class Line(). This is done in both meters and pixels. Though the pixels measurement is fairly meaningless for the project.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `draw_lines()`.  Here is an example of my result on a test image:

![alt text][image7]

I then undistort that image using 'perspective_untransform()' and then combine the two images using 'weighted_img()'.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/output_project_video.mp4) The averaging is very obvious when the car drives over any bumps.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline still fails pretty bad on the challenge videos. I know that I need to implement some more plausability checks when a line is detected. There are multiple times in the challenge videos where the a new line is calculated that is completely wrong (i.e. curving in the wrong direction or crossing the other detected lane line). Discarding this data and trying to reevaluate in the next frame or starting over with the sliding window approach would improve the robustness of my code.

It also took me quite a while to get past the shadowed portion of the project video. Once I implemented a filter on the L channel though, this was much easier to accomplish.
