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

[image1]: ./report_imgs/undistort_output.png "Undistorted"
[image2]: ./report_imgs/transformed.png "Road Transformed"
[image3]: ./report_imgs/combined.png "Binary Example"
[image4]: ./report_imgs/warped_straight_lines.png "Warp Example"
[image5]: ./report_imgs/fitting.png "Fit Visual"
[image6]: ./report_imgs/example_output.png "Output"
[video1]: ./result.mp4 "Video"
[image7]: ./report_imgs/reinforced.png "Reinforced Binary"
[image8]: ./report_imgs/histogram.png "Histogram"
[image9]: ./report_imgs/channels.png "HLS seperate channels"
## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "script.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I had to manually change the inner corner points for some images as (9,6) inner corners were not visible in all the images. The changes made were as follows:

Image 5 = (7, 5)
Image 4 = (6, 5)
Image 1 = (9, 5)

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

[Undistorted](./report_imgs/undistort_output.png)

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
[Road Transformed](./report_imgs/transformed.png)

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (5th code cell of the IPython notebook "script.ipynb"). I also used morphological transformations and blurring to get better results. Here's an example of my output for this step. 

[Binary example](./report_imgs/combined.png)

I used the OpenCV function cv2.dilate to get better results as shown in the following figure.

[Reinforced Binary](./report_imgs/reinforced.png)

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()` in the 4th code cell of my IPython notebook "script.ipynb".  The function takes as inputs an image (`img`) and then hardcoded the source and dest points according to one image "straight_lines1.jpg" in the following manner:

```python
trap_pts = np.array([[197,720], [580, 460], [702, 460], [1115, 720]], np.int32)
dest_pts = np.array([[350, 720], [350, 0], [970, 0], [970, 720]], np.float32)
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

[Warp Example](./report_imgs/warped_straight_lines.png)

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I applied an ROI mask to the image to eliminate clutter in the image and made a histogram like so:

[Histogram](./report_imgs/histogram.png)

And fit my lane lines with a 2nd order polynomial kinda like this:

[Fit Visual](./report_imgs/fitting.png)

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in a seperate function called `util()` which is in the 8th code cell of my IPython notebook "script.ipynb" which takes as input an img and returns the same image with all the text written on it.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 13th code cell of my IPython notebook by calling the `pipeline()` function which returns the output image.

[Output](./report_imgs/example_output.png)

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My first step was rectifying the error with camera calibration as not all the images had (9,6) corner points. I had to do this manually
Next step was experimenting with different kind of thresholding and their values to get the best possible result. 


I chose the HLS color scheme to get optimum results and derivative thresholding as well

[HLS seperate channels](./report_imgs/channels.png)

For derivative thresholding I downscaled the magnitude of y because that introduces noise in the image. This can be seen in the 52nd line of the 9th code cell of my other IPython notebook "jai_shree_ram.ipynb".
Then I used morphological transformations to get the best possible results.
I then used the sliding window approach to get lane line curvature and other values.

The model becomes very shaky when it encounters shadows which is not good for the vehicle or the passengers. This can be improved in several ways:

1. Smoothing over the last n frames of video to obtain a cleaner result by appending it to the list of recent measurements and then take an average over n past measurements to obtain the lane position I want to draw onto the image.
2. Whenever the model notices a lot of change in a new frame which is uncanny it used the last best frame and then searches for the lane lines from scratch in the next frame.
3. By introducing a Line() class in my code (as suggested in the lectures) to keep track of all these variables and come up with a cleaner code.

