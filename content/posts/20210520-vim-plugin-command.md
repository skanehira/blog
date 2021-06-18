---
title: "Vimでシェルコマンドを簡単に実行するcommand.vimを作った"
date: 2021-05-20
toc: true
tags:
  - Vim
  - denops
  - deno
---

## 初めに
普段Vimでターミナルを使ってちょっとしたコマンドを実行することがよくあります。
たとえば`gh pr create`や`lazygit`、`docui`といった、インタラクティブな操作を必要とするコマンドが多いです。
その度に`:term xxx`と入力するのは不便だし、コマンドの履歴補完が効かないので、[command.vim](https://github.com/skanehira/command.vim)というプラグインを作りました。

本記事はプラグインの紹介と作るにあたって苦労したことについて書いていきます。

## 使い方
![](https://i.gyazo.com/3b703f3d888526e282693d386051f59e.gif)

デモのとおり、`command.vim`はコマンドを実行するためのバッファを用意していて、そのバッファでコマンド履歴を補完してくれます。
バッファを開くには次のコマンドを使用します。

```vim
:CommandBufferOpen
```

また、キーマップを用意しているので、次のように設定するとすばやくバッファを開けます。

```vim
nmap c: <Plug>(command_buffer_open)
```

コマンドを入力したら`Enter`で実行します。
シンプルですが、`:xxx`とほぼ同じ感覚でコマンドをターミナル上で実行できるのでストレスがないです。

## しくみ
`command.vim`は[denops.vim](https://github.com/vim-denops/denops.vim)を使用しています。
`denops.vim`については[こちら](https://zenn.dev/lambdalisue/articles/b4a31fba0b1ce95104c9)の記事で詳しく書かれているのでよかったら読んでみてください。

`vim script`だけでも実装できたんですが、`deno`のエコシステムを利用できる`denops.vim`が魅力的だったので使ってみました。
denoはテストを標準でサポートしているし、型システムもあるので、開発体験としてはとても良かったです。

ちょっと話逸れましたが、`command.vim`では自動補完以外の処理は基本`deno`側でやっています。
たとえば、バッファを開く時の処理は以下のようになっています。`denops.vim`は`vim.cmd()`でVimのExコマンド、`vim.call`でVimの関数を実行できるのでそれを利用しています。

```ts
// open buffer for execute shell command
async openExecuteBuffer() {
  // NOTE: using feedkeys because :startinsert doesn't work well in vim
  await vim.cmd(`botright 1new | call feedkeys("i")`);
  await vim.cmd(
    `setlocal buftype=nofile bufhidden=hide noswapfile nonumber nowrap ft=sh`,
  );
  await vim.cmd(
    `inoremap <silent> <buffer> <CR> <Esc>:call denops#notify("${vim.name}", "executeShellCommand", [&shell])<CR>`,
  );
  await vim.cmd(`nnoremap <silent> <buffer> <C-c> :bw!<CR>`);
  await vim.cmd(`inoremap <silent> <buffer> <C-c> <Esc>:bw!<CR>`);
  await vim.call(`command#complete#enable`);
},
```

`command#complete#enable`は自動補完を有効化し、シェル履歴を取得しています。
これはVim scriptで書くしかなかったので、`autoload`に定義しています。

```vim
fun! s:complete() abort
  call feedkeys("\<C-x>\<C-u>")
endfun

fun! command#complete#enable() abort
  let b:histories = denops#request("command", "getShellHistory", [&shell])
  if empty(b:histories)
    return
  endif

  setlocal completefunc=command#complete#shell_history

  let s:old_completeopt = &completeopt
  set completeopt+=noinsert,menuone,noselect

  augroup denops-command-complete
    autocmd!
    autocmd InsertCharPre <buffer> call s:complete()
    autocmd BufWipeout <buffer> call s:wipe_buffer()
  augroup END
endfun
```

### ハマったポイント
#### `completefunc`と`completeopt`
`command.vim`を実装する上で、自動補完にかなりハマってしまいました。
まず`completefunc`は`<C-x><C-u>`で補完するときの関数を指定するのですが、関数が呼ばれるしくみを理解するのに時間がかかりました。
何度も試しながら、少しずつ理解していったという感じです。

そして、`completeopt`は補完の細かい動作を変更するオプションですが、`completefunc`はバッファごとに設定できるのに対して`completeopt`はグローバルの設定になっています。
`completeopt`の設定をコマンドの実行完了と同時に、もとに戻さなければほかのプラグインが動作しなくなることがあります。
特に補完プラグインはこのオプションを使っていることが多いので注意が必要です。

#### 自動補完
入力するたびに、`<C-x><C-u>`で補完を実行すれば自動補完が完成ですが、ここでもかなりハマりました。
Vimには`autocmd`という、何かを操作するたびに発生するイベントをhookして処理できるしくみがあります。
入力に関しては主に以下のイベントがあります。

- TextChanged: ノーマルモードでテキストを変更した場合
- TextChangedI: 挿入モードでテキストを変更した場合
- TextChangedP: 挿入モードでテキストが変更されてポップアップウィンドウが表示されている場合

補完は挿入モードで行えばよいので、`TextChangedI`の`autocmd`を定義すればよいと思ったんですが、`completefunc`はテキストを変更してしまうため、`autocmd(TextChangedI)` -> `completefunc` -> テキストが変更される -> `autocmd(TextChangedI)` -> `completefunc` ... というふうに無限ループになってしまいました。

結局回避方法がよく分からなかったので、`InsertCharPre`イベントと`feedkeys`を使って自動補完を実装しました。
`InsertCharPre`は入力したテキストがバッファに書き込まれる前に動くので`autocmd`時点では入力したテキストを取得できないんですが、`feedkeys`は非同期で動作するので、
実際`<C-x><C-u>`が実行されるのはバッファにテキストが書き込まれたあとのタイミングになるようです。
そのため、この組み合わせであればまず補完は問題なく動くという感じです。

#### シェル履歴ファイル
`command.vim`が対応しているシェルは`bash`、`zsh`、`fish`の3種類ですが、これらの履歴フォーマットがすべて異なっています。
`bash`と`zsh`は次のようなシンプルなテキストになっていますが、改行がある場合のフォーマットが異なっています。

```
vim
echo
ls -la
...
```

`bash`の場合は`\n`として記録しますが、`zsh`は改行ごとで区切っています。
たとえば、次のコマンドを入力した場合、`bash`は`echo 1 2`として記録しますが、`zsh`は`echo 1\\`と`2`で別れます。

```
echo 1 \
2
```

更に`fish`の場合は次のフォーマットになっていて、時間と場所とコマンドの接頭辞がついています。

```
- cmd: echo 1 \\\n2
  when: 1611401590
  paths:
    - tmux
```

このような差分を吸収しつつ、改行の場合は1行に整形する必要があり、ちょっと面倒でした。

#### ターミナルのIF違い
`Vim`と`Neovim`のターミナルのIFが異なる部分もちょっとハマリポイントでした。
`Neovim`の場合シェルを経由してコマンドが実行されるので、`:term echo $EDITOR`を実行すると`vim`が出ますが、
`Vim`はシェルを経由しないため`$EDITOR`と出ます。

幸いなことに、Vimは去年あたりに`++shell`オプションが追加されたので`:term ++shell echo $EDITOR`と実行すればシェル経由してくれます。
それを使って`:terminal`の動作を統一しました。

```ts
if (await vim.call(`has`, "nvim")) {
  await vim.cmd(`new`);
  await vim.call(`termopen`, cmd);
} else {
  await vim.cmd(`terminal ++shell ${cmd}`);
  await vim.cmd(`nnoremap <buffer> <silent> <CR> :bw<CR>`);
}
```

## 最後に
サクッと作れるのかなと思ってやってみたら以外とハマリポイントが多く、結局時間が掛かってしまいました。
`vim-jp`でいろいろ質問しながら、なんとか形にできたのは良かったです。
特に[暗黒美無王のShougoさん](https://twitter.com/ShougoMatsu)と[vim-vsnip](https://github.com/hrsh7th/vim-vsnip)の作者の[hrsh7thさん](https://twitter.com/hrsh7th)が教えてくれていたので、この場を借りてあらめてお礼を申し上げます。ありがとうございました！


