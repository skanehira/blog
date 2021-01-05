---
title: tmuxのpopupが便利
date: 2020-10-25
toc: true
tags: 
  - tmux
---

## 初めに
仕事で複数プロジェクトを同時に使っていることが多く、`tmux`を愛用しています。
`tmux`を使えば、簡易に画面を増やしたり分割したりできますが、次のような不便さを感じるときがあります。

- 画面分割すると画面サイズが小さくなる
- 画面数が増えるとよく使うプロジェクトの画面をすぐ出せない

そこで、最近入ったpopup機能を使ってみたところ上記の課題を解決できそうだったので、軽く紹介をしていきます。
こんな感じで画面分割ではなく、画面中央にウィンドウを出せます。

![](https://storage.googleapis.com/zenn-user-upload/u63jvksh7pulaw5lg843r3v5k89g)

## やり方
筆者の環境は次になっています。

- tmux next-3.3
- fish version 3.1.2

[@yutakatayさん](https://twitter.com/yutakatay)から教えていただいたものをベースにfishの関数を用意します。
bashやzshの方は適宜変えてみてください。

```fish:tmuxpopup.fish
function tmuxpopup -d "toggle tmux popup window"
  set width '80%'
  set height '80%'
  set session (tmux display-message -p -F "#{session_name}")
  if contains "popup" $session
    tmux detach-client
  else
    tmux popup -d '#{pane_current_path}' -xC -yC -w$width -h$height -K -E -R "tmux attach -t popup || tmux new -s popup"
  end
end
```

上記を用意したら、`tmuxpopup`を実行すると画面中央にウィンドウが表示されます。関数の動きは次の通りです。

1. 今のセッションがpopupセッションならdetach
2. popupセッションがなければ新規作成、あればattachする

あとはtmuxのキーバインドを設定します。

```fish:tmux.conf
bind -n C-q run-shell "fish -c \"tmuxpopup\""
```

これでいつでも`C-q`でpopupをtoggleできます。

## 最後に
`tmux`はやはり便利ですね。
