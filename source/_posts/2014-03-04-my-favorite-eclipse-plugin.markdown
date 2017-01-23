---
layout: post
title: "重宝しているEclipse Pluginたち"
date: 2014-03-04 14:32:30 +0900
comments: true
categories: 
---

Java屋として、仕事から普段まで切っては切れない縁のEclipse。intelliJ IDEAに乗り換えようと思った時も何度かありましたが、いまだEclipseを使っています。(intelliJ高い...) 

今回は、あると開発が便利になるプラグインを4つ紹介したいと思います。

1. Terminalプラグイン
2. Glanceプラグイン
3. Quick Junitプラグイン
4. Eclipse Color Themeプラグイン

## [Terminalプラグイン](https://code.google.com/p/elt/)

![Terninaml](/images/20140305/terminal.png)
　その名の通り、ターミナルエミュレータで開発元はGoogle,incです。Aptana StudioのRailsプロジェクトで存在を知ったんですが、Googleらしくシンプルで非常に使いやすく設計されています。

　Javaだとコードを書いてMavenでテスト実行、Gitで差分を確認なんかをやろうとしても、キーボードから手を話してマウスでメニューを辿って...という煩わしさから解放されます。ダントツ一位でおすすめのプラグインです。

## [Glanceプラグイン](https://code.google.com/p/eclipse-glance/)

![Glance](/images/20140305/glance.png)

　Eclipseの非力な検索を改善してくれるプラグインです。Eclipse標準のプラグインはモーダルダイアログで、検索結果がダイアログに隠れてしまうこともしばしば。これは、強力なインクリメンタルサーチでファイル以外にもEclipseのペインも検索対象にしてくれます。解説は、[情報科学屋さんを目指す人のメモ](http://did2memo.net/2012/11/06/eclipse-iterative-search-plugin-glance/)が詳しいので参考にしてみてください。

## [Quick Junitプラグイン](http://quick-junit.sourceforge.jp)

　その名の通り、JUnitの操作を高速にするプラグインです。⌘ + 9でテスティングペア間を移動、⌘ + 0でJUnitを実行できます。

## [Eclipse Color Themeプラグイン](http://eclipsecolorthemes.org)

![Color](/images/20140305/color.png)

　テキストエディタに好みのカラーテーマを適用するプラグインです。VimやEmacsなどで有名なカラースキームが[配布](http://eclipsecolorthemes.org)されています。私の場合はtomasr/molokai風のOvlibionというテーマを適用しています。

その他、Spring IDEやm2eなど外せないプラグインも数多くありますが、１つずつ上げていくと限がないので今回はここまで。

## 参考

* [Eclipseを改善するインクリメンタルサーチプラグイン「Glance」がオススメ！](http://did2memo.net/2012/11/06/eclipse-iterative-search-plugin-glance/)
* [ユカイ、ツーカイ、カイハツ環境！（16）：単体テストを“神速”化するQuick JUnitとMockito - ＠IT](http://www.atmarkit.co.jp/ait/articles/1008/02/news095.html)
