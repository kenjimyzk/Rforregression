---
output: 
  html_document
---
# パネル分析


## データ

```r
library(AER)
library(plm)
data("Grunfeld", package = "plm")
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


```r
pdata <- pdata.frame(Grunfeld, index = c("firm", "year"))
pdim(pdata)
```

```
## Balanced Panel: n = 10, T = 20, N = 200
```

## プーリングOLS
$$
inv_{it} = \beta_0 + \beta_1 value_{it} + \beta_2 capital_{it} + u_{it}
$$

誤差項 $u_{it}$ は $i$ についても $t$ についても独立同一分布と仮定する.
さらに誤差項は説明変数と独立である.

この推計は以下のようにする.

```r
gp <- plm(inv ~ value + capital, data = pdata, model = "pooling")
summary(gp)
```

```
## Pooling Model
## 
## Call:
## plm(formula = inv ~ value + capital, data = pdata, model = "pooling")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## 
## Residuals:
##      Min.   1st Qu.    Median   3rd Qu.      Max. 
## -291.6757  -30.0137    5.3033   34.8293  369.4464 
## 
## Coefficients:
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
summary(lm(inv ~ value + capital, data = pdata))
```

```
## 
## Call:
## lm(formula = inv ~ value + capital, data = pdata)
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



## 固定効果モデル (平均差分法)
次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it} +\alpha_i + u_{it}
$$
この $\alpha_i$ は固定効果と呼ばれている.
$\alpha_i$ は時間 $t$ に対して一定である.
$\alpha_i$ は誤差項と相関があるもしれない.


それぞれの時間平均をとれば以下になる.
$$
\bar{inv}_{i} = \beta_1 \bar{value}_{i} + \beta_2 \bar{capital}_{i} +\alpha_i  + \bar{u}_{i}
$$

それぞれの観測値を時間平均で差し引けば $\alpha_i$ は消去される. このように変換して回帰分析すれば $\alpha_i$ は誤差項と相関があっても一致推定量である.


この推計は以下のようにする.

```r
gi <- plm(inv ~ value + capital, data = pdata, model = "within")
summary(gi)
```

```
## Oneway (individual) effect Within Model
## 
## Call:
## plm(formula = inv ~ value + capital, data = pdata, model = "within")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## 
## Residuals:
##       Min.    1st Qu.     Median    3rd Qu.       Max. 
## -184.00857  -17.64316    0.56337   19.19222  250.70974 
## 
## Coefficients:
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
summary(lm(inv ~ value + capital+0+factor(firm), data = pdata))
```

```
## 
## Call:
## lm(formula = inv ~ value + capital + 0 + factor(firm), data = pdata)
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

### 時間効果
次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it}+ \gamma_t +\alpha_i + u_{it}
$$
この $\gamma_t$ は時間効果と呼ばれている.

この推計は以下のようにする.

```r
gi2 <- plm(inv ~ value + capital, data = pdata, effect="twoways",model = "within")
summary(gi2)
```

```
## Twoways effects Within Model
## 
## Call:
## plm(formula = inv ~ value + capital, data = pdata, effect = "twoways", 
##     model = "within")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## 
## Residuals:
##      Min.   1st Qu.    Median   3rd Qu.      Max. 
## -162.6094  -19.4710   -1.2669   19.1277  211.8420 
## 
## Coefficients:
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

それぞれの効果は以下になる.

```r
fixef(gi2, effect = "individual")
```

```
##           1           2           3           4           5           6 
## -134.227709   72.826531 -269.458508  -38.873866 -139.666304  -31.339066 
##           7           8           9          10 
##  -82.761099  -66.737194 -104.010153   -7.390586
```

```r
fixef(gi2, effect = "time")
```

```
##       1935       1936       1937       1938       1939       1940 
##  -32.83632  -52.03372  -73.52633  -72.06272 -102.30660  -77.07140 
##       1941       1942       1943       1944       1945       1946 
##  -51.64078  -53.97611  -75.81394  -75.93509  -88.51936  -64.00560 
##       1947       1948       1949       1950       1951       1952 
##  -72.22856  -76.55283 -106.33142 -108.73243  -95.31723  -97.46866 
##       1953       1954 
## -100.55428 -126.36254
```

これは以下の回帰分析と同じである.

```r
lm(inv ~ value + capital+0+factor(firm)+factor(year), data = pdata)
```

