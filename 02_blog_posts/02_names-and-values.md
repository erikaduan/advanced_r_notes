Chapter 2: Names and Values
================
Erika Duan
2020-05-24

``` r
#-----load R libraries-----   
if (!require(pacman)) install.packages("pacman")
p_load(lobstr,
       tidyverse)  
```

# Aim

Understanding the distinction between an **object** and its **name**
helps you to:

  - Better predict the performance and memory usage of your code.  
  - Write faster code by avoiding accidental copies, a major source of
    slow code.  
  - Better understand R’s functional programming tools.

**In R, it is the name that has a value bound to it.**

<img src="../01_figures/02_names-to-values.jpg" width="70%" style="display: block; margin: auto;" />

``` r
#-----exercise 1.1-----
a <- 1:10 
b <- a 
c <- b
d <- 1:10

object_funs <- function(object) {
  object
} 

e <- object_funs(object = a)

# (a) is a name that is assigned to a specific object [object 1:10]
# (a) and (b) are both names that point to the same object in memory
# (c) is a name that points to the name (b) - therefore (c) should point to what (a) and (b) are point to
# (d) is a value that is pointing to a new object [object 1:10]

obj_addr(a)
```

    ## [1] "0x18b9dd00"

``` r
obj_addr(b)
```

    ## [1] "0x18b9dd00"

``` r
obj_addr(c)
```

    ## [1] "0x18b9dd00"

``` r
obj_addr(d)
```

    ## [1] "0x18cb8a60"

``` r
obj_addr(e)
```

    ## [1] "0x18b9dd00"

``` r
#-----exercise 1.2-----  
# do the following functions all point to the same nameless object? 
# maybe a new object is generated each time the function runs? 
# it turns out that they are all the same object!

obj_addr(mean) 
```

    ## [1] "0x16d72178"

``` r
obj_addr(base::mean) # same thing as mean
```

    ## [1] "0x16d72178"

``` r
obj_addr(get("mean"))
```

    ## [1] "0x16d72178"

``` r
obj_addr(evalq(mean))
```

    ## [1] "0x16d72178"

``` r
obj_addr(match.fun("mean"))   
```

    ## [1] "0x16d72178"

``` r
#-----exercise 1.3 - 1.5-----  
# read.csv(check.names = T) automatically converts non-syntactic names to syntactic ones  
# names are adjusted by make.names()  
```

``` r
# Whenever you modify an object that has a name assigned to it, 
# you are not actually modifying the object itself,
# but creating a new object and re-binding your name to the new object. 
```

``` r
x <- c(1, 2, 3)
obj_addr(x)
```

    ## [1] "0x1a6c87d0"

``` r
#> [1] "0x296794a4b38"

cat(tracemem(x))
```

    ## <000000001A6C87D0>

``` r
#> <00000296794A4B38>

y <- x 
y[2] <- 1  
```

    ## tracemem[0x000000001a6c87d0 -> 0x000000001a706108]: eval eval withVisible withCallingHandlers handle timing_fn evaluate_call <Anonymous> evaluate in_dir block_exec call_block process_group.block process_group withCallingHandlers process_file <Anonymous> <Anonymous>

``` r
#> tracemem[0x00000296794a4b38 -> 0x00000296793ed828]: 
# tracing only happens when an object has more than one name bound to it now  

y <- c(y, 4, 5) 
obj_addr(y) 
```

    ## [1] "0x1a70ae98"

``` r
#> "0x2967998aa18" # i.e. y now points to a completely different object  

untracemem(x) # turns tracing off  
```

``` r
# lists are shallow copies  
```
