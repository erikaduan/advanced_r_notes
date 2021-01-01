Chapter 4: Subsetting
================
Erika Duan
2021-01-01

  - [Chapter goals](#chapter-goals)
  - [Subsetting atomic vectors](#subsetting-atomic-vectors)
      - [Subsetting factors](#subsetting-factors)
  - [Subsetting lists](#subsetting-lists)
  - [Subsetting matrices and arrays](#subsetting-matrices-and-arrays)
  - [Subsetting data frames and
    tibbles](#subsetting-data-frames-and-tibbles)
      - [Preserving dimensionality in matrices and data
        frames](#preserving-dimensionality-in-matrices-and-data-frames)
  - [Subsetting differences between `[[`, `[` and
    `$`](#subsetting-differences-between-and)
      - [Using `[[` versus `[`](#using-versus)
      - [Using `$`](#using)
  - [Subsetting and assignment](#subsetting-and-assignment)
  - [Subsetting applications](#subsetting-applications)
      - [Creating character lookup
        tables](#creating-character-lookup-tables)
      - [Matching and merging objects from a
        table](#matching-and-merging-objects-from-a-table)
      - [Random samples and bootstraps](#random-samples-and-bootstraps)
      - [Ordering](#ordering)
      - [Expanding aggregate counts](#expanding-aggregate-counts)
      - [Removing columns from data
        frames](#removing-columns-from-data-frames)
      - [Selecting rows based on a
        condition](#selecting-rows-based-on-a-condition)
      - [Boolean algebra versus sets](#boolean-algebra-versus-sets)

``` r
#-----load R libraries-----   
if (!require(pacman)) install.packages("pacman")
p_load(tidyverse)  
```

# Chapter goals

Understanding how **subsetting** is performed helps you to:

  - Understand how atomic vectors can be subsetted in different ways.  
  - Understand how subsetting operations differ for lists, matrices and
    data frames.  
  - Understand the difference between `$`, `[[x]]` and `[x]`.  
  - Use subassignment i.e. combine subsetting and assignment to modify
    parts of an object.

# Subsetting atomic vectors

There are 6 different ways you can subset atomic vectors.

  - Using positive integers - subset using element position in a vector,
    ordered from left to right.  
  - Using negative integers - subset by excluding the position of an
    element in a vector, ordered from left to right.  
  - Using logical vectors - subset by selecting elements where the
    corresponding logical value is `TRUE`.  
  - Using `[]` - subset nothing to return the original vector (useful
    for subassignment whilst preserving original object structure).  
  - Using `[0]` - subset using 0 to return a zero length vector.  
  - Using character vectors if the vector is named - subset using vector
    name.

<!-- end list -->

``` r
#-----subset using positive integers-----  
# positive integers represent the position of the value in the vector from the left to right

v1 <- c("apple", "bear", "cactus", "dog", "elephant")  

v1[c(3, 1)] 
#> [1] "cactus" "apple" 

v1[c(1, 1)] 
#> [1] "apple" "apple"  

# object duplication is allowed

v1[2.9]
#> [1] "bear"   

# floats are silently truncated to integers for positive integer subsetting 
```

``` r
#-----subset using negative integers-----
v1 <- c("apple", "bear", "cactus", "dog", "elephant")  

v1[-c(1, 2, 4)]
#> [1] "cactus"   "elephant"

v1[c(-1, -2, -4)]
#> [1] "cactus"   "elephant"

# v1[-c(1, 2)] is equivalent to v1[c(-1, -2)]  

# v1[c(1, -3)]
#> Show in New WindowClear OutputExpand/Collapse Output
#> Error in string[c(1, -3)] : 
#>   only 0's may be mixed with negative subscripts  

# you cannot mix positive and negative integers together when subsetting  
```

Subsetting using logical vectors is very useful as we can tailor
expressions which evaluate whether a condition is `TRUE` or `FALSE`.

**Note:** A distinct feature of logical vectors is that shorter vectors
are recycled during evaluation.

``` r
#-----subset using a logical vector-----  
v2 <- c(1.5, 2.3, 3.0, 4.5, 5.7)  

v2[c(F, F, T, F, T)]
#> [1] 3.0 5.7  

v2[c(T, F)]
#> [1] 1.5 3.0 5.7  

# expressions shorter than the logical vector length are recycled  

v2 > 3   
#> [1] FALSE FALSE FALSE  TRUE  TRUE

typeof(v2 > 3)
#> [1] "logical"  

v2[v2 > 3]  
#> [1] 4.5 5.7  

# subsetting using a logical vector returns elements where condition == TRUE
```

``` r
#-----subset nothing using []-----  
v2[]
#> [1] 1.5 2.3 3.0 4.5 5.7  

# useful for handling matrices, data frames and arrays inside lapply()       
```

``` r
#-----subset using [0]-----  
v2[0]
#> numeric(0)  

# useful for generating test data  
```

``` r
#-----subset using character vector names-----   
v2 <- c(1.5, 2.3, 3.0, 4.5, 5.7)  
v2 <- setNames(v2, letters[1:5])
v2
#>   a   b   c   d   e    
#> 1.5 2.3 3.0 4.5 5.7   

v2[c("b", "d")]
#>   b   d 
#> 2.3 4.5   

v2[c("b", "b")]
#>   b   b 
#> 2.3 2.3   

# we can subset using duplicate character vector elements      

v2[c("bb", "d")]  
#> <NA>    d 
#>   NA  4.5 

# vector names and character vector elements must match exactly   
```

## Subsetting factors

Named factors can also be subsetted by name using character vectors.
Subsetting factors using positive integers will also return elements by
element position as expected.

To reduce confusion, however, the recommendation is to avoid subsetting
factors and to convert factors into characters for subsetting where
possible.

``` r
#-----similarities in subsetting characters and factors-----  
v3 <- c("yes", "maybe", "no", "no", "no")     
v3 <- setNames(v3, paste0("response", seq(1, 5)))
v3
#> response1 response2 response3 response4 response5 
#>     "yes"   "maybe"      "no"      "no"      "no" 

f3 <- factor(v3, levels = c("yes", "no", "maybe"))
f3
#> response1 response2 response3 response4 response5 
#>       yes     maybe        no        no        no 
#> Levels: yes no maybe

v3[c("response1", "response3")]
#> response1 response3 
#>     "yes"      "no"  

f3[c("response1", "response3")]
#> response1 response3 
#>       yes        no 
#> Levels: yes no maybe

# named factors can still be subsetted by character name    

v3[c(1, 2, 5)]
#> response1 response2 response5 
#>     "yes"   "maybe"      "no"   

f3[c(1, 2, 5)]  
#> response1 response2 response5 
#>       yes     maybe        no 
#> Levels: yes no maybe
```

# Subsetting lists

You can select different components of a list using different
approaches.

  - Using `[]` to subset and return a list.  
  - Using `[[]]` or `[]$` to subset elements from a list and return a
    vector or data frame.

<!-- end list -->

``` r
#-----subset a list using []-----
list_a <- list(list_1 = c("apples", "bread", "milk", "eggs", "chocolate chips"),
               list_2 = c("keys", "wallet", "phone", "20.00"))

list_a[1]
#> $list_1
#> [1] "apples"          "bread"           "milk"            "eggs"            "chocolate chips"

class(list_a[1])
#> [1] "list"  

#-----subset elements from a list using [[]] or $-----   
list_a[[1]]
#> [1] "apples"          "bread"           "milk"            "eggs"            "chocolate chips"  

class(list_a[[1]])
#>[1] "character"  

list_a$list_2
#> [1] "keys"   "wallet" "phone"  "20.00" 

class(list_a$list_2) 
#> [1] "character"  
```

# Subsetting matrices and arrays

Higher dimension structures like matrices and arrays can be subsetted in
4 ways.

  - Using multiple vectors - `[c(rows), c(columns)]`. This returns a
    matrix or an atomic vector if a single row or column is returned.  
  - Using a single vector - `[c(1D vector element position)]`. This
    returns an atomic vector.  
  - Using a matrix - use a 2 column matrix to subset a matrix and a 3
    column matrix to subset a 3D array and etc. This returns an atomic
    vector.

**Note:** Blank subsetting using `[]` can also be incorporated to return
the original matrix or array with its row and/or column structure
preserved.

``` r
#-----subset matrix using multiple vectors-----   
mat_a <- matrix(1:12, nrow = 4)
dimnames(mat_a) <- list(row_names = c("ID1", "ID2", "ID3", "ID4"),
                        col_names = c("A", "B", "C"))
mat_a
#>          col_names
#> row_names A B  C
#>       ID1 1 5  9
#>       ID2 2 6 10
#>       ID3 3 7 11
#>       ID4 4 8 12

mat_a[1:2, 2:3] # first two rows by second and third column
#>          col_names
#> row_names B  C
#>       ID1 5  9
#>       ID2 6 10 

# you can also subset this way using logical vectors

mat_a[c(T, T, F, F), 2:3] 
#>          col_names
#> row_names B  C
#>       ID1 5  9
#>       ID2 6 10   

mat_a[1:2, ]
#>          col_names
#> row_names A B  C
#>       ID1 1 5  9
#>       ID2 2 6 10 

# you can incorporate blank subsetting inside [c(rows), c(columns)] to return all rows and/or columns    

mat_a[1, ]
#> A B C 
#> 1 5 9 

class(mat_a[1, ])
#> [1] "integer"  
```

``` r
#-----subset matrix using a single vector-----  
# treat matrix or array as 1D object and subset by 1D element position  
# not recommended as it does not make use of the 2D properties of matrices and arrays 

mat_a
#>          col_names
#> row_names A B  C
#>       ID1 1 5  9
#>       ID2 2 6 10
#>       ID3 3 7 11
#>       ID4 4 8 12

mat_a[c(1, 2, 3, 12)]
#> [1]  1  2  3 12  

# mat_a[c(1, 2, 3, 12)] subsets using a single vector and should not be confused with mat_a[c(1, 2, 3, 12), ]

class(mat_a[c(1, 2, 3, 12)]) 
#> [1] "integer"  
```

``` r
#-----subset a 2D matrix using a nx2 matrix-----  
mat_a
#>          col_names
#> row_names A B  C
#>       ID1 1 5  9
#>       ID2 2 6 10
#>       ID3 3 7 11
#>       ID4 4 8 12

# create a nx2 matrix where each row represents the [row, column] of a matrix element      

mat_subset <- matrix(c(1, 1, 2, 2, 3, 3), byrow = TRUE, ncol = 2)
mat_subset
#>      [,1] [,2]
#> [1,]    1    1
#> [2,]    2    2
#> [3,]    3    3

mat_a[mat_subset]  
#> [1]  1  6 11   

class(mat_a[mat_subset])
#> [1] "integer"  
```

**Note:** Subsetting a matrix by `[x]` also simplifies the result to the
lowest possible dimension.

# Subsetting data frames and tibbles

Data frames can behave like either lists or matrices depending on how
they are subsetted:

  - Subsetting columns only using a single index i.e. `df[1:2]` selects
    the first two columns and always returns a data frame.  
  - Subsetting with two indices i.e. `df[1:3, ]` selects the first three
    rows and all columns, and returns a data frame or atomic vector.

<!-- end list -->

``` r
#-----subset a data frame using a single index-----  
df1 <- data.frame(id = paste0("id", seq(1, 4)),
                  item = c("book", "pen", "ruler", "stickers"),
                  price = c(20.5, 1.5, 2, 2),
                  stringsAsFactors = FALSE)
df1
```

| id  | item     | price |
| :-- | :------- | ----: |
| id1 | book     |  20.5 |
| id2 | pen      |   1.5 |
| id3 | ruler    |   2.0 |
| id4 | stickers |   2.0 |

``` r
#-----subset a data frame using a single index-----   
single_index <- df1[c(1, 3)] # subsets by column position  

class(single_index)
#> [1] "data.frame"

single_index 
```

| id  | price |
| :-- | ----: |
| id1 |  20.5 |
| id2 |   1.5 |
| id3 |   2.0 |
| id4 |   2.0 |

``` r
#-----subset a data frame using two indices-----  
two_indices <- df1[1, c(1, 3)]

class(two_indices)
#> [1] "data.frame"  

two_indices
```

| id  | price |
| :-- | ----: |
| id1 |  20.5 |

## Preserving dimensionality in matrices and data frames

When you subset a matrix or data frame column using two indices, the
output will automatically simplify into an atomic vector.

This behaviour is avoided:

  - By using `drop = FALSE` during subsetting to preserve the original
    dimensionality,  
  - By converting the data frame into a tibble. A tibble always defaults
    to `drop = FALSE` i.e. always returns another tibble by default.

<!-- end list -->

``` r
#-----avoid subsetting a single data frame column by two indices-----
class(df1[1])
#> [1] "data.frame"

class(df1[, 1])
#> [1] "character"    

class(df1[, 1, drop = FALSE])
#> [1] "data.frame"  

# the argument drop = FALSE exists inside []      

tb1 <- as_tibble(df1)

class(tb1[1])
#> [1] "tbl_df"     "tbl"        "data.frame""

class(tb1[, 1])
#> [1] "tbl_df"     "tbl"        "data.frame"  

# tibbles do not have this inconsistency   
```

``` r
#-----exercise 4.2.6.1-----  
dim(mtcars)
#> [1] 32 11  

dim(mtcars[mtcars$cyl == 4, ])
#> [1] 11 11 

dim(mtcars[-c(1:4), ])
#> [1] 28 11

dim(mtcars[mtcars$cyl <= 5, ])
#> [1] 11 11  

# subsetting rows by a condition requires subsetting by two indices       

dim(mtcars[mtcars$cyl == 4 | mtcars$cyl == 6, ])
#> [1] 18 11  

# each condition must be written out in full 

dim(mtcars[mtcars$cyl %in% c(4, 6), ])
#> [1] 18 11

# simplify multiple conditions using functions like %in% and dplyr::between    

#-----exercise 4.2.6.2-----  
x <- 1:5  

x[NA]
#> [1] NA NA NA NA NA

x[NA_real_]  
#> [1] NA  

# performing operations with NA is infectious  
# NA is of logical type and vectors are recycled to the same length when subset by a logical type    

#-----exercise 4.2.6.3-----  
mat1 <- matrix(1:16, nrow = 4, byrow = TRUE)  

upper.tri(mat1)
#>       [,1]  [,2]  [,3]  [,4]
#> [1,] FALSE  TRUE  TRUE  TRUE
#> [2,] FALSE FALSE  TRUE  TRUE
#> [3,] FALSE FALSE FALSE  TRUE
#> [4,] FALSE FALSE FALSE FALSE  

# for row i, represent last (total columns - i) number of elements as TRUE  

mat1[upper.tri(mat1)]
#> [1]  2  3  7  4  8 12   

# subsetting by upper.tri() returns a simplified atomic vector  

#-----exercise 4.2.6.4----- 
#> mtcars[1:20]
#> Error in `[.data.frame`(mtcars, 1:20) : undefined columns selected  

# the error message tells us that there are non-existent columns being selected  
# this should remind us that subsetting data frames by a single index subsets entire columns    

mtcars[1: ncol(mtcars)]

#-----exercise 4.2.6.5----- 
mat1
#>      [,1] [,2] [,3] [,4]
#> [1,]    1    2    3    4
#> [2,]    5    6    7    8
#> [3,]    9   10   11   12
#> [4,]   13   14   15   16

# the number of elements we can subset is min(c(ncol(mat1), nrow(mat1)))   

diag(mat1) 
#> [1]  1  6 11 16  

diag_mat1 <- matrix(c(1, 1,
                      2, 2,
                      3, 3,
                      4, 4), ncol = 2, byrow = TRUE)

mat1[diag_mat1]
#> [1]  1  6 11 16  

# we need to subset mat1 by the position of its elements 
# we can store these positions in a second matrix  

#-----exercise 4.2.6.6----- 
df2 <- data.frame(id = paste0("id", seq(1, 5)),
                  pet = c("cat", NA, NA, "weaver", "dog"),
                  stringsAsFactors = FALSE)

df2[is.na(df2)]   
#> [1] NA NA  

# subsets all NA and simplifies output into an atomic vector

df2[is.na(df2)] <- "no pet"   
df2   
```

| id  | pet    |
| :-- | :----- |
| id1 | cat    |
| id2 | no pet |
| id3 | no pet |
| id4 | weaver |
| id5 | dog    |

# Subsetting differences between `[[`, `[` and `$`

## Using `[[` versus `[`

You should use `x[[1]]` to extract single items from lists comprising
multiple items. For consistency, you should also use `[[x]]` inside for
loops and functions when subsetting atomic vectors to return single
elements, to set a consistent expectation.

**Note**:: Because `x[[1]]` returns a single item, it only accepts a
single positive integer or string inside `[[]]`.

``` r
#-----difference between list[[x]] and list[x]-----  
list1 <- list(v1 = seq(1, 5),
              v2 = letters[1:5],
              v3 = c("cat", "dog", "kitten"))

list1[1]
#> $v1
#> [1] 1 2 3 4 5

class(list1[1])
#> [1] "list"   

list1[[1]]
#> [1] 1 2 3 4 5  

list1[["v1"]] 
#> [1] 1 2 3 4 5  

class(list1[[1]])
#> [1] "integer"   
```

``` r
#-----subset items from a list stored inside a list-----  
list2 <- list(list1,
              total = 3)  

list2  
#> [[1]]
#> [[1]]$v1
#> [1] 1 2 3 4 5
#> 
#> [[1]]$v2
#> [1] "a" "b" "c" "d" "e"
#> 
#> [[1]]$v3
#> [1] "cat"    "dog"    "kitten"
#> 
#> 
#> $total
#> [1] 3

list2[[1]][[1]]
#> [1] 1 2 3 4 5

list2[[2]]
#> [1] 3  
```

**Note:** We can also use the
[function](https://purrr.tidyverse.org/reference/pluck.html)
`purrr::pluck()` instead of `x[[1]][[1]]` to retrieve single items from
lists which have a complex i.e. deeply nested structure. The function
`pluck()` allows you to mix integer and character indices and provides
more output flexibility when an item does not exist.

``` r
#-----use x[[1]][[2]] or purrr::pluck()-----  
list2[[1]][[1]]
#> [1] 1 2 3 4 5   

list2 %>%
  pluck(1, 1)
#> [1] 1 2 3 4 5    

list2 %>%
  pluck(1, 4)
#> NULL   

list2 %>%
  pluck(1, 4, .default = NA)
#> [1] NA  
```

## Using `$`

The syntax `x$y` is a shorthand equivalent to `x[["y"]]` and is used to
subset columns in data frames.

``` r
#-----similarities between df$y and df[["y"]]-----    
df1 <- data.frame(id = paste0("id", seq(1, 4)),
                  item = c("book", "pen", "ruler", "stickers"),
                  price = c(20.5, 1.5, 2, 2),
                  stringsAsFactors = FALSE)  

df1[["item"]]
#> [1] "book"     "pen"      "ruler"    "stickers"   

df1$item
#> [1] "book"     "pen"      "ruler"    "stickers"  
```

**Note:** An undesirable property of `x$y` is that it uses partial
matching if the column name is not found. This problem can be avoided by
using tibbles or by setting the global option
`options(warnPartialMatchDollar = TRUE)`.

``` r
#-----partial matching with df1$y-----
df1$it   
#> [1] "book"     "pen"      "ruler"    "stickers"  

df1[["it"]]
#> NULL  
```

**Note:** The subsetting operators `@` and `slot()` are used for
handling S4 objects, with `@` behaving similarly to `$` and `slot()`
behaving similarly to `x[[1]]`.

``` r
#-----exercise 4.3.5.1-----  
mtcars$cyl 
mtcars[["cyl"]] 
mtcars[, "cyl"]  

var <- str_subset(colnames(mtcars), "cyl")
mtcars[[var]]

mtcars["cyl"] # note that this returns a data frame and not an atomic vector   

#-----exercise 4.3.5.2-----  
lm1 <- lm(mpg ~ wt, data = mtcars)

str(lm1) # examine deeply nested list structure   

lm1$df.residual
#> [1] 30

lm1[["df.residual"]]
#> [1] 30  

pluck(lm1, "df.residual")
#> [1] 30  

str(summary(lm1)) # examine deeply nested list structure    

summary(lm1)$r.squared
#> [1] 0.7528328  

summary(lm1)[["r.squared"]]  
#> [1] 0.7528328  

pluck(summary(lm1), "r.squared")  
#> [1] 0.7528328  
```

# Subsetting and assignment

You can combine subsetting operators with assignment to modify selective
items inside an object. This is called subassignment and forms the basis
of many data wrangling operations in R.

The basic form of subassignment is `x[i] <- value`.

Because R contains complex rules about when to recycle vectors of
shorter lengths, you should make sure that `length(value) ==
length(x[i])` and that `i` is a unique name.

``` r
#-----atomic vector subassignment-----
v1 <- c(1: 5) 
v1
#> [1] 1 2 3 4 5

v1[c(1, 5)] <- c(10, 50)
v1
#> [1] 10  2  3  4 50  

#-----data frame subassignment-----

df1$id
#> [1] "id1" "id2" "id3" "id4"  

df1[c(1, 5), "id"] <- c("id10", "id50")

df1$id
#> [1] "id10" "id2"  "id3"  "id4"  "id50"  

#-----list subassignment-----  
list1[[1]]
#> [1] 1 2 3 4 5    

list1[[1]][c(3, 5)] <- c(30, 50)

list1[[1]]
#> [1]  1  2 30  4 50 
```

**Note:** You can use `x[[i]] <- NULL` to remove an item within a list.

``` r
#-----removing items inside lists through subassignment-----  
str(list1)
#> List of 3
#>  $ v1: num [1:5] 1 2 30 4 50
#>  $ v2: chr [1:5] "a" "b" "c" "d" ...
#>  $ v3: chr [1:3] "cat" "dog" "kitten"

list1[["v2"]] <- NULL 

str(list1)
#> List of 2
#>  $ v1: num [1:5] 1 2 30 4 50
#>  $ v3: chr [1:3] "cat" "dog" "kitten"
```

Subsetting nothing via `x[]` can be useful during subassignment, if you
want the original structure of your object to be preserved and returned.

``` r
#-----subassignment using x[] to preserve original object structure-----  
iris_features <- datasets::iris[-5]

str(iris_features)
#> 'data.frame':    150 obs. of  4 variables:
#>  $ Sepal.Length: num  5.1 4.9 4.7 4.6 5 5.4 4.6 5 4.4 4.9 ...
#>  $ Sepal.Width : num  3.5 3 3.2 3.1 3.6 3.9 3.4 3.4 2.9 3.1 ...
#>  $ Petal.Length: num  1.4 1.4 1.3 1.5 1.4 1.7 1.4 1.5 1.4 1.5 ...
#>  $ Petal.Width : num  0.2 0.2 0.2 0.2 0.2 0.4 0.3 0.2 0.2 0.1 ...

iris_features <- lapply(iris[1:4], function(x) round(x, digits = 0))
class(iris_features) 
#> [1] "list"  

iris_features <- datasets::iris[-5]

iris_features[] <- lapply(iris[1:4], function(x) round(x, digits = 0))
class(iris_features) 
#> [1] "data.frame"  
```

# Subsetting applications

## Creating character lookup tables

Creating a named lookup vector and then subsetting this vector by its
vector names is a powerful method of creating a lookup table. This
application may seem counter-intuitive as we are subsetting a smaller
lookup vector using a much longer vector of interest
i.e. `length(lookup) < length(x)`, but it achieves the desired output
efficiently.

``` r
#-----creating a lookup table using character subsetting-----  
x <- c("m", "f", "u", "f", "f", "m", "m")

lookup <- c(m = "Male",
            f = "Female",
            u = NA) # name = value 
lookup
#>        m        f        u 
#>   "Male" "Female"       NA   

lookup[x]
#>        m        f        u        f        f        m        m    
#>   "Male" "Female"       NA "Female" "Female"   "Male"   "Male"   
```

## Matching and merging objects from a table

The function `match(x, table)` returns a vector of object positions,
based on whether objects in `x` match objects in `table`. Subsetting
your lookup table by the results of `match(x, table$x)` is the efficient
equivalent of performing a `left_join()` between an atomic vector and a
lookup table.

``` r
#-----matching and merging objects from a table----- 
grades <- c("C", "B", "B", "A", "C")  

info <- data.frame(grade = c("A", "B", "C"),
                   desc = c("Excellent", "Good", "Poor"),
                   fail = c(F, F, T),
                   stringsAsFactors = FALSE)  
info
```

| grade | desc      | fail  |
| :---- | :-------- | :---- |
| A     | Excellent | FALSE |
| B     | Good      | FALSE |
| C     | Poor      | TRUE  |

``` r
#-----testing match outputs-----
match("Good", info$desc)
#> [1] 2  

match(grades, info$grade)
#> [1] 3 2 2 1 3  
```

``` r
#-----outputs position of match based on info$grade object position-----    
by_grade <- match(grades, info$grade)

info[c(2, 1), ] # subsetting the dataframe by row
```

|   | grade | desc      | fail  |
| - | :---- | :-------- | :---- |
| 2 | B     | Good      | FALSE |
| 1 | A     | Excellent | FALSE |

``` r
info[by_grade, ] # still subsetting the dataframe by row but with duplicate values    
```

| grade | desc      | fail  |
| :---- | :-------- | :---- |
| A     | Excellent | FALSE |
| B     | Good      | FALSE |
| C     | Poor      | TRUE  |

**Note:** In the data frame above, automatic unique name conversions
creates confusing row names that should be ignored.

## Random samples and bootstraps

Positive integer indices can be used to randomly sample or bootstrap a
vector or data frame. We can use `sample(n)` to randomly generate a
permutation of integers between `1:n`, and then use this to subset our
vector or data frame.

We can also bootstrap data using the same method, as R allows the
subsetting of duplicate objects.

``` r
#-----use sample(n) to randomly sample or bootstrap data-----  
df <- data.frame(x = 1:5,
                 y = 5:1,
                 z = letters[1:5])

#-----randomly reorder the entire dataframe-----
set.seed(123)
df[sample(nrow(df)), ]
```

|   | x | y | z |
| - | -: | -: | :- |
| 3 | 3 | 3 | c |
| 2 | 2 | 4 | b |
| 5 | 5 | 1 | e |
| 4 | 4 | 2 | d |
| 1 | 1 | 5 | a |

``` r
#-----randomly select 3 rows of data-----
set.seed(234)
df[sample(nrow(df), 3), ]
```

|   | x | y | z |
| - | -: | -: | :- |
| 1 | 1 | 5 | a |
| 3 | 3 | 3 | c |
| 2 | 2 | 4 | b |

``` r
#-----randomly select 6 bootstrap replicates-----  
set.seed(345)
df[sample(nrow(df), 6, replace = T), ]
```

|     | x | y | z |
| --- | -: | -: | :- |
| 5   | 5 | 1 | e |
| 3   | 3 | 3 | c |
| 5.1 | 5 | 1 | e |
| 2   | 2 | 4 | b |
| 4   | 4 | 2 | d |
| 4.1 | 4 | 2 | d |

## Ordering

The function `order()` uses positive integer subsetting as it takes a
vector as its input and returns a positive integer vector describing how
to order the subsetted vector.

``` r
#-----order() outputs a positive integer vector for subsetting purposes-----  
x <- c("d", "a", "b", "c")

order(x)
#> [1] 2 3 4 1  

x[order(x)]
#> [1] "a" "b" "c" "d"  

# we return the output by positive integer subsetting: position 2 is "a", position 3 is "b" etc.     
```

For matrices and data frames, `order()` allows us to order either the
rows or columns of an object using positive integer subsetting.

``` r
#-----using order() to reorder data frame rows-----  
set.seed(123)
df2 <- df[sample(nrow(df)), c(2, 1, 3)] # create a disordered data frame
df2  
```

|   | y | x | z |
| - | -: | -: | :- |
| 3 | 3 | 3 | c |
| 2 | 4 | 2 | b |
| 5 | 1 | 5 | e |
| 4 | 2 | 4 | d |
| 1 | 5 | 1 | a |

``` r
#-----using order() to rearrange data frame rows-----  
order(df2$x)
#> [1] 5 2 1 4 3  

df2[order(df2$x), ]  
```

| y | x | z |
| -: | -: | :- |
| 5 | 1 | a |
| 4 | 2 | b |
| 3 | 3 | c |
| 2 | 4 | d |
| 1 | 5 | e |

``` r
#-----using order() to rearrange data frame columns-----
order(colnames(df2), decreasing = T)
#> [1] 3 1 2  

df2[, order(colnames(df2), decreasing = T)] 
```

|   | z | y | x |
| - | :- | -: | -: |
| 3 | c | 3 | 3 |
| 2 | b | 4 | 2 |
| 5 | e | 1 | 5 |
| 4 | d | 2 | 4 |
| 1 | a | 5 | 1 |

## Expanding aggregate counts

The function `rep(1:nrow(df), df$n)` can be applied to aggregate count
tables to repeat the number of times an individual value occurs in the
original dataset. Combining `rep()` and positive integer subsetting
enables us to expand aggregate counts.

``` r
#-----expand aggregate counts using rep() and integer subsetting-----  
df <- data.frame(item = letters[1:3],
                 type = LETTERS[1:3], 
                 n = c(3, 5, 1))
df
```

| item | type | n |
| :--- | :--- | -: |
| a    | A    | 3 |
| b    | B    | 5 |
| c    | C    | 1 |

``` r
#-----applying rep() to count table-----  
rep(1:nrow(df), df$n)
#> [1] 1 1 1 2 2 2 2 2 3    

df[rep(1:nrow(df), df$n), c("item", "type")] # expansion of aggregate counts   
```

|     | item | type |
| --- | :--- | :--- |
| 1   | a    | A    |
| 1.1 | a    | A    |
| 1.2 | a    | A    |
| 2   | b    | B    |
| 2.1 | b    | B    |
| 2.2 | b    | B    |
| 2.3 | b    | B    |
| 2.4 | b    | B    |
| 3   | c    | C    |

## Removing columns from data frames

You can remove columns from a data frame in two different ways:

  - By assigning an individual column to `NULL`.  
  - By subsetting to only return the columns you want to keep. You can
    use `setdiff()` to extract the names of columns you want to keep.

<!-- end list -->

``` r
#-----using setdiff to subset columns to keep-----  
setdiff(colnames(df), "type")
#> [1] "item" "n"     

df[setdiff(colnames(df), "type")]
```

    ##   item n
    ## 1    a 3
    ## 2    b 5
    ## 3    c 1

## Selecting rows based on a condition

Logical subsetting is the most common technique for extracting rows from
a data frame as you can easily conbine multiple conditions. This method
of subsetting rows from a dataset also forms the basis of
`dplyr::filter()`.

``` r
#-----applying logical subsetting on a data frame-----  
nrow(mtcars[mtcars$cyl <= 6, ])
#> [1] 18  

nrow(mtcars[mtcars$cyl <= 6 & mtcars$wt >= 3, ])
#> [1] 6  

nrow(mtcars[mtcars$cyl != 6, ]) 
#> [1] 25  
```

**Note:** `!(X & Y)` is the same as `!X | !Y` and `!(X | Y)` is the same
as (\!X & \!Y) according to [De Morgan’s
laws](https://en.wikipedia.org/wiki/De_Morgan%27s_laws).

## Boolean algebra versus sets

There is a natural equivalence between set operations (integer
subsetting) and Boolean algebra (logical subsetting).

Using set operations instead of Boolean algebra is recommended:

  - When you want to find the first or last element that evaluates to
    `TRUE`.  
  - When you have very few `TRUE` values and many `FALSE` values.

The function `which()` takes a condition as its input and outputs the
vector positions for which this is true i.e. converts a logical
subsetting approach into an integer subsetting approach.

We can also create our own function to convert an integer subsetting
approach into a logical subsetting approach.

``` r
#-----using which()-----  
v1 <- c(1:8)

which(v1 %% 2 == 0) # input a condition 
#> [1] 2 4 6 8 # output the vector element positions for which the condition is TRUE    
```

``` r
#-----create a function to convert an integer subsetting into a logical subsetting approach-----  
unwhich <- function(x, n){
  out <- rep_len(FALSE, n) # outputs a vector of FALSE  
  out[x] <- TRUE # change vector position to TRUE
  out # return the vector
}

unwhich(which(v1 %% 2 == 0), length(v1))
#> [1] FALSE  TRUE FALSE  TRUE FALSE  TRUE FALSE  TRUE    

# x represents the output of a condition and the vector integer positions which evaluate to TRUE   
```

``` r
#-----explore the relationship between Boolean algebra and set operations-----  
x1 <- 1:6 %% 2 == 0
x1
#> [1] FALSE  TRUE FALSE  TRUE FALSE  TRUE

x2 <- which(x1)
x2
#> [1] 2 4 6  

y1 <- 1:6 %% 3 == 0  
y1  
#> [1] FALSE FALSE  TRUE FALSE FALSE  TRUE

y2 <- which(y1)
y2
#> [1] 3 6 

#-----using intersect()-----  
# appears in both x and y  

x1 & y1
#> [1] FALSE FALSE FALSE FALSE FALSE  TRUE   

intersect(x2, y2) 
#> [1] 6   

#-----using union()----- 
# appears in x or y or both    

x1 | y1  
#> [1] FALSE  TRUE  TRUE  TRUE FALSE  TRUE

union(x2, y2)
#> [1] 2 4 6 3  

#-----using setdiff()-----   
# appears in x but not in y i.e. X & !Y  

x1 & !y1
#> [1] FALSE  TRUE FALSE  TRUE FALSE FALSE

setdiff(x2, y2)
#> [1] 2 4   

#-----applying xor()------  
# the difference between the union versus intersect of x and y 
# i.e. everything in set x and set y excepting the intersection of x and y    

xor(x1, y1)
#> [1] FALSE  TRUE  TRUE  TRUE FALSE FALSE  

setdiff(union(x2, y2), intersect(x2, y2))
#> [1] 2 4 3
```

``` r
#-----exercise 4.5.9.1-----  
# to randomly permute a column in the data frame, we would change the order of rows in that column  
# which breaks the relationship between that column (feature) and all other columns (features)   

df1 <- data.frame(brand = c("a", "b", "a", "c", "c"),
                  price = c(200, 120, 45, 12, 20), 
                  theft_time = c("day", "night", "night", "night", "day"), 
                  is_stolen = c("yes", "yes", "no", "no", "no"))

set.seed(123)
df1$price[sample(nrow(df1))]
#> [1]  45 120  20  12 200   

permutate_feature <- function(df, col) {
  set.seed(123)
  new_col <- df[sample(nrow(df)), col]
  df[col] <- new_col
  df
}

permutate_feature(df1, "price")
```

| brand | price | theft\_time | is\_stolen |
| :---- | ----: | :---------- | :--------- |
| a     |    45 | day         | yes        |
| b     |   120 | night       | yes        |
| a     |    20 | night       | no         |
| c     |    12 | night       | no         |
| c     |   200 | day         | no         |

``` r
#-----exercise 4.5.9.2-----  
# select a sample of m rows from a data frame
# but the sample must be contiguous i.e. preserve data frame row sequence   

df2 <- data.frame(id = paste0("id", 1:26),
                  letter = letters)

set.seed(123) # let m = 5 
first_row <- sample(nrow(df2) - (5 + 1), 1) 
last_row <- first_row + (5 - 1)                 

df2[first_row:last_row, ]   
```

|    | id   | letter |
| -- | :--- | :----- |
| 15 | id15 | o      |
| 16 | id16 | p      |
| 17 | id17 | q      |
| 18 | id18 | r      |
| 19 | id19 | s      |

``` r
#-----exercise 4.5.9.3-----  
# sort data frame column names and output them in alphabetical order  

df3 = data.frame(date = c("2020-12-12", "2020-08-19", "2020-12-01"),
                 item = c("book", "stickers", "pen"), 
                 cost = c(5, 2, 1.5))

order(colnames(df3))
#> [1] 3 1 2  

df3[order(colnames(df3))]
```

| cost | date       | item     |
| ---: | :--------- | :------- |
|  5.0 | 2020-12-12 | book     |
|  2.0 | 2020-08-19 | stickers |
|  1.5 | 2020-12-01 | pen      |
