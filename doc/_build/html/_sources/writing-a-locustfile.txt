======================
编写 locustfile
======================

locustfile 是一个普通的python文件。唯一必须的是它要定义至少一个类-然我们叫它locust类-这个类继承自Locust类。
A locustfile is a normal python file. The only requirement is that it declares at least one class—
let's call it the locust class—that inherits from the class Locust. 

Locust 类
================

一个locust类代表了一个用户(如果你愿意的话也可以是一个蝗虫)。Locust会为每一个模拟的用户产生(孵化)
出一个locust类的实例，一个locust类通常会定义几个属性。

:py:attr:`task_set <locust.core.Locust.task_set>` 属性
---------------------------------------------------------------

:py:attr:`task_set <locust.core.Locust.task_set>` 属性需要指向一个
:py:class:`TaskSet <locust.core.TaskSet>` 类，这个类定义了用户的行为，下面会有想象的描述。

*min_wait* 和 *max_wait* 属性
----------------------------------------

除了task_set属性之外，通常还要定义*min_wait* 和 *max_wait* 属性。他们是一个模拟用户在执行每个任务之间
需要等待最小和最大时间，单位是毫秒。*min_wait* 和 *max_wait* 默认值为1000，因此每个蝗虫总是在每个任务
之间总是会等待1秒，如果*min_wait* and *max_wait* 没有定义的话。

根据如下的locustfile，每个用户在任务之间会等待5至15秒::

    from locust import Locust, TaskSet, task
    
    class MyTaskSet(TaskSet):
        @task
        def my_task(self):
            print "executing my_task"
    
    class MyLocust(Locust):
        task_set = MyTaskSet
        min_wait = 5000
        max_wait = 15000

*min_wait* 和 *max_wait* 属性都可以在TaskSet类中被覆盖。

*weight* 属性
----------------------

你可以在同一个文件中运行两个蝗虫，像这样::

    locust -f locust_file.py WebUserLocust MobileUserLocust

如果你希望其中一个蝗虫执行更多次数，你可以在那些类里设置一个权重属性。比如说，网页用户可能是移动端用户的三倍::

    class WebUserLocust(Locust):
        weight = 3
        ....

    class MobileUserLocust(Locust):
        weight = 1
        ....


*host* 属性
--------------------

host属性是需要被加载的主机的URL前缀(即 "http://google.com")。通常，这个在locust启动时在命令行指定，
使用 :code:`--host` 选项。如果在locust类中定义的host属性，当没有在命令行指定:code:`--host` 时就会使用。

TaskSet 类
=============

如果Locust类代表了一个蝗虫，你可以说TaskSet类代表了蝗虫的大脑。每个Locust类必须有一个*task_set*属性集，
指向一个TaskSet。

一个TaskSet就是，就像它的名字所示的，一个任务的集合。这些任务是普通的可调用的python-如果我们是加载-测试一个
竞拍网站-可以做一些"加载开始页面"，"搜索某些商品"以及"出价"之类的事情。

当一个负载测试开始后，所产生的每一个Locust类将开始执行它们的TaskSet。接下来发生的就是每个TaskSet将要选择其中
一个任务并调用。然后会等待的好秒数是在Locust 类的 *min_wait* and *max_wait* 属性中选择的一个随机数
(除非min_wait/max_wait已经在TaskSet中直接定义了，这种情况下将使用它本身的值)。然后有会选择一个新的任务来调用，
然后又等待，如此往复。

声明 tasks
---------------

典型的为一个TaskSet声明tasks使用:py:meth:`task <locust.core.task>` 装饰器。

这是一个例子::

    from locust import Locust, TaskSet, task
    
    class MyTaskSet(TaskSet):
        @task
        def my_task(self):
            print "Locust instance (%r) executing my_task" % (self.locust)
    
    class MyLocust(Locust):
        task_set = MyTaskSet

 **@task** 需要一个可选的权重参数，可以用来指定任务的执行率。在下面的例子中 *task2* 执行次数将会是 *task1* 两倍::

    from locust import Locust, TaskSet, task
    
    class MyTaskSet(TaskSet):
        min_wait = 5000
        max_wait = 15000
        
        @task(3)
        def task1(self):
            pass
        
        @task(6)
        def task2(self):
            pass
    
    class MyLocust(Locust):
        task_set = MyTaskSet


tasks 属性
--------------

