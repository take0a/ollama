# 問題のトラブルシューティング方法

Ollama が期待どおりに動作しない場合があります。何が起こったのかを確認する最善の方法の一つは、ログを確認することです。**Mac** でログを確認するには、次のコマンドを実行します:

```shell
cat ~/.ollama/logs/server.log
```

systemd を搭載した **Linux** システムでは、次のコマンドでログを見つけることができます:

```shell
journalctl -u ollama --no-pager --follow --pager-end 
```

**コンテナ** 内で Ollama を実行すると、ログはコンテナ内の stdout/stderr に出力されます:

```shell
docker logs <container-name>
```

(コンテナ名を確認するには `docker ps` を使用してください)

`ollama serve` をターミナルで手動で実行した場合、ログはそのターミナルに表示されます。

**Windows** で Ollama を実行する場合、ログの保存場所はいくつかあります。`<cmd>+R` を押して以下のコマンドを入力すると、エクスプローラーウィンドウでログを確認できます。
- `explorer %LOCALAPPDATA%\Ollama` でログを表示します。最新のサーバーログは `server.log` に、古いログは `server-#.log` に保存されます。
- `explorer %LOCALAPPDATA%\Programs\Ollama` でバイナリを参照します (インストーラーによってユーザーの PATH に追加されます)。
- `explorer %HOMEPATH%\.ollama` でモデルと設定の保存場所を参照します。

問題のトラブルシューティングに役立つ追加のデバッグログを有効にするには、まず**トレイメニューから実行中のアプリを終了**し、次に PowerShell ターミナルで実行します。

```powershell
$env:OLLAMA_DEBUG="1"
& "ollama app.exe"
```

