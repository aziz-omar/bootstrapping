# Resampling in R
Justin Sulik  
September 9, 2017  



# Basic tools for resampling

These are a few basic scripts for generating some random data and then resampling it. This is just to familiarize you with the syntax for generating random data sets, and randomly resampling from the data. It doesn't explain *why* we're doing those things - that's what the workshop is for. Eventually, we will use the `boot` package to do our resampling for us, but the following lets you see what's actually happening. 

To be able to interact with these scripts, open the `.Rmd` file with R studio. You may need to install `knitr`. Otherwise you can just view the `.md` file in github in your browser, and copy+paste the scripts to your R console.

### Load packages

If you don't have any of the following packages installed, first install them with `install.packages("packageName")`. 


```r
library(tidyverse)
library(boot)
```

### Generate random data

`rnorm(N,mean,sd)` draws N samples from the normal distribution with given mean and standard deviation. We will also be using other distributions, such as `rbeta` or `rpois`. 


```r
distribution1 <- rnorm(20, 3, 2)
distribution2 <- rnorm(20, 3, 2)
distribution3 <- rnorm(20, 4, 3)

mean(distribution1)
```

```
## [1] 3.415862
```

```r
sd(distribution1)
```

```
## [1] 1.979968
```

```r
mean(distribution2)
```

```
## [1] 2.995942
```

```r
sd(distribution2)
```

```
## [1] 1.632085
```

```r
mean(distribution3)
```

```
## [1] 4.66168
```

```r
sd(distribution3)
```

```
## [1] 3.337688
```

We want to learn more about these distributions. Since we created them, we know that `distribution1` and `distribution2` are sampled from the same distribution, and that `distribution3` isn't. Rerun the above code a few times to see how they change (if you're viewing the `.Rmd` file, place the cursor in the script window and hit shift+cmd+enter, or press the play button in the menu bar. Otherwise copy the scripts to your console to run them). It's quite possible that, occasionally, the mean for `distribution3` will be lower than one of the others. 

### Resampling 

To understand the distributions better, we resample repeatedly, with replacement. 

One resampling looks like the following, where `sample(vector, N, replace)` resamples N times from the `vector`. By default, `replace=FALSE`, but we need to use `replace=TRUE` for bootstrapping. 


```r
newSample <- sample(distribution1,length(distribution1),replace=TRUE)
mean(newSample)
```

```
## [1] 3.82195
```

```r
sd(newSample)
```

```
## [1] 2.574937
```

Note that the `newSample` mean and standard deviation are unlikely to be exactly the same as for `distribution1`. So instead of resampling once, let's resample lots of times (here, 10000)! Create an empty vector `resampleMeans` to store these values.


```r
resampleMeans = numeric(10000)
for(i in 1:10000){
  newSample <- sample(distribution1,length(distribution1),replace=TRUE)
  newSampleMean <- mean(newSample)
  resampleMeans[i] <- newSampleMean
}

mean(resampleMeans)
```

```
## [1] 3.417893
```

```r
sd(resampleMeans)
```

```
## [1] 0.4310896
```

Note two things. First, the mean printed above is close to that of `distribution1` (if you reran the script for generating `distribution1`, you may need to rerun the above script). However, the standard deviation is quite different. Wait for the workshop for an explanation!

To be able to do the same resampling for other data without having to manually replace the variable names each time, we create a custom function. The following do the same task, but the second uses `dplyr` piping, which is a very useful way to be able to handle multiple sequential operations in R - see [here](https://rpubs.com/justmarkham/dplyr-tutorial). Basically, it replaces `mean(sample(data))` with `data %>% sample %>% mean` which is much easier to read. 


```r
resample1 <- function(data){
  resampleMeans = numeric(10000)
  for(i in 1:10000){
    resampleMeans[i] <- mean(sample(data, length(data), replace=TRUE))
  }
  return(resampleMeans)
}

resample2 <- function(data){
  resampleMeans = numeric(10000)
  for(i in 1:10000){
    resampleMeans[i] <- data %>%
      sample(length(data),replace=TRUE) %>%
      mean
  }
  return(resampleMeans)
}

mean(resample1(distribution1))
```

```
## [1] 3.417441
```

```r
distribution1 %>% resample2 %>% mean
```

```
## [1] 3.411058
```

Becase we might be interested in things other than means, we can generalize this by using two functions, one for resampling and the other for extracting whatever variable we're interested in.


```r
resample <- function(data){
  resampleMeans = numeric(10000)
  for(i in 1:10000){
    resampleMeans[i] <- data %>%
      sample(length(data),replace=TRUE) %>%
      myFunction
  }
  return(resampleMeans)
}

myFunction <- function(x){
  return(mean(x, na.rm=T))
}

resampled1 <- resample(distribution1)
resampled2 <- resample(distribution2)
resampled3 <- resample(distribution3)
```

### Plotting

Finally, let's plot the resampled data


```r
data.frame(resampled1, resampled3) %>%
  gather(distribution,value) %>%
  ggplot(aes(x=value,color=distribution)) +
  geom_density() + 
  theme_bw()
```

![](unnamed-chunk-7-1.png)<!-- -->

And the original data


```r
data.frame(distribution1, distribution3) %>%
  gather(distribution,value) %>%
  ggplot(aes(x=value,color=distribution)) +
  geom_density() + 
  theme_bw()
```

![](unnamed-chunk-8-1.png)<!-- -->

