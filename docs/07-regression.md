---
output: html_document
---

# 回帰分析


仮想的に以下のようにデータを生成する.

```r
N <- 100
x<-runif(N)
w<-sample(c("H","T"),N,replace=TRUE)
y <- 10 + 2*x + ifelse(w=="H",1,0) + rnorm(N)
df <- data.frame(w,x,y)
```


```r
plot(y~x)
```

<img src="07-regression_files/figure-html/unnamed-chunk-2-1.png" width="672" />


## 単回帰モデル
次の単回帰モデルを考えてる.
$x$ は説明変数で, 
$y$ は被説明変数である.
$u$ は誤差項である.
パラメータとして $\alpha$ は切片パラメータ, $\beta$ は傾きパラメータである.

$$
y = \alpha + \beta x + u
$$

次の仮定を置いている.

+ $(x_i,y_i)$ は独立同一分布にしたがう.
+ $E[u_i]=0$ である.
+ $u_i$ と $x_i$ は独立である.
+ $u_i$ は正規分布にしたがう.

このとき最小二乗推定量は一致で, 不偏であり, 正規分布にしたがう.
一致とは推定量が観測値を増やすことによって真のパラメータに (確率) 収束することある.
不偏とは推定量の期待値が真のパラメータになることである.
また他の線形不偏推定量のなかで最も分散が小さいことも知られている.


R で回帰分析を実施するには `lm` を実施する.

```r
fm <- lm(y ~ x, data=df)
```

`fm` 自体はリストであり, 以下の要素がある.

```r
typeof(fm)
## [1] "list"
names(fm)
##  [1] "coefficients"  "residuals"     "effects"       "rank"         
##  [5] "fitted.values" "assign"        "qr"            "df.residual"  
##  [9] "xlevels"       "call"          "terms"         "model"
```

係数は次のコマンドを実施する.

```r
coef(fm)
## (Intercept)           x 
<<<<<<< HEAD
##   10.490456    1.967435
=======
##    10.43340     2.27426
>>>>>>> master
# coefficients(fm)
```

作図すると以下のようになる.

```r
plot(y~x,data=df)
abline(fm)
```

<img src="07-regression_files/figure-html/unnamed-chunk-6-1.png" width="672" />


残差は次のコマンドを実施する.

```r
head(resid(fm))
<<<<<<< HEAD
##          1          2          3          4          5          6 
##  0.3559678  0.8515786  0.8909295  0.1643558 -0.5419819  0.1318713
=======
##           1           2           3           4           5           6 
##  0.65672476 -0.42035947  2.28286000  0.26058730 -0.05097468  0.18544409
>>>>>>> master
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
<<<<<<< HEAD
## 12.21255 11.51046 11.64260 12.36637 11.17989 10.67073
=======
## 12.14123 11.41268 11.74830 10.67669 10.68227 10.53695
>>>>>>> master
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
<<<<<<< HEAD
## [1] 142.0024
sum(resid(fm)^2)
## [1] 142.0024
=======
## [1] 106.727
sum(resid(fm)^2)
## [1] 106.727
>>>>>>> master
```

### ティー検定
その結果を見るには `summary` を実施する.

```r
summary(fm)
## 
## Call:
## lm(formula = y ~ x, data = df)
## 
## Residuals:
<<<<<<< HEAD
##      Min       1Q   Median       3Q      Max 
## -2.46097 -0.84156  0.09145  0.74362  3.06475 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.4905     0.2774  37.814  < 2e-16 ***
## x             1.9674     0.4450   4.421 2.54e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.204 on 98 degrees of freedom
## Multiple R-squared:  0.1663,	Adjusted R-squared:  0.1578 
## F-statistic: 19.55 on 1 and 98 DF,  p-value: 2.542e-05
=======
##     Min      1Q  Median      3Q     Max 
## -2.8596 -0.6627  0.0477  0.6917  2.7808 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.4334     0.2176   47.95  < 2e-16 ***
## x             2.2743     0.3548    6.41 5.12e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.044 on 98 degrees of freedom
## Multiple R-squared:  0.2954,	Adjusted R-squared:  0.2882 
## F-statistic: 41.08 on 1 and 98 DF,  p-value: 5.123e-09
>>>>>>> master
```

