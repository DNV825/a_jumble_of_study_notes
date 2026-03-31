---
title: WSL Tips
author: DNV825
created: 2026-03-31
date: 2026-03-31
order: 3
category: Jekyll
layout: post
---

WSL の設定などに関する Tips をまとめる。

## WSLgのアーキテクチャについての説明

<https://devblogs.microsoft.com/commandline/wslg-architecture/>

## 再インストール後にUbuntuが起動しなくなった場合の対応

SDカードをマウントするためにWSL2のカーネルビルドを行い、そのあとにUbuntuを再インストールするとエラーが発生して起動しなくなることがある。

その場合、いったんWSLを終了し、Ubuntuの登録を解除し、そしてUbuntuを再度起動すればよい。

```shell
# powershell
wsl --shutdown
wsl --list
  → Linux 用 Windows サブシステム ディストリビューション:
    Ubuntu-22.04 (既定)
wsl --unregister Ubuntu-22.04
```

## WSLのビープ音を消す手順

WSL にログインし、 `/etc/inputrc`を編集して、ビープ音の設定箇所（set bell-style none）のコメントアウトを外すことで WSL のビープ音を消すことができる。

```shell
wsluser@pc:~$ cat /etc/inputrc
（中略）
# do not bell on tab-completion
 set bell-style none    # ← この行のコメントアウトを外す。
# set bell-style visibl
```

- <https://takake-blog.com/wsl-disable-beep/>

## systemdをPID1で動作させる手順

`/etc/wsl.conf`に以下の記述を追記する。

```text
[boot]
systemd=true
```

そしてPowerShellで`wsl --shutdown`コマンドを実行してからWSLを再起動し、`systemctl`を実行する。エラーが起きなければ、systemdがPID1で起動している。

- <https://snowsystem.net/other/windows/wsl2-ubuntu-systemctl/>

## WSLで起動するx-window-systemのスケールを変更する方法

`/mnt/c/Users/<ユーザー名>/.wslgconfig`というファイルを作り、そこに以下の記述を行うことで x-window-system を Windows 側の設定と同じスケールにできる。

```text
[system-distro-env]
;hi-dpi
WESTON_RDP_HI_DPI_SCALING=true
WESTON_RDP_FRACTIONAL_HI_DPI_SCALING=true
```

ファイル名が「.wsl**g**config」であることに注意。**g**を忘れずに！ なお、`.wslgconfig`のテンプレートは以下のサイトに配置してある。

- WSLg Configuration Options for Debugging
  <https://github.com/microsoft/wslg/wiki/WSLg-Configuration-Options-for-Debugging>

## WSLgを無効化する方法

`/mnt/c/Users/<ユーザー名>/.wslconfig`というファイルを作り、そこに以下の記述を行うことで WSLg を無効化できる。

```text
[wsl2]
guiApplications=false
```

こちらには **g** が付かないことに注意！

- Disable WSLg permanently
  <https://github.com/microsoft/wslg/discussions/523>
- WSLg System Distro
  <https://github.com/microsoft/wslg#wslg-system-distro>

## WSLのディスクスペースを拡張する手順

以下の手順に従い、WSLのディスクスペースを拡張することができます。

<https://learn.microsoft.com/ja-jp/windows/wsl/vhd-size>

リンク先が消失するかもしれないので、念のためこの文書にも手順を記述しておく。

まず、ディストリビューションのインストールパッケージ名を取得する。`wsl -l -v`コマンドでインストール済みのディストリビューション名を確認し、ディスクスペースを拡張したいディストリビューション名に対して `Get-AppxPackage` コマンドを実行する。

Ubuntu を対象にする場合は以下のように実行する。

```powershell
wsl -l -v
Get-AppxPackage -Name "*Ubuntu*" | Select PackageFamilyName
```

コマンドを実行すると、下記のように結果が表示される。

