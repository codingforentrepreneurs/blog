---
title: OpenCV &amp; Python: Extract Faces with a REST API
slug: opencv-python-extract-faces-rest-api-flask

publish_timestamp: Jan. 23, 2020
url: https://www.codingforentrepreneurs.com/blog/opencv-python-extract-faces-rest-api-flask/

---

We're going to build a REST API with Flask to extract faces from images using OpenCV. 

More context coming soon. 

### Installs
```console
pipenv install opencv-contrib-python flask requests
```

### Make Default Project Files & Directories
Use **Terminal** on macOS/linux, **PowerShell** on Windows.
```console
cd ~/dev
mkdir opencv-api
cd opencv-api
mkdir api
mkdir api/cv/
mkdir api/storage
mkdir api/storage/outputs
mkdir api/storage/results
mkdir api/uploads
```

**Mac / Linux (*Terminal*)**
```
touch deploy.sh
touch Dockerfile
touch Procfile
touch api/__init__.py
touch api/views.py
touch api/wsgi.py
touch api/cv/__init__.py
touch api/cv/utils.py
touch api/cv/views.py
```

**Windows (*PowerShell*)**
If `ni` fails, use `echo $null >>` like `echo $null >> deploy.ps1`
```
ni deploy.ps1
ni Dockerfile
ni Procfile
ni api/__init__.py
ni api/views.py
ni api/wsgi.py
ni api/cv/__init__.py
ni api/cv/utils.py
ni api/cv/views.py
```


### Final Project Structure
```
opencv-api/
 |   Dockerfile
 |   Procfile
 |   run.sh
 |   run.ps1
 |
└─── api/
    │   __init__.py
    │   views.py
    │   wsgi.py
    │   
    └─── cv/
    │      __init__.py
    │      utils.py
    │      views.py
    │   
    └─── storage/
    │    └───outputs/
    │     |
    │    └───results/
    │
    └─── uploads/
```

### Setup base Flask app
```python
# api/__init__.py
import os
from flask import Flask, flash, request, redirect, url_for, send_from_directory
from werkzeug.utils import secure_filename

BASE_DIR = os.dirname(os.path.abspath(__file__))
UPLOAD_DIR = os.path.join(BASE_DIR, "storage", "uploads")
RESULTS_DIR = os.path.join(BASE_DIR, "storage", "results")
OUTPUTS_DIR = os.path.join(BASE_DIR, "storage", "outputs")
ALLOWED_EXTENSIONS = {'png', 'jpg', 'jpeg', 'gif'}

for d in [UPLOAD_DIR, RESULTS_DIR, OUTPUTS_DIR]:
    os.makedirs(d, exist_ok=True)

app = Flask(__name__)
app.secret_key = b'_5#y2L"F4Q8z\n\xec]/'

app.config['OUTPUTS_DIR'] = OUTPUTS_DIR

# Directly from the docs
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER
def allowed_file(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

@app.route('/static/uploads/<filename>')
def serve_upload_dir(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'],
                               filename)
                            
@app.route('/static/results/<filename>')
def serve_result_dir(filename):
    return send_from_directory(app.config['OUTPUTS_DIR'],
                               filename)

from .views import *
from .cv.views import *
```


### Basic views for an API upload url
```python
# api/views.py
import os
from flask import send_from_directory
from werkzeug.utils import secure_filename

from api import (
    app,
    allowed_file
)

@app.route('/api/upload', methods=['POST'])
def upload_file():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            return {"detail": "No file found"}, 400
        file = request.files['file']
        if file.filename == '':
            return {"detail": "Invalid file or filename missing"}, 400
        if file and not allowed_file(file.filename):
            return {"detail": "This file is not allowed"}, 401
        filename = secure_filename(file.filename)
        _, ext = os.path.splitext(filename)
        destination_folder = app.config['UPLOAD_FOLDER']
        new_filename = f"{len(os.listdir(destination_folder))}{ext}"
        save_path = os.path.join(destination_folder, new_filename)
        file.save(save_path)
        live_url = url_for('serve_upload_dir', new_filename)
        return {"url": live_url}, 201
        '''
        # respond with the actual image
        return send_from_directory(destination_folder, new_filename)
        '''
```


### OpenCV Custom Methods for 
```python
# api/cv/utils.py

import os
import cv2
import numpy as np

def list_options():
    for x in os.listdir(cv2.data.haarcascades):
        print(x)

def get_face_cascade(cascade='haarcascade_frontalface_default.xml'):
    return os.path.join(cv2.data.haarcascades, cascade)

def open_image(path):
    '''
    If we save the file locally, we can use this one.
    '''
    frame = cv2.imread(image_path)
    return frame

def image_from_buffer(file_buffer):
    '''
    If we don't save the file locally and just want to open
    a POST'd file. This is what we use.
    '''
    bytes_as_np_array = np.frombuffer(buffer.read(), dtype=np.uint8)
    flag = 1
    # flag = 1 == cv2.IMREAD_COLOR
    # https://docs.opencv.org/4.2.0/d4/da8/group__imgcodecs.html
    frame = cv2.imdecode(bytes_as_np_array, flag)
    return frame

def faces_from_frame(frame, save=True, destination=None):
    '''
    This is will extract all faces found in an image
    And save the faces (just the face) as a unique file
    in our destination folder.
    '''
    gray  = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    cascade_xml = get_face_cascade()
    cascade = cv2.CascadeClassifier(cascade_xml)
    faces = cascade.detectMultiScale(frame, scaleFactor=1.5, minNeighbors=5)
    final_dest = None
    if destination is not None:
        secondary = len(os.listdir(destination)) + 1
        final_dest = os.path.join(destination, f"{secondary}")
        os.makedirs(final_dest, exist_ok=True)
    paths = []
    for i, (x, y, w, h) in enumerate(faces):
        roi_color = frame[y:y+h, x:x+w]
        if save == True and destination != None:
            current_face_path = os.path.join(root_path, f"{i}.jpg") 
            cv2.imwrite(current_face_path, roi_color)
            paths.append(this_path)
    return paths
```


