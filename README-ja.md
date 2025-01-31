[English](./README.md) | [简体中文](./README-zh.md) | 日本語

# 概要

Milk-V DuoはCV1800Bをベースにした超小型の組み込みプラットフォームです。LinuxとRTOSが実行可能で、プロフェッショナル、産業用ODM、AIoT、DIY、クリエイターに、高信頼で高費用対効果なプラットフォームを提供します。

## ハードウェア

- CPU: CVITEK CV1800B (C906@1Ghz + C906@700MHz)
- デュアルコアRV64 最大1GHz
- 64MB RAM
- アドオンボード経由での10/100Mbpsイーサネット

# SDKのディレクトリ構造

```text
├── build               // compilation scripts and board configs コンパイルスクリプトとボード設定
├── build_milkv.sh      // one-click compilation script 自動コンパイルスクリプト
├── buildroot-2021.05   // buildroot source code Buildrootのソースコード
├── freertos            // freertos system freeRTOSのシステム
├── fsbl                // fsbl firmware in prebuilt form 完成済みfsblファームウェア
├── install             // temporary images stored here 一時イメージの仮置き場
├── isp_tuning          // camera effect parameters カメラ効果パラメータ
├── linux_5.10          // linux kernel Linuxカーネル
├── middleware          // self-developed multimedia framework 自家製マルチメディアフレームワーク
├── milkv               // configuration files for milkv Milk-Vのコンフィグレーションファイル
├── opensbi             // opensbi library opensbiライブラリ
├── out                 // final image for SD card 完成したSDカード用イメージはここに出てきます
├── ramdisk             // prebuilt ramdisk 完成済みramsidk
└── u-boot-2021.10      // u-boot source code u-bootのソースコード
```

# クイックスタート

Prepare the Compilation Environment. Using a local Ubuntu system, the officially supported compilation environment is `Ubuntu Jammy 22.04.x amd64` only!

If you are using other Linux distributions, we strongly recommend that you use the Docker environment to compile to reduce the probability of compilation errors.

The following describes the compilation methods in the two environments.

## 1. Compiled using Ubuntu 22.04

### Packages to be installed

Install the packages that compile dependencies:

```bash
sudo apt install -y pkg-config build-essential ninja-build automake autoconf libtool wget curl git gcc libssl-dev bc slib squashfs-tools android-sdk-libsparse-utils jq python3-distutils scons parallel tree python3-dev python3-pip device-tree-compiler ssh cpio fakeroot libncurses5 flex bison libncurses5-dev genext2fs rsync unzip dosfstools mtools tcl openssh-client cmake
```

### Get SDK Source Code

```bash
git clone https://github.com/milkv-duo/duo-buildroot-sdk.git --depth=1
```

### <1>. One-click Compilation

- Execute the one-click compilation script `build_milkv.sh`:

```
cd duo-buildroot-sdk/
./build_milkv.sh
```

- After a successful compilation, you can find the generated SD card burning image `milkv-duo-*-*.img` in the `out` directory.

*Note: The first compilation will automatically download the required toolchain, which is approximately 840MB in size. Once downloaded, it will be automatically extracted to the `host-tools` directory in the SDK directory. For subsequent compilations, if the `host-tools` directory is detected, the download will not be performed again*.

### <2>. Step-by-step Compilation

If you wish to perform step-by-step compilation, you can enter the following commands sequentially:

```bash
export MILKV_BOARD=milkv-duo
source milkv/boardconfig-milkv-duo.sh

source build/milkvsetup.sh
defconfig cv1800b_milkv_duo_sd
clean_all
build_all
pack_sd_image
```

Location of the generated image: `install/soc_cv1800b_milkv_duo_sd/milkv-duo.img`.

## 2. Compiled using Docker

