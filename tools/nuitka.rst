
###########################
nuitka
###########################


nuitka是一个将Python程序转换为C++的工具。可以用于Python加密。

:github: https://github.com/Nuitka/Nuitka

安装
==================

::

    pip install nuitka

使用方法
=======================

假设有一个文件 ``main.py`` ::

    import sys

    def divide(a, b):
        return a/b
    
    one = int(sys.argv[1])
    two = int(sys.argv[2])
    ret = divide(one, two)
    print(ret)

**加密一个Python程序**

    ``python -m nuitka --follow-imports main.py``

    这种形式，会转换 main.py 以及其依赖的库，比如上面就引用了 ``sys`` 。

    运行完成后，会在当前目录下生成两个文件:
    
        1. main.build: 转换、编译生成的临时文件。
        2. main.exe： 转换后的程序。

    此时 main.exe 仍然要依赖 Python 环境才能执行。
    如果想要编译一个不依赖于任何外部程序的独立可执行程序。
    可以使用 ``--standalone`` 选项


**编译为完整的可执行程序**

    添加 ``--standalone`` 选项

    ``python -m nuitka --standalone --follow-imports main.py``

    运行完成后，会在当前目录下生成两个文件夹:
    
        1. main.build: 转换、编译生成的临时文件。
        2. main.dist： 编译完成的C++程序以及其所有依赖.

**在windows平台使用**

    使用方法和Linux一样，但是第一次使用 ``--standalone`` 选项时，会提示需要下载一个dependencywalker。

    http://dependencywalker.com/depends22_x64.zip

    输入 ``Y`` , 会自动联网下载，
    如果不能联网，可以手动下载这个文件放到 ``C:\Users\viruser.v-desktop\AppData\Local\Nuitka\Nuitka\x86_64`` 。

    windows平台一般使用 visual studio, 或者 mingw/MSYS(移植到windows平台的一个linux环境)。
    一般来说，Nuitka会自动搜索编译工具并选择一个，如果同时存在 visual studio 和 mingw，
    也可以使用 ``--mingw64`` 强制使用mingw编译器。

    ``python -m nuitka --standalone --mingw64 main.py``


**将Python实现的库编译，供其他python程序调用**

    使用 ``--module`` 选项

    ::

        python -m nuitka --module myfunc.py

    会生成以下文件:

        * myfunc.build: 生成的临时文件，没什么用
        * myfunc.pyd、myfunc.pyi： 生成的python包，其中myfunc.pyi可以删除，
          里面可能包含myfunc的函数声明、源码，用于辅助调试。

    myfunc.pyd 本质上是一个动态链接库，也是一个python包，可以直接在其他python脚本中 ``import myfunc`` 来使用。


原理
================

Nuitka的原理和 ``cython`` 一样， https://cython.org/

因为Python主流实现是用C编写的CPython，可以很方便的和C、C++交互。
比如Python直接调用C，或者在C里面直接写一串Python代码传给Python解释器的执行接口去执行。

Python本身也暴露了一系列C接口供C程序调用。

    https://docs.python.org/3/c-api/index.html

所以，可以直接将Python代码翻译为相应的C接口调用，这种转换是等价的。

Python代码案例::

    t = (1, 2, "three")

转换为C接口调用::

    PyObject *t;

    t = PyTuple_New(3);
    PyTuple_SetItem(t, 0, PyLong_FromLong(1L));
    PyTuple_SetItem(t, 1, PyLong_FromLong(2L));
    PyTuple_SetItem(t, 2, PyUnicode_FromString("three"));

Nuitka的原理就是使用 Python AST 解析Python语法树，并将其翻译为这类C接口，达到转换为C程序的目的。
这种方式兼容性很好，因为本质上还是使用Python方式运行，内部已经看不到任何Python代码，可以实现加密。
但是性能上并不会有显著提升。而且加密后的程序无法调试，因为没有代码，也就丢失了堆栈的大部分信息。

关于性能: 一般来说，性能会有一些提升，大约30%，而且Nuitka在转换为C代码的时候，据说会进行一些优化。

