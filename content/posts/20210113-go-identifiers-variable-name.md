---
title: Goの識別子は変数名として使える
date: 2021-01-12
toc: true
draft: true
tags: 
  - Go
---

最近コツコツとGoの[仕様書](https://golang.org/ref/spec)を日本語に[翻訳](https://github.com/skanehira/blog/pull/4)しています。
その中で識別子とキーワードの違いについてわかったことがあるので、それについて説明していきます。

## 識別子とキーワード
Goには[識別子](https://golang.org/ref/spec#Identifiers)と[キーワード](https://golang.org/ref/spec#Keywords)がありますが、それぞれの説明は次のとおりです。

|          |                                                                         |
|----------|-------------------------------------------------------------------------|
|識別子    |`Identifiers name program entities such as variables and types`          |
|キーワード|`The following keywords are reserved and may not be used as identifiers.`|

識別子はプログラムの実体に名前を付けたもの、キーワードは予約済みで識別子として使えないものという定義になっています。
具体的に識別子とキーワードでそれぞれ定義済みのものがあります。

**識別子**
```
Types:
	bool byte complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr

Constants:
	true false iota

Zero value:
	nil

Functions:
	append cap close complex copy delete imag len
	make new panic print println real recover
```

**キーワード**
```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

## 識別子は変数名として使える
識別子は変数名としても使えます。たとえば次のコードはOKです。

```go
package main

func main() {
	nil := 1
	println(nil) // 1
}
```

対して、キーワードは予約語なので変数名として使えないので、次のコードではコンパイルエラーが起きます。

```go
package main

func main() {
	func := 1
	println(func)
}
```

識別子はプログラムの実体に名前を付けたものなので、変数名と同じ役割なため、事前に定義済みの識別子も変数名として使えるのではないかなと考えています。
仕様書には`The following identifiers are implicitly declared in the universe block:`と書いていて、この`universe block`はすべてのGoのソースコードを含んでいます。
厳密に説明するならば識別子は`univese block`のブロックスコープ内であれば書き換えは可能と理解しています。(自信がないので間違ってたら指摘下さい)

イメージとしては次のコードになるかと思います。

```go
package main

func main() {
	// ここがuniverse blockとする
	int := 1
	{
		int := "a"
		println(int) // a
	}
	println(int) // 1
}
```

個人的にこの仕様はパーサーだと、スコープ内ですでに`nil`が定義されているかどうかを考慮しないと行けないので、ちょいと面倒そうだなと思いました。

```go
package main

func main() {
	var nil int
	nil, i := nil, 1
	if nil == nil {
		println(i) // 1
	}
}
```

## 最後に
英語とGoの勉強を同時にやるため翻訳を始めたんですが、こういった発見があって楽しいです。
これからも、コツコツと翻訳しつつ理解したことを記事にしていきたいと思います。
