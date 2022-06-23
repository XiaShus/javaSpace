## 安装KafkaMQ

### Windows





### Linux





### Docker

#### 启动zookeeper

```shell
docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper
```



#### 启动Kafka

```shell
docker run  -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=宿主机IP:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://宿主机IP:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
```



#### 参数解释

- -e KAFKA_BROKER_ID=0 在kafka集群中，每个kafka都有一个BROKER_ID来区分自己
- -e KAFKA_ZOOKEEPER_CONNECT=10.9.44.11:2181/kafka 配置zookeeper管理kafka的路径10.9.44.11:2181/kafka
- -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.9.44.11:9092 把kafka的地址端口注册给zookeeper

- -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口

- -v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间



#### 修改配置文件

```shell
docker exec -it 容器id /bin/bash
# 进入目录，找到配置文件broker.conf
cd /etc/rocketmq
# 修改broker.conf
vim broker.conf
# 在最后添加一行添加服务器公网IP
brokerIP1=47.116.143.16
# 保存
:wq! 
# 退出到 Linux 操作界面
exit
```



#### 重启broker

```shell
docker restart 容器id
```



#### 安装 rocketmq console

```Shell
docker run -d --name rmqconsole -p 8180:8080 --link rmqserver:namesrv -e "JAVA_OPTS=-Drocketmq.namesrv.addr=namesrv:9876 -Dcom.rocketmq.sendMessageWithVIPChannel=false" -t styletang/rocketmq-console-ng
```



#### 检查启动情况

```shell
docker ps -a
```

**结果如下**

```shell
0274c69ed394        wurstmeister/kafka               "start-kafka.sh"         27 hours ago        Up 17 seconds                     0.0.0.0:9092->9092/tcp                               kafka
ee96b25d1231        wurstmeister/zookeeper           "/bin/sh -c '/usr/sb…"   27 hours ago        Up 26 seconds                     22/tcp, 2888/tcp, 3888/tcp, 0.0.0.0:2181->2181/tcp   zookeeper
```



### Java测试Demo

#### Maven依赖

```xml
<!--kafkaMQ-->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```



#### 生产者

```java
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import javax.jms.*;
import java.util.Properties;

public class KafkaMqQueueProducer {

    public static final String brokerList = "1.15.137.253:9092";

    public static final String topic = "topic-demo";

    public static void main(String[] args) throws JMSException {
        //配置生产者客户端参数
        //将配置序列化
        Properties properties = new Properties();
        properties.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // 重试次数
        properties.put("retries", "3");
        // 重试间隔
        properties.put("retry.backoff.ms", "1000");
        properties.put("bootstrap.servers", brokerList);
        //创建KafkaProducer 实例
        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);
        //构建待发送的消息
        ProducerRecord<String, String> record = new ProducerRecord<String, String>(topic, "hello Kafka!");
        try {
            //尝试发送消息
            producer.send(record);
            //打印发送成功
            System.out.println("send success from producer");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //关闭生产者客户端实例
            producer.close();
        }
    }
}
```



#### 消费者

```java

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import javax.jms.*;
import java.io.IOException;
import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class KafkaMqQueueConsumer {

    public static final String brokerList = "1.15.137.253:9092";

    public static final String topic = "topic-demo";
    public static final String groupId = "ODS-PSR-P.*";

    public static void main(String[] args) throws JMSException, IOException {
        //设置消费组的名称
        //将属性值反序列化
        Properties properties = new Properties();
        properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        properties.put("bootstrap.servers", brokerList);
        properties.put("group.id", groupId);

        //创建一个消费者客户端实例
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
        //订阅主题
        consumer.subscribe(Collections.singletonList(topic));

        //循环消费消息
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.println("receiver a message from consumer client:" + record.value());
            }
        }
    }
}
```