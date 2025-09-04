---
title: "Open WebUI の API エンドポイントを利用した RAG 構築"
date: 2025-09-04T12:30:00+09:00
categories:
  - LLM
tags:
  - Open WebUI
  - API
  - Endpoint
  - RAG
toc: true
---

Docker で導入した Open WebUI の API エンドポイントを利用して RAG を構築する方法の備忘録

## はじめに

Open WebUI の API エンドポイントを利用すると，LLM と連携して様々な処理を Python などで自動化することができる（[Docker で導入した Open WebUI + Ollama の API エンドポイントの使い方](https://natieck.github.io/tips/llm/openwebui_ollama_api_endpoint/)）．

以下では，Open WebUI の API エンドポイントを利用して RAG を構築する方法を説明する．

## 検索拡張生成（RAG）

検索拡張生成（RAG）機能により，外部ソースからのデータを組み込むことで応答を向上させることができる．ここでは，API経由でファイルと知識コレクションを管理し，チャット補完で効果的に使用する方法を説明する．

### ファイルのアップロード

RAG 応答で外部データを活用するには，まず外部ソースデータとなるファイルをアップロードする必要がある．アップロードされたファイルの内容は自動的に抽出され，ベクトルデータベースに保存される．

- **エンドポイント**: `POST /api/v1/files/`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/v1/files/ \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Accept: application/json" \
  -F "file=@/path/to/your/file" 
  ```
  ※ 上のコマンドにおいて，YOUR_API_KEY の部分を API キー に置き換え，`/path/to/your/file` の部分をアップロードするファイルのパスに置き換える．

  これを実行すると，以下のように出力の最初に `"id":"file-id"`（ファイルID）が表示される．

  ```bash
  {"id":"file-id","user_id":"....",....}
  ```

  ここで表示されたファイルID `file-id` を，下記の **知識コレクション（知識ベース）へのファイル追加** や **チャット補完での個別ファイル使用** で指定される `your-file-id-here` に入力する． 

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

### 知識コレクション（ナレッジベース）へのファイル追加

ファイルのアップロード後，それらのファイルを知識コレクション（ナレッジベース）にグループ化したり，チャットで個別に参照したりすることができる．事前に知識コレクション（ナレッジベース）を作成し，`knowledge_id`（ナレッジID）を確認しておく（ブラウザで Open WebUI にアクセスし，「ワークスペース」->「ナレッジベース」でナレッジベースを作成後，そのナレッジベースを選択するとブラウザの URL に `localhost:3000/workspace/knowledge/{knowledge_id}` のようにナレッジIDが表示される）．

- **エンドポイント**: `POST /api/v1/knowledge/{knowledge_id}/file/add`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/v1/knowledge/{knowledge_id}/file/add \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"file_id": "your-file-id-here"}'
  ```
  ※ `knowledge_id` には上で説明したナレッジIDを入力し，`your-file-id-here` にはファイルをアップロードしたときに出力されるファイルIDを入力する．

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
  ※ `knowledge_id` には上で説明したナレッジIDを入力し，`your-file-id-here` にはファイルをアップロードしたときに出力されるファイルIDを入力する．

<br />

### チャット補完でのファイルとコレクションの使用

チャット補完において，以下のように RAG の照会で個別ファイルまたは知識コレクション全体を参照することができる．

#### チャット補完での個別ファイルの使用

この方法は，チャットモデルの応答において特定のファイルの内容に焦点を当てたい場合に有効である．

- **エンドポイント**: `POST /api/chat/completions`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma3:4b",
    "messages": [
      {"role": "user", "content": "この文書ファイルの内容を簡単にまとめてください．"}
    ],
    "files": [
      {"type": "file", "id": "your-file-id-here"}
    ]
  }'
  ```
  ※ 上記の "model" の "gemma3:4b" は実際に導入されているモデルにし，`your-file-id-here` にはファイルをアップロードしたときに出力されるファイルIDを入力する．

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

#### チャット補完での知識コレクション（ナレッジベース）の使用

チャットの質問がより広いコンテキストや複数のドキュメントから参照した方がよい場合は，知識コレクション（ナレッジベース）を活用して応答を向上させることができる．

- **エンドポイント**: `POST /api/chat/completions`

<br />

- **cURL による例**:
  ```bash
  curl -X POST http://localhost:3000/api/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemma3:4b",
    "messages": [
      {"role": "user", "content": "この知識コレクションで取り上げられている要点をまとめてください．"}
    ],
    "files": [
      {"type": "collection", "id": "your-collection-id-here"}
    ]
  }'
  ```
  ※ 上記の "model" の "gemma3:4b" は実際に導入されているモデルにし，`your-collection-id-here` にはナレッジID（**知識コレクション（ナレッジベース）へのファイル追加** で指定した `knowledge_id`）を入力する．

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

これらの方法により，アップロードされたファイルと管理された知識コレクション（ナレッジベース）を通じて外部の知識を効果的に活用し，Open WebUI API を使用したチャットアプリケーションの機能を向上させることができる．ファイルを個別に使用する場合も，知識コレクション（ナレッジベース）内で使用する場合も，特定のニーズに基づいてカスタマイズできる．
