###
API
###


Locust 类
============

class Locust

    表示一个被孵化出的"用户"并攻击(attack)需要进行负载测试的系统。

    这个用户的行为是由task_set属性定义的，它需要指向一个 :py:class:`TaskSet <locust.core.TaskSet>` 类

    这个类通常会被一个定义某种客户端的类子类华。例如，当测试一个HTTP系统是，你通常想要使用
    :py:class:`HttpLocust <locust.core.HttpLocust>` 类。

    max_wait = 1000
        执行locust任务之间的最大等待时间

    min_wait = 1000
        执行locust任务之间的最小等待时间

    task_set = None
        TaskSet类定义locust的执行行为
        """TaskSet class that defines the execution behaviour of this locust"""

    weight = 10
        locust被选中的概率。权重越高，被选中的概率越大。

HttpLocust 类
================

class HttpLocust

    表示一个被孵化出的 HTTP "用户"并攻击(attack)需要进行负载测试的系统。

    这个用户的行为是由task_set属性定义的，它需要指向一个 :py:class:`TaskSet <locust.core.TaskSet>` 类

    这个类创建一个实例化的 *client*，它是一个在请求之间保持用户会话的HTTP客户端。

    client = None:
        HttpSession的实例，创建在一个实例化的Locust上。客户端支持cookie，因此可以在HTTP请求之间保持会话。


TaskSet 类
=============

class TaskSet

    类定义了Locust用户会执行的操作的任务集合。

    当一个TaskSet开始运行，它将会从 *tasks* 属性中选择一个任务，执行它，调用它的等待函数来从
    *min_wait* 和 *max_wait* 毫秒数之间选择一个随机数休眠。然后它会安排另一个任务执行，如此往复。

    TaskSet 可以被嵌套，这就意味着一个TaskSet的 *tasks* 属性可以包含另一个 TaskSet。如果一个嵌套的
    TaskSet是预定要执行的，它将会被实例化然后从当前执行的TaskSet中调用。正在当前运行的TaskSet中执行的将
    会被移交给嵌套的Taskset来继续运行，直到它抛出一个InterruptTaskSet异常，当
    :py:meth:`TaskSet.interrupt() <locust.core.TaskSet.interrupt>` 被调用时(操作会在第一个TaskSet中继续)

    client
        参考根Locust实例的 :py:attr:`client <locust.core.Locust.client>` 属性

    interrupt(self, reschedule=True)
        中断TaskSet并移交执行控制给父TaskSet。

        如果 *reschedule* 为True(默认值)，父Locust将立刻重新安排，并执行一个新的任务。

        这个方法不可以被根TaskSet(直接的，隶属于Locust类的*task_set* 属性的那个)调用，而嵌套的
        TaskSet则进一步进到下一层。

    locust = None
        当TaskSet已经实例化时会指向根Locust类。

    max_wait = None
        执行locust任务间隔的最大等待时间。可以用来覆盖根Locust类中定义的 max_wait，如果TaskSet中
        没有设置的话就会使用。

    min_wait = None
        执行locust任务间隔的最小等待时间。可以用来覆盖根Locust类中定义的 min_wait，如果TaskSet中
        没有设置的话就会使用。

    parent = None
        将会指向父TaskSet，或者Locust，类的实例当TaskSet已经被实例化时。对嵌套TaskSet类是很有用的。

    schedule_task(self, task_callable, args=None, kwargs=None, first=False):
        增加一个任务到Locust的任务执行队列中。

        *参数*:
        * task_callable: 将要安排的Locust任务
        * args: 要传到任务调用的参数
        * kwargs: 关键字参数字典，将被传给任务调用
        * first: 可选的关键字参数。如果为True，任务会被放在队列的第一个。

    tasks = []
        python调用的列表，代表一个locust用户任务。

        如果任务是一个列表，执行的任务会随机挑选。

        如果任务是一个二元元组 *(callable,int)* 列表，或者一个 {callable:int} 字典，任务也会被随机
        选择来执行，但是每个任务将会以它相应的整数值来设置权重。因此在下面的情况下  *ThreadPage* 将会
        被选择15倍于 *write_post* 的次数::

        class ForumPage(TaskSet):
            tasks = {ThreadPage:15, write_post:1}


task 装饰器
==============

在一个类中被作为一个方便的装饰器用来给一个TaskSet定义任务。 例::

        class ForumPage(TaskSet):
            @task(100)
            def read_thread(self):
                pass

            @task(7)
            def create_thread(self):
                pass


