{{{
"title": "做中學 Cycle.js（中）",
"date": "2016/1/24",
"intro": "Drivers 和一些簡單的例子",
"tags": ["cycle.js", "observable"]
}}}

# Intro

還沒看過上一篇的可以先去看上一篇了解 Cycle.js，

這一篇會從 driver 開始講。

drivers 是在控制畫面的 render，

但是我們目前的 driver 都是只能回傳字串，

這一章節我們要真的來認真的操作 DOM，

並且實作幾個小例子來看看 Cycle.js 這個框架是怎樣改變我們思考資料流的方式。

## Making DOM driver more flexible

這裏要來認真處理一下如何去從 object 去表示一個 DOM，

假如你之前實作過一個 Virtual DOM 的話，

我想會相當有幫助。

```js
function createElement(obj) {
    const element = document.createElement(obj.tagName);
    obj.children
            .filter(c => typeof c === 'object')
            .map(createElement)
            .forEach(c => element.appendChild(c));
        obj.children
            .filter(c => typeof c === 'string')
            .forEach(c => element.innerHTML += c);
    return element;
}
```

目前還只是沒加上 props 的簡化版。

```js
function DOMDriver(obj$) {
    obj$.subscribe(obj => {
        const container = document.querySelector('#app');
        const element = createElement(obj)
        // Refresh
        container.innerHTML = '';
        container.appendChild(element);
    })
    const DOMSource = Observable.fromEvent(document, "click");
    return DOMSource
}
```

這裏使用了 appendChild，所以如果不每次都清空的話，

等於每次都會 append 東西上來。

## 在 DOM source 掌控更多事情

回頭看一下我們的 Main，

發現我們唯一能從 DOM 拿到的 event stream，

居然只有 click$，這並不符合我們日常的開發情境，

現在就來解決這個問題。

解法很簡單，就是在 return DOMSource 的時候，

給個能夠選取 tag 和 event type 的 interface。

```js
const DOMSource = {
    selectEvents: function(tagName, eventType) {
        return Observable.fromEvent(document, eventType)
            .filter(e => e.target.tagName === tagName.toUpperCase());
    }
};
```

這裏當然還是不夠 general 的版本，

不過這樣我們在 main function 裡面就能夠簡單的選取另一個 event 了。

## h()

一開始我也很疑惑 h 是啥？

答案很簡單， "h" stands for html

```
function h(tagName, children) {
    return {
        tagName,
        children
    }
} 

function h1(children) {
    return h('H1', children);
}
```

讓我們在 main 中要建造 elements 時省去不少力氣。

而 h1、h2、span⋯⋯等等你想得到的 tag，

都能藉由 function 來表示，

並且語法看起來也很簡單，

連我到後來都不禁思考：「**我們真的需要 jsx 嗎**？」

目前只是比較簡單的語法，還沒考慮到 properties，

在 main 中的長相大概會像這樣：

```js
Observable.timer(0, 1000)
          .map(i =>
                h1([
                    span([
                        `Seconds elapsed ${i}`
                    ])
                ]))
```

## Way to Real Driver

處理完語法後，我們來看看怎樣寫出一個更 serious 一點的 driver。

第一個發現的問題就是我們又把整個 Component 要 mount 的地方寫死了，

```js
DOMDriver(obj$) => {
        obj$.subscribe(obj => {
            // hard code
            const container = document.querySelector('#app');
            const element = createElement(obj)
            container.innerHTML = '';
            container.appendChild(element);
        })
        const DOMSource = {
            selectEvents: function(tagName, eventType) {
                return Observable.fromEvent(document, eventType)
                    .filter(e => e.target.tagName === tagName.toUpperCase());
            }
        };
        return DOMSource
    }
```

這樣的寫法讓我們必須要在 DOM 上一定要有 id 為 app 的 element，

才能夠啟用 DOMDriver。

DOMDriver 是一個 function，

所以我們只要能回傳一個「客製化」的 function，

這件事情不就解決了嗎？

這裏運用到了 JavaScript 中「閉包(Closure)」的概念，



```js
function makeDOMDriver(mountSelector) {
    return (obj$) => {
        obj$.subscribe(obj => {
            const container = document.querySelector(mountSelector);
            const element = createElement(obj)
            container.innerHTML = '';
            container.appendChild(element);
        })
        const DOMSource = {
            selectEvents: function(tagName, eventType) {
                return Observable.fromEvent(document, eventType)
                    .filter(e => e.target.tagName === tagName.toUpperCase());
            }
        };
        return DOMSource
    }
}
const drivers = {
    DOM: makeDOMDriver('#app'),
    Log: consoleLogDriver,
}
```

