## Writeup


**Advanced Lane Finding Project**

In this notebook we will explain how to clearly isolate the portion of the road between two lane lines, on a road. 

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images. Our camera lenses distort the way the geometry appears in reality. We must recalibrate it using a chessboard in order to transform the feed into images fathfull to ground truth.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image. A mix of the Sobel filter and S gradient is used. 
* Apply a perspective transform to rectify binary image ("top-down view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./picturesWriteup/chessboards.jpg
[image2]: ./picturesWriteup/original.jpg
[image3]: ./picturesWriteup/sobel.jpg 
[image5]: ./picturesWriteup/bestfit.jpg 
[image6]: ./picturesWriteup/final.jpg
[video1]: ./project_video_solution.mp4 

### Camera Calibration


The code for this step is contained in the first code cell of the notebook. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]


I used a combination of color and gradient thresholds to generate a binary image. In the `pipeline()` we use both sobel filters and the S (saturation) channel in order to isolate pixels which are part of the lane line. The parameters have been chosen to maximize the contrast between lane lines and the rest of the image. Here's an example of my output for this step:

![alt text][image3]

The code for my perspective transform includes a function called `perspective_change()`, which appears in the 5th code cell of the IPython notebook.  The `perspective_change()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
            [[280,  700],  # Bottom left
             [595,  460],  # Top left
             [725,  460],  # Top right
             [1125, 700]]) # Bottom right # Bottom right
dst = np.float32(
            [[250,  720],  # Bottom left
             [250,    0],  # Top left
             [1065,   0],  # Top right
             [1065, 720]]) # Bottom right   
```


Then I applied the perspective transform to the binary image. This enabled me to create a histogram with each bin containing the cumulative amount of detected pixels in the bottom half of the image. Taking the two peaks gives us the approximate position of each lane line. From there, step by step, we go up the image looking at all points around a certain region of our estimated lane line and we recenter that region around the mean value of points found before going to the next step. When we reach half the height of the image (aproximately where the horizon line is), we plot the best fit line which goes through our detected points. This best fit is a second order polynomial. 

![alt text][image5]



Having our second order polynomial enables us to establish the curvature of the lanes. This is done in the `measure_curvature_real()`. The input parameter is a binary image which has gone through our `pipeline()` method. Here we determine the curvature of the lanes. We solve for the line curvatures with mathematical formula for curvature and converting the result in meter values.


I implemented the final result of displaying the lane as a green region in the `draw_lanes()` method. This is then paired with the `add_metrics()` method which displays the lane line curvatures and the offset of the car between the lanes.

![alt text][image6]

---

### Pipeline (video)

Here's a [link to my video result](./project_video.mp4)

---

### Discussion


Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The main improvement which could be done to the pipeline would be to add a security feature which would take into account the previous frame's findings. Here, every frame is analyzed independently. This can cause issues when the car is going through changes in lighting or where lines are hard to identify.

Another improvement would be to do more analysis on the image filters. We could play around with the blue and green filter to put more weight on the yellow lines. Also the Sobel filter could be improved. Adjusting the prefered gradient directions for the right and left lines perpendicular to the previous line direction would be a good improvement. 
