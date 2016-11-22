===========================================
使用自定义客户端测试其他系统
===========================================

Locust将HTTP作为其主要目标建立的。但是，它可以很懂一的拓展到负载测试任何基于 请求/响应 模式的系统，通过
编写一个自定义的客户端触发 :py:attr:`request_success <locust.events.request_success>` 和
:py:attr:`request_success <locust.events.request_success>` 事件。


简单的 XML-RPC Locust 客户端
============================

这里是一个Locust类的例子， **XmlRpcLocust** ，提供了一个 XML-RPC 客户端，**XmlRpcClient**，
并跟踪所有请求的产生:

.. literalinclude:: ../examples/custom_xmlrpc_client/xmlrpc_locustfile.py

如果你之前写过Locust测试，你会认出叫 *ApiUser* 的类，它是一个标准的Locust类，包含一个有 *tasks* 在它的
*task_set* 属性中的 *TaskSet* 类。然而， *ApiUser* 继承自 *XmlRpcLocust*，你可以在ApiUser的右上方。
*XmlRpcLocust* 类在 *client* 属性下提供了一个 XmlRpcClient 实例。它基本上只是代理了函数调用，但是作为
触发 :py:attr:`locust.events.request_success` 和 :py:attr:`locust.events.request_failure` 事件重要的补充，
我们会将所有的调用在Locust的统计结果中报告。

这里一个XML-RPC服务器作为一个服务端工作的一个实现，参见下面的代码:

.. literalinclude:: ../examples/custom_xmlrpc_client/server.py
