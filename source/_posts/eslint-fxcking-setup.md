---
title: 淺入淺出 eslint 與實作
date: 2016-07-11 11:47:04
tags: [eslint, webpack]
---

相信團隊為了提升程式碼的品質，

第一步通常會是制定 style guide，

但 style guide 越定越複雜後，要靠人工去檢查就顯得有點不切實際。

這時候就需要靠程式自動來做語法上的檢查及 highlight，

> 更殘酷一點可以讓不符合 coding style 的 code 無法被 commit

這就是 linter 的功用。

而這篇會以我最近在實務上以 eslint + webpack + githook 來做舉例。

我知道網路上有許多充滿獨到經驗的 javascript linter，最有名應該就是 airbnb 的），

直接拿來用當然也 ok，不過，我們固然要學工具，

工具背後的想法才是我們更該了解的。

這一篇並不是什麼懶人教學，複製貼上就能用的 eslint extends，

而是一步一步地去理解 eslint 到底能做到什麼事情，

也許你看完以後還是會選擇直接使用 airbnb 或是其他人寫好的 linter，

但這時候的你，

已經完全有能力參考前人經驗並制定出一套符合你們團隊需求的 linter，

甚至去看他們的設定時，能夠對他們為什麼這樣做更有想法。

希望你看完之後，學會的並不是 eslint 這個工具而已，

而是未來你要做類似東西時內心已經有一個架構在。

雖然站在巨人肩膀上能夠看的更遠，

但能夠自己造出一個鋼彈再站上去，那他媽完全是不一樣帥氣的事情。

<!-- more -->

# Catalogue

