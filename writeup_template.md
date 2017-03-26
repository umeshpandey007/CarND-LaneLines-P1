#**Finding Lane Lines on the Road** 

The goal of this project is to find the lane lines in a road

### Reflection

###1. Pipeline

My pipeline consisted of the following steps:

**Step 1**: Convert the input image to grayscale image

**Step 2**: Applied Gaussian Blur filter on the grayscale image with kernel_size=5

**Step 3**: Applied Canny Edge to detect edges in the image with low_threshold=50 and high_threshold=150

**Step 4**: Define ROI & apply the mask on the edge_img

The ROI selected was with the following vertices:

top_left_vertex = (470,310)
top_right_vertex = (520,310)
bottom_left_vertex = (0, height-1)
bottom_right_vertex = (width-1, height-1)

**Step 5**: Apply Hough transform on the masked image:

**_Step 5.1_**: Using the following parameters in cv2.HoughLinesP

rho = 2
theta = np.pi/180
threshold = 40
min_line_length = 15
max_line_gap = 5

The output is set of lines identified by the Hough transform

**_Step 5.2_**: draw_lines function

_Step 5.2.1_: Segregation of points with positive and negative slope
I segregate the end-points of the lines with positive and negative slope in two different arrays(left_array and right_array). While segregating the points in each array, points are included in the array only if the slope of the current line is 10% arround the slope of the previous line. While considering for comparison, care is taken to compare points with positive slopes with positive slopes only, similarly for negative slopes. 

_Step 5.2.2_: Also I note down the min y distance (ymin) for all the points which are included in both the arrays, this help in extrapolation of the line.

_Step 5.2.3_: Once the points are segregated , find the average line with each set of points. I use np.polyfit to fit an average left line using all the points identified in the left_array. Similarly use the np.polyfit to find an average right line using all the points identified in the right_array.

_Step 5.2.4_: With np.polyfit I have slope and intercept for both left and right lane lines. To draw the final line, I find the top x coordinate for both left and right lines using ymin and the slope and intercept from output of np.polyfit for both right and left lines separately. Similarly I find the bottom x coordinate using y=height-1 and the slope and intercept from output of np.polyfit for both right and left lines separately.

_Step 5.2.5_: Draw both the lines with cv2.line function using the vertices from Step 5.2.4

**Step 6**: Superimpose the final right and left lines obtained for Step 5.2.5 with the original image

###2. Identify potential shortcomings with your current pipeline


Some of the potential shortcomings which is also seen when running the pipeline with challenge video:
1. The current lane detection uses extrapolated lines separates the lines based on positive and negative slope. In case of curves, the lines would not correctly fit the lanes, it would require curves to fit for lane detection. EVen if we consider tangent lines to the curves, they might have slopes in the same direction, so the algorithm might break
2. ROI has to be carefully adjusted, it can be impacted if the elevation of the road changes alongwith turns within short distances
3. Line extrapolation fails during curved roads
4. Flickering in the output while detecting lane lines as the line position changes between frame to frame


###3. Suggest possible improvements to your pipeline

These are my thoughts for the problems suggested in Section 2:
1. Using Hough transform to find curves might solve problems while detecting lanes on turns. Also, the threshold and min_lines might need to be adjusted to low values to detect curves
2. ROI selection might need to be dynamic, not sure how it can be done, but we can use optical flow from previous frames to do the same
3. Possibly can be avaoided by solution in 1
4. Using optical flow from the previous frame to compensate the line position for the current frame.