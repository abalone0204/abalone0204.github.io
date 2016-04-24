{{{
"title": "做中學 Cycle.js（下）",
"date": "2016/2/1",
"intro": "Model-View-Intent & Component",
"tags": ["cycle.js", "observable"]
}}}

# Intro

我們會希望寫出來的 code 能夠做成被複用的 Component，

不過首先要來拆解一下越來越肥大的 main function。

而 main 就可以被拆成 Model、View 、Intent。

# Model View Intent

先看一下上次 BMI example 的 main function


```js
function main(sources) {
    const changeWeight$ = sources.DOM.select('.weight').events('input')
        .map(ev => ev.target.value).startWith(70);
    const changeHeight$ = sources.DOM.select('.height').events('input')
        .map(ev => ev.target.value).startWith(170);
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



這麼大一包看起來絕對不是好事。

所以我們會把 main 分成三塊，

分別是 Model, Intent, View

- Intent: to listen to the user

- Model: to process information 

- View: to output back to the user

## Intent

第一塊是「Intent」，

簡單說就是 User 想對 UI 做什麼事情的 Intent，

在這裡當然就是指雙方互動的部分：


```js
// input event 就是這個簡單 app 中 User 跟 UI 互動的部分
const changeWeight$ = sources.DOM.select('.weight').events('input')
        .map(ev => ev.target.value).startWith(70);
const changeHeight$ = sources.DOM.select('.height').events('input')
        .map(ev => ev.target.value).startWith(170);

