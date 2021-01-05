---
title: Gyazoに画像をアップロードするVimプラグインを作った
date: 2020-11-16
toc: true
tags: 
  - Vim
---

zenn.devやQiitaで記事を書いていると画像をアップロードしたくなることがあります。
そして、とても良いことにzenn.devやQiitaにはWebのエディタ上では`Ctrl(Cmd) + v`でクリップボードから画像をアップロードできる機能があります。

しかし普段Vimで記事を書いている自分としては、やはりVimだけで完結したいという気持ちがあります。
クリップボードの画像をアップロードのために、わざわざWebのエディタを開きたくないし、わざわざファイルに書き出したくないです。

しかし、zenn.devとQiitaは画像アップロードのAPIを公開していないので長らく諦めていましたが、[Gyazo](https://gyazo.com)というサービスがあるのを知りました。
GyazoにはAPIがあるので、それをWebエディタと使えば同じことができると思い[gyazo.vim](https://github.com/skanehira/gyazo.vim)というプラグインを作りました。

![](https://i.gyazo.com/2adcdcc57f144bd524bc29bd1affbe75.gif)

やっていることはシンプルで`gyazo`のCLIを使ってAPIを叩いているだけです。URLが返ってくるのでそれをそのまま現在行に挿入させるだけで同じような動作を再現できます。
