---
title: "GoでCLIを作る時のテンプレートリポジトリを作った"
date: 2021-04-21
toc: true
tags:
  - Go
---

## 初めに
普段Goを使ってCLIなどを作ることが多く、毎度CI/CDをセットアップしたり、`go get`でライブラリを導入するが面倒なのでテンプレートリポジトリ[go-cli-template](https://github.com/skanehira/go-cli-template)を用意しました。  
基本的に自分用ですが、ちょっと手を入れればみなさんも使えるので軽く紹介していきます。

## デフォルトセット
GitHub Actions（以降GA）を使って以下を設定してあります。

- golangci-lint
- test
- binary release

`golangci-lint`はGAがあるので、それをそのまま使っています。  
使う人の好みもあるので`.golangci.yaml`はあえて設置していないです。各自好きに置いてくださいという感じです。  

binary releaseは`goreleaser`のGAを使っています。タグをつけてpushすると`Releases`にCHANGELOG、ハッシュ、各OSのバイナリファイルがアップロードされます。便利ですね。

上記に加えて、`spf13/cobra`のベース実装（`version`コマンドのみ）も入っています。  
個人的にこれまで`flag`パッケージを使っていましたが、より簡単にオプションやサブコマンドを実装できるように`cobra`を選びました。  
そのためパッケージ構成も`cobra`推奨の形（サブコマンドは`cmd`配下に置く）になっています。  
依存パッケージがかなり多いライブラリですが、`docker`といった有名なOSSでの採用実績があるため、そちら重視しました。  

## 使い方
1. リポジトリの`Use this tempalte`から新しいリポジトリを作成します。
   ![](https://i.gyazo.com/3166d239ed1e14611d5d43a989bfd988.png)
2. 新しいリポジトリを`git clone`して`make init`を実行して、リポジトリ名を変更します。
   ```diff
   diff --git a/.goreleaser.yml b/.goreleaser.yml
   index 856c0f0..a993558 100644
   --- a/.goreleaser.yml
   +++ b/.goreleaser.yml
   @@ -1,4 +1,4 @@
   -project_name: go-cli-template
   +project_name: test-cli
    env:
      - GO111MODULE=on
    before:
   @@ -8,5 +8,5 @@ builds:
      - main: .
        ldflags:
          - -s -w
   -      - -X github.com/skanehira/go-cli-template/cmd.Version={{.Version}}
   -      - -X github.com/skanehira/go-cli-template/cmd.Revision={{.ShortCommit}}
   +      - -X github.com/skanehira/test-cli/cmd.Version={{.Version}}
   +      - -X github.com/skanehira/test-cli/cmd.Revision={{.ShortCommit}}
   diff --git a/Makefile b/Makefile
   index dbfc09c..12d08c5 100644
   --- a/Makefile
   +++ b/Makefile
   @@ -1,7 +1,7 @@
    .PHONY: init
    init:
    ifeq ($(shell uname -s),Darwin)
   -	@grep -r -l go-cli-template * .goreleaser.yml | xargs sed -i "" "s/go-cli-template/$$(basename `git rev-parse --show-toplevel`)/"
   +	@grep -r -l test-cli * .goreleaser.yml | xargs sed -i "" "s/go-cli-template/$$(basename `git rev-parse --show-toplevel`)/"
    else
   -	@grep -r -l go-cli-template * .goreleaser.yml | xargs sed -i "s/go-cli-template/$$(basename `git rev-parse --show-toplevel`)/"
   +	@grep -r -l test-cli * .goreleaser.yml | xargs sed -i "s/go-cli-template/$$(basename `git rev-parse --show-toplevel`)/"
    endif
   diff --git a/cmd/root.go b/cmd/root.go
   index 27e0f98..b9feb58 100644
   --- a/cmd/root.go
   +++ b/cmd/root.go
   @@ -8,7 +8,7 @@ import (
    )
    
    var rootCmd = &cobra.Command{
   -	Use: "go-cli-template",
   +	Use: "test-cli",
    }
    
    func exitError(msg interface{}) {
   diff --git a/go.mod b/go.mod
   index c586f1b..2598da5 100644
   --- a/go.mod
   +++ b/go.mod
   @@ -1,4 +1,4 @@
   -module github.com/skanehira/go-cli-template
   +module github.com/skanehira/test-cli
    
    go 1.16
    
   diff --git a/main.go b/main.go
   index 900b7d9..ee06357 100644
   --- a/main.go
   +++ b/main.go
   @@ -1,6 +1,6 @@
    package main
    
   -import "github.com/skanehira/go-cli-template/cmd"
   +import "github.com/skanehira/test-cli/cmd"
    
    func main() {
    	cmd.Execute()
   ```
3. 試しにサブコマンドを1つ実装してみます。
   ```diff
   diff --git a/cmd/hello.go b/cmd/hello.go
   new file mode 100644
   index 0000000..fa5e640
   --- /dev/null
   +++ b/cmd/hello.go
   @@ -0,0 +1,24 @@
   +package cmd
   +
   +import (
   +	"fmt"
   +
   +	"github.com/spf13/cobra"
   +)
   +
   +func Hello(name string) string {
   +	return "hello " + name
   +}
   +
   +var helloCmd = &cobra.Command{
   +	Use: "hello",
   +	Run: func(cmd *cobra.Command, args []string) {
   +		if len(args) > 0 {
   +			fmt.Println(Hello(args[0]))
   +		}
   +	},
   +}
   +
   +func init() {
   +	rootCmd.AddCommand(helloCmd)
   +}
   ```
4. テストも書いてみます。
   ```diff
   diff --git a/cmd/hello_test.go b/cmd/hello_test.go
   new file mode 100644
   index 0000000..8842509
   --- /dev/null
   +++ b/cmd/hello_test.go
   @@ -0,0 +1,26 @@
   +package cmd
   +
   +import "testing"
   +
   +func TestHello(t *testing.T) {
   +	tests := []struct {
   +		in   string
   +		want string
   +	}{
   +		{
   +			in:   "gorilla",
   +			want: "hello gorilla",
   +		},
   +		{
   +			in:   "cat",
   +			want: "hello cat",
   +		},
   +	}
   +
   +	for _, tt := range tests {
   +		got := Hello(tt.in)
   +		if tt.want != got {
   +			t.Errorf("unexpected result. want=%s, got=%s", tt.want, got)
   +		}
   +	}
   +}
   ```
5. コミットしてpushするとCIが走ります。ちゃんとテストとlinterが通ることを確認します。
   ![](https://i.gyazo.com/22c98ab4e3ef1957de3a06635dfe7ab5.png)
6. `v0.0.1`タグをつけてバイナリをリリースします。
   ![](https://i.gyazo.com/e16fa631ca34d78abc5db26eb0fcbef7.png)
   ![](https://i.gyazo.com/7f3fd1e6e0dd014ea388412823c18af5.png)
7. 試しにバイナリをダウンロードして使ってみます。
   ```shell
   MacbookPro13% curl -LO https://github.com/skanehira/test-cli/releases/download/v0.0.1/test-cli_0.0.1_Darwin_arm64.tar.gz
     % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
   100   642  100   642    0     0   2051      0 --:--:-- --:--:-- --:--:--  2051
   100 1152k  100 1152k    0     0  1776k      0 --:--:-- --:--:-- --:--:-- 9852k
   MacbookPro13% tar xvf test-cli_0.0.1_Darwin_arm64.tar.gz
   x LICENSE
   x test-cli
   MacbookPro13% ./test-cli version
   Version: 0.0.1
   Revision: 3c302bf
   OS: darwin
   Arch: arm64
   MacbookPro13% ./test-cli hello gorilla
   hello gorilla
   MacbookPro13% ./test-cli hello cat
   hello cat
   MacbookPro13%
   ```

簡単ではありますが、以上がサブコマンド追加からリリースまでの一連の流れの説明となります。  
これでだいぶ手間が省けると思います。

## forkして使う時
`sed`などを使って`skanehira`を自分のユーザー名に置換してください。  
あとは使い方に書いてある通りになります。お試しください。

## 最後に
CI/CDについて説明の記事は多くありますが、テンプレートを用意してさぁ使ってみてという記事が観測範囲内ではなかったので書いてみました。

個人的にいったんこれで満足していますが、使っていくうちに不満が出てきて改善するかもしれません。  
こうするともっと良いよ、というのがあればぜひコメントください。お待ちしています。
