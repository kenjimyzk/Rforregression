---
output: html_document
---
# 操作変数法



## データ

```r
library(AER)
library(PoEdata)
data("mroz", package="PoEdata")
df <- subset(mroz, lfp==1)
```

## 操作変数
これまで回帰モデルで一致推定量を得るためには次の仮定が必要であった.

+ 母集団が線形モデル
+ 標本が無作為抽出
+ 誤差項が平均ゼロで説明変数と無相関
+ 説明変数に多重共線性が存在しない

3つ目の説明変数が必ずしも成立しない場合の推定方法を紹介する.
そのために, 操作変数という概念を導入する.
また説明変数を外生変数と内生変数の2つに分ける.
誤差項と相関が無い説明変数を外生変数といい,
誤差項と相関がある説明変数を内生変数という.

操作変数とは, 説明変数に含まれず, 説明変数と相関をもち, 誤差項と相関をもたない変数のことである.
また操作変数の個数は内生変数の個数より多いと仮定する.

## 2段階最小二乗法
操作変数をつかって, 係数の一致推定量を得るには二段階最小自乗法を用いる.
二段階最小二乗法は次の手順で実行される:

1. それぞれの内生変数を外生変数と操作変数に回帰させて, その予測値を得る.
2. 被説明変数を外生変数と内生変数の予測値に回帰させてその係数を得る.

この係数が一致推定量になるための条件は以下である.

+ 母集団が線形モデル
+ 標本が無作為抽出
+ 誤差項が平均ゼロで操作変数と外生変数に対して独立.
+ 操作変数は内生変数と相関をもつ.
+ 外生変数と内生変数の予測値に多重共線性が存在しない`

R においては次のコマンドを実行すればよい.
ここで被説明変数は `log(wage)`,
内生変数は `educ`, 外生変数は `expr`, `I(expr^2)`,
操作変数は `mothereduc`, `fathereduc` である.


```r
fm  <- ivreg(log(wage)~educ+exper+I(exper^2)|
            exper+I(exper^2)+mothereduc+fathereduc,
            data=df)
summary(fm)
```

```
## 
## Call:
## ivreg(formula = log(wage) ~ educ + exper + I(exper^2) | exper + 
##     I(exper^2) + mothereduc + fathereduc, data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.0986 -0.3196  0.0551  0.3689  2.3493 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)   
## (Intercept)  0.0481003  0.4003281   0.120  0.90442   
## educ         0.0613966  0.0314367   1.953  0.05147 . 
## exper        0.0441704  0.0134325   3.288  0.00109 **
## I(exper^2)  -0.0008990  0.0004017  -2.238  0.02574 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.6747 on 424 degrees of freedom
## Multiple R-Squared: 0.1357,	Adjusted R-squared: 0.1296 
## Wald test: 8.141 on 3 and 424 DF,  p-value: 2.787e-05
```

実際の二段階最小二乗法でも確認できる.
ただし標準誤差の値が異なっている.
なぜなら残差は内生変数および外生変数から算出させる必要があるが,
以下のやりかただと内生変数の予測値および外生変数から算出するためである.


```r
ols1 <- lm(educ~exper+I(exper^2)+mothereduc+fathereduc,  data = df)
ols2 <- lm(log(wage)~fitted(ols1)+exper+I(exper^2),  data = df)
summary(ols2)
```

```
## 
## Call:
## lm(formula = log(wage) ~ fitted(ols1) + exper + I(exper^2), data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.1631 -0.3539  0.0326  0.3818  2.3727 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)   
## (Intercept)   0.0481003  0.4197565   0.115  0.90882   
## fitted(ols1)  0.0613966  0.0329624   1.863  0.06321 . 
## exper         0.0441704  0.0140844   3.136  0.00183 **
## I(exper^2)   -0.0008990  0.0004212  -2.134  0.03338 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.7075 on 424 degrees of freedom
## Multiple R-squared:  0.04978,	Adjusted R-squared:  0.04306 
## F-statistic: 7.405 on 3 and 424 DF,  p-value: 7.615e-05
```

## 複数制約の検定

帰無仮説が複数制約のワルド検定は以下のように実施する.
例えば, 2つの外生変数の係数がゼロのときの仮説検定をRで実行するには以下を実施する.


```r
fm0 <- ivreg(log(wage)~educ|mothereduc+fathereduc,data=df)
waldtest(fm0,fm)
```

```
## Wald test
## 
## Model 1: log(wage) ~ educ | mothereduc + fathereduc
## Model 2: log(wage) ~ educ + exper + I(exper^2) | exper + I(exper^2) + 
##     mothereduc + fathereduc
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1    426                         
## 2    424  2 19.639  5.439e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

LM検定は以下のように実施すればよい.

```r
lmt <- lm(resid(fm0)~educ + exper + I(exper^2) ,data=df)
(lmt <- nrow(df)*summary(lmt)$r.squared)
```

```
## [1] 33.97988
```

```r
1-pchisq(lmt,df=3)
```

```
## [1] 2.000664e-07
```

## 特定化検定
またいくつかの特定化検定も以下のコマンドで実施できる.

```r
summary(fm, diagnostics = TRUE)
```

