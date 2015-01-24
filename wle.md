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
Weight Lifting Excercises Classification
==========================================
## load and clean data

clean csv for fread automatic type detection

```shell
sed -e 's/\"//g' -e 's/#DIV\/0!/NA/g' pml-training.csv > 2.csv
sed -e 's/\"//g' -e 's/#DIV\/0!/NA/g' pml-testing.csv > 4.csv
```

remove variables with almost/all NAs   


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
tc <- trainControl(method='cv', number=4, repeats=1)
timecost <- system.time(modelFit <- train(classe~., data=wle, method='rf', trControl=tc))
timecost
```

```
##    user  system elapsed 
## 883.990   0.913 885.412
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
## Summary of sample sizes: 14716, 14717, 14717, 14716 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa      Accuracy SD   Kappa SD    
##    2    0.9944959  0.9930375  0.0006866537  0.0008686555
##   27    0.9946998  0.9932954  0.0003719858  0.0004706008
##   52    0.9885844  0.9855593  0.0033409311  0.0042268038
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
