---
title: "rsync と cron による ssh 経由の自動バックアップ"
date: 2025-06-27T12:30:00+09:00
categories:
  - Linux
tags:
  - rsync
  - cron
  - ssh
toc: true
---

rsync と cron による ssh 経由の自動バックアップ設定の備忘録

## はじめに

ホストAのデータを定期的にホストBにバックアップする．その際，ホストBにおいて root 権限で cron によって定期的に rsync を実行し，ホストBからホストAに公開鍵認証を用いて SSH 接続して，データ転送を行う．バックアップ方式はミラーリングとし，バックアップ時点でのホストAのデータの状態をホストBに反映する．変更がないファイルは転送されず，ホストAから削除されたデータはホストBからも削除される．

ホストA,Bの情報は以下のとおりとする．

- ホストAのドメイン(IPアドレス)：domain.hostA
- ホストAのデータの場所(ディレクトリ)：`/path/to/source/`
- ホストBのドメイン(IPアドレス)：domain.hostB
- ホストBのバックアップの場所(ディレクトリ)：`/path/to/destination/`

## root での公開鍵認証の設定

### ホストBでの設定

ホストBに root でログインし，以下のように ssh-keygen コマンドで秘密鍵・公開鍵を生成する（暗号化形式はRSAを指定）．パスフレーズを訊かれるが，パスフレーズ無しで SSH 接続するために，何も入力せずにEnterキーを押す．

```bash
# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):　←何も入力せずにEnterキーを押す（パスフレーズの設定はしない）
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
***********************************************
The key's randomart image is:
+---[RSA 3072]----+
|                 |
~                 ~
~                 ~
|                 |
+----[SHA256]-----+
```

ホストBで生成した公開鍵をホストAの authorized_keys（認証キーファイル）に追加する．

```bash
# ssh-copy-id -i ~/.ssh/id_rsa.pub root@domain.hostA
```

### ホストAでの設定

root は公開鍵認証方式でなければ SSH 接続させない設定にする．

```bash
# vi /etc/ssh/sshd_config
```

sshd_config ファイルを編集し，以下のように設定する．

```
PermitRootLogin without-password
```

設定後，ssh の設定をリロードする．

```bash
# systemctl reload sshd.service
```

指定したIPアドレスからのアクセス（rsync）のみ許可する．

```bash
# vi /root/.ssh/authorized_keys
```

ホストBから追加された公開鍵の先頭に，以下のようにホストBのドメインを入力する．

```
from="domain.hostB" ssh-rsa *******
```

設定後，ホストBから root でパスワード入力無しでホストAに SSH 接続できるようになっていることを確認する．

## rsync の設定

ホストBにおいて，root 権限で rsync をテストモード（ドライラン）で実行し，バックアップするデータを確かめる．以下のコマンドでは，SSH 経由で，ホストA（domain.hostA）の `/path/to/source/` 以下のファイル・ディレクトリを，ホストBの `/path/to/destination/` 以下に転送する．`--dry-run` はテストモードとして実行するオプション，`--delete` は転送元にないファイルを転送先からも削除するオプションである．

```bash
# rsync --dry-run -avh --delete -e ssh domain.hostA:/path/to/source/ /path/to/destination
```

rsync の主なオプションを以下に示す．

|オプション|別名|意味|
|---|---|---|
| `-a` | `--archive` | アーカイブモード（`-rlptgoD` と同じ） |
| `-r` | `--recursive` | ディレクトリ配下を再帰的にコピー |
| `-l` | `--links` | シンボリックリンクをシンボリックリンクのままコピー |
| `-p` | `--perms` | パーミッションを保持してコピー |
| `-t` | `--times` | タイムスタンプを保持してコピー |
| `-g` | `--group` | グループを保持してコピー |
| `-o` | `--owner` | 所有者を保持してコピー（rootのみ有効） |
| `-D` |  | `--devices --specials` と同じ |
| | `--devices` | デバイスファイルを保持してコピー（rootのみ有効） |
| | `--specials` | 特殊ファイルを保持してコピー |
| `-v` | `--verbose` | 冗長性を高める（転送情報を表示） |
| `-h` | `--human-readable` | 数字を読みやすい形式で出力 |
| `-n` | `--dry-run` | テストモード（実際に転送せず転送情報のみを表示） |
| | `--delete` | 転送元にないファイルを転送先から削除 |
| `-e` | `--rsh=コマンド` | リモートシェルを指定 |
| | `--exclude=パターン` | 指定したパターンにマッチしたファイルを転送しない |
| `-z` | `--compress` | 転送時にデータを圧縮 |

## cron の設定

上の rsync のテストモードでの実行で問題なければ，ホストBにおいて，root 権限で crontab を編集する．

```bash
# crontab -e
```

cron の設定は以下の書式に従って記述する．

```
分 時 日 月 曜日 <実行コマンド>
```

それぞれのフィールドの値の範囲は，分(0-59)，時(0-23)，日(1-31)，月(1-12)，曜日(0-7)，である（曜日の0と7は日曜日を表す）．分・時・日・月・曜日の部分にアスタリスク `*` を指定すると，そのフィールドのすべての可能な値が指定されていることを意味する．すべてのフィールドが `*` なら毎分コマンドが実行される．

以下のように記述すると，毎日1:00AMに rsync を実行する（`/var/log/backup.log` にログを取るように設定）．

```
0 1 * * * rsync -avh --delete -e ssh domain.hostA:/path/to/source/ /path/to/destination > /var/log/backup.log 2>&1
```
