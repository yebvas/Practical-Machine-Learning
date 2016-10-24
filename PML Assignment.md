---
output: html_document
---
#Practiacal Machine Learning Prediction Assignment

##Synopsis
This assignment is to build a prediction model to apply in the prediction quiz. Two models will be built, a Decision Tree model and a Random Forest model, and evaluated. The model with the better accuracy in predicting on a validation set will be selected.

###Load R Packeges
```{r}
library(caret)
library(randomForest)
library(rpart)
library(rattle)
```

###Load the Data
```{r}
Training.Data <- read.csv("pml-training.csv")
Testing.Data <- read.csv("pml-testing.csv")
```

###Clean the Data
The data set is very large and contains many variables that are not useful as predictors. The first five variables consist of a row number (X) the users names and three time stamps. These are excluded because they do not relate to any of the measurements that are used to make the prediction.
```{r}
Training.Data <- Training.Data[,-(1:5)]
Testing.Data <- Testing.Data[,-(1:5)]
```

Next, variables that have very little variance are removed. Near zero variance variables use up a lot of computational time and offer little additional predictive power.
```{r}
Zero.Var <- nearZeroVar(Training.Data)
Training.Data <- Training.Data[,-Zero.Var]
Testing.Data <- Testing.Data[,-Zero.Var]
```

Finally, there are many variables that contain a lot of NA values. All variables that contain more than five NAs are removed.
```{r}
NAs <- colSums(is.na(Testing.Data))>5
Training.Data <- Training.Data[, NAs == FALSE]
Testing.Data <- Testing.Data[, NAs == FALSE]
```

###Partition the Training Data into a Training and a Validation Set
The Training.Data data set is split into a Training data set and a Validation data set to perform cross-validation on after the models have been built.
```{r}
set.seed(1316)
inTrain <- createDataPartition(Training.Data$classe, p = 0.6, list = FALSE)
Training <- Training.Data[inTrain,]
Validation <- Training.Data[-inTrain,]
```

##Training the Models

###Random Forests Model
The first model built is a random forest model. Also shown are the 20 most important variables in the random forest model.
```{r}
set.seed(1316)
model.RF <- train(classe ~ ., data = Training, method = "rf", trControl=trainControl(method = "cv", number = 9), na.action = na.omit)
model.RF
varImp(model.RF)
```

###rpart Model
The second model built is a decision tree model. Also shown is a plot of the decision tree.
```{r}
set.seed(1316)
model.rpart <- train(classe ~ . , data = Training, method = "rpart", na.action = na.omit)
model.rpart
fancyRpartPlot(model.rpart$finalModel)
```

###Cross-validation: Testing the Models on the Validation Set.
Both models are tested against the Validation data set to see which performs better.

####Random Forest Model
```{r}
Validate.RF <- predict(model.RF, Validation)
confusionMatrix(Validation$classe, Validate.RF)
```

The random forest model was 99.8% accurate in the cross-validation test. This means it had an out of samle error rate of 0.2%

####Decision Tree Model
```{r}
Validate.Rpart <- predict(model.rpart, Validation)
confusionMatrix(Validation$classe, Validate.Rpart)
```

The decision tree model was 49.3% accurate in the cross-validation test. This means it had an out of samle error rate of 50.7%

##Model Selection
Based on the outcome of the validation tests, the Random Forests prediction model as it was 99.8% accurate in the validation test compared to the Decision Tree model at 49.3%

##Predicting on the Test data

Use the random forest model to predict the exercise type based on the variables in the Testing.Data data set.
```{r}
Testing.RF <- predict(model.RF, Testing.Data)
matrix(Testing.RF, 20, 1)
```




