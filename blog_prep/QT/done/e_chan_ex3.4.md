---
layout: post
title: "E Chan Quant Trading Ex. 3.4"
date: 2018-03-15
tags: Exercises, Sharpe Ratio, Long-Only
---

Example 3.4: Calculating Sharpe Ratio for Long-Only Versus
----------------------------------------------------------

This example calculate the Sharpe ratio of a trivial long-only strategy for IGE: buying and holding a share since the close of November 26, 2001, and selling it at close of November 14, 2007.

Assume the average risk-free rate during this period is 4 percent per annum in this example. The following example uses the data from the book but you can download the daily prices from Yahoo! Finance which you can call IGE.xls.

``` r
# make sure previously defined variables are erased.
rm(list = ls())
```

``` r
# read a spreadsheet named "IGE.csv" into R. 
ige_dat <- read.csv(file.path(getwd(), 
                               "data", 
                               "IGE.csv"))
head(ige_dat)                      

# the first column 
# contains the trading days in format yyyy-mm-dd.
tday <- ige_dat$Date

# convert the format into yyyymmdd.
tday <- format(as.Date(tday, "%d/%m/%Y"), "%Y%m%d")
head(tday)

# convert the date strings first into cell arrays and
# then into numeric format.
tday <- as.numeric(as.character(tday))
head(tday)

# the last column contains the adjusted close prices.
cls <- ige_dat$Adj.Close
head(cls)

data <- data.frame(tday = tday,
                   cls = cls)
head(data)
# sort tday into ascending order and cls along with it
data <- data[order(tday),]
head(data)

end <- length(data$cls)
end
# 1504

# daily returns
dailyret <- (data$cls[2:end]-data$cls[1:end-1])/data$cls[1:end-1]

# excess daily returns assuming risk-free rate of 4#
# per annum and 252 trading days in a year
excessRet <- dailyret - 0.04/252

# the output should be 0.7893
sharpeRatio <- sqrt(252)*mean(excessRet)/sd(excessRet)
sharpeRatio
# 0.7893175
```

``` r
# This is a continuation of the above code.

# Read in the SPY data
spy_dat <- read.csv(file.path(getwd(), "data", "SPY.csv"))
head(spy_dat)      

# the first column contains the trading days in format yyyy-mm-dd.
tdaySPY <- spy_dat$Date

# convert the format into yyyymmdd.
tdaySPY <- format(as.Date(tdaySPY, "%d/%m/%Y"), "%Y%m%d")
head(tdaySPY)

# convert the date strings first into cell arrays and
# then into numeric format.
tdaySPY <- as.numeric(as.character(tdaySPY))
head(tdaySPY)

# the last column contains the adjusted close prices.
clsSPY <- spy_dat$Adj.Close
head(clsSPY)

dataSPY <- data.frame(tday = tdaySPY,
                      cls = clsSPY)
head(dataSPY)
# sort tday into ascending order and cls along with it
dataSPY <- dataSPY[order(tdaySPY),]
head(dataSPY)

end <- length(dataSPY$cls)
end
# 1504

# daily returns
dailyretSPY <- (dataSPY$cls[2:end]-dataSPY$cls[1:end-1])/dataSPY$cls[1:end-1]

# excess daily returns assuming risk-free rate of 4#
# per annum and 252 trading days in a year
excessRetSPY <- dailyretSPY - 0.04/252

# net daily returns
# (divide by 2 because we now have twice as much capital.)
netRet <- (dailyret - dailyretSPY)/2

# the output should be 0.783681
sharpeRatioSPY <- sqrt(252)*mean(netRet)/sd(netRet)
sharpeRatioSPY
# 0.783681
```
