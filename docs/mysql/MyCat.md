# Mycat2 调研文档



## 资源列表

- 官网：http://www.mycat.org.cn/
- 资源地址：http://dl.mycat.org.cn/2.0/
- mycat2官方文档： https://www.yuque.com/ccazhw/ml3nkf




## 安装

### 前提

安装前要在服务器上安装jdk1.8

如果是mysql 8, 需要用root用户连接并执行下面语句开启XA事务

```mysql
GRANT XA_RECOVER_ADMIN ON *.* TO 'root'@'%';
```



### 文件下载

把所有文件都下载到/opt/tools目录

```bash
cd /opt/tools

wget http://dl.mycat.org.cn/2.0/install-template/mycat2-install-template-1.21.zip

wget http://dl.mycat.org.cn/2.0/1.21-release/mycat2-1.21-release-jar-with-dependencies.jar
```



### 安装 Mycat 

程序安装目录为/opt，解压并移到到/opt目录

```bash
cd /opt/tools

unzip mycat2-install-template-1.21.zip

mv mycat ../
```

把bin目录的文件加执行权限

```bash
cd /opt/mycat/bin

chmod +x *
```

把所需的jar复制到mycat/lib目录

```bash
cd /opt/mycat/lib/

cp /opt/tools/mycat2-1.21-release-jar-with-dependencies.jar ./
```



### 修改数据源

切换到数据源目录

```bash
cd /opt/mycat/conf/datasources
```

把 mycat 带的数据源配置正确（password、url、user）

```bash
vim prototypeDs.datasource.json
```

```json
{
    "dbType":"mysql",
    "idleTimeout":60000,
    "initSqls":[

    ],
    "initSqlsGetConnection":true,
    "instanceType":"READ_WRITE",
    "maxCon":1000,
    "maxConnectTimeout":3000,
    "maxRetryCount":5,
    "minCon":1,
    "name":"prototypeDs",
    "password":"123456",
    "type":"JDBC",
    "url":"jdbc:mysql://localhost:3306/test?useUnicode=true&amp;serverTimezone=Asia/Shanghai&amp;characterEncoding=UTF-8",
    "user":"root",
    "weight":0
}
```



新增第一个数据源，名称为：113-3306

```bash
vim 113-3306.datasource.json
```

```json
{
    "dbType":"mysql",
    "idleTimeout":60000,
    "initSqls":[

    ],
    "initSqlsGetConnection":true,
    "instanceType":"READ_WRITE",
    "maxCon":1000,
    "maxConnectTimeout":3000,
    "maxRetryCount":5,
    "minCon":1,
    "name":"113-3306",
    "password":"123456",
    "type":"JDBC",
    "url":"jdbc:mysql://localhost:3306/test?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
    "user":"root",
    "weight":0
}
```



新增第二个数据源，名称为：113-3307

```bash
vim 113-3307.datasource.json
```

```json
{
    "dbType":"mysql",
    "idleTimeout":60000,
    "initSqls":[

    ],
    "initSqlsGetConnection":true,
    "instanceType":"READ_WRITE",
    "maxCon":1000,
    "maxConnectTimeout":3000,
    "maxRetryCount":5,
    "minCon":1,
    "name":"113-3306",
    "password":"123456",
    "type":"JDBC",
    "url":"jdbc:mysql://localhost:3307/test?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
    "user":"root",
    "weight":0
}
```



### 修改 mycat 登录密码为：zxcvbn

修改（password）

```bash
cd /opt/mycat/conf/users

vim root.user.json
```

```json
{
    "dialect":"mysql",
    "ip":null,
    "password":"zxcvbn",
    "transactionType":"xa",
    "username":"root"
}
```



### 修改服务器 server 配置

添加 serverVersion 相关信息

```bash
cd /opt/mycat/conf/

vim server.json
```