```
## 
## Call:
## lm(formula = inv ~ value + capital + 0 + factor(firm) + factor(year), 
##     data = pdata)
## 
## Coefficients:
##            value           capital     factor(firm)1     factor(firm)2  
##           0.1177            0.3579          -86.9002          120.1540  
##    factor(firm)3     factor(firm)4     factor(firm)5     factor(firm)6  
##        -222.1310            8.4536          -92.3388           15.9884  
##    factor(firm)7     factor(firm)8     factor(firm)9    factor(firm)10  
##         -35.4336          -19.4097          -56.6827           39.9369  
## factor(year)1936  factor(year)1937  factor(year)1938  factor(year)1939  
##         -19.1974          -40.6900          -39.2264          -69.4703  
## factor(year)1940  factor(year)1941  factor(year)1942  factor(year)1943  
##         -44.2351          -18.8045          -21.1398          -42.9776  
## factor(year)1944  factor(year)1945  factor(year)1946  factor(year)1947  
##         -43.0988          -55.6830          -31.1693          -39.3922  
## factor(year)1948  factor(year)1949  factor(year)1950  factor(year)1951  
##         -43.7165          -73.4951          -75.8961          -62.4809  
## factor(year)1952  factor(year)1953  factor(year)1954  
##         -64.6323          -67.7180          -93.5262
```
決定係数が大きく異なっていることに注意されたい.

時間効果が有効かどうかは以下の検定を実施する.

```r
pFtest(gi2, gi)
```

```
## 
## 	F test for twoways effects
## 
## data:  inv ~ value + capital
## F = 1.4032, df1 = 19, df2 = 169, p-value = 0.1309
## alternative hypothesis: significant effects
```


### 固定効果VSプーリングOLS
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

時間効果をもつ固定効果モデルの場合は, 帰無仮説仮説に応じて変化する.
もし帰無仮説が時間効果を持たないプーリングOLSモデルなら, 以下を実行する.

```r
pFtest(gi2, gp)
```

```
## 
## 	F test for twoways effects
## 
## data:  inv ~ value + capital
## F = 17.403, df1 = 28, df2 = 169, p-value < 2.2e-16
## alternative hypothesis: significant effects
```

もし帰無仮説が時間効果をもつモデルなら, 以下を実行する.
次のようにする.

```r
gpt <- update(gp, . ~ . +factor(year))
pFtest(gi2,gpt)
```

```
## 
## 	F test for twoways effects
## 
## data:  inv ~ value + capital
## F = 52.362, df1 = 9, df2 = 169, p-value < 2.2e-16
## alternative hypothesis: significant effects
```


## 固定効果モデル (一階差分法)
次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it} +\alpha_i + u_{it}
$$
この $\alpha_i$ は固定効果と呼ばれている.
$\alpha_i$ は時間 $t$ に対して一定である.
$\alpha_i$ は誤差項と相関があるもしれない.

それぞれの階差をとれば $\alpha_i$ は消去できる.
$$
\Delta inv_{it} = \beta_1 \Delta value_{it} + \beta_2 \Delta capital_{it} + \Delta u_{it}
$$

このように変換して回帰分析すれば $\alpha_i$ は誤差項と相関があっても一致推定量である.

この推計は以下のようにする.

```r
gf <- plm(inv ~ value + capital, data = pdata, model = "fd")
summary(gf)
```

```
## Oneway (individual) effect First-Difference Model
## 
## Call:
## plm(formula = inv ~ value + capital, data = pdata, model = "fd")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## Observations used in estimation: 190
## 
## Residuals:
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
## -202.05  -15.23   -1.76   -1.39    7.95  199.27 
## 
## Coefficients:
##          Estimate Std. Error t-value  Pr(>|t|)    
## value   0.0890628  0.0082341  10.816 < 2.2e-16 ***
## capital 0.2786940  0.0471564   5.910  1.58e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    584410
## Residual Sum of Squares: 345940
## R-Squared:      0.40876
## Adj. R-Squared: 0.40561
## F-statistic: 129.597 on 1 and 188 DF, p-value: < 2.22e-16
```

これは以下の回帰分析と同じである.

```r
plm(diff(inv) ~ diff(value) + diff(capital) + 0, data = pdata)
```

```
## 
## Model Formula: diff(inv) ~ diff(value) + diff(capital) + 0
## 
## Coefficients:
##   diff(value) diff(capital) 
##      0.086792      0.229800
```

### 時間効果
次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it}+ \gamma_t +\alpha_i + u_{it}
$$
この $\gamma_t$ は時間効果と呼ばれている.

