---
title: あまり知られていないGoのスライス/配列の初期化の調査
date: 2020-12-02
toc: true
tags: 
  - Go
---

## 初めに
先日、[こちらの動画](https://www.youtube.com/watch?v=elGjixFQLYE&t=1054s&ab_channel=mercaridevjp)で見たことがない配列の初期化をしていたので、それについてわかったことを解説してきます。

## 配列の初期化
動画のほうでは次のような配列宣言をしていました。

```go
package main

var array = [...]int{0, 4:1, 2, 1:1}

func main() {
	println(len(array))
}
```

一般的な配列やスライスの宣言は以下のパターンがあると思います

```go
var slice1 = []int{1, 2, 3, 4}
var slice2 = make([]int, 1, 4)
var array1 = [10]int{1, 2, 3, 4}
var array2 = [...]int{1, 2, 3, 4}
```

これらはよく見かけるパターンですが、`{0, 4:1, 2, 1:1}`というのは見たことがなく、どう読めばよいのかわかりませんでした。
これについて[Hajime Hoshi (星一)](https://twitter.com/hajimehoshi)さんに教えていただきました。結論からいうと以下の処理と同じことをしています。

```go
var array = [6]int{}
array[1] = 1
array[4] = 1
array[5] = 2
fmt.Println(array)
```

## 解説
配列、スライスではインデックスを指定して値をセットできます。
つまり`{0, 4:1, 2, 1:1}`は、`0`番目に`0`、`4:1`は`4`番目に値`1`、`5`番目に`2`、`1`番目に`1`をセットしています。
これは構造体の初期化時にフィールドを指定できるのと同じです。

```go
gorilla := Animal{Name: "gorilla", Age: 28}
```

## 最後に
普通にGoを書いていたらこのような書き方は使わないんですが、構造体の初期化と同じ理屈と考えるとすごく腑に落ちます。
Go、奥が深い。。

