---
layout: post
title: "R Installation"
categories: [r]
notfull: 1
tags: ['101', installation, r]
keywords: "r jupyter notebook programming language install 101 windows linux ubuntu extension package lib library requirement LC_ALL utf8 UTF-8"
---

{% assign img-url = '/img/post/r' %}

{% include toc.html %}

## Install R

đ [Home page](https://cran.r-project.org/).

## Install a package in R

<div class="flex-50" markdown="1">
``` r
# directly
install.packages("slidify")
```

``` r
# from github repo
install.packages("devtools")
devtools::install_github("twitter/AnomalyDetection")
library(AnomalyDetection)
```
</div>

## R with jupyter notebook

đ Read [this note](/jupyter-notebook#r-with-jupyter-notebook).

## Error

``` bash
# installation of package âxml2â had non-zero exit status
sudo apt install libxml2-dev
```

``` bash
# cannot install packages
# Setting LC_CTYPE failed, using "C"
# (or something like that)

# without root environnement
# exit R
defaults write org.R-project.R force.LANG en_US.UTF-8
# start R again

# with root (in a docker bash, for example)
# start R by using
LC_ALL=C.UTF-8 R
```