ログの解釈についてサポートを受けるには、[Discord](https://discord.gg/ollama) に参加してください。

## LLMライブラリ

Ollamaには、様々なGPUとCPUベクトル機能向けにコンパイルされた複数のLLMライブラリが含まれています。Ollamaは、システムの性能に基づいて最適なものを選択しようとします。この自動検出に問題が発生した場合、またはその他の問題（GPUのクラッシュなど）が発生した場合は、特定のLLMライブラリを強制的に使用することで回避できます。`cpu_avx2`が最もパフォーマンスが高く、次に`cpu_avx`、そして最も低速ですが互換性が最も高いのは`cpu`です。macOSのRosettaエミュレーションは`cpu`ライブラリで動作します。

サーバーログには、次のようなメッセージが表示されます（リリースによって異なります）:

```
Dynamic LLM libraries [rocm_v6 cpu cpu_avx cpu_avx2 cuda_v12 rocm_v5]
```

**試験的な LLM ライブラリのオーバーライド**

OLLAMA_LLM_LIBRARY を利用可能な LLM ライブラリのいずれかに設定することで、自動検出をバイパスできます。例えば、CUDA カードを使用しているが、AVX2 ベクターをサポートする CPU LLM ライブラリを強制的に使用したい場合は、次のようにします:

```shell
OLLAMA_LLM_LIBRARY="cpu_avx2" ollama serve
```

CPU にどのような機能があるかは次のように確認できます。

```shell
cat /proc/cpuinfo| grep flags | head -1
```

## Linux への旧バージョンまたはプレリリース版のインストール

Linux で問題が発生し、旧バージョンをインストールしたい場合や、正式リリース前にプレリリース版を試用したい場合は、インストールスクリプトでインストールするバージョンを指定できます。

```shell
curl -fsSL https://ollama.com/install.sh | OLLAMA_VERSION=0.5.7 sh
```

## Linux docker

Ollama が Docker コンテナ内の GPU で動作し、しばらくすると CPU で動作するように切り替わり、サーバーログに GPU 検出の失敗を示すエラーが表示される場合は、Docker で systemd cgroup 管理を無効にすることで解決できます。ホスト上の `/etc/docker/daemon.json` を編集し、docker 設定に `"exec-opts": ["native.cgroupdriver=cgroupfs"]` を追加してください。

## NVIDIA GPU 検出

Ollama は起動時に、システム内の GPU のインベントリを取得し、互換性と使用可能な VRAM 容量を判断します。この検出で GPU が見つからない場合があります。一般的に、最新のドライバーを実行すると最良の結果が得られます。

### Linux NVIDIA トラブルシューティング

コンテナを使用して Ollama を実行している場合は、[docker.md](./docker.md) に記載されているように、まずコンテナランタイムがセットアップされていることを確認してください。

Ollama が GPU の初期化に問題を抱える場合があります。サーバーログを確認すると、「3」（初期化されていない）、「46」（デバイスが利用できない）、「100」（デバイスがない）、「999」（不明）など、さまざまなエラーコードが表示される場合があります。以下のトラブルシューティング手法が問題の解決に役立つ場合があります。

- コンテナを使用している場合、コンテナランタイムは動作していますか？「docker run --gpus all ubuntu nvidia-smi」を試してください。これが機能しない場合は、Ollama が NVIDIA GPU を認識できていません。
- uvm ドライバーはロードされていますか？ `sudo nvidia-modprobe -u`
- nvidia_uvm ドライバーをリロードしてみてください - `sudo rmmod nvidia_uvm` を実行した後、`sudo modprobe nvidia_uvm` を実行してください
- 再起動してみてください
- 最新の NVIDIA ドライバーを使用していることを確認してください

上記のいずれでも問題が解決しない場合は、追加情報を収集し、問題を報告してください。
- `CUDA_ERROR_LEVEL=50` を設定して、再度診断ログを取得してみてください
- dmesg でエラーがないか確認します - `sudo dmesg | grep -i nvrm` および `sudo dmesg | grep -i nvidia`


## AMD GPU の検出

Linux では、AMD GPU にアクセスするには通常、`/dev/kfd` デバイスにアクセスするために `video` グループまたは `render` グループのメンバーシップが必要です。権限が正しく設定されていない場合、Ollama はこれを検出し、サーバーログにエラーを報告します。

コンテナ内で実行している場合、一部の Linux ディストリビューションおよびコンテナランタイムでは、ollama プロセスが GPU にアクセスできないことがあります。ホストシステムで `ls -lnd /dev/kfd /dev/dri /dev/dri/*` を実行して、システム上の **数値** グループ ID を確認し、コンテナに必要なデバイスにアクセスできるように `--group-add ...` 引数を追加してください。例えば、次の出力「crw-rw---- 1 0 44 226, 0 Sep 16 16:55 /dev/dri/card0」では、グループID列は「44」です。

Ollamaが推論にGPUを正しく検出または使用できない場合は、以下の手順で問題を特定できる可能性があります。
- `AMD_LOG_LEVEL=3` AMD HIP/ROCmライブラリのinfoログレベルを有効にします。これにより、より詳細なエラーコードが表示され、問題のトラブルシューティングに役立ちます。
- `OLLAMA_DEBUG=1` GPU検出時に追加情報が報告されます。
- dmesgでamdgpuまたはkfdドライバのエラーを確認します。`sudo dmesg | grep -i amdgpu`および`sudo dmesg | grep -i kfd`

## 複数の AMD GPU

Linux 上で複数の AMD GPU にまたがってモデルをロードする際に意味不明な応答が発生する場合は、以下のガイドをご覧ください。

- https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/mgpu.html#mgpu-known-issues-and-limitations

## Windows ターミナルのエラー

Windows 10 の古いバージョン（例：21H1）には、標準ターミナルプログラムが制御文字を正しく表示しないバグがあることが知られています。その結果、`←[?25h←[?25l` のような長い文字列が表示され、場合によっては「パラメータが正しくありません」というエラーが表示されることがあります。この問題を解決するには、Windows 10 22H1 以降にアップデートしてください。
