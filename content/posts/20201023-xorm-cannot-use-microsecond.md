---
title: xormでマイクロ秒の日付保存でハマった話
date: 2020-10-23
toc: true
tags: 
  - Go
  - xorm
---

## 始め
Goの[xorm](https://gitea.com/xorm/xorm)という`ORM`と`MySQL`を仕事で使っていますが、
マイクロ秒の日付を保存できないという謎の動きをしていたので、調査と対処をしたというお話です。

## 結論
2020/10/23時点`xorm`は`Microsoft SQL Server`以外のRDBにマイクロ秒の日付を保存できません。

## 旅の始まり
仕事で次のカラムに`time.Time`のデータを保存しようとしたら、マイクロ秒のところが0のままなっていました。

```
mysql> desc t;
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| date  | datetime(6) | NO   |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+

mysql> select * from t;
+----------------------------+
| date                       |
+----------------------------+
| 2020-10-23 23:00:47.000000 |
+----------------------------+
```

ためしに普通にSQLを発行してみたところ、問題なくマイクロ秒が挿入されていたのでこれはコードもしくは`xorm`が原因かなと疑い始めました。

```
mysql> insert into t (date) values(now(6));
Query OK, 1 row affected (0.00 sec)

mysql> select * from t;
+----------------------------+
| date                       |
+----------------------------+
| 2020-10-23 23:00:47.000000 |
| 2020-10-23 23:02:21.793967 |
+----------------------------+
```

### コードが悪いのか、それとも`xorm`が悪いのか
DBは問題ないなら自分が悪いのかなと疑い初めて、
次の小さなサンプルコードを動かしてみたところ、マイクロ秒が入りませんでした。

```go
engine, err := xorm.NewEngine("mysql", "gorilla:gorilla@/gorilla?charset=utf8")
if err != nil {
	log.Fatal(err)
}

t := struct {
	Date time.Time
}{
	Date: time.Now(),
}

if _, err := engine.Table("t").Insert(&t); err != nil {
	log.Println(err)
}
```

### `xorm`へ
こうなると`xorm`だなと思って`xorm`コードを読み始めました。
`xorm`で`time.Time`を変換している箇所があるはずですので、`Insert`メソッドからたどれば良さそうと思って潜ったら次の処理を見つけました。

```go:internal/statements/values.go
case reflect.Struct:
	if fieldType.ConvertibleTo(schemas.TimeType) {
		t := fieldValue.Convert(schemas.TimeType).Interface().(time.Time)
		tf := dialects.FormatColumnTime(statement.dialect, statement.defaultTimeZone, col, t)
		return tf, nil
```

`dialects.FormatColumnTime`ってメソッドがいかにもそれっぽいなと思って中を覗いたらビンゴでした。
`FormatTime`関数が日付のフォーマットを生成していて、しかも`schemas.DateTime`の場合は秒までのフォーマットになっていました。

```go:dialects/time.go
// FormatTime format time as column type
func FormatTime(dialect Dialect, sqlTypeName string, t time.Time) (v interface{}) {
	switch sqlTypeName {
	case schemas.Time:
		s := t.Format("2006-01-02 15:04:05") // time.RFC3339
		v = s[11:19]
	case schemas.Date:
		v = t.Format("2006-01-02")
	case schemas.DateTime, schemas.TimeStamp, schemas.Varchar: // !DarthPestilane! format time when sqlTypeName is schemas.Varchar.
		v = t.Format("2006-01-02 15:04:05")
	case schemas.TimeStampz:
		if dialect.URI().DBType == schemas.MSSQL {
			v = t.Format("2006-01-02T15:04:05.9999999Z07:00")
		} else {
			v = t.Format(time.RFC3339Nano)
		}
	case schemas.BigInt, schemas.Int:
		v = t.Unix()
	default:
		v = t
	}
	return
}
```

そんなことあるのかとびっくりしましたが、現実は残酷なものです。受け入れて対処しなければいけません。

### パッチを書く
このままだとお仕事困ってしまうので、パッチを書くしかないかなと悩んでいました。
ちょうど同じ現象に遭遇して、すでに[PR](https://gitea.com/xorm/xorm/pulls/1655)を出していた[KoRoN](twitter.com/kaoriya)さんのコードがあったので、
少し拝借してテストも書いて[PR](https://gitea.com/xorm/xorm/pulls/1815)を投げました。

対処方法はシンプルで、PostgreSQLとMySQLで`TimeStamp`と`DateTime`型のときは日付フォーマットをマイクロ秒までにするだけです。

```diff
-	case schemas.DateTime, schemas.TimeStamp, schemas.Varchar: // !DarthPestilane! format time when sqlTypeName is schemas.Varchar.
+	case schemas.Varchar: // !DarthPestilane! format time when sqlTypeName is schemas.Varchar.
 		v = t.Format("2006-01-02 15:04:05")
+	case schemas.TimeStamp, schemas.DateTime:
+		dbType := dialect.URI().DBType
+		if dbType == schemas.POSTGRES || dbType == schemas.MYSQL {
+			v = t.Format("2006-01-02T15:04:05.999999")
+		} else {
+			v = t.Format("2006-01-02 15:04:05")
+		}
 	case schemas.TimeStampz:
 		if dialect.URI().DBType == schemas.MSSQL {
 			v = t.Format("2006-01-02T15:04:05.9999999Z07:00")
```

### 動かしてみる
まだ本体へマージされていないのですが、上記修正で無事動きました。めでたい。
![](https://storage.googleapis.com/zenn-user-upload/xdv2ykyzwztdcjctbupfttt18fk1)

### 最後に
やれることはやったので、後は神頼みするしかないです。
はやくマージしてほしい。

