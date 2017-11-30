---
output: 
  html_document:default
  html_notebook:default
---

# 回帰分析



```r
library(AER)
```


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

<img src="07-regression_files/figure-html/unnamed-chunk-3-1.png" width="672" />


## 単回帰モデル
次の単回帰モデルを考えてる.

$$
y = \alpha + \beta x + u
$$
ここで $x$ は説明変数で,  $y$ は被説明変数である. $u$ は誤差項である.
パラメータとして $\alpha$ は切片パラメータ, $\beta$ は傾きパラメータである.


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
##   10.738046    1.581447
=======
##   10.397333    2.149987
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
# coefficients(fm)
```

作図すると以下のようになる.

```r
plot(y~x,data=df)
abline(fm)
```

<img src="07-regression_files/figure-html/unnamed-chunk-7-1.png" width="672" />


残差は次のコマンドを実施する.

```r
head(resid(fm))
<<<<<<< HEAD
##          1          2          3          4          5          6 
##  0.1737704  0.1509398 -1.4426625  1.1363425  1.1765654  1.8869672
=======
##           1           2           3           4           5           6 
## -0.38750187  0.21272837 -0.71110924 -0.02071343 -1.60459722  1.14570350
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
<<<<<<< HEAD
## 12.30522 12.28274 12.02640 12.06192 10.98926 12.08435
=======
## 11.47536 11.83611 12.26635 11.75227 11.05161 11.22739
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
<<<<<<< HEAD
## [1] 124.8311
sum(resid(fm)^2)
## [1] 124.8311
=======
## [1] 125.9439
sum(resid(fm)^2)
## [1] 125.9439
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
##     Min      1Q  Median      3Q     Max 
<<<<<<< HEAD
## -2.7884 -0.5907  0.1017  0.7619  2.4715 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.7380     0.2268  47.342  < 2e-16 ***
## x             1.5814     0.3750   4.217 5.51e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.129 on 98 degrees of freedom
## Multiple R-squared:  0.1536,	Adjusted R-squared:  0.145 
## F-statistic: 17.79 on 1 and 98 DF,  p-value: 5.514e-05
=======
## -3.3680 -0.6791 -0.0224  0.6166  3.3302 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.3973     0.2215  46.930  < 2e-16 ***
## x             2.1500     0.4071   5.281 7.75e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.134 on 98 degrees of freedom
## Multiple R-squared:  0.2215,	Adjusted R-squared:  0.2136 
## F-statistic: 27.89 on 1 and 98 DF,  p-value: 7.747e-07
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

これをみれば各変数の係数がゼロのティー検定の結果が示されている.
他にも, 残差標準誤差, 決定係数, 修正済み決定係数, 全ての係数がゼロであるエフ検定の結果も示されている.

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
##              Estimate Std. Error   t value     Pr(>|t|)
<<<<<<< HEAD
## (Intercept) 10.738046  0.2268173 47.342281 2.507648e-69
## x            1.581447  0.3749778  4.217443 5.513746e-05
=======
## (Intercept) 10.397333  0.2215496 46.930044 5.700644e-69
## x            2.149987  0.4071202  5.280965 7.746684e-07
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
# coefficients(summary(fm))
```

残差の標準誤差は次のようにして得られる.

```r
sqrt(deviance(fm)/df.residual(fm))
<<<<<<< HEAD
## [1] 1.128621
with(summary(fm),sigma)
## [1] 1.128621
=======
## [1] 1.133641
with(summary(fm),sigma)
## [1] 1.133641
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

決定係数は次のようにして計算する.

