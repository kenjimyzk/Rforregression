---
title: "dataframe"
output: html_document
---
# データ構造


## 因子ベクトル
### factor
文字列ベクトルを引数にした関数 `factor`によって 因子 (factor) ベクトルを作成できる.

```r
(x <- c("L","S","M","M","L"))
## [1] "L" "S" "M" "M" "L"
(x.fac <- factor(x))
## [1] L S M M L
## Levels: L M S
```

因子ベクトルの実体は `level` という属性をもつ整数ベクトルである.

```r
typeof(x.fac)
## [1] "integer"
length(x.fac)
## [1] 5
levels(x.fac)
## [1] "L" "M" "S"
```

関数 `class` によって因数ベクトルがであることが確認でき, `str` で属性を詳しく調べることができる.

```r
class(x.fac)
## [1] "factor"
str(x.fac)
##  Factor w/ 3 levels "L","M","S": 1 3 2 2 1
```

また水準の表示順は自動的にアルファベット順になるが, それを変更するには次のようにする.

```r
(x.factor <- factor(x,levels=c("S","M","L")))
## [1] L S M M L
## Levels: S M L
```

この水準に順序構造を付与するには次のようにする.

```r
(x.order <- ordered(x,levels=c("S","M","L")))
## [1] L S M M L
## Levels: S < M < L
```

### cut
また数値を区間ごとに区分した因子ベクトルも作成可能である.
0から10までの値を発生させる.

```r
x <- runif(10,0,10)
x
##  [1] 8.9051434 7.2759914 2.4085461 9.4198060 0.6427695 8.1549472 0.4498490
##  [8] 7.5295806 6.0218593 9.6888404
```

5等分するには以下のようにする.

```r
cut(x, breaks=5)
##  [1] (7.84,9.7]  (5.99,7.84] (2.3,4.15]  (7.84,9.7]  (0.441,2.3]
##  [6] (7.84,9.7]  (0.441,2.3] (5.99,7.84] (5.99,7.84] (7.84,9.7] 
## Levels: (0.441,2.3] (2.3,4.15] (4.15,5.99] (5.99,7.84] (7.84,9.7]
```
これは登場したデータの最大値と最小値の幅を5等分している.

区間を指定するには以下のようにベクトルで指定する.

```r
cut(x,breaks=c(0,2,4,6,8,10))
##  [1] (8,10] (6,8]  (2,4]  (8,10] (0,2]  (8,10] (0,2]  (6,8]  (6,8]  (8,10]
## Levels: (0,2] (2,4] (4,6] (6,8] (8,10]
```
0より大きく2以下, 2より大きく4以下,... となっている.

0も含めるのなら `include.lowest=TRUE` というオプションをつける.

```r
cut(x, breaks=seq(0,10,2),include.lowest=TRUE)
##  [1] (8,10] (6,8]  (2,4]  (8,10] [0,2]  (8,10] [0,2]  (6,8]  (6,8]  (8,10]
## Levels: [0,2] (2,4] (4,6] (6,8] (8,10]
```

またこれを0以上2未満, 2以上4未満, ... とするにはオプション `right=FALSE` をつける.

```r
cut(x, breaks=seq(0,10,2),right=FALSE,include.lowest=TRUE)
##  [1] [8,10] [6,8)  [2,4)  [8,10] [0,2)  [8,10] [0,2)  [6,8)  [6,8)  [8,10]
## Levels: [0,2) [2,4) [4,6) [6,8) [8,10]
```
このとき, `include.lowest=TRUE` は最大値を含めることを意味する.

また因子の名称はオプション `labels` で変更可能である.

```r
cut(x, breaks=seq(0,10,2),right=FALSE,include.lowest=TRUE,
    labels =c("A","B","C","D","E"))
##  [1] E D B E A E A D D E
## Levels: A B C D E
```

## 行列
ベクトルに縦と横の次元を付与することによって行列 (matrix) を作ることができる.

```r
mat <- matrix(1:10, nrow=2,ncol=5)
mat
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    3    5    7    9
## [2,]    2    4    6    8   10
```

またオプション `byrow=TRUE` で横から行列を作成できる.

```r
matrix(1:10, nrow=2,ncol=5,byrow = TRUE)
##      [,1] [,2] [,3] [,4] [,5]
## [1,]    1    2    3    4    5
## [2,]    6    7    8    9   10
```

