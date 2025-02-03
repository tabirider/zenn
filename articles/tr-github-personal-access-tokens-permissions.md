---
title: "GitHubのPersonal Access Token(PAT)のパーミッションが細かすぎる"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["git","github"]
published: true
---

GitHubのPAT設定が細かすぎて草。
**結論：Repository PermissionsのContentsをRead and writeにする。以上。**

## Personal Access Token

GitHubでは[SSHよりHTTP接続を推奨](https://docs.github.com/ja/get-started/getting-started-with-git/set-up-git#connecting-over-https-recommended)しているが、`HTTPS`接続でのパスワード認証は廃止済。代わりに`Personal Access Token(PAT)`を使う。
..んだけど、特に`Fine-grained tokens`は権限設定が細かすぎて読んでるうちに日が暮れる。
最後まで読んで、ひとまず`Contents`だけ設定すればOK、あとは使っているうちに必要なら追加していけばいい、と理解した。

## PATの作成方法

GitHubサイトの右上アイコン→`Settings`→`Developer settings`→`Personal access tokens`→`Fine-grained tokens`を選択。(2025年2月時点で`Preview`だが、[公式](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#personal-access-token-%E3%81%AE%E7%A8%AE%E9%A1%9E)はこっちを強く推奨)
`Generate new token`→2段階認証。`Token name`に後で分かるよう記入。`Expiration`で有効期間設定。

:::message
**`Expiration`は最近`No expiration`が選択できるようになった**(が非推奨)。

> GitHub **strongly recommends that you set an expiration date** for your token to help keep your information secure.
:::

で、`Permissions`がメチャクチャ細かい。詳細は[公式](https://docs.github.com/ja/rest/authentication/permissions-required-for-fine-grained-personal-access-tokens?apiVersion=2022-11-28)参照。特定の項目を有効にすると、連動して`mandatory`になる項目がある。権限は**PAT作った後でも変更できる**ので必要なら随時更新。
`Generate token`でPATの作成完了。**作成されたトークンは再表示できないので安全な場所にコピーしておく**。

リファレンスとしてメモ。まだまだ変更とか入りそう。

### Repository Access

どのリポジトリに対する権限か選択できる。

|Repository access|Description|
|--|--|
|`Public Repositories (read-only)`|パブリックリポジトリに適用。これを選択するとリポジトリに対する読取**だけ**の権限が設定される(ので、`Repository Permissions`の選択肢は選べなくなる)。間違ってリポジトリを更新したりpull requestしたりしないよう、自動スクリプトに権限を付けたりするのに使える。|
|`All repositories`|`This applies to all current and future repositories you own.Also includes public repositories (read-only).`<br>リポジトリ(これから作るのも)全部に適用。パブリックリポジトリ(の読取権限)も含む。|
|`Only select repositories`|`Select at least one repository. Max 50 repositories.Also includes public repositories (read-only).`<br>適用するリポジトリを選択(50まで)。パブリックリポジトリ(の読取権限)も含む。|

### Permissions

ここからが本題。権限は`No access`, `Read-only`, `Read and write`から選択。

#### Repository Permissions

リポジトリに対する権限。めちゃくちゃ多い。
※契約プランによっては使えないのも混じってる。

|permissions|description|R|RW|
|--|--|--|--|
|`Actions`|`Workflows, workflow runs and artifacts.`<br>ワークフロー、及びその成果物の管理。`gh workflow run`等の実行権限。ワークフロー実行だけなら`Read-only`でOK。<br>`GitHub Actions`はCI/CD(Continuous Integration / Continuos Delivery)ツール。`push`等をトリガに`pytest`流したり、ビルド、デプロイ等を行える。`cron`的にも使える。<br>関係する`Workflows`権限が一番下にあるっていうね..。|`R`|`RW`|
|`Administration`|`Repository creation, deletion, settings, teams, and collaborators.`<br>リポジトリ作成・削除・設定・チーム・コラボレーター管理。|`R`|`RW`|
|`Attestations`|`Create and retrieve attestations for a repository.`<br>証明書の作成・取得。ここで言う`attestations`とはビルドした成果物に付与する署名メタデータ。`GitHub Actions`の`attest-build-provenance`で生成する。|`R`|`RW`|
|`Code scanning alerts`|`View and manage code scanning alerts.`<br>`code scan`による警告参照・管理。Code Scanningはリポジトリ内のコードを分析して、脆弱性やコーディングミスを見つけれくれる機能。GitHub Actionsで動かすもの。|`R`|`RW`|
|`Codespaces`|`Create, edit, delete and list Codespaces.`<br>Code spaces上での作成・編集・削除・閲覧(Code spacesはGitHubが提供するクラウド上の開発環境、VSCodeからアクセスできる)。|`R`|`RW`|
|`Codespaces lifecycle admin`|`Manage the lifecycle of Codespaces, including starting and stopping.`<br>Code space自体の作成・起動・停止など。|`R`|`RW`|
|`Codespaces metadata`|`Access Codespaces metadata including the devcontainers and machine type.`<br>Code spacesのメタ情報(コンテナ、マシン種類等)へのアクセス。|`R`||
|`Codespaces secrets`|`Restrict Codespaces user secrets modifications to specific repositories.`<br>Code spaces上の機密情報(APIキー、DB接続情報等。Code spaceで環境変数としてアクセスできる)の編集を許可。`Account Permissions`にも似たのがあるが、こちらのsecretsはリポジトリ単位に適用されるものを指す。||`RW`|
|`Commit statuses`|`Commit statuses.`<br>コミットステータスの取得・変更。|`R`|`RW`|
|`Contents`|`Repository contents, commits, branches, downloads, releases, and merges.`<br>リポジトリ参照、コミット、ブランチング、pull/clone、リリース、マージ。<br>**`git push`には最低限これが必要**。|`R`|`RW`|
|`Custom properties`|`View and set values for a repository's custom properties, when allowed by the property.`<br>(可能なものについて)カスタムプロパティの参照・変更。|`R`|`RW`|
|`Dependabot alerts`|`Retrieve Dependabot alerts.`<br>`Dependabot`からの警告を受け取る。<br>※Dependabotはリポジトリが依存するライブラリの更新・脆弱性を監視してくれるもの(リポジトリ毎に有効化してあげる必要がある)。|`R`|`RW`|
|`Dependabot secrets`|`Manage Dependabot repository secrets.`<br>Dependabotが必要とする機密情報(プライベートパッケージの更新、認証が必要なリポジトリのPAT等)の管理。|`R`|`RW`|
|`Deployments`|`Deployments and deployment statuses.`<br>デプロイ・デプロイ状態の取得|`R`|`RW`|
|`Discussions`|`Discussions and related comments and labels.`<br>ディスカッション・関連コメント・ラベル|`R`|`RW`|
|`Environments`|`Manage repository environments.`<br>GitHub Actionsのデプロイを管理する環境情報(デプロイ先の環境制限、承認ワークフロー設定、デプロイ履歴等)の管理。|`R`|`RW`|
|`Issues`|`Issues and related comments, assignees, labels, and milestones.`<br>イシュー、関連コメント、担当、ラベル、マイルストンへのアクセス。|`R`|`RW`|
|`Merge queues`|`Manage a repository's merge queues`<br>リポジトリのmergeキュー管理|`R`|`RW`|
|`Metadata`|`Search repositories, list collaborators, and access repository metadata.`<br>リポジトリの検索、コラボレーターの参照、メタデータの取得。**`Contents`を`Read and Write`にすると自動的に`Read`が付与される**|`R`||
|`Pages`|`Retrieve Pages statuses, configuration, and builds, as well as create new builds.`<br>`GitHub Pages`のステータス取得、設定、ビルド。<br>※GitHub Pagesはリポジトリの内容を静的コンテンツとして公開するホスティングサービス。HTML, CSS等を参照し、GitHubや独自ドメインでアクセスできる仕組み。`Jekyll`で`.md`ファイルをHTMLに変換することもできる。|`R`|`RW`|
|`Pull requests`|`Pull requests and related comments, assignees, labels, milestones, and merges.`<br>`Pull Requests`関連、コメント、アサイン、ラベル、マイルストン、マージ。|`R`|`RW`|
|`Repository security advisories`|`View and manage repository security advisories.`<br>リポジトリのセキュリティ勧告の参照・管理。開発者が脆弱性について報告・修正・公開するための仕組み。対応まで非公開に保つことができる。GitHubからCVE(Common Vulnerabilities and Exposures、脆弱性番号)を取得する機能もある(審査が必要)。|`R`|`RW`|
|`Secret scanning alerts`|`View and manage secret scanning alerts.`<br>GitHubの`Secret Scanning`(リポジトリに間違って上げた機密情報を見つけてくれる仕組み)による警告を受け取る。|`R`|`RW`|
|`Secrets`|`Manage Actions repository secrets.`<br>`GitHub Actions`(ワークフロー管理)に必要なキー情報(AWS, GitHub Token等のAPIキー、DB接続情報、SSHキー等)を管理する仕組みへのアクセス許可。通常は暗号化され、GitHub Actions内で環境変数として参照できるもの。ここで登録したキー情報は`Secret scanning`の対象から除外される。|`R`|`RW`|
|`Variables`|`Manage Actions repository variables.`<br>`GitHub Actions`で参照される環境変数情報へのアクセス許可。|`R`|`RW`|
|`Webhooks`|`Manage the post-receive hooks for a repository.`<br>`Webhooks`はGitHub上のイベントをフックしてHTTPリクエストを送信する仕組み。`push`, `pull request`等を引っ掛けてSlackに投げたりできる。受け側はsignatureなどで正規のフックリクエストか検証したほうがいい。|`R`|`RW`|
|`Workflows`|`Update GitHub Action workflow files.`<br>`GitHub Actions`で参照される`.github/workflows/`下のファイル更新権限。||`RW`|

#### Account permissions

アカウントに対する権限。こっちはまだ分かりやすい。

|permissions|description|R|RW|
|--|--|--|--|
|`Block another user`|`View and manage users blocked by the user.`<br>他のユーザをブロック/解除したり、その情報を参照できる。|`R`|`RW`|
|`Codespaces user secrets`|`Manage Codespaces user secrets.`<br>Code spacesのsecrets管理。`Repository Permissions`にも似たのがあるが、こっちはGitHub全体に適用される機密情報を指す。|`R`|`RW`|
|`Copilot Chat`|`This application will receive your GitHub ID, your GitHub Copilot Chat session messages (not including messages sent to another application), and timestamps of provided GitHub Copilot Chat session messages. This permission must be enabled for Copilot Extensions.`<br>Copilotを使うときに必要。|`R`||
|`Copilot Editor Context`|`This application will receive bits of Editor Context (e.g. currently opened file) whenever you send it a message through Copilot Chat.`<br>Copilotがファイルを編集するために必要。|`R`||
|`Email addresses`|`Manage a user's email addresses.`<br>メアド|`R`|`RW`|
|`Events`|`View events triggered by a user's activity.`<br>イベント情報を見れる。|`R`||
|`Followers`|`A user's followers`<br>フォロワーを見れる。|`R`|`RW`|
|`GPG keys`|`View and manage a user's GPG keys.`<br>GPGキーの参照・管理|`R`|`RW`|
|`Gists`|`Create and modify a user's gists and comments.`<br>`Gist`(スニペットやメモをサッと書くやつ)を使える。||`RW`|
|`Git SSH keys`|`Git SSH keys`<br>SSHキーの管理|`R`|`RW`|
|`Interaction limits`|`Interaction limits on repositories`<br>特定ユーザからの操作を制限。既存ユーザのみ、コントリビュータのみ、コラボレータのみ。制限期間も設定できる。スパムのブロック等に。|`R`|`RW`|
|`Knowledge bases`|`View knowledge bases for a user.`<br>`Knowledge bases`は複数リポジトリにあるMarkdownファイルを集約し、Copilot Chatで参照できるもの。|`R`|`RW`|
|`Plan`|`View a user's plan.`<br>ユーザのプランを見れる|`R`||
|`Private repository invitations`|`View a user's invitations to private repositories`<br>プライベートリポジトリへのユーザ招待を見れる|`R`||
|`Profile`|`Manage a user's profile settings.`<br>プロフィール設定||`RW`|
|`SSH signing keys`|`View and manage a user's SSH signing keys.`<br>ユーザのSSHキー管理。GitHub接続に使うSSHキーは`Git SSH keys`のほう。こっちは**GPG代わりに使う署名用**。最近GitHubが始めたもので、`Add new SSH Key`で`Authentication Key`の他に`Signing Key`が選択できるようになっている。|`R`|`RW`|
|`Starring`|`List and manage repositories a user is starring.`<br>星つけてるリポジトリ管理|`R`|`RW`|
|`Watching`|`List and change repositories a user is subscribed to.`<br>サブスクライブしてるリポジトリ管理|`R`|`RW`|

感想：GitHubの知らない機能を色々勉強できたのでよかったです。