# FAQ

## Ollamaをアップグレードするにはどうすればよいですか？

macOSとWindows版のOllamaは、自動的にアップデートをダウンロードします。タスクバーまたはメニューバーの項目をクリックし、「再起動してアップデート」をクリックしてアップデートを適用してください。最新バージョンを[手動で](https://ollama.com/download/)ダウンロードしてアップデートをインストールすることもできます。

Linux では、インストール スクリプトを再実行します:

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

## ログはどのように確認できますか？

ログの使用方法の詳細については、[トラブルシューティング](./troubleshooting.md) ドキュメントをご覧ください。

## 私のGPUはOllamaと互換性がありますか？

[GPUドキュメント](./gpu.md)を参照してください。

## コンテキストウィンドウのサイズを指定するにはどうすればよいですか？

デフォルトでは、Ollama は 4096 トークンのコンテキストウィンドウサイズを使用します。

これは `OLLAMA_CONTEXT_LENGTH` 環境変数で上書きできます。例えば、デフォルトのコンテキストウィンドウを 8K に設定するには、次のようにします。

```shell
OLLAMA_CONTEXT_LENGTH=8192 ollama serve
```

`ollama run` を使用するときにこれを変更するには、`/set parameter` を使用します:

```shell
/set parameter num_ctx 4096
```

API を使用する場合は、`num_ctx` パラメータを指定します:

```shell
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "Why is the sky blue?",
  "options": {
    "num_ctx": 4096
  }
}'
```

## モデルがGPUにロードされたかどうかを確認するにはどうすればよいですか？

現在メモリにロードされているモデルを確認するには、「ollama ps」コマンドを使用してください。

```shell
ollama ps
```

> **Output**:
>
> ```
> NAME      	ID          	SIZE 	PROCESSOR	UNTIL
> llama3:70b	bcfb190ca3a7	42 GB	100% GPU 	4 minutes from now
> ```

`プロセッサ` 列には、モデルがどのメモリにロードされたかが表示されます。
* `100% GPU` は、モデルが完全にGPUにロードされたことを意味します。
* `100% CPU` は、モデルが完全にシステムメモリにロードされたことを意味します。
* `48%/52% CPU/GPU` は、モデルがGPUとシステムメモリの両方に部分的にロードされたことを意味します。

## Ollamaサーバーはどのように設定すればよいですか？

Ollamaサーバーは環境変数を使って設定できます。

### Mac での環境変数の設定

Ollama を macOS アプリケーションとして実行する場合、環境変数は `launchctl` を使用して設定する必要があります。

1. 各環境変数に対して `launchctl setenv` を呼び出します。

    ```bash
    launchctl setenv OLLAMA_HOST "0.0.0.0:11434"
    ```

2. Ollama アプリケーションを再起動します。

### Linux での環境変数の設定

Ollama を systemd サービスとして実行する場合、環境変数は `systemctl` を使用して設定する必要があります。

1. `systemctl edit ollama.service` を実行して、systemd サービスを編集します。エディタが開きます。

2. 各環境変数について、`[Service]` セクションに `Environment` 行を追加します。

    ```ini
    [Service]
    Environment="OLLAMA_HOST=0.0.0.0:11434"
    ```

3. 保存して終了します。

4. `systemd` をリロードし、Ollama を再起動します。

   ```shell
   systemctl daemon-reload
   systemctl restart ollama
   ```

### Windows での環境変数の設定

Windows では、Ollama はユーザー環境変数とシステム環境変数を継承します。

1. まず、タスクバーの Ollama をクリックして終了します。

2. 設定 (Windows 11) またはコントロールパネル (Windows 10) アプリケーションを起動し、「環境変数」を検索します。

3. 「アカウントの環境変数を編集」をクリックします。

4. `OLLAMA_HOST`、`OLLAMA_MODELS` などのユーザーアカウントの環境変数を編集または新規作成します。

5. 「OK/適用」をクリックして保存します。

6. Windows のスタートメニューから Ollama アプリケーションを起動します。

## Ollamaをプロキシ経由で使用するにはどうすればよいですか？

Ollamaはインターネットからモデルを取得するため、モデルへのアクセスにはプロキシサーバーが必要になる場合があります。送信リクエストをプロキシ経由でリダイレクトするには、`HTTPS_PROXY`を使用してください。プロキシ証明書がシステム証明書としてインストールされていることを確認してください。プラットフォームでの環境変数の使用方法については、上記のセクションを参照してください。

> [!NOTE]
> `HTTP_PROXY`の設定は避けてください。Ollamaはモデルの取得にHTTPを使用せず、HTTPSのみを使用します。`HTTP_PROXY`を設定すると、サーバーへのクライアント接続が中断される可能性があります。

### Docker でプロキシの背後にある Ollama を使用するにはどうすればよいですか？

Ollama Docker コンテナイメージは、コンテナの起動時に `-e HTTPS_PROXY=https://proxy.example.com` を渡すことで、プロキシを使用するように設定できます。

あるいは、Docker デーモンをプロキシを使用するように設定することもできます。手順は、[macOS](https://docs.docker.com/desktop/settings/mac/#proxies)、[Windows](https://docs.docker.com/desktop/settings/windows/#proxies)、[Linux](https://docs.docker.com/desktop/settings/linux/#proxies) 上の Docker Desktop、および systemd を使用した Docker [daemon](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy) で提供されています。

HTTPS を使用する場合は、証明書がシステム証明書としてインストールされていることを確認してください。自己署名証明書を使用する場合は、新しい Docker イメージが必要になる場合があります。

```dockerfile
FROM ollama/ollama
COPY my-ca.pem /usr/local/share/ca-certificates/my-ca.crt
RUN update-ca-certificates
```

このイメージをビルドして実行します:

```shell
docker build -t ollama-with-ca .
docker run -d -e HTTPS_PROXY=https://my.proxy.example.com -p 11434:11434 ollama-with-ca
```

## Ollamaは私のプロンプトと回答をollama.comに送信しますか？

いいえ。Ollamaはローカルで実行され、会話データはユーザーのマシンから外部に送信されることはありません。

## Ollama をネットワークに公開するにはどうすればよいですか？

Ollama はデフォルトで 127.0.0.1 のポート 11434 にバインドします。`OLLAMA_HOST` 環境変数でバインドアドレスを変更してください。

お使いのプラットフォームでの環境変数の設定方法については、[上記](#how-do-i-configure-ollama-server) のセクションを参照してください。

## Ollamaをプロキシサーバーで使用するにはどうすればよいですか？

OllamaはHTTPサーバーで動作し、Nginxなどのプロキシサーバーを介して公開できます。そのためには、リクエストを転送するようにプロキシを設定し、必要に応じて必要なヘッダーを設定します（Ollamaをネットワーク上に公開しない場合）。例えば、Nginxの場合は以下のようになります。

```nginx
server {
    listen 80;
    server_name example.com;  # Replace with your domain or IP
    location / {
        proxy_pass http://localhost:11434;
        proxy_set_header Host localhost:11434;
    }
}
```

## Ollamaをngrokで使用するにはどうすればよいですか？

Ollamaは、トンネリングツール用の様々なツールからアクセスできます。例えば、ngrokでは以下のようになります。

```shell
ngrok http 11434 --host-header="localhost:11434"
```

## Ollama を Cloudflare Tunnel と併用するにはどうすればよいですか？

Ollama を Cloudflare Tunnel と併用するには、`--url` フラグと `--http-host-header` フラグを使用します。

```shell
cloudflared tunnel --url http://localhost:11434 --http-host-header="localhost:11434"
```

## Ollama へのアクセスを他のウェブオリジンに許可するにはどうすればよいですか？

Ollama はデフォルトで `127.0.0.1` と `0.0.0.0` からのクロスオリジンリクエストを許可します。追加のオリジンは `OLLAMA_ORIGINS` で設定できます。

ブラウザ拡張機能の場合は、拡張機能のオリジンパターンを明示的に許可する必要があります。すべてのブラウザ拡張機能、または必要に応じて特定の拡張機能へのアクセスを許可する場合は、`OLLAMA_ORIGINS` に `chrome-extension://*`、`moz-extension://*`、`safari-web-extension://*` を含めてください。

```
# Allow all Chrome, Firefox, and Safari extensions
OLLAMA_ORIGINS=chrome-extension://*,moz-extension://*,safari-web-extension://* ollama serve
```

プラットフォームで環境変数を設定する方法については、[上記](#how-do-i-configure-ollama-server)のセクションを参照してください。

## モデルはどこに保存されますか?

- macOS: `~/.ollama/models`
- Linux: `/usr/share/ollama/.ollama/models`
- Windows: `C:\Users\%username%\.ollama\models`

### 別の場所に設定するにはどうすればよいですか？

別のディレクトリを使用する必要がある場合は、環境変数 `OLLAMA_MODELS` を選択したディレクトリに設定してください。

> 注: Linux で標準インストーラを使用する場合、`ollama` ユーザーに指定されたディレクトリへの読み取りおよび書き込み権限が必要です。`ollama` ユーザーにディレクトリを割り当てるには、`sudo chown -R ollama:ollama <directory>` を実行してください。

お使いのプラットフォームでの環境変数の設定方法については、[上記](#how-do-i-configure-ollama-server) のセクションを参照してください。

## Visual Studio Code で Ollama を使用するにはどうすればよいですか？

VSCode や Ollama を活用した他のエディター向けに、既に多数のプラグインが利用可能です。メインリポジトリの Readme の下部にある [拡張機能とプラグイン](https://github.com/ollama/ollama#extensions--plugins) のリストをご覧ください。

## DockerでGPUアクセラレーションを使用してOllamaを使用するにはどうすればよいですか？

Ollama Dockerコンテナは、LinuxまたはWindows（WSL2使用）でGPUアクセラレーションを設定できます。これには[nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit)が必要です。詳細は[ollama/ollama](https://hub.docker.com/r/ollama/ollama)をご覧ください。

GPUパススルーとエミュレーションがないため、macOSのDocker DesktopではGPUアクセラレーションは利用できません。

## Windows 10 の WSL2 でネットワークが遅いのはなぜですか？

これは、Ollama のインストールとモデルのダウンロードの両方に影響する可能性があります。

「コントロール パネル > ネットワークとインターネット > ネットワークの状態とタスクの表示」を開き、左側のパネルで「アダプターの設定の変更」をクリックします。「vEthernel (WSL)」アダプターを見つけて右クリックし、「プロパティ」を選択します。
「構成」をクリックし、「詳細設定」タブを開きます。「Large Send Offload Version 2 (IPv4)」と「Large Send Offload Version 2 (IPv6)」が見つかるまで、それぞれのプロパティを検索します。これらのプロパティは両方とも*無効*にしてください。

## Ollamaにモデルをプリロードして応答時間を短縮するにはどうすればよいですか？

APIを使用している場合は、Ollamaサーバーに空のリクエストを送信することでモデルをプリロードできます。これは、`/api/generate`と`/api/chat`の両方のAPIエンドポイントで機能します。

generateエンドポイントを使用してMistralモデルをプリロードするには、次のコマンドを使用します。

```shell
curl http://localhost:11434/api/generate -d '{"model": "mistral"}'
```

チャット完了エンドポイントを使用するには、以下を使用します:

```shell
curl http://localhost:11434/api/chat -d '{"model": "mistral"}'
```

CLI を使用してモデルをプリロードするには、次のコマンドを使用します:

```shell
ollama run llama3.2 ""
```

## モデルをメモリにロードしたままにしたり、すぐにアンロードしたりするにはどうすればよいですか？

デフォルトでは、モデルはアンロードされるまで5分間メモリに保持されます。これにより、LLMに多数のリクエストを送信する場合の応答時間が短縮されます。モデルをメモリからすぐにアンロードしたい場合は、`ollama stop` コマンドを使用します。

```shell
ollama stop llama3.2
```

API を使用している場合は、`/api/generate` エンドポイントと `/api/chat` エンドポイントで `keep_alive` パラメータを使用して、モデルがメモリ内に保持される時間を設定します。`keep_alive` パラメータには、以下の値を設定できます。
* 期間を表す文字列（例: "10m" または "24h"）
* 秒数（例: 3600）
* 負の数（例: モデルをメモリ内にロードしたままにする）（例: -1 または "-1m"）
* '0'（レスポンス生成後すぐにモデルをアンロードする）

例えば、モデルをプリロードしてメモリ内に残すには、以下のコマンドを使用します。

```shell
curl http://localhost:11434/api/generate -d '{"model": "llama3.2", "keep_alive": -1}'
```

モデルをアンロードしてメモリを解放するには、次を使用します:

```shell
curl http://localhost:11434/api/generate -d '{"model": "llama3.2", "keep_alive": 0}'
```

あるいは、Ollamaサーバーの起動時に`OLLAMA_KEEP_ALIVE`環境変数を設定することで、すべてのモデルがメモリにロードされる時間を変更することもできます。`OLLAMA_KEEP_ALIVE`変数は、前述の`keep_alive`パラメータと同じパラメータ型を使用します。環境変数を正しく設定するには、[Ollamaサーバーの設定方法](#how-do-i-configure-ollama-server)のセクションを参照してください。

`/api/generate` および `/api/chat` API エンドポイントの `keep_alive` API パラメータは、`OLLAMA_KEEP_ALIVE` 設定を上書きします。

## Ollamaサーバーがキューイングできるリクエストの最大数を管理するにはどうすればよいですか？

サーバーにリクエストが多すぎると、サーバーが過負荷であることを示す503エラーが返されます。`OLLAMA_MAX_QUEUE`を設定することで、キューイングできるリクエスト数を調整できます。

## Ollama は同時リクエストをどのように処理しますか？

Ollama は2つのレベルの同時処理をサポートしています。システムに十分な空きメモリ（CPU 推論を使用する場合はシステムメモリ、GPU 推論を使用する場合は VRAM）がある場合、複数のモデルを同時にロードできます。特定のモデルでは、モデルのロード時に十分な空きメモリがある場合、並列リクエスト処理が許可されるように構成されます。

1 つ以上のモデルが既にロードされている状態で、新しいモデルリクエストをロードするための空きメモリが不足している場合、すべての新しいリクエストは新しいモデルがロードされるまでキューに入れられます。以前のモデルがアイドル状態になると、新しいモデルのためのスペースを確保するために、1 つ以上のモデルがアンロードされます。キューに入れられたリクエストは順番に処理されます。GPU 推論を使用する場合、同時モデルのロードを可能にするには、新しいモデルが VRAM に完全に収まる必要があります。

特定のモデルに対する並列リクエスト処理では、並列リクエストの数だけコンテキストサイズが増加します。たとえば、2KB のコンテキストで 4 つの並列リクエストを処理する場合、8KB のコンテキストと追加のメモリ割り当てが必要になります。

以下のサーバー設定は、ほとんどのプラットフォームにおいて、Ollamaが同時リクエストを処理する方法を調整するために使用できます。

- `OLLAMA_MAX_LOADED_MODELS` - 利用可能なメモリに収まる場合に、同時にロードできるモデルの最大数。デフォルトはGPU数の3倍、またはCPU推論の場合は3です。
- `OLLAMA_NUM_PARALLEL` - 各モデルが同時に処理できる並列リクエストの最大数。デフォルトは、利用可能なメモリに基づいて4または1が自動的に選択されます。
- `OLLAMA_MAX_QUEUE` - ビジー時にOllamaがキューに入れるリクエストの最大数。これを超えると、追加のリクエストは拒否されます。デフォルトは512です。

注：Radeon GPUを搭載したWindowsでは、ROCm v5.7の利用可能なVRAMレポートに関する制限により、現在デフォルトで最大1モデルに設定されています。ROCm v6.2が利用可能になると、Windows Radeonは上記のデフォルトに従います。 Windows 上の Radeon で同時モデル ロードを有効にすることができますが、GPU VRAM に収まる以上のモデルをロードしないようにしてください。

## Ollamaはどのようにして複数のGPUにモデルをロードするのでしょうか？

新しいモデルをロードする際、Ollamaはモデルに必要なVRAMと現在利用可能なVRAMを比較します。モデルが1つのGPUに完全に収まる場合、OllamaはそのGPUにモデルをロードします。これにより、推論中にPCIバスを介して転送されるデータ量が削減されるため、通常は最高のパフォーマンスが得られます。モデルが1つのGPUに完全に収まらない場合は、利用可能なすべてのGPUに分散してロードされます。

## Flash Attention を有効にするにはどうすればよいですか？

Flash Attention は、コンテキストサイズの増加に応じてメモリ使用量を大幅に削減できる、ほとんどの最新モデルに搭載されている機能です。Flash Attention を有効にするには、Ollama サーバーの起動時に `OLLAMA_FLASH_ATTENTION` 環境変数を `1` に設定してください。

## K/Vキャッシュの量子化タイプを設定するにはどうすればよいですか？

Flash Attentionが有効になっている場合、K/Vコンテキストキャッシュを量子化することでメモリ使用量を大幅に削減できます。

Ollamaで量子化されたK/Vキャッシュを使用するには、以下の環境変数を設定します。

- `OLLAMA_KV_CACHE_TYPE` - K/Vキャッシュの量子化タイプ。デフォルトは`f16`です。

> 注：現在、これはグローバルオプションです。つまり、すべてのモデルが指定された量子化タイプで実行されます。

現在利用可能なK/Vキャッシュ量子化タイプは以下のとおりです。

- `f16` - 高精度かつメモリ使用量が多い（デフォルト）。
- `q8_0` - 8ビット量子化。`f16`の約半分のメモリ使用量で、精度はわずかに低下しますが、通常はモデルの品質に目立った影響はありません（f16を使用しない場合に推奨）。
- `q4_0` - 4ビット量子化。`f16`の約4分の1のメモリ使用量で、精度は小～中程度低下しますが、コンテキストサイズが大きい場合はより顕著になる可能性があります。

キャッシュ量子化がモデルの応答品質にどの程度影響するかは、モデルとタスクによって異なります。GQA数が多いモデル（例：Qwen2）は、GQA数が少ないモデルよりも量子化による精度への影響が大きい可能性があります。

メモリ使用量と品質の最適なバランスを見つけるには、さまざまな量子化タイプを試してみる必要があるかもしれません。