この行列の実体は `dim` という属性を持つ数値ベクトルである.

```r
typeof(mat)
## [1] "integer"
length(mat)
## [1] 10
dim(mat)
## [1] 2 5
dim(mat)
## [1] 2 5
```

他にもそれぞれの要素の次元は以下で得ることができる.

```r
nrow(mat)
## [1] 2
ncol(mat)
## [1] 5
```

関数 `class` によって因数ベクトルがであることが確認でき, `str` で属性を詳しく調べることができる.

```r
class(mat)
## [1] "matrix"
str(mat)
##  int [1:2, 1:5] 1 2 3 4 5 6 7 8 9 10
```

### 行列のアクセス

行列からベクトルの取り出し

```r
mat[2,]
## [1]  2  4  6  8 10
```

オプション `drop=FALSE` で行列のまま取り出す.

```r
mat[, 3, drop=FALSE]
##      [,1]
## [1,]    5
## [2,]    6
```



```r
mat[,2:3]
##      [,1] [,2]
## [1,]    3    5
## [2,]    4    6
```

要素の取り出し

```r
mat[2,3]
## [1] 6
```

行列に名前を付けることができる.

```r
colnames(mat) <- 1:5
rownames(mat) <- letters[1:2]
mat
##   1 2 3 4  5
## a 1 3 5 7  9
## b 2 4 6 8 10
```

これによって以下のようにしてもアクセス可能である.

```r
mat["a","3"]
## [1] 5
```

### 行列の演算
転置行列は以下のようにすればよい.

```r
t(mat)
##   a  b
## 1 1  2
## 2 3  4
## 3 5  6
## 4 7  8
## 5 9 10
```

通常の `*` は要素ごとの掛け算になる. 
数学で用いられる行列同士の掛け算は `%*%` を実施する.

```r
matb<-matrix(1:4,nrow=2,ncol=2)
matb %*% mat
##       1  2  3  4  5
## [1,]  7 15 23 31 39
## [2,] 10 22 34 46 58
```
行列のことが一致させる必要がある.

また列ごとの合計, 行ごとの合計は以下を実施する.

```r
colSums(mat)
##  1  2  3  4  5 
##  3  7 11 15 19
rowSums(mat)
##  a  b 
## 25 30
```
返り値はベクトルになる. 行列のすべての要素を足すには `sum` でよい.

```r
sum(mat)
## [1] 55
```

また列ごとの平均, 行ごとの平均は以下を実施する.

```r
colMeans(mat)
##   1   2   3   4   5 
## 1.5 3.5 5.5 7.5 9.5
rowMeans(mat)
## a b 
## 5 6
```

## リスト
ベクトルなどを集めたものをリストという.
ベクトル型の違うベクトルを関数 `list` で集める.

リストの属性には長さがある.

```r
(lst <- list("a",c(3,3,2)))
## [[1]]
## [1] "a"
## 
## [[2]]
## [1] 3 3 2
typeof(lst)
## [1] "list"
length(lst)
## [1] 2
```

関数 `class` によってリストがであることが確認でき, `str` で属性を詳しく調べることができる.

```r
class(lst)
## [1] "list"
str(lst)
## List of 2
##  $ : chr "a"
##  $ : num [1:3] 3 3 2
```


リストとベクトルの違いとして, リスト自体もリストとして含められることがある.

```r
typeof(list("b",lst))
## [1] "list"
```

リストベクトルに変換するには `unlist` を用いる.
それぞれの型が違う場合, 同じ型に強制変換される.

```r
lst<-list(1:3,2:6)
lst
## [[1]]
## [1] 1 2 3
## 
## [[2]]
## [1] 2 3 4 5 6
unlist(lst)
## [1] 1 2 3 2 3 4 5 6
unlist(list("a",1:4))
## [1] "a" "1" "2" "3" "4"
```


### リストのアクセス
また次のようにして名前の属性をつけられる.

```r
(lst <- list(name="a",num=c(3,3,2)))
## $name
## [1] "a"
## 
## $num
## [1] 3 3 2
names(lst)
## [1] "name" "num"
```

リストの取りだすには次のようにする.

```r
lst[1]
## $name
## [1] "a"
```
名前の属性がついていれば以下のようにもできる.

