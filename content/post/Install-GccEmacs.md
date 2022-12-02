---
layout: post
title: GccEmacs 在各个平台上的安装
date: 2021-02-23T20:36:16+08:00
tags: [Emacs, GccEmacs, Native-comp]
categories: [Develop Tools]
---
 Emacs 28.1 最近已经发布，内置了对 `--with-native-compilation`的支持。只要安装 28.1 就能体验 native-comp 特性。
 这里主要介绍在不同平台如何安装 Emacs 29.0.50，以体验更多的新功能。

### Apple Mac OS
这里以 Intel 平台的 Mac OS (Monterey 12.4) 为例：

#### 通过脚本自动编译：
这种方法的好处是比较灵活，可以根据需要编译不同的 commit 或者 tag，而且所有需要的内容都打包到 Emacs.app 中，包括源代码。
```bash
git clone https://github.com/jimeh/build-emacs-for-macos
cd build-emacs-for-macos
brew bundle
brew services start dbus

./build-emacs-for-macos --native-full-aot
cd builds
tar -xjvf Emacs.2022-xx-xx.xxxxxxx.master.macOS-12.x86_64.tbz
```
拷贝 Emacs.app 到 `/Applications/Emacs.app`

软链接 `emacs` 到 `/usr/local/bin`，方便有时需要终端启动。
```bash
ln -s /Applications/Emacs.app/Contents/MacOS/bin/emacs /usr/local/bin/emacs
```
#### 直接使用 [Emacs plus](https://github.com/d12frosted/homebrew-emacs-plus) 安装：

``` bash
brew install emacs-plus@29 --with-native-comp
```
### Gnu/Linux
Linux 发行版本较多，不同发行版的安装方式都不相同，这里以 `ArchLinux` 为例说明：
``` bash
sudo pacman -S emacs-native-comp-git
```

### Windows 10
Windows 系统可以通过[GNU官网下载](https://alpha.gnu.org/gnu/emacs/pretest/windows/emacs-29/)预先编译的版本。
我更倾向于只能通过 [Msys2](https://www.msys2.org/) 进行编译安装，这样能用上最新的 Commit。
提醒：建议在环境变量中设置 `HOME` 变量为 `C:\Users\<username>\`, 这里的 `<username>` 填写你的系统用户名。

安装好 `Msys2` 后， 进入 msys2 根目录下的`/etc/pacman.d/`里面，将3个 mirrorlist 文件中 TUNA 和 USTC 的源的顺序提到前面，这样下载速度就非常快了。
然后进入 mingw64 终端，安装好 Msys2 后进入 Mingw64 终端，先执行 `pacman -Syu` 更新系统到最新。

#### 安装依赖
详细说明请参考 Emacs 官方仓库的[安装文档](https://github.com/emacs-mirror/emacs/blob/master/nt/INSTALL.W64)

在 Mingw64 终端中执行以下命令安装依赖。

```bash
  pacman -S --needed base-devel gcc git \
  mingw-w64-x86_64-toolchain \
  mingw-w64-x86_64-xpm-nox \
  mingw-w64-x86_64-libtiff \
  mingw-w64-x86_64-giflib \
  mingw-w64-x86_64-libpng \
  mingw-w64-x86_64-libjpeg-turbo \
  mingw-w64-x86_64-librsvg \
  mingw-w64-x86_64-libwebp \
  mingw-w64-x86_64-lcms2 \
  mingw-w64-x86_64-jansson \
  mingw-w64-x86_64-libxml2 \
  mingw-w64-x86_64-gnutls \
  mingw-w64-x86_64-zlib \
  mingw-w64-x86_64-harfbuzz
```
#### 下载 Emacs 源代码
Github 镜像:
```bash
git clone --depth=1 https://github.com/emacs-mirror/emacs.git
```
官方 Git 仓库:
```bash
git clone --depth=1  https://git.savannah.gnu.org/git/emacs.git
```
提醒：如果执行`./autogen.sh`过程中出错，可能是因为克隆代码后 `checkout` 时用了 `CRLF`.  
有2种方式可以解决：
1. 设置 `git config core.autocrlf false`
2. 通过 Web 浏览器访问 [emacs-mirror/emacs](https://github.com/emacs-mirror/emacs.git) ，点击绿色的`Clone`按钮下载Zip格式的源代码。

#### 编译 Emacs
进入源代码目录，然后通过以下命令编译 Emacs 的源码。

```bash
   ./autogen.sh
   ./configure --with-native-compilation=aot

   make -j$(nproc)
   
   make install prefix=/c/opt/emacs
   cp $( pacman -Ql mingw-w64-x86_64-{libtiff,giflib,libpng,libjpeg-turbo,librsvg,libxml2,gnutls} | grep bin/.*\.dll$ | awk '{print $2}' ) /c/opt/emacs/bin

```
注意：

1. `make -j$(nproc)` 中的 `$(nproc)` 会自动获取当前系统的 CPU 核心数；你也可以自己手动输入，比如`make -j12` 就是使用 12 核心进行编译。
2. `--with-native-compilation=aot` 相当于 `make NATIVE_FULL_AOT=1`， 会强制把所有 `.el` 文件提前编译成 `.eln`, 但编译时间会大幅增加。
3. `make install` 的时候如果不指定 `prefix` 的话是会直接安装到 msys2 目录下，不讲究的话可以这样用。如果需要卸载的话在源码目录里面 `make uninstall`就可以了。个人建议安装到指定目录, 比如我这里是安装到 `c:\opt\emacs`, 注意在路径中使用斜杠"/", 而不是反斜杠"\\"。
3. 如果编译过程出错了，记得`make clean`之后重新`configure`再`make`。

### 使用体验
目前 native-compilation 的 Emacs 已经很成熟了，已经集成到 Emacs 28.1 版本中。但要注意，native-compilation 并不会加快 Emacs 的启动速度。它主要是对使用 LSP 时有性能提升。

至于为什么选择使用 Emacs 29 主要是为了使用以下新特性：

1. 支持像素滚动，可以通过触摸板像 Chrome 浏览器那样滚动屏幕。
2. 内置支持 `sqlite`，这样 `org-roam` 和 `epkg` 这样的包就不需要通过动态模块，可以直接使用内置的 `sqlite`。
3. 在 Windows 平台支持双缓冲，滚动屏幕时，不再会出现闪烁。
4. 修复了 `project.el` 在处理 `Git` 子模块的bug，现在可以在使用 `Borg` 时，在 `.gitsubmodule` 文件中加 `load-path`。
5. 修复了在使用 `destop.el` 和 `saveplace` 时不支持非 UTF-8 的 unicode 编码(比如 UTF-16)。

附上我的个人 Emacs 配置文件：[Eason0210/.emacs.d](https://github.com/Eason0210/.emacs.d)