HttpSession 类
=================

执行web请求并且在请求之间保持会话(为了能登录或者登出网站)的类。每个请求都会记录所以locust可以展示统计数据。

这是一个稍微扩展版本的`python-request <http://python-requests.org>`_的 :py:class:`requests.Session` 类，
并且大部分这个类的工作方式是完全一样的。但是发起请求(get, post, delete, put, head, options, patch, request)
的方法现在可以只使用一个 *url* 参数，这只是URL的路径部分，在这种情况下URL的host部分将会由继承自一个Locust类
的host属性的HttpSession.base_url加上前缀。

每一个发起请求的方法都有两个额外的的可选参数，这是Locust定义的，并不存在于python-请求中。它们是:

    :param name: (可选的) 一个可以被指定用来当作Locust的统计数据中一个标签来替代URL路径的参数。
                 这可以用来分组不同的请求的URL，在Locust统计数据中作为单一的入口。
    :param catch_response: (可选的) 布尔参数，如果设置了，可以用来发起一个请求返回一个上下文管理来作为参数工作的限定语句。
                            这将允许请求基于返回的内容被标记为失败，尽管返回码是ok(2xx).相反的也是成立的，用户可以使用
                            catch_response 来抓取一个请求然后将它标记为成功的尽管返回码并不是(例 500或404)。

delete(url, **kwargs)
    发送一个 DELETE 请求。返回一个 :class:`Request` 对象。

    :param url:  新的 *Requst* 对象的URL。
    :param **kwargs:  *reques* 的可选参数.

    :Return type: `requests.Response <http://requests.readthedocs.io/en/latest/api/#requests.Response>`_

get(url, **kwargs)
    发送一个 GET 请求。返回一个 :class:`Request` 对象。

    :param url:  新的 *Requst* 对象的URL。
    :param **kwargs:  *reques* 的可选参数.

    :Return type:  `requests.Response <http://requests.readthedocs.io/en/latest/api/#requests.Response>`_

head(url, **kwargs)
    发送一个 HEAD 请求。返回一个 :class:`Request` 对象。

    :param url:  新的 *Requst* 对象的URL。
    :param **kwargs:  *reques* 的可选参数.

    :Return type: `requests.Response <http://requests.readthedocs.io/en/latest/api/#requests.Response>`_

options(url, **kwargs)
    发送一个 OPTIONS 请求。返回一个 :class:`Request` 对象。

    :param url:  新的 *Requst* 对象的URL。
    :param **kwargs:  *reques*的可选参数.

    :Return type: `requests.Response <http://requests.readthedocs.io/en/latest/api/#requests.Response>`_

patch(url, data=None, **kwargs)
    发送一个 PATCH 请求。返回一个 :class:`Request` 对象。

    :param url:  新的 *Requst* 对象的URL。
    :param data: (可选的)字典、字节或者类似文件的对象用来在 *Requesr* 的body中发送的。
    :param **kwargs:  *reques* 的可选参数.

    :Return type: `requests.Response <http://requests.readthedocs.io/en/latest/api/#requests.Response>`_

post(url, data=None, json=None, **kwargs)
    发送一个 POST 请求。返回一个 :class:`Request` 对象。

    :param url:  新的 *Requst* 对象的URL。
    :param data: (可选的)字典、字节或者类似文件的对象用来在 *Requesr* 的body中发送的。
    :param json: (可选的)在 *Requesr* 的body中发送的json。
    :param **kwargs:  *reques* 的可选参数.

    :Return type: `requests.Response <http://requests.readthedocs.io/en/latest/api/#requests.Response>`_

put(url, data=None, **kwargs)
    发送一个 PUT 请求。返回一个 :class:`Request` 对象。

    :param url:  新的 *Requst* 对象的URL。
    :param data: (可选的)字典、字节或者类似文件的对象用来在 *Requesr* 的body中发送的。
    :param **kwargs:  *reques* 的可选参数.

    :Return type: `requests.Response <http://requests.readthedocs.io/en/latest/api/#requests.Response>`_

