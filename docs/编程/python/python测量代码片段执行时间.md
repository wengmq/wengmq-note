# python 测量代码片段执行时间：[`python timeit`](https://docs.python.org/zh-cn/3/library/timeit.html#module-timeit)

该模块提供了一种简单的方法来计算一小段 Python 代码的耗时。它有 [命令行界面](https://docs.python.org/zh-cn/3/library/timeit.html#timeit-command-line-interface) 以及一个 [可调用](https://docs.python.org/zh-cn/3/library/timeit.html#python-interface) 方法。它避免了许多用于测量执行时间的常见陷阱。另见 Tim Peters 对 O'Reilly 出版的 _Python Cookbook_ 中“算法”章节的介绍。

## 基本示例

以下示例显示了如何使用 [命令行界面](https://docs.python.org/zh-cn/3/library/timeit.html#timeit-command-line-interface) 来比较三个不同的表达式：

```
$ python3 -m timeit '"-".join(str(n) for n in range(100))'
10000 loops, best of 5: 30.2 usec per loop
$ python3 -m timeit '"-".join([str(n) for n in range(100)])'
10000 loops, best of 5: 27.5 usec per loop
$ python3 -m timeit '"-".join(map(str, range(100)))'
10000 loops, best of 5: 23.2 usec per loop
```

这可以通过 [Python 接口](https://docs.python.org/zh-cn/3/library/timeit.html#python-interface) 实现

\>>>

```
>>> import timeit
>>> timeit.timeit('"-".join(str(n) for n in range(100))', number=10000)
0.3018611848820001
>>> timeit.timeit('"-".join([str(n) for n in range(100)])', number=10000)
0.2727368790656328
>>> timeit.timeit('"-".join(map(str, range(100)))', number=10000)
0.23702679807320237
```

从 [Python 接口](https://docs.python.org/zh-cn/3/library/timeit.html#python-interface) 还可以传出一个可调用对象:

\>>>

```
>>> timeit.timeit(lambda: "-".join(map(str, range(100))), number=10000)
0.19665591977536678
```

但请注意 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.timeit) 仅在使用命令行界面时会自动确定重复次数。 在 [示例](https://docs.python.org/zh-cn/3/library/timeit.html#timeit-examples) 一节你可以找到更多的进阶示例。

## Python 接口

该模块定义了三个便利函数和一个公共类：

- `timeit.`timeit`(_stmt='pass'_, _setup='pass'_, _timer=<default timer>_, _number=1000000_, _globals=None_)

  使用给定语句、 _setup_ 代码和 _timer_ 函数创建一个 [`Timer`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer) 实例，并执行 _number_ 次其 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 方法。可选的 _globals_ 参数指定用于执行代码的命名空间。_在 3.5 版更改:_ 添加可选参数 _globals_ 。

- ` timeit.``repeat `(_stmt='pass'_, _setup='pass'_, _timer=<default timer>_, _repeat=5_, _number=1000000_, _globals=None_)

  使用给定语句、 _setup_ 代码和 _timer_ 函数创建一个 [`Timer`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer) 实例，并使用给定的 _repeat_ 计数和 _number_ 执行运行其 [`repeat()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.repeat) 方法。可选的 _globals_ 参数指定用于执行代码的命名空间。_在 3.5 版更改:_ 添加可选参数 _globals_ 。_在 3.7 版更改:_ _repeat_ 的默认值由 3 更改为 5 。

- ` timeit.``default_timer `()

  默认的计时器，总是 [`time.perf_counter()`](https://docs.python.org/zh-cn/3/library/time.html#time.perf_counter) 。_在 3.3 版更改:_ [`time.perf_counter()`](https://docs.python.org/zh-cn/3/library/time.html#time.perf_counter) 现在是默认计时器。

- _class_ ` timeit.``Timer `(_stmt='pass'_, _setup='pass'_, _timer=<timer function>_, _globals=None_)

  用于小代码片段的计数执行速度的类。构造函数接受一个将计时的语句、一个用于设置的附加语句和一个定时器函数。两个语句都默认为 `'pass'` ；计时器函数与平台有关（请参阅模块文档字符串）。 _stmt_ 和 _setup_ 也可能包含多个以 `;` 或换行符分隔的语句，只要它们不包含多行字符串文字即可。该语句默认在 timeit 的命名空间内执行；可以通过将命名空间传递给 _globals_ 来控制此行为。要测量第一个语句的执行时间，请使用 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 方法。 [`repeat()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.repeat) 和 [`autorange()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.autorange) 方法是方便的方法来调用 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 多次。_setup_ 的执行时间从总体计时执行中排除。_stmt_ 和 _setup_ 参数也可以使用不带参数的可调用对象。这将在一个计时器函数中嵌入对它们的调用，然后由 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 执行。请注意，由于额外的函数调用，在这种情况下，计时开销会略大一些。_在 3.5 版更改:_ 添加可选参数 _globals_ 。`timeit`(_number=1000000_)执行 _number_ 次主要语句。这将执行一次 setup 语句，然后返回执行主语句多次所需的时间，以秒为单位测量为浮点数。参数是通过循环的次数，默认为一百万。要使用的主语句、 setup 语句和 timer 函数将传递给构造函数。注解 默认情况下， [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 暂时关闭 [garbage collection](https://docs.python.org/zh-cn/3/glossary.html#term-garbage-collection) 。这种方法的优点在于它使独立时序更具可比性。缺点是 GC 可能是所测量功能性能的重要组成部分。如果是这样，可以在 _setup_ 字符串中的第一个语句重新启用 GC。例如:` timeit.Timer('for i in range(10): oct(i)', 'gc.enable()').timeit() ``autorange `(_callback=None_)自动决定调用多少次 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 。这是一个便利函数，它反复调用 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) ，以便总时间 >= 0.2 秒，返回最终（循环次数，循环所用的时间）。它调用 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 的次数以序列 1, 2, 5, 10, 20, 50, ... 递增，直到所用的时间至少为 0.2 秒。如果给出 _callback_ 并且不是 `None` ，则在每次试验后将使用两个参数调用它： `callback(number, time_taken)` 。_3.6 新版功能._`repeat`(_repeat=5_, _number=1000000_)调用 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 几次。这是一个方便的函数，它反复调用 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) ，返回结果列表。第一个参数指定调用 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 的次数。第二个参数指定 [`timeit()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer.timeit) 的 _number_ 参数。注解 从结果向量计算并报告平均值和标准差这些是很诱人的。但是，这不是很有用。在典型情况下，最低值给出了机器运行给定代码段的速度的下限；结果向量中较高的值通常不是由 Python 的速度变化引起的，而是由于其他过程干扰你的计时准确性。所以结果的 [`min()`](https://docs.python.org/zh-cn/3/library/functions.html#min) 可能是你应该感兴趣的唯一数字。之后，你应该看看整个向量并应用常识而不是统计。_在 3.7 版更改:_ _repeat_ 的默认值由 3 更改为 5 。`print_exc`(_file=None_)帮助程序从计时代码中打印回溯。典型使用:`t = Timer(...) # outside the try/except try: t.timeit(...) # or t.repeat(...) except Exception: t.print_exc() `与标准回溯相比，优势在于将显示已编译模板中的源行。可选的 _file_ 参数指向发送回溯的位置；它默认为 [`sys.stderr`](https://docs.python.org/zh-cn/3/library/sys.html#sys.stderr) 。

## 命令行界面

从命令行调用程序时，使用以下表单:

```
python -m timeit [-n N] [-r N] [-u U] [-s S] [-h] [statement ...]
```

如果了解以下选项：

- ` -n`` N``, ``--number``=N `

  执行 '语句' 多少次

- ` -r`` N``, ``--repeat``=N `

  重复计时器的次数（默认为 5）

- ` -s`` S``, ``--setup``=S `

  最初要执行一次的语句（默认为 `pass` ）

- ` -p`` , ``--process  `

  测量进程时间，而不是 wallclock 时间，使用 [`time.process_time()`](https://docs.python.org/zh-cn/3/library/time.html#time.process_time) 而不是 [`time.perf_counter()`](https://docs.python.org/zh-cn/3/library/time.html#time.perf_counter) ，这是默认值*3.3 新版功能.*

- ` -u````, ``--unit``=U `

  指定定时器输出的时间单位；可以选择 nsec，usec，msec 或 sec*3.5 新版功能.*

- ` -v`` , ``--verbose  `

  打印原始计时结果；重复更多位数精度

- ` -h`` , ``--help  `

  打印一条简短的使用信息并退出

可以通过将每一行指定为单独的语句参数来给出多行语句；通过在引号中包含参数并使用前导空格可以缩进行。多个 [`-s`](https://docs.python.org/zh-cn/3/library/timeit.html#cmdoption-timeit-s) 选项的处理方式相似。

如果未给出 [`-n`](https://docs.python.org/zh-cn/3/library/timeit.html#cmdoption-timeit-n)，则会通过尝试按序列 1, 2, 5, 10, 20, 50, ... 递增的数值来计算合适的循环次数，直到总计时间至少为 0.2 秒。

[`default_timer()`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.default_timer) 测量可能受到在同一台机器上运行的其他程序的影响，因此在需要精确计时时最好的做法是重复几次计时并使用最佳时间。 [`-r`](https://docs.python.org/zh-cn/3/library/timeit.html#cmdoption-timeit-r) 选项对此有利；在大多数情况下，默认的 5 次重复可能就足够了。 你可以使用 [`time.process_time()`](https://docs.python.org/zh-cn/3/library/time.html#time.process_time) 来测量 CPU 时间。

注解

执行 pass 语句会产生一定的基线开销。这里的代码不会试图隐藏它，但你应该知道它。可以通过不带参数调用程序来测量基线开销，并且 Python 版本之间可能会有所不同。

## 示例

可以提供一个在开头只执行一次的 setup 语句：

```
$ python -m timeit -s 'text = "sample string"; char = "g"'  'char in text'
5000000 loops, best of 5: 0.0877 usec per loop
$ python -m timeit -s 'text = "sample string"; char = "g"'  'text.find(char)'
1000000 loops, best of 5: 0.342 usec per loop
```

\>>>

```
>>> import timeit
>>> timeit.timeit('char in text', setup='text = "sample string"; char = "g"')
0.41440500499993504
>>> timeit.timeit('text.find(char)', setup='text = "sample string"; char = "g"')
1.7246671520006203
```

使用 [`Timer`](https://docs.python.org/zh-cn/3/library/timeit.html#timeit.Timer) 类及其方法可以完成同样的操作:

\>>>

```
>>> import timeit
>>> t = timeit.Timer('char in text', setup='text = "sample string"; char = "g"')
>>> t.timeit()
0.3955516149999312
>>> t.repeat()
[0.40183617287970225, 0.37027556854118704, 0.38344867356679524, 0.3712595970846668, 0.37866875250654886]
```

以下示例显示如何计算包含多行的表达式。 在这里我们对比使用 [`hasattr()`](https://docs.python.org/zh-cn/3/library/functions.html#hasattr) 与 [`try`](https://docs.python.org/zh-cn/3/reference/compound_stmts.html#try)/[`except`](https://docs.python.org/zh-cn/3/reference/compound_stmts.html#except) 的开销来测试缺失与提供对象属性:

```
$ python -m timeit 'try:' '  str.__bool__' 'except AttributeError:' '  pass'
20000 loops, best of 5: 15.7 usec per loop
$ python -m timeit 'if hasattr(str, "__bool__"): pass'
50000 loops, best of 5: 4.26 usec per loop

$ python -m timeit 'try:' '  int.__bool__' 'except AttributeError:' '  pass'
200000 loops, best of 5: 1.43 usec per loop
$ python -m timeit 'if hasattr(int, "__bool__"): pass'
100000 loops, best of 5: 2.23 usec per loop
```

\>>>

```
>>> import timeit
>>> # attribute is missing
>>> s = """\
... try:
...     str.__bool__
... except AttributeError:
...     pass
... """
>>> timeit.timeit(stmt=s, number=100000)
0.9138244460009446
>>> s = "if hasattr(str, '__bool__'): pass"
>>> timeit.timeit(stmt=s, number=100000)
0.5829014980008651
>>>
>>> # attribute is present
>>> s = """\
... try:
...     int.__bool__
... except AttributeError:
...     pass
... """
>>> timeit.timeit(stmt=s, number=100000)
0.04215312199994514
>>> s = "if hasattr(int, '__bool__'): pass"
>>> timeit.timeit(stmt=s, number=100000)
0.08588060699912603
```

要让 [`timeit`](https://docs.python.org/zh-cn/3/library/timeit.html#module-timeit) 模块访问你定义的函数，你可以传递一个包含 import 语句的 _setup_ 参数:

```
def test():
    """Stupid test function"""
    L = [i for i in range(100)]

if __name__ == '__main__':
    import timeit
    print(timeit.timeit("test()", setup="from __main__ import test"))
```

另一种选择是将 [`globals()`](https://docs.python.org/zh-cn/3/library/functions.html#globals) 传递给 _globals_ 参数，这将导致代码在当前的全局命名空间中执行。这比单独指定 import 更方便

```
def f(x):
    return x**2
def g(x):
    return x**4
def h(x):
    return x**8

import timeit
print(timeit.timeit('[func(42) for func in (f,g,h)]', globals=globals()))
```

## ref：

- 参考：https://docs.python.org/zh-cn/3/library/timeit.html

- 源码： [Lib/timeit.py](https://github.com/python/cpython/tree/3.9/Lib/timeit.py)
