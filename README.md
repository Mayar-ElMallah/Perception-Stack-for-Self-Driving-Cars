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

To calibrate the camera, First of all we imported the chessboard Images, and found their corners using the findChessboardCorners method. We also initialized the obj point as objp

We kept finding corners and appended them to an array imgpoints and obj point to an array objpoints. Then We provided these as input to the calibrateCamera method, which returned a matrix. This matrix can now be used to undistort any image using the undistort method of OpenCV. This code is written in the cell 4 of my python notebook.
Undistortion

In cell 6 We used the same matrix to undistort some test Images too These are some Images after Distortion Correction.
![This is an image](writeUp/output_2.png)
## Color space conversion
In this step,We are seeking to get rid of the shadow and brightness of the sun and get the appropriate color channel (with good info about lane edges).
![This is an image](writeUp/output_4.PNG)
from the visualization,We deduced that the appropriate channel for our project is the ***S-channel***.
## Transformation warp
For being hard to detect curved lines in this space so we need to transform the images by the prespective transformatin which enables us to look at them from bird eye view. 

To do this task first We tried several points to be source points (from the original images) and decided their corresponding points at the output
then we chose the good set for our case. at the end We used OpenCV methods to do this transformation 
![This is an image](writeUp/output_5.PNG).

## Experiment Different Colors Channel
We tried converting the Image to various color spaces for detecting the lane lines better. We tried RGB, HSV, HLS, Lab and YCrCb color spaces.
![This is an image](writeUp/output7.png)

![This is an image](writeUp/output_8.png)
Now, we are able to identify the lane lines easily, which are too bright to identify in the original image itself. We have tried those color channels for combining they were easily able to detect the lane lines and were almost free from noise.
## Sobel X and Y
We experimented on Sobel operator to check if it helps in identifying the lane lines. These are some examples of Sobel x applied on the warped images
![This is an image](writeUp/output_9.png)
If Images are not properly warped, the left lane line is completely getting misidentified. Sobel identifies road edge as the lane line. This is due to the low contrast between lane line and the bright road in these two images. However this gets better after removing the road edge from the warped picture.

## Sobel Magnitude
These are some pictures of experimentation on the warped Images using sobel magnitude
![This is an image](writeUp/output_10.png)
```
We can't see any improvement in lane detection using sobel magnitude also. 
Sobel is not able to detect low contrast lane lines and hence will might 
fail in bright road conditions.
```
## Sobel Gradient
These are some images of experimentation using sobel gradient. We tried to filter out some noise using the arctan operator to reduce the near horizontal lines from the Image, however this introduced a lot of noise of its own.
![This is an image](writeUp/output_11.png)
```
Gradient sobel in itself doesn't looks good enough to detect lane lines.
Also there is lot of noise in the images. We'll further try to combine the sobel 
techniques along with the color channels to detect lane lines better and to suppress
the detection of road edges in bright as well as dark conditions.
```
## Combining Channels
We tried combining sobel techniques and channel thresholds to get the binary image of detected lanes but finally deduced that lanes get detected best using the color channels and hence went with channel thresholding for lane detection.
![This is an image](writeUp/output_12.png)

## Histogram Peak Detection
We'll be applying a special algorithm called the Sliding Window Algorithm to detect our lane lines. However, before we can apply it, we need to determine a good starting point for the algorithm. It works well if it starts out in a spot where there are lane pixels present, but how can we detect the location of these lane pixels in the first place? It's actually really simple!

We'll be getting a histogram of the image with respect to the X axis. Each portion of the histogram below displays how many white pixels are in each column of the image. We then take the highest peaks of each side of the image, one for each lane line. Here's what the histogram looks like:
![This is an image](writeUp/output_13.png)

## Sliding Window Search and Fitting the Curve with Green Line ## Draw Lane
i followed those steps of algorithm for getting the line fitted on the lanes:
1. getting a histogram sum of the image pixel values
2. getting the starting position of both lanes from the left and right half of histogram.
3. divide the image into n steps and move two windows seperately over the starting 
   points of the lanes
4. for each window we Identify the nonzero pixels in x and y within the window
5. then we Append these indices to the lists
6. recenter the window to the mean of the previous window's non-zero pixels.
7. then after all the window steps, we extract the x and y location of the total selected pixels 
   and fit a second order polynomial to them.
8. Then we fit a line to it using the formula Ax^2 + Bx + C
9. The last step is to plot these lines using any suitable python libraries.
10. We can also plot the windows using the cv2.rectangle() method.

![This is an image](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/828fbca0b46b99e9772b95a0e0cc5a8a5a5c591a/writeUp/download.png)


