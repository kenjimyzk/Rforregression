---
output: html_document
---
# パネル分析


## データ

```r
library(plm)
```

```
##  要求されたパッケージ Formula をロード中です
```

```r
data("Grunfeld", package="plm")
head(Grunfeld)
```

```
##   firm year   inv  value capital
## 1    1 1935 317.6 3078.5     2.8
## 2    1 1936 391.8 4661.7    52.6
## 3    1 1937 410.6 5387.1   156.9
## 4    1 1938 257.7 2792.2   209.2
## 5    1 1939 330.8 4313.2   203.4
## 6    1 1940 461.2 4643.9   207.2
```


## プーリングOLS
$$
inv_{it} = \beta_0 + \beta_1 value_{it} + \beta_2 capital_{it} + u_{it}
$$

誤差項 $u_{it}$ は $i$ についても $t$ についても独立同一分布と仮定する.
さらに誤差項は説明変数と独立である.

この推計は以下のようにする.

```r
gp <- plm(inv ~ value + capital, data = Grunfeld, model = "pooling")
summary(gp)
```

```
## Pooling Model
## 
## Call:
## plm(formula = inv ~ value + capital, data = Grunfeld, model = "pooling")
## 
## Balanced Panel: n=10, T=20, N=200
## 
## Residuals :
##      Min.   1st Qu.    Median   3rd Qu.      Max. 
## -291.6757  -30.0137    5.3033   34.8293  369.4464 
## 
## Coefficients :
##                Estimate  Std. Error t-value  Pr(>|t|)    
## (Intercept) -42.7143694   9.5116760 -4.4907 1.207e-05 ***
## value         0.1155622   0.0058357 19.8026 < 2.2e-16 ***
## capital       0.2306785   0.0254758  9.0548 < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    9359900
## Residual Sum of Squares: 1755900
## R-Squared:      0.81241
## Adj. R-Squared: 0.8105
## F-statistic: 426.576 on 2 and 197 DF, p-value: < 2.22e-16
```

これは以下の回帰分析と同じである.

```r
summary(lm(inv ~ value + capital, data = Grunfeld))
```

```
## 
## Call:
## lm(formula = inv ~ value + capital, data = Grunfeld)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -291.68  -30.01    5.30   34.83  369.45 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -42.714369   9.511676  -4.491 1.21e-05 ***
## value         0.115562   0.005836  19.803  < 2e-16 ***
## capital       0.230678   0.025476   9.055  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 94.41 on 197 degrees of freedom
## Multiple R-squared:  0.8124,	Adjusted R-squared:  0.8105 
## F-statistic: 426.6 on 2 and 197 DF,  p-value: < 2.2e-16
```

## 固定効果モデル
次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it} +\alpha_i + u_{it}
$$
この $\alpha_i$ は固定効果と呼ばれている.
$\alpha_i$ は時間 $t$ に対して一定である.
$\alpha_i$ は誤差項と相関があるもしれない.

この推計は以下のようにする.

```r
gi <- plm(inv ~ value + capital, data = Grunfeld, model = "within")
summary(gi)
```

```
## Oneway (individual) effect Within Model
## 
## Call:
## plm(formula = inv ~ value + capital, data = Grunfeld, model = "within")
## 
## Balanced Panel: n=10, T=20, N=200
## 
## Residuals :
##       Min.    1st Qu.     Median    3rd Qu.       Max. 
## -184.00857  -17.64316    0.56337   19.19222  250.70974 
## 
## Coefficients :
##         Estimate Std. Error t-value  Pr(>|t|)    
## value   0.110124   0.011857  9.2879 < 2.2e-16 ***
## capital 0.310065   0.017355 17.8666 < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    2244400
## Residual Sum of Squares: 523480
## R-Squared:      0.76676
## Adj. R-Squared: 0.75311
## F-statistic: 309.014 on 2 and 188 DF, p-value: < 2.22e-16
```

固定効果は以下のコマンドで確かめられる.

```r
fixef(gi)
```

```
##           1           2           3           4           5           6 
##  -70.296717  101.905814 -235.571841  -27.809295 -114.616813  -23.161295 
##           7           8           9          10 
##  -66.553474  -57.545657  -87.222272   -6.567844
```

