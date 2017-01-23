---
layout: post
title: "Unixのサーバ群をCygwin上にインストールする"
date: 2014-03-05 15:42:49 +0900
comments: true
categories: 
---
UnixのデーモンプログラムをCygwinで動かしてみます。インストールに関しては、HomebrewやYumで一発OKなほど洗練されているとは言えません。そこがCygwinの楽しみの一つでもあるのですが、たまに躓く自分のために、覚書として残しておきます。

* SSH 6.5
* Apache 2.2
* Mysql 5.5

## Windowsサービスの登録方法

Windowsのサービスに登録するには、cygrunsrvコマンドを使います。このコマンドは、Linuxのinit.dスクリプトとchkconfigの役目を果たします。コマンドの書式は次のとおりです。

    $ cygrunsrv --install <app_name> --path <app_path> [OPTIONS...]

## Cygrunsrv

Unixのデーモンをバックグラウンドで動かすためにCygserverの設定を行います。Cygserverは、Windowsで提供されていなセマフォやメッセージキューなどのプロセス間通信を支援します。まず、Cygwinのインストールが完了したら、Windowsのシステム環境変数に CYGWIN=server する必要があります。続いて、次のコマンドでCygserverの設定を行います。

    $ cygserver-config
    Generating /etc/cygserver.conf file
    
    
    Warning: The following function requires administrator privileges!
    Do you want to install cygserver as service?
    (Say "no" if it's already installed as service) (yes/no) yes
    
    The service has been installed under LocalSystem account.
    To start it, call `net start cygserver' or `cygrunsrv -S cygserver'.
    
    Further configuration options are available by editing the configuration
    file /etc/cygserver.conf. Please read the inline information in that
    file carefully. The best option for the start is to just leave it alone.
        :
    Basic Cygserver configuration finished. Have fun!
    
Windowsのサービス一覧に CYGWIN cygserver が追加されていることを確認します。

## SSH 

Opensshでは、3.3以降からデーモンプロセスを、特権を持たないsshdというユーザーに閉じ込めることで、セキュリティーを担保するようになりました。Cygwinに関しても、しかりでsshdというOSのローカルユーザを別途作成する必要があります。この作業は、対話式のssh-host-configスクリプトで行います。なお特権を分離するため下記の問に yes と応えます。

    $ ssh-host-config
        : 
    *** Query: Should privilege separation be used? (yes/no) yes
   
Windowsのサービスに登録するには、下記の質問にyesと応えます。サービス名はデフォルトのままにします。
 
    *** Query: Do you want to install sshd as a service?
    *** Query: (Say "no" if it is already installed as a service) (yes/no) yes
    *** Query: Enter the value of CYGWIN for the daemon: []
        :

ここで、WIndowsのサービスを起動するcyg_serverユーザを作成するか尋ねられますが、WindowsのSYSTEMローカルアカウントで代用できます。なので、今回はcyg_serverユーザを作らないままインストールを進めます。   
 
    *** Info: This script plans to use 'cyg_server'.
    *** Info: 'cyg_server' will only be used by registered services.
    *** Query: Do you want to use a different name? (yes/no) no

    *** Query: Create new privileged user account 'cyg_server'? (yes/no) no
    *** ERROR: There was a serious problem creating a privileged user.
    *** Query: Do you want to proceed anyway? (yes/no) yes
    *** Warning: Expected privileged user 'cygwin' does not exist.
    *** Warning: Defaulting to 'SYSTEM'
    
    *** Info: The sshd service has been installed under the LocalSystem 
        :

スクリプトが正常に終了したら、メッセージに従って下記のコマンド、またはWindowsサービスから起動しSSHログインできるか確認します。

    $ cygrunsrv --start sshd 

## Apache2 

Apacheのインストールが完了したら、httpd.confの設定を行います。インストール直後はServerNameがコメントアウトされているので、正しいサーバー名を設定してあげます。

    $ hostname --fqdn
    win7.local   <--- サーバー名
    
    $ /usr/sbin/apachectl2 configtest
    Syntax OK
 
設定が完了したら、起動するか確認します。

    $ /usr/sbin/apachectl2 start
        or
    $ /usr/sbin/httpd2 -k start

Apacheの起動にはCygserverの設定とCYGWIN環境変数の設定が必要です。1464 Bad system call などとエラーが出た場合は、これらが正しく設定されているか確認してみてください。

cygrunsrvでサービスに登録します。

    $ cygrunsrv --install httpd --path /usr/sbin/httpd --args '-k' --disp 'CYGWIN httpd'

Windowsのサービス一覧から起動できれば成功です。

## MySQL

MySQLをローカルで起動するにはサーバーとクライアントの両方が必要です。クライアントのインストールが抜けていると、起動時に my_print_defaults が見つからずエラーになります。setup.exe のインストールが完了したら、DBの初期化を行います。

    $ /usr/bin/mysql_installdb

デフォルトでは /var/lib/mysql にデータベースが作成されます。変更したい場合は --datadir=<path> で指定します。データベースの初期化が完了したらメッセージにしたがって mysql.server スクリプトをコピーしておきます。

    $ cp /usr/share/mysql/mysql.server /usr/bin/
    $ chmod +x /usr/bin/mysql.server

次のコマンドで起動と、mysqlクライアントから接続できるか確認します。

    $ mysql.server start
    Starting MySQL...... SUCCESS!
    $ mysql -uroot -p

最後にWindowsのサービスに登録します

    $ cygrunsrv --install mysqld --path /usr/bin/mysql_safe --args '--datadir=/var/lib/mysql --pid-file=/var/lib/mysql/<srv_name>.pid

## 参考
* [特権分離 (Privilege Separated) OpenSSH](https://www.citi.umich.edu/u/provos/ssh/privsep-j.html)
* [3.6. Cygserver](http://www.sixnine.net/cygwin/translation/cygwin-ug-net/using-cygserver.html)
* [Cygwin で Apache2 をサービスとして起動](http://komoriyuichi.web.fc2.com/cygwin/apache2.html)

