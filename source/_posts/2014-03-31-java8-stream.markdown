---
layout: post
title: "Java8の新機能に触れる(2) Stream API編"
date: 2014-03-31 15:41:22 +0900
comments: true
categories: 
---

　Lambda式の投稿から、だいぶ時間が経ってしまいましたが、引き続きJava8の新機能について調べてみたいと思います。今回はjava.util.streamパッケージに追加されたStream APIです。

## 環境

* Oracle JDK 1.8.0
* MacOS 10.9.2

## Strem呼び出しと並列度

コレクションに対し、次のメソッドを呼び出すことでAPIの利用を開始します。`stream()`でシーケンシャル、`parallelStream()`で並列にストリームが開始されます。

{% gist 9886695 %}

## Stream APIについて

Stream APIは、これまでのコレクションAPIに比べ関数型言語でよく使われている便利なメソッド群が豊富に用意されています。例えば次の様なメソッドです。

* filter, forEach
* map, reduce
* min, max, distinct, xMatch 
* sorted, limit

これらは、自分自身を返すメソッドチェーンで実装されているため、可読性に優れたコードを記述することができます。

## FilterとforEach

これらのメソッドとラムダ式を使うことで、匿名関数を書くこと無くコレクションのフィルタリングが可能になります。前回に引き続き、AKBのメンバー情報からチームAだけを抽出します。

{% gist 9886798 %}

結果は以下の通りです。

    Person [name=岩佐美咲, birthday=1995-01-30, gender=FEMALE, team=A, captain=false]
    Person [name=多田愛佳, birthday=1994-12-08, gender=FEMALE, team=A, captain=false]
    Person [name=大家志津香, birthday=1991-12-28, gender=FEMALE, team=A, captain=false]
    Person [name=片山陽加, birthday=1990-05-10, gender=FEMALE, team=A, captain=false]
        :

forEachで個々の要素に対して同じ処理を実行できます。この例ではフィルターで抽出した高橋みなみに対して、キャプテンフラグを設定しています。

{% gist 9886829 %}
    
結果は次のとおりです。

    Person [name=高橋みなみ, birthday=1991-04-08, gender=FEMALE, team=A, captain=true]

見難いですが、正しく設定が反映されています。

## MapとReduce

関数型言語に通ずる方なら馴染みの深い関数です。Rubyで初めてこれらのメソッドに触れた私はピカピカの一年生と言ったところでしょう。

map、collect、また、reduce、injectなど、呼び方がいくつかありますが、前者はLisp、後者はSmalltalkから派生している呼び方だそうです。[Rubyist Magazine](http://magazine.rubyist.net/?0038-MapAndCollect)のコラムに詳しく書かれていました。

これらの関数も前述のコラムを参考に実装してみると次のようになります。

{% gist 9886899 %}

Stream APIには、これらの他にもデータ集合に対するmax, min, join, xxxMatchメソッドが追加されています。[Qiitaのこちらの記事](http://qiita.com/amay077/items/9d2941283c4a5f61f302)で詳しく解説されているので時間を見て試してみたいと思います。

この投稿で使用したコードは[Github](https://github.com/satoshiyamamoto/java8-tutorial)にあげています。

## 参考

* [Java 8 Tutorial -- Lambda Expressions, Streams, Default Methods and More](http://www.coreservlets.com/java-8-tutorial/)
* [Java8のStreamを使いこなす - きしだのはてな](http://d.hatena.ne.jp/nowokay/20130504)
