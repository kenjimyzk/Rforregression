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
##   10.591708    1.937317
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
##  0.8021954 -1.3906290  0.3013971 -1.2231538 -0.2924664 -0.6362500
# residuals(fm)
# with(fm, residuals)
```

予測値は次のコマンドを実施する.

```r
head(fitted(fm))
##        1        2        3        4        5        6 
## 11.25390 12.01461 11.93322 10.95611 10.77714 11.21521
# fitted.values(fm)
# with(fm, fitted.values)
```

残差自乗和は次のコマンドを実施する.

```r
deviance(fm)
## [1] 135.045
sum(resid(fm)^2)
## [1] 135.045
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
## -3.4014 -0.7466 -0.0732  0.6745  2.8607 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.5917     0.2484   42.63  < 2e-16 ***
## x             1.9373     0.4393    4.41 2.65e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.174 on 98 degrees of freedom
## Multiple R-squared:  0.1656,	Adjusted R-squared:  0.1571 
## F-statistic: 19.45 on 1 and 98 DF,  p-value: 2.649e-05
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
##              Estimate Std. Error  t value     Pr(>|t|)
## (Intercept) 10.591708  0.2484345 42.63380 4.498546e-65
## x            1.937317  0.4392700  4.41031 2.649300e-05
# coefficients(summary(fm))
```

残差の標準誤差

```r
sqrt(deviance(fm)/df.residual(fm))
## [1] 1.173887
with(summary(fm),sigma)
## [1] 1.173887
```

決定係数は次のようにして計算する.

```r
1-deviance(fm)/with(df, sum((y-mean(y))^2))
## [1] 0.1656083
with(summary(fm),r.squared)
## [1] 0.1656083
```

調整済み決定係数は次のようにして計算する.

```r
1-(deviance(fm)/df.residual(fm))/with(df, sum((y-mean(y))^2/(nrow(df)-1)))
## [1] 0.1570941
with(summary(fm),adj.r.squared)
## [1] 0.1570941
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
## -3.5931 -0.8229 -0.0732  0.8048  2.8680 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.9836     0.1728  69.345  < 2e-16 ***
## log(x)        0.4425     0.1278   3.463 0.000793 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.213 on 98 degrees of freedom
## Multiple R-squared:  0.109,	Adjusted R-squared:  0.09994 
## F-statistic: 11.99 on 1 and 98 DF,  p-value: 0.0007933
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
## -0.34804 -0.06104 -0.00019  0.06218  0.22358 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  2.35833    0.02159 109.227  < 2e-16 ***
## x            0.16637    0.03818   4.358 3.24e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.102 on 98 degrees of freedom
## Multiple R-squared:  0.1623,	Adjusted R-squared:  0.1538 
## F-statistic: 18.99 on 1 and 98 DF,  p-value: 3.239e-05
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
## -6.518 -1.015  2.357  6.235 11.237 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   18.442      0.913    20.2   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.164 on 99 degrees of freedom
## Multiple R-squared:  0.8047,	Adjusted R-squared:  0.8028 
## F-statistic:   408 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -6.518 -1.015  2.357  6.235 11.237 
## 
## Coefficients:
##   Estimate Std. Error t value Pr(>|t|)    
## x   18.442      0.913    20.2   <2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 5.164 on 99 degrees of freedom
## Multiple R-squared:  0.8047,	Adjusted R-squared:  0.8028 
## F-statistic:   408 on 1 and 99 DF,  p-value: < 2.2e-16
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
## -2.99974 -0.77947 -0.06682  0.79079  2.50138 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  11.1215     0.2768  40.180  < 2e-16 ***
## x             1.6575     0.4219   3.929 0.000160 ***
## wT           -0.8130     0.2257  -3.603 0.000498 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.108 on 97 degrees of freedom
## Multiple R-squared:  0.2641,	Adjusted R-squared:  0.2489 
## F-statistic: 17.41 on 2 and 97 DF,  p-value: 3.474e-07
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
##     Min      1Q  Median      3Q     Max 
## -3.3846 -0.7723 -0.0565  0.6902  2.8764 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.6390     0.3748  28.386   <2e-16 ***
## x             1.6765     1.6036   1.045    0.298    
## I(x^2)        0.2586     1.5285   0.169    0.866    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.18 on 97 degrees of freedom
## Multiple R-squared:  0.1659,	Adjusted R-squared:  0.1487 
## F-statistic: 9.643 on 2 and 97 DF,  p-value: 0.0001514
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
## -3.00668 -0.84157 -0.07698  0.79835  2.48156 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9504     0.3681  29.746  < 2e-16 ***
## x             1.9710     0.6127   3.217  0.00177 ** 
## wT           -0.5143     0.4792  -1.073  0.28584    
## x:wT         -0.5990     0.8468  -0.707  0.48107    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.111 on 96 degrees of freedom
## Multiple R-squared:  0.2679,	Adjusted R-squared:  0.245 
## F-statistic: 11.71 on 3 and 96 DF,  p-value: 1.322e-06
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
## -3.00668 -0.84157 -0.07698  0.79835  2.48156 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  10.9504     0.3681  29.746  < 2e-16 ***
## x             1.9710     0.6127   3.217  0.00177 ** 
## wT           -0.5143     0.4792  -1.073  0.28584    
## x:wT         -0.5990     0.8468  -0.707  0.48107    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 1.111 on 96 degrees of freedom
## Multiple R-squared:  0.2679,	Adjusted R-squared:  0.245 
## F-statistic: 11.71 on 3 and 96 DF,  p-value: 1.322e-06
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
##   Res.Df    RSS Df Sum of Sq      F   Pr(>F)   
## 1     98 135.04                                
## 2     96 118.49  2    16.558 6.7077 0.001876 **
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
##   Res.Df    RSS Df Sum of Sq      F   Pr(>F)   
## 1     96 118.49                                
## 2     98 135.04 -2   -16.558 6.7077 0.001876 **
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
##   Res.Df    RSS Df Sum of Sq      F   Pr(>F)   
## 1     98 135.04                                
## 2     96 118.49  2    16.558 6.7077 0.001876 **
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
ただ不偏推定量は保証できない. また 線形推定量のなかで最小の分散とも言えない.
細かく言えば, 独立でなく無相関のとき,
誤差項 $u_i$ の条件付き分散が一定でなくなる.

