# **Finding Lane Lines on the Road**

---

**Goals of this project**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./output_images/out_solidWhiteCurve.jpg "Solid white curve"
[image2]: ./output_images/out_solidWhiteRight.jpg "Solid white right"
[image3]: ./output_images/out_solidYellowCurve.jpg "Solid yellow curve"
[image4]: ./output_images/out_solidYellowCurve2.jpg "Solid yellow curve 2"
[image5]: ./output_images/out_solidYellowLeft.jpg "Solid yellow left"
[image6]: ./output_images/out_whiteCarLaneSwitch.jpg "White car lane switch"

---

### Reflection

### 1. Pipeline description

My pipeline consists of 5 steps, the corresponding source code is located in the P1.ipynb notbook. Output videos generated by this pipeline can be found in the output_videos directory.

First, the image is grayscaled.

In the second step the image is blured with gaussian filter.

As third step the edges are marked by canny filter.

In the 4. step the region of interest is masked with predefined polygon.

The step 5 is applying the hough transform to the masked segment for finding the lines

Finally the lines detected by the 5 steps above are painted on the original picture.

In order to draw a single line on the left and right lanes, I modified the draw_lines()
function by identifying the left lane with negative slope and right lane with
positive slope. After that find the two points on left and right lane with smallest y coordinate.
From these points interpolate to the two corresponding points for left and right lane with help of the slope. Find below the commented function.

```python
def draw_lines(img, lines, color=[255, 0, 0], thickness=9):
    """
    Extrapolate the lines from line segments.
    1. Find smallest y points on the image.
    2. Extrapolate from there to the top of the image
    """
    rslope = [] # right slope array
    lslope = [] # left slope array

    #init all points to maximum image dimension
    ly1 = ly2 = ry1 = ry2 = img.shape[0]
    lx1 = lx2 = rx1 = rx2 = img.shape[1]
    #process all lines
    for line in lines:
        for x1, y1, x2, y2 in line:
            slope = ((y2-y1)/(x2-x1))
            if slope < 0:
                lslope.append(slope)
                #find left line minimum and store
                if y2 < ly1:
                    ly1 = y2
                    lx1 = x2
            else:
                rslope.append(slope)
                #find right line minimum and store
                if y1 < ry1:
                    ry1 = y1
                    rx1 = x1
    #calculate average slope
    rslope = sum(rslope) / len(rslope)
    lslope = sum(lslope) / len(lslope)
    #extrapolate the x points
    lx2 = int((img.shape[0]-ly1) / lslope) + lx1
    rx2 = int((img.shape[0]-ry1) / rslope) + rx1
    #draw final lines
    cv2.line(img, (lx1,ly1), (lx2,ly2), color, thickness)
    cv2.line(img, (rx1,ry1), (rx2,ry2), color, thickness)
```

Images generated by the pipeline:

![alt text][image1]
![alt text][image2]
![alt text][image3]
![alt text][image4]
![alt text][image5]
![alt text][image6]


### 2. Potential shortcomings

One potential shortcoming would be what would happen when the angle of the camera taking pictures changes.
As of now hard coded values are used for identifying the segment and for Hough transformation parameters.
Current implementation doesn't work with video driving a curve.
Another issue observed in the yellow video example is that at few places the
line jumps to the left or right for a short time.
I assume in the current pipeline is room for improvements to handle different road conditions.


### 3. Possible improvements

A possible improvement would be to not use hard coded values.
As of now I don't know how to avoid using hard coded values,
maybe there is a way to adapt the parameters of segment selection
and Hough parameters dynamically.
Another possibility might be to have a predefined set of values per car model.
Since the camera will be mounted on a fixed position on same car model.
To handle curvy roads the draw_lines function must be adapted, currently simple linear
extrapolation is performed.
