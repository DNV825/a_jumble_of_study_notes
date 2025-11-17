---
title: 2. WSL2 Ollama
author: DNV825
date: 2025-01-02
category: Jekyll
layout: post
created: 2025-03-31
---

## 2.1. Ollama + Open WebUI を Docker で動かす

いきなり Docker で Ollama と Open WebUI を動かせるらしいので試してみる。

```shell
wsluser@pc:~$ sudo docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
Unable to find image 'ollama/ollama:latest' locally
latest: Pulling from ollama/ollama
6414378b6477: Pull complete
4757d34365d2: Pull complete
e1ff184b77bf: Pull complete
f6ca962e3d3f: Pull complete
Digest: sha256:0a39e5c0a3f7f8d1eb017dc677b3fffc1a7c3475a278eb7bb423582c41c95bcd
Status: Downloaded newer image for ollama/ollama:latest
44010c2504fc97d98077d01efeea62cc89cbed0979d0d47ffcc48d907c09ccec

wsluser@pc:~$ sudo docker run -p 3000:8080 --env WEBUI_AUTH=False --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
[sudo] password for wsluser:
Unable to find image 'ghcr.io/open-webui/open-webui:main' locally
main: Pulling from open-webui/open-webui
fd674058ff8f: Pull complete
f7d891cc9f68: Pull complete
96b561a6925a: Pull complete
a59d836915ff: Pull complete
0503d2c39f40: Pull complete
4f4fb700ef54: Pull complete
a960254c47b1: Pull complete
6c61893cbbc8: Pull complete
a75437d367ce: Pull complete
b526c0160cdf: Pull complete
141224bfd6fb: Pull complete
6aad607d26ac: Pull complete
6a59a26a1f8d: Pull complete
53d9fc4e9900: Pull complete
374b22022969: Pull complete
Digest: sha256:a070f41cdf1dae295992050a46bb80cecedb087ca73342729bba743ab09d5fcc
Status: Downloaded newer image for ghcr.io/open-webui/open-webui:main
620c9c48b8eba8cee1f0e96ccac34b10157dda1db19e95f16bd1dd84bc5d3bd0

wsluser@pc:~$ sudo docker stop open-webui
open-webui

wsluser@pc:~$ sudo docker start open-webui
open-webui

wsluser@pc:~$ sudo docker ps -a
CONTAINER ID   IMAGE                                COMMAND               CREATED          STATUS
    PORTS                                           NAMES
44010c2504fc   ollama/ollama                        "/bin/ollama serve"   14 minutes ago   Up 14 minutes            0.0.0.0:11434->11434/tcp, :::11434->11434/tcp   ollama
620c9c48b8eb   ghcr.io/open-webui/open-webui:main   "bash start.sh"       18 minutes ago   Up 5 minutes (healthy)   0.0.0.0:3000->8080/tcp, [::]:3000->8080/tcp     open-webui

wsluser@pc:~$ sudo docker inspect ollama --format='{{json .Mounts}}'
[{"Type":"volume","Name":"ollama","Source":"/var/lib/docker/volumes/ollama/_data","Destination":"/root/.ollama","Driver":"local","Mode":"z","RW":true,"Propagation":""}]

wsluser@pc:~$ sudo ls -l /var/lib/docker/volumes/ollama/_data
total 12
-rw------- 1 root root  387 Jan 16 21:18 id_ed25519
-rw-r--r-- 1 root root   81 Jan 16 21:18 id_ed25519.pub
drwxr-xr-x 4 root root 4096 Jan 16 21:27 models
```

<http://localhost:3000/> にアクセス可能。

`sudo docker exec -it ollama /bin/sh` からの `df -h` や `mount` でもマウントしたディレクトリの場所がわかる。シェルからは `Ctrl + d` で抜ける。また、 `sudo` なしで動かせるようにしておかないと面倒なことになるかもしれない。