```json
{
    "loadBalance":{
        "defaultLoadBalance":"BalanceRandom",
        "loadBalances":[

        ]
    },
    "mode":"local",
    "properties":{

    },
    "server":{
        "bufferPool":{

        },
        "idleTimer":{
            "initialDelay":3,
            "period":60000,
            "timeUnit":"SECONDS"
        },
        "ip":"0.0.0.0",
        "mycatId":1,
        "port":8066,
        "serverVersion":"8.0.20-mycat-2.0",
        "reactorNumber":8,
        "tempDirectory":null,
        "timeWorkerPool":{
            "corePoolSize":0,
            "keepAliveTime":1,
            "maxPendingLimit":65535,
            "maxPoolSize":2,
            "taskTimeout":5,
            "timeUnit":"MINUTES"
        },
        "workerPool":{
            "corePoolSize":1,
            "keepAliveTime":1,
            "maxPendingLimit":65535,
            "maxPoolSize":1024,
            "taskTimeout":5,
            "timeUnit":"MINUTES"
        }
    }
}
```



### 启动 mycat

```bash
cd /opt/mycat/bin/

./mycat start
```



### 查看 mycat 日志

```bash
cd /opt/mycat/logs

tail -f wrapper.log
```



### **其他**

```shell
把原型的配置写入该mycat的本地配置,但不加载到运行时
/*+mycat:syncConfigFromDbToFile{} */ 

把本地配置写入该mycat指向的原型库,但不加载到运行时
/*+mycat:syncConfigFromFileToDb{} */

把本地配置加载到运行时
/*+mycat:loadConfigFromFile{} */ 

检查原型库配置与本地配置是否一致
/*+mycat:checkConfigConsistency{} */

可以连接 mycat用 explain 查看 sql执行计划
```





## 配置文件

### 配置结构

```shell
mycat配置文件夹

+ clusters
	- prototype.cluster.json //无集群的时候自动创建
	- c0.cluster.json
	- c1.cluster.json
+ datasources
	- prototypeDs.datasource.json //无数据源的时候自动创建
	- dr0.datasource.json
	- dw0.datasource.json
+ schemas
	- db1.schema.json
	- mysql.schema.json
+ sequences
	- db1_schema.sequence.json
- server.json //服务器配置
- state.json //mycat运行状态,包含集群选举的主节点信息,配置时间戳
```





### 服务器 server

服务器配置是 MyCat 运行时需要的信息，其中 serverVersion 是我们需要在默认文件中添加的，而 ip 和 port 则关乎于外部连接 MyCat 使用的连接信息。



```json
{
 "datasourceProvider":";io.mycat.datasource.jdbc.datasourceprovider.DruidDatasourceProvider",
 "loadBalance":{
    "defaultLoadBalance":"BalanceRandom",
    "loadBalances":[]
  },
  "mode":"local",
  "serverVersion" : "5.7.33-mycat-2.0" 
  "properties":{},
  "server":{
    "bufferPool":{
    },
    "idleTimer":{
      "initialDelay":3,
      "period":15,
      "timeUnit":"SECONDS";
    },
    "ip":"127.0.0.1",
    "mycatId":1,
    "port":8066,
    "reactorNumber":8,
    "tempDirectory":null,
    "timeWorkerPool":{
      "corePoolSize":0,
      "keepAliveTime":1,
      "maxPendingLimit":65535,
      "maxPoolSize":2,
      "taskTimeout":1,
      "timeUnit":"MINUTES"
    },
    "workerPool":{
      "corePoolSize":8,
      "keepAliveTime":1,
      "maxPendingLimit":65535,
      "maxPoolSize":1024,
      "taskTimeout":1,
      "timeUnit":"MINUTES"
    },
    "mergeUnionSize": 5,
    "ignoreCast": false ,//生成的sql是否忽略类型转换,1.17支持,
    "joinClustering": true ,//开启后进行join重排序,关闭后会加快优化速度,1.18支持
  }
}
```

**server.json**

server.json 保存在`conf`文件夹



如果不配置则使用上述的值进行加载

