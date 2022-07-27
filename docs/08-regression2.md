---
output: 
  html_document:default
  html_notebook:default
editor_options: 
  markdown: 
    wrap: sentence
---

# 現代的仮定のもとでの最小二乗法


```r
library(AER)
library(estimatr)
```

前節において以下の仮定を置いていた.

-   $(x_i,y_i)$ は独立同一分布にしたがう.
-   $E[u_i]=0$ である.
-   $u_i$ と $x_i$ は独立である.
-   $u_i$ は正規分布にしたがう.

これらの仮定を緩めることで分析にどのような影響をあたえるのかを見ていく.

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

<img src="08-regression2_files/figure-html/unnamed-chunk-3-1.png" width="672" />

## 正規性の仮定について

十分な観測値が得られるばあい, $u_i$ が正規分布にしたがっていないくても, 中心極限定理定理より, 最小二乗法推定量は正規分布に近似できる.

ここの係数ゼロのティー検定について, ライブラリ `AER` を導入して `coeftest` を用いればよい.


```r
fm1 <- lm(y~x*w,data=df)
fm0 <- lm(y~x,data=df)
coeftest(fm1,df=Inf)
```

```
## 
## z test of coefficients:
## 
##              Estimate Std. Error z value  Pr(>|z|)    
## (Intercept) 10.983335   0.274367 40.0315 < 2.2e-16 ***
## x            2.160605   0.489039  4.4181 9.959e-06 ***
## wT          -1.072168   0.398958 -2.6874  0.007201 ** 
## x:wT         0.013909   0.717424  0.0194  0.984532    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

ただ十分なデータのもとではティー値のままでもよい.

同様に複数制約の場合, エフ検定統計量に制約の数を乗じた統計量が 自由度が制約数のカイ二乗分布にしたがうことが知られている.
これをR で実施するには `waldtest` を用いればよい.


```r
waldtest(fm0,fm1,test="Chisq")
```

```
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1     98                         
## 2     96  2 25.131   3.49e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

エフ検定も十分なデータのもとではそのままでよいであろう.

オプション `test` を付けなければエフ検定を実施する.


```r
waldtest(fm0,fm1)
```

```
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df Df      F    Pr(>F)    
## 1     98                        
## 2     96  2 12.566 1.421e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

これは `anova` と同じである.


```r
anova(fm0,fm1)
```

```
## Analysis of Variance Table
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 135.36                                  
## 2     96 107.28  2    28.084 12.566 1.421e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

複数制約の検定としてLM検定というのもある.
制約付きの回帰分析を実行し, その残差を制約なしのモデルの説明変数に回帰する.
その決定係数に観測数を掛けた統計量が自由どが制約の数のカイ二乗分布にしたがうことが知られている.


```r
lmt <- lm(I(resid(fm1))~w*x,data=df)
(lmt <- nrow(df)*summary(lmt)$r.squared)
```

```
## [1] 3.238056e-30
```

```r
1-pchisq(lmt,df=1)
```

```
## [1] 1
```

## 誤差項と説明変数が独立の仮定について

また $u_i$ と $x_i$ は独立でなく, $u_i$ と $x_i$ が無相関という弱い条件のもとでも, 一致推定量であることが知られている.
ただ不偏推定量は保証できない.
また 線形推定量のなかで最小の分散とも言えない.[^08-regression2-1]
また独立のときの標準誤差の推定量が一致推定量でない.

[^08-regression2-1]:  正確にいえば, 不偏推定量のとめには条件付き期待値が説明変数に依存しないことが必要である.
    また線形推定量のなかで最小の分散になるためには 条件付き分散が説明変数に依存しないことが必要である.

ただし, 別の分散のもとで正規分布に近似できることがしられている.[^08-regression2-2]
つまり, 説明変数と誤差項が無相関であるが, 独立とまでは言い切れない場合, 最小二乗推定量を実行した際, 別の方法で分散を推定する必要がある.
この別の分散をロバスト分散という.

[^08-regression2-2]:  正確には観測される変数に4次のモーメントが存在するという仮定が必要となる.
    この仮定の直感的な意味は異常値が存在しないことである.

R でロバスト分散を推定するにはパッケージ `AER` を導入するのが簡単である.
は次のコマンド `coeftest` を実行すればよい.


```r
coeftest(fm1,vcov=vcovHC)
```

```
## 
## t test of coefficients:
## 
##              Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.983335   0.296592 37.0318 < 2.2e-16 ***
## x            2.160605   0.543189  3.9776  0.000135 ***
## wT          -1.072168   0.391522 -2.7385  0.007358 ** 
## x:wT         0.013909   0.717667  0.0194  0.984577    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

先の値と標準誤差が違っていることが確認できるであろう.
ただこの値は STATA と少し異なっている.
STATA と同じにするには


```r
coeftest(fm1,vcov=vcovHC(fm1,type="HC1"))
```

```
## 
## t test of coefficients:
## 
##              Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.983335   0.289474 37.9424 < 2.2e-16 ***
## x            2.160605   0.527831  4.0934 8.862e-05 ***
## wT          -1.072168   0.380134 -2.8205  0.005826 ** 
## x:wT         0.013909   0.692728  0.0201  0.984022    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

としなければならない.

またティー分布でなく正規分布とすることもできる.


```r
coeftest(fm0,vcov=vcovHC,df=Inf)
```

