# Rcpp_tricks
Rcpp tricks 骚操作收集

## Type conversion

```
#include <RcppArmadillo.h>
using namespace Rcpp;
//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
arma::mat type_conversion(arma::imat ttij){
  arma::mat ttij_double=arma::conv_to<arma::mat>::from(ttij);
  return(ttij_double*0.001);
}
```

## Element access

```
//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
double field_test(){
  arma::field<arma::mat> F(4,3);
  for(arma::uword col=0;col<F.n_cols;++col)
    for(arma::uword row=0;row<F.n_rows;++row){
      F(row,col)=arma::randu<arma::mat>(2,3);  // each element in field<mat> is a matrix
    }
    F.print("F:");
  return(F(1,1)(1,1));
}

//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
arma::mat divide_each_column(arma::mat AA,arma::vec bb){
  AA.each_col()/=bb;
  return(AA);
}

//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
arma::mat divide_one_column(arma::mat AA,arma::vec bb){
  AA.col(0)=AA.col(0)/bb;
  return(AA);
}
```
## Speed things up

### Fixed dimension in field

```
//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
arma::mat test6(int n){
  arma::field<arma::mat33> AA(4,4);
  arma::mat33 BB;
  BB.fill(0.5);
  AA.fill(BB);
  return(AA(1,2));
}
```
Also note the use of AA.fill(BB).

```
//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
arma::mat test7(int n){
  typedef arma::mat::fixed<10,10> mat00;
  arma::field<mat00> AA(4,4);
  mat00 BB;
  BB.fill(0.5);
  arma::mat CC(10,10,arma::fill::eye);
  AA.fill(BB);
  //AA(1,2)=CC;
  return(AA(1,2));
}
```
Note: things like arma::field<arma::mat::fixed<10,10>> do not work. This is the workaround.
Another tedious workaround:

```
//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
arma::mat test9(int n){
  arma::field<arma::mat::fixed<10,10>::fixed> AA(4,4);
  arma::mat::fixed<10,10> BB;
  BB.fill(0.5);
  arma::mat CC(10,10,arma::fill::eye);
  AA.fill(BB);
  //AA(1,2)=CC;
  return(AA(1,2));
}
```

### Bound check

http://arma.sourceforge.net/docs.html#element_access_bounds_check_note

The bounds checks used by the (n), (i,j) and (i,j,k) access forms can be disabled by defining the ARMA_NO_DEBUG macro before including the armadillo header file (eg. #define ARMA_NO_DEBUG). Disabling the bounds checks is not recommended until your code has been thoroughly debugged -- it's better to write correct code first, and then maximise its speed.

### Other tips

https://scicomp.stackexchange.com/questions/21544/armadillo-library-appears-slow

By Paul Mackenzie

I finally obtained comparable speeds with C++/Armadillo and C/arrays.

Firstly, I switched as suggested by Daniel the order of the loops to column-major order.

Secondly I changed the counters of the loops from int to uword.

Finally I used the function .at() to disable the bound checking.

### Increase Rcpp compile speed

https://stackoverflow.com/questions/31305963/increase-rcpp-compile-speed

By Dirk Eddelbuettel

The best trick I know is to deploy the awesome frontend ccache which most Linux distros have, and which OS X has too (in Brew IIRC). It can be used with both g++ and clang.

So in ~/.R/Makevars I have

```
VER=
CCACHE=ccache
CC=$(CCACHE) gcc$(VER)
CXX=$(CCACHE) g++$(VER)
SHLIB_CXXLD=g++$(VER)
FC=ccache gfortran$(VER)
#FC=gfortran
F77=$(CCACHE) gfortran$(VER)
where VER is currently empty as 4.9 is the default. Now if you re-build the same package over and over, the compile-time is very fast as unchanged code leads to object files being retrieved.
```

To install ccache on MacOS: brew install ccache

Original:
```
FLIBS=-L/usr/local/gfortran/lib/gcc/x86_64-apple-darwin16/6.3.0 -L/usr/local/gfortran/lib -lgfortran -lquadmath -lm
```

Alternatively, use Rcpp11:

http://webcache.googleusercontent.com/search?q=cache:bz78FjVobTMJ:https://www.r-bloggers.com/user2014-rcpp11-tutorial/&hl=en&gl=us&strip=1&vwsrc=0

By Romain Francois

```
devtools::install_github("Rcpp11/Rcpp11")
devtools::install_github("Rcpp11/attributes")
```

### OPENMP
OpenMP flags:
https://gist.github.com/zkamvar/9a7c4b8251a0a662f214
By zkamvar (Zhian N. Kamvar)
