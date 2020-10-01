--- 
title: "R と RStudio"
author: "宮﨑憲治"
date: "2020-10-01"
site: bookdown::bookdown_site
output: bookdown::gitbook
documentclass: bxjsbook
classoption: ja=standard, xelatex
geometry: no
# bibliography: template.bib
biblio-style: jeconunicode
link-citations: yes
# csl: apa-jp_JSAM.csl
lang: ja-JP
# github-repo: (your-github-repository)
description: "This is a description. Please wrote a abstract this contents. "
---
# はじめに {-}



## R とは
Rは統計・データ解析・統計グラフ作成のためのオープンソースソフトである.

## R のインストール
Rをインストールするには

https://cran.r-project.org/

にいき, 該当機種のファイルをダウンロードする.
ダウンロードしたあとに実行すればインストールされる.

Windows の場合, 32bit か 64bit を選択する. 
最近のパソコンの CPU は 64bit と考えられるが,
どちらかわからなければ 32bit にしておけばよい.

Ubuntu なら `ppa` を使って導入してもよい.
```
sudo add-apt-repository ppa:marutter/rrutter
sudo apt-get update
sudo apt-get install r-base r-base-dev
```

## R の設定
設定ファイル `.Rprofile` をホームディレクトリに作成すれば, 設定を変更できる.
ホームディレクトリはユーザー名が `kenji` のとき, Windows なら通常
`C:\Users\kenji` である. バックスラッシュ `\` は $\yen$ と読み替えて頂きたい. 
最初は `.Rprofile` を特に作成しなくても大丈夫である.


## R の使い方
Windows だとコマンドプロンプトから R と入力して立ち上げるか, R のアイコンをダブルクリックすると, Rコンソールと言われる画面が現れる.
コマンドプロンプトからだと最初の表示が文字化けしている可能性があるが, その後の起動に問題ないはずである.
アイコンがなければ, Winキーを押した後, `rgui` と入力すれば起動できる.
Mac や Ubuntu だとターミナルから R と入力して起動できる.

立ち上げた後, コンソールから
そこにコマンドを入力すると, その結果が直後に出力される.
終了には `q()` とする. 
作業スペースを保存するかと聞かれたなら, No を意味する `n` を選択する.

コマンド入力中, 最後の括弧を付け忘れたり, 正しく実行ができないときがある.
たとえば, `rnorm(5` としてEnterキーを押せば, 次の行に `+` とでてくる.
ここでは正しく `)` を付けて再度Enterキーを押せば正しく実行されるが,
ときにはどれを入力すれば正しく実行されるかわからない一方で, 単にEnterを押すだけだと, 再度入力を求められることがある.
そうしたとときは通常左上にあるエスケープキー (ESC) を押せば途中入力がキャンセルされる.

R はRコンソールから対話式にコマンドを入力していく方法と,
拡張子 `R` のスクリプトファイルを実行していくやり方がある.
実行履歴を記録するためにスクリプトファイルを作成していくやり方を推奨する.

スクリプトファイル `project.R` を Rコンソールから実行するには,
```
source("project.R")
```
とすればよい. 

R外部のコマンドプロンプトから実行するには
```
Rscript project.R
```
とすればよい. 起動できないときには,
環境変数の PATH にRの実行ファイルの場所が登録されていない可能性がある.

また外部ファイルを導入する際やファイルを外部出力する際には, 現在の作業ディレクトリに気をつけなければならない.
現在の作業ディレクトリの場所はRコンソールから
```
getwd()
```
とすれば, 確認できる. Windows だと通常の表記と異なっていることに注意されたい.

作業ディレクトリの指定は以下のようにする.
```
setwd(PATH)
```
Windows のとき指定の仕方に注意が必要である.
たとえば作業ディレクトリが `C:\Users\kenji\work\project` のとき,
```
setwd("C:/Users/kenji/work/project")
```
となる. バックスラッシュ `\` ($\yen$) を スラッシュ `/` に変更しなければならない.


