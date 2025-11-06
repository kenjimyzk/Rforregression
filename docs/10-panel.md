---
output: 
  html_document
editor_options: 
  markdown: 
    wrap: sentence
---

# パネル分析



## データ


``` r
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


``` r
pdata <- pdata.frame(Grunfeld, index = c("firm", "year"))
pdim(pdata)
```

```
## Balanced Panel: n = 10, T = 20, N = 200
```

## プーリングOLS

次の重回帰モデルを考える.

$$
inv_{it} = \beta_0 + \beta_1 value_{it} + \beta_2 capital_{it} + u_{it}
$$

誤差項 $u_{it}$ は $i$ についても $t$ についても独立同一分布と仮定する.
さらに誤差項は説明変数と独立である.
この時、パネルデータにおいてもOLS推定法でパラメータは不偏である.
ここでの重回帰モデルをプーリングOLSモデルと呼ぶことにする.

この推計は以下のようにする.


``` r
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


``` r
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

## 固定効果 (平均差分法)

次の重回帰モデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it} +\alpha_i + u_{it}
$$ この $\alpha_i$ は個別固定効果と呼ばれている.
$\alpha_i$ は時間 $t$ に対して一定である.
$\alpha_i$ は誤差項と相関があるもしれない.
この個別固定効果を持つ重回帰モデルを固定効果モデルと呼ぶことにする.

それぞれの時間平均をとれば以下になる.
$$
\bar{inv}_{i} = \beta_1 \bar{value}_{i} + \beta_2 \bar{capital}_{i} +\alpha_i  + \bar{u}_{i}
$$

そして，それぞれの観測値を時間平均で差し引けば以下のように $\alpha_i$ は消去される.
$$
inv_{it}-\overline{inv}_{i} = \beta_1 (value_{it}-\overline{value}_{i}) + \beta_2 (capital_{it}-\overline{capital}_{i})  + \bar{u}_{i} -\bar{u}_{i}
$$ このように変換して回帰分析すれば $\alpha_i$ は誤差項と相関があっても一致推定量である.
このような推定方法を**平均差分法**という.

この推計は以下のようにする.


``` r
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

個別固定効果は以下のコマンドで確かめられる.


``` r
fixef(gi)
```

```
##         1         2         3         4         5         6         7         8 
##  -70.2967  101.9058 -235.5718  -27.8093 -114.6168  -23.1613  -66.5535  -57.5457 
##         9        10 
##  -87.2223   -6.5678
```

これは以下の回帰分析と同じである.


``` r
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

個別固定効果が有効かどうかは以下の検定を実施すればよい.


``` r
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

### 時間効果モデル

次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it}+ \gamma_t +\alpha_i + u_{it}
$$ この $\gamma_t$ は時間固定効果と呼ばれている.
ここでは個別固定効果と時間固定効果の2つの固定効果を持つ重回帰モデルを**時間効果モデル**と呼ぶことにする.

この推計は以下のようにする.


``` r
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

それぞれの固定効果は以下になる.


``` r
fixef(gi2, effect = "individual")
```

```
##         1         2         3         4         5         6         7         8 
##  -86.9002  120.1540 -222.1310    8.4536  -92.3388   15.9884  -35.4336  -19.4097 
##         9        10 
##  -56.6827   39.9369
```

``` r
fixef(gi2, effect = "time")
```

```
##    1935    1936    1937    1938    1939    1940    1941    1942    1943    1944 
##  -86.90 -106.10 -127.59 -126.13 -156.37 -131.14 -105.70 -108.04 -129.88 -130.00 
##    1945    1946    1947    1948    1949    1950    1951    1952    1953    1954 
## -142.58 -118.07 -126.29 -130.62 -160.40 -162.80 -149.38 -151.53 -154.62 -180.43
```

これは以下の回帰分析と同じである.


``` r
summary(lm(inv ~ value + capital+0+factor(firm)+factor(year), data = pdata))
```

```
## 
## Call:
## lm(formula = inv ~ value + capital + 0 + factor(firm) + factor(year), 
##     data = pdata)
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

時間固定効果が有効かどうかは以下の検定を実施すればよい.


