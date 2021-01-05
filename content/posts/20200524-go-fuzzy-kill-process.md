---
title: プロセスをあいまい検索してkillするツールをGoで作ったの調査
date: 2020-05-24
toc: true
tags: 
  - Go
---

## 初めに
仕事しているとプロセスをkillすることがたまにあると思います。

大体は`ps`、`awk`、`grep`で必要なプロセスIDを抽出して`kill`コマンドに渡していますが、
ぼくはそれがとても面倒に感じているので、あいまい検索してプロセスをkillしたいなと思って`fk(fuzzy-finder-killer)`ってコマンドを作りました。

![](https://i.imgur.com/Q6ONFRz.gif)

## 導入と使い方
`go get github.com/skanehira/fk`もしくは[releases](https://github.com/skanehira/fk/releases)からバイナリをダウンロードしてください。

使い方は `fk` を実行するだけです。
`fk` を実行するとあいまい検索できる状態になるので、任意の単語を入力して、`CTRL-i`で選択します。
`Enter`で選択済みのプロセスをkillします。

キーバインドは次になります。

| key    | description     |
|--------|-----------------|
| CTRL-i | select/unselect |
| CTRL-j | go to next      |
| CTRL-k | go to prev      |
| CTRL-c | abort           |
| Enter  | kill            |

## 仕組み
あいまい検索のUIは[go-fuzzyfinder](github.com/ktr0731/go-fuzzyfinder)というライブラリを使っています。
こちらのライブラリは`fzf`と似たインターフェイスを持っていてかつとてもシンプルなので、GoでfzfのようなUIを使いたい場合はぜひ使ってみてください。

プロセス一覧は[go-ps](github.com/mitchellh/go-ps)というライブラリを使っています。このライブラリでプロセスIDとコマンド名を取得できるので、それを`go-fuzzyfinder`に渡します。

メインの処理は次の部分になります。

```go
func processes() ([]process, error) {
	var processes []process
	procs, err := ps.Processes()
	if err != nil {
		return nil, err
	}

	for _, p := range procs {
		// skip pid 0
		if p.Pid() == 0 {
			continue
		}
		processes = append(processes, process{
			Pid: p.Pid(),
			Cmd: p.Executable(),
		})
	}

	return processes, nil
}

func main() {
	...
	procs, err := processes()
	if err != nil {
		fmt.Fprintln(os.Stderr, err)
		os.Exit(1)
	}

	idx, err := fuzzyfinder.FindMulti(
		procs,
		func(i int) string {
			return fmt.Sprintf("%d: %s", procs[i].Pid, procs[i].Cmd)
		},
	)
	...

	for _, i := range idx {
		pid, cmd := procs[i].Pid, procs[i].Cmd
		fmt.Println(pid, cmd)

		if err := kill(pid); err != nil {
			fmt.Fprintln(os.Stderr, err)
		}
	}

}
```


## おわり
小さなツールですが、プロセスをkillする面倒さから開放されます。
是非試してみてください

