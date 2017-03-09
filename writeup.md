#**Finding Lane Lines on the Road** 

##Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./output_images/output_1.png "1"
[image2]: ./output_images/output_2.png "2"
[image3]: ./output_images/output_3.png "3"
[image4]: ./output_images/output_4.png "4"
[image5]: ./output_images/output_5.png "5"
[image6]: ./output_images/output_6.png "6"
[image7]: ./output_images/output_7.png "7"

---

### Reflection

For all details see P1.ipynb for complete code.
Processed videos can be found in test_videos with _processed.mp4 suffix.

###1. Lane finding Pipeline. process_image_core function

Steps:

1 - We start with original image.

![image1]

2 - In this project we only care about lane lines detection, so let's filter the colors we don't care about - I chose to only keep the colors in [220, 170, 0] yellow to [255, 255, 255] white range.

![image2]

3 - Convert color-filtered image to grayscale.

![image3]

4 - Apply Gaussian smoothing with kernel = 5.

![image4]

5 - Apply Canny edge detection.

6 - In this project we only care about lane detection on the highway, where the curvature of the lane lines is very little and can be ignored here - we can assume the lane has the same shape throught the video and lane lines are straight lines. That allows us to keep a certain area of interest instead of dealing with all the image space. Simple 4-vertices polygon around the almost-constant lane shape was used to filter other parts of the image.

![image5]

7 - Use Hough Lines detection in order to detect straight lines in the array of edges we got from Canny. Again, due to the nature of the project - detecting lane lines on the highway - we can apply additional filtering to the line segments to ignore those that couldn't be part of the lines.

  7.1 - Initial filterting can be done by analyzing the slopes of line segments - as the lane shape is quite stable along the video we can filter too horizontal or too vertical line segments out as they cannot be parts of the line - I chose to filter only the horizontal ones by disguarding the line segments with the |slope| < 0.5 (less than 45 degrees).
  
  7.2 - Due to the position of the car strictly between the lane lines we can split the image in half (left and right) and consider segments with x coordinate < imageWidth / 2 as candidates for left line and segments with x coordinate > imageWidth / 2 as candidates for right line.

![image6]  

8 - Now we have a group of left line segments and a group of right line segments. Next step is to fit them with simple straight lines - left and right borders of the lane. Least squares method was used to fit the lines. As the result we get two pairs of parameters - (left_k, left_b) and (right_k, right_b), uniquely defining the borders of the lane (y = k * x + b).

9 - Final step is to draw the lane lines on the original image and output.

![image7]


After the pipeline was doing fine for lane detection on test images the main debugging started by running the same pipeline on videos. Whenever I was seeing issues like missing lines or incorrectly detected line segments I was modifying my pipeline to return not a final video but several steps earlier instead - to see how Canny edge detection works on video, what Hough line detection output looks like, etc. Closely analyzing weird one-off issues helped devising the final parameters for the algorighms used.

I wasn't using step 2 (color range filtering) originally for the first two test videos and was able to have very stable lane detection, but it failed instantly on the challenge video - dark shadow lines were interfering with the candidate line segments. That was the reason I chose to use color filterting.

Step 7.1 was also added after I started working on a challenge video - too many line candidates with incorrect slopes were being detected.

###2. Potential shortcomings with your current pipeline

1. Current pipeline applies colorRange filtering to original image - that disables any detection of lanes colored in different to yellow-white ranged colors. Disabling other colors also means this pipeline can only be applied for detecting lanes - other objects like traffic signs, cars, pedestrians, etc are ignored if colored differently.
2. Pipeline is constructed with a very strong assumption that we're moving on a highway - that allows us to ignore line curvature, but makes it of almost no use for in-the-city lane detection.
3. Pipeline might work incorrectly if there are objects that block lane lines - cars merging into the lane, dead animals, etc.
4. Hard weather conditions like heavy rain, snow, etc can interfere with the lane detection by video distorting - accumulated water on the road can cause light deflection.
5. Traffic signs placed on the road coatings (merge arrows, speed limit, etc) might be considered as lane line segments by pipeline and thus interfere with the actual lane detection.
6. Damaged road coating segments might be considered as line segments if matched by color (can happen under different light conditions).

###3. Suggest possible improvements to your pipeline

1. As can be seen in the processed videos the lane lines are not fully stable - lines are bit shaky (solid lines are more stable), this can easily be seen on the right line of processed "challenge" video. One way to improve it is to keep moving average accross the lines for line parameters - keep an array of the (k,b) line parameters for 3-10 previous frames and before finilazing the (k,b) for current frame average it across the past n parameters : k_current = average(k[past 3-10 frames] + k_current). It will smooth noizy data and incorrectly detected random line segments.
2. Pipeline was designed and debugged on very stable videos. On more harsher road coatings video stream needs to be stabilized before processing for smoother detection.
3. More color ranges can be added to account for other possible line colors under different light conditions.
4. Current pipeline filters line segments with the |slope| < 0.5 out. Better filterting can be applied by closely analyzing possible slopes based on (x,y) coordinate in the image : lower(x,y) < |slope(x,y)| < upper(x,y). I don't like this approach as it makes the pipeline less universal - we basically hardcode possible line segments.
