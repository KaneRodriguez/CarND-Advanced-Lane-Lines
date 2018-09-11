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

[pipeline-result]: ./output_images/pipelineResult.jpg "Pipeline Result"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
     
The code for this step is contained in the first cell of the "Calibrating the Camera" section of the [Main](./main.ipynb) IPython Notebook.

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

Distortion correction is achieved by passing in the camera calibration matrix and distortion coefficients to `cv2.undistort` via the `distortionCorrection` function found in the [Main](./main.ipynb) IPython Notebook.

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
    
The code for my perspective transform includes a function called `applyPerspectiveTransform()`, which appears in the 1st code cell under the "Applying and Reversing Bird's Eye View Perspective Transform" heading in the [Main](./main.ipynb) IPython Notebook. The `applyPerspectiveTransform()` function takes as inputs an image (`img`) as well as source (`src`) and destination (`dst`) points. These points are used with `cv2.getPerspectiveTransform` function to get a transformation matrix. I apply this matrix to the image via the `cv2.warpPerspective` function to get a birds eye view image, as seen below.

Undistorted Test Image with Source Points             |  Warped Image
:-------------------------:|:-------------------------:
![Undistorted Test Image with Source Points Drawn][before-birds-eye-view]  |  ![Birds Eye View of Test Image][after-birds-eye-view]

I chose to hardcode the source and destination points in the following manner:

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

I used a combination of color transformations, color thresholds, and gradient thresholds to generate a binary image. The code can be found in the [Accentuating Lane Lines](./AccentauteLaneLines.ipynb) IPython notebook.

Several images showcasing each stage of this step is given below:

HLS Full                   |  White and Yellow Accentuated            |  Sobel X Gradient         | Combined Result
:-------------------------:|:----------------------------------------:|:-------------------------:|:-------------------------:
![HLS Full][hls-full]      |  ![White and Yellow Attenuated][w-and-y] | ![Sobel X][sobel-x]       | ![Combined Result][accentuated] 

The pipeline for this step is to first convert the image to HLS Full format, use thresholds that filter out all but white and yellow colors of the HLS Full image, apply a Sobel X gradient to the HLS Full image, and combine the results of the Sobel X gradient thresholding and the HLS Full White and Yellow accentuating into one binary image. 

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Identifying lane pixels from a binary image and fitting their positions with a polynomial is done in the cells beneath the "Finding Lane Pixels from a Window Search" of the [Main](./main.ipynb) Python Notebook.

Our pipeline initiates the lane line detection by calling the `identifyLaneLines` function (found under the "Identifying Lane Lines by Combining Window Search and Searching from Previous" section). This function takes an input image (`img`) and performs a `windowSearch` if one or both of the lines has not previously been detected or a `searchFromPrevious` if both lines have been detected prior.

##### Window Search

Internally, the `windowSearch` function works by calling a `find_lane_pixels` function that takes a histogram of the bottom half of the image and assigns a "base left x" and "base right x" based on the frequency of x values in the histogram. Once these bases are used to initialize "left x current" and "right x current", I iterate through the user specified number of windows (`nwindows`) and create the bounds of a window from a specified margin and from the current left and right x coordinates. Any nonzero pixels within the bounds of this window are recorded in "good left lane indices" and "good right lane indices". These indices are added to the list of indices found for each lane in each window (`left_lane_inds` and `right_lane_inds`, respectively). If the number of good indices found is over the number of minumum pixels required to recenter the window, the left and right current x positions are updated to reflect this shift. 