```r
1-deviance(fm)/with(df, sum((y-mean(y))^2))
<<<<<<< HEAD
## [1] 0.153617
with(summary(fm),r.squared)
## [1] 0.153617
=======
## [1] 0.2215339
with(summary(fm),r.squared)
## [1] 0.2215339
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

調整済み決定係数は次のようにして計算する.

```r
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
<<<<<<< HEAD
## [1] 0.1449804
with(summary(fm),adj.r.squared)
## [1] 0.1449804
=======
## [1] 0.2135903
with(summary(fm),adj.r.squared)
## [1] 0.2135903
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## -3.0135 -0.6567  0.1098  0.7037  2.4979 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.9601     0.1549  77.227  < 2e-16 ***
## log(x)        0.4023     0.1068   3.768 0.000281 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.147 on 98 degrees of freedom
## Multiple R-squared:  0.1265,	Adjusted R-squared:  0.1176 
## F-statistic:  14.2 on 1 and 98 DF,  p-value: 0.0002814
=======
## -3.4222 -0.7148 -0.0382  0.5530  3.1086 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.9739     0.1826  65.562  < 2e-16 ***
## log(x)        0.5473     0.1330   4.115 8.08e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.186 on 98 degrees of freedom
## Multiple R-squared:  0.1473,	Adjusted R-squared:  0.1386 
## F-statistic: 16.93 on 1 and 98 DF,  p-value: 8.083e-05
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

作図すると以下のようになる.

```r
plot(y~log(x),data=df)
abline(fm)
```

<img src="07-regression_files/figure-html/unnamed-chunk-18-1.png" width="672" />

被説明変数が対数の場合も同様である.

