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

暗黙に次の仮定を置いている.

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
##   10.121971    2.770465
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
##          1          2          3          4          5          6 
##  0.2018594 -0.2532528  1.0367234  1.6275431 -1.1337232 -1.2961554
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 12.47936 10.62927 10.65450 12.05173 12.38814 11.50682
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 121.6502
sum(resid(fm)^2)
## [1] 121.6502
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
## -2.28559 -0.73261  0.02647  0.78647  2.41601 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.1220     0.2265  44.680  < 2e-16 ***
## x             2.7705     0.3929   7.051 2.52e-10 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.114 on 98 degrees of freedom
## Multiple R-squared:  0.3366,	Adjusted R-squared:  0.3298 
## F-statistic: 49.72 on 1 and 98 DF,  p-value: 2.519e-10
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
##              Estimate Std. Error   t value     Pr(>|t|)
## (Intercept) 10.121971  0.2265431 44.680116 5.681398e-67
## x            2.770465  0.3929089  7.051164 2.518857e-10
# coefficients(summary(fm))
```

残差の標準誤差

```r
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.114149
with(summary(fm),sigma)
## [1] 1.114149
```

決定係数は次のようにして計算する.

```r
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.3365778
with(summary(fm),r.squared)
## [1] 0.3365778
```

調整済み決定係数は次のようにして計算する.

```r
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.3298082
with(summary(fm),adj.r.squared)
## [1] 0.3298082
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
## -2.4269 -0.9088 -0.1085  0.6732  3.2636 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  12.1244     0.1750  69.296  < 2e-16 ***
## log(x)        0.6292     0.1284   4.899 3.81e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.226 on 98 degrees of freedom
## Multiple R-squared:  0.1967,	Adjusted R-squared:  0.1885 
## F-statistic:    24 on 1 and 98 DF,  p-value: 3.808e-06
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
##       Min        1Q    Median        3Q       Max 
## -0.251614 -0.061564  0.008824  0.070517  0.211297 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.31513    0.01983 116.733  < 2e-16 ***
## x            0.24176    0.03440   7.028 2.81e-10 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09754 on 98 degrees of freedom
## Multiple R-squared:  0.3351,	Adjusted R-squared:  0.3284 
## F-statistic:  49.4 on 1 and 98 DF,  p-value: 2.807e-10
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
## -6.869 -1.456  2.480  5.994 11.719 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  18.0559     0.8888   20.32   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.124 on 99 degrees of freedom
## Multiple R-squared:  0.8065,	Adjusted R-squared:  0.8046 
## F-statistic: 412.7 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -6.869 -1.456  2.480  5.994 11.719 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  18.0559     0.8888   20.32   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.124 on 99 degrees of freedom
## Multiple R-squared:  0.8065,	Adjusted R-squared:  0.8046 
## F-statistic: 412.7 on 1 and 99 DF,  p-value: < 2.2e-16
```



## 重回帰モデル
説明変数として $w$ を加えたモデルを考える.
$$
y = \alpha + \beta x +\gamma w+ u
$$

暗黙に次の仮定を置いている.

+ $(w_i, x_i,y_i)$ は独立同一分布にしたがう.
+ $E[u_i]=0$ である.
+ $u_i$ と $(x_i, w_i)$ は独立である.
+ $u_i$ は正規分布にしたがう.
+ 多重共線性は存在しない. つまり $x_i$ は $w_i$ の一次変換で表せない.

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
## -2.77283 -0.77978 -0.04768  0.70921  2.56791 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.6740     0.2312  46.158  < 2e-16 ***
## x             2.8149     0.3524   7.989 2.84e-12 ***
## wT           -1.0076     0.2018  -4.993 2.62e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9989 on 97 degrees of freedom
## Multiple R-squared:  0.4722,	Adjusted R-squared:  0.4613 
## F-statistic: 43.39 on 2 and 97 DF,  p-value: 3.462e-14
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
## -2.34308 -0.71555  0.05383  0.78784  2.37998 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.1976     0.3515  29.013   <2e-16 ***
## x             2.3336     1.5964   1.462    0.147    
## I(x^2)        0.4321     1.5302   0.282    0.778    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.119 on 97 degrees of freedom
## Multiple R-squared:  0.3371,	Adjusted R-squared:  0.3235 
## F-statistic: 24.67 on 2 and 97 DF,  p-value: 2.185e-09
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
## -2.46977 -0.79130  0.05851  0.73540  2.38126 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9920     0.3047  36.076  < 2e-16 ***
## x             2.1710     0.5358   4.052 0.000103 ***
## wT           -1.5683     0.4063  -3.860 0.000206 ***
## x:wT          1.1215     0.7071   1.586 0.116016    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9912 on 96 degrees of freedom
## Multiple R-squared:  0.4857,	Adjusted R-squared:  0.4696 
## F-statistic: 30.22 on 3 and 96 DF,  p-value: 7.641e-14
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
## -2.46977 -0.79130  0.05851  0.73540  2.38126 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9920     0.3047  36.076  < 2e-16 ***
## x             2.1710     0.5358   4.052 0.000103 ***
## wT           -1.5683     0.4063  -3.860 0.000206 ***
## x:wT          1.1215     0.7071   1.586 0.116016    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9912 on 96 degrees of freedom
## Multiple R-squared:  0.4857,	Adjusted R-squared:  0.4696 
## F-statistic: 30.22 on 3 and 96 DF,  p-value: 7.641e-14
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
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 121.650                                  
## 2     96  94.308  2    27.342 13.916 4.932e-06 ***
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
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     96  94.308                                  
## 2     98 121.650 -2   -27.342 13.916 4.932e-06 ***
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
##   Res.Df     RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 121.650                                  
## 2     96  94.308  2    27.342 13.916 4.932e-06 ***
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
## (Intercept) 10.12197    0.22654 44.6801 < 2.2e-16 ***
## x            2.77046    0.39291  7.0512 1.774e-12 ***
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
## 2     96  2 27.832  9.043e-07 ***
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
## 2     96  2 13.916 4.932e-06 ***
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
## 1     98 121.650                                  
## 2     96  94.308  2    27.342 13.916 4.932e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


複数制約の検定としてLM検定というのもある.
制約付きの回帰分析を実行し, その残差を制約なしのモデルの説明変数に回帰する.
その決定係数に観測数を掛けた統計量が自由どが制約の数のカイ二乗分布にしたがうことが知られている.

```r
lmt <- lm(I(resid(fm1))~w*x,data=df)
(lmt <- nrow(df)*summary(lmt)$r.squared)
## [1] 1.687973e-29
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
##             Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.99195    0.32813 33.4988 < 2.2e-16 ***
## x            2.17104    0.66372  3.2710 0.0014891 ** 
## wT          -1.56834    0.41970 -3.7368 0.0003165 ***
## x:wT         1.12150    0.85043  1.3187 0.1903936    
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
## (Intercept) 10.99195    0.31259 35.1644 < 2.2e-16 ***
## x            2.17104    0.62801  3.4570 0.0008155 ***
## wT          -1.56834    0.40273 -3.8943 0.0001820 ***
## x:wT         1.12150    0.81094  1.3830 0.1698849    
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
## (Intercept) 10.12197    0.25129 40.2797 < 2.2e-16 ***
## x            2.77046    0.46452  5.9641 2.459e-09 ***
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
##   Res.Df Df      F    Pr(>F)    
## 1     98                        
## 2     96  2 15.435 1.541e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

