---
title: dockerのイメージ一覧APIのfiltersパラメータについての調査
date: 2020-05-06
toc: true
tags: 
  - Docker
---

## 初めに
こんにちわ。
ゴリラです。

dockerのイメージAPIを`curl`で叩いた時にイメージをフィルターしたいが、詰まっていたのでそのやり方を備忘録として残しておきます。

## やりたいこと
`curl`でdockerのイメージ一覧を絞り込みたい。
`docker`コマンドだとこんな感じ。

```sh
$ docker images -f reference=mysql
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               5.7                 84164b03fa2e        2 months ago        456MB
```

## やりかた
筆者はfishをつかっているため、`urlenc`コマンドの実行結果を`()`で取得していますがbashなどの方は`$()`に置き換えてください。

```sh
curl -s --unix-socket /var/run/docker.sock (urlenc -e 'http://localhost/images/json?filters={"reference": ["mysql"]}') | jq
[
  {
    "Containers": -1,
    "Created": 1583343003,
    "Id": "sha256:84164b03fa2ecb33e8b4c1f2636ec3286e90786819faa4d1c103ae147824196a",
    "Labels": null,
    "ParentId": "",
    "RepoDigests": [
      "mysql@sha256:f4a5f5be3d94b4f4d3aef00fbc276ce7c08e62f2e1f28867d930deb73a314c58"
    ],
    "RepoTags": [
      "mysql:5.7"
    ],
    "SharedSize": -1,
    "Size": 455508074,
    "VirtualSize": 455508074
  }
]
```

## 解説
[公式のAPI定義](https://docs.docker.com/engine/api/v1.40/#operation/ImageList)には
`A JSON encoded value of the filters (a map[string][]string) to process on the images list. Available filters:`
と書いてあって、なるほどわからんって状態だったので、わかったことを解説します。

以下がポイントです

1. 検索条件は`filters`クエリパラメータで指定
2. 指定する時のJSONの形は`{"key": [xxx, yyy]}`という形でなければいけない(goの`go[string][]string}`をJSONにエンコードした形)

今回はイメージ名をつかって絞り込みたいので`filters`には`{"reference": ["イメージ名"]}`をエンコードした値を渡しています。

ちなみに、`urlenc -e 'http://localhost/images/json?filters={"reference": ["mysql"]}'`はエンコードしたURLを取得できます。

```sh
$ urlenc -e 'http://localhost/images/json?filters={"reference": ["mysql"]}'
http://localhost/images/json?filters=%7B%22reference%22%3A+%5B%22mysql%22%5D%7D
```

検証に使った`urlenc`はこちらに置いてあります

https://github.com/skanehira/go-enc
