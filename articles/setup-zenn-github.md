---
title: "ZennとGitHub連携の手順メモ(テスト中)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "Git", "GitHub"]
published: true
---
Zenn公式にはGitのローカルリポジトリ作成の記載がなかったので、それも含めてWindows環境でのZennとGitHubの連携手順メモ。GitHub普段使わない人(私)用。
Markdown記法のファイル(.md)を書いてGitHubに投げれば、自動でZennのサイトに掲載される便利機能。
## メリット
- ブラウザのエディタから開放される。好きなエディタでサッと書ける
- GitHubに版管理を丸投げできる
- Zenn CLIを導入すればlocalhost:8000でプレビューできる(便利)
## デメリット
- Zenn CLIはPC毎にセットアップが必要
- Node.jsのインストールが必要(v14以上)。人によっては環境競合するかも

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
   (場所はだいたいこのへん↓)
   ```powershell
   $env:PATH="$($env:PATH);C:\Program Files\Git\cmd"
   ```
   もしくはスタート→設定(歯車マーク)→システム→バージョン情報→システムの詳細設定→詳細設定タブ→環境変数→システムの環境変数からPathをダブルクリックして「新規」からGitのパスを追加(コマンドラインで打ったほうがずっと早い)
6. Gitの設定
   ```powershell
   git config --global user.name "ユーザ名"       #必須
   git config --global user.email "mail@address" #必須
   git config --global init.defaultBranch main #masterじゃなくてmainにしようね
   ```
   configのリスト確認方法：
   ```powershell
   git config list
   ```
7. GitHubにSSHキー登録
   ```powershell
   ssh-keygen -t rsa -b 4096 -C "mail@address"
   ```
   パスフレーズを要求されるので入力。これで秘密鍵と公開鍵ができるので公開鍵をGitHubに登録。
   公開鍵の場所は`C:\Users\(ユーザ名)\.ssh\id_rsa.pub`
   エクスプローラだとPC→Windows(C:)→ユーザー→(ユーザ名)→.sshで辿れる。
   ファイルを適当なテキストエディタで開いて、`ssh-rsa ～～～`のテキストを丸ごとコピー
   GitHubのサイトから右上のアイコン→Settings→SSH and GPG keys→New SSH keyでKeyにコピーしたテキストをそのまま貼り付け→Add SSH key
8. Node.jsをインストール
   [公式サイト](https://nodejs.org/)からダウンロード
