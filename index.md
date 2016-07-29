---
title       : Presentation for Shiny project
subtitle    : An interface for a machine learning problem
author      : Daxing Gao
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
---

## Introduction

# Goal

I will use variables in iris dataset to predict species information by machine learning. This goal can be achieved by building a shiny interface which allow the user to choose variables and prediction model.

# Ideas

Users can choose at least 2 variables from the 4 variables of iris dataset as well as the a model-random forest or rpart. The server will take the users' input and apply it to machine learning. It will return the selected data and the confusion matrix of prediction.

--- .class #id 

## Shiny -ui.R

```r
shinyUI(
  pageWithSidebar(
    headerPanel("Course Project: Shiny Application--prediction for Iris dataset"),
    sidebarPanel(
      h4('Use the following variables to predict'),
      h4('(Please select 2 or more variables)'),
      p('Sepal.Length,Sepal.Width,Petal.Length, Petal.Width'),
      checkboxGroupInput("checkbox","variables",c('Sepal.Length'='1'
      ,'Sepal.Width'='2','Petal.Length'='3','Petal.Width'='4'),
      selected=c('1','2','3','4')),
        h4('Choose the model you want to fit'),
        p('Randomforest or Rpart'),
      selectInput("Select","Select the model",
      c("Randomforest"='1',"Rpart"='2'),selected='1'),
      submitButton('Go!')),
    mainPanel(p('Selected data'),verbatimTextOutput('dtable'),
      p('Confusion matrix of Prediction'),
      verbatimTextOutput('confusionmatirx'))))
```

--- .class #id 

## Shiny -server.R

```r
a=1:4 #numeric vector
shinyServer(
  function(input,output,session){
     observe({
       y<-as.numeric(input$Select) # selected model, random forest=1, rpart=2
       if(length(input$checkbox)==1){updateCheckboxGroupInput(session,'checkbox',
      selected=c(as.character(a[a!=as.numeric(input$checkbox)][1]),input$checkbox))                           # if only one variable is selected, add one more
         x1<-c(a[a!=as.numeric(input$checkbox)][1],as.numeric(input$checkbox))
       }
       else if(length(input$checkbox)==0){updateCheckboxGroupInput(session,'checkbox',
        selected=c('1','2'));x1<-1:2}                                                                                                     #if no variable is selected, select the first two
         
       else x1<-as.numeric(input$checkbox)
       output$dtable<-renderPrint({df(x1)})                                                                                          #show the top rows of the selected data
       output$confusionmatirx<-renderPrint({fit_iris(x1,y)})                                                                         #fit the model and calculate confusion matrix
       
     })
  })
```

```
## Error in eval(expr, envir, enclos): could not find function "shinyServer"
```

```r
data(iris)#load iris data
###function to fit the data with two classification models-random forest or rpart
fit_iris<-function(x,y){ #x is the selected column number,y is the choice of model
  iris1<-data.frame(cbind(iris[,x],as.character(iris[,5])))
  colnames(iris1)<-c(names(iris)[x],names(iris)[5])
  if(y==1){
    fit<-randomForest(factor(Species)~.,data=iris1)
    cm<-confusionMatrix(predict(fit,iris1),iris[,5])
    }
  if(y==2){
    fit<-rpart(factor(Species)~.,data=iris1)
    cm<-confusionMatrix(predict(fit,iris1,type='class'),iris[,5])
    }
    return(cm) #return confusion matrix
}

###function to select iris data by column number and return the top rows of the data
df<-function(x){ # x is a vector of column numbers
iris1<-data.frame(cbind(iris[,x],as.character(iris[,5])))
colnames(iris1)<-c(names(iris)[x],names(iris)[5])
head(iris1)}
```

--- .class #id 

## Result
Example of results

```
##   Sepal.Length Sepal.Width Petal.Length Petal.Width Species
## 1          5.1         3.5          1.4         0.2  setosa
## 2          4.9         3.0          1.4         0.2  setosa
```

```
## Error in eval(expr, envir, enclos): could not find function "randomForest"
```

```
## Error in eval(expr, envir, enclos): could not find function "confusionMatrix"
```

