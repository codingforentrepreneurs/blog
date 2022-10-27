---
title: OpenCV &amp; Python: How to Change Resolution or Rescale Frame
slug: open-cv-python-change-video-resolution-or-scale

publish_timestamp: April 8, 2018
url: https://www.codingforentrepreneurs.com/blog/open-cv-python-change-video-resolution-or-scale/

---

Let's assume you're working off something simple like [this](https://www.codingforentrepreneurs.com/blog/opencv-python-web-camera-quick-test/), you might need to change your frame size and/or video resolution from time to time and do so in the code.

OpenCV makes it easy to change resolution of your video.
OpenCV also makes it easy to scale your video. 

# Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/y76C3P20rwc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

# Adjust Resolution

```
import cv2

cap = cv2.VideoCapture(0)

def make_1080p():
    cap.set(3, 1920)
    cap.set(4, 1080)

def make_720p():
    cap.set(3, 1280)
    cap.set(4, 720)

def make_480p():
    cap.set(3, 640)
    cap.set(4, 480)

def change_res(width, height):
    cap.set(3, width)
    cap.set(4, height)

make_720p()
change_res(1280, 720)

```

As you might have guessed, you **cannot up-scale a resolution** if your camera does not support it. For example, if your camera supports 720p, that's the maximum resolution the above method(s) will allow. 

#### Scaling Factor (Downscale/Upscaling)
```
import cv2

cap = cv2.VideoCapture(0)

def rescale_frame(frame, percent=75):
    width = int(frame.shape[1] * percent/ 100)
    height = int(frame.shape[0] * percent/ 100)
    dim = (width, height)
    return cv2.resize(frame, dim, interpolation =cv2.INTER_AREA)

while True:
    rect, frame = cap.read()
    frame75 = rescale_frame(frame, percent=75)
    cv2.imshow('frame75', frame75)
    frame150 = rescale_frame(frame, percent=150)
    cv2.imshow('frame150', frame150)

cap.release()
cv2.destroyAllWindows()
```

With **Scaling** you can change the frame size regardless of your camera's resolution which, of course, could lead to poor results on upscaling (aka too pixelated). 

Either way, it's good to have these methods in your wheelhouse.