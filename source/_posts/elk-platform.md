---
title: elk platform
date: 2019-10-11 18:01:30
tags:
---

## ELK日志平台

迁移一篇旧账。

### 什么是ELK

ELK指的是ElasticSearch，Logstash，Kibana这三个工具的首字母缩写，中文指南很详细，按其步骤来，入门基本没啥问题。

### 解决什么问题

公司产品EDI平台的主要任务就是将各个数据源的文件信息处理后再发送第三方，没有实时监控，文件发送不成功时，运维相当被动。

ELK的强项就是实时的日志分析，通过对日志文件的分析（秒级）对比，及时将异常情况通知运维。

计划步骤如下：

1）制定日志规范，在程序中增加日志，记录处理过程。

2）部署Logstash监控日志文件。

3）开发服务程序，定时查询elasticsearch，按文件名比对处理状态，例如：有发送记录，但设定时间内无接收的记录则报警。

计划架构如下：

logstash -> elasticsearch -> kibana

### 踩坑之旅

其实也不算坑，主要还是不了解。

1）指南中说Logstash在windows环境下可能不靠谱，所以替换成Filebeat，研究一圈后发现居然没有字段提取功能，所以架构又变成

Filebeat -> Logstash -> ElasticSearch -> Kibana

2）指南中看到了Kibana的watcher插件，貌似可以替代计划的3），研究一圈后要实现定时查询后遍历再匹配很难，性能估计也扛不住。

3）重新审视计划，更好的方案应该是在日志写入时就匹配，匹配上后更新发送记录状态，watcher只要实现定时查询没有匹配状态的发送记录即可。

4）准备开发logstash的插件，研究后发现通过logstash-input-beats/logstash-filter-grok/logstash-filter-elasticsearch/logstash-output-elasticsearch插件可实现该功能。

5）logstash-filter-elasticsearch插件，原打算通过此步获取_id并在output中更新发送的日志记录，但其他值都能取到，就是没有_id，查看github源码发现

只能获取’hits’][‘hits’][_source]中的内容，而_id为[‘hits’][‘hits’][‘_id’]，修改代码后OK.

6）logstash-out-elasticsearch插件，通过设置document_id及action=update后可以更新成功，但把所有的接收日志信息都更新到了发送日志记录，只有

另取其他插件实现了。

7）logstash-out-http插件，通过该插件直接post更新语句可以实现。

8）原料准备完毕，下一步该写wacher脚本实现预警了，踩完再来补充。