ollama を使った推論を実行すると、Open WebUI でモデルが選べるようになるそうだ。試してみよう。

```shell
wsluser@pc:~$ sudo docker exec -it ollama ollama run jean-luc/tiger-gemma-9b-v3:fp16
pulling manifest
pulling 1c130376047e... 100% ▕████████████████████████████████████████████████▏  18 GB
pulling 282241528150... 100% ▕████████████████████████████████████████████████▏  137 B
pulling 2490e7468436... 100% ▕████████████████████████████████████████████████▏   65 B
pulling bcffed2643f6... 100% ▕████████████████████████████████████████████████▏  412 B
verifying sha256 digest
writing manifest
success
>>> こんにちは！日本語で会話できますか？
はい、日本語で会話できます。何か質問があれば聞いてください！

>>> /bye
```

おお、確かに `jean-luc/tiger-gemma-9b-v3:fp16` が選べるようになった。

次は、 Meta が開発したという Llama3 の uncensord モデルを動かしてみよう。

```shell
wsluser@pc:~$ sudo docker exec -it ollama ollama run hf.co/Orenguteng/Llama-3-8B-Lexi-Uncensored-GGUF:F16
[sudo] password for wsluser:
pulling manifest
pulling cfce48ca57f0... 100% ▕████████████████████████████████████████████████▏  16 GB
pulling 62fbfd9ed093... 100% ▕████████████████████████████████████████████████▏  182 B
pulling b78301c0df4d... 100% ▕████████████████████████████████████████████████▏   38 B
pulling eef4a93c7add... 100% ▕████████████████████████████████████████████████▏  193 B
verifying sha256 digest
writing manifest
success
>>> Hi, can you speak Japanese? If you can, please reply me in Japanese.
Yes, I can. Here's a response in Japanese:
Konnichiwa, nan desu ka? Wagahai wa Nihongo ga yoku hanasu kareru koto ga arimasu.

(Words roughly translate to: "Hello, how are you? I can speak Japanese.")

Please note that my knowledge of Japanese is limited and not perfect. If your question is complex or
requires a nuanced understanding of cultural context, please be patient with any mistakes or inaccuracies
in my responses.

>>> /bye
```

こちらはちと推論に時間がかかってしまった。GPU メモリが多くないと厳しいようだ。

続いて、 DeepSeek を動かしてみよう。

```shell
wsluser@pc:~$  docker exec -it ollama ollama run deepseek-r1:14b
pulling manifest
pulling 6e9f90f02bb3... 100% ▕████████████████████████████████████████████████▏ 9.0 GB
pulling 369ca498f347... 100% ▕████████████████████████████████████████████████▏  387 B
pulling 6e4c38e1172f... 100% ▕████████████████████████████████████████████████▏ 1.1 KB
pulling f4d24e9138dd... 100% ▕████████████████████████████████████████████████▏  148 B
pulling 3c24b0c80794... 100% ▕████████████████████████████████████████████████▏  488 B
verifying sha256 digest
writing manifest
success
>>> Hi can you speak Japanese? If you can, please reply me in Japanese.
<think>
Alright, the user is asking if I can speak Japanese and has instructed to reply in Japanese if possible.

I should confirm that I do understand Japanese but also clarify any limitations.

It's important to let them know I can assist with basic questions but might struggle with complex or
nuanced topics.

Maintaining a polite tone is crucial since we're dealing with another language.
</think>

はい、私は日本語を話すことができます。どうぞよろしくお願いいたします。
```

DeepSeek の方が演算が早いな。科学の進歩ってすげー。 uncensored 版があるらしいので試してみよう。

