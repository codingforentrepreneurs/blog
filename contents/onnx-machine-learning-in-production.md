---
title: Onnx Machine Learning in Production
slug: onnx-machine-learning-in-production

publish_timestamp: Sept. 11, 2020
url: https://www.codingforentrepreneurs.com/blog/onnx-machine-learning-in-production/

---

I recently had a project that I needed to use PyInstaller along with a Keras-trained model. Unfortunately, PyInstaller and Keras only work some of the time.. as in, not that reliable of a build.

### What to do?

Onnx was the solution.

My project didn't need to run training, it just need to run inference (prediction) and that's something `onnx` really excels at.


### So, what is onnx? 
On [onnx.ai](https://onnx.ai), it says: `ONNX is an open format built to represent machine learning models`. I read this as `onnx` is essentially a file format for ML models and `onnx` can run those models. 

Almost any modern ML framework can become an `onnx` model. `PyTorch` -- yep. `Tensorflow` -- yep. `Keras` -- yep. `Caffe2` -- yep. `Scikit-learn` -- yep.


### Why is this important? **interoperability**

Let's say I have a `flask` web app in production serving a ML model. Doing so, is easy enough. Let's say the model I originally created this in was in `Keras`. Then, a new member joins my team that is just a pro at `PyTorch`. How do I deploy these two models on the same project?

Well, you could package up both `tensorflow` and `PyTorch` for running inference but that starts to make our simple web app a bulky one and, more importantly, one that's significantly more difficult to manage.

`onnx` simplifies this problem by providing a standardized way to run models in production. All you have to do is export your ML model to an onnx model. Once you have that, you can easily run it in production as you'll see below.



### A real-world example

Recently, I was working on a Python project that needed to be compiled into a single executable. For this, I used [PyInstaller](https://www.pyinstaller.org/) since it's a very reliable way to turn python into an executable. 

Unfortunately, [pyinstaller](https://www.pyinstaller.org/) doesn't play nice will all packages and package types and containers (like docker) cannot (as far as I know) be compiled into a single binary.

I was having all kinds of trouble getting `PyInstaller` and `tensorflow` to compile correctly so I decided to give `onnx` a try. Not only did it work, but it worked incredibly reliably.

> The post below will show you exactly how to convert a `keras` model into a `onnx` one and then put it into production.


#### Step 1. Virtual Environment
Whenever you build a Python project, use a virtual environment of some kind. For this guide, I'm solving a real-world problem I had with `PyInstaller` so I'm going to use the following: 
- Python 3.7
- `venv` (and not my preferred `pipenv`)

```
$ cd path/to/your/dev/folder
```
```
$ mkdir cfe_onnx
$ cd cfe_onnx
$ python3.7 -m venv .
```

##### Activate
**Mac/Linux**
```
source bin/activate
```

**Windows**

```
.\Scripts\activate
```


#### Step 2. Installations

```
pip install tensorflow keras2onnx onnxruntime numpy pillow
```
- `tensorflow`: our machine learning framework (but using `tf.keras` which is built-in to tensorflow now)
- `keras2onnx`: our conversion package
- `onnxruntime`: how we run inference on onnx models in production
- `numpy`: numerical python; common for dealing with arrays and matrices in Python & ML Projects
- `pillow`: the Python Image Library installer (`PIL`) which makes it easy to open images within python.



#### Step 3. Export a Keras Model to an Onnx Model
As outlined in the `keras2onnx` [docs](https://github.com/onnx/keras-onnx), I'm going to just be using a pre-trained Keras model for illustration purposes, change as needed:

```
from keras.applications.resnet50 import ResNet50
model = ResNet50(include_top=True, weights='imagenet')
model.save("model.h5")
```


**Now run the conversion**
```
onnx_model = keras2onnx.convert_keras(model, model.name)
keras2onnx.save_model(onnx_model,  'model.onnx')
```
> For `PyTorch` you can easily export a model too by using `PyTorch`'s built-in method outlined [here](https://pytorch.org/docs/stable/onnx.html)


After this is done, you will have to saved models:
- `model.h5` (keras)
- `model.onnx` (onnx)

I recommend keeping a keras-saved model for future resumable training.  `onnx` can be converted but I don't use the extra step if I don't need to.



#### Step 4. Prepare for Production
The above model is for Image Classification (aka `imagenet`), so we have to be sure we prepare our data prior to running inference.


##### Preprocessing
```
# preprocessing.py
import numpy as np
from PIL import Image

def process_image(image_path, height=150, width=150):
    '''
    This method opens an image and converts it into a normalized
    array that represents the image.
    '''
    image = Image.open(image_path)
    image = image.convert("RGB")
    new_image = image.resize((width,height))
    np_image = np.asarray(new_image)
    min = np_image.min()
    max = np_image.max()    
    # normalize to the range 0-1
    np_image = np_image.astype('float32')
    np_image -= min
    np_image /= (max - min)
    return [np_image]
```

##### Response Encoding
Below is a `json` encoder that converts numpy data types.

```
# encoding.py
import numpy as np
import json 

class NumpyEncoder(json.JSONEncoder):
    """ Special json encoder for numpy types """
    def default(self, obj):
        if isinstance(obj, np.integer):
            return int(obj)
        elif isinstance(obj, np.floating):
            return float(obj)
        elif isinstance(obj, np.ndarray):
            return obj.tolist()
        return json.JSONEncoder.default(self, obj)
```
Here's a couple ways to use this encoder:

**With Json Dumps

```python
data = {"preds": np.array([0.87, 0.13])}
json.dumps(data, cls=NumpyEncoder)
```

**In Flask**
```python
app = Flask(__name__)
app.json_encoder = NumpyEncoder

@app.route('/numpy')
def get_numpy_response():
    data = {"preds": np.array([0.87, 0.13])}
    return jsonify(data)
```


#### Step 5. Predictions with Onnx

```
# predict.py
import json
import pathlib

import onnxruntime

from .encoding import NumpyEncoder
from .preprocessing import process_image


ONNX_SESSION = None

def get_session():
    global ONNX_SESSION
    if ONNX_SESSION == None:
        model_path = str(pathlib.Path("model.onnx"))
        sess = onnxruntime.InferenceSession(model_path)
        ONNX_SESSION = sess
    return ONNX_SESSION


def predict(img_path, use_array=False, *args, **kwargs):
    onnx_sess = get_session()
    sess_inputs = onnx_sess.get_inputs()[0]
    input_name = sess_inputs.name
    shape = sess_inputs.shape
    im = process_image(img_path, height=shape[1], width=shape[2]) 
    inference_preds = onnx_sess.run(None, {input_name: im}) # this is where the inference_happens
    results = inference_preds[0][0]
    data = {str(k):v for k,v in enumerate(results)}
    return json.dumps(data, cls=NumpyEncoder)
```



#### Next Steps
Now, you just need to take all of the above information and turn it into a webapp or add it to a local python project. I'll leave that to you.

Good luck!