---
title: The Go Programming Language Specification日本語訳
date: 2021-01-08
toc: true
tags: 
  - Go
---

## 初めに
英語とGoの勉強のため [The Go Programming Language Specification](https://golang.org/ref/spec) を翻訳しています。
素人翻訳なので意図や意味が間違っている可能性があります。その場合は[こちら](https://github.com/skanehira/blog)にissueもしくはPRをいただけるとうれしいです。

現時点は、[2020/10/08](https://github.com/golang/go/blob/2b9b2720b89d493dbf8725d0ae6664ac7835b3af/doc/go_spec.html)版を翻訳しています。

## [導入](https://golang.org/ref/spec#Introduction)
これはプログラミング言語Goのリファレンスマニュアルです。詳細と他のドキュメントに関しては[golang.org](https://golang.org)を参照してください。

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

Productionは次の演算子と単語から構築された式です。演算子の優先度は次のリストの昇順になります。

```
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

小文字プロダクション名は字句トークンを識別するために使います。非終端文字はキャメルケースで表されます。
字句トークンは`""`また``` `` ```で囲われます。

`a...b`は`a`から`b`までの文字を代わりに表します。省略記号`...`は指定されていないさまざまな列挙やコードスニペットを示します。
`...`はGo言語のトークンではありません。

### [ソースコードの表現](https://golang.org/ref/spec#Source_code_representation)
ソースコードは[UTF-8](https://en.wikipedia.org/wiki/UTF-8)でエンコードされたテキストです。
テキストは最適化されていないため、単一アクセント付きコードポイントはアクセントと文字の組み合わせで構築された同じ文字と異なります。これらは2つのコードポイントとして扱われます。
簡易化のため、このドキュメントは装飾されていない用語文字を使用してソースコード内のコードポイントを参照します。

それぞれのコードポイントは異なります。たとえば、大文字小文字は異なる文字です。

実装の制約：ほかのツールとの互換性のため、コンパイラはソースコード内のNUL文字（U+0000）を許容しません。

実装の制約：ほかのツールとの互換性のため、コンパイラはソースコードの最初のコードポイントがUTF-8でエンコードされたバイト順マーク（U+FEFE）の場合は無視します。
ほかの場所では許容しません。

## [文字](https://golang.org/ref/spec#Characters)
次の用語は特定のUnicode文字種類を表すために使用されています。

```
newline        = /* コードポイント U+000A */ .
unicode_char   = /* newlineを除いた任意のコードポイント */ .
unicode_letter = /* "文字"として分類されたコードポイント */ .
unicode_digit  = /* "数字、10進数"として分類されたコードポイント */ .
```

[The Unicode Standard 8.0](https://www.unicode.org/versions/Unicode8.0.0/)のセクション4.5の"General Category"で文字のカテゴリを定義しています。GoはLu、LI、Lt、LmまたはLoのいずれかの文字カテゴリをUnicode文字、数値カテゴリNdの文字をUnicode数字として扱います。

### [文と数字](https://golang.org/ref/spec#Letters_and_digits)
アンダースコア文字`_`（U+005F）は字としてみなされます。

```
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```

## [字句要素](https://golang.org/ref/spec#Lexical_elements)

### [コメント](https://golang.org/ref/spec#Comments)
コメントはプログラムのドキュメントとして機能します。次の2つの書式があります。

1. 行コメントは`//`で始まり、行の終わりで終わります。
2. 範囲コメントは`/*`で始まり、後続の最初の`*/`で終わります。

コメントは[ルーン文字]()や[文字列リテラル]()内、またはコメント内から開始できません。
改行を含まない範囲コメントはスペースのように機能します。そのほかのコメントは改行のように機能します。

### [トークン](https://golang.org/ref/spec#Tokens)
トークンはGo言語を形作っています。識別子、キーワード、演算子や区切り文字、リテラルの4種類があります。
スペース（U+0020）、タブ（U+0009）、キャリッジリターン（U+000D）や改行（U+000A）からなるホワイトスペースはそれがないとひとつのトークンに結合されてしまうようなトークンを分割している場合を除いて無視されます。

また、改行とファイル終端に[セミコロン](#セミコロンhttpsgolangorgrefspecsemicolons)が挿入される場合があります。
トークンを入力している間、次のトークンは有効なトークンを形成する最も長い文字列です。[^1]

[^1]: 貪欲法

### [セミコロン](https://golang.org/ref/spec#Semicolons)
形式文法では多くのProductionの終端記号として`;`を使っています。Goのプログラムでは、次の2つの規則を使用して多くのセミコロンを省略できます。

1. 入力をトークンに分割するとき、行の最後のトークンが以下のいずれかであった場合は、トークン列中そのトークンの直後にセミコロンが自動的に挿入されます。
  - [識別子]()
  - [整数]()、[浮動小数点]()、[虚数]()、[ルーン]()、または[文字列]()
  - [キーワード]()の`break`、`continue`、`fallthrough`また`return`のどれか
  - [演算子と区切り文字]()
2. 複雑な式を1行にする場合、`)`または`}`の前のセミコロンを省略できます。

慣用的な使い方を反映するため、このドキュメントのコード例はこれらの規則を使って省略しています。

### [識別子](https://golang.org/ref/spec#Identifiers)
識別子は変数や型といったプログラムの実態に名前を付けたものです。識別子は1つ以上の字または数字からなります。一文字目は字でなければいけません。

```
Identifier = letter { letter | unicode_digit }
```

```
a
_x9
ThisVariableIsExported
αβ
```

いくつかの識別子は[事前に定義されています]()。

### [キーワード](https://golang.org/ref/spec#Keywords)
次のキーワードは予約済みで、識別子として使用できません。

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

### [演算子と区切り文字](https://golang.org/ref/spec#Operators_and_punctuation)
次の文字列は[演算子]()（[代入演算子]()を含む）と区切り文字を表します。

```
+    &     +=    &=     &&    ==    !=    (    )
+    -    |     -=    |=     ||    <     <=    [    ]
+    *    ^     *=    ^=     <-    >     >=    {    }
+    /    <<    /=    <<=    ++    =     :=    ,    ;
+    %    >>    %=    >>=    --    !     ...   .    :
+         &^          &^=
```

### [整数リテラル](https://golang.org/ref/spec#Integer_literals)
整数リテラルは整数定数を表す数字列です。に10進数以外の任意の基数を付けられます。
`0b`または`0B`の場合は2進数、`0`または`0o`、`00`の場合は8進数、`0x`または`0X`の場合は6進数になります。単一の`0`は10進数のゼロとみなされます。
16進数のリテラルでは、`a`から`f`と`A`から`F`は`10`から`15`を表しています。

可読性のため、アンダースコア文字（`_`）は接頭辞の後または連続した数字の間に現れる場合があります。
このようなアンダースコアはリテラル値を変更しません。

```
int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .

decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
binary_digits  = binary_digit { [ "_" ] binary_digit } .
octal_digits   = octal_digit { [ "_" ] octal_digit } .
hex_digits     = hex_digit { [ "_" ] hex_digit } .
```

```
42
4_2
0600
0_600
0o600
0O600       // 2つ目は大文字の'O'
0xBadFace
0xBad_Face
0x_67_7a_2f_cc_40_c6
170141183460469231731687303715884105727
170_141183_460469_231731_687303_715884_105727

_42         // 数字リテラルではなく、識別子
42_         // 無効: _ は連続した数字を区切る必要がある
4__2        // 無効: 1度に1つの _
0_xBadFace  // 無効: _ は連続した数字を区切る必要がある
```
