---
title: "Dockerコマンドリファレンス的な(自分用メモ随時更新)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker"]
published: false
---
使いそうなものから順番にメモ。

:::message
[Dockerドキュメント日本語化プロジェクト](https://docs.docker.jp/index.html)から更に抜粋。内容によって旧バージョンも混じっている。正確な情報は[本家](https://docs.docker.com/)を参照。
:::

## Dockerfile

[公式](https://docs.docker.jp/engine/reference/builder.html)。`docker image build`時に参照される。
書式:
```dockerfile:Dockerfile
# comment
INCTRUCTION arguments
INCTRUCTION arguments
..
```

### Dockerfileのベストプラクティス

[公式](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html)から

- コンテナは可能な限り一時的(使い捨て)であるべき
- レイヤ数を最小に。[マルチステージビルド](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html#id9)で最終イメージの容量を節約できる
- 不要なパッケージは入れない
- 各コンテナはただ一つの用途を持つべき
- apt installなどはアルファベット順に並べ重複を見つけやすく
- 変更頻度の低い順番からレイヤを作成すると効率的

### ディレクティブ一覧

- 必ず`FROM`から始める。(コメント、ディレクティブ、`ARG`は例外)
- `RUN`, `COPY`, `ADD`が実行されるとレイヤが構成される。Dockerイメージはレイヤの積み重ねで、各レイヤは前提レイヤからの差分。
- レイヤはキャッシュされる。`COPY`, `ADD`で作成されるレイヤは、そこに含まれるファイルのチェックサムを参照するので、ファイルの中身が変わったらキャッシュは再利用されない(タイムスタンプはチェックサムに含まれない)。`RUN`の場合は単にコマンドの内容だけでキャッシュの利用可否を判断している。`docker build --no-cache`でキャッシュを無視できる。
- `COPY`は単純なファイルコピー。`ADD`はURLからのダウンロード、圧縮ファイルの展開を行う。通常は`COPY`を利用する。


||内容|
|--|--|
|`FROM`|ベースイメージを指定<br>`FROM [--platform=<プラットフォーム>] <イメージ名> [AS <名前>]`<br>`FROM [--platform=<プラットフォーム>] <イメージ名>[:<タグ>] [AS <名前>]`<br>`FROM [--platform=<プラットフォーム>] <イメージ名>[@<ダイジェスト>] [AS <名前>]`<br>`scratch`指定で何もないイメージが作れる。[マルチステージビルド](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html#id9)で使用。|
|`RUN`|直前のレイヤに対してコマンド実行。デフォルトは`/bin/sh -c`。`apt update`, `apt install`等。複数の`RUN`を実行すると複数レイヤができることに留意、レイヤを分ける必要なければ`&&`でまとめる。`\`で複数行分割できる。`install`はアルファベット順に並べておくと重複を見つけやすい。|
|`CMD`|`CMD ["実行ファイル", "パラメータ1", "パラメータ2", ..]`<br>コンテナ起動時に実行するコマンド。コンテナは上がった段階で仕事なければすぐ落ちる。`CMD ["bash"]`しておけば勝手に落ちない。`docker run`の引数を指定すると`CMD`の設定が上書きされる。|
|`EXPOSE`|`EXPOSE <ポート> [<ポート>/<プロトコル>...]`(プロトコル省略時は`TCP`)<br>ポート公開指示**だが公開されない**。`docker run -p`で割当、`EXPOSE`はメモ代わり。ただし`Docker Compose`では参照する。<br>`EXPOSE 80`<br>`EXPOSE 3000 8080`|
|`ENV`|環境変数。作成したコンテナに引き継がれる。|
|`ARG`|イメージ構築時、ステージ内だけで有効な変数。|
|`ADD`|`ADD [--chown=<ユーザ>:<グループ>] <追加元>... <追加先>`<br>`ADD [--chown=<ユーザ>:<グループ>] ["<追加元>",... "<追加先>"]`<br>通常は`COPY`で足りる。URLから引っ張ったり圧縮ファイルの展開(`tar -x`相当)処理が入る。|
|`COPY`|`[--chown=<ユーザ>:<グループ>] <コピー元>... <コピー先>`<br>`[--chown=<ユーザ>:<グループ>] ["<コピー元>",... "<コピー先>"]`<br>ファイルコピー。`<コピー先>`が相対パスの場合`WORKDIR`からの相対パス。`COPY hoge piyo/`は`hoge`→`<WORKDIR>/piyo/`。絶対パスも可。コピーされたファイルは`UID=0`, `GID=0`。必要に応じてchown指定。<br>ただし、`COPY`できるのは **`docker build`時のディレクトリ下だけ**。`../`のように親階層は参照できない。|
|`ENTRYPOINT`|`ENTRYPOINT ["実行ファイル", "パラメータ1", "パラメータ2"]`<br>`CMD`と同じ動きをするが、`docker run`で上書きされない。必ず実行するコマンドを指定するのに使用。`docker run`の引数は`ENTRYPOINT`の後に追加される。<br>コンテナを開発環境のように操作する場合は`CMD`, DB/Webサーバのようにバイナリとして扱う場合は`ENTRY`と使い分ける。|
|`VOLUME`|`VOLUME ["/data"]`<br>Docker内の匿名の場所にボリュームを生成し、コンテナ内の指定されたパスにマウントする。コンテナ内の実在するパスを指定すると、マウントポイントとして上書きされる。コンテナやイメージ削除しても匿名の場所は削除されない。複数コンテナでのファイル共有や永続化に使える。生成されたvolumeは`docker volume ls`で参照できる。<br>ホストOS上のディレクトリを直接マウントする場合は、Dockerfileではなく`docker run`/`start`で指定する。<br>**`VOLUME`指定したコンテナを上げるとランダムネームのボリュームが生成され、`docker volume create`で作成したボリュームは指定できない**ので、あまり使い勝手は良くない。`docker volume`で作成したボリュームをコンテナにマウントする場合は、`docker container run`/`create`で指定する必要がある。|
|`USER`|`USER <ユーザ>[:<グループ>]`<br>`USER <UID>[:<GID>]`<br>コンテナ内にそのユーザがないとエラーになる。`RUN useradd`とかしないと使えない。|
|`WORKDIR`|作業ディレクトリの指定。なければ作成。以降はDockerfile内の相対パスは`WORKDIR`が起点になる。|
|`ONBUILD`|`ONBUILD [命令]`<br>他のイメージ構築ベースにするとき、トリガとして実行する命令。必要になれば確認|
|`STOPSIGNAL`|コンテナ停止時にDockerがプロセスに送信するシグナル。デフォルトでは`SIGTERM`。他に`SIGKILL`(強制終了), `SIGHUP`(プロセスの再読込要求),`SIGUSR1`(アプリ固有のシグナル)等|
|`HEALTHCHECK`|`HEALTHCHECK [オプション] CMD コマンド`<br>`HEALTHCHECK NONE`(ベースイメージからヘルスチェック設定の継承を無効化)<br>Dockerは指示されたテスト方法でコンテナ動作を確認するようになる。<br>`HEALTHCHECK --interval=5m --timeout=3s --retries=3 CMD curl -f http://localhost/ \|\| exit 1`|
|`SHELL`|デフォルトのシェルコマンドを上書き。以降`RUN`で指定したシェルが使われる。Winだと`SHELL ["powershell","-command"]`しておけば`RUN`の記述がちょっと楽になる。たぶん使い途はそれくらい|

## .dockerignoreファイル
docker image build時のディレクトリに`.dockerignore`があれば読み込まれる。`.gitignore`みたいな感じ。該当するファイルはDockerデーモンに送信されずCOPYもできない。
```:.dockerignore
# comment
*/temp*
*/*/temp*
temp?
.git/*
```

## Docker BuildKit

拡張されたビルダー。**DockerEngine v23.0以降では[デフォルトのビルダーがBuildKit](https://docs.docker.com/build/buildkit/#getting-started)になっている**
[公式](https://docs.docker.com/build/buildkit/#getting-started)より。ざっくりビルドを高速化・機能拡張するもの。
- 未使用ビルドステージを検出し実行をスキップ
- ビルドステージの並列化
- ビルドコンテキスト内の変更ファイルのみ増分転送
- ビルドコンテキスト内の未使用ファイルを検出し転送をスキップ
- [Dockerフロントエンド](https://docs.docker.com/build/buildkit/frontend/)を使用
- REST APIの副作用を回避
- ビルドキャッシュを優先して自動プルーニング(刈り取り)

Low-Level Build(LLB)
ビルド時の依存関係、キャッシュの単位。ビルド周りを全部作り直してこうなったよ
![](https://docs.docker.com/build/images/buildkit-dag.svg)

## Docker CLI(docker)

理解のため、なるべくエイリアスではなく正規(?)の表現を使う。`docker run`→`docker container run`、`docker build`→`docker image build`

## 環境変数

|変数|説明|
|--|--|
|`DOCKER_API_VERSION`|デバッグ用に通信時のAPIバージョンを上書き(例：1.19)|
|`DOCKER_CERT_PATH`|認証鍵の場所。この変数は、dockerCLIとdockerdデーモンの両方で使用|
|`DOCKER_CONFIG`|クライアント設定ファイルの場所|
|`DOCKER_CONTENT_TRUST_SERVER`|使用するNotaryサーバのURL。デフォルトはレジストリと同じURL|
|`DOCKER_CONTENT_TRUST`|DockerがNotaryを使ってイメージの署名と確認をする場合に指定。build、careate、pull、push、run用の--disable-content-trust=falseと同じ|
|`DOCKER_CONTEXT`|使用するdockercontextの名前(DOCKER_HOST環境変数と、dockercontextuseによるデフォルトのcontext設定を上書き)|
|`DOCKER_DEFAULT_PLATFORM`|コマンドを扱うデフォルトのプラットフォームで、--platformプラグでも指定可能|
|`DOCKER_HIDE_LEGACY_COMMANDS`|これを指定すると、Dockerは「レガシーな」(過去の)トップレベルコマンド(dockerrmやdockerpull)を非表示にし、オブジェクト型の管理コマンド(例：dockercontainer)のみ表示します。将来のリリースでは、この環境変数が削除される時点で、デフォルトになる可能性があります。|
|`DOCKER_HOST`|接続しようとするデーモンのソケット|
|`DOCKER_STACK_ORCHESTRATOR`|dockerstack管理コマンドの利用時に使う、デフォルトのオーケストレータを設定|
|`DOCKER_TLS_VERIFY`|DockerでTLSを使い、リモート認証をする時に指定。この変数は、dockerCLIとdockerdデーモンの両方で使用|
|`BUILDKIT_PROGRESS`|構築にBuildKitバックエンドを使用時、進捗の出力タイプ(auto、plain、tty)を指定。plainを使うと、コンテナの出力を表示(デフォルトauto)|

## docker

|コマンド|説明|
|--|--|
|`docker attach`|=`docker container attach`|
|`docker build`|=`docker image build`|
|`docker builder`|構築を管理。`BuildKit`による拡張されたビルド|
|`docker checkpoint`|チェックポイントを管理(experimental)|
|`docker commit`|=`docker container commit`|
|`docker config`|Dockerconfigsを管理|
|`docker container`|コンテナを管理|
|`docker context`|コンテクストを管理|
|`docker cp`|=`docker container cp`|
|`docker create`|=`docker container create`|
|`docker diff`|=`docker container diff`|
|`docker events`|=`docker system events`|
|`docker exec`|=`docker container exec`|
|`docker export`|=`docker container export`|
|`docker history`|=`docker image history`|
|`docker image`|イメージの管理|
|`docker images`|=`docker image ls`|
|`docker import`|=`docker image import`|
|`docker info`|=`docker system info`|
|`docker inspect`|Dockerオブジェクトのlow-level情報を返す|
|`docker kill`|=`docker container kill`|
|`docker load`|=`docker image load`|
|`docker login`|Dockerレジストリにログイン|
|`docker logout`|Dockerレジストリからログアウト|
|`docker logs`|=`docker container logs`|
|`docker manifest`|Dockerイメージマニフェストの管理と、マニフェスト一覧表示(experimental)|
|`docker network`|ネットワークの管理|
|`docker node`|Swarmノードの管理|
|`docker pause`|1つまたは複数のコンテナ内の全プロセスを一次停止|
|`docker plugin`|プラグインの管理|
|`docker port`|=`docker container port`|
|`docker ps`|=`docker container ls`|
|`docker pull`|=`docker image pull`|
|`docker push`|=`docker image push`|
|`docker rename`|=`docker container rename`|
|`docker restart`|=`docker container restart`|
|`docker rm`|=`docker container rm`<br>`docker image rm`ではないので注意|
|`docker rmi`|=`docker image rm`|
|`docker run`|=`docker container run`|
|`docker save`|=`docker image save`|
|`docker search`|DockerHubでイメージを検索|
|`docker secret`|Dockerシークレットの管理|
|`docker service`|サービスの管理|
|`docker stack`|Dockerスタックの管理|
|`docker start`|=`docker container start`|
|`docker stat`|コンテナのリソース利用統計情報を、常時画面に表示し続ける|
|`docker stop`|=`docker container stop`|
|`docker swarm`|Swarmの管理|
|`docker system`|Dockerの管理|
|`docker tag`|=`docker image tag`|
|`docker top`|=`docker container top`|
|`docker trust`|Dockerイメージの信頼性を管理|
|`docker unpause`|=`docker container unpause`|
|`docker update`|=`docker container update`|
|`docker version`|Dockerバージョン情報を表示|
|`docker volume`|ボリュームの管理|
|`docker wait`|=`docker container wait`|


### docker builder

### docker checkpoint

### docker config

#### docker config create

新しいコンフィグを作成します。

```shell-session
$ docker config create [OPTIONS] CONFIG FILE
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--label`, `-l` |  | コンフィグにラベルを設定します。 |
| `--template-driver` |  | テンプレートドライバーを指定します。 |

---

#### docker config inspect

指定したコンフィグの詳細を表示します。

```shell-session
$ docker config inspect [OPTIONS] CONFIG [CONFIG...]
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--format`, `-f` |  | Goテンプレートを使用して出力を整形します。 |

---

#### docker config ls

利用可能なコンフィグを一覧表示します。

```shell-session
$ docker config ls [OPTIONS]
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--filter`, `-f` |  | 指定した条件で出力をフィルタリングします。 |
| `--format` |  | 出力を指定したGoテンプレートで整形します。 |
| `--quiet`, `-q` |  | コンフィグのIDのみを表示します。 |

---

#### docker config rm

指定したコンフィグを削除します。

```shell-session
$ docker config rm CONFIG [CONFIG...]
```

（オプションはありません）

### docker container

#### docker container attach

コンテナの標準入力、標準出力、および標準エラーに接続します。

```shell-session
$ docker container attach [OPTIONS] CONTAINER
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--detach-keys` | | デタッチするためのキーを指定します。 |
| `--no-stdin` | | 標準入力を無効化します。 |
| `--sig-proxy` | `true` | シグナルをプロキシします。 |

---

#### docker container commit

コンテナの変更を新しいイメージとして保存します。

```shell-session
$ docker container commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--author`, `-a` | | イメージの作者情報を指定します。 |
| `--change`, `-c` | | Dockerfile命令を変更します。 |
| `--message`, `-m` | | コミットメッセージを指定します。 |
| `--pause` | `true` | コミット中にコンテナを一時停止します。 |

---

#### docker container cp

ファイルまたはディレクトリをコンテナ内外でコピーします。

```shell-session
$ docker container cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
$ docker container cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
```

（オプションはありません）

---

#### docker container create

新しいコンテナを作成しますが、起動しません。

```shell-session
$ docker container create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--name` | | コンテナの名前を指定します。 |
| `--hostname`, `-h` | | コンテナのホスト名を指定します。 |
| `--env`, `-e` | | 環境変数を設定します。 |
| `--detach`, `-d` | | バックグラウンドで実行します。 |

---

#### docker container diff

コンテナファイルシステムの変更点を表示します。

```shell-session
$ docker container diff CONTAINER
```

（オプションはありません）

---

#### docker container exec

稼働中のコンテナ内でコマンドを実行します。

```shell-session
$ docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--detach`, `-d` | | バックグラウンドでコマンドを実行します。 |
| `--interactive`, `-i` | | 標準入力を有効にします。 |
| `--tty`, `-t` | | 疑似ターミナルを割り当てます。 |
| `--privileged` | | 特権モードで実行します。 |

---

#### docker container export

コンテナのファイルシステムをアーカイブとしてエクスポートします。

```shell-session
$ docker container export [OPTIONS] CONTAINER
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--output`, `-o` | | 出力先ファイルを指定します。 |

---

#### docker container inspect

コンテナの詳細な情報を表示します。

```shell-session
$ docker container inspect [OPTIONS] CONTAINER [CONTAINER...]
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--format`, `-f` | | Goテンプレートを使用して出力を整形します。 |

---

#### docker container kill

コンテナを強制終了します。

```shell-session
$ docker container kill [OPTIONS] CONTAINER [CONTAINER...]
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--signal`, `-s` | `SIGKILL` | 使用するシグナルを指定します。 |

---

#### docker container logs

コンテナのログを表示します。

```shell-session
$ docker container logs [OPTIONS] CONTAINER
```

| オプション | デフォルト | 説明 |
|--|--|--|
| `--follow`, `-f` | | リアルタイムでログを出力します。 |
| `--since` | | 指定した時刻以降のログを表示します。 |
| `--tail` | `all` | 表示するログ行数を指定します。 |
| `--timestamps`, `-t` | | タイムスタンプを表示します。 |

---

### docker context

### docker image

|コマンド|説明|
|--|--|
|`docker image build`|Dockerfileからイメージを構築|
|`docker image history`|イメージの履歴を表示|
|`docker image import`|ファイルシステム・イメージを作成するため、tarボールから内容を取り込み|
|`docker image inspect`|1つまたは複数イメージの詳細情報を表示|
|`docker image load`|tarアーカイブか標準入力からイメージを読み込み|
|`docker image ls`|イメージ一覧表示|
|`docker image prune`|使用していないイメージの削除|
|`docker image pull`|レジストリからイメージやリポジトリを取得|
|`docker image push`|レジストリにイメージやリポジトリを送信|
|`docker image rm`|1つまたは複数のイメージを削除|
|`docker image save`|1つまたは複数イメージをtarアーカイブに保存（デフォルトで標準出力にストリーミング）|
|`docker image tag`|対象イメージに元イメージを参照するタグを作成|

#### docker image build

イメージをビルド

```shell-session
$ docker image build [OPTIONS] PATH | URL | -
```

- `build`時、指定パス以下の全ファイルが(利用有無に関わらず)Dockerデーモンに送信される。`/`を指定してはならない(全ファイルがDockerデーモンに送信される)。指定パスより上のファイルはbuildで参照できない。
- イメージビルドにはビルドキャッシュが使われるので2回目以降は高速になる。

|option|default|説明|
|--|--|--|
|`--add-host`||任意のホストに対しIPを割り当てを追加(host:ip)|
|`--build-arg`||構築時の変数を設定|
|`--cache-from`||イメージに対してキャッシュ元を指定|
|`--cgroup-parent`||コンテナに対する任意の親cgroup|
|`--compress`||構築コンテクストをgzipを使って圧縮|
|`--cpu-period`||CPUCFS(completelyFairScheduler)期間を制限|
|`--cpu-quota`||CPUCFS(completelyFairScheduler)クォータを制限|
|`--cpu-shares`, `-c`||CPU配分(相対ウェイト)|
|`--cpuset-cpus`||アクセスを許可するCPUを指定(0-3,0,1)|
|`--cpuset-mems`||アクセスを許可するメモリノードを指定(0-3,0,1)|
|`--disable-context-trust`|`TRUE`|イメージの検証を無効化|
|`--file`, `-f`||Dockerfileの名前(デフォルトはパス/Dockerfile)|
|`--force-rm`||中間コンテナを常に削除|
|`--iidfile`||イメージIDをファイルに書き込む|
|`--isolation`||コンテナ分離技術|
|`--label`||イメージにメタデータを設定|
|`--memory`, `-m`||メモリの上限|
|`--memory-swap`||スワップの上限は、メモリとスワップの合計と同じ：-1はスワップを無制限にする|
|`--network`||【API1.25+】構築中のRUN命令で使うネットワークモードを指定|
|`--no-cache`||イメージの構築時にキャッシュを使用しない|
|`--output`、`-o`||【API1.40+】アウトプット先を指定(書式：type=local,dest=path)|
|`--platform`||【API1.38+】サーバがマルチプラットフォーム対応であれば、プラットフォームを指定|
|`--progress`|`auto`|進行状況の出力タイプを設定(auto、plain、tty)。plainを使うと、コンテナの出力を表示|
|`--pull`||イメージは、常に新しいバージョンのダウンロードを試みる|
|`--quiet`, `-q`||構築時の出力と成功時のイメージID表示を抑制|
|`--rm`|`TRUE`|構築に成功後、中間コンテナを削除|
|`--secret`||【API1.39+】構築時に利用するシークレットファイル(BuildKit有効時のみ)：id=mysecret,src=/local/secret|
|`--security-opt`||セキュリティのオプション|
|`--shm-size`||/dev/shmの容量|
|`--squash`||【experimental(daemon)|API1.25+】構築するレイヤを、単一の新しいレイヤに押し込む|
|`--ssh`||【API1.39+】構築時に利用するSSHエージェントのソケットやキー(BuildKit有効時のみ)(書式：default|<id>[=<socket>]|<key>[,<key>]])|
|`--stream`||サーバにアクセスし、構築コンテクストの状況を表示し続ける|
|`--tag`, `-t`||名前と、オプションでタグを名前:タグの形式で指定|
|`--target`||構築する対象の構築ステージを指定|
|`--ulimit`||ulimitオプション|

#### docker image history

イメージの履歴を表示

```shell-session
$ docker image history [OPTIONS] IMAGE
```

|option|default|説明|
|--|--|--|
|`--format`| |Goテンプレートを使い、イメージを整えて表示|
|`--human`, `-H`|`true`|人間が読みやすい形式で容量と日付を表示|
|`--no-trunc`| |出力を省略しない|
|`--quiet` , `-q`| |イメージIDのみ表示|

#### docker image import

tarボールから取り込み

```shell-session
$ docker image import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```
|option|default|説明|
|--|--|--|
|`--change`, `-c`| |イメージ作成時に Dockerfile 命令を追加|
|`--message`, `-m`| |読み込んだイメージにコミット・メッセージを設定|
|`--platform`| |【API 1.32+】サーバがマルチプラットフォームに対応している場合、プラットフォームを指定|


#### docker image inspect
#### docker image load
#### docker image ls
#### docker image prune
#### docker image pull
#### docker image push
#### docker image rm
#### docker image save
#### docker image tag


### docker inspect

### docker login

### docker logout

### docker manifest

### docker network

### docker node

### docker pause

### docker plugin

### docker search

### docker secret

### docker service

### docker stack

### docker stat

### docker swarm

### docker system

### docker trust

### docker version

### docker volume

コンテナに**匿名のボリューム**をマウントし永続化できる。
`docker container run`でホストOSのファイルシステムをマウントする手もあるけど、ホストOSとユーザ/グループが異なるとホストOS側に作られたファイルの扱いが面倒になるし、ホストOSへの依存が強くなる。volumeを使うと問題解消するが、これで作られたボリュームの中身はコンテナからしか見えないのに注意。
**`Dockerfile`で`VOLUME`指定すると、ランダムネームのボリュームが自動生成される**ので勝手が悪い。`docker volume create`で作成したボリュームを`docker container run`でマウントするのが基本的な使い方。

|項目|ホストOSディレクトリマウント|Docker Volume利用|
|:----|:----|:----|
|設定の容易さ|ホストOSのディレクトリを指定するだけ|初回にボリュームを作成|
|データの永続化|ホストOSのファイルシステムで維持|Docker管理下で維持|
|パフォーマンス|状況による(WSLでWinフォルダのマウントは論外^[WSL⇔Windowsファイルシステムはパフォーマンスの問題だけでなく、ケース・センシティブやアクセス権限など様々な問題を引き起こすので、基本的に使わないほうがいい])|高速|
|ポータビリティ|ホストOSに依存、環境ごとに調整|Docker内で完結|
|セキュリティ|ホスト側に依存|ホストOSから隔離、管理が容易|
|データの可視性|ホストOSで見れる(ただし権限の問題あり)|Docker CLIかコンテナから確認。**ホストOSからはブラックボックスになる**|
|バックアップ・復元|ホストOSで実行|Dockerコマンドで実行|
|マルチコンテナ共有|マウントパス共有で可能|ボリューム指定で可能|

> Kubernetesにはdocker volume相当の機能として、PersistentVolumeとPersistentVolumeClaimがある。移行時は考慮が必要。

|コマンド|説明|
|:----|:----|
|docker volume create|ボリュームの作成|
|docker volume inspect|1つまたは複数ボリュームの詳細情報を表示|
|docker volume ls|ボリューム一覧表示|
|docker volume prune|使用していないローカルボリュームを削除|
|docker volume rm|1つまたは複数ボリュームを削除|

#### docker volume create

ボリュームを作成。

```shell-session
$ docker volume create [OPTIONS] [VOLUME]
```

名前未指定でランダム名称生成されるので指定したほうがいい。
複数コンテナから同時参照も可能。
Dockerfileの`VOLUME`では、
Linuxのlocalドライバmountコマンドと似たオプションに対応。
```shell-session
$ docker volume create --driver local \
    --opt type=tmpfs \
    --opt device=tmpfs \
    --opt o=size=100m,uid=1000 \
    foo
```

|名前, 省略形|デフォルト|説明|
|:----|:----|:----|
|--driver , -d|local|ボリューム・ドライバ名を指定|
|--label| |ボリュームにメタデータを指定|
|--name| |ボリューム名を指定|
|--opt , -o| |ドライバ固有のオプションを指定|

#### docker volume inspect

ボリュームの詳細情報を表示

```shell-session
$ docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

#### docker volume ls

#### docker volume prune

#### docker volume rm

## 参考

### Dockerのレイヤとイメージサイズの実験

`Dockerfile`で`RUN`, `ADD`, `COPY`を実行する度にレイヤが構築・キャッシュされ、キャッシュによって次回以降のビルドが高速になる。変更頻度の低いものから順に重ねていけばキャッシュを効率良く使えるが、Dockerイメージはレイヤの積層で構成されるため、レイヤが増えるとイメージサイズも大きくなるらしい。Dockerfileを3つ用意して実験。

1.  Ubuntuだけ
    ```dockerfile:Dockerfile1
    # ubuntuのベースイメージそのまま
    FROM ubuntu:latest

    # デフォルトの実行コマンド
    CMD ["bash"]
    ```

2.  ビルドツールを1個ずつ導入
    ```dockerfile:Dockerfile2
    # ubuntuをベースイメージに指定
    FROM ubuntu:latest

    # ビルドツールを1個ずつ導入
    RUN apt update
    RUN apt install -y build-essential
    RUN apt install -y libbz2-dev
    RUN apt install -y libdb-dev
    RUN apt install -y libreadline-dev
    RUN apt install -y libffi-dev
    RUN apt install -y libgdbm-dev
    RUN apt install -y liblzma-dev
    RUN apt install -y libncursesw5-dev
    RUN apt install -y libsqlite3-dev
    RUN apt install -y libssl-dev
    RUN apt install -y zlib1g-dev
    RUN apt install -y uuid-dev
    RUN apt install -y tk-dev

    # デフォルトの実行コマンド
    CMD ["bash"]
    ```

3.  ビルドツールをまとめて導入

    ```dockerfile:Dockerfile3
    # ubuntuをベースイメージに指定
    FROM ubuntu:latest

    # ビルドツールをまとめて導入
    RUN apt update && \
        apt install -y build-essential libbz2-dev libdb-dev libreadline-dev \
                       libffi-dev libgdbm-dev liblzma-dev libncursesw5-dev \
                       libsqlite3-dev libssl-dev zlib1g-dev uuid-dev tk-dev

    # デフォルトの実行コマンド
    CMD ["bash"]
    ```

ビルドしてイメージのサイズを比較
```shell-session
$ docker image build -t testimg0 -f Dockerfile1 .
$ docker image build -t testimg1 -f Dockerfile2 .
$ docker image build -t testimg2 -f Dockerfile3 .
$ docker image ls
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
testimg1         latest    55125c2bfa3e   7 weeks ago      78.1MB
testimg2         latest    a41f517f33ae   3 minutes ago    511MB
testimg3         latest    4d0b6766e19b   13 seconds ago   498MB
```

|Ubuntuのみ|個別インストール|一括インストール|
|--|--|--|
|78.1MB|511MB|498MB|

そこまで極端な差ではない印象。いい塩梅にまとめればよさそう。
各イメージのヒストリ確認。

```shell-session
$ docker image history testimg1
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
55125c2bfa3e   7 weeks ago   CMD ["bash"]                                    0B        buildkit.dockerfile.v0
<missing>      7 weeks ago   /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      7 weeks ago   /bin/sh -c #(nop) ADD file:bcebbf0fddcba5b86…   78.1MB
<missing>      7 weeks ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
<missing>      7 weeks ago   /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
<missing>      7 weeks ago   /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B
<missing>      7 weeks ago   /bin/sh -c #(nop)  ARG RELEASE                  0B
$ #-------------------------------------------------------------------------
$ docker image history testimg2 #大量のhistoryが残っている
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
a41f517f33ae   3 minutes ago   CMD ["bash"]                                    0B        buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y tk-dev # build…   29.7MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y uuid-dev # bui…   1.32MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y zlib1g-dev # b…   1.51MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libssl-dev # b…   14.5MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libsqlite3-dev…   4.38MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libncursesw5-d…   7.31kB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y liblzma-dev # …   1.82MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libgdbm-dev # …   1.42MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libffi-dev # b…   1.39MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libreadline-de…   4.74MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libdb-dev # bu…   4.37MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y libbz2-dev # b…   1.33MB    buildkit.dockerfile.v0
<missing>      3 minutes ago   RUN /bin/sh -c apt install -y build-essentia…   322MB     buildkit.dockerfile.v0
<missing>      4 minutes ago   RUN /bin/sh -c apt update # buildkit            44.6MB    buildkit.dockerfile.v0
<missing>      7 weeks ago     /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      7 weeks ago     /bin/sh -c #(nop) ADD file:bcebbf0fddcba5b86…   78.1MB
<missing>      7 weeks ago     /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
<missing>      7 weeks ago     /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
<missing>      7 weeks ago     /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B
<missing>      7 weeks ago     /bin/sh -c #(nop)  ARG RELEASE                  0B
$ #-------------------------------------------------------------------------
$ docker image history testimg3
IMAGE          CREATED          CREATED BY                                      SIZE      COMMENT
4d0b6766e19b   40 seconds ago   CMD ["bash"]                                    0B        buildkit.dockerfile.v0
<missing>      40 seconds ago   RUN /bin/sh -c apt update &&     apt install…   420MB     buildkit.dockerfile.v0
<missing>      7 weeks ago      /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      7 weeks ago      /bin/sh -c #(nop) ADD file:bcebbf0fddcba5b86…   78.1MB
<missing>      7 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
<missing>      7 weeks ago      /bin/sh -c #(nop)  LABEL org.opencontainers.…   0B
<missing>      7 weeks ago      /bin/sh -c #(nop)  ARG LAUNCHPAD_BUILD_ARCH     0B
<missing>      7 weeks ago      /bin/sh -c #(nop)  ARG RELEASE                  0B
```