![This is an image](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/3a7a9f1c53d8a2104eb91a3d2e875921813f4281/writeUp/download%20(1).png)

## Draw Lane
i followed those lines of code to Draw the Detected Lane Back onto the Original Image:

```
def draw_lane(original_img, binary_img, l_fit, r_fit, Minv):
    new_img = np.copy(original_img)
    if l_fit is None or r_fit is None:
        return original_img
    # Create an image to draw the lines on
    warp_zero = np.zeros_like(binary_img).astype(np.uint8)
    color_warp = np.dstack((warp_zero, warp_zero, warp_zero))
    
    h,w = binary_img.shape
    ploty = np.linspace(0, h-1, num=h)# to cover same y-range as image
    left_fitx = l_fit[0]*ploty**2 + l_fit[1]*ploty + l_fit[2]
    right_fitx = r_fit[0]*ploty**2 + r_fit[1]*ploty + r_fit[2]

    # Recast the x and y points into usable format for cv2.fillPoly()
    pts_left = np.array([np.transpose(np.vstack([left_fitx, ploty]))])
    pts_right = np.array([np.flipud(np.transpose(np.vstack([right_fitx, ploty])))])
    pts = np.hstack((pts_left, pts_right))

    # Draw the lane onto the warped blank image
    cv2.fillPoly(color_warp, np.int_([pts]), (0,255, 0))
    cv2.polylines(color_warp, np.int32([pts_left]), isClosed=False, color=(255,0,255), thickness=15)
    cv2.polylines(color_warp, np.int32([pts_right]), isClosed=False, color=(0,255,255), thickness=15)

    # Warp the blank back to original image space using inverse perspective matrix (Minv)
    newwarp = cv2.warpPerspective(color_warp, Minv, (w, h)) 
    # Combine the result with the original image
    result = cv2.addWeighted(new_img, 1, newwarp, 0.5, 0)
    return result,color_warp

```
##output result:
![This is an image](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/72a8d41519027db1eb426d07459a4791683ea105/writeUp/download%20(2).png)

## Calculating The Distance of the Car from Center and its Direction
Now we are going to calculate the center of the lane to find the distance of the car relative to the lane center.
First we will get the initial left lane position and the initial right lane position by subtitiude in the following polynomial equation 


![equation](http://www.sciweavers.org/tex2img.php?eq=f%28x%29%3DAx%5E2%2BBx%2BC%&bc=White&fc=Black&im=jpg&fs=12&ff=arev&edit=)


by averging the outputs from this equation we will find the center of lane. with assumption that the center of the car is half of the image width we can calculate the diffenernce between the car center and lane center.
For direction we can implement simple function check if the distance between the center of the lane and the center of the car is greater than 0,the car will be at the right of the lane else it will be at the left of it.
```
if center_dist > 0:
        direction = 'right'
    elif center_dist < 0:
        direction = 'left'
```

## Calculating Radius of Curvature
We can find the radius of curvature by fitting polynomials for right and left side of the lane then by using the resultant coefficients  we can calculate the curvature for each of them by the following equation 

![This is an image](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/main/writeUp/Screenshot%20(856).png)
 
 then by averging the two radii of curvature we will get the curvature radius of the lane 
## The Final Image
![This is an image](writeUp/18-draw_data.png)

## Problems faced during the project
There were a lot of challenges in the project. We have enlisted some of them with the solutions.

1) Too much Noise (Lightening & Shadow): We experimented on various color channels and selected the ones with the least noise.
2) Curvature of the lane: We solved it by Eye Bird View
3) Lane Detection: We solve it by Sliding Window
4) Radius, Position, lane values changing frequently: We applied averaging/smoothening over past few frames using queues to suppress the sudden changes in values.



## PipeLine(video)

Please find the link to the output video:
# Links
[Project video-input](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/29ac2f3257e433c54d1b2b2be1b6e30ff1e9b7b1/project_video.mp4)            /           [Project video-output](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/29ac2f3257e433c54d1b2b2be1b6e30ff1e9b7b1/project_video_output.mp4)



[Challenge video-input](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/29ac2f3257e433c54d1b2b2be1b6e30ff1e9b7b1/challenge_video.mp4)         /         [Challenge video-output](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/29ac2f3257e433c54d1b2b2be1b6e30ff1e9b7b1/challenge_video_output2.mp4)


[Harder challenge video-input](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/29ac2f3257e433c54d1b2b2be1b6e30ff1e9b7b1/harder_challenge_video.mp4)   /  [Harder challenge video-output](https://github.com/Mayar-ElMallah/Perception-Stack-for-Self-Driving-Cars/blob/29ac2f3257e433c54d1b2b2be1b6e30ff1e9b7b1/project_video_output3.mp4)


