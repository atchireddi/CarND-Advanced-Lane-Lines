 
# Advanced Lane Finding Project

## Goals:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./test_images/warped_straight_lines1.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

 
 ### Pipeline Overview
 
 ```
 # pipeline
 def process_image(img):
    
    # Undistort image
    undist_img = undistort(img)             # color_img in & out
    
    # Generate binary image from color and gradient
    binary_img = binary_image(undist_img)   # color_img in & binary_img out
    
    # Apply Mask
    masked_img = apply_mask(binary_img)      # binary_img in & binary_img out
    
    # Transform masked image 
    warp_img = warp(masked_img,M)          # binary_img in & binary_img out 
    
    # Mark Lane with green color
    lane_img = mark_lane(warp_img)          # binary_img in & color_img out
    
    # Unwarp lane image
    unwarp_lane_img = warp(lane_img,Minv)   # color_img in & color_img out
    
    # Merge Lane image on to original image
    output = cv2.addWeighted(img, 1, unwarp_lane_img, 0.5, 0.0)   # color_img in & color_img out
    
    return(output)
 ```
 
 
 Let's look at the the details of each  pipeline stage.
 
###Camera Calibration

  The code for this step is contained in the first code cell of the IPython notebook located in "./AdvancedLaneLines.ipynb"

  I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

  I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 




![alt text][image1]

### Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines # through # in `another_file.py`).  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]




#### 3. Perspective Transform

Perspective transformation is implemented as two functions,
  
    - `gen_transform_matrxi()`  :  Computes transformation matrices.
    - `warp()` :                   Takes in image, applies tranformation.
    
    Code can be found in `AdvanacedLaneLines.ipynb`. 
    
The `warper()` function takes inputs an image, as well as transformation matrix(inverse transformation matrix) and
returns warped/unwarped image.  
    
Having a own function for generating transformation matrix, can generate transform matrix once and re-use it for
transforming each frame. I chose the hardcode the source and destination points in the following manner:
    

Below are source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 582, 455      | 300, 0        | 
| 700, 455      | 1000, 0       |
| 1150, 720     | 1000, 0       |
| 150, 720      | 300, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Code can be found in `perspective_transform.ipynb`




![alt text][image4]




####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]




####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`



#### 6. Example image with lane area:

This functionality is split across functions `process_image()` and `mark_lane()` in `AdvancedLaneLines.ipynb`. `mark_lane()` returns an warped lane marking image, which is unwarped and combined with original image in `prcess_image()`

Here is an example of my result on a test image:

![alt text][image6]




### Pipeline Output On Video Stream (video)

Here's a [link to my video result](./output.mp4)




### Discussion

##### Challenges:
   - Lane line identification from high contrast background roads.
   - Filtering noise from tree shades
   - Perspective transformation of curvy lanes.
   - Targetted lane search for curvy lanes.
   - Fitting polynomial with fewer identified lane pixels   

##### Drawbacks:
   - Poor transformation of Hairpin bend lanes.
   - Hairpin bend lanes can fool targetted lane search.
   - Identifying lane lines under varying weather conditions.
   
