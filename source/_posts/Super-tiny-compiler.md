---
title: Super tiny compiler
date: 2016-04-25 23:04:37
tags: [compiler, JavaScript]
desc: compiler, javascript
---

造一個超級小又可愛的 Compiler，

沒有組語，只有 JavaScript。

<!--more-->

<img width="731" alt="THE SUPER TINY COMPILER" src="https://cloud.githubusercontent.com/assets/952783/14413766/134c4068-ff39-11e5-996e-9452973299c2.png"/>

# 前言：被玩壞的工具鏈

之前看到 react-motion 的作者說了這句話：

<blockquote class="twitter-tweet" data-lang="zh-tw"><p lang="en" dir="ltr">Lisp: everything&#39;s data<br>Smalltalk: everything&#39;s an object<br>Haskell: everything&#39;s computation<br>JavaScript: everything&#39;s a library</p>&mdash; Cheng Lou (@_chenglou) <a href="https://twitter.com/_chenglou/status/722640025092009986">2016年4月20日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

老實說，不知道從什麼時候開始，

我們前端工程師大多都變成工具的使用者，而不是設計者了，

真正基礎和有價值的事物其實一直都隱藏在我們常使用的工具底下，

像是我們為了使用 es6 語法的 babel 就是一個很好的例子，

好好靜下心來寫一個小小的 lisp compiler，瞭解底下發生了什麼事情，

以及弄懂 compiler 究竟是多麽偉大的想法，

對於一個前端工程師的身心健康都頗有幫助。

> 至少對我來說是這樣啦！

# Compiler

提到 Compiler 總是讓人望而生卻，

