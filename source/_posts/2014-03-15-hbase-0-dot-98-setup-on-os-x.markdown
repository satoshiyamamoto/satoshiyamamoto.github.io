---
layout: post
title: "HadoopとHBaseをOS Xにインストールする"
date: 2014-03-15 21:18:28 +0900
comments: true
categories: 
---
　Linux上でHDFSの構築ができたので、今回はHomebrewを使ってOS XにHadoopとHBaseを構築します。基本的な作業は前回、前回と同じなので要点を絞って記事に残したいと思います。インストールする環境は次のとおりです。

## 環境
* Mac OS X 10.9.2 (Marvericks)
* JDK 1.8.0 beta 128
* Hadoop 2.2.0
* HBase 0.98

## Homebrewでのインストール方法

　HomebrewでHadoopとHBaseをインストールします。

### Hadoopのインストール
    $ brew info hadoop
    hadoop: stable 2.2.0
        :
    $ brew install hadoop

### HBaseのインストール

HBaseに関しては、0.98系をインストールできるようにFormulaファイルを修正します。brew editコマンドで、HBaseのダウンロードURLとファイルのハッシュ値を変更します。

    $ brew edit hbase
    url 'http://ftp.jaist.ac.jp/pub/apache/hbase/hbase-0.98.0/hbase-0.98.0-hadoop2-bin.tar.gz'
    sha1 '95cbaf1703a3f4719f67ff0e2d3247155b49fccc'

brew installでインストールします。

    $ brew info hbase
    hbase: stable 0.98.0, devel 0.96.1.1
         :

    $ brew install hbase

## HDFSの構築と初期化

あらかじめ、パスワードなしでSSHログインできるように、秘密鍵の作成と公開鍵の登録を済ませておきます。

### NameNode、DataNodeディレクトリの作成

それぞれのディレクトリは、Homebrew管理下の/usr/local配下に構築します。

    $ mkdir -p /usr/local/var/dfs/nn
    $ mkdir -p /usr/local/var/dfs/dn

### Hadoop設定ファイルの編集

基本的には、Linux編とさほど変わりはありません。前回、前々回の記事を参考に次の設定ファイルを編集します。

* hadoop-env.sh
* core-site.xml
* hdfs-site.xml
* yarn-site.xml
* mapred-site.xml

HADOOP_HOMEは/usr/local/opt/hadoop/libexecを指定します。hdfs-site.xmlの各ノードは、前述の/usr/local/var/dfsから始まるパスを指定します。

編集が無事終わったら、hdfsコマンドでNameNodeを初期化します。

    $ hdfs namenode -format

## HBaseの設定

/usr/local/opt/hbase/conf に移動し、hbase-site.xmlを編集します。

    $ vi conf/hbase-site.xml
        :
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:9000/hbase</value>
    </property>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

## 動作確認

HDFSを起動させます。

    $ start-dfs.sh && start-yarn.sh

それぞれ、Webインターフェールで正常に起動できているか確認を行います。

* Cluster Metrics: http://localhost:8088 
* Namenode: http://localhost:50070 
* Secondary NameNode: http://localhost:50090 

続いて、HBaseを起動しブラウザで動作確認を行います。

    $ start-hbase.sh

* HBase: http://localhost:60010 

## HBaseでエラーができる時の対処方法

ZooKeeperがHBaseの管理下で起動されていない場合、次のコマンドで起動します。

    $ hbase-daemons.sh start zookeeper
    $ netstat -an | grep 2181
        :                     # 確認

HDFSのノードやZooKeeperのメタ情報が参照できなくなった場合には、一度それぞれのノードを削除することエラーが解消される場合があります。

    $ hdfs dfs -rm -rf /hbase
    $ zkCli
    [zk: localhost:2181(CONNECTED) 0] rmr /hbase


## 参考

* [MacOSX 10.9にHbaseインストール - Qiita](http://qiita.com/ma2k8/items/194311828775dedaaeba)
