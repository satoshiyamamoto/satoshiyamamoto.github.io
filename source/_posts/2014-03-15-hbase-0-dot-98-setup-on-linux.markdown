---
layout: post
title: "HBase 0.98をSingle-Node ClusterのHDFSに構築する(Linux編)"
date: 2014-03-15 09:55:04 +0900
comments: true
categories: 
---
　前回の続きで、Single-Node Clusterで構築したHDFS上にHBaseをインストールします。環境は次のとおりです。

* CentOS 6.5 x86_64
* JDK 1.7.0 u51
* Hadoop 2.2.0
* HBase 0.98 Hadoop 2

## HBaseのダウンロード

　HBaseのDownloadページからリンクを辿って hbase-0.98.0-hadoop2-bin.tar.gz をダウンロードします。

    $ cd /usr/local/src
    $ wget http://ftp.kddilabs.jp/infosystems/apache/hbase/hbase-0.98.0/hbase-0.98.0-hadoop2-bin.tar.gz
    $ tar zxvf hbase-0.98.0-hadoop2-bin.tar.gz 
    $ mv hbase-0.98.0-hadoop2 ../hbase

## hbase-site.xmlの設定

　次の設定ファイルでHBaseのルートディレクトリを指定します。

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

また、念のためhbase-env.shでJAVA_HOMEを指定しておきましょう。

## 起動と動作確認

まず、HDFSを立ち上げます。

    $ start-dfs.sh && start-yarn.sh

続いてHBaseを立ち上げます。

    $ start-hbase.sh

ブラウザから http://localhost:60010 にアクセスして、Webインターフェースにアクセスできることを確認します。

![HBase Web UII](/images/20140315/hbase1.png)

ターミナルからHBaseシェルを起動します。

    $ hbase shell
    2014-03-15 11:00:39,812 INFO  [main] Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
    HBase Shell; enter 'help<RETURN>' for list of supported commands.
    Type "exit<RETURN>" to leave the HBase Shell
    Version 0.98.0-hadoop2, r1565492, Thu Feb  6 16:46:57 PST 2014

statusコマンドで状態を確認します。

    hbase(main):001:0> status
    SLF4J: Class path contains multiple SLF4J bindings.
    SLF4J: Found binding in [jar:file:/usr/local/hbase/lib/slf4j-log4j12-1.6.4.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: Found binding in [jar:file:/usr/local/hadoop/share/hadoop/common/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
    SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
    Java HotSpot(TM) 64-Bit Server VM warning: You have loaded library /usr/local/hadoop/lib/native/libhadoop.so.1.0.0 which might have disabled stack guard. The VM will try to fix the stack guard now.
    It's highly recommended that you fix the library with 'execstack -c <libfile>', or link it with '-z noexecstack'.
    2014-03-15 11:00:49,768 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
    1 servers, 0 dead, 2.0000 average load

HadoopとHBaseでロガーが競合しているため警告が沢山出ます.... が、今回は無視します。テーブルとデータが作成できるかテストします。

    hbase(main):001:0> create 'testtable', 'fam'
    0 row(s) in 2.6850 seconds
     
    => Hbase::Table - testtable
    hbase(main):002:0> put 'testtable', 'row1', 'fam:col', 'val'
    0 row(s) in 0.4660 seconds

ブラウザから http://localhost:50070 にアクセスし、Browse the filesystem でHBaseのディレクトリが作成されていれば成功です。
![hbase datanode](/images/20140315/hbase2.png)

次回は、Homebrewを使って同じ環境をOS Xで再現したいと思います。

## 参考

* [Apache HBase ブック](http://oss.infoscience.co.jp/hbase/book.html#hbase.site)

