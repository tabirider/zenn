---
title: "WSL2・Ubuntu"
---

:::message
`PowerShell`と`WSL-Ubuntu`が混じるので、

```powershell:PowerShell
> #これはPowerShell
```

```shell-session:WSL
$ #これはWSL(Ubuntu)
```

```dockerfile:wsl.conf
# ↑ファイルの内容はファイル名
```
:::

## WSLの導入

WSLはWindowsに同梱されてるが、もし壊れても[GitHub](https://github.com/microsoft/WSL/releases)から落とせる。

## .wslconfigとwsl.conf
WSLの挙動を定義するconfファイル。2種類ある。詳細は[MSサイト](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config)。

|ファイル|場所|意味|
|--|--|--|
|`.wslconfig`|Windows上の<br>`C:\Users\(ユーザ名)\.wslconfig`|WSL上の全distroに適用。**WSL2のみ**|
|`wsl.conf`|distro上の<br>`/etc/wsl.conf`|WSL上の特定のdistroに適用|

## .wslconfig
**WSL全体に影響するパラメータ**。
配置場所はWindowsの`C:\Users\(ユーザ名)\.wslconfig`。このファイルはデフォルトで存在しないので、必要なら手作業で作成。**記述ミスは無視してデフォルト値でWSLが起動する**。Windowsかよ。

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
|swapFile|path|%USERPROFILE%\ <br>AppData\Local\ <br>Temp\swap.vhdx|スワップ仮想diskへのパス<br>デフォはMSサイトの説明では←だが実際には`C:\Users\takak\AppData\Local\Temp\なんかごちゃごちゃしたところ\swap.vhdx`<br>通常は`wsl --shutdown`で削除されるが時々ゴミが残る。場所は`Temp`内を`swap.vhdx`で検索するのが早い|
|pageReporting|boolean|true|WindowsにWSL未使用メモリの再利用を許可|
|guiApplications|boolean|true|WSLでのGUI(WSLg)サポート|
|debugConsole|boolean|false|distro開始時dmesgのコンソール出す(win11)。<br>**VSCodeをWSLに接続した状態で`wsl --shutdown`するとVSCodeが勝手にWSLを再起動し、その際`~/.bashrc`が走らず想定外の挙動になったりする**。trueにしておくとコンソールが出てWSL起動が見えるので、**true設定がおすすめ**。|
|nestedVirtualization|boolean|true|WSLで入れ子VMを許可(win11)|
|vmIdleTimeout|数値|60000|VMがアイドル状態になってからシャットダウンされるまでのミリ秒(win11)<br>**[WSLではsystemdがインスタンスを維持しない](https://learn.microsoft.com/ja-jp/windows/wsl/systemd#how-does-enabling-systemd-affect-wsl-architecture)から、シェルから抜けたら1分後にdistro自体が落ちる**。0にしたら10秒くらいで落ちる、無制限の設定はできないっぽい。<br>WSLを上げたPoserShellのウィンドウを閉じなければ落ちないので、特に変更する必要はなさそう。|
|dnsProxy|boolean|true|`networkingMode=NAT`時ホストのNATに対してLinuxのDNSサーバーを構成。false時はWindowsからLinuxにDNSサーバミラーリング|
|networkingMode|string|NAT|`mirrored`でネットワークモードミラー化(win11/22H2～)|
|firewall|boolean|true|true時Winファイアウォール＋Hyper-V固有規則でフィルタ(win11/22H2～)|
|dnsTunneling|boolean|true|WSLからWindowsへのDNS要求のプロキシ方法を変更(win11/22H2～)|
|autoProxy|boolean|true|WSLにWindows HTTPプロキシ情報の使用を強制(win11)|
|defaultVhdSize|size|1TB|仮想HDサイズ<br>実際の`.vhdx`を見ると使った分だけ広がるみたい。たぶん最大値|

※`path`は\をエスケープ 例: `C:\\Temp\\myCustomKernel`
※`size`は単位指定 例: `8GB`、`512MB`

### 試験的な設定

[色々便利そう](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config#experimental-settings)。時々チェック。


## wsl.conf
配置場所はdistro上の`/etc/wsl.conf`。デフォルトでは存在しないので必要なら作成する。

```:wsl.conf
[section-label]
key = value
key = value
～～
```

### systemd サポート・コマンド実行

セクション ラベル: [boot]

|key|value|default|内容|
|--|--|--|--|
|systemd|boolean|distroによる|trueで[systemdを起動](https://learn.microsoft.com/ja-jp/windows/wsl/wsl-config#systemd-support)。これを設定しないとsystemdは有効にならない。[Ubuntuではsystemdの起動がデフォルトになった](https://learn.microsoft.com/ja-jp/windows/wsl/systemd#how-to-enable-systemd)らしい|
|command|string|""|WSLインスタンス開始時の実行コマンド(rootで実行される。複数指定不可、その場合はシェルスクリプトを指定。win11)<br>`command = /path/to/script.sh`|


### 自動マウントの設定

Win⇔WSL2はパフォーマンス悪いので注意。あまり使いたくない。

セクション ラベル: [automount]

|key|value|default|内容|
|--|--|--|--|
|enabled|boolean|true|trueで固定ドライブ (C:/とか)が/mnt下にマウント。falseでもfstabで個別にマウントできる。これをfalseにすると**VS CodeからWSL拡張機能で接続できなくなる**|
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
|case|[ディレクトリの大文字小文字の許可関係](https://learn.microsoft.com/ja-jp/windows/wsl/case-sensitivity)。これもwinとlinuxで扱い違うからWSLでwinファイルは扱いたくない。[フォルダ毎にケースセンシティブ設定](https://learn.microsoft.com/ja-jp/windows/wsl/case-sensitivity#modify-case-sensitivity)までできるもよう|off|

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


## WSLアップデート・Ubuntu導入

Powershellから

```powershell:PowerShell
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

ログインしたら、カレントディレクトリがWindowsドキュメントフォルダになってる。気持ち悪いけどこの挙動を変える方法は見当たらない。とりあえずホームに逃げる。
```shell-session:WSL
name@machine:/mnt/c/Users/私の 名前$ cd ~
$ cat /etc/os-release #OS確認
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
～

$ cat /proc/version #カーネル確認
$ #WSL専用品のようです
Linux version 5.15.167.4-microsoft-standard-WSL2 (root@f9c826d3017f) (gcc (GCC) 11.2.0, GNU ld (GNU Binutils) 2.37) #1 SMP Tue Nov 5 00:21:55 UTC 2024
$ #パッケージ最新化
$ sudo apt update
$ sudo apt upgrade
$ #終了
$ exit
```
```powershell:PowerShell
> wsl -l --verbose #distro状態確認
  NAME      STATE           VERSION
* Ubuntu    Stopped         2       #Ubuntuが入っているのを確認
```

これでUbuntuの導入完了。
WSLまわりのコマンド：

```powershell:PowerShell
> wsl -l --online           #導入可能なdistroのリスト
> wsl -l --verbose          #distroの実行状態を確認
> wsl                       #既定distroに既定ユーザで入る
> wsl -d ubuntu             #Ubuntuに既定ユーザで入る
> wsl -u root -d Ubuntu     #Ubuntuにrootで入る
> wsl --terminate Ubuntu    #Ubuntuを終了
> wsl --set-default Ubuntu  #既定distroをUbuntuに(複数distro導入してるとき)
> wsl --shutdown            #全distroを終了
```

なお`wsl --shutdown`しても、`Visual Studio Code`に`WSL`や`Dev Containers`拡張機能を入れてたりすると**勝手にWSLが再起動**する。`Dev Containers`に至っては拡張機能を有効にした瞬間にWSLを上げるようになってる。そういうの要らないんですけど。

ちなみにWSL上のファイルはWindowsの
![](/images/tr-windows-ve/wsl_vhdx_locate.png)

`C:\Users\(ユーザ名)\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_～～\LocalState\ext4.vhdx`
またえらいところにいる。CanonicalはUbuntuの開発元。拡張子`.vhdx`はHyper-Vの仮想ストレージ。
試しにUbuntuを削除すると
```powershell
> wsl --uninstall ubuntu
```
![](/images/tr-windows-ve/wsl_vhdx_del_ubuntu.png)
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
> さらに、`C:\Users\(ユーザ名)\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_～～`を探してフォルダごと削除。これでアンインストール完了(のはず)

## WSLのsystemd有効化

[公式](https://learn.microsoft.com/ja-jp/windows/wsl/systemd#how-to-enable-systemd)だと`Ubuntu`は`systemd`がデフォルトで有効になったらしい。念のため確認。
`systemd`が動いてないと[systemctl](https://manpages.ubuntu.com/manpages/oracular/en/man1/systemctl.1.html)も[microk8s](https://microk8s.io/)も動かない。[systemdの詳細は公式](https://learn.microsoft.com/ja-jp/windows/wsl/systemd)。

wsl.confの内容を確認。Ubuntu以外のdistroだとファイル自体がなかったりする。

```shell-session:WSL
$ #wsl.confが以下のように設定されていればOK
$ cat /etc/wsl.conf
[boot]
systemd=true
$ #systemdがPID 1で上がってることを確認
$ ps -A | grep systemd
      1 ?        00:00:01 systemd
      2 ?        00:00:00 init-systemd(Ub
～
```

systemdが上がっていなければ、wsl.confファイルを作成して`systemd=true`追記。

```shell-session
$ sudo nano /etc/wsl.conf
```

```
[boot]
systemd=true
```

`Ctrl`+`O` → `Enter` → `Ctrl` + `X`で保存して終了。いったんWSLを抜けて再起動する。

```shell-session:WSL
$ exit
```

```powershell:PowerShell
> wsl --shutdown #WSLの全distroを停止
> wsl -l --verbose #distroの稼動状態
  NAME      STATE           VERSION
* Ubuntu    Stopped         2　       #止まってる
> wsl -d ubuntu #再起動
```

systemdの起動を確認できればOK。