```

```js
function intent (DOMSource) {
    const changeWeight$ = DOMSource.select('.weight').events('input')
        .map(ev => ev.target.value).startWith(70);
    const changeHeight$ = DOMSource.select('.height').events('input')
        .map(ev => ev.target.value).startWith(170);
    return {changeWeight$,changeHeight$};
}
```

## Model

model 則是處理資料流的部分：

```js
function model(changeWeight$, changeHeight$) {
    const state$ = Rx.Observable.combineLatest(
        changeWeight$,
        changeHeight$, (weight, height) => {
            const heightM = height/100;
            const bmi = Math.round(weight / (heightM * heightM));
            return {
                bmi, weight, height
            }
        })
    return state$;
}
```
## View

這裏則是依照 Model 中的資料去建 Virtual DOM tree

> 我們不會把最後要 return 給 Driver 的東西也放在這

> 僅放跟 UI 生成相關的而已

```js
function view(state$) {
    const vtree$ = state$.map(state =>
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
    return vtree$;
}
```

## MVI

然後我們的 main 變得簡潔許多，

看起來只是 function 組合起來而已:


```js
function main(sources) {
    const {changeWeight$,changeHeight$} = intent(sources.DOM);
    const state$ = model(changeWeight$, changeHeight$);
    const vtree$ = view(state$);
    return {
        DOM: vtree$
    }
}
```

## Component

那我們該如何減少重複的 Code 呢？

當 UI 的操作越變越複雜以後，

我們不會希望所有事情都能在一個 main 裡面解決，

這時候我們可以把重複的部分抽出來變成 component。

egghead 課程裡面有更精簡的怎麼把 main 提煉成 component 的過程，

不過核心精神蠻簡單的，就是 **props 也是 stream。**

因為 props 是會跟著傳下來的「資料」，

所以很自然的我們就會選擇處理資料的 model 下手。

而 model 收到的 sources 是從 Drivers 來的，

第一步就是先更動 drivers

```js
const drivers = {
    DOM: makeDOMDriver('#app'),
    props: () => Rx.Observable.of({
        label: 'Height',
        unit: 'cm',
        min: 100,
        max: 220,
        init: 170
    })
}
```

再來就是把 props 傳進去：

```js
const state$ = model(upcomingValue$, sources.props);
```

> 記住： props 也是 Observable

```js
function model(upcomingValue$, props$) {
    const initValue$ = props$.map(props => props.init).first();
    const value$ = initValue$.concat(upcomingValue$);
    const state$ = Rx.Observable.combineLatest(value$, props$, 
        (value, props) => {
            return {
                label: props.label,
                unit: props.unit,
                min: props.min,
                max: props.max,
                value: value
            }
        })
    return state$
}
```

initial value 的 stream concat 新進來 value 的 stream，

取代原本的 `startWith`

下一步就是把 label 的名字和單位給 return 出來，

變成一條 UI component 可以吃到的 state stream，

再把對應的值塞進 view 裡面，就能得到我們想要的 vtree$ 了。

## Using component with Main function

我們現在每個 component 中都會有個 main function，

事實上我們能把 main 改成這個 component 的名字，

並且在更上層的 main 中去使用它，

因為事實上他就是一個 function，在 functional programming 中，

"composable" 可以說是最重要的概念之一。

```js
function LabelSlider(sources) {
    const upcomingValue$ = intent(sources.DOM);
    const state$ = model(upcomingValue$, sources.props);
    const vtree$ = view(state$);
    return {
        DOM: vtree$,

    }
}

function main (sources) {
    return LabelSlider(sources)
}

```

而事實上，我們可以把 props 這件事移到 main 中去做

```js
function main (sources) {
    const props$ = Rx.Observable.of({
        label: 'Height',
        unit: 'cm',
        min: 100,
        max: 220,
        init: 170
    })
    return LabelSlider({DOM: sources.DOM, props: props$})
}
const drivers = {
    DOM: makeDOMDriver('#app'),
}
```

## Multiple Components

如果只有ㄧ個 component 的話，那 cycle.js 也太慘，

我們當然是可以組合多個 components，

只是該怎麼做呢？

很簡單，先把 sinks 個別抽出來：

```js
function main (sources) {
    const weightProps$ = Rx.Observable.of({
        label: 'Weight',
        unit: 'kg',
        min: 30,
        max: 220,
        init: 70
    })
    const weightSinks$ = LabelSlider({DOM: sources.DOM, props: weightProps$})
    
    const heightProps$ = Rx.Observable.of({
        label: 'Height',
        unit: 'cm',
        min: 100,
        max: 220,
        init: 170
    });
    const heightSinks$ = LabelSlider({DOM: sources.DOM, props: heightProps$})

    const vtree$ = Rx.Observable.combineLatest(weightSinks$.DOM, heightSinks$.DOM, 
        (weightVtree, heightVtree) => 
        div([
            weightVtree,
            heightVtree
            ]))
    return {
        DOM: vtree$
    }
}
```

這裏會發現一個問題，就是當我們移動其中一個 slider 時，

另一個也會被影響 ，使用者的互動 => intent

因為兩個的 class 都是 slider，

而 intent 中監聽的又是 ".slider" 底下的 input。

其實我們在 LabelSlider 裡就可以讓兩條 stream 分流，

因為我們傳進去的 `sources.DOM`，是可以只要選取 weight 或 height 就好：

```js
const weightSinks$ = LabelSlider({
        DOM: sources.DOM.select('.weight'),
        props: weightProps$
})
```

這裏做的事情就等於在 intent 裡面這樣：

```js
function intent(DOMSource) {
    const change$ = DOMSource.select('.weight').select('.slider').events('input')
        .map(ev => ev.target.value);
    return change$;
}
```

我們 **pre-select** 了在 DOM 上面 class name 為 '.weight'的 stream。

# Isolate component

- [Isolate](https://github.com/cyclejs/isolate)

- 要隔離開每個 Component 如果都像上面那樣做應該會瘋掉，
  所以 Cyclejs 其實提供給我們一個 helper function： isolate

- 使用方法是傳入一個 Component function 當作 argument
  再來會回傳一個 scoped 的 component function，
  同樣吃 sources 進去，吐 sinks 出來

- `isolate(dataflowComponent, scope)`：第二個參數是 optional 的，如同看到的一樣

> 可能會有人覺得沒什麼差別，但如果單純使用 `isolate(dataflowComponent)`，

> 那會是一個不純的 function ，因為每次呼叫都會 return 一個不一樣的 scoped component function

> 但如果我們指定了 scope，那每次回來的就是同一個 scope 下的 component function

> 真正的濃醇香！


```js
const WeightSlider = isolate(LabelSlider, 'weight');
const weightSinks$ = WeightSlider({
    DOM: sources.DOM,
    props: weightProps$
});
const weightVtree$ = weightSinks$.DOM;
```

如此一來又減少了一些 boiler plate

## Final BMI

目前缺的就是把 bmi 給算出來了，

首先我們知道這個運算會放在 main 裡面，

因為這就是這個簡單小 App 的主要**邏輯**。

```js
const bmi$ = Rx.Observable.combineLatest(weightValue$, heightValue$, (weight, height) => {
    const heightMeters = height * 0.01;
    const bmi = Math.round(weight/(heightMeters*heightMeters))
    return bmi;
});
```

現在問題來了：我們要怎樣得到 weightValue$ 以及 heightValue$ 呢？

從 sources 拿啊！

概念很簡單，我們從 main 中拿到的 source，

其實就是從前一層 component 中吐出來的 sinks，

所以我們自然從前一層 component 中回傳的 sinks 下手：

```js
function LabelSlider(sources) {
    const upcomingValue$ = intent(sources.DOM);
    const state$ = model(upcomingValue$, sources.props);
    const vtree$ = view(state$);
    return {
        DOM: vtree$,
        value: state$.map(state=> state.value)
    }
}
```

實作起來也是這麼簡單。

最後我們回到 main 中，

把 bmi$ 也加進去就成啦！

```js
const vtree$ = Rx.Observable.combineLatest(bmi$, weightVtree$, heightVtree$, (bmi, weightVtree, heightVtree) =>
        div([
            weightVtree,
            heightVtree,
            h1(`BMI is: ${bmi}`)
        ]))
    return {
        DOM: vtree$
}
```

## Conclusion

總計 21 回的課程算不上太長，很推薦有興趣的人去把它看完，

儘管實際上要弄懂 Cycle.js 的概念的確需要花點時間，

但學習 FRP 是值得的，畢竟我們就是在處理 dataflow + UI，

再加上 pure function 好測試、composable 的特性，

不由得感慨 Rx 寫起來真是爽。

相較於 React，Cycle.js 當然更接近 functinoal programming，

不論這個東西將來會不會用到產品上，

純函數式的東西總會莫名的吸引我。

> 如果要追求 fp，更應該要感受一下 [elm](http://elm-lang.org/)

這一堂課的影片幾乎都在 jsfiddle 上完成，

（不曉得作者為啥要這樣XD）

我中間練習的程式碼有放在 [github](https://github.com/abalone0204/Learning-Cycle.js-By-Building-it) 上面，

筆記等年假再來好好整理一番。

----

# 參考資料

- [Official doc: Model View Intent](http://cycle.js.org/model-view-intent.html)

- [How Functional Reactive Programming (FRP) is Changing the Face of Web Development](http://www.codemag.com/Article/1601071)