```shell
wsluser@pc:~$ docker exec -it ollama ollama run huihui_ai/deepseek-r1-abliterated:14b
pulling manifest
pulling manifest
pulling 38b5e2007867... 100% ▕████████████████████████████████████████████████▏ 9.0 GB
pulling 369ca498f347... 100% ▕████████████████████████████████████████████████▏  387 B
pulling 6e4c38e1172f... 100% ▕████████████████████████████████████████████████▏ 1.1 KB
pulling f4d24e9138dd... 100% ▕████████████████████████████████████████████████▏  148 B
pulling 15f56c9d303a... 100% ▕████████████████████████████████████████████████▏  488 B
verifying sha256 digest
writing manifest
success
>>> Hi can you speak Japanese? If you can, please reply me in Japanese.
<think>
Okay, the user asked if I can speak Japanese and to reply in Japanese. I responded in Japanese, but let me
make sure it's appropriate.

I want to acknowledge their request and confirm my ability. So, "はい、日本語で話せます。" means "Yes, I
can speak Japanese."

Then, maybe add a friendly note like, "どのようなお問い合わせですか？" which means "What is your inquiry?"
or "How can I assist you?"

That should be clear and helpful.
</think>

はい、日本語で話せます。どのようなお問い合わせですか？

>>> Send a message (/? for help)
```

GPU が動いているか知りたい場合、以下のコマンドを入力すると 2 秒ごとに GPU のメモリ使用量が表示される。

```shell
wsluser@pc:~$ sudo docker exec -it ollama /bin/sh
# watch nvidia-smi
```

表示内容はこんな感じ。

```shell
Every 2.0s: nvidia-smi                                                    44010c2504fc: Thu Jan 16 13:35:22 2025

Thu Jan 16 13:35:22 2025
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 560.35.02              Driver Version: 560.94         CUDA Version: 12.6     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4060 Ti     On  |   00000000:06:00.0 Off |                  N/A |
| 61%   42C    P2             60W /  165W |   15005MiB /  16380MiB |     31%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A      2219      C   /ollama_llama_server                        N/A      |
+-----------------------------------------------------------------------------------------+
```

ctrl+cで終了。 GPU Memory Usage に使用量が表示されるはずなのだが、なぜか N/A になっている。うーむ。

Stable Diffusion WebUI AUTOMATIc1111 Dockerは後にしよう。とりあえず、あれはWindodws版で動かした実績があるので。
でも、pythonのバージョン固定とかが必要なので、dockerにしたほうが良いのかな。

## 2.2. docker コンテナの Ollama をアップデートする

docker コンテナを再構築する必要がある。ちなみに、 -i: interactive, -t: tty, -d: detach, -v: volume という意味である。

```shell
# ollama コンテナを停止する。
wsluser@pc:~$ docker stop ollama
ollama

# ollama コンテナを削除する（データは残る。）
wsluser@pc:~$ docker rm ollama
ollama

# ollama コンテナを再取得してデタッチ実行する。
wsluser@pc:~$ docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:latest
a8e397bfbdc497953ba8322321990fbe417264f2037a0918596751075bcb30a5

# バージョンがそろった。
wsluser@pc:~$ docker exec -it --user root ollama bash
root@a8e397bfbdc4:/# ollama --version
ollama version is 0.12.6

root@a8e397bfbdc4:/# ollama list
NAME                                                    ID              SIZE      MODIFIED
huihui_ai/gpt-oss-abliterated:20b-v2-q4_K_M             35c222309bbd    15 GB     2 hours ago
huihui_ai/deepseek-r1-abliterated:14b                   6b2209ffd758    9.0 GB    8 months ago
deepseek-r1:14b                                         ea35dfe18182    9.0 GB    8 months ago
hf.co/Orenguteng/Llama-3-8B-Lexi-Uncensored-GGUF:F16    fd9d793ede83    16 GB     9 months ago
jean-luc/tiger-gemma-9b-v3:fp16                         eeb453a4d354    18 GB     9 months ago
```

### 2.2.1. 通常インストールした Ollama をアップデートする

