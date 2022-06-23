## FastDFS安装

#### 1.安装FastDFS依赖

 	 FastDFS是C语言开发的应用。安装必须使用 make , cmake 和 gcc编译器。

```
# yum install -y make cmake gcc gcc-c++
```

#### 2 上传并解压libfastcommon-master

​	上传libfastcommon-master 到 /usr/local/tmp下。 libfastcommon是从FastDFS和FastDHT中提取出来的公共C函数库

​	解压 libfastcommon-master.zip 由于是zip文件所以要使用 unzip命令

```
# cd /usr/local/tmp
# unzip libfastcommon-master.zip
```

#### 3 编译并安装

​	libfastcommon没有提供make命令安装文件。使用的是shell脚本执行编译和安装。shell脚本为 make.sh

​	进入解压后的文件

```
# cd libfastcommon-master
```

​	编译

```
#./make.sh
```

​	安装	

```
#./make.sh install
```

​	有固定的默认安装位置。在/usr/lib64 和  /usr/include/fastcommon两个目录中

#### 4 创建软连接

​	 因为FastDFS 主程序设置的lib目录是 /usr/local/lib， 所以需要创建软连接

```
# ln -s /user/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
# ln -s /usr/local/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
```

#### 5 上传并解压FastDFS主程序

​	 上传 FastDFS_v5.08.tar.gz 到 /usr/local/tmp下后解压

```
# cd /usr/local/tmp
# tar zxf FastDFS_v5.08.tar.gz
```

#### 6 编译并安装FastDFS

​	进入到解压后的FastDFS文件中

```
# cd FastDFS
```

​	编译

```
# ./make.sh
```

​	安装

```
# ./make.sh install
```

​	安装后 FastDFS主程序所在的位置是

​	/usr/bin  可执行文件所在的位置

​	/etc/fdfs  配置文件所在的位置

​	/usr/bin  主程序代码所在位置

​	/usr/include/fastdfs 包含一些插件组所在的位置

#### 配置tracker

##### 1 复制配置文件

​	进入到 /etc/fdfs 中 ， 把tracker配置文件复制一份

```
# cd /etc/fdfs
# cp tracker.conf.sample tracker.conf
```

##### 2 创建数据目录

​	创建放置 tracker数据的目录

```
# mkdir -p /usr/local/fastdfs/tracker
```

#####  3 修改配置文件

​	修改 tracker.conf 设置 tracker 内容存储目录

```
base_path=/usr/local/fastdfs/tracker
#vim tracker.conf
```

 	默认端口 22122   不需要修改

##### 4 启动服务 

```
# service fdfs_trackerd start
```

​	启动成功后， 配置文件中 base_path 指向的目录出现 FastDFS服务相关数据目录(data目录， logs 目录)

##### 5 查看服务运行状态

```
# service fdfs_trackerd status
```

​	如果显示 is running 表示正常运行。

##### 6 关闭防火墙

```
# service iptables stop
# chkconfig iptables off
```

####  配置storage

 	  storage可以和tracker不在同一台服务器上。示例中把storage和tracker安装在同一台服务器上了。

#####     1 复制配置文件

​	进入到 /etc/fdfs, 把 storage 配置文件复制一份

```
# cd /etc/fdfs
# cp storage.conf.sample storage.conf
```

#####   2 创建目录

​	 创建两个目录， 把base用于存储基础数据和日志，store用于存储上传数据。

```
# mkdir -p /usr/local/fastdfs/storage/base
# mkdir -p /usr/local/fastdfs/storage/store
```

##### 3 修改配置文件

​	storage.conf配置文件用于描述存储服务的行为，需要进行下述修改

```
# vim /etc/fdfs/storage.conf
```

​	配置内容如下：

```
base_path=/usr/local/fastdfs/storage/base
store_path0=/usr/local/fastdfs/storage/store
tracker_server=tracker 服务IP：22122
```

​	base_path - 基础路径。用于保存storage server 基础数据内容和日志内容的目录。

​	store_path0 - 存储路径。是用于保存FastDFS中存储文件的目录，就是要创建256*256个子目录的位置。

​	base_path 和 store_path0 可以使用同一个目录。

​	tracker_server - 跟踪服务器位置。就是跟踪服务器的IP和端口。

​	启动服务

```
# service fdfs_storaged start
```

