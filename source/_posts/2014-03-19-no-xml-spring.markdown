---
layout: post
title: "@ConfigurationでNoXMLなSpringアプリケーション設定"
date: 2014-03-19 10:05:08 +0900
comments: true
categories: 
---

　前回の続きSpringもXMLなしでapplicationContext.xmlの設定を記述します。JavaConfigのサポートから暫く立ちますが、正直あまり浸透していないようです。Spring IDEのお陰でXMLでも協力な入力補完が効くので、それに比べてあまりメリットが無い(ry

まずは、一般的なSpringMVCとTilesアプリケーションの設定例です。

{% gist 9633609 %}

続いてJavaConfigですが、基本的な考え方はFactoryパターンです。@Beanアノテーションで修飾したメソッドで、対象のBeanインスタンスを生成するだけです。

{% gist 9633643 %}

なお、このConfigクラスをサーブレットに登録するには、DispatcherServletのinit-paramを次のように設定します。

{% gist 9633712 %}

今回、作成したプロジェクトはGithubにあげています。[spring-mvc-quickstart](https://github.com/satoshiyamamoto/spring-mvc-quickstart)を`clone`し、`mvn jetty:run`で起動してみてください。HelloWorldが表示されます。

## 参考

* [Migrate Spring MVC servlet.xml to Java Config - Lucky Ryan](http://www.luckyryan.com/2013/02/07/migrate-spring-mvc-servlet-xml-to-java-config/)
* [WebApplicationInitializer](http://docs.spring.io/spring/docs/3.1.x/javadoc-api/org/springframework/web/WebApplicationInitializer.html)

