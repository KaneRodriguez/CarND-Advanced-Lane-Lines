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

[project-video]: ./output_videos/project_video_output.mp4 "Project Video"

[dist-cal]: ./output_images/distortedCalibrationImage.jpg "Distorted Calibration Image"
[undist-cal]: ./output_images/undistortedCalibrationImage.jpg "Undistorted Calibration Image"

[dist-test]: ./output_images/distortedTestImage.jpg "Distorted Test Image"
[undist-test]: ./output_images/undistortedTestImage.jpg "Undistorted Test Image"

[before-birds-eye-view]: ./output_images/birdsEyeViewBeforeImage.jpg "Before Birds Eye View"
[after-birds-eye-view]: ./output_images/birdsEyeViewAfterImage.jpg "After Birds Eye View"

[hls-full]: ./output_images/bestColorTransform.jpg "HLS Color Transform of Image"
[w-and-y]: ./output_images/whiteAndYellowImage.jpg "White and Yellow Highlight"

[sobel-x]: ./output_images/bestSobelGradient.jpg "Sobel X Gradient"

[accentuated]: ./output_images/accentuatedLaneLines.jpg "Accentuated Lane Lines from Sobel and Color Thresholding"

[window-search]: ./output_images/birdsEyeViewLaneLineImageWindowSearch.jpg "Lane Lines Detected from Window Search"
[previous-search]: ./output_images/birdsEyeViewLaneLineImagePreviousSearch.jpg "Lane Lines Detected from Previous Searches"

# TODO: radius of curvature

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
     
The code for this step is contained in the first cell of the "Calibrating the Camera" section of the IPython notebook located in "./main.ipynb".

I start by creating two empty lists of object points and image points. The object points will contain points in 3-D space and will be used with the `cv2.calibrateCamera` function. I create an object point (x, y, z) that has all z values set to 0 and will be appended to the array of objects points each time an image's corners is successfully detected with `cv2.findChessboardCorners`. 

After this initial setup, each jpg image in the `camera_cal` directory is grayscaled and processed by the `cv2.findChessboardCorners` and the resulting detected corners (if any) are appended to the array of image points (x,y).

Once each image is processed and the image and object points arrays are filled, these arrays are used with the `cv2.calibrateCamera` function to compute the camera calibration matrix and distortion coefficients. Distortion correction is achieved by passing in the matrix and coefficients to `cv2.undistort`. An example of a distorted and undistorted camera calibration image is shown below:

Distorted             |  Undistorted
:-------------------------:|:-------------------------:
![Distorted Calibration Image][dist-cal]  |  ![Undistorted Calibration Image][undist-cal]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Below is an example of a test image with distortion correction applied to it:

Distorted             |  Undistorted
:-------------------------:|:-------------------------:
![Distorted Test Image][dist-test]  |  ![Undistorted Test Image][undist-test]

Distortion correction is achieved by passing in the camera calibration matrix and distortion coefficients to `cv2.undistort` via the `distortionCorrection` function found in "./main.ipynb".

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
    
The code for my perspective transform includes a function called `applyPerspectiveTransform()`, which appears in the 1st code cell under the "Applying and Reversing Bird's Eye View Perspective Transform" heading in "./main.ipynb" IPython notebook. The `applyPerspectiveTransform()` function takes as inputs an image (`img`) as well as source (`src`) and destination (`dst`) points. These points are used with `cv2.getPerspectiveTransform` function to get a transformation matrix. I apply this matrix to the image via the `cv2.warpPerspective` function to get a birds eye view image, as seen below.

Undistorted Test Image with Source Points             |  Warped Image
:-------------------------:|:-------------------------:
![Undistorted Test Image with Source Points Drawn][before-birds-eye-view]  |  ![Birds Eye View of Test Image][after-birds-eye-view]

I chose to code the source and destination points are hardcoded in the following manner:

```python
    src = np.float32(
        [[width-695,    460],
         [203,       height],
         [width-153, height],
         [695,         460]])
    
    dst = np.float32(
        [[width - 960,      0],
         [width - 960, height],
         [960,         height],
         [960,             0]])
```

This resulted in the following source and destination points for an image with width 1280 and height 720:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color transformations, color thresholds, and gradient thresholds to generate a binary image. The code can be found in the [Accentuating Lane Lines]["./AccentauteLaneLines.ipynb"] IPython notebook.

Several images showcasing each stage of this step is given below:

HLS Full                   |  White and Yellow Accentuated            |  Sobel X Gradient         | Combined Result
:-------------------------:|:-------------------------               :|:-------------------------:|:-------------------------:
![HLS Full][hls-full]      |  ![White and Yellow Attenuated][w-and-y] | ![Sobel X][sobel-x]       | ![Combined Result][accentuated] 

The pipeline for this step is to first convert the image to HLS Full format, use thresholds that filter out all but white and yellow colors to the HLS Full image, apply a Sobel X gradient to the HLS Full image, and combine the results of the Sobel X gradient and the HLS Full White and Yellow accentuating.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Identifying lane pixels from a binary image and fitting their positions with a polynomial is done in the cells beneath the "Finding Lane Pixels from a Window Search" of the [Main Python Notebook][./main.ipynb].

Our pipeline calls the `attenuateLaneLines` (the first cell under the 

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
