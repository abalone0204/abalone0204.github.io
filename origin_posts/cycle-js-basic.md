{{{
"title": "做中學 Cycle.js（上）",
"date": "2016/1/23",
"intro": "Observable 是世界上最強的，其他東西都是——",
"tags": ["cycle.js", "observable"]
}}}

# 前言

之前 Redux 和 Cycle.js 作者在爭辯何者才是較好的 paradigm，

其實軟體開發裡面沒有銀彈，

不過這種爭辯也更能夠激發出我們寫出更棒的軟體，

並且去反思現行流行的東西真的是「好」的嗎？

Cycle.js 的作者也寫下一篇他認為為什麼 Redux + React 不那麼好的原因：

- [WHY REACT/REDUX IS AN INFERIOR PARADIGM](http://staltz.com/why-react-redux-is-an-inferior-paradigm.html)

同時間在 egghead.io 上也 release 了一個 Cycle.js 的課程：

- [Lessons of Cycle.js](https://egghead.io/lessons/rxjs-the-cycle-js-principle-separating-logic-from-effects)

(等等，這時機推出課程，真的不是在打廣告嗎？)

我認為這個課程還蠻推薦的原因有底下兩點：

- **作者會告訴我們 Cycle.js 這樣設計的理念**
  對我來說在學習一個框架時，
  如果你不能理解為什麼要這樣設計，
  那你就是用硬背的，這樣很容易忘記；
  但如果你知道為什麼要命名成這樣、為什麼要這樣設計，
  你等於進入了框架本身去使用它，
  而不是被它框住。

- **Observable 給我們不一樣的方式來思考如何 Handle events**
  可以看看 Netflix 的[例子](https://www.youtube.com/watch?v=XRYN2xt11Ek&hd=1)

週末在家拉肚子之餘，順便把課程課完並做了一些筆記。

先來看一下 Cycle.js 的 Get started code，

```js
import Cycle from '@cycle/core';
import CycleDOM from '@cycle/dom';

function main() {
  // ...
}

const drivers = {
  DOM: CycleDOM.makeDOMDriver('#app')
};

Cycle.run(main, drivers);
```

現在看起來很不習慣，但這篇會從無到有的建一個簡單版的 Cycle.js 出來，

第一篇預計會實作很 primitve 的 drivers 以及 main，

接著會把 run 給重構到幾乎跟現在 Cycle 核心中的寫法一樣。

（當然還是只重概念說明的簡化版）

不過這都只是個人的學習筆記，

還是在大大推一下 egghead.io 上的課程

- [Lessons of Cycle.js](https://egghead.io/lessons/rxjs-the-cycle-js-principle-separating-logic-from-effects)

## Prerequisite

- 了解如何操作 collection 
  沒錯，Observable 和 array（或list）都是 collection

- 可以試試這個互動的課程，再來看這系列會更有感覺：
  [http://reactivex.io/learnrx/](http://reactivex.io/learnrx/)

- 對於 Rx 已經有基礎的認識

## Cycle.js

### Basic Principle

- 第一條規則就是要將「logics」跟「effect」分開

要來分清楚這兩個東西是什麼就要先來看一下程式碼了：

```js
// Logic
Observable.timer(0, 1000)
    .map(i => `Seconds elapsed ${i}`)
// Effect 
    .subscribe(text => {
        const container = document.querySelector('#app');
        container.textContent = text;
    })
```
這是一個從 0 開始每一秒一數的計數器，

詳情請見 [Timer](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/operators/timer.md)。

上半部的部分是「Logic」，

而 subscribe 那一行開始，就是他怎樣呈現的 「Effect」。

這裏有個很巧妙的概念，

就是 Effect 才是真正影響到外面世界的地方（DOM），

正如他的名字一樣；

而 Logic 裏的東西只是單純的 Event stream，

我們不去 Subscribe 他們，就不會有任何事情發生。

Cycle.js 的原則就是將這兩大部分分開，

`Effect` 的部分是 Imperative 的，讓 Framework 幫你完成，

身為開發者我們只要關心 `Logic` 的部分就夠了，

而 Logic 的部分是 functional 的。

### Main 以及 Effect 

前面提到我們會將 logic 和 effect 分開，

在 Cycle 中我們習慣會將 Logic 放到 main 裡面。

```js
function main() {
    return {
        DOM: Observable.timer(0, 1000)
                       .map(i => `Seconds elapsed ${i}`),
        Log: Observable.timer(0, 2000)
                       .map(i => 2*i)
        }
}

function DOMEffect(text$) {
    text$.subscribe(text => {
        const container = document.querySelector('#app');
        container.textContent = text;
    })
}

function consoleLogEffect (msg$) {
    msg$.subscribe(msg => console.log(msg));
}

const sink = main();
DOMEffect(sink.DOM);
consoleLogEffect(sink.Log);
```

我們在 Main 裡面建了兩條不同的 stream，

看起來已經將邏輯集中起來放，

但是最下方從 sink 開始，

似乎還是太 imperative 地去做這些事情。

> 我們 Hard Coding 的去指定 consoleLogEffect 這個函數，

> 一旦我們今天把 main 中的 log 拔掉，

> 那整個程式就會報錯了，

> Cycle.js 中不希望我們每次更動 Logic 時需要注意一大堆 effect

再來就要介紹一下 `run` 這個 function。

### `run`

```js

function run(mainFn, effects) {
    const sinks = mainFn();
    Object.keys(effects)
    .forEach(key => {
        effects[key](sinks[key])
    })
}

const effectsFunctions = {
    DOM: DOMEffect,
    Log: consoleLogEffect,
}

run(main, effectsFunctions);
```

`run` 會吃兩個參數，第一個就是我們管邏輯的 main，

第二個則是 effect，

我們如果不想要他在畫面上做事情，

把在 effectFunctions 中的那個 key 給註解掉就行了，

因為我們並沒有很 hard coding 的去呼叫每個 effectFunction。

但是這裡要重新命名一下，將 effectFunctions 改成 drivers，

一來是因為 effectFunctions 聽起來並不是個好命名方式XD

二來是 drivers 即是我們熟悉的驅動程式，建立了硬體和軟體中間溝通的介面；

而這裡的 driver 可以想成我們的程式(logic)，和畫面(effect)中間溝通的介面；

還是很抽象嗎？

那就從字面上的意思來看， driver 就是駕駛員，

現在有一個駕駛員負責開著一台小車車，

嘟嘟嘟的把我們寫的邏輯運送到畫面上，

我們只要寫好邏輯、還有要送去的地方跟方式，

剩下的就交給 driver 幫我們處理啦！

```js

function run(mainFn, drivers) {
    const sinks = mainFn();
    Object.keys(drivers)
    .forEach(key => {
        drivers[key](sinks[key])
    })
}

const drivers = {
    DOM: DOMDriver,
    Log: consoleLogDriver,
}

run(main, drivers);
```

這是我們手刻出來的簡單版本，

而 Cycle.js 首頁的 get started 例子中，

輪廓的確就是這樣子，

只是在 driver 的部分，

Cycle.js 幫我們做了更多事情。

```js
function main() {
  return {
    DOM: Rx.Observable.interval(1000)
      .map(i => CycleDOM.h1('' + i + ' seconds elapsed'))
  };
}

const drivers = {
  DOM: CycleDOM.makeDOMDriver('#app')
};

Cycle.run(main, drivers);
```

### Read effects from the External world

前面有提過 Netflix 解決複雜電影選單的方式，

就是透過 Observable 來重新思考處理 Events 的方式，

但到目前為止，我們都還沒有用到最精髓的部分，

而是把內部 Logic 寫好，沒有接收任何外來的 event stream。

奠基於 Rx 上面的 Cycle.js 最精華的也正是這一段處理 event 的方式，

同時這也是 **Cycle** 這名字的由來。


首先先看前面寫的程式碼：

```js
function DOMDriver(text$) {
    text$.subscribe(text => {
        const container = document.querySelector('#app');
        container.textContent = text;
    })
}
function main() {
    return {
        DOM: Observable.timer(0, 1000)
            .map(i => `Seconds elapsed ${i}`),
        Log: Observable.timer(0, 2000)
            .map(i => 2 * i)
    }
}
```

可以看到它只有 input，沒有 output。

而 main function 則反之，

我們想從外部 read something ，就代表我們的 main 必須要有 input。

> 這裏的前提是你照著 cycle.js 的單向資料流架構走

所以我們先在 main 和 driver 各加上 input 和 output。

接下來在 `run` 中會改回 hard code 的方式，

這是為了更容易去理解，接著就會遇到最奇妙的地方：


```js
function DOMDriver(text$) {
    text$.subscribe(text => {
        const container = document.querySelector('#app');
        container.textContent = text;
    })
    const DOMSource = Observable.fromEvent(document, "click");
    return DOMSource
}
function run(mainFn, drivers) {
    const sinks = mainFn(DOMSource);
    const DOMSource = drivers.DOM(sinks.DOM);
 
}
```

我們看到 run 中間，DOMSource 需要 sinks 才能建立，

但 sinks 也需要 DOMSource 才能被建立，

形成一個很微妙的循環，是一個雞生蛋蛋生雞的問題。

在更抽象化一點就是：

```js
a = f(b)
b = g(a)
```

想要解決這件事其實沒那麼難，

想法上是這樣：

```js
bProxy = ...
a = f(bproxy)
b = g(a)
bProxy.imitate(b)
```

這裏要靠 rx 裡面的 subject 來建立我們的 proxy。

> 瞭解更多關於 Subject: 

> from [rx-book](http://xgrommx.github.io/rx-book/content/getting_started_with_rxjs/subjects.html)

> 簡言之它同時繼承了 Observable 跟 Observer，

> 所以我們既可以 subscribe 它，（Observable）

> 又能夠對他呼叫 onNext、onError，以及 onCompleted（這就是 Observer 在做的事情）

這裏就比較困難要分段看了，

先到 run 裡面看看我們要怎麼按照上方的 pattern 來加入 proxy。

```js
function run(mainFn, drivers) {
    const proxyDOMSource = new Subject();
    const sinks = mainFn(proxyDOMSource);
    const DOMSource = drivers.DOM(sinks.DOM);
    DOMSource.subscribe(click => proxyDOMSource.onNext(click))
}
```
DOMDriver 回傳了一個 click-event 的 stream（Observable），

所以我們 subscribe 它，並且每一次呼叫 click 的 stream，

跟我們前面創造的 proxy 整合在一起，

下來再來看 proxy 傳進 main 發生了什麼事情。

```js
function main(DOMSource) {
    const click$ = DOMSource;
    return {
        DOM: click$
        .startWith(null)
        .flatMapLatest(() =>
            Observable.timer(0, 1000)
            .map(i => `Seconds elapsed ${i}`)
        ), 
        Log: Observable.timer(0, 2000)
            .map(i => 2 * i)
    }
}
```

簡單說就是我們每次在螢幕上按一下(click)，

就會重啟整個 timer。

> 歸功於 flapMapLatest 這個 operator，

> 假如這裡改用 flapMap 的話，會發現舊的 stream 還在繼續跑，

> 整個 timer 會被搗亂，假如還不熟 flatMap 該怎麼用

> 請至 prerequisite 玩一下 [learn-rx](http://reactivex.io/learnrx/)

而 `startWith(null)` 則是製造一次「假的」 event，

來觸發第一次還沒 click 之前的 effect。

現在的 code 看起來很糟糕，尤其是在 main 中 hard code DOMSource 這一點。

首先先從 run 中下手：

```js
function run(mainFn, drivers) {
    const proxySources = {};
    Object.keys(drivers).forEach(key =>{
        proxySources[key] = new Subject();
    })
    const sinks = mainFn(proxySources);
    Object.keys(drivers)
          .forEach(key => {
            const source = drivers[key](sinks[key])
            source.subscribe(x => proxySources[key].onNext(x))
          })
}
```

如此一來我們就不用去 hard code 的指定每個 proxySource，

而在 main 中簡單多了，只要把 click$ 的來源變成 sources.DOM 就好了，

但在這裡我們可能會對一個 undefined 呼叫 subscribe。

> consoleLogDriver 並沒有 return 任何東西（nothing to be read）

要避免這點只要加個判斷式就能夠解決，

不過截至目前為止，我們其實已經把 Cycle core 中的 run 給實作的差不多了！

- [source code of run in Cycle.js](https://github.com/cyclejs/cycle-core/blob/master/src/cycle.js#L97:L118)

> 當然還是有些差異在，像是 error-handling，

> 以及在 Cycle core 的 proxy 中是用 `ReplaySubject` 而不是 `Subject`


----

# 參考資料

- [WHY REACT/REDUX IS AN INFERIOR PARADIGM](http://staltz.com/why-react-redux-is-an-inferior-paradigm.html)

- [UNIDIRECTIONAL USER INTERFACE ARCHITECTURES](http://staltz.com/unidirectional-user-interface-architectures.html)

- [Rx-book](https://xgrommx.github.io)