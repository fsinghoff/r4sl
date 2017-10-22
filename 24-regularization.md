# Regularization

**TODO:** Introduce regularization as a concept.

We will use the `Hitters` dataset from the `ISLR` package to explore two shrinkage methods: **ridge** and **lasso**. These are otherwise known as **penalized regression** methods.


```r
data(Hitters, package = "ISLR")
```

This dataset has some missing data in the response `Salaray`. We use the `na.omit()` function the clean the dataset.


```r
sum(is.na(Hitters))
```

```
## [1] 59
```

```r
sum(is.na(Hitters$Salary))
```

```
## [1] 59
```

```r
Hitters = na.omit(Hitters)
sum(is.na(Hitters))
```

```
## [1] 0
```

The predictors variables are offensive and defensive statistics for a number of baseball players.


```r
names(Hitters)
```

```
##  [1] "AtBat"     "Hits"      "HmRun"     "Runs"      "RBI"      
##  [6] "Walks"     "Years"     "CAtBat"    "CHits"     "CHmRun"   
## [11] "CRuns"     "CRBI"      "CWalks"    "League"    "Division" 
## [16] "PutOuts"   "Assists"   "Errors"    "Salary"    "NewLeague"
```

We use the `glmnet()` and `cv.glmnet()` functions in the `glmnet` package to fit penalized regressions.


```r
# this is a temporary workaround for an issue with glmnet, Matrix, and R version 3.3.3
# see here: http://stackoverflow.com/questions/43282720/r-error-in-validobject-object-when-running-as-script-but-not-in-console
library(methods)
```


```r
library(glmnet)
```

The `glmnet` function does not allow the use of model formulas, so we setup the data for ease of use with `glmnet`.


```r
X = model.matrix(Salary ~ ., Hitters)[, -1]
y = Hitters$Salary
```

First, we fit a regular linear regression, and note the size of the predictors' coefficients, and predictors' coefficients squared. (The two penalties we will use.)


```r
fit = lm(Salary ~ ., Hitters)
coef(fit)
```

```
##  (Intercept)        AtBat         Hits        HmRun         Runs 
##  163.1035878   -1.9798729    7.5007675    4.3308829   -2.3762100 
##          RBI        Walks        Years       CAtBat        CHits 
##   -1.0449620    6.2312863   -3.4890543   -0.1713405    0.1339910 
##       CHmRun        CRuns         CRBI       CWalks      LeagueN 
##   -0.1728611    1.4543049    0.8077088   -0.8115709   62.5994230 
##    DivisionW      PutOuts      Assists       Errors   NewLeagueN 
## -116.8492456    0.2818925    0.3710692   -3.3607605  -24.7623251
```

```r
sum(abs(coef(fit)[-1]))
```

```
## [1] 238.7295
```

```r
sum(coef(fit)[-1] ^ 2)
```

```
## [1] 18337.3
```


## Ridge Regression

We first illustrate **ridge regression**, which can be fit using `glmnet()` with `alpha = 0` and seeks to minimize

$$
\sum_{i=1}^{n} \left( y_i - \beta_0 - \sum_{j=1}^{p} \beta_j x_{ij}    \right) ^ 2 + \lambda \sum_{j=1}^{p} \beta_j^2 .
$$

Notice that the intercept is **not** penalized. Also, note that that ridge regression is **not** scale invariant like the usual unpenalized regression. Thankfully, `glmnet()` takes care of this internally. It automatically standardizes input for fitting, then reports fitted coefficient using the original scale.

The two plots illustrate how much the coefficients are penalized for different values of $\lambda$. Notice none of the coefficients are forced to be zero.


```r
fit_ridge = glmnet(X, y, alpha = 0)
plot(fit_ridge)
```

<img src="24-regularization_files/figure-html/ridge-1.png" width="672" />

```r
plot(fit_ridge, xvar = "lambda", label = TRUE)
```

<img src="24-regularization_files/figure-html/ridge-2.png" width="672" />

```r
dim(coef(fit_ridge))
```

```
## [1]  20 100
```

