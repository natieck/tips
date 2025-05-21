---
title: "オープンソース Ab-initio 計算プログラム「SALMON」のインストール"
date: 2025-05-19T11:30:00+09:00
categories:
  - Physics
tags:
  - SALMON
  - Ab-initio
  - TDDFT
toc: true
---

Ubuntu Server 及び Rocky Linux にオープンソース Ab-initio 計算プログラム「SALMON」をインストールしたときの備忘録

## はじめに

[SALMON](https://salmon-tddft.jp/) は，多様な光と物質の相互作用で起こるナノスケールの電子ダイナミクスに対して非経験的量子力学計算を行うオープンソース計算プログラムであり，時間依存密度汎関数理論に基づいた計算手法により，ノルム保存擬ポテンシャルを含む時間依存コーンシャム方程式を実時間・実空間で解く．[（SALMONの公式サイトより）](https://salmon-tddft.jp/jp/salmon.html)

Intel コンパイラ (oneAPI) を導入した Ununtu Server 24.04 LTS 及び Rocky Linux 9 に SALMON をインストールし，準備されているサンプル（アセチレン分子やシリコン結晶などの系）を実行する．また，SALMON のインストールにおいて，密度汎関数理論で用いる交換相関汎関数を計算するためのライブラリ [Libxc](https://libxc.gitlab.io/) を導入し，並列計算には Intel MPI ライブラリを用いる．事前に Intel oneAPI がインストールされていることを前提とする．

- SALMON のバージョン：2.2.1
- Libxc のバージョン：6.2.2

インストール環境：  
　OS：Ubuntu Server 24.04 LTS または Rocky Linux 9.5  
　コンパイラ：Intel oneAPI 2025.1 (Fortran/C++ コンパイラ，MPI ライブラリ, MKL ライブラリ)

OS が Ubuntu でも Rocky Linux でもインストール方法はほとんど同じである．

## CMake のインストール

SALMON のインストールでは CMake を用いるので，インストールされていなければ，CMake tools をインストールする（CMake 3.14 以降のバージョンが必要）．

CMake がインストールされていれば，以下のコマンドでバージョンを確認できる．
```bash
$ cmake --version
```

CMake のインストール（Ubuntu の場合）
```bash
$ sudo apt install cmake
```

CMake のインストール（Rocky Linux の場合）
```bash
$ sudo dnf install cmake
```

## Libxc のインストール

SALMON をインストールする前に，Libxc をインストールしておく．SALMON v.2.2.1 のパッケージの依存関係から最新版（2025年5月19日時点では，libxc-7.0.0.tar.bz2 (Oct 9th, 2024)）ではなく Libxc 6.2.2 (Jun 14th, 2023) を導入する必要がある．Libxc の公式サイトの [Download ページ](https://libxc.gitlab.io/download/previous/)から libxc-6.2.2.tar.bz2 をダウンロードして展開し，libxc-6.2.2 ディレクトリに移動する．
```bash
$ wget https://gitlab.com/libxc/libxc/-/archive/6.2.2/libxc-6.2.2.tar.bz2
$ tar jxvf libxc-6.2.2.tar.bz2
$ cd libxc-6.2.2
```

configure ファイルを生成するために，autoreconf を実行する．

※ Ubuntu には標準で autoreconf がインストールされていないので，以下のコマンドで事前に Autotools 群をインストールしておく．
```bash
$ sudo apt install autoconf automake libtool
```

configure ファイルを生成し，Intel コンパイラを指定して configure を実行する．
```bash
$ autoreconf -i
$ ./configure FC=ifx CC=icx FCFLAGS="-xHost" CFLAGS="-xHost"
```
configure オプション  
　`FC=ifx`：Fortran コンパイラとして，ifx を指定  
　`CC=icx`：C コンパイラとして，icx を指定  
　`FCFLAGS="-xHost"`：Fortran コンパイラ用のフラグに `-xHost` を設定する  
　`CFLAGS="-xHost"`：C コンパイラ用のフラグに `-xHost` を設定する  
　　※ `-xHost` は，Intel コンパイラにおいて，ホストプロセッサーで利用可能な最上位の命令セット向けのコードを生成するオプション  

デフォルトでは /opt/etsf にインストールされるので，変更する場合は，configure オプション `--prefix=インストールしたいディレクトリ` で指定する．

configure 実行後に make して，インストールする．
```bash
$ make
$ make check
$ sudo make install
```

## SALMON のインストール

SALMON の公式サイトの[ダウンロードページ](https://salmon-tddft.jp/jp/download.html)から最新リリース（2025年5月19日時点では，SALMON-v.2.2.1.tar.gz）をダウンロードして展開し，SALMON-v.2.2.1 ディレクトリに移動する．
```bash
$ wget http://salmon-tddft.jp/download/SALMON-v.2.2.1.tar.gz
$ tar zxvf SALMON-v.2.2.1.tar.gz
$ cd SALMON-v.2.2.1
```
インストール用に一時的なディレクトリ build を作成し，そのディレクトリに移動する．
```bash
$ mkdir build
$ cd build
```

Python スクリプト configure.py を実行する．
```bash
$ python ../configure.py --prefix=/opt --enable-libxc --with-libxc=/opt/etsf --enable-mpi FC=mpiifx CC=mpiicx FFLAGS="-xHost" CFLAGS="-xHost"
```
configure オプション
　`--prefix=/opt`：インストール先を `/opt` に設定する  
　`--enable-libxc`：Libxc ライブラリを有効にする  
　`--with-libxc=/opt/etsf`：Libxc の場所（`/opt/etsf`）  
　`--enable-mpi`：MPI を有効にする  
　`FC=mpiifx`：Fortran コンパイラを mpiifx（Intel の MPI 用 Fortran コンパイラ）に設定する  
　`CC=mpiicx`：C コンパイラを mpiicx（Intel の MPI 用 C コンパイラ）に設定する  
　`FFLAGS="-xHost"`：Fortran コンパイラ用のフラグに `-xHost` を設定する  
　`CFLAGS="-xHost"`：C コンパイラ用のフラグに `-xHost` を設定する  
　　※ `-xHost` は，Intel コンパイラにおいて，ホストプロセッサーで利用可能な最上位の命令セット向けのコードを生成するオプション  

make して，インストールする（salmon コマンドが /opt/bin にインストールされる）．
```bash
$ make
$ sudo make install
```

salmon コマンドへのパスを通しておく．
```bash
$ export PATH=$PATH:/opt/bin
```

## SALMON の実行方法

SALMON を実行するには，拡張子が `inp` の入力ファイルと擬ポテンシャルファイルが必要となる．入力ファイルは，Fortran90 の namelist フォーマットに従い，擬ポテンシャルファイル名は入力ファイルに記載しておく．

SALMON では，以下のフォーマットのノルム保存型の擬ポテンシャルが利用できる：
- [Fritz-Haber-Institute (FHI) 擬ポテンシャル](https://www.abinit.org/sites/default/files/PrevAtomicData/psp-links/lda_fhi.html)（拡張子：fhi）
- [Open MX コード用の擬ポテンシャル](https://t-ozaki.issp.u-tokyo.ac.jp/vps_pao2019/)（拡張子：vps）
- [ABINIT 用の擬ポテンシャル Format 8](https://www.abinit.org/psps_abinit)（拡張子：psp8）
- [Unified-pseudopotential-format（ノルム保存型のみ）](http://www.quantum-espresso.org/pseudopotentials/unified-pseudopotential-format)（拡張子：upf）

入力ファイルとして inputfile.inp を用いて，計算結果を fileout.out というファイルに出力するとする．単一プロセッサ環境で実行する場合，以下のように実行する．
```bash
$ salmon < inputfile.inp > fileout.out
```

マルチプロセッサ環境で並列計算をMPIを用いて実行する場合，以下のように実行する．
```bash
$ mpiexec -n NPROC salmon < inputfile.inp > fileout.out
```
NPROC は MPI プロセスの数である．

## SALMON の Exercise（サンプル）の実行

SALMON をダウンロードして展開したときにできたディレクトリ (SALMON-v.2.2.1) 内の samples というディレクトリ内に様々なサンプルがある．

なぜか孤立分子系のアセチレン(C2H2)のサンプル（exercise_01-03,08,09）を実行すると segmentation fault エラーが出て止まってしまい，バルク結晶系（周期系）のシリコン(Si)のサンプル（exercise_04-07,x1,x2）と金(Au)ナノ粒子における光伝播のサンプル（exercise_10,11）のみエラーなしに計算が終了した．

### Exercise 4（シリコン結晶の基底状態）の実行
適当な場所（以下では，ホームディレクトリ $HOME とする）に適当な名前のディレクトリ（以下では，exercise_04）を作成し，Exercise 4 の実行に必要なファイルを&lt;SALMONのディレクトリ&gt;からコピーする．
```bash
$ mkdir $HOME/exercise_04
$ cd $HOME/exercise_04
$ cp <SALMONのディレクトリ>/samples/exercise_04_bulkSi_gs/* .
```

単一プロセッサ環境で実行する場合
```bash
$ salmon < Si_gs.inp > fileout.out
```

マルチプロセッサ環境でMPIを用いて実行する場合（以下では12プロセスで計算）
```bash
$ mpiexec -n 12 salmon < Si_gs.inp > fileout.out
```

計算が終了すると，fileout.out や他の出力ファイルに結果が出力される．Exercise の計算の詳細は公式サイトの[ドキュメント](https://salmon-tddft.jp/jp/documents.html)に Youtube のチュートリアルで説明されている．