```
## 
## z test of coefficients:
## 
##             Estimate Std. Error z value  Pr(>|z|)    
## (Intercept) 10.47731    0.21625 48.4504 < 2.2e-16 ***
## x            2.21664    0.39355  5.6324 1.778e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

複数の係数についての検定は `waldtest` を実行すればよい.


```r
waldtest(fm0,fm1,vcov=vcovHC)
```

```
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df Df      F    Pr(>F)    
## 1     98                        
## 2     96  2 12.964 1.038e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

先の結果はエフ検定であるが, カイ二乗検定を実施するには以下を実施すればよい.


```r
waldtest(fm0,fm1,vcov=vcovHC, test="Chisq")
```

```
## Wald test
## 
## Model 1: y ~ x
## Model 2: y ~ x * w
##   Res.Df Df  Chisq Pr(>Chisq)    
## 1     98                         
## 2     96  2 25.928  2.344e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

最近開発されたパッケージ `estimatr`のコマンド　`lm_robust`　を用いるとロバスト分散のもとの推定値が簡単に計算できる.


```r
fm2<- lm_robust(y~x*w,data=df)
summary(fm2)
```

```
## 
## Call:
## lm_robust(formula = y ~ x * w, data = df)
## 
## Standard error type:  HC2 
## 
## Coefficients:
##             Estimate Std. Error  t value  Pr(>|t|) CI Lower CI Upper DF
## (Intercept) 10.98333     0.2900 37.87131 1.600e-59   10.408  11.5590 96
## x            2.16061     0.5300  4.07669 9.421e-05    1.109   3.2126 96
## wT          -1.07217     0.3818 -2.80787 6.041e-03   -1.830  -0.3142 96
## x:wT         0.01391     0.6979  0.01993 9.841e-01   -1.371   1.3992 96
## 
## Multiple R-squared:  0.3983 ,	Adjusted R-squared:  0.3795 
## F-statistic: 21.64 on 3 and 96 DF,  p-value: 8.683e-11
```

オプション　`se_type = "stata"` を用いればSTATAと同じ計算が可能である.

また以下のオプションをつければ分散均一の場合も計算できる.


```r
fm3<- lm_robust(y~x*w,data=df,se_type = "classical")
summary(fm3)
```

```
## 
## Call:
## lm_robust(formula = y ~ x * w, data = df, se_type = "classical")
## 
## Standard error type:  classical 
## 
## Coefficients:
##             Estimate Std. Error  t value  Pr(>|t|) CI Lower CI Upper DF
## (Intercept) 10.98333     0.2744 40.03148 1.066e-61   10.439  11.5280 96
## x            2.16061     0.4890  4.41807 2.617e-05    1.190   3.1313 96
## wT          -1.07217     0.3990 -2.68742 8.488e-03   -1.864  -0.2802 96
## x:wT         0.01391     0.7174  0.01939 9.846e-01   -1.410   1.4380 96
## 
## Multiple R-squared:  0.3983 ,	Adjusted R-squared:  0.3795 
## F-statistic: 21.18 on 3 and 96 DF,  p-value: 1.302e-10
```

## 分散均一の検定

誤差項が説明変数と独立のときと無相関のときでは標準誤差の推定量が異なる.
正確にいうと, 条件付き分散が説明変数に依存するかどうかによって標準誤差の推定量が異なる.
このことは分散均一と呼ばれている.

誤差項の分散が均一かどうかは検定可能である.
有名な検定方法としてBP (Breusch-Pagan) 検定というものがある.
BP検定は帰無仮説が分散均一で, 対立仮説が分散が説明変数と線形関係になっている場合の検定である.

残差の自乗を被説明変数として回帰分析をおこない, その決定係数に観測数をかけたものが検定統計量となる.


```r
bpt <- lm(I(resid(fm1)^2)~w*x,data=df)
(bpt <-nrow(df)*summary(bpt)$r.squared)
```

```
## [1] 5.67872
```

```r
1-pchisq(bpt,df=3)
```

```
## [1] 0.1283315
```

ここでの例ではP値が5%を超えているので帰無仮説を棄却できないので, 分散均一を仮定してよいことが示唆されている.

R では以下のように実施すればよい.


```r
bptest(fm1)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
## BP = 5.6787, df = 3, p-value = 0.1283
```

これまでのBPテストは誤差項の分散が説明変数の線形関係あることを暗黙に仮定している.
非線形性を考慮するために説明変数の二次項を導入した分散不均一性の検定をホワイト検定という.
説明変数が複数ある場合ホワイト検定は煩雑になるため, 被説明変数の予測値を使って計算することがある.
そのときホワイトテストは以下で実施する.


```r
wht <- lm(I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2),data=df)
(wht <- nrow(df)*summary(wht)$r.squared)
```

```
## [1] 3.059114
```

```r
1-pchisq(wht,df=2)
```

```
## [1] 0.2166316
```

ホワイト検定でも分散均一が示唆されている.

もしくは以下を実行する.


```r
bptest(fm1,~fitted(fm1)+I(fitted(fm1)^2))
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
## BP = 3.0591, df = 2, p-value = 0.2166
```

このように分散均一性は検定することが可能であるが, そもそも分散均一が疑われる場合は, ロバスト分散で推定するので十分であるため最近の実証分析ではこの検定は実施されない.
