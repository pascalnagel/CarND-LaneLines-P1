# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline consists of the following operations:

 1. **Convertion of the RGB image to grayscale.** I started out using the `cv2.COLOR_RGB2GRAY` method, which worked fine for the first 2 test videos. One of the issues I had with optional video, was that the somewhat faded yellow lane marking were not detected on the light gray tarmac section. Lowering the Canny thresholds helped, but alone did not solve the problem. I came across advice on Student Hub to try out HSV color space and indeed using the V-channel of the `cv2.COLOR_RGB2HSV` output as grayscale input worked well.
 2. **Apply Gaussian blur.** I started out applying a 5x5 filter for the first 2 test videos, but later increased the filter size to 13x13 to reduce the large number of misdetections from tiny edges or noise in the optional video. Especially the roadside tree edges were drastically reduced by this.
 3. **Apply Canny transform.** During threshold tuning I stuck to a ratio of 1:2. Initially I had thresholds of 100 (low) and 200 (high), which was possible due to the strong gradient of the lane edges in the first 2 videos. These high thresholds removed a lot of unwanted weaker edges elsewhere in the image, which was great. However, the optional video had much weaker edges, especially the left faded yellow line on sometimes very bright tarmac. This made it necessary to reduce the thresholds to 50 (low) and 100 (high).
 4. **Apply mask to focus on the ROI in the image.** The helper function sets all pixels outside a defined polygon to black. Applying the helper function to the original RGB image for testing, I tuned the vertex positions of a 4-vertex polygon to capture only the lanes, with some margin.
 5. **Extract Hough lines.** A wide range of parameter settings worked for detecting the lane lines, the key here, was to find a parameter set which suppressed as many unwanted edges as possible. Since many of the unwanted edges did not lie on a straight line, but the lane edges did, setting the threshold to a large value helped most of all. The minimum line length was chosen large enough to suppress the short edges (orthogonal to the direction travel) on dashed lines, but large enough to always detect the dashed lane line edges in the direction of travel.

 The `draw_lines` function was then adapted to aggregate the Hough lines to draw one line each for the left and right lane markings. In order to do this I calculated slope and intercept for each Hough line, removed any lines with an absolute slope outside a reasonable range ([0.3, 2]) and seperated the lines by slope sign (negative slope = left lane, positive slope = right lane) and whether or not they were in the left or right half of the image.
 
 I then started out by taking the left(right) lane line slope and intercept to be the mean of all Hough line slopes and intercepts associated with the left(right) lane. This worked well for the first 2 test videos, but not being able to completely suppress all non-lane edges in the optional video, I found that slopes and intercepts from non-lane edges skewed the position of the lane line very severely. I then switched from using the mean to the median, which is much more robust to outliers. This worked somewhat well, but since the lane line is now drawn along just one Hough line, the resulting lane line becomes more erratic.

 Having obtained slope and intercept of both left and right lane line, I then evaluated the start and end coordinates inside the ROI and plotted them.


### 2. Identify potential shortcomings with your current pipeline

What I have realized most of all is that detecting lane lines in this fashion (or at least the way I have been doing it) is not very robust, at least not to the extent that a safety critical system should be. Seeing what happened when switching to the optional example I can only imagine what would happen in scenarios such as:

* Different lighting conditions (stronger shadows, etc.)
* Different surfaces or surface conditions 
* Differently colored lane markings
* Other markings on the road
* Objects on the same lane (vehicles, pedestrians, traffic cones, ...)

Thresholding the slopes and ignoring Hough lines on the opposite half of the image, although necessary to suppress a lot of unwanted edges in the optional video, will likely fail in sharper corners.

My pipeline also has the shortcoming of failing completely, when no Hough lines are detected on either side.

### 3. Suggest possible improvements to your pipeline

A possible improvement would be to introduce default lane lines when no line can be detected in a given frame, perhaps just taking the lane line of the previous frame. Smoothing the lane lines over a few frames could also help smoothen them.