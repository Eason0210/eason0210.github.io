---
layout: post
title: GccEmacs 在各个平台上的安装
date: 2021-02-23T20:36:16+08:00
tags: [Emacs, GccEmacs, Native-comp]
categories: [Develop Tools]
---
 Emacs 28.1 最近已经发布，内置了对 `--with-native-compilation`的支持。只要安装 28.1 以上版本就能体验 native-compilation 特性。
 这里主要介绍在不同平台如何安装 Emacs 29.0.60，以体验更多的新功能。

### Apple Mac OS
这里以 Intel 平台的 macOS (Ventura 13.1) 为例：
#### 通过 [macports](https://www.macports.org/install.php) 安装
```bash
sudo port install emacs-app-devel
```
#### 手动编译 Emacs 29
参考官方的[安装手册](https://git.savannah.gnu.org/cgit/emacs.git/tree/nextstep/INSTALL?h=emacs-29&id=1d3cbba7dfa26fc74df4d09d40a3cd7ba07279b4)：
```bash
./autogen.sh
./configure --with-native-compilation=aot
make -j8 && make install
```
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
详细说明请参考 Emacs 官方仓库的[安装文档](https://git.savannah.gnu.org/cgit/emacs.git/tree/nt/INSTALL.W64)

在 Mingw64 终端中执行以下命令安装依赖。

```bash
  pacman -S --needed base-devel \
  mingw-w64-x86_64-toolchain \
  mingw-w64-x86_64-xpm-nox \
  mingw-w64-x86_64-gmp \
  mingw-w64-x86_64-gnutls \
  mingw-w64-x86_64-libtiff \
  mingw-w64-x86_64-giflib \
  mingw-w64-x86_64-libpng \
  mingw-w64-x86_64-libjpeg-turbo \
  mingw-w64-x86_64-librsvg \
  mingw-w64-x86_64-libwebp \
  mingw-w64-x86_64-lcms2 \
  mingw-w64-x86_64-jansson \
  mingw-w64-x86_64-libxml2 \
  mingw-w64-x86_64-zlib \
  mingw-w64-x86_64-harfbuzz \
  mingw-w64-x86_64-libgccjit \
  mingw-w64-x86_64-sqlite3 \
  mingw-w64-x86_64-tree-sitter
```

这些软件包包括了基本的开发者工具（autoconf，grep，make，等等），编译器工具链（GCC，GDB，等等），几个图像库，XML库，GnuTLS（安全传输层协议）库，zlib 用于解压缩文本，HarfBuzz 用作文字塑形布局引擎(text shaping engine)，HarfBuzz 主要是将 Unicode 转换为格式正确且位置正确的字形输出， libgccjit 用于支持 native-compilation， SQLite3 用于访问 SQL 数据库，以及一些主要模式（major modes）要使用的 tree-sitter。 只有前四个软件包（base-devel, toolchain, xpm-nox, GMP）和GnuTLS是强烈推荐的；其余的是可选的。 如果你不需要全部的功能，则可以只安装一部分需要的库。

安装好上面这些软件包后，您现在就有了一个完整的 Emacs 构建环境。

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
有几种方式可以解决：
1. 设置 `git config core.autocrlf false`
2. 通过 Web 浏览器访问 [emacs-mirror/emacs](https://github.com/emacs-mirror/emacs.git) ，点击绿色的`Clone`按钮下载Zip格式的源代码。
3. 通过 wget 下载源码, `<commit id>` 在官方仓库获得
```bat
wget https://git.savannah.gnu.org/cgit/emacs.git/snapshot/emacs-<commit id>.tar.gz
tar -xzvf .\emacs-<commit id>.tar.gz

```
#### 编译 Emacs
进入源代码目录，然后通过以下命令编译 Emacs 的源码。

```bash
./autogen.sh
./configure --with-native-compilation=aot --without-dbus

echo $(nproc)

make -j6 && make install prefix=/c/opt/emacs
```
注意：

1. `echo $(nproc)` 会显示当前系统的 CPU 核心数；然后在`make -j6` 就是使用 6 个核心进行编译, 推荐使用总数的一半，既提高了编译速度，也不影响其他应用的运行。
2. `--with-native-compilation=aot` 相当于 `make NATIVE_FULL_AOT=1`， 会把内置包的 `.elc` 前编译成 `.eln`, 这样启动Emacs时就不需要等待编译，如果不希望提前编译，去掉“=aot”就好了。
3. `make install` 的时候如果不指定 `prefix` 的话是会直接安装到 msys2 目录下，不讲究的话可以这样用。如果需要卸载的话在源码目录里面 `make uninstall`就可以了。个人建议安装到指定目录, 比如我这里是安装到 `c:\opt\emacs`, 注意在路径中使用斜杠"/", 而不是反斜杠"\\"。
4. 如果编译过程出错了，记得`make clean`之后重新`configure`再`make`。


#### 运行 Emacs
通过 `c:\opt\emacs\bin\runemacs.exe` 就可以启动 Emacs，也可以运行 `c:\opt\emacs\bin\addpm.exe` 在开始菜单中添加一个快捷方式。

当从 `mingw64 shell` 之外运行 Emacs 时，你需要将 `c:\msys64\mingw64\bin` 添加到您的 Windows PATH 中，或复制所需的 DLL 到 Emacs 的 bin/ 目录中。 否则依赖于这些 DLL 的特性(比如 TLS)将无法使用。
下面的命令可以自动拷贝，但不一定全，仅供参考。个人是使用加入 PATH 的方式。
```bash
cp $( pacman -Ql mingw-w64-x86_64-{libtiff,giflib,libpng,libjpeg-turbo,librsvg,libxml2,gnutls} | grep bin/.*\.dll$ | awk '{print $2}' ) /c/opt/emacs/bin
```
### 使用体验
目前 native-compilation 的 Emacs 已经很成熟了，已经集成到 Emacs 28.1 以上的版本中。但要注意，native-compilation 并不会加快 Emacs 的启动速度。它主要是对使用 LSP 时有性能提升。

至于为什么选择使用 Emacs 29 主要是为了使用以下新特性：

1. 支持像素滚动，可以通过触摸板像 Google Chrome 浏览器那样滚动屏幕。
2. 内置支持 `sqlite`，这样 `org-roam` 和 `epkg` 这样的包就不需要通过动态模块，可以直接使用内置的 `sqlite`。
3. 在 Windows 平台支持双缓冲，滚动屏幕时，不再会出现闪烁。
4. 修复了 `project.el` 在处理 `Git` 子模块的bug，现在可以在使用 `Borg` 时，在 `.gitsubmodule` 文件中加 `load-path`。
5. 修复了在使用 `destop.el` 和 `saveplace` 时不支持非 UTF-8 的 unicode 编码(比如 UTF-16)。

附上我的个人 Emacs 配置文件：[Eason0210/.emacs.d](https://github.com/Eason0210/.emacs.d)