下一個問題則是：

```js
container.innerHTML = '';
```
假如要 bind 到 DOM 上面是一個很大的 object，

那我們會遭遇到效能的問題。

再來則是 `selectEvents` 這個 function：

```js
selectEvents: function(tagName, eventType) {
    ...
}
```

它只能指定 tagName，

不能用更方便的 selector 來選取想要的 element，

我們應該要提供一個更聰明一點的 API 來做這件事情。

關於這兩個問題點該怎麼重構，

作者並沒有詳細說明，但我們可以直接去看 source code，

這也是我們要將 CycleDOM import 進來的時候。

> 小記一下，

> 假如我們繼續用舊有版本的 run，

> 那 `selectEvents` 會沒有被綁進去 source 裡面。

> 蠻好玩的，可以想一想要怎麼解這一個問題。

接下來的正式引進 cycle-dom 中的 makeDOMDriver，

而原本的程式碼也要跟著做變動。

沒有意外的，首先需要更動的就是 selectEvents 

```js
const mouseover$ = sources.DOM.select('span').events('mouseover');
```

這底下有一個 virtual dom來 handle 重繪，

不會像我們先前一樣，每次一有更動，

就重新 flush 整個畫面。

而 h1, h 也變得更加強大，可以試試看在第一個參數傳入物件，

可以自訂 attributes，以及調整 style。

## Hello Wolrd

啊！終於要開始 Hello world 了，

跟以往不一樣的是我們已經跑了一次的底下大概會發生什麼事情，

才跟世界說 hello。

```js
function main(sources) {
    // return a sinks
    return {
        DOM: Rx.Observable.of(
            div([
                label('Name:'),
                input('.field', {
                    type: "text"
                }),
                hr(),
                h1('Hello !')
            ]))
    }
}
```

現在我們用剛剛學到的 select跟 events 來處理一下 input 的 events。

注意到我們在 input function 那裏的第一個參數寫下 `.filed`，

會自動變成帶有 field class 的 input 。

（準確一點來說應該是 return 一個 virtual dom 的 element）

長這樣：

```js
{tagName: "INPUT", properties: Object, children: Array[0], key: undefined, namespace: null…}
```

再來則是把 input event 以及 值給拿出來：

```js
const inputEv$ = sources.DOM.select('.field').events('input'); 
const name$ = inputEv$.map(ev => ev.target.value);
```

再來要做的事情很直觀，

就是把 name$ 裏的值給 map 到 DOM 上面去......嗎？

```js

name$.map(name => 
        div([
            label('Name:'),
            input('.field', {
                type: "text"
            }),
            hr(),
            h1('Hello !')
        ]))
```

實際上這樣的作法會讓畫面上什麼都沒有，

因為 name$ 是 inputEv$ map 過後的結果，

而一開始 inputEv$ 是空的，自然沒有任何東西會 return 啦！

但要解決這個問題也很簡單，只需要`startWith`這個好用的 operator 即可。

```js
function main(sources) {
    const inputEv$ = sources.DOM.select('.field').events('input');
    const name$ = inputEv$
        .map(ev => ev.target.value)
        .startWith('World');
    // return a sinks
    return {
        DOM: name$.map(name =>
            div([
                label('Name:'),
                input('.field', {
                    type: "text"
                }),
                hr(),
                h1(`Hello ${name}!`)
            ]))
    }
}

```

Hello world 完成啦！

## Counter

在開始之前得提醒一下，

跟 Redux 在開發之前得先想好 StateTree 的道理有點像，

在 Cycle 中，我們會體會到要怎樣設計一個 Stream 的流向，

而 UI 只要跟著這個 Flow 去變化就行了

（狀態顯示為 Reactive 狂粉）

> 來個經典的 Counter example 。

廢話不多說，

就先把頁面和 increment 以及 decrement 的 click stream 弄出來：


```js
function main(sources) {
    const decrementClick$ = sources.DOM.select('#decrement').events('click');
    const incrementClick$=sources.DOM.select('#increment').events('click');
    return {
        DOM: Rx.Observable.of(
            div([
                button('#decrement', 'Decrement'),
                button('#increment', 'Increment'),
                p([
                    label('0')
                    ])
                ])
            )
    }
}

```

拿到 Stream 之後呢？

```js
const decrementAction$ = decrementClick$.map(ev => -1);
    const incrementAction$ = incrementClick$.map(ev => 1);
    const number$ = Rx.Observable.of(0)
        .merge(decrementAction$)
        .merge(incrementAction$);
```

