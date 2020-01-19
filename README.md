
# Image Classification with MLPs - Lab

## Introduction

For the final lab in this section, we'll build a more advanced **_Multi-Layer Perceptron_** to solve image classification for a classic dataset, MNIST!  This dataset consists of thousands of labeled images of handwritten digits, and it has a special place in the history of Deep Learning. 

## Objectives 

- Build a multi-layer neural network image classifier using Keras 

## Packages

First, let's import all the classes and packages you'll need for this lab.


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
sns.set_style("whitegrid")
import keras
from keras.models import Sequential
from keras.layers import Dense
from keras.datasets import mnist
```

    Using TensorFlow backend.
    

##  Data 

Before we get into building the model, let's load our data and take a look at a sample image and label. 

The MNIST dataset is often used for benchmarking model performance in the world of AI/Deep Learning research. Because it's commonly used, Keras actually includes a helper function to load the data and labels from MNIST -- it even loads the data in a format already split into training and test sets!

Run the cell below to load the MNIST dataset. Note that if this is the first time you are working with MNIST through Keras, this will take a few minutes while Keras downloads the data. 


```python
(X_train, y_train), (X_test, y_test) = mnist.load_data()
```

Great!  

Now, let's quickly take a look at an image from the MNIST dataset -- we can visualize it using Matplotlib. Run the cell below to visualize the first image and its corresponding label. 


```python
sample_image = X_train[1]
sample_label = y_train[1]
display(plt.imshow(sample_image));
print('Label: {}'.format(sample_label));
```


    <matplotlib.image.AxesImage at 0x1e6719ac940>


    Label: 0
    


![png](index_files/index_8_2.png)


Great! That was easy. Now, we'll see that preprocessing image data has a few extra steps in order to get it into a shape where an MLP can work with it. 

## Preprocessing Images For Use With MLPs

By definition, images are matrices -- they are a spreadsheet of pixel values between 0 and 255. We can see this easily enough by just looking at a raw image:


```python
sample_image
```




    array([[  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,  51, 159, 253, 159,  50,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,  48, 238, 252, 252, 252, 237,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
             54, 227, 253, 252, 239, 233, 252,  57,   6,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,  10,  60,
            224, 252, 253, 252, 202,  84, 252, 253, 122,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0, 163, 252,
            252, 252, 253, 252, 252,  96, 189, 253, 167,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,  51, 238, 253,
            253, 190, 114, 253, 228,  47,  79, 255, 168,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,  48, 238, 252, 252,
            179,  12,  75, 121,  21,   0,   0, 253, 243,  50,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,  38, 165, 253, 233, 208,
             84,   0,   0,   0,   0,   0,   0, 253, 252, 165,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   7, 178, 252, 240,  71,  19,
             28,   0,   0,   0,   0,   0,   0, 253, 252, 195,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,  57, 252, 252,  63,   0,   0,
              0,   0,   0,   0,   0,   0,   0, 253, 252, 195,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0, 198, 253, 190,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0, 255, 253, 196,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  76, 246, 252, 112,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0, 253, 252, 148,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  85, 252, 230,  25,   0,   0,   0,
              0,   0,   0,   0,   0,   7, 135, 253, 186,  12,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  85, 252, 223,   0,   0,   0,   0,
              0,   0,   0,   0,   7, 131, 252, 225,  71,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  85, 252, 145,   0,   0,   0,   0,
              0,   0,   0,  48, 165, 252, 173,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  86, 253, 225,   0,   0,   0,   0,
              0,   0, 114, 238, 253, 162,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  85, 252, 249, 146,  48,  29,  85,
            178, 225, 253, 223, 167,  56,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  85, 252, 252, 252, 229, 215, 252,
            252, 252, 196, 130,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,  28, 199, 252, 252, 253, 252, 252,
            233, 145,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,  25, 128, 252, 253, 252, 141,
             37,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0],
           [  0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,   0,
              0,   0]], dtype=uint8)



This is a problem in its current format, because MLPs take their input as vectors, not matrices or tensors. If all of the images were different sizes, then we would have a more significant problem on our hands, because we'd have challenges getting each image reshaped into a vector the exact same size as our input layer. However, this isn't a problem with MNIST, because all images are black white 28x28 pixel images. This means that we can just concatenate each row (or column) into a single 784-dimensional vector! Since each image will be concatenated in the exact same way, positional information is still preserved (e.g. the pixel value for the second pixel in the second row of an image will always be element number 29 in the vector). 

Let's get started. In the cell below, print the `.shape` of both `X_train` and `X_test`


```python
np.shape(X_test), np.shape(X_train)
```




    ((10000, 28, 28), (60000, 28, 28))



We can interpret these numbers as saying "`X_train` consists of 60,000 images that are 28x28". We'll need to reshape them from `(28, 28)`, a 28x28 matrix, to `(784,)`, a 784-element vector. However, we need to make sure that the first number in our reshape call for both `X_train` and `X_test` still correspond to the number of observations we have in each. 

In the cell below:

* Use the `.reshape()` method to reshape `X_train`. The first parameter should be `60000`, and the second parameter should be `784` 
* Similarly, reshape `X_test` to `10000` and `784`  
* Also, chain both `.reshape()` calls with an `.astype('float32')`, so that we convert our data from type `uint8` to `float32` 


```python
X_train = X_train.reshape(60000, 784)
X_test = X_test.reshape(10000, 784)
```

Now, let's check the shape of our training and test data again to see if it worked. 


```python
np.shape(X_test), np.shape(X_train)
```




    ((10000, 784), (60000, 784))



Great! Now, we just need to normalize our data!

## Normalizing Image Data

Since all pixel values will always be between 0 and 255, we can just scale our data by dividing every element by 255! Run the cell below to do so now. 


```python
X_train = X_train/ 255
X_test = X_test/ 255
```

Great! We've now finished preprocessing our image data. However, we still need to deal with our labels. 

## Preprocessing our Labels

Let's take a quick look at the first 10 labels in our training data:


```python
y_train[:10]
```




    array([5, 0, 4, 1, 9, 2, 1, 3, 1, 4], dtype=uint8)



As we can see, the labels for each digit image in the training set are stored as the corresponding integer value -- if the image is of a 5, then the corresponding label will be `5`. This means that this is a **_Multiclass Classification_** problem, which means that we need to **_One-Hot Encode_** our labels before we can use them for training. 

Luckily, Keras provides a really easy utility function to handle this for us. 

In the cell below: 

* Use the function `to_categorical()` to one-hot encode our labels. This function can be found in the `keras.utils` sub-module. Pass in the following parameters:
    * The object we want to one-hot encode, which will be `y_train`/`y_test` 
    * The number of classes contained in the labels, `10` 


```python
y_train = keras.utils.to_categorical(y_train, 10)
y_test = keras.utils.to_categorical(y_test, 10)
```

Great. Now, let's examine the label for the first data point, which we saw was `5` before. 


```python
np.shape(y_train)
```




    (60000, 10)



Perfect! As we can see, the fifth index is set to `1`, while everything else is set to `0`. That was easy!  Now, let's get to the fun part -- building our model!

## Building our Model

For the remainder of this lab, we won't hold your hand as much -- flex your newfound Keras muscles and build an MLP with the following specifications:

* A `Dense` hidden layer with `64` neurons, and a `'tanh'` activation function. Also, since this is the first hidden layer, be sure to pass in `input_shape=(784,)` in order to create a correctly-sized input layer!
* Since this is a multiclass classification problem, our output layer will need to be a `Dense` layer where the number of neurons is the same as the number of classes in the labels. Also, be sure to set the activation function to `'softmax'` 


```python
model_1  = Sequential()
model_1.add(Dense(64, activation='tanh', input_shape=(784,)))
model_1.add(Dense(10, activation='softmax'))

