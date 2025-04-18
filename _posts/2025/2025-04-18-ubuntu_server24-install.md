---
title: "Ubuntu Server 24.04 LTS のインストール"
date: 2025-04-18T07:00:00+09:00
categories:
  - Linux
tags:
  - Ubuntu
toc: true
---

PCに Ubuntu Server 24.04 LTS をインストールし，各種設定の手順を記した備忘録

## はじめに

実機PCに Ubuntu Server 24.04.2 LTS をUSBメモリからインストールし，日本語環境やデスクトップ環境 (Ubuntu Desktop) を導入する．

PCのスペック：
- CPU: AMD Ryzen 5950X 16Core/32Threads
- Memory： DDR4 32GB
- SSD: 1TB
- GPU: NVIDIA RTX 3080Ti 12GB

## インストールメディア（USBメモリ）の準備

- [Ubuntuの公式ダウンロードページ](https://jp.ubuntu.com/download)から Ubuntu Server 24.04.2 LTS の ISO ファイルをダウンロードする．
- ISOファイルをUSBメモリに書き込む．  
   Windows でインストールメディアを作成する場合は，[Rufus](https://rufus.ie/ja/)（起動可能なUSBドライブを作成するWindowsアプリ）などを使用すればよい（Mac や Linux の場合は，ddコマンドで書き込む）． 

## OSのインストール

作成したUSBインストールメディアをインストール先のPCへ差し込んで起動し，BIOS画面でUSBメディアが優先的にブートされるように変更し，インストーラーを起動する．  
インストーラーが起動したら，Install Ubuntu Server を選択して，以下の設定を順番に行っていく．

- 言語：English（日本語はない）
- キーボードレイアウト：Japanese
- インストールタイプ：Ubuntu Server
- ネットワーク設定
　設定しない場合やDHCPの場合，特に何もせずに進む．  
　固定IPに設定する場合  
　ネットワーク名（enp3s0など）を選択し，「Edit IPv4」でIPv4の設定に進む．  
　IPv4 Method で Manual を選び，以下を設定する．  
　　Subnet: ###.###.###.0/24 (←サブネットマスクが 255.255.255.0 の場合)  
　　Address: ###.###.###.###  
　　Gateway: ###.###.###.###  
　　Name servers: ###.###.###.###, ###.###.###.### （複数の場合はコンマで区切る）  
　　Search domains: （特になければ空欄のまま）  

- Proxy設定：（特になければ空欄のまま）
- ミラーサイト設定：デフォルトのまま（ネットワークにつながっていればデフォルトのミラーサイトが選択される）
- ディクス設定：Use an entire disk（ディスク全体を使って，自動でパーティションを設定）  
　パーティション設定する場合は Custom storage layout を選択
- ユーザ設定  
　　Your name: ユーザ表示名（設定不要）  
　　Your servers name: サーバーのホスト名  
　　Pick a username: ユーザ名  
　　Choose a password: ユーザのパスワード  
　　Confirm your password: パスワードの確認  
- Ubuntu Pro へのアップデート：Skip for now（スキップする）
- OpenSSH設定：Install OpenSSH server（OpenSSHをインストール）  
　Import SSH key （既存のSSH秘密鍵をインポートする必要がなければ特に何もしない）
- Featured server snaps （パッケージンのインストール）
　後から個別にインストールできるので何もしなくてよい．

インストール後，再起動すると Ubuntu Server が起動する（USBメモリを抜いておく）．

## 初期設定
### パッケージの更新
```bash
sudo apt update
sudo apt upgrade -y
```
### タイムゾーンの設定
```bash
sudo timedatectl set-timezone Asia/Tokyo
```

### sudoパスワード入力の省略設定（必要に応じて）
次のコマンドで，設定ファイル `/etc/sudoers` を編集する．
```bash
sudo visudo
```
<br>

`/etc/sudoers` の変更箇所
```diff
-%sudo ALL=(ALL:ALL) ALL
+%sudo ALL=(ALL:ALL) NOPASSWD:ALL
```

### rootユーザを有効にする
以下のコマンドでrootになれる．
```bash
$ sudo -s
[sudo] password for ubuntu:（sudoパスワード省略設定していれば表示されない）
```

または，rootパスワードを設定することで，su コマンドによる切り替えが可能
```bash
$ sudo passwd root
[sudo] password for ubuntu:（sudoパスワード省略設定していれば表示されない）
New password:（rootのパスワード）
Retype new password:（確認再入力）
passwd: password updated successfully

$ su -
Password:（rootのパスワード）
```
rootに切り替わる．

## 日本語環境

### 日本語パッケージのインストール
```bash
sudo apt -y install language-pack-ja-base language-pack-ja
```

### ロケールを日本語に設定
```bash
sudo localectl set-locale LANG=ja_JP.UTF-8 LANGUAGE="ja_JP:ja"
```

## デスクトップ環境

### Ubuntu Desktopのインストール
```bash
sudo apt -y ubuntu-desktop task-gnome-desktop
```

### systemd-networkd-wait-online.serviceの停止
Ubuntu Desktop環境をインストールすると，サーバ起動時に「Starting Wait for Network to be Configured...」というメッセージが出力され，systemd-networkd-wait-online.serviceで2分ほど待機させられるので，このサービスを停止する．  
（※ケーブルを刺していないLANポートでのネットワークへの接続確立まで待機しようとする．）  
GUI環境を構築するとNetworkManagerがデフォルトのネットワークマネージャーになるので，停止してもおそらく問題ないと思われる．
```bash
sudo systemctl disable --now systemd-networkd-wait-online.service
```

## ファイヤーウォール設定
状態の確認
```bash
$ sudo ufw status
状態: 非アクティブ
```
<br>

Ubuntuインストール直後はファイヤーウォールが非アクティブ（無効）になっているので，アクティブ（有効）化する．
```bash
sudo ufw enable
```
<br>

デフォルトではすべての通信を拒否するように設定する．
```bash
sudo ufw default DENY
```
<br>

SSH（ポート番号:22）を許可する（ルールの追加）．
```bash
sudo ufw allow 22
```
もしくは
```bash
sudo ufw allow ssh
```
ポート番号でなくサービス名で指定すると，`/etc/services` に書かれたサービス名に対応したポート番号が指定される．

sshに対して接続頻度を制限する（執拗に接続してくるアクセスの拒否）場合は以下のように設定する．
```bash
sudo ufw limit ssh
```
<br>

設定後，ファイヤーウォールの再読み込みを行う．
```bash
sudo ufw reload
```
<br>

ファイヤーウォールを無効化する場合は以下を実行する．
```bash
sudo ufw disable
```

## その他

### Python3の導入
Pythonでの開発環境を導入する（Ubuntu Server 24.04.2 LTS を導入すると，Python3.12.3 が既にインストールされている）．
仮想環境管理 venv，パッケージ管理 pip，開発ツール dev のインストール
```bash
sudo apt -y install python3-venv python3-pip python3-dev
```

### システムの再起動
```bash
sudo reboot
```
もしくは
```bash
sudo shutdown -r now
```

### システムのシャットダウン
```bash
sudo poweroff
```
もしくは
```bash
sudo shutdown -h now
```