``` r
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

## 固定効果 (一階差分法)

次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it} +\alpha_i + u_{it}
$$ この $\alpha_i$ は固定効果と呼ばれている.
$\alpha_i$ は時間 $t$ に対して一定である.
$\alpha_i$ は誤差項と相関があるもしれない.

それぞれの階差をとれば $\alpha_i$ は消去できる.
$$
\Delta inv_{it} = \beta_1 \Delta value_{it} + \beta_2 \Delta capital_{it} + \Delta u_{it}
$$

このように変換して回帰分析すれば $\alpha_i$ は誤差項と相関があっても一致推定量である.

この推計は以下のようにする.


``` r
gf <- plm(inv ~ value + capital+0, data = pdata, model = "fd")
summary(gf)
```

```
## Oneway (individual) effect First-Difference Model
## 
## Call:
## plm(formula = inv ~ value + capital + 0, data = pdata, model = "fd")
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
## F-statistic: 70.5784 on 2 and 188 DF, p-value: < 2.22e-16
```

時間効果がある場合, 以下を実行する.


``` r
gf2 <-plm(inv ~ value + capital+0+factor(year), data = pdata, model = "fd")
summary(gf2)
```

```
## Oneway (individual) effect First-Difference Model
## 
## Call:
## plm(formula = inv ~ value + capital + 0 + factor(year), data = pdata, 
##     model = "fd")
## 
## Balanced Panel: n = 10, T = 20, N = 200
## Observations used in estimation: 190
## 
## Residuals:
##       Min.    1st Qu.     Median    3rd Qu.       Max. 
## -179.69353  -18.68501    0.49555   14.27860  179.03692 
## 
## Coefficients: (1 dropped because of singularities)
##                    Estimate Std. Error t-value  Pr(>|t|)    
## value             0.0875445  0.0095107  9.2048 < 2.2e-16 ***
## capital           0.3246777  0.0571472  5.6814 5.727e-08 ***
## factor(year)1935 52.1173697 67.1610018  0.7760   0.43883    
## factor(year)1936 44.5420725 65.0001486  0.6853   0.49412    
## factor(year)1937 32.2308400 62.5656219  0.5152   0.60712    
## factor(year)1938 19.6675926 60.6802571  0.3241   0.74625    
## factor(year)1939 -3.0067716 58.4729853 -0.0514   0.95905    
## factor(year)1940 23.9596588 56.8175845  0.4217   0.67378    
## factor(year)1941 48.5989734 54.8059164  0.8867   0.37648    
## factor(year)1942 40.9122279 52.7118790  0.7761   0.43875    
## factor(year)1943 22.8491024 50.5894648  0.4517   0.65209    
## factor(year)1944 23.6577035 48.8480758  0.4843   0.62879    
## factor(year)1945 14.7036587 46.6708001  0.3151   0.75311    
## factor(year)1946 41.6241613 44.2490709  0.9407   0.34821    
## factor(year)1947 27.5209677 40.4024259  0.6812   0.49670    
## factor(year)1948 23.6476936 37.0841079  0.6377   0.52455    
## factor(year)1949 -4.3351290 33.6481639 -0.1288   0.89764    
## factor(year)1950 -4.2709916 30.2836724 -0.1410   0.88801    
## factor(year)1951 16.8493484 26.2244634  0.6425   0.52142    
## factor(year)1952 18.0590591 20.8995024  0.8641   0.38876    
## factor(year)1953 24.3453549 13.9307034  1.7476   0.08235 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    584410
## Residual Sum of Squares: 293000
## R-Squared:      0.49864
## Adj. R-Squared: 0.4393
## F-statistic: 8.5881 on 21 and 169 DF, p-value: < 2.22e-16
```

時間固定効果が有効かどうかは以下の検定を実施すればよい.


``` r
pFtest(gf2,gf)
```

```
## 
## 	F test for individual effects
## 
## data:  inv ~ value + capital + 0 + factor(year)
## F = 1.607, df1 = 19, df2 = 169, p-value = 0.05928
## alternative hypothesis: significant effects
```

### 平均差分法と一階差分法

平均差分法と一階差分法は誤差項の仮定をどのようにおくかによって変わってくる.
誤差項の階差をとることによって時間を通じて無相関になるなら一階差分法が望ましいであろう.
しかしながら, 固定効果, 時間効果の値がきちんと計算して, それが経済学的解釈が可能なら, 平均差分法が望ましい.
さらに他のプーリングOLSの仮定と変量効果モデルとの比較の意味でも平均差分法がよく使われる.

なお時間が2期間のパネルデータのとき, 平均差分法も一階差分法も計算値は同じである.
たとえば $t=2$のときの変数 $x_{it}$ の平均差分値は $$
x_{2t}-\bar{x}_i=x_{2t}-\frac{x_{i1}+x_{i2}}{2}=\frac{x_{i2}-x_{i1}}{2}
$$ となる.

## 変量効果

次のモデルを考える.
$$
inv_{it} = \beta_1 value_{it} + \beta_2 capital_{it} +\alpha_i + u_{it}
$$ この $\alpha_i$ は時間 $t$ について一定であるが, $i$ について独立同一分布の確率変数にしたがう.
さらに $\alpha_i$ は説明変数と無相関である時, この $\alpha_i$ は個別変量効果と呼ばれている.
個別固定効果は説明変数と無相関を仮定していない.
. この個別変量効果を持つ重回帰モデルを変量効果モデルと呼ぶことにする.

変量効果モデルは次のコマンドで実施する.


``` r
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
##               Estimate Std. Error z-value Pr(>|z|)    
## (Intercept) -57.834415  28.898935 -2.0013  0.04536 *  
## value         0.109781   0.010493 10.4627  < 2e-16 ***
## capital       0.308113   0.017180 17.9339  < 2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Total Sum of Squares:    2381400
## Residual Sum of Squares: 548900
## R-Squared:      0.7695
## Adj. R-Squared: 0.76716
## Chisq: 657.674 on 2 DF, p-value: < 2.22e-16
```

変量効果は以下のコマンドで確かめられる.


``` r
ranef(gr)
```

```
##            1            2            3            4            5            6 
##   -9.5242955  157.8910235 -172.8958044   29.9119801  -54.6790089   34.3461316 
##            7            8            9           10 
##   -7.8977584    0.6726376  -28.1393497   50.3144442
```

### ハウスマン検定

帰無仮説が変量効果モデル, 対立仮説が固定効果モデルの検定はハウスマン検定を実施する.
ハウスマン検定は以下で実施する.


``` r
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

## クラスターロバスト分散

固定効果モデルにおいて, 分散不均一が疑われる場合, ではクラスターロバスト分散を用いる.
時間効果がない場合は以下の様にする.


``` r
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


``` r
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


``` r
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

固定効果モデルにおいて, 分散不均一かどうかを検定するには以下のようにすればよい.


``` r
bptest(inv ~ value + capital + factor(firm), data=pdata)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  inv ~ value + capital + factor(firm)
## BP = 85.836, df = 11, p-value = 1.086e-13
```

時間効果モデルの場合, 以下のようにすればよい.


``` r
bptest(inv ~ value + capital + factor(firm) + factor(year),data=pdata)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  inv ~ value + capital + factor(firm) + factor(year)
## BP = 97.357, df = 30, p-value = 4.833e-09
```
