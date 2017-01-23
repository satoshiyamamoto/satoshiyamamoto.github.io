---
layout: post
title: "プロビジョニング事始め (3) Chef Zero編"
date: 2016-01-30 13:07:05 +0900
comments: true
categories:
---

　今更感が拭えませんが、乗りかかった船なのであと数回続けいきます。

## はじめに

　Chefとは、Chef社(旧Opscode社)が開発したプロピジョニングツールです。
レシピと呼ばれるRubyのDSLを使ってサーバのあるべき状態をコードに定義することで、サーバの構築作業を自動化します。テキストファイルの構築手順書と比べた場合、次の様なメリットがあります。

* 構築作業の自動化
* オペレーションミスの排除
* サーバーの抽象化
* コミュニティレシピの利用

　世界中の企業で導入されておりプロビジョニングツールとしてはデファクトスタンダードの実績を誇ります。
サーバーをChefの管理下に置くことで、たった1台のマシンから巨大なクラスタに至るまで、サーバーの構築、オペレーション作業を集中管理できるようになります。
Red Hat社が類似ツールのAnsibleを傘下に入れたことからも重要さを伺えます。

## Chef ServerとChef Zero

　Chefには、大きく分けてChef ServerとChef Zeroという二つの形態があります。

　Chef Serverはエンタープライズ向けで、サーバーの構成情報は中央のChef Serverで管理されます。ChefではこのようなChef Serverで管理されたノードをClientと呼びます。Chefの管理下にある各Clientのノードは、中央のChef Serverから自身の構成情報を取得し、その状態になるようパッケージのインストールや設定ファイルの更新を行います。
Chef Zeroは、小規模向けのツールでLocal Modeと呼ばれる方式を取ります。その名の通りローカルのCookbookファイルなどに従いレシピを自身に適用します。

　今回はChef Zeroの適用がメインなので、Chef ZeroのLocal Modeを使いVagrantのゲストOS、AWSのEC2の二つの異なる環境で同じレシピを適用します。レシピの記述には、Chef社で提供されているchef-dk(Chef Development Kit)を利用します。

## 実践

  では、実際にchef-dkを使ってローカルの仮想マシンにWordpressを構築してみます。また、Chefの基本的な説明は公式サイトや専門書に委ねて、VagrantとAWSのEC2でChefを適用させることに焦点をあてます。

## 環境

  環境は次の通りです。

* Chef Development Kit: 0.10
* ホスト: Mac OS X 10.11.1
* ゲスト: CentOS 7.1
* VirtualBox: 5.1.0
* Vagrant: 1.8.1
* AWS EC2: Amazon Linux AMI release 2015.09

## Chef Development Kitのインストール

### OS X

Homebrew Caskでインストールします。

```
$ brew cask install chefdk
$ chef -v
Chef Development Kit Version: 0.10.0
chef-client version: 12.5.1
berks version: 4.0.1
kitchen version: 1.4.2
```

chefコマンドで、それぞれの雛形を生成するには generator コマンド使っていきます。

```
$ chef generate
Usage: chef generate GENERATOR [options]

Available generators:
  app         Generate an application repo
  cookbook    Generate a single cookbook
  recipe      Generate a new recipe
  attribute   Generate an attributes file
  template    Generate a file template
  file        Generate a cookbook file
  lwrp        Generate a lightweight resource/provider
  repo        Generate a Chef code repository
  policyfile  Generate a Policyfile for use with the install/push commands
  generator   Copy ChefDK's generator cookbook so you can customize it
```

## Chefレポジトリ

　リポジトリとは、Chefが保持するデータの集合で、Cookbookなどのオブジェクトはリポジトリに格納されます。また、リポジトリは規模によって二つのタイプにわけられます。

　一つ目はchef repoです。サーバー台数が数台以上の環境に適しており、Chef管理下のNodeや、Roleと呼ばれるノードの役割、ProduntionやDevelopmentなどのサーバー環境に応じてリソースの属性を分けて管理するためのEnvironmentなどが含まれます。一般的に多く利用されるのはこの構成です。

　もう一つは、chef applicationです。その名の通り単一のアプリケーションを動作させるための非常にシンプル構成となっています。Vagrantfileも一緒に生成されるので開発環境の構築やテストに向いてます。

リポジトリの作成は次のコマンドで行います。
```
$ chef generate [repo|app] NAME
```

## Cookbookの作成

　Cookbookもchefコマンドで作成します。generate cookbook コマンドはカレントディレクトリにcookbookを生成するので、あらかじめchef-repo/cookbooksディレクトリに移動しておくとよいでしょう。

```
$ cd chef-repo/cookbooks
$ chef generate cookbook nginx
```

## Cookbookの開発

　レシピやテンプレートなど、cookbook内のリソースを作成するには generator コマンドにcookbook のパスを指定します。
```
$ cd chef-repo
$ chef generate recipe cookbooks/nginx ssl
$ chef generate template cookbooks/nginx centos/default.conf.erb
```

## レシピの適用

### Vagrant
　VagrantのゲストOSにレシピを適用します。Cookbooksは、[Github](https://github.com/satoshiyamamoto/cookbooks)に置いてあります。

```
$ vagrant box add bento/centos-7.1
==> box: Loading metadata for box 'bento/centos-7.1'
    box: URL: https://atlas.hashicorp.com/bento/centos-7.1
This box can work with multiple providers! The providers that it
can work with are listed below. Please review the list and choose
the provider you will be working with.

1) parallels
2) virtualbox
3) vmware_desktop

Enter your choice: 2
    :
```

Cookbooksを取得します。Vagrant 1.8からはchef_zero provisioner の実行にnodesディレクトリの指定が必要になってます。

```
$ git clone https://github.com/satoshiyamamoto/cookbooks.git
$ mkdir nodes
$ vagrant init bento/centos-7.1
```

最後にVagrantfileを編集してChef Zeroの設定を追加します。

```
config.vm.provision "chef_zero" do |chef|
  chef.cookbooks_path = "cookbooks"
  chef.nodes_path = "nodes"
  chef.add_recipe "wordpress"
end
```

後は、`vagrant up` でChefのレシピが適用され、Wordpressの管理画面が表示されているのが確認できます。

```
$ vagrant up
    :
==> default: Running provisioner: chef_zero...
    default: Installing Chef (latest)...
==> default: Generating chef JSON and uploading...
==> default: Running chef-client (local-mode)...
    :
==> default: Chef Client finished, 28/30 resources updated in 11 minutes 37 seconds
```

### Amazon EC2

　次は、同じレシピをAWSのEC2に適用します。Local Modeは、Sudo権限のみで秘密鍵での認証は必要ありません。CurlコマンドでChefのインストールします。

```
$ curl -L https://www.chef.io/chef/install.sh | sudo bash
```

　前回、作ったCookbookを利用します。

```
$ git clone https://github.com/satoshiyamamoto/cookbooks.git
```

　Chefの実行は、サーバーを構築するという性質上root権限が必要です。ここでは `-z` でLocal Modeと、 `-r` で Run-listsを指定しています。 `-r` オプションはカンマ区切りで複数指定できます。その他、`-E` でEnvironment、`-c` で `client.rb` などの設定ファイルを指定することも可能です。

```
$ sudo chef-client -z -r wordpress
```

　これで、Vagrantと同じWordPress環境が構築できました。もし気が向いたらChef Serverについても調べてみます。
