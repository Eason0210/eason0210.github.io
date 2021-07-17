---
layout: post
title: 在Emacs中安装SLIME+SBCL+Quicklisp
date: 2020-05-25T20:36:16+08:00
tags: [Emacs, Slime, SBCL, Quicklisp]
categories: [Programming]
---
[SLIME](https://common-lisp.net/project/slime/) – SLIME是一个用于Common Lisp开发的Emacs mode。这个项目是受到Emacs Lisp和ILISP启发而开发，主要用于Common Lisp的Hacking工具。除了Slime，还有一个Hacking工具是SLY。

[SBCL](http://www.sbcl.org/) – Steel Bank Common Lisp (SBCL)是一款高性能的 Common Lisp 编译器。除了 ANSI Common Lisp 的编译器和运行时系统之外，它还提供了一个交互式环境，包括调试器、统计分析器、代码覆盖工具和许多其他扩展。

[Quicklisp](https://www.quicklisp.org/beta/) – Quicklisp 是Common Lisp的安装包依赖管理程序，可以用来代替asdf。可以配合已有的Common Lisp实现一起使用，通过几条简单的命令就可以对超过1200多个库进行下载，安装和加载。

以下对如何在Emacs中集成SLIME, SBCL和Quicklisp进行简要介绍。这是目前比较推荐的用于学习Common Lisp的开发环境。
### 安装Slime
可以通过MELPA或者Git安装Slime。参考[Slime的Github主页](https://github.com/slime/slime)。

1. 通过MELPA安装 `M-x package-install RET slime RET`
2. 在`~/.emacs`或者`~/.eamcs.d/init.el`中添加`inferior-lisp-program` 设置使用的Common Lisp实现：
``` emacs-lisp
(setq inferior-lisp-program "sbcl")
```
3. 使用`M-x slime` 启动并连接到一个inferior Lisp。SLIME将会自动能够在当前的Lisp源码缓冲区中使用。

如果从Git仓库安装：
1. 在终端中输入以下命令
```bash
cd path/where/you/want/slime/installed
git clone https://github.com/slime/slime.git
```
2. 在`~/.emacs`或者`~/.emacs.d/init.el`中添加以下代码
```emacs-lisp
;; Setup load-path, autoloads and your lisp system
;; Not needed if you install SLIME via MELPA
(add-to-list 'load-path "~/dir/to/cloned/slime")
(require 'slime-autoloads)
(setq inferior-lisp-program "/opt/sbcl/bin/sbcl")
```
### 安装SBCL
从SBCL的官方网站下载安装进行安装。可以根据自己使用的平台选择相应的安装包。

Mac OS也可以通过Brew安装：
``` bash
brew install sbcl
```
### 安装Quicklisp
下载[Quicklisp安装文件](https://beta.quicklisp.org/quicklisp.lisp)。
将下载的文件放置于Home目录下，打开终端输入cd ` 进入Home目录，并执行以下命令。

```lisp
sbcl --load quicklisp.lisp
```

sbcl启动后，输入以下命令：
```lisp
(quicklisp-quickstart:install)
```
等待Quicklisp安装完毕后，如果希望每次启动Lisp的时候自动加载Quicklisp（推荐），那么就执行以下命令：
```lisp
(ql:add-to-init-file)
```
然后，输入以下命令，将会创建一个`quicklisp-slime-helper.el`文件，通过在Emacs加载该文件，可以设置SLIME加载Quicklisp的正确load-path。
```lisp
(ql:quickload "quicklisp-slime-helper")
```
最后，将会看到以下信息：
```emacs-lisp
To use, add this to your ~/.emacs:

(load (expand-file-name "~/quicklisp/slime-helper.el"))
;; Replace "sbcl" with the path to your implementation
(setq inferior-lisp-program "sbcl")
```

我的个人配置：
```emacs-lisp
(add-hook 'lisp-mode-hook (lambda ()
                            (unless (featurep 'slime)
                              (load (expand-file-name "~/quicklisp/slime-helper.el"))
                              (require 'slime-autoloads)
                              (normal-mode))))

(setq inferior-lisp-program "sbcl")
```
### Enjoy Common Lisp

安装已经完成。现在可以在Emacs中执行` M-x slime`启动Slime.