```r
lst["num"]
## $num
## [1] 3 3 2
```

いずれにせよ取り出したののもリストになることに注意されたい.

```r
typeof(lst[1])
## [1] "list"
typeof(lst["num"])
## [1] "list"
```

ベクトルとして取り出すには次のようにする.

```r
lst[[2]]
## [1] 3 3 2
typeof(lst[[2]])
## [1] "double"
```
名前の属性がついていれば以下のようにもできる.

```r
lst[["num"]]
## [1] 3 3 2
lst$num
## [1] 3 3 2
```

リストのあるベクトルの平均値を求めるには次のようにする.

```r
mean(lst$num)
## [1] 2.666667
```
もしくは以下とする.

```r
with(lst, mean(num))
## [1] 2.666667
```


## データフレイム
同じ長さのベクトルを組み合わせたリストをデータフレイム (dataframe) という.
データフレイムは次のようにして作られる.

```r
df <- data.frame(x = rnorm(10), y = letters[1:10])
```

データフレイムは大規模なことが多いので最初の数行だけをみるためには `head` をもちいる.

```r
head(df)
##             x y
## 1  0.68087660 a
## 2  0.05609143 b
## 3 -0.94880528 c
## 4 -1.88747111 d
## 5 -0.87095316 e
## 6 -1.23644014 f
```

データの簡単な統計表は `summary` を使うとよい.

```r
summary(df)
##        x                 y    
##  Min.   :-1.8875   a      :1  
##  1st Qu.:-0.9293   b      :1  
##  Median :-0.2854   c      :1  
##  Mean   :-0.3605   d      :1  
##  3rd Qu.: 0.3415   e      :1  
##  Max.   : 0.7364   f      :1  
##                    (Other):4
```

よりプログラム言語としてのデータ構造を調べるには `str` を使えばよい.

```r
str(df)
## 'data.frame':	10 obs. of  2 variables:
##  $ x: num  0.6809 0.0561 -0.9488 -1.8875 -0.871 ...
##  $ y: Factor w/ 10 levels "a","b","c","d",..: 1 2 3 4 5 6 7 8 9 10
```

データフレイムはリストである.

```r
typeof(df)
## [1] "list"
class(df)
## [1] "data.frame"
```

リストのように長さと名前をもつ.

```r
length(df)
## [1] 2
names(df)
## [1] "x" "y"
```

行列と同じ次元をもつ.

```r
dim(df)
## [1] 10  2
ncol(df)
## [1] 2
nrow(df)
## [1] 10
```
`ncol(df)` は `length(df)` と同じである.

行列と同じ名前をもつ.

```r
colnames(df)
## [1] "x" "y"
rownames(df)
##  [1] "1"  "2"  "3"  "4"  "5"  "6"  "7"  "8"  "9"  "10"
```
`colnames(df)` は `names(df)` と同じである.


### データフレイムのアクセス.
データのアクセスとして, ベクトルのように一つのカギカッコで取り出すと以下のようになる.

```r
df["x"]
##               x
## 1   0.680876603
## 2   0.056091433
## 3  -0.948805277
## 4  -1.887471105
## 5  -0.870953156
## 6  -1.236440142
## 7   0.436589512
## 8  -0.569579524
## 9  -0.001311829
## 10  0.736418985
```
他にも `df[1]` でもよいが, いずれにせよデータフレイムとして取り出されてしまう.

ベクトルとして取り出すには以下を実施する.

```r
df$x
##  [1]  0.680876603  0.056091433 -0.948805277 -1.887471105 -0.870953156
##  [6] -1.236440142  0.436589512 -0.569579524 -0.001311829  0.736418985
```
他にも `df[["x"]], df[[1]], df[,"x"], df[,1]` でもよい.
`
ある変数 $x$ の5番目の要素を取り出して別の値`100`を代入するには以下のようにする.

```r
df$x[5] <- 100
```
他にも `df[["x"]][5], df[[1]][5], df[5,"x"], df[5,1]` でもよい.

データフレイムのある変数の平均値を求めるには次のようにする.

```r
mean(df$x)
## [1] 9.726637
```
もしくは以下とする.

```r
with(df, mean(x))
## [1] 9.726637
```
他にも `attach(df)` を使うやり方もあるが, 現在では推奨されない.