以下のように docker コンテナをインタラクティブかつターミナルありで動かし、 ollama 公式ドキュメントに従って ollama を再インストールするスクリプトを実行すればよい。なお、以下の実行例は docker 上で実行したものなので、最終的にはサーバー側とクライアント側のバージョンに差が出てしまっている。失敗例なので、 docker 上で実行しないように。

```shell
wsluser@pc:~$ docker start ollama
ollama

wsluser@pc:~$ docker exec -it --user root ollama bash
root@44010c2504fc:/# curl -fsSL https://ollama.com/install.sh | sh
>>> Installing ollama to /usr/local
>>> Downloading Linux amd64 bundle
######################################################################## 100.0%
>>> Nvidia GPU detected.
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.
>>> The Ollama API is now available at 127.0.0.1:11434.
>>> Install complete. Run "ollama" from the command line.
```

curl がインストールされていない場合、 root ユーザーで docker コンテナへログインし、 curl をインストールしてから ollama の更新コマンドを実行する。

```shell
wsluser@pc:~$ docker exec -it --user root ollama bash
root@44010c2504fc:/# apt update
（結果は省略）
root@44010c2504fc:/# apt updgrade
（結果は省略）
root@44010c2504fc:/# apt install curl
（結果は省略）
root@44010c2504fc:/# curl -fsSL https://ollama.com/install.sh | sh
```

ちなみに、 apt update を実行しない場合は curl のインストールに失敗する。

```shell
wsluser@pc:~$ docker exec -it --user root ollama apt install curl
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package curl
```

Ollama のバージョンは `--version` オプションで確認できる。

```shell
wsluser@pc:~$ docker exec -it --user root ollama bash
root@44010c2504fc:/# ollama --version
ollama version is 0.5.6-0-g2539f2d-dirty
Warning: client version is 0.12.6
root@44010c2504fc:/# exit
exit
```

この状態、 Ollama のクライアントバージョンだけが上がっており、サーバーのバージョンは古いまま。

## 2.3. docker コンテナの Open WebUI をアップデートする

