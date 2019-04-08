
#########################
基础工具
#########################

ssh
====================


远程连接

    ::

        ssh username@192.168.0.1

    这条命令会提示输入密码。

远程连接，执行指定命令

    ::

        ssh username@192.168.0.1 "ls -l"

使用ssh传输文件有两种

    1. 交互式sftp

        ::

            # 连接上后，ls 查看文件，cd 切换到指定目录， put上传文件，get下载文件。
            sftp username@192.168.0.1
            

    #. 非交互scp

        ::

            # 本地文件传输到远程
            scp ./test.txt username@192.168.0.1:~/

            # 远程文件传输到本地
            scp username@192.168.0.1:~/test.txt ./

ssh免密码登录

    ::

        # 生成秘钥
        ssh-keygen
        # 然后一路回车即可生成秘钥
        # ... ...

        # 拷贝秘钥到远程
        ssh-copy-id username@192.168.0.1 
        # ... 此处按照提示输入密码 ...

        # 验证， 此时登录应该不需要输入密码了
        ssh username@192.168.0.1 

.. tip:: 如果远程机器的用户名和你当前本地机器的用户名一样，上面所有命令都可以选择省略 username 。 如 ssh 192.168.0.1 。


深度学习设置 gpu id
==============================

查看gpu信息::

    nvidia-smi

使程序识别的gpu id 和nvidia显示的一致::

    export CUDA_DEVICE_ORDER=PCI_BUS_ID

设置使用的gpu::
    
    export CUDA_VISIBLE_DEVICES=0

https://discuss.pytorch.org/t/gpu-devices-nvidia-smi-and-cuda-get-device-name-output-appear-inconsistent/13150

https://codeyarns.com/2016/07/05/how-to-make-cuda-and-nvidia-smi-use-same-gpu-id/
