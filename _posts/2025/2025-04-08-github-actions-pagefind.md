---
title: "GitHub Actions から GitHub Pages にデプロイするときに Pagefind を実行する方法"
date: 2025-04-08T11:30:00+09:00
categories:
  - Web
tags:
  - Pagefind
  - GitHub Actions
  - GitHub Pages
toc: false
---

GitHub Actions から GitHub Pages にデプロイするときに Pagefind を実行する方法についてのメモ

## はじめに

静的サイト向けの全文検索ライブラリである [Pagefind](https://pagefind.app/) を GitHub Pages で公開しているサイトに導入する際，ローカル環境にインストールした Pagefind を実行して検索用ファイルを生成して push するのではなく，GitHub Actions からデプロイするときに Pagefind を実行して デプロイするサーバ上のみに検索用ファイルを生成する．ここでは，Jekyll でビルドせずに，単にローカル環境で作成した静的サイトをデプロイする．

## GitHub Actions から GitHub Pages にデプロイ

リポジトリの Settings ⇒ Pages に移動し，下図のように Build and deployment の Source で GitHub Actions を選択する．

![GitHub Actions]({{site.baseurl}}/images/build_and_deployment_github_actions.png)

GitHub Actions を選択すると，静的サイトをデプロイするGitHub Actions ワークフロー「Static HTML」が提示されるので，Configure ボタンで編集する． 下に示すようにワークフローの「# ここから ～ # ここまで」の間に，Node.js をセットアップして，Pagefine コマンドを実行するフローを追加して保存する．

```yaml
jobs:
  # Single deploy job since we're just deploying
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Setup Pages
        uses: actions/configure-pages@v5
      # ここから
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
      - name: Run pagefind command
        run: npx pagefind --site public
      # ここまで
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          # Upload entire repository
          path: '.'
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

ここで，`run: npx pagefind --site public` の「public」には検索対象となるHTMLファイルがあるディレクトリを指定する．検索用ファイルはそのディレクトリの直下に生成される．