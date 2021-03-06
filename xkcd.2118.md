---
title: "xkcd 2118"
output: html_document
---


## Description of the problem

First, see xkcd 2118 https://xkcd.com/2118/

Imagine a normal distribution cut into 3 horizontal bands like the one below, we'll call `i` a vertical deviation from the midpoint `M`, both up and down, that defines the 2 lines that split the distribution into 3 bands. We'll call the lowest band `A`, the middle `B`, and the upper `C`.

Our goal is to find the value of `i` that makes the density of band `B` equal to 50%.

```r
#just some code to make the description of the problem plot below
midpoint = dnorm(0) * 0.5
s = seq(-3.5, 3.5, .01)
plot(s, dnorm(s), ty='l')
abline(h=midpoint-0.15)
abline(h=midpoint+0.15)
text(x=0, y=midpoint, 'M' )
text(x=0, y=0.38, "C")
text(x=0, y=0.14, "B")
text(x=0, y=0.02, "A")
text(x=0.3, y=0.3, "i")
text(x=-0.3, y=0.1, "i")
lines(x=c(0.2, 0.2), y=c(midpoint, midpoint+0.15))
lines(x=c(-0.2, -0.2), y=c(midpoint, midpoint-0.15))
```

![plot of chunk hist](figure/hist-1.png)

## Find density of B and return sq. error with our target of 50%
We'll make a function called `B_band_density` that will help us find the density of B and return it's sq. error from our target. This function takes a value of `i`, and uses two integrals and a little math to find the density of `B`. It returns the sq. error of `B` from our target of 50%.


```r
B_band_density = function(i, target=0.5){
  #define midpoint and upper and lower bounds of bands that define A, B, and C
  midpoint = 0.5 * dnorm(0) #identify the midpoint of N(0,1)
  upper = midpoint+i #upper bound of band i around midpoint
  lower = midpoint-i #lower bound of band i around midpoint

  #Next indentify the density of A, B, and C by finding integral of A and A+B with the funcs below
  f1 = function(x){ #func for density of A
    return(pmin(dnorm(x), lower)) #need pmin b/c integrate expects vectorized func
  }
  f2 = function(x){ #func for density of A+B
    return(pmin(dnorm(x), upper)) #same logic as f1
  }

  #Now use numerical integration to find the density of A and A+B 
  a = integrate(f1, -Inf, Inf)$value #density A
  a_plus_b = integrate(f2, -Inf, Inf)$value #density A+B

  #Now a little arithmetic to identify density of B
  c = 1 - a_plus_b # C is by defn what's left from 1 after taking away A and B
  b = 1 - (a+c)    # and B is what's left after taking away A and C

  #return the sq. error from target density, makes func convex so we can numerically find minima
  return((b-target)^2)  

  }
```
## Numerical optimization to find `i` such that density B = 50%
Now that we have a function that gives the sq. error of the density of `B` for any given value of `i`, we need to minimize this function. That will produce a value of `i` where `B` is very close to 50%  

```r
midpoint = 0.5 * dnorm(0) #re-identify midpoint, btw this is just sqrt(2*pi)^-1 * 0.5 , cuz mean and var terms drop out
#use numerical optmiziation to find i where density B = 0.5, that's our target
optimal_i = optimize(B_band_density, interval=c(0,midpoint))$minimum #I really love this func in R
```
## Finally find what percent the height at N(0,1,x=0) our optimal `i` is

```r
2 * optimal_i / dnorm(0)
```

```
## [1] 0.5268282
```
That matches the xkcd comic very well, I think we've got it!
