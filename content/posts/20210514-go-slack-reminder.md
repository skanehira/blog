---
title: "Slackのreminder使いづらすぎたのでCLIを作った"
date: 2021-05-13
toc: true
tags:
  - Go
	- Slack
---

## 初めに
みなさんSlackの`/reminder`を使っていますか？
個人的に以下の理由でめちゃくちゃ使いづらいなと感じています。

- 書式覚えられない（特に繰り返し）
- `/reminder`と入力したときに出てくるUIは繰り返し設定に対応していない

しかし、`reminder`はMTGの忘れ防止にとても役に立つので、やはりなくせないものです。
なのでコマンドをわかりやすいUIで出力するCLIを作りました。

こんな感じです。

![](https://i.gyazo.com/27916ee21b8b0b686c187013001d2922.gif)

## 使い方
導入方法は[リポジトリ](https://github.com/skanehira/slack-reminder)を参照してください。
`slack-reminder`を実行すると選択肢を選びながら入力しつつ、最終的にコマンドが出力されるので、それをSlackに貼るだけです。

一回のみの場合は次のように日時と宛先、メッセージを入力します。

```sh
MacbookPro13% slack-reminder
? Kind of remind onetime
? Date(YYYY-MM-DD) 2020-05-13
? Hour(HH:MM) 10:00
? @someone or #channel or me @gorilla
? Message hello, I'm here.
/remind @gorilla "hello, I'm here." at 10:00 on 2020-05-13
```

繰り返しの場合は、繰り返しの種類（毎日、毎週、隔週、毎月、毎年）を選択してから、それぞれの種類に応じて必要な日時を入力します。

```sh
MacbookPro13% slack-reminder
? Kind of remind repetition
? What kind of repetition every week
? What day of week choice
? Choice days Tuesday, Thursday, Sunday
? Hour(HH:MM) 10:00
? @someone or #channel or me me
? Message some remind
/remind me "some remind" at 10:00 on every Tuesday ,Thursday ,Sunday
```

## 最後に
`/reminder`は分かりづらいし覚えられないんで、今までは[こちら](https://slack-remind-creator.netlify.app/)を使っていましたが、ターミナルでサクッとコマンドを出力するCLIがあったらもっと便利だよなぁと思って作りました。

とここまで書いけど、そもそもCLIじゃなくてSlack Appを作ったほうが良かったのでは？って気がしてきました。
でもまぁ、これで`/reminder`地獄から開放されるので、使いづらいと思っている方はぜひ試してみてください。