この推計は以下のようになる.

```r
gft <- plm(inv ~ value + capital+factor(year), data = pdata, model = "fd")
summary(gft)
```

```
## Oneway (individual) effect First-Difference Model
## 
## Call:
## plm(formula = inv ~ value + capital + factor(year), data = pdata, 
##     model = "fd")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## Observations used in estimation: 190
## 
## Residuals:
##       Min.    1st Qu.     Median    3rd Qu.       Max. 
## -179.69353  -18.68501    0.49555   14.27860  179.03692 
## 
## Coefficients:
##                     Estimate  Std. Error t-value  Pr(>|t|)    
## value              0.0875445   0.0095107  9.2048 < 2.2e-16 ***
## capital            0.3246777   0.0571472  5.6814 5.727e-08 ***
## factor(year)1936  -7.5752972  13.6738571 -0.5540   0.58031    
## factor(year)1937 -19.8865297  19.8335377 -1.0027   0.31745    
## factor(year)1938 -32.4497771  23.2978960 -1.3928   0.16550    
## factor(year)1939 -55.1241414  27.1897154 -2.0274   0.04419 *  
## factor(year)1940 -28.1577109  30.3227138 -0.9286   0.35442    
## factor(year)1941  -3.5183963  33.1997669 -0.1060   0.91573    
## factor(year)1942 -11.2051419  35.8488740 -0.3126   0.75500    
## factor(year)1943 -29.2682673  38.4095279 -0.7620   0.44712    
## factor(year)1944 -28.4596662  40.6136977 -0.7007   0.48443    
## factor(year)1945 -37.4137110  42.9014107 -0.8721   0.38440    
## factor(year)1946 -10.4932084  45.1347632 -0.2325   0.81644    
## factor(year)1947 -24.5964020  47.8471043 -0.5141   0.60788    
## factor(year)1948 -28.4696761  50.3295944 -0.5657   0.57237    
## factor(year)1949 -56.4524987  52.7192122 -1.0708   0.28578    
## factor(year)1950 -56.3883613  54.7942767 -1.0291   0.30491    
## factor(year)1951 -35.2680213  57.0562567 -0.6181   0.53732    
## factor(year)1952 -34.0583106  59.9336293 -0.5683   0.57061    
## factor(year)1953 -27.7720148  63.6639644 -0.4362   0.66323    
## factor(year)1954 -52.1173697  67.1610018 -0.7760   0.43883    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    584410
## Residual Sum of Squares: 293000
## R-Squared:      0.49864
## Adj. R-Squared: 0.4393
## F-statistic: 8.40405 on 20 and 169 DF, p-value: < 2.22e-16
```

これが時間効果が有意であるかどうかは, 以下を実施すればよい.

```r
pFtest(gft,gf)
```

```
## 
## 	F test for individual effects
## 
## data:  inv ~ value + capital + factor(year)
## F = 1.607, df1 = 19, df2 = 169, p-value = 0.05928
## alternative hypothesis: significant effects
```

### 平均差分法と一階差分法
平均差分法と一階差分法は誤差項の仮定をどのようにおくかによって変わってくる. 誤差項の階差をとることによって時間を通じて無相関になるなら一階差分法が望ましいであろう.
しかしながら, 固定効果, 時間効果の値がきちんと計算して, それが経済学的解釈が可能なら, 平均差分法が望ましい.
さらに他のプーリングOLSの仮定と変量効果モデルとの比較の意味でも平均差分法がよく使われる.

なお時間が2期間のパネルデータのとき, 平均差分法も一階差分法も計算値は同じである. 
たとえば $t=2$のときの変数 $x_{it}$ の平均差分値は
$$
x_{2t}-\bar{x}_i=x_{2t}-\frac{x_{i1}+x_{i2}}{2}=\frac{x_{i2}-x_{i1}}{2}
$$
となる.


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
gr <- plm(inv ~ value + capital, data = pdata, model = "random")
summary(gr)
```

```
## Oneway (individual) effect Random Effect Model 
##    (Swamy-Arora's transformation)
## 
## Call:
## plm(formula = inv ~ value + capital, data = pdata, model = "random")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## 
## Effects:
##                   var std.dev share
## idiosyncratic 2784.46   52.77 0.282
## individual    7089.80   84.20 0.718
## theta: 0.8612
## 
## Residuals:
##      Min.   1st Qu.    Median   3rd Qu.      Max. 
## -177.6063  -19.7350    4.6851   19.5105  252.8743 
## 
## Coefficients:
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

