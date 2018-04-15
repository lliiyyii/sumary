# 日志收集、存储服务器
> 感觉做好会是很棒的，就记录一下啦啦啦

## 项目概述
实现一个多进程事件驱动的日志收集、存储服务器。
UDP端口上接收来自客户端的日志并实现持久存储，采用多进程+EPOLL机制，redis缓存，glusterfs持久存储方案。

## ubuntu 搭建Hadoop
### 准备
 Java环境
### 安装hadoop
- 下载hadoop 安装包
Hadoop 的安装包从这里可以下载: http://mirror.bit.edu.cn/apache/hadoop/common/
找到链接后使用``` wget ```下载
- 解压
  ```tar -zxf ```
- 默认的安装就是单点配置的
### Hadoop伪分布式配置
Hadoop 可以在单节点上以伪分布式的方式运行，Hadoop 进程以分离的 Java 进程来运行，节点既作为 NameNode 也作为 DataNode，同时，读取的是 HDFS 中的文件。
- 修改配置文件 /etc/hadoop/core-site.xml 将
```
<configuration>
</configuration>
```
修改为下面配置:
```
<configuration>
        <property>
             <name>hadoop.tmp.dir</name>
             <value>file:/usr/local/hadoop/tmp</value>
             <description>Abase for other temporary directories.</description>
        </property>
        <property>
             <name>fs.defaultFS</name>
             <value>hdfs://localhost:9000</value>
        </property>
</configuration>
```
- 修改配置文件 hdfs-site.xml：
```
<configuration>
        <property>
             <name>dfs.replication</name>
             <value>1</value>
        </property>
        <property>
             <name>dfs.namenode.name.dir</name>
             <value>file:/usr/local/hadoop/tmp/dfs/name</value>
        </property>
        <property>
             <name>dfs.datanode.data.dir</name>
             <value>file:/usr/local/hadoop/tmp/dfs/data</value>
        </property>
       <property>
		        <name>dfs.permissions</name>
		        <value>false</value>
	      </property>
</configuration>
 ```
- 修改配置文件sbin/start-dfs.sh和stop-dfs.sh
最前面添加
```
HDFS_DATANODE_USER=root
HADOOP_SECURE_SECURE_USER=hdfs
HDFS_NAMENODE_USER=root
HDFS_SECONDARYNAMENODE_USER=root
```


- 修改文件hadoop-env.sh(不在安装hadoop的地方，用find -name hadoop-env.sh)
使用vi打开后寻址注释了的JAVA添加JAVA_HOME

- ***bug：hadoop启动时提示 Permission denied (publickey,password,keyboard-interactive)***

	1、ssh-keygen -t rsa  一路回车键

	2、cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys



