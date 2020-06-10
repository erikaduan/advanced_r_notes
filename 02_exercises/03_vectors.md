Chapter 3: Vectors
================
Erika Duan
2020-06-10

  - [Chapter goals](#chapter-goals)
  - [Vector types](#vector-types)
      - [Atomic vectors](#atomic-vectors)

``` r
#-----load R libraries-----   
if (!require(pacman)) install.packages("pacman")
p_load(tidyverse)  
```

# Chapter goals

Understanding the properties and behaviours of vectors helps you to:

  - Understand the difference between atomic vectors and lists.  
  - Understand how different object types (i.e. factors, dates,
    matrixes, data frames) are created from vectors.

# Vector types

There are techincally three different types of vectors:

  - Atomic vector - all elements must contain the same element type.  
  - List - elements do not need to contain the same element type.  
  - `NULL` - can be viewed as a generic vector of zero length.

Vectors have attributes, which is a list of metadata. The dimension
attribute turns vectors into matrices and arrays and the class attribute
powers the S3 object system (i.e. factors, date and times, data frames,
and tibbles).

## Atomic vectors
