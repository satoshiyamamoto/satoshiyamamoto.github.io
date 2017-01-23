---
layout: post
title: "Mac OS XにRails開発環境を構築する"
date: 2014-01-06 12:39:45 +0900
comments: true
categories: 
---

## Railsのインストール

gemでrailsをインストールします。[RailsによるアジャイルWebアプリケーション開発 第4版](http://www.amazon.co.jp/RailsによるアジャイルWebアプリケーション開発-第4版-Sam-Ruby/dp/4274068668)の日本語版が、Rails 3.x時点の執筆なのでインストールするバージョンもそれに合わせます。インストールが完了したら、バージョンを確認します。

    $ gem install rails -v "~>3.2.0" #3.2系の最新
    $ rails -v
    Rails 3.2.16

一度、最新の4.x系をインストールしていると正しく3.x系が表示されない場合があります。その時は次のコマンドで4.x系の不要なモジュールが残っていないか確認してみてください。

    $ gem list | grep railties
    railties (4.0.2, 3.2.16)

削除するにはバージョンを指定してrailtiesを削除します。

    $ gem uninstall railties -v "4.0.2"

また、Databaseに関してはローカルの開発ではSQLite3を使い、herokuのデプロイ後は、アドオンでMySQLを使います。コマンドラインからデータを参照するためにsqlite3をインストールします。

    $ brew install sqlite3
    ==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/sqlite-3.8.2.m
    ######################################################################## 100.0%
    ==> Pouring sqlite-3.8.2.mountain_lion.bottle.tar.gz
    ==> Caveats
    This formula is keg-only, so it was not symlinked into /usr/local.
    
    Mac OS X already provides this software and installing another version in
    parallel can cause all kinds of trouble.
    
    OS X provides an older sqlite3.
    
    Generally there are no consequences of this for you. If you build your
    own software and it requires this formula, you'll need to add to your
    build variables:
    
        LDFLAGS:  -L/usr/local/opt/sqlite/lib
        CPPFLAGS: -I/usr/local/opt/sqlite/include
    
    ==> Summary
    🍺  /usr/local/Cellar/sqlite/3.8.2: 9 files, 2.0M

SQLの結果が見やすいようsqlite3のコマンドにヘッダー表示と列幅調整モードのオプションを指定します。

    $ vi ~/.bashrc
         :
    alias sqlite3='sqlite3 -header -column'
    
    $ sqlite3 
    SQLite version 3.7.12 2012-04-03 19:43:07
    Enter ".help" for instructions
    Enter SQL statements terminated with a ";"
    sqlite> .show
         echo: off
      explain: off
      headers: on
         mode: column
    nullvalue: ""
       output: stdout
    separator: "|"
        stats: off
        width:

## IDEを選ぶ

これは完全に好みですが、3.x系でrailsプロジェクトをサポートしているAptana Studio 3をインストールします。NetBeans、EclipseプラグインであるRadRails 2は、3.xのサポートがありませんでした。

* Apatana: http://www.aptana.com/products/studio3/download

次は、IDEの設定とHerokuへデプロイするまでの手順を記します。

### 参考

* [Railsのバージョンがなんかおかしい時はrailtiesをチェック](http://blog.supermomonga.com/articles/rails/rails-version-railties.html)
* [最近のRuby、Railsの環境構築から、herokuへデプロイまでのやり方を教えてもらいました](http://blog.ojimac.com/2012/04/25/最近のruby、railsの環境構築から、herokuへデプロイまでの/)
