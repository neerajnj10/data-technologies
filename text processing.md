---
output: html_document
---

Text processing

Consider the `pointnemotemp.xml` file.

1. Read the `pointnemotemp.xml` into R.
```{r}
## read xml file.

nemo<-readLines("pointnemotemp.xml") #to read xml as text.
head(nemo)
```

2. Use regular expressions to create a data frame with `date` and `temperature` as variables.
```{r}
## Regular expressions.
#extracting date and temperature variable

nchar(nemo)
dateLines <- grep("date..[0-9]..........", nemo)
dateLines
nemo[dateLines]
rep_nemo<-(gsub("^.*(date..[0-9]..........).*$", "\\1", nemo[dateLines])) #search and replace pattern.
rep_nemo
dates<-gsub("^date..", "", rep_nemo)
dates  #date variable created

##extracting temperature variable from the text in the same way as we extracted "date" variable since temperature and date variables are in the same line.

tempLines <- grep("temperature..[0-9]....", nemo)  
tempLines
nemo[tempLines]

#search and replace command for pattern matching.
repo_nemo<-gsub("^.*(temperature..[0-9]....).*$", "\\1", nemo[tempLines]) 
repo_nemo
temp1<-gsub("^temperature..", "", repo_nemo)
temp2<-gsub("280..","280",temp1)  #280 is not in a correct form, has extended characters attached, replacing with correct form
temp2
temp<-gsub("275..","275",temp2)  #275 doesn't have correct form too, replacing and correcting.

temp #temperature variable created.

#create a data frame.
dt<-data.frame(date=dates, temperature=temp)
head(dt)
```

3. Convert date to a real date and plot temperature vs. date. Label axes appropriately.
```{r}
## Date to real date.

d<-dt[,c("date")] #selecting date column.

date1<-as.Date(d, "%d-%B-%Y") # real date format created
date1

## plot temperature vs date.

p<-plot(date1, temp, xlab="date", ylab="temperature", type='l', col="red")
```

