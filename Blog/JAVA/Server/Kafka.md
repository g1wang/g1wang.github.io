## 安装Kafka
https://kafka.apache.org/quickstart

### STEP 1: GET KAFKA
```
# downlaod
$ wget https://dlcdn.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz
$ tar -xzf kafka_2.13-3.3.1.tgz
$ cd kafka_2.13-3.3.1

```

### STEP 2: START THE KAFKA ENVIRONMENT

#### Kafka with ZooKeeper
```
# Start the ZooKeeper service
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

Open another terminal session and run:
```
# Start the Kafka broker service
$ bin/kafka-server-start.sh config/server.properties
```

### STEP 3: CREATE A TOPIC TO STORE YOUR EVENTS
```
# create topic
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092

# details such as the partition count of the new topic
$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092

```

### STEP 4: WRITE SOME EVENTS INTO THE TOPIC
A Kafka client communicates with the Kafka brokers via the network for writing (or reading) events. Once received, the brokers will store the events in a durable and fault-tolerant manner for as long as you need—even forever.
Run the console producer client to write a few events into your topic. By default, each line you enter will result in a separate event being written to the topic.

```
bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
> This is my first event
> This is my second event
```

### STEP 5: READ THE EVENTS
Open another terminal session and run the console consumer client to read the events you just created:
```
$ bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
> This is my first event
> This is my second event
```

## kafka-manager
kafka可视化工具
https://github.com/yahoo/CMAK

- 下载zip包,解压
```
unzip cmak-3.0.0.6.zip
```

- 启动
```
# 修改配置文件 conf/application.conf
# 单个zk
cmak.zkhosts="my.zookeeper.host.com:2181"
# zk集群
cmak.zkhosts="my.zookeeper.host.com:2181,other.zookeeper.host.com:2181"

# 默认端口9000，-Dhttp.port 指定端口
$ bin/cmak -Dconfig.file=/path/to/application.conf -Dhttp.port=8080
```
- 登录管理后台
	- Add Cluster

## kafka 原理

### Kafka架构

- 话题（Topic）：是特定类型的消息流。消息是字节的有效负载（Payload），话题是消息的分类名或种子（Feed）名
- 生产者（Producer）：是能够发布消息到话题的任何对象
- 服务代理（Broker）：已发布的消息保存在一组服务器中，它们被称为代理（Broker）或Kafka集群
- 消费者（Consumer）：可以订阅一个或多个话题，并从Broker拉数据，从而消费这些已发布的消息

### Kafka存储策略

- 1）kafka以topic来进行消息管理，每个topic包含多个partition，每个partition对应一个逻辑log，有多个segment组成