```shell
# 現在のコンテナを停止。
wsluser@pc:~$ docker stop open-webui
open-webui

# 現在のコンテナを削除。
wsluser@pc:~$ docker rm -f open-webui
open-webui

# 最新の docker イメージをプル。
wsluser@pc:~$ docker pull ghcr.io/open-webui/open-webui:main
main: Pulling from open-webui/open-webui
5c32499ab806: Pull complete
923f706c7da6: Pull complete
3305e2b7fc7c: Pull complete
13973b95de7b: Pull complete
b124c04c3dda: Pull complete
4f4fb700ef54: Pull complete
93f7d71a43ad: Pull complete
77127ed2bb0c: Pull complete
5cc688decb4c: Pull complete
11ba7592cf55: Pull complete
94bdc704e2ab: Pull complete
3105e7a030e6: Pull complete
71e9b7206fa8: Pull complete
4b74cd7f42f2: Pull complete
c41c20ad99da: Pull complete
Digest: sha256:d6ab9bce3030f5b58e59144a7db9fe3012b39c94d27bc2979e98e6d6dc2161da
Status: Downloaded newer image for ghcr.io/open-webui/open-webui:main
ghcr.io/open-webui/open-webui:main

# docker イメージを実行する。
wsluser@pc:~$ docker run -p 3000:8080 --env WEBUI_AUTH=False --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
Loading WEBUI_SECRET_KEY from file, not provided as an environment variable.
Generating WEBUI_SECRET_KEY
Loading WEBUI_SECRET_KEY from .webui_secret_key
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade 3781e22d8b01 -> 9f0c9cd09105, Add note table
INFO  [alembic.runtime.migration] Running upgrade 9f0c9cd09105 -> d31026856c01, Update folder table data
INFO  [alembic.runtime.migration] Running upgrade d31026856c01 -> 018012973d35, Add indexes
INFO  [alembic.runtime.migration] Running upgrade 018012973d35 -> 3af16a1c9fb6, update user table
INFO  [alembic.runtime.migration] Running upgrade 3af16a1c9fb6 -> 38d63c18f30f, Add oauth_session table
INFO  [alembic.runtime.migration] Running upgrade 38d63c18f30f -> a5c220713937, Add reply_to_id column to message
INFO  [open_webui.env] 'ENABLE_SIGNUP' loaded from the latest database entry
INFO  [open_webui.env] 'DEFAULT_LOCALE' loaded from the latest database entry
INFO  [open_webui.env] 'DEFAULT_PROMPT_SUGGESTIONS' loaded from the latest database entry
WARNI [open_webui.env]

WARNING: CORS_ALLOW_ORIGIN IS SET TO '*' - NOT RECOMMENDED FOR PRODUCTION DEPLOYMENTS.

INFO  [open_webui.env] VECTOR_DB: chroma
INFO  [open_webui.env] Embedding model set: sentence-transformers/all-MiniLM-L6-v2
WARNI [langchain_community.utils.user_agent] USER_AGENT environment variable not set, consider setting it to identify your requests.

 ██████╗ ██████╗ ███████╗███╗   ██╗    ██╗    ██╗███████╗██████╗ ██╗   ██╗██╗
██╔═══██╗██╔══██╗██╔════╝████╗  ██║    ██║    ██║██╔════╝██╔══██╗██║   ██║██║
██║   ██║██████╔╝█████╗  ██╔██╗ ██║    ██║ █╗ ██║█████╗  ██████╔╝██║   ██║██║
██║   ██║██╔═══╝ ██╔══╝  ██║╚██╗██║    ██║███╗██║██╔══╝  ██╔══██╗██║   ██║██║
╚██████╔╝██║     ███████╗██║ ╚████║    ╚███╔███╔╝███████╗██████╔╝╚██████╔╝██║
 ╚═════╝ ╚═╝     ╚══════╝╚═╝  ╚═══╝     ╚══╝╚══╝ ╚══════╝╚═════╝  ╚═════╝ ╚═╝


v0.6.34 - building the best AI user interface.

https://github.com/open-webui/open-webui

Fetching 30 files: 100%|██████████| 30/30 [00:00<00:00, 297468.37it/s]
INFO:     Started server process [1]
INFO:     Waiting for application startup.
2025-10-24 11:28:56.333 | INFO     | open_webui.utils.logger:start_logger:162 - GLOBAL_LOG_LEVEL: INFO
2025-10-24 11:28:56.333 | INFO     | open_webui.main:lifespan:561 - Installing external dependencies of functions and tools...
2025-10-24 11:28:56.341 | INFO     | open_webui.utils.plugin:install_frontmatter_requirements:283 - No requirements found in frontmatter.

# Ctrl+C を押下。
^C

wsluser@pc:~$ docker stop open-webui
open-webui

wsluser@pc:~$ docker start open-webui
open-webui
```

## 2.4. Ollama のモデルを削除する

docker の中に入って削除する。

```shell
wsluser@pc:~$ docker exec -it --user root ollama bash

root@44010c2504fc:/# ollama list
NAME                                                    ID              SIZE      MODIFIED
huihui_ai/gpt-oss-abliterated:20b                       81c701845c49    13 GB     2 hours ago
huihui_ai/deepseek-r1-abliterated:14b                   6b2209ffd758    9.0 GB    8 months ago
deepseek-r1:14b                                         ea35dfe18182    9.0 GB    8 months ago
hf.co/Orenguteng/Llama-3-8B-Lexi-Uncensored-GGUF:F16    fd9d793ede83    16 GB     9 months ago
jean-luc/tiger-gemma-9b-v3:fp16                         eeb453a4d354    18 GB     9 months ago

root@44010c2504fc:/# ollama rm huihui_ai/gpt-oss-abliterated:20b
Warning: unable to stop model 'huihui_ai/gpt-oss-abliterated:20b'
deleted 'huihui_ai/gpt-oss-abliterated:20b'

root@44010c2504fc:/# ollama list
NAME                                                    ID              SIZE      MODIFIED
huihui_ai/deepseek-r1-abliterated:14b                   6b2209ffd758    9.0 GB    8 months ago
deepseek-r1:14b                                         ea35dfe18182    9.0 GB    8 months ago
hf.co/Orenguteng/Llama-3-8B-Lexi-Uncensored-GGUF:F16    fd9d793ede83    16 GB     9 months ago
jean-luc/tiger-gemma-9b-v3:fp16                         eeb453a4d354    18 GB     9 months ago

root@44010c2504fc:/# exit
exit
```

