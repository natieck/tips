---
title: "Ubuntu Server 24.04 LTS に CUDA 12.8 をインストールする"
date: 2025-04-18T09:30:00+09:00
categories:
  - Linux
tags:
  - Ubuntu
  - CUDA
toc: true
---

Ubuntu Server 24.04 LTS に CUDA Toolkit 12.8 をインストールしたときの備忘録

## はじめに

[Ubuntu Server 24.04 LTS をインストール](https://natieck.github.io/tips/linux/ubuntu_server24-install/) した実機PCに NVIDIA Driver と CUDA Toolkit 12.8 をインストールする．

PCのスペック：
- CPU: AMD Ryzen 5950X 16Core/32Threads
- Memory： DDR4 32GB
- SSD: 1TB
- GPU: NVIDIA RTX 3080Ti 12GB

# 手順

## 準備
バージョンの古い NVIDIA Driver と CUDA Toolkit が既にインストールされている場合は，あらかじめこれらをアンインストールしておく必要がある．

### 既存の NVIDIA Driver 及び CUDA Toolkit 等のアンインストール
