---
title: "新しい docker compose"
date: 2021-06-03
toc: true
tags:
  - Docker
  - docker-compose
---

こちらの記事は[DockerCon 2021 & Docker Meetup Tokyo #35](https://dockerjp.connpass.com/event/194395/)で発表した資料を記事にしたものです。

# 初めに
`docker compose`が使えるようになったので、それについて書いていきます。
正式名称は[Docker Compose CLI](https://github.com/docker/compose-cli)です。

動作検証した環境は次のとおりです。

```sh
$ docker version
Client:
 Cloud integration: 1.0.14
 Version:           20.10.6
 API version:       1.41
 Go version:        go1.16.3
 Git commit:        370c289
 Built:             Fri Apr  9 22:46:57 2021
 OS/Arch:           darwin/arm64
 Context:           default
 Experimental:      true
...
```

## Docker Compose CLIとは
簡単にいうと`docker-compose`のGo実装です。`docker-compose`と互換しています。
`docker-compose`に置き換わることを目標としていて、実装は[compose-spec](https://github.com/compose-spec/compose-spec)の仕様に準拠しています。
既存`docker-compose`のメンテナンスは継続されるそうです。

現時点利用できるコマンド一覧は次になっています。

```
Commands:
  build       Build or rebuild services
  convert     Converts the compose file to platform's canonical format
  create      Creates containers for a service.
  down        Stop and remove containers, networks
  events      Receive real time events from containers.
  exec        Execute a command in a running container.
  images      List images used by the created containers
  kill        Force stop service containers.
  logs        View output from containers
  ls          List running compose projects
  pause       pause services
  port        Print the public port for a port binding.
  ps          List containers
  pull        Pull service images
  push        Push service images
  restart     Restart containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service.
  start       Start services
  stop        Stop services
  top         Display the running processes
  unpause     unpause services
  up          Create and start containers
```

## compose-specとは
プラットフォームに依存しない、マルチコンテナなアプリケーションを定義するための標準仕様です。
もともとは`docker-compose`が提供していた機能を標準な仕様として査定したので、特に大きな変更はないようです。
公式サイトは https://compose-spec.io です。


## Docker Compose CLIの特徴
大きく分けて2つの特徴があります。

### クラウド対応
- マルチコンテナを`ECS`と`ACI(Azure Cloud Instance)`に簡単にデプロイできる
- AWSのECSの場合
	- `docker compose up`でFargateのクラスタ、サービス、タスクやLBなどを作成してくれる
	- `docker compose down`で作成したリソースを削除

### CLI本体
- `docker-compose.yaml`のパースライブラリ[compose-go](https://github.com/compose-spec/compose-go)が公開されている
  - `Docker Compose CLI`もそれを利用している
  - `compose`関連のツールの作成が楽になる
- buildkitがデフォルト有効になっている
- Goによる実装なので、既存のエコシステム（`Docker CLI`のSDKなど）を利用できる
	- 実際内部ではSDKを使って`Docker Engine`とやりとりしている

## docker-composeとの差分
一部`docker-compose`で非推奨になったフラグは対応しないそうです。
たとえば`docker-compose rm --all`など。

詳細は以下を参照してください。issueの方には具体的なオプションが記載されています。
- https://docs.docker.com/compose/cli-command-compatibility
- https://github.com/docker/compose-cli/issues/1283


## docker-composeからの移行について
個人、開発での利用は移行しても良さそうと思っています。
理由として以下になります。

- `docker-compose`の機能をほぼ互換している
	- 一部のフラグは対応していないが、基本的に問題ない
- 便利な新機能が追加される可能性が高い
	- 既存の財産を気軽に利用でくるのと、クラウドもターゲットに入れているので


## docker composeを使ってみる
- https://docs.docker.jp/compose/wordpress.html のyamlを使用
  ```yaml
  version: '3'

  services:
     db:
       ...
     wordpress:
       depends_on:
         - db
       image: wordpress:latest
       ports:
         - "8000:80"
       restart: always
       environment:
         WORDPRESS_DB_HOST: db:3306
         WORDPRESS_DB_USER: wordpress
         WORDPRESS_DB_PASSWORD: wordpress
  volumes:
      db_data:
  ```

- コンテナ作成
  ```sh
  MacbookPro13% docker compose up
  [+] Building 12.3s (3/4)
  [+] Building 12.4s (3/4)
  [+] Building 12.5s (3/4)
  [+] Building 24.3s (5/5) FINISHED
   => [internal] load build definition from Dockerfile          0.0s
   => => transferring dockerfile: 58B                           0.0s
  ...
  [+] Running 4/1
   ⠿ Network demo_default        Created                        3.5s
   ⠿ Volume "demo_db_data"       Created                        0.0s
   ⠿ Container demo_db_1         Created                        0.1s
   ⠿ Container demo_wordpress_1  Created                        0.0s
  Attaching to db_1, wordpress_1
  ...
  wordpress_1  | [Sun May 23 12:28:34.735398 2021] [mpm_prefork:n...
  wordpress_1  | [Sun May 23 12:28:34.735591 2021] [core:notice] ...
  ...
  db_1         | 2021-05-23T12:28:46.521520Z 0 [Note] /usr/sbin/m...
  db_1         | Version: '5.7.33'  socket: '/var/run/mysqld/mysq...
  ```
- 初期画面
  ![](https://i.gyazo.com/4ca0bf51bac235bf75d6c829f20d9582.png)

- コンテナ一覧と停止
  ```sh
  MacbookPro13% docker compose ps
  NAME                SERVICE      STATUS    PORTS
  demo_db_1           db           running   3306/tcp, 33060/tcp
  demo_wordpress_1    wordpress    running   0.0.0.0:8000->80/tcp, :::8000->80/tcp
  MacbookPro13% docker compose down
  [+] Running 3/3
   ⠿ Container demo_wordpress_1  Removed     2.1s
   ⠿ Container demo_db_1         Removed     1.2s
   ⠿ Network demo_default        Removed     2.5s
  MacbookPro13%
  ```

- コンテナにアタッチする
  ```sh
  MacbookPro13% docker compose exec db mysql -uwordpress -pwordpress -Dwordpress
  mysql: [Warning] Using a password on the command line interface can be insecure.
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 2
  Server version: 5.7.33 MySQL Community Server (GPL)

  Copyright (c) 2000, 2021, Oracle and/or its affiliates.

  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  mysql>
  ```

- プロセス一覧
  ```sh
  MacbookPro13% docker compose top
  demo_db_1
  UID   PID   PPID   C    STIME   TTY   TIME       CMD
  999   388   358    0    13:22   ?     00:00:01   /usr/bin/qemu-x86_64 /u...

  demo_wordpress_1
  UID        PID    PPID   C    STIME   TTY   TIME       CMD
  root       809    778    0    13:22   ?     00:00:00   apache2 -DFOREGROUND
  www-data   1175   809    0    13:22   ?     00:00:00   apache2 -DFOREGROUND
  www-data   1176   809    0    13:22   ?     00:00:00   apache2 -DFOREGROUND
  www-data   1177   809    0    13:22   ?     00:00:00   apache2 -DFOREGROUND
  www-data   1178   809    0    13:22   ?     00:00:00   apache2 -DFOREGROUND
  www-data   1179   809    0    13:22   ?     00:00:00   apache2 -DFOREGROUND
  ```
## docker composeを使ってECSにコンテナをデプロイしてみる
:::message
サンプルなので、特にセキュリティを考慮していない内容となっています。
:::

- 利用する`docker-compose.yaml`
  ```sh
  MacbookPro13% cat docker-compose.yaml
  version: "3"

  services:
    nginx:
      image: nginx:1.19.10-alpine
      ports:
        - "80:80"
  MacbookPro13%
  ```

- 既存のAWSのプロファイルを利用
  ```sh
  MacbookPro13% docker context create ecs myecs
  ? Create a Docker context using: An existing AWS profile
  ? Select AWS Profile xxxxxxx
  Successfully created ecs context "myecs"
  MacbookPro13% docker context use myecs
  myecs
  MacbookPro13% docker context ls
  NAME         TYPE      DESCRIPTION...
  default      moby      Current DOC...
  myecs *      ecs
  ```


- コンテナを作成
  ```sh
  $ docker compose up
  [+] Running 12/14
   ⠙ demo                        CreateInProgress User Initiated               177.2s
   ⠿ NginxTCP80TargetGroup       CreateComplete                                  1.0s
   ⠿ CloudMap                    CreateComplete                                 47.1s
   ⠿ NginxTaskExecutionRole      CreateComplete                                 19.9s
   ⠿ LogGroup                    CreateComplete                                  3.1s
   ⠿ DefaultNetwork              CreateComplete                                  5.9s
   ⠿ Cluster                     CreateComplete                                  5.8s
   ⠿ DefaultNetworkIngress       CreateComplete                                  1.0s
   ⠿ Default80Ingress            CreateComplete                                  1.0s
   ⠿ LoadBalancer                CreateComplete                                 92.3s
   ⠿ NginxTaskDefinition         CreateComplete                                  3.1s
   ⠿ NginxServiceDiscoveryEntry  CreateComplete                                  2.0s
   ⠿ NginxTCP80Listener          CreateComplete                                  3.0s
   ⠋ NginxService                CreateInProgress Resource creation Initiated   65.0s
  ```

- サービスを確認
  ```sh
  MacbookPro13% docker compose ps --format=json | jq
  [
    {
      "ID": "arn:aws:ecs:ap-northeast-1:xxxxxxxxxxxx:task/demo/4af498c8586849c691063b7f4d202cec",
      "Name": "task/demo/4af498c8586849c691063b7f4d202cec",
      "Project": "demo",
      "Service": "nginx",
      "State": "Running",
      "Health": "",
      "Publishers": [
        {
          "URL": "demo-LoadBa-187KUDJD32QRX-2082745493.ap-northeast-1.elb.amazonaws.com:80",
          "TargetPort": 80,
          "PublishedPort": 80,
          "Protocol": "http"
        }
      ]
    }
  ]
  ```

- アクセスしてみる
  ```sh
  MacbookPro13% curl -L demo-LoadBa-187KUDJD32QRX-2082745493.ap-northeast-1.elb.amazonaws.com
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
      body {
          width: 35em;
          margin: 0 auto;
          font-family: Tahoma, Verdana, Arial, sans-serif;
      }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  ...
  ```

- リソースを削除
  ```sh
  MacbookPro13% docker compose down
  [+] Running 14/14
   ⠿ demo                        DeleteComplete                         466.1s
   ⠿ Default80Ingress            DeleteComplete                           1.1s
   ⠿ NginxService                DeleteComplete                         414.2s
   ⠿ DefaultNetworkIngress       DeleteComplete                           1.1s
   ⠿ NginxTCP80Listener          DeleteComplete                           1.7s
   ⠿ Cluster                     DeleteComplete                           0.7s
   ⠿ NginxServiceDiscoveryEntry  DeleteComplete                           1.7s
   ⠿ NginxTaskDefinition         DeleteComplete                           1.7s
   ⠿ CloudMap                    DeleteComplete                          47.1s
   ⠿ LogGroup                    DeleteComplete                           2.0s
   ⠿ NginxTaskExecutionRole      DeleteComplete                           1.0s
   ⠿ LoadBalancer                DeleteComplete                           0.0s
   ⠿ NginxTCP80TargetGroup       DeleteComplete                           0.0s
   ⠿ DefaultNetwork              DeleteComplete                          24.1s
  MacbookPro13%
  ```

## Docker Compose CLIのアーキテクチャ
[Docker Compose CLI](https://github.com/docker/compose-cli)より
![](https://github.com/docker/compose-cli/blob/main/docs/images/cli-architecture.png?raw=true)

`Docker Compose CLI`はプラットフォーム依存しないように、各プラットフォームの差異は`Backend Interface`で吸収しています。インタフェースの定義は次のようになっています。

```go
// Service aggregates the service interfaces
type Service interface {
	ContainerService() containers.Service
	ComposeService() compose.Service
	ResourceService() resources.Service
	SecretsService() secrets.Service
	VolumeService() volumes.Service
}
```

`container.Service`はコンテナの作成、起動や停止など、`compose.Service`は`docker compose`のサブコマンド郡に相当します。

現時点で以下の`backend`が実装されています。
- `ECS`
- `ACI`
- `Docker Engine`

また、`API`は`gRPC`のサーバとなっていて言語に縛られず`docker compose`の機能を使用可能のようです。
実際動かしていないんですが、protoファイルは[こちら](https://github.com/docker/compose-cli/tree/main/cli/server/protos)にあるので興味ある方は試してみてください。

## Docker CLI Pluginについて
`docker compose`は`Docker CLI Plugin`として同梱されています。（Mac以外は手動インストールが必要かも）
本体は次のパスにいます。

```sh
MacbookPro13% ls /usr/local/lib/docker/cli-plugins/*
/usr/local/lib/docker/cli-plugins/docker-app
/usr/local/lib/docker/cli-plugins/docker-buildx
/usr/local/lib/docker/cli-plugins/docker-compose  <-- これ
/usr/local/lib/docker/cli-plugins/docker-scan
```

このように、バイナリを特定のディレクトリ配下に置くことで、`docker`のサブコマンドとして実行できるしくみが`Docker CLI Plugin`です。
このしくみについて調べた限りでは公式のドキュメントはなく、プロポーザルissueと実装PRしか見つからなかったので、メモと言う意味でもこの記事に残していきます。

ちなみに、`docker plugin`というのがありますが、こちらは別物です。
`Docker Engine`自体の機能を拡張する物となっていて、たとえばボリュームやネットワークの機能を拡張できたりします。
詳細は[こちら](https://docs.docker.jp/engine/extend/plugins.html)を参照してください。

`Docker CLI Plugin`は[v19.03.0](https://github.com/docker/docker-ce/releases/tag/v19.03.0)で導入された機能です。
上記のパスは`docker`本体が提供しているプラグインですが、ユーザーの場合は`$HOME/.docker/cli-plugins`配下に実行ファイルを配置します。

プラグインのファイル名は`docker-*`にする必要があり、たとえば`docker-hello`なら`docker hello`として実行できます。
また、`docker-cli-plugin-metadata`コマンドを実装する必要があり、出力は次のようなJSON構造になります。

```json
{
  "SchemaVersion": "0.1.0",
  "Vendor": "Gorilla",
  "Version": "v0.0.1",
  "ShortDescription": "Hello Gorilla"
}
```

### 実際プラグインを作ってみる
以下のスクリプトを用意します。

```sh
#!/usr/bin/env bash

docker_cli_plugin_metadata() {
  local vendor="Gorilla"
  local url="https://github.com/skanehira"
  local description="Hello Docker CLI Plugin"
  local version="0.0.1"
  cat <<-EOF
  {"SchemaVersion":"0.1.0","Vendor":"${vendor}","Version":"${version}","ShortDescription":"${description}","URL":"${url
}"}
EOF
}

case "$1" in
  docker-cli-plugin-metadata)
    docker_cli_plugin_metadata
    ;;
  *)
    echo $@
    ;;
esac
```

次に、ディレクトリと`docker-hello`を用意します。

```sh
MacbookPro13% mkdir -p ~/.docker/cli-plugins
MacbookPro13% cd ~/.docker/cli-plugins
MacbookPro13% curl -LO \
https://gist.githubusercontent.com/skanehira/2907d10669d8d7e9e75a39cd97dce9bc/raw/a52e77595fcc29dcba395303ddb405f34ea1e7e1/docker-hello
MacbookPro13% chmod +x docker-hello
```

これで、プラグインの準備はできたので動かしてみます。

```sh
MacbookPro13% docker 2>&1 | grep hello
  hello*      Hello Docker CLI Plugin (Gorilla, 0.0.1)
MacbookPro13% docker hello gorilla
hello gorilla
MacbookPro13% docker hello moby
hello moby
```

無事出力されましたね。自作ツールをこのように配置して`docker`のサブコマンドとして実行すると便利かもしれません。

## 最後に
`docker compose`は今後期待していきたいところですね。
みなさんもよかったら試してみてください。

## 参考資料
- `Docker Compsoe CLI`の公式ドキュメント
  - https://docs.docker.com/compose/cli-command/
- クラウドサービスへのデプロイのドキュメント
  - https://docs.docker.com/cloud/aci-integration/
  - https://docs.docker.com/cloud/ecs-integration/
- docker-composeを`Docker CLI Plugin`として登録する方法
  - https://gist.github.com/thaJeztah/b7950186212a49e91a806689e66b317d
- docker expose プラグイン便利そうなので試す
  - https://qiita.com/tksarah/items/9e46d131107d3e15f7bc

