---
title: Vimのターミナルをすこし使いやすくするの調査
date: 2020-07-26
toc: true
tags: 
  - Vim
---

## 初めに
こんにちわ
ゴリラです

Vimでターミナルを使うときは`:term ++close +shell {cmd}`というふうに実行することが多々ありますが、
オプションが多くなると、いささか不便に感じます。

そこで、`BufReadCmd`を使えば、ターミナルがちょっと使いやすくなります。

# やりかた

```vim
function! s:termopen() abort
  let name = split(bufname(), '\/\/')
  if len(name) < 2
    return
  endif
  call execute(printf('term ++curwin ++close ++shell %s', name[1]))
endfunction

augroup terminal
  au!
  au BufReadCmd term://* call s:termopen()
augroup END
```

上記のコードを`vimrc`に追記&リロードして、`:vnew term://top` すると、ターミナルが起動して`top`コマンドが動きます。

![vim-term.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/346ec215-2109-3c5f-c01c-9601c7ae6054.gif)

## 仕組み
仕組みは簡単で、bufferが作られたら `BufReadcmd` イベントが走るので、その時 `s:termopen` を実行させるように `autocmd` を設定します。
`s:termopen`関数内ではバッファ名からterminalで実行するコマンド(`//`よりも後ろの部分)を取得してターミナルを起動します。

`//`よりも後ろがコマンドになるので、例えば`tabnew term://bash`でも動くし、`new term://docker exec -it golang bash`でも動きます。
設定自体はシンプルでかつ柔軟にコマンドを実行できるので個人的に便利と思っています。

## さいごに
このターミナルのカスタマイズはneovimから着想を得ました。neovimでは標準で同じようなことが出来ます(ヘルプより抜粋)。

```
- Edit a file with a name matching `term://(.{-}//(\d+:)?)?\zs.*`.
  For example:

    :edit term://bash
    :vsplit term://top
```

