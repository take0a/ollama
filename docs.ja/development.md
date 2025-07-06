# 開発環境

インストールの前提条件:

- [Go](https://go.dev/doc/install)
- C/C++ コンパイラ (例: macOS の場合は Clang、Windows amd64 の場合は [TDM-GCC](https://github.com/jmeubank/tdm-gcc/releases/latest) または Windows arm64 の場合は [llvm-mingw](https://github.com/mstorsjo/llvm-mingw)、Linux の場合は GCC/Clang)。

リポジトリのルートディレクトリから Ollama をビルドして実行します。

```shell
go run . serve
```

## macOS (Apple Silicon)

macOS Apple Siliconは、Ollamaバイナリに組み込まれているMetalをサポートしています。追加の手順は必要ありません。

## macOS (Intel)

前提条件となるものをインストールします:

- [CMake](https://cmake.org/download/) または `brew install cmake`

次に、プロジェクトを設定してビルドします:

```shell
cmake -B build
cmake --build build
```

最後に、Ollama を実行します。

```shell
go run . serve
```

## Windows

インストールの前提条件:

- [CMake](https://cmake.org/download/)
- [Visual Studio 2022](https://visualstudio.microsoft.com/downloads/) (ネイティブ デスクトップ ワークロードを含む)
- (オプション) AMD GPU サポート
    - [ROCm](https://rocm.docs.amd.com/en/latest/)
    - [Ninja](https://github.com/ninja-build/ninja/releases)
- (オプション) NVIDIA GPU サポート
    - [CUDA SDK](https://developer.nvidia.com/cuda-downloads?target_os=Windows&target_arch=x86_64&target_version=11&target_type=exe_network)

次に、プロジェクトを設定してビルドします。

```shell
cmake -B build
cmake --build build --config Release
```

> [!IMPORTANT]
> ROCm のビルドには追加のフラグが必要です:
> ```
> cmake -B build -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
> cmake --build build --config Release
> ```


最後に、Ollama を実行します。

```shell
go run . serve
```

## Windows (ARM)

Windows ARMは現時点では追加のアクセラレーションライブラリをサポートしていません。cmakeは使用せず、`go run`または`go build`を使用してください。

## Linux

インストールの前提条件:

- [CMake](https://cmake.org/download/) または `sudo apt install cmake` または `sudo dnf install cmake`
- (オプション) AMD GPU サポート
    - [ROCm](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/quick-start.html)
- (オプション) NVIDIA GPU サポート
    - [CUDA SDK](https://developer.nvidia.com/cuda-downloads)

> [!IMPORTANT]
> CMake を実行する前に、前提条件となるパッケージが `PATH` に含まれていることを確認してください。


次に、プロジェクトを設定してビルドします:

```shell
cmake -B build
cmake --build build
```

最後に、Ollama を実行します:

```shell
go run . serve
```

## Docker

```shell
docker build .
```

### ROCm

```shell
docker build --build-arg FLAVOR=rocm .
```

## Running tests

テストを実行するには、`go test` を使用します:

```shell
go test ./...
```

> NOTE: In rare cirumstances, you may need to change a package using the new
> "synctest" package in go1.24.
>
> If you do not have the "synctest" package enabled, you will not see build or
> test failures resulting from your change(s), if any, locally, but CI will
> break.
>
> If you see failures in CI, you can either keep pushing changes to see if the
> CI build passes, or you can enable the "synctest" package locally to see the
> failures before pushing.
>
> To enable the "synctest" package for testing, run the following command:
>
> ```shell
> GOEXPERIMENT=synctest go test ./...
> ```
>
> If you wish to enable synctest for all go commands, you can set the
> `GOEXPERIMENT` environment variable in your shell profile or by using:
>
> ```shell
> go env -w GOEXPERIMENT=synctest
> ```
>
> Which will enable the "synctest" package for all go commands without needing
> to set it for all shell sessions.
>
> The synctest package is not required for production builds.

## ライブラリ検出

Ollamaは、`ollama`実行ファイルからの相対パスで以下のアクセラレーションライブラリを検索します。

* `./lib/ollama` (Windows)
* `../lib/ollama` (Linux)
* `.` (macOS)
* `build/lib/ollama` (開発用)

ライブラリが見つからない場合、Ollamaはアクセラレーションライブラリなしで実行されません。