We use cross-validation to select a good $\lambda$ value. The `cv.glmnet()`function uses 10 folds by default. The plot illustrates the MSE for the $\lambda$s considered. Two lines are drawn. The first is the $\lambda$ that gives the smallest MSE. The second is the $\lambda$ that gives an MSE within one standard error of the smallest.


```r
fit_ridge_cv = cv.glmnet(X, y, alpha = 0)
plot(fit_ridge_cv)
```

<img src="24-regularization_files/figure-html/unnamed-chunk-8-1.png" width="672" />

The `cv.glmnet()` function returns several details of the fit for both $\lambda$ values in the plot. Notice the penalty terms are smaller than the full linear regression. (As we would expect.)


```r
coef(fit_ridge_cv)
```

```
## 20 x 1 sparse Matrix of class "dgCMatrix"
##                         1
## (Intercept) 213.066443434
## AtBat         0.090095728
## Hits          0.371252756
## HmRun         1.180126956
## Runs          0.596298287
## RBI           0.594502390
## Walks         0.772525466
## Years         2.473494238
## CAtBat        0.007597952
## CHits         0.029272172
## CHmRun        0.217335716
## CRuns         0.058705097
## CRBI          0.060722036
## CWalks        0.058698830
## LeagueN       3.276567828
## DivisionW   -21.889942619
## PutOuts       0.052667119
## Assists       0.007463678
## Errors       -0.145121336
## NewLeagueN    2.972759126
```

```r
coef(fit_ridge_cv, s = "lambda.min")
```

```
## 20 x 1 sparse Matrix of class "dgCMatrix"
##                         1
## (Intercept)   71.78758429
## AtBat         -0.58269657
## Hits           2.51715272
## HmRun         -1.39973428
## Runs           1.07259572
## RBI            0.74825248
## Walks          3.17950553
## Years         -8.35976899
## CAtBat         0.00133718
## CHits          0.12772556
## CHmRun         0.68074413
## CRuns          0.27080732
## CRBI           0.24581306
## CWalks        -0.24120197
## LeagueN       51.41107146
## DivisionW   -121.93563378
## PutOuts        0.26073685
## Assists        0.15595798
## Errors        -3.59749877
## NewLeagueN   -15.89754187
```

```r
sum(coef(fit_ridge_cv, s = "lambda.min")[-1] ^ 2) # penalty term for lambda minimum
```

```
## [1] 17868.18
```

```r
coef(fit_ridge_cv, s = "lambda.1se")
```

```
## 20 x 1 sparse Matrix of class "dgCMatrix"
##                         1
## (Intercept) 213.066443434
## AtBat         0.090095728
## Hits          0.371252756
## HmRun         1.180126956
## Runs          0.596298287
## RBI           0.594502390
## Walks         0.772525466
## Years         2.473494238
## CAtBat        0.007597952
## CHits         0.029272172
## CHmRun        0.217335716
## CRuns         0.058705097
## CRBI          0.060722036
## CWalks        0.058698830
## LeagueN       3.276567828
## DivisionW   -21.889942619
## PutOuts       0.052667119
## Assists       0.007463678
## Errors       -0.145121336
## NewLeagueN    2.972759126
```

```r
sum(coef(fit_ridge_cv, s = "lambda.1se")[-1] ^ 2) # penalty term for lambda one SE
```

```
## [1] 507.788
```

```r
#predict(fit_ridge_cv, X, s = "lambda.min")
#predict(fit_ridge_cv, X)
mean((y - predict(fit_ridge_cv, X)) ^ 2) # "train error"
```

```
## [1] 132355.6
```

```r
sqrt(fit_ridge_cv$cvm) # CV-RMSEs
```

