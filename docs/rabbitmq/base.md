#  
# 快速导航
* [1.安装](#安装)
* [2.核心概念](#核心概念)
* [3.工作模式](#工作模式)

# 安装
MAC 安装
```
brew install rabbitmq
```
进入安装目录
```
cd /usr/local/Cellar/rabbitmq/3.7.5
```
启动
```
brew services start rabbitmq
进入控制台: http://localhost:15672/
用户名和密码：guest,guest
```
启动控制台之前需要先开启插件
```
./rabbitmq-plugins enable rabbitmq_management
```

# 核心概念

* VirtualHost
* Connection
* Exchange
* Channel
* Queue
* Binding



# 工作模式

* 1、Simple 模式
* 2、Work,工作模式
  * 一个消息只能被一个消费者获取

* 3、Publish/Subscribe, 订阅模式
  * 消息被路由投递给多个队列, 一个消息被多个消费者获取

* 4、Routing, 路由模式
  * 一个消息被多个消费者获取.并且消息的目标队列可被生产者指定

* 5、Topic, 话题模式
  * 一个消息被多个消费者获取. 消息的目标queue可用BindingKey以通配符(#:一个或多个词, *:一个词)的方式指定

