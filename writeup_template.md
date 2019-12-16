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

[image1]: ./output_images/undistorted.jpg "Undistorted chessboard"
[image2]: ./output_images/Undistorted.png "Undistorted image"
[image3]: ./output_images/test4_binary.jpg "Binary test image"
[image4]: ./output_images/Transformation.png "Warp Example"
[image5]: ./output_images/warped_poly.jpg "Warped polyline"
[image6]: ./output_images/Final_image "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the fifth code cell of the IPython notebook located in "./AdvLaneLine.ipynb". 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `object_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `image_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  
Here I used the 'get_imagepoints_for_chessboard' method introduced in the Camera calibration lessen.

I then used the output `object_points` and `image_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistorted chessboard][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To see the effects of the undistort and verify it worked I used a little codeblock to plot the original and the undistorted image next to each other.
By applying the previously camera matrix and the distortion coeffitiants to the image it gets undistorted, which can c´be observed especially in the corners and on the sides of the image.
For example, in the left picture the bonnet is much more curved than in the right picture:
![Undistortion example][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I started very simple with the quizzes from the lessons, but soon figured out, that it will only work if I combine different gradients and color channels.
To get an idea of the different color channels on the different lighting situations I wrote a small helper you can find in the ihpython notebook cell three.
The helper functions were taken from the quizzes and implemented in code block four. The transformation pipeline is implemented in block six in the function pipeline.
The most difficult of the images was test4, so here is the result of the image transformation:
![Binary test image][image3]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

To get the parameters for my perspective transform I used a visual aid in code block eight, which I also used later to determine the length of the lane lines in pixels.
Because the calculation of the transformation matrix only needs to be done once, I splitted the calculation and the perspective transformation.
The calculation takes part in the fifth code cell line 20 to 23.
The warping of the image takes place in the image processing pipeline code cell six, line 31.


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 595, 450      | 200, 100        | 
| 685, 450      | 1110, 100      |
| 1110, 720     | 1110, 720      |
| 200, 460      | 200, 720        |


You can see the result of the transformation in the following picture:
![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To find the lane line pixels I used the code from the lessons (code block 4, line 63-142).
First I use a historgram to find the highest concentration of pixels in y direction on the left and right half of the picture.
After that I used a sliding window approach with a window count of nine to find the other pixels for the respective line.
The function returns all pixels for the left and right line found in the sliding window.

The output of this function is then forwarded to another function that fits a 2nd order polynomial in pixel space and in real world space.(code block 4, line 145-153)
The result looks like this:

![Warped polyline][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

To calculate the curvature of the lane and the position of the car, I created the functions calculate_curvature_real (code block 4, 204-209) and calc_offset (code block 4, 195-200).
And finally I have a function display_measures (code block 4, 211-219) which writes the measures onto the image.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The complete processing of a single picture with usage of alle the above mentioned functions takes place in the function process_single_image (code cell 6, 23-38)
The result of the complete chain looks like this:

![Final image][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

For the processing of the video I started with the function process_single_image. But that turned out to be critical, especially when a line could not be detected nicely.
So I took the advice from the project introduction and used a Line class to store the previous 10 line informations. I used the avg of the last 10 fits to find a smoother fit for the current line.
Especially in the processing of the video are a lot of possible improvements which could be done if there is more time.
Because I already have the calculation of the offset and curvature I am not using the suggested Line variables for it.
Here's a [link to my video result](test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Especially under difficult lighting situations or when bad markings are present my pipeline will fail. You also can see the if you use the pipeline on the challenge videos.
The first reason for this is the color and magnitude filter. With more sophisticated filtering and better tuned parameters it will perform w´better and recognize the lane lines better.
The second possibility for improvement is the a improved sanity check. At the moment I use only smoothing to improve the results.
Another possible idea is to segment the lane lines and if there are outages in one segment, the missing part could be interpolated. But when the road gets more curve this will likely fail, at least if the segments are big.
