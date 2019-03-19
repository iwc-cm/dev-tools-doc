#######################################
Python 协程
#######################################

一些前提和基础概念
========================

yield
------------------

一般函数使用return返回一些对象，而 yield 可以用来返回多个对象。
yield 的行为很像range。

.. code-block:: python

    def myrange(number):
        i = 0
        while i < number:
            yield i
            i += 1

    for n in myrange(10):
        print(n)

    # out: 1 2 3

函数执行到yield后，并不会终结，而是保存上下文，将这个值返回，下一次可以接着执行。
这种行为一般都通过for循环来触发，当然也可以通过next手动触发。

.. code-block:: python

    data_iter = myrange(3)
    n = next(data_iter) # out: 0
    n = next(data_iter) # out: 1
    n = next(data_iter) # out: 2
    n = next(data_iter) # out: 异常 raise StopIteration

yield 设计的初衷是赋予函数可以多次返回对象的能力，并且大大降低内存消耗，因为每次都临时计算出下一个值，
而不需要将所有值都计算出来。借助这种方法可以很轻松地实现无限迭代器。

.. code-block:: python

    def infinite_range(number):
        i = 0
        while True:
            yield i
            i += 1

yield from
------------------

yield from 是 yield 的语法糖。用来实现多层迭代器的串联。

举个例子。 有一组单词，需要遍历其所有字符。

.. code-block:: python

    def iter_word(word):
        for char in word:
            yield char
    
    def iter_list(words):
        for word in words:
            # 注意下面调用了iter_word
            for char in iter_word(word):
                yield char

    for c in iter_list(["hello", "world"]):
        print(c)
    # out: h e l l o w o r l d

上面的iter_list调用了另外一个迭代器用于迭代单个word，将其返回值直接返回。 
引入了 yield from 之后，就可以节省一行代码了，直接将迭代器串起来。

.. code-block:: python

    def iter_list(words):
        for word in words:
            # 注意下面串联调用了iter_word
            yield from iter_word(word)

通过yield from 语法糖，可以轻松实现多层 yield 迭代器的串联。

Future/Promise
------------------------

Future是Python异步编程中的概念，对应到javascript中就是Promise。

Future用来包装一个非常耗时的（异步）操作，内部维护了一个状态，调用者可以回过头来检测这个状态从而判断是否执行完成。

    * pending: 执行中
    * cancelled: 取消执行
    * finished: 执行完成

Future创建初始状态是 pending，执行结束后状态会转换到 cancelled或者finished，这种状态转换不可逆，并且只有这一次改变状态的机会。

状态改变后，调用者可以获取调用状态和返回值。

**假想** 的 Future 实现::

    class Future:
        def __init__(func):
            self._func = func
            self._status = "pending"
            t = threading.Thread(target=self._executor)
            t.start()
            t.join()

        def _executor(self):
            self._func()
            self._status = "finished"
        
        def done():
            return self._status == "finished"

    fut = Future(lambda : time.sleep(5))
    while not fut.done():
        time.sleep(0.1)

大致思路是这样，和真实Python实现的 asyncio.Future 还是有很大不同。

协程
====================

原始的yield协程
-----------------------

回过头来看下面的代码，也就是之前说的 yield，当我们手动使用 next 触发yield执行的时候，
其实相当于我们自己控制了迭代器（data_iter）的暂停和执行。

.. yield 在上面说过，可以多次返回对象，其实 yield 还可以接收对象。

.. code-block:: python

    def main():
        data_iter = myrange(3)
        n = next(data_iter) # out: 0
        n = next(data_iter) # out: 1
        n = next(data_iter) # out: 2
        n = next(data_iter) # out: 异常 raise StopIteration

但是，Python的yield设计初衷并不是用来产生协程，这种用法仅仅是意外之喜。而Python社区也很愿意将yield包装成协程，
最后导致的结果是，Python的协程方案比较乱。。。。。。

乱在哪里就不说了。首先要区分 yield 和 协程。yield设计用于在函数中多次返回对象，
而协程是类似于线程的一种动态上下文切换技术，开销更小。

换句话说，yield如果去除它返回数据的功能，就可以看作是协程。

.. code-block:: python

    def sub_func(prefix):
        print(prefix, "do something1")
        yield
        print(prefix, "do something2")
        yield
        print(prefix, "do something3")
        yield
        print(prefix, "do something4")
        yield

    def main():
        coroutine1 = sub_func("coroutine1")
        coroutine2 = sub_func("coroutine2")
        # 主进程可以随时选择一个协程运行，但是必须自己调度。
        next(coroutine1)
        next(coroutine1)
        next(coroutine2)
        next(coroutine2)
        next(coroutine1)
        next(coroutine1)

    main()
    # out:
    # coroutine1 do something1
    # coroutine1 do something2
    # coroutine2 do something1
    # coroutine2 do something2
    # coroutine1 do something3
    # coroutine1 do something4

