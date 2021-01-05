---
title: Vimの:wと:rの便利Tips
date: 2020-10-05
toc: true
tags: 
  - Vim
---

## 始めに
Vimには`:w`と`:r`コマンドがあります。コマンド自体は知っている方が多いかと思います。
ぼく的にちょっと便利な使い方ができるので、それお紹介していこうと思います。

## `:r`
`:r banana.txt`でファイルの中身を読み取って、現在のカーソルの次の行に挿入してくれます。
挿入できる行も指定できます。その場合は`:{lnum}r banana.txt`というふうに先頭に行番号を入力します。

![](https://storage.googleapis.com/zenn-user-upload/g6g1zofrp56zqr5jrj8znju1dvgy)

ここからがイチオシですが、実は`:r !{cmd}`でコマンドの出力も挿入できます。
たとえば、APIのレスポンスをVimでちょっと編集したい場合や、コマンド実行結果を記事に挿入したい場合などに便利です。

![](https://storage.googleapis.com/zenn-user-upload/7denlktdtsg3yvr22v8l010v8jkx)

## `:w`
`:w`についてVimmerのみなさんなら誰もが知っているコマンドなので説明は省きますが、
実は`:w !{cmd}`でバッファの内容を外部コマンドの標準入力として渡してくれます。

つまり、標準入力からコードを受け取って実行できるインタプリタがあれば、
**ファイルをいちいち保存しなくても**サクッとコード片を実行できるのです。

たとえば、`:w !node`ならJavaScriptのコードを実行できるし、`:w !bash`ならシェルスクリプトを実行できます。

実際に、サンプルコード[^1]を実行すると結果が出力されます。

![](https://storage.googleapis.com/zenn-user-upload/ltv5t73vsbeivd3q659r2lyr3w02)

[^1]: [こちら](https://zenn.dev/uhyo/articles/array-n-keys-yamero)の記事から拝借。

## `:r`と`:w`の組み合わせ
この2つコマンドを組み合わせると、Vimでdockerのコンテナをまとめて停止できたりします。

![](https://storage.googleapis.com/zenn-user-upload/3ier74brh8ayykfgdotqa2m2klh3)

この2つのコマンドの詳細は`:h :r`と`:h :w`を参照ください。
解説した内容全部書いてあります。

## さいごに
`:r`と`:w`は便利。
