# **Project 1: Finding Lane Lines on the Road** 

## Introduction
This project introduces a basic program for a self driving car to find lanes on the road.
The program demonstrated to work well on the test images and the two test videos, however its shortcomings were detected in the 'challenge' video test.


## Methodology
For detecting lane lines in a video a software pipeline is developed in a series of images, then tested on 3 short videos.
The pipeline consisted of 5 major steps excluding reading and writing the image. 
1. The test image is first converted to grayscale from RGB using the helper function grayscale(). This produces the below image.
2. The grayscaled image is given a gaussian blur to remove noise or spurious gradients. The blurred image is given below.
3. Canny edge detection is applied on this blurred image and a binary image shown below is produced.
4. This image contains edges that are not relevant for lane finding problem. A region of interest is defined to separate the lanes from sorrounding environment and a masked image containing only the lanes is extracted using cv2.bitwise_and() function from opencv library. This can be seen below.
5. This binary image of identified lane lines is finally merged with the original image using cv2.addweighted() function from opencv library. This produces an image given below. Note that, this is without making any modifications to the drawlines() helper function. It can be observed that the lines are not continuous as required.

## Modification to drawlines() helper function
Since the resulting line segments after the processing the image through the pipeline are not continuous, a modification is made to the drawlines() helper function. Consider the code snippet below:

```
def draw_lines_robust(img, lines, color=[200, 0, 0], thickness = 10):
  
    x_left = []
    y_left = []
    x_right = []
    y_right = []
    imshape = image.shape
    ysize = imshape[0]
    ytop = int(0.6*ysize) # need y coordinates of the top and bottom of left and right lane
    ybtm = int(ysize) #  to calculate x values once a line is found
    
    for line in lines:
        for x1,y1,x2,y2 in line:
            slope = float(((y2-y1)/(x2-x1)))
            if (slope > 0.5): # if the line slope is greater than tan(26.52 deg), it is the left line
                    x_left.append(x1)
                    x_left.append(x2)
                    y_left.append(y1)
                    y_left.append(y2)
            if (slope < -0.5): # if the line slope is less than tan(153.48 deg), it is the right line
                    x_right.append(x1)
                    x_right.append(x2)
                    y_right.append(y1)
                    y_right.append(y2)
    # only execute if there are points found that meet criteria, this eliminates borderline cases i.e. rogue frames
    if (x_left!=[]) & (x_right!=[]) & (y_left!=[]) & (y_right!=[]): 
        left_line_coeffs = np.polyfit(x_left, y_left, 1)
        left_xtop = int((ytop - left_line_coeffs[1])/left_line_coeffs[0])
        left_xbtm = int((ybtm - left_line_coeffs[1])/left_line_coeffs[0])
        right_line_coeffs = np.polyfit(x_right, y_right, 1)
        right_xtop = int((ytop - right_line_coeffs[1])/right_line_coeffs[0])
        right_xbtm = int((ybtm - right_line_coeffs[1])/right_line_coeffs[0])
        cv2.line(img, (left_xtop, ytop), (left_xbtm, ybtm), color, thickness)
        cv2.line(img, (right_xtop, ytop), (right_xbtm, ybtm), color, thickness)
```

## Implementing the pipeline on test videos

The pipeline developed in the project is implemented on 2 different test videos and worked well. The methodolody was the same as the one for the test images.

## Shortcomings observed in the current pipeline

1. Since the first step is converting the image to grayscale from RGB, shadows and light variations in the environment are difficult to capture. This can be gleaned from the fact that the current pipeline while working reasonably well for the first two test videos breaks down for the challenge video.

2. The lane lines detected in the resulting output are not as stable as the ones in "P1_example" video. This is not desirable since it is difficult to follow rapidly changing steering commands.

## Possible improvements to the current pipeline

1. For better results the test image can be preprocessed using RGB normalization. This can help in avoiding issues with shadows and lighting variations. 
A function definition for producing a normalized RGB image is shown below. This approach did not produce an
 improvement when implemented in its current form and should be further improved. 

```
# Reference: http://akash0x53.github.io/blog/2013/04/29/RGB-Normalization/
def normalized_rgb(img):
        imshape = img.shape
        ysize = imshape[0]
        xsize = imshape[1]
        norm=np.zeros((ysize,xsize,3),np.float32)
        norm_rgb=np.zeros((ysize,xsize,3),np.uint8)
        b=img[:,:,0]
        g=img[:,:,1]
        r=img[:,:,2]
        sum = b + g + r
        norm[:,:,0]=b/sum*255.0
        norm[:,:,1]=g/sum*255.0
        norm[:,:,2]=r/sum*255.0
        norm_rgb=cv2.convertScaleAbs(norm)
        return norm_rgb
```

Also, machine learning approaches can be explored to make the lane finding pipeline better.