ただし, 別の分散のもとで正規分布に近似できることがしられている.^[
正確には観測される変数に4次のモーメントが存在するという仮定が必要となる.
この仮定の直感的な意味は異常値が存在しないことである.]
つまり, 説明変数と誤差項が無相関であるが, 独立とまでは言い切れない場合,
最小二乗推定量を実行した際, 別の方法で分散を推定する必要がある.
この別の分散をロバスト分散という.

R でロバスト分散を推定するにはパッケージ `AER` を導入するのが簡単である.
は次のコマンドを実行すればよい.

```r
library(AER)
coeftest(fm1,vcov=vcovHC)
## 
## t test of coefficients:
## 
##             Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) 10.95035    0.35948 30.4618 < 2.2e-16 ***
## x            1.97101    0.58213  3.3859  0.001029 ** 
## wT          -0.51427    0.43165 -1.1914  0.236428    
## x:wT        -0.59900    0.77611 -0.7718  0.442132    
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
## (Intercept) 10.95035    0.34904 31.3729 < 2.2e-16 ***
## x            1.97101    0.56150  3.5102 0.0006836 ***
## wT          -0.51427    0.41752 -1.2317 0.2210560    
## x:wT        -0.59900    0.73995 -0.8095 0.4202196    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
としなければならない.

### 分散不均一の検定
誤差項の分散が均一かどうかは検定可能である.
有名な検定方法としてBP検定というものがある.
BP検定は帰無仮説が分散均一で, 対立仮説が分散が被説名編数と線形関係になっている場合の検定である.

R では以下のように実施すればよい.

```r
bptest(fm1)
## 
## 	studentized Breusch-Pagan test
## 
## data:  fm1
## BP = 2.2834, df = 3, p-value = 0.5157
```

ここでの例ではP値が5%を超えているので帰無仮説を棄却できないので,
分散均一を仮定してよいことが示唆されている.

残差の自乗を被説明変数として回帰分析をおこない,
その決定係数に観測数をかけたものが検定統計量となる.

```r
bpt <- lm(I(resid(fm1)^2)~w*x,data=df)
(bpt <- nrow(df)*summary(bpt)$r.squared)
## [1] 2.283397
1-pchisq(bpt,df=3)
## [1] 0.515709
```

これまでのBPテストは誤差項の分散が説明変数の線形関係あることを暗黙に仮定している.
非線形性を考慮するために説明変数の二次項を導入した分散不均一性の検定をホワイト検定という.
二次項を加えると煩雑になるため, 予測値を使って計算することがある.
ホワイトテストは以下で実施する.

```r
wht <- lm(I(resid(fm1)^2)~fitted(fm1)+I(fitted(fm1)^2),data=df)
(wht <- nrow(df)*summary(wht)$r.squared)
## [1] 4.234388
1-pchisq(wht,df=2)
## [1] 0.1203689
```

ホワイト検定でも分散均一が示唆されている.

このように分散均一性は検定することが可能であるが, そもそも分散均一が疑われる場合は,
ロバスト分散で推定するので十分であるため最近の実証分析ではこの検定は実施されない.



