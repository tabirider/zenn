---
title: "ZennとGitHub連携の手順メモ(テスト中)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["windows", "zenn", "git", "github", "zenncli"]
published: false
---
Zenn公式にはGitの詳細に関する記載がないので、それも含めてWindows環境でのZennとGitHubの連携手順メモ。GitHub普段使わない人(私)、WindowsでGit使わん人用。

Markdown記法のファイル(.md)を書いてGitHubに投げれば、自動でZennのサイトに掲載される便利機能。
## メリット
- ブラウザのエディタから開放される。好きなエディタでサッと書ける
- GitHubに版管理を丸投げできる
- Zenn CLIを導入すればlocalhost:8000でプレビューできる(便利)
## デメリット
- Zenn CLIはPC毎にセットアップが必要
- Node.jsのインストールが必要(v14以上)。毎年2回バージョン上がるので、人によっては環境競合するかも
- Gitにせよ何にせよWindowsで使うのは~~クソ~~面倒

:::message
手順は変わることもあります。基本は[公式](https://zenn.dev/zenn/articles/connect-to-github)を参照ください。
:::

## 手順
前提: Win11環境です。
1. Zennのアカウント作成
[Zennのサイト](https://zenn.dev/)でアカウント作成。
2. GitHubのアカウント作成
[GitHubのサイト](https://github.co.jp/)でアカウント作成。まずは無償版でOK。
3. GitHubでリポジトリ作成
   GitHubのDashboardからTop repositories→Newでリポジトリ名を入力。
   PublicでもPrivateでもOK。READMEファイル→不要、.gitignore→None、licence→NoneでOK。
4. ZennとGitHubの連携
   [公式サイト](https://zenn.dev/zenn/articles/connect-to-github)参照
5. Gitのインストール
   [Gitのサイト](https://git-scm.com/downloads)からダウンロード
   またはPowerShellから
   ```powershell
   winget install --id Git.Git -e --source winget
   ```
   パスが通ってるか確認
   PowerShellから
   ```powershell
   git --version
   ```
   エラー出たらパス通ってないのでGitの場所を確認して環境変数PATHに追加
   ```powershell
   # エクスプローラかなんかでGitの場所を探してパスに追加。
   # (場所はだいたいこのへん)
   $env:PATH="$($env:PATH);C:\Program Files\Git\cmd"
   # ↑の$()はサブエクスプレッション。"$env:PATH;C:\Program Files～"と
   # 書いても問題ないが、PowerShellで文字列中に変数展開するときは$()を使うように
   # 癖つけといたほうがいい。PowerShellは面倒なんで深入りしない
   # ただし、$env:PATHはセッション内限定なので消えてしまう。永続化する場合は
   # ユーザ毎の設定:
   [System.Environment]::SetEnvironmentVariable("PATH", "$($env:PATH);C:\Program Files\Git\cmd", [System.EnvironmentVariableTarget]::User)
   # PC全体の設定(管理者権限必要):
   [System.Environment]::SetEnvironmentVariable("PATH", "$($env:PATH);C:\Program Files\Git\cmd", [System.EnvironmentVariableTarget]::Machine)
   # なんでパス通すのにこんなことなんねん...ほんまMSさん訳分からんで
   ```
   もしくはスタート→設定(歯車マーク)→システム→バージョン情報→「デバイスの仕様」の下にくっついてる「システムの詳細設定」→詳細設定タブ→環境変数→システムの環境変数からPathをダブルクリックして「新規」からGitのパスを追加(どっちにしても~~クソ~~面倒)
6. Gitの設定
   ```powershell
   git config --global user.name "ユーザ名"       #お決まり
   git config --global user.email "mail@address" #お決まり
   git config --global init.defaultBranch main #masterじゃなくてmainにしようね
   ```
   configのリスト確認方法：
   ```powershell
   git config list
   ```
   ↑でメールアドレスとか設定されてるのを確認
   ちなみにWindows版でGit.configファイルの場所は以下の通り。
   ```powershell
   C:\Program Files\Git\config    # デフォルト
   C:\Users\(ユーザ名)\.gitconfig  # ユーザ毎
   repository/.git/config         # リポジトリ固有
   ```
7. GitHubにSSHキー登録
   ```powershell
   ssh-keygen -t rsa -b 4096 -C "mail@address"
   ```
   パスフレーズを要求されるので入力。これで秘密鍵と公開鍵ができるので公開鍵をGitHubに登録。
   公開鍵の場所は`C:\Users\(ユーザ名)\.ssh\id_rsa.pub`
   エクスプローラだとPC→Windows(C:)→ユーザー→(ユーザ名)→.sshで辿れる。
   ファイルを適当なテキストエディタで開いて、中身の`ssh-rsa ～～～`のテキストを丸ごとコピー
   GitHubのサイトから右上のアイコン→Settings→SSH and GPG keys→New SSH keyでKeyにコピーしたテキストをそのまま貼り付け→Add SSH key

8. Node.jsをインストール
   [公式サイト](https://nodejs.org/)からダウンロード


9. SSHエージェントの使用
   パスフレーズを毎回入力するのは面倒なので、SSHエージェントを使う。SSHエージェントはパスフレーズをディスクではなくメモリに保持するのでセキュリティ高め。Windowsでも10以降はOpenSSHクライアントが標準で入っているので、以下手順で設定しておくと便利。
   1. .ssh/configファイルを作成
      `C:\Users\(ユーザ名)\.ssh`フォルダに`config`ファイルを作成(拡張子なし)
    ```powershell
    Host github.com
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_rsa
        AddKeysToAgent yes
    ```
    2. OpenSSH Authentication Agentを起動
       Windowsのスタート→Windowsツール→サービスを開く
       "OpenSSH Authentication Agent"を選択し、「スタートアップの種類」を「自動」に
       さらに「開始」でサービスを立ち上げる
    3. この状態でもっかいてすと