這裏並沒有得到我們想要的東西，

來看一下 merge stream 是怎樣運作的，

```
0---------------- number$
--(-1)-(-1)------ decrementAction$
-------------1---incrementAction$
    [merge]
0-(-1)-(-1)--1---[merged$]
```

我們必須有個東西把 Stream 上所有的值給加總，

想到 array 的 reduce 了嗎？

其實 Rx 有提供一個 Operator 給我們做類似的操作：

它叫做 `scan`。

```js
const number$ = Rx.Observable.of(0)
        .merge(decrementAction$)
        .merge(incrementAction$)
        .scan((prev, cur) => prev+cur);
```

Cycle 強迫我們在一開始就想好資料的流向，

以及事件的處理，如此我們在開發的時候能夠更深思熟慮一點，

不會讓整個 Project 變得很 crazy。

在簡單的 Counter 下這好處還不明顯，我目前也沒用 Cycle 寫過大型的產品，

所以且讓我們繼續看下去。

## Cycle Http Driver

開發 web，我們當然會需要送 http request，

所以我們就需要 http driver。

這裏我們要從 github 的 api 來拿 users 資料。

一樣先把基本的頁面弄出來

```js
function main(sources) {
    return {
        DOM: Rx.Observable.of(
            div([
                button('.get_first', ['Get first user']),
                div('.user_details', [
                    h1('.user_name', '(name)'),
                    h4('.email', '(email)'),
                    a('.web', {href: 'google.com'},'(url)')
                    ])
                ])
            )
    }
}
```

我們想讓使用者點下 get_first 的按鈕後，

就拿到 user 的資料。

前面有提到什麼是 read effect 跟 write effect，

effect 會因應 logics 規則的變化，真正影響到外在世界。

實際講起來太抽象了，我們現在把這個 App 中會發生的 effect 以及分類列出來，

會清楚很多：

```
DOM Read effect : button clicked
HTTP Write effect: send request
HTTP Read effect: receive response
DOM Write effect: user's data displayed
```



```js
function main(sources) {
    // DOM Read effect : button clicked
    const clickEv$ = sources.DOM
        .select('.get_user').events('click');
    // HTTP Write effect: send request
    const request$ = clickEv$.map(_ => {

        return {
            url: API_URL,
            method: 'GET',
        }
    })
    // HTTP Read effect: receive response
    const response$$ = sources.HTTP
        .filter(response$ => response$.request.url === API_URL)
    const response$ = response$$.switch();
    const firstUser$ = response$.map(res => res.body)
    .startWith({});

    // DOM Write effect: user's data displayed
    return {
        DOM: firstUser$.map(user =>
            div([
                button('.get_user', ['Get user']),
                div('.user_details', [
                    h1('.user_name', user.name),
                    h4('.email', user.email),
                    a('.web', {
                        href: user.url
                    }, user.url)
                ])
            ])
        ),
        HTTP: request$
    }
}
```

## BMI


```js
function main(sources) {

    const changeWeight$ = sources.select('.weight').events('input')
        .map(ev => ev.target.value);
    const changeHeight$ = sources.select('.height').events('input')
        .map(ev => ev.target.value);
    // Need to combine two $,
    // Like we use `zip` to arrays.
    const state$ = Rx.Observable.combineLatest(
        changeWeight$,
        changeHeight$, (weight, height) => {
            const heightM = height/100;
            const bmi = Math.round(weight / (heightM * heightM));
            return {
                bmi, weight, height
            }
        })
    return {
        DOM: state$.map(state =>
            div([
                div([
                    label(`Weight: ${state.weight}kg`),
                    input('.weight', {
                        type: 'range',
                        min: 40,
                        max: 150,
                        value: state.weight
                    })
                ]),
                div([
                    label(`Height: ${state.height}cm`),
                    input('.height', {
                        type: 'range',
                        min: 140,
                        max: 250,
                        value: state.height
                    })

                ]),
                h1(`BMI is ${state.bmi}`)
            ])
        )
    }
}
```

在處理 Stream 時，往 Collection 的方向想會舒服很多，

因為我們處理 Array 也是如此，

最後一篇我們將會來看看 Cycle.js 怎樣提高我們程式碼的複用性，

學習用另一種方式去思考該怎樣拆解每個 Component。



----

# 參考資料

- [閉包](http://openhome.cc/Gossip/JavaScript/Closure.html)

- [Master the JavaScript Interview: What is a Closure?](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36#.arfskyb6g)