```powershell
PS C:\Users\winuser> Get-AppxPackage -Name "*Ubuntu*" | Select PackageFamilyName

PackageFamilyName
-----------------
CanonicalGroupLimited.Ubuntu22.04LTS_79rhkp1fndgsc
```

このパッケージ名に対応する以下のパスへ移動する。

`%LOCALAPPDATA%\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState`

以下のコマンドで上記のパスを開けるはず。ただし、複数の WSL ディストリビューションをインストールしている場合、すべてのフォルダを開いてしまう。

```powershell
Get-AppxPackage -Name "*Ubuntu*" | % { start $env:LOCALAPPDATA\Packages\$($_.PackageFamilyName)\LocalState }
```

このフォルダには `ext4.vhdx`というファイルが配置されており、これが WSL のディスクの実体である。このファイルを大きくすればディスクスペースを拡張することができる。

次に、**コマンドプロンプトを管理者権限あり**で起動する。「ファイル名を指定して実行」で「cmd」と入力して Ctrl+Shift+Enter を実行するか、 Windows ターミナルで Ctrl キーを押下しながらコマンドプロンプトを選択することで管理者権限ありでコマンドプロンプトを起動できる。

その後は以下の順にコマンドを入力する。

```powershell
diskpart
select vdisk file="{ext4.vhdxのファイルパス}"
detail vdisk
expand vdisk maximun={拡張後の容量（MB単位）}
exit
```

実行すると以下のような表示となります。この例では 320,000 MB（つまり 320 GB）に拡張している。

```shell
Microsoft DiskPart バージョン 10.0.19041.964

Copyright (C) Microsoft Corporation.
コンピューター: Computer

DISKPART> select vdisk file="C:\Users\winuser\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState\ext4.vhdx"

DiskPart により、仮想ディスク ファイルが選択されました。

DISKPART> attach vdisk

デバイスの種類 ID: 0 (不明)
ベンダー ID: {00000000-0000-0000-0000-000000000000} (不明)
状態: 追加済み
仮想サイズ:  256 GB
物理サイズ:  208 GB
ファイル名: C:\Users\winuser\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState\ext4.vhdx
子: いいえ
親ファイル名:
関連付けられたディスク番号: 見つかりません

DISKPART> expand vdisk maximum=320000

  100% 完了しました

DiskPart により、仮想ディスク ファイルは正常に拡張されました。

DISKPART> exit

DiskPart を終了しています...
```

続いて、WSL側でディスクを再マウントして、拡張後のディスク容量を反映する。

まずは拡張前のディスク容量を`df -h`コマンドで確認する。この例では 251 GB になっている。

```shell
wsluser@pc:~$ df -h
Filesystem     Size  Used Avail Use% Mounted on
/dev/sdb       251G   84G  156G  35% /
tmpfs          6.3G     0  6.3G   0% /mnt/wsl
tools          869G  477G  392G  55% /init
none           6.3G     0  6.3G   0% /dev
none           6.3G  4.0K  6.3G   1% /run
none           6.3G     0  6.3G   0% /run/lock
none           6.3G     0  6.3G   0% /run/shm
none           6.3G     0  6.3G   0% /run/user
none           6.3G     0  6.3G   0% /sys/fs/cgroup
drivers        869G  477G  392G  55% /usr/lib/wsl/drivers
lib            869G  477G  392G  55% /usr/lib/wsl/lib
C:\            869G  477G  392G  55% /mnt/c
D:\            932G  444G  488G  48% /mnt/d
E:\             20G   13G  7.4G  64% /mnt/e
F:\            5.3G  2.9G  2.4G  55% /mnt/f
H:\            199G   39G  161G  20% /mnt/h
```

以下の順にコマンドを入力します。ただし、環境によって変化する値があるので注意が必要。

```shell
sudo mount -t devtmpfs none /dev
mount | grep ext4 # ← このコマンドの結果の /dev/sdx の値をメモすること。
sudo resize2fs /dev/sdx {sizeInMegabytes}M # 先ほどのsdxに対して変更後のサイズをMB単位で指定する。
```

