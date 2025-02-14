# 第07课：基于 Django_crontab、Xadmin 做一套定时任务管理系统

本课时介绍一个定时任务系统 Jcrontab，它用 Python3研发，并用到 Django_crontab 和 Xadmin 等模块。



我们知道在 Linux 环境下，crontab 是一个周期性的任务服务，我们可以根据它的规则来设定系统性的周期性任务。使用 crontab 时，想必你会遇到一个痛点，就是 crontab 需要通过命令行方式进行管理，比较烦琐，并且没有界面化的交互，通常作单机上的定时任务。



而 Jcrontab 是基于 Python3.6 和 Django1.8 版本开发实现的一套前后台界面化的定时任务管理系统，它可以打通企业间的消息服务接口来发送消息。

## 工程介绍

这个工程一共分为 3 个子系统，第 1 个子系统为前端任务系统，它主要记录单个任务的执行情况，如执行的时间、执行任务的结果，等等。它同时也可以控制单个任务的一些属性，使我可以对单个任务进行启动、停止或者修改任务周期。



第 2 个组成部分是后台管理系统，后台管理系统主要是基于 Xadmin 框架搭建的。它主要用于编辑、添加或删除任务。



第 3 个组成部分就是脚本录入系统，它是基于 Python 开发的一套脚本，主要是为了方便运维人员在控制台能够快速的通过脚本方式录入任务。

## 工程演示

接下来我们来演示一下这套系统。



首先进入控制台，然后到代码目录下启动工程，并且让它监听 8000 端口，同时打开浏览器，在浏览器的 URL 地址栏里面输入 IP + 端口号（8000），访问路径为 Xadmin。这样就可以进入后台的登录窗口页面。



登录后，我们会看到 Jcrontab 的后台内容，点击任务表，会看到记录所有任务内容，我们看到这里已经有一个任务在里面，如果需要新建任务的话，你可以点击新建任务。



我在原有任务的基础上来进行演示。在这个任务里我设置了它的启动时间、截止时间和工作周期。如果说我需要临时改变周期的话，可以建一个催单工作周期，同时有工作责任人、工作内容等。



我们看到这里有一栏是企业微通知 URL，它作为一个通知的接口进行调用，也就是说当我们执行完这个任务，程序会调用 URL 去发送消息。



接下来的是任务当前的状态、是否开启催单这两个选项。另外，在任务或命令这里会看到一个执行命令，我这里让这个任务执行的是一个 echo 命令，并且在终端把这个内容输出。



最后一项是工作模式，这里一共有三种模式，如果你只想让它发送信息，可以只选信息模式。如果既要发送信息又要同时执行命令，可以选择信息和命令模式。我这里选择的是信息和命令模式。



点击保存，这样我们就完成了一个定时任务的创建。



接下来看单个任务的前台页面。我们访问的地址同样是 IP + 端口，后面加 taskID，那么 taskID ID的数值为多少呢？我们可以看一下，这里的 ID 是 1，所以我在 URL 地址栏里面直接写的是 task ID 1，然后回车，就可以进入前台界面。



前台界面会展示之前执行这个任务的一些历史记录，同时在控制台可以做相应的管理。



这里我点恢复， 点击之后当前页面的一个状态就会由结单状态变成运行状态。与此同时在前台界面，我们看到当前任务的执行周期是每两分钟执行一次，我们可以等两分钟左右的时间来确认这个任务是否有执行。



刚刚我们讲到了 ，我的任务是信息加命令行模式。因此它的信息会发送到我的企业微信接口。那么企业微信接口的地址是怎么创建的？



首先我们在企业微信的工具里面创建一个群，在这个群里添加一个聊天机器人，机器人里有一个 WebHook 的地址，这个地址可以作为 URL 请求调用的地址，应用程序调用它我们就可以在这个群收到发送的信息了。



可以看到这就是我所发送的消息内容，包含通知需求内容和时间，我们也会看到执行的命令是什么。要通过控制台控制这个任务，可以直接点链接，也通过后台链接进入管理后台进行管理。



另外一块就是执行命令的结果。执行命令的结果会以信息的方式输出到群消息里面，同时还配有一张可爱的图片。这说明我的周期性任务已经执行了，并且已经发送消息，我们可以在消息里看到执行命令的结果。

## 基础知识

接下来我们来讲一讲学习本课时你所需要具备的一些基础知识。



通过演示我们看到了 Jcrontab 是具备前后台的一套完整系统，所以一定的代码开发工作。如果你想要详细了解整套工程，那么你需要具备 Django 和 Python 及部分前端的开发能力。