### Basic views for an API Upload URL
```python
# api/cv/views.py
import os
from flask import send_from_directory
from werkzeug.utils import secure_filename

from api import (
    app,
    allowed_file,
    OUTPUTS_DIR
)
from api.cv.utils import (
    open_image, 
    image_from_buffer,
    faces_from_frame
)

@app.route('/api/opencv-stream', methods=['POST'])
def opencv_upload_stream():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            return {"detail": "No file found"}, 400
        file = request.files['file']
        if file.filename == '':
            return {"detail": "Invalid file or filename missing"}, 400
        if file and not allowed_file(file.filename):
            return {"detail": "This file is not allowed"}, 401
        im = image_from_buffer(file)
        paths = faces_from_frame(im, save=True, destination=OUTPUTS_DIR)
        response_paths = []
        for path in paths:
            relative_path = path.replace(OUTPUTS_DIR, '')
            live_url = url_for('serve_result_dir', relative_path)
            response_paths.append(live_url)
        return {"paths": response_paths}, 201

@app.route('/api/opencv', methods=['POST'])
def opencv_upload():
    if request.method == 'POST':
        # check if the post request has the file part
        if 'file' not in request.files:
            return {"detail": "No file found"}, 400
        file = request.files['file']
        if file.filename == '':
            return {"detail": "Invalid file or filename missing"}, 400
        if file and not allowed_file(file.filename):
            return {"detail": "This file is not allowed"}, 401
        filename = secure_filename(file.filename)
        _, ext = os.path.splitext(filename)
        destination_folder = app.config['UPLOAD_FOLDER']
        new_filename = f"{len(os.listdir(destination_folder))}{ext}"
        save_path = os.path.join(destination_folder, new_filename)
        file.save(save_path)
        im = open_image(save_path)
        paths = faces_from_frame(im, save=True, destination=OUTPUTS_DIR)
        response_paths = []
        for path in paths:
            relative_path = path.replace(OUTPUTS_DIR, '')
            live_url = url_for('serve_opencv_result', relative_path)
            response_paths.append(live_url)
        return {"paths": response_paths}, 201
        '''
        # respond with one of the faces
        final_img = paths[0]
        relative_path = final_img.replace(OUTPUTS_DIR, "")
        if relative_path.startswith("/"):
            relative_path = relative_path[1:]
        return send_from_directory(OUTPUTS_DIR, relative_path)
        '''
```

### WSGI File
```python
# api/wsgi.py
import os
from api import app

if __name__=="__main__":
     port = os.environ.get("PORT") or 8000
     app.run(port=port)
```


### Docker & Dockerfile
```
# Base Image
FROM python:3.6

# create and set working directory
RUN mkdir /app
WORKDIR /app

# Add current directory code to working directory
ADD . /app/

# set default environment variables
ENV PYTHONUNBUFFERED 1
ENV LANG C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive 

# Install system dependencies with OpenCV
RUN apt-get update && apt-get install -y --no-install-recommends \
        tzdata \
        libopencv-dev \ 
        build-essential \
        libssl-dev \
        libpq-dev \
        libcurl4-gnutls-dev \
        libexpat1-dev \
        python3-setuptools \
        python3-pip \
        python3-dev \
        python3-venv \
        git \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# install environment dependencies
RUN pip3 install --upgrade pip 
RUN pip3 install pipenv
RUN pip3 install opencv-contrib-python

# Install project dependencies
RUN pipenv install --skip-lock --system --dev

CMD gunicorn api.wsgi:app --bind 0.0.0.0:$PORT
```



### Heroku & Build Shortcut

Below we'll use the following:

**opencv-rest-api-app**: heroku app name
**opencv-rest-api**: Docker container tag

You should change yours as needed.

**1. Create Heroku App**
```console
heroku create opencv-rest-api-app
```

**2. Login to the Heroku Container Registry**
```console
heroku container:login
```

**3. Create `run_d.sh`/`run_d.ps1` file**
```bash
docker build -t opencv-rest-api -f Dockerfile .
docker run -e SERVER_NAME="0.0.0.0:8000" -e PORT="8000" -it -p 8000:8000 opencv-rest-api
```


**4. Create `deploy.sh`/`deploy.ps1` file**
```bash
docker build -t opencv-rest-api -f Dockerfile .
heroku container:push web -a opencv-rest-api-app
heroku container:release web -a opencv-rest-api-app
```

**5. Mac/Linux give `run_d.sh` and `deploy.sh` permission to run**
```
chmod +x run_d.sh
chmod +x deploy.sh
```

**8. Run Local**
Use `run_d.ps1` if on Windows
```
./run_d.sh
```

**9. Deploy**
```
./deploy.sh # or ./deploy.ps1
```

**10. Open**
```
heroku open
```