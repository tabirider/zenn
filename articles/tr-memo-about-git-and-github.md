---
title: "Git/GitHubリファレンス的な(自分用メモ随時更新)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git","github"]
published: false
---

## Git

## GitHub

### private/publicリポジトリの差異

|内容|プライベートリポジトリ|パブリックリポジトリ|
|--|--|--|
|`pull`|所有者・招待ユーザのみ|無制限|
|`push`|所有者・招待ユーザのみ|所有者・招待ユーザのみ<br>pull requestで変更を提案|
|用途|プライベートプロジェクト|オープンソース・公共プロジェクト|
|コスト|無償※|無償※|

> ※GitHubのプランはFree/Team/Enterprise。Freeでどちらのリポジトリも無償利用可能。Team/EnterpriseではPull Requestに複数者アサイン、GitHub Packages(ソースコードではなくパッケージ、バイナリ等を保管)容量増加など機能が拡張される。

### GitHub Packages

ソースコードではなくバイナリイメージ、コンテナ等を保管するもの。Free:500MB、Team:2GB、Enterprise:50GB。
DockerHubの代替としても使える。
- GitHubリポジトリと統合して管理
- 無料でも複数イメージをホスティング可能(DockerHubはイメージ1つまで)
- GitHub Actionsを使ってDockerイメージの自動ビルド・デプロイ可能

### GitHubへの接続