これは以下の回帰分析と同じである.

```r
summary(lm(inv ~ value + capital+0+factor(firm), data = Grunfeld))
```

```
## 
## Call:
## lm(formula = inv ~ value + capital + 0 + factor(firm), data = Grunfeld)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -184.009  -17.643    0.563   19.192  250.710 
## 
## Coefficients:
##                  Estimate Std. Error t value Pr(>|t|)    
## value             0.11012    0.01186   9.288  < 2e-16 ***
## capital           0.31007    0.01735  17.867  < 2e-16 ***
## factor(firm)1   -70.29672   49.70796  -1.414   0.1590    
## factor(firm)2   101.90581   24.93832   4.086 6.49e-05 ***
## factor(firm)3  -235.57184   24.43162  -9.642  < 2e-16 ***
## factor(firm)4   -27.80929   14.07775  -1.975   0.0497 *  
## factor(firm)5  -114.61681   14.16543  -8.091 7.14e-14 ***
## factor(firm)6   -23.16130   12.66874  -1.828   0.0691 .  
## factor(firm)7   -66.55347   12.84297  -5.182 5.63e-07 ***
## factor(firm)8   -57.54566   13.99315  -4.112 5.85e-05 ***
## factor(firm)9   -87.22227   12.89189  -6.766 1.63e-10 ***
## factor(firm)10   -6.56784   11.82689  -0.555   0.5793    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 52.77 on 188 degrees of freedom
## Multiple R-squared:  0.9616,	Adjusted R-squared:  0.9591 
## F-statistic:   392 on 12 and 188 DF,  p-value: < 2.2e-16
```
決定係数が大きく異なっていることに注意されたい.

次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it}+ \gamma_t +\alpha_i + u_{it}
$$
この $\gamma_t$ は時間効果と呼ばれている.

この推計は以下のようにする.

```r
gt <- plm(inv ~ value + capital, data = Grunfeld, effect="twoways",model = "within")
summary(gt)
```

```
## Twoways effects Within Model
## 
## Call:
## plm(formula = inv ~ value + capital, data = Grunfeld, effect = "twoways", 
##     model = "within")
## 
## Balanced Panel: n=10, T=20, N=200
## 
## Residuals :
##      Min.   1st Qu.    Median   3rd Qu.      Max. 
## -162.6094  -19.4710   -1.2669   19.1277  211.8420 
## 
## Coefficients :
##         Estimate Std. Error t-value  Pr(>|t|)    
## value   0.117716   0.013751  8.5604 6.653e-15 ***
## capital 0.357916   0.022719 15.7540 < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    1615600
## Residual Sum of Squares: 452150
## R-Squared:      0.72015
## Adj. R-Squared: 0.67047
## F-statistic: 217.442 on 2 and 169 DF, p-value: < 2.22e-16
```

これは以下の回帰分析と同じである.

```r
summary(lm(inv ~ value + capital+0+factor(firm)+factor(year), data = Grunfeld))
```

