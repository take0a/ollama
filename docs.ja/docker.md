# Ollama Dockerイメージ

### CPU only

```shell
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

### Nvidia GPU
[NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html#installation) をインストールします。

#### Aptでインストール
1. リポジトリを設定する

    ```shell
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
        | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
    curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
        | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
        | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    sudo apt-get update
    ```

2. NVIDIA Container Toolkitパッケージをインストールする

    ```shell
    sudo apt-get install -y nvidia-container-toolkit
    ```

#### YumまたはDnfでインストール
1. リポジトリを設定する

    ```shell
    curl -s -L https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo \
        | sudo tee /etc/yum.repos.d/nvidia-container-toolkit.repo
    ```

2. NVIDIA Container Toolkitパッケージをインストールする

    ```shell
    sudo yum install -y nvidia-container-toolkit
    ```

#### Nvidia ドライバーを使用するように Docker を構成する

```shell
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

#### コンテナを起動する

```shell
docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

> [!NOTE]  
> NVIDIA JetPack システムで実行している場合、Ollama は正しい JetPack バージョンを自動的に検出できません。コンテナに環境変数 JETSON_JETPACK=5 または JETSON_JETPACK=6 を渡して、バージョン 5 または 6 を選択してください。

### AMD GPU

AMD GPU を搭載した Docker を使用して Ollama を実行するには、`rocm` タグと次のコマンドを使用します。

```shell
docker run -d --device /dev/kfd --device /dev/dri -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:rocm
```

### モデルをローカルで実行

これでモデルを実行できます:

```shell
docker exec -it ollama ollama run llama3.2
```

### さまざまなモデルを試してみてください

[Ollamaライブラリ](https://ollama.com/library) でさらに多くのモデルを見つけることができます。
