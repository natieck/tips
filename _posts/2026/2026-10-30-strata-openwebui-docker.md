---
title: "Docker with NVIDIA GPU で Strata + Open WebUI を導入する"
date: 2026-10-30T12:00:00+09:00
categories:
  - LLM
tags:
  - Strata
  - Open WebUI
  - Docker
  - CUDA
  - Qwen3.8-Flash-Next
toc: true
---

Ubuntu Server 24.04 LTS をインストールした PC に，Docker with NVIDIA GPU で Strata と Open WebUI を 導入したときの備忘録である．

---

## はじめに

### Strata とは

[Strata](https://github.com/Niko1221/Strata/blob/main/README.ja.md) は Qwen3.8-Flash-Nextなどの「MoE（Mixture of Experts）構造」と「巨大なn-gram検索テーブル（PLE表）」の2つを併せ持つLLM（GGUF 形式）を OpenAI 互換 API として提供できる軽量な推論サーバーである．

主な特徴は以下の通り．

- OpenAI API 互換
- GGUF モデルを高速に実行可能
- NVIDIA GPU に対応
- Open WebUI と容易に連携可能
- コンテナで簡単に導入できる
- モデルの自動ダウンロード機能を備える

主な対応モデル：
- Qwen3.8-Flash-Next 系列 (125B)
- GLM-5.3-Flash (321B)
- DeepSeek-V4.1-Flash (552B)

Open WebUI や各種クライアントからは OpenAI API として利用できるため，ローカル LLM サーバーとして扱いやすい．

### Open WebUI とは

[Open WebUI](https://openwebui.com/) は ChatGPT のような Web ベースのチャットインターフェースである．対応するバックエンドとして，

- OpenAI API
- Ollama
- vLLM
- llama.cpp
- Strata
- 各種 OpenAI 互換 API

などを利用できる．ブラウザから手軽にローカル LLM を利用できるため，個人利用や学内サーバー運用にも適している．

※ ただし，Strata の導入のみでも，ブラウザからチャットを利用できる．

### システム構成

今回の構成は以下の通りである．

```text
+------------------+
|   Open WebUI     |
|     Port 3020    |
+--------+---------+
         |
         | OpenAI API
         |
+--------v---------+
|      Strata      |
|     Port 8585    |
+--------+---------+
         |
         | CUDA
         |
+--------v---------+
| NVIDIA RTX3080Ti |
+------------------+
```

Open WebUI は Strata に対して OpenAI API 互換エンドポイント経由でアクセスする．

### 使用した PC

- CPU : AMD Ryzen 9 5950X (16 Core / 32 Threads)
- Memory : DDR4 32GB
- SSD : 1TB
- GPU : NVIDIA RTX 3080Ti 12GB

通常，Strata で Qwen3.8-Flash-Next を実行するためには 12GB 以上の VRAM をもつ GPU と 64GB 以上のシステム RAM が必要であるが，超軽量量子化モデル Qwen3.8-Flash-Next-Gyro-S （重みサイズ 27.64GiB）を用いることで，メモリ 32GB 程度の環境でも実行可能となる．

### 事前に導入済みのソフトウェア

以下が導入済みであることを前提とする．

以下を導入済みとする．
- NVIDIA CUDA Toolkit
- NVIDIA Container Toolkit
- Docker

GPU が Docker コンテナから認識されることを事前に確認しておく．

```bash
docker run --rm --gpus all ubuntu nvidia-smi
```

正常であれば GPU 情報が表示される．

---

## 作業ディレクトリの作成

任意の作業ディレクトリを作成する．

```bash
mkdir ~/strata
cd ~/strata
```

---

## APIキーの設定

Strata の OpenAI API 互換インターフェースを利用するため API キーを設定する．

`.env` ファイルを作成する．

```bash
nano .env
```

内容は以下のようにする．

```env
STRATA_API_KEY=your-api-key
```

任意の長い文字列を設定しておく．

例：

```env
STRATA_API_KEY=my-secret-key
```

---

## docker-compose.yml の作成

以下の内容で docker-compose.yml を作成する．

```yaml
services:
  strata:
    image: ${STRATA_IMAGE:-ghcr.io/niko1221/strata:0.2.7-cuda13}
    container_name: strata-backend

    restart: unless-stopped
    tty: true
    stdin_open: true

    ports:
      - "8585:8080"

    volumes:
      - ./strata-data:/data

    environment:
      FAMILY: ${STRATA_FAMILY:-gyro}
      MODEL: ${STRATA_MODEL:-Gyro-S}
      CONTEXT: ${STRATA_CONTEXT:-4096}
      PARALLEL: ${STRATA_PARALLEL:-1}
      KV: ${STRATA_KV:-int8}
      GPU: "0"
      API_KEY: ${STRATA_API_KEY}
      TUNE: ${STRATA_TUNE:-skip}

    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities:
                - gpu

    shm_size: "2gb"

    ulimits:
      memlock:
        soft: -1
        hard: -1

    stop_grace_period: 2m

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui-strata

    restart: unless-stopped

    ports:
      - "3020:8080"

    volumes:
      - ./open-webui-data:/app/backend/data

    environment:
      OPENAI_API_BASE_URL: http://strata:8080/v1
      OPENAI_API_KEY: ${STRATA_API_KEY}

    depends_on:
      - strata
```

---

## コンテナの起動

起動する．

```bash
docker compose up -d
```

状態を確認する．

```bash
docker compose ps
```

正常起動していれば，

```text
strata-backend
open-webui-strata
```

の2つのコンテナが表示される．

初回の起動では，モデルのダウンロード，検証，パック作成，GPU/RAM割り当て，モデルロードが行われるため時間がかかる．

---

## Strata の状態確認

Strata のヘルスチェック API にアクセスする．

```bash
curl http://127.0.0.1:8585/health
```

例：

```json
{
  "status": "ok",
  "loaded": true,
  "model": "gyro-s"
}
```

のような応答が返れば正常に動作している．

ブラウザから
```text
http://サーバIPアドレス:8585
```
へアクセスすると，Strata のモニター画面（ダッシュボード）を通じて様々な情報を確認できる．

---

## Open WebUI にアクセス

ブラウザから
```text
http://サーバIPアドレス:3020
```
へアクセスする．

同一マシンからアクセスする場合は
```text
http://localhost:3020
```
でもよい．

初回アクセス時には管理者アカウントを作成する．

メールアドレスは実在する必要はない．

---

## 動作確認

Open WebUI にログイン後，

```text
こんにちは
```

などと入力して応答が返れば構築成功である．

---

## ログの確認

Strata のログ確認

```bash
docker logs -f strata-backend
```

Open WebUI のログ確認

```bash
docker logs -f open-webui-strata
```

---

## コンテナの停止

停止

```bash
docker compose stop
```

停止と削除

```bash
docker compose down
```

---

## モデル設定の変更

使用するモデルやコンテキスト長を変更する場合は `.env` ファイルで設定する．

例：

```env
STRATA_MODEL=Gyro-M
STRATA_CONTEXT=8192
```

変更後に再起動する．

```bash
docker compose down
docker compose up -d
```

---

## GPU 使用状況の確認

推論中の GPU 使用状況は以下で確認できる．

```bash
watch -n 1 nvidia-smi
```

GPU メモリ使用量や GPU 使用率が表示される．