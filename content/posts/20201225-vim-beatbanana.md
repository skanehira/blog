---
title: Vimでビートマニアする
date: 2020-12-25
toc: true
tags: 
  - Vim
---

以前、トイレ入ってたら突然Vimでビートマニアしたくなったので[skanehira/beatbanana.vim](https://github.com/skanehira/beatbanana.vim)というのを作っていました。
最近はGitHubのプラグイン[gh.vim](https://github.com/skanehira/gh.vim)を作っているので、こちらの開発は止まっていますが、少しその話を書いていこうと思います。

## どんな感じ？
作りかけですが、こんな感じです。

![](https://storage.googleapis.com/zenn-user-upload/qnf3fo9dghqm54qaq6dg33z42za9)

上からブロックがランダムで降ってくるので、`A~L`のキーを使ってジャストヒットをしていけるレベルにはなっています。
現時点ではブロックはランダム生成ですが、これからはサウンド機能と譜面編集機能を実装していく予定です。
この2つが完成すれば、おおよそ遊べるようにはなると思います。

## どんなしくみ？
ブロックの生成とはVimの`popup window`を使っています。こちらは`timer_start()`を使ってランダムに生成しています。

```vim
function! s:make_block(opt, timer) abort
  call timer_start(rand(srand()) % 1000, function('s:new_block', [a:opt]))
endfunction

（中略）
  for i in range(8)
    let press_key = s:bottom_bar_keys[i]
    let opt = {
          \ 'press_key': press_key,
          \ 'col_len': col_len,
          \ 'col_pos': col_pos,
          \ }
    call s:make_bottom_block(opt)
    exe printf('nnoremap <silent> <buffer> %s :call <SID>press_bottom_bar("%s")<CR>', press_key, press_key)
    call timer_start(1000, function('s:make_block', [opt]), {'repeat': -1})
    let col_pos += (col_len + 4)
  endfor
```

ちなみに、Vim scriptの関数を非同期（操作をブロックしない）で実行する関数がないですが、`timer_start(0, f)`というふうに使うと、操作はブロックされないので擬似的に非同期処理が可能です。
コレを使うことで、ブロックが降ってくる処理をしながらキー押下のイベントを拾って判定処理を行えます。

ブロックが降ってくるのは`popup window`のポジションを`timer_start`を使って繰り返し+1していきます。

```vim
function! s:move_block_down(winid, press_key, timer) abort
  let opt = popup_getpos(a:winid)
  if opt.line is# s:winheight
    call timer_stop(a:timer)
    call popup_close(a:winid)
    call s:delete_block_winid(a:press_key, a:winid)
    return
  endif

  let opt.line += 1
  call popup_move(a:winid, opt)
endfunction

（中略）
function! s:new_block(opt, timer) abort
  let text = s:make_block_text('0', a:opt.col_len)
  let winid = popup_create(text, {
        \ 'col': a:opt.col_pos,
        \ 'line': 1,
        \ 'minwidth': strlen(text),
        \ })

  call win_execute(winid, 'syntax match beatbanana_bar /0/')

  call timer_start(60, function("s:move_block_down", [winid, a:opt.press_key]), {
        \ 'repeat': -1,
        \ })
  call add(s:bar_winid_set[a:opt.press_key], winid)
endfunction
```

判定処理ですが、簡単に説明すると下部のバーを押したときのキーを、降りてくるブロックが持っているキーが一致しているかどうかを確認したうえで位置が一致しているかどうかを確認しています。
たとえば`J`のキーの列には`J`のキーだよという情報を持ったブロックが降ってくるので、そのブロックからキーを取り出して、今押しているキーと一致しているかどうかを確認しているという感じです。

細いロジックはコードを読んでてみてください。

```vim
function! s:collision_detection(key) abort
  let winids = s:bar_winid_set[a:key]
  if empty(winids)
    return 0
  endif

  let bar_winid = winids[0]
  let opt = popup_getpos(bar_winid)

  return opt.line is# s:winheight || opt.line is# s:winheight - 1
endfunction

function! s:press_bottom_bar(key) abort
  let winid = s:bottom_bar_winid_set[a:key]
  if s:collision_detection(a:key)
    call win_execute(winid, 'syntax match beatbanana_hit_good /1/')
  else
    call win_execute(winid, 'syntax match beatbanana_hit /1/')
  endif
  call timer_start(150, function('s:restore_bottom_bar_highlight', [winid]))
endfunction
```

## さいごに
まだまだ作っている最中で、まともに遊べるかと言われると怪しいですが、できたらちゃんとプラグイン化して公開したいと思います。
ご期待くださいー
