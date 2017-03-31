#**Finding Lane Lines on the Road** 

---

[//]: # (Image References)

[step1]: ./writeup_images/step_1.jpg
[step2]: ./writeup_images/step_2.jpg
[step3]: ./writeup_images/step_3.jpg
[step4]: ./writeup_images/step_4.jpg
[step5]: ./writeup_images/step_5.jpg
[step6]: ./writeup_images/step_6.jpg

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 6 steps. Below each steps are briefly discussed:

#### Step 1 (Grayed):
First the image was converter gray scale converting 3 channel to 1 channel image and the pixel ranging from 0 - 255. 
![alt text][step1]

#### Step 2 (Blurred):
Next the image is blurred using gaussian averaging technique with kernal size 7.
![alt text][step2]

#### Step 3 (Edge Detection):
The blurred image is then converted into binary image and the gradients are detected, which extracts the edges in the image.
![alt text][step3]

#### Step 4 (Focusing on region of interest):
Next, image is croped for the region of interest using fillpoly tool.
![alt text][step4]

#### Step 5 (Interpolation of line):
To convert multiple lines into one line for both left and right side of the lane, Here averaging technique was used.  

First, the lines were classified as left and right based on sign of slope; namely, negative slope belongs to left lane and positive slope belongs to right.

```python
    # Classify lines to either right or left based on slope.
    # Negative slope means left line, and positive means right.
    for line in lines:
        for x1,y1,x2,y2 in line:
            
            slope  = (y2 - y1)/(x2 - x1)
            center = [ (x1 + x2)/2, (y1 + y2)/2]
            
            slope_min = 0.50
            slope_max = 0.80
            
            # Also ignore lines with unrealistic slope.
            if ( (abs(slope) > slope_min) and (abs(slope) < slope_max) and (slope != 0) ) :
                if slope < 0:
                    lm.append(slope)
                    lc.append(center)
                else:
                    rm.append(slope)
                    rc.append(center)
            else:
                continue
```

Next, average of slope and center point of the lines were taken for both left and right side, resulting in one slope and one center for each side.  


```python    
    # Find average of slope and center
    r_slope  = np.sum(rm)/len(rm)
    r_center = np.divide( np.sum(rc, axis=0), len(rc) )
    
    l_slope  = np.sum(lm)/len(lm)
    l_center = np.divide( np.sum(lc, axis=0), len(lc) )
```


Finally, for each side, using the center of the lane a two end points of a new line was calculated to draw the line.


```python 
    # Capture height and width of image
    H = img.shape[0]
    W = img.shape[1]
           
    # Draw lines with averaged center and slope data
    # Also check for situation where we may not find any line in some frame of the video
    if ( (r_slope > 0) and (l_slope < 0) and (len(r_center) > 0) and (len(l_center) > 0) ):
        # Drawing lines (Left)
        x_c = int(l_center[0])
        y_c = int(l_center[1])
        x_1 = int(W*0.45)
        y_1 = int(y_c - l_slope*( x_c - x_1 ))
        y_2 = int(H)
        x_2 = int(x_c - ( y_c - y_2 )/l_slope)
        cv2.line(img, (x_1, y_1), (x_2, y_2), color, thickness)

        # Drawing lines (Right)
        x_c = int(r_center[0])
        y_c = int(r_center[1])
        x_1 = int(W*0.55)
        y_1 = int(y_c - r_slope*( x_c - x_1 ))
        y_2 = int(H)
        x_2 = int(x_c - ( y_c - y_2 )/r_slope)
        cv2.line(img, (x_1, y_1), (x_2, y_2), color, thickness)
```  
![alt text][step5]


#### Step 6 (Overlay line):
In the last step, the line was overlayed on top of the input image.
![alt text][step6]


###2. Identify potential shortcomings with your current pipeline

The pipleline was not able to handle images with mix of bright light and shades, possibly the thresholds ( e.g., canny thresholds ) started falling apart for those frames. Video 3 (optional challenge) clearly demonstrates the limitation. 

Also, the lines were shaking (relatively high frequency) possibly due to change is center point in succesive image.  


###3. Suggest possible improvements to your pipeline

I believe, in order to increase success rate of lane detection per frame, we need to process the images; essentially, to normalize the effect mix sun and shade.  In addition, we could tune both canny thresholds and hough thresholds even better such that it captures maximum possible variations observed in the image.

For the jittery lanes, I think, we could do a running average of center point from succesive image, something similar to filtering out noise, which should smooth the line overlay transition from one image to the next.





