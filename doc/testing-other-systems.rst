===========================================
使用自定义客户端测试其他系统
===========================================

Locust将HTTP作为其主要目标建立的。但是，它可以很懂一的拓展到负载测试任何基于 请求/响应 模式的系统，通过
编写一个自定义的客户端触发 :py:attr:`request_success <locust.events.request_success>` 和
:py:attr:`request_success <locust.events.request_success>` 事件。


简单的 XML-RPC Locust 客户端
============================

这里是一个Locust类的例子， **XmlRpcLocust** ，提供了一个 XML-RPC 客户端，**XmlRpcClient**，
并跟踪所有请求的产生::

    import time
    import xmlrpclib

    from locust import Locust, events, task, TaskSet


    class XmlRpcClient(xmlrpclib.ServerProxy):
        """
        Simple, sample XML RPC client implementation that wraps xmlrpclib.ServerProxy and
        fires locust events on request_success and request_failure, so that all requests
        gets tracked in locust's statistics.
        """
        def __getattr__(self, name):
            func = xmlrpclib.ServerProxy.__getattr__(self, name)
            def wrapper(*args, **kwargs):
                start_time = time.time()
                try:
                    result = func(*args, **kwargs)
                except xmlrpclib.Fault as e:
                    total_time = int((time.time() - start_time) * 1000)
                    events.request_failure.fire(request_type="xmlrpc", name=name, response_time=total_time, exception=e)
                else:
                    total_time = int((time.time() - start_time) * 1000)
                    events.request_success.fire(request_type="xmlrpc", name=name, response_time=total_time, response_length=0)
                    # In this example, I've hardcoded response_length=0. If we would want the response length to be
                    # reported correctly in the statistics, we would probably need to hook in at a lower level

            return wrapper


    class XmlRpcLocust(Locust):
        """
        This is the abstract Locust class which should be subclassed. It provides an XML-RPC client
        that can be used to make XML-RPC requests that will be tracked in Locust's statistics.
        """
        def __init__(self, *args, **kwargs):
            super(XmlRpcLocust, self).__init__(*args, **kwargs)
            self.client = XmlRpcClient(self.host)


    class ApiUser(XmlRpcLocust):

        host = "http://127.0.0.1:8877/"
        min_wait = 100
        max_wait = 1000

        class task_set(TaskSet):
            @task(10)
            def get_time(self):
                self.client.get_time()

            @task(5)
            def get_random_number(self):
                self.client.get_random_number(0, 100)


如果你之前写过Locust测试，你会认出叫 *ApiUser* 的类，它是一个标准的Locust类，包含一个有 *tasks* 在它的
*task_set* 属性中的 *TaskSet* 类。然而， *ApiUser* 继承自 *XmlRpcLocust*，你可以在ApiUser的右上方。
*XmlRpcLocust* 类在 *client* 属性下提供了一个 XmlRpcClient 实例。它基本上只是代理了函数调用，但是作为
触发 :py:attr:`locust.events.request_success` 和 :py:attr:`locust.events.request_failure` 事件重要的补充，
我们会将所有的调用在Locust的统计结果中报告。

这里一个XML-RPC服务器作为一个服务端工作的一个实现，参见下面的代码::

    import time
    import random
    from SimpleXMLRPCServer import SimpleXMLRPCServer
    import xmlrpclib

    def get_time():
        time.sleep(random.random())
        return time.time()

    def get_random_number(low, high):
        time.sleep(random.random())
        return random.randint(low, high)

    server = SimpleXMLRPCServer(("localhost", 8877))
    print "Listening on port 8877..."
    server.register_function(get_time, "get_time")
    server.register_function(get_random_number, "get_random_number")
    server.serve_forever()