```r
fm <- lm(log(y)~x,data=df)
summary(fm)
## 
## Call:
## lm(formula = log(y) ~ x, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
<<<<<<< HEAD
## -0.28289 -0.04638  0.01334  0.07175  0.19481 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.37024    0.02026 116.974  < 2e-16 ***
## x            0.13774    0.03350   4.112 8.17e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1008 on 98 degrees of freedom
## Multiple R-squared:  0.1471,	Adjusted R-squared:  0.1384 
## F-statistic: 16.91 on 1 and 98 DF,  p-value: 8.167e-05
=======
## -0.33693 -0.05530  0.00347  0.05720  0.27580 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.33795    0.01982 117.984  < 2e-16 ***
## x            0.19149    0.03641   5.259 8.52e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1014 on 98 degrees of freedom
## Multiple R-squared:  0.2201,	Adjusted R-squared:  0.2121 
## F-statistic: 27.65 on 1 and 98 DF,  p-value: 8.516e-07
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

作図すると以下のようになる.

```r
plot(log(y)~x,data=df)
abline(fm)
```

<img src="07-regression_files/figure-html/unnamed-chunk-20-1.png" width="672" />



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
## -6.256 -1.438  2.284  6.465 11.878 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   16.980      0.907   18.72   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.486 on 99 degrees of freedom
## Multiple R-squared:  0.7797,	Adjusted R-squared:  0.7775 
## F-statistic: 350.5 on 1 and 99 DF,  p-value: < 2.2e-16
=======
## -5.906 -1.153  3.030  6.469 12.357 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   18.565      1.004   18.49   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.465 on 99 degrees of freedom
## Multiple R-squared:  0.7754,	Adjusted R-squared:  0.7731 
## F-statistic: 341.8 on 1 and 99 DF,  p-value: < 2.2e-16
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## -6.256 -1.438  2.284  6.465 11.878 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   16.980      0.907   18.72   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.486 on 99 degrees of freedom
## Multiple R-squared:  0.7797,	Adjusted R-squared:  0.7775 
## F-statistic: 350.5 on 1 and 99 DF,  p-value: < 2.2e-16
=======
## -5.906 -1.153  3.030  6.469 12.357 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   18.565      1.004   18.49   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.465 on 99 degrees of freedom
## Multiple R-squared:  0.7754,	Adjusted R-squared:  0.7731 
## F-statistic: 341.8 on 1 and 99 DF,  p-value: < 2.2e-16
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## -2.37602 -0.65665 -0.02924  0.70968  2.41506 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0669     0.2257  49.040  < 2e-16 ***
## x             1.7755     0.3516   5.050 2.07e-06 ***
## wT           -0.8613     0.2116  -4.069 9.62e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.048 on 97 degrees of freedom
## Multiple R-squared:  0.277,	Adjusted R-squared:  0.2621 
## F-statistic: 18.58 on 2 and 97 DF,  p-value: 1.469e-07
=======
## -2.78600 -0.57783 -0.08083  0.78883  2.77144 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0320     0.2531  43.587  < 2e-16 ***
## x             1.6575     0.3931   4.216 5.58e-05 ***
## wT           -0.9404     0.2211  -4.253 4.86e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.046 on 97 degrees of freedom
## Multiple R-squared:  0.3439,	Adjusted R-squared:  0.3304 
## F-statistic: 25.42 on 2 and 97 DF,  p-value: 1.328e-09
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
##     Min      1Q  Median      3Q     Max 
<<<<<<< HEAD
## -2.8142 -0.5979  0.1130  0.7669  2.4536 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.6836     0.3405  31.372   <2e-16 ***
## x             1.9150     1.5948   1.201    0.233    
## I(x^2)       -0.3295     1.5306  -0.215    0.830    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.134 on 97 degrees of freedom
## Multiple R-squared:  0.154,	Adjusted R-squared:  0.1366 
## F-statistic:  8.83 on 2 and 97 DF,  p-value: 0.0002999
=======
## -3.3269 -0.6930 -0.0070  0.6476  3.3138 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.4854     0.3453  30.362   <2e-16 ***
## x             1.5960     1.7107   0.933    0.353    
## I(x^2)        0.5773     1.7309   0.334    0.739    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.139 on 97 degrees of freedom
## Multiple R-squared:  0.2224,	Adjusted R-squared:  0.2064 
## F-statistic: 13.87 on 2 and 97 DF,  p-value: 5.023e-06
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## -2.37097 -0.64236 -0.03219  0.72460  2.42515 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0251     0.2792  39.487  < 2e-16 ***
## x             1.8618     0.4880   3.815 0.000241 ***
## wT           -0.7658     0.4290  -1.785 0.077404 .  
## x:wT         -0.1813     0.7074  -0.256 0.798319    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.054 on 96 degrees of freedom
## Multiple R-squared:  0.2775,	Adjusted R-squared:  0.255 
## F-statistic: 12.29 on 3 and 96 DF,  p-value: 7.121e-07
=======
## -2.85078 -0.61710 -0.07475  0.71975  2.70407 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.1263     0.2956  37.640  < 2e-16 ***
## x             1.4824     0.4842   3.062  0.00286 ** 
## wT           -1.1635     0.4211  -2.763  0.00686 ** 
## x:wT          0.5202     0.8345   0.623  0.53456    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.049 on 96 degrees of freedom
## Multiple R-squared:  0.3465,	Adjusted R-squared:  0.3261 
## F-statistic: 16.97 on 3 and 96 DF,  p-value: 6.381e-09
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## -2.37097 -0.64236 -0.03219  0.72460  2.42515 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0251     0.2792  39.487  < 2e-16 ***
## x             1.8618     0.4880   3.815 0.000241 ***
## wT           -0.7658     0.4290  -1.785 0.077404 .  
## x:wT         -0.1813     0.7074  -0.256 0.798319    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.054 on 96 degrees of freedom
## Multiple R-squared:  0.2775,	Adjusted R-squared:  0.255 
## F-statistic: 12.29 on 3 and 96 DF,  p-value: 7.121e-07
=======
## -2.85078 -0.61710 -0.07475  0.71975  2.70407 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.1263     0.2956  37.640  < 2e-16 ***
## x             1.4824     0.4842   3.062  0.00286 ** 
## wT           -1.1635     0.4211  -2.763  0.00686 ** 
## x:wT          0.5202     0.8345   0.623  0.53456    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.049 on 96 degrees of freedom
## Multiple R-squared:  0.3465,	Adjusted R-squared:  0.3261 
## F-statistic: 16.97 on 3 and 96 DF,  p-value: 6.381e-09
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
対立仮説の残差自乗和を $SSR$ とし, その自由度を $df$ とする.
自由度は観測数から説明変数の数を減じた数である.
帰無仮説の残差自乗和を $SSR_0$ とし, 制約の数を $q$ とする.
制約の数は帰無仮説の自由度から帰無仮説の自由度を差し引いた数である.
このとき, 以下のF値は帰無仮説が正しいもと自由度 $df$ と $q$ のF分布にしたがう.
$$
\frac{(SSR_0-SSR)/q}{SSR/df}
$$

R でF値は次のようにして算出する.

```r
fm0 <- lm(y~x,data=df)
fm1 <- lm(y~x*w,data=df)
dof <- fm1$df
q <- fm0$df-dof
SSR0 <- deviance(fm0)
SSR <- deviance(fm1)
(F <- ((SSR0-SSR)/q)/(SSR/dof))
<<<<<<< HEAD
## [1] 8.232624
=======
## [1] 9.182515
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

