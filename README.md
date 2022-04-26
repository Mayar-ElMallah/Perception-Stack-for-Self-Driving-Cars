# Perception-Stack-for-Self-Driving-Cars
In this project we are going to create a simple perception stack for self-driving cars (SDCs.) Although a typical perception stack for a self-driving car may contain different data sources from different sensors (ex.: cameras, lidar, radar, etc...), we’re only going to be focusing on video streams from cameras for simplicity. We’re mainly going to be analyzing the road ahead, detecting the lane lines, detecting other cars/agents on the road, and estimating some useful information that may help other SDCs stacks. The project is split into two phases. We’ll be going into each of them in the following parts.

## Phase 1 - Lane Line detection:
In any driving scenario, lane lines are an essential component of indicating traffic flow and where a vehicle should drive. It's also a good starting point when developing a self-driving car!The lane detection system was written in Python using the OpenCV library. Here's the current image processing pipeline:
1) Distortion Correction
2) Perspective Warp
3) Filteration to get the binary image
4) Histogram Peak Detection
5) Sliding Window Search
6) Curve Fitting
7) Overlay Detected Lane
8) Apply to Video

## Distortion Correction
Camera lenses distort incoming light to focus it on the camera sensor. Although this is very useful in allowing us to capture images of our environment, they often end up distorting light slightly inaccurately. This can result in inaccurate measurements in computer vision applications. However, we can easily correct this distortion.

How would you do this? You can calibrate your image against a known object, and generate a distortion model which accounts for lens distortions. This object is often an asymmetric checkerboard, similar to the below Images:
### Why is the checkerboard pattern so widely used in calibration?
Checkerboard patterns are distinct and easy to detect in an image. Not only that, the corners of squares on the checkerboard are ideal for localizing them because they have sharp gradients in two directions. In addition, these corners are also related by the fact that they are at the intersection of checkerboard lines. All these facts are used to robustly locate the corners of the squares in a checkerboard pattern.

![This is an image](writeUp/output_3_1.png)
## Camera Calibration
Camera Calibration is a necessary step before processing Images taken by the camera. Every Camera has some irregularities and tend to distort images to some extent. This can be due to the uneven curvature, material of lens, environment irregularities etc.

So after loading the images we calibrate the camera with them. Open CV provides some handy functions like findChessboardCorners and calibrateCamera to help us do this.  
  ```
  Original and calibrated images
  ```

![This is an image](writeUp/output_1.png)

### Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

To calibrate the camera, First of all I imported the chessboard Images, and found their corners using the findChessboardCorners method. I also initialized the obj point as objp

I kept finding corners and appended them to an array imgpoints and obj point to an array objpoints. Then I provided these as input to the calibrateCamera method, which returned a matrix. This matrix can now be used to undistort any image using the undistort method of OpenCV. This code is written in the cell 4 of my python notebook.
Undistortion

In cell 6 I used the same matrix to undistort some test Images too These are some Images after Distortion Correction.
![This is an image](writeUp/output_2.png)
## Color space conversion
In this step,I am seeking to get rid of the shadow and brightness of the sun and get the appropriate color channel (with good info about lane edges).
![This is an image](writeUp/output_4.PNG)
from the visualization,I deduced that the appropriate channel for our project is the ***S-channel***.
## Transformation warp
For being hard to detect curved lines in this space so i need to transform the images by the prespective transformatin which enables us to look at them from bird eye view. 

To do this task first I tried several points to be source points (from the original images) and decided their corresponding points at the output
then I chose the good set for our case. at the end I used OpenCV methods to do this transformation 
![This is an image](writeUp/output_5.PNG).

## Experiment Different Colors Channel
We tried converting the Image to various color spaces for detecting the lane lines better. We tried RGB, HSV, HLS, Lab and YCrCb color spaces.
![This is an image](writeUp/output7.PNG)