これをみれば各変数の係数がゼロのティー検定も可能である.
他にも, 残差標準誤差, 決定係数, 修正済み決定係数, 全ての係数がゼロであるF検定も可能である.


`summary(fm)` もリストであり, それぞれの要素は以下である.

```r
typeof(summary(fm))
## [1] "list"
names(summary(fm))
##  [1] "call"          "terms"         "residuals"     "coefficients" 
##  [5] "aliased"       "sigma"         "df"            "r.squared"    
##  [9] "adj.r.squared" "fstatistic"    "cov.unscaled"
```

単なる `fm` と同じ名前もあるが, 中身が違っている場合もある.
例えば `residuals` は同じであるが, 係数はより情報が加わっている.

```r
coef(summary(fm))
<<<<<<< HEAD
##              Estimate Std. Error   t value     Pr(>|t|)
## (Intercept) 10.490456  0.2774195 37.814415 2.939851e-60
## x            1.967435  0.4450165  4.421039 2.542008e-05
=======
##             Estimate Std. Error   t value     Pr(>|t|)
## (Intercept) 10.43340  0.2175904 47.949742 7.565964e-70
## x            2.27426  0.3548123  6.409756 5.123301e-09
>>>>>>> master
# coefficients(summary(fm))
```

残差の標準誤差

```r
sqrt(deviance(fm)/df.residual(fm))
<<<<<<< HEAD
## [1] 1.203746
with(summary(fm),sigma)
## [1] 1.203746
=======
## [1] 1.043576
with(summary(fm),sigma)
## [1] 1.043576
>>>>>>> master
```

決定係数は次のようにして計算する.

```r
1-deviance(fm)/with(df, sum((y-mean(y))^2))
<<<<<<< HEAD
## [1] 0.1662809
with(summary(fm),r.squared)
## [1] 0.1662809
=======
## [1] 0.2953948
with(summary(fm),r.squared)
## [1] 0.2953948
>>>>>>> master
```

調整済み決定係数は次のようにして計算する.

```r
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
<<<<<<< HEAD
## [1] 0.1577735
with(summary(fm),adj.r.squared)
## [1] 0.1577735
=======
## [1] 0.2882049
with(summary(fm),adj.r.squared)
## [1] 0.2882049
>>>>>>> master
```



### 対数変換
次のモデルを考える.
$$
y = \alpha + \beta \log(x) + u
$$

対数変換をおこなう場合, `log` を用いるとよい.

```r
fm <- lm(y~log(x),data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ log(x), data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
<<<<<<< HEAD
## -2.6049 -0.8300  0.0647  0.8116  3.1670 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  12.0650     0.1791  67.382  < 2e-16 ***
## log(x)        0.6105     0.1682   3.629 0.000454 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.238 on 98 degrees of freedom
## Multiple R-squared:  0.1185,	Adjusted R-squared:  0.1095 
## F-statistic: 13.17 on 1 and 98 DF,  p-value: 0.0004543
=======
## -3.2657 -0.7096  0.0201  0.8088  2.9774 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  12.0906     0.1501  80.565  < 2e-16 ***
## log(x)        0.4666     0.1057   4.416 2.59e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.135 on 98 degrees of freedom
## Multiple R-squared:  0.1659,	Adjusted R-squared:  0.1574 
## F-statistic:  19.5 on 1 and 98 DF,  p-value: 2.595e-05
>>>>>>> master
```

作図すると以下のようになる.

```r
plot(y~log(x),data=df)
abline(fm)
```

<img src="07-regression_files/figure-html/unnamed-chunk-17-1.png" width="672" />

被説明変数が対数の場合も同様である.