```
## 
## Call:
## ivreg(formula = log(wage) ~ educ + exper + I(exper^2) | exper + 
##     I(exper^2) + mothereduc + fathereduc, data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.0986 -0.3196  0.0551  0.3689  2.3493 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)   
## (Intercept)  0.0481003  0.4003281   0.120  0.90442   
## educ         0.0613966  0.0314367   1.953  0.05147 . 
## exper        0.0441704  0.0134325   3.288  0.00109 **
## I(exper^2)  -0.0008990  0.0004017  -2.238  0.02574 * 
## 
## Diagnostic tests:
##                  df1 df2 statistic p-value    
## Weak instruments   2 423    55.400  <2e-16 ***
## Wu-Hausman         1 423     2.793  0.0954 .  
## Sargan             1  NA     0.378  0.5386    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.6747 on 424 degrees of freedom
## Multiple R-Squared: 0.1357,	Adjusted R-squared: 0.1296 
## Wald test: 8.141 on 3 and 424 DF,  p-value: 2.787e-05
```

### Weak instruments
操作変数が内生変数と弱い相関関係しかない場合, 弱操作変数という.
それぞれの内生変数に対して, 
帰無仮説を内生変数を外生変数のみ回帰させたモデルとし,
対立仮説を内生変数を外生変数および操作変数のみ回帰させたモデルとし, F検定を実施する.

```r
ols0 <- lm(educ~exper+I(exper^2),  data = df)
anova(ols0,ols1)
```

```
## Analysis of Variance Table
## 
## Model 1: educ ~ exper + I(exper^2)
## Model 2: educ ~ exper + I(exper^2) + mothereduc + fathereduc
##   Res.Df    RSS Df Sum of Sq    F    Pr(>F)    
## 1    425 2219.2                                
## 2    423 1758.6  2    460.64 55.4 < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

### Du-Hausman
Du-Hausman 検定は
帰無仮説が誤差項と説明変数が無相関, 対立仮説が誤差項と説明変数が相関ありの検定をおこなう.
帰無仮説のもと, OLSも2SLSも一致推定量である.
よって検定統計量のP値が十分小さいなら,
帰無仮説は棄却して, より効率的な最小二乗法を実施する.
そうでなければ操作変数法を選択する.

### Sargan
Sargan 検定は
誤差項が操作変数 (および外生変数) と相関しているかどうかを検定する.
帰無仮説が相関が無い場合で, 対立仮説は相関がある場合である.
LM検定を実施する.

```r
jt <- lm(resid(fm)~exper+I(exper^2)+mothereduc+fathereduc,data=df)
(jt <- nrow(df)*summary(jt)$r.squared)
```

```
## [1] 0.3780715
```

```r
1-pchisq(jt,df=3)
```

```
## [1] 0.9447342
```


## ロバスト分散
以上の分析は, 誤差項が操作変数と独立の場合の分析である.
独立出ない場合, 推定量の分散が変わりうる.
そうした分散をロバスト分散という.
ロバスト分散は次のコマンドで実施する.

```r
summary(fm, vcov = sandwich, df = Inf)
```

```
## 
## Call:
## ivreg(formula = log(wage) ~ educ + exper + I(exper^2) | exper + 
##     I(exper^2) + mothereduc + fathereduc, data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.0986 -0.3196  0.0551  0.3689  2.3493 
## 
## Coefficients:
##               Estimate Std. Error z value Pr(>|z|)   
## (Intercept)  0.0481003  0.4277846   0.112  0.91047   
## educ         0.0613966  0.0331824   1.850  0.06427 . 
## exper        0.0441704  0.0154736   2.855  0.00431 **
## I(exper^2)  -0.0008990  0.0004281  -2.100  0.03572 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.6747 on Inf degrees of freedom
## Multiple R-Squared: 0.1357,	Adjusted R-squared: 0.1296 
## Wald test: 18.61 on 3 DF,  p-value: 0.0003291
```

また次のコマンドでも可能である.

```r
coeftest(fm, vcov=sandwich)
```

```
## 
## t test of coefficients:
## 
##                Estimate  Std. Error t value Pr(>|t|)   
## (Intercept)  0.04810030  0.42778460  0.1124 0.910527   
## educ         0.06139663  0.03318243  1.8503 0.064969 . 
## exper        0.04417039  0.01547356  2.8546 0.004521 **
## I(exper^2)  -0.00089897  0.00042807 -2.1001 0.036314 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

ロバスト分散のもとでの複数制約の検定は以下を実施する.

```r
waldtest(fm0,fm, vcov=sandwich)
```

```
## Wald test
## 
## Model 1: log(wage) ~ educ | mothereduc + fathereduc
## Model 2: log(wage) ~ educ + exper + I(exper^2) | exper + I(exper^2) + 
##     mothereduc + fathereduc
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1    426                         
## 2    424  2 15.018  0.0005483 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


## 分散不均一の検定
誤差項が操作変数と独立なら条件付き分散は操作変数に無関係で均一である.
これを利用して分散均一を帰無仮説に, 分散不均一を対立仮説にしたBP検定が実行可能である.
ただ通常のコマンド `bptest` では正しく実行できないので注意が必要である.


```r
bpt <- lm(I(resid(fm)^2)~exper + I(exper^2) + mothereduc + fathereduc,data=df)
(bpt <- nrow(df)*summary(bpt)$r.squared)
```

```
## [1] 12.41758
```

```r
1-pchisq(bpt,df=3)
```

```
## [1] 0.006081394
```

