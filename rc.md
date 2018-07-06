[`Rcpp`](http://rcpp.org/) = R + C++(made easy)
=====

This material is based on the
*[High performance functions with Rcpp](http://adv-r.had.co.nz/Rcpp.html)*
chapter of the *Advanced R* book by Hadley Wickham.

# Getting started

## Requirement

> R is interpreted; C++ is a compiled language.

We will focus on short simple code snippets using [`Rcpp`](http://rcpp.org/):


```r
install.packages("Rcpp")
```
We need a C compiler:
- [Rtools](http://cran.r-project.org/bin/windows/Rtools/) on Windows
- Xcode on Mac
- package manager on Linux

(see [here](https://github.com/lgatto/teachingmaterial/wiki/R-and-C-code) for relevant links)


```r
library("Rcpp")
```

## Data types

> R is dynamically typed; C++ is statically typed.

| R         | Scalar | Vector          | Matrix          |
|-----------|--------|-----------------|-----------------|
| numeric   | double | NumericVector   | NumericMatrix   |
| integer   | int    | IntegerVector   | IntegerMatrix   |
| character | string | CharacterVector | CharacterMatrix |
| logical   | bool   | LogicalVector   | LogicalMatrix   | 


| R        | C++      |
|----------|----------|
| list     | List     |
| function | Function |

## Defining functions

In C++, we need to
- we don't assign the function body
- define input and output types
- explicitly return a value
- use `;`
- define `for` loops explicitly
- call a method using `.`: `x.size()` to get the size (length) of `x`


```r
sumR <- function(x) {
  total <- 0
  for (i in seq_along(x)) {
    total <- total + x[i]
  }
  total
}
```

```
double sumC(NumericVector x) {
  int n = x.size();
  double total = 0;
  for(int i = 0; i < n; ++i) {
    total += x[i];
  }
  return total;
}
```

## Compile and use in `R`

### Inline


```r
cppFunction('double sumC(NumericVector x) {
  int n = x.size();
  double total = 0;
  for(int i = 0; i < n; ++i) {
    total += x[i];
  }
  return total;
}')
sumC
```

```
## function (x) 
## .Call(<pointer: 0x2aaf32b3aef0>, x)
```

```r
sumC(c(1, 2, 1:4, rnorm(3)))
```

```
## [1] 12.26603
```

### Sourcing C++ code

The `./src/ex_sumC.cpp` file contains:

- include header and namespace statements
- the export directive (see
  [Rcpp attributes](http://dirk.eddelbuettel.com/code/rcpp/Rcpp-attributes.pdf))
- the C++ code
- as comment, R code that will be evaluated


```
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
double sumC(NumericVector x) {
  int n = x.size();
  double total = 0;
  for(int i = 0; i < n; ++i) {
    total += x[i];
  }
  return total;
}

/*** R
# Comparing with R
(x <- c(1, 3, rnorm(10)))
sumC(x)
sum(x)
*/
```

To compile the code and create the R wrapper, we use:


```r
sourceCpp("./src/ex_sumC.cpp")
```

```
## 
## > (x <- c(1, 3, rnorm(10)))
##  [1]  1.0000000  3.0000000  0.9034087  0.5264740  1.0087976 -0.3438904
##  [7]  0.8372165  1.2515454 -2.8218494  0.9934861  1.3197611  0.2468353
## 
## > sumC(x)
## [1] 7.921785
## 
## > sum(x)
## [1] 7.921785
```

## An example with a matrix


```
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
NumericVector rowSumsC(NumericMatrix x) {
  int nrow = x.nrow(), ncol = x.ncol();
  NumericVector out(nrow);

  for (int i = 0; i < nrow; i++) {
    double total = 0;
    for (int j = 0; j < ncol; j++) {
      total += x(i, j);
    }
    out[i] = total;
  }
  return out;
}
```

Note:
- The use of `()` instead of `[]`
- `.nrow` and `.ncol` to get the number of rows/cols

## An example with `if`/`else`


```r
signR <- function(x) {
  if (x > 0) {
    1
  } else if (x == 0) {
    0
  } else {
    -1
  }
}
```

```
int signC(int x) {
  if (x > 0) {
    return 1;
  } else if (x == 0) {
    return 0;
  } else {
    return -1;
  }
}
```

# Exercises

- Get familiar with the syntax and write/test the `sumC` and
  `rowSumsC` functions above.

- Try and implement the following R functions.


```r
fibR <- function(n) {
    if (n == 0) return(0)
    if (n == 1) return(1)
    return(fibR(n - 1) + fibR(n - 2))
}

pdistR <- function(x, ys)
    sqrt( (x - ys) ^ 2 )

sety <- function(x, y) {
    x[x > 0] <- y
    x[x < 0] <- -y
    x
}
lgl_biggerY <- function(x, y) x > y

biggerY <- function(x, y) x[x > y]

foo <- function(x, y) ifelse(x < y, x*x, -(y*y))
```

## Fibonacci

- Once you have a C++ version for the fibonacci function, compare the
  R, byte-compiled R and C implementions.


- Instead of using recursion, which is particularly slow in R, compare
  with the following implementation in R, byte-compiled R and your C++
  version.
  

```r
fib <- function(n) {
    res <- c(1, 1, numeric(n-2))
    for (i in 3:length(res))
        res[i] <- res[i-1] + res[i-2]
    return(res)
}
```

If you are interested in more implementations and timings, see this
[post](http://lgatto.github.io/fibo/).

# Rcpp sugar

/sugar/ (for syntactic sugar ) is a set of C++ functions that (mostly)
work and look like their R couterparts. Allows for example compact
vectorised expression. Looks like R with the C++ efficiency (see the
[Rcpp-sugar](http://dirk.eddelbuettel.com/code/rcpp/Rcpp-sugar.pdf) vignette/paper).

- Vectorised arithmetic and logical operators: +, >, !, ...
- Functions: seq_len, seq, sapply, rnorm, abs, sum, ..

### Arithmetic and logical operators

`+ *, -, /, pow, <, <=, >, >=, ==, !=, !`

```
NumericVector pdistC2(double x, NumericVector ys) {
  return sqrt(pow((x - ys), 2));
}
```

### Logical summary functions

`any()` and `all()` with `.is_true()`, `.is_false()`, `.is_na()` (to
convert to `bool`).

```
bool any_naC(NumericVector x) {
  return is_true(any(is_na(x)));
}
```
### Vector views

`head(), tail(), rep_each(), rep_len(), rev(), seq_along(), seq_len()`

### Other useful functions

- Math functions: `abs(), acos(), asin(), atan(), beta(), ceil(),
  ceiling(), choose(), cos(), cosh(), digamma(), exp(), expm1(),
  factorial(), floor(), gamma(), lbeta(), lchoose(), lfactorial(),
  lgamma(), log(), log10(), log1p(), pentagamma(), psigamma(),
  round(), signif(), sin(), sinh(), sqrt(), tan(), tanh(),
  tetragamma(), trigamma(), trunc()`.

- Scalar summaries: `mean(), min(), max(), sum(), sd(), and (for
  vectors) var()`.

- Vector summaries: `cumsum(), diff(), pmin(), and pmax()`.

- Finding values: `match(), self_match(), which_max(), which_min()`.

- Dealing with duplicates: `duplicated(), unique()`.

- `d/q/p/r` for all standard distributions.

## Exercises

Revise the following functions to employ sugar:

```
rowSums pdist lgl_bigger foo
```

For example, the `sumC` function above can be rewritten

```
// [[Rcpp::export]]
double sumC2(NumericVector x) {
  double ans = sum(x);
  return ans;
}
```

Implement [deferred evaluation](https://github.com/lgatto/rccpp/blob/master/deferred-eval.md)
in C++.

# Other

## Missing values

Will require extra care to emulate the standard `R` behaviour. See
[Advanced R Rcpp section](http://adv-r.had.co.nz/Rcpp.html#rcpp-na).

## More C++ 

### STL

C++ is a widely used for complex data structures and algorithms, which
can be leveraged via the `Rcpp` package. The **Standard Template
Library** (STL) provides such facilities.

- For example, iterators, i.e. iteration (`for` loop) abstraction

```
#include <Rcpp.h>
using namespace Rcpp;

// [[Rcpp::export]]
double sum3(NumericVector x) {
  double total = 0;
  
  NumericVector::iterator it;
  for(it = x.begin(); it != x.end(); ++it) {
    total += *it;
  }
  return total;
}
```

- [Standard Template Library Algorithms](http://www.cplusplus.com/reference/algorithm/):
  Thef header `<algorithm>` defines a collection of functions especially
  designed to be used on ranges of elements.

## Boost

[Boost](http://www.boost.org/) is a set of libraries for the C++
programming language that provide support for tasks and structures
such as linear algebra, pseudorandom number generation,
multithreading, image processing, regular expressions, and unit
testing.

## Modules

[Modules](http://dirk.eddelbuettel.com/code/rcpp/Rcpp-modules.pdf)
enables to expose C++ classes and methods to R.

# A package with `Rcpp` code

Starting a new package. The `Rcpp.pacakage.skeleton` will, like the
usual `package.skeleton`, initialise the appropriate package
structure, including specific `Rcpp` requirements. In particular, this
code


```r
library("Rcpp")
Rcpp.package.skeleton("NewCppPackage", example_code = FALSE,
                      cpp_files = "code.cpp")
```

will copy `code.cpp` into `NewCppPackage/src/.`, run `sourceCpp` and
properly generate the export wrappers for the C++ code. The resulting
package will also have

In the `DESCRIPTION` file:

```
LinkingTo: Rcpp
Imports: Rcpp
```

In the `NAMESPACE` file:

```
useDynLib(mypackage)
importFrom(Rcpp, sourceCpp)
```

Whenever any existing C++ function's signature is modified, new C++
functions are removed or added, one needs to run
`Rcpp::compileAttributes("/path/to/NewCppPackage")` to update the
export wrappers.

### Exercise

- Use `Rcpp.package.skeleton` as descibed above and any C++ function
  witten so far and create a package as outlined above.

and/or

- Add a C++ function of your choice to an existing package (for
  example, add your own `gccount` to the `sequences` package).


# References

- [Rcpp book](http://www.rcpp.org/book/) and [web page](http://rcpp.org/)
- [High performance functions with Rcpp](http://adv-r.had.co.nz/Rcpp.html)
  chapter of the *Advanced R* book by Hadley Wickham.
- [Rcpp.org](http://rcpp.org/)
- [Rcpp book](http://www.rcpp.org/book/)


