---
title: "Docker で導入した Open WebUI + Ollama の API エンドポイントの使い方"
date: 2025-08-26T10:30:00+09:00
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

Open WebUI の API へアクセスするためには認証が必要となる．認証は Bearer Token 方式で行われ，Open WebUI の「設定」->「アカウント」で取得できる API キー，または JWT（JSON Web Token）を使用して認証する．

<span style="font-size:small;color:gray;">*Bearer Token 方式：API認証においてクライアントが発行されたアクセストークンをHTTPリクエストの「Authorization」ヘッダーに付与してサーバーに送信する方式</span>

![GET_API_KEY]({{site.baseurl}}/images/open_webui_get_api_key.png)

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
  ※ 上のコマンドにおいて，"model" の "gemma3:4b" は実際に導入されているモデルにし，YOUR_API_KEY の部分を API キー に置き換える．
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

Ollama モデルと直接やり取りする場合（埋め込み生成や生のプロンプトストリーミングを含む），Open WebUI API を介して Ollama API を利用できる（一般に，Ollama API で標準的な 11434 ポートは，Ollama がバンドルされた Open WebUI の Docker コンテナでは使えない）．ただし，利用には Open WebUI の認証が必要となる．

Ollama API の詳細は [Ollama API ドキュメント](https://github.com/ollama/ollama/blob/main/docs/api.md) を参照．

- **ベースURL**: `/ollama/<api>`

<br />

#### 利用可能なモデル一覧を取得
  ```bash
  curl -H "Authorization: Bearer YOUR_API_KEY" http://localhost:3000/ollama/api/tags
  ```

<br />

#### 補完生成（ストリーミング）
  ```bash
  curl -H "Authorization: Bearer YOUR_API_KEY" -H "Content-Type: application/json" http://localhost:3000/ollama/api/generate -d '{"model": "gemma3:4b", "prompt": "こんにちは！"}'
  ```

<br />

#### 埋め込み生成
  ```bash
  curl -H "Authorization: Bearer YOUR_API_KEY" -H "Content-Type: application/json" http://localhost:3000/ollama/api/embed -d '{"model": "llama3.2:3b", "input": ["Open WebUI は素晴らしい！", "埋め込みを生成しよう！"]}'
  ```

<br />

## 検索拡張生成（RAG）

検索拡張生成（RAG）機能により，外部ソースからのデータを組み込むことで応答を向上させることができる．ここでは，API経由でファイルと知識コレクションを管理し，チャット補完で効果的に使用する方法を説明する．

### ファイルのアップロード

RAG応答で外部データを活用するには，まずファイルをアップロードする必要がある．アップロードされたファイルの内容は自動的に抽出され，ベクトルデータベースに保存される．

- **エンドポイント**: `POST /api/v1/files/`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/v1/files/ \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json" \
  -F "file=@/path/to/your/file" 
  ```

<br />

- **Python による例**:
  ```python
  import requests

  def upload_file(token, file_path):
      url = 'http://localhost:3000/api/v1/files/'
      headers = {
          'Authorization': f'Bearer {token}',
          'Accept': 'application/json'
      }
      files = {'file': open(file_path, 'rb')}
      response = requests.post(url, headers=headers, files=files)
      return response.json()
  ```

<br />

### 知識コレクションへのファイル追加

ファイルのアップロード後，それらのファイルを知識コレクションにグループ化したり，チャットで個別に参照したりすることができる．

- **エンドポイント**: `POST /api/v1/knowledge/{id}/file/add`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/v1/knowledge/{knowledge_id}/file/add \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"file_id": "your-file-id-here"}'
  ```

<br />

- **Python による例**:
  ```python
  import requests

  def add_file_to_knowledge(token, knowledge_id, file_id):
      url = f'http://localhost:3000/api/v1/knowledge/{knowledge_id}/file/add'
      headers = {
          'Authorization': f'Bearer {token}',
          'Content-Type': 'application/json'
      }
      data = {'file_id': file_id}
      response = requests.post(url, headers=headers, json=data)
      return response.json()
  ```

<br />

### チャット補完でのファイルとコレクションの使用

充実した応答を得るために，RAGクエリで個別ファイルまたはコレクション全体を参照できる．

#### チャット補完での個別ファイル使用

この方法は，特定のファイルの内容にチャットモデルの応答を焦点を当てたい場合に有効である．

- **エンドポイント**: `POST /api/chat/completions`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4-turbo",
    "messages": [
      {"role": "user", "content": "Explain the concepts in this document."}
    ],
    "files": [
      {"type": "file", "id": "your-file-id-here"}
    ]
  }'
  ```

<br />

- **Python による例**:
  ```python
  import requests

  def chat_with_file(token, model, query, file_id):
      url = 'http://localhost:3000/api/chat/completions'
      headers = {
          'Authorization': f'Bearer {token}',
          'Content-Type': 'application/json'
      }
      payload = {
          'model': model,
          'messages': [{'role': 'user', 'content': query}],
          'files': [{'type': 'file', 'id': file_id}]
      }
      response = requests.post(url, headers=headers, json=payload)
      return response.json()
  ```

<br />

#### チャット補完での知識コレクション使用

問い合わせがより広いコンテキストや複数のドキュメントから恩恵を受ける可能性がある場合，知識コレクションを活用して応答を向上させる．

- **エンドポイント**: `POST /api/chat/completions`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4-turbo",
    "messages": [
      {"role": "user", "content": "Provide insights on the historical perspectives covered in the collection."}
    ],
    "files": [
      {"type": "collection", "id": "your-collection-id-here"}
    ]
  }'
  ```

<br />

- **Pythonの例**:
  ```python
  import requests

  def chat_with_collection(token, model, query, collection_id):
      url = 'http://localhost:3000/api/chat/completions'
      headers = {
          'Authorization': f'Bearer {token}',
          'Content-Type': 'application/json'
      }
      payload = {
          'model': model,
          'messages': [{'role': 'user', 'content': query}],
          'files': [{'type': 'collection', 'id': collection_id}]
      }
      response = requests.post(url, headers=headers, json=payload)
      return response.json()
  ```

これらの方法により，アップロードされたファイルと管理された知識コレクションを通じて外部知識を効果的に活用し，Open WebUI API を使用したチャットアプリケーションの機能を向上させることができる．ファイルを個別に使用する場合も，コレクション内で使用する場合も，特定のニーズに基づいて統合をカスタマイズできる．
