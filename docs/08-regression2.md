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
##             Estimate Std. Error z value  Pr(>|z|)    
## (Intercept) 11.61023    0.25836 44.9387 < 2.2e-16 ***
## x            1.06084    0.43505  2.4385 0.0147502 *  
## wT          -1.70373    0.50918 -3.3460 0.0008198 ***
## x:wT         1.05244    0.80140  1.3132 0.1891007    
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
##   Res.Df Df Chisq Pr(>Chisq)    
## 1     98                        
## 2     96  2 29.36  4.212e-07 ***
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
##   Res.Df Df     F    Pr(>F)    
## 1     98                       
## 2     96  2 14.68 2.738e-06 ***
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
##   Res.Df     RSS Df Sum of Sq     F    Pr(>F)    
## 1     98 127.373                                 
## 2     96  97.542  2    29.832 14.68 2.738e-06 ***
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
## [1] 1.39078e-29
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

[^08-regression2-1]: 正確にいえば, 不偏推定量のとめには条件付き期待値が説明変数に依存しないことが必要である.
    また線形推定量のなかで最小の分散になるためには 条件付き分散が説明変数に依存しないことが必要である.

ただし, 別の分散のもとで正規分布に近似できることがしられている.[^08-regression2-2]
つまり, 説明変数と誤差項が無相関であるが, 独立とまでは言い切れない場合, 最小二乗推定量を実行した際, 別の方法で分散を推定する必要がある.
この別の分散をロバスト分散という.

[^08-regression2-2]: 正確には観測される変数に4次のモーメントが存在するという仮定が必要となる.
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
##             Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 11.61023    0.26225 44.2714 < 2.2e-16 ***
## x            1.06084    0.42631  2.4884 0.0145527 *  
## wT          -1.70373    0.49881 -3.4156 0.0009344 ***
## x:wT         1.05244    0.83069  1.2670 0.2082372    
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
##             Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 11.61023    0.25628 45.3026 < 2.2e-16 ***
## x            1.06084    0.41610  2.5495 0.0123724 *  
## wT          -1.70373    0.47365 -3.5970 0.0005109 ***
## x:wT         1.05244    0.79527  1.3234 0.1888551    
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
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept) 11.18367    0.27435 40.7650  < 2e-16 ***
## x            1.04381    0.44364  2.3528  0.01863 *  
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
## 2     96  2 15.385 1.601e-06 ***
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
##   Res.Df Df Chisq Pr(>Chisq)    
## 1     98                        
## 2     96  2 30.77  2.082e-07 ***
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
##             Estimate Std. Error t value  Pr(>|t|) CI Lower CI Upper DF
## (Intercept)   11.610     0.2566  45.245 1.507e-66  11.1009   12.120 96
## x              1.061     0.4169   2.545 1.253e-02   0.2333    1.888 96
## wT            -1.704     0.4810  -3.542 6.147e-04  -2.6585   -0.749 96
## x:wT           1.052     0.8043   1.308 1.938e-01  -0.5442    2.649 96
## 
## Multiple R-squared:  0.2823 ,	Adjusted R-squared:  0.2599 
## F-statistic: 15.51 on 3 and 96 DF,  p-value: 2.657e-08
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
##             Estimate Std. Error t value  Pr(>|t|) CI Lower CI Upper DF
## (Intercept)   11.610     0.2584  44.939 2.811e-66  11.0974   12.123 96
## x              1.061     0.4350   2.438 1.659e-02   0.1973    1.924 96
## wT            -1.704     0.5092  -3.346 1.171e-03  -2.7145   -0.693 96
## x:wT           1.052     0.8014   1.313 1.922e-01  -0.5383    2.643 96
## 
## Multiple R-squared:  0.2823 ,	Adjusted R-squared:  0.2599 
## F-statistic: 12.59 on 3 and 96 DF,  p-value: 5.213e-07
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
## [1] 2.976469
```

```r
1-pchisq(bpt,df=3)
```

```
## [1] 0.3952674
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
## BP = 2.9765, df = 3, p-value = 0.3953
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
## [1] 1.637699
```

```r
1-pchisq(wht,df=2)
```

```
## [1] 0.4409387
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
## BP = 1.6377, df = 2, p-value = 0.4409
```

このように分散均一性は検定することが可能であるが, そもそも分散均一が疑われる場合は, ロバスト分散で推定するので十分であるため最近の実証分析ではこの検定は実施されない.
