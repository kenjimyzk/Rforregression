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
##   10.023499    2.150754
# coefficients(fm)
```

傾きの推定値は以下でも実行可能である.

```r
with(df, cov(x,y)/var(x))
## [1] 2.150754
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
##          1          2          3          4          5          6 
## -0.4002569 -0.4948268 -0.3294694  0.8559347  0.6781065 -1.2631742
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.22489 10.21803 11.76267 10.07515 11.95904 10.19354
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 97.54873
sum(resid(fm)^2)
## [1] 97.54873
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
## -2.4272 -0.7150 -0.0124  0.7409  2.0455 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.0235     0.1837  54.575  < 2e-16 ***
## x             2.1508     0.3334   6.452 4.22e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9977 on 98 degrees of freedom
## Multiple R-squared:  0.2981,	Adjusted R-squared:  0.291 
## F-statistic: 41.63 on 1 and 98 DF,  p-value: 4.216e-09
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
## (Intercept) 10.023499  0.1836639 54.575237 3.695690e-75
## x            2.150754  0.3333533  6.451874 4.216474e-09
# coefficients(summary(fm))
```

残差の標準誤差は次のようにして得られる.

```r
with(summary(fm),sigma)
## [1] 0.9976949
sqrt(deviance(fm)/df.residual(fm))
## [1] 0.9976949
```

決定係数は次のようにして計算する.

```r
with(summary(fm),r.squared)
## [1] 0.2981284
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.2981284
```

調整済み決定係数は次のようにして計算する.

```r
with(summary(fm),adj.r.squared)
## [1] 0.2909665
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.2909665
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
## -2.5485 -0.8177  0.0038  0.7833  2.1618 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.6132     0.1562  74.371  < 2e-16 ***
## log(x)        0.5257     0.1017   5.169 1.24e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.056 on 98 degrees of freedom
## Multiple R-squared:  0.2143,	Adjusted R-squared:  0.2062 
## F-statistic: 26.72 on 1 and 98 DF,  p-value: 1.242e-06
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
##       Min        1Q    Median        3Q       Max 
## -0.267678 -0.063377  0.003739  0.073017  0.176159 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.30259    0.01699 135.501  < 2e-16 ***
## x            0.19703    0.03084   6.388 5.66e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09231 on 98 degrees of freedom
## Multiple R-squared:  0.294,	Adjusted R-squared:  0.2868 
## F-statistic: 40.81 on 1 and 98 DF,  p-value: 5.661e-09
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
## -6.441 -1.254  3.343  7.090 10.639 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   17.425      1.009   17.26   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.562 on 99 degrees of freedom
## Multiple R-squared:  0.7506,	Adjusted R-squared:  0.7481 
## F-statistic:   298 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -6.441 -1.254  3.343  7.090 10.639 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   17.425      1.009   17.26   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.562 on 99 degrees of freedom
## Multiple R-squared:  0.7506,	Adjusted R-squared:  0.7481 
## F-statistic:   298 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -2.59070 -0.71651  0.04425  0.69793  2.23241 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9648     0.1978  55.425  < 2e-16 ***
## x             1.9589     0.3459   5.663 1.51e-07 ***
## wT           -0.9796     0.1979  -4.951 3.11e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9698 on 97 degrees of freedom
## Multiple R-squared:  0.3287,	Adjusted R-squared:  0.3149 
## F-statistic: 23.75 on 2 and 97 DF,  p-value: 4.02e-09
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
## -3.09952 -0.70765 -0.04141  0.83326  2.12192 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.4981     0.3309  31.727   <2e-16 ***
## x             2.5575     1.6007   1.598    0.113    
## I(x^2)       -0.9391     1.5697  -0.598    0.551    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.083 on 97 degrees of freedom
## Multiple R-squared:  0.1622,	Adjusted R-squared:  0.1449 
## F-statistic: 9.389 on 2 and 97 DF,  p-value: 0.0001873
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
## -2.58216 -0.71785  0.03446  0.70799  2.23829 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9355     0.2435  44.904  < 2e-16 ***
## x             2.0286     0.4829   4.201 5.96e-05 ***
## wT           -0.9106     0.3868  -2.354   0.0206 *  
## x:wT         -0.1447     0.6957  -0.208   0.8357    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9746 on 96 degrees of freedom
## Multiple R-squared:  0.329,	Adjusted R-squared:  0.3081 
## F-statistic: 15.69 on 3 and 96 DF,  p-value: 2.214e-08
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
## -2.58216 -0.71785  0.03446  0.70799  2.23829 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9355     0.2435  44.904  < 2e-16 ***
## x             2.0286     0.4829   4.201 5.96e-05 ***
## wT           -0.9106     0.3868  -2.354   0.0206 *  
## x:wT         -0.1447     0.6957  -0.208   0.8357    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9746 on 96 degrees of freedom
## Multiple R-squared:  0.329,	Adjusted R-squared:  0.3081 
## F-statistic: 15.69 on 3 and 96 DF,  p-value: 2.214e-08
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
## [1] 12.15762
```

この時のP値は以下である.

```r
1-pf(F,df1=q,df2=dof)
## [1] 1.966205e-05
```

これらの手順はコマンド `anova` を用いれば簡単に実現できる.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 114.27                                  
## 2     96  91.18  2    23.094 12.158 1.966e-05 ***
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
## 1     96  91.18                                  
## 2     98 114.27 -2   -23.094 12.158 1.966e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


