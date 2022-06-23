## 安装RocketMQ

### Windows





### Linux





### Docker

#### 启动NameServer

```shell
docker run -d -p 9876:9876 --name rmqserver foxiswho/rocketmq:server-4.5.1
```



#### 启动broker

```shell
docker run -d -p 10911:10911 -p 10909:10909 --name rmqbroker --link rmqserver:namesrv -e "NAMESRV_ADDR=namesrv:9876" -e "JAVA_OPTS=-Duser.home=/opt" -e "JAVA_OPT_EXT=-server -Xms128m -Xmx128m" foxiswho/rocketmq:broker-4.5.1
```



#### 修改配置文件

```shell
docker exec -it 容器id /bin/bash
# 进入目录，找到配置文件broker.conf
cd /etc/rocketmq
# 修改broker.conf
vim broker.conf
# 在最后添加一行添加服务器公网IP
brokerIP1=宿主机IP
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
docker ps|grep rocketmq
```

**结果如下**

```shell
67ad6c4f3707        foxiswho/rocketmq:broker-4.5.1   "/bin/sh -c 'cd ${RO…"   17 minutes ago      Up 14 minutes       0.0.0.0:10909->10909/tcp, 0.0.0.0:10911->10911/tcp   rmqbroker
9231c8592227        styletang/rocketmq-console-ng    "sh -c 'java $JAVA_O…"   45 minutes ago      Up 45 minutes       0.0.0.0:8180->8080/tcp                               rmqconsole
f514b1f02101        foxiswho/rocketmq:server         "/bin/sh -c 'cd ${RO…"   45 minutes ago      Up 45 minutes       10909/tcp, 0.0.0.0:9876->9876/tcp, 10911-10912/tcp   rmqserver

```



### Java测试Demo

#### Maven依赖

```xml
<!--RocketMq-->
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.5.0</version>
</dependency>
```



#### 生产者

```java
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.common.message.Message;
import org.apache.rocketmq.remoting.common.RemotingHelper;
import javax.jms.JMSException;

public class RocketMqQueueProducer {

    public static void main(String[] args) throws JMSException, InterruptedException, MQClientException {

        // 需要一个producer group名字作为构造方法的参数，这里为producer1
        DefaultMQProducer producer = new DefaultMQProducer("producer1");

        // 设置NameServer地址,此处应改为实际NameServer地址，多个地址之间用；分隔
        // NameServer的地址必须有，但是也可以通过环境变量的方式设置，不一定非得写死在代码里
        producer.setNamesrvAddr("1.15.137.253:9876");
        producer.setVipChannelEnabled(false);

        // 为避免程序启动的时候报错，添加此代码，可以让rocketMq自动创建topickey
        producer.setCreateTopicKey("AUTO_CREATE_TOPIC_KEY");
        producer.start();

        for (int i = 0; i < 10; i++) {
            try {
                // topic 主题名称
                // pull 临时值 在消费者消费的时候 可以根据msg类型进行消费
                // body 内容
                Message message = new Message("producer-topic", "msg", ("Hello RocketMQ " + i).getBytes(RemotingHelper.DEFAULT_CHARSET));
                SendResult sendResult = producer.send(message);

                System.out.println("发送的消息ID:" + sendResult.getMsgId() + "--- 发送消息的状态：" + sendResult.getSendStatus());
            } catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(1000);
            }
        }
        producer.shutdown();
    }

}
```



#### 消费者

```java
import java.io.IOException;
import java.util.List;
import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.common.consumer.ConsumeFromWhere;
import org.apache.rocketmq.common.message.MessageExt;
import javax.jms.JMSException;

public class RocketMqQueueConsumer {

    public static void main(String[] args) throws JMSException, IOException, MQClientException {
        // 设置消费者组
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer-group");

        consumer.setVipChannelEnabled(false);
        consumer.setNamesrvAddr("1.15.137.253:9876");
        // 设置消费者端消息拉取策略，表示从哪里开始消费
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

        // 设置消费者拉取消息的策略，*表示消费该topic下的所有消息，也可以指定tag进行消息过滤
        consumer.subscribe("producer-topic", "msg");

        // 消费者端启动消息监听，一旦生产者发送消息被监听到，就打印消息，和rabbitmq中的handlerDelivery类似
        consumer.registerMessageListener(new MessageListenerConcurrently() {

            @Override
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                for (MessageExt messageExt : msgs) {
                    String topic = messageExt.getTopic();
                    String tag = messageExt.getTags();
                    String msg = new String(messageExt.getBody());
                    System.out.println("*********************************");
                    System.out.println("消费响应：msgId : " + messageExt.getMsgId() + ",  msgBody : " + msg + ", tag:" + tag + ", topic:" + topic);
                    System.out.println("*********************************");
                }

                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });

        // 调用start()方法启动consumer
        consumer.start();
        System.out.println("Consumer Started....");
    }
}
```