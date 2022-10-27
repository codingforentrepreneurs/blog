---
title: OpenCV &amp; Python // Web Camera Quick Test
slug: opencv-python-web-camera-quick-test

publish_timestamp: Feb. 22, 2018
url: https://www.codingforentrepreneurs.com/blog/opencv-python-web-camera-quick-test/

---

This post assumes you have either a USB webcam, a built-in webcam, or even a Pi Camera (if you're using OpenCV on Raspberry Pi) .

This is meant to show a quick way to test your camera is working as it should. 

Need help installing OpenCV for Python? Check out the [Mac](https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-mac) or [Windows](https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-windows/) install guides (Linux and Raspberry Pi coming soon).

Create a python file, say `camera-test.py` containing:
```
import numpy as np
import cv2

cap = cv2.VideoCapture(0)

while(True):
    # Capture frame-by-frame
    ret, frame = cap.read()

    # Our operations on the frame come here
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
   
    # Display the resulting frame
    cv2.imshow('frame',frame)
    cv2.imshow('gray',gray)
    if cv2.waitKey(20) & 0xFF == ord('q'):
        break

# When everything done, release the capture
cap.release()
cv2.destroyAllWindows()
```
now run:
```
C:\> python camera-test.py
```
You should see 2 images from your webcam. 1 in gray scale, 1 in color just like the one above albeit with your face!
.

**To quit**,  hit the key `q` "on" the video window(s) to stop the camera.