---
title: Vimで画像をツイートする
date: 2020-08-17
toc: true
tags: 
  - Vim
---

## 初めに
ども、連休が終わったのを未だに信じられないゴリラです

先日 [code2img.vim](https://github.com/skanehira/code2img.vim) というプラグインをつくったんですが、
基本ソースコードを画像にして共有するのはTwitterなので、それらをまとめてVimで出来たら良いなと思って、簡単なスクリプトを書きました。

![tweet_with_image.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/99185e3a-90d5-9f73-ea7d-c8d52a266c63.gif)

コードの本体は[ここ](https://gist.github.com/skanehira/7dd6ed0dc8da8c6e87a11ab70ea83b53)に置いてあります。

## 必要なもの
- [code2img](https://github.com/skanehira/code2img)
- [twty](https://github.com/mattn/twty)

## 使い方
コードをvimrcに記述すれば`TweetWithImg`コマンドが使えるようになります。

- 選択した範囲のテキストが画像として出力される
- コマンドの引数はツイート本文として送信される

ので、`TweetWithImg 私はゴリラです`というふう実行すれば`私はゴリラです`と画像つきのツイートがつぶやけます。

仕組みはシンプルで、`code2img`でソースコードを画像ファイルに出力して、`twty`で画像ファイルと本文をツイートするだけです。
`job_start`なので非同期で動くのでVimが固まることがないです(これ重要)

## さいごに
やはりコードを画像化してさくっとTwtterで共有できるのは良い体験

