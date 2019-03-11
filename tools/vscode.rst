######################################
VSCode
######################################

VSCode是一种基于Chromium内核开发的编辑器，使用的是 electron 技术。

VSCode插件目录

    * :Windows: %USERPROFILE%\.vscode\extensions
    * :macOS: ~/.vscode/extensions
    * :Linux: ~/.vscode/extensions

Code-Server
==============================

vscode是基于Chromium内核，虽然只能在本地运行，但是总体上是前端的技术栈，
所以有人做了个可以在服务端运行的VSCode，可以通过浏览器访问。

:code-server官网: https://github.com/codercom/code-server

运行

    ::
    
        # 在端口 50001 上运行，并允许所有机器访问
        ./code-server -h 0.0.0.0 -p 50001

    运行后会输出一个随机的密码。
    
    访问 http://localhost:5001 输入密码就可以访问。


安装插件

    将插件在其他机器安装好，复制过去就可以使用了。

    因为VSCode使用的是 js 技术栈，属于平台无关的语言，插件也是js编写的，
    所以可以将其他平台的插件直接拷贝到Linux也是可以使用。

    默认插件目录

        * :Windows: （暂时不支持）%USERPROFILE%\.code-server\extensions
        * :macOS: ~/.code-server/extensions
        * :Linux: ~/.code-server/extensions

