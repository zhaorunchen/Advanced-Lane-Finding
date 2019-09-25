## Advanced Lane Finding

### This project includes two main parts, camera calibration and advanced lane finding. 

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

[image1]: ./Calibration_Result.png "Undistorted"
[image2]: ./Undistort_Scene.png "Road Transformed"
[image3]: ./Binary_line_detection_result.png "Binary Example"
[image4]: ./Perspective_view.png "Warp Example"
[image5]: ./Fitting_result_a.png "Fit Visual"
[image6]: ./Curvature_equation.png "Curvature calculation"
[image7]: ./Curvature_result.png "Curvature result"
[image8]: ./Warp_back_result.png "Warp back result"
[video1]: ./project_video.mp4 "Video"
[video2]: ./challenge_video_result.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points


---


### Camera Calibration ([CSDN link](https://blog.csdn.net/lql0716/article/details/71973318))

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./examples/example.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Apply distortion correction on in-scene image
#### Provide an example of a distortion-corrected image.

To demonstrate this step, I simply use the results from "Camera Calibration" to obtain the distortion corrected test images as shown below:
![alt text][image2]


#### 2. Binary line detection using combination of different Sobel results
#### Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at cell 3 in `Advanced Lane Detection.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)
The detection approach is mainly based on Sobel [openCV link](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_imgproc/py_gradients/py_gradients.html). There are serval things needed to know when using Sobel
1) data type should be float
2) results should be converted to absolute value
3) finally, normalize results and scale to 255. This is to convert value to uint8

To avoid the issue in first project, we convert RGB to HLS space and grab Saturation channel. Then we calculate Sobelx and Sobely. There are serval ways can be combined to detect the line:
1) Line detection using gradient magnitude
2) Line detection using angle 
3) Line detection using color thresholding

In my project, I combined the Sobelx(Sobel results along x direction) and color thresholding. The result is shown below:
![alt text][image3]

#### 3. Apply perspective transformation on in-scene image
#### Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`. The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[490, 482], 
     [810, 482],  
     [1250, 720],  
     [40, 720]]) 
dst = np.float32(
    [[0, 0], 
    [1280, 0], 
    [1250, 720],
    [40, 720]]) 
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 490, 482      | 0, 0          | 
| 810, 482      | 1280, 0       |
| 1250, 720     | 1250, 720     |
| 40, 720       | 40, 720       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Identify lane-line pixels
#### Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

For this task I applied a convolution find_window_centroids() based on the help from [juano2310](https://github.com/juano2310/CarND-Advanced-Lane-Lines-Juan/blob/master/README.md). This method is to maximize the number of "hot" pixels in each window.

By sliding user defined window template across the image from left to right, we sum all the overlapping values together, creating the convolved signal. The peak of the convolved signal determines the highest overlap of pixels and these pixels have higher probability to be the "lane marker" pixels.

The result is shown below:
![alt text][image5]

#### 5. Calculate radius of curvature of the lane and current position of the car
#### Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
As shown in the equation below:
![alt text][image6]
The curvature and the location of the car related to the road center can be calculated, the result is shown below:
![alt text][image7]

#### 6. Project result back to the original image (fill the area with color)
#### Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
After detecting and determing the left and right "lane marker" pixels, we can fill the area between these two lines with color. The test result is shown below:

![alt text][image8]

---

### Pipeline (video)
The code is based on [juano2310](https://github.com/juano2310/CarND-Advanced-Lane-Lines-Juan/blob/master/README.md), which provide additional approach which makes sure that the lanes have a valid separation, and if this fails, the method can draw the previous estimation.
Here's a [link to my video result](./project_video.mp4)

Additional result using challenge video is shown here: [link to my video result](./challenge_video_result.mp4)

---

### Discussion

In thie project, I followed the approach taught in the lecture to finish the camera calibration and binary detection. The curvature calculation and video generation were stuided based on [juano2310](https://github.com/juano2310/CarND-Advanced-Lane-Lines-Juan/blob/master/README.md). The idea and math is relatively straight forward, but coding is one of the difficult part to me.
There is a concern about my approach:
src area selection and Sobel feature selection are mainly based on heavy manually adjustments. Therefore the robustness may not be stable enough.

