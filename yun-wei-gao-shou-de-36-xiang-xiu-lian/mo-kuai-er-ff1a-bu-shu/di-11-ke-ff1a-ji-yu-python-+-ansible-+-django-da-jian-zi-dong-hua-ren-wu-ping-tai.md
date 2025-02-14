# 第11课：基于 Python+Ansible+Django 搭建自动化任务平台

在上课时我们讲解了一套 Devops 理念的工程它的 CMDB 是如何实现的。CMDB 可以把所有的资产做一个收集。有了资产，就好比一个仓库里面所有的货物已经准备好了，那么接下来该怎么去进行分门别类的管理和维护等操作呢？这个时候就涉及自动化的任务。



本课时我们就来讲解 Devops 工程剩下的部分：自动化任务平台是如何实现的。 



这一套自动化任务平台工程里主要用到了 Ansible 模块，我将先为你介绍 Ansible 模块的基本功能，然后再详细介绍 Ansible 模块和 Devops 工程是怎么进行融合的。

## Ansible 模块

Ansible 是系统中的一套自动化工具，它可以支持在 Linux 终端进行命令化的执行，同时它也是 Python 里面的一个模块，可以通过 Python 进行模块化的调用。Ansible 主要用来实现系统的管理、自动化的命令等任务。



Ansible 的应用通常有两种方式，第 1 种是 adhoc（命令模式），第 2 种是playbook（剧本模式）。这两种方式存在差异，命令模式主要以模块的方式运行 Ansible 任务，而 playbook 是一个剧本，相当于有一个写好的命令化执行记录文件，playbook 只需要按照这个文件执行要求来作相关任务就可以了。playbook 比 adhoc 的场景更加丰富，而且运维管理更加方便。



在 playbook 的基础上，如果你想要将 Ansible 应用得更加灵活，如有整体的工程化结构管理或者是需要更大的运维自动化场景需求，我们可以通过 roles 方式来做 Ansible 的任务。roles 是基于 playbook 的一套丰富工程化结构使用方式，所以它比一般的 playbook，更适合在工程化场景所使用。

## adhoc

讲到了 Ansible，我们先来了解安装 Ansible 必要配置的两个文件，第 1 个是 ansible.cfg，它是一个全局性的配置文件，可以用来配置 Ansible 全局性的一些变量和设置。 第 2 个是 hosts，这个文件主要用来配置 Ansible 的主机资产关系管理信息。通过 Ansible 来做自动化任务需要在 hosts 里面配置它主机相关的资产管理信息，如 IP 、SSH 的连接信息、主机名、主机组分类，等等。



我们刚刚讲到了 adhoc 模式，它在终端执行时是这样的：`ansible  <host-pattern>  [options]`。举个例子，这里我用 ansible 命令执行，对 testsever1 这台主机执行任务，-m 表示它执行什么样的模块化的任务； shell 是 Ansible 和 adhoc 里面的一个模块，-a 代表执行 shell 里面的哪个命令及参数；touch /tmp/jesonc.txt 代表它在执行创建新的文件。

## playbook
![](/static/image/Ciqah155wHCARXKPAAEGxYJzasw197.png)

playbook 是 Ansibe的一种模式，需要根据 yml 及 palybook 语法要求写一个剧本文件，这个剧本文件以 yml 作为格式，我们可以看这样的一张图，在执行同样一个我刚讲的任务，它的剧本展示如上：在这里定义主机、执行的对象，执行的用户、定义一个变量。



tasks 是 playbook 具体执行的任务，这里任务是执行 shell:`touch /tmp/{{touch_file}}` 文件。我们会看到用 touch 命令添加的文件是可以用变量 (touch_file) 的方式来引用。在做一个更大规模的自动化任务时，用 playbook 整体做会更好，因为它可以把很多东西抽象用剧本方式管理。



这就是 playbook 的执行方式：ansible-playbook+ 写好的 playbook 的配置文件（也就是刚刚这张图里面的内容）+选项。

## roles

刚讲到了 adhoc 和playbook 它们的执行用例，我们再来讲一讲 roles。 roles是在一个大型的任务场景里，执行一个整体、工程化任务。

![](/static/image/Ciqah155wHCAXLU8AAFRHO4gLxc907.png)

我们会看到 roles 有一个特定结构的目录结构（如图所示），接下来我们来讲解下这个事例结构：



首先我们看看files 这个一个文件目录，这个目录里面可以存放要执行自动化任务里需要上传的文件内容，比如说我们的源码包、工程所需安装文件等。



production 主要是存放线上环境下的配置文件，比如 Tomcat 的 server.xml 配置文件。



