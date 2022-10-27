---
title: How to Record Video in OpenCV &amp; Python
slug: how-to-record-video-in-opencv-python

publish_timestamp: April 9, 2018
url: https://www.codingforentrepreneurs.com/blog/how-to-record-video-in-opencv-python/

---


OpenCV makes it simple to record video. Keep in mind that this isn't also recording audio.

Do you need [OpenCV installed click here](https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-windows/)?

This code snippet is meant to be a reference as well as a quick way to discover how easy it easy to use OpenCV with Python.

The code below is adapted from the OpenCV documentation but it's improved so we can make our recording process more dynamic and based simply off the filename.
# Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/1eHQIu4r0Bc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

# The Code

```python
import numpy as np
import os
import cv2


filename = 'video.avi'
frames_per_second = 24.0
res = '720p'

# Set resolution for the video capture
# Function adapted from https://kirr.co/0l6qmh
def change_res(cap, width, height):
    cap.set(3, width)
    cap.set(4, height)

# Standard Video Dimensions Sizes
STD_DIMENSIONS =  {
    "480p": (640, 480),
    "720p": (1280, 720),
    "1080p": (1920, 1080),
    "4k": (3840, 2160),
}


# grab resolution dimensions and set video capture to it.
def get_dims(cap, res='1080p'):
    width, height = STD_DIMENSIONS["480p"]
    if res in STD_DIMENSIONS:
        width,height = STD_DIMENSIONS[res]
    ## change the current caputre device
    ## to the resulting resolution
    change_res(cap, width, height)
    return width, height

# Video Encoding, might require additional installs
# Types of Codes: http://www.fourcc.org/codecs.php
VIDEO_TYPE = {
    'avi': cv2.VideoWriter_fourcc(*'XVID'),
    #'mp4': cv2.VideoWriter_fourcc(*'H264'),
    'mp4': cv2.VideoWriter_fourcc(*'XVID'),
}

def get_video_type(filename):
    filename, ext = os.path.splitext(filename)
    if ext in VIDEO_TYPE:
      return  VIDEO_TYPE[ext]
    return VIDEO_TYPE['avi']



cap = cv2.VideoCapture(0)
out = cv2.VideoWriter(filename, get_video_type(filename), 25, get_dims(cap, res))

while True:
    ret, frame = cap.read()
    out.write(frame)
    cv2.imshow('frame',frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break


cap.release()
out.release()
cv2.destroyAllWindows()
```
