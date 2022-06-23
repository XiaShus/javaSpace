## 安装 Git

### Windows





### Linux

安装编译 Git 所需要的依赖：

```bash
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```

安装编译源码所需依赖的时候，yum 自动安装了 Git，需要先卸载这个旧版的 Git：

```bash
yum -y remove git
```

下载 Linux 下 Git 的安装包，地址：

```bash
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.22.0.tar.gz
```

解压压缩包：

```bash
tar -zxvf git-2.22.0.tar.gz
```

进入到解压后的文件夹：

```bash
cd git-2.22.0
```

编译 Git 源码：

```bash
make prefix=/usr/local/git all
```

安装 Git 至 `/usr/local/git` 路径：

```bash
make prefix=/usr/local/git install
```

配置环境变量：

```bash
vim /etc/profile
```

在环境变量配置文件底部加上：

```bash
export PATH=$PATH:/usr/local/git/bin
```

刷新环境变量：

```bash
source /etc/profile
```

查看Git是否安装完成：

```bash
git --version
```