通常はF検定であるが, カイ二乗検定を実施するには以下を実施すればよい.

```r
waldtest(fm0,fm1,vcov=vcovHC, test="Chisq")
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x + w + x:w
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1     98                         
## 2     96  2 30.871  1.979e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


### 分散均一の検定
誤差項が説明変数と独立のときと無相関のときでは標準誤差の推定量が異なっている.
正確にいうと, 条件付き分散が説明変数に依存するかどうかによって標準誤差の推定量が異なる. このことは分散均一と呼ばれている.

誤差項の分散が均一かどうかは検定可能である.
有名な検定方法としてBP検定というものがある.
BP検定は帰無仮説が分散均一で, 対立仮説が分散が説明変数と線形関係になっている場合の検定である.

残差の自乗を被説明変数として回帰分析をおこない,
その決定係数に観測数をかけたものが検定統計量となる.

```r
bpt <- lm(I(resid(fm1)^2)~w*x,data=df)
(bpt <- nrow(df)*summary(bpt)$r.squared)
## [1] 7.476128
1-pchisq(bpt,df=3)
## [1] 0.058175
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
## BP = 7.4761, df = 3, p-value = 0.05817
```

これまでのBPテストは誤差項の分散が説明変数の線形関係あることを暗黙に仮定している.
非線形性を考慮するために説明変数の二次項を導入した分散不均一性の検定をホワイト検定という.
説明変数が複数ある場合ホワイト検定は煩雑になるため, 
被説明変数の予測値を使って計算することがある.
そのときホワイトテストは以下で実施する.

```r
wht <- lm(I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2),data=df)
(wht <- nrow(df)*summary(wht)$r.squared)
## [1] 11.0468
1-pchisq(wht,df=2)
## [1] 0.003992252
```
ホワイト検定でも分散均一が示唆されている.

もしくは以下を実行する.

```r
bptest(fm1,I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2))
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
## BP = 11.047, df = 2, p-value = 0.003992
```

このように分散均一性は検定することが可能であるが, そもそも分散均一が疑われる場合は,
ロバスト分散で推定するので十分であるため最近の実証分析ではこの検定は実施されない.



