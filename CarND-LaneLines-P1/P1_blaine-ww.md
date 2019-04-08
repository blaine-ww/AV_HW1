# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/solidWhiteCurve/solidWhiteCurve_gray.jpg "Grayscale"
[image2]: ./test_images_output/solidWhiteCurve/solidWhiteCurve_blurGray.jpg "Gaussian Smoothing"
[image3]: ./test_images_output/solidWhiteCurve/solidWhiteCurve_edges.jpg "Canny Edges"
[image4]: ./test_images_output/solidWhiteCurve/solidWhiteCurve_maskedEdges.jpg "Masked Edges"
[image5]: ./test_images_output/solidWhiteCurve/solidWhiteCurve_lines.jpg "Hough Transform"
[image6]: ./test_images_output/solidWhiteCurve/solidWhiteCurve_weighted_final.jpg "Weighted"


---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:
1. Convert the images to grayscale
* ![Grayscale][image1]

2. Apply Gaussian smoothing
* ![Gaussian Smoothing][image2]

3. Define the Canny edges in the image
* ![Canny Edges][image3]

4. Declare the region of interest to mask for the lane lines
* ![Masked Edges][image4]

5. Execute the Hough Transform on the canny edges to determine the segments of canny edges that are lane lines 
* ![Hough Transform][image5]

6. (optional) Overlay the weighted image of the lane lines on the original image.
* ![Weighted][image6]


#### Modifying draw_lines() function
In order to draw a single line on the left and right lanes, I modified the draw_lines() function by first separating the potential Canny edges points (x1,y1,x2,y2) into two segments, `left_line` and `right_line`, based on the negative and positive slope calculated, respectively. After iterating through the multiple `line` points, I averaged the x1, y1, x2, & y2 values stored in `left_line` and `right_line` array columns, then calculated the overall slope for left and right segments, `left_slope` & `right_slope`. Lastely, to generate the *y=m*x+b* linear equation for extrapolation, we calculate the y-intercept *b* through previous calculated slopes by rearranging the linear equation to give *b=y-m*x*, therefore calculating `left_y_intercept` & `right_y_intercept` with previously calculated slopes.

To draw a line with `cv2.line`, I need to determine the top and bottom coordinates for the line to be drawn. Utilizing the known values of the bottom of the image as the known `y_bottom` y-coordinate and also the y-coordinate for the top coordinate, `apex_y` established by the apex region mmasking, the two missing values are the x-coordinate paired for the top and bottom points for each left and right line, labeled `left_x_bottom` & `left_x_top` along with `right_x_bottom` & `right_x_top`, respectively. When we arrange the linear equation we solved earlier to solve for the x-coordinate, we get *x=(y-b)/m* where we already know the *m* slope value as well as the y-coordinate associated with the top `apex_y` and bottom `y_bottom` coordinates. Therefore,
* `left_x_top=(apex_y-left_y_intercept)/left_slope`
* `left_x_bottom=(y_bottom-left_y_intercept)/left_slope`
Apply the same method for right line segment.

To ensure the averaged points derived are valid, these three sanity checks will be run before cv2.line (drawing the line):
* _left_slope<0.0_ and _right_slope>0.0_
* _left_x_bottom < half of horizontal length of image_ and _right_x_bottom > half of horizontal length of image_ 
*_left_x_top > half of horizontal length of image_ and _right_x_top > half of horizontal length of image_

Then line is drawn!


### 2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

When the slope identified is zero as in a line parallel to horizontal frame, then it is ignored, when that is the case, no output line is drawn which means some frames are missing lines due these parameters.


Another shortcoming could be ...
Since I set my extrapolated vertical y-values for my lines to be the bottom of the image and the apex, when the lanes curves, my extrapolated lines are drawn tangent to the curves rather than 
the lines curving with the vehicle as it turns. This may not be pratical with sharp lanes turns.


### 3. Suggest possible improvements to your pipeline



A possible improvement would be to have several segments of extrapolated lines rather than just one segment that connects the coordinates at the bottom of the image to the apex. This segments can account for the curvature and isolate other anomalies in the extrapolation to just that portion of the lane line, rather than affecting the whole extrapolation line as a whole.

Another potential improvement could be to also idenitify the neighboring lane lines as well. This may be useful for when the vehicle decides to switch lanes and you want to already have the lane lines determined for the neighboring lanes to follow seamlessly.
