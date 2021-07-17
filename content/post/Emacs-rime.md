---
layout: post
title: Windows 10 安装emacs-rime
date: 2020-04-25T20:36:16+08:00
tags: [Emacs]
categories: [Tools]
---
 emacs-rime是RIME输入法的Emacs UI前端，所有行为都通过RIME配置文件来配置。但是Emacs-rime在Windwos系统下比较难安装，最近有人更新了Windows下编译emacs-rime的脚本，可以成功在Windows上使用。

### 安装RIME输入法
通过[RIME官网](https://rime.im/)安装，Windows下名字叫小狼豪。

### 配置RIME输入法

#### 默认使用简体

创建配置文件 %appdata%/Rime/luna_pinyin.custom.yaml，内容填上：

```yaml
# luna_pinyin.custom.yaml

patch:
  switches:                   # 注意缩进
    - name: ascii_mode
      reset: 0                # reset 0 的作用是当从其他输入法切换到本输入法重设为指定状态
      states: [ 中文, 西文 ]   # 选择输入方案后通常需要立即输入中文，故重设 ascii_mode = 0
    - name: full_shape
      states: [ 半角, 全角 ]   # 而全／半角则可沿用之前方案的用法。
    - name: simplification
      reset: 1                # 增加这一行：默认启用「繁→簡」转换。
      states: [ 漢字, 汉字 ]
```

#### 修改默认后选词数量

创建配置文件 %appdata%/Rime/default.custom.yaml，内容填上：

```yaml
patch:
  "menu/page_size": 6
```

#### 配置模糊音

对于拼音发音不标准的人，可以在 %appdata%/Rime/default.custom.yaml 配置文件中继续追加下面配置(我本人不用)

```yaml
  'speller/algebra':
    - erase/^xx$/                      # 第一行保留

    # 模糊音定義
    - derive/^([zcs])h/$1/             # zh, ch, sh => z, c, s
    - derive/^([zcs])([^h])/$1h$2/     # z, c, s => zh, ch, sh

    - derive/^n/l/                     # n => l
    - derive/^l/n/                     # l => n

    # 這兩組一般是單向的
    #- derive/^r/l/                     # r => l

    - derive/^ren/yin/                 # ren => yin, reng => ying
    #- derive/^r/y/                     # r => y

    # 下面 hu <=> f 這組寫法複雜一些，分情況討論
    #- derive/^hu$/fu/                  # hu => fu
    #- derive/^hong$/feng/              # hong => feng
    #- derive/^hu([in])$/fe$1/          # hui => fei, hun => fen
    #- derive/^hu([ao])/f$1/            # hua => fa, ...

    #- derive/^fu$/hu/                  # fu => hu
    #- derive/^feng$/hong/              # feng => hong
    #- derive/^fe([in])$/hu$1/          # fei => hui, fen => hun
    #- derive/^f([ao])/hu$1/            # fa => hua, ...

    # 模糊音定義先於簡拼定義，方可令簡拼支持以上模糊音
    - abbrev/^([a-z]).+$/$1/           # 簡拼（首字母）
    - abbrev/^([zcs]h).+$/$1/          # 簡拼（zh, ch, sh）

    # 自動糾正一些常見的按鍵錯誤
    - derive/([aeiou])ng$/$1gn/        # dagn => dang
    - derive/([dtngkhrzcs])o(u|ng)$/$1o/  # zho => zhong|zhou
    - derive/ong$/on/                  # zhonguo => zhong guo
    - derive/ao$/oa/                   # hoa => hao
    - derive/([iu])a(o|ng?)$/a$1$2/    # tain => tian
```

#### 添加搜狗词库

网上搜索文件 luna_pinyin.sogou.dict.yaml，放到目录  %appdata%/Rime 下
然后在  %appdata%/Rime/default.custom.yaml 文件中添加下面配置：

```yaml
translator/dictionary: luna_pinyin.sogou
```

重新部署即可体验词库。

### 安装依赖

因为[posframe](https://github.com/tumashu/posframe)可以让后选词显示在光标处，所以建议安装

### 编译librime
这步可以偷懒，直接用编译好的[liberime](https://github.com/merrickluo/liberime/releases/download/v0.0.3/liberime-v0.0.3-windows-x86_64.zip)。
解压后，只要将下图的几个文件拷贝到emacs安装目录/bin就可以了。
![emacs-rime]({{site.url}}/pics/emacs-rime/liberime-win1.png)

如果安装目录已经有的dll，用系统自带的也行，那么只要拷贝librime.dll即可。

同时将解压的文件夹中的rime-data文件留着后续配置emacs时作为`rime-share-data-dir`
### 编译 librime-emacs.dll
```bash
git clone https://github.com/DogLooksGood/emacs-rime
```
安装好gcc，直接用cmd shell cd 到 emac-rime直接gcc： 先将 librime.dll rime_levers_api.h rime_api.h （librime就是上述解压出来的，其他两个在librime的src下面，见下图）复制到emacs-rime项目下，执行以下命令：
```bash
gcc lib.c -o librime-emacs.dll -O2 -shared -I. -I/path/to/emacs/include -L. -llibrime
```
比如的我的路径是：`D:\emacs-28.0.50\include` 则执行以下命令：
```bash
gcc lib.c -o librime-emacs.dll -O2 -shared -I. -ID:\emacs-28.0.50\include -L. -llibrime
```
编译如果报错，一般是某个.h文件未找到，加上-I路径即可；librime.dll找不到，在-L后面加上路径即可。

![emacs-rime]({{site.url}}/pics/emacs-rime/librime-win1.png)
### 安装emacs-rime


把 emacs-rime 目录放到 ```load-path``` 下，增加下面配置:



``` elisp
;;; Require
(require 'rime)

;;; Code:
(when (eq system-type 'windows-nt)
  (setq rime-user-data-dir "~/.emacs.d/emacs-rime/Rime")
  (setq rime-share-data-dir "~/.emacs.d/emacs-rime/data")
  (setq rime-posframe-properties
        (list :background-color "#333333"
              :foreground-color "#dcdccc"
              :font "华文楷体"
              :internal-border-width 10)))

(setq default-input-method "rime"
  rime-show-candidate 'posframe)
```
上面的配置分别设置emacs-rime读取RIME配置的路径、UI细节和使用posframe来显示候选词。

配置上主要是要设置两个路径：
`rime-user-data-dir` 如果不设置，默认是`~/.emacs.d/rime`, 这里直接拷贝小狼毫的用户配置文件即可。一般是在 %appdata%下的Rime文件夹。
`rime-share-data-dir`设置为上述第一步解压出来的rime-data文件夹，我只保留了opencc这个文件夹。
### emacs-rime的优点

1. 只是RIME的前端，代码量比较小，有问题还可以提交个补丁；
2. 中英文混合输入的体验很好，英文输入完成后，按回车或者空格就可以继续输入中文；
3. UI默认配色不错，看着很现代；
4. 与Emacs集成，主题直接适配Emacs主题，而且配合翻译功能，体验更好。
