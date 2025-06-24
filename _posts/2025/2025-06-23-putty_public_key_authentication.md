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

接続先は Rocky Linux や Ubuntu などが導入された Linux サーバとし，SSH接続が許可されているものとする．このサーバにユーザ名 `user` のアカウントが存在し，このユーザ `user` でSSH接続する<span style="color:red;">（ただし，`user` のホームディレクトリに，グループや他のユーザの書き込み権限が与えられていると公開鍵認証での接続ができないので注意）</span>．

## PuTTY-ranvis のインストール

接続元のWindows PCにおいて，[PuTTY-ranvisのサイト](https://www.ranvis.com/putty)から PuTTY-ranvis の最新版(0.83 (2025-02-09))をダウンロードし，任意のフォルダに展開する．32bit か 64bit，7z圧縮版かインストーラ版があるので自身の環境にあったものをダウンロードすればよい（インストーラ版をダウンロードした場合は，インストーラを起動し，指示に従ってインストールする）．

展開した PuTTY-ranvis フォルダ内には以下のファイルが存在する．

![PuTTY-ranvis]({{site.baseurl}}/images/PuTTY-ranvis_files.png)

## 公開鍵認証方式でのSSH接続の設定

### PuTTYgen による公開/秘密鍵ペアの生成

PuTTY-ranvis フォルダ内の puttygen.exe をクリックして PuTTYgen を起動すると以下のウィンドウが表示される．パラメータ欄で生成する鍵の種類と強度（ビット数）を設定できるが，デフォルト（鍵の種類：RSA，ビット数：2048）のままで構わない（RSAとDSAではほとんどの目的で2048ビットで十分とされている）．

![PuTTYgen1]({{site.baseurl}}/images/PuTTYgen1.png)

アクション欄の一番上の「生成」ボタンを押すと，以下の表示になるので，「乱数を生成するために空白のエリア上でマウスを動かしてください。」の下のエリアで薄赤線で示したように適当にマウスを動かし続ける．緑のバーが右端に達したら鍵の生成が終了する．

![PuTTYgen2]({{site.baseurl}}/images/PuTTYgen2.png)

鍵の生成が終了したら，以下の表示になるので，必要に応じてパスフレーズの入力を行う．パスフレーズを空欄に設定するとパスフレーズ無しで接続できる（自動化したバッチスクリプトでSSH接続を確立するなどの特別な場合以外ではパスフレーズを設定するべき）．

![PuTTYgen3]({{site.baseurl}}/images/PuTTYgen3.png)

パスフレーズを入力後，公開鍵と秘密鍵を保存する．ユーザフォルダ直下に `.ssh` フォルダを作成し，公開鍵を `id_rsa.pub` ，秘密鍵を `id_rsa.ppk` というファイル名で保存する．保存したら，PuTTYgenを終了してよい．

### 公開鍵のサーバへの転送

上で生成した公開鍵をSSH接続先のサーバへ転送する．コマンドプロンプトでPuTTY-ranvis フォルダ（`pscp.exe` の存在するフォルダ）に移動して以下を実行し，サーバ上の `user` のホームディレクトリ直下に `id_rsa.pub` を転送する．

```powershell
C:/PuTTY-ranvis> .\pscp C:\Users\user\.ssh\id_rsa.pub user@サーバのIPアドレス:
```

サーバの `user` のパスワードを求められるので，入力して問題なく転送されると以下のメッセージが表示される．

```powershell
id_rsa.pub        | 0 kB |   0.5 kB/s | ETA: 00:00:00 | 100%
```

次に，サーバに `user` でログインする．ホームディレクトリに `.ssh` フォルダがなければ作成してアクセス権限を700に設定する．

```bash
$ mkdir .ssh
$ chmod 700 .ssh
```

ホームディレクトリに公開鍵 `id_rsa.pub` が転送されていることを確認し，以下のコマンドで公開鍵を `.ssh/authorized_keys` に追加し，ホームディレクトリの `id_rsa.pub` を削除しておく．

```bash
$ ssh-keygen -i -f id_rsa.pub >> .ssh/authorized_keys
$ rm id_rsa.pub
```

`authorized_keys` のアクセス権限を600に設定する（既に設定されていれば何もしなくてよい）．

```bash
$ chmod 600 .ssh/authorized_keys
```

`.ssh` ディレクトリや `authorized_keys` に，グループや他のユーザの書き込み権限を与えていると，公開鍵認証での接続ができなくなる．

### SSH の設定

必要に応じて `/etc/ssh/sshd_config` ファイルを`root`権限で編集する（公開鍵認証はデフォルトで有効になっている）．

```bash
# vim /etc/ssh/sshd_config
```

1. パスワード認証の禁止  
公開鍵認証でSSH接続できるようになったら，セキュリティを高めるためにパスワード認証によるSSH接続を禁止した方が良い．

```
#PasswordAuthentication yes
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓
```
PasswordAuthentication no
```

ただし，パスワード認証によるSSH接続を禁止すると，公開鍵認証でのみ接続可能となるので，公開鍵認証の設定をしないでパスワード認証を禁止するとリモート接続ができなくなる．

2. 公開鍵認証の許可（SSH2のみ）  
コメントアウトされていてもデフォルトで有効になっているので，変更する必要はないが念のため．

```
#PubkeyAuthentication yes
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓
```
PubkeyAuthentication yes
```

3. ポート番号の変更  

```
#Port 22
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↓
```
Port ○○
```
○○ は任意のポート番号．

## PuTTY による公開鍵認証方式でのSSH接続

PuTTY-ranvis フォルダ内の putty.exe をクリックして PuTTY を起動すると，以下のPuTTY設定ウィンドウが開き，セッション設定画面が表示されるので，「ホスト名（または IP アドレス）」欄に接続先サーバのホスト名か IP アドレスを入力し，ポート番号を変更した場合はその番号に設定する．

![PuTTY_setting1]({{site.baseurl}}/images/PuTTY_setting1.png)

次に，左のカテゴリ内の「接続」⇒「SSH」⇒「認証」⇒「クレデンシャル」を選択すると，以下の表示になるので，「認証のための秘密鍵ファイル」の参照ボタンをクリックして，秘密鍵の生成で保存した秘密鍵ファイル `id_rsa.ppk` を指定する．

![PuTTY_setting2]({{site.baseurl}}/images/PuTTY_setting2.png)

これらの設定は，セッション設定画面で適当に名前を付けて保存しておくとよい．

PuTTY設定ウィンドウの下にある「開く」ボタンを押すと接続先サーバに公開鍵認証方式でSSH接続が行われる．