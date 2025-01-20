# **Finding Lane Lines on the Road ECE 495 Assignment 1** 
### Prabhjyot Singh 20891880
### Date: January 20th, 2025

---
**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./intermediate_pipeline_steps/combined_hsv_mask.png "Masked Out Image"
[image2]: ./intermediate_pipeline_steps/grayscale.png "Grayscale"
[image3]: ./intermediate_pipeline_steps/canny_img.png "Canny Image"
[image4]: ./intermediate_pipeline_steps/masked_edges.png "Edges Masked"
[image5]: ./intermediate_pipeline_steps/result.png "Final Result"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 7 steps. First, I converted the image into an HSV image to perform color thresholding using the cvtColor function from OpenCV. The first step to colour thresholding was to create a low and high HSV NumPy array to store the HSV values that I will be searching for in the image. 

```
low_hsv_white = np.array([  0,   0, 200], dtype=np.uint8) #The low HSV threshold for the white colour mask
high_hsv_white = np.array([180,  25, 255], dtype=np.uint8) #The high HSV threshold
mask_white = cv2.inRange(hsv_img, low_hsv_white, high_hsv_white) #The cv2.inRange then uses the values to grab the mask of the image
```
After the color mask was then applied for both white and yellow, I then utilized bitwise_or to combine the yellow and white masks and then a bitwise_and to combine them with the original image.
```
combined_hsv_mask = cv2.bitwise_or(mask_yellow, mask_white)
masked_img = cv2.bitwise_and(img, img, mask=combined_hsv_mask)
```
![alt text][image1]

Then for the second step, I simply used the provided grayscale() helper function to convert the masked image to grayscale.

![alt text][image2]

The next step involved using the provided gaussian_blur function on the gray scaled image. Due to the only parameter of the function being the kernel size, I attempted using the pipeline at different kernel size values of 3, 5, and 7 to determine qualitatively which value resulted in the most accurate line being drawn. This was done after step 4 was implemented to determine the best results for the edges detected. For step 4 which was the canny edge detector, the provided canny() function was used along with a high and low threshold and iteratively adjusted the kernel_size, low_threshold and high_threshold.

```
#3. Apply Smoothing to the img
kernel_size = 5
smoothed = gaussian_blur(gray, kernel_size)

#4. Apply Canny Edge Detector
low_threshold = 100
high_threshold = 180
canny_img = canny(smoothed, low_threshold, high_threshold)
```

After iterating through a grid search of varying variables, I was able to settle on the low threshold of 100 and a high threshold of 180 with a kernal size of 5. The resulting image was the following.

![alt text][image3]

The next step or step 5 of the pipeline was to select the region of interest using a trapezoid. The method to draw acquire the region of interest was using the img.shape function to get the shape of the image and subsequently use it to dynamically determine the height and width of the image. Relative to the height and width, the vertices of the region of interest were determined. Then the canny image and the vertices were passed to the region_of_interest function, which returns an image with the edges masked. The values for the corners were manually tuned for the test images and then confirmed with the results of the videos.

![alt text][image4]

The final two steps of the pipeline, which are to run the Hough on the masked image as well as overlay the Hough lines on the original image. Simply for step 6 the parameters for the hough_lines helper function were declared and tuned manually based on the results provided. However, the main change to this function was in the draw_lines() helper function, which was heavily modified to more accurately extrapolate the lines provided by the hough_lines function.

The draw_lines function was changed to involve a three-step process to draw a single line based on the information provided by the Hough method. This involved first separating the lines based on their slope to identify them as a left or right line. A singular line was then fitted to the provided left and right data using the polyfit command from numpy. 

```
if len(left_x) > 1:
    m_left, b_left = np.polyfit(left_x, left_y, 1)
    # Define the region where we draw the line
    y_bottom = img.shape[0] #bottom of the image
    y_top = int(y_bottom * 0.6) #draw up to 60% of the image size
    x_bottom = int((y_bottom - b_left) / m_left) #inverse of y = mx + b to determine X based on Y
    x_top = int((y_top - b_left) / m_left) #Similar
    cv2.line(img, (x_bottom, y_bottom), (x_top, y_top), color, thickness) #Use the cv2.line function to draw the line
```
The result of this new draw lines function overlayed on the original image was an accurate depiction of where the lines are for all the test images.

![alt text][image5]

Utilizing this pipeline, I was able to achieve the required results for both the videos and the line fitting when qualitatively observed was accurate to the lines on the road for both the white and yellow lines. The challenge video, however, proved to be much different as the curvature of the road was not something the current pipeline could handle, but this is discussed further in the next section of the report.


### 2. Identify potential shortcomings with your current pipeline

One potential shortcoming of the current pipeline is definitely its inability to handle curves in the road. With its current design, the pipeline and the functions provided would be able to handle straight roads of white and yellow color. However, there are many example of curved roads of varying angles that would limit the functionality of the pipeline provided. This situation was demonstrated by the challenge video at the end of the notebook. Even though the line was able to fit to the start of the curve, to extrapolate and better fit the curve while maintaining accuracy, the ability to fit to second order functions would be required. 

Another shortcoming of the current implementation is the hard coded values for all the helper function that take in parameters. Currently, the only way to improve upon the values provided to the helper function is to utilize a trail and error method, which is slow and often inaccurate compared to some form of automated process.


### 3. Suggest possible improvements to your pipeline

One improvement to the pipeline would be to modify the draw lines function further to enable fitting to a curve and therefore allow for a larger variety of images and videos to be within the Operation Design Domain. This would involve implementing a more complex draw_lines function that would be able to threshold for a certain level of curvature in the road. This would also need a substantial amount of tuning to ensure that the threshold does not allow for near horizontal lines to be considered as a valid component to the curve and also fitting to a 2nd or 3rd degree polynomial would be necessary.

In addition, another improved would be to implement a tuning algorithm involving an 8 dimensional search of the optimal parameters to use in each function. Since we are limited by the fact that manual tuning of the parameters is needed, we must implement a automated search for those parameters. However, to do this using classical methods would be inefficient and there are better methods such as machine learning which are designed to accomplish these goals and tune parameters for a pipeline such as this. The use of an automated process on a limited data set would allow for a highly optimized parameter list.