- [為什麼之前不用 linter？](#為什麼之前不用-linter？)

- [Intro & Philosophy](#Intro-amp-Philosophy)

- [Configuration of ESLint](#Configuration-of-ESLint)

    - [What can I configure?](#What-can-I-configure)

    - [Parser Options & Parser](#Parser-Options-amp-Parser)

    - [Environments](#Environments)

    - [Globals](#Globals)

    - [Plugins](#Plugins)

    - [Rules](#Rules)

    - [Extends](#Extends)

    - [Others](#Others)

- [Practical Usage](#Practical-Usage)

    - [Integration with webpack](#Integration-with-webpack)

    - [Integration with git hooks](#Integration-with-git-hooks)

    - [Other solution](#Other-solution)

- [Conclusion](#Conclusion)

----

# 為什麼之前不用 linter？

我得承認在寫這篇文章前，

我並沒有使用 linter 的習慣，

因為說真的，在專案長到一定大小前，

linter 更像是 nice to have 而不是 must have 的東西，

儘管我們知道越早用它越好⋯⋯

大家可能在國文課本都看過方孝儒的指喻，

其實髒髒的 code 這件事就像指喻一樣「始以為不足治，而終至於不可為。」。

![finter](http://i.imgur.com/yTf6igd.jpg)

之前不去使用 `eslint` 的藉口都是沒空仔細研究，

的確，現實生活中的時程可能不允許你直接花大把時間在 linter 上，

所以這篇文章是從幾個禮拜的零碎時間中擠出來的。

# Intro & Philosophy

首先要理解的是：linter 做的事情其實是「靜態的語法分析」。

這意味著我們不需要去執行 script，就能標記出不符合 coding styles 的地方。

另外，

`eslint`的所有規則(rule)都是 `pluggable` 的，

沒有什麼東西是「太重要」而不能把它關掉，

包括你去下載別人的 eslint 設定，

你也可以把不適合你團隊的 rule 給關掉。

最後，

`eslint` 的 rules 是 "agenda free"，

官方並沒有提倡哪種 coding style 是好的，

你想怎麼樣組合你的 rules 就怎麼樣做。

# Configuration of ESLint

## What can I configure?

官方文件中有介紹許多種 config 的方式，

可以從 command line、package.json 裡面設定，

但其實最常看到的還是從 `.eslintrc`去設定，

所以這篇也會以 `.eslintrc`，並且以 json 格式為主。

在寫 config 之前，你要先了解你可以對什麼東西設定 config，

其實只有 3 + 1 個東西而已：

- **Environments**
    
    - 設定環境，不同的環境中會有不同的全域變數（global variable），eslint 中有提供各種不一樣的 enviroments，很少需要自己寫一個。

- **Globals** 
    
    - 前面有提到過，linter 做的是「靜態」的語法分析，所以它對你程式的運行環境是一無所知的，你必須自己把一些全域變數給加上去。

    - e.q: 開發 chrome extension 時，你要 call chrome 的 API 必須從 `chrome` 這個 global variable。

- **Rules**
    
    - 規則 XD 就是你 style guide 的規定，除了指定哪些規範要遵守之外，你也可以去決定違反的 error level。

> 有點像是違反這條規則的嚴重程度，

> 有些比較輕微的你可以設定噴 warning 嚇嚇他就好，

> 但有一些你覺得寫出這些 code 來真是天理難容，你可以直接拿 error 噴死他。

- **Parser**

    - 這裡我把它放在多出來的 1，因為我們通常指定完 parser 之後就不會再其上面更改太多設定，甚至根本不需要指定 XD

    - 的確，你有可能這輩子都不會寫 parser，但我相信探究技術的本質是一個技術人該有的初心，軟體工程師對於知識不該有太浮躁的心 ：）

    - 總之知其然而知其所以然是相當重要的，不管你有沒有修過編譯器（compiler），接下來會馬上科普地介紹 parser 是什麼，以及我們知道這些之後可以做什麼

## Parser Options & Parser

這一小節會解釋如果你要啟用 jsx, es6 或 es7 語法你該做些什麼。

但要先解釋一下 Parser 是什麼？

> 如果你已經知道 Parser 在做什麼，

> 可以直接跳過分隔線中間的這一小段科普文


----

我們都知道電腦看不懂我們寫的 code：

```js
var a = 1 + 2
```

Parser 會把我們的程式碼 parse 成 AST(Abstract Syntax Tree)，

讓我們的程式碼能夠簡單的去操作這個 tree，

最後才會編譯成 binary 的形式。

延續上面的例子，

這段程式碼經過 Espree 這個 parser，「最終」可能會變成：

```js
{
    type: 'VariableDeclarator',
    id: {
        type: 'Identifier',
        name: 'a'
    },
    init: {
        type: 'BinaryExpression',
        left: {
            type: 'Literal',
            value: 1,
        },
        operator: '+',
        right: {
            type: 'Literal',
            value: 2,
        }
    }
}
```

> 為了簡單一點，我把它在檔案的位置標記給移掉了，

但整體而言你可以感受一下，比起直接操作純文字，

轉成 AST 後能用更結構化的方式來取用程式碼。

科普就到此為止了，

想對 Parser 有更深入了解，可以參考一下我之前寫的筆記：

- [Super tiny compiler](http://abalone0204.github.io/2016/04/25/Super-tiny-compiler/)

下面參考文章部分也有放一些我當初學習時讀的文章。

一言以蔽之， 

** Parser 就是將我們對語法的理解給「程式化」成一個樹狀的結構。**

----

前面有說過， Parser 會將純文本的 code 轉成 AST，

`eslint` 中的 parser 只有預設支援 es5 語法，

所以其他額外的語法：es6, es7, jsx，都必須要另外設定。

> Note: 

> **支援 jsx 的 parser 不代表支援 React 的語法。**

> 如果你想要直接使用 React 語法的話可以安裝 `eslint-plugin-react`

接著就直接來看要怎樣設置 Parser 的 options，

在 `.eslintrc.json`中的 `parserOptions`去設置：

```json
{
  "parserOptions": {
    "ecmaVersion": 6,
    "sourceType": "module",
    "ecmaFeatures": {
      "jsx": true,
      "impliedStrict": true
    }
  },
  "rules": {
    "semi": 2
  }
}
```

- `ecmaVersion`: 顧名思義就是 ECMA script 的 version，有 3, 5, 6, 7 可以供你挑選

- `ecmaFeatures`
    
    - 啟用 jsx、strict mode 等 feature，預設都是關閉，可以用 boolean 的方式來開啟。

    - 詳情可以到[官網文件](http://eslint.org/docs/user-guide/configuring)看

- `sourceType`: 

    - 預設是 script，但如果你的 code 是被包在 ECMA script 的 module 中，就要設成 module

    - 白話文：你用 webpack 的話就把它設成 module 

最後，雖然 eslint 預設的 parser 是 `Espree`，但你還是可以換成其他的 parser：

- Esprima, Babel-ESLint

> parserOptions 是共通的，在以上的 parser 裡面不用擔心要寫不一樣的 parserOptions


## Environments

可以在不同 environment 去 predefine 所需的 global variables。

> 注意：

> 這裡並不是去 assign global variables，

> 而是把不同 environments 的選項給打開，

> 接著你就會取到在這個 env 底下 predefine 的 global variables 了

一樣用 `.eslintrc`舉例：

```json
{
    "env": {
        "browser": true,
        "node": true
    }
}
```

可以到官方文件看有哪些 environments 可以用：

- [官方文件](http://eslint.org/docs/user-guide/configuring#specifying-environments)

假如你想要自訂或是用別人 plugin 中的 env 也很簡單：

```json
{
    "plugins": ["example"],
    "env": {
        "example/custom": true
    }
}
```

## Globals

定義你需要的 global variables，

這裡並不是像平常寫 code 時在 assign 值給 variable 一樣，

因為我們並不會去執行程式碼，只會進行靜態的分析，

所以我們做的事情只是確認這個 global variable 是有被 define 的，

舉例來說：

```js
const a = globalB
```

上述的 `globalB` 就是未被 define 的 global variable。

假如我們定義了不能接受未 define variable 的規則(rule)，

linter 就會把這個視為語法檢查不通過。

所以我們必須要讓 eslint 知道這個全域變數是有被 define 過的：

```json
{
    "globals": {
        "globalB": true
    }
}
```

如此一來 parser 在看到 `globalB` 時就會知道：

「啊！這不就是寫在 globals 裡面的 globalB 嗎？沒事兒沒事兒」

## Plugins

eslint 能夠很靈活的安裝第三方的 plugin。

這裡就是我們平常在使用 airbnb 的 config 時會用到的地方 XD，

通常名字會長這樣：`eslint-plugin-*`，你可以省略掉前面這一段 prefix，

比如說`eslint-plugin-demo`，在 `.eslintrc` 可以這樣引入：

```json
{
    "plugins": [
        "demo"
    ]
}
```

待會提到 rule 時，再來解釋要怎樣引用 plugin 裡的東西。

## Rules

rule 就是我們在 style guide 中定義的規則，

可以針對嚴重程度設定 error level：

分別是 `off`, `warn`, `error`，

> 它們分別對應到 0, 1, 2 三個數字，

> 也就是說 `{"curly": "error"}` 和 `{"curly": 2}` 是一樣的意思。

看例子可能會清楚一點：

```json
{
    "plugins": [
        "plugin1"
    ],
    "rules": {
        "eqeqeq": "off",
        "curly": "error",
        "plugin1/rule1": "error"
    }
}
```

這裡的 rule 都是 eslint 事先定義好的規則，

想看有哪些的話一樣可以到官方文件去看，

不過我想應該是不會有人一條條看完就是：[eslint: rules](http://eslint.org/docs/rules/)

同樣的，我們可以藉由 `<pluginName>/rule` 來獲取 plugin 底下定義的 rule。

補充一下，你也可以在一些特別的時候 enable 或 disable rule。

舉例來說有一條規則是 code 裡面不要有任何 `console.log`，

但有些地方一定要存在 `console.log` 該怎麼辦呢？

你可以讓 eslint ignore 掉這整個 file，不過這不是個好解法。

更好的做法應該是在那幾行 code 前面加上 "inline disable" 的 comment：

```js
server.listen(8080, '0.0.0.0', (err) => {
  /* eslint-disable no-console */
  if (err) {
    console.log(err);
  }
  console.log('Listening at http://0.0.0.0:8080/');
  /* eslint-enable no-console */
});
```

`/* eslint-disable no-console */`底下的 code 都會關閉 `no-console`這個規則，

但`/* eslint-enable no-console */`會把 `no-console` 這個規則再次打開。

兩個搭配起來的結果就是在這兩段 comment 中間的 code 不會啟用 `no-console` 這個規則。

## Extends

總結上述幾點，其實 extends 這個 array 裡面放的其實是個完整的 config，

你也可以直接 extend 別人 export 出來的 config。

這也是為什麼當你安裝別人的 extends 時，

會以 `eslint-config-*`來當作 package 名稱，

而不是用 extension 的原因。

> 同樣的，在寫 config 時，可以忽略 `eslint-config` 這個 prefix

```
{
    "extends": "airbnb"
}
```

## Others

- 跟 git 一樣，可以忽略掉某些檔案：`.eslintignore`

- 前面的 rules 看起來都是別人幫你建好的，你也可以參照 [working with rules](http://eslint.org/docs/developer-guide/working-with-rules) 來定義你自己的 rule。

> 對我而言現有的 rules 幾乎已經把我能想到的模組都開發完，

> 理解 linter 對我來說最重要的是知道「哪些東西是我要的」，

> 最後再將其組裝起來，而不是從頭造一遍輪子。

- 同理，你也可以自己開發 plugin。

- 你可以 extend 其他人的 config，但你也可以在最外層去把 rule 給覆寫掉。


# Practical Usage

如果寫完 eslint 設定，卻還要每次寫完 code 都自己跑一次：

```
$> eslint file.js
```

這是相當反人性的事情，所以在有了前述的背景知識後，

我們來看看日常開發中是如何使用的，

以下就兩個比較常見的方法來介紹，沒有誰好誰壞，

全看怎樣比較適合你的團隊。

我盡量不為讀的人預設什麼預備知識，

但懂一些 git 以及 webpack 的話，

設定起來會相當的 trivial：

- Webpack
    
    - 邊開發時就邊檢測你是否有違反規則

- git hook

    - 在 commit 前檢查是否符合規則

其實編輯器也有許多可以搭配 linter 的東西，

但鑑於所有開發者使用的編輯器種類太多，

而且編輯器的設置相對簡單，

所以這裡不會再贅述編輯器上面的設定。

## Integration with webpack

首先要先安裝 `eslint-loader`

```
$> npm install -D eslint-loader
```

接著到 `webpack.config.js` ，只要看 loaders 這個屬性就好了:

```js
{
    module: {
        loaders: [
            {
                test: /\.js[x]?$/,
                exclude: /node_modules/,
                loaders: ['babel', 'eslint']
            }
        ]
    }
}
```

loaders array 中的順序是相當重要的，因為它的順序是從最後面開始往前執行。

也就是說在讀取到 js 或是 jsx 檔案時，

會先經過 eslint 檢查，再進去 babel 轉譯。

> 反之的話，一定會噴錯噴得滿天飛 XD 

> 而且這樣 eslint 去檢查的就是轉譯過後的程式碼了

但更好的方式是把 eslint-loader 放在 `preloaders` 中：

> 馬的我也是第一次知道有這東西，webpack 的 config 就是這麼令人驚喜

```js

{
    module: {
        preLoaders: [
            {
                test: /\.js[x]?$/,
                exclude: /node_modules/,
                loader: 'eslint'
            }
        ],
        loaders: [
            {
                test: /\.js[x]?$/,
                exclude: /node_modules/,
                loader: 'babel'
            }
        ]
    }
}
```

如此在開發途中，只要語法有違反規則，

就是視同為 error。

現在你已經有一個自動化的 linter 了。

補充一下，有些人可能會用 webpack 的 provide plugin，

去省掉一些 `../../../../actions/doSomething.js` 的程式碼，

這時候你可能會需要這個 plugin: [eslint-import-resolver-webpack](https://www.npmjs.com/package/eslint-import-resolver-webpack)，

然後你要在 `.eslintrc` 的 settings 裡面加上：

```json
{
  "settings": {
    "import/resolver": {
      "webpack": "webpack.config.js"
    }
  }
}
```

這樣子當你使用 `no-unresolved` 這條規則時，

eslint 會先去檢查你在 provide 裡的 alias，

再來看有沒有辦法 resolve 這個 path。

## Integration with git hooks

> 如果你很熟悉 git 的話甚至可以不用安裝這套件 XD 

git hook 能做的事情就是讓開發者在執行某些 git 操作前，

自動得去執行某些 scripts，

不過首先，我們要先將 lint 的 script 給寫出來才行

接著在 `package.json` 裡面：

```json
{
    "scripts": {
        "lint": "./node_modules/eslint/bin/eslint.js target.js",
    },
}
```

再來只要執行：

```
$> npm run lint
```

就會自動的去做 lint，簡單的 script 完成了，

接下來就是跟 git hook 結合的時候，

會需要用到 `husky` 這個套件：

```
$> npm install husky --save-dev
```

會用它的原因是因為它用起來簡單粗暴，

不管你要用哪個 hook，只要把名字放在 npm script 中：

```js
{
    "scripts": {
        "lint": "./node_modules/eslint/bin/eslint.js demoStore.js",
        "precommit": "npm run lint"
    }
}
```

你的 git hook 就設定完成了

> precommit 就是在 commit 之前我們會去執行 `npm run lint`這個 script，

> 這裡有個對應的 [hooks 表格](https://github.com/typicode/husky/blob/master/HOOKS.md)

> 你可以挑一個在適當的時機執行語法的 lint

## Other solution

不過如果每次 Commit 都要跑一次 lint，讓你很煩的話，

實務上的做法也可以只整合在 IDE 以及 CI server 上就足夠。

假如你還是要保留原本的流程，

但在某些整理 commit 時並不需要重新執行 lint的話，

也可以在 commit 時加上 `--no-verify`或`-n`。

還有一個做法是使用 [`lint-staged`](https://github.com/okonet/lint-staged) 這個套件，

簡單的說，它會讓 linter 只檢查新放上 stage 的 code，

> 在 git 中執行 add 之後，會把 file 放到 stage 上，

> 這就是為什麼他要命名為 `lint-staged`。

這樣每次的 Commit 就不用重新檢查一次全部的程式碼了。

> 快速的想過一遍之後，`lint-staged`可能會有個小 gotcha，

> 可能在 `no-unresolved` 這條 rule 上面犯錯。

> 舉例來說 a 檔案會 require `./b`，

> 然後我把 b 刪掉了，這次 a 檔案並不會上 stage，

> 所以這次檢查並不會把這個錯誤給檢查出來，

> 但整體而言其實還是為我們省了不少時間，

> 就看個人怎麼選擇啦！

# Conclusion

在學習 eslint 的過程中，

對於這種自動化、提升團隊程式碼品質的東西又有了一些心得，

像是 eslint-loader 以及 git hook 的使用時機都是。

同時也理解到這種 pluggable 的特性在開發者的世界裡能帶來極大的成功，

看看 webpack、babel，以及 eslint 都是因為其容易製作 plugin 的特性，

讓開發者自主的開發出優質的插件來讓整個生態系更蓬勃。

也許工具會一代代推陳出新，

但是這種我為人人、人人為我的系統思維是不會變的。

最後，

在前面有說過 coding style 之於指喻，

最近實務上有個算小型的 project 從一開始開發時並沒有去使用 eslint，

到了今天把簡單的 style guide 訂出來之後，（大概就是四條 rules 而已，沒有直接引用 airbnb 那一套）

結果是這個樣子：

![errro](http://i.imgur.com/FFLkTXA.png)

嗯⋯⋯

晚了一點以後更新成 airbnb 的 config 然後自己修改一下 rule，

變成：

![error2](http://i.imgur.com/PYzEmMf.png)

....

![eyes](http://i.imgur.com/2yJBwXp.jpg)

這一定是假的。

# References

- [eslint](http://eslint.org/)

- [Super tiny compiler](http://abalone0204.github.io/2016/04/25/Super-tiny-compiler/)

- [谈谈Parser - 王垠](http://www.jianshu.com/p/35b9fc5467e0)

- [Handwritten Parsers & Lexers in Go - Gopher Academy](https://blog.gopheracademy.com/advent-2014/parsers-lexers/)

- [eslint-loader](https://github.com/MoOx/eslint-loader)

> 感謝 [`@ctwu`](https://github.com/wuct) 、李俊緯對開發流程中整合 eslint 的建議

> 感謝 Amobiz Chen 提供 `lint-staged`這個工具

> 感謝陳威霖提醒我要加上 inline-disable 的用法 XD
