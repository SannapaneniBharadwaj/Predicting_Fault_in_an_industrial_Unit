## Problem Description
ExampleCo, Inc has a problem: maintenance on their widgets is expensive. They have contracted with Tagup to help them implement predictive maintenance. They want us to provide a model that will allow them to prioritize maintenance for those units most likely to fail, and in particular to gain some warning---even just a few hours!---before a unit does fail.

They collect two kinds of data for each unit. First, they have a remote monitoring system for the motors in each unit, which collects information about the motor (rotation speed, voltage, current) as well as two temperature probes (one on the motor and one at the inlet). Unfortunately, this system is antiquated and prone to communication errors, which manifest as nonsense measurements. Second, they have a rule-based alarming system, which can emit either warnings or errors; the system is known to be noisy, but it's the best they've got. 

They have given us just over 100MB of historical remote monitoring data from twenty of their units that failed in the field. The shortest-lived units failed after a few days; the longest-lived units failed after several years. Typical lifetimes are on the order of a year. This data is available in .csv files under `data/train` in this repository. In addition, they have provided us with operating data from their thirty active units for the past month; this data is available under `data/test` in this repository.

You have two main objectives. First, **tell us as much as you can about the process that generated the data**. Does it show meaningful clustering? Do the observations appear independent? How accurately can we forecast future observations, and how long a window do we need to make an accurate forecast? Feel free to propose multiple models, but be sure to discuss the ways each is useful and the ways each is not useful. Second, **predict which of the thirty active units are most likely to fail**. The data from these units are in `data/test`.


## Work Done

##### Notebook file has detailed comments explaining the work https://nbviewer.jupyter.org/github/SannapaneniBharadwaj/Predicting_Fault_in_an_industrial_Unit/blob/master/Prognostics_Notebook.ipynb

#### Data Preprocessing and exploratory analysis

1) After inspecting the given data from RMS00 unit, I've realized that there are very large outliers and seasonality in the data. So, I've removed the outliers using IQR method and removed the seasonality by subtracting it with a modeled seasonal function.

2) I've combined the data from all available units by resampling the  sensor readings hourly. I've also merged the alarms data with RMS data by converting 'message' attribute into 'warning_frequency_per_hour' and 'error_freqency_per_hour'

3) To get the information regarding the unit failure, I've created a variable 'number of hours to the failure'. I've also created a label called 'status', which is '1' if the 'number of hours to the failure' is less than 36 hrs.

4) Here I've inspected the data using Pairplots and correlation plots, and realized that there is no correlation to the 'number of hours to the failure' variable with any other variables.


Throughout the analysis mentioned below, I've used only data from 18 units to train any classifier. Unit 18 data(I mean '0018' data) is kept separately for validation purposes, including selecting the model. Unit 19('0019') data is kept separately for Testing puropose, it is used only on the best classifier that I've got, this is not even used for selecting the models.
#### Simple classification using independent sensor readings

Input: Current sensor readings
Ouput: Status Label

5) To see, if there is information in the independent sensor readings that can predict the unit fault, I've implemented few classifiers to classify the status label using sensor readings. However, the classification is not successful.


## Timeseries Analysis

Timeseries vector: Past 30 attributes as a sequence of shape 7x30

#### Dimensionality reduction using DTW

6) Now to see if there is any information in the 'sequences of sensor readings' that can seperate the classes, I've implemented Dimensionality Reduction Techniques (MDS and t-SNE), using 'Dynamic Time Warping' as the distance measure between two time sequences.

#### Timeseries clustering using DTW

7) As the data observed in MDS and t-SNE plots seems to have few clusters based on classes. I have tried if DBSCAN can capture those clusters.

#### Timeseries Forecasting

8) To see if we can forecast future sensor readings from the past readings, I've implemented a LSTM that can take the past 30 hr sensor readings sequence as input and output the future 10 hr sensor readings. The forecasting model is very accurate with a RMSE of 0.077 on a test data that is kept seperately. 

#### Timeseries Classification to predict the fault

Input: Past 30 attributes as a sequence of shape 7x30
Output: Status label

9) As I've observed clusters, although overlapping, in the tSNE and MDS plots. I felt that KNN would be a good classification algorithm for this kind of data distribution. So, I've decided to implement KNN using DTW as distance metric. However as the complexity of the KNN and DTW are very high I couldn't implement the algorithm on the whole data, so I've subsampled the data and used it for training.

10) Classification model using KNN is not good, so I've implemented LSTM with the same attributes as input and output. I've used LSTMs becuase they are the best at remembering long term dependencies and are proven to work well with time series sequences. Problem here is that the data is highly unbiased, so I've added class weights to nullify, but it wasn't helpful.

11) As CNNs has some history of working well with noisy time series data, I've used recpolots to convert time series sequences into images, and trained CNN. In recplot conversion, each sensor sequence of len N will be converted into an image of shape NxN. I've added multiple sensor readings as different channels in an image. So each time series sequence of shape 7x30 is converted into an image of shape 7x30x30
Input: Recplot image of shape 7x30x30, obtained from each time series sequence 
Output: Status label

12) Even the CNN trained above suffered from data imbalance, so I've decided to use subsampled data for training the CNN. However, it seems like the subsampled data is very small that it couldn't train the CNN well.


13) As the LGBM used directly on the instances, is the model that gave better accuracy compared to other complicated models, I've used it to classify the test data from Unit 19. 

14) LGBM is also used to predict the Unit faults that are in the given 'Test data'. To decide if the Unit is failing or not, I've taken the last 10 hrs of readings and passed it through LGBM and using the majority voting, I have classified units ['20', '35', '40'](ie units '0020','0035','0040') as failing.

## -----------------------------------------------------------------------------------------------------------

##### * Observations in the data are found to be very noise and have seasonality. 
##### * Observations are not independent, like other time sequences they depend on the previous observations. 
##### * Clusters in the data are meaningful but are not well separated and are hard to extract. 
##### * We can forecast the future 10 hr observations very accurately with an RMSE of 0.077 using the past 30 hrs observations. 
##### * Units '0020', '0035', '0040' are predicted to fail.



##### Related work that is done but not included in this notebook: 

I have tried different techniques to improve classification by balancing the data. 

1) Augmented the class1 data. 

2) Performed undersampling and oversampling. 

3) Divided 'status' classes equally based on their 'number of hours to failure' so that the data will be balanced

but all these techniques are giving similar results



#### Future Work: 
1) Recheck and validate the assumptions that are made while performing the analysis.
2) Feature Engineering, could generate new features and use them for classification. 
