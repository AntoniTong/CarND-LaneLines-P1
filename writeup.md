# **Finding Lane Lines on the Road** 

### Project Summary

**In this finding lane lines project, I created a pipeline to process color images from vehicle camera and to identify the lane lines of the road that the vehicle is driving on. The function was at first tested and validated using images, and then transferred to process videos (a series of photos)**


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

[image2]: ./test_images_output/solidWhiteCurve.jpg 
[image3]: ./test_images_output/solidWhiteRight.jpg 
[image4]: ./test_images_output/solidYellowCurve.jpg 
[image5]: ./test_images_output/solidYellowCurve2.jpg 
[image6]: ./test_images_output/solidYellowLeft.jpg 
[image7]: ./test_images_output/whiteCarLaneSwitch.jpg 
[image14]: ./test_images_output/challenge.jpg

[image8]: ./test_images/solidWhiteCurve.jpg 
[image9]: ./test_images/solidWhiteRight.jpg 
[image10]: ./test_images/solidYellowCurve.jpg 
[image11]: ./test_images/solidYellowCurve2.jpg 
[image12]: ./test_images/solidYellowLeft.jpg 
[image13]: ./test_images/whiteCarLaneSwitch.jpg
[image15]: ./test_images/challenge.jpg
---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I implemented Canny transform to detect edges of the images. After that, I implemented hough transform to extract line segments given the region of interest is the area covering the road ahead. I implemented a draw_lines function to draw the left and right lanes. Finaly, I combined the original image with the line drawing. 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by first filtering the lines via looking at the tangent of the slopes. If it is between [0.5, 4], it belongs to the left lane. If it is between [-4, -0.5], it belongs to the right lane. All the other lines are considered noise. Then, I extracted the pixels from each line and put them together to fit two functions of y = p[0]x + p[1] for left and right lane lines. Because longer lines generally indicate the actual lane lines, I assigned more weights on the pixels from longer lines. For instance, a line of 20-pixel length contributes four times the data points than a line of 10-pixel length.

After tuning the hyper-parameters, I end up with the following values: 

	kernel_size = 3
	low_threshold = 50
	high_threshold = 150 
	rho = 1 
	theta = np.pi/360 
	threshold = 20   
	min_line_length = 20 
	max_line_gap =  20  
	
The algorithm successfully detected lane lines of Clip 1~2, but was failed for Clip 3. 

![Raw image][image8]
![With lane detection][image2]
*Results on white lane lines*

The main reason is that in Clip 3, there are a lot of distractions such as changing of road color, obstruction from the trees and their shades. To improve that, I reduced the low/high threshold of the Canny transform. The blowback is that it will pick up more noisy lines. As a counteraction, I increased the value of threshold and max_line_gap for Hough transform. 

	kernel_size = 3
	low_threshold = 20
	high_threshold = 60
	rho = 1 
	theta = np.pi/360 
	threshold = 30   
	min_line_length = 20 
	max_line_gap = 60
	
![Raw image][image15]
![With lane detection][image14]
*Results on yellow lane lines with shades and changing road colors*

As a result, I was able to detect lane lines for Clip 3, although the overall line following accuracy was slightly compromised. 

### 2. Identify potential shortcomings with your current pipeline

Several shortcomings exist. First, the lane line following are focused on straight roads or roads with shallow turns. However, there exist steep turns where lane lines are curves instead of straight lines.  Second, the algorithm is already very sensitive to the lane line quality. If more disruptions are introduced by roads, lane line markings, surroundings and other vehicles, the algorithm will not work properly. Finally, current images are from a camera of fixed position. What if the relative position of the lane in a image changes due to changing of terrains or vehicle steering. The fixed region-of-interest will become invalid.    


### 3. Suggest possible improvements to your pipeline

The algorithm could become 'smarter' in detecting the region-of-interest and adaptively adjust to the change of lanes by switching to a machine learning based algorithm like CNN. In addition, right now each analysis was based on a single image. We can improve that by considering past knowledge via processing multiple images of consecutive time steps at once. It will help smoothing out the result.  