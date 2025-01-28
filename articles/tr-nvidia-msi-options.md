---
title: "nvidia-smiのオプション"
emoji: "✍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nvidia","cuda"]
published: false
---

毎回ヘルプ見るのめんどいので一覧化。暇を見て少しずつ更新。
※訳: だいたいGemini先生

```
nvidia-smi [OPTION1 [ARG1]] [OPTION2 [ARG2]] ...
```

## よく使うオプション

|オプション|説明|使用例|
|--|--|--|
|`-L`, `--list-gpus`|システムに接続されているGPUのリストを表示します。| |
|`-q`, `--query-gpu`|GPUに関する情報を問い合わせます。| |
|`--format=csv`|-qオプションと組み合わせて、CSV形式で出力します。| |
|`--query-gpu=utilization.gpu`|GPUの利用率を表示します。| |
|`--query-gpu=memory.used`|GPUのメモリ使用量を表示します。| |
|`-l`, `--loop`|指定した秒数ごとに情報を更新し続けます。|`nvidia-smi -l 1`|
|`--help`|コマンドのヘルプを表示します。| |

## LIST OPTIONS

|オプション|説明|
|--|--|
|`-L`, `--list-gpus`|システムに接続されている全てのGPUのリストを表示します。|
|`-B`, `--list-excluded-gpus`|システムから除外されているGPUのリストを表示します。通常、このオプションは、特定のGPUをシステムから切り離したり、使用不可にしたりした場合に利用します。|

## SUMMARY OPTIONS

|オプション|説明|使用例|
|--|--|--|
|`-i`, `--id=`|システム内の特定のGPUを指定します。GPUのインデックス番号を指定します。|`nvidia-smi -i 0 --query-gpu=utilization.gpu`|
|`-f`, `--filename=`|コマンドの実行結果を標準出力ではなく、指定したファイルに保存します。|`nvidia-smi -q --format=csv -f gpu_info.csv`|
|`-l, --loop=`|指定した秒数ごとにコマンドを繰り返し実行し、GPUの情報をリアルタイムで監視します。|`nvidia-smi -l 2 --query-gpu=temperature.gpu`|

## QUERY OPTIONS

|オプション|説明|使用例|
|--|--|--|
|`-q`, `--query`|GPU または Unit に関する情報を問い合わせます。| |
|`-u`, `--unit`|Unit (例えば、ECC) に関する属性を表示します。| |
|`-i`, `--id=`|特定の GPU または Unit を指定します。|`nvidia-smi -i 0 -q`|
|`-f`, `--filename=`|出力結果を指定したファイルに保存します。|`nvidia-smi -q --format=csv -f gpu_info.csv`|
|`-x`, `--xml-format`|XML 形式で出力を生成します。|`nvidia-smi -q -x`|
|`--dtd`|XML 出力に DTD を埋め込みます。|`nvidia-smi -q -x --dtd`|
|`-d`, `--display=`|表示する情報を指定します。|`nvidia-smi -q -d TEMPERATURE,UTILIZATION`|
|`-l`, `--loop=`|指定した秒数ごとに情報を更新し続けます。|`nvidia-smi -l 1`|
|`-lms`, `--loop-ms=`|指定したミリ秒ごとに情報を更新し続けます。|`nvidia-smi -lms 100`|

## SELECTIVE QUERY OPTIONS

|オプション|説明|備考|
|--|--|--|
|`--query-gpu`|GPU に関する情報を取得します。|`nvidia-smi --help-query-gpu` を参照してください。|
|`--query-supported-clocks`|サポートされているクロックのリストを取得します。|`nvidia-smi --help-query-supported-clocks` を参照してください。|
|`--query-compute-apps`|現在アクティブなコンピューティングプロセスの一覧を取得します。|`nvidia-smi --help-query-compute-apps` を参照してください。|
|`--query-accounted-apps`|アカウントされたコンピューティングプロセスの一覧を取得します。|`nvidia-smi --help-query-accounted-apps` を参照してください。<br>vGPU ホストではサポートされていません。|
|`--query-retired-pages`|リタイアされたデバイスメモリページの一覧を取得します。|`nvidia-smi --help-query-retired-pages` を参照してください。|
|`--query-remapped-rows`|リマップされた行に関する情報を取得します。|`nvidia-smi --help-query-remapped-rows` を参照してください。|
|`--format=`|(必須)出力形式を指定します。<br>`csv`:カンマ区切り値<br>`noheader`:ヘッダー行を出力しません。<br>`nounits`:数値の単位を出力しません。||
|`-i`, `--id=`|対象とする特定の GPU または Unit を指定します。||
|`-f`, `--filename=`|標準出力ではなく、指定したファイルに出力を記録します。||
|`-l`, `--loop=`|指定した秒間隔で繰り返しプローブを行います。||
|`-lms`, `--loop-ms=`|指定したミリ秒間隔で繰り返しプローブを行います。||

