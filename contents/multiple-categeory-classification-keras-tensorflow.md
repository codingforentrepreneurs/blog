---
title: Multiple Category Classification with Keras, Tensorflow, Pandas, Numpy, &amp; Python
slug: multiple-categeory-classification-keras-tensorflow

publish_timestamp: Aug. 13, 2019
url: https://www.codingforentrepreneurs.com/blog/multiple-categeory-classification-keras-tensorflow/

---


In this one, we'll be creating a deep neural network since it will help us do is find patterns in a large amount of data to make the best possible prediction on new data. Before we get started, let's revisit an old adage:

> Good data in, good data out. Garbage in, garbage out.

Building a neural network is easy even if you're new to writing code. The part that's hard is getting the data ready, setting the correct parameters, and understanding the math behind the neural network. The thing is, we don't have to understand the math to actually use a neural network.

In this post, we'll do 4 things:

1. Pre-process pre-existing data with built in tools
2. Run a neural network with Keras and Tensorflow
3. Plot neural network performance
4. Save trained model for continued-training and predictions (inference)


Thanks to [Peter Nagy](https://github.com/nagypeterjob) for the HUGE inspiration for creating this notebook.

### Jupyter Notebook is [here](https://github.com/codingforentrepreneurs/Notebooks/blob/master/src/Multiple%20Category%20Classification%20with%20Keras%2C%20Tensorflow%2C%20Pandas%2C%20Numpy%2C%20%26%20Python.ipynb) | [Google Colab](https://colab.research.google.com/github/codingforentrepreneurs/Notebooks/blob/master/src/Multiple%20Category%20Classification%20with%20Keras%2C%20Tensorflow%2C%20Pandas%2C%20Numpy%2C%20%26%20Python.ipynb)
The jupyter notebook is an amazing way to run live python code for data science and deep learning, it's also how this post is formatted.  Learn how to create your own [jupyter notebook server here](https://www.codingforentrepreneurs.com/blog/jupyter-notebook-server-aws-ec2-aws-vpc).

### Installation Requirements:

```
pip install Keras==2.2.4 pandas>=0.25.0 numpy<17.0 sklearn tensorflow==1.14.0
```
> Simply run `!pip install ...` within a cell to install within a jupyter notebook.

Full notebook requirements are located: `../requirements/multiple_category_classification_keras.txt`


### Recommend Hardware Setup & Guides:
- **Nvidia GPU with Cuda/CuDNN** GPUs are critical in processing large matrix operations; that's essentially what's happening in a neural network. We have gaming to thank for the huge advancements in GPUs.
    - [Install Tensorflow GPU on Windows using CUDA and cuDNN](https://www.codingforentrepreneurs.com/blog/install-tensorflow-gpu-windows-cuda-cudnn)
- **Jupyter Notebook** Using jupyter makes your life much easier as your working out your code especially as it relates to data science and visualizing what you're working on. 
    - A [Jupyter Notebook Server](https://www.codingforentrepreneurs.com/blog/jupyter-notebook-server-aws-ec2-aws-vpc) can be very useful so then you don't have to invest in your own local hardware (like the guide above).


```python
# !pip install Keras==2.2.4 pandas>=0.25.0 numpy<17.0 sklearn tensorflow==1.14.0 matplotlib==3.1.1
```


```python
import os

import numpy as np 
import pandas as pd
from keras.callbacks import ModelCheckpoint
from keras.layers import Dense, Embedding, LSTM, SpatialDropout1D
from keras.models import Sequential
from keras.preprocessing.text import Tokenizer
from keras.preprocessing.sequence import pad_sequences

from keras.utils.np_utils import to_categorical
from keras.callbacks import EarlyStopping, ModelCheckpoint
# disable tensorflow warnings
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '3' 

from sklearn.feature_extraction.text import CountVectorizer
from sklearn.model_selection import train_test_split

import pickle
import matplotlib.pyplot as plt
%matplotlib inline
```


```python
CURRENT_DIR = os.getcwd()
BASE_DIR = os.path.dirname(CURRENT_DIR)
data_dir = os.path.join(BASE_DIR, 'data')
models_dir = os.path.join(BASE_DIR, 'neural_networks', 'models')
checkpoints_dir = os.path.join(BASE_DIR, 'neural_networks', 'checkpoints')
pickles_dir = os.path.join(BASE_DIR, 'neural_networks', 'pickles')
os.makedirs(models_dir, exist_ok=True)
os.makedirs(checkpoints_dir, exist_ok=True)
os.makedirs(pickles_dir, exist_ok=True)

dataset_path = os.path.join(data_dir, 'uci-news-aggregator.csv')
DEFAULT_MODEL_NAME = 'multi'
```


```python
def get_callbacks(name=DEFAULT_MODEL_NAME):
    early_stopping = EarlyStopping(monitor='val_loss',
        patience=7, 
        min_delta=0.0001)
    checkpoint_path = os.path.join(checkpoints_dir, name)
    os.makedirs(checkpoint_path, exist_ok=True)
    filepath = os.path.join(
        checkpoint_path,
        'weights.{epoch:02d}-{val_loss:.2f}.hdf5'
    )
    checkpoint = ModelCheckpoint(filepath, 
        monitor='val_loss',  
        save_best_only=True,
        save_weights_only=True
    )
    callbacks = [early_stopping, checkpoint]
    return callbacks

def save_model(model, name=DEFAULT_MODEL_NAME):
    filename = os.path.join(models_dir, name + '.hdf5')
    return model.save(filename)
```


```python
df = pd.read_csv(dataset_path, usecols=['TITLE', 'CATEGORY'])
df.head()
```

##### Handling Duplicates
As a part of pre-processing data, we want to remove duplicate data as much as possible. Let's take a look to see if we have any title duplicates within our dataset. We use `df.TITLE` because `TITLE` is the actual name of the column. `df.CATEGORY` is the other column. 


```python
duplicate_title_distribution = df.TITLE.value_counts()[:10]
duplicate_title_distribution
```


```python
most_common_title = duplicate_title_distribution.index[0]
most_common_title
```


```python
df[df['TITLE'].str.contains(most_common_title)][:5]
```

Here's the most common article title: 

`The article requested cannot be found! Please refresh your browser or go back  ...` 

That occurred 145 times! If you're familiar with web scraping, you'll know that this is probably a 404 page error but it's clearly in our data and causing issues. Let's just remove *all duplicate titles* to ensure we don't have data like `PR Newswire` as being one of our possible data points.

Basically, my thought is, if the article title is duplicated, it's probably not a good article title.


```python
df = df.drop_duplicates(subset='TITLE', keep=False)
df[df['TITLE'].str.contains(most_common_title)]
```

Calculate the distribution of each article title and it's respective category. We're looking for the category with the **least** number of titles associated to it. We want an even distribution of titles / categories for best results otherwise we should anticipate or results to be skewed incorrectly. 


```python
category_dict = {
    'e': 'entertainment', 
    'b':'business', 
    't': 'science/tech', 
    'm': 'health'
}
df.CATEGORY.value_counts()
```


```python
category_labels = {
    'e': 'entertainment', 
    'b': 'business', 
    't': 'science/tech', 
    'h': 'health'
}
```


```python
max_num_of_labels = df.CATEGORY.value_counts()[-1] # based on the least_number of categories in the value counts above.

data_df = df.copy() # I create copies of the input data to ensure I always have the original copy readily available.

shuffled_df = data_df.reindex(np.random.permutation(data_df.index)) # always shuffle data when you can.


e = shuffled_df[shuffled_df['CATEGORY'] == 'e'][:max_num_of_labels]
b = shuffled_df[shuffled_df['CATEGORY'] == 'b'][:max_num_of_labels]
t = shuffled_df[shuffled_df['CATEGORY'] == 't'][:max_num_of_labels]
m = shuffled_df[shuffled_df['CATEGORY'] == 'm'][:max_num_of_labels]

concated_df = pd.concat([e,b,t,m], ignore_index=True)
#Shuffle the dataset
concated_df = concated_df.reindex(np.random.permutation(concated_df.index))
concated_df['LABEL'] = 0
concated_df.head()
```


```python
#One-hot encode the label
concated_df.loc[concated_df['CATEGORY'] == 'e', 'LABEL'] = 0 # e = index 0
concated_df.loc[concated_df['CATEGORY'] == 'b', 'LABEL'] = 1 # b = index 1
concated_df.loc[concated_df['CATEGORY'] == 't', 'LABEL'] = 2 # t = index 2
concated_df.loc[concated_df['CATEGORY'] == 'm', 'LABEL'] = 3 # m = index 3
print(concated_df['LABEL'][:10])
labels = to_categorical(concated_df['LABEL'], num_classes=4)
print(labels[:10])
if 'CATEGORY' in concated_df.keys():
    concated_df = concated_df.drop(['CATEGORY'], axis=1)
'''
 [1. 0. 0. 0.] e
 [0. 1. 0. 0.] b
 [0. 0. 1. 0.] t
 [0. 0. 0. 1.] m
'''
```


```python
concated_df.head()
```


```python
n_most_common_words = 8000
max_len = 130
token_filter = '!"#$%&()*+,-./:;<=>?@[\]^_`{|}~'
tokenizer = Tokenizer(num_words=n_most_common_words, filters=token_filter, lower=True)
tokenizer.fit_on_texts(concated_df['TITLE'].values)
```


```python
sequences = tokenizer.texts_to_sequences(concated_df['TITLE'].values)
word_index = tokenizer.word_index
print('Found %s unique tokens.' % len(word_index))
```


```python
X = pad_sequences(sequences, maxlen=max_len)
```


```python
X_train, X_test, y_train, y_test = train_test_split(X , labels, test_size=0.25, random_state=42)
```


```python
epochs = 10
emb_dim = 128
batch_size = 256
labels[:2]
```


```python
callbacks = get_callbacks(name='multi')
```


```python
print((X_train.shape, y_train.shape, X_test.shape, y_test.shape))
```

### Create your Neural Network
Keras makes it easy to create neural networks on top of the tensorflow library. You don't have to know exactly how this works to run it. 

Below will define your neural network's architecture. 

> [From Keras docs](https://keras.io/losses/): When using the categorical_crossentropy loss, your targets should be in categorical format (e.g. if you have 10 classes, the target for each sample should be a 10-dimensional vector that is all-zeros except for a 1 at the index corresponding to the class of the sample)...

We're doing categorical classification vs binary classification. 

**Binary classification** means predictions will be one or the other ie, `cat vs dog` or `liked vs disliked`.

**Category classification** means prediction "category" your new data belows to ie, `cat vs dog vs bird vs chair` or what we did here.


```python
model = Sequential()
model.add(Embedding(n_most_common_words, emb_dim, input_length=X.shape[1]))
model.add(SpatialDropout1D(0.7))
model.add(LSTM(64, dropout=0.7, recurrent_dropout=0.7))
model.add(Dense(4, activation='softmax'))
model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])
print(model.summary())
```

#### Run training
This step will take a while. It will be much faster if you lower the number of epochs (above) as well as use a GPU (as mentioned at the top).


```python
training = model.fit(X_train, y_train, epochs=epochs, batch_size=batch_size,validation_split=0.2,callbacks=callbacks)
```


```python
accr = model.evaluate(X_test, y_test)
print('Test set\n  Loss: {:0.3f}\n  Accuracy: {:0.3f}'.format(accr[0],accr[1]))
```


```python
accuracy = training.history['acc']
val_accuracy = training.history['val_acc']
loss = training.history['loss']
val_loss = training.history['val_loss']
```


```python
epochs = range(1, len(accuracy) + 1)

plt.plot(epochs, accuracy, 'bo', label='Training accuracy')
plt.plot(epochs, val_accuracy, 'b', label='Validation accuracy')
plt.title('Training and validation accuracy')
plt.legend()

plt.figure()

plt.plot(epochs, loss, 'bo', label='Training loss')
plt.plot(epochs, val_loss, 'b', label='Validation loss')
plt.title('Training and validation loss')
plt.legend()

plt.show()
```


```python
txt = ["Regular fast food eating linked to fertility issues in women"]
seq = tokenizer.texts_to_sequences(txt)
padded = pad_sequences(seq, maxlen=max_len)
pred = model.predict(padded)
labels = ['entertainment', 'business', 'science/tech', 'health']
print(pred, labels[np.argmax(pred)])
```


```python

```


```python
def extract_label(index):
    '''
    The labels correspond to exact label indices, in other words, the 
    order is absolutely important.
    '''
    labels = ['entertainment', 'business', 'science/tech', 'health']
    return labels[index]
```


```python
def predict(text, model_klass=model):
    seq = tokenizer.texts_to_sequences([text])
    padded = pad_sequences(seq, maxlen=max_len)
    pred = model_klass.predict(padded)
    top_prediction_index = np.argmax(pred)
    predicted_label = extract_label(top_prediction_index)
    predictions = pred.tolist()[0]
    extracted_predictions = [{extract_label(i):"%.2f%%"%(x*100)} for i, x in enumerate(predictions)]
    top_percent = "%.2f%%"% (predictions[top_prediction_index] * 100)
    print(f"{text}\t\t{top_percent} {predicted_label}")
    return extracted_predictions
```


```python
predict("The startup is doing very well")
```


```python
predict("The startup company is booming")
```


```python
predict("The startup company's growth has been amazing so far.")
```


```python
predict("Sales are through the roof!")
```


```python
predict("Stocks are booming.")
```


```python
predict("That was an incredible performance by the actors")
```


```python
predict("That was an incredible performance!")
```


```python
predict("The health of the company is poor.")
```


```python
predict("The health of the kid is poor.")
```

### Save and Prepare for Reusable Prediction
First, we'll save the model. Then we'll save the tokenizer with `pickle`. After that, we'll adjust our predict method to be resuable as well.


```python
save_model(model, name='multi_category_classification')
```


```python
multi_category_tokenizer_pkl = os.path.join(pickles_dir, 'multi_category_tokenizer.pkl')
multi_category_tokenizer_pkl
```


```python
write_mode = 'wb'
with open(multi_category_tokenizer_pkl, write_mode) as f:
    pickle.dump(tokenizer, f)
```

### Fully Reusable Model
Below is all we need: the trained model and the pickled tokenizer and now we can use our model at any time.


```python
from keras.models import load_model

stored_model  = os.path.join(models_dir, 'multi_category_classification')
model_obj = load_model(f'{stored_model}.hdf5')
```


```python
multi_category_tokenizer_pkl = os.path.join(pickles_dir, 'multi_category_tokenizer.pkl')

write_mode = 'rb'
with open(multi_category_tokenizer_pkl, write_mode) as f:
    tokenizer_obj = pickle.load(f)
```


```python
def predict(text, model_obj=None, tokenizer_obj=None):
    assert(tokenizer_obj != None)
    assert(model_obj != None)
    seq = tokenizer_obj.texts_to_sequences([text])
    padded = pad_sequences(seq, maxlen=max_len)
    pred = model_obj.predict(padded)
    top_prediction_index = np.argmax(pred)
    predicted_label = extract_label(top_prediction_index)
    predictions = pred.tolist()[0]
    extracted_predictions = [{extract_label(i):"%.2f%%"%(x*100)} for i, x in enumerate(predictions)]
    top_percent = "%.2f%%"% (predictions[top_prediction_index] * 100)
    print(f"{text}\t\t{top_percent} {predicted_label}")
    return extracted_predictions
```


```python
predict("This is working well.", model_obj=model_obj, tokenizer_obj=tokenizer_obj)
```


```python
predict("The market viability is uncertain.", model_obj=model_obj, tokenizer_obj=tokenizer_obj)
```
