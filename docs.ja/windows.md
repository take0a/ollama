# Ollama Windows

Ollama for Windows へようこそ。

もう WSL は不要です！

Ollama は、NVIDIA および AMD Radeon GPU のサポートを含むネイティブ Windows アプリケーションとして動作します。
Ollama for Windows をインストールすると、Ollama はバックグラウンドで実行され、`ollama` コマンドラインは `cmd`、`powershell`、またはお好みのターミナルアプリケーションで使用できます。通常通り、Ollama [api](./api.md) は `http://localhost:11434` で提供されます。

## システム要件

* Windows 10 22H2以降、HomeまたはPro
* NVIDIAカードをお持ちの場合、NVIDIA 452.39以降のドライバー
* Radeonカードをお持ちの場合、AMD Radeonドライバー https://www.amd.com/en/support

Ollamaは進行状況表示にUnicode文字を使用します。Windows 10の一部の古いターミナルフォントでは、不明な四角形として表示されることがあります。このような場合は、ターミナルのフォント設定を変更してみてください。

## ファイルシステム要件

Ollamaのインストールには管理者権限は不要で、デフォルトでホームディレクトリにインストールされます。バイナリのインストールには少なくとも4GBの空き容量が必要です。Ollamaのインストール後、数十GBから数百GBになることもある大規模言語モデルを保存するための追加の空き容量が必要になります。ホームディレクトリに十分な空き容量がない場合は、バイナリのインストール場所とモデルの保存場所を変更できます。

### インストール場所の変更

Ollamaアプリケーションをホームディレクトリ以外の場所にインストールするには、次のフラグを指定してインストーラを起動してください。

```powershell
OllamaSetup.exe /DIR="d:\some\location"
```

### モデルの場所の変更

Ollamaがダウンロードしたモデルを保存する場所をホームディレクトリではなく変更するには、ユーザーアカウントで環境変数「OLLAMA_MODELS」を設定します。

1. 設定（Windows 11）またはコントロールパネル（Windows 10）を起動し、「環境変数」を検索します。

2. 「アカウントの環境変数を編集」をクリックします。

3. モデルを保存する場所として、ユーザーアカウントの環境変数「OLLAMA_MODELS」を編集または新規作成します。

4. 「OK/適用」をクリックして保存します。

Ollamaがすでに実行されている場合は、トレイアプリケーションを終了し、スタートメニューから再起動するか、環境変数を保存した後に新しいターミナルを起動してください。

## API アクセス

`powershell` からの API アクセスを示す簡単な例を以下に示します。

```powershell
(Invoke-WebRequest -method POST -Body '{"model":"llama3.2", "prompt":"Why is the sky blue?", "stream": false}' -uri http://localhost:11434/api/generate ).Content | ConvertFrom-json
```

## トラブルシューティング

Windows 版 Ollama は、ファイルを複数の場所に保存します。
エクスプローラーウィンドウで `<Ctrl>+R` を押して以下のパスを入力すると、ファイルを表示できます。
- `explorer %LOCALAPPDATA%\Ollama` には、ログとダウンロードしたアップデートが含まれています。
    - *app.log* には、GUI アプリケーションから送信された最新のログが含まれています。
    - *server.log* には、最新のサーバーログが含まれています。
    - *upgrade.log* には、アップグレードのログ出力が含まれています。
- `explorer %LOCALAPPDATA%\Programs\Ollama` には、バイナリが含まれています（インストーラーによってユーザー PATH に追加されます）。
- `explorer %HOMEPATH%\.ollama` には、モデルと設定が含まれています。

## アンインストール

Ollama Windowsインストーラーは、アンインストーラーアプリケーションを登録します。Windows設定の「プログラムの追加と削除」からOllamaをアンインストールできます。

> [!NOTE]
> [OLLAMA_MODELSの場所を変更](#changing-model-location)した場合、インストーラーはダウンロードしたモデルを削除しません。


## スタンドアロンCLI

WindowsにOllamaをインストールする最も簡単な方法は、`OllamaSetup.exe`インストーラを使用することです。

このインストーラは、管理者権限を必要とせずに、アカウントにインストールされます。
Ollamaは最新モデルに対応するために定期的にアップデートされており、このインストーラを使用すると最新の状態を維持できます。

Ollamaをサービスとしてインストールまたは統合する場合は、Ollama CLIとNvidiaのGPUライブラリ依存関係のみを含むスタンドアロンの`ollama-windows-amd64.zip` zipファイルをご利用いただけます。
AMD GPUをお使いの場合は、追加のROCmパッケージ`ollama-windows-amd64-rocm.zip`もダウンロードして、同じディレクトリに解凍してください。
これにより、Ollamaを既存のアプリケーションに組み込んだり、[NSSM](https://nssm.cc/)などのツールを使用して`ollama serve`経由でシステムサービスとして実行したりすることができます。

> [!NOTE]
> 以前のバージョンからアップグレードする場合は、まず古いディレクトリを削除する必要があります。
