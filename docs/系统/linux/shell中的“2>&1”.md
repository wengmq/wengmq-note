# 如何理解 Linux shell 中的“2>&1”

## **前言**

有时候我们常看到类似这样的脚本调用：

```text
./test.sh  > log.txt 2>&1
```

这里的 2>&1 是什么意思？该如何理解？
先说结论：上面的调用表明将./test.sh 的输出重定向到 log.txt 文件中，同时将标准错误也重定向到 log.txt 文件中。

## **有何妙用**

（如果已经明白是什么作用，可跳过此小节）
上面到底是什么意思呢？我们来看下面的例子，假如有脚本 test.sh：

```text
#!/bin/bash
date         #打印当前时间
while true   #死循环
do
    #每隔2秒打印一次
    sleep 2
    whatthis    #不存在的命令
    echo -e "std output"
done
```

脚本中先打印当前日期，然后每隔 2 秒执行 whatthis 并打印一段字符。由于系统中不存在 whatthis 命令，因此执行会报错。
假如我们想保存该脚本的打印结果，只需将 test.sh 的结果重定向到 log.txt 中即可：

```text
./test.sh > log.txt
```

执行结果如下：

```text
ubuntu$ ./test.sh >log.txt
./test.sh: 行 7: whatthis: 未找到命令
```

我们明明将打印内容重定向到 log.txt 中了，但是这条错误信息却没有重定向到 log.txt 中。如果你是使用程序调用该脚本，当查看脚本日志的时候，将会**完全看不到这条错误信息**。而使用下面的方式则会将出错信息也重定向到 log.txt 中：

```text
./test.sh  > log.txt 2>&1
```

以这样的方式调用脚本，可以很好的将错误信息保存，**帮助我们定位问题**。

## **如何理解**

每个程序在运行后，都会至少打开三个[文件描述符](https://www.zhihu.com/search?q=文件描述符&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A47765176})，分别是 0：标准输入；1：标准输出；2：标准错误。
例如，对于前面的 test.sh 脚本，我们通过下面的步骤看到它至少打开了三个文件描述符：

```text
./test.sh    #运行脚本
ps -ef|grep test.sh  #重新打开命令串口，使用ps命令找到test.sh的pid
hyb       5270  4514  0 19:20 pts/7    00:00:00 /bin/bash ./test.sh
hyb       5315  5282  0 19:20 pts/11   00:00:00 grep --color=auto test.sh
```

可以看到 test.sh 的 pid 为 5270，进入到相关 fd 目录：

```text
cd /proc/5270/fd   #进程5270所有打开的文件描述符信息都在此
ls -l              #列出目录下的内容
 0 -> /dev/pts/7
 1 -> /dev/pts/7
 2 -> /dev/pts/7
 255 -> /home/hyb/workspaces/shell/test.sh
```

可以看到，test.sh 打开了 0，1，2 三个文件描述符。同样的，如果有兴趣，也可以查看其他运行进程的文件描述符打开情况，除非关闭了否则都会有这三个文件描述符。

那么现在就容易理解前面的疑问了，2>&1 表明将文件描述 2（标准错误输出）的内容重定向到文件描述符 1（标准输出），为什么 1 前面需要&？当没有&时，1 会被认为是一个普通的文件，有&表示重定向的目标不是一个文件，而是一个文件描述符。在前面我们知道，test.sh >log.txt 又将文件描述符 1 的内容重定向到了文件 log.txt，那么最终标准错误也会重定向到 log.txt。我们同样通过前面的方法，可以看到 test.sh 进程的文件描述符情况如下：

```text
 0 -> /dev/pts/7
 1 -> /home/hyb/workspaces/shell/log.txt
 2 -> /home/hyb/workspaces/shell/log.txt
 255 -> /home/hyb/workspaces/shell/test.sh
```

我们可以很明显地看到，**文件描述符 1 和 2 都指向了 log.txt 文件，**也就得到了我们最终想要的效果：**将标准错误输出重定向到文件中**。
它们还有两种等价写法：

```text
./test.sh  >& log.txt
./test.sh  &> log.txt
```

## **总结**

我们总结一下前面的内容：

- 程序运行后会打开三个文件描述符，分别是标准输入，标准输出和标准错误输出。
- 在调用脚本时，可使用 2>&1 来将标准错误输出重定向。
- 只需要查看脚本的错误时，可将标准输出重定向到文件，而标准错误会打印在控制台，便于查看。
- \>>log.txt 会将重定向内容追加到 log.txt 文件末尾。
- 通过查看/proc/[进程 id](https://www.zhihu.com/search?q=进程id&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A47765176})/fd 下的内容，可了解进程打开的文件描述符信息。

## **思考**

Q:下面的调用会将标准错误输出重定向到文件中吗？为什么？

```text
./test.sh 2>&1 >log.txt
```

A:

何 2>&1 要写在后面？
command > file 2>&1
首先是 command > file 将标准输出重定向到 file 中， 2>&1 是标准错误拷贝了标准输出的行为，也就是同样被重定向到 file 中，最终结果就是标准输出和错误都被重定向到 file 中。
command 2>&1 >file
2>&1 标准错误拷贝了标准输出的行为，但此时标准输出还是在终端。>file 后输出才被重定向到 file，但标准错误仍然保持在终端。

用 strace 可以看到：

1. command > file 2>&1
   这个命令中实现重定向的关键系统调用序列是：
   open(file) == 3
   dup2(3,1)
   dup2(1,2)

2. command 2>&1 >file
   这个命令中实现重定向的关键系统调用序列是：
   dup2(1,2)
   open(file) == 3
   dup2(3,1)

可以考虑一下不同的 dup2()调用序列会产生怎样的文件共享结构。请参考 APUE 3.10, 3.12

## 参考：

- https://zhuanlan.zhihu.com/p/47765176
