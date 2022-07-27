---
output: 
  html_document:default
  html_notebook:default
---

# 古典的仮定のもとでの最小二乗法



```r
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

```r
N <- 100
x <- runif(N)
y <- 10 + 2*x + rnorm(N)
df <- data.frame(x,y)
```
作図すると以下である.

```r
plot(y~x)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-3-1.png" width="672" />

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
##    9.825092    2.595725
# coefficients(fm)
```

傾きの推定値は以下でも実行可能である.

```r
with(df, cov(x,y)/var(x))
## [1] 2.595725
```


作図すると以下のようになる.

```r
plot(y~x,data=df)
abline(fm)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-8-1.png" width="672" />

残差は次のコマンドを実施する.

```r
head(resid(fm))
##           1           2           3           4           5           6 
##  0.61680334  1.03346637 -1.09867015  0.05208694  1.12629875  0.10051494
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##         1         2         3         4         5         6 
## 12.279635  9.929375 12.062531 11.551779 12.199447 10.109367
# fitted.values(fm)
# with(fm, fitted.values)
```
予測値の平均は被説明変数の平均と等しい．

```r
mean(fitted(fm))
## [1] 11.16045
mean(df$y)
## [1] 11.16045
```


残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 111.3742
sum(resid(fm)^2)
## [1] 111.3742
```

残差自乗和は残差変動とも呼ばれる．予測値の偏差の自乗和を回帰変動といい，被説明変数の自乗和を全変動といえば，全変動は回帰変動と残差変動に分解できる．

```r
sum((df$y-mean(df$y))^2)
## [1] 175.2875
sum((fitted(fm)-mean(df$y))^2)+deviance(fm)
## [1] 175.2875
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
## -2.7129 -0.7539  0.1114  0.7628  2.3040 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   9.8251     0.2075  47.341  < 2e-16 ***
## x             2.5957     0.3461   7.499 2.92e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.066 on 98 degrees of freedom
## Multiple R-squared:  0.3646,	Adjusted R-squared:  0.3581 
## F-statistic: 56.24 on 1 and 98 DF,  p-value: 2.922e-11
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
##             Estimate Std. Error   t value     Pr(>|t|)
## (Intercept) 9.825092  0.2075392 47.340904 2.514512e-69
## x           2.595725  0.3461326  7.499222 2.922017e-11
# coefficients(summary(fm))
```

残差の標準誤差は次のようにして得られる.

```r
with(summary(fm),sigma)
## [1] 1.066054
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.066054
```

決定係数は次のようにして計算する.

```r
with(summary(fm),r.squared)
## [1] 0.3646197
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.3646197
```

調整済み決定係数は次のようにして計算する.

```r
with(summary(fm),adj.r.squared)
## [1] 0.3581362
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.3581362
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
##      Min       1Q   Median       3Q      Max 
## -3.13316 -0.76469  0.03053  0.92350  2.77371 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.8037     0.1568  75.267  < 2e-16 ***
## log(x)        0.6232     0.1039   5.996 3.38e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.144 on 98 degrees of freedom
## Multiple R-squared:  0.2684,	Adjusted R-squared:  0.2609 
## F-statistic: 35.96 on 1 and 98 DF,  p-value: 3.378e-08
```

作図すると以下のようになる.

```r
plot(y~log(x),data=df)
abline(fm)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-21-1.png" width="672" />

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
## -0.30191 -0.06517  0.01497  0.06681  0.21289 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.28217    0.01925 118.535  < 2e-16 ***
## x            0.23886    0.03211   7.439 3.92e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0989 on 98 degrees of freedom
## Multiple R-squared:  0.3609,	Adjusted R-squared:  0.3543 
## F-statistic: 55.33 on 1 and 98 DF,  p-value: 3.917e-11
```

作図すると以下のようになる.

```r
plot(log(y)~x,data=df)
abline(fm)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-23-1.png" width="672" />


被説明変数および説明変数が対数の場合以下である.

