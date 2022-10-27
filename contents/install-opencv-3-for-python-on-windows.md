---
title: Install OpenCV 3 for Python on Windows
slug: install-opencv-3-for-python-on-windows

publish_timestamp: Feb. 22, 2018
url: https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-windows/

---

OpenCV, aka Open Computer Vision, is an incredible library for doing everything from Face Recognition, Object Recognition, to edge detection, image manipulation, and much more. This guide is to help you get it installed on your Windows 10 computer.

**[Mac OS OpenCV Python Install Guide](https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-mac)**

**Linux Guide** Try using [Pi Awesome](https://www.codingforentrepreneurs.com/blog/raspberry-pi-awesome/)


===============
## Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/Fcc_jemaoNU" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
===============

This was tested on Windows 10 with Python 3 as the default install. Which is the setup I recommend for you anyways. If you're using Python 2.7, consider [this guide](https://kirr.co/0feu3d).

#### 1. Install Via PIP

```
pip install opencv-contrib-python --upgrade
```
or without extra modules:
```
pip install opencv-python 
```


#### 2. Test OpenCV Installation
```
C:\> python
>>> import cv2
>>> print(cv2.__version__)
'3.4.0' # your version may be a newer one
```

#### 3. Test a webcam with [this post](https://www.codingforentrepreneurs.com/blog/opencv-python-web-camera-quick-test/).

Now you're ready to rock with OpenCV on your Windows