```

Now, compile your model with the following parameters:

* `loss='categorical_crossentropy'`
* `optimizer='sgd'`
* `metrics = ['accuracy']`


```python
model_1.compile(loss='categorical_crossentropy', optimizer='sgd', metrics = ['accuracy'])
```

Let's quickly inspect the shape of our model before training it and see how many training parameters we have. In the cell below, call the model's `.summary()` method. 


```python
model_1.summary()
```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_1 (Dense)              (None, 64)                50240     
    _________________________________________________________________
    dense_2 (Dense)              (None, 10)                650       
    =================================================================
    Total params: 50,890
    Trainable params: 50,890
    Non-trainable params: 0
    _________________________________________________________________
    

50,890 trainable parameters! Note that while this may seem large, deep neural networks in production may have hundreds or thousands of layers and many millions of trainable parameters!

Let's get on to training. In the cell below, fit the model. Use the following parameters:

* Our training data and labels
* `epochs=5`
* `batch_size=64`
* `validation_data=(X_test, y_test)`


```python
results_1 = model_1.fit(x=X_train, y=y_train, epochs=5, batch_size=64, validation_data=(X_test,y_test))
```

    Train on 60000 samples, validate on 10000 samples
    Epoch 1/5
    60000/60000 [==============================] - 2s 37us/step - loss: 0.8675 - acc: 0.7839 - val_loss: 0.5060 - val_acc: 0.8738
    Epoch 2/5
    60000/60000 [==============================] - 2s 36us/step - loss: 0.4608 - acc: 0.8803 - val_loss: 0.3941 - val_acc: 0.8964
    Epoch 3/5
    60000/60000 [==============================] - 2s 37us/step - loss: 0.3865 - acc: 0.8961 - val_loss: 0.3485 - val_acc: 0.9061
    Epoch 4/5
    60000/60000 [==============================] - 3s 52us/step - loss: 0.3495 - acc: 0.9039 - val_loss: 0.3213 - val_acc: 0.9114
    Epoch 5/5
    60000/60000 [==============================] - 3s 44us/step - loss: 0.3255 - acc: 0.9093 - val_loss: 0.3027 - val_acc: 0.9163
    

## Visualizing our Loss and Accuracy Curves

Now, let's inspect the model's performance and see if we detect any overfitting or other issues. In the cell below, create two plots:

* The `loss` and `val_loss` over the training epochs
* The `acc` and `val_acc` over the training epochs

**_HINT:_** Consider copying over the visualization function from the previous lab in order to save time!


```python
def visualize_training_results(results):
    """
    Input- Output of a Keras Model fit
    Output- Visualizing of Loss and Accuracy Curves
    """
    history = results.history
    plt.figure()
    plt.plot(history['val_loss'])
    plt.plot(history['loss'])
    plt.legend(['val_loss', 'loss'])
    plt.title('Loss')
    plt.xlabel('Epochs')
    plt.ylabel('Loss')
    plt.show()
    
    plt.figure()
    plt.plot(history['val_acc'])
    plt.plot(history['acc'])
    plt.legend(['val_acc', 'acc'])
    plt.title('Accuracy')
    plt.xlabel('Epochs')
    plt.ylabel('Accuracy')
    plt.show()
    
