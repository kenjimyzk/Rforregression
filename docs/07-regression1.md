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
##    9.782484    2.451933
# coefficients(fm)
```

傾きの推定値は以下でも実行可能である.

```r
with(df, cov(x,y)/var(x))
## [1] 2.451933
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
##         1         2         3         4         5         6 
## 0.9905755 2.0116401 1.4462317 0.5534805 0.2471162 0.5916036
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.57538 10.35135 10.20188 11.16864 10.96629 10.55721
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 120.2486
sum(resid(fm)^2)
## [1] 120.2486
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
##      Min       1Q   Median       3Q      Max 
## -2.36080 -0.77545  0.06128  0.86065  2.23023 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   9.7825     0.2134  45.843  < 2e-16 ***
## x             2.4519     0.3817   6.424 4.79e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.108 on 98 degrees of freedom
## Multiple R-squared:  0.2963,	Adjusted R-squared:  0.2892 
## F-statistic: 41.27 on 1 and 98 DF,  p-value: 4.791e-09
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
##             Estimate Std. Error  t value     Pr(>|t|)
## (Intercept) 9.782484  0.2133912 45.84295 5.133101e-68
## x           2.451933  0.3816685  6.42425 4.791380e-09
# coefficients(summary(fm))
```

残差の標準誤差は次のようにして得られる.

```r
with(summary(fm),sigma)
## [1] 1.107713
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.107713
```

決定係数は次のようにして計算する.

```r
with(summary(fm),r.squared)
## [1] 0.2963358
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.2963358
```

調整済み決定係数は次のようにして計算する.

```r
with(summary(fm),adj.r.squared)
## [1] 0.2891556
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.2891556
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
## -2.70622 -0.70587  0.01096  0.80468  2.43042 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.6395     0.1616  72.032  < 2e-16 ***
## log(x)        0.6325     0.1064   5.944 4.28e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.132 on 98 degrees of freedom
## Multiple R-squared:  0.265,	Adjusted R-squared:  0.2575 
## F-statistic: 35.33 on 1 and 98 DF,  p-value: 4.281e-08
```

作図すると以下のようになる.

```r
plot(y~log(x),data=df)
abline(fm)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-19-1.png" width="672" />

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
## -0.24954 -0.06684  0.01301  0.08001  0.19737 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.27762    0.01998 113.988  < 2e-16 ***
## x            0.22762    0.03574   6.369 6.18e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1037 on 98 degrees of freedom
## Multiple R-squared:  0.2928,	Adjusted R-squared:  0.2855 
## F-statistic: 40.57 on 1 and 98 DF,  p-value: 6.176e-09
```

作図すると以下のようになる.

```r
plot(log(y)~x,data=df)
abline(fm)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-21-1.png" width="672" />



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
## -6.307 -1.114  2.906  6.677 10.457 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.4067     0.9339   18.64   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.221 on 99 degrees of freedom
## Multiple R-squared:  0.7782,	Adjusted R-squared:  0.776 
## F-statistic: 347.4 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -6.307 -1.114  2.906  6.677 10.457 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.4067     0.9339   18.64   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.221 on 99 degrees of freedom
## Multiple R-squared:  0.7782,	Adjusted R-squared:  0.776 
## F-statistic: 347.4 on 1 and 99 DF,  p-value: < 2.2e-16
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
作図すると以下である.

```r
plot(y~x)
```

<img src="07-regression1_files/figure-html/unnamed-chunk-25-1.png" width="672" />


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
## -2.95242 -0.51936 -0.02574  0.65752  2.58107 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.7453     0.2072  51.869  < 2e-16 ***
## x             2.7438     0.3587   7.650 1.48e-11 ***
## wT           -1.1572     0.2148  -5.388 4.98e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.056 on 97 degrees of freedom
## Multiple R-squared:  0.4491,	Adjusted R-squared:  0.4377 
## F-statistic: 39.54 on 2 and 97 DF,  p-value: 2.765e-13
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
## -3.3502 -0.7136  0.0970  0.7641  3.2231 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.7799     0.3206  33.620   <2e-16 ***
## x            -0.3007     1.5974  -0.188   0.8511    
## I(x^2)        2.8938     1.5865   1.824   0.0712 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.184 on 97 degrees of freedom
## Multiple R-squared:  0.3079,	Adjusted R-squared:  0.2937 
## F-statistic: 21.58 on 2 and 97 DF,  p-value: 1.765e-08
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
## -2.95918 -0.53587 -0.02991  0.64991  2.54121 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.7727     0.2473  43.566  < 2e-16 ***
## x             2.6794     0.4775   5.611 1.94e-07 ***
## wT           -1.2269     0.4018  -3.054  0.00292 ** 
## x:wT          0.1497     0.7281   0.206  0.83751    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.061 on 96 degrees of freedom
## Multiple R-squared:  0.4493,	Adjusted R-squared:  0.4321 
## F-statistic: 26.11 on 3 and 96 DF,  p-value: 1.95e-12
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
## -2.95918 -0.53587 -0.02991  0.64991  2.54121 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.7727     0.2473  43.566  < 2e-16 ***
## x             2.6794     0.4775   5.611 1.94e-07 ***
## wT           -1.2269     0.4018  -3.054  0.00292 ** 
## x:wT          0.1497     0.7281   0.206  0.83751    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.061 on 96 degrees of freedom
## Multiple R-squared:  0.4493,	Adjusted R-squared:  0.4321 
## F-statistic: 26.11 on 3 and 96 DF,  p-value: 1.95e-12
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
## [1] 14.39493
```

この時のP値は以下である.

```r
1-pf(F,df1=q,df2=dof)
## [1] 3.40732e-06
```

これらの手順はコマンド `anova` を用いれば簡単に実現できる.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 140.58                                  
## 2     96 108.15  2    32.433 14.395 3.407e-06 ***
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
## 1     96 108.15                                  
## 2     98 140.58 -2   -32.433 14.395 3.407e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


