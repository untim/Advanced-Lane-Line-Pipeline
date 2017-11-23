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

[image1]: ./output_images/binary_combined.png 
[image2]: ./output_images/Chessboard.png  
[image3]: ./output_images/Distorted_test_image.png 
[image4]: ./output_images/final_img.png 
[image5]: ./output_images/Undistorted_Chessboard.png 
[image6]: ./output_images/Undistorted_test_image.png 
[image7]: ./output_images/warped_binary.png 
[image8]: ./output_images/lines_detected.png 
[video1]: ./output_images/project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the third and fourth code cell of the IPython notebook located in “./project.ipynb"

i started by setting the corners of my chessboard.
Next i prepared the "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` and the data which is received from the cv2.calibrateCamera() function and obtained this result for the distortion corrected image. Below you see the distorted and undistorted chessboard. 

“Chessboard”
![alt text][image2]
“Undistorted Chess Board”
![alt text][image5]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image3]

Actually most of the work has been done already in the camera calibration section. I used the outputs of the camera calibration and used cv2.undistort function with the outputs and distorted the test images.
The function used in the pipeline is then the undistort(img) function.
This is how the undistorted test image looks like:

![alt text][image6]


#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The whole process can be seen in between code cells 9-14:
To get the best overview what channels to take I first started to separate both HSV and HLS into their respective channels and experimented with all thresholds and all threshold differences. Then I choose the ones which looked the most promising for me. Next I also used sober operator which also gave promising results.
The next step was then to combine the promising HSV channel and its appropriate threshold with the sober operator threshold which gave a very good binary output.
The function used in the pipeline is then the binary_image(img) function. The output can be seen here:

![alt text][image1]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

First of all I set tried to find appropriate source points which I found with an exteral photo application to find the pixels. Then I also set the destination for the for the warping the pictures later on. I kept the photo source and destination points static as the images are all the same size. The table can be seen below

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 702, 460      | 1000, 0        | 
| 1010, 663     | 1000, 650      |
| 285, 663     | 200, 650    |
| 576, 460      | 200, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Then I used the cv2.getPerspectiveTransform to get M which is later needed for the cv2.warpPerspective to actually warp the picture.
This is how the final binary picture looks once the perspective has been transformed

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I implemented this in cell 18 and 21.

Mostly I used the code which was already given by Udacity. But in general the code uses a histogram in the perspective transformed photo to search for the area with most pixels. The picture is then separated into different areas. This knowledge is used as a starting basis to identify further sections of the lane in the next areas.Once all locations have been determined we the the location of the x and y locations of the left and right lane and se np.polyfit to generate a polynomial with the least squared error term.
This than creates this:




![alt text][image8]


Furthermore after the line has been once found the pipeline uses a different function to only look around the margin of the line which was detected before. 
To further smooth the resulting line I took the average of the last 10 lines drawn which has an good effect in the later video,

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I implemented this in cell 20
To calculate the curvature of the the street I also used techniques introduced by Udacity. It was very important to first have a conversion from pixels to meters which was given by Udacity. Then the polynomial was again created with np.polyfit however this time with the conversion from pixel to m.
The function for converting a polynomial to the actual radius was given by a tutorial in the internet which used second degree and first degree derivatives. For the distance to the middle of the street I used the both x intercepts of the lines and calculated the mean of the intercepts which then is already in m. Next I subtracted half of the length of image which can also be transformed from pixel space to m. For that I used the fact which was given by the Udacity team that the camera is in the middle of the car. The resulting number already gives the distance from the middle of the lane.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in cell 22.
The draw lines function also not only warps the image back to how it normally looks but also draw the Curvature and distance from middle data on it.
The final image then looks like this:



![alt text][image4]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Heres the video:https://www.youtube.com/watch?v=nB7CaXLqDeo&feature=youtu.be

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

It was certainly difficult to find the right channels for the binary output.
Furthermore I had some problems with classes and creating objects at first but after I had understand the concept classes where very helpful especially for smoothing the lines and I can imagine that it they can be also very helpful later on for the more challenging videos. 
For the challenging I found it very tricky as in some images the current pipeline has no chance to differentiate between and white line and the black line which is not part of the lane. Furthermore the lane sometimes almost disappears as you drive below a bridge for example.
In the more challenging video I quickly noticed that the pipeline is also not made for many bends especially very sharp ones where nothing of the road ahead can be even seen anymore as the car is still pointing in the wrong direction before steering in the right direction,..
