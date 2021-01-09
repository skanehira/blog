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

### [ソースコードの表現](https://golang.org/ref/spec#Source_code_representation)
ソースコードは[UTF-8](https://en.wikipedia.org/wiki/UTF-8)でエンコードされたテキストです。
テキストは最適化されていないため、単一アクセント付きコードポイントはアクセントと文字の組み合わせで構築された同じ文字と異なります。これらは2つのコードポイントとして扱われます。
簡易化のため、このドキュメントは装飾されていない用語文字を使用してソースコード内のコードポイントを参照します。

それぞれのコードポイントは異なります。たとえば、大文字小文字は異なる文字です。

実装の制約：他のツールとの互換性のため、コンパイラはソースコード内のNUL文字(U+0000)を許容しません。

実装の制約：他のツールとの互換性のため、コンパイラはソースコードの最初のコードポイントがUTF-8でエンコードされたバイト順マーク(U+FEFE)の場合は無視します。
他の場所では許容しません。

## [文字](https://golang.org/ref/spec#Characters)
次の用語は特定のUnicode文字クラスを表すために使用されています。

```
newline        = /* コードポイント U+000A */ .
unicode_char   = /* newlineを除いた任意のコードポイント */ .
unicode_letter = /* "文字"として分類されたコードポイント */ .
unicode_digit  = /* "数字、10進数"として分類されたコードポイント */ .
```

[The Unicode Standard 8.0](https://www.unicode.org/versions/Unicode8.0.0/)のセクション4.5の"General Category"で文字のカテゴリを定義しています。GoはLu、LI、Lt、LmまたはLoのいずれかの文字カテゴリをUnicode文字、数値カテゴリNdの文字をUnicode数字として扱います。

### [文と数字](https://golang.org/ref/spec#Letters_and_digits)

アンドースコア文字`_`(U+005F)は字としてみなされます。

```
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```
