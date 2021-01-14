---
title: VimでGitHub Actionsのステータスを確認
date: 2020-11-29
toc: true
tags: 
  - Vim
---

## 始めに
最近少しずつGitHub Actionsを使い始めています。とても便利でよくできているエコシステムだなって感心しました。
そこで、VimでGitHub Actionsのステータスを確認できたら便利かなと思って[ブラウザをやめてVimでGitHubを使う](https://zenn.dev/skanehira/articles/2020-09-19-vim-plugin-for-github)で作った[gh.vim](https://github.com/skanehira/gh.vim)に機能を追加しました。

`gh://:owner/:repo/actions` を開くとそのリポジトリにGtHub Actionsが動いていれば、ワークフローとジョブのステータスが表示されます。

![](https://i.gyazo.com/7550a3a547f6a1d94cecd68497f72cf0.gif)

### UI
ワークフローとジョブがツリー状になっています。
ルートはリポジトリ名、その下はワークフロー一覧となっています。
さらにその下はジョブとそのステップになっています。

```
- clipboard-image                                             <-- リポジトリ名
| - ✗ add github actions... @skanehira [add-github-actions]   <-- ワークフロー
| | - ✗ Test (ubuntu-latest)                                  <-- ジョブ
| | | | ✓ #1 Set up job                                       <-+
| | | | ✓ #2 Run actions/checkout@v2                            |
| | | | ✓ #3 Run actions/setup-go@v2                            | ジョブステップ
| | | | ✓ #4 install xclip                                      |
| | | | ✗ #5 Test                                               |
| | | | ✓ #10 Post Run actions/checkout@v2                      |
| | | | ✓ #11 Complete job                                    <-+
```

ワークフロー行では次のフォーマットになっていて、
ジョブが成功しているかどうかが一目で分かるようになっています。

```
実行結果 コミットメッセージ 作者 ブランチ名
------------------------------------------------------
✓ add github actions... @skanehira [add-github-actions]
✗ add github actions... @skanehira [add-github-actions]
```

GitHubではワークフローとジョブのレスポンスに`status`と`conclusion`というフィールドがあって、`status`はActionsの実行状態、`conclusion`が実行結果です。
↑の実行結果は`conclusion`をもとに判定しています。

![](https://i.gyazo.com/f38c64240c590e49efe127604e1fb0ed.png)

### 機能
現時点ではワークフローとジョブの表示とURLのコピー・ブラウザで開くの機能があります。
一応ジョブのre-runはやろうと思えばできますが、これ本当に必要かな？って思っているので未実装です。

gh.vim自体をできるだけシンプルに保ちたいので、
とりあえずあったら良いんじゃない？という感じで実装をしないようにしています。

選択したワークフロー/ジョブのURLをコピーする機能を実装したのは、たとえばURLを開発メンバーにここ落ちているよって共有するのに便利です。

2020/11/29追記
ジョブのログを確認できるようになりました。

![](https://i.gyazo.com/c5ed35bbb72fdc247265a6a5b7808600.gif)

### 実装
GitHub v3のAPIでリポジトリのGitHub Actionsの情報を取得しています。

https://developer.github.com/v3/actions/workflow-runs/#list-workflow-runs-for-a-repository

このAPIはGitHub Actionsの情報のみ取得するので、たとえばCIをAWSのcodebuildで行っているといったケースでは情報は取れないです。
また、このAPIではワークフロー一覧しか取得できないのでさらにその中にあるジョブとステップ情報は次のAPIから取得する必要があります。REST APIだとこういうのがツライですよね。..

https://developer.github.com/v3/actions/workflow-jobs/#list-jobs-for-a-workflow-run

本当はRESTではなくGraphQLを使いたかったんですが、v4 APIはGitHub Actions未対応なのでひとまずv3を使っています。
早くv4でGitHub Actionsの情報を取れるようになってほしいですね。

ツリー実装は[VimでGitHubのプロジェクトボードを使う](https://zenn.dev/skanehira/articles/2020-11-22-vim-github-project)のときに作ったものをそのまま使っているので、難なく実装できました。
汎用なツリーモジュールにしといて良かったです。
ライブラリにするのはちょっと難しいんですが、VimでツリーのUIを実現したい方は適当にコードからパクってもらって大丈夫です。

### 余談
![](https://i.gyazo.com/7c3b40585f4ca937f5f0f71e7f4fd03d.png)
↑は初期UIですが、文字情報しかなくてとても分かりづらかったので、GitHubのUIを参考にみたら今の形になりました。自分のデザインのセンスがなさすぎてツライ。..

![](https://i.gyazo.com/2cc3c5d9521fef08ddb349ea0d3796d6.png)
