# shell 中各种括号的作用及用法

# 一、单小括号()

单小括号常见的两个作用：命令替换以及数组的初始化

#### 1.1 命令替换

当碰到$()结构时，shell 就把括号的内命令执行，并返回结果

```
$ tmp=$(date)
$ echo $tmp
Fri Sep 18 10:22:30 CST 2020
```

#### 1.2 初始化数组

初始化数组

```
$ array=(linux nginx mysql php)

$ for i in $array; do echo $i; done
linux
nginx
mysql
php
```

# 二、双小括号(())

#### 2.1 数值运算

记住双括号有一个特点，就是括号内的要符合 c 语言的语法，使用变量名时不需要加上$符号的。

双小括号的用法比较多，它可以用作整数计算（不支持小数)。例如：$((3+2))

```
$ echo $((1+3))
4

$ n1=3
$ echo $((n1+7))
10
```

```
$ a=1
$ b=2
$ echo $((a+b))
3
```

只要符合 c 语言语法的运算扩展，都可以写在括号里。效果啥上和 expr 指令类似

```
$ echo $((3+4>5 ? 1 :0))
1

$ echo $((3+4>9 ? 1 :0))
0
```

重新给变量赋值，这个时候不可在括号外加$符

```
$ i=1;((i++));echo $i;
2

$ i=1;((i=100));echo $i;
100
```

双括号还经常用在 for 循环中

```
$ for ((i=0;i<5;i++));do echo $i;done
0
1
2
3
4
```

# 三、单方括号[]

#### 3.1 表示判断条件

- 1）bash 的内部命令，`[]` 和`test`是等同的。if/test 结构中的左中括号是调用 test 的命令标识，右中括号是关闭条件判断的。这个命令把它的参数作为比较表达式或者作为文件测试，并且根据比较的结果来返回一个退出状态码。
- 2）在进行比较运算时使用。test 和[]中可用的比较运算符只有==和!=，两者都是用于字符串比较的，不可用于整数比较；整数比较只能使用-eq，-gt 这种形式。无论是字符串比较还是整数比较都不支持大于号小于号。如果实在想用，对于字符串比较可以使用转义形式，如果比较”ab”和”bc”：[ ab \< bc ]，结果为真，也就是返回状态为 0。[ ]中的逻辑与和逻辑或使用-a 和-o 表示。

```
$ if [ 2 -eq 1 ];then echo "true"; else; echo "false"; fi
false

$ if [ 2 -eq 2 ];then echo "true"; else; echo "false"; fi
true
```

#### 3.2 表示数组下标

注意数组的下标是从 1 开始的，如果 echo $array[0]则不会有输出

```
$ array=(linux nginx mysql php)

$ echo $array[1]
linux
```

#### 3.3 表示 shell 下通配符

| `[list]`             | 匹配 list 中的任意单一字符                  | a[xyz]b a 与 b 之间必须也只能有一个字符, 但只能是 x 或 y 或 z, 如: axb, ayb, azb。 |
| -------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------- |
| `[!list]或[^list]`   | 匹配 除 list 中的任意单一字符               | a[!0-9]b a 与 b 之间必须也只能有一个字符, 但不能是阿拉伯数字, 如 axb, aab, a-b。   |
| `[c1-c2]`            | 匹配 c1-c2 中的任意单一字符 如：[0-9] [a-z] | a[0-9]b 0 与 9 之间必须也只能有一个字符 如 a0b, a1b... a9b。                       |
| `[!c1-c2]或[^c1-c2]` | 匹配不在 c1-c2 的任意字符                   | a[!0-9]b 如 acb adb                                                                |

```
$ ls
test0.txt test1.txt test2.txt test3.txt test4.txt test5.txt test6.txt test7.txt test8.txt test9.txt

$ ls test[0-3].txt
test0.txt test1.txt test2.txt test3.txt
```

# 四、双方括号[[]]