変量効果は以下のコマンドで確かめられる.

```r
ranef(gr)
```

```
##            1            2            3            4            5 
##   -9.5242955  157.8910235 -172.8958044   29.9119801  -54.6790089 
##            6            7            8            9           10 
##   34.3461316   -7.8977584    0.6726376  -28.1393497   50.3144442
```

### 時間効果
変量効果モデルの時間効果は二種類ある.
時間効果が確率変数である場合は次のコマンドを実施する.

```r
gr2 <- plm(inv ~ value + capital, data = pdata, effect= "twoways",model = "random")
summary(gr2)
```

```
## Twoways effects Random Effect Model 
##    (Swamy-Arora's transformation)
## 
## Call:
## plm(formula = inv ~ value + capital, data = pdata, effect = "twoways", 
##     model = "random")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## 
## Effects:
##                   var std.dev share
## idiosyncratic 2675.43   51.72 0.274
## individual    7095.25   84.23 0.726
## time             0.00    0.00 0.000
## theta: 0.864 (id) 0 (time) 5.551e-17 (total)
## 
## Residuals:
##      Min.   1st Qu.    Median   3rd Qu.      Max. 
## -177.1700  -19.7576    4.6048   19.4676  252.7596 
## 
## Coefficients:
##               Estimate Std. Error t-value Pr(>|t|)    
## (Intercept) -57.865377  29.393359 -1.9687   0.0504 .  
## value         0.109790   0.010528 10.4285   <2e-16 ***
## capital       0.308190   0.017171 17.9483   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    2376000
## Residual Sum of Squares: 547910
## R-Squared:      0.7694
## Adj. R-Squared: 0.76706
## F-statistic: 328.647 on 2 and 197 DF, p-value: < 2.22e-16
```

時間効果が固定されている場合は次のコマンドを実施する.

```r
grt <- update(gr, .~. + factor(year))
summary(grt)
```

```
## Oneway (individual) effect Random Effect Model 
##    (Swamy-Arora's transformation)
## 
## Call:
## plm(formula = inv ~ value + capital + factor(year), data = pdata, 
##     model = "random")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## 
## Effects:
##                   var std.dev share
## idiosyncratic 2675.43   51.72 0.274
## individual    7095.25   84.23 0.726
## theta: 0.864
## 
## Residuals:
##        Min.     1st Qu.      Median     3rd Qu.        Max. 
## -160.759401  -19.805349   -0.028228   19.194961  214.295364 
## 
## Coefficients:
##                    Estimate Std. Error t-value  Pr(>|t|)    
## (Intercept)      -29.828275  32.380484 -0.9212  0.358203    
## value              0.113779   0.011759  9.6763 < 2.2e-16 ***
## capital            0.354336   0.022594 15.6826 < 2.2e-16 ***
## factor(year)1936 -17.690058  23.612087 -0.7492  0.454729    
## factor(year)1937 -38.006448  24.356323 -1.5604  0.120433    
## factor(year)1938 -38.400547  23.303431 -1.6478  0.101148    
## factor(year)1939 -67.669031  23.605147 -2.8667  0.004648 ** 
## factor(year)1940 -42.210436  23.716150 -1.7798  0.076812 .  
## factor(year)1941 -16.896674  23.640596 -0.7147  0.475711    
## factor(year)1942 -19.950610  23.442180 -0.8511  0.395882    
## factor(year)1943 -41.303361  23.564907 -1.7527  0.081367 .  
## factor(year)1944 -41.301975  23.603031 -1.7499  0.081866 .  
## factor(year)1945 -53.418089  23.807547 -2.2437  0.026081 *  
## factor(year)1946 -28.601243  23.973397 -1.1930  0.234441    
## factor(year)1947 -37.647517  23.832869 -1.5796  0.115963    
## factor(year)1948 -41.944013  24.029174 -1.7455  0.082615 .  
## factor(year)1949 -71.515032  24.236975 -2.9507  0.003598 ** 
## factor(year)1950 -73.609655  24.379280 -3.0194  0.002906 ** 
## factor(year)1951 -59.205876  24.754226 -2.3917  0.017810 *  
## factor(year)1952 -60.963457  25.209460 -2.4183  0.016602 *  
## factor(year)1953 -62.886188  26.252610 -2.3954  0.017638 *  
## factor(year)1954 -88.564196  26.819791 -3.3022  0.001159 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    2376000
## Residual Sum of Squares: 479720
## R-Squared:      0.7981
## Adj. R-Squared: 0.77428
## F-statistic: 33.506 on 21 and 178 DF, p-value: < 2.22e-16
```