```r
fm <- lm(log(y)~x,data=df)
summary(fm)
## 
## Call:
## lm(formula = log(y) ~ x, data = df)
## 
## Residuals:
<<<<<<< HEAD
##      Min       1Q   Median       3Q      Max 
## -0.23371 -0.07044  0.01157  0.06917  0.23284 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.34828    0.02416  97.211  < 2e-16 ***
## x            0.17080    0.03875   4.408 2.68e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1048 on 98 degrees of freedom
## Multiple R-squared:  0.1654,	Adjusted R-squared:  0.1569 
## F-statistic: 19.43 on 1 and 98 DF,  p-value: 2.676e-05
=======
##       Min        1Q    Median        3Q       Max 
## -0.292214 -0.056380  0.007969  0.062166  0.233327 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.34548    0.01887  124.28  < 2e-16 ***
## x            0.19481    0.03077    6.33 7.39e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09051 on 98 degrees of freedom
## Multiple R-squared:  0.2902,	Adjusted R-squared:  0.283 
## F-statistic: 40.07 on 1 and 98 DF,  p-value: 7.389e-09
>>>>>>> master
```

作図すると以下のようになる.

```r
plot(log(y)~x,data=df)
abline(fm)
```

<img src="07-regression_files/figure-html/unnamed-chunk-19-1.png" width="672" />



### 切片なし回帰モデル
次のモデルを考える.
$$
y = \beta x + u
$$

切片なしモデルを推定したい場合は次のように `-1` とする.

```r
fm <- lm(y~x-1,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x - 1, data = df)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
<<<<<<< HEAD
## -6.405 -1.312  1.665  4.910 11.435 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.1288     0.7586   22.58   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.729 on 99 degrees of freedom
## Multiple R-squared:  0.8374,	Adjusted R-squared:  0.8358 
## F-statistic: 509.8 on 1 and 99 DF,  p-value: < 2.2e-16
=======
## -5.879 -1.034  1.946  5.846 11.680 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.2030     0.8374   20.54   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.135 on 99 degrees of freedom
## Multiple R-squared:   0.81,	Adjusted R-squared:  0.8081 
## F-statistic: 422.1 on 1 and 99 DF,  p-value: < 2.2e-16
>>>>>>> master
```

もしくは `+0` を加える.

```r
fm <- lm(y~x+0,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + 0, data = df)
## 
## Residuals:
##    Min     1Q Median     3Q    Max 
<<<<<<< HEAD
## -6.405 -1.312  1.665  4.910 11.435 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.1288     0.7586   22.58   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.729 on 99 degrees of freedom
## Multiple R-squared:  0.8374,	Adjusted R-squared:  0.8358 
## F-statistic: 509.8 on 1 and 99 DF,  p-value: < 2.2e-16
=======
## -5.879 -1.034  1.946  5.846 11.680 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.2030     0.8374   20.54   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.135 on 99 degrees of freedom
## Multiple R-squared:   0.81,	Adjusted R-squared:  0.8081 
## F-statistic: 422.1 on 1 and 99 DF,  p-value: < 2.2e-16
>>>>>>> master
```



## 重回帰モデル
説明変数として $w$ を加えたモデルを考える.
$$
y = \alpha + \beta x +\gamma w+ u
$$

暗黙に次の仮定を置いている.

+ $(w_i, x_i,y_i)$ は独立同一分布にしたがう.
+ 誤差項の期待値はゼロである. $E[u_i]=0$ である.
+ 誤差項 $u_i$ は説明変数 $(x_i, w_i)$ に対して独立である.
+ 誤差項 $u_i$ は正規分布にしたがう.
+ 説明変数間に多重共線性は存在しない. つまり $x_i$ は $w_i$ の一次変換で表せない.

説明変数を加えたいときには `+` と変数名を使うことができる. 