Once each window has been processed, the left and right line pixel positions are assigned to the values corresponding to the good indices found within the windows. These pixels are then used to create a polynomial representing the line via the `fitPoly` function of the `Line` object (see [Line Helper Class](#Line-Helper-Class) below).

Below is an example image detailing the result of this process. The windows are bounded with green rectangles and the good indices are colored in red for left and blue for right.

Test Image                  |  Lane Lines Detected From Window Search           
:--------------------------:|:----------------------------------------:
![accentuated][accentuated] |  ![Window Search][window-search] 

#### Search From Previous

The `searchFromPrevious` function utilizes the previously computed polynomial fits for the left and right lines to deterimine the next wave of left and right lane indices. I generate a list of all non-zero x and y values and use those in tandem with the polynomial and a margin to filter out all non zero indices that fit within the given window. These x and y lane indices are passed into each lines `Line.fitPoly` function in order to create a new polynomial to both represent a lane line on the image and to aid in the lane line search in the next processed frame.

Below is an example image detailing the result of this process. The search region is highlighted in green and the good indices are colored in red for left and blue for right.

Test Image                  |  Lane Lines Detected From Previous Searches           
:--------------------------:|:----------------------------------------:
![accentuated][accentuated] |  ![Previous Search][previous-search] 

#### Line Helper Class

The `Line` helper class, found under the "Lane Lines Helper Class" section of the [Main](./main.ipynb) IPython notebook, keeps a history of computed polynomial fits for a given line. The `fitPoly` function is the `Line` method that contains the logic for computing the fits based on previous fits. This function takes in a list of found lane line x and y image coordinates (`laneX` and `laneY`) and an image shape (`imgShape`). The line x and y coordinates are fed into `np.polyfit` to create a temporary fit for the line. This fit is added to the `fitHistory` list and this list is truncated if it's size is larger than the allowed history for this line. 

The "current fit" is assigned to the average of all previous fits (which includes the newly added polynomial from the passed in lane coordinates) via `np.mean(self.fitHistory, axis=0)`. An array of y-values are generated from 0 to image height and these values are passed into the current fit polynomial to generate corresponding x-values. These generated (x, y) points represent the points of the lane line on the image.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

##### Radius of Curvature

Calculating the radius of curvature of the lane is done via the `updateRadiusOfCurvature` method of the `Line` class. This method takes in an image shape (`imgShape`) and a polynomial (`fit`) representing the lane line (in meters). 

First, the maximum y-value (bottom of the image) is converted to world space and the first two parameters of `fit` are assigned to `A` and `B`. The variables `A`, `B`, and `y` are then plugged into the radius of curvature equation seen below.

```python
    self.radius_of_curvature = (1 + (2*A*y + B)**2)**(3/2) / (np.abs(2*A))
```

Note: This function is called each time a `Line` objects has it's `fitPoly` function called. Also, the passed in `fit` is generated by converting sample (x, y) points made from the line's current fit into meters and passing them in as parameters to the `np.polyfit` function.

##### Vehicle Position w.r.t. Lane Center

Calculating the vehicle position with respect to the lane center is done in the `carDistanceFromCenter` function found in the first cell under the "Car Distance from Lane Center" heading of the [Main](./main.ipynb) IPython Notebook.

In order to calculate the distance between the car position and the center of the lane, the lane center position in world space and the car position in world space must be determined. The `laneCenterPosition` is the midpoint between the left and right polynomial fits and is calculated by applying the left and right polynomials to the same y-value (in world space), summing the resulting x's, and taking their average. The `carPosition` is assumed to be in the middle of the image and is therefore calculated by dividing the width of the image (in world space) by 2. The equation for the resulting `carDistanceFromCenter` is found below:

```python
    carDistanceFromCenter = carPosition - laneCenterPosition
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Here is an example of the end result of applying my pipeline to a test image:

![Pipeline Result][pipeline-result]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

##### The Approach

The approach I took involved transforming a given image to a warped perspective (one that captures the lane lines) and converting to HLS Full color space and accentuating all white and yellow colors. Separately, a Sobel gradient is taken along the x-direction. I combine the results of the gradient and color thresholding into a binary image.

This resulting binary image is ran through a lane line detection algorithm that applies a window search or a previous detection search to locate the (x, y) points that correspond to each lane line. These points are then used to generate polynomials that represent each lane line. The polynomials are then mapped to the image and a lane visualization is created on the binary image. This binary image, which only holds the lane line visualization and the mapped polynomials, is warped back into a normal perspective and overlayed over the original image.

##### Future Improvements / Causes of Potential Failure

Possible factors that can cause failure include: images that are not the width and height that this pipeline was tuned on, a line of cars in the lane could throw off the detection, changing lanes, and frequent sharp curves in varying directions. Selecting the source and destination points was done statically based on the width and height of the project video images. Fixing this issue would entail running a pipeline that does not leverage a warped perspective to detect the intial lane lines and then using those lines to determine the source and destination warp points for subsequent images. 

One way to make the pipeline more robust would be to apply weights to each detected lane line polynomial instead of taking the average of the last few polynomials. Weights could be determined from a combination of how much the new polynomial varies from the previous ones and how abnormal the radius of curvature of it is. Also, if one lane line is found to be abnormal and the other is not, perhaps the expected lane line could contribute to the weight of the abnormal lane line.
