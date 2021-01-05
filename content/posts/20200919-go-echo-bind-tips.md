---
title: EchoのBindでクエリとパスパラメータを構造体に入れる方法
date: 2020-09-19
toc: true
tags: 
  - Go
---

## 始めに
普段GoでAPIを作るときはよく[Echo](https://echo.labstack.com/)を使っていますが、
コードリーディングしたらクエリとパスパラメータを構造体にサクッと入れる機能を知ったので紹介します。

### 前提知識
`Echo`では次のやり方で値を取得できます。

- `e.Param(key)`でパスパラメータ
- `e.QueryParam(key)`でクエリパラメータ

- クエリパラメータ
```go
// curl localhost/users?name=gorilla
func(c echo.Context) error {
	name := c.QueryParam("name")
	// gorilla
	fmt.Println(name)
	...
}
```

- パスパラメータ
```go
// curl localhost/users/gorilla
e := echo.New()
e.GET("/users/:name", func(c echo.Context) error {
	name := c.Param("name")
	// gorilla
	fmt.Println(name)
})
```

### 例
- API定義は`PUT /users/:id`
- リクエストボディは`{"name": "xxx"}`

という定義になっているとします。

その場合、次の処理をします。

1. `c.Param()`でパスパラメータからidを取得
2. `c.Bind()`でリクエストボディからは更新データを取得
3. idとnameを構造体に入れて、O/RマッパのUpdateを使って更新

```go
type User struct {
	ID   int    `json:"id"`
	Name string `json:"name"`
}

e.PUT("/users/:id", func(c echo.Context) error {
	id := c.Param("id")
	var u User
	if err := c.Bind(&u);err != nil {
		// error handling
	}
	u.ID = id
	db.Update(&u)
	...
})
```

### やり方
ただ、これだとidとnameのデータ取得方法がそれぞれ異なるのでチョット面倒です。

そこで、構造体に`query` or `param`をつけて`c.Bind`すれば`Echo`がよしなに
クエリパラメータorパスパラメータを構造体に入れてくれます。

次の例は`param`をつけてパスパラメータのidを取得しています。

```go
type User struct {
	ID   int    `json:"id" param:"id"`
	Name string `json:"name"`
}

e.PUT("/users/:id", func(c echo.Context) error {
	var u User
	if err := c.Bind(&u);err != nil {
		// error handling
	}
	db.Update(&u)
	...
})
```

### 雑感
一応この機能は[公式ガイド](https://echo.labstack.com/guide/request)に書いてありましたが、コードリーディングして見つけました…
ちゃんと公式ドキュメントを読もうねって気持ちになりました。
