# Introduction

HBase is a data model that is designed to provide quick random access to huge amounts of structured data. This tutorial shows how to set up HBase on Hadoop File Systems.

One can store the data in HDFS through HBase. Data consumer reads/accesses the data in HDFS randomly using HBase. HBase sits on top of the Hadoop File System and provides read and write access.

Applications such as HBase, Cassandra, couchDB, Dynamo, and MongoDB are some of the databases that store huge amounts of data and access the data in a random manner.

**Storage Mechanism in HBase**

HBase is a column-oriented database and the tables in it are sorted by row. The table schema defines only column families, which are the key value pairs. A table have multiple column families and each column family can have any number of columns. Subsequent column values are stored contiguously on the disk. Each cell value of the table has a timestamp. In short, in an HBase:

* Table is a collection of rows.
* Row is a collection of column families.
* Column family is a collection of columns.
* Column is a collection of key value pairs.

# Google Cloud Instance

For the purpose of this tutorial, we use google clound instance virtual machine to demontrate how to install hbase. However, in real life, if you should most likely use [Bigtable](https://cloud.google.com/bigtable/docs/) provided by Google.

> **Bigtable**
> 
> Cloud Bigtable is a fully managed NoSQL database that supports the popular open-source Apache HBase 1.0 API. You can provision Cloud Bigtable instances for your workload, then use the Bigtable HBase client to develop applications using the standard open-source Big Data tools you're familiar with.

But for learning purposes, we will install HBase with a barebone Ubuntu virtual machine. 

![](./create_project.png)

![](./create_instance.png)

![](./firewall.png)

Creating a User

	yzhong_cs@instance-1:~$ sudo useradd hadoop
	yzhong_cs@instance-1:~$ sudo passwd hadoop
	Enter new UNIX password: password
	Retype new UNIX password: password
	passwd: password updated successfully

SSH Setup and Key Generation
SSH setup is required to perform different operations on the cluster such as start, stop, and distributed daemon shell operations. To authenticate different users of Hadoop, it is required to provide public/private key pair for a Hadoop user and share it with different users.

The following commands are used to generate a key value pair using SSH. Copy the public keys form id_rsa.pub to authorized_keys, and provide owner, read and write permissions to authorized_keys file respectively.

	yzhong_cs@instance-1:~$ ssh-keygen -t rsa
	Generating public/private rsa key pair.
	Enter file in which to save the key (/home/yzhong_cs/.ssh/id_rsa): 
	Enter passphrase (empty for no passphrase): 
	Enter same passphrase again: 
	Your identification has been saved in /home/yzhong_cs/.ssh/id_rsa.
	Your public key has been saved in /home/yzhong_cs/.ssh/id_rsa.pub.
	The key fingerprint is:
	SHA256:w6PwtUhfZ/vBwTG3P5kXeP2GuPW0b39yElH4HWAz0VE yzhong_cs@instance-1
	The key's randomart image is:
	+---[RSA 2048]----+
	|             *+oE|
	|            . +o.|
	|              oo+|
	|       .     .o+=|
	|    . . S . o.o=.|
	|     + = = o +oo*|
	|      + o   o ==*|
	|             +o=B|
	|            . .*B|
	+----[SHA256]-----+
	yzhong_cs@instance-1:~$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
	yzhong_cs@instance-1:~$ chmod 0600 ~/.ssh/authorized_keys
	
Verify ssh
	
	yzhong_cs@instance-1:~$ ssh localhost
	The authenticity of host 'localhost (127.0.0.1)' can't be established.
	ECDSA key fingerprint is SHA256:foTRdAvYNamSZnErNmLUAuKC+OL/3YtqkzbdOraTe2c.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
	Welcome to Ubuntu 16.10 (GNU/Linux 4.8.0-51-generic x86_64)
	
	 * Documentation:  https://help.ubuntu.com
	 * Management:     https://landscape.canonical.com
	 * Support:        https://ubuntu.com/advantage
	
	  Get cloud support with Ubuntu Advantage Cloud Guest:
	    http://www.ubuntu.com/business/services/cloud
	
	0 packages can be updated.
	0 updates are security updates.
	
	New release '17.04' available.
	Run 'do-release-upgrade' to upgrade to it.
	
	
	Last login: Sun May 14 02:38:00 2017 from 173.194.90.32

Exit from ssh

	yzhong_cs@instance-1:~$ exit
	
Install Java
	
	yzhong_cs@instance-1:~$ sudo apt-get update
	yzhong_cs@instance-1:~$ sudo sudo apt-get install default-jre
	yzhong_cs@instance-1:~$ sudo apt-get install default-jdk
	yzhong_cs@instance-1:~$ java -version
	openjdk version "1.8.0_131"
	OpenJDK Runtime Environment (build 1.8.0_131-8u131-b11-0ubuntu1.16.10.2-b11)
	OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
	
	
	yzhong_cs@instance-1:~$ export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
	yzhong_cs@instance-1:~$ export PATH=$PATH:$JAVA_HOME/bin

Install Hadoop

	yzhong_cs@instance-1:~$ wget http://apache.mirrors.tds.net/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz
	yzhong_cs@instance-1:~$ tar -xzvf hadoop-2.8.0.tar.gz
	yzhong_cs@instance-1:~$ sudo mv hadoop-2.8.0 /usr/local/hadoop
	
Set `JAVA_HOME`:

	nano /usr/local/hadoop/etc/hadoop/hadoop-env.sh
	
	export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64


Validate hadoop installation by running the example project: 

	yzhong_cs@instance-1:~$ mkdir ~/input
	yzhong_cs@instance-1:~$ cp /usr/local/hadoop/etc/hadoop/*.xml ~/input
	yzhong_cs@instance-1:~$ /usr/local/hadoop/bin/hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.8.0.jar grep ~/input ~/grep_example 'principal[.]*'
	
	yzhong_cs@instance-1:~$ cat ~/grep_example/*
	6       principal
	1       principal.

Reference: https://www.digitalocean.com/community/tutorials/how-to-install-hadoop-in-stand-alone-mode-on-ubuntu-16-04 

Add these to `~/.profile`

	export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
	export PATH=$PATH:$JAVA_HOME/bin
	
	export HADOOP_HOME=/usr/local/hadoop
	export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin

Then 

	yzhong_cs@instance-1:~$ source .profile

Check Hadoop version

	yzhong_cs@instance-1:~$ hadoop version
	Hadoop 2.8.0
	Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 91f2b7a13d1e97be65db92ddabc627cc29ac0009
	Compiled by jdu on 2017-03-17T04:12Z
	Compiled with protoc 2.5.0
	From source with checksum 60125541c2b3e266cbf3becc5bda666
	This command was run using /usr/local/hadoop/share/hadoop/common/hadoop-common-2.8.0.jar


# Configurations

	cd /usr/local/hadoop/etc/hadoop

**core-site.xml**

```
<configuration>
   <property>
      <name>fs.default.name</name>
      <value>hdfs://localhost:9000</value>
   </property>
</configuration>
```


**hdfs-site.xml**

```
<configuration>
   <property>
      <name>dfs.replication</name >
      <value>1</value>
   </property>
	
   <property>
      <name>dfs.name.dir</name>
      <value>file:///home/hadoop/hadoopinfra/hdfs/namenode</value>
   </property>
	
   <property>
      <name>dfs.data.dir</name>
      <value>file:///home/hadoop/hadoopinfra/hdfs/datanode</value>
   </property>
</configuration>
```

**yarn-site.xml**

```
<configuration>
   <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
   </property>
</configuration>
```

**mapred-site.xml**

```
<configuration>
   <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
   </property>
</configuration>
```

Verifying Hadoop Installation

	$ cd ~
	$ sudo su
	$ hdfs namenode -format
	
	...
	...
	...
	17/05/14 20:43:10 INFO namenode.FSImage: Allocated new BlockPoolId: BP-1902175770-10.128.0.3-1494794590744
	17/05/14 20:43:10 INFO common.Storage: Storage directory /home/hadoop/hadoopinfra/hdfs/namenode has been successfully formatted.
	17/05/14 20:43:10 INFO namenode.FSImageFormatProtobuf: Saving image file /home/hadoop/hadoopinfra/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 using no compression
	17/05/14 20:43:10 INFO namenode.FSImageFormatProtobuf: Image file /home/hadoop/hadoopinfra/hdfs/namenode/current/fsimage.ckpt_0000000000000000000 of size 321 bytes saved in 0 seconds.
	17/05/14 20:43:11 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
	17/05/14 20:43:11 INFO util.ExitUtil: Exiting with status 0
	17/05/14 20:43:11 INFO namenode.NameNode: SHUTDOWN_MSG: 
	/************************************************************
	SHUTDOWN_MSG: Shutting down NameNode at instance-2/10.128.0.3
	************************************************************/


Verifying Hadoop dfs

```
yzhong_cs@instance-2:~$ start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-namenode-instance-2.out
localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-datanode-instance-2.out
Starting secondary namenodes [0.0.0.0]
The authenticity of host '0.0.0.0 (0.0.0.0)' can't be established.
ECDSA key fingerprint is SHA256:Wqxi6v86+591GR4M7UhZi8fCMtDk11GTeC1BDBdEXeM.
Are you sure you want to continue connecting (yes/no)? yes
0.0.0.0: Warning: Permanently added '0.0.0.0' (ECDSA) to the list of known hosts.
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-secondarynamenode-instance-2.out
```

```
yzhong_cs@instance-2:~$ start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-namenode-instance-2.out
localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-datanode-instance-2.out
Starting secondary namenodes [0.0.0.0]
The authenticity of host '0.0.0.0 (0.0.0.0)' can't be established.
ECDSA key fingerprint is SHA256:Wqxi6v86+591GR4M7UhZi8fCMtDk11GTeC1BDBdEXeM.
Are you sure you want to continue connecting (yes/no)? yes
0.0.0.0: Warning: Permanently added '0.0.0.0' (ECDSA) to the list of known hosts.
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-secondarynamenode-instance-2.out
yzhong_cs@instance-2:~$ stop-dfs.sh
Stopping namenodes on [localhost]
localhost: no namenode to stop
localhost: no datanode to stop
Stopping secondary namenodes [0.0.0.0]
0.0.0.0: stopping secondarynamenode
yzhong_cs@instance-2:~$ start-dfs.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-namenode-instance-2.out
localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-datanode-instance-2.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-yzhong_cs-secondarynamenode-instance-2.out
```


```
yzhong_cs@instance-2:~$ start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-yzhong_cs-resourcemanager-instance-2.out
localhost: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-yzhong_cs-nodemanager-instance-2.out
```

```
yzhong_cs@instance-2:~$ curl localhost:50070
<!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="REFRESH" content="0;url=dfshealth.html" />
<title>Hadoop Administration</title>
</head>
</html>
```





# Install Hbase

	yzhong_cs@instance-2:~$ wget http://apache.mirror.gtcomm.net/hbase/stable/hbase-1.2.5-bin.tar.gz
	yzhong_cs@instance-2:~$ tar -zxvf hbase-1.2.5-bin.tar.gz 
	yzhong_cs@instance-2:~$ sudo mv hbase-1.2.5 /usr/local/Hbase/

ADD THESE TO `.profile`	

	export HBASE_HOME=/usr/local/Hbase
	export PATH=$PATH:$HBASE_HOME/bin

Edit `JAVA_HOME`

	yzhong_cs@instance-2:~$ nano /usr/local/Hbase/conf/hbase-env.sh
	
	# The java implementation to use.  Java 1.7+ required.
	# export JAVA_HOME=/usr/java/jdk1.6.0/
	export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
	

Configurations:

	yzhong_cs@instance-2:~$ nano /usr/local/Hbase/conf/hbase-site.xml 
	
```
<configuration>

   <property>
      <name>hbase.rootdir</name>
      <value>hdfs://localhost:8030/hbase</value>
   </property>
	
   <property>
      <name>hbase.zookeeper.property.dataDir</name>
      <value>/home/hadoop/zookeeper</value>
   </property>
   
   <property>
     <name>hbase.cluster.distributed</name>
     <value>true</value>
   </property>

</configuration>
```

```
yzhong_cs@instance-2:~$ /usr/local/Hbase/bin/start-hbase.sh 
localhost: starting zookeeper, logging to /usr/local/Hbase/bin/../logs/hbase-yzhong_cs-zookeeper-instance-2.out
starting master, logging to /usr/local/Hbase/bin/../logs/hbase-yzhong_cs-master-instance-2.out
OpenJDK 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
starting regionserver, logging to /usr/local/Hbase/bin/../logs/hbase-yzhong_cs-1-regionserver-instance-2.out
```

Entering hbase shell

```
root@instance-2:/home/yzhong_cs# hbase shell
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/Hbase/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.5, rd7b05f79dee10e0ada614765bb354b93d615a157, Wed Mar  1 00:34:48 CST 2017

hbase(main):001:0> 
```

All in One Logs:

```
root@instance-2:/home/yzhong_cs# start-dfs.sh
startyStarting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-root-namenode-instance-2.out
-yarn.shlocalhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-root-datanode-instance-2.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop/logs/hadoop-root-secondarynamenode-instance-2.out
root@instance-2:/home/yzhong_cs# start-yarn.sh
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-root-resourcemanager-instance-2.out
localhost: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-root-nodemanager-instance-2.out
root@instance-2:/home/yzhong_cs# start-hbase.sh
localhost: starting zookeeper, logging to /usr/local/Hbase/bin/../logs/hbase-root-zookeeper-instance-2.out
starting master, logging to /usr/local/Hbase/logs/hbase-root-master-instance-2.out
OpenJDK 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
OpenJDK 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
starting regionserver, logging to /usr/local/Hbase/logs/hbase-root-1-regionserver-instance-2.out
root@instance-2:/home/yzhong_cs# hadoop fs -ls /hbase
Found 7 items
drwxr-xr-x   - root supergroup          0 2017-05-15 02:03 /hbase/.tmp
drwxr-xr-x   - root supergroup          0 2017-05-15 02:03 /hbase/MasterProcWALs
drwxr-xr-x   - root supergroup          0 2017-05-15 02:02 /hbase/WALs
drwxr-xr-x   - root supergroup          0 2017-05-15 02:03 /hbase/data
-rw-r--r--   1 root supergroup         42 2017-05-15 02:02 /hbase/hbase.id
-rw-r--r--   1 root supergroup          7 2017-05-15 02:02 /hbase/hbase.version
drwxr-xr-x   - root supergroup          0 2017-05-15 02:02 /hbase/oldWALs
```

Check running dashboard with curl

	curl localhost:16010/master-status

Tips:

* Use `jps` to check running process

Reference: https://www.tutorialspoint.com/hbase/hbase_installation.htm