```
##  [1] 452.7654 450.9923 450.2040 449.9386 449.6484 449.3313 448.9850
##  [8] 448.6069 448.1944 447.7445 447.2542 446.7202 446.1390 445.5069
## [15] 444.8201 444.0744 443.2658 442.3899 441.4422 440.4182 439.3133
## [22] 438.1230 436.8428 435.4685 433.9960 432.4217 430.7423 428.9553
## [29] 427.0589 425.0520 422.9348 420.7084 418.3753 415.9396 413.4067
## [36] 410.7837 408.0795 405.3041 402.4695 399.5892 396.6779 393.7514
## [43] 390.8261 387.9192 385.0475 382.2280 379.4766 376.8083 374.2367
## [50] 371.7732 369.4279 367.2085 365.1209 363.1687 361.3531 359.6739
## [57] 358.1288 356.7141 355.4251 354.2557 353.1996 352.2496 351.3983
## [64] 350.6368 349.9655 349.3736 348.8498 348.3903 347.9882 347.6449
## [71] 347.3505 347.0968 346.8795 346.7014 346.5510 346.4239 346.3214
## [78] 346.2369 346.1689 346.1117 346.0627 346.0215 345.9842 345.9457
## [85] 345.9089 345.8682 345.8281 345.7755 345.7207 345.6527 345.5842
## [92] 345.5018 345.4123 345.3084 345.2008 345.0812 344.9534 344.8178
```

```r
sqrt(fit_ridge_cv$cvm[fit_ridge_cv$lambda == fit_ridge_cv$lambda.min]) # CV-RMSE minimum
```

```
## [1] 344.8178
```

```r
sqrt(fit_ridge_cv$cvm[fit_ridge_cv$lambda == fit_ridge_cv$lambda.1se]) # CV-RMSE one SE
```

```
## [1] 371.7732
```


## Lasso

We now illustrate **lasso**, which can be fit using `glmnet()` with `alpha = 1` and seeks to minimize

$$
\sum_{i=1}^{n} \left( y_i - \beta_0 - \sum_{j=1}^{p} \beta_j x_{ij}    \right) ^ 2 + \lambda \sum_{j=1}^{p} |\beta_j| .
$$

Like ridge, lasso is not scale invariant.

The two plots illustrate how much the coefficients are penalized for different values of $\lambda$. Notice some of the coefficients are forced to be zero.


```r
fit_lasso = glmnet(X, y, alpha = 1)
plot(fit_lasso)
```

<img src="24-regularization_files/figure-html/lasso-1.png" width="672" />

```r
plot(fit_lasso, xvar = "lambda", label = TRUE)
```

<img src="24-regularization_files/figure-html/lasso-2.png" width="672" />

```r
dim(coef(fit_lasso))
```

```
## [1] 20 80
```

Again, to actually pick a $\lambda$, we will use cross-validation. The plot is similar to the ridge plot. Notice along the top is the number of features in the model. (Which changed in this plot.)


```r
fit_lasso_cv = cv.glmnet(X, y, alpha = 1)
plot(fit_lasso_cv)
```

<img src="24-regularization_files/figure-html/unnamed-chunk-10-1.png" width="672" />

`cv.glmnet()` returns several details of the fit for both $\lambda$ values in the plot. Notice the penalty terms are again smaller than the full linear regression. (As we would expect.) Some coefficients are 0.


```r
coef(fit_lasso_cv)
```

```
## 20 x 1 sparse Matrix of class "dgCMatrix"
##                        1
## (Intercept) 167.91202818
## AtBat         .         
## Hits          1.29269756
## HmRun         .         
## Runs          .         
## RBI           .         
## Walks         1.39817511
## Years         .         
## CAtBat        .         
## CHits         .         
## CHmRun        .         
## CRuns         0.14167760
## CRBI          0.32192558
## CWalks        .         
## LeagueN       .         
## DivisionW     .         
## PutOuts       0.04675463
## Assists       .         
## Errors        .         
## NewLeagueN    .
```

```r
coef(fit_lasso_cv, s = "lambda.min")
```

```
## 20 x 1 sparse Matrix of class "dgCMatrix"
##                        1
## (Intercept)  123.7520756
## AtBat         -1.5473426
## Hits           5.6608972
## HmRun          .        
## Runs           .        
## RBI            .        
## Walks          4.7296908
## Years         -9.5958375
## CAtBat         .        
## CHits          .        
## CHmRun         0.5108207
## CRuns          0.6594856
## CRBI           0.3927505
## CWalks        -0.5291586
## LeagueN       32.0650811
## DivisionW   -119.2990171
## PutOuts        0.2724045
## Assists        0.1732025
## Errors        -2.0585083
## NewLeagueN     .
```

