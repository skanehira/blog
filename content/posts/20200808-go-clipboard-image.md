---
title: Goで画像をクリップボードに保存、読み取るライブラリを作った
date: 2020-08-08
toc: true
tags: 
  - Go
---

## 初めに
こんにちわ
ゴリラです

画像をクリップボードにコピー、コピーした画像をクリップボードから読み取るGoのライブラリ [clipboard-image](https://github.com/skanehira/clipboard-image) をつくったので紹介します。
このライブラリはMac、Linux、Windowsで動作します。

先日 [Goでソースコードを画像化するCLIを作った](https://qiita.com/gorilla0513/items/013aea9060bca1455137) で紹介したCLIではこのライブラリを使っています。

## 使い方
関数は2つのみ

- `CopyToClipboard`
- `ReadFromClipboard`

画像ファイルをクリップボードにコピーするときは `CopyToClipboard`

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/598e7a39-59fc-fc35-9982-9093fefde5ff.png)

クリップボードから画像ファイルを読み取るときは `ReadFromClipboard`

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/5c676cb6-5a69-70d6-4af2-0c5cc9e93178.png)

## 仕組み
クリップボードへのアクセスは外部コマンドを用いています。
それらをGoでラップしてライブラリとして提供しています。それぞれ次になっています。

| OS      | コマンド   |
|---------|------------|
| Mac OS  | osascript  |
| Windows | PowerShell |
| Linux   | xclip      |

Mac OSではApple Scriptを使えばクリップボードにアクセスできます。
Apple Scriptは標準でMac OSに入っているのでライブラリ意外は特に不要です。

```sh
# copy to clipboard
osascript -e 'set the clipboard to (read "/path/to/file.png" as TIFF picture)'

# write file from clipboard
osascript -e 'write (the clipboard as «class PNGf») to (open for access "/tmp/file.png" with write permission)'
```

WindowsではPowershellでクリップボードにアクセスできます。

```sh
# copy to clipboard
PowerShell -Command Add-Type -AssemblyName System.Windows.Forms;[Windows.Forms.Clipboard]::SetImage([System.Drawing.Image]::FromFile('/path/to/file.png'));

# write file from clipboard
PowerShell -Command Add-Type -AssemblyName System.Windows.Forms;$clip=[Windows.Forms.Clipboard]::GetImage();if ($clip -ne $null) { $clip.Save('/path/to/file.png') };
```

Linuxでは`xclip`というコマンドをインストールことでクリップボードにアクセスできます。

```sh
# copy to clipboard
xclip -selection clipboard -t image/png < /path/to/file.png

# write file from clipboard
xclip -selection clipboard -o > /path/to/file.png
```

これらのコマンドをGoの`exec.Command`で実行することで、Goでクリップボードにアクセスできます。

## 詰まったポイント
`xclip`を使う時、標準入力でファイルの中身を渡すのですが、次のように書いても標準入力に渡っていかないです。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/3a94a766-6962-8f61-181b-01500522f5e1.png)

標準入力に渡すには`StdinPipe`でpipeを取得してファイルの中身を書き込んだ後に stdin を閉じる必要あります。
`xclip`の実装は[atotto/clipboard](https://github.com/atotto/clipboard)を参考にしました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/66178/14aae070-760d-9abb-af14-0ac5c9313aaf.png)

## さいごに
クリップボード周りは結構苦戦して、いろいろな方に質問してなんとか作れました。
ライブラリとしてはまだ完成形とは考えていなくて、今後も拡張していきたいと考えています。

ちなみに、この記事の画像はVimで書いたコードを[code2img.vim](https://github.com/skanehira/code2img.vim)でクリップボードにコピーしてサクッとQiitaにアップロードしたものです。
当初手軽にVimでコードを画像化したいという目的を達成できて、実際使って便利ので満足しています。

コードブロックが使えないチャットやTwitterといったSNSでコード貼りたいときにぜひCLIを使ってみてください。

