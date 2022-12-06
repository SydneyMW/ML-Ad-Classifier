# ECS 171 Ad Classification Project: Final Writeup

## Introduction

&nbsp;&nbsp;&nbsp;&nbsp;Among the plethora of elements on a modern website, viewers are almost guaranteed to come across the familiar, bothersome advertisement images. Popular search engines are taking note of this fact, as seen in Google’s Panda update, where the Panda algorithm helped the search engine rank the quality of a web page based on the locations and density of the page’s ads. And as the Internet continues to evolve, the ability to classify images as ads or not may be what decides whether a website appears on the front page of searches, or becomes buried in the sea of links. Therefore, having an accurate predictive classification model at one’s disposal will be vital for companies or individuals who want to be seen. Additionally, advertising also serves as a way of communicating information in an attempt to convince an audience of a particular idea. Young people are particularly susceptible to being targets of these ideas as they grow and develop. Therefore the power to recognize advertisements is an important tool for many reasons.  
&nbsp;&nbsp;&nbsp;&nbsp;While advertisements can come in all shapes and sizes, there may exist certain strongly correlated properties that allow for accurately classifying an image as an ad or not an ad. These properties or features include the geometry of the image, phrases in the URL, the image's URL and alt text, the anchor text, and words occurring near the anchor text. Using UC Irvine’s Internet Advertisements dataset, we leverage a multitude of supervised machine learning methods to tackle the problem of classifying images as ads, including logistic regression, neural network, and support vector machine.


## Figures

