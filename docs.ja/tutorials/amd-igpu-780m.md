# AMD iGPU 780M で Ollama を実行

Ollama は、ROCm ベースの Linux 環境で、AMD Ryzen CPU の iGPU 780M 上で動作しました。RX7000 シリーズなどの Radeon dGPU と比べて、若干の設定変更が必要です。

## 使用上の注意
- Ryzen 7000s/8000s CPU（iGPU 780M搭載）
- amdgpuドライバーとrocm6.0
- Linux OSが必要です（WindowsとWSL2はサポートされていません）
- BIOSでiGPUを有効にし、1GB以上のRAMをVRAMに割り当てる必要があります
- HSA_OVERRIDE_GFX_VERSION="11.0.0" が設定されています（AMD iGPU-780M向けの追加設定）

## 前提条件
0. BIOSでiGPUのUMAを設定します。(最低1GB以上、Llama3:8b q4_0モデルの場合は8GB以上を推奨。サイズは4.7GBです)
1. GPUドライバーとROCmをインストールします。
参照：
- [AMD ROCm™ドキュメント - ROCmドキュメント](https://rocmdocs.amd.com/en/latest/)
- [AMD ROCm™クイックスタートインストール](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html#rocm-install-quick)

2. Ollamaをインストールします。
	
 	*`curl -fsSL https://ollama.com/install.sh | sh`*

## 手順
iGPUはデフォルトではOllamaによって検出されません。有効にするには追加の手順が必要です。
1. ollama.serviceを停止します
   
	`sudo systemctl stop ollama.service`
	   
2. ollama.service 設定を変更して、iGPU 780 w/ ROCm で ROCm を有効にします (WSL では動作しないため、Linux で実行する必要があります)

	`sudo systemctl edit ollama.service`

	内容を /etc/systemd/system/ollama.service.d/override.conf に追加して保存します。

	```
	[Service]
	Environment="HSA_OVERRIDE_GFX_VERSION=11.0.0"
	```

	次に、新しい設定で ollama.service を再起動します。

	  `sudo systemctl restart ollama.service`

3. ollamaでllmを実行する
   
   `ollama run tinyllama`
   
   ROCm で ollama を実行するときは、rocm-smi を使用して iGPU の使用率を監視します。


### GPU使用率を確認

ollamaでllmを実行した際にGPUが動作しているかどうかを確認するには、「ollama ps」を実行します。

```
$ ollama ps
NAME            ID              SIZE    PROCESSOR       UNTIL
llama2:latest   78e26419b446    5.6 GB  100% GPU        4 minutes from now
```

**Examples of iGPU 780M w/ ROCm** 
```
$HSA_OVERRIDE_GFX_VERSION="11.0.0" /usr/local/bin/ollama serve &

$ollama run llama2:latest "where was beethoven born?" --verbose
	
	Ludwig van Beethoven was born in Bonn, Germany on December 16, 1770.
	total duration:       4.385911867s
	load duration:        2.524807278s
	prompt eval count:    27 token(s)
	prompt eval duration: 465.157ms
	prompt eval rate:     58.04 tokens/s
	eval count:           26 token(s)
	eval duration:        1.349772s
	eval rate:            19.26 tokens/s
```

## ベンチマーク

**テストプラットフォーム**：AOOSTAR GEM12 AMD Ryzen 7 8845HS ミニPC

**ベンチマークコマンド**：

`ollama run tinyllama "where was beethoven born?" --verbose`

`for run in {1..10}; do echo "where was beethoven born?" | ollama run tinyllama --verbose 2>&1 >/dev/null | grep "eval rate:"; done`   
 
| Model          | Model Size | Radeon 780M<br>(@ubuntu+ROCm6) |
| -------------- | ---------- | --------------------------- |
| tinyllama      | 637MB      | 92                          |
| llama2:latest  | 3.8GB      | 18                          |
| llama2-chinese | 3.8GB      | 18                          |
| llama3:8b      | 4.7GB      | 16                          |
| qwen:1.8b      | 1.1GB      | 61                          |

*注*
- パフォーマンス（トークン/秒）
- Ollamaでは、LLMはデフォルトでQ4_0として量子化されます