这种协程乍一看也并没有什么卵用，也不会加快速度，而且主进程也不能随意暂停协程，还必须协程内部自觉地写上yield。接着往下看。

借助于 yield from 语法糖，我们还可以实现多层协程之间的串联。


.. code-block:: python

    def sub_func2(prefix):
        print(prefix, "do otherthing1")
        yield
        print(prefix, "do otherthing2")
        yield


    def sub_func(prefix):
        print(prefix, "do something1")
        # 注意!!! 使用 yield from 串联
        yield from sub_func2()
        print(prefix, "do something2")
        yield from sub_func2()
        print(prefix, "do something3")
        yield from sub_func2()
        print(prefix, "do something4")
        yield from sub_func2()

这下就有点意思了，因为 yield from 在 sub_func中的作用类似于return，用来返回一些对象。
但是 yield 关键词似乎仍然没什么作用。而且这种协程手动调度也很麻烦。

这个时候 Future 就派上用场了！！！

真正意义上的协程，yield 必须返回一个 Future 。 主进程通过一直监听 Future 的完成情况来进行调度。
而Future中往往放着一些很耗时却又不占CPU的操作，比如网络请求，读写磁盘。

我们可以很容易就想到下面的调度方法，轮训每一个 yield 返回的 Future，Future执行完成后，再next调度。
下面这段代码不能运行，因为这里面的 Future 是假想的。

.. code-block:: python

    def sub_func(prefix):
        print(prefix, "do something1")
        yield Future(lambda : requests.get("www.github.com"))
        yield Future(lambda : requests.get("www.google.com"))
        yield Future(lambda : requests.get("www.baidu.com"))

    def loop_coroutines():
        while True:
            future, coroutine = future_queue.get(timeout = 3)
            while not future.done():
                time.sleep(0.1)
            try:
                future_queue.push((next(coroutine), coroutine))
            except StopIteration:
                pass

    def main():
        coroutine1 = sub_func("coroutine1")
        coroutine2 = sub_func("coroutine2")

        # 初始化 Future
        future_list = []
        future_list.append((next(coroutine1), coroutine1))
        future_list.append((next(coroutine1), coroutine1))

        # 循环遍历每一个协程。等待Future对象。
        while future_list:
            old_future = []
            new_future = []
            for future, coroutine in future_list:
                if future_list.done():
                    try:
                        new_future.append((next(coroutine), coroutine))
                    except StopIteration:
                        pass
                else:
                    old_future.append((future, coroutine))
            future_list = old_future + new_future


这段代码可以大致理解协程的工作方式，其中的Future是使用线程实现的异步，实际上Python原生的协程并非使用线程实现。

使用这种调度方式，可以在单线程内同时执行多个逻辑，避免io时间浪费cpu，将cpu的性能压榨到极致。

新协程
--------------------

python3.5 之后，新的协程函数可以使用 async def 声明，内部的yield、yield from 转换为 await.

并且也不需要我们自己编写调度的循环了，直接使用 asyncio 中的 loop 。

.. code-block:: python

    import asyncio

    async def sub_func2():
        print("Hello")
        return 9

    async def sub_func():
        await sub_func2()

    loop = asyncio.get_event_loop()
    loop.run_until_complete(sub_func())
    loop.close()

更为重要的是，通过 asyncio、Task、Future 提供的一系列接口，可以实现协程的随意退出、监控。

接口说明
------------------------

asyncio.run

    运行一个协程

    .. code-block:: python
     
        import asyncio
        import time

        async def say_after(delay, what):
            await asyncio.sleep(delay)
            print(what)

        async def main():
            print(f"started at {time.strftime('%X')}")

            await say_after(1, 'hello')
            await say_after(2, 'world')

            print(f"finished at {time.strftime('%X')}")

        asyncio.run(main())
    
    输出::

        started at 17:13:52
        hello
        world
        finished at 17:13:55

asyncio.create_task/asyncio.ensure_future

    将协程转换为一个异步task， task之间在运行的时候是并行的。

    .. code-block:: python

        async def main():
            task1 = asyncio.create_task(
                say_after(1, 'hello'))

            task2 = asyncio.create_task(
                say_after(2, 'world'))

            print(f"started at {time.strftime('%X')}")

            # Wait until both tasks are completed (should take
            # around 2 seconds.)
            await task1
            await task2

            print(f"finished at {time.strftime('%X')}")


    输出，可以观察到比之前运行时间少了一秒::

        started at 17:14:32
        hello
        world
        finished at 17:14:34

    解释： 当协程被包含在 asyncio.create_task 的那一刻，就已经立即开始运行了，运行到 await task1 的时候，表示等待 task1 执行完成，
    task1 执行完成后，接着  await task2 , 等待task2执行完成。