### Figure 1 &mdash; Pairplot
...
![image](https://user-images.githubusercontent.com/37519138/202835315-090892b8-6d0a-45a2-ac63-aa27daae4087.png)

### Figure 2 &mdash; Correlation Matrix
...
![image](https://user-images.githubusercontent.com/37519138/204183883-2f1ec76b-3907-4616-9d80-2567d45840af.png)

### Figure 3 &mdash; Neural Net Training Curves
...
![image](https://user-images.githubusercontent.com/37519138/205465185-981df9ed-3c46-440e-bb3d-24583296ba08.png)
![image](https://user-images.githubusercontent.com/37519138/205465190-b6bce9be-270f-49a1-9f5a-422972cdd67c.png)


## Methods

### 1. Data Pre-processing &mdash; Getting unstructured data into a structured format
The used [dataset](https://archive.ics.uci.edu/ml/datasets/internet+advertisements) comes in a custom unformatted document (described in the [dataset documentation](./ad.DOCUMENTATION)), so the first step is to parse the dataset and convert it to the more familiar and Pandas-friendly .csv format.  We use the features described in the [names file](./dataset/ad.names) to convert the [data](./dataset/ad.data) to a pandas dataframe, which we save as our [data csv file](./dataset/data.csv). 
The original data csv contains the observations but has no column names and looks like this:
|    |    0 |    1 |      2 |   3 |   4 |   5 |   6 |   7 |   8 |   9 |
|---:|-----:|-----:|-------:|----:|----:|----:|----:|----:|----:|----:|
|  0 |  125 |  125 | 1      |   1 |   0 |   0 |   0 |   0 |   0 |   0 |
|  1 |   57 |  468 | 8.2105 |   1 |   0 |   0 |   0 |   0 |   0 |   0 |
|  2 |   33 |  230 | 6.9696 |   1 |   0 |   0 |   0 |   0 |   0 |   0 |
|  3 |   60 |  468 | 7.8    |   1 |   0 |   0 |   0 |   0 |   0 |   0 |
|  4 |   60 |  468 | 7.8    |   1 |   0 |   0 |   0 |   0 |   0 |   0 |

Whereas the names file contains unstructured feature names in this format:
|    | Original                                                  |
|---:|:----------------------------------------------------------|
|  0 | | "w:\c4.5\alladA" names file -- automatically generated! |
|  1 |                                                           |
|  2 | ad, nonad | classes.                                      |
|  3 |                                                           |
|  4 | height: continuous.                                       |
|  5 | width: continuous.                                        |
|  6 | aratio: continuous.                                       |
|  7 | local: 0,1.                                               |
|  8 | | 457 features from url terms                             |
|  9 | url\*images+buttons: 0,1.                                  |
| 10 | url\*likesbooks.com: 0,1.                                  |
| 11 | url\*www.slake.com: 0,1.                                   |

The names are in order, and must be parsed and assigned to the dataframe to create the final csv of our features and observations.
This data processing and csv creation is accomplished in the [ad_data_parse notebook](./1_ad_data_parse.ipynb).
The final csv is in a much more accessible format:
|    |   height |   width |   aratio |   local |   url*images+buttons |   url*likesbooks.com |   url*www.slake.com |   url*hydrogeologist |   url*oso |   url*media |
|---:|---------:|--------:|---------:|--------:|---------------------:|---------------------:|--------------------:|---------------------:|----------:|------------:|
|  0 |      125 |     125 |   1      |       1 |                    0 |                    0 |                   0 |                    0 |         0 |           0 |
|  1 |       57 |     468 |   8.2105 |       1 |                    0 |                    0 |                   0 |                    0 |         0 |           0 |
|  2 |       33 |     230 |   6.9696 |       1 |                    0 |                    0 |                   0 |                    0 |         0 |           0 |
|  3 |       60 |     468 |   7.8    |       1 |                    0 |                    0 |                   0 |                    0 |         0 |           0 |
|  4 |       60 |     468 |   7.8    |       1 |                    0 |                    0 |                   0 |                    0 |         0 |           0 |

### 2. Data Pre-Processing &mdash; Cleaning, imputing and scaling data
Once we have formatted data, we want analyze its features, check for null or unknown values, and perform dropping/imputing/scaling as needed.  We find that the dataset has over 1500 features and over 3000 observations, with all but four features (height, width, aratio, and local) being binary-encoded.  Those that are not binary-encoded are represented with fixed-digit string literals due to the odd unstructured format of the original document, with unknown values represented by a '?' character.  

We find that for image height, width, and aspect ratio, about 27% of the records have unknown values, so we impute them with the respective mean values of those features.  For the 'local' variable, less than 1% of the data has unknown values, so we simply drop these observations.  The imputing and dropping is performed in the [preprocessing notebook](./3_preprocess_logreg_neuralnet.ipynb) with the following code:
```
valid_height_mean = int(pd.to_numeric(data.height[data.height != '   ?']).mean())
valid_width_mean = int(pd.to_numeric(data.width[data.width != '   ?']).mean())
valid_aratio_mean = int(pd.to_numeric(data.aratio[data.aratio != '     ?']).mean())

data.height[data.height == '   ?'] = height_mean
data.width[data.width == '   ?'] = width_mean
data.aratio[data.aratio == '     ?'] = aratio_mean
data = data[data.local != '   ?']
data = data[data.local != '?']
```
After imputing and dropping the unknown values, we must convert the data to numeric format.
```
data = data.apply(pd.to_numeric)
```
Once we convert the string values to numeric formatting, we can see that most of the features, such as url, are already one-hot encoded, so there's no additional encoding to be done.  The target label column 'ad' is represented with strings 'ad.' or 'nonad.', and must be converted to binary encoding:
```
data['is_ad'] = data.ad == 'ad.'
data = data.drop(columns = 'ad')
```
Min-max scaling is unnecessary for binary-encoded data, but can be applied to our several non-binary-encoded features, including height, width, and aspect ratio.  We therefore implement a min-max scaler to scale our data for more efficient computation and feature comparison in the future.  We choose min-max scaling due to its simplicity and the lack of normal distribution among all continuous variables, shown by our pairplot (Figure 1) in the data exploration section (described next).
We also split our data into training and testing data, with an 80:20 train:test ratio, and implement scaling with the code:
```
train, test = train_test_split(data, test_size=0.2, random_state=1)
X_train, y_train = train.drop(columns=['is_ad']), train['is_ad']
X_test, y_test = test.drop(columns=['is_ad']), test['is_ad']

scaler = MinMaxScaler()
X_train_mm = scaler.fit_transform(X_train)
X_test_mm = scaler.fit_transform(X_test)
```
This processing, imputing, splitting, and scaling is performed in the [preprocess_logreg_neuralnet notebook](./3_preprocess_logreg_neuralnet.ipynb) prior to model development.

### 3. Data Exploration &mdash; Visualizing data
The next step is to get to know the shape of the data, the statistical attributes of each feature, and its distribution. In order to do that, visual analysis can be conducted with the use of a pairplot and correlation matrix in the [data_exploration notebook](./2_data_exploration.ipynb). 

The pairplot (Figure 1) is generated using:
```
df_to_plot = df[['height', 'width', 'aratio', 'is_ad']]
sns.pairplot(df_to_plot, hue='is_ad', plot_kws=dict(size=0.5))
```
It shows the distribution of the height, width, and aratio variables, grouped by ad/non-ad classification.  The datapoints appear highly overlapping in the scatter plots, but from the histogram of the width and the aratio, we can see a clear distinction in the distribution of ads vs non-ads, with ads showing bimodal distribution for width and aratio, and non-ads showing roughly normal distribution.

The correlation matrix (Figure 2) is generated using:
```
heatmap_df = df[['height','width','aratio','is_ad']]
corr = heatmap_df.corr()
sns.heatmap(corr, cmap='RdBu', vmin=-1, vmax=1, annot=True)
```
It shows the correlation coefficients between these same features.  Notably, there is a high correlation between ad classification and image width, so width will certainly be an important feature to include in our model building.

### 4. Model 1 &mdash; Logistic Regression
The first model we chose to test and create with our pre-processed data is a logistic regression model.

The following code in the [preprocess_logreg_neuralnet notebook](./3_preprocess_logreg_neuralnet.ipynb) is to generate and fit the model on the training data:
```
logreg = LogisticRegression()
logreg.fit(X_train_mm, y_train)
```
The trained model is then used to predict the results of both the training and testing data as well as create a classification report of our model:
```
yhat_train = logreg.predict(X_train_mm)
print(classification_report(y_train, yhat_train))

yhat_test = logreg.predict(X_test_mm)
print(classification_report(y_test, yhat_test))
```

### 5. Model 2 &mdash; Adversarial Neural Net Classifier
The second model we chose to test and create with our pre-processed data is a simple neural network. Besides the neural network’s output layer, the relu activation function is used. The input layer has 16 nodes, the first hidden layer has 8 nodes, and the second hidden layer as 6 nodes. The output layer has 1 node and uses sigmoid function.

We compile the neural network with rmsprop optimizer, binary_crossentropy loss function, and accuracy metrics, and then fit the model with a batch size of 1 and 25 epochs in the [preprocess_logreg_neuralnet notebook](./3_preprocess_logreg_neuralnet.ipynb):
```
classifier = Sequential() # Initialising the ANN
classifier.add(Dense(units = 16, activation = 'relu', input_dim = 1558))
classifier.add(Dense(units = 8, activation = 'relu'))
classifier.add(Dense(units = 6, activation = 'relu'))
classifier.add(Dense(units = 1, activation = 'sigmoid')) # use sigmoid for final classifier unit
classifier.compile(optimizer = 'rmsprop', loss = 'binary_crossentropy', metrics = ["accuracy"])
hist = classifier.fit(X_train_mm.astype(float), y_train, batch_size = 1, epochs = 25, validation_data=(X_test_mm, y_test))
```
The trained model is then used to make predictions on the training and testing data:
```
yhat_train = classifier.predict(X_train_mm.astype(float), verbose=False)
yhat_train_binary = [ 1 if y>=0.5 else 0 for y in yhat_train ]
print(classification_report(y_train, yhat_train_binary))

yhat_test = classifier.predict(X_test_mm.astype(float), verbose=False)
yhat_test_binary = [ 1 if y>=0.5 else 0 for y in yhat_test ]
print(classification_report(y_test, yhat_test_binary))
```

Then, we plot the model’s performance:
```
NN_history = pd.DataFrame(hist.history)
NN_history.rename(
    columns={
        "loss": "train_loss",
        "val_loss": "test_loss",
        "accuracy": "train_accuracy",
        "val_accuracy": "test_accuracy",
        },
    inplace=True,
    )
sns.lineplot(NN_history[["train_loss", "test_loss"]])
sns.lineplot(NN_history[["train_accuracy", "test_accuracy"]])
```

### 6. Model 3 &mdash; Support Vector Machine
The last model we created to classify the data is a support vector machine.

In the [preprocess_logreg_neuralnet notebook](./3_preprocess_logreg_neuralnet.ipynb), we generate the SVM with rbf kernel and fit the model with the training data set:
```
svm = SVC(kernel="rbf", random_state=69420)
svm.fit(X_train_mm,y_train)
```
Then, we predicted the outcome of the testing set with the trained SVM:
```
yhat = svm.predict(X_test_mm)
print(classification_report(y_test, yhat))
```

## Results
...

## Discussion
...

## Conclusion
From our results, we see that we can predict with high accuracy whether an image on a website  with the same independent features is an advertisement or not with the models we created with UC Irvine’s Internet Advertisements dataset. We believe our models could be useful in identifying advertisements in websites for ad blocking or search engine results. Because the Internet undergoes many changes, our models’ accuracy could be much worse when trying to predict on more recent data since the dataset we trained with is from 1998. Therefore, we could train our models with more recent data for further optimization of our classifying models.

## Collaboration

**Giulio:** code for SVM classification, improved Neural Network training curve plots, code for exploratory histogram plot, wrote 1st version of the README

**Sarah:** wrote about scaling, logistic reg model, and neural net model for milestone, wrote about model 1, 2, and 3 in methods, wrote part of conclusion

**Liudmila:**

**Henry:** code discussion, writeup part of introduction

**Rongshan:**

**Sydney:** code to turn unformatted data into useable csv, code to perform scaling and imputing on data and generate pairplot, code for preliminary logistic regression and neural net models and evaluation, wrote “Model Fitting” section of milestone md, wrote data pre-processing and data exploration sections of readme md
