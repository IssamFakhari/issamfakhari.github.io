---
published: true
title: Age Prediction with keras 
layout: single
author_profile: false
read_time: true
tags: [machine learning , Deep Learning , CNN] 
categories: [deeplearning]
excerpt: " Deep Learning , CNN "
comments : true
toc: true
toc_sticky: true
---


------------------------

This article is just a **copy paste** from the following blog article and github repository  : 
- https://sefiks.com/2019/02/13/apparent-age-and-gender-prediction-in-keras/
- https://github.com/serengil/tensorflow-101/blob/master/python/apparent_age_prediction.ipynb

The original writer is **Sefik Ilkin Serengil**  ( https://github.com/serengil ) 

The content in the github repository https://github.com/kasamoh/Image_processing_learning/tree/master/Age_detection was also downloaded from the Sefik Ilkin Serengil github repository


-------------------------

Computer vision researchers of ETH Zurich University (Switzerland)  [announced](https://www.vision.ee.ethz.ch/en/publications/papers/proceedings/eth_biwi_01229.pdf)  a very successful apparent age and gender prediction models. They both shared how they designed the machine learning model and  [pre-trained weights](https://data.vision.ee.ethz.ch/cvl/rrothe/imdb-wiki/)  for transfer learning. Their implementation was based on Caffe framework. Even though I tried to convert Caffe model and weights to Keras / TensorFlow, I couldn’t handle this. That’s why, I intend to adopt this research from scratch in Keras.

![katy-perry-ages](https://i2.wp.com/sefiks.com/wp-content/uploads/2019/02/katy-perry-ages.jpg?resize=618%2C347&ssl=1)

Katy Perry Transformation

### 1-Dataset

The original work consumed face pictures collected from IMDB (7 GB) and Wikipedia (1 GB). You can find these data sets  [here](https://data.vision.ee.ethz.ch/cvl/rrothe/imdb-wiki/). In this post, I will just consume wiki data source to develop solution fast. You should  **download faces only**  files.

----------


Extracting wiki_crop.tar creates 100 folders and an index file (wiki.mat). The index file is saved as Matlab format. We can read Matlab files in python with SciPy.
```
import scipy.io
mat = scipy.io.loadmat('wiki_crop/wiki.mat')
 ```
 Converting pandas dataframe will make transformations easier.
 ```
import pandas as pd
df = pd.DataFrame(index = range(0,instances), columns = columns)
 
for i in mat:
if i == "wiki":
current_array = mat[i][0][0]
for j in range(len(current_array)):
df[columns[j]] = pd.DataFrame(current_array[j][0])
```
![wiki-crop-dataset](https://i1.wp.com/sefiks.com/wp-content/uploads/2019/02/wiki-crop-dataset.png?resize=1140%2C272&ssl=1)

Data set contains date of birth (dob) in Matlab datenum format. We need to convert this to Python datatime format. We just need the birth year.

 ```
from datetime import datetime, timedelta
def datenum_to_datetime(datenum):
days = datenum % 1
hours = days % 1 * 24
minutes = hours % 1 * 60
seconds = minutes % 1 * 60
exact_date = datetime.fromordinal(int(datenum)) \
+ timedelta(days=int(days)) + timedelta(hours=int(hours)) \
+ timedelta(minutes=int(minutes)) + timedelta(seconds=round(seconds)) \
- timedelta(days=366)
 
return exact_date.year
 
df['date_of_birth'] = df['dob'].apply(datenum_to_datetime)
 ```
![wiki-crop-dataset-dob](https://i0.wp.com/sefiks.com/wp-content/uploads/2019/02/wiki-crop-dataset-dob.png?resize=1140%2C262&ssl=1)

Extracting date of birth from matlab datenum format

Now, we have both date of birth and photo taken time. Subtracting these values will give us the ages.
 ```
df['age'] = df['photo_taken'] - df['date_of_birth']
 ```


#### 2- Data cleaning

Some pictures don’t include people in the wiki data set. For example, a vase picture exists in the data set. Moreover, some pictures might include two person. Furthermore, some are taken distant. Face score value can help us to understand the picture is clear or not. Also, age information is missing for some records. They all might confuse the model. We should ignore them. Finally, unnecessary columns should be dropped to occupy less memory.

```
#remove pictures does not include face
df = df[df['face_score'] != -np.inf]
 
#some pictures include more than one face, remove them
df = df[df['second_face_score'].isna()]
 
#check threshold
df = df[df['face_score'] >= 3]
 
#some records do not have a gender information
df = df[~df['gender'].isna()]
 
df = df.drop(columns = ['name','face_score','second_face_score','date_of_birth','face_location'])
```
Some pictures are taken for unborn people. Age value seems to be negative for some records. Dirty data might cause this. Moreover, some seems to be alive for more than 100. We should restrict the age prediction problem for 0 to 100 years.

```
#some guys seem to be greater than 100. some of these are paintings. remove these old guys
df = df[df['age'] <= 100]
 
#some guys seem to be unborn in the data set
df = df[df['age'] > 0]
```
The raw data set will be look like the following data frame.

![wiki-crop-dataset-raw](https://i1.wp.com/sefiks.com/wp-content/uploads/2019/02/wiki-crop-dataset-raw.png?resize=452%2C231&ssl=1)
We can visualize the target label distribution.
```
histogram_age = df['age'].hist(bins=df['age'].nunique())
histogram_gender = df['gender'].hist(bins=df['gender'].nunique())
```
![age-gender-distribution](https://i2.wp.com/sefiks.com/wp-content/uploads/2019/02/age-gender-distribution.png?resize=1083%2C403&ssl=1)

Full path column states the exact location of the picture on the disk. We need its pixel values.

```
target_size = (224, 224)
 
def getImagePixels(image_path):
img = image.load_img("wiki_crop/%s" % image_path[0], grayscale=False, target_size=target_size)
x = image.img_to_array(img).reshape(1, -1)[0]
#x = preprocess_input(x)
return x
 
df['pixels'] = df['full_path'].apply(getImagePixels)
```

We can extract the real pixel values of pictures

![wiki-crop-dataset-pixels](https://i2.wp.com/sefiks.com/wp-content/uploads/2019/02/wiki-crop-dataset-pixels.png?resize=822%2C233&ssl=1)
#### Apparent age prediction model

Age prediction is a regression problem. But researchers define it as a classification problem. There are 101 classes in the output layer for ages 0 to 100. they applied  [transfer learning](https://sefiks.com/2017/12/10/transfer-learning-in-keras-using-inception-v3/)  for this duty. Their choice was VGG for imagenet.

#### Preparing input output

Pandas data frame includes both input and output information for age and gender prediction tasks. Wee should just focus on the age task.

```

classes = 101 #0 to 100
target = df['age'].values
target_classes = keras.utils.to_categorical(target, classes)
 
features = []
 
for i in range(0, df.shape[0]):
features.append(df['pixels'].values[i])
 
features = np.array(features)
features = features.reshape(features.shape[0], 224, 224, 3)
```

The final data set consists of 22578 instances. It is splitted into 15905 train instances and 6673 test instances .

#### 3- Transfer learning

As mentioned, researcher used VGG imagenet model. Still, they tuned weights for this data set. Herein, I prefer to use **VGG-Face**  model. Because, this model is tuned for face recognition task. In this way, we might have outcomes for patterns in the human face.
```

#VGG-Face model
model = Sequential()
model.add(ZeroPadding2D((1,1),input_shape=(224,224, 3)))
model.add(Convolution2D(64, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(64, (3, 3), activation='relu'))
model.add(MaxPooling2D((2,2), strides=(2,2)))
 
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(128, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(128, (3, 3), activation='relu'))
model.add(MaxPooling2D((2,2), strides=(2,2)))
 
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(256, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(256, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(256, (3, 3), activation='relu'))
model.add(MaxPooling2D((2,2), strides=(2,2)))
 
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(512, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(512, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(512, (3, 3), activation='relu'))
model.add(MaxPooling2D((2,2), strides=(2,2)))
 
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(512, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(512, (3, 3), activation='relu'))
model.add(ZeroPadding2D((1,1)))
model.add(Convolution2D(512, (3, 3), activation='relu'))
model.add(MaxPooling2D((2,2), strides=(2,2)))
 
model.add(Convolution2D(4096, (7, 7), activation='relu'))
model.add(Dropout(0.5))
model.add(Convolution2D(4096, (1, 1), activation='relu'))
model.add(Dropout(0.5))
model.add(Convolution2D(2622, (1, 1)))
model.add(Flatten())
model.add(Activation('softmax'))
```
Load the pre-trained weights for VGG-Face model. 
```
model.load_weights('vgg_face_weights.h5')
```
We should lock the layer weights for early layers because they could already detect some patterns. Fitting the network from scratch might cause to lose this important information. I prefer to freeze all layers except last 3 convolution layers (make exception for last 7 model.add units). Also, I cut the last convolution layer because it has 2622 units. I need just 101 (ages from 0 to 100) units for age prediction task. Then, add a custom convolution layer consisting of 101 units.

```
for layer in model.layers[:-7]:
	layer.trainable = False
 
base_model_output = Sequential()
base_model_output = Convolution2D(101, (1, 1), name='predictions')(model.layers[-4].output)
base_model_output = Flatten()(base_model_output)
base_model_output = Activation('softmax')(base_model_output)
 
age_model = Model(inputs=model.input, outputs=base_model_output)
```

#### 4-Training

This is a multi-class classification problem. Loss function must be categorical  crossentropy. Optimization algorithm will be  Adam to converge loss faster. I create a checkpoint to monitor model over iterations and avoid overfitting. The iteration which has the minimum validation loss value will include the optimum weights. That’s why, I’ll monitor validation loss and save the best one only.

To avoid overfitting, I feed random 256 instances for each epoch.

```
age_model.compile(loss='categorical_crossentropy', optimizer=keras.optimizers.Adam(), metrics=['accuracy'])
 
checkpointer = ModelCheckpoint(filepath='age_model.hdf5'
, monitor = "val_loss", verbose=1, save_best_only=True, mode = 'auto')
 
scores = []
epochs = 250; batch_size = 256
 
for i in range(epochs):
print("epoch ",i)
 
ix_train = np.random.choice(train_x.shape[0], size=batch_size)
 
score = age_model.fit(train_x[ix_train], train_y[ix_train]
, epochs=1, validation_data=(test_x, test_y), callbacks=[checkpointer])
 
scores.append(score)
```

It seems that validation loss reach the minimum. Increasing epochs will cause to overfitting.
![age-prediction-loss](https://i1.wp.com/sefiks.com/wp-content/uploads/2019/02/age-prediction-loss.png?resize=512%2C356&ssl=1)

#### 5- Model evaluation on test set

We can evaluate the final model on the test set.
```
age_model.evaluate(test_x, test_y, verbose=1)
```
This gives both validation loss and accuracy respectively for 6673 test instances. It seems that we have the following results.
				[2.871919590848929, 0.24298789490543357]

24% accuracy seems very low, right? Actually, it is not. Herein, researchers develop an age prediction approach and convert classification task to regression. They propose that you should multiply each softmax out with its label. Summing this multiplications will be the apparent age prediction.

![age-prediction-approach](https://i2.wp.com/sefiks.com/wp-content/uploads/2019/02/age-prediction-approach.png?resize=393%2C310&ssl=1)

Age prediction approach

This is a very easy operation in Python numpy.
```

predictions = age_model.predict(test_x)
 
output_indexes = np.array([i for i in range(0, 101)])
apparent_predictions = np.sum(predictions * output_indexes, axis = 1)
```

Herein, mean absolute error metric might be more meaningful to evaluate the system.


```
mae = 0
 
for i in range(0 ,apparent_predictions.shape[0]):
prediction = int(apparent_predictions[i])
actual = np.argmax(test_y[i])
 
abs_error = abs(prediction - actual)
actual_mean = actual_mean + actual
 
mae = mae + abs_error
 
mae = mae / apparent_predictions.shape[0]
 
print("mae: ",mae)
print("instances: ",apparent_predictions.shape[0])
```

Our apparent age prediction model averagely predict ages ± 4.65 error. This is acceptable.

#### 6- Testing model on custom images

We can feel the power of the model when we feed custom images into it.
```
from keras.preprocessing import image
from keras.preprocessing.image import ImageDataGenerator
 
def loadImage(filepath):
test_img = image.load_img(filepath, target_size=(224, 224))
test_img = image.img_to_array(test_img)
test_img = np.expand_dims(test_img, axis = 0)
test_img /= 255
return test_img
 
picture = "marlon-brando.jpg"
prediction = age_model.predict(loadImage(picture))
```

Prediction variable stores distribution for each age class. Monitoring it might be intersting.

```
y_pos = np.arange(101)
plt.bar(y_pos, prediction[0], align='center', alpha=0.3)
plt.ylabel('percentage')
plt.title('age')
plt.show()
```
![age-prediction-distribution](https://i2.wp.com/sefiks.com/wp-content/uploads/2019/02/age-prediction-distribution.png?resize=492%2C324&ssl=1)
This is the age prediction distribution of Marlon Brando in Godfather. The most dominant age class is 44 whereas weighted age is 48 which is the exact age of him in 1972.

We’ll calculate apparent age from these age distributions : 
```
img = image.load_img(picture)
plt.imshow(img)
plt.show()
 
print("most dominant age class (not apparent age): ",np.argmax(prediction))
 
apparent_age = np.round(np.sum(prediction * output_indexes, axis = 1))
print("apparent age: ", int(apparent_age[0]))
```
Results are very satisfactory even though it does not have a good perspective. Marlon Brando was 48 and Al Pacino was 32 in Godfather Part I.

![age-prediction-for-godfather](https://i0.wp.com/sefiks.com/wp-content/uploads/2019/02/age-prediction-for-godfather.png?resize=603%2C339&ssl=1)


I pushed the source code for  [ age prediction](https://github.com/kasamoh/Image_processing_learning/tree/master/Age_detection) to GitHub.
