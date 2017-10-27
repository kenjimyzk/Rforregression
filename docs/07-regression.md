---
output: html_document
---

# データ生成


仮想的に以下のようにデータを生成する.

```r
N <- 100
x<-runif(N)
w<-sample(c("H","T"),N,replace=TRUE)
y <- 10 + 2*x + ifelse(w=="H",1,0) + rnorm(N)
df <- data.frame(w,x,y)
```

## 単回帰モデル
次の単回帰モデルを考えてる.
$x$ は説明変数で, 
$y$ は被説明変数である.
$u$ は誤差項である.
パラメータとして $\alpha$ は切片パラメータ, $\beta$ は傾きパラメータである.

$$
y = \alpha + \beta x + u
$$

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
##   10.833137    1.221059
# coefficients(fm)
```

残差は次のコマンドを実施する.

```r
head(resid(fm))
##          1          2          3          4          5          6 
## -0.6396949  0.2600457 -1.2399773 -1.1219448 -0.5234153  0.1615355
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.85603 11.34434 11.34370 11.20312 11.95007 11.69020
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 105.6954
sum(resid(fm)^2)
## [1] 105.6954
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
## -2.43787 -0.71718 -0.03723  0.61788  2.79511 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.8331     0.2088   51.88  < 2e-16 ***
## x             1.2211     0.3689    3.31  0.00131 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.039 on 98 degrees of freedom
## Multiple R-squared:  0.1006,	Adjusted R-squared:  0.09137 
## F-statistic: 10.96 on 1 and 98 DF,  p-value: 0.001307
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
## (Intercept) 10.833137  0.2088022 51.882284 4.467267e-73
## x            1.221059  0.3689086  3.309924 1.306750e-03
# coefficients(summary(fm))
```

残差の標準誤差

```r
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.03852
with(summary(fm),sigma)
## [1] 1.03852
```

決定係数は次のようにして計算する.

```r
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.100551
with(summary(fm),r.squared)
## [1] 0.100551
```

調整済み決定係数は次のようにして計算する.

```r
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.09137298
with(summary(fm),adj.r.squared)
## [1] 0.09137298
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
## -2.56691 -0.76114 -0.02482  0.57312  3.01456 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.6902     0.1538  76.003   <2e-16 ***
## log(x)        0.2549     0.1098   2.323   0.0223 *  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.066 on 98 degrees of freedom
## Multiple R-squared:  0.05218,	Adjusted R-squared:  0.04251 
## F-statistic: 5.395 on 1 and 98 DF,  p-value: 0.02226
```

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
## -5.895 -1.530  2.950  6.291 13.366 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   17.826      0.974    18.3   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.513 on 99 degrees of freedom
## Multiple R-squared:  0.7719,	Adjusted R-squared:  0.7695 
## F-statistic: 334.9 on 1 and 99 DF,  p-value: < 2.2e-16
```

## 重回帰モデル
説明変数として $w$ を加えたモデルを考える.
$$
y = \alpha + \beta x +\gamma w+ u
$$

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
## -1.85158 -0.51554  0.01107  0.54670  2.40866 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.2157     0.1912  58.649  < 2e-16 ***
## x             1.4651     0.3206   4.570 1.44e-05 ***
## wT           -1.0690     0.1808  -5.912 5.05e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.895 on 97 degrees of freedom
## Multiple R-squared:  0.3388,	Adjusted R-squared:  0.3251 
## F-statistic: 24.85 on 2 and 97 DF,  p-value: 1.937e-09
```

R の特徴は因子もとくに変換することなくダミー変数として扱える.

### 自乗項
説明変数として自乗項を加えたモデルを考える.
$$
y = \alpha + \beta x + \gamma x^2 + u
$$



```r
fm <- lm(y~x+I(x^2),data=df)
summary(fm)
## 
## Call:
## lm(formula = y ~ x + I(x^2), data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -2.3236 -0.7064 -0.1047  0.6179  2.6184 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.1007     0.3172  34.995   <2e-16 ***
## x            -0.3634     1.4627  -0.248    0.804    
## I(x^2)        1.5934     1.4235   1.119    0.266    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.037 on 97 degrees of freedom
## Multiple R-squared:  0.112,	Adjusted R-squared:  0.09371 
## F-statistic: 6.118 on 2 and 97 DF,  p-value: 0.003144
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
## -1.91261 -0.49145  0.00374  0.58261  2.28133 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.3476     0.2292  49.506  < 2e-16 ***
## x             1.1764     0.4235   2.778 0.006582 ** 
## wT           -1.4056     0.3700  -3.799 0.000255 ***
## x:wT          0.6754     0.6478   1.043 0.299749    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.8946 on 96 degrees of freedom
## Multiple R-squared:  0.3462,	Adjusted R-squared:  0.3257 
## F-statistic: 16.94 on 3 and 96 DF,  p-value: 6.552e-09
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
## -1.91261 -0.49145  0.00374  0.58261  2.28133 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.3476     0.2292  49.506  < 2e-16 ***
## x             1.1764     0.4235   2.778 0.006582 ** 
## wT           -1.4056     0.3700  -3.799 0.000255 ***
## x:wT          0.6754     0.6478   1.043 0.299749    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.8946 on 96 degrees of freedom
## Multiple R-squared:  0.3462,	Adjusted R-squared:  0.3257 
## F-statistic: 16.94 on 3 and 96 DF,  p-value: 6.552e-09
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
## 1     98 105.695                                  
## 2     96  76.832  2    28.864 18.032 2.245e-07 ***
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
## 1     96  76.832                                  
## 2     98 105.695 -2   -28.864 18.032 2.245e-07 ***
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
## 1     98 105.695                                  
## 2     96  76.832  2    28.864 18.032 2.245e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

