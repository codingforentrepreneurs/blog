---
title: Install OpenCV 3 for Python on Mac
slug: install-opencv-3-for-python-on-mac

publish_timestamp: Feb. 21, 2018
url: https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-mac/

---

OpenCV, aka Open Computer Vision, is an incredible library for doing everything from Face Recognition, Object Recognition, to edge detection, image manipulation, and much more. This guide is to help you get it installed on your Mac.

**[Windows OpenCV Python Install Guide](https://www.codingforentrepreneurs.com/blog/install-opencv-3-for-python-on-windows/)**

**Linux Guide** Try using [Pi Awesome](https://www.codingforentrepreneurs.com/blog/raspberry-pi-awesome/)

## Via pip

```console
(cfestart) $ pipenv install opencv-contrib-python
```
Not using `pipenv`? Run `pip install opencv-contrib-python --upgrade`

## Via Homebrew

#### 1. Install & Update [Homebrew](http://brew.sh/)
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

```
$ brew update
```


#### 2. Tap homebrew-bio
```
$ brew tap brewsci/bio
```


#### 3. Install OpenCV with Homebrew including Python3
```
$ brew install opencv3 --with-contrib --with-python3
```


#### 5. Change into the OpenCV directory

Since we installed OpenCV with Homebrew, the package was should be listed in homebrew's Cellar (`/usr/local/Cellar/`)
To see what's in the Brew Cellar, do this:

```
$ brew list
# or
$ ls /usr/local/Cellar/
```

Now, let's go into the `opencv` directory.
```
$ cd /usr/local/Cellar/opencv
$ ls 
3.3.0_3 3.3.1_1 
```
In my case, the numbers `3.3.0_3` and  `3.3.1_1` are different versions of OpenCV. This is a good thing. You'll want the highest version number listed here, in my case it's `3.3.1_1`

```
$ cd 3.3.1_1
$ pwd 
/usr/local/Cellar/opencv/3.3.1_1
```
Awesome. This is the OpenCV Version we want to use. We're very close to getting the `cv2.<version>.so` we need.

#### 5. Find the `cv2.<version>.so` file in OpenCV

Okay, so we have the OpenCV version we're going to use and where it's located. Now we need to link it to our version of Python.

- What Python Version are you using?

    ```
    $ python --version
    Python 2.7.10
    ```

    ```
    $ python3 --version
    Python 3.6.3
    ```

I have `Python 3.6` and `Python 2.7` installed on my system. If I want both versions to have OpenCV, I can. I'll just need to add the correct `cv2.<version>.so` to your Python's site packages. We'll do this part in the next step. In the meantime, let's get the `cv2.<version>.so` for each version of Python. You can install whatever version you want.


##### Get the Python 3.6 `cv2.<version>.so` File Path
```
$ cd /usr/local/Cellar/opencv/3.3.1_1
$ ls
INSTALL_RECEIPT.json    bin         share
LICENSE         include
README.md       lib
$ cd lib
ls 
# /usr/local/Cellar/opencv/3.3.1_1/lib
...
...
python2.7
python3.6 
...
...
$ cd python3.6
$ cd site-packages
$ pwd
/usr/local/Cellar/opencv/3.3.1_1/lib/python3.6/site-packages
$ ls
cv2.cpython-36m-darwin.so # this name might be different but... bingo!
```

**OpenCV Python3.6 Path**:  `/usr/local/Cellar/opencv/3.3.1_1/lib/python3.6/site-packages/cv2.cpython-36m-darwin.so`

##### Get the Python 2.7 `cv2.<version>.so` File Path
```
$ cd /usr/local/Cellar/opencv/3.3.1_1
$ ls
INSTALL_RECEIPT.json    bin         share
LICENSE         include
README.md       lib
$ cd lib
ls 
# /usr/local/Cellar/opencv/3.3.1_1/lib
...
...
python2.7
python3.6 
...
...
$ cd python2.7
$ cd site-packages
$ pwd
/usr/local/Cellar/opencv/3.3.1_1/lib/python3.6/site-packages
$ ls
cv2.so # this name might be different but... bingo!
```

**OpenCV Python2.7 Path**:  `/usr/local/Cellar/opencv/3.3.1_1/lib/python2.7/site-packages/cv2.so`



#### 6. Add `cv2.<version>.so` to System Python (virtualenvs are next)


##### Python 3.6 Site Packages & `cv2.<version>.so`
- Open Python 3.6

    ```
    $ python3
    Python 3.6.3 (default, Oct 22 2017, 16:00:10) 
    [GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.38)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>>
    ```

- Get the Site Packages location

    ```
    >>> import sys
    >>> print(sys.path)
    [..., '/usr/local/lib/python3.6/site-packages', ...]
    ```

- Reference the OpenCV Python3.6 Path
    
    (from above)
    **OpenCV Python3.6 Path**:  `/usr/local/Cellar/opencv/3.3.1_1/lib/python3.6/site-packages/cv2.cpython-36m-darwin.so`


- Link `cv2.<version>.so` to site packages 

    ```
    ln -s /usr/local/Cellar/opencv/3.3.1_1/lib/python3.6/site-packages/cv2.cpython-36m-darwin.so /usr/local/lib/python3.6/site-packages/cv2.so
    ```


##### Python 2.7 Site Packages & `cv2.<version>.so`
- Open Python 2.7

    ```
    $ python2
    Python 2.7.14 (default, Oct 22 2017, 16:05:07) 
    [GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.38)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>>
    ```

- Get the Site Packages location

    ```
    >>> import sys
    >>> print(sys.path)
    [..., '/usr/local/lib/python2.7/site-packages', ...]
    ```

- Reference the OpenCV Python2.7 Path

    (from above)
    **OpenCV Python2.7 Path**:  `/usr/local/Cellar/opencv/3.3.1_1/lib/python2.7/site-packages/cv2.so`


- Link `cv2.<version>.so` to site packages 

    ```
    ln -S /usr/local/Cellar/opencv/3.3.1_1/lib/python2.7/site-packages/cv2.so /usr/local/lib/python2.7/site-packages/cv2.so
    ```


#### 7. Add OpenCV to Virtualenvs
All you have to do is add the `cv2.<version>.so` to your virtualenv's site-packages. Example:


```
$ mkdir ~/Dev
$ cd ~/Dev
$ virtualenv -p python3 newcvtest
$ cd newcvtest
$ source bin/activate
(newcvtest) $ python --version 
Python 3.6.5
(newcvtest) $ pip install numpy
(newcvtest) $ cd lib/python3.6/site-packages
(newcvtest) $ ln -S /usr/local/Cellar/opencv/3.3.1_1/lib/python3.6/site-packages/cv2.cpython-36m-darwin.so cv2.so
```


```
$ mkdir ~/Dev
$ cd ~/Dev
$ virtualenv -p python2 newcvtestpy2
$ cd newcvtestpy2
$ source bin/activate
(newcvtestpy2) $ python --version 
Python 2.7.10
(newcvtestpy2) $ pip install numpy
(newcvtestpy2) $ cd lib/python3.6/site-packages
(newcvtestpy2) $ ln -S /usr/local/Cellar/opencv/3.3.1_1/lib/python2.7/site-packages/cv2.so cv2.so
```


#### 8. Test OpenCV Installation
```
$ python3  # or python2
>>> import cv2
>>> print(cv2.__version__)
3.3.1 # your version may be a newer one
```

#### 9. Test a webcam with [this post](https://www.codingforentrepreneurs.com/blog/opencv-python-web-camera-quick-test/).

===============
## Watch
<iframe width="560" height="315" src="https://www.youtube.com/embed/iluST-V757A" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

===============

Now you're ready to rock with OpenCV on your Mac