```r
fm <- lm(log(y)~log(x),data=df)
summary(fm)
## 
## Call:
## lm(formula = log(y) ~ log(x), data = df)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.34035 -0.06662  0.00824  0.08860  0.25693 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 2.464544   0.014506 169.894  < 2e-16 ***
## log(x)      0.057633   0.009613   5.995 3.39e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1058 on 98 degrees of freedom
## Multiple R-squared:  0.2684,	Adjusted R-squared:  0.2609 
## F-statistic: 35.94 on 1 and 98 DF,  p-value: 3.392e-08
```

作図すると以下のようになる.

```r
plot(log(y)~log(x),data=df)
abline(fm)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-25-1.png" width="672" />



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
## -4.862 -1.050  2.026  6.451 11.827 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  16.6550     0.8642   19.27   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.182 on 99 degrees of freedom
## Multiple R-squared:  0.7895,	Adjusted R-squared:  0.7874 
## F-statistic: 371.4 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -4.862 -1.050  2.026  6.451 11.827 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  16.6550     0.8642   19.27   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.182 on 99 degrees of freedom
## Multiple R-squared:  0.7895,	Adjusted R-squared:  0.7874 
## F-statistic: 371.4 on 1 and 99 DF,  p-value: < 2.2e-16
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

```r
N <- 100
x<-runif(N)
w<-sample(c("H","T"),N,replace=TRUE)
y <- 10 + 2*x + ifelse(w=="H",1,0) + rnorm(N)
df <- data.frame(w,x,y)
```

説明変数を加えたいときには `+` と変数名を使うことができる. 

```r
fm <- lm(y~x+w,data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + w, data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -2.3079 -0.6550  0.0271  0.6736  2.3129 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.2570     0.2188  51.459  < 2e-16 ***
## x             1.1940     0.3620   3.298 0.001363 ** 
## wT           -0.6917     0.1966  -3.519 0.000662 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.976 on 97 degrees of freedom
## Multiple R-squared:  0.181,	Adjusted R-squared:  0.1641 
## F-statistic: 10.72 on 2 and 97 DF,  p-value: 6.241e-05
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
##      Min       1Q   Median       3Q      Max 
## -2.52279 -0.60787 -0.03984  0.69538  2.30351 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5583     0.3427  30.813   <2e-16 ***
## x             3.1906     1.5613   2.044   0.0437 *  
## I(x^2)       -2.0855     1.5001  -1.390   0.1676    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.026 on 97 degrees of freedom
## Multiple R-squared:  0.09445,	Adjusted R-squared:  0.07578 
## F-statistic: 5.059 on 2 and 97 DF,  p-value: 0.008133
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
## -2.32128 -0.66655  0.02865  0.66750  2.30769 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 11.23721    0.28557  39.351   <2e-16 ***
## x            1.23743    0.54105   2.287   0.0244 *  
## wT          -0.65388    0.40023  -1.634   0.1056    
## x:wT        -0.07935    0.73113  -0.109   0.9138    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.981 on 96 degrees of freedom
## Multiple R-squared:  0.1811,	Adjusted R-squared:  0.1555 
## F-statistic: 7.075 on 3 and 96 DF,  p-value: 0.00024
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
## -2.32128 -0.66655  0.02865  0.66750  2.30769 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 11.23721    0.28557  39.351   <2e-16 ***
## x            1.23743    0.54105   2.287   0.0244 *  
## wT          -0.65388    0.40023  -1.634   0.1056    
## x:wT        -0.07935    0.73113  -0.109   0.9138    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.981 on 96 degrees of freedom
## Multiple R-squared:  0.1811,	Adjusted R-squared:  0.1555 
## F-statistic: 7.075 on 3 and 96 DF,  p-value: 0.00024
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
## [1] 6.134135
```

この時のP値は以下である.

```r
1-pf(F,df1=q,df2=dof)
## [1] 0.003111441
```

これらの手順はコマンド `anova` を用いれば簡単に実現できる.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df     RSS Df Sum of Sq      F   Pr(>F)   
## 1     98 104.203                                
## 2     96  92.396  2    11.808 6.1341 0.003111 **
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
##   Res.Df     RSS Df Sum of Sq      F   Pr(>F)   
## 1     96  92.396                                
## 2     98 104.203 -2   -11.808 6.1341 0.003111 **
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