この時のP値は以下である.

```r
1-pf(F,df1=q,df2=dof)
<<<<<<< HEAD
## [1] 0.0005013471
=======
## [1] 0.000224359
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

これらの手順はコマンド `anova` を用いれば簡単に実現できる.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
<<<<<<< HEAD
## 1     98 124.83                                  
## 2     96 106.56  2    18.276 8.2326 0.0005013 ***
=======
## 1     98 125.94                                  
## 2     96 105.72  2    20.224 9.1825 0.0002244 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
<<<<<<< HEAD
## 1     96 106.56                                  
## 2     98 124.83 -2   -18.276 8.2326 0.0005013 ***
=======
## 1     96 105.72                                  
## 2     98 125.94 -2   -20.224 9.1825 0.0002244 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

## 正規性の仮定について
単回帰において以下の仮定を置いていた.

+ $(x_i,y_i)$ は独立同一分布にしたがう.
+ $E[u_i]=0$ である.
+ $u_i$ と $x_i$ は独立である.
+ $u_i$ は正規分布にしたがう.

十分な観測値が得られるばあい, $u_i$ が正規分布にしたがっていないくても, 中心極限定理定理より, 最小二乗法推定量は正規分布に近似できる.

ここの係数ゼロのティー検定について, ライブラリ `AER` を導入して `coeftest` を用いればよい.

```r
coeftest(fm0,df=Inf)
## 
## z test of coefficients:
## 
##             Estimate Std. Error z value  Pr(>|z|)    
<<<<<<< HEAD
## (Intercept) 10.73805    0.22682 47.3423 < 2.2e-16 ***
## x            1.58145    0.37498  4.2174 2.471e-05 ***
=======
## (Intercept) 10.39733    0.22155  46.930 < 2.2e-16 ***
## x            2.14999    0.40712   5.281 1.285e-07 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## Model 2: y ~ x * w
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1     98                         
<<<<<<< HEAD
## 2     96  2 16.465  0.0002658 ***
=======
## 2     96  2 18.365  0.0001028 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## Model 2: y ~ x * w
##   Res.Df Df      F    Pr(>F)    
## 1     98                        
<<<<<<< HEAD
## 2     96  2 8.2326 0.0005013 ***
=======
## 2     96  2 9.1825 0.0002244 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
これは `anova` と同じである.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
<<<<<<< HEAD
## 1     98 124.83                                  
## 2     96 106.56  2    18.276 8.2326 0.0005013 ***
=======
## 1     98 125.94                                  
## 2     96 105.72  2    20.224 9.1825 0.0002244 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

複数制約の検定としてLM検定というのもある.
制約付きの回帰分析を実行し, その残差を制約なしのモデルの説明変数に回帰する.
その決定係数に観測数を掛けた統計量が自由どが制約の数のカイ二乗分布にしたがうことが知られている.

