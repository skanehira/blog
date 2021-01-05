---
title: ブラウザをやめてVimでGitHubを使う
date: 2020-09-19
toc: true
tags: 
  - Vim
---

## 始めに
最近仕事でGitHubを使っていますが、ターミナルとブラウザの行き来が面倒になってきたので
Vim上でGitHubを操作できるプラグイン[gh.vim](https://github.com/skanehira/gh.vim)[^1]を作りました。

ようやくまともに動くようになったので、
プラグインの紹介と実装について解説していきます。

[^1]: 現時点ではNeovimでは動作しない、対応したい気持ちはある

## デモ
こんな感じで使います。

![](https://i.imgur.com/VK6rebH.gif)

## 機能一覧
- issue一覧、作成、更新、close
- repository一覧、作成、削除（デフォルトOFF）
- Pull Request一覧、差分

## 使い方
詳細はREADMEやhelpを読んでいただければと思います。ここでは大まかな使い方を解説します。

`gh.vim`は通常のプラグインのようなコマンドを用意してません。
代わりに`gh://`から始まるバッファを開くことでGitHubに対するさまざまな操作を可能にします。

たとえば`:new gh://:owner/:repo/issues`でバッファを開くとissue一覧が表示されます。
`:owner`はユーザー名もしくはorg名で、`:repo`はリポジトリ名です。

現時点で使用可能なバッファは次になります。
ちょっとわかりづらいので補足します。
`gh://user/repos`は自身がアクセスprivateやorgnizationのリポジトリ一覧
`gh://:owner/repos`は他のユーザーの公開リポジトリ一覧
を確認できます。

| buffer                                     | description                            |
|--------------------------------------------|----------------------------------------|
| `gh://:owner/:repo/issues[?state=open&..]` | get issue list                         |
| `gh://:owner/:repo/issues/:number`         | edit issue                             |
| `gh://:owner/:repo/issues/new`             | create issue                           |
| `gh://:owner/repos`                        | get repository list                    |
| `gh://user/repos`                          | get authenticated user repository list |
| `gh://user/repos/new`                      | create repository                      |
| `gh://:owner/:repo/readme`                 | get repository readme                  |
| `gh://:owner/:repo/pulls[?state=open&...]` | get Pull Request list                  |
| `gh://:owner/:repo/pulls/:number/diff`     | show Pull Request list diff            |

さて、お気付きの方がいるかもしれないんですが、HTTPのURIと似た形になっています（`http` -> `gh`に変わったくらい）
普段エンジニアの我々からすれば馴染みがありますね。

このようなIFにしたのは
**これまでにない形のプラグインを作ってみたい（1つもコマンドを提供しない）**
というのが一番です。

現時点で特にお気に入りな機能は次です。

- issue作成と更新
- Pull Requestの差分表示

どちらも業務で実際結構使用するので、それがVimできるようになったのはコストが低下して良きです。

## しくみ
コマンドを提供しないで、どうやってバッファを開いたときに処理させているかについて解説します。

Vimには`BufReadCmd`というautocmdがあります。こちらはパターンにマッチしたバッファ（名）が読み込まれるときに発行して指定した関数を実行します。

この機能を利用して、`gh://*`から始まるバッファが読み込まれるときに任意の関数を処理させています。
実際の定義は次になっています。（一部抜粋）

```vim
au BufReadCmd gh://*/repos call gh#repos#list()
au BufReadCmd gh://*/repos\?* call gh#repos#list()
au BufReadCmd gh://*/*/issues call gh#issues#list()
au BufReadCmd gh://*/*/issues\?* call gh#issues#list()
au BufReadCmd gh://*/*/issues/[0-9]* call gh#issues#issue()
```

バッファ名はVimの正規表現が使えます。
そして関数内でバッファ名から必要な：ownerと：repo情報を取得して`curl`でGitHubのAPIを叩いています。

実は、これは以前[thincaさん](https://twitter.com/thinca)に教えていただいていました。
さらに[lambdaalisue](https://twitter.com/lambdalisue)さんの[gina.vim](https://github.com/lambdalisue/gina.vim)というプラグインも同じしくみを使っていました。

このしくみを使うメリット次になります。
やりたいこと次第では実装が結構楽になります。

- バッファを開いた時の処理と開く前の処理を分断できる
- 同じ処理をさせたければバッファを開くだけで良くなる

## 今後
当初VimでGitHubのPRをレビュー（コメント、approve、change requestなど）したいというのが目的でした。
個人的な目的はまだ達成していないので、今後はそこを達成したらさらにUI周りもリッチにして行きたいと感がています。

ひとまず[Rloadmap](https://github.com/skanehira/gh.vim#roadmap)を片付けてからってところですね。

## 雑感
しくみ自体はシンプルですが、Vimのバッファ操作周りが相変わらず面倒なのでいろいろとたいへんでした。
みなさんのフィードバックをお待ちしています。