在 ember conf 上， James Kyle講的這個 [talk](https://www.youtube.com/watch?v=Tar4WgAfMr4)，

讓我覺得該記錄一下這篇文章，

好好推廣一下 Babel 底下發生了什麼事情。

首先先來談談 Compiler 是在做什麼的，

Compiler 的工作是將「來源代碼」轉成「目標語言」。

除了我們熟知的 gcc 之外，還有 Babel，

沒錯，我們的生活周遭充滿了 compiler，

就算是寫 JavaScript，

如果要使用 es6 以上的語法，

你就必須得用到 Babel 這個 compiler。

將你寫的 code(來源代碼)轉成現在瀏覽器上跑得動的 JavaScript（目標代碼）。

> 關於這樣子使用是否合法，已經在 twitter 問過作者，

> 相關的 license 也放在最下方了。

# Prerequisite

- 懂 JavaScript(老實說不懂也沒什麼差)

- 不需要懂 Lisp（沒錯，雖然我很喜歡 Lisp）

> 有人要一起開 SICP 的讀書會嗎？XD

- 懂得寫出簡單的遞回函數

- Regular Expression

這裡不會使用 ES 6 的語法，

主要原因只是想讓環境配置盡量單純簡單，

只要有辦法使用 JavaScript 的人就可以讓這個小 compiler 跑起來。

這裡要 compile 的不是 babel 或 js，

而是 lisp，原因是因為語法單純簡單得多，

能夠讓我們更專注在 compiler 的概念和抽象化上。



# 三個步驟

影片中有提到，要做出一個 compiler，

基本上只需要三步就完成了：

- Parsing：將 code 轉成抽象化的樹狀格式，方便轉化。

- Trasformation：將 Abastract syntax tree（之後會講到）轉化成好生成 code 的形式

- Code Generation：產生目標的程式碼，這裡是 JavaScript

坦白說現在看起來是蠻直觀的想法，

但這種事情，都碼是你想出來之後就覺得很簡單，

所以做完一遍之後，反而更能感受到設計 compiler 是一件多麽偉大的事情。

> 想當年臉書紅的時候，多少人說過自己當年也想要做一個社群網站呢 XD

# Goal

開始之前，先定義一下我們要完成什麼事情。

我們要將：

```scheme
(add 22 (subtract 43 2))
```

Compile 成：

```js
add(22, subtract(43, 2))
```

就是這樣而已。~~應該沒有很難吧？~~

綜合一下前面講的，我們要寫的 compiler 會長這個樣子：

```js
function compiler(input) {
    var tokens = lexer(input);
    var ast = parser(tokens);
    var nextAst = transformer(ast);
    var output = codeGenerator(nextAst);
    return output;
}
```

# 1. Parsing

Parser（或稱 lexer），

會將 raw code 先切成一塊一塊後，

再根據這些小塊的語義來建立一個 **Abstract Syntax Tree**（以下簡稱 AST）。


這裡很明顯的分成兩個步驟：

1. __Lexical Analysis__：就是分詞啦！把 code 切成一塊塊的 tokens。

2. __Syntatic Analysis__：將上一個步驟的 tokens 轉成 AST


## 1-a Parsing: Lexical Analysis

Lisp 的語法相當簡單，而且我們沒有要實作所有的語法 XD

目前看到的就是分成三種：Letters、Numbers，跟 Paranthesis。

> 會將多餘的空白忽略，因為空白的區隔是為了讓開發者好讀

就以這三個去分，先寫出三個 regular expression 來：

```js
var WHITE_SPACE = /[\s]/;
var NUMBERS = /[0-9]/;
var LETTERS = /[a-z]/i;
```

接著要 iterate 輸入的 raw code：

```js
var current = 0;
var tokens = [];
while(current < input.length) {
    // Lexical analysis
}
```

首先第一個就是先略過空白

```js
if (WHITE_SPACE.test(char)) {
    current += 1;
    continue;
}
```

再來是括號

```js
if (char === '(' || char === ')') {
    tokens.push({type: 'parenthesis', value: char})
    current += 1; 
    continue;
}
```

最後兩個有點像，分別是 `NUMBERS` 和 `LETTERS`：

```js
if (NUMBERS.test(char)) {
    var value = '';
    while(NUMBERS.test(char)) {
        value += char
        current += 1;
        char = input[current];
    }
    tokens.push({type: 'number', value: value});
    continue;
}

if (LETTERS.test(char)) {
    var value = '';
    while(LETTERS.test(char)) {
        value += char
        current += 1;
        char = input[current];
    }
    tokens.push({type: 'name', value: value});
    continue;
}
```

以數字為例，只要碰到了第一個數字，

就會接著把剩下遇到連續的數字一起推進去。

所以這一階段我們會得到一個 tokens：

```js
[ { type: 'parenthesis', value: '(' },
  { type: 'name', value: 'add' },
  { type: 'number', value: '22' },
  { type: 'parenthesis', value: '(' },
  { type: 'name', value: 'subtract' },
  { type: 'number', value: '43' },
  { type: 'number', value: '2' },
  { type: 'parenthesis', value: ')' } ]
```

最後，如果沒有對應的 type，

會丟出一個 Type Error：

```js
throw new TypeError('I dont know what this character is: ' + char);
```

完整的 `lexer` 在[這裡](https://github.com/abalone0204/super-tiny-compiler-practice/blob/master/super-tiny-compiler.js#L1:L56)可以看到

> 為了方便寫成文章，所以做了一些改寫 XD

> 順序也有所調動。

> 畢竟原本是在半小時內要講完的事情，

> 原作者 James Kyle 也樂見有更多人對 Compiler 有興趣，

> 詳細情形見最下方 License，請不要擔心。

## 1-b. Parsing: Syntatic Analysis

這一階段的任務就是把 tokens 轉成 **AST**。

這裡應該是最容易卡關的部分 XD，

不過讓我們慢慢來，並感受一下為什麼要這樣做。

這裡會運用到遞迴，減少各種迴圈，大大減低了 code 的數量，

也提高了可讀性，而且看起來還很帥。

有句話說得好：

>「嫩嫩迴圈，大大遞迴。」

只要能遞迴，就一定要遞迴一下。

來看一下 parser 的結構是怎麼樣：

```js
function parser(tokens) {
    var current = 0;
    function walk() {
        // Walk
    }
    var ast = {
        type: 'Program',
        body: []
    };
    while (current < tokens.length) {
        ast.body.push(walk());
    }
    return ast;

}
```

可以看到我們這裡還是會移動 current 來遍歷每個 token，

只是改成呼叫 `walk`函數，利用 JavaScript closure 的特性，

呼叫並且更改 current。

> 如果對 closure 和 funcitonal programming with js有興趣，

> 可以參考一下這篇文章：

> [The Two Pillars of JavaScript — Pt 2: Functional Programming](https://medium.com/javascript-scene/the-two-pillars-of-javascript-pt-2-functional-programming-a63aa53a41a4#.ggszw4hmu)


來看看 `walk`：

首先當然是先拿到 token：

```js
function walk() {
    var token = tokens[current];
    // get token
}
```

先看一下比較單純的遇到數字該怎麼辦（以下幾個 type 的確認都在 walk 裡完成）：

```js
if (token.type === 'number') {
    current += 1;
    return {
        type: 'NumberLiteral',
        value: token.value
    };
}
```

再來是如果遇到 `(` 該做什麼事，

直接看一整段太長了，所以我將它拆成兩半：

```js
if (token.type === 'parenthesis') {
    if (token.value === '(') {
        
        current += 1;
        token = tokens[current];

        var node = {
            type: 'CallExpression',
            name: token.value,
            params: []
        };

        current += 1;
        token = tokens[current];
        // To be continued with part 2
        
    }
}
```

往後移一個，拿到下一個 token ，按照 lisp 的語法，

這裡會是一個 expression 的名字（可以想成 function name）。

建立一個 node object，params 裡面放的就是這個 expression 吃的參數。

再來繼續往下看下個 token，

這裡會比較困難一點點：

```js
if (token.type === 'parenthesis') {
    if (token.value === '(') {
        // ....
        // 接續前面 part 1
        while (
            (token.type !== 'parenthesis') ||
            (token.type === 'parenthesis' && token.value !== ')')
        ) {
            node.params.push(walk());
            token = tokens[current];
        }

        current += 1;


        return node;
    }
}
```

假如不是 `)` 的話，就會繼續往下走，

因為expression 中可能還是會有 expression：

```
(add (add 2 1) (subtract 1 2))
```

所以假如遇到 `(`，就會在執行一次 walk，

前面已經知道 walk 的功用就是解析一個expression，

要解析一個 expression 中的 expression 的方法，

那就在 `walk` 裡面再 call 一次 `walk` 就好了，

我想可以人體 compile 一下上面的那行 lisp，

會更理解這個概念。

最後在 walk 函數的最後，一樣加上 type error 的 handling

```js
throw new TypeError(token.type);
```

實作完 `walk` 函數以後，要建出 AST 就簡單多了

```js
function parser(tokens) {
    function walk() {
        // plz ref to source code
    }
    
    var ast = {
        type: 'Program',
        body: []
    };

    while (current < tokens.length) {
        ast.body.push(walk());
    }

    return ast;
}
```

語法盡量寫的非常淺顯易懂，除了講解容易之外

在很難設中斷點的 js 裡，寫太聰明的 code 只是在搞自己而已。

這裡一樣附上完整的原碼: [parser](https://github.com/abalone0204/super-tiny-compiler-practice/blob/master/super-tiny-compiler.js#L58:L116)


# 2. Transformation

這一階段的目標是要將 AST 轉成專為生成 JavaScript 而生的 `nextAST`，

我認為稍微抽象一點的應該就是 Traverser 的部份，

不過如果你有遍歷各種樹的概念，那以下應該會是非常簡單的事情。

這裡要分成兩個函數來實作：

- Traverser：去遍歷我們前面造出來的 AST，並執行我們想要執行在每個節點上的 function

- Transformer：利用前面做出來的 Traverser 轉化成專為 JavaScript 而生的 `nextAst`

為了更清楚知道我們在做什麼，先看一下 transform 的結果：

```js
/**
 * ----------------------------------------------------------------------------
 *   Original AST                     |   Transformed AST
 * ----------------------------------------------------------------------------
 *   {                                |   {
 *     type: 'Program',               |     type: 'Program',
 *     body: [{                       |     body: [{
 *       type: 'CallExpression',      |       type: 'ExpressionStatement',
 *       name: 'add',                 |       expression: {
 *       params: [{                   |         type: 'CallExpression',
 *         type: 'NumberLiteral',     |         callee: {
 *         value: '22'                 |           type: 'Identifier',
 *       }, {                         |           name: 'add'
 *         type: 'CallExpression',    |         },
 *         name: 'subtract',          |         arguments: [{
 *         params: [{                 |           type: 'NumberLiteral',
 *           type: 'NumberLiteral',   |           value: '22'
 *           value: '43'               |         }, {
 *         }, {                       |           type: 'CallExpression',
 *           type: 'NumberLiteral',   |           callee: {
 *           value: '2'               |             type: 'Identifier',
 *         }]                         |             name: 'subtract'
 *       }]                           |           },
 *     }]                             |           arguments: [{
 *   }                                |             type: 'NumberLiteral',
 *                                    |             value: '43'
 * ---------------------------------- |           }, {
 *                                    |             type: 'NumberLiteral',
 *                                    |             value: '2'
 *                                    |           }]
 *  (sorry the other one is longer.)  |         }]
 *                                    |       }
 *                                    |     }]
 *                                    |   }
 * ----------------------------------------------------------------------------
 */
```

## 2-a. Traverser

為了要遍歷我們的 AST，

我們要先寫一個 helper function 來 traverse 每一個 token 的節點。

以下一樣把 traverser 分成兩部分來看，

現在先只要專注在最上方的 `traverseArray` 就行了：

traverseArray會做的事情就是對每個子節點執行 traverseNode。


```js
function traverser(ast, visitor) {
    function traverseArray(array, parent) {
        array.forEach(function(child) {
          traverseNode(child, parent);
        });
    }
    function traverseNode(node, parent) {
        // to be continued
    }
    traverseNode(ast, null);
}
```

> traverser 裡面的 `visitor`，

> 面放著我們**「拜訪」**每個節點時要執行的方法，

> Transform 的工作就是由 visitor 完成的，這裡先不要急，

> 到 `transform`這個函數時就會看到 visitor 是如何作用的。

首先我們根據子節點的 type 呼叫對應執行的 method，

找到的話執行它，待會一再對子節點要執行的就是這一部份：


```js
function traverser(ast, visitor) {
    function traverseArray(array, parent) {
        array.forEach(function(child) {
          traverseNode(child, parent);
        });
    }
    function traverseNode(node, parent) {
        var method = visitor[node.type];

        if (method) {
            method(node, parent);
        }
        // to be continued
    }

    traverseNode(ast, null);
}
```

接著再根據子節點的 type，去執行 `traverseArray`，

Program 的子節點是 `body`，

CallExpression 的事 `params`，

而單純的 NumberLiteral 則沒有子節點需要被遍歷。

```js
function traverser(ast, visitor) {
    function traverseArray(array, parent) {
        array.forEach(function(child) {
          traverseNode(child, parent);
        });
    }
    function traverseNode(node, parent) {
        var method = visitor[node.type];

        if (method) {
            method(node, parent);
        }
        switch(node.type) {
            case 'Program':
                traverseArray(node.body, node);
                break;
            case 'CallExpression':
                traverseArray(node.params, node);
                break;
            case 'NumberLiteral':
                break;
            default:
                throw new TypeError(node.type);
        }        
    }

    traverseNode(ast, null);
}
```

- [traverser source code](https://github.com/abalone0204/super-tiny-compiler-practice/blob/master/super-tiny-compiler.js#L118:L148)

## 2-b. Transformer

再來則是重頭戲： `transformer`，

`transformer`是個相當 powerful 的概念，

~~至少在麥考貝拍爛它之前都是~~。

將我們一路 parse 過來的東西，轉成跟目標語言非常相近的 **AST**。


首先先造出一個新的 `nextAst`：

```js
function transformer(ast) {
    // init
    var nextAst = {
        type: 'Program',
        body: []
    };
    // To be continued
}
```

再來這裡是有點 tricky 的部份，

我們對 ast 底下增加了一個隱藏的屬性：`_context`，

下面我們對子節點的操作也會常常用到這個非常 naive 的方法。

> 其實只是在名字前面加上底線，並不是真正的隱藏

再來則是前面有提的的 `visitor`，

這裡就能夠看出為什麼選擇 Lisp 了，

語法非常的簡單且直觀，

但 `transformer`仍然是一個相較之下較為複雜的函數：

```js
function transformer(ast) {
    var nextAst = {
        type: 'Program',
        body: []
    };

    ast._context = nextAst.body;

    var visitor = {
        NumberLiteral: function (node, parent) {
            parent._context.push({
                type: 'NumberLiteral',
                value: node.value
            });
        },
        CallExpression: function (node, parent) {
            // to be continued
        }
    }
    traverser(ast, visitor)
}
```

`NumberLiteral` 這個 method，做的事情並不難，

只是 push 一個節點到父節點的 `_context` 中而已。

假如今天我們的程式什麼都沒有，只有一個單純的數字：

```
2
```

那 transfomer 會造出來的`nextAst`就是這樣

```js
{
    type: 'Program'
    body: [{type: 'NumberLiteral', value: '2'}]
}
```

跟前面的 ast 幾乎是沒有差的。

接著看如果遇到 function call 時要怎麼做，

為了簡潔我省略了其他部分的 code。

我們同樣先造出一個 expression 的 object，

包含一個 callee 屬性。

> callee 就是被呼叫的 function

> `(add 2 3)` 中，被呼叫的就是 `add` 這個 expression

再來同樣在這個節點建立一個 `_context`，

並將其指到我們剛剛剛創造的 `expression`，

以下是 `CallExpression` 這個 method 的 上半部：

```js
function transformer(ast) {
    // pass
    var visitor = {
        // pass
        CallExpression: function (node, parent) {
            var expression = {
                type: 'CallExpression',
                callee: {
                    type: 'Itentifier',
                    name: node.name
                },
                arguments: []
            };

            node._context = expression.arguments;
            // pass
        }
    }
    traverser(ast, visitor)
}
```

再來則是跟 JavaScript 比較相關的部份，

因為 JavaScript 最上層的 Call Expression 其實是 statement，

所以在確定該個 expression 的父節點 type 不是 `CallExpression` 時，

要再多加一層 `ExpressionStatement`。

> Statement 就是敘述句，像是 `var i = 0`；

> Expression 則是會產生值的，像是`yo()`。

> 但你知道的，有些 Statement 的地方我們仍然可以產生值，

> 因此也就有了 `Expression Statement` 的存在。

> 如此概括是有點草率，不過這裡還是將重點放在我們的 Compiler 上。

> 關於 Expression 和 Statement 在下方補充資料有放上一篇我覺得既短小又很不錯的文章！

```js
function transformer(ast) {
    // pass
    var visitor = {
        // pass
        CallExpression: function (node, parent) {
            var expression = {
                type: 'CallExpression',
                callee: {
                    type: 'Itentifier',
                    name: node.name
                },
                arguments: []
            };

            node._context = expression.arguments;
            if (parent.type !== 'CallExpression') {
                expression = {
                      type: 'ExpressionStatement',
                      expression: expression
                }
            }
            parent._context.push(expression);

        }
    }
    traverser(ast, visitor)
}
```

於是我們就完成了 `transformer` 了！

下一階段就是根據這個 `nextAst` 來生成 code 了。

一樣附上 `transformer` 的 [code](https://github.com/abalone0204/super-tiny-compiler-practice/blob/master/super-tiny-compiler.js#L150:L192)

# 3. Code Generator

終於到最後啦！

其實有了前一段專為 JavaScript 生成的 `nextAst`之後，

要生成 JavaScript 真的是毫不費力。

直接來看 `codeGenerator` 這個函數，

- `Program`: 用換行來區分各個小 Program。

- `Expression`:再來是在每個 Expression 後面加上 `;`。

- `CallExpression`: 
    - `callee` 就是被呼叫的函數，
    - `arguments` 則會被逗號分開來，如果 argument 是 expression 的話會繼續遞迴的呼叫 `codeGenerator`

- `Identifier`: expression （函數）的的名稱。

- `NumberLiteral`: 毫無反應，就是個數字。

- 最後則是不包含以上 type 的 node，就會丟出 type error。~~真的是非常 robust。~~

```js
function codeGenerator(node) {
    switch(node.type) {
        case 'Program':
            return node.body.map(codeGenerator)
                .join('\n');
        case 'ExpressionStatement':
            return (
                codeGenerator(node.expression) + ';'
                )
        case 'CallExpression':
            return (
                codeGenerator(node.callee)+
                '('+
                node.arguments.map(codeGenerator).join(', ')+
                ')' 
            );
        case 'Identifier':
            return node.name;
        case 'NumberLiteral':
            return node.value;
        default:
            throw new TypeError(node.type);
    }
}
```
- [source](https://github.com/abalone0204/super-tiny-compiler-practice/blob/master/super-tiny-compiler.js#L194:L217)

# Super Tiny Compiler

將以上的 function 組合起來，

就是一個 compiler 了：

```js
function compiler(input) {
    var tokens = lexer(input);
    var ast = parser(tokens);
    var nextAst = transformer(ast);
    var output = codeGenerator(nextAst);
    return output;
}
```

# 後記

很多時候我們都會覺得有好用的工具，

幹麻要自己造輪子呢？

但在造輪子的過程中，我們獲得的往往更多，

畢竟盲目的 call api 和使用 library 並不能體現一個軟體工作者的價值，

懂得何時該使用，甚至創造工具才是我們的天職所在。

能掌握更多知識，就能設計出更有創意的東西，

因為我們更了解所謂的「極限」在哪裡。

雖然只是一個「簡單的」from lisp to js compiler，

但相較於過度困難的屠龍本而言，

我想這是一個能相對友善了解 compiler 的起點。

# References

- 可以到我的 repo 去看：[Super tiny compiler practice](https://github.com/abalone0204/super-tiny-compiler-practice)，也可以看下方原作的XD 有些微妙的不同

- [Super tiny compiler](https://github.com/thejameskyle/the-super-tiny-compiler)

- [The Two Pillars of JavaScript — Pt 2: Functional Programming](https://medium.com/javascript-scene/the-two-pillars-of-javascript-pt-2-functional-programming-a63aa53a41a4#.ggszw4hmu)

- [Expressions versus statements in JavaScript](http://www.2ality.com/2012/09/expressions-vs-statements.html)

# License

- [licensed through Creative Commons](http://creativecommons.org/licenses/by/4.0/)
