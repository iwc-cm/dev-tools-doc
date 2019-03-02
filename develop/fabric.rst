################################
fabric
################################


简介
=================

fabric 是一个远程部署工具。底层使用了ssh协议，可以讲命令发送到其他机器执行，然后返回结果。非常适合编写运维自动化脚本。

安装
---------------

::

    pip install fabric

fab命令的使用
---------------

安装fabric后，会有一个fab命令。使用fab可以执行当前目录的 ``fabfile.py`` 脚本中的某个函数.

这个例子其实不太合适。因为是基础教程，只演示fab命令的用法还是绰绰有余的。

1. 新建一个名为 ``fabfile.py`` 的脚本,

    ::

        def hello(name="world"):
            print("Hello, {name}!".format(name=name))

#. 执行fabric命令

    ::

        $ fab hello
        Hello world!
        Done.

    这个命令会找到当前目录下的 fabfile.py , 执行其中的hello函数。

    带有参数的执行::

        $ fab hello:Jack
        Hello Jack!
        Done.

    执行多个函数::

        fab hello hello2 hello3 ......

之所以说这个例子不太适合，是因为里面仅仅是个普通的Python脚本，而fabric真正的作用在于，
可以使用fabric的接口批量在远程服务器上执行命令并返回结果。

fabric的使用
========================

在本地执行命令 - local
-----------------------------

.. code-block:: python

    from fabric.api import local

    def do_something():
        local("ls -l")
        local("su root")

local 可以执行本地命令，并且可以执行交互式命令，比如上面的 su。

错误处理 -- warn_only
----------------------------------

如果一个命令执行失败会直接退出脚本。可以通过warn_only将错误转化为一个警告， 保证脚本继续运行。

下面的程序会判断拦截错误，并询问用户是否继续执行。

.. code-block:: python

    from fabric.api import local, settings, abort
    from fabric.contrib.console import confirm

    def test():
        with settings(warn_only=True):
            result = local('./manage.py test my_app', capture=True)
        if result.failed and not confirm("Tests failed. Continue anyway?"):
            abort("Aborting at user request.")

执行命令 - run
---------------------------

run的用法和local差不多。

run会在当前连接的主机上执行命令，而local在本地机器上执行命令。

.. code-block:: python

    def do_something():
        local("ls -l")
        run("ls -l")

如果当前并没有连接任何主机，运行过程中会提示用户输入需要连接的主机。

切换目录 - cd
---------------------

这里有个陷阱， 下面的脚本试图实现 ``ls -l /etc`` 的功能。

.. code-block:: python

    def do_something():
        run("cd /etc/")
        run("ls -l")

但是并不能实现预期，因为两条命令是独立的，
每次执行一次命令，都会重新开启一个命令行，
导致第一句命令虽然切换到了etc目录，但是对第二句命令没有任何影响。

正确的写法

    .. code-block:: python

        def do_something():
            run("cd /etc/ && ls -l")

另外一种方法是使用fabric的cd。

    .. code-block:: python

        def do_something():
            with cd("/etc"):
                run("ls -l")
                run("ls -l")

定义远程连接
--------------------------

像上面一样，每次执行run都需要输入一次远程主机，很不方便。
可以通过 ``fabric.api.env`` 定义远程主机。 形式为 ``[user@]host[:port]``

.. code-block:: python

    env.hosts = ['192.168.0.101']

    def do_something():
        with settings(warn_only=True):
            with cd("/etc"):
                run("ls -l")

如果不希望输入密码，需要事先手动配置ssh免密码登录, 或者定义env.password。

使用 ``env.password`` 定义默认密码, 每次遇到主机需要登录的场合，就是用这个密码。

::

    env.password = "123456"

使用 ``env.passwords`` 为指定主机定义密码, passwords是一个字典，key是主机，value是密码。
需要注意的是，key必须是主机全称,即 ``user@host:port`` 的形式。

::

    env.passwords = {"root@192.168.0.101:22": "123456"}

如果定义了多个主机，可以通过 ``with settings(host='192.168.0.101')`` 指定特定主机，
或者通过 ``with settings(hosts=['192.168.0.101'])`` 指定多个主机

详细说明
======================

（待补充）