## 2.5. エラー対応

### 2.5.1. `500: template: :3: function "currentDate" not defined`

gpt-oss とその改造版で発生を確認した。

```shell
root@44010c2504fc:/# ollama run huihui_ai/gpt-oss-abliterated:20b-v2-q4_K_M
Error: template: :3: function "currentDate" not defined
```

- <https://ollama.com/library/gpt-oss>
- <https://huggingface.co/huihui-ai/Huihui-gpt-oss-20b-BF16-abliterated>
- <https://huggingface.co/huihui-ai/Huihui-gpt-oss-20b-BF16-abliterated-v2>

どうやらテンプレートファイルの `currentDate` というマクロが機能しなかったことが原因らしい（ Ollama が提供しているはずの API が動かなかったそうだ。）

> <https://ollama.com/huihui_ai/gpt-oss-abliterated:20b-v2-q4_K_M/blobs/51468a0fd901#:~:text=,language%20model%20trained%20by%20OpenAI>
>
> ```text
> <|start|>system<|message|>You are ChatGPT, a large language model trained by OpenAI.
> Knowledge cutoff: 2024-06
> Current date: {{ currentDate }}
> {{- if and .IsThinkSet .Think (ne .ThinkLevel "") }}
> ```

Ollama v0.11.2 で解決済みらしいので、バージョンを上げればよい。しかし、 docker 上の Ollama のバージョンを上げるには通常のバージョンアップ方法ではなく、 docker コンテナの再ダウンロードが必要なので要注意。

### 2.5.2. Ollama でモデルのダウンロードが終わらない

何度もダウンロードをやり直してしまうことがある。原因は謎。

ダウンロード途中のファイルは `-partial` という名前になり、ダウンロードに失敗した断片ファイルは `-partial-n` という名前になるっぽいので、ダウンロード途中のファイル名から `-partial` を削除し、残りの `-partial-n` のファイルたちはファイル削除して再度ダウンロードすると、ダウンロードに成功する可能性がある。

