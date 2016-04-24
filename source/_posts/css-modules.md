---
title: "CSS Modules: 模組化 CSS"
intro: 「如果你覺得 CSS 寫起來很亂的話，那代表你心中沒有架構。」
date: 2016/1/9
tags: CSS Modules, PostCSS
---


時至今日，我最討厭的東西就是亂七八糟的 CSS 還有 KMT，

這兩個東西有一點很一致：

> 不管我們再怎麼討厭它，
> 都還是得面對它、處理它。

## 先說結論：

- 如果討厭寫 CSS，就更應該用這種方式來寫

- 我們應該用 module 化的方式來思考每個畫面上的東西

- 讓需要「工人智慧」的地方減到最少

> 「如果你覺得 CSS 很亂的話，那代表你心中沒有架構。」

## Prerequisite

- 會使用 webpack（幾乎只要會改 config 就行了）

- 把 CSS 當一回事的人

----

## 為什麼需要去思考 CSS 的「架構」？

曾幾何時，我也覺得 CSS 是一個他媽有夠亂七八糟的東西，

直到不小心開始寫前端，我才發現前端不只是 JavaScript，

從 CSS 到 html 的設計，都需要仔細去思考「架構」這件事，

否則很容易讓技術債債台高築，到最後一發不可收拾。

使用起來合邏輯的東西，不代表能夠用很「邏輯化」的方式寫出來，

這正是 CSS 為什麼很容易亂七八糟的原因，

因為我們常常需要去指定很多畫面上的細節（imperative）：

>「欸欸，你這邊 width 要 300px，然後 margin 要設成 0 auto 才能置中」

而不是直觀的用程式碼來宣告我們想要畫面長怎樣（declarative）：

>「我們要一個看起來不錯的畫面」

處理太多細節很容易出錯，像是螢幕或視窗大小不一樣 300px 就不一定 ok 了，

而第二個 declarative way 似乎又太過理想化。

而我認為折衷的方式就是 module 化 CSS，

雖然也需要去實作 module 內的細節（imperative），

但完成之後，就可以將這些 module 組裝起來，

重複使用時就不需要去實做那麼多的細節，

沒錯，我們又往 declarative programming更進一步了。

現在看起來還是比較 high level 的概念，

但我認為知道為什麼要這樣做很重要，

稍後會在例子裏看到這樣做的好處是什麼。

----

在開始之前先講解一下兩個會推薦使用的工具，

（你也可以依自己喜歡的配置啦！）

分別是 Autoprefixer 以及 PostCSS。

## Autoprefixer

假如熟悉 postcss 和 autoprefixer 在幹嘛的人可以直接跳下一段了。

其實我們平常在寫 CSS 的時候，為了處理跨瀏覽器的問題，

常常需要寫很噁心的 prefix，

就算有 SASS 的 include 語法，prefix 還是很噁心。

看到 autoprefixer 出現真是讓人痛哭流涕的一件事，

因為這代表以後有人會幫我們處理好 prefix，

同時還會把太舊的 prefix 給移除掉。（像是 `border-radius`）

這裏就直接來安裝進專案吧！

`webpack.config.js`

```js
var autoprefixer = require('autoprefixer');

module.exports = {
    module: {
        loaders: [
            {
                test:   /\.css$/,
                loader: "style-loader!css-loader!postcss-loader"
            }
        ]
    },
    postcss: [ autoprefixer({ browsers: ['last 2 versions'] }) ]
}
```

唯一需要說明一下的就是可以指定我們要 support 到多老舊的 browser啦！

就這樣，恭喜你！

----

## PostCSS

PostCSS 是一個可以用 JavaScript plugins 將 style 轉成我們想要樣子的工具。

（包括 lint, variables, mixins，以及好多東西......）

確切一點來說， PostCSS 是一個 node.js 的 package，

它可以將我們原本的 CSS 檔案轉成 AST(Abstraction Syntax Tree)，

接著我們就可以藉由這個 API 來對 CSS 做事情，

做完後再將它轉成 String，輸出成我們想要的 CSS，

如果你懶得自己寫 plugin 來處理也不用擔心，

現在已經有兩百多個 plugins 在那裡等你愛智求真了。

