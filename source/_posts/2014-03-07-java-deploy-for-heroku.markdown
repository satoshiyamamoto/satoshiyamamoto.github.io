---
layout: post
title: "JavaアプリをHerokuにデプロイする"
date: 2014-03-07 10:31:53 +0900
comments: true
categories: 
---

HerokuのPaaSは、Railsの他にScala、PHPなど、様々な言語に対応しています。もちろん、Javaアプリケーションを稼働させることができます。Javaの場合、デプロイは次のような流れで行なわれます。

1. Mavenプロジェクトの作成
2. Gitリポジトリにpush
3. Mavenビルド (Heroku)
4. Procfileの実行 (Heroku)

この様に、Railsのデプロイと比べると、Javaの場合は少しだけコツが必要です。今回は、その方法をめとめておきます。

## JDKのバージョン指定

HerokuのJDKは、デフォルトでOpenJDK 1.6が使用されます。Maven Compilerプラグインで1.7を指定している場合は、pom.xmlと同じ階層に system.propertiesを作成します。内容は次のとおりです。

    java.runtime.version=1.7

こうすることで、JDK1.7の構文で書かれたプロジェクトをビルドできるようになります。

## Jetty Runnerプラグインの設定

HerokuでJavaアプリを起動させるために、コマンドラインからwarプロジェクトを起動できるようにします。それにはpom.xmlに次のプラグインを追加します。

    <plugin>
    	<groupId>org.apache.maven.plugins</groupId>
    	<artifactId>maven-dependency-plugin</artifactId>
    	<version>2.8</version>
    	<executions>
    		<execution>
    			<phase>package</phase>
    			<goals>
    				<goal>copy</goal>
    			</goals>
    			<configuration>
    				<artifactItems>
    					<artifactItem>
    						<groupId>org.eclipse.jetty</groupId>
    						<artifactId>jetty-runner</artifactId>
    						<version>9.1.2.v20140210</version>
    						<destFileName>jetty-runner.jar</destFileName>
    					</artifactItem>
    				</artifactItems>
    			</configuration>
    		</execution>
    	</executions>
    </plugin>

これで、Mavenのpackageフェーズでtargetディレクトリにjetty-runner.jarがコピーされます。ローカルで起動するには次のコマンドを実行します。

    java -cp target/dependency/* -jar target/depencency/jetty-runner.jar --port <PORT> target/*.war

warプロジェクトを直接アップロードしてデプロイする方法もあるみたいですが調べきれてません...

## Procfileの作成

先ほどの system.propertiesと同じディレクトリ階層に起動スクリプトのProcfileを作成します。jetty-runner.jarを使ったコマンドは次のとおりです。

    web: java $JAVA_OPTS -jar target/dependency/jetty-runner.jar --port $PORT target/*.war

## Herokuのプロセス確認

デプロイが完了したら、プロセスが立ち上がっているか確認しましょう。まず、herokuにログインします。

    $ heroku login
    Enter your Heroku credentials.
    Email: <your email>
    Password (typing will be hidden):
    Authentication successful.

heroku psコマンドでプロセスを確認します。

    $ heroku ps --app mogi === web (1X): `java $JAVA_OPTS -jar target/dependency/jetty-runner.jar --port $PORT target/*.war`
    web.1: idle 2014/03/06 11:20:05 (~ 23h ago)

## まとめ

個人的にはProcfileの作成とJetty Runnerプラグインの設定でハマりました。HerokuのGitリポジトリにpushすると、自動的にMavenのビルドが始まるので、このような設定が必要だとは思いもよりませんでした。最初からオフィシャルのマニュアル読めよ！って話ですね。

## 参考
* [Heroku Java](http://java.heroku.com)
* [全くの初心者だが Heroku の Java サンプルを試してみた。 - 酒浸りの日々](http://d.hatena.ne.jp/beercan/20110912/1315835179)
