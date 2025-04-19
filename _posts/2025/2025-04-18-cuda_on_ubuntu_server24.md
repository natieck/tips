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

Ubuntu Server 24.04 LTS に CUDA Toolkit 12.8 および cuDNN 9.8 をインストールしたときの備忘録

## はじめに

[Ubuntu Server 24.04 LTS をインストール](https://natieck.github.io/tips/linux/ubuntu_server24-install/) した実機PCに NVIDIA Driver と CUDA Toolkit 12.8 をインストールする．

PCのスペック：
- CPU: AMD Ryzen 5950X 16Core/32Threads
- Memory： DDR4 32GB
- SSD: 1TB
- GPU: NVIDIA RTX 3080Ti 12GB

## 準備
PC には Ubuntu Server 24.04 LTS が既にインストールされているものとする．

バージョンの古い NVIDIA Driver と CUDA Toolkit が既にインストールされている場合は，あらかじめこれらをアンインストールしておく必要がある．

### 既存の NVIDIA Driver 及び CUDA Toolkit 等のアンインストール
```bash
sudo apt --purge remove -y nvidia-*
sudo apt --purge remove -y cuda-*
sudo apt --purge remove -y libcudnn*
sudo apt --purge remove -y cudnn-*
sudo apt autoremove -y
```
最後のコマンド `apt autoremove` はパッケージの削除に伴い不要と判断されたパッケージを削除する．

## NVIDIA Driver のインストール
nvidia-driver-*のどれをインストールすればよいかは ubuntu-drivers を実行することで分かる．

ubuntu-drivers を使うために ubuntu-drivers-common をインストールする．
```bash
sudo apt install -y ubuntu-drivers-common
```
<br>
ubuntu-drivers の確認
```bash
sudo ubuntu-drivers --help
```
<br>

インストール可能なGPUドライバー一覧を出力する．
```bash
$ sudo ubuntu-drivers --gpgpu list
nvidia-driver-550, (kernel modules provided by linux-modules-nvidia-550-generic)
nvidia-driver-535, (kernel modules provided by linux-modules-nvidia-535-generic)
nvidia-driver-470, (kernel modules provided by linux-modules-nvidia-470-generic)
nvidia-driver-535-server, (kernel modules provided by linux-modules-nvidia-535-server-generic)
nvidia-driver-470-server, (kernel modules provided by linux-modules-nvidia-470-server-generic)
nvidia-driver-570-server-open, (kernel modules provided by linux-modules-nvidia-570-server-open-generic)
nvidia-driver-550-open, (kernel modules provided by linux-modules-nvidia-550-open-generic)
nvidia-driver-535-open, (kernel modules provided by linux-modules-nvidia-535-open-generic)
nvidia-driver-570-server, (kernel modules provided by linux-modules-nvidia-570-server-generic)
nvidia-driver-535-server-open, (kernel modules provided by linux-modules-nvidia-535-server-open-generic)
```
上の出力は，執筆時点で実際に上記 PC で実行したときのものである．
<br>

表示されている中で最も新しく open のついていないドライバー（nvidia-driver-570-server）をインストールする．
```bash
sudo apt install -y nvidia-driver-570-server
```
<br>

再起動
```bash
sudo reboot
```

## CUDAのインストール
`nvidia-smi` を実行すると，GPUドライバーが対応している最大のCUDAバージョンが表示される．
```bash
$ nvidia-smi
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 570.86.15              Driver Version: 570.86.15      CUDA Version: 12.8     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3080 Ti     Off |   00000000:07:00.0 Off |                  N/A |
|  0%   32C    P8              9W /  350W |      39MiB /  12288MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
```
⇒ CUDA バージョンは 12.8

[CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive) にアクセスし，上で確認したバージョン（12.8）を選択する．  
さらに，以下のように選択  
Operating System: Linux  
Architecture: x86_64  
Distribution: Ubuntu  
Version: 24.04  
Installer Type: deb(network)

CUDA Toolkit 12.8 の場合，インストール方法が以下のように提示されるので，実行する
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-8
```
<br>

CUDA ライブラリへの Path を通すため，`/etc/profile.d/cuda.sh` を作成し，以下を記述し保存する．
```plain
export PATH="/usr/local/cuda/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda/lib64:$LD_LIBRARY_PATH"
```

### cuda-samples を用いてセットアップができたかどうか確認
CUDA Toolkit の機能をデモするサンプル集をセットアップする．
```bash
git clone https://github.com/NVIDIA/cuda-samples.git
sudo apt install cmake （cmakeがインストールされていなければ）
cd cuda-samples
mkdir build && cd build
cmake ..
make -j$(nproc)
```
<br>

デモの実行例  
・CUDAデバイスのプロパティの参照
```bash
./Samples/1_Utilities/deviceQuery/deviceQuery
```
・GPU のコピー帯域幅と PCI-e 間のコピー帯域幅を測定
```bash
./Samples/1_Utilities/bandwidthTest/bandwidthTest
```
最後に「Result = PASS」と表示されれば正しく動作している．

## cuDNN (CUDA Deep Neural Network library) のインストール
[NVIDIA cuDNN](https://developer.nvidia.com/cudnn)にアクセスし，Download cuDNN Libraryに進む．  
以下のように選択  
Operating System: Linux  
Architecture: x86_64  
Distribution: Ubuntu  
Version: 24.04  
Installer Type: deb(network)

cuDNN 9.8.0 の場合，インストール方法が以下のように提示されるので，実行する（上の3行はCUDAインストール時に実行済みなので省略可）．
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cudnn
```

## NVIDIA Container Toolkit のインストール

Docker で NVIDIA GPU を利用する場合は NVIDIA Container Toolkit をインストールする必要がある．

まだ，Docker がインストールされていなければ，以下のコマンドでインストールする（https://get.docker.com で配布されているDockerインストール用shellスクリプトを利用）．
```bash
curl https://get.docker.com | sudo sh
```
<br>

[NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) の手順に従ってインストールする．
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update
sudo apt-get install -y nvidia-container-toolkit
```
<br>

### Docker で NVIDIA Container Runtime を認識させる  
daemon.json の作成
```bash
sudo vi /etc/docker/daemon.json
```
以下を記述
```json
{
   "runtimes" : {
      "nvidia" : {
         "path" : "/usr/bin/nvidia-container-runtime",
         "runtimeArgs" : []
      }
   }
}
```

### Docker デーモンの設定
```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```
<br>

サンプルCUDAコンテナを実行して，Docker 内で GPU が動作するか確認する．
```bash
sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```
