---
title: Vimでマインドマップをプレビュー
date: 2020-10-07
toc: true
tags: 
  - Vim
---

## 始めに
思考整理のため、Vimでマインドマップ書けないかなってちょっと調べて試したらできたので、
やり方の備忘録として残しておきます。

## 必要なもの
- Docker
- [previm](https://github.com/previm/previm)

## やり方
`previm`を導入すればMarkdownをプレビューできますが、Markdownだけではなく`plantuml`もプレビューしてくれます。
`plantuml`をプレビューするにはサーバを立て、`previm`を設定する必要があります。

まず、Dockerを使ってサーバを建てます。

```shell
docker run -d -p 8888:8080 plantuml/plantuml-server:jetty
```

これで`plantuml`のサーバが起動するので、次に`previm`の設定をします。
ここで注意点ですがurlは`/`終わっている必要があります。

```vim
let g:previm_plantuml_imageprefix = 'http://localhost:8888/png/'
```

これだけど、プレビュー画面で画像が表示されます。

たとえば次のumlを書いた場合は、スクショのように出力されます。

> \`\`\`plantuml
> @startmindmap
> \* a
> ** b
> *** c
> ** d
> ** f
> @endmindmap
> \`\`\`

![](https://storage.googleapis.com/zenn-user-upload/facc40opwhn4bg52cmbb68biqntc)

## さいごに
`previm`と`plantuml` is便利。
