---
title: Vim初めて2年間つくったVimプラグインを振り返る
date: 2020-09-28
toc: true
tags: 
  - Vim
---

## 初めに
最近zenn.devでVimプラグイン周りの記事がいくつか投稿されています。

- [スニペットプラグインについて 2020 年版](https://zenn.dev/shougo/articles/snippet-plugins-2020)
- [Vim の超軽量ファイラを作った](https://zenn.dev/mattn/articles/1da58d6ee91a8e36e690)
- [Vimにたくさんあるファジーファインダー系プラグインを比較してみる](https://zenn.dev/yutakatay/articles/vim-fuzzy-finder)
- [2020秋 Vim のファイラー系プラグイン比較](https://zenn.dev/lambdalisue/articles/3deb92360546d526381f)

せっかくだし、2018年の8月ころからVimを使い始めて2年くらい経ったので、これまで作成したVimプラグインを時系列で振り返ってみます。

## 初めてのプラグイン
Vim触り始めて数ヵ月のベイビーVimmerの状態だったが、すっかりVimにハマっていたので、自分でも何かプラグインを作りたいと思っていました。
ちょうどVimのセッション機能を知ったし、これチョット使いやすくできるかもと思って作ったのが[vsession](https://github.com/skanehira/vsession)です。

初のプラグイン開発ですので、 [こちら](https://qiita.com/aratana_tamutomo/items/4f754c301fb911ff54e8)の記事を参考に作っていましたが、
plugin、autoload、開発フロー周りがよくわからず困っていた記憶があります。
ただ、思ったよりも難しくなかったので、意外とこんなもんかって思ったりしました。

## 翻訳プラグイン
英語苦手なのでよくGoogle翻訳を使っていましたが、Vim上でGoogle翻訳できたら便利と思って、プラグインを作り始めました。
このころpopup window機能が入ったので、オシャレだし新機能だから試してみようと、翻訳したメッセージをpopup windowで表示させていました。

popup window周りをいろいろといじっていたのが記憶に残っています。

![](https://i.imgur.com/p3WsE8P.gif)

## 電車乗換案内プラグイン
ある日突然Vimで電車乗り換えを検索できたら便利と思い作りました。

しくみはシンプルに、Yahoo!乗換案内をスクレイピングした情報をpopup windowに表示させていました。
しかしこちらは利用規約に反すると怒られたため非公開となりました。

利用規約、ダイジ、ゼッタイ。

@[tweet](https://twitter.com/gorilla0513/status/1140401248115367936)

## `Bad Apple!!`
このころは、何でもかんでもVimで何かをやろうと思っていました。Vimで`Bad Apple!!`を流せたら良いなと思って作ったのが[badapple.vim](https://github.com/skanehira/badapple.vim)です。
業務中にひっそり懐かしい`Bad Apple!!`を見れたらやる気も出て効率上がるんじゃないかなという狙いでしたが、結局一度も業務中で使用することはありませんでした。

![](https://github.com/skanehira/badapple.vim/blob/master/screenshots/screenshot.gif?raw=true)

## dockerを操作するプラグイン
普段の開発でdockerの操作を自作の[docui](https://github.com/skanehira/docui)を使っていたが、
画面分割してからdocuiを起動するまでの操作による時間ロスが気になっていました。
ですので、Vimでサクッと操作できるdockerを操作できるプラグインを作りました。

このプラグインはこれまで一番頑張ったもので、便利そうと思った機能もたくさん実装しました。
でも実際使っているのはコンテナの起動、停止、アタッチ、イメージのpullと削除くらいです。
ほかにも高機能（CPU、MEM使用率をリアルタイムで表示するなど）がありますが、ほぼ使っていないので無駄な高機能になってしまいました。

必要な機能だけ実装して、需要や要望があれば実装、というのが良いんだろうなと今になって思います。

![](https://imgur.com/5h1FufL.gif)

## タピオカプラグイン
このころ、世の中ではタピオカブームで、せっかくなのでVimでタピオカを量産するプラグインを作りました。
残念ながら実務で使用することはついぞありませんでした。

![](https://i.imgur.com/Up93dk7.gif)

## コードを画像化するプラグイン
普段、社内チャットでやSNSでサンプルコードを貼ったりしますが、
コードシンタックスが使えないのでとても不便でした。

一番理想ですので、Vimで書いたコードをそのまま選択して画像化、そしてクリップボードにコピーすることです。
それができる[code2img](https://github.com/skanehira/code2img)を作り、Vimから使えるよう[code2img.vim](https://github.com/skanehira/code2img.vim)を作りしました。

うれしいことでcode2imgは[MOONGIFT](https://www.moongift.jp/2020/08/code2img-コードを画像化するコマンド/)で紹介されました。

## GitHubを操作できるプラグイン
今年に入って仕事でGitHubを使うようになってブラウザでの作業がだいぶ増えました。
issue作成、確認、コメント、PR作成、レビューなどができます。

ブラウザでの作業が増えるということは必然的にVimを触る時間が減るということですので、Vimmerとしてはあまりうれしくないことです。
そこで、できるだけVimで作業できるように[gh.vim](https://github.com/skanehira/gh.vim)を作りました。

できたてホヤホヤのプラグインですが、普段の作業の6割くらいはVimできるようになったので、一安心しています。

![](https://i.imgur.com/VK6rebH.gif)

## 時系列まとめ
他にも解説していないプラグインがいくつかありますが、時系列にまとめるとこうなりました。
こうやってみると、今年入ってからプラグインを作っている回数が減りましたね。
個人的に2年間でこれしか作っていないのかという気持ちです。

| 日時       | プラグイン名                                                              | 説明                                                   |
|------------|---------------------------------------------------------------------------|--------------------------------------------------------|
| 2018/12/21 | [vsession](https://github.com/skanehira/vsession)                         | セッションのプラグイン                                 |
| 2019/05/06 | [translate.vim](https://github.com/skanehira/translate.vim)               | Google翻訳                                             |
| 2019/06/16 | train.vim                                                                 | 電車乗換案内（非公開）                                 |
| 2019/06/19 | [docker.vim](https://github.com/skanehira/docker.vim)                     | Dockerのコンテナ、イメージなどの操作                   |
| 2019/06/21 | weather.vim                                                               | 天気予報（非公開）                                     |
| 2019/07/20 | [badapple.vim](https://github.com/skanehira/badapple.vim)                 | `Bad Apple!!`を再生                                    |
| 2019/08/17 | [generatedir.vim](https://github.com/skanehira/generatedir.vim)           | プロジェクトレイアウトを生成                           |
| 2019/09/26 | [docker-compose.vim](https://github.com/skanehira/docker-compose.vim)     | docker-composeのラッパ                                 |
| 2019/10/05 | [say.vim](https://github.com/skanehira/say.vim)                           | Macの`play`のラッパ                                    |
| 2019/11/10 | [popupfiles.vim](https://github.com/skanehira/popupfiles.vim)             | popup windowを使ったファイルセレクタ                   |
| 2019/11/13 | [tapioca.vim](https://github.com/skanehira/tapioca.vim)                   | 文字を打つとタピオカになる                             |
| 2020/01/08 | [preview-markdown.vim](https://github.com/skanehira/preview-markdown.vim) | Markdownのプレビュー                                   |
| 2020/03/03 | [google-map.vim](https://github.com/skanehira/google-map.vim)             | Google Mapsのルート検索ができる                        |
| 2020/08/02 | [code2img.vim](https://github.com/skanehira/code2img.vim)                 | 選択した範囲のコードを画像にしてクリップボードにコピー |
| 2020/09/06 | [gh.vim](https://github.com/skanehira/gh.vim)                             | GitHubのissueやPR、リポジトリなどを操作できる          |


## 最後に

いくつかはお遊びプラグインですが、業務で役に立っているプラグインもあって、作ってよかったなと思っています。
これからも、Vimに引きこもって生活できるようにプラグインを作っていきます。
