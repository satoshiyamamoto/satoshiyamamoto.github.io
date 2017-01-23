---
layout: post
title: "React初めました (基礎編)"
date: 2017-01-23 10:05:19 +0900
comments: true
categories:
---

近頃のフロントエンド技術の過熱ぶりに触発されて、サーバーサイドの裏方から日の当たる表舞台に興味が湧き始めました。egghead.ioの[Start Using React to Build Web Applications](https://egghead.io/courses/react-fundamentals)を受講したので忘れないよう覚書を残しておきます。

## 1. Reactとは

![Capture](/images/20170123/react_logo.png =400x100)

ReactはFacebookが開発したユーザーインターフェイスを構築するためのJavaScriptライブラリです。主に次の三つの特徴を持ちます。

- 宣言的
- Componentベース
- 一度学べばどこでも書ける (Web/iOS/Android/Node.js SSR)

## 2. プロジェクト作成
一般的に、Reactアプリケーションのプロジェクト作成は、npmモジュールの依存解決やwebpackの設定が必要で、お世辞にも敷居が低いとは言えませんでした。

現在では、Facebookが[react-create-app](https://github.com/facebookincubator/create-react-app)というBoilerplateを提供していますので、それを利用するのが一番簡単です。また、単純なコードを動かすだけであれば[CodePen](http://codepen.io)でインタラクティブに動作確認ができます。

react-create-appを使ったプロジェクト作成手順は次の通りです。
{% codeblock lang:bash %}
$ npm install -g react-create-app
$ react-create-app myapp
$ cd my app
$ npm start
{% endcodeblock %}

## 3. JSX
JSXは、ReactでUIを記述するためのJavaScript拡張構文で、コンパイラによって次の様な関数に変換されます。

{% codeblock lang:javascript %}
<h1>Hello World</h1>

=> React.createElement(‘h1’, null, ‘Hello World’);
{% endcodeblock %}

## 4. Component
Componentは概念的にJavaScript関数で、入力としてPropsを受け取り、レンダリングするための何かしらの要素を戻り値として返します。

<p data-height="265" data-theme-id="0" data-slug-hash="QGzwQy" data-default-tab="js,result" data-user="satoshiyamamoto" data-embed-version="2" data-pen-title="React - Hello World" class="codepen">See the Pen <a href="http://codepen.io/satoshiyamamoto/pen/QGzwQy/">React - Hello World</a> by Satoshi Yamamoto (<a href="http://codepen.io/satoshiyamamoto">@satoshiyamamoto</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

ES2015のClass構文を使う場合は `React.Component` クラスを継承します。Atomエディタのreactプラグインを使えば `rcd6` のスニペットで簡単にクラス定義の雛形が挿入できます。

### 4.1 Props
前述のようにPropsは、外部からComponentに渡される不変なパラメータです。JSXではHTML属性に似た構文で値を渡します。

{% codeblock lang:javascript %}
<App text=“Hello from the Props” />
{% endcodeblock %}

ComponentでPropsを参照する方法です。
{% codeblock lang:javascript %}
// ES5
const App = (props) => <h1>{props.text}</h1>

// ES2015
class App extends React.Component {
  render() {
    return (
      <h1>{this.props.text}</h1>
    )
  }
}

=> <h1>Hello from the Props</h1>
{% endcodeblock %}

#### 4.1.1 バリデーション
Propsの型と値の有無をチェックできます。バリデーションで不備を検知すると実行時エラーが発生します。

{% codeblock lang:javascript %}
App.propTypes = {
  text: React.PropTypes.string
  count: React.PropTypes.number.isRequired
}
{% endcodeblock %}
スニペット: `pt6`

#### 4.1.2 デフォルト値
Componentの呼び出しでPropsが省略された場合、ここで設定したデフォルト値が使われます。

{% codeblock lang:javascript %}
App.defaultProps = {
  text: 'Please input text...',
  count: 0,
}
{% endcodeblock %}
スニペット: `dp`

### 4.2 State

Componentの状態やユーザー入力など、可変で動的な値を管理します。Stateの詳細については次の記事で紹介する予定です。

{% codeblock lang:javascript %}
constructor() {
  // initialize
  this.state = { … };
}

onChangeInput(e) {
  // update
  this.setState({ name: e.target.value });
}
{% endcodeblock %}
スニペット: `state`

## まとめ

Reactでアプリケーションを開発するにあたり、基本的なプロジェクトの作成方法とComponentの実装を紹介しました。Reactを使えば、ステートレスでメンテナンス性の高い再利用可能なViewComponentを作成できます。次は、実際にReactを使ったアプリケーションの設計や実装方法について記していきます。
