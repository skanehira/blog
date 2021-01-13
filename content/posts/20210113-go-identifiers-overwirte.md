---
title: Goの定義済みの識別子は上書きできる
date: 2021-01-12
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

この識別子は事前に定義されているものがいくつかあり、それが次になっています。

**[定義済みの識別子](https://golang.org/ref/spec#Predeclared_identifiers)**

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

## 定義済みの識別子は上書きできる
実は、事前に定義されている識別子を上書きできます。たとえば次のコードはOKです。

```go
package main

func main() {
	true := false
	println(true) // false
}
```

仕様書には`The following identifiers are implicitly declared in the universe block:`とあり、この`universe block`は一番外側のスコープのことを指します。
詳細は[こちら](https://motemen.github.io/go-for-go-book/#%E3%83%A6%E3%83%8B%E3%83%90%E3%83%BC%E3%82%B9%E3%82%B9%E3%82%B3%E3%83%BC%E3%83%97)を参照下さい。
つまり、外側のスコープで定義された識別子(`true`や`false`)を今のスコープで上書きできる、ということです。次の例と同じです。

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
英語とGoの勉強を同時にやるため翻訳を始めたんですが、こういった発見があって楽しいです。
これからも、コツコツと翻訳しつつ理解したことを記事にしていきたいと思います。
