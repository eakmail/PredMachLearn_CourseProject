# Prediction on Human Activity Dataset
Evgeny Kuznetsov  

## Synopsis

This is Practical Machine Learning Course Project. In this project, we predict performing of Unilateral Dumbbell Biceps Curl in five different fashions: exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E).

Read more: http://groupware.les.inf.puc-rio.br/har#ixzz3RLF7zGmr

## Data 


The training data for this project are available here: 

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here: 

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv


## Load and cleanup

We have unrecognizable "#DIV/0!" strings in numerical data, which we count as NA:


```r
pml_training <- read.csv("pml-training.csv", na.strings=c("NA", "#DIV/0!"))
pml_testing <- read.csv("pml-testing.csv", na.strings=c("NA", "#DIV/0!"))
```

Clean out first 7 service information columns and columns with NA:


```r
useful <- (colSums(is.na(pml_training)) == 0)  &  c(rep(FALSE, 7), rep(TRUE, ncol(pml_training) - 7))

pml_useful <- pml_training[,useful]
final_test_useful <- pml_testing[,useful]
```

## Partition data

Make data partitions to cross-validate, 60% train data and 40% test data:


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
suppressWarnings(library(randomForest))
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
set.seed(11)

inTrain <- createDataPartition(y=pml_useful$classe, p = 0.6, list= FALSE)
training <- pml_useful[inTrain,]
testing <- pml_useful[-inTrain,]
```

## Train and estimate model perfomance

Use Breiman's random forest algorithm:


```r
fit <- randomForest(classe ~ . , data = training)

fit
```

```
## 
## Call:
##  randomForest(formula = classe ~ ., data = training) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 7
## 
##         OOB estimate of  error rate: 0.6%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3340    7    0    1    0 0.002389486
## B   10 2265    4    0    0 0.006143045
## C    0   18 2031    5    0 0.011197663
## D    0    0   18 1911    1 0.009844560
## E    0    0    2    5 2158 0.003233256
```

OOB (out of bag) estimate is 0.6% error rate. 

## Cross-validataion

Let's check error rate on testing set:


```r
mean(predict(fit, testing) == testing$classe)
```

```
## [1] 0.9928626
```

Our error rate is 0.71%. Which is pretty close to 0.6% estimation.



```r
confusionMatrix(predict(fit, testing), testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2231    6    0    0    0
##          B    1 1512   15    0    0
##          C    0    0 1351   17    3
##          D    0    0    2 1268   11
##          E    0    0    0    1 1428
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9929          
##                  95% CI : (0.9907, 0.9946)
##     No Information Rate : 0.2845          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.991           
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9996   0.9960   0.9876   0.9860   0.9903
## Specificity            0.9989   0.9975   0.9969   0.9980   0.9998
## Pos Pred Value         0.9973   0.9895   0.9854   0.9899   0.9993
## Neg Pred Value         0.9998   0.9991   0.9974   0.9973   0.9978
## Prevalence             0.2845   0.1935   0.1744   0.1639   0.1838
## Detection Rate         0.2843   0.1927   0.1722   0.1616   0.1820
## Detection Prevalence   0.2851   0.1947   0.1747   0.1633   0.1821
## Balanced Accuracy      0.9992   0.9968   0.9922   0.9920   0.9951
```

Kappa value is 0.991.

## Final results


```r
final_results <- predict(fit, final_test_useful)
final_results
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```

## Appendix A

Create files for submissions based on provided by instructor code:


```r
pml_write_files = function(x){
  n = length(x)
  for(i in 1:n){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}
pml_write_files(final_results)
```
