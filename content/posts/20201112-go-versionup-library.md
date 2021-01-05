---
title: Goで作ったライブラリのバージョンアップ手順
date: 2020-11-12
toc: true
tags: 
  - Go
---

## 初めに
以前Goでクリップボードから画像を取得&保存できるライブラリ[clipboard-image](https://github.com/skanehira/clipboard-image)を作りました。

ただ、関数名が気に入らなかったので名前を変えてv1->v2にアップして`go get`したら次のエラーが出ました。

```
go get github.com/skanehira/clipboard-image@v2.0.0: github.com/skanehira/clipboard-image@v2.0.0: invalid version: module contains a go.mod file, so major version must be compatible: should be v0 or v1, not v2
```

`go modules`を使っているからなのか、少々面倒だったのでv1以上にする手順を残しておきます。

### 手順
以下通りにやれば、バージョンアップできるはずです。

1. ライブラリの`go.mod`に`/v2`をつける
   変更前
   ```
   module github.com/skanehira/clipboard-image
   ```
   変更後
   ```
   module github.com/skanehira/clipboard-image/v2
   ```

2. タグをつけてpush
   ```bash
   git tag v2.0.0
   git push origin --tags
   ```

3. 利用する側で`go get module/v2`
   ```bash
   go get github.com/skanehira/clipboard-image/v2
   ```

4. importにバージョンをつける
   ```go
   import (
   	"github.com/skanehira/clipboard-image/v2"
   )
   ```

### 解説
どうやら`go modules`を使っているライブラリをv1以上にする場合は、`module@2.0.0`ではなく`module/v2`というふうに`go.mod`に定義する必要があるようです。
詳細は参考文献の`go get の動作メモ`を参照ください。

### 参考文献
- [go get の動作メモ](https://www.kaoriya.net/blog/2020/06/13/)
- [Go Module Mirror, Index, and Checksum Database](https://proxy.golang.org) のFAQ
