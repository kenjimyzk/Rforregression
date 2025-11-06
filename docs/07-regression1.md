---
output: 
  html_document:default
  html_notebook:default
---

# 古典的仮定のもとでの最小二乗法



``` r
library(AER)
```



## 単回帰モデル
次の単回帰モデルを考える.

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


仮想的に以下のモデルを考える.

``` r
N <- 100
x <- runif(N)
y <- 10 + 2*x + rnorm(N)
df <- data.frame(x,y)
```
作図すると以下である.

``` r
plot(y~x)
```

![](07-regression1_files/figure-latex/unnamed-chunk-3-1.pdf)<!-- --> 

R で回帰分析を実施するには `lm` を実施する.

``` r
fm <- lm(y ~ x, data=df)
```

`fm` 自体はリストであり, 以下の要素がある.

``` r
typeof(fm)
## [1] "list"
names(fm)
##  [1] "coefficients"  "residuals"     "effects"       "rank"         
##  [5] "fitted.values" "assign"        "qr"            "df.residual"  
##  [9] "xlevels"       "call"          "terms"         "model"
```

係数は次のコマンドを実施する.

``` r
coef(fm)
## (Intercept)           x 
##   10.113756    1.707754
# coefficients(fm)
```

傾きの推定値は以下でも実行可能である.

``` r
with(df, cov(x,y)/var(x))
## [1] 1.707754
```


作図すると以下のようになる.

``` r
plot(y~x,data=df)
abline(fm)
```

![](07-regression1_files/figure-latex/unnamed-chunk-8-1.pdf)<!-- --> 

残差は次のコマンドを実施する.

