---
layout: post
title: "Mac OS XにRuby 2.0をインストールする"
date: 2013-11-14 21:08:43 +0900
comments: true
categories: 
---

HomebrewでRuby 2.0をインストールします。一般的に端末に異なるバージョンのRubyをインストールするには、別途、rbenv、RVMを利用するケースが多いのですが、今回はRailsを動作させることが目的なので[「homebrewでMacに複数バージョンのrubyをインストールする」](http://blog.lampetty.net/blog_ja/index.php/archives/475)を参考に作業を進めます。

brew info ruby のメッセージを参考に、あらかじめ依存パッケージをインストールしておきます。

* libyaml
* openssl
* readline (推奨)

インストール可能なバージョンを確認

     $ brew versions ruby
     2.0.0-p247 git checkout 33cb28e /usr/local/Library/Formula/ruby.rb  <-- これをインストール
     2.0.0-p195 git checkout dad5917 /usr/local/Library/Formula/ruby.rb
     2.0.0-p0 git checkout 3085c40 /usr/local/Library/Formula/ruby.rb
     1.9.3-p392 git checkout 6c26d0a /usr/local/Library/Formula/ruby.rb
            :

2.0.0-p247を指定します。

    $ cd /usr/local
    $ git checkout 33cb28e /usr/local/Library/Formula/ruby.rb

rdocのドキュメントも合わせてインストールします。

    $ brew install ruby --with-doc
    ==> Downloading http://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p247.tar.bz2
    Already downloaded: /Library/Caches/Homebrew/ruby-2.0.0-p247.tar.bz2
    ==> ./configure --prefix=/usr/local/Cellar/ruby/2.0.0-p247 --enable-shared --wit
    ==> make
    ==> make install
    ==> Caveats
    NOTE: By default, gem installed binaries will be placed into:
      /usr/local/opt/ruby/bin
    
    You may want to add this to your PATH.
    ==> Summary
    🍺  /usr/local/Cellar/ruby/2.0.0-p247: 14945 files, 82M, built in 3.7 minutes

環境変数を設定します。

    $ vi ~/.bash_profile
    
    RUBY_HOME='/usr/local/opt/ruby' #インストールディレクトリ
    RI="--format ansi" # riを256カラーで表示
    PAGER="less -R" # ページャーにlessを指定
    PATH=$RUBY_HOME/bin:$PATH 
    
    export PATH PAGER RUBY_HOME RI
    ~

インストールされたか確認

    $ ruby -v
    ruby 2.0.0p247 (2013-06-27 revision 41674) [x86_64-darwin12.5.0]

リファレンスを見る

    $ ri GC --format ansi
    GC
    
    (from ruby core)
    ------------------------------------------------------------------------------
    The GC module provides an interface to Ruby's mark and sweep garbage
    collection mechanism.
            :

カラー表示されていればOKです。

最後にRuby2.0対応のデバッガ、byebugをインストールします。

    $ gem install byebug
    Building native extensions.  This could take a while...
    Successfully installed byebug-2.3.1
            :

次はRailsの開発環境を整えます。
