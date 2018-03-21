# Kafka 安装和配置

## Kafka 介绍

> A distributed streaming platform

- 官网：<https://kafka.apache.org/>
- 官网下载：<https://kafka.apache.org/downloads>
- 当前最新稳定版本（201803）：**1.0.1**
- 官网 quickstart：<https://kafka.apache.org/quickstart>
- 核心概念：
	- Producer：生产者（业务系统），负责发布消息到 broker
	- Consumer：消费者（业务系统），向 broker 读取消息的客户端
	- Broker：可以理解为：存放消息的管道（kafka）
	- Topic：可以理解为：消息主题、消息标签（物理上不同 Topic 的消息分开存储，逻辑上一个 Topic 的消息虽然保存于一个或多个 broker 上但用户只需指定消息的 Topic 即可生产或消费数据而不必关心数据存于何处）
    - Partition：Partition 是物理上的概念，每个Topic包含一个或多个Partition.
    - Consumer Group：每个 Consumer 属于一个特定的 Consumer Group（可为每个 Consumer 指定 group name，若不指定 group name 则属于默认的 group）一般一个集群指定一个 group
- 业界常用的 docker 镜像：
	- [wurstmeister/kafka-docker（不断更新，优先）](https://github.com/wurstmeister/kafka-docker/)
		- 运行的机子不要小于 2G 内存
		- 修改 docker-compose.yml 中参数 KAFKA_ADVERTISED_HOST_NAME，改为你宿主机的 IP 地址
		- 先启动 zookeeper：`docker-compose up -d`
		- 再添加 kafka 节点：`docker-compose scale kafka=3`
		- 停止容器：`docker-compose stop`
		- 进入容器：`docker exec -it 54f /bin/bash`
	- [spotify/docker-kafka](https://github.com/spotify/docker-kafka)
	- Spring 项目选用依赖包的时候，对于版本之间的关系可以看这里：<http://projects.spring.io/spring-kafka/>
		- 目前（201803） spring-kafka 必须是 2.1.x 才能对应 kafka 1.0.x
- 官网 quickstart 指导：<https://kafka.apache.org/quickstart>
- 常用命令：
	- 容器中 kafka home：`/opt/kafka`
	- 我的 zookeeper 地址：`10.135.157.34:2181`，有多个节点用逗号隔开
	- 列出所有 topic：`bin/kafka-topics.sh --list --zookeeper 10.135.157.34:2181`
	- 创建 topic：`bin/kafka-topics.sh --create --topic kafka-test-topic-1 --partitions 3 --replication-factor 1 --zookeeper 10.135.157.34:2181`
		- 创建名为 kafka-test-topic-1 的 topic，3个分区分别存放数据，数据备份总共 2 份
	- 查看特定 topic 的详情：`bin/kafka-topics.sh --describe --topic kafka-test-topic-1 --zookeeper 10.135.157.34:2181`
	- 删除 topic：`bin/kafka-topics.sh --delete --topic kafka-test-topic-1 --zookeeper 10.135.157.34:2181`
	- 更多命令可以看：<http://orchome.com/454>

## 单个实例部署

- 我的宿主机 ip：`172.16.0.2`，下面会用到
- 部署 zookeeper：`docker run -d --name one-zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime zookeeper:3.4`
- 部署 kafka：
	- 目前 latest 用的时候 kafka 1.0.1，要指定版本可以去作者 [github](https://github.com/wurstmeister/kafka-docker) 看下 tag 目录，切换不同 tag，然后看下 Dockerfile 里面的 kafka 版本号

```
docker run -d --name one-kafka -p 9092:9092 \
--link one-zookeeper \
--env KAFKA_ZOOKEEPER_CONNECT=one-zookeeper:2181 \
--env KAFKA_ADVERTISED_HOST_NAME=172.16.0.2 \
--env KAFKA_ADVERTISED_PORT=9092 \
-v /etc/localtime:/etc/localtime \
wurstmeister/kafka:latest
```

- 测试：
	- 进入 kafka 容器：`docker exec -it one-kafka /bin/bash`
	- 根据官网 Dockerfile 说明，kafka home 应该是：`cd /opt/kafka`
	- 创建 topic 命令：`bin/kafka-topics.sh --create --zookeeper one-zookeeper:2181 --replication-factor 1 --partitions 1 --topic my-topic-test`
	- 查看 topic 命令：`bin/kafka-topics.sh --list --zookeeper one-zookeeper:2181`
	- 给 topic 发送消息命令：`bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-topic-test`，然后在出现交互输入框的时候输入你要发送的内容
	- 再开一个终端，进入 kafka 容器，接受消息：`bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic my-topic-test --from-beginning`
	- 此时发送的终端输入一个内容回车，接受消息的终端就可以收到。


## 资料

- <http://www.ituring.com.cn/article/499268>
- <http://orchome.com/kafka/index>
- <http://blog.51cto.com/wangchunyi/2046163>
- <https://www.jianshu.com/p/263164fdcac7>
- <http://www.bijishequ.com/detail/536308>
- <http://lanxinglan.cn/2017/10/18/%E5%9C%A8Docker%E7%8E%AF%E5%A2%83%E4%B8%8B%E9%83%A8%E7%BD%B2Kafka/>
- <https://www.cnblogs.com/ding2016/p/8282907.html>
- <http://blog.csdn.net/fuyuwei2015/article/details/73379055>
- <https://segmentfault.com/a/1190000012990954>
- <http://www.54tianzhisheng.cn/2018/01/04/Kafka/>
- <https://renwole.com/archives/442>
- <http://www.bijishequ.com/detail/542646?p=85>
- <http://blog.csdn.net/zhbr_f1/article/details/73732299>
- <http://wangzs.leanote.com/post/kafka-manager%E5%AE%89%E8%A3%85>
- <https://cloud.tencent.com/developer/article/1013313>
- <http://blog.csdn.net/boling_cavalry/article/details/78309050>
- <https://www.jianshu.com/p/d77149efa59f>
- <http://www.bijishequ.com/detail/536308>
- <http://blog.51cto.com/13323775/2063420>
- <http://lanxinglan.cn/2017/10/18/%E5%9C%A8Docker%E7%8E%AF%E5%A2%83%E4%B8%8B%E9%83%A8%E7%BD%B2Kafka/>
- <http://www.cnblogs.com/huxi2b/p/6592862.html>
- <http://www.cnblogs.com/huxi2b/p/7929690.html>
- <http://blog.csdn.net/HG_Harvey/article/details/79198496>
- <http://blog.csdn.net/vtopqx/article/details/78638996>
- <http://www.weduoo.com/archives/2047>
- <http://www.jishurensheng.com/461884086.html>
- <https://blog.52itstyle.com/archives/2358/>