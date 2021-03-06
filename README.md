# DEEP LEARNING USING KERAS TUTORIAL

## INTRODUCTION
This post will take you through a simple implementation of **convolutional neural netwotks** using [keras](https://keras.io/) for classification of **MNIST** dataset.

Keras is a deep learning library built over [theano](deeplearning.net/software/theano/) and [tensorflow](https://www.tensorflow.org/).It is very easy for beginners to get started on neural networks implementation using keras.So let's start by importing the essentials.You can learn to install them from [here](installation)
```python
import numpy as np
import cv2
from matplotlib import pyplot as plt 
```
NumPy is the fundamental package for scientific computing with Python.OpenCV is used along with matplotlib just for showing some of the results in the end.
```python
from keras.models import Sequential 
```
The core data structure of Keras is a model, a way to organize layers. The simplest type of model is the Sequential model.It is a linear stack of neural network layers for feed forward cnn
```python
from keras.layers import Dense, Dropout, Activation, Flatten  
from keras.layers import Convolution2D, MaxPooling2D
```
Now we are importing core layers for our CNN netwrok.Dense layer is actually a [fully-connected layer](http://cs231n.github.io/convolutional-networks/#fc).

[Dropout](https://en.wikipedia.org/wiki/Dropout_(neural_networks)) is a regularization technique used for reducing overfitting by randomly dropping output units in the network.Activation Function is used for introducing
non-linearity and Flatten is used for converting output matrices into a vector.

The second line is used for importing convolutional and pooling layers.
```python
from keras.utils import np_utils
```
This library will be useful for converting shape of our data.
```python
from keras.datasets import mnist
```
MNIST dataset is already available in keras library.So, you dont need to download it separately.
## LOAD DATA
MNIST datset cosists of 28x28 size images of handwritten digits.We will load pre-shuffled data into training  and testing  sets.
```python
(X_train, y_train), (X_test, y_test) = mnist.load_data() 
```
We can plot and see any image(let's say the first image)
```python
plt.imshow(X_train[0])
plt.show()
```
We need to reshape our data to include number of channels(i.e depth).Since we have grayscal images, no. of channels is equal to 1(for RGB, it is 3).The present shape of our data is (N,H,W) where N = total number of examples or batch size while H and W refer to height and width of the image respectively. Now reshaping will depend on which library your keras is build on:-

- For theano, the format is (N,C,H,W)
```python
X_train = X_train.reshape(X_train.shape[0],1, 28, 28) 
X_test = X_test.reshape(X_test.shape[0],1, 28, 28)
```
- For tensorflow, the format is (N,H,W,C)
```python
X_train = X_train.reshape(X_train.shape[0],28, 28, 1) 
X_test = X_test.reshape(X_test.shape[0], 28, 28, 1)
```
where C means number of channels.

We can now check the shape of our input using the following command
```python
print X_train.shape 
#(60000,28,28,1)
```
Conversion to float32 data type
```python
X_train = X_train.astype('float32')
X_test = X_test.astype('float32')
```
[Normalising/Feature scaling](https://www.coursera.org/learn/machine-learning/lecture/xx3Da/gradient-descent-in-practice-i-feature-scaling) input data for getting all our data in a similar range (i.e [0,1]).It helps in faster convergence of model
```python
X_train /= 255
X_test /= 255
```
For classification of digits, we need 10 classes ranging from 0-9. So, our output set must contain such labels.We can check present shape of our output and even print first 10 of them using 
```python
print y_train.shape
print y_test.shape[:10] 
```
So we need to convert 1-dimensional class arrays to 10-dimensional class matrices
```python
Y_train = np_utils.to_categorical(y_train, 10) 
Y_test = np_utils.to_categorical(y_test, 10)
print y_test.shape
```
So our data is now divided into 10 classes.
## NETWORK CONSTRUCTION
Let's start the construction of our model.
```python
model = Sequential()
```
Now we will add our first convulation layer with 32 kernels of sixe 3x3. The default stride value is (1,1).The activation function used is [ReLu](http://cs231n.github.io/neural-networks-1/#actfun) which is defined as max(x,0).Hence,it sets the negative elements to zero..Ourinput sample must be fed in (depth,width,height) format
```python
model.add(Convolution2D(32, (3, 3), activation='relu', input_shape=(28,28,1)))
```
Now we can check the shape of our output from this layer.
```python
 model.output_shape
 #(32,26,26)
 ```
Let's add more layers. Note that we need to define the shape of our input layer for only the first convolution layer.
We are adding another convolution layer with 32 kernels of the same size as before with same activation function.

Next we add a pooling layer which is used for downsampling as well as making our model somewhat translation invariant.

The last layer is dropout.It is used for regularisation by randomly disabling/dropping neurons during learning phase. Here, 0.25 refers to the probability of keeping output neurons.

```python 
model.add(Convolution2D(32, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2,2)))
model.add(Dropout(0.25))
```
Lastly we will add the Full-connected layers. We reshape out output into a vector. The first FC layer contains 128 output neurons. The second Fc layer is the main classification layer with 10 output neurons for every one of the 10 digits.We are using [Softmax classifier](http://cs231n.github.io/linear-classify/#softmax)

```python
model.add(Flatten())
model.add(Dense(128, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(10, activation='softmax'))
```
Now we will define our cost/loss function and optimizing algorithm.
```python
model.compile(loss='categorical_crossentropy',optimizer='adam',metrics=['accuracy'])
```
Let's have a look over our model
```python
model = Sequential()
model.add(Convolution2D(32, (3, 3), activation='relu', input_shape=(28,28,1)))
model.add(Convolution2D(32, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2,2)))
model.add(Dropout(0.25))
model.add(Flatten())
model.add(Dense(128, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(10, activation='softmax'))
model.compile(loss='categorical_crossentropy',optimizer='adam',metrics=['accuracy'])
```
## FITTING THE MODEL
To fit our model to trainig data, we need to define the batch size and number of [epochs](https://stackoverflow.com/questions/31155388/meaning-of-an-epoch-in-neural-networks-training). Setting verbose = 1 gives an insight into the training process i.e. it displays remaining time, loss and accuracy for each epoch.
```python
model.fit(X_train, Y_train, batch_size=32, nb_epoch=1, verbose=1)
```