接下来我们来讲一下 roles 目录，它主要存放定义角色，举个例子，如果我们部署 Java 的 Web 服务，那么整体工程需要前端通过一台 Nginx 做代理，或者我们要部署 JDK 和 Tomcat ，它们就是三个角色，这些角色都有各自所需要执行的自动化任务，可以在 roles 这一节目录里定义各自的 playbook 相关任务。所有角色都可能需要调用到的一些公共文件，我们可以存放到 roles 的同级目录中。



production 表示的是线上环境配置环境，而 staging 则是线下的使用环境，它们的主要区别在于存放配置文件针对的环境不同。



templates 里面主要放一些模板和相关的配置，我们可以把一些 HTML 的模板或者模板配置。



最后一个是 webserver.yml文件，他是一个主文件，我们在部署这样一套自动化工程时，就可以通过`ansible + webserver.yml` 整体执行 roles 剧本。



以上就是整个 roles 的样例，我们会发现 roles 这种模式相比 单纯的playbook 来说，更加适合在大的自动化运维项目中使用。Ansible 这三种默认的方式虽能提升自动化任务效率，但不能满足我们所有场景。你可以思考一下：



第 1 点，我们要考虑把已有的 CMDB 平台和 Ansible 做融合， 如果通过 Ansible 的hosts、groups 配置作主机管理，那它该怎么和已有的 CMDB 系统融合呢？ 



第 2 点， 无论是通过 adhoc、roles 或者 playbook，它们都是通过终端的方式去管理，这样的方式依然非常烦琐，在更大的工程下面还是需要考虑如何进行可视化的展示和管理。



第 3 点，如果你要更加贴近于你自己公司的架构和业务系统，就要避免做一些自定义场景功能的定制和性能的优化。但这个时候如果完全通过 Ansible 的默认模式来做的话，可能就具备局限性了，这个时候我们就需要考虑自己来构建一套自动化任务执行平台。
## 实现自动化执行平台


接下来我来讲一下这套自动化任务执行平台如何来实现？



在学习这个部分之前，你需要了解一些 Ansible 的使用基础、 Python 开发基础以及 Django 的一些基础。

### 工程设计理念

接下来我们来讲解这个工程的设计理念是如何实现的。我们先来讲解工程化理念中后台设计这一部分，我们会看到用户端整体请求的一个过程，首先用户端通过浏览器发送 POST/GET 请求，也就是把需要执行的任务发送给后端，由后端来具体执行，这里会看到最右侧用到了三个数据库存储服务，分别是 MySQL、 Redis 和 Mongo。

![](/static/image/Ciqah155wHCARtpTAAHVwx-gTak127.png)

绝大部分关系型数据都会放在 MySQL 中，Redis 在这里主要作任务锁的作用，自动化任务场景中不希望任务重复执行，或者说我在执行一个任务的时候，不希望有再次调用，就会把它锁在 Redis 里面。最后是 Mongo，Mongo 主要用来记录执行的日志，这会方便溯源，当我们要查找某一个自动化任务执行的过程在哪一个位置出现问题，也可以通过 Mongo 日志直接来进行分析。



在这个主体工程中，第 1 层就是 API 接口层，它用于接收用户的请求，并且调用核心层的具体逻辑。



这里的逻辑层是用 Django 框架开发，通过它开发了前端的 API 层和核心层，同时会调用底层的 Ansible 模块，Ansible 本身可以作为 Python 的模块。



最后是输出并执行任务，工程会从 MySQL 里面获取主机资产信息，然后对这些目标机器的对象执行自动化任务。
### 工程技术栈

前面的这张图示里，我们大概介绍了从用户请求自动化任务到中间逻辑的执行，需要调用哪些服务。接下来我们再来讲一讲，我的这套工程整个技术栈的使用。



这套工程是基于 Python3.6 和 Django 1.8 开发的，ansible用到了 Ansible2.4.1 版本。演示环境用到的是 MySQL 5.7 和 Redis4.1 版本，同时你需要一个开发工具，这里给你推荐 PyCharm，你可以用 PyCharm 工具打开源码，源码请在课程最后的链接里进行下载，安装方式请参考第 10 课时，这里就不再多讲了。

![](/static/image/Ciqah155wHGAdMFQAADEpb7Dwsw655.png)

下面我们来看下自动化部分的代码内容，在整体的任务调用到服务端以后，在 urls.py 文件里定义了一个对外的 API 接口路径，即 /adhocdo 接口。

### 核心类功能

views.py 视图里面定义了一个 adhoc_task() 函数，它会来负责接收请求，并且执行对应的逻辑。同时，在这套工程里面把前端发过来的任务请求、资产信息关联起来，当它在调用底层的 Ansible 接口时，也需要进行一次封装，这个会在 ansible_api.py 文件里把 Ansible 模块的默认内核方法重做封装提供外部调用。
![](/static/image/Ciqah155wHGAJLotAAFmNJ-edRA051.png)

