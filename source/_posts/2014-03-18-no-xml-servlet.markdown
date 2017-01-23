---
layout: post
title: "Servlet 3.0でNoXMLなContext設定"
date: 2014-03-18 18:54:59 +0900
comments: true
categories: 
---

　サーブレットをアノテーションで記述できるようになったServlet 3.0ですが、同じようにweb.xmlもJavaConfigで記述できるようです。JAX-RS 2.0などと同じイメージです。

参考までに、それぞれ同じ設定のweb.xmlとJavaConfigを比較しておきます。まずは、web.xmlから。

{% gist 9617142 web.xml %}

このファイルでは、SpringのDispatcherServletとフィルタの登録を行っています。続いて、JavaConfigです。

{% gist 9617044 Application.java %}

こちらは、前述の設定とApplicationContextの設定を行っています。長年、XMLに慣れてしまったため違和感がありますが、自分はこちらの方が好みです。

長い時間を掛けてシステムを運用していくと、設定ファイルが肥大化していく傾向にあるため、プログラミングベースで設定を記述できれば、単純なスペルミスなど防ぎやすくなると思います。やはり、Converntion over Configurationですね。
