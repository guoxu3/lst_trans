安装
============

Locust 已经加入PyPI并且可以通过pip或easy_install安装

::

    pip install locustio

或::

    easy_install locustio

当安装好Locust后，你的shell中就有一个 **locust** 命令了(如果你没有使用virtualenv-你可以使用-确保你的python脚本
目录在你的系统路径中)

To see available options, run::

    locust --help


支持的 Python 版本
-------------------------

Locust 需要 **Python 2.6+** 版本.  目前还不兼容 Python 3.x 版本。


安装 ZeroMQ
-----------------

如果你想在多进程/机器上分布式运行Locust，我们推荐你同时安装上 **pyzmq**::

    pip install pyzmq

或::

    easy_install pyzmq

在Windows上安装 Locust
----------------------------

最简单的让Locust在Windows上运行的方法是先安装goevnt和greenlet预先编译好的二进制包，然后按照上面的说明操作。

你在这里可以找到非官方的支持windows的预编译python包：
`http://www.lfd.uci.edu/~gohlke/pythonlibs/ <http://www.lfd.uci.edu/~gohlke/pythonlibs/>`_

.. note::

    在Windows上运行Locust进行开发和测试你的负载测试脚本是可以很好的工作的。但是，当运行大规模测试时，就
    需要你在Linux机器上进行了，因为在Windows上gevent的性能是非常差的。


在 OS X 上安装 Locust
-------------------------

下面的是目前目前最快捷的方式来在OS X上使用Homebrew安装gevent。

#. 安装 `Homebrew <http://mxcl.github.com/homebrew/>`_.
#. 安装 libevent (gevent的依赖包)::

    brew install libevent

#. 然后按照上面的说明操作。

增大最大打开文件数量限制数
---------------------------------------------

在一个机器上的每一个HTTP连接都要打开一个新的文件(严格来说是一个文件描述符)。操作系统会为可以打开文件的最大数量设置一个
比较地低的限制,如果这个限制比一次测试中模拟的用户数要小的话，就会出现故障。

增大操作系统默认的文件限制最大值到一个超过你想要模拟的的用户数的值。如何来操作取决于实用的操作系统。
