---
title: "ZennとGitHub連携の手順メモ"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["windows", "zenn", "git", "github", "zenncli"]
published: false
---
Zenn公式にはGitの詳細に関する記載がないので、それも含めてWindows環境でのZennとGitHubの連携手順メモ。普段GitHub使わない人(私)、WindowsでGit使わん人用。
Markdown記法のファイル(.md)を書いてGitHubに投げれば、自動でZennのサイトに掲載される。ここではテキストエディタにVisual Studio Codeを使い、そこからGitHubに投げるまでをまとめます。

## メリット
- ブラウザのエディタから開放される。好きなエディタでサッと書ける
- GitHubに版管理を丸投げできる
- Zenn CLIを導入すればlocalhost:8000でプレビューできる(便利)
## デメリット
- Zenn CLI、Git、VSCode諸々PC毎にセットアップ必要
- Node.jsのインストール必要(23年12月時点でv14以上)。毎年2回バージョン上がるので、人によっては環境競合するかも
- Gitにせよ何にせよWindowsで使うのは~~クソ~~面倒

:::message
手順は変わることもあります。基本は[公式](https://zenn.dev/zenn/articles/connect-to-github)を参照ください。
:::

## 手順
前提: Win11環境です。
> (お約束)エクスプローラのオプションから「登録されている拡張子は表示しない」チェック外す、「隠しファイル、～を表示」チェック

1. Zennのアカウント作成
    [Zennのサイト](https://zenn.dev/)でアカウント作成。

2. GitHubのアカウント作成
    [GitHubのサイト](https://github.co.jp/)でアカウント作成。まずは無償版でOK。

3. GitHubでリポジトリ作成
    GitHubの`Dashboard`から`Top repositories`→`New`でリポジトリ名を入力。
    `Public`でも`Private`でもOK。`README`ファイル→不要、`.gitignore`→`None`、`licence`→`None`でOK。

4. ZennとGitHubの連携
    [公式](https://zenn.dev/zenn/articles/connect-to-github)参照

5. Gitのインストール
    [Gitのサイト](https://git-scm.com/downloads)からダウンロード
    またはPowerShellから
    ```powershell
    > winget install --id Git.Git -e --source winget
    ```
    パスが通ってるか確認
    PowerShellから
    ```powershell
    > git --version
    ```
    バージョン出たらOKなのでGitの設定に進む。
    エラー出たらパス通ってないのでGitの場所を確認して環境変数PATHに追加
    ```powershell
    # エクスプローラかなんかでGitの場所を探してパスに追加。
    # (場所はだいたいこのへん)
    > $env:PATH="$($env:PATH);C:\Program Files\Git\cmd"
    # ただし、$env:PATHはセッション内限定なので消えてしまう。永続化する場合は
    # ユーザ毎の設定:
    > [System.Environment]::SetEnvironmentVariable("PATH", "$($env:PATH);C:\Program Files\Git\cmd", [System.EnvironmentVariableTarget]::User)
    # PC全体の設定(管理者権限必要):
    > [System.Environment]::SetEnvironmentVariable("PATH", "$($env:PATH);C:\Program Files\Git\cmd", [System.EnvironmentVariableTarget]::Machine)
    # なんでパス通すのにこんなことなんねん...MSさん訳分からんで
    ```
    または画面からPATH書き換える。(これほんま場所よく忘れる)
    スタート→設定(歯車マーク)
    ![](/images/setup-zenn-github/setting-win-path-01.png)
    →システム→バージョン情報
    ![](/images/setup-zenn-github/setting-win-path-02.png)
    →「デバイスの仕様」の下にくっついてる「システムの詳細設定」
    ![](/images/setup-zenn-github/setting-win-path-03.png)
    →詳細設定タブ→環境変数
    ![](/images/setup-zenn-github/setting-win-path-04.png)
    →ユーザかシステムの環境変数からPathを選択して「編集」
    ![](/images/setup-zenn-github/setting-win-path-05.png)
    →「新規」からGitのパスを追加
    ![](/images/setup-zenn-github/setting-win-path-06.png)
    (なんにしても~~クソ~~面倒)

6. Gitの設定(最初だけ)
    ```powershell
    > git config --global user.name "ユーザ名"       #お決まり
    > git config --global user.email "mail@address" #お決まり
    > git config --global init.defaultBranch main #masterじゃなくてmainにしようね
    ```
    configのリスト確認方法：
    ```powershell
    > git config list
    ```
    ↑でメールアドレスとか設定されてるのを確認
    ちなみにWindows版でGit.configファイルの場所は以下の通り。
    ```powershell
    C:\Program Files\Git\config    # デフォルト
    C:\Users\(ユーザ名)\.gitconfig  # ユーザ毎
    (リポジトリフォルダ)\.git/config # リポジトリ固有
    ```

7. GitHubにSSHキー登録
    まず鍵作る。
    ```powershell
    > ssh-keygen -t rsa -b 4096 -C "mail@address"
    ```
    パスフレーズを要求されるので入力(パスフレーズは省略できるが非推奨)。これで秘密鍵と公開鍵ができるので公開鍵をGitHubに登録。
    公開鍵の場所は
    ```powershell
    C:\Users\(ユーザ名)\.ssh\id_rsa.pub
    ```
    エクスプローラだとPC→Windows(C:)→ユーザー→(ユーザ名)→.sshで辿れる。
    `id_rsa.pub`ファイルを適当なテキストエディタで開いて、「中身の`ssh-rsa ～～～`のテキスト」を丸ごとコピー。(テキストエディタを入れてなければ「メモ帳」を開いて`id_rsa.pub`ファイルをドラッグ&ドロップすればOK)
    GitHubのサイトから右上のアイコン→`Settings`→`SSH and GPG keys`→`New SSH key`で`Key`にコピーしたテキストをそのまま貼り付け→`Add SSH key`

8. Visual Studio Codeをインストール
    [公式](https://azure.microsoft.com/ja-jp/products/visual-studio-code)からダウンロード。
    起動したら左側の`拡張機能`から`Japanese Language Pack for Visual Studio Code`、`GitHub Pull Requests`、`Markdown All in One`あたりをダウンロードしておく。

9.  **Zennで管理するフォルダ**を作成。このフォルダは同時に**Gitのローカルリポジトリ**にもなる。(ここでは`D:\Zenn`とする。エクスプローラで作ってもOK)
    ```powershell
    > mkdir D:\Zenn
    # ちなみにmkdirはPowerShellのコマンドレットNew-Itemのエイリアス。
    # PowerShell的に書くと
    > New-Item -Path "D:\Zenn" -ItemType Directory
    # になるらしい。無理やり古のDOS窓に似た動きにしてくれてるだけ。
    # インターネッツ老人会のオッサンは非常に混乱する。
    ```
    ![](/images/setup-zenn-github/setting-win-path-07.png)

10. Node.jsをインストール
    [公式サイト](https://nodejs.org/)参照

11. Zenn CLIをインストール
    [公式サイト](https://zenn.dev/zenn/articles/install-zenn-cli)参照。このとき、**作成済のZenn用フォルダ**で実行する。
    ```powershell
    > cd D:\Zenn\
    > npm init --yes # プロジェクトをデフォルト設定で初期化
    > npm install zenn-cli # zenn-cliを導入
    > npx zenn init
    ```
    これで`D:\Zenn\articles`フォルダが作成される。

12. Gitのローカルリポジトリ作成
    これも**作成済のZenn用フォルダ**で実行。
    ```powershell
    > cd D:\Zenn
    > git init
    ```
    これで`Zenn`フォルダの中に隠しフォルダ`.git`が作成され、Gitのローカルリポジトリとして機能するようになる。
    リポジトリが機能しているかどうかは、`git init`を実行したフォルダで
    ```powershell
    > git status
    ```
    で確認できる。リポジトリでなければ
    ```powershell
    fatal: not a git repository (or any of the parent directories): .git
    ```
    のように怒られる。
    ちなみに、隠しフォルダ`.git`を削除すればGitのローカルリポジトリは完全に消滅する。

13. GitのローカルリポジトリとGitHubのリポジトリを紐付け
    GitHubのサイトから作成済のリポジトリに移動し、SSHのリモートアドレス(`git@github.com:～～/～～.git`)をコピー。
    ```powershell
    > git remote add origin git@github.com:～～/～～.git
    # originはGitHubのリポジトリ名に対する別名。originでなくてもいいがお決まり。
    ```
    これでローカルリポジトリとGitHubが連携する。紐付けの状態は
    ```powershell
    > git remote -v
    origin  git@github.com:～～/～～.git (fetch)
    origin  git@github.com:～～/～～.git (push)
    ```
    のように確認できる。

14. Markdown形式で記事の作成
    [公式](https://zenn.dev/zenn/articles/zenn-cli-guide)参照。ただし、公式の
    ```powershell
    > npx zenn new:article
    ```
    だとファイル名([slug](https://zenn.dev/zenn/articles/what-is-slug))がランダムに生成されるため、
    ```powershell
    > npx zenn new:article --slug file-name-of-text
    ```
    のようにslugだけは指定したほうがいい。(英数・ハイフン・アンスコ12〜50文字)
    Visual Studio Codeから`ファイル`→`フォルダーを開く`でZennフォルダーを開いて、`article`フォルダの下にできた.mdファイルを更新していく。
    Markdown記法は[公式](https://zenn.dev/zenn/articles/markdown-guide)参照。

15. プレビューの起動
    [公式](https://zenn.dev/zenn/articles/zenn-cli-guide)参照。
    ```powershell
    # これもZennフォルダで
    > npx zenn preview
    ```
    ブラウザで`http://localhost:8000`を開くと、.mdファイルを更新する度にプレビューも更新されていく。便利。

16. Gitのステージング・GitHubにプッシュ(普段Git使わない人向け)
    Gitにはステージングという構造がある。
    ```mermaid
    flowchart LR
    A[working<br>directory<br>#040;編集できる#041;] -- add --> B[staging<br>area] -- commit --> C[ローカル<br>リポジトリ] -- push --> D[(GitHub)]
    ```
    VSCodeで編集している.mdファイルはワーキングディレクトリ・作業ツリーと呼ばれ、そこからステージングエリアに移動させてから、ローカルリポジトリにコミットできる。そこから更にpushしてGitHubに辿り着く。ファイルを編集できるのはワーキングディレクトリだけ。
    ステージングエリアへの移動：
    ```powershell
    > git add .\articles\(ファイル名).md #特定のファイルをステージング
    > git add .\articles\               #articles内の全ファイルをステージング
    ```
    または、Visual Studio CodeでGitHub Pull Requestsを導入していれば、`ソース管理`から変更したファイルの`+`マークでステージングできる。
    ![](/images/setup-zenn-github/vscode-staging.png)
    ステージングが完了したら、ローカルリポジトリにコミットする。
    ```powershell
    > git commit -m "コミットメッセージ"
    ```
    またはVS Codeから
    ![](/images/setup-zenn-github/vscode-commit.png)
    これでローカルリポジトリが更新され、GitHubにpushできる状態になる。
    ```powershell
    # 最初は-uオプションでローカルとGitHubのブランチ(main)を紐づける。
    > git push -u origin main
    # 一度-uオプション指定すれば、次からは
    > git push
    # だけでOK。
    ```






17. SSHエージェントの使用
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
    3. SSHの起動・動作確認
        ```powershell
        > Get-Service ssh-agent
        # サービスが起動しているとこんな感じになる
        Status   Name               DisplayName
        ------   ----               -----------
        Running  ssh-agent          OpenSSH Authentication Agent
        > ssh -T git@github.com
        # 最初はパスフレーズを聞かれる
        Enter passphrase for key 'C:\Users\(ユーザ名)/.ssh/id_rsa':
        Hi (ユーザ名)! You ve successfully authenticated, but GitHub does not provide shell access.
        > ssh -T git@github.com
        # 2回目はパスフレーズを聞かれない(成功)
        Hi (ユーザ名)! You ve successfully authenticated, but GitHub does not provide shell access.
        ```
    4. Gitが使っているSSHクライアントを確認
        Windowsに導入したGitが、OpenSSHではなくGitにバンドルされているものになっていたらうまく動作しない。
        ```powershell
        > git config core.sshCommand
        # 何も表示されなければバンドルのSSHを使ってる可能性
        # 明示的にWIndowsのSSHサービスを使うように設定
        > git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
        ```
