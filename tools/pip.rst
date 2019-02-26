################################
pip使用方法
################################

pip是Python官方的包管理器。

使用pip自动从远程源下载包到本地并安装。

使用方法
=====================

下面以django、requests、tensorflow 包为例。

搜索所有名称中出现django的包::

    pip search django

安装django::

    pip install django

同时安装多个包::

    pip install django requests tensorflow

安装某个版本的包::

    pip install "django==1.8.0"
    pip install "django<=1.9" 
    pip install "django>2.0" "1.8<=tensorflow<=1.12"

如果包已经存在，并且不指定版本，pip会拒绝安装，提示已经存在。
可以指定 ``-U`` 选项升级

::

    pip install django -U

只下载而不安装::

    # 下载django以及其所有依赖
    pip download django
    pip download "django==1.8.0"

    # 只下载django，不下载依赖
    pip download django --no-deps 


卸载django::

    pip uninstall django



download/install/uninstall




升级pip
=====================

使用pip升级pip（自升级）::

    python -m pip  install --upgrade pip


wheel与pip
==================

pip是目前python官方标准，在这之前有distutil、setuptools 两个标准。
同时还存在 wheel 标准。pip集成了所有这些标准并进行了扩充。

其中wheel标准比较特殊，后缀为 .whl 。当一个Python库用 C、C++ 编写时，在安装的时候就需要临时调用gcc编译为二进制。
wheel标准定义了 当一个库被编译为二进制后，怎么打成一个二进制的包，
然后其他机器就可以直接那这个二进制的包安装，而不需要再次编译，节省时间，也不用在目标机器安装gcc编译工具。

缺点是，在某个平台编译的某个Python版本的包，只能在相同平台相同Python版本安装和使用。

wheel包命名案例: tensorflow-1.12.0-cp36-cp36m-win_amd64.whl

表示这个包 只能用于 Python3.6，windows 64位。

pip 提供一个子命令 调用wheel工具。

将一个包编译为 wheel::

    pip wheel ./pyltp.tar.gz # 将当前目录下pyltp包编译为whl
    pip wheel pyltp # 自动下载pyltp的包并编译为wheel


requirements描述文件
==============================

requirements描述文件指定每个包的版本，通过 pip 的 ``-r`` 选项 使用。

内容如下:

::

    django==1.8.0
    tensorflow<=1.8



使用 requirements.txt 描述文件批量安装::

    pip install -r requirements.txt

上面的命令会安装所有出现在 requirements里面的包。

一般在项目里面写一个这种依赖文件，就可以方便一次性安装所有依赖。

requirements描述文件不仅可以用于 ``pip install`` , 
也可以用在 ``download`` 、 ``uninstall`` 、 ``wheel`` 。


安装本地磁盘的包
===========================

将本地某个目录作为源，安装所有包。

借助以下两个选项:

* --no-index: 不要使用任何远程pip源，也就是不联网下载库。
* -f, --find-links <path>: 将某个路径作为源，可以使本地目录。


::

    pip download tensorflow
    pip install tensorflow --no-index -f ./

真香！！！


pip全局选项
==============

-i, --index-url     手动指定远程源地址，不适用系统配置中的源。
--extra-index-url   指定额外的远程源，会同时使用配置的源。
--no-index          禁用所有源，只保留 -f, --find-links 指定的源。
-f, --find-links <url>  将指定路径作为源，可以使本地目录，或者远程url
--process-dependency-links  允许安装setup.py中使用dependency_links指定的依赖。
                            一般不会用到这个选项，而且以后会废弃

-h, --help      显示帮助
--isolated      运行pip时，忽略环境变量和所有配置
-v, --verbose   打印额外信息
-V, --version   查看pip版本
-q, --quiet     打印很少信息，除非出现错误、警告
--log <path>    将所有打印追加到指定log
--proxy <proxy>     使用代理，格式为 [user:passwd@]proxy.server:port
--retries <num>     如果远程源连接不上，重试的次数，默认是5
--timeout <sec>     连接远程源超时时间
--exists-action <action>    当目录已经存在时，采取什么操作，
                            s-使用已存在目录，i-忽略，w-删除重建, b-备份，a-中断操作
--trusted-host <host>       信任某个远程源，如果源是http而不是https，需要加这个信任。
--cert <path>               手动指定远程源的证书
--client-cert <path>        没用过
--cache-dir <dir>           pip安装包时，会从远程下载缓存到本地某个目录，如果下次还要安装，
                            直接读取本地缓存读取，这个选项指定缓存目录。
--no-cache-dir              禁用缓存。
--disable-pip-version-check     不要提示pip有更新。
--no-color                  输出文字时不要带有颜色，某些古老的终端可能显示不正常。


pip每个子命令也有自己的独有选项。

比如 ``pip install -r``， ``-r`` 就是属于 install 子命令的选项。


配置
========================

配置文件路径
--------------------

老版本pip配置文件位置

    * Linux: $HOME/.pip/pip.conf
    * windows: %HOME%\pip\pip.ini

    也可以通过环境变量 PIP_CONFIG_FILE 指定一个默认位置。

新版本配置文件位置

    * Linux: $HOME/.config/pip/pip.conf , 由XDG_CONFIG_HOME(默认 $HOME/.config)环境变量决定，
    * windows: %APPDATA%\pip\pip.ini

配置文件格式
-------------------------

官方说法: https://pip.pypa.io/en/stable/user_guide/#configuration

::

    [global]
    index-url=https://pypi.tuna.tsinghua.edu.cn/simple
    timeout = 60

    [install]
    index-url=https://pypi.org/simple/
    timeout = 10

    [download]
    timeout = 20

global配置

    一般都指定一个 global 配置，其中存放上面说的全局选项。
    配置名称和长选项一样，比如上面 --index-url ，存放到配置里面就叫 index-url,

    执行pip，除非制定了--isolated，否则都会使用global中的配置。

子命令配置

    上面案例中， 当执行 pip install 子命令时，除了使用global配置，还会使用 install中的配置， 
    如果install和global配置冲突了，优先使用install配置，
    并且install中还可以存放 pip install 子命令的独有配置

执行 ``pip install django`` 等价于 ``pip install django --index-url=https://pypi.org/simple/ --timeout=10``

执行 ``pip download django`` 等价于 ``pip install django --index-url=https://pypi.tuna.tsinghua.edu.cn/simple --timeout=20``
