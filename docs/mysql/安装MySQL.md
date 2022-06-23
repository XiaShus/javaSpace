## 安装 MySQL

### Windows

下载 msi 直接安装



### Linux

1、检查是否已经安装过mysql

```bash
rpm -qa | grep mysql
```

如果安装，则删除

```bash
rpm -e --nodeps mysql-libs-版本
```

通过whereis mysql 和 find / -name mysql查找，删除相关

```bash
//rpm包安装方式卸载
查包名：rpm -qa|grep -i mysql
删除命令：rpm -e –nodeps 包名
 
//yum安装方式下载
1.查看已安装的mysql
命令：rpm -qa | grep -i mysql
2.卸载mysql
命令：yum remove mysql-community-server-5.6.36-2.el7.x86_64
查看mysql的其它依赖：rpm -qa | grep -i mysql
 
//卸载依赖
yum remove mysql-libs
yum remove mysql-server
yum remove perl-DBD-MySQL
yum remove mysql
```

1.1、检查 物理安装

```bash
find / -name mysql
```

将相关文件删除

```bash
rm -rf /var/lib/mysql
...
```



2、检查mysql用户组和用户，没有则创建

```bash
cat /etc/group | grep mysql
cat /etc/passwd |grep mysql
groupadd mysql
useradd -r -g mysql mysql
```

3、下载mysql包（可到官网寻找其他版本）

```bash
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

解压

```bash
tar xzvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

4、解压完成后，移动该文件到/usr/local/mysql

```bash
mv mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/mysql
创建data目录
mkdir /usr/local/mysql/data
```

5、更改mysql目录下所有的目录及文件夹所属的用户组和用户，以及权限

```bash
chown -R mysql:mysql /usr/local/mysql
chmod -R 755 /usr/local/mysql
```

6、编译安装并初始化mysql,务必记住初始化输出日志末尾的密码（数据库管理员临时密码root@localhost:后的字符串）

```bash
cd /usr/local/mysql/bin
./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql

编译完成后最后有一句：2019-12-05T03:08:28.258928Z 1 [Note] A temporary password is generated for root@localhost: NwLkW5t;ia8k 
NwLkW5t;ia8k是临时密码
```

7、启动mysql服务器

```bash
/usr/local/mysql/support-files/mysql.server start
如果出现如下提示信息
Starting MySQL.Logging to '/usr/local/mysql/data/iZge8dpnu9w2d6Z.err'.
#查询服务
ps -ef|grep mysql
ps -ef|grep mysqld

#结束进程
kill -9 PID
```

8、添加软连接，并重启mysql服务

```bash
ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
service mysql restart
```

9、登录mysql,密码为刚才的临时密码

```mysql
mysql -u root -p

首先安装后，执行任何指令都会提示：
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
将临时密码修改
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER;
刷新权限

mysql> flush privileges;
```

10、开放远程连接

```mysql
mysql> use mysql;
mysql> update user set user.Host='%' where user.User='root';
mysql> flush privileges;
```

11、设置开机自启

```bash
1、将服务文件拷贝到init.d下，并重命名为mysql
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
2、赋予可执行权限
chmod +x /etc/init.d/mysqld
3、添加服务
chkconfig --add mysqld
```





### Docker

* 拉取 MYSQL 镜像
``` linux
docker pull mysql:5.6
```
* 查看本地镜像 是否安装完成 MYSQL 镜像
``` linux
docker images
```
* 启动 MYSQL
``` linux
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql

docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql：5.6
```
* 进入容器
``` linux
docker exec -it mysql bash
```
* 登录 MYSQL
``` linux
mysql -u root -p
# 在用navicat连接MySQL8+时会出现2059错误，这是由于新版本的MySQL使用的是caching_sha2_password验证方式，但此时的navicat还没有支持这种验证方式。
解决方法就是将验证方式改为以前版本(5.7及以下)使用的验证方式mysql_native_password。具体的验证方式可以查看默认数据库'mysql'中user表plugin字段。
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
# localhost 也能 改为 % 则表示所有 ip都可以访问
ALTER USER 'root'@'%' IDENTIFIED BY '123456';
```
* 至此 MYSQL 也算是部署好了
* MYSQL 1251报错
``` mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password'; #修改加密规则 
ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER; #更新一下用户的密码 
FLUSH PRIVILEGES; #刷新权限 
```

Docker安装MySql
https://blog.csdn.net/weixin_39752157/article/details/111883235



本地连接不到虚拟机 MySQL，可以尝试关闭防火墙

```
1:查看防火状态

systemctl status firewalld
service  iptables status

2:暂时关闭防火墙

systemctl stop firewalld
service iptables stop

3:永久关闭防火墙

systemctl disable firewalld
chkconfig iptables off

4:重启防火墙

systemctl enable firewalld
service iptables restart
```

