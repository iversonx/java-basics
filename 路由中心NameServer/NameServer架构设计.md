消息中间件的设计思路一般基于主体的订阅发布机制。

- 消息生产者发送某一主题的消息到消息服务器

- 消息服务器负责消息的持久化存储。

- 消息消费者订阅感兴趣的主题

- 消息服务器根据订阅信息将消息推送（PUSH模式）到消费者或消费者主动向消息服务器拉取（PULL模式）

这样实现消费生产者与消费者解耦。

## NameServer 设计目的

在消息服务器集群环境中，会有这么两个问题：

1、消息生产者如何知道消息要发往哪台消息服务器

Broker消息服务器在启动时向所有NamerServer注册，生产者在发送消息前先从NameServer获取Broker服务器地址列表，然后根据负载均衡算法从列表中选择一台消息服务器进行消息发送。



2、当某台消息服务器宕机了，生产者如何在不重启服务的情况下感知

NameServer与每台Broker保持长连接，每隔30s检查Broker是否存活，如果检测的Broker宕机，则从路由注册表中将其移除。但不会马上通知消息生产者。



## NameServer启动流程

NamServer启动类是`org.apache.rocketmq.namesrv.NamesrvStartup`。

第一步：首先解析配置文件，填充`NameServerConfig`,`NettyServerConfig`属性值。



第二步：根据启动属性创建`NameSrvController`，并初始化该实例，`NameSrvController`实例为NameServer核心控制器

在`NameController`的初始化方法中，加载KV配置，创建NettyServer网络处理对象，然后开启两个定时任务进行心跳检测。

一个定时任务每隔10s扫描一次Broker，移除处于不激活状态的Broker

一个定时任务每隔10分钟打印一次KV配置



第三步：注册JVM钩子函数并启动服务器，以便监听Brokier，消息生产者的网络请求。








