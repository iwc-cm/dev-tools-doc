
#########################
git基础
#########################

.. important:: 一般的git教程都会先说git三连 clone/commit/push ，以便初学者快速入门，但是这个教程不这么讲，而是以git工作流的角度开始。\
               如果你之前学过一些git基础操作而不精通，就先把他们完全忘了吧。


一些概念
===================

仓库

    物理上的表现形式就是一个文件夹，如果把这个文件夹做成共享，就是一个url，比如github上的各种仓库。

分支

    在进行各种git操作比如提交、合并时，都是以分支为最小单位的，而不是仓库。

    每一个仓库可以有很多个分支，不同分支的代码可以不一样，分支之间可以对比不同之处并进行相互合并。
    
    这种分支的概念和svn大不相同，因为svn分支实际上是在一个仓库的基础上新建了另一个仓库，
    话句话说，svn并没有真正的分支概念，全都是仓库。
    而git的所有分支都是在同一个仓库中的，也就是说同一个文件夹中可以包含很多个分支。
    
    如果不能理解这种概念，可以想象一个仓库只有一个分支的情形，暂时将分支和仓库看成是一个东西，
    因为后面会详细说。

标签

    开发者可以修改分支的内容，然后提交，和svn是一样的。但是可以给某一次历史提交起一个名字，便于以后容易找到。
    
    一般这种别名都是用来标记一些历史上的重要提交，比如标记一个稳定版本、完成某个新功能的实现。
    github还有release的概念，release类似于标签，但是可以附加一些其他附件，用来发布版本。


本地仓库
=======================

1. 在本地新建一个空目录，将其初始化为git仓库。

    ::

        mkdir test_repo && cd test_repo
        git init

    现在使用这个文件夹仓库进行版本管理，尝试修改一些文件。

#. 新建一个文件 test.txt, 加入一句话，保存。

    比如使用 ``echo "this is a test file" > test.txt``

#. 查看当前git状态

    ::

        git status 

    会看到列出来test.txt是红色，表示新建的文件，并且没有被跟踪。还会看到提示使用git add 来跟踪文件。

#. 将test.txt加入到跟踪范围

    ::

        git add test.txt

    再使用git status查看状态，会发现test.txt变成了绿色，表示已经纳入了跟踪范围(暂存区)。

#. 使用 git commit 将所有暂存区的文件提交，使用 -m 选项附加提交的log。

    ::

        git commit -m "add test.txt"

#. 查看当前仓库的所有提交日志

    ::

        git log

完成！

这里只是说了如何创建一个git本地仓库，进行版本管理。

需要注意的是，上面所有操作都是在本地，commit也是在本地，
不像svn所有commit都需要连接上服务器。git版本提交可以不连接服务器，先在本地提交很多个版本，
在有条件联网的时候再连接服务器将所有提交推送到服务器。甚至从头到尾都可以不需要服务器，只在本地进行版本管理。


.. important::  关于本地分支, 同一时刻最多只能处在一个本地分支，所以，也可能当前并不处于任何本地分支。
                之前使用本地仓库commit前，实际上并没有任何本地分支，commit之后，git默认建立了一个master分支并把代码提交上去。
                使用 git branch 可以查看本地分支。

.. image:: /_static/git_local_flow.jpg
    :width: 340px
    :align: center


仓库和分支
=================

首先，需要强调的是，在本地使用 ``git init`` 初始化的仓库，其实是具有完备的版本控制能力的，
github做的事情，只不过是把这个文件夹放到网络上共享出来。

其次，git仓库中最重要的是 ``.git`` 隐藏文件夹，这个文件夹存放了git的所有信息和数据，
我们平常使用git的时候，在文件夹中看到的那些项目代码文件，仅仅是从 .git 文件夹中拷贝出来的一份镜像(或者理解为引用)。
``.git`` 文件夹里面也存放了分支、仓库之间的关系

仓库的对应
-------------------

新建一个空的本地仓库::

    mkdir tools-doc && cd tools-doc
    git init

其实是初始化了一个 ``.git`` 隐藏文件夹。

建立与远程仓库的对应关系,取别名为origin::

    git remote add origin https://github.com/iwc-cm/dev-tools-doc.git

这种对应关系，其实也是存放在 .git 文件夹中。

