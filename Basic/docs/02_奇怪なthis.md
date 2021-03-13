# 奇怪なthis

JavaScriptのthisで混乱する人は少なくありません、ここではJavaScriptのthisについて理解を深めていきます。

またChromeの検証ツールを使っていろいろと実験をしていきます。



## グローバル空間でthisを書くと

```js
this;
```

ただ`this`と書くと、ConsoleにはWindowと表示されると思います。

一般的に`this`というのは自分自身を指すとか言われますが、今回は「thisは自分が所属している場所」という表現をしてみます。

そう考えるとJavaScriptというのは、言ってしまえばブラウザウィンドウの中のお話なので、**グローバルな場所＝ブラウザウィンドウの中**と考えるといいかもしれません。

グローバル空間はウィンドウに所属しているので、グローバル空間で`this`と書くと、現在所属しているのはwindowオブジェクトだよーということでWindowオブジェクトがthisに該当します。

このあたりの感覚が他の言語とは一線を画している気がするのですが、この感覚にまず慣れるのがポイントになると思います。



## 関数の中でthisを書くと

```js
function test() {
    console.log(this);
}
test();
```

この場合も`this`はWindowと表示されるかと思います。

これは`test`という関数は実質、`window.test`というウィンドウオブジェクトのプロパティ(メソッド)なので、`test`はwindowに所属していると言えます。

なので関数の中で`this`を表示すると、「現在所属している場所」としてWindowと表示されます。



## クラスの中でthisを書くと

とても適当なクラスですがこんなクラスを書いてみます。

```js
class Hoge {
    constructor() {
        this.a = 10;
    }
    onClick() {
        console.log(this);
    }
}
// インスタンスを生成
hoge = new Hoge();
```



### `hoge.onClick`を呼び出すと

```js
// onClickを呼び出すと、Windowではなくhogeオブジェクトが表示される
hoge.onClick();
```

この処理は正しく書くと以下のようになります

```
window.hoge.onClick();
```

一見、hogeはwindowオブジェクトに所属しているように見えますが`onClick`が所属しているのは`hoge`オブジェクトなので、`onClick`の中の`this`は`hogeオブジェクト`になります。



### h1タグを作ってみる

```js
h1 = document.createElement("h1");
h1.innerText = "テスト";
document.body.appendChild(h1);
```



#### h1のクリックに`hoge.onClick`をセットして`h1.onclick`を呼び出すと

```js
h1.onclick = hoge.onClick;
h1.onclick();
```

consoleには`<h1>テスト</h1>`と表示されるはずです。

この処理も正しく書くといかのようになります

```
window.h1.onclick();
```

onclickという関数(メソッド)は`h1`オブジェクトに所属しているため、`onclick`の中の`this`は`h1`オブジェクトになるという仕組みです。

このようにJavaScriptでは関数(メソッド)の中の`this`というのは自分が所属しているオブジェクトになるという特徴があり、これがとても混乱を招きます。

※ちなみにブラウザの画面で普通にh1をクリックしても動作します。



#### thisを固定する(bind)

関数(メソッド)が呼び出されるたびに、`this`に何が入ってくるかわからないと普通に困ります。

なのでJavaScriptにあは`bind`というものがあり、これで`this`の中身を固定できます。

```js
// このようにするとhoge.onClickのthisはhogeに固定できる
h1.onclick = hoge.onClick.bind(hoge);

// これでonclickを呼ぶと、h1に所属はしているが、onlickの中のthisはhogeになる。
h1.onclick();
```



JavaScriptは関数(メソッド)もオブジェクトなので、`hoge.onClick`を普通の関数(メソッド)ではなくオブジェクト(インスタンス)と捉えられるといいかなと思います。

以下は疑似的なコードですが、`hoge.onClick`もオブジェクトと捉えて、こんなイメージができるといいかと

```js
// onClickのthisというメンバ変数にhogeをセットする
hoge.onClick.this = hoge;

// onClickを呼び出した時↑でthisはhogeを指定しているので、onClick内のthisはhogeになる。
hoge.onClick();

// thisにnullをセットしたら
hoge.onClick.this = null;

// このonClickの中だとthisがnullになる
hoge.onClick(); 
```



## Tips:JavaScriptの関数はnewできる

JavaScriptの関数はクラスとして使うこともできてしまったりします。

最近のJavaScriptは普通に`class`キーワードが使えるのであまりなじみがないかもしれませんが、ReactのHooksや関数コンポーネントだと、考え方としては登場するので簡単に紹介します。



```js
function Hoge() {
    this.a = 10;
    this.onClick = () => {
        console.log(this);
    }
}
```

JavaScriptの関数は

```js
hoge = new Hoge();
```

このように`new`してインスタンス化することができます。

え？関数なのに`new`できるの？と思われるかもしれませんが、JavaScriptの関数というのは関数にもなれるし、クラスにもなれます。

単純に関数の定義もクラスの定義も同じfunctionというキーワードを使えるので、単純に書き方がキモイのです。

昔は`function`しかなかったので`function`でクラスも書いていましたが、最近は`class`キーワードが使えますし、そっちで書いた方がわかりやすいと思います。

また、普通に書くとめんどいけど、これ使うと楽に書けるよというのを**糖衣構文**とか**シンタックスシュガー**といいます。

そういうのがJavaScriptには沢山あります。