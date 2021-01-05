---
title: Vimで快適に記事を書くため環境
date: 2020-11-16
toc: true
tags: 
  - Vim
---

## 初めに
こんにちは、Neovimを使い初めたゴリラです。

普段zenn.devに載せる記事をVimで書いています。
しかしVimで記事を書くとどうしても画像アップロードとリンク挿入の手間がかかってしまったり、
誤字脱字があったり、文章表現がバラバラになったりという問題があります。

こういった問題を長らく放置してきましたが、重い腰を上げて対策しました。
本記事は、Vim/Neovimで快適に記事を書くためにどんなことをしたのかについて解説していきます。

### 環境
筆者の環境は次になっています。`Node.js`と`npm`と`Go`は必要です。

| Environments | Version                         |
|--------------|---------------------------------|
| OS           | Ubuntu 20.04 TLS                |
| Vim          | 8.2.1992                        |
| Neovim       | v0.5.0-834-g7c4f34966           |
| Node.js      | v14.15.0                        |
| npm          | 6.14.8                          |
| Go           | go version go1.15.3 linux/amd64 |

### 要件
まず、記事を書くにあたり次の要件があります。

1. 誤字脱字を可能な限りなくす
2. 文章の表現を統一
3. クリップボードや画像ファイルを選択してアップロードして、そのリンクを本文に挿入

これらはmustと考えています。特に3は記事を書いていると画像やgifとを貼ったりするのでそれを毎度ブラウザを使ってアップロードするのは非常に手間です。
エディタで記事を書きながら、シームレスに画像をアップロードできたらこれだけで少なくとも5秒くらいは短縮できるでしょう。

この要件をもとに、対策をしていきます。

### 誤字脱字、表記ゆれ対策
1と2に関してはいわゆる校正作業ですが、こちらは[textlint](https://github.com/textlint/textlint)を使います。

- zenn.devは記事をリポジトリと連携できる
- Node.jsを使っている

ということで、zenn.devと相性も良いです。（インストールして設定するだけ）

合わせて、校正ルールのプリセットもインストールします。

```sh
# textlint のインストール
$ npm install textlint

# ルールセット
$ npm install textlint-rule-prh textlint-rule-preset-jtf-style textlint-rule-preset-ja-technical-writing
```

使用しているルールは次の通りです。詳細はリンク先を参照ください。ひとまずこれらがあれば問題ないでしょう。

| ルール                                                                                                                | 説明                                                                          |
|-----------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------|
| [textlint-rule-preset-jtf-style](https://github.com/textlint-ja/textlint-rule-preset-JTF-style)                       | JTF日本語標準スタイルガイド                                                   |
| [textlint-rule-preset-ja-technical-writing](https://github.com/textlint-ja/textlint-rule-preset-ja-technical-writing) | 技術文章向けのルールプリセット                                                |
| [textlint-rule-prh](https://github.com/textlint-rule/textlint-rule-prh)                                               | [prh](https://github.com/prh/prh)という校正ツールの`textlint`ルールプリセット |

インストール完了後`.textlintrc`に次の設定をします。（`preset-ja-technical-writing`でいくつかルールを無効にしています）

```json:.textlintrc
{
  "filters": {},
  "rules": {
    "preset-ja-technical-writing": {
      "ja-no-weak-phrase": false,
      "ja-no-mixed-period": false,
      "no-exclamation-question-mark": false
    },
    "preset-jtf-style": true,
    "prh": {
      "rulePaths": [
        "node_modules/prh/prh-rules/media/WEB+DB_PRESS.yml",
        "node_modules/prh/prh-rules/media/techbooster.yml"
      ]
    }
  }
}
```

設定完了したら、試しにエラーが起きる文章に`textlint`をかけてみると、次のように修正すべき箇所が出てきます。

![](https://i.gyazo.com/697868b83322fb9333f83c4cfa7a15c0.png)

これらをまとめて修正するには`textlint --fix [file]`すればよいです。
Vim上で実行する場合は`:terminal npx textlint --fix [file]`もしくは`:!npx textlint --fix [file]`すればよいです。
これでひとまず校正すべき箇所を検出してくれる環境を用意できました。次はVim/Neovimの設定を行っていきます。

Vim/Neovimでは[LSP(Language Server Protocol)](https://microsoft.github.io/language-server-protocol/)を使って、動的に`textlint`で校正箇所を検出します。
次のプラグインをインストールことで、Vim/NeovimでもLSPを使えます。

- [vim-lsp](https://github.com/prabirshrestha/vim-lsp)
- [vim-lsp-settings](https://github.com/mattn/vim-lsp-settings)（LSPのサーバの導入を補助してくれる）

プラグインをインストール後、`:LspInstallServer efm-langserver`で`efm-langserver`をインストールします。`efm-langserver`は`textlint`をLSPサーバとして動かすのに必要です。

プラグインの用意ができたら、プラグインの設定をします。`vimrc`または`init.vim`に次の設定を書きます。

```vim
" lsp settings {{{
let g:lsp_signs_error = {'text': 'ｳﾎ'}
let g:lsp_signs_warning = {'text': '🍌'}
if !has('nvim')
  let g:lsp_diagnostics_float_cursor = 1
endif
let g:lsp_log_file = ''

let g:lsp_settings = {
      \ 'efm-langserver': {
      \   'disabled': 0,
      \   'allowlist': ['markdown'],
      \  }
      \ }

function! s:on_lsp_buffer_enabled() abort
  setlocal completeopt=menu
  setlocal omnifunc=lsp#complete
endfunction

augroup lsp_install
  au!
  au User lsp_buffer_enabled call s:on_lsp_buffer_enabled()
augroup END
" }}}
```

最後に`textlint`をLSPとして動かすために`efm-langserver`の設定を行います。
`~/.config/efm-langserver/config.yaml`に次の設定をします。

```yaml:config.yaml
version: 2
tools:
  markdown-textlint: &markdown-textlint
    lint-command: 'npx --no-install textlint -f unix --stdin --stdin-filename ${INPUT}'
    lint-ignore-exit-code: true
    lint-stdin: true
    lint-formats:
      - '%f:%l:%c: %m [%trror/%r]'
    root-markers:
      - .textlintrc
languages:
  markdown:
    - <<: *markdown-textlint
```

これで設定は終わりです。Vim/NeovimでMarkdownのファイルを開くと次のようにtextlintの指摘が表示されます。
編集、保存するたびに`efm-langserver`が`textlint`を実行しその結果を返してくれるので記事を書きながら校正できます。

![](https://i.gyazo.com/20662c65af0d9c33e5fac45cfec42491.png)

### 画像アップロード
zenn.devやQiitaはそれぞれ、Webエディタ上で`Ctrl(Cmd)+v`を使うとクリップボードから画像をアップロード&リンクを挿入してくれます。
これはすばらしい機能と体験ですが、残念ながらVim/Neovimはそのような機能を持っていません。

そこで、Vim/Neovimでクリップボードから画像をアップロード&リンクを挿入してくれるプラグイン[gyazo.vim](https://github.com/skanehira/gyazo.vim)を作りました。
画像のアップロード先は[Gyazo](https://gyazo.com)になるので別途アカウント&トークンを取得する必要はありますが、次のようにスクショを撮ってそのままアップロードできるので要件を満たせます。

![](https://i.gyazo.com/2adcdcc57f144bd524bc29bd1affbe75.gif)

## 終わりに
これらの設定でだいぶ執筆環境が整って捗りました。今回はMarkdownだけ動くようにしていますが、ほかの拡張子でも動かせます。
少し手間がかかりますが、Vimmerで同じように不便を感じている方はぜひ試してみてください。
