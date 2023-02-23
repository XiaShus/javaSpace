## 安装 Docker

### Windows

下载 msi 直接安装



### Linux

- 更新 yum 源

```bash
yum update
```

- 安装 yml 组件

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 设置 yum 源

```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 安装 docker 所需组件

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

- 安装 docker

```bash
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

- 开启 docker

```bash
systemctl start docker
```

- 设置 docker 开机自启

```bash
systemctl enable docker
```

- 查看 docker 版本

```bash
docker version
```



#### 安装 Docker-compose

- 下载 docker-compose

```bash
sudo curl -L "https://get.daocloud.io/docker/compose/releases/download/v2.10.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- 设置执行权限

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

- 查看 docker-compose 版本

```bash
docker-compose --version
```