GitからGitHubリポジトリへの接続には`SSH`と`HTTPS`があるが、[公式はHTTPSを推奨](https://docs.github.com/ja/get-started/getting-started-with-git/set-up-git#connecting-over-https-recommended)。セキュリティに差はないし、SSHポートは塞がれがちだけどHTTPSならどこからでも接続できるよ、みたいな理由らしい。Dockerコンテナで作業するような場合も`SSH`だと秘密鍵ファイルをコンテナでマウントして読めるようにしたり工夫が要るが、`HTTPS`なら面倒な作業を省ける。
`HTTPS`接続の場合、`Personal Access Token(PAT)`による認証が必要。`PAT`の発行方法は下記。

```shell-session
$ git remote add origin https://<GitHubユーザー名>@github.com/<GitHubユーザー名>/<リポジトリ名>.git
```

### Personal Access Token(PAT)

`HTTPS`接続でのパスワード認証は廃止済。代わりに`Personal Access Token(PAT)`を利用する。
GitHubサイトの右上アイコン→`Settings`→`Developer settings`→`Personal access tokens`→`Fine-grained tokens`を選択。(2025年1月時点で`Preview`だが、[公式](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#personal-access-token-%E3%81%AE%E7%A8%AE%E9%A1%9E)はこっちを強く推奨してる)
`Generate new token`→2段階認証。`Token name`に後で分かるよう記入。`Expiration`は`Custom`でも最長1年まで。
`Permissions`はメチャクチャ細かい。詳細は[公式](https://docs.github.com/ja/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens?apiVersion=2022-11-28#repository-permissions-for-actions)参照。特定の項目を有効にすると、連動して`mandatory`になる項目があるので注意。権限は変更できるので必要なら随時更新。
`Generate token`でPATの作成完了。**作成されたトークンは再表示できないので安全な場所にコピーしておく**。

リファレンスとしてメモ。

#### Repository Access

|Repository access|Description|
|--|--|
|`Public Repositories (read-only)`||
|`All repositories`|`This applies to all current and future repositories you own.Also includes public repositories (read-only).`<br>あなたのrepo(これから作るのも)全部適用。publicも含む。|
|`Only select repositories`|`Select at least one repository. Max 50 repositories.Also includes public repositories (read-only).`<br>適用するリポジトリを選択。|

#### Permissions

権限は`No access`, `Read-only`, `Read and write`から選択。

##### Repository Permissions

|permissions|description|R|RW|
|--|--|--|--|
|`Actions`|`Workflows, workflow runs and artifacts.`|`R`|`RW`|
|`Administration`|`Repository creation, deletion, settings, teams, and collaborators.`|`R`|`RW`|
|`Attestations`|`Create and retrieve attestations for a repository.`|`R`|`RW`|
|`Code scanning alerts`|`View and manage code scanning alerts.`|`R`|`RW`|
|`Codespaces`|`Create, edit, delete and list Codespaces.`|`R`|`RW`|
|`Codespaces lifecycle admin`|`Manage the lifecycle of Codespaces, including starting and stopping.`|`R`|`RW`|
|`Codespaces metadata`|`Access Codespaces metadata including the devcontainers and machine type.`|`R`||
|`Codespaces secrets`|`Restrict Codespaces user secrets modifications to specific repositories.`||`RW`|
|`Commit statuses`|`Commit statuses.`|`R`|`RW`|
|`Contents`|`Repository contents, commits, branches, downloads, releases, and merges.`<br>**※`git push`には最低限これが必要**。|`R`|`RW`|
|`Custom properties`|`View and set values for a repository's custom properties, when allowed by the property.`|`R`|`RW`|
|`Dependabot alerts`|`Retrieve Dependabot alerts.`|`R`|`RW`|
|`Dependabot secrets`|`Manage Dependabot repository secrets.`|`R`|`RW`|
|`Deployments`|`Deployments and deployment statuses.`|`R`|`RW`|
|`Discussions`|`Discussions and related comments and labels.`|`R`|`RW`|
|`Environments`|`Manage repository environments.`|`R`|`RW`|
|`Issues`|`Issues and related comments, assignees, labels, and milestones.`|`R`|`RW`|
|`Merge queues`|`Manage a repository's merge queues`|`R`|`RW`|
|`Metadata`|`Search repositories, list collaborators, and access repository metadata.`<br>`Contents`を`Read and Write`にすると自動的に`Read`が付与される|`R`||
|`Pages`|`Retrieve Pages statuses, configuration, and builds, as well as create new builds.`|`R`|`RW`|
|`Pull requests`|`Pull requests and related comments, assignees, labels, milestones, and merges.`|`R`|`RW`|
|`Repository security advisories`|`View and manage repository security advisories.`|`R`|`RW`|
|`Secret scanning alerts`|`View and manage secret scanning alerts.`|`R`|`RW`|
|`Secrets`|`Manage Actions repository secrets.`|`R`|`RW`|
|`Variables`|`Manage Actions repository variables.`|`R`|`RW`|
|`Webhooks`|`Manage the post-receive hooks for a repository.`|`R`|`RW`|
|`Workflows`|`Update GitHub Action workflow files.`||`RW`|

##### Account permissions

|permissions|description|R|RW|
|--|--|--|--|
|`Block another user`|`View and manage users blocked by the user.`|`R`|`RW`|
|`Codespaces user secrets`|`Manage Codespaces user secrets.`|`R`|`RW`|
|`Copilot Chat`|`This application will receive your GitHub ID, your GitHub Copilot Chat session messages (not including messages sent to another application), and timestamps of provided GitHub Copilot Chat session messages. This permission must be enabled for Copilot Extensions.`|`R`||
|`Copilot Editor Context`|`This application will receive bits of Editor Context (e.g. currently opened file) whenever you send it a message through Copilot Chat.`|`R`||
|`Email addresses`|`Manage a user's email addresses.`|`R`|`RW`|
|`Events`|`View events triggered by a user's activity.`|`R`||
|`Followers`|`A user's followers`|`R`|`RW`|
|`GPG keys`|`View and manage a user's GPG keys.`|`R`|`RW`|
|`Gists`|`Create and modify a user's gists and comments.`||`RW`|
|`Git SSH keys`|`Git SSH keys`|`R`|`RW`|
|`Interaction limits`|`Interaction limits on repositories`|`R`|`RW`|
|`Knowledge bases`|`View knowledge bases for a user.`|`R`|`RW`|
|`Plan`|`View a user's plan.`|`R`||
|`Private repository invitations`|`View a user's invitations to private repositories`|`R`||
|`Profile`|`Manage a user's profile settings.`||`RW`|
|`SSH signing keys`|`View and manage a user's SSH signing keys.`|`R`|`RW`|
|`Starring`|`List and manage repositories a user is starring.`|`R`|`RW`|
|`Watching`|`List and change repositories a user is subscribed to.`|`R`|`RW`|

### Credential Manager
