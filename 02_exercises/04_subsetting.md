Chapter 4: Subsetting
================
Erika Duan
2020-08-05

  - [Chapter goals](#chapter-goals)
  - [Selecting multiple elements](#selecting-multiple-elements)
      - [Selecting atomic vectors](#selecting-atomic-vectors)

``` r
#-----load R libraries-----   
if (!require(pacman)) install.packages("pacman")
p_load(tidyverse)  
```

# Chapter goals

Understanding how subsetting is performed in R helps you to:

  - Perform complex operations in a succinct and fast manner.  
  - Understand how atomic vectors can be subsetted in 6 ways, and how
    subsetting operations differ for lists, matrices and data frames.  
  - Use subassignment (combine subsetting and assignment to modify parts
    of an object).

# Selecting multiple elements

## Selecting atomic vectors

There are 6 different ways you can subset atomic vectors.

  - Using positive integers (subset via position of element in
    vector).  
  - Using negative integers (subset by excluding via position of element
    in vector).  
  - Using logical vectors (subset by selecting elements where the
    corresponding logical value is `TRUE`).  
  - Using `[]` (subset everything and returns the original vector).  
  - Using ‘\[0\]’ (subset nothing and returns a zero length vector).  
  - Using character vectors if the vector is named via `setNames`
    (subset elements with matching names to character vector).

<!-- end list -->

``` r
#-----subset using positive integers-----  
# positive integers represent the position of the value in the vector (from the left to right)    

string <- c("apple", "bear", "cactus", "dog", "elephant")  

string[c(3, 1)] 
#> [1] "cactus" "apple" 

string[c(1, 1)] 
#> [1] "apple" "apple"  
```

``` r
#-----subset using negative integers-----
# negative integers exclude elements by position

string <- c("apple", "bear", "cactus", "dog", "elephant")  

string[-c(3, 1)]
#> [1] "bear"     "dog"      "elephant" 

string[c(-3, -1)]
#> [1] "bear"     "dog"      "elephant"

# note that x[-c(1, 2)] == x[c(-1, -2)]  

# string[c(1, -3)]
#> Show in New WindowClear OutputExpand/Collapse Output
#> Error in string[c(1, -3)] : 
#>   only 0's may be mixed with negative subscripts  

# note that you cannot mix positive and negative integers together when subsetting  
```

``` r
#-----subset using logical vectors-----  
# most popular method as we can subset via evaluated conditions  

string <- c(1.5, 2.3, 3.0, 4.5, 5.7)  

string[c(F, F, T)] # shorter lengths are not recycled
#> [1] 3  

string > 3 # evaluate a condition  
#> [1] FALSE FALSE FALSE  TRUE  TRUE

string[string > 3] # returns elements where condition == TRUE 
#> [1] 4.5 5.7  
```

``` r
#-----subset using []-----  
string <- c(1.5, 2.3, 3.0, 4.5, 5.7)  

string[]
#> [1] 1.5 2.3 3.0 4.5 5.7  
```

``` r
#-----subset using zero-----  
string <- c(1.5, 2.3, 3.0, 4.5, 5.7)   

string[0]
#> numeric(0)  
```

``` r
#-----subset via vector name-----   
string <- c(1.5, 2.3, 3.0, 4.5, 5.7)  

string <- setNames(string, letters[1:5])

string[c("b", "d")]
#>   b   d 
#> 2.3 4.5 

string[c("bb", "d")] # vector names and character vectors must be matched exactly  
#> <NA>    d 
#>   NA  4.5 
```

**Note:** Factors cannot be subsetted using character vectors (i.e. `NA`
is returned). Subsetting factors using positive integers will return
elements via the pposition of the element in the vector per usual.

``` r
#-----do not subset factor levels using character vectors-----  
string <- c("yes", "yes", "no", "yes", "no")   
string <- factor(string, levels = c("yes", "no"))

string["yes"]
#> [1] <NA>
#> Levels: yes no 

# positive and negative integer subsetting of factors works as per normal

string[3]
#> [1] no
#> Levels: yes no
```
