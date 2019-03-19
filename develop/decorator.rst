#####################################
Python 装饰器
#####################################

要了解装饰器，需要先理解 python 的普通函数，以及将函数作为值传递的知识。

普通函数
========================

编码过程中经常遇到这种情况，

1. 实现了一个函数

::

    def divide(one, two):
        return one/two

2. 发现这个实现并不完美, 比如 ``divide(9, 0)`` 会抛异常, 所以打个补丁。

::

    def catch_except_divide(one, two):
        try:
            return divide(one, two)
        except Exception:
            return None

现实生活中，上面两个函数是一个比较典型的两个部分：功能实现、异常处理。

实际上，这段错误处理的代码并不是特有的，很多其他函数也可以用得到。


函数返回函数
===========================

Python 中，函数也是一个对象。可以在一个函数中返回另外一个函数, 甚至可以将函数当做一个参数。

::

    def multi(one, two):
        return one*two

    def divide(one, two):
        return one/two

    def calc(op):
        ops = {
            "*": multi,
            "/": divide,
        }
        return ops[op]

    # 先获取函数然后调用它。
    func = calc("*")
    func(3, 4) # out: 12

通过这种特性，可以改造上面的除法代码.

::

    def divide(one, two):
        return one/two
    
    def catch_except(func, *args, **kwargs):
        try:
            return func(*args, **kwargs)
        except:
            return None
    
    catch_except(divide, 4, 2) # out: 2
    catch_except(divide, 4, 0) # out: None

通过这种方法，可以实现将异常处理模块剥离。

只不过，这块代码比较丑陋。下面是另一种高级一点的封装方法，称为函数的科里化。

好处在于，不必每次调用都写上 ``catch_except`` .

::

    def catch_except(func):
        def _inner_func(*args, **kwargs):
            try:
                return func(*args, **kwargs)
            except:
                return None
        return _inner_func

    func = catch_except(divide)
    func(4, 2) # out: 2
    func(4, 0) # out: None


装饰器
=============================

实际上上面的最后一种实现方式就是装饰器。

对照上面的catch_except感受一下他的定义: 装饰器本身是一个函数，它接受另外一个函数，将其修改后返回包装后的函数，替换原函数。

装饰器是一种语法糖，借助于装饰器，我们没有必要像上面一样每次调用divide之前先使用 catch_except 包装一下 ``func = catch_except(divide)`` ,
而是改成下面的写法。使用 ``@`` 。 顺便加一些打印看看执行流程


::

    def catch_except(func):
        def _inner_func(*args, **kwargs):
            try:
                print("enter _inner_func")
                ret = func(*args, **kwargs)
                print("leave _inner_func")
                return ret
            except:
                return None
        return _inner_func

    @catch_except
    def divide(one, two):
        print("enter divide")
        return one/two

    # 现在可以直接调用divide，等价于使用 catch_except 封装了一下。
    divide(4, 2) 
    # enter _inner_func
    # enter divide
    # leave _inner_func
    # out: 2
    divide(4, 0) 
    # 打印省略 ...
    # out: None

这样做的好处也显而易见，完全不会更改 divide 函数的定义， 而且以前调用divide的地方也不用修改任何代码。

装饰器函数
=================================

上面的装饰器已经有了基本的样子。这还不够。上面捕获异常的装饰器，在执行失败后会自动返回None，
如果被装饰的函数不想返回None，这是没法控制的。

这时候可以在装饰器上面再加一层。首先执行那个函数，生成一个装饰器。

::

    def catch_except(default=None):
        def _real_catch_except(func):
            def _inner_func(*args, **kwargs):
                try:
                    return func(*args, **kwargs)
                except:
                    return default
            return _inner_func
        return _real_catch_except

    @catch_except(default=0)
    def divide(one, two):
        return one/two


装饰成员函数
=============================

类的成员函数和普通函数有点不同。普通函数一般永远都是普通函数，而类的方法有两种不同的状态

1. 定义一个class的时候，class上的方法仅仅是普通函数。(状态一)
2. 使用一个class实例化生成对象后，会将class的方法绑定到生成的对象上。 （状态二）

::

    class MyCalc:
        def __init__(self, one, two):
            self.one = one
            self.two = two
        
        def divide(self):
            return self.one/self.two

    # 实例化一个mc对象后，会将divide转化为状态二并绑定到mc。
    # 状态二与状态一的区别在于，状态二的时候调用不需要提供第一个self参数，
    mc = MyCalc(4, 2)
    mc.divide()

    # 以状态一的身份调用 MyCalc.divide ，需要显式提供第一个self参数
    MyCalc.divide(mc)

装饰器发生作用的时候是在状态一。

换句话说，类的成员函数可以当做普通函数传入装饰器。两者并没有区别, 
只不过需要稍微注意，self也是算作参数的。

::

    class MyCalc:
        def __init__(self, one, two):
            self.one = one
            self.two = two
        
        @catch_except(default=0)
        def divide(self):
            return self.one/self.two

消除装饰器的副作用
===========================

假设有下面的函数.（这个例子的代码几乎是从上面抄过来的）

::

    def catch_except(default=None):
        def _real_catch_except(func):
            def _inner_func(*args, **kwargs):
                try:
                    return func(*args, **kwargs)
                except:
                    return default
            return _inner_func
        return _real_catch_except

    @catch_except(default=0)
    def divide(one, two):
        """this is a divider"""
        return one/two

一切看起来没什么问题。因为我们写了docstring注释，可以尝试在命令行尝试查看函数帮助，以及其他元信息

::

    In [4]: divide.__name__
    Out[4]: '_inner_func'

    In [6]: help(divide)
    Help on function _inner_func in module __main__:

    _inner_func(*args, **kwargs)

并没有如期显示帮助信息，函数名称也变了！！！ 这是因为装饰器本质上是修改了原来的函数，返回的已经是另外一个函数了。

所以， Python 提供了 functools 。使用 functools.wraps 可以获取到原函数的这些元信息，然后拷贝到新生成的函数上，
进而消除这些副作用。  functools.wraps 本身也是一个装饰器。

::

    import functools

    def catch_except(default=None):
        def _real_catch_except(func):
            @functools.wraps(func)  # <--- 注意这一行
            def _inner_func(*args, **kwargs):
                try:
                    return func(*args, **kwargs)
                except:
                    return default
            return _inner_func
        return _real_catch_except

    @catch_except(default=0)
    def divide(one, two):
        """this is a divider"""
        return one/two

获取元信息

::

    In [10]: divide.__name__
    Out[10]: 'divide'

    In [11]: help(divide)
    Help on function divide in module __main__:

    divide(one, two)
        this is a divider
