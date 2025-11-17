---
title: 1. docker study notes
author: DNV825
date: 2025-01-01
category: Jekyll
layout: post
created: 2025-03-31
---

今度は WSL2 で docker を使っていろいろ試そう。

## 1.1. WSL のインストール

WSL を有効化し、micorsoft ストアから Ubuntu 24.04.1 LTS をインストールした。

```shell
PS C:\Users\xxx> wsl --list
Linux 用 Windows サブシステム ディストリビューション:
Ubuntu (既定)
Ubuntu-24.04
```

この `Ubuntu (規定)` が一緒に入ってきたのは謎だが、デフォルトインストールされるやつなのだろうか？

## 1.2. メモリを確保する

`C:\Users\[ユーザー名]\.wslconfig` を作成し、とりえず以下のように記述した。
`memory` には搭載されているメモリのサイズを指定し、すべてのメモリをフルで使えるようにしている。
`swap` はスワップ領域のサイズだが、どれくらいが適切なのかわからなかったので、とりあえずメモリの 2 倍のサイズを指定した。

```ini
[wsl2]
memory=64GB
swap=128GB
```

このファイルを作成したのちに、いったん PowerShell で　`wsl --shutdown` を実行して wsl をシャットダウンし、 wsl を再度起動する。起動後にメモリ領域を確認すると、利用可能なメモリサイズが増えていることがわかる。

```shell
wsluser@pc:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:            60Gi       1.1Gi        59Gi       3.0Mi       332Mi        59Gi
Swap:          128Gi          0B       128Gi
```

## 1.3. nVIDIA Driver がインストールされているか確認する

eGPU を接続し、デバイスドライバのインストールに成功していると、`nvidia-smi` コマンドを利用できるようになる。試しに Khadas Mind Graphics GForce RTS 4060 Ti 16GB GDDR6 を接続した状態で PowerShell で入力してみると以下のように表示された。

```shell
PS C:\Users\xxx> nvidia-smi
Thu Jan 16 19:34:22 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 560.94                 Driver Version: 560.94         CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                  Driver-Model | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 Ti   WDDM  |   00000000:06:00.0 Off |                  N/A |
| 61%   33C    P8              9W /  165W |      16MiB /  16380MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
+-----------------------------------------------------------------------------------------+
```

WSL2 側でも同じ結果が返ってきた。

```shell
wsluser@pc:~$ nvidia-smi
Thu Jan 16 19:35:15 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 560.35.02              Driver Version: 560.94         CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 Ti     On  |   00000000:06:00.0 Off |                  N/A |
|  0%   33C    P8              8W /  165W |      16MiB /  16380MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

EGX-TBT-A500 のデバイスドライバである `NVIDIA RTS A500 Embedded GPU` もインストール済みなのだが、非接続の状態だと表示されないようだ（コマンドオプションがあるかもしれないけれど、未確認。）

## 1.4. Docker をインストールする

Docker Engine というもののみをインストールする。Docker Desktop という GUI アプリもあるそうだが、こちらは有料になったりすることがあるらしい。公式ドキュメントは以下のアドレスにて公開されている。

- <https://docs.docker.com/engine/install/ubuntu/>

以下のスクリプトを実行すればインストールすることができるのだが、これは本番環境への使用は推奨されていないそうだ。

```shell
wsluser@pc:~$ curl -fsSL https://get.docker.com -o get-docker.sh
wsluser@pc:~$ sudo sh get-docker.sh
[sudo] password for wsluser:
# Executing docker install script, commit: 4c94a56999e10efcf48c5b8e3f6afea464f9108e

WSL DETECTED: We recommend using Docker Desktop for Windows.
Please get Docker Desktop from https://www.docker.com/products/docker-desktop/


You may press Ctrl+C now to abort this script.
+ sleep 20
+ sh -c apt-get -qq update >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get -y -qq install ca-certificates curl >/dev/null
+ sh -c install -m 0755 -d /etc/apt/keyrings
+ sh -c curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" -o /etc/apt/keyrings/docker.asc
+ sh -c chmod a+r /etc/apt/keyrings/docker.asc
+ sh -c echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" > /etc/apt/sources.list.d/docker.list
+ sh -c apt-get -qq update >/dev/null
+ sh -c DEBIAN_FRONTEND=noninteractive apt-get -y -qq install docker-ce docker-ce-cli containerd.io docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin >/dev/null
+ sh -c docker version
Client: Docker Engine - Community
 Version:           27.5.0
 API version:       1.47
 Go version:        go1.22.10
 Git commit:        a187fa5
 Built:             Mon Jan 13 15:25:08 2025
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          27.5.0
  API version:      1.47 (minimum version 1.24)
  Go version:       go1.22.10
  Git commit:       38b84dc
  Built:            Mon Jan 13 15:25:08 2025
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.25
  GitCommit:        bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
 runc:
  Version:          1.2.4
  GitCommit:        v1.2.4-0-g6c52b3f
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