```r
fm <- lm(y~x+w,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + w, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
<<<<<<< HEAD
## -1.89724 -0.71736 -0.02745  0.68874  2.50285 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0256     0.2653  41.561  < 2e-16 ***
## x             2.0021     0.3937   5.085 1.79e-06 ***
## wT           -1.1319     0.2130  -5.313 6.88e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.065 on 97 degrees of freedom
## Multiple R-squared:  0.3542,	Adjusted R-squared:  0.3409 
## F-statistic:  26.6 on 2 and 97 DF,  p-value: 6.164e-10
=======
## -2.42177 -0.65472  0.04092  0.64626  2.22152 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0114     0.2245  49.054  < 2e-16 ***
## x             2.0914     0.3184   6.568 2.53e-09 ***
## wT           -0.9593     0.1873  -5.122 1.54e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9306 on 97 degrees of freedom
## Multiple R-squared:  0.4454,	Adjusted R-squared:  0.4339 
## F-statistic: 38.95 on 2 and 97 DF,  p-value: 3.837e-13
>>>>>>> master
```

R の特徴は因子もとくに変換することなくダミー変数として扱える.

### 自乗項
説明変数として自乗項を加えたモデルを考える.
$$
y = \alpha + \beta x + \gamma x^2 + u
$$

R では単に自乗するのでなく `I(x^2)` としなければならない.

```r
fm <- lm(y~x+I(x^2),data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + I(x^2), data = df)
## 
## Residuals:
<<<<<<< HEAD
##     Min      1Q  Median      3Q     Max 
## -2.5974 -0.7907  0.0295  0.7883  3.0848 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.7595     0.4558  23.606   <2e-16 ***
## x             0.5848     1.9089   0.306    0.760    
## I(x^2)        1.3060     1.7532   0.745    0.458    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.206 on 97 degrees of freedom
## Multiple R-squared:  0.171,	Adjusted R-squared:  0.1539 
## F-statistic: 10.01 on 2 and 97 DF,  p-value: 0.000112
=======
##      Min       1Q   Median       3Q      Max 
## -2.82747 -0.65000  0.03918  0.69146  2.72676 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5520     0.3281  32.160   <2e-16 ***
## x             1.5751     1.4865   1.060    0.292    
## I(x^2)        0.6850     1.4140   0.484    0.629    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.048 on 97 degrees of freedom
## Multiple R-squared:  0.2971,	Adjusted R-squared:  0.2826 
## F-statistic:  20.5 on 2 and 97 DF,  p-value: 3.754e-08
>>>>>>> master
```

### 交差項
説明変数として交差項を加えたモデルを考える.
$$
y = \alpha + \beta x + \gamma w + \delta xw + u
$$

説明変数 `x` と `w`の自乗項は自乗項は `x:w` である.

```r
fm<-lm(y~x+w+x:w,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + w + x:w, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
<<<<<<< HEAD
## -1.86445 -0.71417 -0.05085  0.68808  2.48555 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9807     0.3423  32.076  < 2e-16 ***
## x             2.0826     0.5523   3.771 0.000281 ***
## wT           -1.0389     0.4936  -2.104 0.037952 *  
## x:wT         -0.1656     0.7917  -0.209 0.834795    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.07 on 96 degrees of freedom
## Multiple R-squared:  0.3545,	Adjusted R-squared:  0.3343 
## F-statistic: 17.57 on 3 and 96 DF,  p-value: 3.584e-09
=======
## -2.43067 -0.65038  0.03755  0.64841  2.24452 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 10.98339    0.29496  37.236  < 2e-16 ***
## x            2.14055    0.46164   4.637 1.12e-05 ***
## wT          -0.90829    0.39347  -2.308   0.0231 *  
## x:wT        -0.09457    0.64056  -0.148   0.8829    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9354 on 96 degrees of freedom
## Multiple R-squared:  0.4455,	Adjusted R-squared:  0.4282 
## F-statistic: 25.71 on 3 and 96 DF,  p-value: 2.714e-12
>>>>>>> master
```

もしくは以下のように `*` を使って簡便的に表せる.

