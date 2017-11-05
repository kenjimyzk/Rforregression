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
##   10.474748    1.902005
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
##  0.2147517  0.6039587  0.4704234 -1.3112341  0.0745794  0.4912187
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.83856 11.17844 10.53432 11.27165 10.61291 10.95906
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 117.0161
sum(resid(fm)^2)
## [1] 117.0161
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
## -2.4939 -0.6092  0.0598  0.6769  2.7345 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.4747     0.2069  50.616  < 2e-16 ***
## x             1.9020     0.3720   5.113 1.57e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.093 on 98 degrees of freedom
## Multiple R-squared:  0.2106,	Adjusted R-squared:  0.2025 
## F-statistic: 26.14 on 1 and 98 DF,  p-value: 1.573e-06
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
## (Intercept) 10.474748  0.2069439 50.616362 4.612677e-72
## x            1.902005  0.3720021  5.112887 1.572654e-06
# coefficients(summary(fm))
```

残差の標準誤差

```r
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.092722
with(summary(fm),sigma)
## [1] 1.092722
```

決定係数は次のようにして計算する.

```r
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.210579
with(summary(fm),r.squared)
## [1] 0.210579
```

調整済み決定係数は次のようにして計算する.

```r
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.2025237
with(summary(fm),adj.r.squared)
## [1] 0.2025237
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
## -2.68759 -0.67766  0.04604  0.75336  3.10346 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.8759     0.1689  70.319  < 2e-16 ***
## log(x)        0.4660     0.1156   4.031  0.00011 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.139 on 98 degrees of freedom
## Multiple R-squared:  0.1422,	Adjusted R-squared:  0.1335 
## F-statistic: 16.25 on 1 and 98 DF,  p-value: 0.0001099
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
##      Min       1Q   Median       3Q      Max 
## -0.24388 -0.04808  0.01079  0.06222  0.22019 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.34583    0.01855 126.486  < 2e-16 ***
## x            0.16847    0.03334   5.053 2.01e-06 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09793 on 98 degrees of freedom
## Multiple R-squared:  0.2067,	Adjusted R-squared:  0.1986 
## F-statistic: 25.54 on 1 and 98 DF,  p-value: 2.015e-06
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
## -6.333 -1.346  3.258  7.054 12.765 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   17.892      1.018   17.57   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.664 on 99 degrees of freedom
## Multiple R-squared:  0.7572,	Adjusted R-squared:  0.7548 
## F-statistic: 308.8 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -6.333 -1.346  3.258  7.054 12.765 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   17.892      1.018   17.57   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.664 on 99 degrees of freedom
## Multiple R-squared:  0.7572,	Adjusted R-squared:  0.7548 
## F-statistic: 308.8 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -2.47444 -0.57854  0.05756  0.58664  2.17843 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9945     0.2044  53.778  < 2e-16 ***
## x             1.9677     0.3264   6.028 3.00e-08 ***
## wT           -1.0591     0.1919  -5.519 2.84e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9582 on 97 degrees of freedom
## Multiple R-squared:  0.3992,	Adjusted R-squared:  0.3868 
## F-statistic: 32.23 on 2 and 97 DF,  p-value: 1.852e-11
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
## -2.40493 -0.70565  0.06074  0.67577  2.82469 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.6450     0.3075  34.613   <2e-16 ***
## x             0.8314     1.4759   0.563    0.575    
## I(x^2)        1.0842     1.4462   0.750    0.455    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.095 on 97 degrees of freedom
## Multiple R-squared:  0.2151,	Adjusted R-squared:  0.1989 
## F-statistic: 13.29 on 2 and 97 DF,  p-value: 7.903e-06
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
## -2.44546 -0.58384  0.04494  0.58990  2.19662 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0855     0.2612  42.446  < 2e-16 ***
## x             1.7703     0.4796   3.691 0.000371 ***
## wT           -1.2335     0.3645  -3.384 0.001035 ** 
## x:wT          0.3700     0.6566   0.563 0.574410    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9616 on 96 degrees of freedom
## Multiple R-squared:  0.4012,	Adjusted R-squared:  0.3825 
## F-statistic: 21.44 on 3 and 96 DF,  p-value: 1.033e-10
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
## -2.44546 -0.58384  0.04494  0.58990  2.19662 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.0855     0.2612  42.446  < 2e-16 ***
## x             1.7703     0.4796   3.691 0.000371 ***
## wT           -1.2335     0.3645  -3.384 0.001035 ** 
## x:wT          0.3700     0.6566   0.563 0.574410    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.9616 on 96 degrees of freedom
## Multiple R-squared:  0.4012,	Adjusted R-squared:  0.3825 
## F-statistic: 21.44 on 3 and 96 DF,  p-value: 1.033e-10
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
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 117.02                                  
## 2     96  88.76  2    28.256 15.281 1.732e-06 ***
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
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     96  88.76                                  
## 2     98 117.02 -2   -28.256 15.281 1.732e-06 ***
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
##   Res.Df    RSS Df Sum of Sq      F    Pr(>F)    
## 1     98 117.02                                  
## 2     96  88.76  2    28.256 15.281 1.732e-06 ***
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
library(AER)
coeftest(fm1,vcov=vcovHC)
## 
## t test of coefficients:
## 
##             Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 11.08553    0.24375 45.4788 < 2.2e-16 ***
## x            1.77028    0.44971  3.9365 0.0001566 ***
## wT          -1.23352    0.35739 -3.4515 0.0008305 ***
## x:wT         0.37000    0.61568  0.6010 0.5492791    
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
## (Intercept) 11.08553    0.23492 47.1895 < 2.2e-16 ***
## x            1.77028    0.43386  4.0803 9.296e-05 ***
## wT          -1.23352    0.34647 -3.5603 0.0005783 ***
## x:wT         0.37000    0.59447  0.6224 0.5351553    
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
## (Intercept) 10.47475    0.20939 50.0245 < 2.2e-16 ***
## x            1.90200    0.35569  5.3474 8.922e-08 ***
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
## 2     96  2 14.544 3.038e-06 ***
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
## 2     96  2 29.088  4.825e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

