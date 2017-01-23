---
layout: post
title: "Hadoop 2.2をSingle-Node Clusterでインストールする(Linux編)"
date: 2014-03-14 12:02:09 +0900
comments: true
categories: 
---
HBaseをローカルで動かすためにHDFSを構築します。Standaloneモードで起動する場合、HDFSは不要となりますが、そうとは言えHadoopとの関係は切っても切り離せません。そこでHadoopエコシステムの学習も含めHDFS上にHBaseを構築してみたいと思います。環境は次のとおりです。

## 環境
* CentOS 6.5 x86_64
* JDK 1.7.0 u51
* Hadoop 2.2.0

## Hadoopユーザーの作成
まず、OSにHadoopユーザーを作成します。

    # groupadd hadoop
    # useradd -g hadoop hadoop

続いて、パスワード無しでSSH接続できるように鍵の作成を行います。

    $ ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): ↵ 
    Enter passphrase (empty for no passphrase): ↵ 
    Enter same passphrase again: ↵ 
    Your identification has been saved in /home/hadoop/.ssh/id_rsa.
    Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
    The key fingerprint is:
    36:93:c4:83:fa:39:52:de:d1:f1:ed:19:6a:c3:97:22 hadoop@centos.local
    The key's randomart image is:
    +--[ RSA 2048]----+
         :
    $ cd .ssh
    $ cat id_rsa.pub >> authorized_keys
    $ chmod 600 authorized_keys
    $ chmod 400 id_rsa

鍵の作成が終わったら、SSHでパスワードのプロンプトが表示されずにログインできるか確認します。

    $ ssh localhost
    The authenticity of host 'localhost (::1)' can't be established.
    RSA key fingerprint is 27:05:7f:46:bd:cd:10:7d:d8:2c:38:60:21:46:05:14.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'localhost' (RSA) to the list of known hosts.
    Last login: Fri Mar 14 13:56:43 2014 from 10.0.2.2
    $

無事ログインできれば成功です。

## Hadoopのダウンロード

HadoopウェブサイトのDownload Hadoopからリンクを辿ってhadoop-2.2.0.tar.gzをダウンロードします。ダウンロードが完了したら/usr/local/hadoopに展開します。

    $ /usr/local/src
    $ wget http://ftp.jaist.ac.jp/pub/apache/hadoop/common/hadoop-2.2.0/hadoop-2.2.0.tar.gz 
    $ tar zxvf hadoop-2.2.0.tar.gz
    $ mv hadoop-2.2.0 ../hadoop

## HDFSの構築

Namenode、Datanodeを格納するディレクトリを作成します。この設定は、後述のhdfs-site.xmlで指定できます。

    # mkdir -p /var/dfs/nn  # namenode
    # mkdir -p /var/dfs/dn  # datanode
    # chown -R hadoop:hadoop /var/dfs

### 環境変数の設定

.bash_profileに次のHadoop用の環境変数を設定します。

    $ vi ~/bash_profile
    export HADOOP_COMMON_HOME=/usr/local/hadoop
    export HADOOP_HDFS_HOME=$HADOOP_COMMON_HOME
    
    export HADOOP_MAPRED_HOME=$HADOOP_COMMON_HOME
    export YARN_HOME=$HADOOP_COMMON_HOME

    export PATH=$HADOOP_COMMON_HOME/sbin:$HADOOP_COMMON_HOME/bin:$PATH

続いて、HADOOP_HOME/etc/hadoopに移動し次の環境変数を設定します。

    $ cd /usr/local/hadoop/etc/hadoop
    $ vi hadoop-env.sh 
    export JAVA_HOME=/usr/java/default
    export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
    export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=$HADOOP_HOME/lib"

### Hadoop設定ファイルの編集

Hadoop設定ファイルのそれぞれに次の設定を追加します。

core-site.xml (基本情報の設定)

    <property>
      <name>fs.default.name</name>
      <value>hdfs://localhost:9000</value>
    </property> 
 
