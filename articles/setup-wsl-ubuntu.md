---
title: "WindowsでのWSL設定・Ubuntu導入(自分用メモ)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["windows","wsl2","ubuntu","環境構築"]
published: true
---

WSLの設定・Ubuntu導入までの手順メモ。

## WSLの導入

WSLはWindowsに同梱されてるが、もし壊れても[GitHub](https://github.com/microsoft/WSL/releases)から落とせる。

## .wslconfigとwsl.conf
WSLの挙動を定義するconfファイル。2種類ある。詳細は[MSサイト](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config#wslconfig)。

|ファイル|場所|意味|
|--|--|--|
|`.wslconfig`|Windows上の<br>`C:\Users\(ユーザ名)\.wslconfig`|WSL上の全distroに適用。**WSL2のみ**|
|`wsl.conf`|distro上の<br>`/etc/wsl.conf`|WSL上の特定のdistroに適用|

## .wslconfig
**WSL全体に影響するパラメータ**。
配置場所はWindowsの`C:\Users\(ユーザ名)\.wslconfig`。このファイルはデフォルトで存在しないので、必要なら手作業で作成。**記述にエラーがあると無視してデフォルト値でWSLが起動する**。

```:.wslconfig
[wsl2]
memory=4GB
processors=2
～～
```

### 主なWSLの設定

セクション ラベル: [wsl2]
|key|value|default|注|
|--|--|--|--|
|kernel|path|カーネルで提供される受信トレイ|カスタムLinuxカーネルのパス|
|memory|size|Windows合計メモリの50%|WSLに割り当てるメモリの量。最大値でWSL起動時この値が確保されるわけではない|
|processors|数値|Windows論理プロセッサと同数|WSLに割り当てる論理プロセッサ数|
|localhostForwarding|boolean|true|localhost:port経由でwin→WSL接続|
|kernelCommandLine|string|Blank|追加のカーネルコマンドライン引数|
|safeMode|boolean|false|WSLをセーフモードで実行|
|swap|size|Windowsメモリの25%|WSLに追加するスワップ領域、スワップファイルなければ0|
|swapFile|path|%USERPROFILE%\ <br>AppData\Local\ <br>Temp\swap.vhdx|スワップ仮想diskへのパス<br>デフォ値はMSサイトでは←だが、実際には`C:\Users\takak\AppData\Local\Temp\なんかごちゃごちゃしたところ\swap.vhdx`<br>**`swap.vhdx`検索、あとは更新日付で探す。このフォルダはゴミ溜まるので注意**|
|pageReporting|boolean|true|WindowsにWSLの未使用のメモリ再利用を許可|
|guiApplications|boolean|true|WSLでのGUI(WSLg)サポート|
|debugConsole|boolean|false|distro開始時dmesgのコンソール出す(win11)|
|nestedVirtualization|boolean|true|WSLで入れ子VMを許可(win11)|
|vmIdleTimeout|数値|60000|VMがアイドル状態になってからシャットダウンされるまでのミリ秒(win11)<br>[WSLではsystemdがインスタンスを維持しない](https://learn.microsoft.com/ja-jp/windows/wsl/systemd#how-does-enabling-systemd-affect-wsl-architecture)。DOS窓閉じたら1分後に落ちる、Windowsみ深い。0にしたら10秒くらいで落ちる|
|dnsProxy|boolean|true|`networkingMode=NAT`時ホストのNATに対してLinuxのDNSサーバーを構成。false時はWindowsからLinuxにDNSサーバミラーリング|
|networkingMode|string|NAT|`mirrored`でネットワークモードミラー化(win11/22H2～)|
|firewall|boolean|true|true時Winファイアウォール＋Hyper-V固有規則でフィルタ(win11/22H2～)|
|dnsTunneling|boolean|true|WSLからWindowsへのDNS要求のプロキシ方法を変更(win11/22H2～)|
|autoProxy|boolean|true|WSLにWindows HTTPプロキシ情報の使用を強制(win11)|
|defaultVhdSize|size|1TB|仮想HDサイズ<br>実際の`.vhdx`を見ると使った分だけ広がるみたい。たぶん最大値|

※`path`は\をエスケープ 例: `C:\\Temp\\myCustomKernel`
※`size`は単位指定 例: `8GB`、`512MB`

### 試験的な設定

[色々便利そう](https://learn.microsoft.com/ja-jp/windows/wsl/systemd#how-does-enabling-systemd-affect-wsl-architecture)。時々チェック。


## wsl.conf
配置場所はdistro上の`/etc/wsl.conf`。デフォルトでは存在しないので必要なら作成する。

```:wsl.conf
[section-label]
key = value
key = value
～～
```

### systemd サポート

多くのdistro(Ubuntu含む)は既定でsystemdが実行される。
セクション ラベル: [boot]
|key|value|default|内容|
|--|--|--|--|
|systemd|boolean|true|systemdを起動<br>(Ubuntuでは指定省略してもsystemdが上がるのを確認)|

### 自動マウントの設定

Win⇔WSL2はパフォーマンス悪いので注意。あまり使いたくない。

セクション ラベル: [automount]

|key|value|default|内容|
|--|--|--|--|
|enabled|boolean|true|trueで固定ドライブ (C:/とか)が/mnt下にマウント。falseでもfstabで個別にマウントできる|
|mountFsTab|boolean|true|WSL開始時に処理されるよう/etc/fstab(SMB=Server Message Block共有など他のファイル システムを宣言するもの)を設定|
|root|string|/mnt/|Win固定ドライブマウントのエントリポイント|
|options|以下参照|""|マウントしたWindowsファイルシステムに対するオプションの指定<br>`options = "metadata,uid=1003,gid=1003,umask=077,fmask=11,case=off"～`|

optionsの詳細

|key|内容|default|
|--|--|--|
|uid|uid|WSL既定(初回インス<br>トール時は1000)|
|gid|gid|〃|
|umask|全ファイル・ディレクトリに除外するアクセス許可のマスク|022|
|fmask|全ファイルに除外するアクセス許可のマスク|000|
|dmask|全ディレクトリに除外するアクセス許可のマスク|000|
|metadata|Windowsファイルシステムへのメタデータの追加|disabled|
|case|[ディレクトリの大文字小文字の許可関係](https://learn.microsoft.com/ja-jp/windows/wsl/case-sensitivity)|off|

### ネットワーク設定

セクション ラベル: [network]

|key|value|default|内容|
|--|--|--|--|
|generateHosts|boolean|true|/etc/hosts(IPアドレスに対応するホスト名の静的マップ)を生成|
|generateResolvConf|boolean|true|/etc/resolv.conf(ホスト名のDNSリスト)を生成|
|hostname|string|Windowsホスト名|WSL配布に使用するホスト名|

### 相互運用の設定

セクション ラベル: [interop]

|key|value|default|内容|
|--|--|--|--|
|enabled|boolean|true|WSLでWindowsプロセスの起動をサポート|
|appendWindowsPath|boolean|true|WSLがWindowsパス要素を$PATHに追加|

### ユーザー設定

セクション ラベル: [user]

|key|value|default|内容|
|--|--|--|--|
|default|string|初回作成時のユーザ名|WSL開始時に使用されるユーザー|

### ブート設定

**(Windows11 or Server2022のみ)**

セクション ラベル: [boot]

|key|value|default|内容|
|--|--|--|--|
|command|string|""|WSLインスタンス開始時の実行コマンド(rootで実行される。複数指定不可、その場合はシェルスクリプトを指定)<br>**command内のパスはWSL内の絶対パスで指定**|


## WSLアップデート・Ubuntu導入

Powershellから

```powershell
> wsl --update      #WSL2に更新
> wsl -l --online   #導入できるdistro確認
～～
NAME                            FRIENDLY NAME
Ubuntu                          Ubuntu
Debian                          Debian GNU/Linux
kali-linux                      Kali Linux Rolling
～

> wsl --install ubuntu #Ubuntuを導入
～
Enter new UNIX username:    #ユーザ名
New password:               #pw
Retype new password:        #pw
～
# Ubuntuが起動
```
```shell-session
$ cat /etc/os-release #OS確認
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
～

$ cat /proc/version #カーネル確認
$ #WSL専用品のようです
Linux version 5.15.167.4-microsoft-standard-WSL2 (root@f9c826d3017f) (gcc (GCC) 11.2.0, GNU ld (GNU Binutils) 2.37) #1 SMP Tue Nov 5 00:21:55 UTC 2024
$ exit
```
```powershell
> wsl -l --verbose #distro状態確認
  NAME      STATE           VERSION
* Ubuntu    Stopped         2       #Ubuntuが入っているのを確認
```

これでUbuntuの導入完了。
WSLまわりのコマンド：

```powershell
> wsl -u root -d Ubuntu     #rootでログイン
> wsl --terminate Ubuntu    #Ubuntuを終了
> wsl --set-default Ubuntu  #既定distroをUbuntuに(複数distro導入してるとき)
> wsl                       #既定distroに既定ユーザで入る
> wsl --shutdown            #全distroを終了
```

ちなみにWSL上のファイルはWindowsの
![](/images/setup-wsl-ubuntu/wsl_vhdx_locate.png)

`C:\Users\(ユーザ名)\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_～～\LocalState\ext4.vhdx`
またえらいところにいる。拡張子`.vhdx`はHyper-Vの仮想ストレージ。
試しにUbuntuを削除すると
```powershell
> wsl --uninstall ubuntu
```
![](/images/setup-wsl-ubuntu/wsl_vhdx_del_ubuntu.png)
**消えない。**
もう一度Ubuntuを導入しようとしたら

```powershell
> wsl --update
要求された操作には管理者特権が必要です。
ダウンロード中: Linux 用 Windows サブシステム 2.3.26
～
> wsl -l --verbose
  NAME      STATE           VERSION
* Ubuntu    Stopped         2        #消えてない
```
深く考えないことにした。

> UbuntuとWSLを完全にアンインストールする方法:
```powershell
> #Ubuntuをアンインストール
> wsl --uninstall ubuntu #導入済の全distroを落とす
> #残存リソースも削除
> Remove-Item -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Lxss" -Recurse
```
さらに、`C:\Users\(ユーザ名)\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_～～`を探してフォルダごと削除。これで完全に消える(はず)