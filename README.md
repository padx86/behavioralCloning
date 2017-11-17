# Project 3: Behavioral cloning

## Data collection

In order to collect an appropriate amount of data 2 Laps clockwise und 2 Laps counter clockwise where manually driven on Track1 trying to stay around the middle of the track.
The data capture resulted in 2080 images counter clockwise and 1707 images clockwise with corresponding steering angles giving a total of 3787 images.

## Data preparation and augmentation
Data augmentation was done by using the images of all 3 cameras mounted to the vehicle and applying penalties to the steering angles for the left and right images enhancing the total number of samples to 11361. 
To generate even more data, the array containing the samples was duplicated and marked for the images to be flipped horizontally and multiply the steer angle by -1 in the generator.

The data preparation, specifically normalization and cropping, was applied to the beginning of the Network as Lambda and Cropping2D layers. The details will be given in the section describing the Behavioral Cloning Network (Model architecture).

## Data generation 

To give the network training the ability to handle a large amount of data a *generator* was implemented. The generator takes a list of all samples consisting of the image file paths on the aws vm and corresponding steer angles. The paths were corrected to work on aws vm during csv read operation and appended to the sample-list at first. Within the generator the images are read into the current batch and flipped horrizontally if desired. Finally features and labels are converted to numpy arrays and a batch is returned for network training.

## Model architecture

After trial of different network architectures(see trial section) a finally architecture was chosen with appropriate performance.
For data preparation means a normalization layer was added taking the input image shaped(160,320,3) and normalizing it for each pixel to be in range of [-0.5,0.5]. Then a cropping layer was used as the second layer cropping the image(70px off top, 25px off bottom) to retrieve the region of interest.

The actual network consists of a convolutional layer as first layer using (6,8,14) convolutions with strides of (2,2) and 'relu' activation. This layer is follow by an inception layer merging (3,3,8) and (5,5,8) convolutional layers and a (3,3) MaxPooling layer. The final layer is a fully connected layer outputting the steering angle.

Note: Using more than one Dense Layers seems to make the network "underreact"

## Overfitting prevention

In order to prevent overfitting the network dropout was applied to the nn at two stages. The first dropout of 50% was used after the first convolutional layer. The second dropout affects the output of the entire inception module. The dropout layers had a significant positive effect on the network performance.

## Model summary

The complete network architecture is viewed using:
```
model.summary()
```
outputting:
```
____________________________________________________________________________________________________
Layer (type)                     Output Shape          Param #     Connected to                     
====================================================================================================
input_1 (InputLayer)             (None, 160, 320, 3)   0                                            
____________________________________________________________________________________________________
lambda_1 (Lambda)                (None, 160, 320, 3)   0           input_1[0][0]                    
____________________________________________________________________________________________________
cropping2d_1 (Cropping2D)        (None, 65, 320, 3)    0           lambda_1[0][0]                   
____________________________________________________________________________________________________
convolution2d_1 (Convolution2D)  (None, 30, 157, 14)   2030        cropping2d_1[0][0]               
____________________________________________________________________________________________________
dropout_1 (Dropout)              (None, 30, 157, 14)   0           convolution2d_1[0][0]            
____________________________________________________________________________________________________
convolution2d_2 (Convolution2D)  (None, 30, 157, 8)    120         dropout_1[0][0]                  
____________________________________________________________________________________________________
convolution2d_4 (Convolution2D)  (None, 30, 157, 8)    120         dropout_1[0][0]                  
____________________________________________________________________________________________________
maxpooling2d_1 (MaxPooling2D)    (None, 30, 157, 14)   0           dropout_1[0][0]                  
____________________________________________________________________________________________________
convolution2d_3 (Convolution2D)  (None, 30, 157, 8)    584         convolution2d_2[0][0]            
____________________________________________________________________________________________________
convolution2d_5 (Convolution2D)  (None, 30, 157, 8)    1608        convolution2d_4[0][0]            
____________________________________________________________________________________________________
convolution2d_6 (Convolution2D)  (None, 30, 157, 8)    120         maxpooling2d_1[0][0]             
____________________________________________________________________________________________________
merge_1 (Merge)                  (None, 30, 157, 24)   0           convolution2d_3[0][0]            
                                                                   convolution2d_5[0][0]            
                                                                   convolution2d_6[0][0]            
____________________________________________________________________________________________________
dropout_2 (Dropout)              (None, 30, 157, 24)   0           merge_1[0][0]                    
____________________________________________________________________________________________________
flatten_1 (Flatten)              (None, 113040)        0           dropout_2[0][0]                  
____________________________________________________________________________________________________
dense_1 (Dense)                  (None, 1)             113041      flatten_1[0][0]                  
====================================================================================================
Total params: 117,623
Trainable params: 117,623
Non-trainable params: 0
____________________________________________________________________________________________________
```

An advantage of this network architecture is the small number of only 117 623 trainable parameters giving a model.h5 of around 1.4 mb.

## Model training

The model was trained using Adaptive Moment Estimation (Adam) optimizer minimizing the mean square error (mse) during training. A batch size of 128 was used. An epoch count of 8 was used to train the model.

## Trial

Multiple architectures were applied and the model architectures for each network can be found in Networksummaries.html. The stored network .h5 can be found in ./saved directory.
An overview of the network performances is given in the following table:

|Network|Performance\*|Point of failure\*\*|comment|
Net001.h5|0|immideatly|poor performance high steering angles|
Net002.h5|0|immideatly|seems smoother than Net001.h5|
Net003.h5|0|immideatly|like Net002.h5|
Net004.h5|3|at bridge|Drives high steering angles in sharp corners|
Net005.h5|5|first dirt curb directly after bridge|Very smooth steering(low angels)on straight tracks|
Net006.h5|2|first corner|does not steer over 1 %|
Net007.h5|6|after bridge to the left|smooth steering, but large enough angles to get round sharp curves|
Net008.h5|7|after bridge to the right|extremly smooth steering|
Net009.h5|6|after bridge to the right|oversteers(looks like sine), same topology as Net007.h5|
Net010.h5|5|first sharp corner to the right|
Net011.h5|7|-|performance not ideal, but made whole track|
Net012.h5|8|-|performance not ideal, but made whole track|
Net013.h5|5|first sharp corner to the right	oversteering|
Net014.h5|3|hit Bridge	oversteering on yellow line, not middle of road|
Net015.h5|9|-|good performance. Didn't leave lane once|
Net016.h5|6|first sharp corner to the right|smooth steering|
Net017.h5|8|-|Drives over yellow marking|

\* Performance is a personal rating in range 0-10, 0 being very poor, 10 being very good
\*\* Point of failure describes the position the vehicle left the road
