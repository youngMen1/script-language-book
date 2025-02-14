# 第06课：系统性能验收：Unixbench、FIO 性能压测

本课时给你推荐两款系统性能验收测试工具，分别是 Unixbench 和 FIO。 系统验收和性能测试常用于整体业务部署前，是考核性能是否达标的一个关键环节，所以我们获得一台服务器时就需要对它的整体性能做一番了解，以便评估它是否合适部署服务和对应的业务。



掌握本课时的内容，建议你课前先了解 Linux 的一些使用基础及操作系统原理。

## 测试前关注点

![](/static/image/CgpOIF5gtPGAQgQyAAbHfhNzTM4078.png)

通常在对一个系统进行基准测试前，先了解其用途和系统本身的情况是十分必要的。



对于操作系统，我们本身并不陌生，在做系统性能测试时，需要重点关注引起性能变化的设置是否需要优化。如服务器 CPU 功耗性能管理设置， p-state 和 c-state 状态值是不是都已经调到最优了，尽可能的把系统性能、CPU 的计算性能发挥到最优。



另外影响系统性能参数是否已经优化也是需要关注的，如系统内核的一些参数（包含文件打开句柄、网络队列大小等）。当然对于系统性能参数的具体内容，我会在稍后的一个单独的课时，给你详细讲解。



另外一个方面，我们也要了解系统本身的缺陷（驱动模块、开发库缺陷、补丁等）。我们来举一个例子，曾经我给系统打了一些熔断、幽灵安全补丁后，就发现因此导致操作系统性能损耗增大，所以了解这些有助于我们分析结论和提高系统能力。



这都是我认为你在做系统性能测试前，需要对系统本身累积沉淀的两个方面内容。



除此之外，就是需要了解业务需求情况，为什么需要了解业务需求情况呢？因为系统最终需要承载业务。所以这个时候就需要了解业务特性是什么样子的，特性主要体现在业务有没有高峰期和低峰期，它什么时候需要承载多大的 QPS 等。



然后，对于业务就要结合考虑它的资源利用类型，是运行内存型的服务还是运行计算型的服务。你需要知道服务架构大体是什么样子的，假设部署 Nginx，那就需要了解部署反向代理服务？还是需要做缓存服务？或者只是作为一个静态元素的 Web 服务使用？等等。



了解了业务的整体情况，我们心里就可以有了一杆秤，让测试结果更有针对性，我们就不只是在作执行，而是在向更高一层考虑问题。

## 章节思维导图

![](/static/image/CgpOIF5gtQiADB-UAASwQDk9GjA321.png)

和运维相关的性能测试这里主要归纳为四大类，分别是业务压测、服务压测、网络压测、系统压测。



所谓业务压测，比如一个 Web 程序服务，我们想对某一个接口模拟海量请求压测，这时就可使用 AapcheBench(ab)\WebBench 等工具来模拟业务用户端请求。



另外就是服务压测，比如使用 Sysbench 来压测 MySQL 服务指标。



网络压测这里单独列出来，是因为需要对网络的带宽和承受能力进行压测，比如使用 Netperf 工具进行网络压测。



最后系统压测是本课时给你重点介绍的知识点，一个完整系统测试主要涵盖这几个组成方面：一是硬件，物理单元是由 CPU、内存、磁盘、网卡等几个核心组件组成。



其中标红色字体的 CPU、内存、磁盘是硬件系统压测需要重点观测的指标，因为这三者很大程度上决定了很多服务的承载能力，影响整体业务的性能。



另外两个组成为操作系统本身及相关开发库，在系统压测的时候，其实就是对硬件、系统、开发库这三个组成部分进行测试。

## Unixbench 使用

接下来我们介绍系统压测，本课时我要给你重点介绍 Unixbench。



Unixbench 是一个基于系统（Linux、Unix）的基准测试工具。什么是基准测试呢？所谓基准测试是给出一个通用标准基准的分数值，在它基础上对其他操作系统进行打分，用于衡量这个操作系统到底应该给多少分，和基础的差异有多大。



那我们怎么知道这个分数是高还是低呢？就像老师给很多学生进行一个考试，对考卷来进行打分，考试就有标准，大于 60 分是及格，大于80 分或 90 分是优，所以这地方也有一个衡量的标准，这个衡量标准就是一个基准。



Unixbench 的基准分是基于 1995 年一个叫作 George 的基准测试操作系统。它有一个工作站，但是配置比较低，大概 128 MB 内存，使用 Solaris 系统，然后做了一番压测，它的压测数值检测结果是 10 分，这就是它的基准测试的分数。



当前，操作系统和硬件发展很快。再用 Unixbench 去压测的话，会得出更高的分值，我们可以拿这个分值和它的基准分值去做对比，这样就可以非常形象地进行对比分析，同时还可以进行横向的对比，比如和其他服务器结果作对比，都可以参考到这个基准分数。



