---
title: "Docker で導入した Open WebUI + Ollama の API エンドポイントの使い方"
date: 2025-08-26T10:30:00+09:00
last_modified_at: 2026-3-07T010:30:30+09:00
categories:
  - LLM
tags:
  - Open WebUI
  - Ollama
  - API
  - Endpoint
toc: true
---

Docker で導入した Open WebUI + Ollama において，LLM と連携して自動化するための API エンドポイントの使い方の備忘録

## はじめに

API エンドポイントとは，API が提供する特定の機能やデータに外部クライアントがアクセスするための，サーバー上にある URL などのデジタル空間に公開された場所を指す．その場所を通じて API によるリクエストの受け取りとレスポンスの送信が行われる．

Open WebUI の API エンドポイントを利用することで，導入された LLM と連携して様々な処理を Python などで自動化することができる．  
※ この仕組みは実験段階であり，将来的に改善や変更が加えられる可能性がある．

Open WebUI の [Quick Start ページ](https://docs.openwebui.com/#open-webui-bundled-with-ollama)の Open WebUI Bundled with Ollama(With GPU Support) に記載された方法で，Docker を用いて Ollama + Open WebUI が既に導入済みとする．

参照：[Docker with NVIDIA GPU で Ollama + Open WebUI を導入する](https://natieck.github.io/tips/llm/ollama-openwebui-docker/)

## 認証

Open WebUI の API へアクセスするためには認証が必要となる．認証は Bearer Token 方式で行われ，Open WebUI の「設定」->「アカウント」で取得できる API キー，または JWT（JSON Web Token）を使用して認証する．API キーが表示されない場合は，「管理者パネル」->「設定」->「一般」の「API キーを有効にする」をオンにする．

<span style="font-size:small;color:gray;">*Bearer Token 方式：API認証においてクライアントが発行されたアクセストークンをHTTPリクエストの「Authorization」ヘッダーに付与してサーバーに送信する方式</span>

![GET_API_KEY]({{site.baseurl}}/images/open_webui_get_api_key.png)

上図の API キーを表示させて，キーの文字列をコピーし，以下に出てくる YOUR_API_KEY の部分に貼り付ける．

## 主な API エンドポイント

### 利用可能なモデル一覧を取得

* **エンドポイント**: `GET /api/models`
* **説明**: Open WebUI を通じて作成または追加されたすべてのモデルを取得する．
* **cURL による使用例**:
  ```bash
  curl -H "Authorization: Bearer YOUR_API_KEY" http://localhost:3000/api/models
  ```
  ※ 上のコマンドにおいて YOUR_API_KEY の部分を API キー に置き換える．

ただ，このコマンドで出力される結果は非常に見難く，後述の Ollama API を介したモデル一覧で出力させた方が見易い．

### チャット補完

- **エンドポイント**: `POST /api/chat/completions`
- **説明**: Open WebUI 上のモデル（Ollamaモデル，OpenAIモデル，Open WebUI Function モデルを含む）向けの OpenAI API 互換チャット補完エンドポイントとして機能する．

<br />

- **cURL による使用例**:
  ```bash
  curl -X POST http://localhost:3000/api/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma3:4b",
    "messages": [
      {
        "role": "user",
        "content": "こんにちは！"
      }
    ]
  }'
  ```
  ※ 上のコマンドにおいて，YOUR_API_KEY の部分を API キー に置き換え，"model" の "gemma3:4b" は実際に導入されているモデルにする．ここではモデルとして Gemma3 (Google) の パラメータ数 4B のものを使用している．

<br />

- **Python による使用例**:
  ```python
  import requests

  def chat_with_model(token):
      url = 'http://localhost:3000/api/chat/completions'
      headers = {
          'Authorization': f'Bearer {token}',
          'Content-Type': 'application/json'
      }
      data = {
          "model": "gemma3:4b",
          "messages": [
              {
                  "role": "user",
                  "content": "こんにちは！"
              }
          ]
      }
      response = requests.post(url, headers=headers, json=data)
      return response.json()

  print(chat_with_model('YOUR_API_KEY'))
  ```
  ※ 上記の "model" の "gemma3:4b" は実際に導入されているモデルにし，最後の行の YOUR_API_KEY の部分を API キー に置き換える．

<br />

- **出力結果**
  ```bash
  {'id': 'gemma3:4b-fd6fe37d-5a5e-45b1-ab8f-ca5173c99ab8', 'created': 1756901585, 'model': 'gemma3:4b', 'choices': [{'index': 0, 'logprobs': None, 'finish_reason': 'stop', 'message': {'role': 'assistant', 'content': 'こんにちは！何かお手伝いできることはありますか？ 😊\n'}}], 'object': 'chat.completion', 'usage': {'response_token/s': 123.97, 'prompt_token/s': 72.96, 'total_duration': 2418639251, 'load_duration': 2154160502, 'prompt_eval_count': 11, 'prompt_tokens': 11, 'prompt_eval_duration': 150777862, 'eval_count': 14, 'completion_tokens': 14, 'eval_duration': 112927849, 'approximate_total': '0h0m2s', 'total_tokens': 25, 'completion_tokens_details': {'reasoning_tokens': 0, 'accepted_prediction_tokens': 0, 'rejected_prediction_tokens': 0}}}
  ```

<br />

### Ollama API プロキシサポート

Ollama モデルと直接やり取りする場合（埋め込み生成や生のプロンプトストリーミングを含む），Open WebUI API を介して Ollama API を利用できる（通常使用される Ollama API で標準的な 11434 ポートは，Ollama がバンドルされた Open WebUI の Docker コンテナでは使えない）．ただし，利用には Open WebUI の認証が必要となる（つまり，curl コマンドでは，`-H "Authorization: Bearer YOUR_API_KEY"` オプションを必ずつけなければならない）．

Ollama API の詳細は [Ollama API ドキュメント](https://github.com/ollama/ollama/blob/main/docs/api.md) を参照．

- **ベースURL**: `/ollama/<api>`

<br />

#### 利用可能なモデル一覧を取得

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" http://localhost:3000/ollama/api/tags
```
※ 上のコマンドにおいて YOUR_API_KEY の部分を Open WebUI の API キー に置き換える．

- **出力結果**
```bash
{"models":[{"name":"phi4-mini-reasoning:3.8b","model":"phi4-mini-reasoning:3.8b","modified_at":"2025-08-22T08:53:54.845553249Z","size":3152479391,"digest":"3ca8c2865ce91b6be853a25e56edfefa4f473db55a391611989b753ecf0fa419","details":{"parent_model":"","format":"gguf","family":"phi3","families":["phi3"],"parameter_size":"3.8B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]},{"name":"mistral:7b","model":"mistral:7b","modified_at":"2025-08-22T07:59:25.197413575Z","size":4372824384,"digest":"6577803aa9a036369e481d648a2baebb381ebc6e897f2bb9a766a2aa7bfbc1cf","details":{"parent_model":"","format":"gguf","family":"llama","families":["llama"],"parameter_size":"7.2B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]},{"name":"gpt-oss:20b","model":"gpt-oss:20b","modified_at":"2025-08-21T08:05:09.246764891Z","size":13780173724,"digest":"aa4295ac10c3afb60e6d711289fc6896f5aef82258997b9efdaed6d0cc4cd8b8","details":{"parent_model":"","format":"gguf","family":"gptoss","families":["gptoss"],"parameter_size":"20.9B","quantization_level":"MXFP4"},"connection_type":"local","urls":[0]},{"name":"huggingface.co/elyza/Llama-3-ELYZA-JP-8B-GGUF:latest","model":"huggingface.co/elyza/Llama-3-ELYZA-JP-8B-GGUF:latest","modified_at":"2025-08-21T01:58:09.03479703Z","size":4920742705,"digest":"df34de06196de585e28e5566769067ad64413b3518e9e93c39f15b7367e54fce","details":{"parent_model":"","format":"gguf","family":"llama","families":["llama"],"parameter_size":"8.03B","quantization_level":"unknown"},"connection_type":"local","urls":[0]},{"name":"huggingface.co/janhq/Jan-v1-4B-GGUF:latest","model":"huggingface.co/janhq/Jan-v1-4B-GGUF:latest","modified_at":"2025-08-21T01:37:21.386917308Z","size":2497283764,"digest":"cefaf98f5786a447034a3df499b582f74feb8b5ce01fca3000b77f6d32d0226f","details":{"parent_model":"","format":"gguf","family":"qwen3","families":["qwen3"],"parameter_size":"4.02B","quantization_level":"unknown"},"connection_type":"local","urls":[0]},{"name":"johnnyboy/qwen2.5-math-7b:latest","model":"johnnyboy/qwen2.5-math-7b:latest","modified_at":"2025-05-11T02:07:28.036691414Z","size":4683076408,"digest":"c8121d6a2d5e8eb1a86b3fbe10dd6c3821fb9dec23753a86434f06524bb77a08","details":{"parent_model":"","format":"gguf","family":"qwen2","families":["qwen2"],"parameter_size":"7.6B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]},{"name":"qwen3:8b","model":"qwen3:8b","modified_at":"2025-05-02T00:59:52.258727732Z","size":5225387923,"digest":"e4b5fd7f8af048d3c02e0357274238a9e93da51936665599ccb957aa42bfe173","details":{"parent_model":"","format":"gguf","family":"qwen3","families":["qwen3"],"parameter_size":"8.2B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]},{"name":"lucas2024/llama-3-elyza-jp-8b:q5_k_m","model":"lucas2024/llama-3-elyza-jp-8b:q5_k_m","modified_at":"2025-04-17T11:04:35.211607576Z","size":5732987760,"digest":"5f598c49d9cef767486fc60b7c61678f4c450c3e57a2c78efb2b720f620cf116","details":{"parent_model":"","format":"gguf","family":"llama","families":["llama"],"parameter_size":"8.0B","quantization_level":"Q5_K_M"},"connection_type":"local","urls":[0]},{"name":"llava:7b","model":"llava:7b","modified_at":"2025-04-17T05:45:09.3572384Z","size":4733363377,"digest":"8dd30f6b0cb19f555f2c7a7ebda861449ea2cc76bf1f44e262931f45fc81d081","details":{"parent_model":"","format":"gguf","family":"llama","families":["llama","clip"],"parameter_size":"7B","quantization_level":"Q4_0"},"connection_type":"local","urls":[0]},{"name":"qwen2.5:7b","model":"qwen2.5:7b","modified_at":"2025-04-17T05:11:22.169027822Z","size":4683087332,"digest":"845dbda0ea48ed749caafd9e6037047aa19acfcfd82e704d7ca97d631a0b697e","details":{"parent_model":"","format":"gguf","family":"qwen2","families":["qwen2"],"parameter_size":"7.6B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]},{"name":"phi4-mini:latest","model":"phi4-mini:latest","modified_at":"2025-04-17T04:56:03.296941547Z","size":2491876774,"digest":"78fad5d182a7c33065e153a5f8ba210754207ba9d91973f57dffa7f487363753","details":{"parent_model":"","format":"gguf","family":"phi3","families":["phi3"],"parameter_size":"3.8B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]},{"name":"deepseek-r1:7b","model":"deepseek-r1:7b","modified_at":"2025-04-17T04:49:36.23297861Z","size":4683075271,"digest":"0a8c266910232fd3291e71e5ba1e058cc5af9d411192cf88b6d30e92b6e73163","details":{"parent_model":"","format":"gguf","family":"qwen2","families":["qwen2"],"parameter_size":"7.6B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]},{"name":"gemma3:4b","model":"gemma3:4b","modified_at":"2025-04-16T13:04:58.937520803Z","size":3338801804,"digest":"a2af6cc3eb7fa8be8504abaf9b04e88f17a119ec3f04a3addf55f92841195f5a","details":{"parent_model":"","format":"gguf","family":"gemma3","families":["gemma3"],"parameter_size":"4.3B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0],"expires_at":1756965676},{"name":"llama3.2:3b","model":"llama3.2:3b","modified_at":"2025-04-16T12:58:01.337558653Z","size":2019393189,"digest":"a80c4f17acd55265feec403c7aef86be0c25983ab279d83f3bcd3abbcb5b8b72","details":{"parent_model":"","format":"gguf","family":"llama","families":["llama"],"parameter_size":"3.2B","quantization_level":"Q4_K_M"},"connection_type":"local","urls":[0]}]}
```

<br />

#### 補完生成（ストリーミング）

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" -H "Content-Type: application/json" http://localhost:3000/ollama/api/generate -d '{"model": "gemma3:4b", "prompt": "こんにちは！"}'
```
※ 上のコマンドにおいて YOUR_API_KEY の部分を Open WebUI の API キー に置き換え，"model" の "gemma3:4b" は実際に導入されているモデルにする．

- **出力結果**
```bash
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.394013609Z","response":"こんにちは","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.456856904Z","response":"！","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.519538024Z","response":"何か","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.581960925Z","response":"お手","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.645470098Z","response":"伝","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.708429663Z","response":"い","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.771287296Z","response":"できる","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.83443665Z","response":"ことは","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.898471836Z","response":"あります","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:14.963824571Z","response":"か","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:15.028227421Z","response":"？","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:15.092128555Z","response":" 😊","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:15.155975947Z","response":"\n","done":false}
{"model":"gemma3:4b","created_at":"2025-09-04T06:27:15.219659749Z","response":"","done":true,"done_reason":"stop","context":[105,2364,107,85141,237354,106,107,105,4368,107,85141,237354,98662,203956,239542,236985,17125,41277,17442,237116,237536,103453,107],"total_duration":2784548390,"load_duration":1837923107,"prompt_eval_count":11,"prompt_eval_duration":119637668,"eval_count":14,"eval_duration":826388202}
```

<br />

- **Python による使用例**

```python
import requests

OLLAMA_URL = "http://localhost:3000/ollama/api/generate"
MODEL_NAME = "gemma3:4b"   # モデルの指定
TEMPERATURE = 0   # 温度の指定（0から1の範囲）
TOKEN = "YOUR_API_KEY"   # ← Open WebUI の API キー に置き換える

def query_ollama(prompt,model=MODEL_NAME, temperature=TEMPERATURE, token=TOKEN):
    headers = {
        'Authorization': f'Bearer {token}',
        'Content-Type': 'application/json'
    }
    payload = {
        "model": model,
        "prompt": prompt,
        "stream": False,
        "options": {
            "temperature": temperature
        }
    }
    response = requests.post(OLLAMA_URL, headers=headers, json=payload)
    response.raise_for_status()
    return response.json()["response"]

print(query_ollama('こんにちは！'))
```

- **出力結果**

```bash
こんにちは！何かお手伝いできることはありますか？ 😊
```

query_ollama の呼び出し時にモデルや温度を指定できるようにしている．また，stream を False にして，ストリームせずに一度に全応答を取得するようにし，応答 (response) のみを返すようにしている．

<br />

#### 埋め込み生成
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" -H "Content-Type: application/json" http://localhost:3000/ollama/api/embed -d '{"model": "llama3.2:3b", "input": ["Open WebUI は素晴らしい！", "埋め込みを生成しよう！"]}'
```
※ 上のコマンドにおいて YOUR_API_KEY の部分を Open WebUI の API キー に置き換え，"model" の "llama3.2:3b" は実際に導入されている埋め込みに対応したモデルにする．

これを実行すると，入力されたテキスト ["Open WebUI は素晴らしい！", "埋め込みを生成しよう！"] を数値ベクトルに変換した結果が出力される．

この埋め込み生成により，Open WebUI 背後の Ollama モデルを使用して検索インデックス，検索システム，またはカスタムパイプラインを構築することができる．
