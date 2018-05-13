---
output: html_document
---
# 操作変数法



## データ

```r
library(AER)
library(wooldridge)
data("mroz", package="wooldridge")
df <- subset(mroz, inlf==1)
```

## 操作変数
これまで回帰モデルで一致推定量を得るためには次の仮定が必要であった.

1. 母集団が線形モデル
2. 標本が無作為抽出
3. 誤差項が平均ゼロで説明変数と無相関
4. 説明変数に多重共線性が存在しない

3つ目の説明変数が必ずしも成立しない場合の推定方法を紹介する.

そのために, 外生変数と内生変数と操作変数の3つの概念を導入する.
説明変数を外生変数と内生変数に分ける.
誤差項と相関が無い説明変数を **外生変数** といい,
誤差項と相関がある説明変数を **内生変数** という.
**操作変数** とは, 説明変数に含まれず, 説明変数と相関をもち, 誤差項と相関をもたない変数のことである.
なお操作変数の個数は内生変数の個数より多いと仮定する.

R においては次のコマンドを実行すればよい.
ここで被説明変数は `log(wage)`,
内生変数は `educ`,
操作変数は `fatheduc` である.


```r
fm  <- ivreg(log(wage)~educ|fatheduc, data=df)
coef(fm)
```

```
## (Intercept)        educ 
##  0.44110339  0.05917348
```

傾きの推定値は以下でも実行可能である.

```r
with(df, cov(log(wage),fatheduc)/cov(educ,fatheduc))
```

```
## [1] 0.05917348
```

## 2段階最小二乗法
説明変数が複数あり, 操作変数の数が内生変数の数以上のとき, 係数の一致推定量を得るには二段階最小自乗法を用いる.
二段階最小二乗法は次の手順で実行される:

1. それぞれの内生変数を外生変数と操作変数に回帰させて, その予測値を得る.
2. 被説明変数を外生変数と内生変数の予測値に回帰させて, その係数を得る.

この係数が一致推定量になるための条件は以下である.

+ 母集団が線形モデル
+ 標本が無作為抽出
+ 誤差項が平均ゼロで操作変数と外生変数に対して独立.
+ 操作変数は内生変数と相関をもつ.
+ 外生変数と内生変数の予測値に多重共線性が存在しない`

R においては次のコマンドを実行すればよい.
ここで被説明変数は `log(wage)`,
内生変数は `educ`, 外生変数は `expr`, `I(expr^2)`,
操作変数は `motheduc`, `fatheduc` である.


```r
fm  <- ivreg(log(wage)~educ+exper+I(exper^2)|
            exper+I(exper^2)+motheduc+fatheduc,
            data=df)
summary(fm)
```

```
## 
## Call:
## ivreg(formula = log(wage) ~ educ + exper + I(exper^2) | exper + 
##     I(exper^2) + motheduc + fatheduc, data = df)
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
ols1 <- lm(educ~exper+I(exper^2)+motheduc+fatheduc,  data = df)
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

### 複数制約の検定

帰無仮説が複数制約のワルド検定は以下のように実施する.
例えば, 2つの外生変数の係数がゼロのときの仮説検定をRで実行するには以下を実施する.


```r
fm0 <- ivreg(log(wage)~educ|motheduc+fatheduc,data=df)
waldtest(fm0,fm)
```

```
## Wald test
## 
## Model 1: log(wage) ~ educ | motheduc + fatheduc
## Model 2: log(wage) ~ educ + exper + I(exper^2) | exper + I(exper^2) + 
##     motheduc + fatheduc
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
## [1] 33.97987
```

```r
1-pchisq(lmt,df=3)
```

```
## [1] 2.000665e-07
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
##     I(exper^2) + motheduc + fatheduc, data = df)
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
ols0 <- lm(educ ~ exper + I(exper^2), data = df)
anova(ols0, ols1)
```

