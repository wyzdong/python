# python
[scrapyd]
> 项目蛋将被存储的目录。

eggs_dir    = eggs
> Scrapy日志的存储目录。 如果要禁用存储日志，请将此选项设置为空，

logs_dir    = logs
> Scrapy项目将被存储的目录。 此选项默认处于禁用状态，因为您需要使用数据库或Feed导出器。 通过覆盖scrapy设置FEED_URI将其设置为非空结果可将抓取的项目源存储到指定的目录。

items_dir   =
> 每个蜘蛛保持完成的工作数量。 默认为5.这是指日志和项目。此设置在以前的版本中被命名为logs_to_keep。

jobs_to_keep = 5
> 项目数据库将被存储的目录（这包括蜘蛛队列）。

dbs_dir     = dbs
> 将开始的并发Scrapy进程的最大数量。 如果未设置或0，它将使用系统中可用的cpus数量乘以max_proc_per_cpu选项中的值。 默认为0。

max_proc    = 0
> 每个cpu将启动的最大并发Scrapy进程数。 默认为4。

max_proc_per_cpu = 4
> 保留在启动器中的完成进程的数量。 默认为100.这仅反映在网站/作业端点和相关的json web服务上。

finished_to_keep = 100
> 用于轮询队列的时间间隔，以秒为单位。 默认为5.0。 可以是浮点数，如0.2

poll_interval = 5.0
> 绑定地址

bind_address = 127.0.0.1
bind_address = 192.168.11.11

> 端口
http_port   = 6800
> 是否启用调试模式。 默认为关闭。 当启用调试模式时，当处理JSON API调用时出现错误时，将返回完整的Python回溯（作为纯文本响应）。

debug       = off
> 将用于启动子流程的模块。 您可以使用自己的模块自定义从Scrapyd启动的Scrapy进程。

runner      = scrapyd.runner
> 返回要使用的（Twisted）应用程序对象的函数。 如果要通过添加和删除自己的组件和服务来扩展Scrapyd，可以使用此选项。

application = scrapyd.app.application

launcher    = scrapyd.launcher.Launcher
> 代表到达scrapyd的接口的扭曲网络资源。 Scrapyd包含与网站的接口，以提供对应用程序Web资源的简单监控和访问。 此设置必须提供扭曲的Web资源的根类。

webroot     = scrapyd.website.Root

[services]
schedule.json     = scrapyd.webservice.Schedule
cancel.json       = scrapyd.webservice.Cancel
addversion.json   = scrapyd.webservice.AddVersion
listprojects.json = scrapyd.webservice.ListProjects
listversions.json = scrapyd.webservice.ListVersions
listspiders.json  = scrapyd.webservice.ListSpiders
delproject.json   = scrapyd.webservice.DeleteProject
delversion.json   = scrapyd.webservice.DeleteVersion
listjobs.json     = scrapyd.webservice.ListJobs
daemonstatus.json = scrapyd.webservice.DaemonStatus

> Scrapyd可以管理多个项目，每个项目可以上传多个版本，但只有最新版本才会用于启动新的蜘蛛。

用于版本名称的常用（且有用的）约定是您用于跟踪Scrapy项目代码的版本控制工具的版本号。 
例如：r23。 这些版本不是按字母顺序进行比较，而是使用更智能的算法（相同的distutils使用），
因此r10比r9更大。

Scrapyd还同时运行多个进程，将它们分配到由max_proc和max_proc_per_cpu选项给定的固定数量的插槽中，
从尽可能多的进程开始处理负载。

除了调度和管理流程之外，Scrapyd还提供JSON Web服务来上传新的项目版本（如鸡蛋）和安排蜘蛛。 此功能是可选的，
如果要实现自己的自定义Scrapyd，则可以禁用该功能。 
如果您熟悉Scrapyd实施的Twisted Application Framework，那么这些组件是可插入的并且可以更改。

daemonstatus.json
检测服务器的负载状态
curl http://192.168.11.11:6800/daemonstatus.json

addversion.json
将版本添加到项目中，如果项目不存在则创建该项目
curl http://192.168.11.11:6800/addversion.json -F project=tutorial -F version=r23 -F egg=@tutorial.egg
Scrapyd使用distutils LooseVersion来解释您提供的版本号。

schedule.json和listspiders.json允许您明确设置所需的项目版本。

schedule.json
安排蜘蛛跑（也称为工作），返回作业ID。
curl http://192.168.11.11:6800/schedule.json -d project=tutorial -d spider=somespider

传递蜘蛛参数（arg1）和设置（DOWNLOAD_DELAY）的示例请求：
curl http://192.168.11.11:6800/schedule.json -d project=tutorial -d spider=somespider -d setting=DOWNLOAD_DELAY=2 -d arg1=val1

cancel.json
取消蜘蛛跑（又名作业）。 如果作业正在等待，它将被删除。 如果作业正在运行，它将被终止。
curl http://192.168.11.11:6800/cancel.json -d project=tutorial -d job=6487ec79947edab326d6db28a2d86511e8247444

listprojects.json
获取上传到Scrapy服务器的项目列表。
curl http://192.168.11.11:6800/listprojects.json

listversions.json
获取可用于某个项目的版本列表。 版本按顺序返回，最后一个是当前使用的版本。
curl http://192.168.11.11:6800/listversions.json?project=tutorial

listspiders.json
获取某个项目的最后（除非被重写）版本中可用的蜘蛛列表。
curl http://192.168.11.11:6800/listspiders.json?project=tutorial

listjobs.json
获取某个项目的挂起，正在运行和已完成的作业列表。
curl http://192.168.11.11:6800/listjobs.json?project=tutorial

delversion.json
删除一个项目版本。 如果给定项目没有更多版本可用，该项目也将被删除。
curl http://192.168.11.11:6800/delversion.json -d project=tutorial -d version=r99

delproject.json
删除一个项目及其所有上传版本。
curl http://192.168.11.11:6800/delproject.json -d project=tutorial
