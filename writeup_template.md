# **Finding Lane Lines on the Road**

## Writeup

**The goals/steps of this project are the following:**
1. Make a pipeline that finds lane lines on the road.
2. Reflect on the work in a written report (this markdown file).

[//]: # (Image References)

[grayscale_example]: ./examples/solidWhiteCurve_1_gray.jpg "Grayscale Example"
[blur_example]: ./examples/solidWhiteCurve_2_blurred.jpg "blur Example"
[edges_example]: ./examples/solidWhiteCurve_3_edges.jpg "Edge Detection Example"
[roi_example]: ./examples/solidWhiteCurve_4_masked.jpg "Region of Interest"
[hough_example]: ./examples/solidWhiteCurve_5_hough_lines.jpg "Hough Lines Example"
[line_fitting]: ./examples/solidWhiteCurve_6_lane_lines.jpg "Lane Drawing Example"
[final_example]: ./examples/solidWhiteCurve_7_final.jpg "Final Overlay"

---

## **Reflection**

### **1. Describe your pipeline.**  

The following is the detailed description of my pipeline:

I decided to remove the HSV filter from the suggested pipeline as it caused a lot of issues with recongizing the lines later on, which is something I was not expecting. 

**Pipeline Steps**  
1. **Grayscale**  
   - Convert the original RGB/BGR image into a grayscale image to reduce complexity to a single channel.  
   - ![Grayscale][grayscale_example]

2. **Gaussian Blur**  
   - Smooth the grayscale image with a Gaussian blur to reduce noise. The final decided kernel choice was 5, since it produced a result where there is still information from the lanes left but filters out a lot of the noises in the image.
   - ![Blur Example][blur_example]

3. **Canny Edge Detection**  
   - Detect edges using the Canny algorithm. The lower and higher threshold was set to 40 and 120, allowing for the detection of more lane points / lines. Using a higher value before caused the lost of information, and a very harsh jitter when fitting the lines later on.
   - Example:  
     ```python
     edges = cv2.Canny(blurred, 50, 150)
     ```
   - ![Edge Detection Example][edges_example]

4. **Region of Interest Mask**  
   - Keep only the part of the frame where we expect lane lines.  
   - Create a polygonal mask (e.g. trapezoid) and apply it via bitwise operations.  
   - ![Region of Interest][roi_example]

5. **Hough Transform**  
   - Perform the Hough line transform to detect line segments in the masked edge image.  
   - Example:  
     ```python
     lines = cv2.HoughLinesP(
         masked_edges, 
         rho=1, 
         theta=np.pi/180, 
         threshold=5, 
         minLineLength=30, 
         maxLineGap=20
     )
     ```
   - ![Hough Lines Example][hough_example]

6. **Separate and Fit Lanes (RANSAC + Linear Regression)**  
   - Separate line segments based on slope (negative ⇒ left, positive ⇒ right).  
   - For each side, collect all points and fit a line using:
     1. **RANSAC** (to robustly handle outliers), and  
     2. **Regular Linear Regression** (as a comparison).  
   - Pick whichever model yields the best score.  
   - Extrapolate a single straight line for each lane from the bottom of the image to around 60% of the height.
   - ![Lane Drawing Example][line_fitting]

7. **Draw and Overlay**  
   - Draw the final two lines (left and right lanes) on a blank image.  
   - Overlay these lines on the original image using weighted addition.  
   - ![Final Overlay][final_example]

**Modifications to `draw_lines()`**  
- Instead of drawing each line segment directly, I created a function (sometimes called `draw_lanes()`) that:  
  1. Groups line segments into left vs. right points.  
  2. Computes a single best-fit line per lane using RANSAC + Linear Regression.  
  3. Draws only the two resulting lines.  

This ensures we see **one continuous line** per lane rather than many small segments.

---

### **2. Identify potential shortcomings with your current pipeline**

1. **Handling Curves**  
   - The pipeline relies on a single slope-intercept model for each lane, which works primarily on **relatively straight** lanes or gentle curves. On sharper curves, a polynomial or spline fit might be needed.

2. **Varying Lighting & Weather**  
   - Dramatic changes in lighting (e.g., heavy shadows, nighttime, glare) can disrupt edge detection. Additional preprocessing or adaptive thresholding might be required.

3. **Occlusions**  
   - Other vehicles, debris, or faded lane markings can lead to incomplete or noisy lane segments. The algorithm might fail or be less accurate under these conditions.

4. **Multiple Lane Markings**  
   - Highways with multiple lanes, merge areas, or forked roads could cause confusion if the pipeline only expects two dominant lane markings.

---

### **3. Suggest possible improvements to your pipeline**

1. **Polynomial Fits**  
   - Instead of a straight-line model (slope + intercept), consider a **second-order polynomial**. This would better capture curved roads.

2. **Color Thresholding**  
   - Use color filtering to specifically detect white and yellow lane lines. Combining color thresholding with Canny edges helps reduce spurious edges.

3. **Region of Interest Adaptation**  
   - Dynamically define the ROI based on camera calibration or on-the-fly horizon detection, improving flexibility for different road angles or camera positions.

4. **Temporal Smoothing (For Video)**  
   - Store and average fits over several frames to reduce flicker or jitter. A Kalman filter or rolling average can keep lanes stable in video sequences.

5. **Advanced (Deep Learning)**  
   - Employ a **deep neural network** (e.g., a segmentation model) for more robust lane detection under various conditions (curves, heavy traffic, poor weather).

---

## **Conclusion**

This pipeline demonstrates a basic classical computer vision approach:
- **Edge Detection** (Canny),
- **Line Extraction** (Hough),
- **Lane Consolidation** (RANSAC + Linear Regression).

While effective in moderate conditions, real-world scenarios may require more advanced or adaptive techniques. Nevertheless, this project highlights how to **analyze** an image to isolate lane lines, **detect** them with robust fitting, and **visualize** the results.

**Thank you for reading!**

