---
title: "GitHubのTUIツールを作った"
date: 2021-02-22
toc: true
tags: 
  - Go
---

以前に [VimでGitHubを操作するプラグインgh.vimの紹介](https://skanehira.github.io/blog/posts/20201203-introduce-gh-vim/) という記事を書きました。
こちらの記事で紹介した`gh.vim`はVim上でGitHubの機能を使えるようにしたVimプラグインです。仕事でがっつりGitHubを使うならとても便利なプラグインと思っています。

しかしVimを使っていない人もたくさんいるので、そういった人向けのツールあったほうが良いかなと思い、GitHubのTUIツール [github-tui](https://github.com/skanehira/github-tui) を作りました。
CLIと比べ、TUIはインタラクティブに、そして直感的に操作できるというメリットがあるので便利なのではないかなと思っています。
余談ですが過去にDockerのTUIツール [docui](https://github.com/skanehira/docui) を作ったりもしました。

[github-tui](https://github.com/skanehira/github-tui) は未完成ではありますが、最低限形になったし、需要があるかどうかを見たかったのでひとまず一般公開しました。
本記事は`github-tui`について、紹介と展望について書いていきますので興味ある方はぜひ最後まで読んでみてください。

### 画面構成
![](https://i.gyazo.com/ca2dc9bed29bb5b446d58d9aeae94e8c.png)

`github-tui`では大きく分けて以下の3種類のパネルがあります。

1. 入力
2. 一覧
3. プレビュー

詳細は次のとおりです。

| パネル          | 説明                                        |
|-----------------|---------------------------------------------|
| Filters         | 画面上部のクエリを入力する部分              |
| issues          | issue一覧                                   |
| assignees       | 選択中のissueにアサインされているユーザー覧 |
| labels          | 選択中のissueに付けたラベル一覧             |
| milestones      | 選択中のissueに紐付いているマイルストーン   |
| projects        | 選択中のissueに紐付いているプロジェクト一覧 |
| issue preview   | 選択中のissue本文のプレビュー               |
| comments        | 選択中のissueのコメント一覧                 |
| comment preview | 選択中のコメント本文のプレビュー            |
| search          | 画面下部の検索キーワードを入力する部分      |

### 機能
現時点ではissue関連の機能が実装されています。未実装の機能は[こちら](https://github.com/skanehira/github-tui#still-under-development)を参照ください。

- Issue
  - list
  - create
  - close
  - open
  - open browser
  - preview
  - edit
- Issue comment
  - list
  - preview
  - delete
  - edit
	- add

#### 基本的な使い方
リポジトリを指定して起動するか、指定しない場合はカレントディレクトリのリポジトリが自動で指定されます。

```bash
# current repository
$ ght

# specified repository
$ ght owner/repo
```

起動後にissueの検索が行われ、一覧が表示されます。パネルを移動するには`Ctrl-N`/`Ctrl-P`を使います。

#### 検索
`github-tui`では[search](https://docs.github.com/en/github/searching-for-information-on-github/searching-issues-and-pull-requests#search-only-issues-or-pull-requests)のクエリを使ってissueを検索できます。現時点ではissueのみ検索可能ですが、将来的にはPRなども検索できるようにする予定です。

どんなクエリが使えるかは、searchのドキュメントを参照していただければと思いますが、たとえばissueのタイトルと本文に"ゴリラ"という文字を含んだissueを検索したい場合は次のクエリを入力してEnterを押すと検索できます。

```
is:issue state:open ゴリラ in:body,title
```

![](https://i.gyazo.com/9dbfc882ce9305b7a1652c5c71c00737.gif)

demoのように、基本的に選択しているissueが変わる度、issue本文とコメントのプレビューが変わるようになっています。
検索時にissueの本文、それに紐づくラベルやコメントなどを一斉に取得するため、タイムラグなしにプレビューできます。

デフォルトでは検索で取得するissueは30件までとなっていますが、`f`を使って更に検索結果を取得できます。
GitHubの仕様では1度に100件まで取得可能ですが、前述したとおりissueに関連している情報も色々取得しているため30件に制限しています。
将来的にデフォルトのlimitをconfigで変更できるようにしたいと考えています。

#### キーワード検索
一覧とプレビューパネルでは`/`を押すとキーワードを検索できます。一覧の場合は表示されている項目、プレビューパネルの場合は表示されている中身をそれぞれ検索します。

![](https://i.gyazo.com/91631e116d3349fb043cae76f5627df1.gif)

プレビューパネルの場合はキーワードがハイライトされ、`n`で次の、`N`で前のキーワードにジャンプできます。Vimmerならこの操作に馴染みやすいかと思います。

#### プレビューパネルの拡大
`github-tui`はターミナルのサイズに合わせてパネルのサイズを動的に変更します。各パネルのサイズの割合は決まっています。
そのため、画面が小さいとプレビュー画面も小さくなりissueやコメントの本文が読みづらくなります。
その際は`o`でパネルを拡大でき、再度`o`を押すと元に戻ります。

![](https://i.gyazo.com/6e25f0e970f0ab6ef1dbbe17b55d7ebf.gif)

#### issueのOpenとClose
issue一覧パネルでは`Ctrl-J`/`Ctrl+K`を使って複数のissueにチェックを付けて、それらをまとめてOpenまたはCloseできます。
チェックしていない場合は、現在選択しているissueに作用します。issueをOpenするには`o`、Closeするには`c`を使います。

![](https://i.gyazo.com/b30c8107ae75fda1c7da42c8170c871e.gif)

また、選択したissueを`Ctrl-O`を使ってブラウザで開くこともできます。

#### issue作成
issue一覧パネルで`n`を使ってissueを新規作成できます。作成の際にフォームが表示されるので、`Tab`で各項目を切り替えながら必要な部分を埋めていく使い方になります。

![](https://i.gyazo.com/fb665369057c5f096517a24e606e7884.png)

`Repo`は`Filters`にある`repo:owner/repo`から取得するので、作成する際にはそのクエリが必須です。
`Assignees`と`Labels`、`Projects`は入力補完が可能ですので、任意のキーワードを入力すると補完候補が表示されます。ただし、複数の候補を追加したい場合は`,`で区切る必要があります。これはUIによる制限です。（ツライ）

`MileStone`と`Temaplte`はオプションになっていて、`↑`と`↓`で選択でき、`Esc`で選択をキャンセルできます。
`Edit Body`ボタンは`$EDITOR`で設定されているエディタを使ってissue本文を編集できますが、`$EDITOR`が特に設定されていない場合は`Vim`がデフォルトで使用されます。
`Create`ボタンで`Enter`を押下するとissueが作成されます。成功するとフォームが閉じられ1秒経ってからissue一覧が更新されます。1秒待つ理由としては作成後すぐにissue一覧を取得してもすぐに反映されないためです。

![](https://i.gyazo.com/c9b132ac7f44d0366dbd5d932529816d.gif)

注意点として、以下のデータがない場合は項目自体が表示されない様になっています。

- `Labels`
- `Projects`
- `MileStone`
- `Temaplte`

### 今後の展望
基本的にブラウザでやっていたことをターミナルでもできるようにするという方針です。たとえば、issueの作成、PRの作成、PRのレビューなどです。
ただし、使用しているライブラリでは実現が難しい可能性もあるので、ある程度妥協も必要かなと考えています。
直近はまずisssueとPR周りの機能を充実させていきます。やることは多いですがコツコツ作っていきます。

### 最後に
一通り、現時点の実装済みの機能について紹介しましたが、どうでしたか？便利そうだなと思ったらぜひ試してみてください。要望などはissueで受け付けますので、こういう機能欲しい！というのがあればぜひリクエストください。検討します。

そして、もし開発に興味あるなら[Twitter](https://twitter.com/gorilla0513)で連絡をいただけるとうれしいです。
