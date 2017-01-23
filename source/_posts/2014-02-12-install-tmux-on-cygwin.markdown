---
layout: post
title: "Cygwin 64bit版にTmuxをインストールする"
date: 2014-02-12 15:58:25 +0900
comments: true
categories: 
---

ついにCygwinでlibeventがビルドできるようになりました。そこで、長年の悲願だったCyginwでTmuxを動かしてみたいと思いますす。

## バージョン情報

* Tmux: 1.9
* libevent: 2.0.21-stable
* Cygwin x86_64: 1.7.28
* Windows 7 64bit: SP1

![Capture](/images/cygwin-tmux.png)

## インストールにあたって

Cygwin 64bit対応でパッケージリポジトリの階層が変わったため、[apt-cyg](https://github.com/rcmdnk/apt-cyg)がそのままでは正しく機能しなくなってます。なので、今回は従来のsetup.exeで依存パッケージをインストールを行います。

また、libeventのインストール先は、/usrとします。（/usr/localでは、tmuxがlibeventを見つけられずエラーになりました。）

## 依存ライブラリの一覧

Cygwinのsetup.exeからlibevent、Tmuxのビルドに必要なライブラリをインストールします。依存ライブラリは次の通りです。

* autoconf
* gcc-core
* git
* make
* ncurses
* libncurses-devel
* libtool
* pkg-config
* wget

## libeventのビルドとインストール

Githubから安定版のソースをダウンロードしてきます。

    $ cd /usr/local/src
    $ wget https://github.com/libevent/libevent/archive/release-2.0.21-stable.tar.gz

autogen.shからconfigureを生成します。

    $ ./autogen.sh

そうすると、Makefile.amでテストが失敗するので、以下の行をコメントアウトします。

    $ vi test/Makefile.am
        :
    22 #TESTS = $(top_srcdir)/test/test.sh

再度autogen.shを実行すると正常終了します。

    $ ./autogen.sh

prefixオプションでインストール先を指定し、ビルド＆インストールします。

    $ ./configure --prefix=/usr
    $ make && make install

## Tmuxのインストール

SourceforgeからGitでソースコードを取得します。

    $ git clone http://git.code.sf.net/p/tmux/tmux-code

autogen.shでconfigureを生成しmake && installします。

    $ ./autogen.sh
    $ ./configure --prefix=/usr
    $ make && make install

インストールされているか確認します。

    $ which tmux
    /usr/bin/tmux

    $ tmux -V
    tmux 1.9

### 参考

* [windowsだってtmuxを使いたい - @numa08　猫耳帽子の女の子](http://numa08.hateblo.jp/entry/2013/07/27/094528)
* [tmuxがCygwin上で動作するようになったのでインストール手順をまとめてみた - atdxfe's Blog](http://atdxfe.hatenablog.com/entry/2013/11/27/031058)

