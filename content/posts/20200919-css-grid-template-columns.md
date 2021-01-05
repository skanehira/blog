---
title: CSSのgrid-template-columnsを使ってレスポンシブに要素を並べる
date: 2020-09-19
toc: true
tags: 
  - CSS
---

## 始めに
レスポンシブに要素を並べたいという要望があって、久しぶりにCSSを使ったのでその備忘録です。
これが適切なやり方かどうはわからないので、変なところがあればアドバイスいただけるとうれしいです。

## 要件
- 複数の要素を横に並べたい
- 画面サイズが小さくなったら自動で要素をwrap（次の行に移動）してほしい

イメージは次になります。コードは[デモ](https://jsfiddle.net/skanehira/ea6Ldk40/102/)です。
![](https://storage.googleapis.com/zenn-user-upload/7hokbaq2dbwk8apm4j1bi7ekhofr)

## やり方
[grid-template-columns](http://www.htmq.com/css/grid-template-columns.shtml)と[repeat](https://developer.mozilla.org/ja/docs/Web/CSS/repeat)を使うことで実現できます。

```html
<div class="wrapper">
  <div class="item">item1</div>
  <div class="item">item2</div>
  <div class="item">item3</div>
  <div class="item">item4</div>
  <div class="item">item5</div>
  <div class="item">item6</div>
  <div class="item">item7</div>
  <div class="item">item8</div>
  <div class="item">item9</div>
  <div class="item">item10</div>
</div>

<style>
  .wrapper {
    display: grid;
    grid-template-columns: repeat(auto-fill, 60px);
    grid-gap: 5px;
  }

  .item {
    text-align: center;
    background-color: gray;
    color: white;
  }
</style>
```

詳細を解説していきます。
前提として`display: grid`は必須になります。

### `grid-template-columns`
要素のサイズと要素数を指定できます。
たとえば`grid-template-columns: 100px 100px`なら次の動きになります。

- **1要素のwidthが100px**
- **横2つまで表示する**
- **2つ以上の場合はwrap**

つまり横に個数を増やしたければサイズ指定を増やせば良いだけです。

ただ、これだけだと画面のサイズ内に目いっぱい表示したい場合は困ります。
なぜなら画面のサイズによって表示できる個数が変わるので、それを動的に計算する必要があるからです。

そこで、`repeat`の出番です。

### `repeat`
要素の繰り返しを定義できる関数です。2つの引数を受け取ることができ、
**1つ目： 繰り返す回数**
**2つ目： 1要素のサイズ**
になっています。

たとえば繰り返す回数を2、サイズを100pxにする場合は次のように書きます。

```css
grid-template-columns: repeat(2, 100px);
```

これは次と同じ定義になります。

```css
grid-template-columns: 100px 100px;
```

ただ、このままだとやはり回数指定になってしまうのでそこで[auto-fill](https://www.webprofessional.jp/difference-between-auto-fill-and-auto-fit/)です。
`auto-fill`は**親要素のサイズ内で要素を何個配置できるかを計算してくれる**ので、こちらで個数を指定する必要がなくなります。

`auto-fill`のほかに`auto-fit`というのがありますが、違いは[こちら](https://www.webprofessional.jp/difference-between-auto-fill-and-auto-fit/)の記事のgifを見れば一発でわかります。

### `grid-gap`
要素間の間隔を指定します。たとえば10pxずつ要素を離したいなら`grid-gap: 10px;`になります。
これに関してはシンプルでわかりやすいですね。

### 最後に
CSS、奥深すぎてやばいですね。魔界です。
じっくり勉強していきます。

### 参考情報
- https://parashuto.com/rriver/development/responsive-layout-with-css-grid-and-flexbox
- https://www.webprofessional.jp/difference-between-auto-fill-and-auto-fit/