Unixbench 工具需要手动下载，你可以到 GitHub下载，安装成功后用 run 命令启动基准测试。run 命令后面可以选择添加几个参数：`-q、-v、-i <count>、-c <n>`。

* -q 不显示测试过程。

* -v 显示测试过程，也就是把测试过程是否显示做了一个控制。

* -i `<count> 执行次数`，最低 3 次，默认 10 次，也就是控制最大每一次基准测试项到底执行多少次， Unixbench 可以对很多项操作系统能力打基准分，总的分数值是由很多个测试项目综合得分组成，每一个测试项它测试了多少次，就是由 -i 加数字的方式去控制的。

* -c 是在多核 CPU 场景测试指标，可以了解到操作系统上多核 CPU 的性能程度。比如一台机器是四核 CPU，这个时候设置为 -c 4，也就是运行 4 个 Unixbench 任务同时去执行基准测试打分，这样我们就能更加客观的得出，在 4 个 CPU 同时被使用的时候，大概的一个分数会是什么样子的。

以上就是安装完 Unixbench 以后，run 命令后面所默认可以加的参数。



这是一个执行示例，我们可以用 ./Run 直接来运行，等待结果需要一段时间，如果你不能等待那么久可以选择后台运行，后台的运行方式我们可以选择使用 nohup 直接运行。这种方式在前面直接加个 nohup，后面加一个 & 符号，就可以直接把 run 方式放到后台中运行了。

![](/static/image/Cgq2xl5gtSGACDIrAAWj5rI_m3s047.png)

对于刚讲到的，Unixbench 有很多个测试项，那么分别都有哪一些测试项呢？



这里我列出了它的相关测试项，你可以看到它所测试每一个基准测试项，比如，它可以测试浮点的超时效率；还会测试 Excel 每秒可以执行的系统调用，也会测试管道的直接吞吐能力，等等。



对于 Unixbench，在使用完以后大概的执行结果会是这样子的，分别是：

* 第一是展示测试进度；

* 第二是系统信息的汇总，也就是它会摘取并展示操作系统当前 CPU 的一些基础的操作系统信息；

* 最后一个当然是我们更关注的测试结果。

测试结果会分为两部分：
* 如果你只启用了一个单副本的测试，也就是我们在刚刚讲到的，比如 -c=1 就是一个单核 CPU 的测试；

* 如果你是对多副本进行测试，也就是 -c＞1 时，它会分别列出单副本测试和多副本测试是什么样的测试结果。

![](/static/image/Cgq2xl5gtXGAB9S3AAH4TWbDc3k323.png)

这里再举一个案例，假设我们现在要去对一台 AMD 主机和一个英特尔处理器来做一个性能上的差异对比。那我选择了两台机器，前提是同样的配置，磁盘类型保持一致，操作系统也保持一致，只是 CPU 类型不一致。这种情况下，我们来对比最新一代的 AMD，也就是 Rome 这一代的 CPU 和 Intel 的 CPU 的性能差异。



那么现在就登录到了我的一台主机，这台主机上面就是我接下来所要测试的这其中的一台目标系统，在这台目标系统下演示分别在安装、使用 Unixbench 的大概情况是什么样子的。你在安装 Unixbench 的时候，可以选择在 GitHub 上直接下载并且安装，也可以使用我提供的脚本去进行整个软件包的下载、安装，并且最后直接去单独执行 run 命令，脚本的下载地址见本课时最后链接。



接下来，那么我用 sh install_unixbench.sh 直接去执行，这个时候它开始安装基础包，安装完成以后会去下载这个软件包，下载完成以后，它会执行 make，编译完成后开始执行 run 命令。我们会看到这里展示的是一个 logo，下面会提取信息，这里可以还看到测试的大概进度，当前测试的项是第一项。



在第一项中，如果没有加指定 -i 的话，最多测试 10 次。如果我们给 -i 指定的更多则可以测试更多次数。



这就是在操作系统上使用 Unixbench 的一个测试情况。由于测试时间较长，所以我这里就直接中断了。我们直接来看一下，在整个执行完以后大概会是什么样的结果。



这里记录了分别在 AMD 机型上测试的结论和结果、数据，及在 Inter 机型上测试的数值。我们看到首先列出每一项的数值是测试内容，以及所花费的时间。那么在后面的这一段里面会看到每一项的基线分数。



以上这些列出两个 CPU 在跑一个副本的时候的测试结果。



下面就是两个 CPU 的系统上启用两个副本的（-c 2）测试结果。同样是先测出每一项的数值、单位、所花时间。同样看到每一项测试结论、当前对系统测试打分的情况。那么我们同样来对比一下 AMD 机型和英特尔机型的差异。



