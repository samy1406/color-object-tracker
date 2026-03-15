# What I Learned — CV Project: Color Object Tracker
| Mar 2026 | OpenCV, NumPy

---

## 1. Why HSV over BGR
In BGR/RGB, brightness is mixed into all three color channels. 
This means the same colored object looks completely different 
under different lighting conditions. HSV separates color (Hue) 
from intensity (Saturation, Value), making color-based detection 
robust to lighting changes. This is why almost all real-time 
color tracking systems use HSV instead of BGR.

## 2. Morphological Opening
Morphological opening = erosion followed by dilation.
Erosion shrinks all white regions — small noise blobs disappear 
entirely. Dilation then grows the remaining regions back to their 
original size. Net result: noise is removed, the ball blob survives.
We used an elliptical kernel because the ball is circular — it 
preserves the ball's shape better than a rectangular kernel.

## 3. Why Red Needs Two Masks
Red sits at both ends of the HSV hue circle — H=0 and H=179 
represent the same red color. A single inRange() call cannot 
wrap around the boundary. Solution: two separate masks, one for 
H=0–10 and one for H=165–179, combined with cv2.bitwise_or().

## 4. How cv2.moments() Calculates Centroid
Moments are weighted sums of pixel positions in a binary image.
- m00 = total white pixel area
- m10 = sum of (x × pixel value) across all white pixels  
- m01 = sum of (y × pixel value) across all white pixels

Centroid x = m10 / m00, y = m01 / m00
This is essentially the center of mass of the white blob.
Always guard against m00 = 0 to avoid ZeroDivisionError.

## 5. Biggest Challenge
The hardest problem was the similarity between the ball's HSV 
value and the background. For cricket footage, red hoardings and 
knee pads share similar hue values with the ball. For tennis, 
the grass and ball are both yellow-green. Classical HSV thresholding 
cannot distinguish between them reliably. Additionally, broadcast 
video has the ball occupying only 5–10 pixels at field distance, 
causing motion blur and low saturation readings that shift the 
detected HSV value significantly from the calibrated range.

## 6. What I Would Do Differently
- Use YOLOv8 trained on a ball detection dataset — eliminates 
  all HSV calibration issues
- Apply CSRT or ByteTrack for temporal consistency across frames
- Use background subtraction as a preprocessing step to isolate 
  moving objects before color masking
- Restrict detection to a pitch ROI using perspective transform 
  (connects to Project 1 — Lane Detection)