## DEVICE MODIFICATION OPTIONS

|オプション|説明|備考|
|--|--|--|
|`-e`, `--ecc-config=`|ECC サポートの有効/無効を切り替えます。<br> `0`/`DISABLED`: 無効<br> `1`/`ENABLED`: 有効||
|`-p`, `--reset-ecc-errors=`|ECC エラーカウントのリセット方法を指定します。<br> `0`/`VOLATILE`: 揮発的なリセット<br> `1`/`AGGREGATE`: 集計リセット||
|`-c`, `--compute-mode=`|コンピュートアプリケーションのモードを設定します。<br> `0`/`DEFAULT`: デフォルト<br> `1`/`EXCLUSIVE_THREAD` (非推奨): スレッド排他モード<br> `2`/`PROHIBITED`: 禁止<br> `3`/`EXCLUSIVE_PROCESS`: プロセス排他モード||
|`-dm`, `--driver-model=`|GPU ドライバーモデルを設定します。<br> `0`/`WDDM`<br> `1`/`TCC`<br> `2`/`MCDM`||
|`-fdm`, `--force-driver-model=`|GPU ドライバーモデルを強制的に設定します。<br> `0`/`WDDM`<br> `1`/`TCC`<br> `2`/`MCDM`<br> ディスプレイが接続されている場合のエラーを無視します。||
|`--gom=`|GPU 操作モードを設定します。<br> `0`/`ALL_ON`: 全ての機能を有効<br> `1`/`COMPUTE`: コンピュートモード<br> `2`/`LOW_DP`: 低消費電力モード||
|`-lgc`, `--lock-gpu-clocks=`|GPU クロックをロックします。<br> <minGpuClock,maxGpuClock> の形式で指定します (例: `1500,1500`)。<br> 単一のクロック値 (例: <GpuClockValue>) を指定することもできます。<br> `--mode` オプションと組み合わせて使用できます。|アプリケーションのクロック設定を上書きします。|
|`-m`, `--mode=`|`--locked-gpu-clocks` オプションのモードを指定します。<br> 有効なモード: `0`, `1`||
|`-rgc`, `--reset-gpu-clocks`|GPU クロックをデフォルト値にリセットします。||
|`-lmc`, `--lock-memory-clocks=`|メモリクロックをロックします。<br> <minMemClock,maxMemClock> の形式で指定します (例: `5100,5100`)。<br> 単一のクロック値 (例: <MemClockValue>) を指定することもできます。||
|`-rmc`, `--reset-memory-clocks`|メモリクロックをデフォルト値にリセットします。||
|`-lmcd`, `--lock-memory-clocks-deferred=`|メモリクロックをロックします (次回の GPU 初期化時に適用)。<br> カーネルモジュールのアンロードとリロードにより適用されます。|root 権限が必要です。|
|`-rmcd`, `--reset-memory-clocks-deferred`|遅延適用されたメモリクロックのリセットを行います。||
|`-ac`, `--applications-clocks=`|アプリケーション実行時の GPU クロック速度を指定します。<br> <memory,graphics> の形式で指定します (例: `2000,800`)。||
|`-rac`, `--reset-applications-clocks`|アプリケーションクロックをデフォルト値にリセットします。||
|`-pl`, `--power-limit=`|最大電力管理制限をワット単位で指定します。<br> `--scope` オプションと組み合わせて使用できます。||
|`-sc`, `--scope=`|`--scope` のデバイスタイプを指定します。<br> `0`/`GPU`<br> `1`/`TOTAL_MODULE` (Grace Hopper のみ)||
|`-cc`, `--cuda-clocks=`|CUDA クロックを上書きまたはデフォルト値に復元します。<br> CUDA アプリケーション実行時に GPU クロックをより高い周波数に設定します。<br> Volta シリーズ以降の対応デバイスでのみ使用可能です。<br> 管理者権限が必要です。<br> `0`/`RESTORE_DEFAULT`: デフォルト値に復元<br> `1`/`OVERRIDE`: 上書き||
|`-am`, `--accounting-mode=`|アカウントモードを有効/無効にします。<br> `0`/`DISABLED`: 無効<br> `1`/`ENABLED`: 有効||
|`-caa`, `--clear-accounted-apps`|バッファ内のアカウントされた PID を全てクリアします。||
|`--auto-boost-default=`|デフォルトの自動ブーストポリシーを設定します。<br> `0`/`DISABLED`: 無効<br> `1`/`ENABLED`: 有効<br> 最後のブーストクライアントが終了した後にのみ変更が適用されます。||
|`--auto-boost-permission=`|自動ブーストモードに対する非管理者/root ユーザーの制御を許可します。<br> `0`/`UNRESTRICTED`: 制限なし<br> `1`/`RESTRICTED`: 制限あり||
|`-mig`, `--multi-instance-gpu=`|マルチインスタンス GPU を有効/無効にします。<br> `0`/`DISABLED`: 無効<br> `1`/`ENABLED`: 有効|root 権限が必要です。|
|`-gtt`, `--gpu-target-temp=`|GPU の目標温度 (摂氏) を設定します。|管理者権限が必要です。|
|`-i`, `--id=`|対象のGPUを指定します。|
|`-eow`, `--error-on-warning`|警告について非ゼロ値の結果を返却します。|

