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

```
## Loading tidyverse: ggplot2
## Loading tidyverse: tibble
## Loading tidyverse: tidyr
## Loading tidyverse: readr
## Loading tidyverse: purrr
## Loading tidyverse: dplyr
```

```
## Warning: パッケージ 'dplyr' はバージョン 3.4.2 の R の下で造られました
```

```
## Conflicts with tidy packages ----------------------------------------------
```

```
## filter(): dplyr, stats
## lag():    dplyr, stats
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
##    time          X          Y         Z
##   <int>      <dbl>      <dbl>     <dbl>
## 1  2010  0.6293810  1.4169667  1.929605
## 2  2011 -1.6580129  1.0370836  4.357563
## 3  2012  0.4207063  3.5639349  6.275122
## 4  2013 -0.6887907 -1.1305699 -2.691399
## 5  2014  1.0434867 -0.4531038  2.899786
```

それぞれの変数名をキーとして, 値を示した表は `gather` で作れる.

```r
df_gather <- df %>% gather(key,value,-time)
df_gather
```

```
## # A tibble: 15 x 3
##     time   key      value
##    <int> <chr>      <dbl>
##  1  2010     X  0.6293810
##  2  2011     X -1.6580129
##  3  2012     X  0.4207063
##  4  2013     X -0.6887907
##  5  2014     X  1.0434867
##  6  2010     Y  1.4169667
##  7  2011     Y  1.0370836
##  8  2012     Y  3.5639349
##  9  2013     Y -1.1305699
## 10  2014     Y -0.4531038
## 11  2010     Z  1.9296053
## 12  2011     Z  4.3575630
## 13  2012     Z  6.2751223
## 14  2013     Z -2.6913992
## 15  2014     Z  2.8997856
```

`spread` でもとに戻ることができる.

```r
df_gather %>% spread(key, value)
```

```
## # A tibble: 5 x 4
##    time          X          Y         Z
## * <int>      <dbl>      <dbl>     <dbl>
## 1  2010  0.6293810  1.4169667  1.929605
## 2  2011 -1.6580129  1.0370836  4.357563
## 3  2012  0.4207063  3.5639349  6.275122
## 4  2013 -0.6887907 -1.1305699 -2.691399
## 5  2014  1.0434867 -0.4531038  2.899786
```

`spread` で `time` にすると別の形で展開できる.

```r
df_spread <- df_gather %>% spread(time, value)
df_spread
```

```
## # A tibble: 3 x 6
##     key   `2010`    `2011`    `2012`     `2013`     `2014`
## * <chr>    <dbl>     <dbl>     <dbl>      <dbl>      <dbl>
## 1     X 0.629381 -1.658013 0.4207063 -0.6887907  1.0434867
## 2     Y 1.416967  1.037084 3.5639349 -1.1305699 -0.4531038
## 3     Z 1.929605  4.357563 6.2751223 -2.6913992  2.8997856
```

以下のようにすればもとに戻る.

```r
df_spread %>% gather(time,value,-key)
```

```
## # A tibble: 15 x 3
##      key  time      value
##    <chr> <chr>      <dbl>
##  1     X  2010  0.6293810
##  2     Y  2010  1.4169667
##  3     Z  2010  1.9296053
##  4     X  2011 -1.6580129
##  5     Y  2011  1.0370836
##  6     Z  2011  4.3575630
##  7     X  2012  0.4207063
##  8     Y  2012  3.5639349
##  9     Z  2012  6.2751223
## 10     X  2013 -0.6887907
## 11     Y  2013 -1.1305699
## 12     Z  2013 -2.6913992
## 13     X  2014  1.0434867
## 14     Y  2014 -0.4531038
## 15     Z  2014  2.8997856
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
##    name      `2010`     `2011`      `2012`
##   <chr>       <dbl>      <dbl>       <dbl>
## 1     a  0.54526213  0.2129262 -1.19758856
## 2     b -0.14287345 -0.3619152  0.23161425
## 3     c -0.46127337  1.6378412  0.99236131
## 4     d  0.04435301 -1.2559851 -0.61822128
## 5     e -0.63677230 -0.6669380  0.03118362
## 6     f -1.32648226 -0.2426282  0.37369492
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
## 1     a  2010  0.54526213
## 2     b  2010 -0.14287345
## 3     c  2010 -0.46127337
## 4     d  2010  0.04435301
## 5     e  2010 -0.63677230
## 6     f  2010 -1.32648226
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
##    name      runif  time
##   <chr>      <dbl> <int>
## 1     a 0.36776594  2010
## 2     b 0.97536725  2010
## 3     c 0.00743164  2010
## 4     d 0.33568691  2010
## 5     e 0.97664153  2010
## 6     f 0.77630955  2010
```
それぞれの年の変数を付け加えている.

これを `full_join` を用いてつなげる.

```r
df_full <- full_join(df_rnorm,df_runif,by=c("name","time"))
head(df_full)  
```

```
## # A tibble: 6 x 4
##    name  time       rnorm      runif
##   <chr> <dbl>       <dbl>      <dbl>
## 1     a  2010  0.54526213 0.36776594
## 2     b  2010 -0.14287345 0.97536725
## 3     c  2010 -0.46127337 0.00743164
## 4     d  2010  0.04435301 0.33568691
## 5     e  2010 -0.63677230 0.97664153
## 6     f  2010 -1.32648226 0.77630955
```
