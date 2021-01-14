---
title: ブラウザのテキスト入力をVim化
date: 2020-06-21
toc: true
tags: 
  - Vim
---

## 初めに
こんにちは
ゴリラです

仕事に限らず、普段はブラウザでテキストを入力することも多々あると思います。
ぼくは、その度にVimのキーバインドを使えたらなと思ったことがこれまで何度もありました。

そこで、[wasavi](http://appsweets.net/wasavi/)というChromeの拡張を教えてもらいました。
試してみた結果とても良かったので紹介します。

Chromeは[こちら](https://chrome.google.com/webstore/detail/wasavi/dgogifpkoilgiofhhhodbodcfgomelhe?hl=ja)からインストールできます。

## どんな感じ？
こんな感じです。
![wasavi.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/9969098a-225a-b73c-b3f8-c1392de16987.gif)

## 使い方
デフォルトは`CTRL-Enter`でViに切り替わります。
`:w`で入力が反映され、`:wq`で反映してViを終了します。

他にもオプションなどを設定できます。

## 設定
いくつかの`ex`コマンドが使用できます。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/89b155b0-bc81-2484-3066-84cc5344340e.png)

デフォルトではbellが鳴るのでそれをオフにすると良いです。（bellのオプションがあるが、なぜか効かなかったので）

```
set bellvolume=0
```


デフォルトは`textarea`のみ対象になっていますが、`input`系も入れておくと便利です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/fb1ebfdb-e878-5122-be35-16a33bfc7483.png)

Vimに切り替えるキーをいくつか設定できます。デフォルトは`CTRL-Enter`です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/1eb7b914-aac1-938a-6367-6182514bf7c7.png)

## さいごに
日本語入力の削除の挙動が若干変ですが、とても便利なのでVimmerの方はぜひ試して見ください
ぼくはQOLが上りまくりです

