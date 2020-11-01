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
##    10.17148     1.57162
# coefficients(fm)
```

傾きの推定値は以下でも実行可能である.

```r
with(df, cov(x,y)/var(x))
## [1] 1.57162
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
##  0.9483031  0.7678967 -1.4065005 -0.7162350  0.9020216 -0.1908716
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 10.24772 11.26291 10.44368 11.30872 10.79385 10.59254
# fitted.values(fm)
# with(fm, fitted.values)
```
予測値の平均は被説明変数の平均と等しい．

```r
mean(fitted(fm))
## [1] 10.96036
mean(df$y)
## [1] 10.96036
```


残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 87.40475
sum(resid(fm)^2)
## [1] 87.40475
```

残差自乗和は残差変動とも呼ばれる．予測値の偏差の自乗和を回帰変動といい，被説明変数の自乗和を全変動といえば，全変動は回帰変動と残差変動に分解できる．

```r
sum((df$y-mean(df$y))^2)
## [1] 106.6696
sum((fitted(fm)-mean(df$y))^2)+deviance(fm)
## [1] 106.6696
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
## -2.17507 -0.78154 -0.00715  0.77901  2.04845 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.1715     0.1942  52.365  < 2e-16 ***
## x             1.5716     0.3382   4.648 1.05e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9444 on 98 degrees of freedom
## Multiple R-squared:  0.1806,	Adjusted R-squared:  0.1722 
## F-statistic:  21.6 on 1 and 98 DF,  p-value: 1.048e-05
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
## (Intercept) 10.17148  0.1942425 52.364875 1.860235e-73
## x            1.57162  0.3381577  4.647595 1.047602e-05
# coefficients(summary(fm))
```

残差の標準誤差は次のようにして得られる.

```r
with(summary(fm),sigma)
## [1] 0.9443967
sqrt(deviance(fm)/df.residual(fm))
## [1] 0.9443967
```

決定係数は次のようにして計算する.

```r
with(summary(fm),r.squared)
## [1] 0.180603
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.180603
```

調整済み決定係数は次のようにして計算する.

```r
with(summary(fm),adj.r.squared)
## [1] 0.1722418
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.1722418
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
## -2.31203 -0.69445 -0.01104  0.68437  2.28570 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.3001     0.1425  79.325  < 2e-16 ***
## log(x)        0.3490     0.1053   3.314  0.00129 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9893 on 98 degrees of freedom
## Multiple R-squared:  0.1008,	Adjusted R-squared:  0.09163 
## F-statistic: 10.99 on 1 and 98 DF,  p-value: 0.001288
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
##       Min        1Q    Median        3Q       Max 
## -0.218229 -0.069047  0.003131  0.070851  0.172041 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.31674    0.01793 129.219  < 2e-16 ***
## x            0.14551    0.03121   4.662 9.89e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.08717 on 98 degrees of freedom
## Multiple R-squared:  0.1815,	Adjusted R-squared:  0.1732 
## F-statistic: 21.73 on 1 and 98 DF,  p-value: 9.89e-06
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
##       Min        1Q    Median        3Q       Max 
## -0.230893 -0.059997  0.003387  0.063457  0.189108 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 2.421147   0.013156 184.031  < 2e-16 ***
## log(x)      0.032227   0.009725   3.314  0.00129 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09137 on 98 degrees of freedom
## Multiple R-squared:  0.1008,	Adjusted R-squared:  0.09158 
## F-statistic: 10.98 on 1 and 98 DF,  p-value: 0.001291
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
##     Min      1Q  Median      3Q     Max 
## -6.0193 -0.6367  2.4605  5.8660 11.2137 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.0454     0.8806   19.36   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.058 on 99 degrees of freedom
## Multiple R-squared:  0.791,	Adjusted R-squared:  0.7889 
## F-statistic: 374.7 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -6.0193 -0.6367  2.4605  5.8660 11.2137 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.0454     0.8806   19.36   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.058 on 99 degrees of freedom
## Multiple R-squared:  0.791,	Adjusted R-squared:  0.7889 
## F-statistic: 374.7 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -3.6858 -0.6636  0.0157  0.6711  2.4343 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9208     0.2189  49.894  < 2e-16 ***
## x             2.1524     0.3388   6.352 6.88e-09 ***
## wT           -1.0301     0.2117  -4.866 4.41e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.058 on 97 degrees of freedom
## Multiple R-squared:  0.3931,	Adjusted R-squared:  0.3806 
## F-statistic: 31.41 on 2 and 97 DF,  p-value: 3.034e-11
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
## -4.1370 -0.5892  0.0205  0.9274  2.9344 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5477     0.3126  33.747   <2e-16 ***
## x             1.3345     1.5736   0.848    0.398    
## I(x^2)        0.7961     1.5490   0.514    0.608    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.179 on 97 degrees of freedom
## Multiple R-squared:  0.247,	Adjusted R-squared:  0.2315 
## F-statistic: 15.91 on 2 and 97 DF,  p-value: 1.059e-06
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
## -3.7037 -0.6323  0.0253  0.7159  2.4909 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9953     0.2668  41.210  < 2e-16 ***
## x             1.9957     0.4659   4.284 4.36e-05 ***
## wT           -1.1919     0.3914  -3.045    0.003 ** 
## x:wT          0.3356     0.6819   0.492    0.624    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.062 on 96 degrees of freedom
## Multiple R-squared:  0.3946,	Adjusted R-squared:  0.3757 
## F-statistic: 20.86 on 3 and 96 DF,  p-value: 1.734e-10
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
## -3.7037 -0.6323  0.0253  0.7159  2.4909 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9953     0.2668  41.210  < 2e-16 ***
## x             1.9957     0.4659   4.284 4.36e-05 ***
## wT           -1.1919     0.3914  -3.045    0.003 ** 
## x:wT          0.3356     0.6819   0.492    0.624    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.062 on 96 degrees of freedom
## Multiple R-squared:  0.3946,	Adjusted R-squared:  0.3757 
## F-statistic: 20.86 on 3 and 96 DF,  p-value: 1.734e-10
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
## [1] 11.86678
```

この時のP値は以下である.

```r
1-pf(F,df1=q,df2=dof)
## [1] 2.481166e-05
```

これらの手順はコマンド `anova` を用いれば簡単に実現できる.

```r
anova(fm0,fm1)
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 135.11                                  
## 2     96 108.33  2    26.781 11.867 2.481e-05 ***
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
## 1     96 108.33                                  
## 2     98 135.11 -2   -26.781 11.867 2.481e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