​	启动成功后，配置文件中base_path 指向的目录中出现FastDFS服务相关数据目录（data目录、logs目录）配置文件中的store_path0指向的目录中同样出现FastDFS存储相关数据录（data目录）。其中$store_path0/data/目录中默认创建若干子孙目录（两级目录层级总计256*256个目录），是用于存储具体文件数据的。

​	Storage 服务器启动比较慢，因为第一次启动的时候，需要创建256*256个目录。

​	查看启动状态

```
# service fdfs_storaged status
```

### 文件上传流程

#### 1     时序图

![](E:/马士兵架构/二、（P5）Java工程师（入门课）/08.八、了解分布式基础/分布式资料/FastDFS+Nginx/文档/FastDFS+NGINX.assets/FastDFS+NGINX-04.jpg)

### 2     流程说明

1.  客户端访问Tracker
2.  Tracker 返回Storage的ip和端口
3.  客户端直接访问Storage，把文件内容和元数据发送过去。
4.  Storage返回文件存储id。包含了组名和文件名

![](E:/马士兵架构/二、（P5）Java工程师（入门课）/08.八、了解分布式基础/分布式资料/FastDFS+Nginx/文档/FastDFS+NGINX.assets/FastDFS+NGINX-05.jpg)



###  Fastdfs-java-client

#### 1     添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>cn.bestwu</groupId>
        <artifactId>fastdfs-client-java</artifactId>
        <version>1.27</version>
    </dependency>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.4</version>
    </dependency>
</dependencies> `     
```

####  2     编写配置文件

​	文件名：fdfs_client.conf

​	修改成自己的tracker服务器ip

```
connect_timeout = 10
network_timeout = 30
charset = UTF-8
http.tracker_http_port = 8080
tracker_server = 192.168.93.10:22122   
```

#### 3    导入工具类

​	在com.utils.FastDFSClient 下粘贴配置工具类