以下の例では `/dev/sdb` に対して 320000M を指定している。

```shell
wsluser@pc:~$ sudo mount -t devtmpfs none /dev
[sudo] password for wsluser:
mount: /dev: none already mounted on /dev.
wsluser@pc:~$ mount | grep ext4
/dev/sdb on / type ext4 (rw,relatime,discard,errors=remount-ro,data-ordered)
wsluser@pc:~$ sudo resize2fs /dev/sdb 320000M
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/sdb is mounted on /; on-oline resizing required
old_desc_blocks = 32, new_desk_blocks = 40
The filesystem on /dev/sdb is now 8192000 (4k) blocks long.
```

ここまで実行した後にディスク容量を再確認すると、307Gに拡張されていることが確認できる。

```shell
wsluser@pc:~$ df -h
Filesystem     Size  Used Avail Use% Mounted on
/dev/sdb       307G   84G  209G  35% /
tmpfs          6.3G     0  6.3G   0% /mnt/wsl
tools          869G  477G  392G  55% /init
none           6.3G     0  6.3G   0% /dev
none           6.3G  4.0K  6.3G   1% /run
none           6.3G     0  6.3G   0% /run/lock
none           6.3G     0  6.3G   0% /run/shm
none           6.3G     0  6.3G   0% /run/user
none           6.3G     0  6.3G   0% /sys/fs/cgroup
drivers        869G  477G  392G  55% /usr/lib/wsl/drivers
lib            869G  477G  392G  55% /usr/lib/wsl/lib
C:\            869G  477G  392G  55% /mnt/c
D:\            932G  444G  488G  48% /mnt/d
E:\             20G   13G  7.4G  64% /mnt/e
F:\            5.3G  2.9G  2.4G  55% /mnt/f
H:\            199G   39G  161G  20% /mnt/h
```

## WSL のディスクスペースを圧縮する手順

### apt のキャッシュを削除する

apt のキャッシュは定期的に削除してディスクの空き領域を確保するとよいらしい。 `apt clean` を実行すればよい。

```shell
wsluser@pc:~$ du /var/cache/apt
du: ディレクトリ '/var/cache/apt/archives/partial' を読み込めません: 許可がありません
4       /var/cache/apt/archives/partial
452976  /var/cache/apt/archives
593916  /var/cache/apt
wsluser@pc:~$ sudo apt clean
[sudo] wsluser のパスワード:
wsluser@pc:~$ du /var/cache/apt
du: ディレクトリ '/var/cache/apt/archives/partial' を読み込めません: 許可がありません
4       /var/cache/apt/archives/partial
24      /var/cache/apt/archives
28      /var/cache/apt
```

### ディスクスペースを圧縮する手順

WSL 側でファイルを削除しても `ext4.vhdx` のファイルサイズは減らない。そのため、PCのディスク空き容量が一見するとわからなくなる。そういう場合は vhdx ファイルを圧縮する。圧縮する場合も `diskpart` コマンドを利用する。

その後は以下の順にコマンドを入力する。

```powershell
diskpart
select vdisk file="{ext4.vhdxのファイルパス}"
attach vdisk readonly
compact vdisk
detach vdisk
exit
```

実行例は以下の通り。

```shell
Microsoft DiskPart バージョン 10.0.19041.964

Copyright (C) Microsoft Corporation.
コンピューター: BSC0028386

DISKPART> select vdisk file="C:\Users\winuser\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState\ext4.vhdx"

DiskPart により、仮想ディスク ファイルが選択されました。

DISKPART> attach vdisk readonly

  100% 完了しました

DiskPart により、仮想ディスク ファイルがアタッチされました。

DISKPART> compact vdisk

  100% 完了しました

DiskPart により、仮想ディスク ファイルは正常に圧縮されました。

DISKPART> detach vdisk

DiskPart により、仮想ディスク ファイルがデタッチされました。

DISKPART> exit
```
