---
title: "WSL - Linux関係メモ"
emoji: "✍️"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["bookmark"]
published: false
---
Linux関係、通常とWSL2-Ubuntuでの動作の違いも含めたメモ。

## あとで確認することメモ

[コンテナのゾンビプロセス](https://qiita.com/t_katsumura/items/ed105f1c139b24f7fe4f)
namespace

## コマンド引数の表記方法
|形式|表記|
|--|--|
|UNIX標準オプション|-Aa (先頭ハイフン付き、複数続けて表記)|
|BSDオプション|Aa (ハイフン無し、複数続けて表記)
|GNUオプション|--all (ハイフン2個, スペースで区切って表記)|

コマンドによっては歴史経緯から複数形式のオプションが混じっている(psなど)。

## リダイレクト

|種類|意味|
|--|--|
|\> hoge.txt|上書き|
|\>\> hoge.txt|追加|
|0\> hoge.txt, 0\>\> hoge.txt|標準入力|
|1\> hoge.txt, 1\>\> hoge.txt|標準出力|
|2\> hoge.txt, 2\>\> hoge.txt|標準エラー出力|
|>> hoge.txt 2\>&1|標準エラー出力と標準出力をまとめてリダイレクト|
|&\> hoge.txt<br>&\>> hoge.txt|まとめてリダイレクト|

```shell-session
$ echo $(date)
Tue Jan 7 18:26:55 JST 2025
$ echho $(date)                     #スペルミス
Command 'echho' not found, did you mean～
$ echo $(date) > hoge.txt           #標準出力をリダイレクト(上書)
$ cat hoge.txt                      #ファイルに出力されている
Tue Jan 7 18:27:27 JST 2025
$ echho $(date) >> hoge.txt         #標準出力をリダイレクト(追加) スペルミス
Command 'echho' not found, did you mean～
$ cat hoge.txt                      #エラーはリダイレクトされない
Tue Jan 7 18:27:27 JST 2025
$ echho $(date) 2>> hoge.txt        #エラー出力をリダイレクト スペルミス
$ cat hoge.txt                      #エラーが書かれた
Tue Jan 7 18:27:27 JST 2025
Command 'echho' not found, did you mean～
$ echho $(date) >> hoge.txt 2>&1    #エラー出力を標準出力に統合
$ cat hoge.txt                      #エラーが追記された
Tue Jan 7 18:27:27 JST 2025
Command 'echho' not found, did you mean～
Command 'echho' not found, did you mean～
$ echho $(date) &> hoge.txt         #この記法でも同意
$ cat hoge.txt                      #エラーが書かれた
Command 'echho' not found, did you mean～
$ echho $(date) &>> hoge.txt        #追記版
$ cat hoge.txt
Command 'echho' not found, did you mean～
Command 'echho' not found, did you mean～
```

## runlevel

システムの起動段階。

|runlevel|一般的な定義|`.target`ファイル<br>(WSL-Ubuntu)|
|--|--|--|
|0|シャットダウン|poweroff.target<br>※機能しない|
|1|シングルモード(root)|rescue.target|
|2|マルチユーザモード(ネットワークなし)|multi-user.target|
|3|マルチユーザモード(テキストログイン)|multi-user.target|
|4|-|multi-user.target|
|5|マルチユーザモード(グラフィカル)|graphical.target|
|6|再起動|reboot.target<br>※機能しない|

実装としては、sytemdが起動する`.target`に対応する。
ランレベルとターゲットの対応は`/lib/systemd/system/runlevelx.target`を参照(ターゲットファイルへのリンクになっている)。デフォルトのランレベルは`/lib/systemd/system/default.target`で決定される。
変更は`init x`(xはランレベル)。

```bash:WSL
$ #systemdがブートプロセスで参照するターゲットファイル
$ ls /lib/systemd/system -l
～
lrwxrwxrwx 1 root root   16 Aug  8 23:51  default.target -> graphical.target
～
lrwxrwxrwx 1 root root   15 Aug  8 23:51  runlevel0.target -> poweroff.target
lrwxrwxrwx 1 root root   13 Aug  8 23:51  runlevel1.target -> rescue.target
lrwxrwxrwx 1 root root   17 Aug  8 23:51  runlevel2.target -> multi-user.target
lrwxrwxrwx 1 root root   17 Aug  8 23:51  runlevel3.target -> multi-user.target
lrwxrwxrwx 1 root root   17 Aug  8 23:51  runlevel4.target -> multi-user.target
lrwxrwxrwx 1 root root   16 Aug  8 23:51  runlevel5.target -> graphical.target
lrwxrwxrwx 1 root root   13 Aug  8 23:51  runlevel6.target -> reboot.target
～
$ #デフォルトで起動するgraphical.targetの内容
$ cat /lib/systemd/system/graphical.target
#  SPDX-License-Identifier: LGPL-2.1-or-later
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Graphical Interface
Documentation=man:systemd.special(7)
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

## プロセス

### pid

プロセス毎に振られる番号。PID 1はsystemdが利用。
```shell-session:WSL
$ ps -A
    PID TTY          TIME CMD
      1 ?        00:00:01 systemd
      2 ?        00:00:00 init-systemd(Ub
      8 ?        00:00:00 init
    ～～
```

:::message alert
WSLは[wsl.confで設定](https://zenn.dev/tabirider/articles/setup-wsl-ubuntu#systemd-%E3%82%B5%E3%83%9D%E3%83%BC%E3%83%88%E3%83%BB%E3%82%B3%E3%83%9E%E3%83%B3%E3%83%89%E5%AE%9F%E8%A1%8C)しないとsystemdが上がらない。最近Ubuntuなどは[デフォでsystemdが有効化](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config#systemd-support)されてる。
:::

### psコマンド

ややこしすぎる。歴史が長いので引数表記がごちゃまぜ。
psコマンドのオプション(使いそうなもの。全部確認するときはps --help all, man ps)
|option|意味|
|--|--|
|(省略)|自身が起動しているプロセス|
|-A, -e|全プロセス|
|a|端末を持つプロセス|
|x|端末を持たないプロセス|
|r|実行中のプロセス|
|T|このTTYの全プロセス|
|x|TTYで制御されていないプロセス|
|-C コマンド|コマンド名の指定(複数の場合は,で区切る)<br>`ps -C bash`|
|-s|セッションIDを表示|
|T|端末の全プロセス|
|-u ユーザ名<br>-g グループ名|実効ユーザ/グループを指定して表示|
|-U ユーザ名<br>-G グループ名|実ユーザ/グループを指定して表示|
|s|シンプルな表示|
|-l|長いフォーマットで表示|
|u|ユーザが読みやすいフォーマットで表示|
|e|環境変数を表示|
|f|階層表示|
|o 項目名|出力項目を,で区切って指定<br>pid:プロセスID<br>ppid:親プロセス<br>s, stat:状態<br>pcpu:CPU使用率<br>pmem:メモリ使用量<br>time:CPU累積使用時間<br>nice:nice値(プロセスの優先順位)<br>args:コマンド名(引数付)<br>comm:コマンド名<br>tt:端末(TTY)<br>uid/user:実効ユーザID/ユーザ名<br>gid/group:実効グループID/グループ名<br>ruid/ruser:実ユーザID/ユーザ名<br>rgid/rgroup:実グループ/グループ名|
|w|出力時の幅を広げる|
|--cols 文字数|出力時の幅を指定|
|--rows 行数|出力時の行数を指定|
|k, --sort 項目名|並べ替え(項目名はoを参照。項目名の前に+-)|

> 実効ユーザ/グループは動作時の権限。実ユーザ/グループはプロセスを起動したユーザ。通常両者は一致する。

### statの状態表示

|表示|意味|
|--|--|
|D|スリープ(割込不可、だいたいIO中)
|I|アイドル
|R|実行中or実行待ち
|S|スリープ(割込可能)
|T|停止中(SIGCONTシグナルで再開可能)
|t|デバッガが停止中
|W|未使用(かつてはページング中でスワップアウトの意味)
|X|終了(普通はない)
|Z|ゾンビ(終了しているが親プロセスが終了ステータスを受け取っていない状態)

付加記号

|表示|意味|
|--|--|
|<|優先(nice値低い)|
|N|劣後(nice値高い)|
|L|メモリページをロック中(リアルタイム処理、カスタムIO)|
|s|他のセッションを作った人(セッションリーダー)|
|l|マルチスレッド|
|+|フォアグラウンドプロセスグループ(端末接続されてて入力を受けることができる状態)|

### プロセス一覧の自動更新
```shell-session
$ #3秒毎に更新してくれる
$ watch -n3 'ps -A u --sort -pmem'
```

## systemd

[SUSE](https://documentation.suse.com/ja-jp/sle-micro/6.0/html/Micro-systemd-basics/)から。distro違うので差異があるかも。

|用語|意味|
|--|--|
|systemd|システム・サービスマネージャ。`/sbin/init`でインストールされブート中に起動。SID 1(init)プロセスで実行される|
|ユニット|システムが認識するリソース。ユニットファイルで定義|
|ターゲット|ブート中に同期ポイントとして機能する関連ユニットのグループ。ターゲットのユニットは依存関係連鎖を通じて各systemdユニットをまとめたもの。拡張子は`.target`。デフォルトターゲットは`/lib/systed/system/default.target`|
|systemctl|initシステムを制御する管理ツール|

### systemdのブートプロセス
1.  Linuxブートプロセス
    - ステージ1: BIOS起動、POST(Power On Self Test)実行
        - WSLにはない(?)
    - ステージ2: ブートローダ
        MBR(マスターブートレコード, 512byte。通常は`/dev/sda`または`/dev/hda`に配置)のブートローダプログラムをロード
        - WSLでは`/dev/sda`がある
        - ブートローダの種類はLILO, GRUB, GRUB2。GRUB2が最新、`/boot/grub2/grub2.cfg`
        - WSLでは`/boot`は空。ここは省略されてるみたい
    - ステージ3: Linuxカーネル初期化
        ブートローダがカーネルをロード。その後`/sbin/init`を実行
        - WSLでは`/sbin/init`は`/lib/systemd/systemd`へのリンク
    - ステージ4: systemd
        カーネルがsystemdをロード
2.  systemdによるブートプロセス
    systemdはシステムに必要なサービス(ネットワーク, ログインマネージャ等)を起動。
    ターゲットユニットは実行順に並列化。
    - WSLではデフォルトでsystemdが有効化されている。WSLのinitプロセスはsystemdの子プロセスとして動作
    - **WSLのsystemdはWSLインスタンスを維持しない**。このため全TTYを落とすとデフォルトで60秒後にWSLが停止する。
3.  systemd-analyzeコマンドによるブートプロセスのパフォーマンス分析
    - 起動時間の確認
        ```shell-session
        $ systemd-analyze time
        Startup finished in 1.946s (userspace)
        graphical.target reached after 1.941s in userspace.
        ```
    - WSLでのブートプロセス
        ```shell-session
        $ systemd-analyze critical-chain
        graphical.target @1.941s
        └─multi-user.target @1.941s
        └─docker.service @637ms +1.302s         #dockerデーモン
            └─containerd.service @448ms +188ms  #containerd
            └─dbus.service @424ms +20ms
                └─basic.target @423ms
                └─sockets.target @423ms
                    └─snapd.socket @422ms +746us
                    └─sysinit.target @417ms
                        └─systemd-resolved.service @348ms +68ms
                        └─systemd-tmpfiles-setup.service @335ms +10ms
                            └─systemd-journal-flush.service @235ms +98ms
                            └─systemd-journald.service @203ms +24ms
                                └─systemd-journald.socket @196ms #
                                └─-.mount @190ms
                                    └─-.slice @190ms
        ```
    - プートプロセスで開始されたサービスリスト
        ```shell-session
        $ systemd-analyze blame
        32.696s motd-news.service
        6.498s apt-daily.service
        1.302s docker.service
        872ms apt-daily-upgrade.service
        ～～
        ```

### ユニットファイルの構造

```
[Section]
    Directive1=value
    Directive2=value
    ～～
```
### ユニットファイルのタイプ

`/lib/systemd/system`を覗けばふいんきはわかる

|拡張子|内容|
|--|--|
|.service|サービス・アプリの管理方法。開始・停止方法、自動開始条件、関連ユニットの依存関係等|
|.scope|systemdによってD-Busから受信した情報から自動作成。外部で作成されたシステムプロセスを管理|
|.path|パスを使用した有効化で使用するパスを定義。デフォルトでは「同一ファイル名.service」が有効。inotifyはファイル変更通知を受け取るプログラムで使用するカーネルAPI|
|.snapshot|systemctl snapshotコマンドで自動作成。システムの現在の状態を表すスナップショット。一時的な状態のロールバックに使える|
|.timer|systemdで管理するタイマ。設定値に達すると「同一ファイル名.service」が起動|
|.slice|スライス※毎に関連プロセスのリソース割当・制限ができる|
|.target|ブート・init時の状態を定義|
|.socket|ソケットベースの有効化でsystemdが使用するネットワーク、IPCソケット、FIFOバッファ|
|.device|udev, sysfsでsystemdによる管理が指示されているデバイス|
|.swap|システムのスワップ空間|
|.mount|systemdで管理するシステムのマウントポイント。ファイル名の`-`を`/`に読み替える。/etc/fstabのエントリには自動作成されたユニットを指定できる|
|.automount|自動マウントされるマウントポイント。ファイル名の読み替えは`.mount`と同じ。「同一ファイル名.mount」が必要|

> cgroupとslice
  プロセスをグループ化して一括でリソース管理とか殺したりできる。[cgroupファイルシステム](https://gihyo.jp/admin/serial/01/linux_containers/0003)で操作。コンテナ機能の根幹。
  ```shell-session
  $ #グループツリー。.sliceで分離されてるのが分かる。
  $ systemd-cgls
  CGroup /:
  -.slice
  ├─user.slice
  │ └─user-1000.slice
  │   ├─user@1000.service …
  │   │ └─init.scope
  ～～
  ├─init.scope
  │ ├─   1 /sbin/init
  │ ├─   2 /init
  ～～
  └─system.slice
  ├─containerd.service …
  │ └─821 /usr/bin/containerd
  ├─systemd-udevd.service …
  ```
  :::message
  cgroupとcgroupファイルシステムは後で追記
  :::

## ファイル関連

### ファイルタイプ・アクセス権限

```shell-session
$ ls -l
-rwxr-xr-x 1 root root      2217 Apr  7  2024 applygnupgdefaults
-rwxr-xr-x 1 root root     26960 Mar 31  2024 arpd
lrwxrwxrwx 1 root root        27 Apr  9  2024 arptables -> /etc/alternatives/arptables
～～
```
#### ファイルタイプ(先頭)
|記号|ファイルタイプ|
|--|--|
|l|シンボリックリンク|
|d|ディレクトリ|
|c|キャラクタデバイス|
|b|ブロックデバイス|
|s|ソケット|

#### アクセス権限
```shell
  rwx r-x r-x
#  |   |   他ユーザ
#  |   所有グループ
#  所有者
```
|モードビット|値|意味|
|--|--|--|
|--x|1|実行|
|-w-|2|更新|
|r--|4|読込|

### シンボリックリンクとハードリング

|特徴|ハードリンク|シンボリックリンク|
|--|--|--|
|実体|iノードを増やす|ファイルパス|
|リンク対象|同一バーティション内|パスを参照できればOK|
|元ファイル削除時|参照できる(ファイル本体はハード<br>リンクが残っていれば消えない)|壊れたリンクになる|
|ディレクトリへのリンク|通常は不可|可能|
|作成コマンド|ln TARGET LINK_NAME|ln -s TARGET LINK_NAME|

```shell-session
$ touch hoge.txt            # hogeを作成
$ ln hoge.txt piyo.txt      # hogeへのハードリンクを作成
$ ls -l
total 0
-rw-r--r-- 2 takak takak 0 Jan  7 06:39 hoge.txt
-rw-r--r-- 2 takak takak 0 Jan  7 06:39 piyo.txt
$         #↑ハードリンクの数
$ ln -s hoge.txt fuga.txt   # シンボリックリンクを作成
$ ls -l
total 8
lrwxrwxrwx 1 takak takak 8 Jan  7 06:41 fuga.txt -> hoge.txt
-rw-r--r-- 2 takak takak 5 Jan  7 06:40 hoge.txt
-rw-r--r-- 2 takak takak 5 Jan  7 06:40 piyo.txt
```

### ディスクブロック

```shell-session
$ ls -ls
total 12
$     #↑ディスクブロックの数(サブディレクトリ・ファイルも含む)
0 lrwxrwxrwx 1 takak takak    8 Jan  7 06:41 fuga.txt -> hoge.txt
4 -rw-r--r-- 2 takak takak    5 Jan  7 06:40 hoge.txt
$ #先頭の数字がブロックサイズ
$ #ブロックサイズの確認
$ stat --printf="%s bytes per block\n" .
4096 bytes per block
$ #↑WSLではブロックサイズがけっこう大きいもよう
```
