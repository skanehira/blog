---
title: Vimで:cexprを使ってgrep結果をquickfixに流す
date: 2020-09-18
toc: true
tags: 
  - Vim
---

## 始めに
みなさんはVimで一度くらいはgrepしたことがあるかと思います。
基本`:grep`で物足りますが、ちょっとシェルの画面になるのはあまり行けていない感じがします。

そこで`cexpr`と`system`と`cw`を組み合わせることでシェル画面にならず、quickfixに結果を流し込めるので、
それをコマンド化してチョット使いやすくする方法を紹介します。

### やり方
次をvimrcに貼り付ければ`Grep`が使えます。

```vim
function! s:grep(word) abort
  cexpr system(printf('ag "%s"', a:word)) | cw
endfunction

command! -nargs=1 Grep call <SID>grep(<q-args>)
```

![](https://storage.googleapis.com/zenn-user-upload/3adzzygk1ggqaqu1g45dwitc9525)

`cexpr {expr}`は`{expr}`の結果をquickfixに流し込んでくれます。
`cw`はquickfixを開きます。

`system`は外部コマンドを実行してくれるので、お好きなgrepコマンドを置き換えても問題ないです。
`grep`コマンドの出力結果によっては`errorformat`の設定を返る必要はありますのでご注意ください。

#### 追記2020-10-16
`cgetexpr`を使えば、grep結果の1つ目にジャンプしなくなります。
自動でジャンプしたくない方は`cgetexpr`を使うと良きです。

### 余談
もともと、[fzf.vim](https://github.com/junegunn/fzf.vim)の`Ag`コマンドでgrepをしてましたが、
次のエラーが起きてgrepできなくなっていました。

```vim
Error running 'fzf'  '--ansi' '--prompt' 'Ag> ' '--multi' '--bind' 'alt-a:select-all,alt-d:deselect-all' '
--delimiter' ':' '--preview-window' '+{2}-5' '--color' 'hl:4,hl+:12' --expect=ctrl-v,ctrl-x,ctrl-t --heigh
t=22 > /tmp/vjUnOWu/3
```

ちょっと原因調べるの面倒ですので、ちょっと自作コマンドを用意することにしました。
今のところは快適ですが、`system`はVimの操作をブロックしてしまうので、
困ったら`job_start`を使って改善するかもしれません。