``` r
head(resid(fm))
##           1           2           3           4           5           6 
## -0.09805546  0.85078160 -0.25488300 -1.52085368  0.25452751  2.73785084
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

``` r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.43640 10.58811 10.20084 11.65548 10.96895 11.37792
# fitted.values(fm)
# with(fm, fitted.values)
```
予測値の平均は被説明変数の平均と等しい．

``` r
mean(fitted(fm))
## [1] 10.98207
mean(df$y)
## [1] 10.98207
```


残差自乗和は次のコマンドを実施する.

``` r
deviance(fm)
## [1] 105.5249
sum(resid(fm)^2)
## [1] 105.5249
```

残差自乗和は残差変動とも呼ばれる．予測値の偏差の自乗和を回帰変動といい，被説明変数の自乗和を全変動といえば，全変動は回帰変動と残差変動に分解できる．

``` r
sum((df$y-mean(df$y))^2)
## [1] 130.7204
sum((fitted(fm)-mean(df$y))^2)+deviance(fm)
## [1] 130.7204
```


### ティー検定
その結果を見るには `summary` を実施する.

``` r
summary(fm)
## 
## Call:
## lm(formula = y ~ x, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.80427 -0.61411  0.02092  0.54014  2.73785 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.1138     0.2073  48.778  < 2e-16 ***
## x             1.7078     0.3530   4.837 4.89e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.038 on 98 degrees of freedom
## Multiple R-squared:  0.1927,	Adjusted R-squared:  0.1845 
## F-statistic:  23.4 on 1 and 98 DF,  p-value: 4.894e-06
```

これをみれば各変数の係数がゼロのティー検定の結果が示されている.
他にも, 残差標準誤差, 決定係数, 修正済み決定係数, 全ての係数がゼロであるエフ検定の結果も示されている.

`summary(fm)` もリストであり, それぞれの要素は以下である.

``` r
typeof(summary(fm))
## [1] "list"
names(summary(fm))
##  [1] "call"          "terms"         "residuals"     "coefficients" 
##  [5] "aliased"       "sigma"         "df"            "r.squared"    
##  [9] "adj.r.squared" "fstatistic"    "cov.unscaled"
```

単なる `fm` と同じ名前もあるが, 中身が違っている場合もある.
例えば `residuals` は同じであるが, 係数はより情報が加わっている.

``` r
coef(summary(fm))
##              Estimate Std. Error  t value     Pr(>|t|)
## (Intercept) 10.113756  0.2073419 48.77815 1.509594e-70
## x            1.707754  0.3530439  4.83723 4.894208e-06
# coefficients(summary(fm))
```

残差の標準誤差は次のようにして得られる.

``` r
with(summary(fm),sigma)
## [1] 1.037683
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.037683
```

決定係数は次のようにして計算する.

``` r
with(summary(fm),r.squared)
## [1] 0.1927432
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.1927432
```

調整済み決定係数は次のようにして計算する.

``` r
with(summary(fm),adj.r.squared)
## [1] 0.1845059
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.1845059
```



### 対数変換
次のモデルを考える.
$$
y = \alpha + \beta \log(x) + u
$$

対数変換をおこなう場合, `log` を用いるとよい.

``` r
fm <- lm(y~log(x),data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ log(x), data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -3.02695 -0.66260  0.01192  0.53370  2.79625 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.4741     0.1525  75.225  < 2e-16 ***
## log(x)        0.5141     0.1153   4.459 2.19e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.053 on 98 degrees of freedom
## Multiple R-squared:  0.1687,	Adjusted R-squared:  0.1602 
## F-statistic: 19.88 on 1 and 98 DF,  p-value: 2.194e-05
```

作図すると以下のようになる.

``` r
plot(y~log(x),data=df)
abline(fm)
```

![](07-regression1_files/figure-latex/unnamed-chunk-21-1.pdf)<!-- --> 

被説明変数が対数の場合も同様である.

``` r
fm <- lm(log(y)~x,data=df)
summary(fm)
## 
## Call:
## lm(formula = log(y) ~ x, data = df)
## 
## Residuals:
##       Min        1Q    Median        3Q       Max 
## -0.304143 -0.057846  0.006416  0.052564  0.220286 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.31142    0.01900 121.635  < 2e-16 ***
## x            0.15615    0.03236   4.826 5.12e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0951 on 98 degrees of freedom
## Multiple R-squared:  0.192,	Adjusted R-squared:  0.1838 
## F-statistic: 23.29 on 1 and 98 DF,  p-value: 5.123e-06
```

作図すると以下のようになる.

``` r
plot(log(y)~x,data=df)
abline(fm)
```

![](07-regression1_files/figure-latex/unnamed-chunk-23-1.pdf)<!-- --> 


被説明変数および説明変数が対数の場合以下である.

``` r
fm <- lm(log(y)~log(x),data=df)
summary(fm)
## 
## Call:
## lm(formula = log(y) ~ log(x), data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.32453 -0.05559  0.00583  0.05165  0.22567 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.43573    0.01398 174.194  < 2e-16 ***
## log(x)       0.04693    0.01057   4.441 2.36e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09653 on 98 degrees of freedom
## Multiple R-squared:  0.1675,	Adjusted R-squared:  0.159 
## F-statistic: 19.72 on 1 and 98 DF,  p-value: 2.356e-05
```

作図すると以下のようになる.

``` r
plot(log(y)~log(x),data=df)
abline(fm)
```

![](07-regression1_files/figure-latex/unnamed-chunk-25-1.pdf)<!-- --> 



### 切片なし回帰モデル
次のモデルを考える.
$$
y = \beta x + u
$$

切片なしモデルを推定したい場合は次のように `-1` とする.

``` r
fm <- lm(y~x-1,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x - 1, data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -6.7896 -0.7792  2.8768  6.4210 10.5270 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  16.6168     0.8838    18.8   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.191 on 99 degrees of freedom
## Multiple R-squared:  0.7812,	Adjusted R-squared:  0.779 
## F-statistic: 353.5 on 1 and 99 DF,  p-value: < 2.2e-16
```

もしくは `+0` を加える.

``` r
fm <- lm(y~x+0,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + 0, data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -6.7896 -0.7792  2.8768  6.4210 10.5270 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  16.6168     0.8838    18.8   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.191 on 99 degrees of freedom
## Multiple R-squared:  0.7812,	Adjusted R-squared:  0.779 
## F-statistic: 353.5 on 1 and 99 DF,  p-value: < 2.2e-16
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


仮想的に以下のモデルを考える.

``` r
N <- 100
x<-runif(N)
w<-sample(c("H","T"),N,replace=TRUE)
y <- 10 + 2*x + ifelse(w=="H",1,0) + rnorm(N)
df <- data.frame(w,x,y)
```

説明変数を加えたいときには `+` と変数名を使うことができる. 

``` r
fm <- lm(y~x+w,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + w, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.47174 -0.62062  0.06871  0.60639  2.14306 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.1593     0.2035  54.846  < 2e-16 ***
## x             1.5145     0.3255   4.653 1.04e-05 ***
## wT           -0.8289     0.1874  -4.424 2.54e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9327 on 97 degrees of freedom
## Multiple R-squared:  0.3089,	Adjusted R-squared:  0.2946 
## F-statistic: 21.67 on 2 and 97 DF,  p-value: 1.654e-08
```

R の特徴は因子もとくに変換することなくダミー変数として扱える.

### 自乗項
説明変数として自乗項を加えたモデルを考える.
$$
y = \alpha + \beta x + \gamma x^2 + u
$$

R では単に自乗するのでなく `I(x^2)` としなければならない.

``` r
fm <- lm(y~x+I(x^2),data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + I(x^2), data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.13080 -0.61718 -0.03541  0.63704  2.49772 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 10.73989    0.28182  38.109   <2e-16 ***
## x            1.61430    1.36311   1.184    0.239    
## I(x^2)      -0.03033    1.37797  -0.022    0.982    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.023 on 97 degrees of freedom
## Multiple R-squared:  0.1694,	Adjusted R-squared:  0.1523 
## F-statistic: 9.894 on 2 and 97 DF,  p-value: 0.0001229
```

### 交差項
説明変数として交差項を加えたモデルを考える.
$$
y = \alpha + \beta x + \gamma w + \delta xw + u
$$

説明変数 `x` と `w`の自乗項は自乗項は `x:w` である.

``` r
fm<-lm(y~x+w+x:w,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + w + x:w, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.46006 -0.60094  0.05924  0.60568  2.05457 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0512     0.2668  41.414  < 2e-16 ***
## x             1.7357     0.4800   3.616 0.000479 ***
## wT           -0.6331     0.3637  -1.741 0.084898 .  
## x:wT         -0.4118     0.6549  -0.629 0.530983    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9357 on 96 degrees of freedom
## Multiple R-squared:  0.3117,	Adjusted R-squared:  0.2902 
## F-statistic: 14.49 on 3 and 96 DF,  p-value: 7.345e-08
```

もしくは以下のように `*` を使って簡便的に表せる.

``` r
fm <- lm(y~x*w,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x * w, data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -2.46006 -0.60094  0.05924  0.60568  2.05457 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0512     0.2668  41.414  < 2e-16 ***
## x             1.7357     0.4800   3.616 0.000479 ***
## wT           -0.6331     0.3637  -1.741 0.084898 .  
## x:wT         -0.4118     0.6549  -0.629 0.530983    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9357 on 96 degrees of freedom
## Multiple R-squared:  0.3117,	Adjusted R-squared:  0.2902 
## F-statistic: 14.49 on 3 and 96 DF,  p-value: 7.345e-08
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

``` r
fm0 <- lm(y~x,data=df)
fm1 <- lm(y~x*w,data=df)
dof <- fm1$df
q <- fm0$df-dof
SSR0 <- deviance(fm0)
SSR <- deviance(fm1)
(F <- ((SSR0-SSR)/q)/(SSR/dof))
## [1] 9.921497
```

この時のP値は以下である.

``` r
1-pf(F,df1=q,df2=dof)
## [1] 0.0001211351
```

これらの手順はコマンド `anova` を用いれば簡単に実現できる.

``` r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 101.418                                  
## 2     96  84.046  2    17.372 9.9215 0.0001211 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

順番を変えても, 検定統計量自体に変更はない.

``` r
anova(fm1,fm0)
## Analysis of Variance Table
## 
## Model 1: y ~ x * w
## Model 2: y ~ x
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     96  84.046                                  
## 2     98 101.418 -2   -17.372 9.9215 0.0001211 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

