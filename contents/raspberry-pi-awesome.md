---
title: Raspberry Pi Awesome  // Install Scripts for Python 3, OpenCV, Dli &amp; Others
slug: raspberry-pi-awesome

publish_timestamp: Feb. 24, 2018
url: https://www.codingforentrepreneurs.com/blog/raspberry-pi-awesome/

---

I've found setting up my Raspberry Pi microcontrollers a little cumbersome and, frankly, annoying. Typing the same commands over and over.  Sometimes I need to flash my drive and start from 0. Sometimes I need a new pi. All these things happen over and over. So, here we are, some install scripts to hopefully make our lives easier.

You may hit snags. If you find the solution please comment below and/or make a pull request on our [official Pi Awesome repo](https://github.com/codingforentrepreneurs/Raspberry-Pi-Awesome).

Enjoy!


The goal is to have these installed and ready to rock:
- [Python 3](https://python.org)
- [Face Recognition](https://github.com/ageitgey/face_recognition)
- [OpenCV](https://opencv.org/) for Python
- [dlib](https://github.com/davisking/dlib)
- [Django](https://www.djangoproject.com)
- [PiCamera](https://picamera.readthedocs.io/en/release-1.13/)
- and more


### Suggestions / Requirements
- Are you comfortable with `terminal`? If not, this is probably not for you.
- Raspbian Stretch Installed. Here's a guide we made to install it as well as set it up on your local network: [Guide here](https://www.codingforentrepreneurs.com/blog/raspberry-pi-network-server-guide-with-django-ssh/). 
- Do you have a USB Web Camera? I use [this one](http://amzn.to/2HLhKdI) <- affiliate link ([non-affiliate](https://www.amazon.com/Logitech-Widescreen-Calling-Recording-Desktop/dp/B006JH8T3S/ref=sr_1_3?s=pc&ie=UTF8&qid=1519505895&sr=1-3&keywords=logitech+webcam))
- You could also use the Raspberry Pi Camera Module V2 ([affiliate link](http://amzn.to/2BLntzD))

These scripts may work with other versions of Linux (specifically Debian) but we haven't tested. Have you? Please let us know.


### Install OpenCV on Raspberry Pi
**Time**: Up to 3 hours.
```
$ cd ~/
$ wget https://raw.githubusercontent.com/codingforentrepreneurs/Raspberry-Pi-Awesome/master/scripts/setup-opencv.sh  && chmod +x setup-opencv.sh && sudo ./setup-opencv.sh
```

Looking for install OpenCV on [Mac](https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-mac)? [Windows](https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-windows/)?

**Test OpenCV Install with [this post](https://www.codingforentrepreneurs.com/blog/opencv-python-web-camera-quick-test/)**


### Install Dlib & Face Recognition
**Time**: Up to 1.5 hours. 
**Recommend**: OpenCV already installed to fully make use of Face Recognition


```
$ cd ~/
$ wget https://raw.githubusercontent.com/codingforentrepreneurs/Raspberry-Pi-Awesome/master/scripts/setup-face-recognition.sh  && chmod +x setup-face-recognition.sh && sudo ./setup-face-recognition.sh
```


### Install gPhoto2 for DSLR Control
Our script is an exact copy of [gphoto2-updater](https://github.com/gonzalo/gphoto2-updater). We may change it in the future which is why it exists with these other scripts (instead of just being linked). If you have time, please thank the creater of the original script [Gonzalo Cao Cabeza de Vaca](https://github.com/gonzalo).

```
$ cd ~/
$ wget https://raw.githubusercontent.com/codingforentrepreneurs/Raspberry-Pi-Awesome/master/scripts/gphoto2-updater.sh  && chmod +x gphoto2-updater.sh && sudo ./gphoto2-updater.sh
```