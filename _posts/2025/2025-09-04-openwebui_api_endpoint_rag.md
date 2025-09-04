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
    "model": "gemma3:4b",
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
    "model": "gemma3:4b",
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
