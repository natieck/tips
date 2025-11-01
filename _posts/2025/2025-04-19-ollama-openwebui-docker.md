---
title: "Docker with NVIDIA GPU で Ollama + Open WebUI を導入する"
date: 2025-04-19T10:30:00+09:00
categories:
  - LLM
tags:
  - Ollama
  - Open WebUI
  - Docker
  - CUDA
toc: true
---

Ubuntu Server 24.04 LTS をインストールした PC に Docker with NVIDIA GPU で Ollama + Open WebUI を導入したときの備忘録

## はじめに

[Ollama](https://ollama.com/)（オープンソースLLMをローカル環境で実行できるプラットフォーム（LLM ランナー））と [Open WebUI](https://openwebui.com/)（LLMランナーを用いて ChatGPT のように手軽に LLM を利用できる Web インターフェース）を NVIDIA GPU を有効にした Docker コンテナで導入する．

PCのスペック：
- CPU: AMD Ryzen 5950X 16Core/32Threads
- Memory： DDR4 32GB
- SSD: 1TB
- GPU: NVIDIA RTX 3080Ti 12GB

以下を導入済みとする．
- NVIDIA CUDA Toolkit
- NVIDIA Container Toolkit
- Docker

参照：[Ubuntu Server 24.04 LTS に CUDA 12.8 をインストールする](https://natieck.github.io/tips/linux/cuda_on_ubuntu_server24/)

## Ollama + Open WebUI コンテナの起動
Open WebUI の [Quick Start ページ](https://docs.openwebui.com/#open-webui-bundled-with-ollama)を参照し，Open WebUI Bundled with Ollama(With GPU Support) の箇所に記載されているコマンドでコンテナを起動するだけで簡単に導入できる．

GPU を用いて起動するコマンド
```bash
docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
```
<br>

dokcer run のオプションの説明：
- `-d`　コンテナを「デタッチ」モード（バックグラウンド）で実行
- `-p 3000:8080`　コンテナの8080番のポートをホストの3000番にマッピング
- `--gpus=aal`　利用可能なすべての GPU を使用
- `-v ollama:/root/.ollama`　ホストの ollama をコンテナの /root/.ollama にマウント
- `-v open-webui:/app/backend/data`　ホストの open-webui をコンテナの /app/backend/data にマウント  
`-v` オプションのホストの volume の指定において，ディレクトリ名のみの場合，デフォルトでは /var/lib/docker/volumes/ 以下のディレクトリとなる（なければ作成される）
- `--name open-webui`　コンテナ名を open-webui に設定
- `--restart always`　コンテナを常に再起動（PC起動時に Docker デーモンが自動でコンテナを起動）

Docker イメージについては，GitHub Container Registry から公開されている最新版の Open WebUI (+ Ollama) Docker イメージを起動する．一度実行すれば，PC起動時に自動的にコンテナが起動し，Open WebUI を利用できる状態となる．

もし GPU を用いずに CPU のみで起動する場合は以下のコマンドを実行する．
```bash
docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
```

### LLM モデルのセットアップ
最初は，LLM が何も入っていないので，セットアップする必要がある．

以下のコマンドで LLM をセットアップできる．
```bash
docker exec open-webui ollama pull モデル名
```
<br>

利用可能なモデルは [Ollama のモデルライブラリ](https://ollama.com/library)で検索できる．

モデルのセットアップの例
```bash
docker exec open-webui ollama pull llama3.2:3b
docker exec open-webui ollama pull gemma3:4b
docker exec open-webui ollama pull phi4-mini:3.8b
docker exec open-webui ollama pull deepseek-r1:7b
docker exec open-webui ollama pull qwen2.5:7b
```
<br>
[HuggingFace](https://huggingface.co/) の GGUF ファイルとして提供されている LLM をセットアップするには，以下のように実行すればよい．

```bash
docker exec open-webui ollama pull huggingface.co/リポジトリ名
```
※ GGUF：C/C++で書かれた機械学習フレームワーク「GGML」で使われるモデルのバイナリファイルの形式

例）[日本語特化モデル ELYZA](https://huggingface.co/elyza) の Llama-3-ELYZA-JP-8B をセットアップする場合
```bash
docker exec open-webui ollama pull huggingface.co/elyza/Llama-3-ELYZA-JP-8B-GGUF
```
<br>

もしくは，下記のように Open WebUI の設定（「設定」->「管理者設定」->「モデル」）から GUI でモデルをセットアップすることもできる．

## Open WebUI にアクセス
上記のように Ollama + Open WebUI コンテナが起動していれば，導入した PC でブラウザを起動して `http://localhost:3000` にアクセスし，Open WebUI を利用できる．

初回アクセス時には，管理者アカウント登録（あくまでもローカルでの登録処理）が必要となるので，名前・メールアドレス・パスワードを設定する（メールアドレスは実在してない適当なものでも構わない）．

Open WebUI で新たなモデルをセットアップする場合，管理者でログインし，「設定」->「管理者設定」->「モデル」を選択し，「モデルの管理」ボタン（ダウンロードボタン）をクリックする．「Ollama.com からモデルをプル」の枠内にモデル名を入力し，ダウンロードボタンをクリックする．モデル名は [Ollama のモデルライブラリ](https://ollama.com/library)にあるものを入力すればよい．

### LAN内の他の端末から Open WebUI にアクセスできるようにする
ファイヤーウォールが有効（アクティブ）であることを確認
```bash
sudo ufw status
```
有効でなければ有効化
```bash
sudo ufw enable
```
ファイヤーウォールの設定で 3000番のポートの通信を許可
```bash
sudo ufw allow 3000
```

他の端末からは `http://IPアドレス:3000` でアクセスできる（IPアドレスは Open WebUI を導入した PC のIPアドレスを指定）．

ufw がインストールされていなければ iptables で設定する．
ファイヤーウォールが有効（アクティブ）であることを確認
```bash
sudo systemctl is-enabled iptables
```
有効でなければ起動
```bash
sudo systemctl enable iptables
```
TCPの3000番ポートへの受信を許可する
```bash
sudo iptables -A INPUT -p tcp --dport 3000 -j ACCEPT
```
設定の確認
```bash
sudo iptables -L INPUT -n --line-numbers
```
以下のように表示されれば OK
```bash
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3000
```
上記設定は再起動後にリセットされるので，以下のコマンドで設定を維持するために保存する．
```bash
sudo netfilter-persistent save
```

※ iptables-persistent がインストールされていなければ
```bash
sudo apt install -y iptables-persistent
```
でインストールしておく．

### Open WebUI のユーザーの追加
管理者以外でも Open WebUI を利用できるようにするには，ユーザーを追加する必要がある．

Open WebUI に管理者でログインし，「設定」->「管理者設定」->「ユーザ」を選択し，+ボタンでユーザーを追加する（名前・メールアドレス・パスワードを適当に設定）．

### すべてのユーザがセットアップした LLM モデルを使えるように設定

Open WebuUI に管理者でログインし，「設定」->「管理者設定」->「モデル」を選択し，各モデルの編集ボタンをクリックして，Visibility が「Private」になっていれば「Public」に変更する．

## Ollama のアップデート

新たな LLM をダウンロード（Ollama で pull）しようとすると

The model you are attempting to pull requires a newer version of Ollama.

というエラーが表示される場合，Ollama をアップデートする必要がある．Docker コンテナの Ollama を最新版にアップデートするには
```bash
docker exec -it open-webui bash
```
で Open WebUI のコンテナに入って
```bash
curl -fsSL https://ollama.com/install.sh | sh
```
を実行すればよい．

## Open WebUI のアップデート

### 手動アップデート

1. 古いコンテナの停止と削除
    ```bash
    docker rm -f open-webui
    ```
    `-f` (`--force`) は実行中のコンテナを強制的に削除するオプション

1. 最新の Docker イメージの pull
    ```bash
    docker pull ghcr.io/open-webui/open-webui:ollama
    ```

1. 更新したイメージでコンテナを再起動
    GPU を用いて起動するコマンド
    ```bash
    docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
    ```

上記のアップデートを行っても，Ollama の設定やこれまでダウンロードした LLM はホストの格納場所(/var/lib/docker/volumes/ollama や /var/lib/docker/volumes/open-webui)にそのまま残っているので，これまで通り使用できる．

<span style="color:red;">※ Docker イメージでは，Ollama のバージョンが最新ではないことがあるので，このアップデートにより Ollama のバージョンが下がって新しめの LLM が使えなくなる場合がある．その場合，改めて上記の Ollama のアップデートを行えばよい．</span>
