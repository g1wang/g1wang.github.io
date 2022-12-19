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

# create 
$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
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
