---
layout: post
title: 在 WSL2 上安装 Archlinux 并运行 Emacs
date: 2021-03-06T20:36:16+08:00
tags: [Emacs, WSL2, Arch]
categories: [Develop Tools]
---
最近使用 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 安装 Emacs 的人越来越多，并且在 [Using Emacs on Windows with WSL2](https://emacsredux.com/blog/2020/09/23/using-emacs-on-windows-with-wsl2/) 这篇文章中了解到，Emacs 通过 [X410](https://x410.dev/) 可以方便地在 Windows 10 上运行 WSL2 的 GUI Apps 。于是萌生了安装 WSL2 体验 Emacs 的想法。  

### 在 WSL2 中安装 ArchLinux
#### 1. 启用WSL
用管理员打开 PowerShell 并输入
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
#### 2. 升级为WSL2的必要条件
* 对于x64的系统要求win10版本为1903 或者更高
* win + R 输入 winver查看版本

#### 3. 启用虚拟平台
 用管理员打开 PowerShell 并输入
 ```powershell
 dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
 ```
#### 4. 下载Linux内核升级包
下载地址：https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
下载完成后双击安装
#### 5. 将WSL2设置为默认版本
用管理员打开 PowerShell 输入
```powershell
wsl --set-default-version 2
```
到这里WSL就安装好了，下面安装ArchLinux

#### 6. 安装LxRunOffline
通过 Scoop 安装更加方便。也可以直接下载：https://github.com/DDoSolitary/LxRunOffline/releases 解压后将LxRunOffline.exe放入任意一个path文件夹下。
```powershell
scoop install LxRunOffline
```
#### 7. 下载Archlinux
下载地址：https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/
找到 `archlinux-bootstrap-2020.10.01-x86_64.tar.gz`， 注意是 `tar.gz` 文件
#### 8. 安装archlinux到WSL

```powershell
LxRunOffline i -n ArchLinux -f C:\Users\Aqua\Downloads\archlinux-bootstrap-2021.03.01-x86_64.tar.gz -d C:\Users\Aqua\Linux -r root.x86_64
```
设置使用 WSL2:

```powershell
wsl --set-version ArchLinux 2
```

#### 9. 进入系统
```powershell
wsl -d ArchLinux
```
在Arch中执行
```bash
cd /etc/
explorer.exe .
```
注意后面的点，执行这条命令后会用windows的文件管理器打开/etc目录，然后找到pacman.conf，在这个文件最后加入
```
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
然后进入下一级目录pacman.d,编辑里面的mirrolist文件，将China的源注释去掉（选择部分即可）
然后回到Arch，执行
```bash
pacman -Syy
pacman-key --init
pacman-key --populate
pacman -S archlinuxcn-keyring
pacman -S base base-devel vim git wget
```

然后别忘了给当前的root设置密码
```bash
passwd
```
新建一个普通用户
```bash
useradd -m -G wheel -s /bin/bash arch
passwd arch
```

将文件`/etc/sudoers`中的`wheel ALL=(ALL) ALL`那一行前面的注释去掉
```bash
vim /etc/sudoers
```
查看当前用户id
```bash
id -u arch
```
#### 10. 设置使用普通用户登录Archlinux
紧接上一步，退出Arch
```bash
exit
```
在 PowerShell 中执行
```powershell
lxrunoffline su -n ArchLinux -v arch
```
到这里 ArchLinux 在 WSL 上的安装就结束了。

### 安装 Emacs
推荐直接安装 GccEmacs:
```bash
sudo pacman -S emacs-native-comp-git
```
由于 WSL2 是使用虚拟机的方式，每次重启系统都会变更主机 IP, 需要通过下面的方法获取 IP.
将下面的命令加入`.bashrc`:
```
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2; exit;}'):0.0
```
然后执行以下命令应用环境：
 ```
 source ~/.bashrc
 ```
### 启动 Emacs
首先在 Windows 10 中启动 X410, 并在右下角的 X410 图标中打开`Allow Public Access` 选项。在弹出的防火墙对话框中点击允许。
然后在 WSL 终端中执行`emacs &`, 即可打开 GUI 的 Emacs 。

### 存在的问题
1. 在高分辨率的情况下，显示模糊。
可以通过设置 Emacs 的大号字体解决。
2. Emacs 中无法使用 `C-x` 按键。
是因为安装了 AutoHotkey 模拟 Emacs 按键的缘故。

### 后记
Mircosoft 以后将直接支持 Gui Apps, 不需要通过 Xserver 实现， 参考[这个文章](https://devblogs.microsoft.com/commandline/whats-new-in-the-windows-subsystem-for-linux-september-2020/#gui-apps).

### 安装其他依赖
以下内容只是针对本人使用的 [Emacs 配置](https://github.com/Eason0210/emacs.d.git)
#### 安装TTF/OTF字体
Linux下面安装TTF字体已经在最近几年的版本中变得非常容易，双击打开然后点击安装即可。但是之前我们都是将字体拷贝到字体目录中，然后更新字体缓存实现的。
现在 WSL 没有安装 GUI 界面，又该如何安装呢，其实沿用之前的好办法即可，将字体文件拷贝到字体目录更新字体缓存即可。
以下三个目录都是字体目录
```
/usr/share/fonts
/usr/local/share/fonts
/home/arch/.fonts
```
下载、解压并拷贝字体到 .fonts/Noto 目录
下载 SF Mono 字体到 .fonts 目录
```bash
mkdir Downloads
wget https://noto-website-2.storage.googleapis.com/pkgs/Noto-hinted.zip
unzip Noto-hinted.zip
mkdir -p ~/.fonts/Noto
cp *otf *otc ~/.fonts/Noto

git clone https://github.com/Eason0210/SF_Mono.git ~/.fonts

sudo fc-cache -fv .fonts
```
#### 安装 Aspell
```bash
sudo pacman -S aspell aspell-en
```
#### 安装 Emacs-rime
安装 [librime](https://github.com/rime/librime)
```bash
sudo pacman -S fcitx5-rime librime
```
第一次在 Emacs 中启动 rime 的时候会自动编译 [Emacs-rime](https://github.com/DogLooksGood/emacs-rime), 并在 ~/.emacs.d/rime 目录生成相应的配置文件。
建议先拷贝自己的配置到该目录，再启动 rime 。

#### 安装 ripgrep
```bash
sudo pacman -S ripgrep
```
#### 安装 yay
```
sudo pacman -S yay
```
在 WSL2 下 yay 安装软件包的速度非常慢，有待解决。

#### 安装 multimarkdown
该命令用于 Markdown mode 下 `C-c, C-c p` 在浏览器中预览当前 buffer 。
```bash
yay -S multimarkdown
```
#### 安装 google chrome
```bash
sudo pacman -S google-chrome
```
