# Rcpp_tricks
Rcpp tricks 骚操作收集

```
#include <RcppArmadillo.h>
using namespace Rcpp;
//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
arma::mat type_conversion(arma::imat ttij){
  arma::mat ttij_double=arma::conv_to<arma::mat>::from(ttij);
  return(ttij_double*0.001);
}

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
