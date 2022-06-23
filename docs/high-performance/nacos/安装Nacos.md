## 安装 Nacos

Nacos资源网址 https://github.com/alibaba/nacos/releases



### Windows

下载  [nacos-server-2.1.0.zip](https://github.com/alibaba/nacos/releases/download/2.1.0/nacos-server-2.1.0.zip)



解压



运行



### Linux

下载安装包

```bash
wget https://github.com/alibaba/nacos/releases/download/2.1.0/nacos-server-2.1.0.tar.gz
```

解压

```bash
tar -xvf nacos-server-2.1.0.tar.gz
```

进入 bin 目录

```bash
cd nacos/bin
```

运行

```bash
# 单机启动
sh startup.sh -m standalone
```



### Docker







## 验证

进入 http://ip:8848/nacos 输入默认账号密码 nacos/nacos



## 其他

### 启动服务器

#### Linux/Unix/Mac

启动命令(standalone代表着单机模式运行，非集群模式):

```bash
sh startup.sh -m standalone
```

如果您使用的是ubuntu系统，或者运行脚本报错提示[[符号找不到，可尝试如下运行：

```bash
bash startup.sh -m standalone
```

#### Windows

启动命令(standalone代表着单机模式运行，非集群模式):

```bash
startup.cmd -m standalone
```



### 关闭服务器

#### Linux/Unix/Mac

```bash
sh shutdown.sh
```

#### Windows

```bash
shutdown.cmd
```

或者双击shutdown.cmd运行文件。



### 修改为mysql存储配置

#### 修改conf/application.properties

核心配置

```properties
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=zgqazxsw
```

#### 导入sql文件

nacos-mysql.sql