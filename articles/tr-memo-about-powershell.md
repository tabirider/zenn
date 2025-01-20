---
title: "PowerShellリファレンス的な(自分用メモ)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["powershell"]
published: false
---

## ps1実行できねぇ

`.ps1`をダブルクリックしても起動しない。ファイルの中身開くだけ。
じゃぁPowerShellから実行しようとすると
```powreshell
> hoge.ps1
hoge.ps1 : 用語 'hoge.ps1' は、コマンドレット、関数、スクリプト ファイル、または操作可能なプログラムの名前として認識されません。～～
```
PowerShellではカレントディレクトリがパスに入ってないので
```
> .\hoge.ps1
```
これならOK
