Calculating the Optimal Allocation Using Kelly Formula
------------------------------------------------------

We pick three sector-specific ETFs and see how we should allocate capital among them to achieve the maximum growth rate for the portfolio.

The three ETFs are: OIH (oil service), RKH (regional bank), and RTH (retail). The daily prices are downloaded from Yahoo! Finance and saved in epchan.com/book as OIH.xls, RKH.xls, and RTH.xls.

This file calculates M, C, and F\* (F\* = C^-1\*M), where: M is the column vector of mean returns for the strategies; C is the covariance matrix such that Cij is the covariance of returns of the ith and jth strategies; F is the kelly optimal leverages

This is taken from exercise 6.3 of E.Chans book Quantitative Trading.

``` r
rm(list = ls()) # clear the workspace
library(dplyr)
library(echanFuncs)
```

``` r
# OIH data
oih_in <- read.csv(file.path("data", "OIH.csv"))

# the first column (starting from the second row) is the trading days in
# format mm-dd-yyyy.
tday1 <- oih_in$Date

# convert the format into yyyymmdd.
tday1 <- format(as.Date(tday1, "%d-%m-%Y"), "%Y%m%d")

# convert the date strings first into cell arrays and then into numeric format.
tday1 <- as.numeric(as.character(tday1))

# the last column contains the adjusted close prices.
adjcls1 <- oih_in$Adj.Close

# RKH data 
rkh_in <- read.csv(file.path("data", "RKH.csv"))

# the first column (starting from the second row) is the trading days in
# format mm/dd/yyyy.
tday2 <- rkh_in$Date

# convert the format into yyyymmdd.
tday2 <- format(as.Date(tday2, "%d-%m-%Y"), "%Y%m%d")

# convert the date strings first into cell arrays and then into numeric format.
tday2 <- as.numeric(as.character(tday2)) 

# the last column contains the adjusted close prices.
adjcls2 <- rkh_in$Adj.Close

# RTH data
rth_in <- read.csv(file.path("data", "RTH.csv"))

# the first column (starting from the second row) is the trading days in
# format mm/dd/yyyy.
tday3 <- rth_in$Date

# convert the format into yyyymmdd.
tday3 <- format(as.Date(tday3, "%d-%m-%Y"), "%Y%m%d")

# convert the date strings first into cell arrays and then into numeric format.
tday3 <- as.numeric(as.character(tday3)) 

# the last column contains the adjusted close prices.
adjcls3 <- rth_in$Adj.Close
```

``` r
# merge these data - tday is the same as octave: tested
tday <- union(tday1, tday2)
tday <- union(tday, tday3)
tday <- sort(tday) # this brings it in line with echan

adjcls <- matrix(NaN, length(tday), 3)

#========================================
# tday1
foo <- dplyr::intersect(tday1, tday)

# idx1 is the indices of tday1
idx1 <- match(foo, tday1)

# idx is the indices of tday
idx <- match(foo, tday)

adjcls[idx, 1] <- adjcls1[idx1]

#========================================
# tday2
foo <- dplyr::intersect(tday2, tday)

# idx1 is the indices of tday2
idx2 <- match(foo, tday2)

# idx is the indices of tday
idx <- match(foo, tday)

adjcls[idx, 2] <- adjcls2[idx2]

#========================================
# tday3
foo <- dplyr::intersect(tday3, tday)

# idx1 is the indices of tday3
idx3 <- match(foo, tday3)

# idx is the indices of tday
idx <- match(foo, tday)

adjcls[idx, 3] <- adjcls3[idx3]
```

``` r
# returns
# ret=(adjcls-lag1(adjcls))./lag1(adjcls); 
ret <- (adjcls - lag1(adjcls)) / lag1(adjcls)
```

    ## [1] "is numeric"
    ## [1] "is numeric"

``` r
# days where any one return is
# baddata=find(any(~isfinite(ret), 2));  missing
baddata <- which(rowSums(is.na(ret))>0)

# eliminate days where any one return is missing
# ret(baddata, :)=[]; 
ret <- ret[-baddata,]

library(matlab)
```

    ## 
    ## Attaching package: 'matlab'

    ## The following object is masked from 'package:stats':
    ## 
    ##     reshape

    ## The following objects are masked from 'package:utils':
    ## 
    ##     find, fix

    ## The following object is masked from 'package:base':
    ## 
    ##     sum

``` r
# excess returns: assume annualized risk free rate is 4%
excessRet=ret-repmat(0.04/252, size(ret)); 

# annualized mean excess returns
# Should equal: 0.139568072  0.029400294 -0.007346459
M <- 252*colMeans(excessRet, 1)
M
```

    ## [1]  0.139568072  0.029400294 -0.007346459

``` r
# annualized covariance matrix
# Should equal:
# [1,] 0.11090093 0.02001371 0.01825486
# [2,] 0.02001371 0.03716455 0.02689284
# [3,] 0.01825486 0.02689284 0.04196684

C <- 252*cov(excessRet)
C
```

    ##            [,1]       [,2]       [,3]
    ## [1,] 0.11090093 0.02001371 0.01825486
    ## [2,] 0.02001371 0.03716455 0.02689284
    ## [3,] 0.01825486 0.02689284 0.04196684

``` r
library(matlib)
# Kelly optimal leverages
# Should equal:
# 1.291908  1.172265 -1.488213
F <- colSums(inv(C)*M)
F
```

    ## [1]  1.291908  1.172265 -1.488213

``` r
# Maximum annualized compounded growth rate
# Should equal:
# g = 0.1529
g <- 0.04 + F%*%C%*%F/2
g
```

    ##           [,1]
    ## [1,] 0.1528536

``` r
# Sharpe ratio of portfolio
# Should equal:
# S = 0.4751
S <- sqrt(F%*%C%*%F)
S
```

    ##           [,1]
    ## [1,] 0.4750865
