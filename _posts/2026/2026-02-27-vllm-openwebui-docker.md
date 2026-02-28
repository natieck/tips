---
title: "Docker with NVIDIA GPU で vLLM + Open WebUI を導入する"
date: 2026-02-27T10:30:00+09:00
last_modified_at: 2026-2-28T010:30:30+09:00
categories:
  - LLM
tags:
  - vLLM
  - Open WebUI
  - Docker
  - CUDA
toc: true
---

Ubuntu Server 24.04 LTS をインストールした PC に Docker with NVIDIA GPU で vLLM + Open WebUI を導入したときの備忘録

## はじめに

[vLLM](https://docs.vllm.ai/en/latest/)（オープンソースLLMをローカル環境で実行できるプラットフォーム（LLM ランナー））と [Open WebUI](https://openwebui.com/)（LLMランナーを用いて ChatGPT のように手軽に LLM を利用できる Web インターフェース）を NVIDIA GPU を有効にした Docker コンテナで導入する．

vLLM は，PagedAttention（モデル内部のメモリ（KVキャッシュ）を効率よく管理）で GPU メモリを効率化して LLM を「高速に・大量同時実行で」動かすためのエンジンである．多くのベンチマークで，Ollama などの他のエンジンと比較して，圧倒的に高いパフォーマンスを示す．

**vLLM のポイント**：
- 高速：他のエンジンより推論速度が速い
- 大量同次処理に強い：API提供に向く
- 多くのモデルに対応：HuggingFace系モデルを幅広く動かせる
- サーバ運用に最適：REST API や OpenAI API互換で使える

ここでは，vLLM と Open WebUI を別個のコンテナで起動し，コンテナ間の通信を有効にして Open WebUI コンテナから vLLM コンテナに API エンドポイントを介してアクセスする．

PCのスペック：
- CPU: AMD Ryzen 5950X 16Core/32Threads
- Memory： DDR4 32GB
- SSD: 1TB
- GPU: NVIDIA RTX 3080Ti 12GB

以下を導入済みとする．
- NVIDIA CUDA Toolkit (バージョン12.8)
- NVIDIA Container Toolkit
- Docker

参照：[Ubuntu Server 24.04 LTS に CUDA 12.8 をインストールする](https://natieck.github.io/tips/linux/cuda_on_ubuntu_server24/)

## Docker の共通ネットワークの作成
vLLM コンテナと Open WebUI コンテナを連携させるために Docker の共通ネットワークを作成する（ネットワーク名を vllm-net とする）．
```bash
docker network create vllm-net
```
作成したネットワークは `docker network ls` で確認できる．

## vLLM コンテナの起動
vLLM サイトのユーザーガイドの [Using Docker ページ](https://docs.vllm.ai/en/latest/deployment/docker/)において Open WebUI Bundled with Ollama(With GPU Support) の箇所に記載されているコマンドを参考に，上で作成したネットワーク(vllm-net)を指定してコンテナを起動する．

```bash
docker run -d --name vllm --network vllm-net --gpus all -v ~/.cache/huggingface:/root/.cache/huggingface -p 8000:8000 --ipc=host vllm/vllm-openai:v0.10.2 --model Qwen/Qwen3-1.7B
```
<br>

オプションの説明：
- `-d`　コンテナを「デタッチ」モード（バックグラウンド）で実行
- `--name vllm`　コンテナ名を vllm に設定
- `--network vllm-net` ネットワークに vllm-net を指定
- `--gpus=all`　利用可能なすべての GPU を使用
- `-v ~/.cache/huggingface:/root/.cache/huggingface`　ホストの ~/.cache/huggingface フォルダをコンテナの /root/.cache/huggingface にマウント
- `-p 8000:8000`　コンテナの8000番のポートをホストの8000番にマッピング
- `--ipc=host`　コンテナの IPC（Inter-Process Communication：プロセス間通信）名前空間をホストと共有（ホストの共有メモリ `/dev/shm` を利用するため）
- `--model Qwen/Qwen3-1.7B`　vLLM で起動する LLM として Qwen3-1.7B を使用（モデルはGPUの容量に合わせて選択すればよい）
　※ vLLM は起動時にモデルを指定するため，途中でモデルを切り替えることはできない．

<br>

ここで，vllm の Docker イメージ(vllm/vllm-openai)のバージョンとして，v0.10.2 を指定している．これは導入している NVIDIA CUDA Toolkit のバージョン 12.8 に合わすためである（現時点で vllm/vllm-openai:latest の CUDA バージョンは 12.9.1）．

vLLM の Using Docker ページの `docker run` コマンドでは，オプションとして `--runtime nvidia` や `--env "HF_TOKEN=$HF_TOKEN"` を指定しているが，Docker 19.03 以降は，`--gpus all` を指定すれば `--runtime nvidia` は不要である．また，指定する LLM が公開モデルであれば，HF_TOKEN は不要である（Qwen/Qwen3-1.7Bは公開モデル）．公開モデルでなければ，HF_TOKEN には HuggingFace で発行した「Read」アクセストークン文字列を設定しておく．

### Open WebUI コンテナの起動
vLLM サイトのユーザー外の [Open WebUI ページ](https://docs.vllm.ai/en/latest/deployment/frameworks/open-webui/)に記載のコマンドを参考に，上で作成したネットワーク(vllm-net)を指定してコンテナを起動する（必ず vLLM コンテナの起動後に起動する）．

```bash
docker run -d --name open-webui-vllm --network vllm-net -p 3001:8080 -v open-webui-vllm:/app/backend/data -e OPENAI_API_BASE_URL=http://vllm:8000/v1 ghcr.io/open-webui/open-webui:main
```
<br>

オプションの説明：
- `-d`　コンテナを「デタッチ」モード（バックグラウンド）で実行
- `--name open-webui-vllm`　コンテナ名を open-webui-vllm に設定
- `--network vllm-net` ネットワークに vllm-net を指定
- `-p 3001:8080`　コンテナの8080番のポートをホストの3001番にマッピング
- `-v open-webui-vllm:/app/backend/data`　ホストの open-webui-vllm フォルダ（/var/lib/docker/volumes/open-webui-vllm）をコンテナの /app/backend/data にマウント
- `-e OPENAI_API_BASE_URL=http://vllm:8000/v1`　vllm のエンドポイントを OpenAI のエンドポイントとして登録
　※ コンテナが同じ Docker ネットワーク内なら http://vllm のようにコンテナ名で指定できる．

## Open WebUI にアクセス
上記のように vLLM コンテナと Open WebUI コンテナが起動していれば，導入した PC でブラウザを起動して `http://localhost:3001` にアクセスし，Open WebUI を利用できる．

初回アクセス時には，管理者アカウント登録（あくまでもローカルでの登録処理）が必要となるので，名前・メールアドレス・パスワードを設定する（メールアドレスは実在してない適当なものでも構わない）．

### LAN内の他の端末から Open WebUI にアクセスできるようにする
ファイヤーウォールが有効（アクティブ）であることを確認
```bash
sudo ufw status
```
有効でなければ有効化
```bash
sudo ufw enable
```
ファイヤーウォールの設定で 3001番のポートの通信を許可
```bash
sudo ufw allow 3001
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
sudo iptables -A INPUT -p tcp --dport 3001 -j ACCEPT
```
設定の確認
```bash
sudo iptables -L INPUT -n --line-numbers
```
以下のように表示されれば OK
```bash
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:3001
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