Docker support is required on hosts running Linux systems. For how to use Docker, please refer to the [official documentation](https://docs.docker.com/) or other tutorials.

We put the SDK source code on the Linux host system and call the Docker image environment provided by Milk-V to compile it.

### Pull SDK code on Linux host

```
git clone https://github.com/milkv-duo/duo-buildroot-sdk.git --depth=1
```

### Enter the SDK code directory

```
cd duo-buildroot-sdk
```

### Pull the Docker image and run

```
docker run -itd --name duodocker -v $(pwd):/home/work milkvtech/milkv-duo:latest /bin/bash
```

Description of some parameters in the command:
- `duodocker` Docker name, you can use the name you want to use.
- `$(pwd)` The current directory, here is the duo-buildroot-sdk directory that was 'cd' to in the previous step.
- `-v $(pwd):/home/work`  Bind the current code directory to the /home/work directory in the Docker image.
- `milkvtech/milkv-duo:latest` The Docker image provided by Milk-V will be automatically downloaded from hub.docker.com for the first time.

After Docker runs successfully, you can use the `docker ps -a` command to view the running status:
```
$ docker ps -a
CONTAINER ID   IMAGE                        COMMAND       CREATED       STATUS       PORTS     NAMES
8edea33c2239   milkvtech/milkv-duo:latest   "/bin/bash"   2 hours ago   Up 2 hours             duodocker
```

### <1>. One-click compilation using Docker

```
docker exec duodocker /bin/bash -c "cd /home/work && cat /etc/issue && ./build_milkv.sh"
```

Description of some parameters in the command:
- `duodocker` The name of the running Docker must be consistent with the name set in the previous step.
- `"*"` In quotes is the shell command to be run in the Docker image.
- `cd /home/work` Switch to the /home/work directory. Since this directory has been bound to the host's code directory during runtime, the /home/work directory in Docker is the source code directory of the SDK.
- `cat /etc/issue` Displays the version number of the image used by Docker. It is currently Ubuntu 22.04.3 LTS and is used for debugging.
- `./build_milkv.sh` Execute one-click compilation script.

After successful compilation, you can see the generated SD card burning image `milkv-duo-*-*.img` in the `out` directory.

### <2>. Compile step by step using Docker

Step-by-step compilation requires logging into Docker to operate. Use the command `docker ps -a` to view and record the ID number of the container, such as 8edea33c2239.

Enter Docker:
```
docker exec -it 8edea33c2239 /bin/bash
```

Enter the code directory bound in Docker：
```
root@8edea33c2239:/# cd /home/work/
```

Compile step by step：
```bash
export MILKV_BOARD=milkv-duo
source milkv/boardconfig-milkv-duo.sh

source build/milkvsetup.sh
defconfig cv1800b_milkv_duo_sd
clean_all
build_all
pack_sd_image
```

Generated firmware location: `install/soc_cv1800b_milkv_duo_sd/milkv-duo.img`.

After compilation is completed, you can use the `exit` command to exit the Docker environment:
```
root@8edea33c2239:/home/work# exit
```
The generated firmware can also be seen in the host code directory.

### Stop Docker

After compilation is completed, if the above Docker running environment is no longer needed, you can stop it first and then delete it:
```
docker stop 8edea33c2239
docker rm 8edea33c2239
```

## Other compilation considerations

If you want to try to compile this SDK in an environment other than the above two environments, the following are things you may need to pay attention to, for reference only.

### cmake version

Note：`cmake` minimum version requirement is `3.16.5`.

Check the version of `cmake` in the system:

```bash
cmake --version
```

For example, the version of `cmake` installed using apt in the `Ubuntu 20.04` is:

```
cmake version 3.16.3
```

The minimum requirement of this SDK is not met. Manual installation of the latest version `3.27.6` is needed:

```bash
wget https://github.com/Kitware/CMake/releases/download/v3.27.6/cmake-3.27.6-linux-x86_64.sh
chmod +x cmake-3.27.6-linux-x86_64.sh
sudo sh cmake-3.27.6-linux-x86_64.sh --skip-license --prefix=/usr/local/
```

When manually installed, `cmake` is located in `/usr/local/bin`. To check its version, use the command `cmake --version`, which should display:

```
cmake version 3.27.6
```

### Compiling with Windows Linux Subsystem (WSL)

If you wish to perform the compilation with WSL, there's an small issue building the image.
The $PATH, due Windows interoperability, has Windows environment variables which include some spaces between the paths.

To solve this problem you need to change the `/etc/wsl.conf` file and add the following lines:

```
[interop]
appendWindowsPath = false
```

After that, you need to reboot the WSL with `wsl.exe --reboot`. Then you able to run the `./build_milkv.sh` script or the `build_all` line in the step-by-step compilation method.
To rollback this change in `/etc/wsl.conf` file set `appendWindowsPath` as true. To reboot the WSL, can you use the Windows PowerShell command `wsl.exe --shutdown` then `wsl.exe`, after that the Windows environment variables become avaliable again in $PATH.

## SDカードへの書き込み

> 注意：書き込むとmicroSDカード内のデータはすべて削除されます。重要なデータはバックアップしてください！！！
- Windows上で生成したイメージを書き込む場合`balenaEtcher`や`Rufus`、`Win32 Disk Imager`などのツールを使えます。
- Linux上で生成したイメージを書き込む場合ddコマンドを使えます。 **ddコマンドで書き込む場合指定するデバイスが書き込むmicroSDカードであることを十二分に確認してください**

  ```
  sudo dd if=milkv-duo-*-*.img of=/dev/sdX
  ```

## 電源投入

- Milk-V DuoのmicroSDカードスロットにmicroSDカードを挿入します。
- シリアルケーブルを接続します。(必ずしも必要ではありません)
- 電源に接続するとシステムが立ち上がります。
- シリアルケーブルが接続されている場合、ブートログをシリアルコンソールから見ることができます(`mobarXterm`、`Xshell` など)。システムが立ち上がればコンソールからログインしてLinuxコマンドを実行できます。

### Duoのターミナルに入る方法

- シリアルケーブルを使う
- USBネットワークを使う(RNDIS)
- イーサネットインターフェースを使う(アドオンボードが必要)

Duoのターミナルに入る際に必要なユーザー名とパスワードは以下の通りです。

```text
root
milkv
```

### LEDの点滅の無効化

もしDuoに載っているLEDの点滅を無効にしたいならDuoのターミナルで以下のコマンドを実行してください。

```bash
mv /mnt/system/blink.sh /mnt/system/blink.sh_backup && sync
```

以上のコマンドはLEDの点滅スクリプトをリネームしています。そしてDuoを再起動するとLEDは点滅しなくなります。

DuoのLEDをまた点滅させたい場合、スクリプトのファイル名を元に戻します。

```bash
mv /mnt/system/blink.sh_backup /mnt/system/blink.sh && sync
```

### IO-Boardを使う

IO-Boardを使用する場合、USBネットワーク(RNDIS)は使用できないので、IO-Boardのイーサネットインターフェースを使用してください。
IO-BoardのEthernetポートに固定MACアドレスを割り当てる必要がある場合は、以下のコマンドを実行してください(**コマンド中のMACアドレスは設定したいMACアドレスに置き換えてください。また、同一ネットワークセグメント内の異なるデバイスのMACアドレスは重複してはいけません**)。

```bash
echo "pre-up ifconfig eth0 hw ether 78:01:B3:FC:E8:55" >> /etc/network/interfaces"
```

それからボードを再起動してください。

IO-Board上の4発のUSBポートを有効化する：

```bash
rm /mnt/system/usb.sh
ln -s /mnt/system/usb-host.sh /mnt/system/usb.sh
sync
```

それからボードを再起動してください。

たとえば、USBフラッシュドライブがIO-Board上のUSBポートに接続されている場合`ls /dev/sd*`で認識していることを確認できます。

USBフラッシュドライブをマウントしてその内容を見る(/dev/sda1を例に)。

```bash
mkdir /mnt/udisk
mount /dev/sda1 /mnt/udisk
```

`/mnt/udisk`ディレクトリの内容が予想と一致するか確認する。

```bash
ls /mnt/udisk
```

USBフラッシュドライブをアンマウントするコマンド

```bash
umount /mnt/udisk
```

USBネットワーク(RNDIS)の機能をIO-Board不使用時に使う場合は以下のコマンドを実行してください

```bash
rm /mnt/system/usb.sh
ln -s /mnt/system/usb-rndis.sh /mnt/system/usb.sh
sync
```

それからボードを再起動してください。

# よくある質問

1. なぜ1つのコアしか表示されないのですか。

   CV1800Bチップはデュアルコアですが、現在、Linuxシステムは1つのコアで実行され、もう1つのコアはリアルタイムシステムの実行に使用されています。このコアのSDKはまだリリースされておらず、今後アップデートされる予定です。

2. なぜ28MBしかRAMが使えないのですか。
 
   RAMの一部が[ION](https://github.com/milkv-duo/duo-buildroot-sdk/blob/develop/build/boards/default/dts/cv180x/cv180x_default_memmap.dtsi#L15)に割り当てられており、カメラを使ってアルゴリズムを実行するときに使用するメモリだからです。カメラを使用しない場合は、この[ION_SIZE](https://github.com/milkv-duo/duo-buildroot-sdk/blob/develop/build/boards/cv180x/cv1800b_milkv_duo_sd/memmap.py#L43)の値を0に変更し、再コンパイルしてください。

## チップ製造元のドキュメント

- CV181x/CV180x MMF SDK開発ドキュメント (英語と中国語): [MultiMedia Framework Software Development Document](https://developer.sophgo.com/thread/471.html)
- CVシリーズのTPU SDK開発ドキュメント (中国語): [CV series chip TPU SDK development data summary](https://developer.sophgo.com/thread/473.html)

# Milk-Vについて

- [公式ウェブサイト](https://milkv.io/)

# フォーラム

- [MilkV Community](https://community.milkv.io/)