```r
fm <- lm(y~x*w,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x * w, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
<<<<<<< HEAD
## -1.86445 -0.71417 -0.05085  0.68808  2.48555 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9807     0.3423  32.076  < 2e-16 ***
## x             2.0826     0.5523   3.771 0.000281 ***
## wT           -1.0389     0.4936  -2.104 0.037952 *  
## x:wT         -0.1656     0.7917  -0.209 0.834795    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.07 on 96 degrees of freedom
## Multiple R-squared:  0.3545,	Adjusted R-squared:  0.3343 
## F-statistic: 17.57 on 3 and 96 DF,  p-value: 3.584e-09
=======
## -2.43067 -0.65038  0.03755  0.64841  2.24452 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 10.98339    0.29496  37.236  < 2e-16 ***
## x            2.14055    0.46164   4.637 1.12e-05 ***
## wT          -0.90829    0.39347  -2.308   0.0231 *  
## x:wT        -0.09457    0.64056  -0.148   0.8829    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9354 on 96 degrees of freedom
## Multiple R-squared:  0.4455,	Adjusted R-squared:  0.4282 
## F-statistic: 25.71 on 3 and 96 DF,  p-value: 2.714e-12
>>>>>>> master
```

### エフ検定
今, 帰無仮説が
$$
y = \alpha + \beta x + u
$$
で, 対立仮説が
$$
y = \alpha + \beta x + \gamma w + \delta xw + u
$$
となる検定を実施したい. 

これは複数の係数がゼロであるエフ検定である.
コマンド `anova` を用いればエフ検定できる.

```r
fm0 <- lm(y~x,data=df)
fm1 <- lm(y~x*w,data=df)
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
<<<<<<< HEAD
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 142.00                                  
## 2     96 109.95  2    32.057 13.995 4.638e-06 ***
=======
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 106.727                                  
## 2     96  83.991  2    22.736 12.994 1.014e-05 ***
>>>>>>> master
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

順番を変えても, 検定統計量自体に変更はない.

```r
anova(fm1,fm0)
## Analysis of Variance Table
## 
## Model 1: y ~ x * w
## Model 2: y ~ x
<<<<<<< HEAD
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     96 109.95                                  
## 2     98 142.00 -2   -32.057 13.995 4.638e-06 ***
=======
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     96  83.991                                  
## 2     98 106.727 -2   -22.736 12.994 1.014e-05 ***
>>>>>>> master
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

またモデルの更新として `update` を使えば簡単になる場合もある.

```r
fm1 <- update(fm0, .~. +w + w:x)
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x + w + x:w
<<<<<<< HEAD
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 142.00                                  
## 2     96 109.95  2    32.057 13.995 4.638e-06 ***
=======
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 106.727                                  
## 2     96  83.991  2    22.736 12.994 1.014e-05 ***
>>>>>>> master
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

## 最小二乗推定量のための仮定について
単回帰において以下の仮定を置いていた.

+ $(x_i,y_i)$ は独立同一分布にしたがう.
+ $E[u_i]=0$ である.
+ $u_i$ と $x_i$ は独立である.
+ $u_i$ は正規分布にしたがう.

十分な観測値が得られるばあい, $u_i$ が正規分布にしたがっていないくても, 中心極限定理定理より, 最小二乗法推定量は正規分布に近似できる.

ここの係数ゼロのティー検定について, ライブラリ `AER` を導入して `coeftest` を用いればよい.

```r
library(AER)
##  要求されたパッケージ car をロード中です
##  要求されたパッケージ lmtest をロード中です
##  要求されたパッケージ zoo をロード中です
## 
##  次のパッケージを付け加えます: 'zoo'
##  以下のオブジェクトは 'package:base' からマスクされています: 
## 
##      as.Date, as.Date.numeric
##  要求されたパッケージ sandwich をロード中です
##  要求されたパッケージ survival をロード中です
coeftest(fm0,df=Inf)
## 
## z test of coefficients:
## 
##             Estimate Std. Error z value  Pr(>|z|)    
## (Intercept) 10.43340    0.21759 47.9497 < 2.2e-16 ***
## x            2.27426    0.35481  6.4098 1.458e-10 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
ただ十分なデータのもとではティー値のままでもよい.