.. image:: /_static/git_remote.jpg
    :width: 340px
    :align: center

.. tip:: 使用git remote -h 查看remote子命令的详细使用方法

查看所有映射的远程仓库::

    git remote -v

可以查看到两个 fetch/push , 因为允许对同一个名字的远程分支的push、fetch对应不同url。

下载代码
----------------------

将(origin)远程仓库代码同步到本地仓库:

    git fetch origin

同步完后，可以看到当前文件夹并没有多出文件，仍然是空文件夹，
因为fetch仅仅是把远程仓库的数据下载到了本地的 ``.git`` 文件夹，
文件夹的内容永远都是显示的对本地分支的引用。

所以，如果希望本地文件夹看到内容，必须建立一个本地分支，并切换到本地分支, 后面会说。

.. image:: /_static/git_fetch.jpg
    :width: 340px
    :align: center

分支对应关系
--------------------

查看分支
*****************

::

    git branch
    git branch -r
    git branch -a
    # out: remotes/origin/master

因为 ``.git`` 文件夹会保存所有的**本地分支**和**远程分支**的信息，所以先忽略远程仓库，只考虑 .git 中存放的远程分支信息。
如下:

.. image:: /_static/git_fetch2.jpg


从远程分支基础上新建本地分支并切换到本地分支
***************************************************
::

    git checkout -b master origin/master

.. image:: /_static/git_checkout_b.jpg


以本地分支为基础新建本地分支
****************************************
::

    git branch bug

.. image:: /_static/git_branch.jpg

切换到另外一个本地分支
***********************************

::

    git checkout bug

使用 git branch 命令查看所有本地分支，前面有星号的是当前所在的本地分支。


将当前本地bug分支推送到origin远程仓库的同名分支
*****************************************************

::

    git push origin bug

.. image:: /_static/git_push.jpg

将远程仓库origin的master分支的所有更新下载到本地
*****************************************************
::

    git fetch origin master

将某个远程分支（如origin/bug）的代码合并到当前所处的本地分支
**************************************************************************
::

    git merge origin/bug

.. image:: /_static/git_merge.jpg


将本地bug分支的代码合并到当前所处的本地分支
***********************************************

::

    git merge bug

.. image:: /_static/git_merge_local.jpg

gitignore
=======================

git项目下新建 ``.gitignore`` 文件，符合其中模式的文件名称，不会纳入git版本管理。

一个典型的Python项目gitignore配置::

    /.idea/
    __pycache__/
    *.pyc

具有的模式如下, 以 test.txt文件 和 data/ 目录为例.

忽略文件

    test.txt

忽略目录

    data/

之忽略根目录下的文件，即绝对路径

    \/test.txt

之忽略根目录下的文件夹，即绝对路径

    \/data\/

忽略根目录data目录下的所有txt文件，但是不忽略子目录的txt文件

    \/data\/\*.txt

忽略根目录data目录及其子目录下的所有txt文件

    /data/\*\*/\*.txt

取反，即不忽略某种模式的文件

    !/data/\*\*/\*.txt

另外，使用 ``#`` 可以注释某一行。

ssh免密码
==================

github默认使用https连接进行远程仓库克隆，也可以切换为ssh。
如果设置了ssh免密码，就可以在提交的时候不用再次输入密码。
如下界面点击 ``use ssh`` 可以切换为ssh形式的url。

.. image:: /_static/clone_ssh.jpg

1. 打开github主页，个人信息 设置 界面.

    .. image:: /_static/git_setting.jpg
        :height: 350px

#. 找到 ``SSH and GPG keys`` 配置
#. 点击 ``New SSH Key`` 新建一个可信的key。

    .. image:: /_static/git_ssh_setting.jpg

#. 将本机的 ``$HOME/.ssh/id_rsa.pub`` 的内容拷贝进去。
#. 如果没有 ``$HOME/.ssh/id_rsa.pub`` ，就使用 ``ssh-keygen`` 命令， 一路回车，生成一个 ssh key.


场景案例
==========================

克隆代码
-----------------------

日常修改、提交
-----------------------
保持master是干净的

同步代码到远程
-----------------------

基于别人的代码上开发
--------------------------

个人主页
=====================
待补充

项目文档
==================
待补充
