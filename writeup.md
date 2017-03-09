#**Finding Lane Lines on the Road** 

##Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images/.jpg "Grayscale"

---

### Reflection

For all details see P1.ipynb for complete code.
Processed videos can be found in test_videos with _processed.mp4 suffix.

###1. Lane finding Pipeline. process_image_core function

Pipeline is very similar to what we have in the lesson description / quiz.

Steps:

1. In this project we only care about lane lines detection, so first the method filters the colors we don't care about - I chose to only keep the colors from [220, 170, 0] to [255, 255, 255].
2. Convert resulting image to grayscale.
3. To smooth the image Gaussian smoothing with kernel = 5 is applied.
4. Next Canny edge detection is applied to 
5. In this project we only care about lane detection on the highway, where the curvature of the lane lines is very little and can be ignored here, thus we can easily see that car is placed at the same place throught its movement along the lane. That allows us to keep a certain area of interest instead of dealing with all the image space. I chose to use the same 4-vertices polygon for detection.
6. Use Hough Lines detection in order to detect straight lines in the array of edges we got from Canny.
7. Once we have candidate line segments next step is to fit them with two simple straight lines - left border of the lane and the right border. Again, due to the nature of the project - detecting lane on the highway, we can apply some filtering in order to eliminate line segments that couldn't be part of the lines - I chose to devide the image in half horizontally - left line can only consist of pixels with X part < ImageWidth / 2, for right line : X part > ImageWidth / 2. We also shouldn't consider line segments that have incorrect slopes - I chose to ignore slopes less than 0.5 due to the nature of highway detection.


![alt text][image1]


###2. Potential shortcomings with your current pipeline

1. Current pipeline applies colorRange filtering to original image - that disables any detection of lanes colored in different to yellow-white ranged colors. Disabling other colors also means this pipeline can only be applied for detecting lanes - other objects like traffic signs, cars, pedestrians, etc are ignored if colored differently.
2. Pipeline is constructed with a strong assumption that we're moving on a highway - that allows us to ignore line curvature, which makes it of almost no use for in-the-city lane detection.
3. Pipeline will work incorrectly if there are cars that merge into our lane as part of the lines will be blocked for detection (altough the right way to merge is to keep a good distance ahead of the car on the lane before merging).


###3. Suggest possible improvements to your pipeline

1. As can be seen in the processed videos the lines of the lane are not fully stable - lines are bit shaky (solid line is more stable), especially seen on the right line of processed challenge video. One way to improve it is to keep moving average accross the lines - keep an array of the (k,b) line parameters for 3-10 previous frames and before finilazing the (k,b) for current frame average it across the past n pairs : k_current = average(k[past 3-10 frames] + k_current). It will smooth noizy data and incorrectly detected random line segments.
2. 