我们就可以用 vim -d 把两个记录 Unixbench 测试结论的文件都打开，分析数值跑分结果并作对比。AMD 的跑分数值会比英特尔的 CPU 的分数更高一些，所以会发现最新一代的 Rome AMD CPU 的性能优于另外一款英特尔 CPU 。



Unixbench 测试项里面更多关注的是计算性能和内存的 IO 等场景。而操作系统除了需要重点关注这两个性能指标外，我们还想知道磁盘 IO 处理能力是什么样子，这个时候就需要对磁盘的 IO 进行一个测试，磁盘的 IO 往往很大程度决定了在操作系统上服务的性能，比如说 MySQL 等数据库属于高 IO 型的服务，这些都是需要用到磁盘的 IO，所以我们需要对磁盘进行一个测试，接下来介绍的 FIO 工具就是做这个用的。

## FIO 使用

![](/static/image/Cgq2xl5gtYiAI2f-AAJp6_7gP_I248.png)

FIO 是一个开源的主流的 Linux 磁盘 IO 测试工具。通常对于磁盘的测试主要会关注这几个测试项，分别是：Iops，也就是每秒进行读写（I/O）操作的次数，它评估的是，每秒对于磁盘的操作次数和能力，执行次数越多说明执行数据越快。



另外一个指标就是对磁盘的吞吐率，它表示每秒对于磁盘的读写数据量，单位为 MB/s，数值越高表示读写数据越多。



第三个指标就是读写延迟，表示单个 IO 去写磁盘，做一次 IO 操作的耗时是多少。



我们使用 FIO 这个工具可以针对这三项指标去进行测试，并分析结论，在使用 FIO 之前建议你尽量使用一台空闲的机器来运行，以免因为 FIO 对操作系统上的磁盘或者文件的读写而损坏文件系统，这个地方你需要注意一下。

![](/static/image/Cgq2xl5gtaCAGekxAAmbidRaDZI069.png)

关于 FIO 的使用，参数相比于 Unixbench 就更多了。我们会看到这里有很多的参数说明，值得重点注意的有：bs 这个参数表示块的大小， iodepth 表示队列的请求深度，我们可以理解为在请求 IO 时会有多少个并发，如果深度数值越大说明并发请求越多。



另外一个 rw，它可以控制磁盘的读写模式。我们知道磁盘分为顺序读、顺序写、随机读写、混合读写等相关的读写场景，如果你需要指定特定场景，那么可以通过 rw 的参数来设置。



这就是 FIO 的基础使用所涉及的一些参数，其他参数请你自己课后看一下。



我们接下来讲一下，对于 FIO 测试的时候，我们所需要进行的针对性的测试 IO 指标（吞吐率的测试结论，延迟的测试结论），那就需要我们控制好 FIO 测试的参数。



比如说我们想对延迟进行测试，这个时候我们会把它对应深度调为 1，也就是模拟一个队列对磁盘进行操作的时候，把块大小设置为 4k，这样的话就能够做到单个队列上读写的延迟测试。我们能够着重测出硬盘的延迟的指标。



rw 参数里可以指定磁盘读写类型，如果为 randrw 那就是作随机读写的延迟测试。



第二个就是做硬盘的吞吐测试。



硬盘的吞吐测试就是硬盘上最大的吞吐率带宽。需要尽可能的把总线带宽或者 IO 带宽跑满，那怎么做呢？



我们把队列调到更大（32），然后把 bs 也就是单个块的大小调到比较大的一个值（128k），这样的话就能够测试最大能力去跑满整个磁盘带宽，这样的设置更加能够反映出硬盘的吞吐能力。

        

最后一个就是 iops 指标测试。关注 iops 的话，做到的是要在单位时间内尽可能的多的读磁盘，所以我就会把 bs 这块的能力给它调小，但同时会把队列调到最大，这个时候才能够测试在单位时间一秒内能够读写操作磁盘数。



对于这三种性能指标测试，你都可以参考对应设置，来得出磁盘能力对应的结论。

![](/static/image/Cgq2xl5gtbaADDl1AAJw4GDZ46A089.png)

最后再介绍 FIO 的案例。主要测的就是我本地盘的 SAS、SATA、SSD 在性能上的差异性。这里我会拿三台机器，这三台机器只是本地的磁盘类型不同，其他的配置保持一致。通过 FIO 测试后会看到一个测试数据，这里分别会展示出每一个磁盘的 iops 及吞吐能力的大概情况。我只是测试顺序读写模式下我们可以看到 SSD 盘整个的吞吐能力和 iops 会比我使用 SATA 或者 SAS 盘的能力会更加的高，约高出 4 倍以上。



本课时给你介绍的两款性能压测的工具，你可以自己课后多练习。本专栏课中的所有案例配置及源代码,你可以课后通过这个地址 http://www.jesonc.com/jeson/2020/02/07/ywgs36/ 自己下载，密码为：mukelaoshi。