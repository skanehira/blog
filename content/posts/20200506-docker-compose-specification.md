---
title: docker-composeの仕様であるCompose Specificationについて
date: 2020-05-06
tags: 
  - docker-compose
  - Docker
---

## 初めに
こんにちは。
ゴリラです。

先日、Docker社が`docker-compose`が提供していた機能の仕様を[Compose Specification](https://compose-spec.io/)としてオープンな仕様にしていく発表がありました。

`docker-compose`自体はオープンソースとして[GitHub.ocm/docker/compose](https://github.com/docker/compose)に公開されてはいますが、
具体的な仕様については定まっていなかったので、今後はそれらの機能の仕様を標準化をしていくそうです。

現時点決まっている仕様は[compose-spec/compose-spec](https://github.com/compose-spec/compose-spec)で公開されています。
仕様に沿ってリファレンス実装である[compose-ref](https://github.com/compose-spec/compose-ref)も公開されています。

本記事は`compose-ref`を試してわかったことを共有するのが目的です。
気になる方はぜひ最後まで読んでみてください。

## `compose-ref`のインストール
`compose-ref`はGoで書かれていますので、Goをあらかじめ用意して次の手順でインストールします。

```sh
$ git clone https://github.com/compose-spec/compose-ref.git
$ cd compose-ref
$ go install
$ compose-ref

 .---.
(     )
 )@ @(
//|||\\

NAME:
   compose-ref - Reference Compose Specification implementation

USAGE:
   compose-ref [global options] command [command options] [arguments...]

COMMANDS:
   up       Create and start application services
   down     Stop services created by `up`
   help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --file FILE, -f FILE          Load Compose file FILE (default: "compose.yaml")
   --project-name NAME, -n NAME  Set project name NAME (default: Compose file's folder name)
   --help, -h                    show help (default: false)
```

## `Compose file`について
`docker-compose`ではデフォルトで`docker-compose.yaml`もしくは`docker-compose.yml`を読み込むようになっていますが、
本仕様では`compose.yaml`もしくは`compose.yml`をデフォルトで読み込みます。

ただ、後方互換を維持するために`docker-compose.yaml`もサポートするとのことです。
これについては[こちら](https://github.com/compose-spec/compose-spec/blob/master/spec.md#compose-file)に記述されています。

## 動作確認
現時点では`up`と`down`しか実装されていませんが、最低限コンテナの作成と削除はできますので、試してみましょう。
まず次の`compose.yaml`を用意します。

```sh
$ cat compose.yaml
version: '3.7'

services:
   wordpress:
     image: wordpress:latest
     container_name: wordpress
     depends_on:
       - db
     ports:
       - "8000:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
       WORDPRESS_DB_NAME: wordpress
     restart: always
   db:
     image: mysql:5.7
     container_name: wordpress_db
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: root
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
```

用意できたら次にイメージをpullしてきます。`docker-compose`ではイメージがなければ自動でpullしてくれますが、残念ながらこのリファレンス実装では自動でイメージをpullする実装になっていないです。
イメージがないと次のように`No suc image: wordpress:latest`というメッセージが出ます。

```sh
$ compose-ref up

 .---.
(     )
 )@ @(
//|||\\

Creating container for service db ... 4f50328e447116aebe091c150607eb5dee1bee83b01e622785c8141c4dbf5a09
Creating container for service wordpress ... 2020/05/06 15:35:05 Error response from daemon: No such image: wordpress:latest
```

イメージを用意できたら`up`しましょう。

```sh
$ compose-ref up

 .---.
(     )
 )@ @(
//|||\\

Creating container for service wordpress ... 12cee476290011aae97abe9982dc43be164449adade2cdb55467828d92362cf6
Stopping containers for service hello-world ... 3135cb302bba563ee4da9c54428d5083d45487727b41c2a28f7c4abbb2273aea

$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS                  NAMES
12cee4762900        wordpress:latest    "docker-entrypoint.s…"   47 seconds ago      Up 46 seconds                   0.0.0.0:8000->80/tcp   blissful_edison
4f50328e4471        mysql:5.7           "docker-entrypoint.s…"   4 minutes ago       Restarting (1) 11 seconds ago                          inspiring_blackwell
```

現時点ではまだ、`container_name`は適用されないようですね。
ちなみに、`compose-ref`で作られたコンテナは`Config.Labels`にいくつかラベルが貼られます。確認してみましょう。

```sh
$ docker inspect -f "{{ .Config.Labels }}" blissful_edison inspiring_blackwell
map[io.compose-spec.config:container_name: wordpress
depends_on:
- db
environment:
  WORDPRESS_DB_HOST: db:3306
  WORDPRESS_DB_NAME: wordpress
  WORDPRESS_DB_PASSWORD: wordpress
  WORDPRESS_DB_USER: wordpress
image: wordpress:latest
ports:
- mode: ingress
  target: 80
  published: 8000
  protocol: tcp
restart: always
 io.compose-spec.project:compose-ref io.compose-spec.service:wordpress]
map[io.compose-spec.config:container_name: wordpress_db
environment:
  MYSQL_DATABASE: wordpress
  MYSQL_PASSWORD: wordpress
  MYSQL_ROOT_PASSWORD: root
  MYSQL_USER: wordpress
image: mysql:5.7
restart: always
 io.compose-spec.project:compose-ref io.compose-spec.service:db]
```

ちゃんとありますね。

次に`down`の動作を見てみましょう。

```sh
compose-ref down

 .---.
(     )
 )@ @(
//|||\\

Stopping containers for service wordpress ... 12cee476290011aae97abe9982dc43be164449adade2cdb55467828d92362cf6
Stopping containers for service db ... 4f50328e447116aebe091c150607eb5dee1bee83b01e622785c8141c4dbf5a09
Deleting network compose-ref-default ... compose-ref-default

$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

ちゃんと消えてますね。ネットワークまで消してくれています。

以上、簡単ではありますがリファレンス実装の動作確認です。まだ実装途中ですが仕様を流し読みした感じではこれまでと大きく変わることはなさそうです。

## おまけ
`compose-ref up`するとき、イメージがなければpullしてくれないのは不便なのでPRを投げました

https://github.com/compose-spec/compose-ref/pull/46

## まとめ
`Compose Specification`で仕様がある程度定まったらいろいろとおもしろくなりそうだなというのが個人的な感想です。
せっかくですので、この仕様を元に実用的なCLIを作ろうと思っています。できあたらそのうち公開します。

