===========================
分布式运行 Locust
===========================

当单个机器不够模拟出你需要的用户数时，Locust支持在多个机器上分布式运行负载测试。

要实现这个，你要启动一个Locust实例通过使用 ``--master`` 标识。这个实例将会运行Locust的web接口，就是你
开始测试和查看实施统计数据的地方。master节点自身并不模拟任何用户。相反你必须穷一个或者-很可能是-多个slave
Locust节点使用 ``--slave`` 标识，同时还有 ``--master-host`` (来指定master节点的IP/主机名)。

常见的设置是在一台机器上运行一个单独的master，然后在slave机器的每一个处理器核上运行一个slave 实例。

.. note::
    在分布式运行Locust时，master机器和每个slave机器都必须有一份locust测试脚本的拷贝。


例子
=======

在 master 节点上启动locust::

    locust -f my_locustfile.py --master

然后在每个slave上 (替换 ``192.168.0.14`` 为master 机器的IP)::

    locust -f my_locustfile.py --slave --master-host=192.168.0.14


选项
=======

``--master``
------------

在master 模式上设置locust。 web 接口会运行在这个节点上。


``--slave``
-----------

在slave 模式上设置locust.


``--master-host=X.X.X.X``
-------------------------

和 ``--slave`` 一起使用来设置master节点的 主机名/IP (默认是 127.0.0.1)

``--master-port=5557``
----------------------

和 ``--slave`` 一起使用来设置master节点的端口号 (默认是 5557)。注意locust会使用指定的端口，
以及还有端口号+1。所以如果5557被使用了，locust会同时使用5557和5558。

``--master-bind-host=X.X.X.X``
------------------------------

和 ``--master`` 一起使用. 指定master节点会绑定到的哪个网络接口，默认是 * (所有可用的接口).

``--master-bind-port=5557``
------------------------------

和 ``--master`` 一起使用. 指定master节点将会监听的网络端口。默认是5557. 注意locust会使用指定的端口，
此外还有端口号+1。所以如果5557被使用了，locust会同时使用5557和5558。
