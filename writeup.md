## Writeup Template

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

[image1]: ./dist.jpg "Undistorted"
[image2]: ./dist2.jpg "Road Transformed"
[image3]: ./thre.jpg "Binary Example"
[image4]: ./thre2.jpg "Binary Example"
[image5]: ./warp.jpg "Warp Example"
[image6]: ./fitting.jpg "Fit Visual"
[image7]: ./result.jpg "Output"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  
[Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 3rd code cell of the IPython notebook located in "./AdvancedLaneLines.ipynb" .

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. 
Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. 
Thus, `objpoints` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  
`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  
I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  
Provide an example of a binary image result.

I used a combination of color(HLS, Luv, Lab) to generate a binary image (thresholding steps at 8,9,10,11th cells in `./AdvancedLaneLines.ipynb`).  
Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]


This is combined image.
![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I applied region of interest mask to image before perspective transform because fitting line goes well.
The code for my perspective transform includes a function called `perspective_transform()` and `fitting_line()`, 
which appear in 10th and 11th cells in the file `./AdvancedLaneLines.ipynb`.
I chose the hardcode the source and destination points in the following manner:

```python

    ysize = img.shape[0] #720

    (x1, y1) = (275, 670)
    (x2, y2) = (1030, y1)
    (x3, y3) = (725, 480)
    (x4, y4) = (550, y3)

    src = np.float32([[x1,y1],[x2,y2],[x3,y3],[x4,y4]])        
    dst = np.float32([[340,ysize], [940, ysize], [940, 0], [340, 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 275, 670      | 340, 720        | 
| 1030, 670      | 940, 720      |
| 725, 480     | 940, 0      |
| 550, 480      | 340, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and 
its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified lane-line using the histgram and window. Non-zero pixel in the window is checked and then these pixels are fitted by polynominal fitting function using 'numpy.polyfit()'.
Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image6]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in 11th cell in the file `./AdvancedLaneLines.ipynb`.
The calculation of curvature is refferd http://www.intmath.com/applications-differentiation/8-radius-curvature.php.
And the position of the vehicle with respect to center is calculated by 0.5 * (leftline position + rightline position - image origin-camera origin offset) * coefficient of pixel and meter.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 15th cell in the function `fillPolyLane()`.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video 
(wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here is techniques I used. Region of interest, averaging curve and initial value for seaching lane line
because they make lane detection more robust.
It's difficult to detect line when the car is in the boundary between sun and shade.
When the other car is overlapping the lane line, it's difficult to detect line too.
In this case, to avoid miss-detection, sanity check is important.(for example, comparing the calculated curve with previous one)