刚讲到的需要封装 Ansible 类，那么具体封装了哪些类呢？在 ansible_api.py 这个文件里面包含 4 个大类，一个是 MyInventory()，另外一个是 ModelResultsCollector(CallbackBase)，还有一个是 PlayBookResultsCollector(CallbackBase)，最后是 ANSRunner(object)。



我们从上往下来进行介绍， MyInventory() 是把 Inventory 默认的 Ansible 类做了一个定义，它定义了主机和主机组的相关关系。



ModelResultsCollector(CallbackBase) 把按照默认的回调类 (CallbackBase) 重新做了封装，修改一些任务的显示和返回的内容，并回执 adhoc 任务成功失败的状态，都是在这个类里面进行封装。



PlayBookResultsCollector(CallbackBase) 跟前面类是一样的，只不过它封装的是playbook 类，同样也会返回相关任务的执行状态。



最后一个就是 ANSRunner(object)，这是一个任务执行类，我们在视图的 adhoc_task() 里调用 Ansiblerunner 来执行它的具体化任务，所以它是直接对外暴露执行方法。

### 工程执行过程

最后我来给你演示一下整个工程的执行过程。

![](/static/image/Ciqah155wHGAVPdAAADGi3JVaHk602.png)

刚讲到了对外暴露的是  /adhocdo  的接口地址，现在我们来请求这个地址，并且按照它的参数要求来进行请求。这里用 POST 方法，提交的是 taskid 这个参数，主要是任务的 ID。mod_type 主要是模块的类型，因为 adhoc 这个任务是需要以模块的方式来进行任务执行的。执行的参数就代表具体的任务命令了。sn_key 对主机起到唯一标识的作用，我们需要对哪一台主机就用 sn_key 来对应的执行。

![](/static/image/Ciqah155wHKAPwtfAACH5Q2NW3o196.png)

这里有具体的参数，我用 JSON 的格式提交的任务， JSON 的格式是这样的，举个例子，我们会看到这里有执行的任务 ID，然后类型是 shell，执行的命令是 touch 一个文件，group 它执行的是主机组（可选择不加）。最后 加入机器的sn_key。

![](/static/image/Ciqah155wHKAO35aAAGx4BbLhL0386.png)


接下来我来演示一下 IMOOCC自动化任务执行的过程。在上一个课时里面我们讲到了 IMOOCC 工程对自动化资产的收集。有了上一课时的基础，我在本地的这套工程已经收集到了对应的资产信息，如本地的这台输入主机的信息就已经在我的这套资产管理系统里面直接提取。

![](/static/image/Ciqah155wHOATKf0AATlRJB7cOI285.png)


提交的Json格式如图所示，exec_args中执行命令的方式是 touch 一个文件，我这里执行命令是在/tmp目录下touch 一个 test22 的文件。sn_key 是唯一主机的标识，我们可以在资产管理系统里面点击详细这一部分来提取出 sn 号，这个就是它的唯一标识。

![](/static/image/Cgq2xl55wHOAH6vtAAMlclXaedE645.png)

mod_type 是直接执行 adhoc 的模块方式，这里我执行的是 Shell 的命令。



接下来，我在 Chrome 下使用模拟 post 请求客户端插件 PostMan，来模拟提交这个任务到工程的自动化任务 API 接口，并开始执行。


![](/static/image/Cgq2xl55wHOACyO2AAXukl_6PSE019.png)

大家按照 JSON 的格式配置好后，可以参考设置的方式，然后点击 send按钮，这样的话就会把我的数据进行 POST 提交，我们会看到它这里已经在 loading 中，需要等待服务端返回的结果。


![](/static/image/Ciqah155wHSAPhPbAANojZQK3YM533.png)

当服务端返回了内容，看到 http reponse 的状态是成功的，并看到信息"sucess"（成功的），并看到执行此任务具体日志（它是ansible的接口所返回的）。

![](/static/image/Ciqah155wHSAYKeEAAJCjs_ILvU244.png)


然后通过另外一种方式我们再来验证是不是成功，可以直接 ssh 登录服务器主机上，然后进入 tmp 目录，输入 ls，会发现 test22 这个文件在我录制视频的时间点有对应生成这样的一个文件，说明自动化任务是已经执行成功了。



通过这个课时我们能够理解 Ansible 如何融合到 Devops 里面，以及在这个过程中对应需要做的一些开发工作。



本专栏课中的所有案例配置及源代码，你可以课后通过这个地址 http://www.jesonc.com/jeson/2020/02/07/ywgs36/ 自己下载，密码为：mukelaoshi。