```r
sum(abs(coef(fit_lasso_cv, s = "lambda.min")[-1])) # penalty term for lambda minimum
```

```
## [1] 177.4942
```

```r
coef(fit_lasso_cv, s = "lambda.1se")
```

```
## 20 x 1 sparse Matrix of class "dgCMatrix"
##                        1
## (Intercept) 167.91202818
## AtBat         .         
## Hits          1.29269756
## HmRun         .         
## Runs          .         
## RBI           .         
## Walks         1.39817511
## Years         .         
## CAtBat        .         
## CHits         .         
## CHmRun        .         
## CRuns         0.14167760
## CRBI          0.32192558
## CWalks        .         
## LeagueN       .         
## DivisionW     .         
## PutOuts       0.04675463
## Assists       .         
## Errors        .         
## NewLeagueN    .
```

```r
sum(abs(coef(fit_lasso_cv, s = "lambda.1se")[-1])) # penalty term for lambda one SE
```

```
## [1] 3.20123
```

```r
#predict(fit_lasso_cv, X, s = "lambda.min")
#predict(fit_lasso_cv, X)
mean((y - predict(fit_lasso_cv, X)) ^ 2) # "train error"
```

```
## [1] 123931.3
```

```r
sqrt(fit_lasso_cv$cvm)
```

```
##  [1] 450.8003 442.8562 432.4240 423.5952 415.2621 406.5620 398.1843
##  [8] 390.9293 384.9755 379.9812 375.7765 372.2209 368.9755 365.8844
## [15] 362.9606 359.9936 357.1508 354.7424 352.7400 351.0457 349.6077
## [22] 348.4232 347.4475 346.6485 345.9918 345.4503 345.0115 344.7254
## [29] 344.5411 344.4334 344.3728 344.3420 344.3228 344.3321 344.4195
## [36] 344.7930 345.4617 346.0350 346.2109 345.7920 345.0573 344.0683
## [43] 343.0567 342.2926 341.7547 341.3876 341.1247 340.9404 340.8108
## [50] 340.7421 340.7472 340.8184 340.9044 340.9414 340.9061 340.8793
## [57] 340.8565 340.8614 340.8355 340.8175 340.8717 340.9660 341.1123
## [64] 341.2479 341.3776 341.5175 341.6573 341.7958 341.9202 342.0316
## [71] 342.1347 342.2377 342.3682
```

```r
sqrt(fit_lasso_cv$cvm[fit_lasso_cv$lambda == fit_lasso_cv$lambda.min]) # CV-RMSE minimum
```

```
## [1] 340.7421
```

```r
sqrt(fit_lasso_cv$cvm[fit_lasso_cv$lambda == fit_lasso_cv$lambda.1se]) # CV-RMSE one SE
```

```
## [1] 368.9755
```

## `broom`

Sometimes, the output from `glmnet()` can be overwhelming. The `broom` package can help with that.


```r
library(broom)
#fit_lasso_cv
tidy(fit_lasso_cv)
```

