---
output: html_document
---
structures


Certain strains of the bacterium _Escherichia coli_ often found in undercooked foods become a serious health risk if they enter the blood stream. The organism is covered with a chemical compound called a lipopolysaccharide (LPS) that has a toxic effect on the hearts of infected animals. When LPS enters the circulatory system, heart function is affected and heart rate becomes highly elevated. A medical scientist wants to know if the residual effect on heart rate is different for LPS than for other compounds also known to increase heart rate.

An experiment is designed to see how heart rate decreases over time after it has been elevated either with LPS or another compound that will serve as a control. LPS is used on 3 rats and the control compound on another 3. A monitor records continuous measurements (one per second) of the rats’ heart rates, but the measures to be used in the analysis are when each rat’s heart rate reaches a maximum and every 20 minutes thereafter. The experimenter wants to compare the effect of the two compounds on heart rate during the hour after it has reached the maximum number of beats per minute.

1 Read the coli.csv file into R.
```{r}

coli<-read.csv("coli.csv") #reading .csv file.

head(coli) #displaying only first 6 rows.
    ```

2 Using the coli data frame, create a table of means where rows represent treatment and columns represent time.
```{r}
## Table of means.

tapply( X = coli$hrate, INDEX = list(coli$treatment,coli$time), FUN = mean )

#time in column with"0", "20", "40" and "60" minutes representing different levels.
#treatment in rows
```

3 Plot the interaction plot, i.e., time is on the x-axis and two segmented lines are drawn---one for treatment level.
```{r}
## On the x-axis time variable is represented, while on y-axis there is hrate to display the differences in two treatment level-LPS and Control.

interaction.plot(coli$time,coli$treatment,coli$hrate,type="l",xlab="time",ylab="hrate",trace.label="treatment", fun=mean)

#An interaction plot is a visual representation of the interaction between the effects of two factors, or between a factor and a numeric variable. It is suitable for experimental data.
#It plots the mean (or other summary) of the response for two-way combinations of factors, thereby illustrating possible interactions.
```

Here, the interaction plot is basically the visual representation of the "Mean table" obtained in Q2 for two levels of treatment with respect to time, that is at time 0, 20, 40, and 60 minues when the heart rate of rats was monitored for peak value and for every 20 minute. "Control" level treatment has higher hrate for the rats at the start fo the experiment in comparison to the LPS level, however decreases sharply then after against its counterpart treatment-Control. We can state an assumption that LPS seem to have an effect on the heart rate and that it decreases over time.

4 Convert the data frame to wide format.
```{r}
## Put R code here.

#wideformat 

library(reshape2)

data.wide <- dcast(coli, treatment + rat ~ time, value.var="hrate") #using dcast function
data.wide

```

5 Compute the mean for each treatment level by time from the wide-format data frame.
```{r}
##by time, mean for each treatment level

colnames(data.wide) <- c("treatment","rat","time1","time2","time3","time4")  
#to improve the readablility

data.wide

# treatment mean for each time level individually (optional)

time0<-aggregate(time1~treatment, data.wide, mean) #time0 mean
time0

time20<-aggregate(time2~treatment, data.wide, mean) #time20 mean
time20

time40<-aggregate(time3~treatment, data.wide, mean) #time40 mean
time40

time60<-aggregate(time4~treatment, data.wide, mean) #time60 mean
time60

# mean for each treatment level by time in a single dataframe

data.wide$rat=NULL
time.frame<-aggregate(.~treatment, data.wide, mean)
time.frame
```

EJH: This can be done in one line using `aggregate`.


6 Subset the wide-format data frame into two data frames---one for each treatment level.
```{r}
##Subsetting dataframe into two splits.

#lps level dataframe

lps<-subset(data.wide[,],treatment=="LPS")
lps

#control level dataframe

control<-subset(data.wide[,],treatment=="Control")
control
```

7 Compute the mean for each data frame by time.
```{r}
## Mean for each data frame by time.

# mean for control data frame

controlmean<-aggregate(.~treatment, control, mean) 
#aggregate-similar to by, but instead of printing the output, aggregate sticks everything into a dataframe, also calls mean function on each column independently, which is why you get independent means.
controlmean

# mean for lps data frame by time.

lpsmean<-aggregate(.~treatment, lps, mean)
lpsmean

```

8 Reconstruct the wide-format data frame from the individual treatment data frames.
```{r}
## Recontructing the wide format again.

#rbind() function combines vector, matrix or data frame by rows.The column of the two datasets must be same, otherwise the combination will be meaningless

re.data.wide<-rbind(control, lps)
as.data.frame(re.data.wide) #to check if the object is a data frame, or coerce it if possible.

```