```r
lmt <- lm(I(resid(fm1))~w*x,data=df)
(lmt <- nrow(df)*summary(lmt)$r.squared)
<<<<<<< HEAD
## [1] 5.119844e-30
=======
## [1] 3.100841e-30
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
1-pchisq(lmt,df=1)
## [1] 1
```

## 誤差項と説明変数が独立の仮定について
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
##             Estimate Std. Error t value  Pr(>|t|)    
<<<<<<< HEAD
## (Intercept) 11.02512    0.19072 57.8072 < 2.2e-16 ***
## x            1.86176    0.38246  4.8678 4.428e-06 ***
## wT          -0.76579    0.42760 -1.7909   0.07646 .  
## x:wT        -0.18126    0.67006 -0.2705   0.78734    
=======
## (Intercept) 11.12632    0.33748 32.9689 < 2.2e-16 ***
## x            1.48241    0.51630  2.8712  0.005031 ** 
## wT          -1.16353    0.43456 -2.6775  0.008726 ** 
## x:wT         0.52017    0.83517  0.6228  0.534874    
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
##             Estimate Std. Error t value  Pr(>|t|)    
<<<<<<< HEAD
## (Intercept) 11.02512    0.18516 59.5449 < 2.2e-16 ***
## x            1.86176    0.36956  5.0378 2.209e-06 ***
## wT          -0.76579    0.41301 -1.8542   0.06678 .  
## x:wT        -0.18126    0.64656 -0.2804   0.77981    
=======
## (Intercept) 11.12632    0.32568 34.1635 < 2.2e-16 ***
## x            1.48241    0.49872  2.9724  0.003735 ** 
## wT          -1.16353    0.41942 -2.7741  0.006652 ** 
## x:wT         0.52017    0.79857  0.6514  0.516361    
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
<<<<<<< HEAD
## (Intercept) 10.73805    0.21006 51.1188 < 2.2e-16 ***
## x            1.58145    0.35120  4.5029 6.702e-06 ***
=======
## (Intercept) 10.39733    0.23230 44.7577 < 2.2e-16 ***
## x            2.14999    0.40279  5.3377 9.413e-08 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

複数の係数についての検定は `waldtest` を実行すればよい.

```r
waldtest(fm0,fm1,vcov=vcovHC)
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df Df      F   Pr(>F)    
## 1     98                       
<<<<<<< HEAD
## 2     96  2 7.7367 0.000767 ***
=======
## 2     96  2 8.0846 0.000569 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

先の結果はエフ検定であるが, カイ二乗検定を実施するには以下を実施すればよい.

```r
waldtest(fm0,fm1,vcov=vcovHC, test="Chisq")
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1     98                         
<<<<<<< HEAD
## 2     96  2 15.473  0.0004365 ***
=======
## 2     96  2 16.169  0.0003082 ***
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


## 分散均一の検定
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
## [1] 12.51186
1-pchisq(bpt,df=3)
## [1] 0.00582045
=======
## [1] 1.333464
1-pchisq(bpt,df=3)
## [1] 0.7212024
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
<<<<<<< HEAD
## BP = 12.512, df = 3, p-value = 0.00582
=======
## BP = 1.3335, df = 3, p-value = 0.7212
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
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
## [1] 3.526818
1-pchisq(wht,df=2)
## [1] 0.1714593
=======
## [1] 1.620089
1-pchisq(wht,df=2)
## [1] 0.4448384
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```
ホワイト検定でも分散均一が示唆されている.

もしくは以下を実行する.

```r
bptest(fm1,~fitted(fm1)+I(fitted(fm1)^2))
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
<<<<<<< HEAD
## BP = 3.5268, df = 2, p-value = 0.1715
=======
## BP = 1.6201, df = 2, p-value = 0.4448
>>>>>>> 7a907db56f9a902d6e0d8b7b6b40751f6048538a
```

このように分散均一性は検定することが可能であるが, そもそも分散均一が疑われる場合は,
ロバスト分散で推定するので十分であるため最近の実証分析ではこの検定は実施されない.



