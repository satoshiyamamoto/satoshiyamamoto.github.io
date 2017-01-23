---
layout: post
title: "Java8の新機能に触れる(1) Lambda式編"
date: 2014-03-23 16:58:08 +0900
comments: true
categories: 
---
　Java8が正式リリースされて数日が経ちました。今回のアップデートは、大きく次のような機能を含んでいます。

* ラムダ式
* 型推論
* Stream APIの追加
* java.timeパッケージの追加
* Interfaceのデフォルトメソッドサポート

オブジェクト原理主義、冗長と揶揄されがちなJavaですが、世の流れを受けての洗練されてきたように感じています。Java8は、今後のJavaの方向性を示す意味でもTiger以来の大きな変化では無いでしょうか。

何はともあれ、まずはラムダ式から新しいJavaに触れてみたいと思います。

## 環境
* Oracle JDK 1.8.0_20-ea
* MacOS 10.9.2

## コードの例

人物を表す`Person`クラスがあります。このクラスに対応する`PersonRepository`は、ファインダメソッドで`PersonFinder`インターフェースを受け取り、条件に一致する人物を返します。それぞれのコードは次のとおりです。

{% gist 9720178 %}

{% gist 9720307 %}

{% gist 9720280 %}

## 匿名クラスからラムダへ

テストに使うデータは、AKB48のメンバー情報です。[pyakb48](https://github.com/moriyoshi/pyakb48)からCSVを拝借しました。チームAのメンバーだけ検索したい場合、Java7までの構文で実装すると次のようになります。

{% gist 9720387 %}

出力されるデータを確認してみます。

    ## Team A
    Person [name=岩佐美咲, birthday=1995-01-30, gender=FEMALE, team=A]
    Person [name=多田愛佳, birthday=1994-12-08, gender=FEMALE, team=A]
    Person [name=大家志津香, birthday=1991-12-28, gender=FEMALE, team=A]
    Person [name=片山陽加, birthday=1990-05-10, gender=FEMALE, team=A]
        :

分かりにくいですが、正しくフィルタリングされてますね。これをラムダ式で記述すると次のようになります。

{% gist 9720443 %}

同様に、ラムダ式で20歳以下を検索するには次の様に記述します。

{% gist 9720489 %}

従来までのコードに比べて無駄な情報が減った為か、コードが読みやすくなったと思います。また、今回実装した内容は、同じことが新しいCollection APIで出来るようです。それについては、後ほど記事にしたいと思います。

この投稿で利用したコードは[Github](https://github.com/satoshiyamamoto/java8-tutorial)にあげています。

## 参考

* [Lambda Expressions The Java™ Tutorials > Learning the Java Language > Classes and Objects](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html)
* [Java 8 新機能についてのまとめ 1 - A Memorandum](http://etc9.hatenablog.com/entry/2013/09/15/005516)
