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

[Undistorted]: ./output_images/undistorted0.png "Undistorted"
[Road Transformed]: ./output_images/test1.jpg
[Binary Example]: ./examples/binary_combo_example.jpg
[Warp Example]: ./examples/warped_straight_lines.jpg
[Fit Visual]: ./examples/color_fit_lines.jpg
[Output]: ./examples/example_output.jpg
[Video]: ./project_video.mp4
[Completed Video]: ./project_video_output.mp4

---
### Camera Calibration

The code for this step is contained in the camera_calibration method of the IPython notebook "model.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

[Calibrated](/Lane-Finding/tree/master/output_images/calibrated0.png)

### Pipeline (single images)

#### 1. Distortion-correction

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

[Undistorted](/Lane-Finding/tree/master/output_images/undistorted0.png)

#### 2. Color transforms, gradients and other methods to create a thresholded binary image

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in the `transform_image` method). I used a mask to find all yellow objects. Also, I thresholded the S channel of the image. Lastly, I used a combination of Sobel, Magnitude and Direction of the gradient to find lane lines. Here's an example of my output for this step.
undistorted0.png 

[Thresholded](/Lane-Finding/tree/master/output_images/threshold0.png)

#### 3. Perspective Transform

The code for my perspective transform includes a function called `perspective_transform`. I chose the hardcode the source and destination points in the following manner:

```
# 4 source coordinates for transform
    p1 = [732, 460] # topRx, topRy INC
    p2 = [width, height - offset_y]# bottomRx, bottomRy
    p3 = [0, height - offset_y]# bottomLx, bottomLy
    p4 = [546, 460]# topLx, topLy DEC
    
    # the source points for the lane lines 
    src = np.float32([p1, p2, p3, p4])
    
    # the destination points that define how the new image should look
    # [350, 350], [930, 720], [350, 720], [930, 350]
    dst = np.float32([[width, 0],
                      [width, height],
                      [0, height],
                      [0, 0]])

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 732, 460      | 1280, 0        | 
| 1280, 710      | 1280, 720      |
| 0, 710     | 0, 720      |
| 546, 460      | 0, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

[Perspective Transform](/Lane-Finding/tree/master/output_images/perspective0.png)

#### 4. Identifying Lane-Line Pixels and Fit their positions with a polynomial

I created a histogram of the lower half of each transformed image. I found the 2 locations where there pixel count was highest and set them as the start of each lane.  For the first image, I started from scratch and searched from the bottom of each image in the `Find_Lane_Start` method. Then, I used a sliding window approach to search for the rest of the lane. Once all the lane pixels were found, I used numpy to fit a 2nd Order polynomial to each line. After the first image, I used the values calculated from the previous image to find the lanes again in the `find_lane` method.

[Lane Lines](/Lane-Finding/tree/master/output_images/plotted-birdseye.png)

#### 5. Radius of Curvature of the Lane and Center Offset.

I did this at the bottom of the `Find_Lane_Start` and `find_lane` methods. Anytime the radius in higher than 1100m, I set the value to zero. This is because as a line becomes straighter, the radius approaches infinity, which makes no sense when you're driving.  

#### 6. Final Output.

In the `illustrate` method, I take all the information that has been collected and combine it into the final output image.

[Final Output](/Lane-Finding/tree/master/output_images/final_output.png)

---

### Pipeline (video)

Here's a [link to my video result](Lane-Finding/project_video_output.mp4)

---

### Discussion

The hardest part of this project is finding the right values for the thresholds and gradients. I played around with doing perspective transforms on the original thresholded images and thresholding transformed images. It's hard to say that there is a perfect way to find lanes. The experimentation process took forever. Also, there is a  considerable amount of time taken to process each image which makes the whole pipeline very slow when running on just a CPU. It is much faster on GPUs.
