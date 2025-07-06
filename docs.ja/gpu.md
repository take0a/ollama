# GPU
## Nvidia

Ollamaは、Compute Capability 5.0以降、ドライババージョン531以降のNvidia GPUをサポートしています。

お使いのカードがサポートされているかどうかを確認するには、コンピューティングの互換性をご確認ください。
[https://developer.nvidia.com/cuda-gpus](https://developer.nvidia.com/cuda-gpus)

| Compute Capability | Family              | Cards                                                                                                       |
| ------------------ | ------------------- | ----------------------------------------------------------------------------------------------------------- |
| 9.0                | NVIDIA              | `H200` `H100`                                                                                               |
| 8.9                | GeForce RTX 40xx    | `RTX 4090` `RTX 4080 SUPER` `RTX 4080` `RTX 4070 Ti SUPER` `RTX 4070 Ti` `RTX 4070 SUPER` `RTX 4070` `RTX 4060 Ti` `RTX 4060`  |
|                    | NVIDIA Professional | `L4` `L40` `RTX 6000`                                                                                       |
| 8.6                | GeForce RTX 30xx    | `RTX 3090 Ti` `RTX 3090` `RTX 3080 Ti` `RTX 3080` `RTX 3070 Ti` `RTX 3070` `RTX 3060 Ti` `RTX 3060` `RTX 3050 Ti` `RTX 3050`   |
|                    | NVIDIA Professional | `A40` `RTX A6000` `RTX A5000` `RTX A4000` `RTX A3000` `RTX A2000` `A10` `A16` `A2`                          |
| 8.0                | NVIDIA              | `A100` `A30`                                                                                                |
| 7.5                | GeForce GTX/RTX     | `GTX 1650 Ti` `TITAN RTX` `RTX 2080 Ti` `RTX 2080` `RTX 2070` `RTX 2060`                                    |
|                    | NVIDIA Professional | `T4` `RTX 5000` `RTX 4000` `RTX 3000` `T2000` `T1200` `T1000` `T600` `T500`                                 |
|                    | Quadro              | `RTX 8000` `RTX 6000` `RTX 5000` `RTX 4000`                                                                 |
| 7.0                | NVIDIA              | `TITAN V` `V100` `Quadro GV100`                                                                             |
| 6.1                | NVIDIA TITAN        | `TITAN Xp` `TITAN X`                                                                                        |
|                    | GeForce GTX         | `GTX 1080 Ti` `GTX 1080` `GTX 1070 Ti` `GTX 1070` `GTX 1060` `GTX 1050 Ti` `GTX 1050`                       |
|                    | Quadro              | `P6000` `P5200` `P4200` `P3200` `P5000` `P4000` `P3000` `P2200` `P2000` `P1000` `P620` `P600` `P500` `P520` |
|                    | Tesla               | `P40` `P4`                                                                                                  |
| 6.0                | NVIDIA              | `Tesla P100` `Quadro GP100`                                                                                 |
| 5.2                | GeForce GTX         | `GTX TITAN X` `GTX 980 Ti` `GTX 980` `GTX 970` `GTX 960` `GTX 950`                                          |
|                    | Quadro              | `M6000 24GB` `M6000` `M5000` `M5500M` `M4000` `M2200` `M2000` `M620`                                        |
|                    | Tesla               | `M60` `M40`                                                                                                 |
| 5.0                | GeForce GTX         | `GTX 750 Ti` `GTX 750` `NVS 810`                                                                            |
|                    | Quadro              | `K2200` `K1200` `K620` `M1200` `M520` `M5000M` `M4000M` `M3000M` `M2000M` `M1000M` `K620M` `M600M` `M500M`  |

古い GPU をサポートするためにローカルでビルドする場合は、[developer.md](./development.md#linux-cuda-nvidia) を参照してください。

### GPUの選択

システムに複数のNVIDIA GPUが搭載されており、Ollamaが使用するGPUをサブセットに制限したい場合は、「CUDA_VISIBLE_DEVICES」にカンマ区切りのGPUリストを設定できます。
数値IDも使用できますが、順序が変動する可能性があるため、UUIDの方が信頼性が高くなります。
GPUのUUIDは、「nvidia-smi -L」を実行することで確認できます。GPUを無視してCPUを強制的に使用したい場合は、無効なGPU ID（例：-1）を使用してください。

### Linux のサスペンド/レジューム

Linux では、サスペンド/レジュームサイクルの後、Ollama が NVIDIA GPU を検出できず、CPU での実行にフォールバックすることがあります。
このドライバのバグを回避するには、「sudo rmmod nvidia_uvm && sudo modprobe nvidia_uvm」コマンドで NVIDIA UVM ドライバをリロードします。

## AMD Radeon

Ollama は次の AMD GPU をサポートしています:

### Linux Support

| Family         | Cards and accelerators                                                                                                               |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| AMD Radeon RX  | `7900 XTX` `7900 XT` `7900 GRE` `7800 XT` `7700 XT` `7600 XT` `7600` `6950 XT` `6900 XTX` `6900XT` `6800 XT` `6800` `Vega 64` `Vega 56`    |
| AMD Radeon PRO | `W7900` `W7800` `W7700` `W7600` `W7500` `W6900X` `W6800X Duo` `W6800X` `W6800` `V620` `V420` `V340` `V320` `Vega II Duo` `Vega II` `VII` `SSG` |
| AMD Instinct   | `MI300X` `MI300A` `MI300` `MI250X` `MI250` `MI210` `MI200` `MI100` `MI60` `MI50`                                                               |

### Windows サポート

ROCm v6.1 では、Windows で以下の GPU がサポートされます。

| Family         | Cards and accelerators                                                                                                               |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| AMD Radeon RX  | `7900 XTX` `7900 XT` `7900 GRE` `7800 XT` `7700 XT` `7600 XT` `7600` `6950 XT` `6900 XTX` `6900XT` `6800 XT` `6800`    |
| AMD Radeon PRO | `W7900` `W7800` `W7700` `W7600` `W7500` `W6900X` `W6800X Duo` `W6800X` `W6800` `V620` |


### Linux でのオーバーライド

Ollama は AMD ROCm ライブラリを利用しています。このライブラリはすべての AMD GPU をサポートしているわけではありません。
場合によっては、システムに類似した LLVM ターゲットを使用するように強制できます。
例えば、Radeon RX 5400 は `gfx1034` (別名 10.3.4) ですが、ROCm は現在このターゲットをサポートしていません。
最も近いサポートは `gfx1030` です。`x.y.z` 構文で環境変数 `HSA_OVERRIDE_GFX_VERSION` を使用できます。
例えば、システムを RX 5400 で強制的に実行させるには、サーバーの環境変数として `HSA_OVERRIDE_GFX_VERSION="10.3.0"` を設定します。
サポートされていない AMD GPU を使用している場合は、以下のサポートされているタイプの一覧を使用して試してみることができます。

異なるGFXバージョンのGPUが複数ある場合は、環境変数に数値のデバイス番号を追加して個別に設定してください。

例：`HSA_OVERRIDE_GFX_VERSION_0=10.3.0`、`HSA_OVERRIDE_GFX_VERSION_1=11.0.0`

現在、Linuxでサポートされている既知のGPUタイプは以下のLLVMターゲットです。
以下の表は、これらのLLVMターゲットにマッピングされるGPUの例を示しています。

| **LLVM Target** | **An Example GPU** |
|-----------------|---------------------|
| gfx900 | Radeon RX Vega 56 |
| gfx906 | Radeon Instinct MI50 |
| gfx908 | Radeon Instinct MI100 |
| gfx90a | Radeon Instinct MI210 |
| gfx940 | Radeon Instinct MI300 |
| gfx941 | |
| gfx942 | |
| gfx1030 | Radeon PRO V620 |
| gfx1100 | Radeon PRO W7900 |
| gfx1101 | Radeon PRO W7700 |
| gfx1102 | Radeon RX 7600 |

AMDは、ROCm v6の拡張に取り組んでおり、将来のリリースではGPUファミリーのサポート範囲を拡大する予定です。これにより、より多くのGPUのサポートが拡大される予定です。

さらにサポートが必要な場合は、[Discord](https://discord.gg/ollama) にお問い合わせいただくか、[issue](https://github.com/ollama/ollama/issues) を報告してください。

### GPUの選択

システムに複数のAMD GPUが搭載されており、Ollamaが使用するGPUをサブセットに制限したい場合は、`ROCR_VISIBLE_DEVICES`にカンマ区切りのGPUリストを設定できます。
デバイスのリストは`rocminfo`で確認できます。GPUを無視してCPU使用率を強制したい場合は、無効なGPU ID（例：-1）を使用してください。
デバイスを一意に識別するために、数値ではなく`Uuid`を使用してください（Uuidが利用可能な場合）。

### コンテナの権限

一部のLinuxディストリビューションでは、SELinuxによってコンテナがAMD GPUデバイスにアクセスできない場合があります。
ホストシステムで「sudo setsebool container_use_devices=1」を実行すると、コンテナがデバイスを使用できるようになります。

### Metal (Apple GPU)

Ollamaは、Metal APIを介してAppleデバイスのGPUアクセラレーションをサポートします。