================================================================================

To run Docker as a non-privileged user, consider setting up the
Docker daemon in rootless mode for your user:

    dockerd-rootless-setuptool.sh install

Visit https://docs.docker.com/go/rootless/ to learn about rootless mode.


To run the Docker daemon as a fully privileged service, but granting non-root
users access, refer to https://docs.docker.com/go/daemon-access/

WARNING: Access to the remote API on a privileged Docker daemon is equivalent
         to root access on the host. Refer to the 'Docker daemon attack surface'
         documentation for details: https://docs.docker.com/go/attack-surface/

================================================================================
```

バージョン確認すると Docker のインストールに成功したことがわかる。

```shell
wsluser@pc:~$ docker -v
Docker version 27.5.0, build a187fa5
```

WSL2 は再起動が必要らしいので、またシャットダウンして次へ進む。

```shell
PS C:\Users\xxx> wsl --shutdown
```

再起動後、サービスのステータスを見ると動いていることがわかる。

```shell
wsluser@pc:~$ service docker status
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Thu 2025-01-16 20:18:50 JST; 44min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 1148 (dockerd)
      Tasks: 14
     Memory: 29.5M ()
     CGroup: /system.slice/docker.service
             └─1148 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

Jan 16 20:18:49 pc dockerd[1148]: time="2025-01-16T20:18:49.860109506+09:00" level=info msg="Default brid>
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.264234246+09:00" level=info msg="Loading cont>
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.276255565+09:00" level=warning msg="WARNING: >
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.276282047+09:00" level=warning msg="WARNING: >
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.276286685+09:00" level=warning msg="WARNING: >
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.276290840+09:00" level=warning msg="WARNING: >
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.276303987+09:00" level=info msg="Docker daemo>
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.276331946+09:00" level=info msg="Daemon has c>
Jan 16 20:18:50 pc dockerd[1148]: time="2025-01-16T20:18:50.313634025+09:00" level=info msg="API listen o>
Jan 16 20:18:50 pc systemd[1]: Started docker.service - Docker Application Container Engine.
```

## 1.5. NVIDIA Docker をインストールする

Docker コンテナから NVIDIA GPU を利用するための nvidia-container-toolkit が必要らしいのでインストールする。インストール方法は <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html> に書かれている通り、以下のコマンドを実行すればよい。

```shell
wsluser@pc:~$ curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
[sudo] password for wsluser:
deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://nvidia.github.io/libnvidia-container/stable/deb/$(ARCH) /
#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://nvidia.github.io/libnvidia-container/experimental/deb/$(ARCH) /

wsluser@pc:~$ sudo sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list