```
##         lambda estimate std.error conf.high  conf.low nzero
## 1  255.2820965 203220.9  26977.16  230198.0 176243.72     0
## 2  232.6035386 196121.6  26766.24  222887.9 169355.38     1
## 3  211.9396813 186990.5  25453.77  212444.3 161536.72     2
## 4  193.1115442 179432.9  24311.77  203744.7 155121.15     2
## 5  175.9560468 172442.6  23361.66  195804.3 149080.95     3
## 6  160.3245966 165292.6  22563.43  187856.1 142729.20     4
## 7  146.0818013 158550.7  21873.44  180424.2 136677.30     4
## 8  133.1042967 152825.7  21304.24  174130.0 131521.50     4
## 9  121.2796778 148206.2  20801.05  169007.2 127405.11     4
## 10 110.5055255 144385.7  20397.63  164783.4 123988.11     4
## 11 100.6885192 141208.0  20086.97  161294.9 121121.01     5
## 12  91.7436287 138548.4  19853.64  158402.1 118694.77     5
## 13  83.5933775 136142.9  19691.46  155834.4 116451.49     5
## 14  76.1671723 133871.4  19610.91  153482.3 114260.50     5
## 15  69.4006906 131740.4  19581.16  151321.5 112159.22     6
## 16  63.2353245 129595.4  19570.22  149165.6 110025.19     6
## 17  57.6176726 127556.7  19576.15  147132.8 107980.52     6
## 18  52.4990774 125842.1  19608.70  145450.9 106233.44     6
## 19  47.8352040 124425.5  19662.78  144088.3 104762.73     6
## 20  43.5856563 123233.1  19729.17  142962.3 103503.93     6
## 21  39.7136268 122225.5  19803.06  142028.6 102422.49     6
## 22  36.1855776 121398.8  19889.03  141287.8 101509.73     6
## 23  32.9709506 120719.7  19978.87  140698.6 100740.87     6
## 24  30.0419022 120165.2  20068.62  140233.8 100096.56     6
## 25  27.3730624 119710.3  20157.08  139867.4  99553.25     6
## 26  24.9413150 119335.9  20242.43  139578.3  99093.46     6
## 27  22.7255973 119033.0  20324.17  139357.1  98708.79     6
## 28  20.7067179 118835.6  20397.77  139233.4  98437.81     6
## 29  18.8671902 118708.6  20464.07  139172.6  98244.49     6
## 30  17.1910810 118634.4  20522.12  139156.5  98112.26     7
## 31  15.6638727 118592.6  20575.22  139167.9  98017.43     7
## 32  14.2723374 118571.4  20625.54  139196.9  97945.85     7
## 33  13.0044223 118558.2  20672.55  139230.8  97885.67     9
## 34  11.8491453 118564.6  20719.62  139284.2  97844.97     9
## 35  10.7964999 118624.8  20769.21  139394.0  97855.56     9
## 36   9.8373686 118882.2  20840.33  139722.5  98041.87     9
## 37   8.9634439 119343.8  20941.61  140285.4  98402.14     9
## 38   8.1671562 119740.2  21032.12  140772.3  98708.08    11
## 39   7.4416086 119862.0  21034.93  140896.9  98827.05    11
## 40   6.7805166 119572.1  21015.12  140587.3  98557.02    12
## 41   6.1781542 119064.6  20925.69  139990.2  98138.86    12
## 42   5.6293040 118383.0  20732.89  139115.9  97650.11    13
## 43   5.1292121 117687.9  20595.88  138283.7  97091.99    13
## 44   4.6735471 117164.2  20521.06  137685.3  96643.17    13
## 45   4.2583620 116796.3  20460.03  137256.3  96336.26    13
## 46   3.8800609 116545.5  20402.33  136947.8  96143.17    13
## 47   3.5353670 116366.1  20352.30  136718.4  96013.78    13
## 48   3.2212947 116240.4  20297.99  136538.3  95942.38    13
## 49   2.9351238 116152.0  20222.99  136375.0  95929.03    13
## 50   2.6743755 116105.2  20140.40  136245.6  95964.79    13
## 51   2.4367913 116108.7  20073.30  136182.0  96035.36    13
## 52   2.2203135 116157.2  20013.59  136170.8  96143.62    14
## 53   2.0230670 116215.8  19950.49  136166.3  96265.33    15
## 54   1.8433433 116241.0  19854.17  136095.2  96386.87    15
## 55   1.6795857 116217.0  19738.30  135955.3  96478.70    17
## 56   1.5303760 116198.7  19649.42  135848.1  96549.28    17
## 57   1.3944216 116183.2  19578.76  135761.9  96604.41    17
## 58   1.2705450 116186.5  19518.85  135705.3  96667.65    17
## 59   1.1576733 116168.8  19470.76  135639.6  96698.08    17
## 60   1.0548288 116156.6  19428.26  135584.8  96728.33    17
## 61   0.9611207 116193.5  19386.36  135579.9  96807.14    17
## 62   0.8757374 116257.8  19342.74  135600.5  96915.06    17
## 63   0.7979393 116357.6  19298.75  135656.4  97058.87    17
## 64   0.7270526 116450.1  19256.57  135706.7  97193.57    17
## 65   0.6624632 116538.7  19225.30  135764.0  97313.36    18
## 66   0.6036118 116634.2  19198.57  135832.8  97435.64    18
## 67   0.5499886 116729.7  19176.35  135906.1  97553.35    18
## 68   0.5011291 116824.3  19153.12  135977.5  97671.23    17
## 69   0.4566102 116909.4  19126.64  136036.0  97782.76    18
## 70   0.4160462 116985.6  19110.25  136095.9  97875.38    18
## 71   0.3790858 117056.2  19098.41  136154.6  97957.75    18
## 72   0.3454089 117126.6  19085.37  136212.0  98041.27    18
## 73   0.3147237 117216.0  19079.17  136295.1  98136.79    18
```

