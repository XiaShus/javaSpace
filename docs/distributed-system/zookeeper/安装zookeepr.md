```
wget https://mirrors.sonic.net/apache/zookeeper/zookeeper-3.5.9/apache-zookeeper-3.5.9-bin.tar.gz

tar -zxvf apache-zookeeper-3.5.9-bin.tar.gz 

cd apache-zookeeper-3.5.9-bin/

ls

cd conf/

mv zoo_sample.cfg zoo.cfg

cd ..

bin/zkServer.sh start
```

