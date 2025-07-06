# Linux

## インストール

Ollama をインストールするには、次のコマンドを実行します:

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

## 手動インストール

> [!NOTE]
> 以前のバージョンからアップグレードする場合は、まず `sudo rm -rf /usr/lib/ollama` で古いライブラリを削除してください。

パッケージをダウンロードして解凍します:

```shell
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```

Ollamaを起動します:

```shell
ollama serve
```

別のターミナルで、Ollama が実行されていることを確認します:

```shell
ollama -v
```

### AMD GPU のインストール

AMD GPU をお持ちの場合は、追加の ROCm パッケージもダウンロードして解凍してください。

```shell
curl -L https://ollama.com/download/ollama-linux-amd64-rocm.tgz -o ollama-linux-amd64-rocm.tgz
sudo tar -C /usr -xzf ollama-linux-amd64-rocm.tgz
```

### ARM64 インストール

ARM64 専用パッケージをダウンロードして解凍します。

```shell
curl -L https://ollama.com/download/ollama-linux-arm64.tgz -o ollama-linux-arm64.tgz
sudo tar -C /usr -xzf ollama-linux-arm64.tgz
```

### Ollama をスタートアップサービスとして追加する（推奨）

Ollama のユーザーとグループを作成します:

```shell
sudo useradd -r -s /bin/false -U -m -d /usr/share/ollama ollama
sudo usermod -a -G ollama $(whoami)
```

`/etc/systemd/system/ollama.service` にサービス ファイルを作成します:

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=$PATH"

[Install]
WantedBy=multi-user.target
```

次にサービスを開始します:

```shell
sudo systemctl daemon-reload
sudo systemctl enable ollama
```

### CUDA ドライバーをインストールします（オプション）

[CUDA をダウンロードしてインストール](https://developer.nvidia.com/cuda-downloads)

以下のコマンドを実行してドライバーがインストールされたことを確認します。GPU の詳細が表示されます:

```shell
nvidia-smi
```

### AMD ROCm ドライバーのインストール（オプション）

[ダウンロードとインストール](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html) ROCm v6.

### Ollama を起動します

Ollama を起動し、実行されていることを確認します:

```shell
sudo systemctl start ollama
sudo systemctl status ollama
```

> [!NOTE]
> AMDは公式Linuxカーネルソースに`amdgpu`ドライバをアップストリーム提供していますが、バージョンが古く、ROCmのすべての機能をサポートしていない可能性があります。Radeon GPUを最適にサポートするには、[AMD](https://www.amd.com/en/support/download/linux-drivers.html)から最新のドライバをインストールすることをお勧めします。

## カスタマイズ

Ollamaのインストールをカスタマイズするには、次のコマンドを実行して、systemdのサービスファイルまたは環境変数を編集します:

```shell
sudo systemctl edit ollama
```

あるいは、`/etc/systemd/system/ollama.service.d/override.conf` にオーバーライド ファイルを手動で作成します。

```ini
[Service]
Environment="OLLAMA_DEBUG=1"
```

## アップデート

インストールスクリプトを再度実行して、Ollama をアップデートします:

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

または、Ollama を再度ダウンロードします。

```shell
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```

## 特定のバージョンのインストール

インストールスクリプトで `OLLAMA_VERSION` 環境変数を使用すると、プレリリース版を含む特定のバージョンの Ollama をインストールできます。バージョン番号は [リリースページ](https://github.com/ollama/ollama/releases) で確認できます。

例:

```shell
curl -fsSL https://ollama.com/install.sh | OLLAMA_VERSION=0.5.7 sh
```

## ログの表示

スタートアップサービスとして実行されているOllamaのログを表示するには、次のコマンドを実行します。

```shell
journalctl -e -u ollama
```

## アンインストール

ollama サービスを削除します。

```shell
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service
```

bin ディレクトリ (`/usr/local/bin`、`/usr/bin`、または `/bin`) から ollama バイナリを削除します:

```shell
sudo rm $(which ollama)
```

ダウンロードしたモデルと Ollama サービスのユーザーとグループを削除します:

```shell
sudo rm -r /usr/share/ollama
sudo userdel ollama
sudo groupdel ollama
```

インストールされたライブラリを削除します:

```shell
sudo rm -rf /usr/local/lib/ollama
```
