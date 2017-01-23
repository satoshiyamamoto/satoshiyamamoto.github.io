---
layout: post
title: "Cygwinでクリップボードと仲良くなる"
date: 2014-02-13 19:49:14 +0900
comments: true
categories: 
---

## getclip/putclipのインストール

Cygwinのコンソールからクリップボードにアクセスするgetclipとputclipをインストールします。これらはcygutils-extraパッケージに含まれています。

## 文字化け対策

putclip/getclipは、デフォルトのままでは以下のとおり2バイト文字が文字化けしてしまいます。Windowsに戻って他のアプリケーションに貼付けすると残念な結果に...

    $ echo $LANG
    ja_JP.UTF-8@cjknarrow

    $ date | putclip
    2014蟷ｴ 2譛・13譌･ 譛ｨ譖懈律 21:56:25 JST # 結果

これはnkfを使って文字コードを変換することで解消できます。sourceforgeからnkfのソースを取得しmakeインストールします。

    $ tar zxvf nkf-2.1.3.tar.gz
    $ cd nkf-2.1.3
    $ make && make install

インストールが終わったらnkfにパスを通し文字コードを変換してみます。

    $ date | nkf -s | putclip
    2014年 2月 13日 木曜日 22:14:36 JST  # UTF-8 -> Shift_JIS

    $ getclip | nkf -w
    2014年 2月 13日 木曜日 22:14:36 JST  # Shift_JIS -> UTF-8

デフォルトでこの挙動になるようaliasを設定しておくと便利です。

    vi .bashrc
        :
    alias getclip='getclip | nkf -w'
    alias putclip='nkf -s | putclip'

### 参考

* [cygwin の putclip/getclip の文字化け対策 - by edvakf in hatena](http://d.hatena.ne.jp/edvakf/20101017/1287285851)
* [Cygwin使いならば絶対に身につけておきたいコマンド5選+apt-cyg | Futurismo](http://futurismo.biz/archives/1364)
