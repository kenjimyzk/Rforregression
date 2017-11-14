---
title: "datawrangling"
output: html_document
---

# 整然データ


http://r4ds.had.co.nz/tidy-data.html

データが整然 (tidy) であるとは次の条件を満たすデータのことである.

1. 個々の変数 (variable) が1つの列 (column) である
2. 個々の観測 (observation) が1つの行 (row) である
3. 個々の値 (value) が1つのセル (cell) である

![tidy data](http://r4ds.had.co.nz/images/tidy-1.png)

これをこのようなデータを使って, データを整形する方法,
またはそうしたデータにする方法を紹介する.

ここでは以下のライブラリに全面に依存する.

```r
library(tidyverse)
```
特に `dplyr` と `tidyr` を用いる.

## データ整形
データセット `mtcars` を取り扱う.
この最初の6つを見るには以下を実施する.

```r
mtcars %>% head()
```

```
##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1
```
ここで `%>%` はパイプ処理といって, 

```r
head(mtcars)
```
と同じ効果をもたらす. 括弧が重複する場合, こちらのほうがわかりやすい.

ライブラリ `dplyr` でデータフレイムの処理が簡単になる.
`select` で変数を選択できる.

```r
mtcars %>% select(mpg, disp) %>% head()
```

```
##                    mpg disp
## Mazda RX4         21.0  160
## Mazda RX4 Wag     21.0  160
## Datsun 710        22.8  108
## Hornet 4 Drive    21.4  258
## Hornet Sportabout 18.7  360
## Valiant           18.1  225
```

`filter` により条件に応じた抽出ができる.

```r
mtcars %>% select(mpg, disp) %>% filter(disp > 300) %>% head()
```

```
##    mpg disp
## 1 18.7  360
## 2 14.3  360
## 3 10.4  472
## 4 10.4  460
## 5 14.7  440
## 6 15.5  318
```

`rename` により変数名を変更することができる. 日本語でも一応対応している.

```r
mtcars %>% select(mpg, disp) %>% rename(速度 =mpg, 距離 =disp) %>% head()
```

```
##                   速度 距離
## Mazda RX4         21.0  160
## Mazda RX4 Wag     21.0  160
## Datsun 710        22.8  108
## Hornet 4 Drive    21.4  258
## Hornet Sportabout 18.7  360
## Valiant           18.1  225
```

`arrange` により順序を変更できる.

```r
mtcars %>% select(mpg, disp) %>% arrange(mpg) %>% head()
```

```
##    mpg disp
## 1 10.4  472
## 2 10.4  460
## 3 13.3  350
## 4 14.3  360
## 5 14.7  440
## 6 15.0  301
```

逆順には以下のようにすればよい.

```r
mtcars %>% select(mpg, disp) %>% arrange(desc(mpg)) %>% head()
```

```
##    mpg  disp
## 1 33.9  71.1
## 2 32.4  78.7
## 3 30.4  75.7
## 4 30.4  95.1
## 5 27.3  79.0
## 6 26.0 120.3
```

新しい変数を作成するには以下のように `mutate` を用いる.

```r
mtcars %>% select(mpg) %>% mutate(gpm = 1/mpg) %>% head()
```

```
##    mpg        gpm
## 1 21.0 0.04761905
## 2 21.0 0.04761905
## 3 22.8 0.04385965
## 4 21.4 0.04672897
## 5 18.7 0.05347594
## 6 18.1 0.05524862
```

以下は20より大きいと`TRUE`, そうでないと `FALSE` をとる変数を作成している.

```r
mtcars %>% select(mpg) %>% mutate(binarympg = ifelse(mpg>20,TRUE,FALSE)) %>% head()
```

```
##    mpg binarympg
## 1 21.0      TRUE
## 2 21.0      TRUE
## 3 22.8      TRUE
## 4 21.4      TRUE
## 5 18.7     FALSE
## 6 18.1     FALSE
```

`summarize` により変数の基本統計表を作成できる.

```r
mtcars %>% summarize(avg = mean(mpg), sd =sd(mpg))
```

```
##        avg       sd
## 1 20.09062 6.026948
```


`group_by` と `summarize` を組み合わせてグループごとの基本統計量も作成できる.


```r
mtcars %>% group_by(cyl) %>% summarize(n = n(), avg = mean(mpg), sd =sd(mpg))
```

```
## # A tibble: 3 x 4
##     cyl     n      avg       sd
##   <dbl> <int>    <dbl>    <dbl>
## 1     4    11 26.66364 4.509828
## 2     6     7 19.74286 1.453567
## 3     8    14 15.10000 2.560048
```

## データ結合
`dplyr` 2つのデータフレイムを結合する便利なコマンドがある.
列の追加は `bind_rows` を用いる.

```r
df1 <- data_frame(X=1:2, Y=1:2)
df2 <- data_frame(X=4, Y=4)
bind_rows(df1,df2)
```

```
## # A tibble: 3 x 2
##       X     Y
##   <dbl> <dbl>
## 1     1     1
## 2     2     2
## 3     4     4
```

行の追加は `bind_rows` を用いる.

```r
df3 <- data_frame(Z=5:6)
bind_cols(df1,df3)
```

```
## # A tibble: 2 x 3
##       X     Y     Z
##   <int> <int> <int>
## 1     1     1     5
## 2     2     2     6
```

2つのデータフレイムで共通部分を用いて結合させるには4つのやり方がある.

```r
dfx <- data_frame(id=c("A","B","C"), X=1:3)
dfy <- data_frame(id=c("A","B","D"), Y=c(TRUE,FALSE,TRUE))
```

左側の `dfx` がすべて残るように結合するには, `left_join` を実行する.

```r
left_join(dfx,dfy,by="id")
```

```
## # A tibble: 3 x 3
##      id     X     Y
##   <chr> <int> <lgl>
## 1     A     1  TRUE
## 2     B     2 FALSE
## 3     C     3    NA
```

右側の `dfx` がすべて残るように結合するには, `right_join` を実行する.

```r
right_join(dfx,dfy,by="id")
```

```
## # A tibble: 3 x 3
##      id     X     Y
##   <chr> <int> <lgl>
## 1     A     1  TRUE
## 2     B     2 FALSE
## 3     D    NA  TRUE
```

両方がすべて残るように結合するには, `full_join` を実行する.

```r
full_join(dfx,dfy,by="id")
```

```
## # A tibble: 4 x 3
##      id     X     Y
##   <chr> <int> <lgl>
## 1     A     1  TRUE
## 2     B     2 FALSE
## 3     C     3    NA
## 4     D    NA  TRUE
```

両方にある行のみ残して結合するには, `inner_join` を実行する.

```r
inner_join(dfx,dfy,by="id")
```

```
## # A tibble: 2 x 3
##      id     X     Y
##   <chr> <int> <lgl>
## 1     A     1  TRUE
## 2     B     2 FALSE
```



## tidyr

以下のデータセットを考える.

```r
df <- data_frame(
  time = 2010:2014,
  X = rnorm(5, 0, 1),
  Y = rnorm(5, 0, 2),
  Z = rnorm(5, 0, 4)
)
df
```

```
## # A tibble: 5 x 4
##    time          X         Y          Z
##   <int>      <dbl>     <dbl>      <dbl>
## 1  2010  1.1965219 1.7206587  -5.713069
## 2  2011 -0.9256137 0.2992468  -4.270570
## 3  2012  1.0729644 0.4861500  -2.440908
## 4  2013  0.1519230 1.5298740  -2.138360
## 5  2014 -0.0661224 0.1758807 -11.287628
```

それぞれの変数名をキーとして, 値を示した表は `gather` で作れる.

```r
df_gather <- df %>% gather(key,value,-time)
df_gather
```

```
## # A tibble: 15 x 3
##     time   key       value
##    <int> <chr>       <dbl>
##  1  2010     X   1.1965219
##  2  2011     X  -0.9256137
##  3  2012     X   1.0729644
##  4  2013     X   0.1519230
##  5  2014     X  -0.0661224
##  6  2010     Y   1.7206587
##  7  2011     Y   0.2992468
##  8  2012     Y   0.4861500
##  9  2013     Y   1.5298740
## 10  2014     Y   0.1758807
## 11  2010     Z  -5.7130690
## 12  2011     Z  -4.2705704
## 13  2012     Z  -2.4409076
## 14  2013     Z  -2.1383600
## 15  2014     Z -11.2876277
```

`spread` でもとに戻ることができる.

```r
df_gather %>% spread(key, value)
```

```
## # A tibble: 5 x 4
##    time          X         Y          Z
## * <int>      <dbl>     <dbl>      <dbl>
## 1  2010  1.1965219 1.7206587  -5.713069
## 2  2011 -0.9256137 0.2992468  -4.270570
## 3  2012  1.0729644 0.4861500  -2.440908
## 4  2013  0.1519230 1.5298740  -2.138360
## 5  2014 -0.0661224 0.1758807 -11.287628
```

`spread` で `time` にすると別の形で展開できる.

```r
df_spread <- df_gather %>% spread(time, value)
df_spread
```

```
## # A tibble: 3 x 6
##     key    `2010`     `2011`    `2012`    `2013`      `2014`
## * <chr>     <dbl>      <dbl>     <dbl>     <dbl>       <dbl>
## 1     X  1.196522 -0.9256137  1.072964  0.151923  -0.0661224
## 2     Y  1.720659  0.2992468  0.486150  1.529874   0.1758807
## 3     Z -5.713069 -4.2705704 -2.440908 -2.138360 -11.2876277
```

以下のようにすればもとに戻る.

```r
df_spread %>% gather(time,value,-key)
```

```
## # A tibble: 15 x 3
##      key  time       value
##    <chr> <chr>       <dbl>
##  1     X  2010   1.1965219
##  2     Y  2010   1.7206587
##  3     Z  2010  -5.7130690
##  4     X  2011  -0.9256137
##  5     Y  2011   0.2992468
##  6     Z  2011  -4.2705704
##  7     X  2012   1.0729644
##  8     Y  2012   0.4861500
##  9     Z  2012  -2.4409076
## 10     X  2013   0.1519230
## 11     Y  2013   1.5298740
## 12     Z  2013  -2.1383600
## 13     X  2014  -0.0661224
## 14     Y  2014   0.1758807
## 15     Z  2014 -11.2876277
```

`gather` をうまく使えば変数ごとの基本統計量の表を作ることができる.

```r
cars %>% gather(variable,value) %>% group_by(variable) %>%
  summarize(nobs = n(), avg = mean(value), sd =sd(value))
```

```
## # A tibble: 2 x 4
##   variable  nobs   avg        sd
##      <chr> <int> <dbl>     <dbl>
## 1     dist    50 42.98 25.769377
## 2    speed    50 15.40  5.287644
```

日本語だと次のようにすればよい.

```r
tab <- cars %>% rename(距離=dist,速度=speed ) %>%
  gather(変数,value) %>% group_by(変数) %>%
  summarize(観測数 = n(), 平均 = mean(value), 標準偏差 =sd(value))
head(tab)
```

```
## # A tibble: 2 x 4
##    変数 観測数  平均  標準偏差
##   <chr>  <int> <dbl>     <dbl>
## 1  距離     50 42.98 25.769377
## 2  速度     50 15.40  5.287644
```

## 実践例
tidyr を用いた別の例をみてみよう.
横軸を年としたデータセット `df` がある.

```r
df <- data_frame(name=letters, "2010"=rnorm(26),"2011"=rnorm(26),"2012"=rnorm(26)) 
head(df)
```

```
## # A tibble: 6 x 4
##    name      `2010`       `2011`     `2012`
##   <chr>       <dbl>        <dbl>      <dbl>
## 1     a  1.18740184 -0.374725013  0.1329990
## 2     b  0.08926124 -1.383157218  0.7050418
## 3     c -0.98475038 -1.279833988  1.2406488
## 4     d  0.72549173  0.603919931 -0.4531015
## 5     e  0.46533016  0.002044949  0.7365002
## 6     f  0.05959534 -1.032577896 -0.2336940
```
`data.frame` でないことに注意されたい.

また, 年ごとのデータセット `df_2010`, `df_2011`, `df_2012` が3つある.

```r
df_2010 <- data_frame(name=letters,runif=runif(26))
df_2011 <- data_frame(name=letters,runif=runif(26))
df_2012 <- data_frame(name=letters,runif=runif(26))
```
これら4つのデータセットを1つにまとめよう.

まずデータ `df` についてであるが, `gather` をつかう.

```r
df_rnorm <- df %>% gather(time,rnorm,-name) %>%
  mutate(time=as.numeric(time))
head(df_rnorm)
```

```
## # A tibble: 6 x 3
##    name  time       rnorm
##   <chr> <dbl>       <dbl>
## 1     a  2010  1.18740184
## 2     b  2010  0.08926124
## 3     c  2010 -0.98475038
## 4     d  2010  0.72549173
## 5     e  2010  0.46533016
## 6     f  2010  0.05959534
```
時間の変数を数値に変換している.

データセット `df_2010`, `df_2011`, `df_2012` をつなげる.

```r
df_runif <- bind_rows(df_2010,df_2011,df_2012) %>% 
  bind_cols(time=rep(2010:2012,each=26)) 
head(df_runif)
```

```
## # A tibble: 6 x 3
##    name     runif  time
##   <chr>     <dbl> <int>
## 1     a 0.3470124  2010
## 2     b 0.3747049  2010
## 3     c 0.4459528  2010
## 4     d 0.2198896  2010
## 5     e 0.4818801  2010
## 6     f 0.4591683  2010
```
それぞれの年の変数を付け加えている.

これを `full_join` を用いてつなげる.

```r
df_full <- full_join(df_rnorm,df_runif,by=c("name","time"))
head(df_full)  
```

```
## # A tibble: 6 x 4
##    name  time       rnorm     runif
##   <chr> <dbl>       <dbl>     <dbl>
## 1     a  2010  1.18740184 0.3470124
## 2     b  2010  0.08926124 0.3747049
## 3     c  2010 -0.98475038 0.4459528
## 4     d  2010  0.72549173 0.2198896
## 5     e  2010  0.46533016 0.4818801
## 6     f  2010  0.05959534 0.4591683
```
