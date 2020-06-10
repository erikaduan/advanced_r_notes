Chapter 2: Names and Values
================
Erika Duan
2020-06-10

  - [Chapter goals](#chapter-goals)
  - [Binding basics](#binding-basics)
  - [Copy-on-modify](#copy-on-modify)
      - [Lists](#lists)
      - [Data frames](#data-frames)
      - [Character vectors](#character-vectors)
  - [Object size](#object-size)
  - [Modify-in-place](#modify-in-place)
  - [Unbinding and the garbage
    collector](#unbinding-and-the-garbage-collector)

``` r
#-----load R libraries-----   
if (!require(pacman)) install.packages("pacman")
p_load(lobstr,
       bench,
       tidyverse)  
```

# Chapter goals

Understanding the distinction between an **object** and its **name**
helps you to:

  - Better predict the performance and memory usage of your code.  
  - Write faster code by avoiding accidental copies, a major source of
    slow code.  
  - Better understand R’s functional programming tools.

# Binding basics

**In R, it is the name that has a value bound to it.**

<img src="../01_figures/02_names-to-values-1.jpg" width="50%" style="display: block; margin: auto;" />

``` r
#-----exercise 2.2.2.1-----
a <- 1:10 
b <- a 
c <- b
d <- 1:10  

# (a) is a name that is assigned to a specific object in memory i.e. object c(1:10)
# (a) and (b) are both names that point to the same object in memory
# (c) is a name that points to the name (b) - therefore (c) should point to what (a) and (b) are point to
# (d) is a value that is pointing to a new object in memory i.e. a second object c(1:10)   

# obj_addr(a)
#> [1] "0x198432d8f78"

# obj_addr(b)
#> [1] "0x198432d8f78"

# obj_addr(c)
#> [1] "0x198432d8f78"

# obj_addr(d)
#> [1] "0x1983d4afcb8"  

#-----what if the object assignment occurs via a function-----
object_funs <- function(object) {
  return(object)
} 

e <- object_funs(object = a)
# (e) is a name that points to the object assigned to (a)

# obj_addr(e)  
#> [1] "0x198432d8f78"  
```

``` r
#-----exercise 2.2.2.2-----  
# do the following functions all point to the same nameless object? 
# maybe a separate new object is generated each time the function runs? 
# it turns out that they are all the same object!

# obj_addr(mean) 
#> [1] "0x1983dd792b0"

# obj_addr(base::mean) # same thing as mean 
#> [1] "0x1983dd792b0"

# obj_addr(get("mean")) # get() returns the value of a named object  
#> [1] "0x1983dd792b0"  

# obj_addr(evalq(mean))  
#> [1] "0x1983dd792b0"

# obj_addr(match.fun("mean")) 
#> [1] "0x1983dd792b0"

#-----what happens if we bind mean(x) to a value-----
a <- mean(c(1, 3))
b <- mean(c(1, 3))  
c <- a  

# (a) and (c) will point to the same object in memory
# (b) will point to a different object in memory 

# obj_addr(a)
#> [1] "0x19842f4ca30"  

# obj_addr(b) 
#> [1] "0x19842eb3130"  

# obj_addr(c) 
#> [1] "0x19842f4ca30"  
```

Note that the evaluation or expression of a function always points to
the same object in memory. However, once the output of a function has
separate values assigned to it, they now exist as separate objects in
memory.

``` r
#-----exercise 2.2.2.3-----   
# read.csv() contains the default argument check.names = T
# this automatically converts non-syntactic names to syntactic ones via make.names()   

#-----exercise 2.2.2.4-----   
# make.names() 
# X is added as a prefix if the original name begins with a "." or "_" or number
# all invalid characters are translated to "."  
# missing value is translated to "NA"
# "." is appended to original names matching R keywords i.e. TRUE or if 
# duplicated values are altered by make.unique  

#-----exercise 2.2.2.5-----   
# .123e1 is also not a syntactic name because it contains a "." followed by a number  
```

# Copy-on-modify

## Lists

Copy-on-modify refers to the fact that when a copy of an object is
modified, R creates a separate new object and the original name is bound
to the new object instead.

<img src="../01_figures/02_names-to-values-2.jpg" width="50%" style="display: block; margin: auto;" />

A list superficially appears to stores a collection (i.e. list) of
values/objects of differing lengths. It actually stores a collection of
references (i.e. an object in memory itself) pointing to values/objects
stored in memory.

Note that a copy of a list is a shallow copy (i.e. the list and its
reference bindings are copied, but the values that the reference is
bound to are not copied).

<img src="../01_figures/02_names-to-values-3.jpg" width="60%" style="display: block; margin: auto;" />

``` r
#-----copy-on-modify behaviour of lists-----  
list_1 <- list(1, 2, 3)
list_2 <- list_1

# ref(list_1, list_2)  
#> o [1:0x20269f86a30] <list> 
#> +-[2:0x20267266668] <dbl> 
#> +-[3:0x20267266630] <dbl> 
#> \-[4:0x202672665f8] <dbl> 


# modifying a single value in list 2 results in a shallow copy

list_2[[3]] <- 10

# ref(list_1, list_2)

#> o [1:0x20269f86a30] <list> 
#> +-[2:0x20267266668] <dbl> 
#> +-[3:0x20267266630] <dbl> 
#> \-[4:0x202672665f8] <dbl> 
  
#> o [5:0x20269cc9740] <list> 
#> +-[2:0x20267266668] 
#> +-[3:0x20267266630] 
#> \-[6:0x2026794dbc8] <dbl>
```

## Data frames

Data frames can also be considered as lists of vectors. When you modify
a single column, only that column will be modified (the column points to
different objects in memory) and the remaining columns will still point
to their original objects in memory.

<img src="../01_figures/02_names-to-values-4.jpg" width="60%" style="display: block; margin: auto;" />

``` r
#-----modifying a df by column-----
df_1 <- data.frame(c(1, 2, 3),
                   c(4, 5, 6))
df_2 <- df_1

# ref(df_1, df_2)  
#> o [1:0x20269eb2840] <df[,2]> 
#> +-c.1..2..3. = [2:0x2026c4ee388] <dbl> 
#> \-c.4..5..6. = [3:0x2026c4ee298] <dbl>  

df_2[, 2] <- df_2[, 2] - 2
 
# ref(df_1, df_2)   
#> o [1:0x20269eb2840] <df[,2]>    
#> +-c.1..2..3. = [2:0x2026c4ee388] <dbl>    
#> \-c.4..5..6. = [3:0x2026c4ee298] <dbl>    
  
#> o [4:0x20268636840] <df[,2]>   
#> +-c.1..2..3. = [2:0x2026c4ee388]    
#> \-c.4..5..6. = [5:0x2026752a860] <dbl>    

#-----modifying a df by row modifed every column (all columns are re-created)-----
df_3 <- df_1
df_3[1, ] <- df_3[1, ] * 1

# ref(df_1, df_3)
#> o [1:0x20269eb2840] <df[,2]>    
#> +-c.1..2..3. = [2:0x2026c4ee388] <dbl>   
#> \-c.4..5..6. = [3:0x2026c4ee298] <dbl>   
  
#> o [4:0x20269dbdf48] <df[,2]>   
#> +-c.1..2..3. = [5:0x2026c9507d8] <dbl>    
#> \-c.4..5..6. = [6:0x2026c950788] <dbl>   
```

## Character vectors

Character vectors are stored in a different manner to numerical vectors.
For character vectors, R uses a global string pool (a pool containing
all unique strings) where each element of the character vector points to
a unique string.

<img src="../01_figures/02_names-to-values-5.jpg" width="60%" style="display: block; margin: auto;" />

``` r
#-----examining character vector referencing------
x <- c("an", "ant", "ate", "an", "ant", "aye?")

# ref(x, character = T) 
#> o [1:0x2026d27f020] <chr> 
#> +-[2:0x202606756f8] <string: "an"> 
#> +-[3:0x20268c9d0e0] <string: "ant"> 
#> +-[4:0x20268fe93d0] <string: "ate"> 
#> +-[2:0x202606756f8] 
#> +-[3:0x20268c9d0e0] 
#> \-[5:0x20268fe92b8] <string: "aye?"> 
```

``` r
#-----exercise 2.3.6.1-----   
# tracemem(1:10) is not useful as the values 1:10 have not been assigned a name 
# tracemem() is useful for tracing when named objects are subsequently copied

#-----exercise 2.3.6.2-----  
x <- c(1L, 2L, 3L)

# tracemem(x)

x[[3]] <- 4 # creates a new copy of x as double type, then modified x[[3]] with double == 4

# untracemem(x) 
#> tracemem[0x000002026871ca60 -> 0x0000020268955060]: 
#> tracemem[0x0000020268955060 -> 0x000002026cbf2788]: 

y <- c(1L, 2L, 3L)
# tracemem(y)

y[[3]] <- 4L 

# untracemem(y) 
#> tracemem[0x0000020269ea9460 -> 0x0000020269eacaa0]:   

#-----exercise 2.3.6.3-----   
a <- 1:10
b <- list(a, a)
c <- list(b, a, 1:10)

# a is a value that points to an object in memory c(1:10)
# b is a separate list that points back twice to a
# c is a separate list that points back to b (a, a) and a, and a new object in memory

# ref(a, b, c)
#> [1:0x20267d504f8] <int> # a is bound to a new object in memory (series of integers)
  
#> o [2:0x20269f9f160] <list> 
#> +-[1:0x20267d504f8] # points back to object bound to a  
#> \-[1:0x20267d504f8] # points back to object bound to a  
  
# o [3:0x2026d31e2d0] <list> 
# +-[2:0x20269f9f160] # points back to list b 
# +-[1:0x20267d504f8] # points back to object bound to a  
# \-[4:0x20267decb48] <int> # points to a new object in memory (series of integers) 

#-----exercise 2.3.6.4-----    
z <- list(1:10)

# ref(z)
#> o [1:0x202673fb928] <list> 
#> \-[2:0x20266446110] <int> 

# tracemem(z)

z[[2]] <- z # adds a new vector i.e. z <- c(1:10) in the original list 
#> tracemem[0x00000202673fb928 -> 0x00000202683fc6c8]: 

# untracemem(z)

# z is modified so a new list reference is produced  
# the second value in list z points to the original object in memory (integer)
 
# ref(z)
 
#> o [1:0x20269ea5160] <list> # new list reference 
#> +-[2:0x20266446110] <int> 
#> \-o [3:0x202673fb928] <list> 
#>   \-[2:0x20266446110]  
```

# Object size

The function `lobstr::obj_size` can be used to calculate the size of an
object. Note that `obj_size(x)` + `obj_size(y)` will only equal
`obj_size(x, y)` if there are no shared values.

``` r
#-----calculating how much memory an object takes-----
# dim(iris)
#> [1] 150   5    

# obj_size(iris)
#> 7,688 B 

#-----list sizes are smaller when referencing the same values-----
x <- runif(1e6) # 1 million random values

# obj_size(x)
#> 8,000,048 B

list_x <- list(x, x, x)
# obj_size(list_x)
#> 8,000,128 B # only 80 B larger than x

# obj_size(list(NULL, NULL, NULL))
#> 80 B  

# obj_size(list(NA, NA, NA))
#> 248 B

# obj_size(list(1, 1, 1))
#> 248 B 

# note that obj_size(list(NA, NA, NA)) produces a slightly larger object 
# this is because NA is treated as a logical type that denotes a missing value  
# https://www.r-bloggers.com/r-na-vs-null/  
```

``` r
#-----character vectors are smaller when referencing the same strings-----
# obj_size("banana")
#> 112 B

# obj_size(c(rep("banana"), 10000))
#> 176 B
```

Note that R 3.5.0 and later versions operates using compact storage of
certain vector types (i.e. for numbers `1:j`, only the first and last
numbers are stored).

``` r
#-----ALTREP in R 3.5.0 and later versions-----
# obj_size(1:5)
#> 680 B

# obj_size(1:50000)
#> 680 B 

# note that c(1:50000) does not work because it assigns a name to each individual value  

# obj_size(c(1:50000))
#> 200,048 B
```

``` r
#-----exercise 2.4.1.1-----  
y <- rep(list(runif(1000)), 100)

# y is a list of 100 identical values (each containing a numerical vector of n = 1000)

# object.size(y)
#> 8005648 bytes
# obj_size(y)
#> 80,896 B

# object.size(y) >> obj_size(y) because it does not detect if elements of a list are shared

x <- rep("cat", 100)
# object.size(x)
#> 904 bytes

# obj_size(x)
#> 904 B 

# otherwise object.size(y) == obj_size(y)

#-----exercise 2.4.1.3-----  
a <- runif(1e6)
# obj_size(a)
#> 8,000,048 B 

b <- list(a, a)
# obj_size(b) # similar size to but slightly larger than a
#> 8,000,112 B 

# obj_size(a, b) # identical size to b
#> 8,000,112 B

b[[1]][[1]] <- 10
# obj_size(b) 
#> 16,000,160 B

# I thought it would be identical in size to original b 
# but two different lists have now been created because of copy-on-modify!    

# obj_size(a, b) # identical to b because a == b[[2]]
#> 16,000,160 B

b[[2]][[1]] <- 10 
# obj_size(b)
#> 16,000,160 B

# I thought it would be back to 8,000,112 B but because of copy-on-modify, 
# a new object (numerical vector) has now been created in a new space!  

# obj_size(a, b) 
#> 24,000,208 B
```

# Modify-in-place

Although modifying an R object usually creates a copy, two exceptions
exist. Objects with a single binding get a special performance
optimisation.

``` r
#-----exploring objects with a single binding-----  
v <- c(1, 2, 3)

# tracemem(v)  
#> [1] "<000002026FB6D550>" 

v[3] <- 10   
#> tracemem[0x000002026fb6d550 -> 0x0000020268fb75d0]:   

# does this mean that a copy is made and object 000002026FB6D550 still exists?   
```

Environments, a special type of object, are always modified in place.
When you modify an environment all existing bindings to that environment
continue to have the same reference. This idea can be used to create
advanced functions that “remember” their previous state (i.e. the R6
object oriented-programming system).

``` r
#-----modifying an environment-----
e1 <- rlang::env(a = 1, b = 2, c = 3)
e2 <- e1

# ref(e1, e2)
#> o [1:0x2026dc7a2c8] <env> 
#> +-a = [2:0x202676bfcf8] <dbl> 
#> +-b = [3:0x202676bfd30] <dbl> 
#> \-c = [4:0x202676bfd68] <dbl> 
 
# [1:0x2026dc7a2c8] 

e1$c <- 10 

# modifying e1$c modifies the object that c is now pointing to 

# e2$c
#> [1] 10

# therefore e2$c also changes (it is now pointing to the same new object)
```

``` r
#-----exercise 2.5.3.1-----  
# why is a circular loop (where empty lists keep getting added to x) not created?
x <- list()

# obj_addr(x)
#> [1] "0x25a8bcfd368"

# tracemem(x)

x[[1]] <- x 

#> tracemem[0x0000025a8bcfd368 -> 0x0000025a91283258]: 

# untracemem(x)  

# a copy of the original object x was created, modified and x is newly assigned to a reference
# the original object x still remains as an object in memory  

# ref(x)

#> o [1:0x25a917c73c0] <list> # copied object has new memory address
#> \-o [2:0x25a8bcfd368] <list> # list element separately points to original object  

#-----exercise 2.5.3.2-----
x <- data.frame(matrix(runif(1), ncol = 1))

# tracemem(x)
#> [1] "<000001F68F724140>"

for (i in seq_along(x)) {
  x[[i]] <- x[[i]] + 1
}

# untracemem(x)

#> tracemem[0x000001f68f724140 -> 0x000001f68f8e9750]: 
#> tracemem[0x000001f68f8e9750 -> 0x000001f68f8e9600]: [[<-.data.frame [[<- 
#> tracemem[0x000001f68f8e9600 -> 0x000001f68f8e9478]: [[<-.data.frame [[<- 

#> tracemem[0x000001f68f8e9478 -> 0x000001f68f987e10]: as.list.data.frame as.list vapply which .rs.frameCols.rs.toDataFrame <Anonymous>   

#-----exercise 2.5.3.3-----  
# Error in tracemem(e1) : 
#   'tracemem' is not useful for promise and environment objects 
#    because environments are not duplicated object!   
```

# Unbinding and the garbage collector

Removing a value does not clear up your memory, as the only the value
that points to an object in memory is removed (i.e. the objects
themselves are valueless but still stored in memory). In R, the job of
deleting valueless objects in memory belongs to the garbage collector.

<img src="../01_figures/02_names-to-values-6.jpg" width="60%" style="display: block; margin: auto;" />

Note that the garbage collector (GC) runs automatically whenever R needs
more memory to create a new object, and that there is no need to
manually call `gc()` yourself.
