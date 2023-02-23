# docker开启安全远程

## 编写批处理文件

您如果已经有了docker-ca.sh，批处理文件。就直接编辑。如果你还没有批处理文件，则自己编写一个sh批处理文件,docker-ca.sh。

例如这里我文件放在`/home/dockerca`目录

```shell
sudo mkdir /home/dockerca
sudo vim /home/dockerca/docker-ca.sh
```

内容如下

内容如下

```shell
#!/bin/bash
  
#相关配置信息（这里修改成你自己服务器信息）
SERVER=""  	#输入你的服务器外网IP
PASSWORD=""      	#输入你的密码
COUNTRY="CN"				#输入你的国家
STATE="AnHui"				#输入你的省份
CITY="HeFei"				#输入你的城市
ORGANIZATION="wxsz"			#输入你的组织
ORGANIZATIONAL_UNIT="Dev"	#输入你的单位
EMAIL="2453893123@qq.com"	#输入你的邮箱

###开始生成文件###
echo "开始生成文件"

#切换到生产密钥的目录
if [ ! -d "/opt/docker_ca" ]; then
mkdir -p /opt/docker_ca
fi
cd /opt/docker_ca
#生成ca私钥(使用aes256加密)
openssl genrsa -aes256 -passout pass:$PASSWORD  -out ca-key.pem 4096
#生成ca证书，填写配置信息
openssl req -new -x509 -passin "pass:$PASSWORD" -days 3650 -key ca-key.pem -sha256 -out ca.pem -subj "/C=$COUNTRY/ST=$STATE/L=$CITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$SERVER/emailAddress=$EMAIL"

#生成server证书私钥文件
openssl genrsa -out server-key.pem 4096
#生成server证书请求文件
openssl req -subj "/CN=$SERVER" -sha256 -new -key server-key.pem -out server.csr
#配置白名单，多个用逗号隔开
sh -c 'echo subjectAltName = IP:'$SERVER',IP:0.0.0.0 >> extfile.cnf'
#把 extendedKeyUsage = serverAuth 键值设置到extfile.cnf文件里，限制扩展只能用在服务器认证
sh -c 'echo extendedKeyUsage = serverAuth >> extfile.cnf'
#使用CA证书及CA密钥以及上面的server证书请求文件进行签发，生成server自签证书
openssl x509 -req -days 3650 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -\CAcreateserial -out server-cert.pem -extfile extfile.cnf

#生成client证书RSA私钥文件
openssl genrsa -out key.pem 4096
#生成client证书请求文件
openssl req -subj '/CN=client' -new -key key.pem -out client.csr
#继续设置证书扩展属性
sh -c 'echo extendedKeyUsage = clientAuth >> extfile.cnf'
#生成client自签证书（根据上面的client私钥文件、client证书请求文件生成）
openssl x509 -req -days 3650 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem -passin "pass:$PASSWORD" -\CAcreateserial -out cert.pem -extfile extfile.cnf

#更改密钥权限
chmod 0400 ca-key.pem key.pem server-key.pem
#更改密钥权限
chmod 0444 ca.pem server-cert.pem cert.pem
#删除无用文件
rm client.csr server.csr

echo "生成文件完成"

```





## 执行批注里文件

```shell
#进入目录
cd /home/dockerca
#给批处理文件授权(如果已有权限则直接执行)
sudo chmod 777  docker-ca.sh
#执行
./docker-ca.sh
```

执行结果如下：

```shell
root@iZuf67m7dewb3m23qvnb8hZ:/home/dockerca# ./docker-ca.sh 
开始生成文件
Generating RSA private key, 4096 bit long modulus (2 primes)
.......................++++
........................................................................++++
e is 65537 (0x010001)
Generating RSA private key, 4096 bit long modulus (2 primes)
....................................++++
........++++
e is 65537 (0x010001)
Signature ok
subject=CN = 101.132.147.161
Getting CA Private Key
Generating RSA private key, 4096 bit long modulus (2 primes)
.................................................................................................................................................++++
...................................++++
e is 65537 (0x010001)
Signature ok
subject=CN = client
Getting CA Private Key
生成文件完成
```



## 自己编写批处理执行异常

shell脚本报错/bin/bash^M: bad interpreter: No such file or directory，通过查阅资料得知，shell脚本格式必须是unix才行，但我这个脚本是在windows上编写完成传到Linux服务器上的，所以一执行就报错.

解决方法：

```shell
#替换特殊字符
sed -i "s/\r//" docker-ca.sh  
```

再次执行就行了





## 修改docker 添加证书信息

```shell
#编辑docker服务
sudo vim /lib/systemd/system/docker.service
```

将 `ExecStart` 属性值进行替换如下：

```shell
ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=/opt/docker_ca/ca.pem --tlscert=/opt/docker_ca/server-cert.pem  --tlskey=/opt/docker_ca/server-key.pem  -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H fd:// --containerd=/run/containerd/containerd.sock
```

![](F:\github\javaSpace\docs\docker\Docker安全远程.assets\01.png)



重新加载daemon并重启docker

```shell
systemctl daemon-reload
systemctl restart docker
```



## 保存认证文件给客户端使用

通过ftp工具将认证证书下载到本地给客户端使用

![](F:\github\javaSpace\docs\docker\Docker安全远程.assets\02.png)

