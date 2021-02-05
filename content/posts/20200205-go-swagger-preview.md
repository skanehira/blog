---
title: Swaggerをプレビューする簡単なCLI
date: 2021-02-05
toc: true
tags: 
  - Go
  - Swagger
---

## 初めに
仕事でAPIのドキュメントにswaggerを使っていますが、プレビューするために毎度VSCodeを立ち上げて使っていました。
しかし、このためだけにVSCodeを起動するのはさすがにオーバースペックでそれに耐えられず、簡単なCLIを作りました。

https://github.com/skanehira/swagger-preview

### 使い方
コマンドの引数にswagger.yamlを指定します。そうするとサーバが立ち上がってブラウザが開きます。

```sh
$ spr api.yaml
2021/02/02 21:51:46 start server: 9999
2021/02/02 21:51:46 watching swagger.yaml
```

デフォルトポートは9999ですが、環境変数`PORT`を使えば変更できます。

`PORT=8080 spr api.yaml`

あとはファイルを編集するたびに画面も更新されます。

### しくみ
簡潔に言うと次のようなしくみになっています。

- プレビューは[swagger-ui](https://github.com/swagger-api/swagger-ui)を使っている
- WebSocketを使って同期をしている
- 具体的にサーバ側でファイル変更を監視し、変更がある度にWebSocketでファイルの中身を送信

### 工夫点
#### ポート
swagger-uiはcdnを使えば、特に実ファイルをこちらで要しなくて良いので今回はそのまま次のようにimportして使っています。

```html
<link rel="stylesheet" type="text/css" href="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/3.41.1/swagger-ui.css" >
<script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/3.41.1/swagger-ui-bundle.js"> </script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/swagger-ui/3.41.1/swagger-ui-standalone-preset.js"> </script>
```

サーバでHTMLを返していますが、`index.html`ファイルを用意せずその中身をGoの文字リテラルに入れています。

```go
var indexHTML = `
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>mpr</title>
</head>
  ...
</html>
`

...

port := "9999"
if p := os.Getenv("PORT"); p != "" {
	port = p
}
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
	body := fmt.Sprintf(indexHTML, port)
	w.Write([]byte(body))
	return
})
```

なぜそうしているかというと…
WebSocketを使ってサーバと通信しているので、サーバのポートを自由に変更できるようにするなら画面側でもちゃんとポートも変更する必要があります。
`index.html`としてファイルを切り出すとポート変更を合わせられないので、文字リテラルとして定義してポートの部分を変えられうようにしています。

```go
fmt.Sprintf(indexHTML, port)
```

そして、文字リテラルにしたことでファイルを埋め込む必要がないのでシングルバイナリで動きます。

#### ファイル監視
ファイル監視は[fsnotify](https://github.com/fsnotify/fsnotify)というパッケージを使っています。既存のパッケージがあるしファイル監視なんてさくっと動くでしょと思っていました。 しかし、ここでやっかいなことが起たのです。
Vimでファイルを編集して保存すると変更ではなく、リネームはパーミッション変更といったイベントをキャッチしちゃっていました。これだとVimでファイルを保存するたびに複数回イベントをキャッチして、その度にファイルの中身を読み取るってWebSocketで送信してしまいます。

```
2021/02/05 21:58:04 event: "testdata/api.yaml": RENAME
2021/02/05 21:58:04 event: "testdata/api.yaml": CHMOD
2021/02/05 21:58:04 event: "testdata/api.yaml": REMOVE
```

fsnotifyはちゃんと起きたイベントをャッチしているだけなので問題ないですが、
Vimのファイル保存のしくみがファイルを直接書き換えないようになっているので実装で回避するしかない状態でした。
次に考えたのポーリングの案で、500ミリ秒でファイルの更新日を取得して前回と変更があった場合のみWebSocketで送信する実装です。

```go
fi, err := os.Stat(fileName)
if err != nil {
	log.Println(err)
	return
}
old := fi.ModTime()

go func() {
	for {
		<-time.After(500 * time.Millisecond)

		fi, err := os.Stat(fileName)
		if err != nil {
			log.Println(err)
			return
		}
		now := fi.ModTime()
		if !old.Equal(now) {
			old = now

			log.Println("update")
			b, err := ioutil.ReadFile(fileName)
			if err != nil {
				log.Println(err)
				return
			}
			msg <- b
		}
	}
}()
```

これでやりたいことは一応できたんですが、不要なI/Oが定期的に発生するのは納得行かず悶々としていました。
ちょうどTwitterでfsnotifyについてつぶやいていたところ、mattnさんからアドバイスをいただきました。

https://twitter.com/mattn_jp/status/1356280779353907200

そして、上記のアドバイスを元に以下のように実装したら期待通りに動きました。（感謝）
ポーリング実装の前に`sync.Once`は一応試したんですが、書き方が良くなかったようで、参考のコードを見て理解しました。（多分）

```go
go func() {
	var once sync.Once
	for {
		select {
		case event, ok := <-watcher.Events:
			if !ok {
				return
			}

			go func() {
				once.Do(func() {
					<-time.After(100 * time.Millisecond)
					if filepath.Base(event.Name) == filepath.Base(fileName) {
						fi, err := os.Stat(fileName)
						if err != nil {
							log.Println(err)
							return
						}
						now := fi.ModTime()
						if !old.Equal(now) {
							old = now

							log.Println("update")
							b, err := ioutil.ReadFile(fileName)
							if err != nil {
								log.Println(err)
								return
							}
							msg <- b
						}
					}
				})
				once = sync.Once{}
			}()
		case err, ok := <-watcher.Errors:
			if !ok {
				return
			}
			log.Println("error:", err)
		}
	}
}()
```

## まとめ
ファイル監視周りは紆余曲折がありましたが、`sync.Once`とdelayを入れるテクニックとVimのファイル保存のしくみを知れて良かったです。
正直こんなに苦戦るとは思わなかったんで、まだまだ精進せねばなという気持ちです。がんばります！

