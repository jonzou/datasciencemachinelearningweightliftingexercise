---
title: "Weight Lifting Excercises Classification"
output:
  pdf_document:
    toc: no
  html_document:
    theme: united
    toc: no
    self_contained: true
---
## load and clean data
remove variables with all NAs

```r
library(data.table)
wle <- fread('2.csv')
testing <- fread('4.csv')
wle <- subset(wle, select=-c(1:7))
naall <- apply(wle, 2, function(x) sum(is.na(x)))
cols <- names(which(naall==0))
wle <- subset(wle, select=cols)
wle$classe <- factor(wle$classe)
```

## build model
use randomforest for classification  
use 4 folds crossvalidation for accuracy estimation

```r
library(caret) 
library(kernlab)
library(nnet)
system.time(modelFit <- train(classe~., data=wle, method='rf', trControl=trainControl(method='cv', number=4, repeats=1)))
```

```
## Loading required package: randomForest
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```
##    user  system elapsed 
## 869.153   1.040 870.690
```

```r
modelFit
```

```
## Random Forest 
## 
## 19622 samples
##    52 predictors
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (4 fold) 
## 
## Summary of sample sizes: 14718, 14716, 14717, 14715 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa      Accuracy SD  Kappa SD   
##    2    0.9935275  0.9918119  0.001228911  0.001555504
##   27    0.9937823  0.9921345  0.001111903  0.001407643
##   52    0.9885330  0.9854928  0.002168385  0.002745397
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 27.
```
we can see from model output that the accuracy estimate is about 99%  
## prediction


```r
predictions <- predict(modelFit, newdata=testing)
predictions
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```


## write to submit files

```r
pml_write_files = function(x){
    n = length(x)
    for(i in 1:n){
        filename = paste0("problem_id_",i,".txt")
        write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
    }
}
pml_write_files(predictions)        
```
