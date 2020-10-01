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
##   10.175303    1.637939
# coefficients(fm)
```

傾きの推定値は以下でも実行可能である.

```r
with(df, cov(x,y)/var(x))
## [1] 1.637939
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
##            1            2            3            4            5            6 
## -1.456943382 -1.018246796  0.008625895 -0.745907415  1.552082406 -1.599191308
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.16229 10.84730 10.56691 10.80326 10.55394 10.50580
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 91.65505
sum(resid(fm)^2)
## [1] 91.65505
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
## -2.7170 -0.7208  0.1273  0.6365  2.4806 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.1753     0.1985  51.269  < 2e-16 ***
## x             1.6379     0.3442   4.759  6.7e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9671 on 98 degrees of freedom
## Multiple R-squared:  0.1877,	Adjusted R-squared:  0.1795 
## F-statistic: 22.65 on 1 and 98 DF,  p-value: 6.704e-06
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
## (Intercept) 10.175303  0.1984690 51.268980 1.375148e-72
## x            1.637939  0.3441544  4.759316 6.704460e-06
# coefficients(summary(fm))
```

残差の標準誤差は次のようにして得られる.

```r
with(summary(fm),sigma)
## [1] 0.9670861
sqrt(deviance(fm)/df.residual(fm))
## [1] 0.9670861
```

決定係数は次のようにして計算する.

```r
with(summary(fm),r.squared)
## [1] 0.1877404
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.1877404
```

調整済み決定係数は次のようにして計算する.

```r
with(summary(fm),adj.r.squared)
## [1] 0.1794521
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.1794521
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
## -2.32100 -0.76612  0.05587  0.65795  2.74756 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.4138     0.1493  76.439  < 2e-16 ***
## log(x)        0.4377     0.1170   3.742 0.000308 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.004 on 98 degrees of freedom
## Multiple R-squared:  0.125,	Adjusted R-squared:  0.1161 
## F-statistic:    14 on 1 and 98 DF,  p-value: 0.0003082
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
## -0.25872 -0.06652  0.01463  0.06128  0.19698 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.31891    0.01816 127.674   <2e-16 ***
## x            0.14758    0.03150   4.686    9e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.0885 on 98 degrees of freedom
## Multiple R-squared:  0.183,	Adjusted R-squared:  0.1747 
## F-statistic: 21.96 on 1 and 98 DF,  p-value: 9.001e-06
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
##     Min      1Q  Median      3Q     Max 
## -7.9048 -0.7399  1.9294  5.8755 10.9723 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.0459     0.8801   19.37   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.075 on 99 degrees of freedom
## Multiple R-squared:  0.7912,	Adjusted R-squared:  0.7891 
## F-statistic: 375.2 on 1 and 99 DF,  p-value: < 2.2e-16
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
##     Min      1Q  Median      3Q     Max 
## -7.9048 -0.7399  1.9294  5.8755 10.9723 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.0459     0.8801   19.37   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.075 on 99 degrees of freedom
## Multiple R-squared:  0.7912,	Adjusted R-squared:  0.7891 
## F-statistic: 375.2 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -2.65280 -0.55880 -0.07384  0.59757  2.27899 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5944     0.2093  50.627  < 2e-16 ***
## x             2.7300     0.3225   8.466 2.72e-13 ***
## wT           -0.7277     0.1971  -3.692 0.000368 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9729 on 97 degrees of freedom
## Multiple R-squared:  0.4661,	Adjusted R-squared:  0.4551 
## F-statistic: 42.34 on 2 and 97 DF,  p-value: 6.052e-14
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
## -2.44137 -0.60700  0.00071  0.59665  2.69090 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   9.9837     0.2969  33.623  < 2e-16 ***
## x             4.5784     1.3347   3.430 0.000887 ***
## I(x^2)       -1.8156     1.2595  -1.442 0.152639    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.028 on 97 degrees of freedom
## Multiple R-squared:  0.4039,	Adjusted R-squared:  0.3916 
## F-statistic: 32.86 on 2 and 97 DF,  p-value: 1.272e-11
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
##     Min      1Q  Median      3Q     Max 
## -2.6898 -0.5470 -0.0726  0.6292  2.2415 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5440     0.2545  41.432  < 2e-16 ***
## x             2.8281     0.4276   6.615 2.11e-09 ***
## wT           -0.6084     0.3926  -1.550    0.124    
## x:wT         -0.2305     0.6551  -0.352    0.726    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9773 on 96 degrees of freedom
## Multiple R-squared:  0.4668,	Adjusted R-squared:  0.4501 
## F-statistic: 28.01 on 3 and 96 DF,  p-value: 4.239e-13
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
##     Min      1Q  Median      3Q     Max 
## -2.6898 -0.5470 -0.0726  0.6292  2.2415 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5440     0.2545  41.432  < 2e-16 ***
## x             2.8281     0.4276   6.615 2.11e-09 ***
## wT           -0.6084     0.3926  -1.550    0.124    
## x:wT         -0.2305     0.6551  -0.352    0.726    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9773 on 96 degrees of freedom
## Multiple R-squared:  0.4668,	Adjusted R-squared:  0.4501 
## F-statistic: 28.01 on 3 and 96 DF,  p-value: 4.239e-13
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
## [1] 6.814768
```

この時のP値は以下である.

```r
1-pf(F,df1=q,df2=dof)
## [1] 0.00170803
```

これらの手順はコマンド `anova` を用いれば簡単に実現できる.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df     RSS Df Sum of Sq      F   Pr(>F)   
## 1     98 104.707                                
## 2     96  91.689  2    13.018 6.8148 0.001708 **
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
## 1     96  91.689                                
## 2     98 104.707 -2   -13.018 6.8148 0.001708 **
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