使用@task装饰来定义tasks是一个方便的，通常也是最好的方式。然而，也可以通过设置 :py:attr:`tasks <locust.core.TaskSet.tasks>`
属性来定义一个TaskSet的tasks(使用@task装饰器会只填充*tasks*属性)。

*tasks*属性既是一个python可调用的列表，也是一个*<callable : int>*字典。tasks是可调用的python，它接收一个参数-
The *tasks* attribute is either a list of python callables, or a *<callable : int>* dict. 
The tasks are python callables that receive one argument—the TaskSet class instance that is executing 
the task. Here is an extremely simple example of a locustfile (this locustfile won't actually load test anything)::

    from locust import Locust, TaskSet
    
    def my_task(l):
        pass
    
    class MyTaskSet(TaskSet):
        tasks = [my_task]
    
    class MyLocust(Locust):
        task_set = MyTaskSet


If the 
tasks attribute is specified as a list, each time a task is to be performed, it will be randomly 
chosen from the *tasks* attribute. If however, *tasks* is a dict—with callables as keys and ints 
as values—the task that is to be executed will be chosen at random but with the int as ratio. So 
with a tasks that looks like this::

    {my_task: 3, another_task:1}

*my_task* would be 3 times more likely to be executed than *another_task*.

TaskSets can be nested
----------------------

A very important property of TaskSets is that they can be nested, because real websites are usually 
built up in an hierarchical way, with multiple sub-sections. Nesting TaskSets will therefore allow 
us to define a behaviour that simulates users in a more realistic way. For example 
we could define TaskSets with the following structure:

* Main user behaviour

 * Index page
 * Forum page
 
  * Read thread
  
   * Reply
   
  * New thread
  * View next page
  
 * Browse categories
 
  * Watch movie
  * Filter movies
  
 * About page

The way you nest TaskSets is just like when you specify a task using the **tasks** attribute, but 
instead of referring to a python function, you point it to another TaskSet::

    class ForumPage(TaskSet):
        @task(20)
        def read_thread(self):
            pass
        
        @task(1)
        def new_thread(self):
            pass
        
        @task(5)
        def stop(self):
            self.interrupt()
    
    class UserBehaviour(TaskSet):
        tasks = {ForumPage:10}
        
        @task
        def index(self):
            pass

So in above example, if the ForumPage would get selected for execution when the UserBehaviour 
TaskSet is executing, then the ForumPage TaskSet would start executing. The ForumPage TaskSet 
would then pick one of its own tasks, execute it, then wait, and so on. 

There is one important thing to note about the above example, and that is the call to 
self.interrupt() in the ForumPage's stop method. What this does is essentially that it will 
stop executing the ForumPage task set and the execution will continue in the UserBehaviour instance. 
If we wouldn't have had a call to the :py:meth:`interrupt() <locust.core.TaskSet.interrupt>` method 
somewhere in ForumPage, the Locust would never stop running the ForumPage task once it has started. 
But by having the interrupt function, we can—together with task weighting—define how likely it 
is that a simulated user leaves the forum.

It's also possible to declare a nested TaskSet, inline in a class, using the 
:py:meth:`@task <locust.core.task>` decorator, just like when declaring normal tasks::

    class MyTaskSet(TaskSet):
        @task
        class SubTaskSet(TaskSet):
            @task
            def my_task(self):
                pass


The on_start function
---------------------

A TaskSet class can optionally declare an :py:meth:`on_start <locust.core.TaskSet.on_start>` function. 
If so, that function is called when a simulated user starts executing that TaskSet class.


Referencing the Locust instance, or the parent TaskSet instance
---------------------------------------------------------------

A TaskSet instance will have the attribute :py:attr:`locust <locust.core.TaskSet.locust>` point to 
its Locust instance, and the attribute :py:attr:`parent <locust.core.TaskSet.parent>` point to its 
parent TaskSet (it will point to the Locust instance, in the base TaskSet).


Making HTTP requests 
=====================

So far, we've only covered the task scheduling part of a Locust user. In order to actually load test 
a system we need to make HTTP requests. To help us do this, the :py:class:`HttpLocust <locust.core.HttpLocust>`
class exists. When using this class, each instance gets a 
:py:attr:`client <locust.core.Locust.client>` attribute which will be an instance of 
:py:attr:`HttpSession <locust.core.client.HttpSession>` which can be used to make HTTP requests.

.. autoclass:: locust.core.HttpLocust
    :members: client
    :noindex:

When inheriting from the HttpLocust class, we can use its client attribute to make HTTP requests 
against the server. Here is an example of a locust file that can be used to load test a site 
with two URLs; **/** and **/about/**::

    from locust import HttpLocust, TaskSet, task
    
    class MyTaskSet(TaskSet):
        @task(2)
        def index(self):
            self.client.get("/")
        
        @task(1)
        def about(self):
            self.client.get("/about/")
    
    class MyLocust(HttpLocust):
        task_set = MyTaskSet
        min_wait = 5000
        max_wait = 15000

Using the above Locust class, each simulated user will wait between 5 and 15 seconds 
between the requests, and **/** will be requested twice as much as **/about/**.

The attentive reader will find it odd that we can reference the HttpSession instance 
using *self.client* inside the TaskSet, and not *self.locust.client*. We can do this 
because the :py:class:`TaskSet <locust.core.TaskSet>` class has a convenience property 
called client that simply returns self.locust.client.


Using the HTTP client
======================

Each instance of HttpLocust has an instance of :py:class:`HttpSession <locust.clients.HttpSession>` 
in the *client* attribute. The HttpSession class is actually a subclass of 
:py:class:`requests.Session` and can be used to  make HTTP requests, that will be reported to Locust's
statistics, using the :py:meth:`get <locust.clients.HttpSession.get>`, 
:py:meth:`post <locust.clients.HttpSession.post>`, :py:meth:`put <locust.clients.HttpSession.put>`, 
:py:meth:`delete <locust.clients.HttpSession.delete>`, :py:meth:`head <locust.clients.HttpSession.head>`, 
:py:meth:`patch <locust.clients.HttpSession.patch>` and :py:meth:`options <locust.clients.HttpSession.options>` 
methods. The HttpSession instance will preserve cookies between requests so that it can be used to log in 
to websites and keep a session between requests. The client attribute can also be reference from the Locust 
instance's TaskSet instances so that it's easy to retrieve the client and make HTTP requests from within your 
tasks.

Here's a simple example that makes a GET request to the */about* path (in this case we assume *self* 
is an instance of a :py:class:`TaskSet <locust.core.TaskSet>` or :py:class:`HttpLocust <locust.core.Locust>` 
class::

    response = self.client.get("/about")
    print "Response status code:", response.status_code
    print "Response content:", response.content

And here's an example making a POST request::

    response = self.client.post("/login", {"username":"testuser", "password":"secret"})

Safe mode
---------
The HTTP client is configured to run in safe_mode. What this does is that any request that fails due to 
a connection error, timeout, or similar will not raise an exception, but rather return an empty dummy 
Response object. The request will be reported as a failure in Locust's statistics. The returned dummy 
Response's *content* attribute will be set to None, and its *status_code* will be 0.


Manually controlling if a request should be considered successful or a failure
------------------------------------------------------------------------------

By default, requests are marked as failed requests unless the HTTP response code is OK (2xx). 
Most of the time, this default is what you want. Sometimes however—for example when testing 
a URL endpoint that you expect to return 404, or testing a badly designed system that might 
return *200 OK* even though an error occurred—there's a need for manually controlling if 
locust should consider a request as a success or a failure.

One can mark requests as failed, even when the response code is OK, by using the 
*catch_response* argument and a with statement::

    with client.get("/", catch_response=True) as response:
        if response.content != "Success":
            response.failure("Got wrong response")

Just as one can mark requests with OK response codes as failures, one can also use **catch_response** 
argument together with a *with* statement to make requests that resulted in an HTTP error code still 
be reported as a success in the statistics::

    with client.get("/does_not_exist/", catch_response=True) as response:
        if response.status_code == 404:
            response.success()


Grouping requests to URLs with dynamic parameters
-------------------------------------------------

It's very common for websites to have pages whose URLs contain some kind of dynamic parameter(s). 
Often it makes sense to group these URLs together in Locust's statistics. This can be done 
by passing a *name* argument to the :py:class:`HttpSession's <locust.clients.HttpSession>` 
different request methods. 

Example::

    # Statistics for these requests will be grouped under: /blog/?id=[id]
    for i in range(10):
        client.get("/blog?id=%i" % i, name="/blog?id=[id]")
