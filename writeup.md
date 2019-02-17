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

[image1]: ./output_images/undistorted_chess.jpg "Undistorted"
[image2]: ./output_images/undistorted_test.jpg "Road Transformed"
[image3]: ./output_images/binary_test.jpg "Binary Example"
[image4]: ./output_images/warped_test.jpg "Warp Example"
[image5]: ./output_images/masked_test.jpg "Region of interest"
[image6]: ./output_images/detectedlines_test.jpg "Fit Visual"
[image7]: ./output_images/final.jpg "Output"
[video1]: ./output_video/project_video.mp4 "Video"


## Camera Calibration

The code for this step is contained in the section *Camera calibration* of the IPython notebook located in "advanced_lane_finding
.ipynb".  

In this section, I computed the camera calibration matrix and distortion coefficients given a set of chessboard images. First, I defined the number of inside corners in x and in y as 9 and 6 respectively. Then, I imported the set of chessboard images and called the ``find_corners`` function that returns the object and image points lists from the chessboard images. The ``find_corners`` function uses ``cv2.findChessboardCorners`` in each image to find the image points. The same object points array is replicated in object points list, once I assumed the chessboard is fixed on the (x, y) plane at z=0.

Finally, I call the function ``cal_undistort`` that receives the objpoints and imgpoints and calculate the calibration matrix and distortion coefficients of the camera using the function ``cv2.calibrateCamera``. So, I use the ``cv2.undistort`` function to apply this calibration and distortion correction to a test image and the result is shown in the image bellow.

![alt text][image1]

## Pipeline (single images)

### Distortion correction

The code for this step is contained in the section *Distortion correction* of the IPython notebook located in "advanced_lane_finding
.ipynb".

In the first step of my pipeline, I applied the function ``cv2.undistort(img, mtx, dist, None, mtx)`` to one of the test images. The undistorted image result is shown below.

![alt text][image2]

### Binary image

The code for this step is contained in the section *Binary image* of the IPython notebook located in "advanced_lane_finding
.ipynb".

I used a combination of white and yellow color thresholds to generate a binary image. Here's an example of my output for this step. 

![alt text][image3]

### Perspective transformation

The code for this step is contained in the section *Perspective transformation* of the IPython notebook located in "advanced_lane_finding.ipynb".

I used the function ``unwarp(img,src,dst)`` to performe a perspective transform. This function takes as inputs an image (img), as well as source (src) and destination (dst) points and return a warped image, a transfrom matrix, and an inverse transfrom matrix. I chose the source and destination points manually, as shown bellow.

```python
# source coordinates
src = np.array([
        [img.shape[1] * 0.6, img.shape[0] * 0.65],
        [img.shape[1]*0.94, img.shape[0]],
        [img.shape[1]*0.09, img.shape[0]],
        [img.shape[1] * 0.44, img.shape[0] * 0.65],
    ],dtype='float32')

# Destination coordinates
dst = np.array([
        [img.shape[1] * 0.8, 0],
        [img.shape[1] * 0.8, img.shape[0]],
        [img.shape[1] * 0.2, img.shape[0]],
        [img.shape[1] * 0.2, 0],
    ],dtype='float32')
```

This resulted in the following source and destination points:

| Source 	| Destination 	|
|:------------:	|:-----------:	|
| 768., 468. 	| 1024., 0. 	|
| 1203.2, 720. 	| 1024., 720. 	|
| 115.2, 720. 	| 256., 720. 	|
| 563.3, 468. 	| 256., 0. 	|

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

### Find lane lines

The code for this step is contained in the section *Find lane lines* of the IPython notebook located in "advanced_lane_finding.ipynb".

To find the lane lines my first step was to create a region of interest mask to focus only in the road in front of the vehicle, as shown below.

![alt text][image5]

Then, I used the histogram to find the peaks, which represents the left and right lane positions. After that, I used sliding windows to identify left and right line pixel positions. So, I Fitted a second order polynomial to each line. The result image is shown below

![alt text][image6]

### Radius of curvature

The code for this step is contained in the section *Radius of curvature* of the IPython notebook located in "advanced_lane_finding.ipynb".

I used the function ``measure_curvature_real`` to calculate the radius of curvature in meters for both lane lines. As for the distance from the center I used the code below.

```
center = ( ( ( (left_fit[0]*(y_eval**2) + left_fit[1]*y_eval + left_fit[2]) +
            (right_fit[0]*(y_eval**2) + right_fit[1]*y_eval + right_fit[2]) )/2.0) - (warped.shape[1]/2.0))*3.7/700
```

### Draw lines

The code for this step is contained in the section *Draw lines* of the IPython notebook located in "advanced_lane_finding.ipynb".

I used the function ``draw_lines`` for overlaying the lane area on the original image. Here is an example of my result on a test image.

![alt text][image7]

---

## Pipeline (video)


Here's a [link to my video result](./output_video/project_video.mp4)

---

### Discussion

The color and gradient thresholds don't work well with shadows and high luminosity on the road. I spent a lot of time trying different combinations of thresholds but I wasn't able to find a perfect threshold combination. This approach works better than the first project but yet suffers detect lane lines when the road surface color changing. 