```


```python
visualize_training_results(results_1)
```


![png](index_files/index_35_0.png)



![png](index_files/index_35_1.png)


Pretty good! Note that since our validation scores are currently higher than our training scores, its extremely unlikely that our model is overfitting to the training data. This is a good sign -- that means that we can probably trust the results that our model is ~91.7% accurate at classifying handwritten digits!

## Building a Bigger Model

Now, let's add another hidden layer and see how this changes things. In the cells below, create a second model. This model should have the following architecture:

* Input layer and first hidden layer same as `model_1`
* Another `Dense` hidden layer, this time with `32` neurons and a `'tanh'` activation function
* An output layer same as `model_1` 


```python
model_2 = Sequential()
model_2.add(Dense(64, activation='tanh', input_shape=(784,)))
model_2.add(Dense(32, activation='tanh'))
model_2.add(Dense(10, activation='softmax'))
```

Let's quickly inspect the `.summary()` of the model again, to see how many new trainable parameters this extra hidden layer has introduced.


```python
model_2.summary()
```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_3 (Dense)              (None, 64)                50240     
    _________________________________________________________________
    dense_4 (Dense)              (None, 32)                2080      
    _________________________________________________________________
    dense_5 (Dense)              (None, 10)                330       
    =================================================================
    Total params: 52,650
    Trainable params: 52,650
    Non-trainable params: 0
    _________________________________________________________________
    

This model isn't much bigger, but the layout means that the 2080 parameters in the new hidden layer will be focused on higher layers of abstraction than the first hidden layer. Let's see how it compares after training. 

In the cells below, compile and fit the model using the same parameters you did for `model_1`.


```python
model_2.compile(loss='categorical_crossentropy', optimizer='sgd', metrics = ['accuracy'])
```


```python
results_2 = model_2.fit(x=X_train, y=y_train, epochs=5, batch_size=64, validation_data=(X_test,y_test))
```

    Train on 60000 samples, validate on 10000 samples
    Epoch 1/5
    60000/60000 [==============================] - 3s 42us/step - loss: 0.9415 - acc: 0.7678 - val_loss: 0.5381 - val_acc: 0.8719
    Epoch 2/5
    60000/60000 [==============================] - 2s 40us/step - loss: 0.4669 - acc: 0.8807 - val_loss: 0.3932 - val_acc: 0.8998
    Epoch 3/5
    60000/60000 [==============================] - 3s 44us/step - loss: 0.3739 - acc: 0.8990 - val_loss: 0.3344 - val_acc: 0.9114
    Epoch 4/5
    60000/60000 [==============================] - 3s 43us/step - loss: 0.3278 - acc: 0.9094 - val_loss: 0.3002 - val_acc: 0.9197
    Epoch 5/5
    60000/60000 [==============================] - 2s 41us/step - loss: 0.2984 - acc: 0.9159 - val_loss: 0.2771 - val_acc: 0.9251
    

Now, visualize the plots again. 


```python
visualize_training_results(results_2)
```


![png](index_files/index_44_0.png)



![png](index_files/index_44_1.png)


Slightly better validation accuracy, with no evidence of overfitting -- great! If you run the model for more epochs, you'll see the model's performance continues to improve until the validation metrics plateau and the model begins to overfit to training data. 

## A Bit of Tuning

As a final exercise, let's see what happens to the model's performance if we switch activation functions from `'tanh'` to `'relu'`. In the cell below, recreate  `model_2`, but replace all `'tanh'` activations with `'relu'`. Then, compile, train, and plot the results using the same parameters as the other two. 


```python
model_3 = Sequential()
model_3.add(Dense(64, activation='relu', input_shape=(784,)))
model_3.add(Dense(32, activation='relu'))
model_3.add(Dense(10, activation='softmax'))
```


```python
model_3.compile(loss='categorical_crossentropy', optimizer='sgd', metrics = ['accuracy'])
```


```python
model_3.summary()
```

    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    dense_9 (Dense)              (None, 64)                50240     
    _________________________________________________________________
    dense_10 (Dense)             (None, 32)                2080      
    _________________________________________________________________
    dense_11 (Dense)             (None, 10)                330       
    =================================================================
    Total params: 52,650
    Trainable params: 52,650
    Non-trainable params: 0
    _________________________________________________________________
    


```python
results_3 = model_3.fit(x=X_train, y=y_train, epochs=5, batch_size=64, validation_data=(X_test,y_test))
```

    Train on 60000 samples, validate on 10000 samples
    Epoch 1/5
    60000/60000 [==============================] - 3s 43us/step - loss: 1.0214 - acc: 0.7312 - val_loss: 0.4504 - val_acc: 0.8783
    Epoch 2/5
    60000/60000 [==============================] - 2s 40us/step - loss: 0.3965 - acc: 0.8902 - val_loss: 0.3350 - val_acc: 0.9057
    Epoch 3/5
    60000/60000 [==============================] - 2s 40us/step - loss: 0.3256 - acc: 0.9080 - val_loss: 0.2969 - val_acc: 0.9155
    Epoch 4/5
    60000/60000 [==============================] - 2s 40us/step - loss: 0.2908 - acc: 0.9169 - val_loss: 0.2677 - val_acc: 0.9229
    Epoch 5/5
    60000/60000 [==============================] - 2s 40us/step - loss: 0.2663 - acc: 0.9234 - val_loss: 0.2491 - val_acc: 0.9296
    


```python
visualize_training_results(results_3)
```


![png](index_files/index_51_0.png)



![png](index_files/index_51_1.png)


Performance improved even further! ReLU is one of the most commonly used activation functions around right now -- it's especially useful in computer vision problems like image classification, as we've just seen. 

## Summary

In this lab, you once again practiced and reviewed the process of building a neural network. This time, you built a more complex network with additional layers which improved the performance of your model on the MNIST dataset! 