同様に複数制約の場合, エフ検定統計量に制約の数を乗じた統計量が
自由度が制約数のカイ二乗分布にしたがうことが知られている.
これをR で実施するには `waldtest` を用いればよい. 

```r
waldtest(fm0,fm1,test="Chisq")
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x + w + x:w
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1     98                         
## 2     96  2 25.988  2.274e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
エフ検定も十分なデータのもとではそのままでよいであろう.

オプション `test` を付けなければエフ検定を実施する.

```r
waldtest(fm0,fm1)
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x + w + x:w
##   Res.Df Df      F    Pr(>F)    
## 1     98                        
## 2     96  2 12.994 1.014e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
これは `anova` と同じである.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x + w + x:w
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 106.727                                  
## 2     96  83.991  2    22.736 12.994 1.014e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


複数制約の検定としてLM検定というのもある.
制約付きの回帰分析を実行し, その残差を制約なしのモデルの説明変数に回帰する.
その決定係数に観測数を掛けた統計量が自由どが制約の数のカイ二乗分布にしたがうことが知られている.

```r
lmt <- lm(I(resid(fm1))~w*x,data=df)
(lmt <- nrow(df)*summary(lmt)$r.squared)
## [1] 5.824083e-30
1-pchisq(lmt,df=1)
## [1] 1
```

### ロバスト分散
また $u_i$ と $x_i$ は独立でなく, $u_i$ と $x_i$ が無相関という弱い条件のもとでも,
一致推定量であることが知られている.
ただ不偏推定量は保証できない. また 線形推定量のなかで最小の分散とも言えない.^[
正確にいえば, 不偏推定量のとめには条件付き期待値が説明変数に依存しないことが必要である. また線形推定量のなかで最小の分散になるためには
条件付き分散が説明変数に依存しないことが必要である. ]
また独立のときの標準誤差の推定量が一致推定量でない.

ただし, 別の分散のもとで正規分布に近似できることがしられている.^[
正確には観測される変数に4次のモーメントが存在するという仮定が必要となる.
この仮定の直感的な意味は異常値が存在しないことである.]
つまり, 説明変数と誤差項が無相関であるが, 独立とまでは言い切れない場合,
最小二乗推定量を実行した際, 別の方法で分散を推定する必要がある.
この別の分散をロバスト分散という.

R でロバスト分散を推定するにはパッケージ `AER` を導入するのが簡単である.
は次のコマンド `coeftest` を実行すればよい.

```r
coeftest(fm1,vcov=vcovHC)
## 
## t test of coefficients:
## 
<<<<<<< HEAD
##             Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.98072    0.36486 30.0953 < 2.2e-16 ***
## x            2.08265    0.58516  3.5591 0.0005805 ***
## wT          -1.03885    0.50292 -2.0656 0.0415578 *  
## x:wT        -0.16555    0.80771 -0.2050 0.8380322    
=======
##              Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.983387   0.285844 38.4245 < 2.2e-16 ***
## x            2.140552   0.502818  4.2571 4.824e-05 ***
## wT          -0.908291   0.367531 -2.4713   0.01522 *  
## x:wT        -0.094574   0.650574 -0.1454   0.88472    
>>>>>>> master
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

先の値と標準誤差が違っていることが確認できるであろう.
ただこの値は STATA と少し異なっている. STATA と同じにするには

```r
coeftest(fm1,vcov=vcovHC(fm1,type="HC1"))
## 
## t test of coefficients:
## 
<<<<<<< HEAD
##             Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.98072    0.34986 31.3856 < 2.2e-16 ***
## x            2.08265    0.56168  3.7079 0.0003498 ***
## wT          -1.03885    0.48467 -2.1434 0.0346056 *  
## x:wT        -0.16555    0.77807 -0.2128 0.8319535    
=======
##              Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.983387   0.272357 40.3271 < 2.2e-16 ***
## x            2.140552   0.481918  4.4417 2.389e-05 ***
## wT          -0.908291   0.353860 -2.5668   0.01181 *  
## x:wT        -0.094574   0.626653 -0.1509   0.88036    
>>>>>>> master
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
としなければならない.

