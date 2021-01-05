---
title: Vim上でGitHubのファイルツリーを表示する話
date: 2020-12-09
toc: true
tags: 
  - Vim
---

## 初めに
先日、[gh.vim](https://github.com/skanehira/gh.vim)にファイルツリー機能を実装しました。
Vim上でリポジトリのファイルツリーを見ることができる機能です。
その周りの話をしていきます。

## やり方
`:30vnew gh://:owner/:repo/:branch|:tree_sha`を実行すると、ファイルツリーを開けます。
`:repo`の後はブランチ名、もしくはコミットハッシュを指定できます。
また、`ghe`でカーソル上にあるファイルの中身を見ることができます。

![](https://i.imgur.com/HC7f6b5.gif)


## 実装
GitHubには[Trees](https://docs.github.com/en/free-pro-team@latest/rest/reference/git#get-a-tree)APIが用意されていて、それをたたくとファイルとディレクトリの情報を取得できます。

しかし、このレスポンスのではデータ構造が入れ子になっておらず、`path`にリポジトリのルートディレクトリからのパス情報などが配列になっています。
たとえば次のディレクトリ構成だった場合

```
.
├── README.md
├── doc
│   └── gh.txt
└── plugin
    └── gh.vim
```

これが次のようなレスポンスになります。

```json
{
  "sha": "xxx",
  "url": "xxx",
  "tree": [
    {
      "path": "README.md",
      ...
    },
    {
      "path": "doc",
      ...
    },
    {
      "path": "doc/gh.txt",
      ...
    },
    {
      "path": "plugin",
      ...
    }
    {
      "path": "plugin/gh.vim",
      ...
    }
  ]
}
```

ツリーを表現するにはフラットなデータ構造ではなく、ネストしたデータ構造のが都合良いので、これらの`path`を元に次のようなデータ構造に変換する必要があります。

```json
{
  "name": "gh.vim",
  "path": "gh.vim",
  "children": [
    {
      "name": "README.md",
      "path": "gh.vim/README.md"
    },
    {
      "name": "doc",
      "path": "gh.vim/doc",
      "children": [
        {
          "name": "gh.txt",
          "path": "gh.vim/doc/gh.txt"
        }
      ]
    },
    {
      "name": "plugin",
      "path": "gh.vim/plugin",
      "children": [
        {
          "name": "gh.vim",
          "path": "gh.vim/plugin/gh.vim"
        }
      ]
    }
  ]
}
```

`gh.vim`では`s:make_node`という関数で、1ファイルずつ、上記のデータ構造を構築しています。


```vim
function! s:make_node(tree, file) abort
  let paths = split(a:file.path, '/')
  let parent_path = join(paths[:-2], "/")
  let tree = a:tree
  let item = {
        \ 'name': a:file.type is# 'tree' ? paths[-1] .. '/' : paths[-1],
        \ 'path': a:file.path,
        \ 'info': a:file,
        \ 'markable': 1,
        \ }

  if a:file.type is# 'tree'
    let item['children'] = []
    let item['state'] = 'close'
  endif

  if has_key(b:tree_node_cache, parent_path)
    call add(b:tree_node_cache[parent_path], item)
    return
  endif

  if exists('tree.children')
    for node in tree.children
      call s:make_node(node, a:file)
    endfor
  endif

  if tree.path is# parent_path
    call add(a:tree.children, item)
    let b:tree_node_cache[parent_path] = a:tree.children
  endif
endfunction
```

基本的な処理の流れは次です。

1. `tree`（親要素）と、`file`（GitHubから取得した1ファイルの情報）を受け取る
2. `file`の`path`から親パスを取得
3. `tree`に追加するデータ`item`を作成
4. `file`がディレクトリ（`a:file.type is# 'tree'`の部分）の場合は`tree`に`children`を追加
5. すでに`tree`に`children`がある場合は、子要素の数だけ再帰処理
6. `tree`の`path`が`file`の親パスと一致した場合、`tree.children`に`file`を追加
7. これをAPIレスポンスの`tree`の配列分繰り返す

ただ、このロジックではファイルの数とパスの深さと比例して再帰の回数がえげつない回数になるので、ファイル数が3000個くらいあるプロジェクトだとかなりと処理に時間がかかります。

そこですでに作成したノードをキャッシュして子要素の親パスがキャッシュにあったら、
キャッシュしたデータに追加することでパフォーマンスを改善しました。

それが次の部分のコードになります

```vim
" キャッシュがあればキャッシュに要素を追加
if has_key(b:tree_node_cache, parent_path)
  call add(b:tree_node_cache[parent_path], item)
  return
endif

...

if tree.path is# parent_path
  call add(a:tree.children, item)
  " 作成済みの要素をキャッシュ
  let b:tree_node_cache[parent_path] = a:tree.children
endif
```

## 最後に
キャッシュ戦略でいくぶんパフォーマンスが改善されたとはいえ、
`golang/go`といった巨大なプロジェクトのファイルツリーを開くのは時間がかかってしまいます。
より良いロジックがないか、年末あたりに模索してみたいと思います。

`gh.vim`自体はファイルツリー以外にも、プロジェクトやGitHub Actionsをツリーでみたりできますので、
興味ある方は一度触ってみてください。Vim/Neovimともに動きます。
