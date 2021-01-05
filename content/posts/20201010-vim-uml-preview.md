---
title: VimでUMLをプレビュー
date: 2020-10-10
toc: true
tags: 
  - Vim
---

## 始めに
先日 [Vimでマインドマップをプレビュー](https://zenn.dev/skanehira/articles/2020-10-07-vim-write-plantuml) という記事を書きました。
その前後に、フォロワーさんからUMLをプレビューできる [vim-slumlord](https://github.com/scrooloose/vim-slumlord) ってのを教えてもらいました。

リンク先のGIFを見たらとてもよさそうでしたが、2年以上更新されていなくてかつJavaの実行環境が必要でした。
UMLを書くためにわざわざJavaの環境を作りたくない、dockerで実行のイメージを用意するのもアリですが、
プレビューなのでこまめに保存することを考えるとオーバーヘッドが気になってしまいます。

### じゃあ作るか
自分のニーズに合わないと感じたので、じゃあ自作するかと思って [preview-uml.vim](https://github.com/skanehira/preview-uml.vim) を作りました。

![](https://i.imgur.com/9Uyr1xC.gif)

### しくみ
図の生成は`plantuml server`に任せています。なのでserverをローカルもしくは[外部のサーバー](http://www.plantuml.com/plantuml)を使うかの2択があります。基本プロダクトでは情報を外部サーバに、送らないようにすべきなのでローカルでサーバを用意すると良いでしょう。
dockerがあれば次のコマンドで簡単にローカルサーバをサクッと用意できます。

```shell
docker run -d -p 8888:8080 plantuml/plantuml-server:jetty
```

あとは、Vimから`curl`でサーバにテキストを送信すれば、図を取得できます。
ただ、図を取得するのがちょっと面倒で、次の処理をしないと行けないです。

1. `curl`でテキストを送信
2. レスポンスヘッダのLocationからURLを抜き出して、アスキーアートを取得できるURLに変換
3. `curl`で2のURLからデータを取得

これをシェルで書くと次のとおりです。

```shell
$ curl -s -i -X POST http://localhost:8888 -d "text=@startuml
Alice -> Bob
@enduml
"
HTTP/1.1 302 Found
Location: http://localhost:8888/uml/Syp9J4vLqBLJSCfF0W00
Content-Length: 0
Server: Jetty(9.4.31.v20200723)

$ curl -s http://localhost:8888/txt/Syp9J4vLqBLJSCfF0W00
     ┌─────┐          ┌───┐
     │Alice│          │Bob│
     └──┬──┘          └─┬─┘
        │               │
        │──────────────>│
     ┌──┴──┐          ┌─┴─┐
     │Alice│          │Bob│
     └─────┘          └───┘
```

一発でアスキーアートのURLを取得できる方法があればよいんですが、調べた限りではなさそうです。
どなたか、ご存じでしたら教えてください。

あとは上記の処理をVim scriptから実行して、レスポンスをバッファに書き込むだけなので実装はとてもシンプルです。

```vim
function! s:update() abort
  let body = getline(1, '$')
  if empty(body)
    return
  endif

  let cmd = printf('curl -X POST -s -i %s/form -d "text=%s"', s:url,
        \ body->join("\n")->escape('"'))
  let resp = systemlist(cmd)

  let location = resp->filter('v:val =~ "Location"')
  if empty(location)
    call <SID>echo_err('invalid response')
    call win_execute(s:preview.winid, 'bw!')
    return
  endif

  let url = location[0][10:]->trim()->substitute('uml', 'txt', 'g')
  let resp = systemlist(printf('curl -s %s', url))

  call win_execute(s:preview.winid, '%d_')
  call setbufline(s:preview.bufid, 1, resp)
endfunction
```

### 余談
これまでVSCodeでUMLを書いてきましたが、これでVSCodeとはさよならになりそうかなぁって気がしてます。
でも、多分そんなことないと思われます笑。

ちなみに、ちょうど[Go Conference'20 in Autumn SENDAI](https://sendai.gocon.jp)のセッション視聴中に思いついて作り始めたんで途中でセッション諦めました。
もちろん、あとでちゃんと動画を見返しました（えらい）
