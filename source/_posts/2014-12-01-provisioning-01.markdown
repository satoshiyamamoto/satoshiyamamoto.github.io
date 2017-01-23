---
layout: post
title: "プロビジョニング事始め (1) ruby環境構築編"
date: 2014-12-01 14:49:00 +0900
comments: true
categories: 
---

しばらく仕事が忙しかったせいかブログの更新が滞ってました。なんと、実に5ヶ月ぶりの投稿です。

その期間サボっていた訳ではなく、仕事の開発スタイルがJIRAとGithubに移行したり、ミドルウェアのHBase、ZooKeeper、FlumeNGなんかを復習したり。

一方、プライベートではChef (solo)、Vagrantなどの環境構築まわりを勉強していました。ここ最近、ようやく落ち着いてきたので、忘れないうちに一旦記事にしていこうと思ってます。

## 動作環境
* Mac OS X 10.8.5
* Homebrew 0.9.5

まずは、Homebrewを使ってMacの定番Ruby環境(rbenv+ruby-build)を構築します。

はじめに、Homebrewをアップデートしておきます。

{% codeblock lang:bash %}
$ cd $(brew --previx)
$ brew update
{% endcodeblock %}

## ruby-buildとrbenvのインストール


{% codeblock lang:bash %}
$ brew install ruby-build
==> Downloading https://github.com/sstephenson/ruby-build/archive/v20141128.tar.gz
######################################################################## 100.0%
==> ./install.sh
🍺  /usr/local/Cellar/ruby-build/20141128: 139 files, 596K, built in 5 seconds
{% endcodeblock %}


次に、Rubyのバージョン管理のためrbenvをインストールします。

{% codeblock lang:bash %}
$ brew install rbenv
==> Downloading https://github.com/sstephenson/rbenv/archive/v0.4.0.tar.gz
######################################################################## 100.0%
==> Caveats
To use Homebrew's directories rather than ~/.rbenv add to your profile:
  export RBENV_ROOT=/usr/local/var/rbenv

To enable shims and autocompletion add to your profile:
  if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
==> Summary
🍺  /usr/local/Cellar/rbenv/0.4.0: 31 files, 152K, built in 6 seconds
{% endcodeblock %}

 ユーザごとに ~/.rbenvを使い分けるので RBENV_ROOT は設定しないでおきます。後は、Summaryの案内に従ってprofileにパスを追加します。

{% codeblock lang:bash %}
$ vi ~/.bash_profile
if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi

$ echo $PATH
$HOME/.rbenv/shims:
{% endcodeblock %}

追加されてますね。

rbrnv rehash の煩わしさから解放されるためにrbenv-gem-rehashを追加でイントールします。

{% codeblock lang:bash %}
$ brew install rbenv-gem-rehash
==> Downloading https://github.com/sstephenson/rbenv-gem-rehash/archive/v1.0.0.tar.gz
######################################################################## 100.0%
==> Downloading https://github.com/sstephenson/rbenv-gem-rehash/commit/0756890cfd9c7b
######################################################################## 100.0%
==> Patching
patching file README.md
Hunk #1 succeeded at 23 (offset -2 lines).
patching file etc/rbenv.d/exec/~gem-rehash.bash
patching file gems/rbenv-gem-rehash-1.0.0/lib/rubygems_plugin.rb
patching file rubygems_plugin.rb
patching file specifications/rbenv-gem-rehash-1.0.0.gemspec
==> Caveats
If the GEM_PATH environment variable is undefined, rbenv-gem-rehash must
first execute the gem env gempath command to retrieve RubyGems' default path
so that it can can append to the path rather than override it. This can take
a while--from a few hundred milliseconds on MRI to several seconds on
JRuby--so the default path for the current Ruby version is cached to the
filesystem the first time it is retrieved.
==> Summary
🍺  /usr/local/Cellar/rbenv-gem-rehash/1.0.0: 7 files, 24K, built in 4 seconds
{% endcodeblock %}

## インストールバージョンの確認

{% codeblock lang:bash %}
$ rbenv install --list
Available versions:
  1.8.6-p383
  1.8.6-p420
    :
{% endcodeblock %}

今回は現時点での安定版 2.1.5 をインストールします。(環境によりますが結構時間かかります。)

{% codeblock lang:bash %}
$ rbenv install 2.1.5
Downloading ruby-2.1.5.tar.gz...
-> http://dqw8nmjcqpjn7.cloudfront.net/4305cc6ceb094df55210d83548dcbeb5117d74eea25196a9b14fa268d354b100
Installing ruby-2.1.5...
Installed ruby-2.1.5 to /Users/a12019/.rbenv/versions/2.1.5

$ rbenv versions
* system (set by /Users/a12019/.rbenv/version)
2.1.5
{% endcodeblock %}

インストールできてますね。

## デフォルトのRubyバージョンを変更

{% codeblock lang:bash %}
$ rbenv global 2.1.5
$ rbenv versions
system
* 2.1.5 (set by /Users/a12019/.rbenv/version)

$ ruby -v
ruby 2.1.5p273 (2014-11-13 revision 48405) [x86_64-darwin12.0]
{% endcodeblock %}

変わりました。

  SystemのRubyを見に行く場合は、PATHが.rbenv/shims
  を見に行ってるか確認してみてください。PATHが正しいようなら rbenv rehash でshims を更新させます。

## Rubyバージョンの変更

インストールの手順と同じようにインストール可能なバージョンを確認します。

{% codeblock lang:bash %}
$ rbenv install --list
$ rbenv instal <version>
$ rbenv global <version>
$ ruby -v
{% endcodeblock %}

バージョンが切り替わらない場合は、rbenv rehashします。

{% codeblock lang:bash %}
$ rbenv rehash
{% endcodeblock %}

## インストールバージョンの更新

Homebrewからruby-buildのバージョンをアップデートします。

{% codeblock lang:bash %}
$ brew upgrade ruby-build
{% endcodeblock %}

これで ruby linstall --list の結果が更新されるはずです。

## まとめ
rbenvとruby-buildを使うことで簡単にRubyのバージョンを切り替えることが出来るようになりました。rbenvはバージョンごとにgemのリポジトリを管理するので、PC本体の環境を汚さずに済みます。

今回、Qiitaの投稿を参考にさせてもらいましたが、お陰で理解が深まりました。とは言え、公式ドキュメント読むために英語勉強せねば...

## 参考
* [Ruby - rbenv を使っているなら rbenv-gem-rehash を使おう - Qiita](http://qiita.com/riocampos/items/f0fe7217972b312c4f3a)
* [rbenv, ruby-buildのインストール - Qiita](http://qiita.com/isaoshimizu/items/07ddd1e25996e19d45bf)
