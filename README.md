Getting and Cleaning Data: Week 1 Quiz
Bill Roka
March 11, 2016
GETTING AND CLEANING DATA: WEEK 1 QUIZ
QUESTION 1
The American Community Survey distributes downloadable data about United States communities. Download the 2006 microdata survey about housing for the state of Idaho using download.file() from here:

https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv

and load the data into R. The code book, describing the variable names is here:

https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FPUMSDataDict06.pdf

How many properties are worth $1,000,000 or more?

# if (!file.exists("data")) {
#   dir.create("data")
# }
#install.packages("data.table")
library(data.table) 
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06hid.csv"
download.file(fileUrl, destfile = "america_comm_survey.csv")
dateDownloaded <- date()

data <- read.csv("america_comm_survey.csv")
head(data$VAL) # take a look at the contents of the VAL (property worth) variable
## [1] 17 NA 18 19 20 15
DT = data.table(data) # create data.table version of data
DT[, .N, by=VAL==24]  # use built-in ".N" function to find counts. 24 is $1M+ in lookup.
##      VAL    N
## 1: FALSE 4367
## 2:    NA 2076
## 3:  TRUE   53
# ANSWER = 53 AT $1m+ (NUMBER 24)
head(DT$FES, 20)
##  [1]  2 NA  7  1  1  2 NA NA  2 NA  7  2  1 NA NA  7  1 NA  7  1
QUESTION 2
Tidy data one variable per column

QUESTION 3
Download the Excel spreadsheet on Natural Gas Aquisition Program here:

https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FDATA.gov_NGAP.xlsx

Read rows 18-23 and columns 7-15 into R and assign the result to a variable called:

dat

What is the value of:

sum(datZip*datExt,na.rm=T)

library(xlsx)
# if (!file.exists("data")) {
#      dir.create("data")
# }
fileXLS <- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2FDATA.gov_NGAP.xlsx"
# use mode = "wb" forces binary mode - doesn't read correctly without this!
download.file(fileXLS,destfile="NGAP.xlsx", mode = "wb")
dateDownloadedXLS <- date() # if you want to store the date of download
colIndex <- 7:15 
rowIndex <- 18:23
dat <- read.xlsx("NGAP.xlsx",sheetIndex=1, colIndex = colIndex, rowIndex = rowIndex) #select first sheet, specific col/rows.
sum(dat$Zip*dat$Ext,na.rm=T) # code lesson gives your to run
## [1] 36534720
# QUESTION 3 ANSWER#     
# [1] 36534720
QUESTION 4
Read the XML data on Baltimore restaurants from here:

https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Frestaurants.xml

How many restaurants have zipcode 21231?

#install.packages("XML")
library(XML) 
library(RCurl) 
library(dplyr)
fileXML <- "https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Frestaurants.xml"

# had to remove s from https in above xml file. This method commented below:
#   fileXML <- "http://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Frestaurants.xml"
#   doc <- xmlTreeParse(fileXML,useInternal = TRUE)

# using RCurl, can leave https.  Use getURL first, then parse with xmlParse
xData <- getURL(fileXML) # This allows you to use https
doc <- xmlParse(xData)
rootNode <- xmlRoot(doc)
#xmlName(rootNode) # just displays top root node name
# one version, no data frame required - no need for zips, zips_dt 
sum(xpathSApply(rootNode, "//zipcode", xmlValue) == "21231")
## [1] 127
# another version, create data frame and find answer there
zips <- xpathSApply(rootNode,"//zipcode", xmlValue) # getting the zip code data
zips_dt <- data.frame(zips, row.names = NULL) # creating a data frame from them
summary(zips_dt$zips== 21231) # find count of 21231
##    Mode   FALSE    TRUE    NA's 
## logical    1200     127       0
# another method, finds count of True instance of 21231. Uses dplyr.
count(zips_dt, zips == 21231) 
## Source: local data frame [2 x 2]
## 
##   zips == 21231     n
##           (lgl) (int)
## 1         FALSE  1200
## 2          TRUE   127
# QUESTION 4 ANSWER        
  127 
## [1] 127
QUESTION 5
The American Community Survey distributes downloadable data about United States communities. Download the 2006 microdata survey about housing for the state of Idaho using download.file() from here:

https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06pid.csv

Using the fread() command load the data into an R object

DT

The following are ways to calculate the average value of the variable pwgtp15

broken down by sex. Using the data.table package, which will deliver the fastest user time?

mean(DTpwgtp15,by=DTSEX)

mean(DT[DTSEX==1,]pwgtp15);mean(DT[DTSEX==2,]pwgtp15)

tapply(DTpwgtp15,DTSEX,mean)

DT[,mean(pwgtp15),by=SEX]

sapply(split(DTpwgtp15,DTSEX),mean)

rowMeans(DT)[DTSEX==1]; rowMeans(DT)[DT$SEX==2]$

download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fdata%2Fss06pid.csv", destfile = "ACS.csv")
DT <- fread("ACS.csv", sep = ",")

# microbenchmark package allows you to run multiple versions of query "n" amount of times
# the example below runs 100 times
#install.packages("microbenchmark")
library("microbenchmark")

mbm = microbenchmark(
  v3 = sapply(split(DT$pwgtp15,DT$SEX),mean),
  v6 = DT[,mean(pwgtp15),by=SEX],
  v7 = tapply(DT$pwgtp15,DT$SEX,mean),
  v8 = mean(DT$pwgtp15,by=DT$SEX),
  times=100
)
mbm
## Unit: microseconds
##  expr      min        lq       mean    median        uq      max neval
##    v3  566.277  588.3125  635.00895  598.5755  613.3655 1516.510   100
##    v6  632.987  662.8700  703.84950  689.4330  718.2600 1222.506   100
##    v7 1185.378 1213.7520 1315.08976 1226.4295 1265.0665 2319.741   100
##    v8   26.564   30.7895   32.49231   31.9975   33.5060   57.353   100
##   cld
##   b  
##    c 
##     d
##  a