```r
glance(fit_lasso_cv) # the two lambda values of interest
```

```
##   lambda.min lambda.1se
## 1   2.674375   83.59338
```


## Simulation Study, p > n

Aside from simply shrinking coefficients (ridge) and setting some coefficients to 0 (lasso), penalized regression also has the advantage of being able to handle the $p > n$ case.


```r
set.seed(1234)
n = 1000
p = 5500
X = replicate(p, rnorm(n = n))
beta = c(1, 1, 1, rep(0, 5497))
z = X %*% beta
prob = exp(z) / (1 + exp(z))
y = as.factor(rbinom(length(z), size = 1, prob = prob))
```

We first simulate a classification example where $p > n$.


```r
# glm(y ~ X, family = "binomial")
# will not converge
```

We then use a lasso penalty to fit penalized logistic regression. This minimizes

$$
\sum_{i=1}^{n} L\left(y_i, \beta_0 + \sum_{j=1}^{p} \beta_j x_{ij}\right) + \lambda \sum_{j=1}^{p} |\beta_j|
$$

where $L$ is the appropriate *negative* **log**-likelihood.


```r
library(glmnet)
fit_cv = cv.glmnet(X, y, family = "binomial", alpha = 1)
plot(fit_cv)
```

<img src="24-regularization_files/figure-html/unnamed-chunk-15-1.png" width="672" />


```r
head(coef(fit_cv), n = 10)
```

```
## 10 x 1 sparse Matrix of class "dgCMatrix"
##                      1
## (Intercept) 0.02397452
## V1          0.59674958
## V2          0.56251761
## V3          0.60065105
## V4          .         
## V5          .         
## V6          .         
## V7          .         
## V8          .         
## V9          .
```


```r
fit_cv$nzero
```

```
##  s0  s1  s2  s3  s4  s5  s6  s7  s8  s9 s10 s11 s12 s13 s14 s15 s16 s17 
##   0   2   3   3   3   3   3   3   3   3   3   3   3   3   3   3   3   3 
## s18 s19 s20 s21 s22 s23 s24 s25 s26 s27 s28 s29 s30 s31 s32 s33 s34 s35 
##   3   3   3   3   3   3   3   3   3   3   3   3   4   6   7  10  18  24 
## s36 s37 s38 s39 s40 s41 s42 s43 s44 s45 s46 s47 s48 s49 s50 s51 s52 s53 
##  35  54  65  75  86 100 110 129 147 168 187 202 221 241 254 269 283 298 
## s54 s55 s56 s57 s58 s59 s60 s61 s62 s63 s64 s65 s66 s67 s68 s69 s70 s71 
## 310 324 333 350 364 375 387 400 411 429 435 445 453 455 462 466 475 481 
## s72 s73 s74 s75 s76 s77 s78 s79 s80 s81 s82 s83 s84 s85 s86 s87 s88 s89 
## 487 491 496 498 502 504 512 518 523 526 528 536 543 550 559 561 563 566 
## s90 s91 s92 s93 s94 s95 s96 s97 s98 
## 570 571 576 582 586 590 596 596 600
```

Notice, only the first three predictors generated are truly significant, and that is exactly what the suggested model finds.


```r
fit_1se = glmnet(X, y, family = "binomial", lambda = fit_cv$lambda.1se)
which(as.vector(as.matrix(fit_1se$beta)) != 0)
```

