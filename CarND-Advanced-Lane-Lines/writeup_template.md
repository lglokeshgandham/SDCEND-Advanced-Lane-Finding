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

[undistorted]: ./output_images/undistorted_calibration2.jpg "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[binarythresholded]: ./output_images/binary_thresholded_straight_lines1.jpg "Binary thresholded"
[warped]: ./output_images/binary_warped_straight_lines1.jpg "Warped example"
[polyfit1]: ./output_images/poly_fit_1.jpg "Fit Visual 1"
[polyfit]: ./output_images/poly_fit.jpg "Fit Visual"
[Outputim]: ./output_images/final_result_image.jpg "Output"
[video1]: ./project_video_output.mp4 " output Video"


## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][undistorted]  

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
 
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. Here's an example of my output for this step.
first I applied horizontal gradient (along x directon)(using abs_sobel_thresh function) and then applied direction gradient with threshold 30,90 degrees as lane lines are close to vertical.Then I applied threshold on R ,G,S channels so that yellow lines are detected and applied threshold on L channel also so that edges due to shadows are left out.And finally applied region selection where lane lines fall into to get final binary thresholded image.

![alt text][binarythresholded]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform I chose source points which form trapezoidal shape.then computed transform matrix and its inverse.


| Source        | Destination   | 
|:-------------:|:-------------:| 
| 220, 720      | 320, 0        | 
| 1110, 720     | 320, 720      |
| 570, 470      | 960, 720      |
| 722, 470      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image as shown below.

![alt text][warped]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

for fitting my lane lines with a 2nd order polynomial ,First I divided my image and found out bottom postion of lane line using histogram method.then I applied sliding window method by taking length 100px and taking 10 windows on image. I took mininum threshold of 100 to recenter window.then I fit the polynomial with the help of all non zero pixels in the windows band.This is implemented in brute_search method. the output by this method is below
![alt text][polyfit1]
once we fit the polynomial we can search in margin around the polynomial for a new frame image as lane lines doesnt change much in consecutive frames.output by this method is

![alt text][polyfit]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

for calculating the radius of curvature in real world dimensions first I applied pixel to meter transformation. then I calculated radius of curvature by applying polynomial fit on transformed attributes. center of image gives center of car , mean of lane pixels at bottom of image gives lane center .subtracting both give offset.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I filled out the lane area and then unwarped the image using the inverse transform matrix which i computed earlier while doing perspective transform and combine original image with processed image .the output looks as follows.

![alt text][Outputim]

---

### Pipeline (video)

While doing the final pipeline 
In some frames lane lines might not be detected or lane lines doesn't make sense. I determined these as badframes if
1.no pixels were detected using sliding window search/search around previous detected line
2.average gap between lines is < 0.6 times or >1.3 times the globally maintained avg line gap

if bad frame is detected I performed sliding window search again (using brute_search method),if output is still a bad frame I took previously well detected frame.

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The first major finding in the project is to get well thresholded binary image.whenever car drove by next lane I had to apply refgion masking and reduce the sliding window length so as to not consider the pixels of that car while computing polynomial fit.

this pipeline might fail if there are more sharper turns in short spans.