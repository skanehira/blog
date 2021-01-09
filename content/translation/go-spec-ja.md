---
title: The Go Programming Language Specification日本語訳
date: 2021-01-08
toc: true
tags: 
  - Go
---

## 初めに
英語とGoの勉強のため [The Go Programming Language Specification](https://golang.org/ref/spec) を翻訳しています。
素人翻訳なので意図や意味が間違っている可能性があります。その場合は[こちら](https://github.com/skanehira/blog)にissueもしくはPRをいただけると嬉しいです。

現時点は、[2020/10/08](https://github.com/golang/go/blob/2b9b2720b89d493dbf8725d0ae6664ac7835b3af/doc/go_spec.html)版を翻訳しています。

## [導入](https://golang.org/ref/spec#Introduction)
これはプログラミング言語Goのリファレンスマニュアルです。詳細および他のドキュメントに関しては[golang.org](https://golang.org)を参照して下さい。

Goはシステムプログラミング向けに設計された汎用的な言語です。強い型付けとガベージコレクション、そして並行プログラミングをサポートしています。プログラムはパッケージから構成されていて、それらのプロパティは依存関係の効率的な管理を可能にします。

文法がコンパクトで解析しやすく、統合開発環境のような自動ツールによる解析を容易にします。

## [表記](https://golang.org/ref/spec#Notation)
Goの構文は拡張バッカスナウア記法を使用しています。

```
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

プロダクションは次の演算子と単語から構築された式です。演算子の優先度は次のリストの昇順になります。

```
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

小文字プロダクション名は字句トークンを識別するために使います。非終端文字はキャメルケースで表されます。
字句トークンは`""`また``` `` ```で囲われます。

`a...b`は`a`から`b`までの文字を代わりに表します。省略記号`...`は指定されていない様々な列挙やコードスニペットを示します。
`...`はGo言語のトークンではありません。