如果你只是想对 Jcrontab 的开发模式和开发思路进行了解，那么你只需要熟悉 Linux 的 crontab，同时具备一些 Python 和 Django 的开发基础，就可以继续学习了。

工程部署、安装及启用

接下来，我给你介绍 Jcrontab 工程如何部署、安装及启用。



首先，我们来看如何安装基础环境，课时开始讲到技术栈的时候有几个关键点，首先我们需要安装 Python 3.6，同时需要通过 Python3.6 安装它相关的模块。我们知道 Django 是 Python 开发的一个框架，而 Django_crontab 是属于 Django 的一个模块，在工程的 requirement.txt 中说明的所有的模块都需要我们去进行安装。



另外就是需要安装基础服务，这里用到的一个基础服务是 MySQL，所以你需要安装 MySQL 5.7 版本。



具体安装方式请看下篇文章（http://imoocc.com/jeson/2020/03/07/jcrontab），你可以通过课时最后URL地址查看并下载 Jcrontab源代码。

## 安装步骤

详细的安装步骤我就不具体介绍了，大体来说一下几个核心的步骤。



在安装完数据库以后，首先需要新建 MySQL 建立用户和数据库名等内容。 建好数据库以后，需要修改 Django 的 settings 文件来配置它的数据库连接信息。这里需要将新建的用户名、密码及数据库名对应在 settings 文件中修改。其次我们需要初始化数据库，初始化数据库也是遵循 Django 框架进行的模型初始化。



在初始化数据模型以后，接下来需要做的就是创建 Xadmin 的后台用户密码。你可以在我给的这篇文章链接（http://imoocc.com/jeson/2020/03/07/jcrontab）按照Django_admin 的方式创建一个 super 用户。 以上就是整体的安装思路。



下面我来为你介绍如何启用工程。对于 Django 的启用方式，我们知道是通过 Python manage 方式启动的，这里会添加监听地址及端口，尤其是在使用调试模式的时候，直接监听的是本地的 0.0.0.0地址，并且是 8000 端口。



如果你没有企业微信环境，可以使用我提供的专门建立的一个用于测试的群环境，首先下载一个企业微信 App，然后扫码入群，就可以用这个环境进行测试。

## 代码目录结构

代码下载下来以后，我先来给你介绍一下整个代码目录的结构。



推荐大家用 PyCharm 工具打开工程，如截图所示，我们会看到有一个文件目录结构，

![](/static/image/Cgq2xl5nIHuANsKGAAPpqjib5LM425.png)

目录结构从上往下是这样子的：首先是 extra App，这里存放第三方的模块，工程后台使用 Xadmin，所以需要把 Xadmin 模块也集成到工程中。



Jcron 目录里面是 Django 的一些相关配置文件，如：settings 配置文件等，都是遵循Django 框架模式存放的。



Jcrontab 里面是放的是 Django 的相关应用，这个也遵循 Django 框架的目录结构。



logs 是存放日志的目录，这里新建一个专用于存放日志文件目录，会把相关的日志输出到目录下。



static，static 目录下存放相关的静态文件，前台页面的图片 CSS、JS 等都会存放在 static 下。



templates 里存放 Django 的模板文件，主要是一些 HTML 的页面模板。



manage.py 是 Django 框架的管理文件，我们在安装的过程中，需要用到 manage.py 去建立



super 用户，并且做模型的初始化，启动也是通过它来执行 manage.py runserver 启动工程的。



requirements.txt 是一个模块列表，我会把整个工程所依赖的相关模块，以列表的方式存放在requirements.txt 中。你安装的时候需要从 requirements.txt 中提取相关的模块。所以它是一个 Jcrontab 工程所需要用到的安装模块的一个介绍说明。

## 代码框架结构

接下来我们再来了解整个代码的框架结构。这个工程用到的是 Django 框架，我们知道 Django 是一个基于 MVC 或是 MVT 模式的框架。



M 是指模型，V 是视图，T 是我们刚刚讲的目录结构里面的 templates目录，它里面存放的是相关的模板。



在下图中，我们可以看到 MVT 大概的框架模式。当用户请求过来时会通过三个大的层次，分别是 URL、Views 还有后台的 models。

![](/static/image/Cgq2xl5nIHyAM5EnAAb-zfht2e4708.png)

URL 主要用于做 URL 请求地址的路由。Views 主要是为前端请求的接口来实现具体的视图逻辑。Views 后面还会有一层，这里面是我的一些核心请求方法或类，可以通过 Views 层来直接调用。