- 1）[[是 bash 程序语言的关键字。并不是一个命令，[[]] 结构比[ ]结构更加通用。在[[和]]之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
- 2）支持字符串的模式匹配，使用=~操作符时甚至支持 shell 的正则表达式。字符串比较时可以把右边的作为一个模式，而不仅仅是一个字符串，比如[[hello == hell?]]，结果为真。[[]] 中匹配字符串或通配符，不需要引号。
- 3）使用[[…]]条件判断结构，而不是[ … ]，能够防止脚本中的许多逻辑错误。比如，&&、||、<和> 操作符能够正常存在于[[]]条件判断结构中，但是如果出现在[ ]结构中的话，会报错。
- 4） bash 把双中括号中的表达式看作一个单独的元素，并返回一个退出状态码。

```
$ if [[ 2 > 1 ]];then echo "true"; else; echo "false"; fi
true

$ if [[ 2 > 3 ]];then echo "true"; else; echo "false"; fi
false

$ if [[ aaa == bbb ]];then echo "true"; else; echo "false"; fi
false

$ if [[ aaa == aaa ]];then echo "true"; else; echo "false"; fi
true

$ a="aaa"
$ if [[ $a == aaa ]];then echo "true"; else; echo "false"; fi
true

$ if [[ $a == aaa && 2 > 1 ]];then echo "true"; else; echo "false"; fi
true

$ if [[ $a == aaa && 2 < 1 ]];then echo "true"; else; echo "false"; fi
false
```

# 五、花括号{}

#### 5.1 划定变量名的边

当执行 echo ”$aa“的时候系统会打印变量$aa 的值，当执行 echo "${a}a"时打印的是${a}和字母 a,所以使用中括号{ }来划定变量名的边界。如果不需要为变量名划分边界的话，$a和${a}是完全相等的。

```
$ a="my"
$ echo $a
my

$ echo ${a}self
myself
```

#### 5.2.用大括号进行拓展

此时可以使用大括号对文件进行批量操作

1）第一种：对大括号中的以逗号分割的文件列表进行拓展。如：touch {file1,file2,file3}.sh。

```
$ touch {file1,file2,file3}.sh

$ ls
file1.sh file2.sh file3.sh
```

2）第二种：对大括号中以点点（..）分割的顺序文件列表起拓展作用。如：touch {1..10}.sh

```
$ touch {1..10}.sh

$ ls
1.sh  10.sh 2.sh  3.sh  4.sh  5.sh  6.sh  7.sh  8.sh  9.sh
```

#### 5.3 文本处理

##### 【1】获取字符串长度 `${#a}`

```
[root@linuxforliuhj ~]# a=hello
[root@linuxforliuhj ~]# echo ${a}
hello
[root@linuxforliuhj ~]# echo ${#a}
```

##### 【2】字符串切片`${a:b:c}`

将字符串变量 a 从第 b 个位置开始向后截取 c 个字符，b 是指下标，下标从 0 开始

```
[root@linuxforliuhj ~]# a='hello linux!'
[root@linuxforliuhj ~]# echo ${a:0:5}
hello
[root@linuxforliuhj ~]# echo ${a:6}
linux!
[root@linuxforliuhj ~]# echo ${a:6:5}
linux
[root@linuxforliuhj ~]# echo ${a:(-1)}
!
[root@linuxforliuhj ~]# echo ${a:(-5)}
inux!
#截取从倒数第 5 个字符后的 3 个字符
[root@linuxforliuhj ~]# echo ${a:(-5):3}
inu
```

##### 【3】替换字符串`${a/b/c}`

将变量 a 中的 b 全部替换为 c，开头一个正斜杠为只匹配第一个字符串，两个正斜杠为匹配所有字符。

```
[root@linuxforliuhj ~]# a='hello linux linux'

#将a中的第一个linux替换为java
[root@linuxforliuhj ~]# echo "${a/linux/java}"
hello java linux

#将a中全部的linux替换为java，使用双斜杠
[root@linuxforliuhj ~]# echo "${a//linux/java}"
hello java java

#替换正则匹配为空
[root@linuxforliuhj ~]# VAR=123abc
[root@linuxforliuhj ~]# echo ${VAR//[^0-9]/}
123
[root@linuxforliuhj ~]# echo ${VAR//[0-9]/}
abc
```

##### 【4】字符串截取

格式：
${parameter#word} # 删除匹配前缀
${parameter##word}
${parameter%word} # 删除匹配后缀
${parameter%%word}

#去掉左边，#最短匹配模式，##最长匹配模式。
% 去掉右边，%最短匹配模式，%%最长匹配模式。

```
[root@linuxforliuhj ~]# URL="http://www.baidu.com/baike/user.html"

#以//为分隔符截取右边字符串
[root@linuxforliuhj ~]# echo ${URL#*//}
www.baidu.com/baike/user.html

#以/为分隔符截取右边字符串，##表示尽可能多的删除，保留最少内容
[root@linuxforliuhj ~]# echo ${URL##*/}
user.html
[root@linuxforliuhj ~]# echo ${URL#*/}
/www.baidu.com/baike/user.html

#以//为分隔符截取左边字符串
[root@linuxforliuhj ~]# echo ${URL%%//*}
http:

#以/为分隔符截取左边字符串,%%表示尽可能多的删除，即保留最少内容
[root@linuxforliuhj ~]# echo ${URL%/*}
http://www.baidu.com/baike
[root@linuxforliuhj ~]# echo ${URL%%/*}
http:
```

##### 【5】变量状态赋值

${VAR:-string} 如果 VAR 变量为空则返回 string
${VAR:+string} 如果 VAR 变量不为空则返回 string
${VAR:=string} 如果 VAR 变量为空则重新赋值 VAR 变量值为 string
${VAR:?string} 如果 VAR 变量为空则将 string 输出到 stderr

```
#如果变量为空就返回 hello world!： # VAR=
echo ${VAR:-'hello world!'}
hello world!
#如果变量不为空就返回 hello world!： # VAR="hello"
echo ${VAR:+'hello world!'}
hello world!
#如果变量为空就重新赋值：
VAR=
echo ${VAR:=hello}
hello
echo $VAR
hello
#如果变量为空就将信息输出 stderr： # VAR=
echo ${VAR:?value is null}
-bash: VAR: value is null
```

## 参考

- https://www.php.cn/linux-460140.html
- https://www.pianshen.com/article/1396464079/
- https://blog.csdn.net/swjtuwyp/article/details/51817472?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1.pc_relevant_default&utm_relevant_index=2
- https://blog.csdn.net/qq_43382735/article/details/121606145