固定された時間効果が有意かどうかは次の検定を実施すれば良い.

```r
pFtest(grt,gr)
```

```
## 
## 	F test for individual effects
## 
## data:  inv ~ value + capital + factor(year)
## F = 1.3511, df1 = 19, df2 = 178, p-value = 0.1572
## alternative hypothesis: significant effects
```

### ハウスマン検定
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

時間効果が確率変数である変量モデルの場合以下を実行すればよい.

```r
phtest(gi2,gr2)
```

```
## 
## 	Hausman Test
## 
## data:  inv ~ value + capital
## chisq = 13.46, df = 2, p-value = 0.001194
## alternative hypothesis: one model is inconsistent
```

時間効果が確率変数でない変量モデルの場合以下を実行すればよい.

```r
phtest(gi2,grt)
```

```
## 
## 	Hausman Test
## 
## data:  inv ~ value + capital
## chisq = 6.5733, df = 2, p-value = 0.03738
## alternative hypothesis: one model is inconsistent
```

どちらを採用するかによって結果が変わってしまうので注意されたい.

## クラスターロバスト分散
固定効果モデルにおいて, 分散不均一が疑われる場合, ではクラスターロバスト分散を用いる.
時間効果がない場合は以下の様にする.

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

分散不均一が疑われる場合, クラスターロバスト分散を用いる.
時間効果がある場合は以下の様にする.

```r
coeftest(gi2,vcov=vcovHC(gi2,type="sss"))
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

STATA の計算結果に合わせるには次のように実施する必要がある.

```r
git <- update(gi, .~. + factor(year))
coeftest(git,vcov=vcovHC(git,type="sss"))
```

```
## 
## t test of coefficients:
## 
##                    Estimate Std. Error t value  Pr(>|t|)    
## value              0.117716   0.010794 10.9055 < 2.2e-16 ***
## capital            0.357916   0.047715  7.5012 3.424e-12 ***
## factor(year)1936 -19.197405  20.640669 -0.9301 0.3536580    
## factor(year)1937 -40.690009  33.190087 -1.2260 0.2219160    
## factor(year)1938 -39.226404  15.692472 -2.4997 0.0133837 *  
## factor(year)1939 -69.470288  26.923231 -2.5803 0.0107211 *  
## factor(year)1940 -44.235085  17.323706 -2.5534 0.0115505 *  
## factor(year)1941 -18.804463  17.797543 -1.0566 0.2922130    
## factor(year)1942 -21.139792  14.125147 -1.4966 0.1363608    
## factor(year)1943 -42.977623  12.509017 -3.4357 0.0007437 ***
## factor(year)1944 -43.098772  10.965103 -3.9305 0.0001234 ***
## factor(year)1945 -55.683040  15.159383 -3.6732 0.0003212 ***
## factor(year)1946 -31.169284  20.858408 -1.4943 0.1369549    
## factor(year)1947 -39.392242  26.363118 -1.4942 0.1369835    
## factor(year)1948 -43.716514  38.769856 -1.1276 0.2610913    
## factor(year)1949 -73.495099  38.147491 -1.9266 0.0557069 .  
## factor(year)1950 -75.896112  36.695524 -2.0683 0.0401383 *  
## factor(year)1951 -62.480912  49.279892 -1.2679 0.2065854    
## factor(year)1952 -64.632341  51.417852 -1.2570 0.2104874    
## factor(year)1953 -67.717966  43.622288 -1.5524 0.1224442    
## factor(year)1954 -93.526221  31.637576 -2.9562 0.0035603 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

どちらを採用するかによって結果が変わってしまうので注意されたい.

### 分散不均一の検定
固定効果モデルにおいて, 
分散不均一かどうかを検定するには, 
時間効果がない場合, 以下のようにすればよい.

```r
bptest(inv ~ value + capital + factor(firm), data=pdata)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  inv ~ value + capital + factor(firm)
## BP = 85.836, df = 11, p-value = 1.086e-13
```

時間効果がある場合, 以下のようにすればよい.

```r
bptest(inv ~ value + capital + factor(firm) + factor(year),data=pdata)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  inv ~ value + capital + factor(firm) + factor(year)
## BP = 97.357, df = 30, p-value = 4.833e-09
```