request(self, method, url, name=None, catch_response=False, **kwargs):
    构造并发送一个 :py:class:`requests.Request`。返回 :py:class:`requests.Response` 对象。


    :param method: 新 :class:`Request` 对象的方法.
    :param url: 新 :class:`Request` 对象的 URL.
    :param name: (可选的) 一个可以被指定用来当作Locust的统计数据中一个标签来替代URL路径的参数。
                 这可以用来分组不同的请求的URL，在Locust统计数据中作为单一的入口。
    :param catch_response: (可选的) 布尔参数，如果设置了，可以用来发起一个请求返回一个上下文管理来作为参数工作的限定语句。
                            这将允许请求基于返回的内容被标记为失败，尽管返回码是ok(2xx).相反的也是成立的，用户可以使用
                            catch_response 来抓取一个请求然后将它标记为成功的尽管返回码并不是(例 500或404)。
    :param params: (可选的) :class:`Request` 中用来在查询字符串中发送的字典或者字节.
    :param data: (可选的) 在 :class:`Request` body中发送的字典或者字节。
    :param headers: (可选的) :class:`Request` 发送的HTTP 头部字典.
    :param cookies: (可选的) :class:`Request` 发送的字典或者CookieJar对象.
    :param files: (可选的)  多重编码上传的  ``'filename': file-like-objects``字典 .
    :param auth: (可选的) 认证元组或者调用接口提供 基础/摘要/自定义 的HTTP认证.
    :param timeout: (可选的) 服务器在放弃之前发送数据等待多长时间，浮点数,或者一个 (`connect timeout, read timeout <user/advanced.html#timeouts>`_) 元组。
    :type timeout: 浮点数 或 元组
    :param allow_redirects: (可选的) 默认设置为True.
    :type allow_redirects: 布尔值
    :param proxies: (可选的) 将协议映射到策略URL的字典。
    :param stream: (可选的) 是否直接下载响应内容。默认为 ``False``.
    :param verify: (可选的) 如果为 ``True``, SSL证书将被验证，也可以提供一个ca_bundle路径
    :param cert: (可选的) 如果是字符串，则是ssl客户端证书文件(.pem)的路径。如果是元组，是一个 ('cert', 'key') 对.


Response 类
==============

这个类实际上属于 `python-requests <http://python-requests.org>`_ 库，就是Locust用来发起HTTP请求的，
但是它包含在locust的API文档中因为它在写locust负载测试的时候是很核心的。你还可以在
`requests documentation <http://python-requests.org>`_ 查看 :py:class:`Response <requests.Response>` 类。

class Response
    :class:`Request` 对象，包含了服务端对一个HTTP请求的响应。

apparent_encoding
    可见的编码，有chardet库提供。
    The apparent encoding, provided by the chardet library

close()
    释放连接到连接池中。一旦这个方法被底层的原始对象调用了就不能再被访问到。


    Note: 通常不需要显式调用.

content
    响应的内容，以字节为单位。

cookies = None
    服务器发回的 Cookies的 CookieJar。

elapsed = None
    在发送请求和响应到达之间的经过时间总和(作为一个时间间隔)。这个特性明确的测量发送请求的第一个字节和
    完成解析报头之间花费的时间。因此它是由消费的响应内容或者流的关键字参数的值影响的。

encoding = None
    当访问 r.text 时进行编码解码。

headers = None
    Case-insensitive Dictionary of Response Headers. For example, headers['content-encoding'] will return the value of a 'Content-Encoding' response header.

history = None
    A list of Response objects from the history of the Request. Any redirect responses will end up here. The list is sorted from the oldest to the most recent request.

is_permanent_redirect
    True if this Response one of the permanent versions of redirect

is_redirect
    True if this Response is a well-formed HTTP redirect that could have been processed automatically (by Session.resolve_redirects()).

iter_content(chunk_size=1, decode_unicode=False)
    Iterates over the response data. When stream=True is set on the request, this avoids reading the content at once into memory for large responses.
    The chunk size is the number of bytes it should read into memory. This is not necessarily the length of each item returned as decoding can take place.

    chunk_size must be of type int or None. A value of None will function differently depending on the value of stream.
    stream=True will read data as it arrives in whatever size the chunks are received. If stream=False, data is returned as a single chunk.

    If decode_unicode is True, content will be decoded using the best available encoding based on the response.

    iter_lines(chunk_size=512, decode_unicode=None, delimiter=None)
    Iterates over the response data, one line at a time. When stream=True is set on the request,
    this avoids reading the content at once into memory for large responses.

    Note

    This method is not reentrant safe.