```shell
wsluser@pc:~$ docker exec -it --user root ollama bash

root@44010c2504fc:/# ollama pull huihui_ai/gpt-oss-abliterated:20b-v2-q4_K_M
pulling manifest
pulling ba378c67d733:  97% ▕████████████████████████████████████████████████  ▏  15 GB/ 15 GB  1.2 MB/s   5m43s
^C

# ~/.ollama/models/blobs/ を見る。
root@44010c2504fc:/# ls ~/.ollama/models/blobs/sha256-
sha256-15f56c9d303aa516488cde7730723817f7c6553a4b3983a527a6ba86d7c6df0d
sha256-1c130376047ee3a780a3a39eef057a096c9af4bacaaa43d831656cab59aa65cc
sha256-2490e7468436707d5156d7959cf3c6341cc46ee323084cfa3fcf30fe76e397dc
sha256-282241528150437678e0c62aaebf03d84527f1ab9763b4f8df58d6b2f4b3345e
sha256-369ca498f347f710d068cbb38bf0b8692dd3fa30f30ca2ff755e211c94768150
sha256-38b5e20078675a1e3040eced1859e432b423ec732c42f5dab03b0a8ae7ba1bdd
sha256-3c24b0c80794f0eb6e0de0033f8e3203075db1c4640e837e097712a4b88d393b
sha256-62fbfd9ed093d6e5ac83190c86eec5369317919f4b149598d2dbb38900e9faef
sha256-6e4c38e1172f42fdbff13edf9a7a017679fb82b0fde415a3e8b3c31c6ed4a4e4
sha256-6e9f90f02bb3b39b59e81916e8cfce9deb45aeaeb9a54a5be4414486b907dc1e
sha256-b78301c0df4d16b2d9ad21cc9448817c816e2efe94fc2ed072cb85b0928e4030
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-0
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-1
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-10
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-11
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-12
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-13
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-14
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-15
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-2
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-3
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-4
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-5
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-6
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-7
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-8
sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial-9
sha256-bcffed2643f6a41a924a7b2b6f9ca84032ae19284747fff688f8873883eaebb5
sha256-cfce48ca57f022cd3676c0623dbc0feb387a043d403ed32b216d3fabd9e5c5ca
sha256-eef4a93c7addcb9b7d2119faa60a38125905cd149def1467ac7168159f74e57b
sha256-f4d24e9138dd4603380add165d2b0d970bef471fac194b436ebd50e6147c6588

# -partial-0 から -partial-n と名前が付いたファイルはダウンロードに失敗したファイルなので消す。
root@44010c2504fc:/# rm ~/.ollama/models/blobs/sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558
801564-partial-*

# *-partial と名前が付いたファイルがサイズの大きいファイル。このファイルは名前から -partial を消す。
root@44010c2504fc:/# mv ~/.ollama/models/blobs/sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564-partial ~/.ollama//models/blobs/sha256-ba378c67d733876c123f6cdde7655ea8074c87f619e0fd546841cb4558801564

# 再度プルするとダウンロードに成功する（ことがある。）
root@44010c2504fc:/# ollama pull huihui_ai/gpt-oss-abliterated:20b-v2-q4_K_M
pulling manifest
pulling ba378c67d733: 100% ▕██████████████████████████████████████████████████▏  15 GB
pulling 51468a0fd901: 100% ▕██████████████████████████████████████████████████▏ 7.4 KB
pulling f60356777647: 100% ▕██████████████████████████████████████████████████▏  11 KB
pulling d8ba2f9a17b3: 100% ▕██████████████████████████████████████████████████▏   18 B
pulling f0413f74bed0: 100% ▕██████████████████████████████████████████████████▏  492 B
verifying sha256 digest
writing manifest
success
```

### 2.5.3. docker コンテナの Ollama のサーバー（デーモン）バージョンが上がらない

docker コンテナ上の Ollama を `curl -fsSL https://ollama.com/install.sh | sh` でバージョンアップすると、サーバーとクライアントのバージョンに差分が出てしまう。差分が出ている場合には以下のように Warning が表示される。

```shell
wsluser@pc:~$ docker exec -it --user root ollama bash
root@44010c2504fc:/# ollama --version
ollama version is 0.5.6-0-g2539f2d-dirty
Warning: client version is 0.12.6
```

docker コンテナ上の Ollama をバージョンアップする場合、 Ollama の docker コンテナを再構築しなければならない。

```shell
# ollama コンテナを停止する。
wsluser@pc:~$ docker stop ollama
ollama

# ollama コンテナを削除する（データは残る。）
wsluser@pc:~$ docker rm ollama
ollama

# ollama コンテナを再取得してデタッチ実行する。
wsluser@pc:~$ docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:latest
a8e397bfbdc497953ba8322321990fbe417264f2037a0918596751075bcb30a5

# バージョンがそろった。
wsluser@pc:~$ docker exec -it --user root ollama bash
root@a8e397bfbdc4:/# ollama --version
ollama version is 0.12.6

root@a8e397bfbdc4:/# ollama list
NAME                                                    ID              SIZE      MODIFIED
huihui_ai/gpt-oss-abliterated:20b-v2-q4_K_M             35c222309bbd    15 GB     2 hours ago
huihui_ai/deepseek-r1-abliterated:14b                   6b2209ffd758    9.0 GB    8 months ago
deepseek-r1:14b                                         ea35dfe18182    9.0 GB    8 months ago
hf.co/Orenguteng/Llama-3-8B-Lexi-Uncensored-GGUF:F16    fd9d793ede83    16 GB     9 months ago
jean-luc/tiger-gemma-9b-v3:fp16                         eeb453a4d354    18 GB     9 months ago
```

