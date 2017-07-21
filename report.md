# **Report for Finding lanes on the road** 

## By : Shreyansh Bhatt
---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline consists of 5 stages.
1. Convert image to a grayscale and apply guassian blur.

2. Use canny edge detection to detect edges. Filter = 5 was applied for gussian blur.
[image2]: ./examples/gussian_blur.png

3. Create region of interest: A polygon was created according to vertices. Vertices of a polygon are variables based on a statically defined width, height around a horizontal mid point. Adjusting width and height results in a change of region of interest. It should be according to the camera angel, change in car's direction, and lane markings. An example region of interest looks like a following,
[image3]: ./examples/masked_edges.png

4. Line detection: Define line detection parameters for Hough line detection technique and call Hough line detection helper method to find lines. The parameters were chosen such that the algorithm can filter lanes based on an average lane marking length. These parameters also requires tuning based on different lane markings.

5. Average, extrapolate, and draw lines : draw_lines() function was modified to extrapolate line based on detected lines using hough transform. The key steps in process include,
  5.1 Ignore lines for errorneous slopes : Absolute slope of lane marking usually lies between a given bound. For example line detected with slope = 0 is mostly an error since lane markings are not horizontal. This helped filter out noise but a possible side effect is during shart left or right turns. However, the slope range can be changed/tuned for such a scenario (See Improvements for more details).
  5.2 Store all the points and slope of lines detected using hough transform and divide the list in two halfs. Each half consists of a list of slope-point pairs in which all the slopes are within a threshold range. This way, points that fall on left lane and right lane are separated. Most of the time, left lane has a positive slope while the right lane has negative but it may not be true in many instances. Hence, the division of left lane points and right lane points is based on the closeness between the line segments that they belong to. Again, a correct choice of threshold is critical.
  5.3 Now we have a list of points that make left lane and right lane. OpenCV's fitLine was used to approximate line that passes through these points. It returns a point and a slope. Using that information a line is drawn from y = 300, y = 540. Again, the choice of upper y (y = 300) is a variable that should be tuned based on various scenarios.

### 2. Potential shortcomings
1. Step 5.2 of line extrapolation creates two lists of points (points on left lane and right lane). List of (slope-point) is sorted and then divided at a point when a slope difference between two pairs exceeds threshold. A choice of threshold shouold be according to the lane curves, camera angel and change in the direction that a car is driving to. It is sensitive to the curves and that is apparent in the challenge.mp4. It is able to identify lanes but can not properly divide the list into two.

2. Choice of polygon width : One major shortcoming is a statically defined polygon. The height and width is not changing according to the car driving direction, curves etc. In one frame of a video, a statically defined polygon works but if a picture shifts to right/left due to curve and/or car movement in lane, the same static polygon is not idea. A larger polygon will start including noise which is quite apparent in the challenge.mp4 example.

3. Color of lanes : The current technique works only when lanes are marked with either sharp yellow or sharp white color. For example, in challenge.mp4, as soon as the car gets onto a road with slight change in intensity of lane color, the technique can not detect those lines.

4. Choice of Hough line detection parameters : The parametrs are statically chosen as they are with polygon width and height. If a lane marking is shorter sometime, the technique won't be able to detect such marking as a line and it will directly affect the slope of resulting line extrapolation.

### 3. Improvements
1. Learning based approach : A technique that I think can overcome all the shortcomings is a learning based approach. One can come up with a finetuned parameter selection which works for these 3 scenarios (videos) but different situations are very likely to hit either one (or more) of the shortcomings. If all the parameters defined in the earlier section are learned according to different kinds of situations i.e. lane curves, turns, camera ange, car direction which causes the picture shift to left or right, different weather conditions etc. Then all these shortcomings can be addressed and the system can be more robust. For the same reason, each parameter is defined at the beginning of a method in the implemented section. I would love to work on extending this project by learning these parameters and sending them to these pipeline.

2. Color intensity : A proper choice of color intentensity is critical. It should include various shades of yellow and white.