```
## 
## Call:
## lm(formula = inv ~ value + capital + 0 + factor(firm) + factor(year), 
##     data = Grunfeld)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -162.609  -19.471   -1.267   19.128  211.842 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## value               0.11772    0.01375   8.560 6.65e-15 ***
## capital             0.35792    0.02272  15.754  < 2e-16 ***
## factor(firm)1     -86.90023   56.04663  -1.550 0.122893    
## factor(firm)2     120.15401   29.16688   4.120 5.93e-05 ***
## factor(firm)3    -222.13103   28.59744  -7.768 7.37e-13 ***
## factor(firm)4       8.45361   20.41784   0.414 0.679377    
## factor(firm)5     -92.33883   20.91106  -4.416 1.79e-05 ***
## factor(firm)6      15.98841   19.88487   0.804 0.422498    
## factor(firm)7     -35.43362   20.17003  -1.757 0.080772 .  
## factor(firm)8     -19.40972   20.49076  -0.947 0.344868    
## factor(firm)9     -56.68267   19.81211  -2.861 0.004756 ** 
## factor(firm)10     39.93689   20.40337   1.957 0.051951 .  
## factor(year)1936  -19.19741   23.67586  -0.811 0.418596    
## factor(year)1937  -40.69001   24.69541  -1.648 0.101277    
## factor(year)1938  -39.22640   23.23594  -1.688 0.093221 .  
## factor(year)1939  -69.47029   23.65607  -2.937 0.003780 ** 
## factor(year)1940  -44.23508   23.80979  -1.858 0.064930 .  
## factor(year)1941  -18.80446   23.69400  -0.794 0.428519    
## factor(year)1942  -21.13979   23.38163  -0.904 0.367219    
## factor(year)1943  -42.97762   23.55287  -1.825 0.069808 .  
## factor(year)1944  -43.09877   23.61020  -1.825 0.069701 .  
## factor(year)1945  -55.68304   23.89562  -2.330 0.020974 *  
## factor(year)1946  -31.16928   24.11598  -1.292 0.197957    
## factor(year)1947  -39.39224   23.78368  -1.656 0.099522 .  
## factor(year)1948  -43.71651   23.96965  -1.824 0.069945 .  
## factor(year)1949  -73.49510   24.18292  -3.039 0.002750 ** 
## factor(year)1950  -75.89611   24.34553  -3.117 0.002144 ** 
## factor(year)1951  -62.48091   24.86425  -2.513 0.012911 *  
## factor(year)1952  -64.63234   25.34950  -2.550 0.011672 *  
## factor(year)1953  -67.71797   26.61108  -2.545 0.011832 *  
## factor(year)1954  -93.52622   27.10786  -3.450 0.000708 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 51.72 on 169 degrees of freedom
## Multiple R-squared:  0.9668,	Adjusted R-squared:  0.9607 
## F-statistic: 158.8 on 31 and 169 DF,  p-value: < 2.2e-16
```
決定係数が大きく異なっていることに注意されたい.

時間効果が有効かどうかはワルド検定を実施する.

```r
git<-update(gi,.~.+factor(year))
library(lmtest)
```

```
##  要求されたパッケージ zoo をロード中です
```

```
## 
##  次のパッケージを付け加えます: 'zoo'
```

```
##  以下のオブジェクトは 'package:base' からマスクされています: 
## 
##      as.Date, as.Date.numeric
```

```r
waldtest(gi,git)
```

```
## Wald test
## 
## Model 1: inv ~ value + capital
## Model 2: inv ~ value + capital + factor(year)
##   Res.Df Df  Chisq Pr(>Chisq)
## 1    188                     
## 2    169 19 26.662     0.1128
```


## 固定効果VSプーリングOLS
帰無仮説が固定効果, 対立仮説が固定効果モデルの検定は以下のコマンドを実行すればよい.

```r
pFtest(gi,gp)
```

```
## 
## 	F test for individual effects
## 
## data:  inv ~ value + capital
## F = 49.177, df1 = 9, df2 = 188, p-value < 2.2e-16
## alternative hypothesis: significant effects
```

時間効果モデルの場合は次のようにする.

```r
gpt <- plm(inv ~ value + capital + factor(year), data = Grunfeld, model = "pooling")
pFtest(gt,gpt)
```

```
## 
## 	F test for twoways effects
## 
## data:  inv ~ value + capital
## F = 52.362, df1 = 9, df2 = 169, p-value < 2.2e-16
## alternative hypothesis: significant effects
```

## 変量効果モデル
次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it} +\alpha_i + u_{it}
$$
この $\alpha_i$ は変量効果と呼ばれている.
$\alpha_i$ は時間 $t$ について一定であるが, $i$ について独立同一分布の確率変数にしたがう.
さらに $\alpha_i$ は説明変数と無相関であることが必要である.
このため固定効果モデルと違い, 欠落変数バイアスを除去することができない.
しかしながら, $i$ についてのダミー変数を付け加えることができる.

変量効果モデルは次のコマンドで実施する.

```r
gr <- plm(inv ~ value + capital, data = Grunfeld, model = "random")
summary(gr)
```

