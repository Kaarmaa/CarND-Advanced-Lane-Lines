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

[image1]: ./writeup/calibration2.jpg "Distorted"
[image2]: ./writeup/calibration2_ChessboardUndistorted.jpg "Undistorted"
[image3]: ./writeup/Projection.jpg "Projection"
[image4]: ./writeup/lane_pixels.JPG "Lane Pixel Isonlation"
[image5]: ./writeup/warp_back_down.jpg "Lane Polynomial"
[image6]: ./examples/color_fit_lines.jpg "Fit Visual"
[image7]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

All code referenced in this writeup is available in the Project4.ipynb file. This file also has the corresponding images and outputs included.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

Cells 2 - 4
Cell 2: Loaded images, and generated chessboard points
Cell 3: Calibrated the camera with the points from Cell 2
Cell 4: Applied camera calibration to the real world, test images

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
![Distorted Image][image1]
![Undistorted Image][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Cell 4:
Using OpenCV's calibrateCamera method in Cell 3, I used the outputs in Cell 4 to undistort the test image set

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Cell 5:
The first use of a color and gradient transforms is in Cell 5. This is admittedly, very basic, as I just needed to get the grayscale image from RGB for the Sobel filter. There are more interesting uses later on in the project where HLS colorspaces are used to help with binary thresholding of images.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Cell 6: 

The code for my perspective transform includes a function called `perspective_swap()`, which appears in Cell 6. This function takes as inputs an image (`img`), as well as optional params swap_back, draw_regions, and plot_enable.  

I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 45, img_size[1] / 2 + 100],
    [((img_size[0] / 6) + 80), img_size[1]],
    [(img_size[0] * 5 / 6) - 30, img_size[1]],
    [(img_size[0] / 2 + 45), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. HOWEVER, parallel lines only looked nice to a human eye. The predefined limits allowed too much extraneous data on the edges that was causing additional issues. It is for this reason that new src points were chosen, and do not guarantee a perfrectly parallel projection.

![Projection Comparison][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Cell 10 / 11:
Cell 10 includes the `detect_lane_edges()` function. It is in this function that I implemented a windowing search and processing to isolate points that belong to the edges of the lane. I iterate over these windows, accumulating the points, and performning a 2nd order polynomial fit. This function returns the coefficients to the calling function.
Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

Cell 11 includes the manual test section, and both the plain binary thresholded image `src` is available, as well as a second image with the windowing scheme (green), Pixels associated to lane markings (Red / Blue for L/R), and the histogram used to place the first window is in pink on the bottom.

![Lane Pixel detection][image4]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Cell 12:
Cell 12 has the `calc_curve_radius()` function. The inputs are the coeffients for each side of the lane(output from `detect_lane_edges()`), as well as a pixel index indicating the center pixel of the parent image. This is needed to convert from px distance from center image to a real world measurement in meters from center of lane.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Cell 13 - 15
Cell 13 has the `LanePoly()` function which is the parent function for most functionality highlighting the lane. It will call functions like to `detect_lane_edges()` to retrieve the polynomials describing the edges of the lane, as well as build the actual polygon overlay. It also handles calling the function to calcaulate curve radius and lane offset of host.
Cell 14 is the test code for generating a warped polygon and unwarping it, back into the same projection as the test image.
Cell 15 takes the lane masks from Cell 14 and overlays them onto the test image.

![Lane Polygon][image5]

#### 7. Additional Cells.
Cell 16 defines a helper function of overlay the outputs for L/R radius as well as center offset calculations

---
### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
Cell 17:
Cell 17 has the definition of the Video pipeline, cleaned up from the debug cells before.
Here's a [link to my video result](./project_output.mp4)

![Video][video1]
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

When looking into the project, I made a decision up front to try and achieve acceptable performance with a frame by frame approach, not relying on other frames than the current one. If this could be accomplished (it was), then additional stability can be implemented with historical data to improve the performance, rather than using it as a crutch to make it just "good enough". 

#### Future Improvements 

In order to deploy this on a vehicle, more testing will be needed with night data. I am concerned about the lumninance filter being too high for night scenarios and losing lane information. 

There also needs to be a better way to handle non-level host position. When going on the overpass in the given scenario, one can see the car bounce, and this causes an immediate degredation in the quality of the calculated lane.
#### Rework Notes
1) Changes made to fix lane deviations. I determined that shadows were causing more issues than expected, so I added a lumninance filter to the binary masked image generation block. This eliminated a lot of noise around the pixels of interest.
