---
title: 100種類以上のターミナルのカスタムテーマをサクッと導入
date: 2020-08-13
toc: true
tags: 
  - terminal
---

## 初めに
こんにちわ
最近メインPCがThinkpadになって、メインOSがUbuntuになったゴリラです

普段ターミナルで作業する方はカスタムテーマを使っていますか？
割とカスタムテーマをダウンロードして適用したりするの面倒と感じている方もいると思いますが、
今日はサクッとテーマを適用する方法を紹介します

## やり方
こちらを使います。MacとLinux対応しています。
https://github.com/Mayccoll/Gogh

ぼくはLinuxなので、次のコマンドを実行します。

```sh
bash -c  "$(wget -qO- https://git.io/vQgMr)"
```

100種類以上のテーマがあるので、[こちら](https://mayccoll.github.io/Gogh/)を見て使ってみたいものの番号を選択して下さい
もしくは`ALL`で全テーマを入れてもOKです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/7aa499a5-9e6f-c63d-60bf-1bd40b24bc2e.png)


入れたあと、デフォルト設定にしてターミナルを再起動すれば適用されます(gnome-terminalの場合)
あとはフォントを変えるなりして見てください。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/ddf36f95-5650-2792-c106-82ebbf9130b4.png)

## さいごに
100種類以上のテーマをサクッと導入できるのは便利ですね