json(**kwargs)
    Returns the json-encoded content of a response, if any.

    Parameters:	**kwargs – Optional arguments that json.loads takes.
    Raises:	ValueError – If the response body does not contain valid json.
    links
    Returns the parsed header links of the response, if any.

raise_for_status()
    Raises stored HTTPError, if one occurred.

raw = None
    File-like object representation of response (for advanced usage). Use of raw requires that stream=True be set on the request.

reason = None
    Textual reason of responded HTTP Status, e.g. “Not Found” or “OK”.

request = None
    The PreparedRequest object to which this is a response.

status_code = None
    Integer Code of responded HTTP Status, e.g. 404 or 200.

text
    Content of the response, in unicode.

    If Response.encoding is None, encoding will be guessed using chardet.

    The encoding of the response content is determined based solely on HTTP headers, following RFC 2616 to the letter.
    If you can take advantage of non-HTTP knowledge to make a better guess at the encoding,
    you should set r.encoding appropriately before accessing this property.

url = None
    Final URL location of Response.

ResponseContextManager 类
============================

A Response class that also acts as a context manager that provides the ability to manually
control if an HTTP request should be marked as successful or a failure in Locust's statistics

This class is a subclass of :py:class:`Response <requests.Response>` with two additional
methods: :py:meth:`success <locust.clients.ResponseContextManager.success>` and
:py:meth:`failure <locust.clients.ResponseContextManager.failure>`.

failure(exc):
    将响应报告为失败的.

    exc既可以是一个python的异常，也可以是一个字符串，这种情况下它可以被包装在 CatchResponseError 中。

    例子::

        with self.client.get("/", catch_response=True) as response:
            if response.content == "":
                response.failure("No data")

success(self)
    将响应报告为成功的

    例子::

        with self.client.get("/does/not/exist", catch_response=True) as response:
            if response.status_code == 404:
                response.success()



InterruptTaskSet Exception
==========================
class InterruptTaskSet(Exception):

    Exception that will interrupt a Locust when thrown inside a task


.. _events:

Event hooks
===========

The event hooks are instances of the **locust.events.EventHook** class:

class EventHook(object):

    Simple event class used to provide hooks for different types of events in Locust.

    Here's how to use the EventHook class::

        my_event = EventHook()
        def on_my_event(a, b, **kw):
            print "Event was fired with arguments: %s, %s" % (a, b)
        my_event += on_my_event
        my_event.fire(a="foo", b="bar")

Available hooks
---------------

The following event hooks are available under the **locust.events** module:

request_success = EventHook()

    *request_success* is fired when a request is completed successfully.

    Listeners should take the following arguments:

    * *request_type*: Request type method used
    * *name*: Path to the URL that was called (or override name if it was used in the call to the client)
    * *response_time*: Response time in milliseconds
    * *response_length*: Content-length of the response


request_failure = EventHook()

    *request_failure* is fired when a request fails

    Event is fired with the following arguments:

    * *request_type*: Request type method used
    * *name*: Path to the URL that was called (or override name if it was used in the call to the client)
    * *response_time*: Time in milliseconds until exception was thrown
    * *exception*: Exception instance that was thrown


locust_error = EventHook()

    *locust_error* is fired when an exception occurs inside the execution of a Locust class.

    Event is fired with the following arguments:

    * *locust_instance*: Locust class instance where the exception occurred
    * *exception*: Exception that was thrown
    * *tb*: Traceback object (from sys.exc_info()[2])


report_to_master = EventHook()

    *report_to_master* is used when Locust is running in --slave mode. It can be used to attach
    data to the dicts that are regularly sent to the master. It's fired regularly when a report
    is to be sent to the master server.

    Note that the keys "stats" and "errors" are used by Locust and shouldn't be overridden.

    Event is fired with the following arguments:

    * *client_id*: The client id of the running locust process.
    * *data*: Data dict that can be modified in order to attach data that should be sent to the master.


slave_report = EventHook()

    *slave_report* is used when Locust is running in --master mode and is fired when the master
    server receives a report from a Locust slave server.

    This event can be used to aggregate data from the locust slave servers.

    Event is fired with following arguments:

    * *client_id*: Client id of the reporting locust slave
    * *data*: Data dict with the data from the slave node


hatch_complete = EventHook()

    *hatch_complete* is fired when all locust users has been spawned.

    Event is fire with the following arguments:

    * *user_count*: Number of users that was hatched


quitting = EventHook()

    *quitting* is fired when the locust process in exiting