- `mergeUnionSize`为使用一次`union all`合拼同一个存储节点上多个`dataNode`的sql数量，该参数在`1.15`版本以后才存在
- `mycatId`是保证多个mycat公用存储节点的时候必须配置这个值,并且唯一,他用于生成序列号,Xid等.
- `serverVersion`用于客户端与服务端适配，比如`SELECT @@session.transaction_isolation`



### 库 schema

这个是 Mycat 上的逻辑库，也就是我们连接到 MyCat 后看到的库。

Mycat2支持使用SQL直接建表,请看[Mycat2入门](https://www.w3cschool.cn/mycat2/mycat2-fksq3krj.html),可以通过建立SQL执行脚本,在客户端执行即可.

**配置的schema的逻辑库逻辑表必须在原型库(prototype)中有对应的物理库物理表,否则不能启动**



{库名}.schema.json保存在`schemas`文件夹 库配置

```json
{
  "customTables": {},
  "globalTables": {},
  "normalTables": {},
  "schemaName": "test",
  "shardingTables": {},
  "targetName": "prototype"
}
```

`targetName`自动从`prototype`目标加载`test`库下的物理表或者视图作为单表,`prototype`必须是`mysql`服务器

该配置对应`1.6`的`schema`上配置`dataNode`用于读写分离



#### 单表配置

```json
{
  "schemaName": "mysql-test",
  "normalTables": {
    "role_edges": {
      "createTableSQL":null,//可选
      "locality": {
        "schemaName": "mysql",//物理库
        "tableName": "role_edges",//物理表
        "targetName": "prototype"//指向集群,或者数据源
      }
    }
    ......
```



#### 全局表配置

```json
{
  "schemaName": "mysql-test",
  "globalTables": {
    "role_edges": {
      "broadcast": [{"targetName": "c0"},{"targetName": "c1"}]
    }
    ......
```



#### 分片表配置

```json
{
  "customTables": {},
  "globalTables": {},
  "normalTables": {},
  "schemaName": "db1",
  "shardingTables": {
    "travelrecord": {
      "function": {
        "properties": {
          "dbNum": "2",//分库数量
          "tableNum": "2",//分表数量
          "tableMethod": "hash(id)",//分表分片函数
          "storeNum": 2,//实际存储节点数量
          "dbMethod": "hash(id)"//分库分片函数
        }
      }
    }
  }
}
```



上述配置自动使用`c0`,`c1`两个集群作为存储节点

分片表配置`-hash`型自动分片算法

```json
{
  "customTables": {},
  "globalTables": {},
  "normalTables": {},
  "schemaName": "db1",
  "shadringTables": {
    "travelrecord": {
      "function": {
        "properties": {
          "dbNum": "2",//分库数量
          "tableNum": "2",//分表数量
          "tableMethod": "hash(id)",//分表分片函数
          "storeNum": 2,//实际存储节点数量
          "dbMethod": "hash(id)",//分库分片函数
          "mappingFormat": "c${targetIndex}/db1_${dbIndex}/travelrecord_${tableIndex}"
        }
      }
    }
  }
}
```



#### 分片表配置-自定义分片算法

```json
{
  "customTables": {},
  "globalTables": {},
  "normalTables": {},
  "schemaName": "db1",
  "shadingTables": {
    "travelrecord": {
      "function": {
        "clazz": ,//具体自定义分片算法
        "properties": {
          ...分片算法参数
        }
      },
     "partition":{
         "targetNames":"c$0-1",
         "schemaNames":"db1_$0-1",
         "tableNames":"t1_$0-1"
     }
    }
  }
}
```



`partitio`n配置存储节点,在分片算法无法使用的时候就扫描这些存储节点,所以分片算法无法正确配置的时候仍然可以查询,但是可能插入报错 需要注意的是在此处

`partition`中的生成表达式不支持`groovy`运算只支持`$0-1`语法生成

`partition`中的`targetName-schemaName-tableName`不能重复



样例

```json
{
  "customTables":{},
  "globalTables":{},
  "normalTables":{},
  "schemaName":"db1",
  "shadingTables":{
    "sharding":{
      "createTableSQL":"CREATE TABLE db1.`sharding` (\n  `id` bigint NOT NULL AUTO_INCREMENT,\n  `user_id` varchar(100) DEFAULT NULL,\n  `create_time` date DEFAULT NULL,\n  `fee` decimal(10,0) DEFAULT NULL,\n  `days` int DEFAULT NULL,\n  `blob` longblob,\n  PRIMARY KEY (`id`),\n  KEY `id` (`id`)\n) ENGINE=InnoDB  DEFAULT CHARSET=utf8",
      "function":{
        "clazz":"io.mycat.router.mycat1xfunction.PartitionByHotDate",
        "properties":{
          "lastTime":90,
          "partionDay":180,
          "dateFormat":"yyyy-MM-dd",
          "columnName":"create_time"
        },
        "ranges":{}
      },
      "partition":{
        "schemaNames":"db1",
        "tableNames":"sharding_$0-1",
        "targetNames":"c0"
      }
    }
  }
}
```







### 数据源 datasource

数据源是 Mycat 中的一个基础组件，相当于一个物理库的地址，在后面的主从、切片等都必须要使用的一项。



```json
{
  "dbType": "mysql",
  "idleTimeout": 60000,
  "initSqls": [],
  "initSqlsGetConnection": true,
  "instanceType": "READ_WRITE",
  "maxCon": 1000,
  "maxConnectTimeout": 3000,
  "maxRetryCount": 5,
  "minCon": 1,
  "name": "prototype",
  "password": "123456",
  "type": "JDBC",
  "url": "jdbc:mysql://127.0.0.1:3306?useUnicode=true&serverTimezone=UTC",
  "user": "root",
  "weight": 0,
  "queryTimeout":30//mills
}
```

**prototype.datasource.json**

{数据源名字}.datasource.json 保存在`datasources`文件夹

- `maxConnectTimeout`:单位`millis`,配置中的定时器主要作用是定时检查闲置连接
- `initSqlsGetConnection`,`true|false`,默认:`false`,对于jdbc每次获取连接是否都执行`initSqls`
- `type`:数据源类型
- `NATIVE`:只使用`NATIVE`协议(即Mycat自研的连接MySQL的协议)
- `JDBC`:默认,只使用JDBC驱动连接
- `NATIVE_JDBC`:该数据源同一个配置同时可以使用`NATIVE`,`JDBC`
- `queryTimeout`:jdbc查询超时时间 默认`30mills`
- `JDBC`禁用`SSL`属性有助提高性能
- `instanceType`:配置实例只读还是读写，可选值：`READ_WRITE`,`READ`,`WRITE`
- `weight`:负载均衡特定用的权重



### 集群 cluster

**集群配置**

```json
{
    "clusterType":"MASTER_SLAVE",
    "heartbeat":{
        "heartbeatTimeout":1000,
        "maxRetryCount":3,//2021-6-4前是maxRetry，后更正为maxRetryCount
        "minSwitchTimeInterval":300,
        "slaveThreshold":0
    },
    "masters":[ //配置多个主节点,在主挂的时候会选一个检测存活得数据源作为主节点
        "prototypeDs"
    ],
  "replicas":[//配置多个从节点
        "xxxx"
    ],
    "maxCon":200,
    "name":"prototype",
    "readBalanceType":"BALANCE_ALL",
    "switchType":"SWITCH"
  
  ////////////////////////////////////可选//////////////////////////////////
  ,
  "timer":{ //MySQL集群心跳周期,配置则开启集群心跳,Mycat主动检测主从延迟以及高可用主从切换
    "initialDelay": 30,
  "period":5,
  "timeUnit":"SECONDS"
    }
}
```

**c0.cluster.json**

{集群名字}.cluster.json 保存在`clusters`文件夹 **clusterType**

- `SINGLE_NODE`:单一节点
- `MASTER_SLAVE`:普通主从
- `GARELA_CLUSTER`:`garela cluster/PXC`集群
- `MHA`:(v1.16提供,实验)
- `MGR`:(v1.16提供,实验)

`MHA`与`MGR`集群会在心跳过程中根据`READ_ONLY`状态判断当前节点是否从节点(`READ_ONLY=ON`),主节点(`READ_ONLY=OFF`)动态更新主从节点信息,这样可以支持多主,单主.但是实际上生产上建议暂时使用单主模式,或者多主作为单主使用



## 注释语句

注释使用SQL注释方式表达,可以用于动态更新Mycat配置并且把配置持久化,它的设计目标是为了动态的更新mycat的配置.但是由于配置的属性复杂,它不会自动的更改真实数据库的schema.



通过注解配置不会自动创建物理库物理表(与直接使用自动化建表语句不同,它会自动建物理库物理表),所以要保证物理库物理表的在真实数据库上是与配置对应的.一般来说,原型库(prototype)上必须存在与逻辑库逻辑表完全一致得物理库物理表,以便mycat读取表和字段信息.



如果搞不懂配置,可以尝试使用自动化建表语句创建测试的物理库物理表,它会自动生成配置文件,然后通过查看本地的配置文件,观察它的属性,就可以知道什么回事.因为自动化建表语句过于简单,可能不适合公司的业务,此时需要更改配置文件的属性来调整.这种自己更改调整的属性值不属于mycat的开发测试范畴之内,也不能受mycat为自动化建表的测试保证.



### 创建用户

```sql
/*+ mycat:createUser{
	"username":"user",
	"password":"",
	"ip":"127.0.0.1",
	"transactionType":"xa"
} */
```

### 删除用户

```sql
/*+ mycat:dropUser{
	"username":"user"} */
```

### 显示用户

```sql
/*+ mycat:showUsers */
```

### 修改表序列号为雪花算法

```sql
/*+ mycat:setSequence{"name":"db1_travelrecord","time":true} */;
```

### 创建数据源

```sql
/*+ mycat:createDataSource{
	"dbType":"mysql",
	"idleTimeout":60000,
	"initSqls":[],
	"initSqlsGetConnection":true,
	"instanceType":"READ_WRITE",
	"maxCon":1000,
	"maxConnectTimeout":3000,
	"maxRetryCount":5,
	"minCon":1,
	"name":"dr0",
	"password":"123456",
	"type":"JDBC",
	"url":"jdbc:mysql://127.0.0.1:3306?useUnicode=true&serverTimezone=UTC&characterEncoding=UTF-8",
	"user":"root",
	"weight":0
} */;
```

### 删除数据源

```sql
/*+ mycat:dropDataSource{
	"dbType":"mysql",
	"idleTimeout":60000,
	"initSqls":[],
	"initSqlsGetConnection":true,
	"instanceType":"READ_WRITE",
	"maxCon":1000,
	"maxConnectTimeout":3000,
	"maxRetryCount":5,
	"minCon":1,
	"name":"newDs",
	"type":"JDBC",
	"weight":0
} */;
```

### 显示数据源

```sql
/*+ mycat:showDataSources{} */
```

### 创建集群

```sql
/*! mycat:createCluster{
	"clusterType":"MASTER_SLAVE",
	"heartbeat":{
		"heartbeatTimeout":1000,
		"maxRetry":3,
		"minSwitchTimeInterval":300,
		"slaveThreshold":0
	},
	"masters":[
		"dw0" //主节点
	],
	"maxCon":2000,
	"name":"c0",
	"readBalanceType":"BALANCE_ALL",
	"replicas":[
		"dr0" //从节点
	],
	"switchType":"SWITCH"
} */;
```

### 删除集群

```sql
/*! mycat:dropCluster{
	"name":"testAddCluster"
} */;
```

### 显示集群

```sql
/*+ mycat:showClusters{} */
```



### 创建Schema

确保原型库上,存在test_add_Schema物理库,以下注解才能正常运行.

```sql
/*+ mycat:createSchema{
	"customTables":{},
	"globalTables":{},
	"normalTables":{},
	"schemaName":"test_add_Schema",
	"shardingTables":{},
	"targetName":"prototype"
} */;
```



建表注释可以参考

[schema配置](https://www.yuque.com/ccazhw/ml3nkf/00d1d4fc145edf34b2d13fcd95908722)

以下注解相当于把配置推送到mycat中进行更新



### 创建单表(用于读写分离,映射物理表)

```sql
/*+ mycat:createTable{
	"normalTable":{
		"createTableSQL":"create table normal(id int)",
		"dataNode":{
			"schemaName":"testSchema", //物理库
			"tableName":"normal", //物理表
			"targetName":"prototype" //目标
		}
	},
	"schemaName":"testSchema",//逻辑库
	"tableName":"normal" //逻辑表
} */;
```

#### 1.18后

```sql
/*+ mycat:createTable{
	"normalTable":{
		"createTableSQL":"create table normal(id int)",
		"locality":{
			"schemaName":"testSchema", //物理库
			"tableName":"normal", //物理表
			"targetName":"prototype" //目标
		}
	},
	"schemaName":"testSchema",//逻辑库
	"tableName":"normal" //逻辑表
} */;
```

当目标是集群的时候,自动进行读写分离,根据集群配置把查询sql根据事务状态发送到从或主数据源,如果目标是数据源,就直接发送sql到这个数据源.在Mycat2中,是否使用Mycat的集群配置应该是整体的架构选项,只能选其一.当全体目标都是数据源,要么全体目标都是集群.前者一般在数据库集群前再部署一个SLB服务,Mycat访问这个SLB服务,这个SLB服务实现读写分离和高可用.后者则是Mycat直接访问数据库,Mycat负责读写分离和集群高可用.当配置中出现集群和数据源的情况,尽量配置成他们的表的存储节点在一个物理库的实例中没有交集,这样可以避免因为多使用连接导致事务一致性和隔离级别破坏产生的问题.





### 创建全局表

```sql
/*+ mycat:createTable{
	"globalTable":{
		"createTableSQL":"create table global(id int)",
		"dataNodes":[
			{
				"targetName":"prototype"
			}
		]
	},
	"schemaName":"testSchema",
	"tableName":"global"
} */;
```



#### 1.18后

```sql
/*+ mycat:createTable{
	"globalTable":{
		"createTableSQL":"create table global(id int)",
		"broadcast":[
			{
				"targetName":"prototype"
			}
		]
	},
	"schemaName":"testSchema",
	"tableName":"global"
} */;
```



### 创建范围分表

```sql
/*+ mycat:createTable{
	"schemaName":"testSchema",
	"shadingTable":{
		"createTableSQL":"create table sharding(id int)",
		"dataNode":{
			"schemaNames":"testSchema",
			"tableNames":"sharding",
			"targetNames":"prototype"
		},
		"function":{
			"clazz":"io.mycat.router.mycat1xfunction.PartitionConstant",
			"properties":{
				"defaultNode":"0",
				"columnName":"id"
			}
		}
	},
	"tableName":"sharding"
} */;
```



#### 1.18后

```sql
/*+ mycat:createTable{
	"schemaName":"testSchema",
	"shardingTable":{
		"createTableSQL":"create table sharding(id int)",
		"partition":{
			"schemaNames":"testSchema",
			"tableNames":"sharding",
			"targetNames":"prototype"
		},
		"function":{
			"clazz":"io.mycat.router.mycat1xfunction.PartitionConstant",
			"properties":{
				"defaultNode":"0",
				"columnName":"id"
			}
		}
	},
	"tableName":"sharding"
} */;
```



### 显示session引用的IO缓冲块计数

```sql
/*+ mycat:showBufferUsage{}*/
```

### 显示用户

```sql
/*+ mycat:showUsers{}*/
```

### 显示schema

```sql
/*+ mycat:showSchemas{}*/
```

### 显示调度器

```sql
/*+ mycat:showSchedules{}*/
```

### 显示心跳配置

```sql
/*+ mycat:showHeartbeats{}*/
```

### 显示心跳状态

```sql
/*+ mycat:showHeartbeatStatus{}*/
```

### 显示实例状态

```sql
/*+ mycat:showInstances{}*/
```

### 显示Reactor状态

```sql
/*+ mycat:showReactors{}*/
```

### 显示线程池状态

```sql
/*+ mycat:showThreadPools{}*/
```

### 显示表

```sql
/*+ mycat:showTables{"schemaName":"mysql"}*/
```

### 显示mycat连接

```sql
/*+ mycat:showConnections{}*/
```

### 显示存储节点

```sql
/*+ mycat:showDataNodes{//1.18前
	"schemaName":"db1",
	"tableName":"normal"
} */;


/*+ mycat:showTopology{//1.18后
	"schemaName":"db1",
	"tableName":"normal"
} */;
```

### 刷新文件

```sql
/*+ mycat:repairPhysicalTable{} */;
```





### 重置配置（约等于删库，不要用！）

```sql
-- /*+ mycat:re--setConfig{} */
```

### 





## 集群搭建

### 主从集群

#### 数据源 datasource

130-3306.datasource.json

```json
{
    "dbType":"mysql",
    "idleTimeout":60000,
    "initSqls":[],
    "initSqlsGetConnection":true,
    "instanceType":"READ_WRITE",
    "maxCon":1000,
    "maxConnectTimeout":3000,
    "maxRetryCount":5,
    "minCon":1,
    "name":"130-3306",
    "password":"123456",
    "type":"JDBC",
    "url":"jdbc:mysql://192.168.137.130:3306?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
    "user":"root",
    "weight":0
}
```



128-3306.datasource.json

```
{
    "dbType":"mysql",
    "idleTimeout":60000,
    "initSqls":[],
    "initSqlsGetConnection":true,
    "instanceType":"READ_WRITE",
    "maxCon":1000,
    "maxConnectTimeout":3000,
    "maxRetryCount":5,
    "minCon":1,
    "name":"128-3306",
    "password":"123456",
    "type":"JDBC",
    "url":"jdbc:mysql://192.168.137.128:3306?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
    "user":"root",
    "weight":0
}
```



#### 集群 cluster

ms.cluster.json

```json
{
    "clusterType":"MASTER_SLAVE",
    "heartbeat":{
        "heartbeatTimeout":1000,
        "maxRetry":3,
        "minSwitchTimeInterval":300,
        "slaveThreshold":0
    },
    "masters":[
        "128-3306"
    ],
    "replicas":[
        "130-3306"
    ],
    "maxCon":200,
    "name":"ms",
    "readBalanceType":"BALANCE_ALL",
    "switchType":"SWITCH"
}
```



#### 库 schema

test.schema.json

```json
{
    "customTables":{},
    "globalTables":{},
    "normalProcedures":{},
    "normalTables":{
        "travelrecord":{
            "createTableSQL":"CREATE TABLE db1.`travelrecord` (\n\t`id` bigint NOT NULL AUTO_INCREMENT,\n\t`user_id` varchar(100) DEFAULT NULL,\n\t`traveldate` date DEFAULT NULL,\n\t`fee` decimal(10, 0) DEFAULT NULL,\n\t`days` int DEFAULT NULL,\n\t`blob` longblob,\n\tPRIMARY KEY (`id`),\n\tKEY `id` (`id`)\n) ENGINE = InnoDB CHARSET = utf8",
            "locality":{
                "schemaName":"test",
                "tableName":"travelrecord",
                "targetName":"ms"
            }
        }
    },
    "schemaName":"tset",
    "shardingTables":{},
    "views":{}
}
```







### 切片集群

数据源、集群、库相关配置不再赘述，与主从集群搭建一样。



#### 创建表

在 Mycat 下执行

```sql
CREATE TABLE db1.`travelrecord` (
  `id` bigint NOT NULL AUTO_INCREMENT,
  `user_id` varchar(100) DEFAULT NULL,
  `traveldate` date DEFAULT NULL,
  `fee` decimal(10,0) DEFAULT NULL,
  `days` int DEFAULT NULL,
  `blob` longblob,
  PRIMARY KEY (`id`),
  KEY `id` (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 dbpartition by mod_hash(id) tbpartition by mod_hash(id) tbpartitions 2 dbpartitions 2;
```

 dbpartition by mod_hash(id) tbpartition by mod_hash(id) tbpartitions 2 dbpartitions 2;就是分库分表语法

- mod_hash(id)：根据表中 id 进行hash mod计算所在库。
- dbpartition：指定分库算法
- tbpartition ：指定分表算法
- tbpartitions ：分成几个库
- dbpartitions ：分成几个表



#### 插入数据

```mysql
INSERT INTO `travelrecord2`(`user_id`, `traveldate`, `fee`, `days`, `blob`) VALUES ( '1', '2022-05-05', 1, 1, NULL);
```

#### 查看数据

```mysql
select * from travelrecord2 ;
```

主键生成默认策略为雪花算法，如果需要修改为自增需要设置序列。



#### 创建序列

##### 初始化序列表

只有第一次使用序列功能的时候才需要初始化，主要功能就是使用 db 来存储 id。

在 Mycat 的 prototypeDs.datasource.json 中的库初始化 SQL，这个库必须要是Mycat 中的一个虚拟库，我们可以单独创建一个这样的库来存储序列。

在物理库运行 `dbseq.sql`





##### 添加序列记录

tset_travelrecord2 的含义是 逻辑库名_逻辑表名

```mysql
INSERT INTO `MYCAT_SEQUENCE`(`name`, `current_value`, `increment`) VALUES ('tset_travelrecord2', 1, 1);
```

- name：序列名
- current_value：当前值
- increment：步长



##### 设置相应的序列

设置指定逻辑库逻辑表的 id 规则

```sql
/*+ mycat:setSequence{
"name":"tset_travelrecord2",
"clazz":"io.mycat.plug.sequence.SequenceMySQLGenerator",
"name":"tset_travelrecord2",
  "targetName": "prototype",
  "schemaName":"db1"
  } */;
```



##### 重新插入数据查看

值自增且均匀分布在每个表中了。





#### 批量数据插入

##### JavaDemo

```java

/**
 * Mycat 分库分表插入数据
 *
 * @author LZQ
 */
public class MyCatDemo {
    private static BasicDataSource mycatDatasource;
    private static JdbcTemplate mycatTemplate;

    // 静态代码块,设置连接数据库的参数
    static {
        mycatDatasource = new BasicDataSource();
        mycatDatasource.setDriverClassName("com.mysql.jdbc.Driver");
        mycatDatasource.setUrl("jdbc:mysql://192.168.137.132:8066/tset");
        mycatDatasource.setUsername("root");
        mycatDatasource.setPassword("123456");
        mycatTemplate = new JdbcTemplate(mycatDatasource);
    }

    public static void main(String[] args) throws InterruptedException {
        String sql = "INSERT INTO `travelrecord2`(`user_id`, `traveldate`, `fee`, `days`, `blob`) VALUES ( '1', '2022-05-05', 1, 1, NULL)";
        int n = 100000;
        for (int i = 0; i < n; i++) {
            mycatTemplate.execute(sql);
        }
    }
}
```



##### 测试结果

| 表名            | 记录个数 |
| --------------- | -------- |
| travelrecord2_0 | 25000    |
| travelrecord2_1 | 25000    |
| travelrecord2_2 | 25000    |
| travelrecord2_3 | 25000    |

