---
title: Vimでソースコードを画像化してクリップボードにコピー
date: 2020-08-06
toc: true
tags: 
  - Vim
---

## 初めに
こんにちわ
ゴリラです

[Goでソースコードを画像化するCLIを作った](https://qiita.com/gorilla0513/items/013aea9060bca1455137) でソースコードを画像化するCLIを紹介しましたが、Vimからもう少し使いやすくするためプラグインを作りました。

<a href="https://github.com/skanehira/code2img.vim"><img src="https://github-link-card.s3.ap-northeast-1.amazonaws.com/skanehira/code2img.vim.png" width="460px"></a>

## 使い方
次のとおりです

```vim
" 選択した範囲を画像化してクリップボードにコピー
:'<'>Code2img

" ファイル全体を画像化してクリップボードにコピー
:Code2img

" ファイルに出力
:Code2img image.png

" 選択範囲を画像化してファイルに出力
:'<'>Code2img image.png
```

## 仕組み
Vimの`job_start`を使って`code2img`を実行しているだけです。
引数がなければクリップボードにコピー、引数があればファイルに出力するように引数を組み合わせています。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/32395da9-4f89-5baf-12e3-934d3fddd221.png)

## さいごに
久しぶりにVimプラグイン作りましたが、楽かったです
あとやはり書かないと色々忘れるなと思いました
定期的にプラグインを作りたい（気持ちはある）

