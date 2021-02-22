---
title: VimでGitHubを操作するプラグインgh.vimの紹介
date: 2020-12-03
toc: true
tags: 
  - Vim
---

## 始めに
最近GitHubのVimプラグイン[gh.vim](https://github.com/skanehira/gh.vim)というのを作っています。
issueやPR、プロジェクト、GitHub ActionsのステータスなどをVim/Neovim上で確認、操作できます。
よい感じにいろいろな機能が入ったので、あらためてちゃんとプラグインを紹介したいと思います。

![](https://i.gyazo.com/061eea10fef226b1dc104e24f0b6995c.png)

### 機能
現時点では次の機能が実装されています。

| カテゴリ      | 機能                                        |
|---------------|---------------------------------------------|
| issue         | 一覧/作成/更新                              |
| issue comment | 一覧/作成/更新                              |
| issue         | open/close                                  |
| pull request  | 一覧/パッチ（差分）                         |
| repository    | 一覧/README                                 |
| project       | 一覧/カード・カラム一覧                     |
| actions       | ワークフロー・ジョブステップ一覧/ジョブログ |
| bookmark      | 一覧/バッファを開く                         |

`gh.vim`では`gh://xxx`といった仮想バッファのみ提供していて、Exコマンドは用意していません。なので一般的なプラグインとちょっと異なる使い方をします。
たとえば`gh://golang/go/issues?state=all`というバッファ名を開くと、そのバッファにissue一覧が表示され、キーマップが設定されます。

仮想バッファというのは、実際ファイルを作成せず一時的なバッファにデータを表示したり、キーマップを設定したりする手法です。
仮想バッファのみにした理由はこれまでにない形のプラグインを作ってみたかったからです。
あとは実装しやすさがあります。詳細については[仕組み](#仕組み)の項で解説します。

現在`gh.vim`が提供している仮想バッファは次になります。大体機能ごとにバッファが別れています。

| buffer                                           | description                  |
|--------------------------------------------------|------------------------------|
| `gh://:owner/:repo/issues[?state=open&..]`       | issue一覧                    |
| `gh://:owner/:repo/issues/:number`               | issue編集                    |
| `gh://:owner/:repo/:branch/issues/new`           | 新規issue作成                |
| `gh://:owner/:repo/issues/comments[?page=1&..]`  | issueコメント一覧            |
| `gh://:owner/:repo/issues/comments/new`          | 新規issueコメント作成        |
| `gh://:owner/repos`                              | レポジトリ一覧               |
| `gh://user/repos`                                | 認証済みユーザリポジトリ一覧 |
| `gh://:owner/:repo/readme`                       | リポジトリのREADME           |
| `gh://:owner/:repo/pulls[?state=open&...]`       | PR一覧                       |
| `gh://:owner/:repo/pulls/:number/diff`           | PRの差分                     |
| `gh://:owner/:repo/projects`                     | project一覧                  |
| `gh://orgs/:org/projects`                        | Organazationのproject一覧    |
| `gh://projects/:id/columns`                      | projectのカラム一覧          |
| `gh://:owner/:repo/actions[?status=success&...]` | actionsステータス一覧        |
| `gh://bookmarks`                                 | ブックマーク一覧             |

次に、機能の詳細について紹介していきます。

#### Issue機能
issueの一覧や作成、編集、及びコメントの作成&編集などを行えます。

![](https://i.gyazo.com/19d67ef4bc621b5f9fe1a09609844284.gif)

**issue一覧**  
`gh://:owner/:repo/issues[?state=open&...]`でissueの一覧を表示できます。
`?`より後ろはクエリパラメータとして認識されるので、APIで使用できるクエリパラメータをそのまま使えます。
たとえば`?state=all`をつけるとclosedしたissueも一覧に表示されます。詳細は`gh.vim`のヘルプを参照下さい。
実行可能なアクションは次になります。

| キー    | 説明                  |
|---------|-----------------------|
| `<C-h>` | 前のページ            |
| `<C-l>` | 次のページ            |
| `<C-o>` | issueをブラウザで開く |
| `ghe`   | issueを編集           |
| `ghc`   | issueをclose          |
| `gho`   | issueをopen           |
| `ghm`   | issueのコメントを開く |
| `ghy`   | issueのURLをコピー    |

**issueの作成**  
`gh://:owner/:repo/:branch/issues/new`で新規issueを作成できます。
リポジトリにテンプレートがある場合、テンプレートを選択してissueを作成できます。

![](https://i.gyazo.com/6373b68c39829a5c3c4288e94bd1be20.gif)

**issueの編集**  
`gh://:owner/:repo/:branch/issues/:number`でissueの本文を編集&更新できます。
本文を編集後`:w`で更新されます。更新の際タイトルも変更するか聞かれるので合わせて修正したいときは新しいタイトルを入力します。

**issueのコメント一覧**  
`gh://:owner/:repo/issues/:number/comments[?page=1&...]`でissueのコメント一覧を表示できます。
実行可能なアクションは次になります。

| キー    | 説明                     |
|---------|--------------------------|
| `<C-h>` | 前のページ               |
| `<C-l>` | 次のページ               |
| `<C-o>` | コメントをブラウザで開く |
| `ghn`   | 新規コメント             |
| `ghy`   | コメントのURLをコピー    |

#### Pull Request機能
PRの一覧&差分を確認できます。

![](https://i.gyazo.com/ba4aa068fdaeebcf8168c8994ac73db4.gif)

**PR一覧**  
`gh://:owner/:repo/pulls[?page=1&...]`でPR一覧を表示できます。

実行可能なアクションは次になります。

| キー    | 説明             |
|---------|------------------|
| `<C-h>` | 前のページ       |
| `<C-l>` | 次のページ       |
| `<C-o>` | PRをブラウザで開く |
| `ghd`   | PRの差分         |
| `ghy`   | PRのURLをコピー  |

**PRの差分**
`gh://:owner/:repo/pull/:number/diff`でPRの差分を確認できます。
現時点では差分確認のみですが将来的にはレビューコメントもできるようにする予定です。

#### Repository機能
リポジトリ一覧&READMEを確認できます。

**リポジトリ一覧**
`gh://:owner/repos[?page=1&...]`でリポジトリ一覧を表示できます。
`:owner`が`user`の場合、認証されたユーザ（tokenを発行したユーザ）のプライベートやOrganazationのリポジトリも表示されます。

**リポジトリのREADME**
`gh://:owner/:repo/readme`でリポジトリのREADMEを確認できます。

![](https://i.gyazo.com/cd2d9bfebce0975fa8d75eaad1582f00.gif)

#### Project機能
リポジトリのproject一覧&カード一覧を確認できます。
プロジェクトとカラム、カードはツリーになっていて、折りたたみが可能です。
また、カードの移動も出来ます。

この機能の実装については[こちら](https://zenn.dev/skanehira/articles/2020-11-22-vim-github-project)を参照ください。

![](https://i.gyazo.com/e1dfc9d4880ca68f8ca84dfe025bd160.gif)

**プロジェクト一覧**
`gh://:owner/:repo/projects`で指定したリポジトリのproject一覧を表示します。
`:owner`が`orgs`の場合、`:repo`はOrganazationの名前を指定する必要があります。

実行可能なアクションは次になります。

| キー    | 説明                         |
|---------|------------------------------|
| `<CR>`  | カード一覧を開く             |
| `<C-o>` | プロジェクトをブラウザで開く |
| `ghy`   | プロジェクトのURLをコピー    |

**カード一覧**
`gh://projects/:id/columns`でprojectのカラムとカード一覧とカードの操作が出来ます。

| キー    | 説明                                              |
|---------|---------------------------------------------------|
| `<C-o>` | 選択したカードのURLを開く（現時点issueのみ対応）  |
| `gho`   | 選択したカードの詳細を開く（現時点issueのみ対応） |
| `ghm`   | 選択したカードを現在のカラムに移動                |
| `ghy`   | 選択したカードのURLをコピー                       |

#### GitHub Actions機能
`gh://:owner/:repo/actions[?status=success&...]`でリポジトリのGitHub Actionsのワークフローとジョブを確認できます。
実装の詳細については[こちら](https://zenn.dev/skanehira/articles/2020-11-29-vim-gh-add-actions)を参照下さい。

![](https://i.gyazo.com/bb920d9694c126b571e4f90fa7cc9a9a.gif)

| キー    | 説明                                        |
|---------|---------------------------------------------|
| `<C-o>` | 選択したワークフロー/ジョブをブラウザで開く |
| `ghy`   | 選択したワークフロー/ジョブのURLをコピー    |
| `gho`   | 選択したジョブのログを確認                  |

#### キーマップ
`gh.vim`では各種バッファで使用できるキーマップを用意しています。
詳細なキーマップはヘルプを参照いただければと思いますが、vimrcに次のような設定を書くことで独自のキーマップを設定できます。

```vim
function! s:gh_map_add() abort
  if !exists('g:loaded_gh')
    return
  endif
  call gh#map#add('gh-buffer-issue-list', 'nnoremap', 'x', ':bw!<CR>')
  call gh#map#add('gh-buffer-issue-list', 'map', 'h', '<Plug>(gh_issue_list_prev)')
  call gh#map#add('gh-buffer-issue-list', 'map', 'l', '<Plug>(gh_issue_list_next)')
endfunction

augroup gh-maps
  au!
  au VimEnter * call <SID>gh_map_add()
augroup END
```

### 仕組み
#### 通信
Vimの`job`機能を使って非同期に`curl`でGithubのv3 APIを叩いています。取得した結果をいい感じに表示させているだけなので、すごくムズカシイことをやっているわけではないです。
たとえばissue一覧のバッファを開くと、裏では次のコマンドが実行されます。

```sh
curl -H "Accept: application/vnd.github.v3+json" -H "Authorization: token xxxxxxxxxxxxxxxxxxx" "https://api.github.com/repos/golang/go/issues"
```

ただ、生の`job`機能を使うのはけっこう面倒なので、それをいい感じに使いやすくしてくれた[vital.vim](https://github.com/vim-jp/vital.vim)と`vital.vim`の追加モジュールである[vital-Whisky](https://github.com/lambdalisue/vital-Whisky)を使っています。
複雑なVimプラグインを作る上で欠かせないライブラリとなっているので、知らない方は一度使ってみると良いと思います。本当によく出来ています。

#### 仮想バッファ
仮想バッファの仕組みはVimの`autocmd`の`BufReadCmd`を使って実現しています。
`BufReadCmd`は新たなバッファが作られたときに何かしらの処理をしたいときに使えます。`gh.vim`では`gh://*`と一致したバッファ名が作られたときに`gh#gh#init()`という関数を呼び、バッファ名をもとに処理を分岐させています。

```vim:gh/plugin.vim
augroup gh
  au!
  au BufReadCmd gh://* call gh#gh#init()
augroup END
```

```vim:autoload/gh/gh.vim
function! gh#gh#init() abort
  setlocal nolist
  let bufname = bufname()
  if bufname is# 'gh://user/repos/new'
    call gh#repos#new()
  elseif bufname =~# '^gh:\/\/[^/]\+\/repos$' || bufname =~# '^gh:\/\/[^/]\+\/repos?\+'
    call gh#repos#list()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/readme$'
    call gh#repos#readme()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/issues$'
        \ || bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/issues?\+'
    call gh#issues#list()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/issues\/[0-9]\+$'
    call gh#issues#issue()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/[^/]\+\/issues\/new$'
    call gh#issues#new()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/issues\/\d\+\/comments$'
        \ || bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/issues\/\d\+\/comments?\+'
    call gh#issues#comments()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/issues\/\d\+\/comments\/new$'
    call gh#issues#comment_new()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/pulls$'
        \ || bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/pulls?\+'
    call gh#pulls#list()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/pulls\/\d\+\/diff$'
    call gh#pulls#diff()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/projects$'
        \ || bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/projects?\+'
    call gh#projects#list()
  elseif bufname =~# '^gh:\/\/projects\/[^/]\+\/columns$'
        \ || bufname =~# '^gh:\/\/projects\/[^/]\+\/columns?\+'
    call gh#projects#columns()
  elseif bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/actions$'
        \ || bufname =~# '^gh:\/\/[^/]\+\/[^/]\+\/actions?\+'
    call gh#actions#list()
  elseif bufname =~# '^gh:\/\/bookmarks$'
    call gh#bookmark#list()
  endif
endfunction
```

仮想バッファのメリットは簡単にいうとバッファを開く処理とバッファ初期化の処理を分断できる、ということです。
分断することで、たとえばissue編集バッファをissue一覧やprojectバッファから開くときは以下を実行するだけで済みます。

```vim
exe 'gh://:owner/:repo/issues/:number'
```

しかし分断されていない場合、バッファ作成したあとに初期化の処理関数を呼び出す必要があります。
そして関数名が変わったら修正範囲も広がってしまいます。

```vim
exe 'gh://:owner/:repo/issues/:number'
call gh#issue#edit()
```

このように、結合度を低く保てることにメリットがあるため`gh.vim`は積極的に仮想バッファを採用しています。

#### モジュール
`gh.vim`では大きく分けて次のモジュール群があります。

|                            |                                            |
|----------------------------|--------------------------------------------|
| `autoload/gh/http.vim`     | http通信を提供する                         |
| `autoload/gh/tree.vim`     | treeバッファを提供する                     |
| `autoload/gh/github/*.vim` | http通信をラップしてわかりやすくした       |
| `autoload/gh/*.vim`        | 各種バッファを提供している                 |
| `autoload/gh/gh.vim`       | 全バッファ共通のutil関数などを提供している |

ディレクトリ構成は次になっています。
他のプラグインと名前空間がかぶらないように、`gh`というディレクトリ配下にコードを置くようにしています。

```
 gh.vim
 |- autoload/
  |- gh/
   |- github/
    |  actions.vim
    |  issues.vim
    |  projects.vim
    |  pulls.vim
    |  repos.vim
   |  actions.vim
   |  bookmark.vim
   |  buffer.vim
   |  gh.vim
   |  http.vim
   |  issues.vim
   |  map.vim
   |  projects.vim
   |  pulls.vim
   |  repos.vim
   |  tree.vim
```

基本、各種バッファから`gh#http#xxx()`または`gh#github#xxx()`を呼び出して、結果を受け取って画面描画とキーマップの設定を行っています。
次はissue一覧バッファを開いたときに実行される関数です。中で`gh#github#issues#list()`を呼び出していてissue一覧を取得しています。
大体どの機能も、バッファの処理と通信の処理に分かれているのでファイル単位で分割しました。

```vim:autoload/gh/issue.vim
function! gh#issues#list() abort
  setlocal ft=gh-issues
  let m = matchlist(bufname(), 'gh://\(.*\)/\(.*\)/issues?*\(.*\)')

  call gh#gh#delete_buffer(s:, 'gh_issues_list_bufid')
  let s:gh_issues_list_bufid = bufnr()

  let param = gh#http#decode_param(m[3])
  if !has_key(param, 'page')
    let param['page'] = 1
  endif

  let s:issue_list = {
        \ 'repo': {
        \   'owner': m[1],
        \   'name': m[2],
        \ },
        \ 'param': param,
        \ }

  call gh#gh#init_buffer()
  call gh#gh#set_message_buf('loading')

  call gh#github#issues#list(s:issue_list.repo.owner, s:issue_list.repo.name, s:issue_list.param)
        \.then(function('s:issue_list'))
        \.then({-> gh#map#apply('gh-buffer-issue-list', s:gh_issues_list_bufid)})
        \.catch({err -> execute('call gh#gh#error_message(err.body)', '')})
        \.finally(function('gh#gh#global_buf_settings'))
endfunction
```

以上が`gh.vim`の大まかな仕組みです。
まだまだリファクタリングしないといけない箇所がたくさんありますが、より詳細な実装を知りたい方はコードを覗いてみて下さい。

### 課題
**一覧バッファの共通化**
現在、一覧バッファは大体どれも同じ仕組みですが、共通の部分を分けられていない状態です。
`tree.vim`のように、`list.vim`を作って一覧の処理の共通化をする必要があります。
共通化しておかないと、今後一覧画面が増えるたびに似通った処理が増えてメンテナンスが大変なので、いまのうちに手を付けておきたいと思っています。

**API通信量の削減**
v3のAPIを使っているため、通信量が結構多いです。
たとえばプロジェクトのカード一覧を取得するAPIがありますが、こちらにはカードの種類とURLの情報くらいしかなくて、種類がissueやPRの場合は別途APIを叩く必要があります。
カードがN百枚の場合、Vimが死ぬのでv4のGraphQLを使って通信量と回数を減らすのが直近一番やらないと行けない課題です。

**エラーハンドリング**
`vital.vim`の`promise`を使っていることによる原因かわからないんですが、処理でエラーが起きたときに握りつぶされることがあります。
実装時結構大変なのでこれも早い段階で解決しなければ行けないんですが、良い解決策が浮かばずという状態です。
知見がある方はアドバイスをお願いします。

### 最後
少し長くなりましたが、`gh.vim`の大体の機能について紹介しました。
このプラグインはまだまだ未完成なので、今後もコツコツと作っていきたいと思います。

プラグイン気になる方は、ぜひ一度使ってみて下さい。

