---
title: "PuTTYを用いた公開鍵認証でのSSH接続"
date: 2025-05-19T11:30:00+09:00
categories:
  - Windows
tags:
  - SSH
  - Public Key Authentication
  - PuTTY
toc: true
---

[PuTTY-ranvis](https://www.ranvis.com/putty)を用いた公開鍵認証でのSSH接続の設定方法についての備忘録

## はじめに

Simon Tatham氏によるWindows用リモート接続クライアントである[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/)のカスタムビルド版[PuTTY-ranvis](https://www.ranvis.com/putty)（公式版に日本語UI，ISO 2022対応などのパッチを適用したもの）を用いて公開鍵認証方式でSSH接続を行う．

接続先は Rocky Linux もしくは Ubuntu が導入された Linux サーバとし，SSH接続が許可されているものとする．

## PuTTY-ranvis のインストール

[PuTTY-ranvisのサイト](https://www.ranvis.com/putty)から最新版(0.83 (2025-02-09))をダウンロードし，任意のフォルダに展開する．32bit か 64bit，7z圧縮版かインストーラ版があるので自身の環境にあったものをダウンロードすればよい（インストーラ版をダウンロードした場合は，インストーラを起動し，指示に従ってインストールする）．

展開した PuTTY-ranvis フォルダ内には以下のファイルが存在する．

![PuTTY-ranvis]({{site.baseurl}}/images/PuTTY-ranvis_files.png)

## 