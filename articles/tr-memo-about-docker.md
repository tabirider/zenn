---
title: "Dockerコマンドリファレンス的な(自分用メモ)"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["docker"]
published: false
---

## Dockerの詳細

:::message alert
あとで追記
:::

[Dockerfileのベストプラクティス](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html)

- コンテナは可能な限り一時的(使い捨て)であるべき
- レイヤ数を最小に。[マルチステージビルド](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html#id9)で最終イメージの容量を節約できる
- 不要なパッケージは入れない
- 各コンテナはただ一つの用途を持つべき
- apt installなどはアルファベット順に並べ重複を見つけやすく
- 変更頻度の低い順番からレイヤを作成すると効率的

### Dockerfileファイル

[公式](https://docs.docker.jp/engine/reference/builder.html)参照。`docker image build`時に参照される。
書式:
```dockerfile:Dockerfile
# comment
INCTRUCTION arguments
INCTRUCTION arguments
..
```

- 必ず`FROM`から始める。(コメント、ディレクティブ、`ARG`は例外)
- `RUN`, `COPY`, `ADD`が実行されるとレイヤが構成される。Dockerイメージはレイヤの積み重ねで、各レイヤは前提レイヤからの差分。
- レイヤはキャッシュされる。`COPY`, `ADD`で作成されるレイヤは、そこに含まれるファイルのチェックサムを参照するので、ファイルの中身が変わったらキャッシュは再利用されない(タイムスタンプはチェックサムに含まれない)。`RUN`の場合は単にコマンドの内容だけでキャッシュの利用可否を判断している。`docker build --no-cache`でキャッシュを無視できる。
- `COPY`は単純なファイルコピー。`ADD`はURLからのダウンロード、圧縮ファイルの展開を行う。通常は`COPY`を利用する。


||内容|
|--|--|
|`FROM`|ベースイメージを指定<br>`FROM [--platform=<プラットフォーム>] <イメージ名> [AS <名前>]`<br>`FROM [--platform=<プラットフォーム>] <イメージ名>[:<タグ>] [AS <名前>]`<br>`FROM [--platform=<プラットフォーム>] <イメージ名>[@<ダイジェスト>] [AS <名前>]`<br>`scratch`指定で何もないイメージが作れる。[マルチステージビルド](https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html#id9)で使用|
|`RUN`|直前のレイヤに対してコマンド実行。デフォルトは`/bin/sh -c`。`apt update`, `apt install`等。複数の`RUN`を実行すると複数レイヤができることに留意、レイヤを分ける必要なければ`&&`でまとめる。`\`で複数行分割できる。`install`はアルファベット順に並べておくと重複を見つけやすい|
|`CMD`|`CMD ["実行ファイル", "パラメータ1", "パラメータ2", ..]`<br>コンテナ起動時に実行するコマンド。コンテナは上がった段階で仕事なければすぐ落ちる。`CMD ["bash"]`しておけば勝手に落ちない。`docker run`の引数を指定すると`CMD`の設定が上書きされる|
|`EXPOSE`|`EXPOSE <ポート> [<ポート>/<プロトコル>...]`(プロトコル省略時は`TCP`)<br>ポート公開指示**だが公開されない**。`docker run -p`で割当、`EXPOSE`はメモ代わり。ただし`Docker Compose`では参照する<br>`EXPOSE 80`<br>`EXPOSE 3000 8080`|
|`ENV`|環境変数。作成したコンテナに引き継がれる|
|`ARG`|イメージ構築時、ステージ内だけで有効な変数|
|`ADD`|`ADD [--chown=<ユーザ>:<グループ>] <追加元>... <追加先>`<br>`ADD [--chown=<ユーザ>:<グループ>] ["<追加元>",... "<追加先>"]`<br>通常は`COPY`で足りる。URLから引っ張ったり圧縮ファイルの展開(`tar -x`相当)処理が入る。|
|`COPY`|`[--chown=<ユーザ>:<グループ>] <コピー元>... <コピー先>`<br>`[--chown=<ユーザ>:<グループ>] ["<コピー元>",... "<コピー先>"]`<br>ファイルコピー。`<コピー先>`が相対パスの場合`WORKDIR`からの相対パス。`COPY hoge piyo/`は`hoge`→`<WORKDIR>/piyo/`。絶対パスも可能。コピーされたファイルは`UID=0`, `GID=0`。必要に応じてchown指定|
|`ENTRYPOINT`|`ENTRYPOINT ["実行ファイル", "パラメータ1", "パラメータ2"]`<br>`CMD`と同じ動きをするが、`docker run`で上書きされない。必ず実行するコマンドを指定するのに使用。`docker run`の引数は`ENTRYPOINT`の後に追加される。<br>コンテナを開発環境のように操作する場合は`CMD`, DB/Webサーバのようにバイナリとして扱う場合は`ENTRY`と使い分ける|
|`VOLUME`|`VOLUME ["/data"]`<br>コンテナ内の指定されたパスを**Docker内の匿名の場所**にマウントする。コンテナ内の実在するパスを指定すると、マウントポイントとして上書きされる。コンテナやイメージ削除しても匿名の場所は削除されない。複数コンテナでのファイル共有や永続化に使える**がややこしいので使わない**|
|`USER`|`USER <ユーザ>[:<グループ>]`<br>`USER <UID>[:<GID>]`<br>コンテナ内にそのユーザがないとエラーになる。`RUN useradd`とかしないと使えない|
|`WORKDIR`|作業ディレクトリの指定。なければ作成。以降はDockerfile内の相対パスは`WORKDIR`が起点になる|
|`ONBUILD`|`ONBUILD [命令]`<br>他のイメージ構築ベースにするとき、トリガとして実行する命令。必要になれば確認|
|`STOPSIGNAL`|コンテナ停止時にDockerがプロセスに送信するシグナル。デフォルトでは`SIGTERM`。他に`SIGKILL`(強制終了), `SIGHUP`(プロセスの再読込要求),`SIGUSR1`(アプリ固有のシグナル)等|
|`HEALTHCHECK`|`HEALTHCHECK [オプション] CMD コマンド`<br>`HEALTHCHECK NONE`(ベースイメージからヘルスチェック設定の継承を無効化)<br>Dockerは指示されたテスト方法でコンテナ動作を確認するようになる。<br>`HEALTHCHECK --interval=5m --timeout=3s --retries=3 CMD curl -f http://localhost/ \|\| exit 1`|
|`SHELL`|デフォルトのシェルコマンドを上書き。Winじゃないと使い道なさそう|


### .dockerignoreファイル
docker image build時のディレクトリに`.dockerignore`があれば読み込まれる。
該当するファイルはDockerデーモンに送信されずCOPYもできない。
```:.dockerignore
# comment
*/temp*
*/*/temp*
temp?
.git/*
```

### docker image build

```shell-session
$ docker image build
```

- `build`時、指定ディレクトリ以下の全ファイルがDockerデーモンに送信される。`/`を指定してはならない(全ファイルがDockerデーモンに送信される)。
- イメージビルドにはビルドキャッシュが使われるので2回目以降は高速になる。

### docker volume

`Dockerfile`で`VOLUME`使うときだけ。コンテナにマウントした匿名のボリュームを管理。中身見れないし訳分からなくなるから使いたくない。
```shell-session:WSL
$ #ボリュームの一覧
$ docker volume ls
DRIVER    VOLUME NAME
local     fb1306dd20239a61c07f671b0ec089429d961c90e5a21fbb5471701bd29ad227
$ #ボリューム名の先頭だけ指定しても消せない
$ docker volume rm fb13
Error response from daemon: get fb13: no such volume
$ #これもダメ
$ docker volume rm local
Error response from daemon: get local: no such volume
$ #やっと消えた
$ docker volume rm fb1306dd20239a61c07f671b0ec089429d961c90e5a21fbb5471701bd29ad227
fb1306dd20239a61c07f671b0ec089429d961c90e5a21fbb5471701bd29ad227
```
