```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

---
title: "FBI CrimesDatabaseanalysis (Chicago Area)"
output: html_document
---


## Libraries Used for Analysis

```{r}
##install.packages("datasets")
##install.packages("highcharter")
##install.packages("ggplot2")
##install.packages("dplyr")
##install.packages("tidyr")
##install.packages("viridis")
##install.packages("plotly")
##install.packages("lubridate")
##install.packages("xts")
##install.packages("ggmap")
##install.packages("gridExtra")
options(warn = -1)
library(datasets)
library(highcharter)
library(ggplot2)
library(dplyr)
library(tidyr)
library(viridis)
library(plotly)
library(lubridate)
library(xts)
library(maps)
library(ggmap)
library(gridExtra)

crime<-read.csv("D:/DataMiningProject/Crimes_-_2001_to_present.csv")
(names(crime))
nrow(crime)
#The dataset chosen for this project consists of incidents of crime reported in the city of Chicago from 2001 to 2022.
#DataSet Cleaning
## Filtering Data by Year Variable

#By the data cleaning our data analysis is become easier.
#a)The data set is reduced to year '2007-to-2022' as it was too large with 7473529 reports.
```{r}
#Eliminating observations for year 2001-2006

crime<-crime[crime$Year!="2001",]
crime<-crime[crime$Year!="2002",]
crime<-crime[crime$Year!="2003",]
crime<-crime[crime$Year!="2004",]
crime<-crime[crime$Year!="2005",]
crime<-crime[crime$Year!="2006",]
#backup of dataset
crimeChicago<-crime

##Spliting Date Variables to Day,Month,Year,Weekday: As we would require all the sub categories of date to analyze our data we will split date data into day, month, year and weekday of that particular date which will make our analysis way easier and observatory.
```{r}
crimeChicago$Day<-factor(day(as.POSIXlt(crimeChicago$Date, format="%m/%d/%Y %I:%M:%S %p")))
crimeChicago$Month <- factor(month(as.POSIXlt(crimeChicago$Date, format="%m/%d/%Y %I:%M:%S %p"), label = TRUE))
crimeChicago$Year <- factor(year(as.POSIXlt(crimeChicago$Date, format="%m/%d/%Y %I:%M:%S %p")))
crimeChicago$Weekday <- factor(wday(as.POSIXlt(crimeChicago$Date, format="%m/%d/%Y %I:%M:%S %p"), label = TRUE))
crimeChicago$Date <- as.Date(crimeChicago$Date, "%m/%d/%Y %I:%M:%S %p")

##Details about variables
```{r}
summary(crimeChicago)
#Converting to categorical variables
crimeChicago$Primary.Type<-as.factor(crimeChicago$Primary.Type)
crimeChicago$Arrest<-as.factor(crimeChicago$Arrest)
crimeChicago$Month<-as.factor(crimeChicago$Month)
crimeChicago$Weekday<-as.factor(crimeChicago$Weekday)
crimeChicago$Year<-as.factor(crimeChicago$Year)
crimeChicago$Beat<-as.factor(crimeChicago$Beat)
crimeChicago$District<-as.factor(crimeChicago$District)
#Removing Whitespaces
crimeChicago<-crimeChicago[crimeChicago$Primary.Type!="",]
crimeChicago<-crimeChicago[crimeChicago$Location.Description!="",]
crimeChicago<-crimeChicago[crimeChicago$Arrest!="",]
crimeChicago<-crimeChicago[crimeChicago$Beat!="",]
crimeChicago<-crimeChicago[crimeChicago$Weekday!="",]
crimeChicago<-crimeChicago[crimeChicago$Month!="",]
crimeChicago<-crimeChicago[crimeChicago$District!="",]
#Removing missing values
sum(is.na(crimeChicago))
crimeChicago<-na.omit(crimeChicago)
```
## Working with 'Non-Criminal' category in Primary.Type variable
```{r}
unique(crimeChicago$Primary.Type)          


#'Non-criminal' crime type which has same meaning were labeled slightly differently. So converting different format into one. 
```{r}
crimeChicago$Primary.Type <- gsub("(.*)NON(.*)CRIMINAL(.*)","NON-CRIMINAL",crimeChicago$Primary.Type)

#Now we are left with just one ''non-criminal'' crime type

##Removing duplicate observations:
```{r}
crimeChicago<-crimeChicago[!duplicated(crimeChicago$Case.Number,crimeChicago$Date),]
write.table(crimeChicago,"D:/DataMiningProject/MJIBRANEP20101036/final_crime_dataset.csv",sep="\t")

##Investigational Analysis:
Data investigational helps to predict the course of crimes and how it is prevent on the same side the data give the knowledge about completed task. This analysis can be a base for building a predictive model.
This graph represent the crime statistics for Chicago taking into account and arrest rate of the number of crimes over the period of the year.                                                                                                                                                             Over all summary describe to give a better helped off the crimes occurrence and arrest that happened throughout the year "2007-to-2022".                                                                                                                                                   ##This graph is clearly show that the arrest rates and the crime rate are diminish over the year and definitely the data was key in this reduction.


###Crime and Arrest rate by year
```{r}
d <- data.frame(Year = crimeChicago$Year, Arrest = crimeChicago$Arrest) 
ggplot(d, aes(Year,fill = factor(Arrest))) +stat_count(width = 0.5)

#Above Figure represent the crime statistics for Chicago taking into account the number of crime and arrest rate over the period of year. This helped to give a better overall summary of the crimes occurrence and arrests that happened throughout the year. It is clear from the graph that the arrest rates and the crime rates are decreasing over year.

