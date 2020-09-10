# 第22课：ELK 构建 MySQL 慢日志收集平台详解

本课时我们来讲解如何通过一套开源日志存储和检索系统 ELK 构建 MySQL 慢日志收集及分析平台。

## ELK、EFK 简介
想必你对 ELK、EFK 都不陌生，它们有一个共同的组件：Elasticsearch（简称ES），它是一个实时的全文搜索和分析引擎，可以提供日志数据的收集、分析、存储 3 大功能。另外一个组件 Kibana 是这套检索系统中的 Web 图形化界面系统，可视化展示在 Elasticsearch 的日志数据和结果。

ELF/EFK 工具集中还有 l 和 F 这两个名称的缩写，这两个缩写代表的工具根据不同的架构和使用方式而定。

L 通常是 Logstash 组件，它是一个用来搜集、分析、过滤日志的工具 。

F 代表 Beats 工具（它是一个轻量级的日志采集器），Beats 家族有 6 个成员，Filebeat 工具，它是一个用于在客户端收集日志的轻量级管理工具。

F 也可以代表工具 fluentd，它是这套架构里面常用的日志收集、处理转发的工具。

那么它们（Logstash VS Beats VS fluentd）有什么样的区别呢？Beats 里面是一个工具集，其中包含了 Filebeat 这样一个针对性的日志收集工具。Logstash 除了做日志的收集以外，还可以提供分析和过滤功能，所以它的功能会更加的强大。

Beats 和 fluentd 有一个共同的特点，就是轻量级，没有 Logstash 功能全面。但如果比较注重日志收集性能，Beats 里面的 Filebeat 和 fluentd 这两个工具会更有优势。

Kafka 是 ELK 和 EFK 里面一个附加的关键组件（缩写 K），它主要是在支持高并发的日志收集系统里面提供分布式的消息队列服务。

这就是 ELK 和 EFK ，我们常常会关注的一些组件的介绍，可以说它们不同的组成方式可以决定架构的不同，具体会有一些什么样的区别呢？接下来我会把常见的一些架构给你做一个展示。

## ELK 的优势

在此之前，先介绍 ELK 日志分析会有一些什么样的优势？主要有 3 点：

1.它是一套开源、完整的日志检索分析系统，包含收集、存储、分析、检索工具。我们不需要去开发一些额外的组件去完成这套功能，因为它默认的开源方式就提供了一整套组件，只要组合起来，就可以完成从日志收集、检索、存储、到整个展示的完整解决方案了。
2.支持可视化的数据浏览。运维人员只要在控制台里选择想关注的某一段时间内的数据，就可以查看相应的报表，非常快捷和方便。
3.它能广泛的支持一些架构平台，比如我们现在讲到的 K8s 或者是云原生的微服务架构。

## ELK 常见架构

那么接下来我们来讲一讲 ELK 的架构，以及常见的一些使用模式。企业在不同的结构下通常使用的一些具体架构是什么样子的？

首先第 1 套比较简单，而且是中小型企业才会用到的一种方式。即通过 Logstash 来做日志的收集，我们会看到这样的一张图，在每个客户端部署一套 Logstash 工具，主要用于日志收集，并且也可以提供一些日志的过滤功能。而服务端通过 Elasticsearch 和 Kibana 实现，这样只可以实现简单的日志收集。


Ciqc1F6z5SGAQhkKAAU8iq4TdH8042.png

Ciqc1F6z5SaAKUmwAAYyNWdHooE182.png

第 2 套架构是在整个客户端里，用到了 Beats 工具来做日志收集。服务端还是维持使用 Elasticsearch 和 Kibana，Beats 代替了 Logstash，Beats 里有一个非常重要的日志收集工具，就是 Filebeat，用来代替 Logstash。主要的优势是：

对内存 CPU 的资源消耗会比 Logstash 低，所以它的性能会更好，专门用于做日志收集比较合适。

另外， Beats 提供了非常丰富的场景，除了使用 Filebeat 工具外，还可以看到会有其他的一些家族成员，如 Packetbeat 、Metricbeat、Winlogbeat ，每个家族成员都会针对性地收集一些不同的数据。所以我们想要收集其他数据内容时，就可以用 Beats 里面的其他工具针对性的收集，如 Winlog 主要收集 Windows 上的相关日志。

Ciqc1F6z5TeAC7N2AAdHW2m1zI8345.png

第 3 套方案，我们看到在第 2 套方案中加入了很多的组件，其中最重要的一个组件就是 Kafka。

Kafka 作为日志消息队列，客户端通过 Filebeat 收集数据（日志）后将其先存入 Kafka，然后由 Logstash 提取并消费，这套架构的好处是：当我们有海量日志同步情况下，直接存入服务端 ES 很难直接应承接海量流量，所以 Kafka 会进行临时性的存取和缓冲，再由 Logstash 进行提取、过滤，通过 Logstash 以后，再把满足条件的日志数据存入 ES。

ES 不再是以单实例的方部署，而是采用集群架构，考虑 Kafka 的集群模式， Logstash 也使用集群模式。

我们会看到这套架构稍微庞大，大中型的企业往往存储海量数据（上百 T 或 P 级）运维日志、或者是系统日志、业务日志。

搭建演示