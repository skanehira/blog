---
title: Goの宣言済みの識別子は再宣言できる
date: 2021-01-13
toc: true
tags: 
  - Go
---

最近コツコツとGoの[仕様書](https://golang.org/ref/spec)を日本語に[翻訳](https://github.com/skanehira/blog/pull/4)しています。
その中で識別子について分かったことがあるので、それについてまとめた記事になります。

## 識別子とは
Goでは[識別子](https://golang.org/ref/spec#Identifiers)という概念があります。

```
Identifiers name program entities such as variables and types.
An identifier is a sequence of one or more letters and digits.
The first character in an identifier must be a letter.
```

これを意訳すると`識別子（文字や数字）を使って、変数や型といったプログラムの構成要素に名前を付けることができます。`になります。
たとえば`name := "gorilla"`の`name`は識別子、といった感じです。

この識別子は事前に宣言されているものがいくつかあり、それが次になっています。

**[事前宣言された識別子](https://golang.org/ref/spec#Predeclared_identifiers)**

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

この宣言済みの識別子のスコープは`universe block`と呼ばれる一番外側のスコープで宣言されています。
詳細は[こちら](https://motemen.github.io/go-for-go-book/#%E3%83%A6%E3%83%8B%E3%83%90%E3%83%BC%E3%82%B9%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97)を参照下さい。

## 宣言済みの識別子は再宣言できる
Goでは宣言済の識別子を上書きできます。たとえば次のコードはOKです。

```go
package main

func main() {
	true := false
	println(true) // false
}
```

これは[宣言とスコープ](https://golang.org/ref/spec#Declarations_and_scope)で次のように明言されています。

```
An identifier declared in a block may be redeclared in an inner block.
```

つまり、外側のスコープ(`universe block`)で宣言された識別子(`true`や`false`)を内側のスコープ(上記の例でいうと`main`関数内)で再定義されたということです。
次の例も同様に`{}`内で`int`を再宣言しています。

```go
package main

func main() {
	int := 1
	{
		int := "a"
		println(int) // a
	}
	println(int) // 1
}
```

## 最後に
Goでは型などは予約語ではなく識別子、そして識別子は再宣言可能で`true := false`が有効になるのは意外でした。
なぜ識別子を予約語にしていないのかわからないんですが、仕様書を読んでいく内にこの疑問も解決できるかもしれないので引き続き頑張って翻訳しつつ理解していきます。
