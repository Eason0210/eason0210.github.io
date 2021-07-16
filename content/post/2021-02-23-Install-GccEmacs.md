---
layout: post
title: GccEmacs 在各个平台上的安装
date: 2021-02-23T20:36:16+08:00
tags: [Emacs, GccEmacs, Native-comp]
categories: [Develop Tools]
---
`GccEmacs` 最近已经逐渐成熟，几个月前已经加入了 Emacs 仓库的 `feature/nativecomp` 分支，未来还将合并到 `Master` 分支。但由于 Emacs 的历史包袱太重，维护者对加入新功能是保持比较谨慎的态度，也许是人手不足，对 `GccEmacs` 加入 `Master` 分支推进的很慢，以至于 `GccEmacs` 的维护者都有“[意见](https://lists.gnu.org/archive/html/emacs-devel/2021-02/msg00878.html)”了。

进入正题，下面针对三大平台分别介绍 `GccEmacs` 的安装：

### Apple Mac OS
这里以 Intel 平台的 Mac OS (Big Sur 11.2) 为例：
Mac 平台比较推荐直接使用 [Emacs plus](https://github.com/d12frosted/homebrew-emacs-plus), 通过以下命令安装：
``` bash
brew install emacs-plus@28 --with-native-comp
```
注意：第一次启动 Emacs 时，建议在终端中执行 `emacs -q` 启动纯净版本的 Emacs, 并执行一些命令，使得 Emacs 开始编译内置的包。完成后再正常启动 Emacs 。

### Gnu/Linux
Linux 发行版本较多，不同发行版的安装方式都不相同，这里以 `ArchLinux` 为例说明：
``` bash
sudo pacman -S emacs-native-comp-git
```

### Windows 10
Windows 系统目前还没有预先编译的版本，只能安装 [Msys2](https://www.msys2.org/) 进行编译安装。
提醒：建议在环境变量中设置 `HOME` 变量为 `C:\Users\Aqua\`, 这里的 `Aqua` 是我的 Windows 系统用户名。
#### 安装 Msys2
推荐使用 [Scoop](https://scoop.sh/) 进行软件的安装，更加方便，而且不会污染 `Path`.
如果安装了 Scoop, 可以执行下面的命令安装 Msys2:

```bash
scoop install msys2
```
进入 msys2 根目录下的`/etc/pacman.d/`里面，将3个 mirrorlist 文件中 TUNA 和 USTC 的源的顺序提到前面，这样下载速度就非常快了。
然后进入 mingw64 终端，安装好 Msys2 后进入 Mingw64 终端，先执行 `pacman -Syu` 更新系统到最新。

#### 安装依赖
在 Mingw64 终端中执行以下命令安装依赖。

```bash
  pacman -S --needed base-devel gcc git\
  mingw-w64-x86_64-toolchain \
  mingw-w64-x86_64-xpm-nox \
  mingw-w64-x86_64-libtiff \
  mingw-w64-x86_64-giflib \
  mingw-w64-x86_64-libpng \
  mingw-w64-x86_64-libjpeg-turbo \
  mingw-w64-x86_64-librsvg \
  mingw-w64-x86_64-lcms2 \
  mingw-w64-x86_64-jansson \
  mingw-w64-x86_64-libxml2 \
  mingw-w64-x86_64-gnutls \
  mingw-w64-x86_64-zlib \
  mingw-w64-x86_64-harfbuzz
```
#### 下载 Emacs 源代码
github mirror:
```bash
git clone --depth=1 https://github.com/emacs-mirror/emacs.git
```
gitee mirror:
```bash
git clone --depth=1 https://gitee.com/mirrors/emacs.git
```
official git repo:
```bash
git clone --depth=1  https://git.savannah.gnu.org/git/emacs.git
```
#### 编译 Emacs
进入源代码目录，然后通过以下命令编译 Emacs 的源码。

```bash
   ./autogen.sh
   ./configure --with-native-compilation --without-dbus
   make -j$(nproc)

   make install prefix=/d/Dev_Tools/emacs28-native
   cp $( pacman -Ql mingw-w64-x86_64-{libtiff,giflib,libpng,libjpeg-turbo,librsvg,libxml2,gnutls} | grep bin/.*\.dll$ | awk '{print $2}' ) /D/Dev_Tools/emacs28-native/bin

```
注意：

1. `make -j$(nproc)` 中的 `$(nproc)` 会自动获取当前系统的 CPU 核心数；你也可以自己手动输入，比如`make -j12` 就是使用 12 核心进行编译。
2. `make NATIVE_FULL_AOT=1` 会强制把所有 el 文件提前编译成 eln, 但编译时间会大幅增加。
3. `make install` 的时候如果不指定 `prefix` 的话是会直接安装到 msys2 目录下，不讲究的话可以这样用。如果需要卸载的话在源码目录里面 `make uninstall`就可以了。个人建议安装到指定目录, 比如我这里是安装到 `D:\Dev_Tools\emacs28-native`, 注意在路径中使用斜杠"/", 而不是反斜杠"\\"。
3. 如果编译过程出错了，记得`make clean`之后重新`configure`再`make` 。
4. 启动 Emacs 时会自动编译 eln, 但会出现下面这个错误，不过不影响使用，原因未知。  

```bash
Warning: arch-dependent data dir '%emacs_dir%/libexec/emacs/28.0.50/x86_64-w64-mingw32/': No such file or directory

```

### 使用体验
目前在 Mac 和 Linux 上的使用体验是最好的，Windows 也完全可以正常使用了，不过速度的提升不是特别的明显。
期待 `Gccemacs` 早日加入 `Master` 分支。
