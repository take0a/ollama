# Template

Ollamaは、Goの組み込みテンプレートエンジンを基盤とした強力なテンプレートエンジンを提供し、大規模な言語モデル用のプロンプトを構築します。この機能は、モデルを最大限に活用するための貴重なツールです。

## 基本的なテンプレート構造

基本的な Go テンプレートは、主に 3 つの部分で構成されます。

* **レイアウト**: テンプレートの全体的な構造。
* **変数**: テンプレートがレンダリングされる際に実際の値に置き換えられる動的なデータのためのプレースホルダ。
* **関数**: テンプレートのコンテンツを操作するために使用できるカスタム関数またはロジック。

以下に、シンプルなチャットテンプレートの例を示します。

```go
{{- range .Messages }}
{{ .Role }}: {{ .Content }}
{{- end }}
```

この例では、次の要素が含まれています。

* 基本的なメッセージ構造（レイアウト）
* 3 つの変数：「Messages」、「Role」、「Content」（変数）
* アイテムの配列（「range.Messages」）を反復処理して各アイテムを表示するカスタム関数（アクション）

## モデルへのテンプレートの追加

デフォルトでは、Ollamaにインポートされたモデルは`{{ .Prompt }}`というデフォルトテンプレートを使用します。つまり、ユーザー入力はそのままLLMに送信されます。これはテキスト補完やコード補完モデルには適していますが、チャットやインストラクションモデルには不可欠なマーカーが欠けています。

これらのモデルでテンプレートを省略すると、入力を正しくテンプレート化する責任がユーザーに課せられます。テンプレートを追加することで、ユーザーはモデルから最適な結果を簡単に得ることができます。

モデルにテンプレートを追加するには、モデルファイルに`TEMPLATE`コマンドを追加する必要があります。MetaのLlama 3を使用した例を以下に示します。

```dockerfile
FROM llama3.2

TEMPLATE """{{- if .System }}<|start_header_id|>system<|end_header_id|>

{{ .System }}<|eot_id|>
{{- end }}
{{- range .Messages }}<|start_header_id|>{{ .Role }}<|end_header_id|>

{{ .Content }}<|eot_id|>
{{- end }}<|start_header_id|>assistant<|end_header_id|>

"""
```

## 変数

`System` (文字列): システムプロンプト

`Prompt` (文字列): ユーザープロンプト

`Response` (文字列): アシスタントの応答

`Suffix` (文字列): アシスタントの応答の後に挿入されるテキスト

`Messages` (リスト): メッセージのリスト

`Messages[].Role` (文字列): 役割。`system`、`user`、`assistant`、`tool` のいずれか

`Messages[].Content` (文字列): メッセージの内容

`Messages[].ToolCalls` (リスト): モデルが呼び出すツールのリスト

`Messages[].ToolCalls[].Function` (オブジェクト): 呼び出す関数

`Messages[].ToolCalls[].Function.Name` (文字列): 関数名

`Messages[].ToolCalls[].Function.Arguments` (マップ): 引数名と引数値のマッピング

`Tools` (リスト): モデルが使用できるツールのリストアクセス

`Tools[].Type` (文字列): スキーマタイプ。`type` は常に `function` です。

`Tools[].Function` (オブジェクト): 関数定義

`Tools[].Function.Name` (文字列): 関数名

`Tools[].Function.Description` (文字列): 関数の説明

`Tools[].Function.Parameters` (オブジェクト): 関数のパラメータ

`Tools[].Function.Parameters.Type` (文字列): スキーマタイプ。 `type` は常に `object` です

`Tools[].Function.Parameters.Required` (リスト): 必須プロパティのリスト

`Tools[].Function.Parameters.Properties` (マップ): プロパティ名とプロパティ定義のマッピング

`Tools[].Function.Parameters.Properties[].Type` (文字列): プロパティの型

`Tools[].Function.Parameters.Properties[].Description` (文字列): プロパティの説明

`Tools[].Function.Parameters.Properties[].Enum` (リスト): 有効な値のリスト

## ヒントとベストプラクティス

Go テンプレートを使用する際は、以下のヒントとベストプラクティスに留意してください。

* **ドットに注意してください**: `range` や `with` などの制御フロー構造は、`.` の値を変更します。
* **スコープ外の変数**: ルートから始めて、現在スコープ外の変数を参照するには、`$.` を使用します。
* **空白の制御**: 先頭の空白 (`{{-`) と末尾の空白 (`-}}`) を削除するには、`-` を使用します。

## Examples

### Example Messages

#### ChatML

ChatMLは人気のテンプレート形式です。DatabrickのDBRX、IntelのNeural Chat、MicrosoftのOrca 2などのモデルに使用できます。

```go
{{- range .Messages }}<|im_start|>{{ .Role }}
{{ .Content }}<|im_end|>
{{ end }}<|im_start|>assistant
```

### Example Tools

テンプレートに `{{ .Tools }}` ノードを追加することで、モデルにツールサポートを追加できます。この機能は、外部ツールを呼び出すようにトレーニングされたモデルに役立ち、リアルタイムデータの取得や複雑なタスクの実行に役立つ強力なツールとなります。

#### Mistral

Mistral v0.3 および Mixtral 8x22B はツールの呼び出しをサポートしています。

```go
{{- range $index, $_ := .Messages }}
{{- if eq .Role "user" }}
{{- if and (le (len (slice $.Messages $index)) 2) $.Tools }}[AVAILABLE_TOOLS] {{ json $.Tools }}[/AVAILABLE_TOOLS]
{{- end }}[INST] {{ if and (eq (len (slice $.Messages $index)) 1) $.System }}{{ $.System }}

{{ end }}{{ .Content }}[/INST]
{{- else if eq .Role "assistant" }}
{{- if .Content }} {{ .Content }}</s>
{{- else if .ToolCalls }}[TOOL_CALLS] [
{{- range .ToolCalls }}{"name": "{{ .Function.Name }}", "arguments": {{ json .Function.Arguments }}}
{{- end }}]</s>
{{- end }}
{{- else if eq .Role "tool" }}[TOOL_RESULTS] {"content": {{ .Content }}}[/TOOL_RESULTS]
{{- end }}
{{- end }}
```

### 中間補完の例

テンプレートに `{{ .Suffix }}` ノードを追加することで、モデルに中間補完のサポートを追加できます。この機能は、コード補完モデルなど、ユーザー入力の途中でテキストを生成するようにトレーニングされたモデルに役立ちます。

#### CodeLlama

CodeLlama [7B](https://ollama.com/library/codellama:7b-code) と [13B](https://ollama.com/library/codellama:13b-code) コード補完モデルは中間補完をサポートしています。

```go
<PRE> {{ .Prompt }} <SUF>{{ .Suffix }} <MID>
```

> [!NOTE]
> CodeLlama 34B および 70B コード補完、およびすべての命令と Python の微調整モデルは、中間補完をサポートしていません。

#### Codestral

Codestral [22B](https://ollama.com/library/codestral:22b)は、中間補完をサポートしています。

```go
[SUFFIX]{{ .Suffix }}[PREFIX] {{ .Prompt }}
```