```
## Oneway (individual) effect Random Effect Model 
##    (Swamy-Arora's transformation)
## 
## Call:
## plm(formula = inv ~ value + capital, data = Grunfeld, model = "random")
## 
## Balanced Panel: n=10, T=20, N=200
## 
## Effects:
##                   var std.dev share
## idiosyncratic 2784.46   52.77 0.282
## individual    7089.80   84.20 0.718
## theta:  0.8612  
## 
## Residuals :
##      Min.   1st Qu.    Median   3rd Qu.      Max. 
## -177.6063  -19.7350    4.6851   19.5105  252.8743 
## 
## Coefficients :
##               Estimate Std. Error t-value Pr(>|t|)    
## (Intercept) -57.834415  28.898935 -2.0013  0.04674 *  
## value         0.109781   0.010493 10.4627  < 2e-16 ***
## capital       0.308113   0.017180 17.9339  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    2381400
## Residual Sum of Squares: 548900
## R-Squared:      0.7695
## Adj. R-Squared: 0.76716
## F-statistic: 328.837 on 2 and 197 DF, p-value: < 2.22e-16
```

時間効果のある変量モデルとは次のコマンドを実施する.

```r
grt <- plm(inv ~ value + capital + factor(year), data = Grunfeld, model = "random")
summary(grt)
```

```
## Oneway (individual) effect Random Effect Model 
##    (Swamy-Arora's transformation)
## 
## Call:
## plm(formula = inv ~ value + capital + factor(year), data = Grunfeld, 
##     model = "random")
## 
## Balanced Panel: n=10, T=20, N=200
## 
## Effects:
##                   var std.dev share
## idiosyncratic 2675.43   51.72 0.264
## individual    7458.35   86.36 0.736
## theta:  0.8673  
## 
## Residuals :
##        Min.     1st Qu.      Median     3rd Qu.        Max. 
## -160.249203  -19.705021   -0.023872   19.282175  214.023744 
## 
## Coefficients :
##                    Estimate Std. Error t-value  Pr(>|t|)    
## (Intercept)      -29.932675  32.908999 -0.9096  0.364285    
## value              0.113911   0.011820  9.6374 < 2.2e-16 ***
## capital            0.354516   0.022573 15.7055 < 2.2e-16 ***
## factor(year)1936 -17.741195  23.585305 -0.7522  0.452916    
## factor(year)1937 -38.098671  24.338353 -1.5654  0.119271    
## factor(year)1938 -38.432863  23.272433 -1.6514  0.100413    
## factor(year)1939 -67.734952  23.577926 -2.8728  0.004563 ** 
## factor(year)1940 -42.284029  23.690295 -1.7849  0.075986 .  
## factor(year)1941 -16.967455  23.613424 -0.7186  0.473360    
## factor(year)1942 -19.999001  23.411276 -0.8542  0.394117    
## factor(year)1943 -41.368571  23.535549 -1.7577  0.080517 .  
## factor(year)1944 -41.371185  23.574297 -1.7549  0.080992 .  
## factor(year)1945 -53.503398  23.781401 -2.2498  0.025686 *  
## factor(year)1946 -28.697568  23.948970 -1.1983  0.232401    
## factor(year)1947 -37.720827  23.801945 -1.5848  0.114792    
## factor(year)1948 -42.020754  23.997685 -1.7510  0.081663 .  
## factor(year)1949 -71.600868  24.205455 -2.9580  0.003517 ** 
## factor(year)1950 -73.706966  24.348310 -3.0272  0.002835 ** 
## factor(year)1951 -59.337596  24.727838 -2.3996  0.017444 *  
## factor(year)1952 -61.111948  25.183631 -2.4267  0.016236 *  
## factor(year)1953 -63.078394  26.233291 -2.4045  0.017219 *  
## factor(year)1954 -88.765641  26.797397 -3.3125  0.001120 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    2369700
## Residual Sum of Squares: 478420
## R-Squared:      0.79811
## Adj. R-Squared: 0.77429
## F-statistic: 33.5086 on 21 and 178 DF, p-value: < 2.22e-16
```

時間効果が有意かどうかは次の検定を実施すれば良い.

```r
waldtest(gr,grt)
```

```
## Wald test
## 
## Model 1: inv ~ value + capital
## Model 2: inv ~ value + capital + factor(year)
##   Res.Df Df  Chisq Pr(>Chisq)
## 1    197                     
## 2    178 19 25.422     0.1471
```