```java
package com.msb.utils;
import java.io.ByteArrayInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

import org.apache.commons.lang3.StringUtils;
import org.csource.common.NameValuePair;
import org.csource.fastdfs.ClientGlobal;
import org.csource.fastdfs.StorageClient;
import org.csource.fastdfs.StorageClient1;
import org.csource.fastdfs.StorageServer;
import org.csource.fastdfs.TrackerClient;
import org.csource.fastdfs.TrackerServer;

/**
 * FastDFS分布式文件系统操作客户端.
 */
public class FastDFSClient {

   private static final String CONF_FILENAME = Thread.currentThread().getContextClassLoader().getResource("").getPath() + "fdfs_client.conf";

   private static StorageClient storageClient = null;

   /**
    * 只加载一次.
    */
   static {
      try {
         ClientGlobal.init(CONF_FILENAME);
         TrackerClient trackerClient = new TrackerClient(ClientGlobal.g_tracker_group);
         TrackerServer trackerServer = trackerClient.getConnection();
         StorageServer storageServer = trackerClient.getStoreStorage(trackerServer);
         storageClient = new StorageClient(trackerServer, storageServer);
      } catch (Exception e) {
         e.printStackTrace();
      }
   }
   
   /**
    * 
    * @param inputStream
    *    上传的文件输入流
    * @param fileName
    *    上传的文件原始名
    * @return
    */
   public static String[] uploadFile(InputStream inputStream, String fileName) {
      try {
         // 文件的元数据
         NameValuePair[] meta_list = new NameValuePair[2];
         // 第一组元数据，文件的原始名称
         meta_list[0] = new NameValuePair("file name", fileName);
         // 第二组元数据
         meta_list[1] = new NameValuePair("file length", inputStream.available()+"");
         // 准备字节数组
         byte[] file_buff = null;
         if (inputStream != null) {
            // 查看文件的长度
            int len = inputStream.available();
            // 创建对应长度的字节数组
            file_buff = new byte[len];
            // 将输入流中的字节内容，读到字节数组中。
            inputStream.read(file_buff);
         }
         // 上传文件。参数含义：要上传的文件的内容（使用字节数组传递），上传的文件的类型（扩展名），元数据
         String[] fileids = storageClient.upload_file(file_buff, getFileExt(fileName), meta_list);
         return fileids;
      } catch (Exception ex) {
         ex.printStackTrace();
         return null;
      }
   }

   /**
    * 
    * @param file
    *            文件
    * @param fileName
    *            文件名
    * @return 返回Null则为失败
    */
   public static String[] uploadFile(File file, String fileName) {
      FileInputStream fis = null;
      try {
         NameValuePair[] meta_list = null; // new NameValuePair[0];
         fis = new FileInputStream(file);
         byte[] file_buff = null;
         if (fis != null) {
            int len = fis.available();
            file_buff = new byte[len];
            fis.read(file_buff);
         }

         String[] fileids = storageClient.upload_file(file_buff, getFileExt(fileName), meta_list);
         return fileids;
      } catch (Exception ex) {
         return null;
      }finally{
         if (fis != null){
            try {
               fis.close();
            } catch (IOException e) {
               e.printStackTrace();
            }
         }
      }
   }

   /**
    * 根据组名和远程文件名来删除一个文件
    * 
    * @param groupName
    *            例如 "group1" 如果不指定该值，默认为group1
    * @param remoteFileName
    *            例如"M00/00/00/wKgxgk5HbLvfP86RAAAAChd9X1Y736.jpg"
    * @return 0为成功，非0为失败，具体为错误代码
    */
   public static int deleteFile(String groupName, String remoteFileName) {
      try {
         int result = storageClient.delete_file(groupName == null ? "group1" : groupName, remoteFileName);
         return result;
      } catch (Exception ex) {
         return 0;
      }
   }

   /**
    * 修改一个已经存在的文件
    * 
    * @param oldGroupName
    *            旧的组名
    * @param oldFileName
    *            旧的文件名
    * @param file
    *            新文件
    * @param fileName
    *            新文件名
    * @return 返回空则为失败
    */
   public static String[] modifyFile(String oldGroupName, String oldFileName, File file, String fileName) {
      String[] fileids = null;
      try {
         // 先上传
         fileids = uploadFile(file, fileName);
         if (fileids == null) {
            return null;
         }
         // 再删除
         int delResult = deleteFile(oldGroupName, oldFileName);
         if (delResult != 0) {
            return null;
         }
      } catch (Exception ex) {
         return null;
      }
      return fileids;
   }

   /**
    * 文件下载
    * 
    * @param groupName 卷名
    * @param remoteFileName 文件名
    * @return 返回一个流
    */
   public static InputStream downloadFile(String groupName, String remoteFileName) {
      try {
         byte[] bytes = storageClient.download_file(groupName, remoteFileName);
         InputStream inputStream = new ByteArrayInputStream(bytes);
         return inputStream;
      } catch (Exception ex) {
         return null;
      }
   }
   
   public static NameValuePair[] getMetaDate(String groupName, String remoteFileName){
      try{
         NameValuePair[] nvp = storageClient.get_metadata(groupName, remoteFileName);
         return nvp;
      }catch(Exception ex){
         ex.printStackTrace();
         return null;
      }
   }

   /**
    * 获取文件后缀名（不带点）.
    * 
    * @return 如："jpg" or "".
    */
   private static String getFileExt(String fileName) {
      if (StringUtils.isBlank(fileName) || !fileName.contains(".")) {
         return "";
      } else {
         return fileName.substring(fileName.lastIndexOf(".") + 1); // 不带最后的点
      }
   }
}   
```

####  4     编写测试代码

​	随意新建一个包含主方法的类。com.msb.MyMain

```java
public class MyMain {
    public static void main(String[] args) {
        try {
            File file = new File("D:/b.png");
            InputStream is = new FileInputStream(file);
            String fileName = UUID.randomUUID().toString()+".png";
            String[] result = FastDFSClient.uploadFile(is, fileName);
            System.out.println(Arrays.toString(result));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}   
```

###  文件下载

#### 1     时序图

![](E:/马士兵架构/二、（P5）Java工程师（入门课）/08.八、了解分布式基础/分布式资料/FastDFS+Nginx/文档/FastDFS+NGINX.assets/FastDFS+NGINX-06.jpg)

#### 2     下载说明

1. client询问tracker下载文件的storage，参数为文件标识（组名和文件名）；

2. tracker返回一台可用的storage；

3. client直接和storage通讯完成文件下载。

#### 3     代码实现

​	直接使用工具方法完成下载。

```
try {
    InputStream is = FastDFSClient.downloadFile("group1", "M00/00/00/wKg0gF3zAKCARs6kAAASjQVYlWA098.png");
    OutputStream os = new FileOutputStream(new File("D:/jqk.png"));
    int index = 0 ;
    while((index = is.read())!=-1){
        os.write(index);
    }
    os.flush();
    os.close();
    is.close();
} catch (IOException e) {
    e.printStackTrace();
}
```

###  

**关注我的公众号回复 002 ，或添加我的微信 即可获取 FastDFS 相关资源**

![](../../../media/pictures/gzh.jpg)