## 安装 RabbitMQ

### Windows

下载 msi 直接安装



### Linux





### Docker

* 拉取 RabbitMQ 镜像

``` linux
docker pull rabbitmq
```

* 启动 RabbitMQ 

``` linux
docker run -d --name rabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 -p 25672:25672 -p 61613:61613 -p 1883:1883 rabbitmq:management
```

* 复制文件到容器中

``` linux
docker cp /opt/docker/rabbitmq/rabbitmq_delayed_message_exchange-3.11.1.ez c26fb39cf7e8:/plugins/
```

* 进入容器

``` linux
docker exec -it rabbit bash
```

* 切换插件目录

``` mysql
rcd /plugins
```

* 启动控制台插件

``` mysql
rabbitmq-plugins enable rabbitmq_management
```

* 启动延迟队列插件

``` mysql
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```