hdfs-site.xml (NameNodeとDataNodeの設定)

    <property>
      <name>dfs.replication</name>
      <value>1</value>
    </property>

    <property>
      <name>dfs.namenode.name.dir</name>
      <value>file:///var/dfs/nn</value>
    </property>

    <property>
      <name>dfs.datanode.data.dir</name>
      <value>file:///var/dfs/dn</value>
    </property>

yarn-site.xml (ResourceManager、NodeManager、History Serverの設定)

    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>

    <property>
      <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
      <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>

mapred-site.xml (MapReduce、JobHistory Serverの設定)

    $ cp mapred-site.xml.template mapred-site.xml
    $ vi mapred-site.xml
        :
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property> 

### NameNodeの初期化

次のコマンドで、NameNodeの初期化を行います。

    $ hdfs namenode -format
    14/03/14 15:37:42 INFO namenode.NameNode: STARTUP_MSG:
    /************************************************************
    STARTUP_MSG: Starting NameNode
    STARTUP_MSG:   host = centos/127.0.0.1
    STARTUP_MSG:   args = [-format]
    STARTUP_MSG:   version = 2.2.0
    STARTUP_MSG:   classpath = /usr/local/hadoop/etc/hadoop:/usr/local/hadoop/share/hadoop/common/lib/jetty-ut
        :
    SHUTDOWN_MSG: Shutting down NameNode at centos/127.0.0.1
    ************************************************************/

## Hadoopの起動とプロセスの確認

次の起動スクリプトでHadoopを起動します。(start-all.sh は、非推奨になったようです)

    $ start-dfs.sh
    14/03/14 15:45:39 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    Starting namenodes on [localhost]
    localhost: starting namenode, logging to /usr/local/hadoop/logs/hadoop-hadoop-namenode-centos.out
    localhost: starting datanode, logging to /usr/local/hadoop/logs/hadoop-hadoop-datanode-centos.out
    Starting secondary namenodes [0.0.0.0]

64bit OSで実行すると、NativeCodeLoaderの警告文が出ます。これはHadoopのNativeライブラリが32bitを使っているためです。

    $ start-yarn.sh
    starting yarn daemons
    starting resourcemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-resourcemanager-centos.out
    localhost: starting nodemanager, logging to /usr/local/hadoop/logs/yarn-hadoop-nodemanager-centos.out

起動が完了したら、jpsコマンドでプロセスを確認します。jpsが見つからない場合は JAVA_HOME/binにパスを通してください。

    $ jps
    7529 SecondaryNameNode
    8083 Jps
    7684 ResourceManager
    7792 NodeManager
    7255 NameNode

停止するには、それぞれ逆の stop-dfs.sh、stop-yarn.sh を実行します。

## 動作確認

各ノードの状態は、次のWebインターフェースで確認できます。

* Cluster Metrics: http://localhost:8088
![cluster metrics](/images/20140314/hadoop1.png)
* Namenode: http://localhost:50070
![namenode](/images/20140314/hadoop2.png)
* Secondary NameNode: http://localhost:50090
![secondary namenode](/images/20140314/hadoop3.png)

次は、HBaseのインストール手順をまとめたいと思います。

## 参考
* [Hadoop 2.2 Single Node Installation on CentOS 6.5 | AJ's Data Storage Tutorials](http://alanxelsys.com/2014/02/01/hadoop-2-2-single-node-installation-on-centos-6-5/)
* [Installing single node Hadoop 2.2.0 on Ubuntu « BigData Handler](http://bigdatahandler.com/2013/11/02/installing-single-node-hadoop-2-2-0-on-ubuntu/)
* [Linux Install Hadoop 2.2.0 on Ubuntu Linux 13.04 (Single-Node Cluster)](http://www.ercoppa.org/Linux-Install-Hadoop-220-on-Ubuntu-Linux-1304-Single-Node-Cluster.htm)
