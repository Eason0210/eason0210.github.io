---
layout: post
title: Emacs宏在批量化生成代码中的应用
date: 2020-05-16T20:36:16+08:00
tags: [Emacs]
categories: [Develop Tools]
---
需求：

比如需要批量输入以下的Verilog代码，可能有成百上千行，如何快速输入？
```
case(write_ptr)
                         5'd0 : memory[0] <= data_w;
                         5'd1 : memory[1] <= data_w;
                         5'd2 : memory[2] <= data_w;
                         5'd3 : memory[3] <= data_w;
                         5'd4 : memory[4] <= data_w;
                         5'd5 : memory[5] <= data_w;
                         5'd6 : memory[6] <= data_w;
```

这个的主要需求是用在case语句中给一片连续内存单元赋值，需要能够做到生成从0~n的格式化代码，n同时最好能有各种进制转换及用0补齐到k位（主要是2进制和16进制转换，同时保证位对齐）

希望的操作是输入第一个行：
```
 5'b00000 : memory[0] <= data_w;
```
再经过一系列操作得到结果：
```
 5'b00000 : memory[0] <= data_w;
 5'b00001 : memory[1] <= data_w;
 5'b00010 : memory[2] <= data_w;
 5'b00011 : memory[3] <= data_w;
 5'b00100 : memory[4] <= data_w;
 5'b00101 : memory[5] <= data_w;
 5'b00110 : memory[6] <= data_w;
 ......
 5'b01000 : memory[16] <= data_w;

```
### 通过在Emacs中录制宏实现
1. C-x, ( 开始宏的录制
2. 输入第一行的内容
```
5'd0 : memory[0] <= data_w;
```
3. 复制第一行的内容到第二行（在我的配置中按C-. l即可复制，到第二行按C-y粘贴）
4. 光标移动到第一个0后面，按M-+ 即可将0增加1，同样方法操作第二个0
5. 操作完成将光标移动到第三行
6. C-x, ) 结束宏的录制
7. 要执行刚录制好的宏，按C-x, e
8. 要执行n次，则按 C-u, n C-x, e
9. 需要给刚刚记录的宏记录编辑一个名字 M+x name-last-kbd-marco
10. 把刚刚起名字的宏记录写入到文件里面 M+x insert-kbd-marco
11. 先建立一个记录宏记录的文件，比如可以建立~/.emacs.d/my_macro.el文件并把宏记录写入到里面；
12. 在init.el中添加 (load-file "~/.emacs.d/my_macro.el")就能加载了，再用(global-set-key (kbd "...." ) '....)就能用绑定快捷键到一个相应名字的宏操作了
13. 要重用这段宏，还可以用snippet，yasnippet支持动态执行elisp

备注：这种方法也可以用来给一段文字的每个行首或者行尾添加序号。在vim中也有类似操作。

### 通过直接使用(eval-expression)的方式实现
M-x (eval-expression)，然后输入
```elisp
(dotimes (i 17)
  (insert (format "5'd%d : memory[%d] <= data_w;\n" i i)))
```
这种方式最灵活。如果嫌minibuffer里输入比较麻烦，实际上也可以直接在对应的源文件里直接写elisp代码，然后C-x C-e (eval-last-sexp) 就行了，它会直接输入在当前的buffer里。

### Emacs宏的另一种玩法
由于Emacs自带的键盘宏可以使用计数器，可以通过下面的方式录制宏：

1. 按 F3, 输入 5'd [counter=0]
2. 按 F3 插入 0 [counter=1]
3. 继续输入到 memory[, 按 C-u C-x C-k C-c [counter=0]
4. 按 F3 插入 0 [counter=1]
5. 输入完整行，回车，按 F4

备注：［counter=0］这里只是说明当前的计数器，操作中不用输入。

如果Emacs配置中增加了复制数字的功能则更加方便，还可以通过下面的方式录制宏：

1. 按 F3，输入 5'd
2. 按 F3，会插入一个 0。把这个数字复制一下（我的配置中使用的是M-+增加，M--减少）
3. 继续输入到 memory[, 粘贴刚复制的数字
4. 输入完整行，回车，按 F4
5. 一直按 F4

### 总结
Emacs 的宏功能非常强大，可以干很多事情，这里的使用方法只是Emacs 宏的冰山一角。
另外，这个的Verilog代码只是用来举例，不一定需要用宏来解决，其实还有更好的方法。但由于我本人并不写Verilog，所以就不进行更深一步的研究了。感兴趣的可以关注verilog-mode ，其中有生成索引号的func。Verilog还有[generate block](https://stackoverflow.com/questions/19875899/how-to-define-a-parameterized-multiplexer-using-systemverilog)的功能。