複数制約の検定としてLM検定というのもある.
制約付きの回帰分析を実行し, その残差を制約なしのモデルの説明変数に回帰する.
その決定係数に観測数を掛けた統計量が自由どが制約の数のカイ二乗分布にしたがうことが知られている.

```r
lmt <- lm(I(resid(fm1))~w*x,data=df)
(lmt <- nrow(df)*summary(lmt)$r.squared)
## [1] 3.401379e-30
1-pchisq(lmt,df=1)
## [1] 1
```



### 分散均一の検定
誤差項が説明変数と独立のときと無相関のときでは標準誤差の推定量が異なっている.
正確にいうと, 条件付き分散が説明変数に依存するかどうかによって標準誤差の推定量が異なる. このことを分散均一と呼ばれている.

誤差項の分散が均一かどうかは検定可能である.
有名な検定方法としてBP検定というものがある.
BP検定は帰無仮説が分散均一で, 対立仮説が分散が説明変数と線形関係になっている場合の検定である.

残差の自乗を被説明変数として回帰分析をおこない,
その決定係数に観測数をかけたものが検定統計量となる.

```r
bpt <- lm(I(resid(fm1)^2)~w*x,data=df)
(bpt <- nrow(df)*summary(bpt)$r.squared)
## [1] 0.850312
1-pchisq(bpt,df=3)
## [1] 0.837399
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
## BP = 0.85031, df = 3, p-value = 0.8374
```

これまでのBPテストは誤差項の分散が説明変数の線形関係あることを暗黙に仮定している.
非線形性を考慮するために説明変数の二次項を導入した分散不均一性の検定をホワイト検定という.
二次項を加えると煩雑になるため, 予測値を使って計算することがある.
ホワイトテストは以下で実施する.

```r
wht <- lm(I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2),data=df)
(wht <- nrow(df)*summary(wht)$r.squared)
## [1] 0.0523853
1-pchisq(wht,df=2)
## [1] 0.9741474
```
ホワイト検定でも分散均一が示唆されている.

もしくは以下を実行する.

```r
bptest(fm1,I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2))
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
## BP = 0.052385, df = 2, p-value = 0.9741
```

このように分散均一性は検定することが可能であるが, そもそも分散均一が疑われる場合は,
ロバスト分散で推定するので十分であるため最近の実証分析ではこの検定は実施されない.