### Crime and Arrest by Month.
```{r}
b <- data.frame(Month = crimeChicago$Month, Arrest = crimeChicago$Arrest) 
ggplot(b, aes(Month,fill = factor(Arrest))) +stat_count(width = 0.5)

#The crime rate is decreasing over years. Along this overall decrease,a periodic pattern can be observed every year where the
#crime rate is higher in summer compared to the rest of the
#year

### Crime and Arrest by Weekday
The below visualisations depicts the number of crime occurences and arrests on a particular day of week.
```{r}
crime_by_day<-crimeChicago%>%group_by(Weekday) %>% summarise(Total=n())
hchart(crime_by_day,"column",hcaes(Weekday,Total))%>% hc_title(text="Crime by Day")
arrest_by_day<-crimeChicago[crimeChicago$Arrest=="true",]
arrest_by_day <- arrest_by_day %>% group_by(Weekday) %>% summarise(Total=n())
hchart(arrest_by_day,"column",hcaes(Weekday,Total))%>% hc_title(text="Arrest by Day")
```
Figure 'Crime by Day' shows the number of crime occurrence over the weekdays in Chicago. Where Friday has the maximum number of crimes occurrence and Sunday has the least number of crimes occurrence.

Figure 'Arrest by Day' shows the number of arrest of crimes over the weekdays in Chicago. Where Friday has the maximum number of crimes arrest and Sunday has the least number of crimes arrest. 

Overall, there are some similarities between the occurrence of crime and arrest rate in two figures.


###Crime and Arrest by Primary Type of Crime.

```{r}
crime_by_type<-crimeChicago %>% group_by(Primary.Type) %>% summarise(Total=n())
crime_by_type<-crime_by_type[order(-crime_by_type$Total),]
top_crime_type<-crime_by_type[1:10,]
hchart(top_crime_type, "column", hcaes(Primary.Type, Total))%>% hc_title(text="Crime by Type")
arrest_type<-crimeChicago[crimeChicago$Arrest=="true",]
arrest_by_type<-arrest_type %>% group_by(Primary.Type)  %>% summarise(Total =n())
arrest_by_type<-arrest_by_type[order(-arrest_by_type$Total),]
top_arrest_type<-arrest_by_type[1:10,]
hchart(top_arrest_type,"column",hcaes(Primary.Type,Total))%>% hc_title(text="Arrest by Type")
```


Above Figures provide a statistical comparison between types of crime committed and arrest rate of the crime. We chose to limit the comparison to top 10 crimes and arrests in order to get a fair comparison. While most of the crime occurrence belong to Theft crime type we graphed over the arrest rate and found that the arrests rate are more for narcotics crime type among total number of crimes. It helps to better understand and visualize the numbers of crime committed and compare the same with the arrest rate for that particular crime.

###Crime by Location 

```{r}
crime_by_location<-crimeChicago %>% group_by(Location.Description) %>% summarise(Total=n())
crime_by_location<-crime_by_location[order(-crime_by_location$Total),]
top_crime_by_order<-crime_by_location[1:15,]
hchart(top_crime_by_order,"column",hcaes(Location.Description,Total))%>% hc_title(text="Crime by Location")
```

This Figure shows the number of crime occurrences over different locations in Chicago. With many locations, we chose to display top fifteen different areas. Street having the most crime rate among the 158 locations while Department store has the minimum crime rate in top 15 areas of the total crimes. Overall, the street seems to be the most dangerous area.


##Prediction Model
We identified predictive tasks based on the information available in the data set.
Predicting whether an arrest will be made for a committed crime.

#Taking a random sample for our prediction model
```{r}
crimeChicago<-data.frame(crimeChicago)
index<-sample(1:nrow(crimeChicago),25000)
Model<-crimeChicago[index,]
```

#Considering useful variables for our model
```{r}
preditive_sample<-subset(Model, select = c('Primary.Type','Location.Description','Arrest','Beat','Weekday','Month','District','Year'))
```

Created a sample dataset by taking under consideration all the useful variables required for our analysis purpose, by which we have reduce the variable count and optimized the process.

#Classifying data on label basis and creating dummy variables
```{r}
label_arrest<-preditive_sample$Arrest
#Creating dummies for categorical variables
DummyVariables<-model.matrix(Arrest~.-1,data = preditive_sample)
```
In situations where we have categorical variables (factors) but need to use them in analytical methods that require numbers (for example, K nearest neighbors (KNN), Linear Regression), we need to create dummy variables.

##Splitting dataset into testion and training dataset
```{r}
set.seed(1234)
indicies=sample(1:2, length(preditive_sample$Arrest),replace= T, prob = c(.8,.2))
dim(DummyVariables[indicies==1,]);dim(DummyVariables)
```
Splitting data in the proportion of 80%(training dataset) and 20%(testing dataset).

```{r}
training.input=DummyVariables[indicies==1,]
testing.input=DummyVariables[indicies==2,]
training.label=label_arrest[indicies==1]
testing.label=label_arrest[indicies==2]
```

#KNN Model:
```{r}
#install.packages('class')
require(class)
predictions<-knn(train=training.input, test=testing.input, cl=training.label, k=1)
head(data.frame(predictions,testing.label))
```

```{r}
#Accuracy Rate
sum(predictions==testing.label)/length(predictions)
#Confusion Matrix
table(predictions, testing.label)
```
Accuracy rate is the proportional value by which the model has predicted the dependent variable for the test dataset.

In order to have an idea about the accuracy of the predictions with the training labels.


