## 安装 Redis

### Windows





### Linux

Redis 安装非常简单，只需要几行命令就可以了。[官网](https://redis.io/download)也给出了具体的命令。

#### 下载并编译

```bash
# 下载 redis 的压缩资源包
wget https://download.redis.io/releases/redis-6.2.6.tar.gz
# 解压缩文件
tar xzf redis-6.2.6.tar.gz
# 进入源码路径
cd redis-6.2.6
# 编译
make
```

##### 可能出现的异常

- 执行make命令时提示：CC adlist.o /bin/sh: cc: 未找到命令

  ```bash
  # 缺失 gcc 环境
  yum -y install gcc automake autoconf libtool make
  ```

- jemalloc/jemalloc.h：没有那个文件或目录

  ```bash
  # 编译时指定参数，也就是使用 libc 内存分配器，有些系统没有 jemalloc
  make MALLOC=libc
  ```



#### 启动服务端

```bash
# 启动 src 路径下的 redis-server 文件
src/redis-server

# 如果启动后 Java 客户端连接爆出安全问题，可以在启动时指定参数
src/redis-server --protected-mode no

# 或者在客户端执行命令
config set protected-mode no
```



#### 启动客户端

```bash
# 启动 src 路径下的 redis-cli 文件
src/redis-cli
# 如果现实 redis > 则代表已经完成了
redis> set foo bar
OK
redis> get foo
"bar"
```



### Docker

一般来说不推荐在容器下运行数据库，可能会导致文件异常丢失等。

- 查看 Redis 镜像

```
docker search redis
```

- 拉取最新的 Redis 镜像

```
docker pull redis:latest
```

- 查看本地镜像

```
docker images
```

- 运行容器

```
docker run -itd --name redis-test -p 6379:6379 redis
```

- 运行退出的Docker

```
docker start  <CONTAINER ID>
例如：docker start 2c2085e096b5
```

- redis-cli 连接测试使用 redis 服务

```
docker exec -it redis-test /bin/bash
```

- 安装完毕