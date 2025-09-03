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

## 認証（Authentication）

Open WebUI の API へアクセスするためには認証が必要となる．認証は Bearer Token 方式で行われ，Open WebUI の「設定」->「アカウント」で取得できる API キー，または JWT（JSON Web Token）を使用して認証する．

Bearer Token 方式：API認証においてクライアントが発行されたアクセストークンをHTTPリクエストの「Authorization」ヘッダーに付与してサーバーに送信する方式

![GET_API_KEY]({{site.baseurl}}/images/open_webui_get_api_key.png)

