---
layout: post
title: "プロビジョニング事始め (2) Vagrant編"
date: 2014-12-18 10:49:04 +0900
comments: true
categories: 
---

Vagrantは、VirtualBoxなどの仮想化ソフトウェアのラッパーです。フリーで公開されている様々なゲストOSのイメージを利用でき、煩わしいインストーラに時間を取られることなく、素早く簡単に開発環境を構築できます。

まずは、Vagrantのインストールから始め、Guest OSの基本的な設定と、さらには簡単なプロビジョニングを実行するところまでを記していきます。(VirtualBoxは既にインストールされているものとします)

## 動作環境
* Mac OS X 10.8.5
* Vagrant 1.7.2
* VirtualBox 4.3.20
* Opscode Bento

## Vagrantのインストール

[Homebrew-cask](https://github.com/caskroom/homebrew-cask)を使ってコマンドラインからVagrantのdmgパッケージを取得します。まず、先に```Homebrew-cask```をインストールします。```brew```コマンドは次のとおりです。

{% codeblock lang:bash %}
$ brew install caskroom/cask/brew-cask
==> Installing brew-cask from caskroom/homebrew-cask
==> Cloning https://github.com/caskroom/homebrew-cask.git
Updating /Library/Caches/Homebrew/brew-cask--git
==> Checking out tag v0.53.3
🍺  /usr/local/Cellar/brew-cask/0.53.3: 2421 files, 9.5M, built in 75 seconds
{% endcodeblock %}

続いて```vagrant```をインストールします。

{% codeblock lang:bash %}
9$ brew cask install vagrant                                                          
==> Downloading https://dl.bintray.com/mitchellh/vagrant/vagrant_1.7.2.dmg
Already downloaded: /Library/Caches/Homebrew/vagrant-1.7.2.dmg
==> Running installer for vagrant; your password may be necessary.
==> Package installers may write to any location; options such as --appdir are ignored.
==> installer: Package name is Vagrant
==> installer: Upgrading at base path /
==> installer: The upgrade was successful.
🍺  vagrant staged at '/opt/homebrew-cask/Caskroom/vagrant/1.7.2' (6 files, 79M)
{% endcodeblock %}

インストールが完了すると```vagrant```コマンド本体と実行に必要なモジュールが一緒にインストールされます。```vagrant```コマンドは以前RubyGemsで配布されていたようですが、依存関係の問題を解消するため、実行時に同梱されたRubyモジュールを使用します。

## ゲストOSの管理

Vagrantでは、ゲストOSのことをBoxと呼びます。VagrantのBoxはフリーで公開されているものも多く目的によって適宜選ぶと良いかと思います。

* Bento (Opscode): https://github.com/opscode/bento
* Vagrantbox.es: http://www.vagrantbox.es

boxはvagrantのサブコマンドとなっていて下記の書式で利用します。
{% codeblock lang:bash %}
$ vagrant box
Usage: vagrant box <subcommand> [<args>]

Available subcommands:
     add
     list
     outdated
     remove
     repackage
     update

For help on any individual subcommand run `vagrant box <subcommand> -h`
{% endcodeblock %}

新たにBoxを追加するには```vagrant box add```の後にBox名、イメージのURLを指定します。例えばOpscodeのCentOS 7.0を追加するには次のようにします。

{% codeblock lang:bash %}
$ vagrant box add opscode-centos-7.0 http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-7.0_chef-provisionerless.box 
{% endcodeblock %}

```vagrant box list```コマンドでBoxの一覧を確認できます。

{% codeblock lang:bash %}
$ vagrant box list
opscode-centos-6.6   (virtualbox, 0)
opscode-centos-7.0   (virtualbox, 0)
opscode-ubuntu-14.04 (virtualbox, 0)
{% endcodeblock %}

## Pluginのインストール

Pluginを利用すればVagrantの機能を大きく拡張できます。Pluginもvagrantのサブコマンドになっており次の書式で呼び出せます。

{% codeblock lang:bash %}
$ vagrant plugin -h
Usage: vagrant plugin <command> [<args>]

Available subcommands:
     install
     license
     list
     uninstall
     update

For help on any individual command run `vagrant plugin COMMAND -h`
{% endcodeblock %}

今回は仮想マシンの状態をロールバック可能にする必須の```sahara```プラグインをインストールします。

{% codeblock lang:bash %}
$ vagrant plugin install sahara
{% endcodeblock %}

## 基本設定

### 仮想マシンの初期化

仮想マシンを初期化するには```vagrant init <box name>```コマンドを実行します。

{% codeblock lang:bash %}
$ vagrant init opscode-centos-7.0
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
{% endcodeblock %}

 ```init```コマンドを実行するとカレントディレクトリに```Vagrantfile```が作成されます。

 ```Vagrantfile```は言わば仮想マシンの設定ファイルです。Vagrantは仮想マシンを立ち上げる際に、この```Vagrantfile```の内容に従ってマシンを構築します。

### ネットワークの設定

仮想マシンにNAT、またはブリッジを使って接続したい場合、```Vagrantfile```を編集してネットワークの設定を有効にします。

NATでホストからのみアクセスさせるには、下記のように```private_network```を有効にします。これでホストOSから仮想マシンには```192.168.33.10```でアクセスできるようになります。
{% codeblock lang:bash %}
config.vm.network "private_network", ip: "192.168.33.10"                     
{% endcodeblock %}

ブリッジ接続でLANに参加させるには、```public_network```の設定を有効にします。デフォルトでDHCP接続、固定IPを割り当てる場合は、```ip: "<ipv4>"```を指定します。
{% codeblock lang:bash %}
# dhcp
config.vm.network "public_network"
# static
config.vm.network "public_network", ip: "192.168.0.17"
{% endcodeblock %}

### ホスト名の設定

ホスト名は、```Vagrantfile```で次のように指定します。

{% codeblock lang:bash %}
config.vm.hostname = "www.example.com"
{% endcodeblock %}

### 仮想マシンの起動

仮想マシンを起動するには```vagrant up```コマンドを呼ぶだけです。仮想マシンの初回起動時に限り、後述する```provision```がタスクとして実行されます。

{% codeblock lang:bash %}
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'opscode-centos-7.0'...
==> default: Matching MAC address for NAT networking...
==> default: Setting the name of the VM: fuelphp_default_1423639951154_30886
    :
{% endcodeblock %}

プロンプトが戻った後に、```vagrant status```コマンドを実行すると```running```になっているのが確認できます。

{% codeblock lang:bash %}
$ vagrant status
Current machine states:

default                   running (virtualbox)
{% endcodeblock %}

### SSHによるログイン

仮想マシンには、```vagrant ssh```コマンドでログインできます。また、仮想マシンのSSHポートはホストの```2222```ポートにマッピングされているので、通常の```ssh```コマンドでもログインできます。

{% codeblock lang:bash %}
$ vagrant ssh
    or
$ ssh -p 2222 vagrant@localhost # password: vagrant
{% endcodeblock %}

## 高度な設定

Vagrantでは、```provision```コマンドを使って```Vagrantfile```に記載されたShellスクリプトやChefのレシピを実行することができます。例えば、Shellスクリプトでカーネルパラメータの設定を変更したり、```Chef```で構成管理を行うことができます。

### Shellによる簡単なプロビジョニング

Shellスクリプトで```yum fastestmirror```プラグインを実行する例です。

{% codeblock lang:bash %}
config.vm.provision "shell", inline: <<-SHELL
   sudo yum clean all
SHELL       
{% endcodeblock %}

 ```vagrant provision```コマンドで実行できます。
{% codeblock lang:bash %}
$ vagrant provision
==> default: Running provisioner: shell...
    default: Running: inline script
==> default: Loaded plugins: fastestmirror
     :
{% endcodeblock %}

### Chef Zeroによるプロビジョニング

ChefとローカルモードのZeroについては別エントリーで取り上げるとして、今回は簡単に```provision```でChef Zeroを実行する例を述べます。DSLの詳細なオプションは、Vagrantの[ウェブサイト](https://docs.vagrantup.com/v2/provisioning/chef_zero.html)で確認できます。

{% codeblock lang:bash %}
config.vm.provision "chef_zero" do |chef|
  chef.cookbooks_path = "berks-cookbooks"
  chef.run_list = "recipe[roles::node.erver]"
end
{% endcodeblock %}

先ほどと同様に```vagrant provision```コマンドで実行します。
{% codeblock lang:bash %}
$ vagrant provision
==> default: Running provisioner: chef_zero...
==> default: Detected Chef (latest) is already installed
Generating chef JSON and uploading...
==> default: Running chef-zero...
{% endcodeblock %}

### 複数VMを立ち上げる

VagrantでゲストVMを複数立ち上げるには```config.vm.define```で定義します。VMの起動はシーケンシャルに実行されるため低スペックマシンの場合、起動が終わるまで多少時間掛かります。タイムアウトが発生する場合は、```config.vm.boot_timeout```の秒数をデフォルトの300秒より長くするとよいでしょう。

{% codeblock lang:bash %}
config.vm.define "node01" do |node|
  node.vm.hostname = "node01.example.com"
  node.vm.network "private_network", ip: "192.168.33.10"
end

config.vm.define "node02" do |node|
  node.vm.hostname = "node02.example.com"
  node.vm.network "private_network", ip: "192.168.33.11"
end
{% endcodeblock %}

次はChef ZeroとBerkshelfを使ってサーバ構築の自動化に挑戦したいと思います。

## 参考

* [config.vm - Vagrantfile - Vagrant Documentation](http://docs.vagrantup.com/v2/vagrantfile/machine_settings.html)
* [Vagrant で複数のVM を立ち上げて、お互いに通信できるようにするには - Qiita](http://qiita.com/sho7650/items/cf5a586713f0aec86dc0)
