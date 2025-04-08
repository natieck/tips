---
title: "GitHub Actions から GitHub Pages にデプロイするときに Pagefind を実行する方法"
date: 2025-04-08T11:30:00+09:00
categories:
  - Web
tags:
  - Pagefind
  - GitHub Actions
  - Jekyll
toc: false
---

GitHub Actions から GitHub Pages にデプロイするときに Pagefind を実行する方法についてのメモ

## はじめに

静的サイト向けの全文検索ライブラリである [Pagefind](https://pagefind.app/) を GitHub Pages で公開しているサイトに導入する際，ローカル環境にインストールした Pagefind を実行して検索用ファイルを生成して push するのではなく，GitHub Actions からデプロイするときに Pagefind を実行して デプロイするサーバ上のみに検索用ファイルを生成する．

## GitHub Actions から GitHub Pages にデプロイ

リポジトリの Settings ⇒ Pages に移動し，下図のように，Build and deployment の Source で GitHub Actions を選択する．

![GitHub Actions]({{site.baseurl}}/images/build_and_deployment_github_actions.png)

