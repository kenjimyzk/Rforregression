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
##    10.51121     1.93195
# coefficients(fm)
```

残差は次のコマンドを実施する.

```r
head(resid(fm))
##           1           2           3           4           5           6 
## -0.87722518  0.18168918  0.41593104 -0.90564694  0.88160299  0.01965143
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.33626 11.66676 11.99121 11.66062 12.20678 10.72944
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 122.3837
sum(resid(fm)^2)
## [1] 122.3837
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
## -3.3385 -0.5172  0.0893  0.7840  2.2811 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5112     0.2455  42.822  < 2e-16 ***
## x             1.9319     0.4135   4.673 9.48e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.118 on 98 degrees of freedom
## Multiple R-squared:  0.1822,	Adjusted R-squared:  0.1739 
## F-statistic: 21.83 on 1 and 98 DF,  p-value: 9.484e-06
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
##             Estimate Std. Error   t value     Pr(>|t|)
## (Intercept) 10.51121  0.2454649 42.821634 2.988209e-65
## x            1.93195  0.4134601  4.672638 9.483566e-06
# coefficients(summary(fm))
```

残差の標準誤差

```r
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.117503
with(summary(fm),sigma)
## [1] 1.117503
```

決定係数は次のようにして計算する.

```r
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.182199
with(summary(fm),r.squared)
## [1] 0.182199
```

調整済み決定係数は次のようにして計算する.

```r
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.173854
with(summary(fm),adj.r.squared)
## [1] 0.173854
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
## -3.0726 -0.4988  0.1321  0.7683  2.2202 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  12.0356     0.1518  79.283  < 2e-16 ***
## log(x)        0.5668     0.1167   4.856 4.53e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.109 on 98 degrees of freedom
## Multiple R-squared:  0.194,	Adjusted R-squared:  0.1857 
## F-statistic: 23.58 on 1 and 98 DF,  p-value: 4.53e-06
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
## -6.978 -1.236  2.270  5.458 11.289 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x  17.6958     0.8315   21.28   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 4.936 on 99 degrees of freedom
## Multiple R-squared:  0.8206,	Adjusted R-squared:  0.8188 
## F-statistic:   453 on 1 and 99 DF,  p-value: < 2.2e-16
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
##     Min      1Q  Median      3Q     Max 
## -2.9710 -0.6846  0.1717  0.6148  1.8278 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.1254     0.2411  46.148  < 2e-16 ***
## x             1.8461     0.3618   5.103 1.66e-06 ***
## wT           -1.0939     0.1957  -5.590 2.09e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9769 on 97 degrees of freedom
## Multiple R-squared:  0.3814,	Adjusted R-squared:  0.3687 
## F-statistic: 29.91 on 2 and 97 DF,  p-value: 7.625e-11
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
## -3.0422 -0.4153  0.1532  0.7222  2.2090 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.1113     0.3799  26.619   <2e-16 ***
## x             4.0913     1.6233   2.520   0.0134 *  
## I(x^2)       -2.1037     1.5298  -1.375   0.1722    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.112 on 97 degrees of freedom
## Multiple R-squared:  0.1978,	Adjusted R-squared:  0.1813 
## F-statistic: 11.96 on 2 and 97 DF,  p-value: 2.274e-05
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
## -2.8552 -0.6378  0.1203  0.6419  2.0275 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.5306     0.3460  33.328  < 2e-16 ***
## x             1.0966     0.5855   1.873 0.064111 .  
## wT           -1.7324     0.4393  -3.944 0.000153 ***
## x:wT          1.2003     0.7409   1.620 0.108482    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9688 on 96 degrees of freedom
## Multiple R-squared:  0.3979,	Adjusted R-squared:  0.3791 
## F-statistic: 21.15 on 3 and 96 DF,  p-value: 1.34e-10
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
## -2.8552 -0.6378  0.1203  0.6419  2.0275 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.5306     0.3460  33.328  < 2e-16 ***
## x             1.0966     0.5855   1.873 0.064111 .  
## wT           -1.7324     0.4393  -3.944 0.000153 ***
## x:wT          1.2003     0.7409   1.620 0.108482    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9688 on 96 degrees of freedom
## Multiple R-squared:  0.3979,	Adjusted R-squared:  0.3791 
## F-statistic: 21.15 on 3 and 96 DF,  p-value: 1.34e-10
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
## 1     98 122.384                                  
## 2     96  90.105  2    32.279 17.196 4.141e-07 ***
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
## 1     96  90.105                                  
## 2     98 122.384 -2   -32.279 17.196 4.141e-07 ***
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
## 1     98 122.384                                  
## 2     96  90.105  2    32.279 17.196 4.141e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