我知道一定有人這時候在想：「那 SASS 呢？」

沒錯，這兩者看起來似乎有點像，不過可以先看一下這篇文章：

- [I'm Excited About PostCSS But I'm Scared to Leave Sass](http://davidtheclark.com/excited-about-postcss/)

這裏則是值得一看的補充資料，其實官方的 readme 裏也都有寫：

- [tut+的教學](http://webdesign.tutsplus.com/series/postcss-deep-dive--cms-889)

- [一個前端用 node.js 來寫 CSS 的 preprocessor 也是很正常的事](http://nicolasgallagher.com/custom-css-preprocessing/)

簡言之，PostCSS 跟 SASS 或 LESS 最不一樣的點是：

「我們可以只採用我們想要的部分，並將其組裝起來。」

這不就是 Compoasable 和模組化嗎？

接著就來看看如何在 webpack 中設定 postcss，

和使用各種 plugins。

（坦白說這裏才是最頭痛的部分）

使用 webpack 雖然簡單，但 config 的寫法太雜亂了，

完成同樣一件事可以有好幾種方法，

目前連官方文件上也沒有一個一致的 best practice。

而[阮義峰的這篇教學](https://github.com/ruanyf/webpack-demos)是我目前看過寫的最清楚易懂的，

從 entry 到跟 react 一起使用都有說到。

----

## CSS modules

假如你直接跳過前兩個工具，其實也是 ok 啦！

因為 webpack 的 css-loader 本身就內建 module 功能：

```js
{
    module: {
        loaders: [{
            test: /\.[s]?css$/,
            loader: 'style!css?modules!sass'
        }]
}
```

現在終於要來講一下 CSS modules 可以做到什麼事情。

- 組合（Composition）

我們能夠將 selector 組合在一起

```css
.className {
  color: green;
  background: red;
}

.otherClassName {
  composes: className;
  color: yellow;
}
```

這裏要注意的是 composes 必須寫在其他 properties 的前面。

而我們也可以 compose 多個 className：

`composes: classNameA classNameB;`

乍看之下跟 SASS 的 extend 有點像，

但讓我們繼續看下去。

## Dependencies

假設我們現在有另一個檔案: style.css

```css
.className {
    // some style
}
```

```css
.otherClassName {
  composes: className from "./style.css";
}
```

這給了我們很大的彈性，但小心不要 override properties，

我覺得官方文件的這一句話寫得很棒：

> Best if classes do a single thing and dependencies are hierarchic.

這的確是我們在設計 CSS module 時，要常存心中的一句話。

## Usage with preprocessors

這裏主要是說要如何運用 preprocessor ，

因為我們有時候還是需要 global 的 class。

```less
:global {
  .global-class-name {
    color: green;
  }
}
```

----

## Rewrite with CSS Modules

如果你是打從專案一開始就使用 css module ，

那恭喜你！

但「通常」現有的專案上都是用 SASS 來解決，

這裡就以我工作上的專案來做例子。

這裏要提一下我們後端用的是 Rails，

Rails 有個邪惡的好東西叫做 [Asset Pipeline](https://ihower.tw/rails4/assets-pipeline.html)，

它會將靜態資源壓成一個檔案，減少 request 數。

自動幫你做這件事聽起來很美好，

但實際上因為 css 有 global scope 的問題，

所以要怎麼確保每一頁只 load 到自己要的 style 呢？

我的做法是每一頁會有一個專屬的 id，

而命名的方式就是以 controller 加上 action 的名稱來命名。

像是 posts_controller 的首頁，

我就會給它專屬的一支檔案`posts_index.scss`

```css
#posts_index {
    // some style
}
```

這樣做的第一個好處很明顯，

就是每個頁面裡的樣式就只會影響 id 裡的 scope。

那說好的 module 呢？

這裏就要用到 SASS 的 `extend`，

假設 posts 和 show 都有一模一樣的 header，

這時候我就會把 header 抽出來像下面這樣：

```css
%header {
    header {
        //  some style
    }
}
```

```css
import "./header.scss";

#posts_index {
    @extend %header;
    // some style
}
```

```css
import "./header.scss";

#posts_show {
    @extend %header;
    // some style
}
```

看起來挺方便，

而且 Rails 的 routing 通常都是 restful 的，

所以理論上這樣 CSS 的名字也有一定的規則可循，

不會找不到檔案在哪裡。

（就算有自動搜尋，也要知道下哪些關鍵字吧！）

但，

如果今天根據 user 的身份不同，

會 render 不一樣的頁面呢？

`#posts_index_super_user`？

沒錯，問題又變得開始複雜起來，

原因就出在它仍然是 global scope，

而我試圖想從命名來解決這件事情，

我常常在想：「啊！如果 CSS 是 local scope該有多好？」

> A CSS Module is a CSS file in which all class names and animation names are scoped locally by default.

天啊！這解決了根本上的問題！

假如能夠用 component-based 的方式來思考，

讓 react component 從 css module 之間有對應的 name 來讀取樣式，

那不就更棒了嗎？

以後的資料夾結構會長這樣子：


```
├── components
│   ├── ui-App
│   │   ├── index.css
│   │   └── index.js
│   ├── ui-Avatar
│   │   ├── index.css
│   │   └── index.js
│   └── ui-Profile
│       ├── fonts
│       │   └── opensans-regular-webfont.woff
│       ├── images
│       │   └── icon-user.png
│       ├── index.css
│       └── index.js
└── styles
    ├── base.css
    └── theme.css
```

一個資料夾底下就放著 component.js, component.css，

本身就是一個 micro-service，

而我們要做的正是把這些 micro-service 給組裝起來變成一個頁面，

最後再把這些頁面組裝起來變成 Application，相當舒服。

不過要如何從現有的專案改寫呢？

這裏就拿這個小小的部落格來舉例，

因為我一開始是用[自己寫的 generator](https://github.com/abalone0204/generator-suku) 生成專案，

（小打一下廣告，

平常開發前端 component 就是在這個生成的專案上開發，

弄好 react 和 hmr 之後，其實蠻方便的。）

順帶一提，這是開始改寫前的樣子：

```
stylesheets/
    ├── animations
    │   ├── blink.scss
    │   ├── loading.scss
    │   └── spins.scss
    ├── code_highlights
    │   └── default.scss
    ├── colors.scss
    ├── components
    │   ├── Nav
    │   │   └── _icon_bar.css
    │   └── common
    │       └── loading.scss
    ├── nav.scss
    ├── pages
    │   ├── about.scss
    │   ├── home.scss
    │   └── post.scss
    └── style.scss
```

到最後 stylesheets 裡面只會剩下 global 的 css 檔案，

像是 base.css 或是 theme.css 。

首先第一步當然就是處理 global 的 css，

思考的方向很簡單，就是哪些東西是每一個頁面都用得到的呢？

所以我們把 body, a, h1~h5之類的東西先拔出來：

```css
:global {
    a {
        color: inherit;
        text-decoration: none;
    }

    body {
        margin: 0;
        letter-spacing: 1px;
        color: #23263a;
    }

    * {
        font-family: 'Noto Sans TC',Microsoft JhengHei,Microsoft YaHei, LiHei Pro, Heiti TC, sans-serif;
        font-weight: 200;
    }

    .wf-loading {
        * {
            font-family: Microsoft JhengHei, Microsoft YaHei, LiHei Pro, Heiti TC, sans-serif;
        }

        font-family: Microsoft JhengHei, Microsoft YaHei, LiHei Pro, Heiti TC, sans-serif;
    }
}
```

接著來處理我們的 Nav bar，

從這裡開始，就要進入 module 化的思考方式，

一開始的時候你可能會覺得，欸？幹嘛這樣做？

但越到後面你會發現一旦你習慣這樣思考，

很多原本難解的問題都會迎刃而解，

尤其是用組裝的方式來思考畫面的元件，

能讓多狀態的呈現變得更簡單，

也更能明白哪個部分該抽象化出來變成 base。

先來看看這個 Nav 的例子。

----

預計會在以下幾個步驟循序漸進地去思考如何去寫 CSS Modules：

- 讀一下舊有的 js, css

- 最外層的 global selector

- 沒有狀態改變的 local selector

- 有狀態改變的 local selector

### 1. 分析舊有的 js, css

```jsx
class Container extends Component {
    constructor(props) {
        super(props);
        this.state = {show: false};
        this.toggleIcon = this.toggleIcon.bind(this);
    }
    toggleIcon() {
        this.setState({show: !this.state.show})
    }
    render() {
        let {show} = this.state;
        let className = show ? "active" : "";
        return (
            <nav>
                <div id="logo" className={className}/>
                <div id="toggle_icon" 
                     className={className}
                     onClick={this.toggleIcon}
                />
                {
                    show ? 
                    (
                        <ul id="nav_list" className={className}>
                            <li><Link to="/about">About</Link></li>
                            <li><i className="fa fa-github-alt"></i></li>
                            <li><i className="fa fa-facebook"></i></li>
                        </ul>
                    ) :
                    null
                }
            </nav>
            )
    }
}
```

可以看到我們的 toggle_icon 會隨著 show 的值而改變樣式，

至於怎樣改變？就來看看原先架構下的 CSS 怎麼寫。


```css
@import "./colors.scss";
@import "./components/Nav/icon_bar";

nav {
    position: fixed;
    z-index: 5;
    top: 0;
    width: 100%;
    color: white;
    background: $deep_blue;
    padding: 14px;
    height: 28px;

    a {
        color: inherit;
        text-decoration: none;
    }

    #logo {
        height: 28px;
        width: 28px;
        display: inline-block;
        background-image: url("../img/icon.png");
        background-size: cover;
        transition: transform 1s ease;

        &:hover {
            animation: shake;
        }
    }

    #logo.active {
        color: $sudo_green;
    }

    #toggle_icon {
        position: absolute;
        top: 50%;
        transform: translateY(-50%);
        right: 50px;
        display: inline-block;

        @extend %icon_bar;

        cursor: pointer;

        &:before,
        &:after {
            @extend %icon_bar;

            content: '';
            display: block;
            position: absolute;
        }

        &:before {
            margin-top: -10px;
        }

        &:after {
            margin-top: 10px;
        }
    }

    #toggle_icon.active {
        background: transparent;
        transition-property: background-color, transform;
        transition-duration: .2s;

        &:before, &:after {
            background: $sudo_green;
            transition-property: background-color, transform;
            transition-duration: .2s;
        }

        &:before {
            transform: rotate(45deg);
            transform-origin: 0 0;
        }

        &:after {
            transform: rotate(-45deg);
            transform-origin: 0 5px;
        }
    }

    #nav_list {
        position: fixed;
        height: 100vh;
        background: #23263a;
        text-align: center;
        top: 56px;
        left: 0;
        display: block;
        padding: 5px 15px;
        margin: 0;

        li {
            display: block;
            padding: 5px;
        }
    }
}
```

----

## 2. 最外層的 global selector

如果你有寫過 react native 的話，

就能體會到 style object 的好處，

假如沒有，那現在這是好好來玩玩看的時候。

我們從最外層開始拆解。

（其實由內而外、由外而內各有好壞，但這可能又要寫另外一篇了）

最外層的當然就是原生的 nav tag，

這裏其實大可直接給他 global

```css
:global {
    nav {
        position: fixed;
        z-index: 5;
        top: 0;
        width: 100%;
        color: white;
        background: #23263a;
        padding: 14px;
        height: 28px;
    }
}
```

----

## 3. 沒有狀態改變的 local selector

往下看到 logo ：

```css
.logo {
    height: 28px;
    width: 28px;
    display: inline-block;
    background-image: url("../../../static/img/icon.png");
    background-size: cover;
}
```

要怎麼 import 它呢？

首先別忘記在 webpack 的 config 裡開啟 css modules 的功能。

再來只要這樣：

```js
import style from "./Nav.scss";

export default class Nav extends Component {
    render(){
       return ( 
           ...
           <div className={style.logo}/>
           ...
       )
    }
}
```

`style.logo` 讀到的就會是 webpack 幫我們生成的唯一字串，

不用擔心會跟其他 class 重複，不相信的話 console.log 看一下，

而跟以往相同，webpack 也會自動去幫我們寫入 style 到 head 裡面，

對應到的 class name 就是剛剛生成的唯一字串。

原理大概是這樣子。

----

### 4. 有狀態改變的 local selector

再來則是為什麼我仍然使用 SASS 的原因： extend

來看看 toggle_icon，他就是我們平常看到手機版的選單，

按了之後會變形。

先直接看它原本的 CSS 長怎樣：

```css
#toggle_icon {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    right: 50px;
    display: inline-block;
    @extend %icon_bar;
    cursor: pointer;
    &:before,
    &:after {
        @extend %icon_bar;
        content: '';
        display: block;
        position: absolute;
    }
    &:before {
        margin-top: -10px;
    }

    &:after {
        margin-top: 10px;
    }
}
```

> 我知道有一些 PostCSS 的插件可以解決，
> 但這篇的重點在於模組化 CSS 的思考，所以就暫時先擱著啦！）

因為那個 icon 有三個橫條，每個橫條的設定都差不多，

所以我寫了一個 icon_bar 來被 extend。

```css
%icon_bar {
    width: 30px;
    height: 5px;
    transition-property: background-color, transform;
    transition-duration: .2s;
}
```

接著則是重頭戲，

對於畫面來說，這個 toggle_icon 會有兩個狀態，

也就是說我們會有兩個 class 來處理它，

但這兩個狀態又有許多共同點，怎麼辦呢？

答案很簡單：

> 抽出來當 base，讓兩個狀態的 class 去 composes 這個 base 就好啦！

```css
.toggle_icon_base {
    @extend %icon_bar;
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    right: 50px;
    display: inline-block;
    cursor: pointer;
    transition-property: background-color, transform;
    transition-duration: .2s;
    &:before,
    &:after {
        // pseudo-selector 是不能使用 composes 的
        // 這就是為什麼我仍需要 @extend
        @extend %icon_bar; 
        content: '';
        display: block;
        position: absolute;
    }

    &:before {
        margin-top: -10px;
    }

    &:after {
        margin-top: 10px;
    }
}
```

這裏抽出來的就是兩方都不會變的 properties，

把 transition 放在 base 裏的好處就是能看到狀態之間的變化，

這樣能實現一些簡單的動畫。

接著就是把我們寫好的 base 組裝起來而已，

toggle_icon！附身合體！

```css
.toggle_icon {
    composes: toggle_icon_base; // 記得要放在其他 properties 前面
    background-color: white;

    &:before,
    &:after {
        background-color: white;
    }

    &:hover {
        background-color: #50e2c2;

        &:before,
        &:after {
            background-color: #50e2c2;
        }
    }
}

```

狀態的改變每個人都有自己喜好的方式，可以自行調整：

```css
.toggle_icon--active {
    composes: toggle_icon_base;
    background: transparent;
    

    &:before, &:after {
        background: #50e2c2;
        transition-property: background-color, transform;
        transition-duration: .2s;
    }

    &:before {
        transform: rotate(45deg);
        transform-origin: 0 0;
    }

    &:after {
        transform: rotate(-45deg);
        transform-origin: 0 5px;
    }
}
```

而 component 中該如何對應呢？

```jsx
class Nav extends Component {
    render() {
        return (
        ...
        <div className={show ? style["toggle_icon--active"] : style.toggle_icon}
              onClick={this.toggleIcon}
        />
        ...
        );
    }
}
```

沒錯，就是這麼簡單而已。

# 結論

回頭看看重構後的 CSS，

你會發現我們已經不是昔日把所有東西都丟在越來越多層的 class 裡面，

而是變成扁平且一塊一塊的了，

如果要重構的話我們也能夠將重複的部分抽出來。

再來更棒的是除了 global 的地方，

我們不用再擔心全域命名污染的問題，

畢竟沒有 import 到的 class 就永遠不會發生作用啊！

如果有寫錯的地方或是建議，很歡迎留言告訴我。

我真的最討厭寫 CSS 了。

## 參考連結：

- [css module](https://github.com/css-modules/css-modules)

- [autoprefixer](https://github.com/postcss/autoprefixer)

- [Using webpack to build React components and their assets](http://simonsmith.io/using-webpack-to-build-react-components-and-their-assets/)