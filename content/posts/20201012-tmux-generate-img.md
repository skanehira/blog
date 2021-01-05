---
title: tmuxで選択したテキストを画像化&クリップボードにコピー
date: 2020-10-12
toc: true
tags: 
  - tmux
---

## 始めに
tmuxで選択した範囲のテキストを画像にしてクリップボードにコピーしたいことがたまにあります。
ちょうど、tmuxにはコピーモードでテキストをコピーする方法があるので、あとはコマンドを用意すれば実現できそうだなと思ってやってみました。

## やり方
ゴリラ製の[code2img](https://github.com/skanehira/code2img)をインストールして、tmux.confに次の設定をします。

```
# コピーモードでvimキーバインドを使う
setw -g mode-keys vi

# 選択範囲を画像化
bind-key -T copy-mode-vi C-i send-keys -X copy-pipe-and-cancel "code2img -c -ext sh"
```

あとはtmuxを再起動して、範囲選択したら`Ctrl + i`で画像がクリップボードにコピーされます。キーバインドはご自由に変えてください。

## 余談
Vimで選択したテキストを画像化したいときは[code2img.vim](https://github.com/skanehira/code2img.vim)を使うと便利です。
お試しください。
