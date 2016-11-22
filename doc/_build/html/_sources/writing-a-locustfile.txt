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
属性来定义一个TaskSet的tasks(使用@task装饰器会只填充 *tasks* 属性)。

*tasks* 属性既是一个python可调用的列表，也是一个 *<callable : int>* 字典。tasks是可调用的python，它接收一个参数-TaskSet类的
实例用来执行任务。这里是一个非常简单的locustfile例子(这个locustfile并不实际负载测试任何东西)::

    from locust import Locust, TaskSet
    
    def my_task(l):
        pass
    
    class MyTaskSet(TaskSet):
        tasks = [my_task]
    
    class MyLocust(Locust):
        task_set = MyTaskSet


如果任务属性被定义为一个列表，每次都会在 *tasks* 属性中随机挑选一个任务来执行。然而，如果 *tasks* 是一个字典-
以key来调用，int型作为值-将会一个整数作为比例来被选择执行。所以一个task看起来就像这样::

    {my_task: 3, another_task:1}

*my_task* 将会被执行3倍于 *another_task* 执行的次数.

TaskSets 可以嵌套
----------------------

TaskSets一个非常重要的特性是可以被嵌套，因为实际的网站通常是以分层的方式构建的，有很多子功能。因此嵌套的TaskSet
允许我们以一个更逼真的方式来定义模拟用户的行为。例如，我们可以要按一下结构来定义TaskSet:

* 主用户行为

 * 索引页
 * 论坛页
 
  * 读线程
  
   * 回复
   
  * 新线程
  * 查看下一页
  
 * 浏览分类
 
  * 看电影
  * 筛选电影
  
 * 关于页

嵌套TaskSet的方式就像你使用 **tasks** 属性指定一个任务一样，但是你指向另一个TaskSet来取代指向一个python函数::

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

所以在上面的例子中，如果ForumPage在 UserBehaviour TaskSet执行时被选来执行，ForumPage的TaskSet就会开始执行。
论坛页的TaskSet就会选择它自己的任务中的一个，执行它，然后等待，如此往复。

在上面的例子中有一个重要的事情需要指出的是，就是在ForumPage的stop方法中调用的self.interrupt()。它本质上是
停止执行ForumPage的任务集合，然后UserBehaviour实例中的操作会继续执行。如果我们没有在ForumPage的某个地方
调用 :py:meth:`interrupt() <locust.core.TaskSet.interrupt>` 方法，那么Locust将永远不会停止运行ForumPage
任务一旦它开始执行了。但是有了interrupt函数，我们就能-和任务权重一起-定义模拟用户离开论然的可能性。

在一个类的内部定义一个嵌套的TaskSet也是可以的，使用 :py:meth:`@task <locust.core.task>` 装饰器，
就像定义一个普通任务时那样::

    class MyTaskSet(TaskSet):
        @task
        class SubTaskSet(TaskSet):
            @task
            def my_task(self):
                pass


on_start 函数
---------------------

一个TaskSet类可以随意的定义一个 :py:meth:`on_start <locust.core.TaskSet.on_start>` 函数。如果这样做了的话，这个函数
就会在一个模拟用户开始执行TaskSet类的时候被调用。

引用 Locust 实例或者父 TaskSet 实例
---------------------------------------------------------------

一个TaskSet实例会有一个 :py:attr:`locust <locust.core.TaskSet.locust>` 属性指向它的 Locust 实例，
:py:attr:`parent <locust.core.TaskSet.parent>` 属性指向它的父TaskSet(它会指向Locust实例，在基础的TaskSet中)。


发起 HTTP 请求
=====================

到目前为止，我们只讨论了一个Locust用户的任务调用部分。为了实际去负载测试一个系统我们需要发起HTTP请求。为了帮助我们做这些
:py:class:`HttpLocust <locust.core.HttpLocust>` 类诞生了。当使用这个类时，每个实例拥有一个
:py:attr:`client <locust.core.Locust.client>` 属性，它是 :py:attr:`HttpSession <locust.core.client.HttpSession>`
的一个实例，可以用来发起HTTP请求的。

.. autoclass:: locust.core.HttpLocust
    :members: client
    :noindex:

当继承了HttpLocust类之后，我们可以使用它的client属性来对服务器发起HTTP请求。这里是一个locust 文件的例子，可以用来负载一个有
两个URL的站点；**/** 和 **/about/**::

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

使用上面的Locust类，每个模拟用户在请求间隔会等待5-15秒，并且**/** 会被请求两倍于 **/about/**。

细心的读者会发现一个奇怪的地方就是我们可以在TaskSet使用 *self.client* 引用HttpSession实例
而不是 *self.locust.client*。我们可以这样做是因为 :py:class:`TaskSet <locust.core.TaskSet>`
类有一个方便的属性叫做client，简单的返回 self.locust.client 。


使用 HTTP client
======================

每个HttpLocust实例都有一个 :py:class:`HttpSession <locust.clients.HttpSession>` 实例在*client*属性中。
HttpSession 类实际是 :py:class:`requests.Session` 的一个子类，可以被用来发起HTTP请求，并且会反馈给Locust的
统计结果，通过使用:py:meth:`get <locust.clients.HttpSession.get>`,
:py:meth:`post <locust.clients.HttpSession.post>`, :py:meth:`put <locust.clients.HttpSession.put>`,
:py:meth:`delete <locust.clients.HttpSession.delete>`, :py:meth:`head <locust.clients.HttpSession.head>`,
:py:meth:`patch <locust.clients.HttpSession.patch>` 和 :py:meth:`options <locust.clients.HttpSession.options>`
这些方法。HttpSession 实例将会在请求之间保存cookie，所以它可以用来登录网站并且在请求之间保持会话。client 属性也可以引用
Locust实例的TaskSet实例，所以在你的任务中找到client并发起HTTP请求是很容易的。

这里是一个简单的例子，发起一个GET请求到 */about* 路径(在这里我们假设 *self* 是 :py:class:`TaskSet <locust.core.TaskSet>`
或 :py:class:`HttpLocust <locust.core.Locust>` 类的一个实例)::

    response = self.client.get("/about")
    print "Response status code:", response.status_code
    print "Response content:", response.content

这里是一个发起POST请求的例子::

    response = self.client.post("/login", {"username":"testuser", "password":"secret"})

安全模式
---------

HTTP client 被配置成运行在安全模式下。它所做的就在任意请求因为一个连接错误、超时或者类似情况导致的失败都不会抛出异常，
只返回一个空的虚拟的 Response 对象。这个请求会在Locust统计中被报告为一个失败，返回的虚拟的Response的 *content*
属性会被设置为None，并且它的 *status_code* 会被置为0。

手动控制如果一个请求需要被当作是成功或者失败
------------------------------------------------------------------------------

默认情况下，请求都会被标记为失败的请求除非HTTP返回码是OK(2xx)。大多数情况下，默认的就是你需要的。
但是有些时候-例如测试一个URL端点你希望返回404，或者测试一个设计的很差的系统即便返回 *200 OK*
也是有错误的-这时就需要一个手动的控制如果locust可以将一个请求当作成功或者失败。

通过使用 *catch_response* 声明和一个描述，用户可以将一个请求标记为失败，即便返回码是OK::

    with client.get("/", catch_response=True) as response:
        if response.content != "Success":
            response.failure("Got wrong response")

就像可以将一个OK返回码的请求标记为失败一样，同样可以使用 **catch_response** 声明 和一个 *with* 描述一起来
让一个HTTP错误码结果的请求仍然在统计结果中被报告为成功的::

    with client.get("/does_not_exist/", catch_response=True) as response:
        if response.status_code == 404:
            response.success()


使用动态参数对URL进行分组请求
-------------------------------------------------

网站的页面URL包含某些动态的参数的情况是非常常见的。通常在Locust的统计结果中将这些URL归类是有意义的。
这可以同多传一个 *name* 属性给 :py:class:`HttpSession's <locust.clients.HttpSession>` 不同的请求方法。

例如::

    # Statistics for these requests will be grouped under: /blog/?id=[id]
    for i in range(10):
        client.get("/blog?id=%i" % i, name="/blog?id=[id]")