## UNIT MODIFICATION OPTIONS

|オプション|説明|
|--|--|
|`-t`,`--toggle-led=`|ユニットのLEDを設定します。`0`/`GREEN`, `1`/`AMBER`|
|`-i`, `--id=`|対象のユニットを指定します。|

## SHOW DTD OPTIONS

|オプション|説明|
|--|--|
|`--dtd`|デバイス DTD を出力して終了します。|
|`-f`, `--filename=`|標準出力ではなく、指定したファイルに出力を記録します。|
|`-u`, `--unit`|デバイスではなく、ユニットの DTD を表示します。|
|`--debug=`|指定したファイルに暗号化されたデバッグ情報を記録します。|

## Device Monitoring

|オプション|説明|備考|
|--|--|--|
|`dmon`|スクロール形式でデバイスの統計情報を表示します。|`nvidia-smi dmon -h` を参照してください。|
|`daemon`|デーモンプロセスとしてバックグラウンドで実行され、デバイスを監視します。<br>実験的な機能です。<br>Windows ベアメタルではサポートされていません。<br>`nvidia-smi daemon -h` を参照してください。||
|`replay`|デーモンによって生成された永続的な統計情報を再生/抽出するために使用します。<br>実験的な機能です。<br>`nvidia-smi replay -h` を参照してください。||

## Process Monitoring

|オプション|説明|備考|
|--|--|--|
|`pmon`|スクロール形式でプロセスの統計情報を表示します。|`nvidia-smi pmon -h` を参照してください。|

## NVLINK

|オプション|説明|備考|
|--|--|--|
|`nvlink`|デバイスの NVLink 情報を表示します。|`nvidia-smi nvlink -h` を参照してください。|

## C2C

|オプション|説明|備考|
|--|--|--|
|`c2c`|デバイスの C2C (Chip-to-Chip) 情報を表示します。|`nvidia-smi c2c -h` を参照してください。|

## CLOCKS

|オプション|説明|備考|
|--|--|--|
|`clocks`|クロック情報を制御およびクエリします。|`nvidia-smi clocks -h` を参照してください。|

##  ENCODER SESSIONS

|オプション|説明|備考|
|--|--|--|
|`encodersessions`|デバイスのエンコーダーセッション情報を表示します。|`nvidia-smi encodersessions -h` を参照してください。|

## FBC SESSIONS

|オプション|説明|備考|
|--|--|--|
|`fbcsessions`|デバイスの FBC (Frame Buffer Compression) セッション情報を表示します。|`nvidia-smi fbcsessions -h` を参照してください。|

## MIG

|オプション|説明|備考|
|--|--|--|
|`mig`|MIG (Multi-Instance GPU) の管理に関する機能を提供します。|`nvidia-smi mig -h` を参照してください。|

## COMPUTE POLICY

|オプション|説明|備考|
|--|--|--|
|`compute-policy`|コンピュートポリシーを制御およびクエリします。|`nvidia-smi compute-policy -h` を参照してください。|

## BOOST SLIDER

|オプション|説明|備考|
|--|--|--|
|`boost-slider`|ブーストスライダーを制御およびクエリします。|`nvidia-smi boost-slider -h` を参照してください。|

## POWER HINT

|オプション|説明|備考|
|--|--|--|
|`power-hint`|GPU の電力使用量を推定します。|`nvidia-smi power-hint -h` を参照してください。|

## BASE CLOCKS

|オプション|説明|備考|
|--|--|--|
|`base-clocks`|GPU のベースクロックをクエリします。|`nvidia-smi base-clocks -h` を参照してください。|

## GPU PERFORMANCE MONITORING

|オプション|説明|備考|
|--|--|--|
|`gpm`|GPU Performance Monitoring Unit を制御およびクエリします。|`nvidia-smi gpm -h` を参照してください。|
|`pci`|デバイスの PCI 情報を表示します。|`nvidia-smi pci -h` を参照してください。|