```
## [1] 1 2 3
```

We can also see in the following plots, the three features entering the model well ahead of the irrelevant features.


```r
plot(glmnet(X, y, family = "binomial"))
```

<img src="24-regularization_files/figure-html/unnamed-chunk-19-1.png" width="672" />


```r
plot(glmnet(X, y, family = "binomial"), xvar = "lambda")
```

<img src="24-regularization_files/figure-html/unnamed-chunk-20-1.png" width="672" />

We can extract the two relevant $\lambda$ values.


```r
fit_cv$lambda.min
```

```
## [1] 0.03087158
```

```r
fit_cv$lambda.1se
```

```
## [1] 0.0514969
```

Since `cv.glmnet()` does not calculate prediction accuracy for classification, we take the $\lambda$ values and create a grid for `caret` to search in order to obtain prediction accuracy with `train()`. We set $\alpha = 1$ in this grid, as `glmnet` can actually tune over the $\alpha = 1$ parameter. (More on that later.)

Note that we have to force `y` to be a factor, so that `train()` recognizes we want to have a binomial response. The `train()` function in `caret` use the type of variable in `y` to determine if you want to use `family = "binomial"` or `family = "gaussian"`.


```r
library(caret)
cv_5 = trainControl(method = "cv", number = 5)
lasso_grid = expand.grid(alpha = 1, 
                         lambda = c(fit_cv$lambda.min, fit_cv$lambda.1se))
lasso_grid
```

```
##   alpha     lambda
## 1     1 0.03087158
## 2     1 0.05149690
```


```r
sim_data = data.frame(y, X)
fit_lasso = train(
  y ~ ., data = sim_data,
  method = "glmnet",
  trControl = cv_5,
  tuneGrid = lasso_grid
)
fit_lasso$results
```

```
##   alpha     lambda  Accuracy     Kappa AccuracySD    KappaSD
## 1     1 0.03087158 0.7679304 0.5358028 0.03430230 0.06844656
## 2     1 0.05149690 0.7689003 0.5377583 0.02806941 0.05596114
```


## External Links

- [`glmnet` Web Vingette](https://web.stanford.edu/~hastie/glmnet/glmnet_alpha.html) - Details from the package developers.


## RMarkdown

The RMarkdown file for this chapter can be found [**here**](15-shrink.Rmd). The file was created using `R` version 3.4.2 and the following packages:

- Base Packages, Attached


```
## [1] "methods"   "stats"     "graphics"  "grDevices" "utils"     "datasets" 
## [7] "base"
```

- Additional Packages, Attached


```
## [1] "caret"   "ggplot2" "lattice" "broom"   "glmnet"  "foreach" "Matrix"
```

- Additional Packages, Not Attached


```
##  [1] "Rcpp"         "lubridate"    "tidyr"        "class"       
##  [5] "assertthat"   "rprojroot"    "digest"       "ipred"       
##  [9] "psych"        "R6"           "plyr"         "backports"   
## [13] "stats4"       "evaluate"     "e1071"        "rlang"       
## [17] "lazyeval"     "kernlab"      "rpart"        "rmarkdown"   
## [21] "splines"      "CVST"         "ddalpha"      "gower"       
## [25] "stringr"      "foreign"      "munsell"      "compiler"    
## [29] "pkgconfig"    "mnormt"       "dimRed"       "htmltools"   
## [33] "nnet"         "tibble"       "prodlim"      "DRR"         
## [37] "bookdown"     "codetools"    "RcppRoll"     "dplyr"       
## [41] "withr"        "MASS"         "recipes"      "ModelMetrics"
## [45] "grid"         "nlme"         "gtable"       "magrittr"    
## [49] "scales"       "stringi"      "reshape2"     "bindrcpp"    
## [53] "timeDate"     "robustbase"   "lava"         "iterators"   
## [57] "tools"        "glue"         "DEoptimR"     "purrr"       
## [61] "sfsmisc"      "parallel"     "survival"     "yaml"        
## [65] "colorspace"   "knitr"        "bindr"
```