## ハウスマン検定
帰無仮説が変量効果モデル, 対立仮説が固定効果モデルの検定はハウスマン検定を実施する.
ハウスマン検定は以下で実施する.

```r
phtest(gi,gr)
```

```
## 
## 	Hausman Test
## 
## data:  inv ~ value + capital
## chisq = 2.3304, df = 2, p-value = 0.3119
## alternative hypothesis: one model is inconsistent
```

時間効果がある場合以下を実行すればよい.

```r
phtest(gt,grt)
```

```
## 
## 	Hausman Test
## 
## data:  inv ~ value + capital
## chisq = 4.6434, df = 2, p-value = 0.09811
## alternative hypothesis: one model is inconsistent
```

## 分散不均一の検定
分散不均一かどうかは時間効果がない場合, 以下のようにすればよい.

```r
bptest(inv ~ value + capital + factor(firm), data=Grunfeld)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  inv ~ value + capital + factor(firm)
## BP = 85.836, df = 11, p-value = 1.086e-13
```

分散不均一が疑われる場合, クラスターロバスト分散を用いる.

```r
coeftest(gi,vcov=vcovHC(gi,type="sss"))
```

```
## 
## t test of coefficients:
## 
##         Estimate Std. Error t value  Pr(>|t|)    
## value   0.110124   0.015156  7.2660 9.596e-12 ***
## capital 0.310065   0.052618  5.8927 1.726e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

分散不均一かどうかは時間効果がある場合, 以下のようにすればよい.

```r
bptest(inv ~ value + capital + factor(firm) + factor(year),data=Grunfeld)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  inv ~ value + capital + factor(firm) + factor(year)
## BP = 97.357, df = 30, p-value = 4.833e-09
```

分散不均一が疑われる場合, クラスターロバスト分散を用いる.

```r
coeftest(gt,vcov=vcovHC(gt,type="sss"))
```

```
## 
## t test of coefficients:
## 
##         Estimate Std. Error t value  Pr(>|t|)    
## value   0.117716   0.010263 11.4697 < 2.2e-16 ***
## capital 0.357916   0.045367  7.8893  3.62e-13 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

また次のようにしても求められる.

```r
coeftest(git,vcov=vcovHC(git,type="HC1",cluster = "group"))
```

```
## 
## t test of coefficients:
## 
##                    Estimate Std. Error t value  Pr(>|t|)    
## value              0.117716   0.010266 11.4667 < 2.2e-16 ***
## capital            0.357916   0.045380  7.8872 3.665e-13 ***
## factor(year)1936 -19.197405  19.630596 -0.9779 0.3295053    
## factor(year)1937 -40.690009  31.565895 -1.2890 0.1991424    
## factor(year)1938 -39.226404  14.924544 -2.6283 0.0093713 ** 
## factor(year)1939 -69.470288  25.605714 -2.7131 0.0073565 ** 
## factor(year)1940 -44.235085  16.475952 -2.6848 0.0079797 ** 
## factor(year)1941 -18.804463  16.926601 -1.1109 0.2681716    
## factor(year)1942 -21.139792  13.433918 -1.5736 0.1174471    
## factor(year)1943 -42.977623  11.896875 -3.6125 0.0003996 ***
## factor(year)1944 -43.098772  10.428514 -4.1328 5.630e-05 ***
## factor(year)1945 -55.683040  14.417542 -3.8622 0.0001598 ***
## factor(year)1946 -31.169284  19.837679 -1.5712 0.1180028    
## factor(year)1947 -39.392242  25.073011 -1.5711 0.1180294    
## factor(year)1948 -43.716514  36.872612 -1.1856 0.2374409    
## factor(year)1949 -73.495099  36.280703 -2.0257 0.0443649 *  
## factor(year)1950 -75.896112  34.899790 -2.1747 0.0310435 *  
## factor(year)1951 -62.480912  46.868328 -1.3331 0.1842881    
## factor(year)1952 -64.632341  48.901664 -1.3217 0.1880611    
## factor(year)1953 -67.717966  41.487585 -1.6322 0.1044895    
## factor(year)1954 -93.526221  30.089357 -3.1083 0.0022084 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```