await

    await 是用在 async def 函数中的关键字。 用于在协程内部调用执行其他协程。

    ::

        import asyncio

        async def nested():
            return 42

        async def main():
            # 这种方法调用时错误的，没有任何效果，
            # 因为 nested 仅仅返回了一个协程，并没有触发其执行。
            nested()

            # 调用其他协程的正确方法
            print(await nested())  # will print "42".

        asyncio.run(main())

asyncio.sleep

    协程的睡眠，和time.sleep的区别在于，它不会占用cpu，直接返回一个Future对象。

asyncio.gather

    并行启动一批协程，并返回其结果。

asyncio.shield

    保护一个协程，使其不会 cancelled 退出。

asyncio.wait_for

    等待一个协程一段时间，如果一段时间后还没有执行完，就抛出异常。

asyncio.wait

    等待一组协程一段时间，如果一段时间后还没有执行完，就抛出异常。
    和wait_for的区别在于，它还可以指定一个参数，用于设置退出的条件，可以有下面几种值。

    * FIRST_COMPLETED: 这组协程中只要有一个完成就退出。
    * FIRST_EXCEPTION: 如果有一个协程抛出异常或者正常执行完，就算完成，然后退出。
    * ALL_COMPLETED: 所有协程都结束或者cancelled退出， 才算完成

asyncio.as_completed(aws, *, loop=None, timeout=None)

    等待一组协程执行完，会返回一个Future迭代器，按照执行完成的顺序排列。
    如果在timeout内还没有执行完成，就抛出 `` asyncio.TimeoutError`` 异常。

asyncio.run_coroutine_threadsafe(coro, loop)

    将一个协程交给某个loop执行，这个操作是线程安全的。

asyncio.current_task(loop=None)

    获取某个loop事件循环正在执行的task

asyncio.all_tasks(loop=None)

    获取某个loop事件循环正在调度的所有task

更详细的介绍见 ``Python官方文档 > library.pdf > asyncio > Coroutines and Tasks``


补充
============================

Future
-------------------------

result()

    判断当前task是否正常执行完成。

set_result(result)

    协程执行完成后，设置协程的返回值

set_exception(exception)

    如果协程抛异常了，通过这个接口设置协程的异常。

cancel()

    将一个Task退出。

cancelled()

    返回当前task是否是通过 cancell 退出的。

done()

    判断当前task是否正常执行完成。

exception()

    如果task执行过程中抛出异常，通过这个接口获取这个异常。

add_done_callback(callback, *, context=None)/remove_done_callback(callback)

    注册/注销task完成后的回调函数

get_loop()

    获取调用当前协程的loop.

Task对象接口
----------------------

cancel()

    将一个Task退出。

cancelled()

    返回当前task是否是通过 cancell 退出的。

done()

    判断当前task是否正常执行完成。

result()

    获取当前task的返回值。

exception()

    如果task执行过程中抛出异常，通过这个接口获取这个异常。

add_done_callback(callback, *, context=None)/remove_done_callback(callback)

    注册/注销task完成后的回调函数

get_stack(*, limit=None)

    如果当前task仍在执行，可以获取task的堆栈，如果已经执行完，返回一个空列表。

print_stack(*, limit=None, file=None)

    直接打印task堆栈


yield
----------------

为了避免给大家造成像我之前一样的混乱。就不说 yield 方式的旧式协程了。
只是顺便提一下yield的另外一个特性： yield不仅能用来返回数据，还可以用来接收数据。
它自带了一个send方法。


.. code-block:: pytohn

    def pow_calc():
        ret = 0
        while True:
            # 注意，一般都是 yield ret 返回对象，
            # 这里返回对象的同时还接受了一个 number。
            number = yield ret
            ret = number*number

    def main():
        calc = pow_calc()
        next(calc)  # 这也是yield协程比较丑陋的地方，需要手动使用next触发一下

        print(calc.send(9)) # out: 81
        print(calc.send(7)) # out: 49
        print(calc.send(6)) # out: 36

    main()

虽然之前的代码都是使用 next 触发 yield 协程的执行，现实中一般都是通过 send 方法触发执行的，这样可读性更好。

虽然Python协程和yield现在是分家了，但是内部实现方式还是相通的。
协程触发之后，也会有一个send方法，但是前面也提到过，yield去掉数据传递的功能就是协程，
所以规定协程调用send的时候，除了None不能传递其他对象。

协程是更轻量级的线程，作用是上下文切换和调度，和yield传递数据的功能有天壤之别。