## 2.6. 参考資料

### 2.6.1. 公式資料

- Ollama 公式サイト: <https://ollama.com/>
- Ollama 公式ドキュメント: <https://github.com/ollama/ollama/blob/main/docs/linux.md>
- Open WebUI 公式サイト: <https://docs.openwebui.com/>
- Open WebUI 公式アップデート手順: <https://docs.openwebui.com/getting-started/updating/>

### 2.6.2. ありがたき先人たちの教え

- karaage0703, Dockerで構築する機械学習環境【2024年版】, Zenn, 2024-10-08, <https://zenn.dev/mkj/articles/33befbaf38c693>
- karaage0703, DeepSeekが凄そうなのでOllamaを使ってローカルで動かして体感してみた, Zenn, 2025-02-15, <https://zenn.dev/karaage0703/articles/3135a88f603e3e>
- yumizu, WSL2+Ubuntu24.04+Docker＋GPUでつくる機械学習環境, Zenn, 2024-06-06, <https://zenn.dev/yumizz/articles/627d4e4821c636>
- yumizu, GPUの型番にあったCUDAバージョンの選び方, Zenn, 2024-05-09, <https://zenn.dev/yumizz/articles/73d6c7d1085d2f>
- -, Installing the NVIDIA Container Toolkit, NVIDIA CONTAINER TOOLKIT, 2024-12-23, <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html>
- JAP, ローカルにWebUI付きLLM環境を構築 with Ollama, note, 2024-03-03, <https://note.com/jap4ai/n/n632a2270b084#eeb9b78f-d97c-4989-a1d5-0664ff664f02>
- Catapp-Art3D, 【無修正】Llama3 Uncensored を試す【脱獄モデル】, 2024-04-26, <https://note.com/catap_art3d/n/n4cfcfa41289e>
- -, Llama3とは？使い方から性能、商用利用まで分かりやすく解説！, EdgeHUB, 2024-09-11, <https://highreso.jp/edgehub/machinelearning/llama3-howto.html>
- ©nkmk.me, watch nvidia-smiでGPU使用率などを確認・リアルタイムモニタリング, note.nkmk.me, 2021-03-06, <https://note.nkmk.me/nvidia-smi-monitoring-gpu/>
- @kazokmr, Docker を使っている時に調べたことまとめ, Qiita, 2021-06-05, <https://qiita.com/kazokmr/items/1ffc77d01a67aff90c75>
- @kosuke_aizawa (宏亮 相澤), No.6 新卒未経験エンジニアがDockerを使ってチョメチョメしてみた〜チュートリアル編〜, Qiita, 2017-10-28, <https://qiita.com/kosuke_aizawa/items/4abab88caaae119545cf>
- Mikael Svenson, How to Run Uncensored DeepSeek R1 on Your Local Machine, Apidog, 2025-02-15, <https://apidog.com/blog/deepseek-r1-abliterated/>
- tunamayo, ollamaモデルの削除・探し方・追加, note, 2025/07/27, <https://note.com/tororo000/n/n766dd35dd3df>

## 2.7. より高度な話

- Karaage0703, Dockerで構築する機械学習環境【2024年版】, Zenn, 2024-10-08, <https://zenn.dev/mkj/articles/33befbaf38c693>
