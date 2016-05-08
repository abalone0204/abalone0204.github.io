---
title: 淺入淺出 Generator Function
tags:
  - es6
  - generator
  - function
date: 2016-05-08 21:43:08
---


es 2015 中有一項新的 feature 叫做 **generator function**，

假如熟稔其他語言的人，

可能都知道 generator function 是什麼，

不過對於一位平常都在寫 JavaScript 的人，這就很新鮮了。

（前提是如果你現在還是很習慣不小心製造 callback hell 的話）

使用 Generator function 並不是一件求新求潮的一件事情，

活用 Generator function 能讓測試以及開發非同步的程式碼都變得更直觀。

這篇文章就來淺淺的介紹一下 Generator function 究竟是什麼。

> 其實本來是要寫關於`redux-saga`的，
> 只是不先介紹 generator 真的講不下去 QQ

<!--more-->

這篇會包含以下幾個主題：

- [Generator function 是什麼](#Generator-function-是什麼)

- [Syntax](#Syntax)
    
    - [宣告一個 Generator function](#宣告一個-generator-function)

    - [`yield`](#yield)

    - [`next`](#next)

    - [在 `next` 中傳入參數](#在-next-中傳入參數)

    - [`for...of`](#for-of)

    - [Error handling(optional)](#Error-handling-Optional)

    - [Delegating Generators - Generator 中的 generator (optional)](#Delegating-Generators-Generator-中的-generator-optional)

- [如何啟用 Generator function](#如何啟用-Generator-function)

- [實際上的應用](#實際上的應用)

----

# Generator function 是什麼

先來講講我們熟悉的 function：

```js
function foo() {
    for (var i=0; i<=1E10; i++) {
        console.log(i);
    }
}
// 0, 1, 2.... 1E10
```

是個 run-to-completion 的 function，

一旦進去了，就會一直執行到結束，

看上述的 code 就知道這個東西會執行的非常非常久，

因為它一旦進入，就要執行到被完成為止。

generator function 特別的地方就是它可以被暫停，

等到下次進來時再繼續呼叫它。


先看下方這個改寫過後的小例子：

```js
function* generatorFoo() {
    for (var i=0; i<=1E10; i++) {
        console.log(i)
        yield i
    }
}

const iterator = generatorFoo()

iterator.next() // 0
iterator.next() // 1
iterator.next() // 2
```
可能會有點不熟悉這樣的語法，

不過可以先感受一下一次拿一個值出來，

以及可以被暫停的 function 是長什麼樣子。

下面來更清楚地敘述一下 generator function 的語法。


# Syntax

## 宣告一個 generator function

```js
function* generatorFoo() {
    // ...
}
```

`function` 後面或多個 `*`。

> 有人會爭論到底是要放在 function 關鍵字後面，
> 還是直接放在 function 名字前面
> e.q: `function *generatorFoo`
> 兩個都是合格的語法，
> 不過我習慣放在 `function` 關鍵字後面，
> 我認為這是個不同的 `function`，
> 而且 function name 本身並不該包含 `*`
> 至於參考資料裡面有附上 [MDN 中對於 generator function 的語法介紹](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)，
> 也將 `*` 放在緊接著 `function` 關鍵字的後方。
> 這一部份還沒做過更全面的研究，
> 畢竟最近也才在實作中加入 generator function 而已


## `yield` 

這個關鍵字估計就是 generator 中最特別的概念了，

```js
function* generatorFoo() {
    for (var i=0; i<=1E10; i++) {
        console.log(i)
        yield i
    }
}
```

當我們呼叫 `generatorFoo` 時，

會得到一個 iterator，

當我們每次呼叫這個 iterator 的 `next` 方法時，

就會執行 `generatorFoo`，一直到出現 `yield` 關鍵字的地方，

接下來會暫停，直到下次呼叫 `next`。

> 我知道還有 `yield*`，不過這個概念等後面再說

## `next`

```js
const iterator = generatorFoo()

iterator.next() // 0
```

像以上的例子，第一次呼叫 `next` 時，

就會執行到 `yield i` 這個位置，接著暫停這個函數，

直到下次執行`next`。

- `next()` 返回什麼？

`next` function 會返回一個物件，裡面包含著兩個 properties，

分別是 `value` 和 `done`：

`value`，就是我們在前一段中從 `yield` 那個位置，

接到的「值」。



`done` 是個 boolean 值，

假如這個 generator function 完全被執行完的話，

`done`就會變成 `true`，反之亦然。

這裡要注意的是當執行到最後一個 `yield` 時，

`done` 仍然會是 `false`，

再執行一次才會得到 `done` 為 `true` 的結果。

而 generator function 仍然是一個 function，

我們可以在裡面 `return` 東西，

如此在執行到 `return` 這一行時，

`next` 就會返回 `value` 為 `return`的東西，

並且 `done` 為 `true`。

> **提醒**：
> 如果你真的需要 return，那你很可能只需要普通的 function 就足夠
> 在 generator function 裡面 return 東西，
> 容易令人感到困惑，簡言之，沒事別這樣做。

## 在 `next` 中傳入參數

我們可以這樣做：`next(x)` ，

這樣做的結果就是將`x`塞入前一個 `yield` 產生的地方。

直接看例子會更有感覺


```js
// source from:  https://davidwalsh.name/es6-generators
function *foo(x) {
    const y = 2 * (yield (x + 1));
    const z = yield (y / 3);
    return (x + y + z);
}

const iterator = foo(5);

console.log(iterator.next());       // { value:6, done:false }
console.log( iterator.next( 12 ) );   // { value:8, done:false }
console.log( iterator.next( 13 ) );   // { value:42, done:true }
```

第一個 `next` 不傳入參數是因為在這之前，

不會有前面一個 `yield`。

而第二個 `next` 中傳入的 `12`完全替代掉了前面 `x+1` 的值，

所以後面的 `z` 會等於 `12*2/3`，也就是 `8`。

最後一個傳入 `13`，是替代掉第二個 `yield` 所產生的值，

這裡已經可以完全忽略 `y/3` 是什麼，直接替代成 `13｀了。

將上述的值全部替代進去會長成下面這樣：

```js
function *foo(x) {
    const y = 2* (12)
    const z = (13)
    return 5+24+13
}
```

> 看起來很蠢沒錯，
> 不過這樣替代值的方式也許比直接文字描述來的更直觀一些


## `for...of`

```js
function* foo() {
    yield 0
    yield 1
    return 2
}
```

我們一樣可以使用 `for` 來遍歷整個 iterator，

不過要注意的是，我們只會拿出 `done`為 `false` 的值，

也就是說上述的 `2` 並不會在 `for...of` 中被拿到。

## Error handling (Optional)

雖然說是 optional，

不過為了在實戰中時寫出更 robust 的程式碼，

瞭解 error 要如何處理是很重要的，

畢竟你連丟出來都沒辦法， unit-test 就測不了啦！

在 generator 中可以用我們熟悉的 try...catch 技法來做到 error handling：

```js

function *foo() {
    try {
        const x = yield 3;
        console.log( "x: " + x ); // may never get here!
    }
    catch (err) {
        console.log( "Error: " + err );
    }
}

```

比較不一樣的是我們能在外面直接把 error 丟進去：

```js
const iterator = foo()
iterator.next() // x: 3
                // {value: 3, done: false}
iterator.throw('error messages') // Error: error messages
```

比較 tricky 的地方是我們把 error 給丟進去，

所以如果在 generator 內部沒有catch 到，

這個 error 會丟出來外面被 catch 住：

```js
function *foo() { }

const iterator = foo();
try {
    it.throw( "error message" );
}
catch (err) {
    console.log( "Error: " + err ); // Error: error message
}
```

## Delegating Generators - Generator 中的 generator (optional)

我其實不喜歡用 delegate 這個字來解釋，

總覺得有點在賣弄的感覺 XD。

簡言之就是將遍歷 generator 的控制權交（delegate=委託）給內層的 generator。

Talk is cheap, show me the code:

```js
function* foo() {
    yield 3
    yield 4
}

function* bar() {
    yield 1
    yield 2
    yield* foo() // `yield *` delegates iteration control to `foo()`
    yield 5
}

for (let v of bar()) {
    console.log(v)
}

```

執行到 `yield* foo()` 時，

就會把控制權交到 `foo()` 所產生的 iterator 上，

所以最後那個 `for...of` 就會印出 1~5 。

基本上就是這樣而已，我認為了解到這點就已足夠，

在這裡敘述太多語法卻沒加上實際應用，

真的只會搞混而已，

所以請容我到之後應用篇時再繼續說明 delegating 的好處。

如果現在就想再鑽下去，可以看下方的參考資料。

# 如何啟用 Generator function

假如你是使用 webpack 來做前端資源的打包，

恭喜你，這是一件再簡單不過的事情；

假如不是的話，你可以：

- 學會使用 webpack

- 或是想辦法跟 Babel 搭起來

我們這裡會運用 babel 來幫我們非常簡便的在專案中啟用 generator function。

> [https://babeljs.io/](https://babeljs.io/)

當然，如果要連 webpack 一起介紹會太囉唆，

以下都假設你已經會實際使用 webpack 的 babel-loader：

es2015 這個 preset 已經包含了 generator function。

preset 只幫我們做到 transform 的功能，

真正要在實際環境中動起來還需要 polyfill 的幫忙：

所以我們必須要安裝：

```
npm install --save-dev babel-preset-es2015
npm install --save babel-polyfill
```

接著在 `.babelrc` 中

```js
{
  "presets": ["es2015", "react"]
}
```

最後在你要用到 generator function 的地方加上 `import 'babel-polyfill'`。

> 其實這裡不一定需要用到整包 polyfil，
> 只要有`regeneratorRuntime`被定義好就行了
> 有興趣的人可以參考一下 facebook 的 [regenerator](https://github.com/facebook/regenerator)
> babel-polyfill 裡面也是用到這個 project

# 實際上的應用

其實這篇文章主要是系統性的介紹 generator 到底是什麼，

下篇文章會介紹我們實際在應用時，

generator 能幫助我們做到什麼。

如果要一言以蔽之的話，

那就是：

「能將非同步的程式碼，用同步的語法來呈現。」

乍看之下很神奇，

但在你了解 generator function 不過就是個能夠暫停、繼續的 function 後，

就大概能對他能做到的是有最初步的想像了。


# 參考資料

- [The Basics Of ES6 Generators](https://davidwalsh.name/es6-generators)

- [Diving Deeper With ES6 Generators](https://davidwalsh.name/es6-generators-dive)

- [function*](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)