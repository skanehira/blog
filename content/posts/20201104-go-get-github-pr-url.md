---
title: コミットIDからGitHubのPRリンクを取得するCLI
date: 2020-11-04
toc: true
tags: 
  - Go
  - GitHub
---

## 初めに
普段GitHubを使って仕事で、次のフローで開発をしています。

1. PRを出す
2. レビュー
3. マージ

そして、たまにバグを見つけるので、`git blame`を使ってどのコミットが原因なのかを特定することがありますが、どのPRなのかまで知りたいことがあります。

そこで、GitHub限定ではありますが、コミットIDからPRのリンクを取得できるCLIを作りました。
本記事はその紹介と作るに至るまでの過程について書きました。

## CLIについて
Goで[getpr](https://github.com/skanehira/getpr)というCLIを作りました。

![](https://i.imgur.com/VrXQw15.gif)

使い方引数にコミットIDを指定するだけです。

```bash
$ getpr 02b3cb3
https://github.com/skanehira/getpr/pull/2
```

特定のリポジトリを指定することもできます。

```bash
$ getpr skanehira/getpr 02b3cb3
https://github.com/skanehira/getpr/pull/2
```

## コミットIDからPRを取得する方法
このCLIを作るまで結構試行錯誤していました。
[こちら](https://qiita.com/awakia/items/f14dc6310e469964a8f7)を見るとコミットメッセージに`Merge pull request #`があれば、PRの番号を取得できます。
番号さえわかればあとはURLを組み立てるだけですので割と簡単ですが、問題点はマージだけどメッセージが`Merge pull request`になっていない場合です。

たとえば、squashマージする際はメッセージが変わります。このようにメッセージから取得するとどうしても溢れるケースが出てきます。
特に僕はプロジェクトでsquashマージするのでむしろこのやり方ではPRのリンクを取れないです。

squashマージでもメッセージからPR番号を取ることはできますが、結局の所メッセージに依存してしまうので、根本解決できない問題は変わらないです。

## GitHubのblame
GitHubにはblame機能があって、画像のようにコミットの詳細にPRのリンクが表示されたりします。

![](https://storage.googleapis.com/zenn-user-upload/telwvr0au8eka1mpx4oinnl0xm17)

これはつまりGitHubがコミットとPRのリレーションを持っているということでもあります。
であれば、GitHubのAPIから取れるのではないか考え調べたところ、取れることがわかりました。

## コミットIDからPRを取得
[こちら](https://github.community/t/find-a-commit-by-oid-sha-via-graphql-v4-api/14118/9)を見ると、GitHubのv4のAPIを使えば次のようにコミットIDから関連のPR情報を取得できます。

```graphql
{
  repository(owner: "skanehira", name: "getpr") {
    object(expression: "02b3cb3") {
      ... on Commit {
        associatedPullRequests(first: 1) {
          nodes {
            url
          }
        }
      }
    }
  }
}
```

あとはこれをGoで実装すればよいだけですので、それほど難しくありません。
実際データを取得する部分の実装は次だけです。とてもシンプルです。

```go
func getPullRequest(owner, name, id string) (*PullRequest, error) {
	var q struct {
		Repository struct {
			Object struct {
				Commit struct {
					AssociatedPullRequests struct {
						Nodes []PullRequest
					} `graphql:"associatedPullRequests(first: 1)"`
				} `graphql:"... on Commit"`
			} `graphql:"object(expression: $id)"`
		} `graphql:"repository(owner: $owner, name: $name)"`
	}

	variables := map[string]interface{}{
		"id":    githubv4.String(id),
		"owner": githubv4.String(owner),
		"name":  githubv4.String(name),
	}

	if err := client.Query(context.Background(), &q, variables); err != nil {
		return nil, err
	}

	if len(q.Repository.Object.Commit.AssociatedPullRequests.Nodes) < 1 {
		return nil, errors.New("not found pull request")
	}

	return &q.Repository.Object.Commit.AssociatedPullRequests.Nodes[0], nil
}
```

## 後日談
この記事を書いている最中に[こちら](https://qiita.com/kyoshidajp/items/18d5385ef8375c7bcd14)を見つけて車輪の再開発だったことが判明しました。
いろいろと勉強になったので、プロジェクト外での車輪の再開発は悪いことばかりではないですが、ちょっと悔しかったのは内緒です。