MVT 中的 M 就是 models，models 表示数据库模型，所以我们会看到 Views 层和核心方法层，都会调用数据库模型，因为数据需要从数据模型中获取和写入。



需要注意中间这一层，我加入了一个 Xadmin ，你看图中如果用户请求的路径是 /xadmin 的 URL 地址，就会交给 Xadmin 的框架模块。



整体上，当用户请求不同的 URL 地址时，首先会通过 URL 层基于请求路径做路由，然后给不同的视图进行对应的逻辑处理，最后会调用到相关执行方法及数据库模型。这就是整个工程的一个框架结构。

## 数据模型

刚说到了 Django 的 models，我们知道 models 主要用于定义数据库的相关模型，那么我的数据库模型是怎么来定义的？这里给你来介绍一下。



整个工程主要用到两张表，一张是 Demandorder 表，一张是 Demandorder_log 表。



Demandorde 是主表，会记录每一个任务的相关属性，如启动时间、截止时间、工作周期、催单周期，以及工作责任人，还有调用企业微信的 URL、任务状态、是否开启催单、任务命令和工作类型。这就是围绕一个任务来设计的表。



另外，它有一张外键表 Demandorder_log，外键表主要用于记录每一次执行任务的结果和执行每一个任务的时间，这里单独用了一张表来存放这些结果数据。



以上就是我在 Django 的 models 里面做的模型设计。

## 特色知识点

刚刚讲到的这一块是整个工程里面主体的代码开发框架的一些内容。接下来我为你介绍工程实现两个特色知识点。



虽然我们看到整个定时任务调用了操作系统的 crontab 服务，但是框架程序内部又是如何来调动这个服务的呢？这里就用到了 Django_crontab（第 1 个特色知识点）。



Django_crontab 是 Django 框架中的一个定时任务模块，我们可以在 Python 里面通过 pip 的方式去安装 Django_crontab。安装完成后，需要在 Django 的 settings.py 文件中加载 Django_crontab 应用，这里在 INSTALLED APP 里面先把 Django_crontab 加载进去。



接下来就可以具体配置标准的 crontab 执行方式：先在 settings 里设置定时任务，这个定时任务用 CRONTAB 变量赋值一个列表数据，列表里面是一个元组，每一个元组就是一个定时任务内容，如这里我规定了以每 5 分钟的执行周期去执行某一个函数，这是一个写死的定时任务。



一个标准 crontab 任务配置好后，是按照如下方式去管理的的：通过 manage.py 管理文件来进行定时任务的加载、删除和展示。通过 Python 执行 manage.py 及对应的选项。其中，crontab add 表示添加 settings 里面刚刚添加的定时任务；show 表示展示当前 crontab 里面有哪些定时任务；remove 表示删除刚刚设置的 settings 里面的定时任务。



按照标准模式来配置的话，我们需要把所有的任务都以这种固定的任务方式配置在 settings 里面。但是由于我们的工程需要做到前后台的联动，并且动态的加载，改变任务周期，这个时候我们就需要在标准的模式上做一定量的改动。



这里给你看看我的做法。我把 settings 里面的 CRONTAB 全局变量改写成了调用 todo() 方法，todo() 方法会每次加载的时候执行一次，把模型中任务属性按照 crontab 模块的要求生成任务列表数据，然后赋值给 crontab 变量。这样的话就可以做到和数据库直接进行交互，而不用把每一个命令以固定的方式写在 settings 配置文件里面。



第 2 个特色就是要使前台和后台做到动态的交互。在演示过程中，当点击前台的催单或者恢复按钮时，每一次点按钮都需要重新加载定时任务（crontab），这就需要用辅助工具来实现。这里我会在 Django 里面再调用一个 Shell 脚本，这个 Shell 脚本会把之前 crontab 模块里面已经有的任务进行 remove。然后再执行一个加载，把改动的任务加载进来。有了这个脚本就可以通过前端按钮来调用脚本，相当于操作直接 reload 任务。



最后值得一提的，就是在这个工程里面我用到了一个 Xadmin 模块。如果你了解 Django 就知道 admin 模块是 Django 里面默认管理后台的插件，Xadmin 是在 admin 的基础上开发的一个第三方管理后台，它的功能比 admin 更加强大。但是你在使用的时候一定要注意到版本问题及与之相关联的插件问题。如果你对 Django 或者 admin 模块还不是非常熟悉，建议你遵循我介绍的版本和安装方式，这样的话你就正常使用xadmin管理工程。