wsluser@pc:~$ sudo apt update
Get:1 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  InRelease [1477 B]
Get:2 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  InRelease [1489 B]
Get:3 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  Packages [15.8 kB]
Hit:4 https://download.docker.com/linux/ubuntu noble InRelease
Get:5 https://nvidia.github.io/libnvidia-container/experimental/deb/amd64  Packages [9632 B]
Hit:6 http://security.ubuntu.com/ubuntu noble-security InRelease
Hit:7 http://archive.ubuntu.com/ubuntu noble InRelease
Hit:8 http://archive.ubuntu.com/ubuntu noble-updates InRelease
Hit:9 http://archive.ubuntu.com/ubuntu noble-backports InRelease
Fetched 28.4 kB in 3s (8810 B/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.

wsluser@pc:~$ sudo apt install nvidia-container-toolkit
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libnvidia-container-tools libnvidia-container1 nvidia-container-toolkit-base
The following NEW packages will be installed:
  libnvidia-container-tools libnvidia-container1 nvidia-container-toolkit nvidia-container-toolkit-base
0 upgraded, 4 newly installed, 0 to remove and 0 not upgraded.
Need to get 5793 kB of archives.
After this operation, 27.7 MB of additional disk space will be used.
Do you want to continue? [Y/n] Y
Get:1 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  libnvidia-container1 1.17.3-1 [924 kB]
Get:2 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  libnvidia-container-tools 1.17.3-1 [20.1 kB]
Get:3 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  nvidia-container-toolkit-base 1.17.3-1 [3663 kB]
Get:4 https://nvidia.github.io/libnvidia-container/stable/deb/amd64  nvidia-container-toolkit 1.17.3-1 [1186 kB]
Fetched 5793 kB in 1s (4272 kB/s)
Selecting previously unselected package libnvidia-container1:amd64.
(Reading database ... 41350 files and directories currently installed.)
Preparing to unpack .../libnvidia-container1_1.17.3-1_amd64.deb ...
Unpacking libnvidia-container1:amd64 (1.17.3-1) ...
Selecting previously unselected package libnvidia-container-tools.
Preparing to unpack .../libnvidia-container-tools_1.17.3-1_amd64.deb ...
Unpacking libnvidia-container-tools (1.17.3-1) ...
Selecting previously unselected package nvidia-container-toolkit-base.
Preparing to unpack .../nvidia-container-toolkit-base_1.17.3-1_amd64.deb ...
Unpacking nvidia-container-toolkit-base (1.17.3-1) ...
Selecting previously unselected package nvidia-container-toolkit.
Preparing to unpack .../nvidia-container-toolkit_1.17.3-1_amd64.deb ...
Unpacking nvidia-container-toolkit (1.17.3-1) ...
Setting up nvidia-container-toolkit-base (1.17.3-1) ...
Setting up libnvidia-container1:amd64 (1.17.3-1) ...
Setting up libnvidia-container-tools (1.17.3-1) ...
Setting up nvidia-container-toolkit (1.17.3-1) ...
Processing triggers for libc-bin (2.39-0ubuntu8.3) ...

wsluser@pc:~$ sudo nvidia-ctk runtime configure --runtime=docker
INFO[0000] Config file does not exist; using empty config
INFO[0000] Wrote updated config to /etc/docker/daemon.json
INFO[0000] It is recommended that docker daemon be restarted.

wsluser@pc:~$ sudo systemctl restart docker
```

## 1.6. 利用可能な CUDA のバージョンを把握する

まず、[Your GPU Compute Capability](https://developer.nvidia.com/cuda-gpus) から、自分の使う GPU の Compute Capability を把握する。

先ほど実行した `nvidia-smi` コマンドで、GPU が対応する CUDA のバージョンが表示されていた。 Khadas Mind Graphics GForce RTS 4060 Ti 16GB GDDR6 が対応する CUDA は v12.6 らしい。

```shell
wsluser@pc:~$ nvidia-smi
Thu Jan 16 19:35:15 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 560.35.02              Driver Version: 560.94         CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 Ti     On  |   00000000:06:00.0 Off |                  N/A |
|  0%   33C    P8              8W /  165W |      16MiB /  16380MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|  No running processes found                                                             |
+-----------------------------------------------------------------------------------------+
```

Docker イメージを扱う際、この CUDA バージョンが重要になる。

EGX-TBT-A500 のデバイスドライバである `NVIDIA RTS A500 Embedded GPU` が有効になっている状態であれば、`nvidia-smi` を実行したところ、以下の表示であった。 CUDA Version が少し古いようだ。

```shell
PS C:\Users\xxx> nvidia-smi
Mon Jan 20 09:37:09 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 551.61                 Driver Version: 551.61         CUDA Version: 12.4     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                     TCC/WDDM  | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA RTX A500 Embedded...  WDDM  |   00000000:7C:00.0 Off |                  N/A |
| N/A   37C    P0             22W /   25W |     799MiB /   4096MiB |     75%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A     21156    C+G   ...team\steamapps\common\Elin\Elin.exe      N/A      |
+-----------------------------------------------------------------------------------------+
```

## 1.7. nVIDIA が提供する Docker イメージを使う

（いったんスキップ。多分 runtime 版を動かせばよい。）

nVIDIA が CUDA 環境を整備した Docker イメージを提供しているので、それを使ってみる。<https://catalog.ngc.nvidia.com/orgs/nvidia/containers/cuda> に配置されている。

## 1.8. docker を sudo なしで利用可能にする

docker グループが追加されるので、そこに自分を追加する。すると、 docker コマンドを sudo なしで使えるようになる。

```shell
wsluser@pc:~$ getent group docker
docker:x:989:
wsluser@pc:~$ sudo usermod -aG docker $USER
[sudo] password for wsluser:
wsluser@pc:~$ getent group docker
docker:x:989:wsluser
```

## 1.9. 参考資料

### 1.9.1. 公式資料

- Docker Engine 公式ドキュメント: <https://docs.docker.com/engine/>

### 1.9.2. ありがたき先人たちの教え

- karaage0703, WSL関係のTips, Zenn, 2024-07-13, <https://zenn.dev/karaage0703/articles/0a3c2b3daa389e>
- karaage0703, ゲーミングPCのWindows環境セットアップ, Zenn, 2024-07-23, <https://zenn.dev/karaage0703/articles/211d89cc0a29a1>
- karaage0703, WSL2/Ubuntu/Raspberry Piでのメモリ不足を解消する, Zenn, 2023-04-16, <https://zenn.dev/karaage0703/articles/d38e17bd6efbaa>
- ©nkmk.me, watch nvidia-smiでGPU使用率などを確認・リアルタイムモニタリング, note.nkmk.me, 2021-03-06, <https://note.nkmk.me/nvidia-smi-monitoring-gpu/>
- @kazokmr, Docker を使っている時に調べたことまとめ, Qiita, 2021-06-05, <https://qiita.com/kazokmr/items/1ffc77d01a67aff90c75>
- @kosuke_aizawa (宏亮 相澤), No.6 新卒未経験エンジニアがDockerを使ってチョメチョメしてみた〜チュートリアル編〜, Qiita, 2017-10-28, <https://qiita.com/kosuke_aizawa/items/4abab88caaae119545cf>
