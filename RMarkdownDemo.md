
# R Markdown Demo


## Configure

If you want to beautify your output, it always starts here. 
There are many options, and a few are laid out below. 
The `knitr` package has lots of options explained [here](http://yihui.name/knitr/options#chunk_options)

Always load all packages together at the top. That way future users will know exactly what they need to install. 




If you ever want someone else to be able to perfectly reproduce your results, always set the random seed at the top. Any number will do. 


```r
set.seed(1415)
```




Make up some data. 


```r
x <- 1:100
y <- rnorm(100, sd=3) + seq(10.05, 20, 10/100)
z <- factor(rep(letters[1:5], each=20))
dat <- data.frame(x, y, z)
```


This is an ugly way to preview data or display tables. 


```r
head(dat)
```

```
  x      y z
1 1 13.615 a
2 2 13.997 a
3 3  9.891 a
4 4 11.290 a
5 5 10.473 a
6 6 12.369 a
```


The `knitr` package has a simple built-in function for dealing with tables. This works well in either html or pdf output. 


```r
kable(head(dat))
```



|  x|      y|z  |
|--:|------:|:--|
|  1| 13.615|a  |
|  2| 13.997|a  |
|  3|  9.891|a  |
|  4| 11.290|a  |
|  5| 10.473|a  |
|  6| 12.369|a  |

This table has 100 rows and 3 columns. The 'x' variable starts at 1 and ends at 100. 


-----------










Plot the data - a trend emerges! 


```r
plot(dat$y ~ dat$x)
```

![plot of chunk plotData](figs/plotData1.png) 

```r
plot(dat$y ~ dat$z, col=unique(dat$z))
```

![plot of chunk plotData](figs/plotData2.png) 

```r
plot(dat$y ~ dat$x, col=dat$z)
```

![plot of chunk plotData](figs/plotData3.png) 

```r
plot(dat$y ~ dat$x, col=dat$z, pch=16)
```

![plot of chunk plotData](figs/plotData4.png) 


Let's see if the trends are significant using a linear model. 


```r
lm.xy <- lm(y ~ x, data=dat)
lm.zy <- lm(y ~ z, data=dat)

kable(summary(lm.xy)$coefficients)
```



|            | Estimate| Std. Error| t value| Pr(>&#124;t&#124;)|
|:-----------|--------:|----------:|-------:|------------------:|
|(Intercept) |   9.8493|     0.5971| 16.4959|             0.0000|
|x           |   0.0943|     0.0103|  9.1877|             0.0000|

```r
kable(summary(lm.zy)$coefficients)
```



|            | Estimate| Std. Error| t value| Pr(>&#124;t&#124;)|
|:-----------|--------:|----------:|-------:|------------------:|
|(Intercept) |  10.7411|     0.6683| 16.0730|             0.0000|
|zb          |   2.6180|     0.9451|  2.7702|             0.0067|
|zc          |   2.9348|     0.9451|  3.1054|             0.0025|
|zd          |   5.9349|     0.9451|  6.2798|             0.0000|
|ze          |   7.8656|     0.9451|  8.3227|             0.0000|


Since we have a clear pattern, plot the line we just modeled. 


```r
plot(dat$y ~ dat$x, pch=16, col='gray50', 
     las=1, bty='l', 
     xlab='The Independent Variable', 
     ylab='The Dependent Variable')
abline(lm.xy, col='tomato', lwd=4, lty=2)
text(0, max(dat$y), 'p < 0.0001', font=4, pos=4)
```

![plot of chunk plotLM](figs/plotLM.png) 


----------

Default R colors are useful but not that aesthetic. So we can assign them however we want to. Assigning them to the same dataframe keeps all data points lined up perfectly. ***R does not line up your data automatically!! You have to make sure everything is lined up before you can trust results!!***


```r
dat$col <- 'gray40'
dat$col[dat$z == 'b'] <- 'darkorange'
dat$col[dat$z == 'c'] <- 'olivedrab4'
dat$col[dat$z == 'd'] <- 'cornflowerblue'
dat$col[dat$z == 'e'] <- 'cyan2'
colors5 <- unique(dat$col)
```


And check the new colors. Note they are now a bit more colorblind-proof. 


```r
plot(dat$y ~ dat$x, col=dat$col, pch=16)
```

![plot of chunk replotNewColors](figs/replotNewColors.png) 


------------


The linear model created above seems to be pseudoreplicated. Lets create a dataset that combines data from each group (a, b, c, d). The `aggregate` function is perfect. 

R does not have a default fuction for standard error, so we'll create one. 


```r
se <- function(a) {sd(a)/sqrt(length(a))}
```



```r
grouped <- data.frame(matrix(0, nrow=nlevels(dat$z), ncol=3))
names(grouped) <- c('mean', 'sd', 'se')
row.names(grouped) <- levels(dat$z)
grouped$mean <- aggregate(dat$y, by=list(dat$z), FUN='mean')$x
grouped$sd <- aggregate(dat$y, by=list(dat$z), FUN='sd')$x
grouped$se <- aggregate(dat$y, by=list(dat$z), FUN='se')$x
grouped
```

```
   mean    sd     se
a 10.74 3.072 0.6870
b 13.36 2.589 0.5789
c 13.68 3.632 0.8121
d 16.68 2.731 0.6106
e 18.61 2.805 0.6272
```



```r
y.lim <- c(min(grouped$mean - grouped$sd), 
           max(grouped$mean + grouped$sd))
plot(grouped$mean ~ c(1:5), 
     type='n', ylim=y.lim, las=1, bty='l', 
     xlab = 'This other variable', ylab='The response variable')
arrows(x0 = c(1:5), y0 = grouped$mean + grouped$sd, 
       x1 = c(1:5), y1 = grouped$mean - grouped$sd, 
       col='gray50', angle=90, code=3, length=0.08, lwd=2)
points(grouped$mean ~ c(1:5), 
       pch=21, bg=colors5, col='gray20', cex=2)
```

![plot of chunk plotGrouped](figs/plotGrouped.png) 












