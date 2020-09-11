# 第34课：分析 Anycast 应用程度及场景

在前面的课程里我们有提过 Anycast 这个概念，本课时我们来重点讲解一下什么是 Anycast，它的优势是什么以及对应的案例介绍。

单播、组播、广播
在讲 Anycast 之前，我们先来了解一下网络通信里常见的几种通信形式。

第一种通信模式就是单播，单播是一对一的通信模式，两个实体之间相互间通信，如下图所示：

CgqCHl7y7AaAWhCJAADaSjK7P-4901.png

在这张图中，会看到有多个圆圈，每个圆圈代表一个实体，红色和绿色圆圈之间的箭头表示红色和绿色在进行一对一的通信。单播是在网络协议或服务里大多会采用的一种传播方式，这个是非常常见的。举个例子，这种形式就好比一个人和另外一个人在进行对话。

第二种通信模式就是组播，组播是一对多的模式，通过使用一个组播地址将数据在同一时间以高效的方式发往处于 TCP/IP 网络上的多个接收者。我们看到下面这张图：

CgqCHl7y7BeAPQ_tAAEXgA5hLBQ539.png

可以看到，图中有一个红色的节点，它同时发给了三个绿色的实体节点，这就是组播的通信形式。这种模式就好比你在大街上大喊一声“美女”，这个时候就会有一大群女性回头看你。

第三个通信模式就是广播，它属于一对所有的模式，广播里的每一台主机发送出去的信号都会无条件的复制和转发，通常这种使用模式需要限制在局域网内部进行应用。好比今天公司发一个通知，告诉大家全体可以休息半天，所以大家都会比较高兴。

Ciqc1F7y7CWADPR8AAFAo4N1AMU000.png

## 任播
接下来我们来讲任播是什么样子的。首先我们来总结一下单播和组播。对于单播来说，我们会发现网络地址和网络节点是一对一的关系，通信也是一对一的关系。而组播和广播就不一样，它们的网络地址和对应的网络节点存在一对多的关系，并且通信过程也是一对多的关系，接收者有多个节点，并且同时进行通信。

任播则区别于单播和组播，它则是网络地址和网络节点有一对多的关系，但在通信过程中却保持着一对一的关系，注意，这里是指在同一时刻的时候，保持一对一的关系，所以它结合了单播和组播两者的特点。网络地址和网络节点存在一对多的关系，但是通信过程中却保持着一对一的关系。任播模式通常应用在更大范围或者全局性网络架构中，大公网或者大规模网络架构中才应用到的一种任播方式。

CgqCHl7y7C6ATeCAAAFmj3UsAXs135.png

了解了它的接入模式之后接下来我们来具体介绍一下任播的优势。任播的优势是这样的：

第一个优势是就近访问，它提高了访问的速度。因为它可以有多组接入地址，用户可以基于接入的入口，选择最近的接入节点，从而减少用户请求上的延迟。

第二个优势就是它分担流量，提供了负载均衡的功能，因为任播同时有多种接收地址，那么不同地区的用户可以选择不同地区的接入节点，这样就分担了负载流量。

第三个优势就是容灾调度了，因为同样是有多组地址，结合 BGP 一些全局路由的动态协议，这时如果单个节点出现了故障，就能够动态地进行容灾，切换到其他的正常结点。

第四个优势在于它能够防止大流量攻击，我们知道在企业中想要在后端来防止这种大流量攻击是比较难的。即使是做流量清洗也有一定的局限性，更需要基于一套全局性的负载均衡，在接入用户更贴近的一层使用anycast就能够有效地防御，并阻止大流量的攻击。

接下来我们结合它的优势来介绍一个 Anycast 的应用案例。

## Anycast 应用案例介绍
第一个应用案例是在局域网内部自建一套分布式 DNS 系统。在公网 DNS 同样也可以结合 Anycast +BGP 来做 DNS 的公网分布式系统，在公网更多的企业会选择购买一些第三方服务，局域网的 DNS 系统和公网的 DNS 这两套系统和anycast结合模式，它们在原理上几乎是一致的。

我们来介绍一下在局域网内部自建这套 Anycast，结合 OSPF 的方式来构建一个分布式的 DNS 系统。那么为什么要选择 DNS 系统？我们知道，DNS 是企业的核心业务应用，如果 DNS 出现了故障，那么企业里面的绝大部分业务都会受到影响，所以我们要尽量减少 DNS 的压力，把 DNS 做到分布式，同时利用 Anycast 这种全局性的负载均衡允许用户就近进行接入，同时又结合 OSPF 这种最短路径优先的协议，使得它不仅能够提供网络中的动态路由，还能够实现给用户节点分配最短有效请求路径。

我们来看下面这张图：

CgqCHl7y7DeASsvgAAHdnr6HsEg948.png