```
## Analysis of Variance Table
## 
## Model 1: educ ~ exper + I(exper^2)
## Model 2: educ ~ exper + I(exper^2) + motheduc + fatheduc
##   Res.Df    RSS Df Sum of Sq    F    Pr(>F)    
## 1    425 2219.2                                
## 2    423 1758.6  2    460.64 55.4 < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

### Du-Hausman 検定
Du-Hausman 検定は
帰無仮説が誤差項と説明変数が無相関, 対立仮説が誤差項と説明変数が相関ありの検定をおこなう.
帰無仮説のもと, OLSも2SLSも一致推定量である.
よって検定統計量のP値が十分小さいなら,
帰無仮説は棄却して, より効率的な最小二乗法を実施する.
そうでなければ操作変数法を選択する.

具体的には以下のF検定を実施する.

1. それぞれの内生変数を外生変数に回帰したときの残差を得る. (`resid(ols1)`)
2. 被説明変数を説明変数に回帰する (`ols3`)
3. 被説明変数を説明変数および先程の残差に回帰する (`ols4`)
4. これらの残差の係数はゼロであるという帰無仮説のもとF検定を実施する.

```r
ols3 <- lm(log(wage) ~ educ  + exper + I(exper^2), data = df)
ols4 <- update(ols3, . ~ . + resid(ols1))
anova(ols3,ols4)
```

```
## Analysis of Variance Table
## 
## Model 1: log(wage) ~ educ + exper + I(exper^2)
## Model 2: log(wage) ~ educ + exper + I(exper^2) + resid(ols1)
##   Res.Df    RSS Df Sum of Sq      F  Pr(>F)  
## 1    424 188.31                              
## 2    423 187.07  1     1.235 2.7926 0.09544 .
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

### Sargan 検定
Sargan 検定は
誤差項が操作変数 (および外生変数) と相関しているかどうかを検定する.
帰無仮説が相関が無い場合で, 対立仮説は相関がある場合である.

LM検定を実施する.

1. 二段階最小二乗法を実施したときの残差を得る. (`resid(fm)`)
2. 残差を外生変数および操作変数に回帰する. 
3. 回帰の決定係数に観測数を乗じたLM統計量をえる.
4. 検定統計量は, 帰無仮説のもと, 操作変数の数から内生変数の数を差し引いた自由度のカイ二乗分布にしたがう.


```r
jt <- lm(resid(fm)~exper+I(exper^2)+motheduc+fatheduc,data=df)
(jt <- nrow(df)*summary(jt)$r.squared)
```

```
## [1] 0.3780714
```

```r
1-pchisq(jt,df=1)
```

```
## [1] 0.5386372
```

## ロバスト分散
以上の分析は, 誤差項が操作変数と独立の場合の分析である.
独立でない場合, 推定量の分散が変わりうる.
そうした分散をロバスト分散という.
ロバスト分散にもとづく, 推定結果は次のコマンドで実施する.

```r
summary(fm, vcov = vcovHC, df = Inf)
```

```
## 
## Call:
## ivreg(formula = log(wage) ~ educ + exper + I(exper^2) | exper + 
##     I(exper^2) + motheduc + fatheduc, data = df)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3.0986 -0.3196  0.0551  0.3689  2.3493 
## 
## Coefficients:
##               Estimate Std. Error z value Pr(>|z|)   
## (Intercept)  0.0481003  0.4337795   0.111  0.91171   
## educ         0.0613966  0.0336597   1.824  0.06815 . 
## exper        0.0441704  0.0157661   2.802  0.00508 **
## I(exper^2)  -0.0008990  0.0004391  -2.047  0.04062 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.6747 on Inf degrees of freedom
## Multiple R-Squared: 0.1357,	Adjusted R-squared: 0.1296 
## Wald test: 18.11 on 3 DF,  p-value: 0.0004168
```

また次のコマンドで係数結果のみ出力可能である.

```r
coeftest(fm, vcov=vcovHC)
```

```
## 
## t test of coefficients:
## 
##                Estimate  Std. Error t value Pr(>|t|)   
## (Intercept)  0.04810030  0.43377952  0.1109 0.911759   
## educ         0.06139663  0.03365975  1.8240 0.068850 . 
## exper        0.04417039  0.01576605  2.8016 0.005318 **
## I(exper^2)  -0.00089897  0.00043908 -2.0474 0.041233 * 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

ロバスト分散のもとでの複数制約の検定は以下を実施する.

```r
waldtest(fm0,fm, vcov=vcovHC)
```

```
## Wald test
## 
## Model 1: log(wage) ~ educ | motheduc + fatheduc
## Model 2: log(wage) ~ educ + exper + I(exper^2) | exper + I(exper^2) + 
##     motheduc + fatheduc
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1    426                         
## 2    424  2 14.582  0.0006816 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```


### 分散不均一の検定
誤差項が操作変数と独立なら条件付き分散は操作変数に無関係で均一である.
これを利用して分散均一を帰無仮説に, 分散不均一を対立仮説にしたBP検定が実行可能である.
ただ通常のコマンド `bptest` では正しく実行できないので注意が必要である.


```r
bpt <- lm(I(resid(fm)^2)~exper + I(exper^2) + motheduc + fatheduc,data=df)
(bpt <- nrow(df)*summary(bpt)$r.squared)
```

```
## [1] 12.41758
```

```r
1-pchisq(bpt,df=4)
```

```
## [1] 0.01450172
```