またティー分布でなく正規分布とすることもできる.

```r
coeftest(fm0,vcov=vcovHC,df=Inf)
## 
## z test of coefficients:
## 
##             Estimate Std. Error z value  Pr(>|z|)    
## (Intercept) 10.43340    0.19463 53.6073 < 2.2e-16 ***
## x            2.27426    0.33865  6.7157 1.871e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

複数の係数についての検定は `waldtest` を実行すればよい.

```r
waldtest(fm0,fm1,vcov=vcovHC)
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x + w + x:w
##   Res.Df Df      F   Pr(>F)    
## 1     98                       
## 2     96  2 12.937 1.06e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

通常はF検定であるが, カイ二乗検定を実施するには以下を実施すればよい.

```r
waldtest(fm0,fm1,vcov=vcovHC, test="Chisq")
## Wald test
## 
<<<<<<< HEAD
## data:  fm1
## BP = 0.43119, df = 3, p-value = 0.9337
=======
## Model 1: y ~ x
## Model 2: y ~ x + w + x:w
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1     98                         
## 2     96  2 25.875  2.407e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
>>>>>>> master
```


### 分散均一の検定
誤差項が説明変数と独立のときと無相関のときでは標準誤差の推定量が異なる.
正確にいうと, 条件付き分散が説明変数に依存するかどうかによって標準誤差の推定量が異なる. このことは分散均一と呼ばれている.

誤差項の分散が均一かどうかは検定可能である.
有名な検定方法としてBP (Breusch-Pagan) 検定というものがある.
BP検定は帰無仮説が分散均一で, 対立仮説が分散が説明変数と線形関係になっている場合の検定である.

残差の自乗を被説明変数として回帰分析をおこない,
その決定係数に観測数をかけたものが検定統計量となる.

```r
bpt <- lm(I(resid(fm1)^2)~w*x,data=df)
(bpt <- nrow(df)*summary(bpt)$r.squared)
<<<<<<< HEAD
## [1] 0.4311883
1-pchisq(bpt,df=3)
## [1] 0.933727
=======
## [1] 2.346718
1-pchisq(bpt,df=3)
## [1] 0.5036301
```

ここでの例ではP値が5%を超えているので帰無仮説を棄却できないので,
分散均一を仮定してよいことが示唆されている.

R では以下のように実施すればよい.

```r
bptest(fm1)
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
## BP = 2.3467, df = 3, p-value = 0.5036
>>>>>>> master
```

これまでのBPテストは誤差項の分散が説明変数の線形関係あることを暗黙に仮定している.
非線形性を考慮するために説明変数の二次項を導入した分散不均一性の検定をホワイト検定という.
説明変数が複数ある場合ホワイト検定は煩雑になるため, 
被説明変数の予測値を使って計算することがある.
そのときホワイトテストは以下で実施する.

```r
wht <- lm(I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2),data=df)
(wht <- nrow(df)*summary(wht)$r.squared)
<<<<<<< HEAD
## [1] 0.311932
1-pchisq(wht,df=2)
## [1] 0.8555883
=======
## [1] 0.9784255
1-pchisq(wht,df=2)
## [1] 0.6131089
>>>>>>> master
```
ホワイト検定でも分散均一が示唆されている.

もしくは以下を実行する.

```r
bptest(fm1,I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2))
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
## BP = 0.97843, df = 2, p-value = 0.6131
```

このように分散均一性は検定することが可能であるが, そもそも分散均一が疑われる場合は,
ロバスト分散で推定するので十分であるため最近の実証分析ではこの検定は実施されない.



