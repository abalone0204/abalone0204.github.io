---
title: "Building Abstractions with Procedures"
tags: [sicp, plt, computer-science]
---

Lisp 是很先進的語言，只是生不逢時，

以往 MIT 的入門課都會以 Lisp 來教學，

使用的教材則是 SICP(Structure and Interpretation of Computer Programs)，

與其說是在教你特定一門語言，不如說是在教你什麼叫做真正的「程式思維」。

大概是在一個下著雨的午後，看著瀏覽器開著的視窗分別是 vuejs、machine learning，

各式各樣琳琅滿目根本學不完的東西，學習不系統、扎實絕對是一件傷害很大的事情，

更想從語言、工具庫、框架中解放出來，

想了又想，總歸該是靜下心好好學所謂基礎，所以打開了這本書。

> 這本書是開源的，所以理論上你可以在 [MIT 的網站找到它](https://mitpress.mit.edu/sicp/full-text/book/book.html)

<!--more-->

# 前言

這不是一系列讓人速成的文章，

寫程式本身是一種藝術，在藝術上要有所成就，

沒有什麼速成的事情，

只是紀錄一下現階段的我看完之後得到的想法或啟發。

也許一年後的我回頭看，就會覺得這些想法是很幼稚的，

但這同時也代表未來的我比現在的我要來的進步多，

而當我有這種感覺時，當然也會回頭重構這篇文章，

這種跟自己討論的感覺，極爽。

> 但也很期待有人在下方留言能一起交流囉

最後我非常主觀地認為，觀看這本書時，

直接讀原文是較恰當的，並不是因為中文翻的不好，

而是這樣能強迫你慢下來思考和消化，

語言也影響一個人思考的模式，

要進入作者思考的迴路中，最好的路徑就是以他的語言。

# Prerequisite

- No，但我並不覺得這是一個簡單的課。

- 安裝 Chez Scheme（最近[開源了](https://github.com/cisco/ChezScheme)）

```bash
git clone git@github.com:cisco/ChezScheme.git
cd ChezScheme
./configure
sudo make install
```

# Building Abstractions with Procedures

抽象化是寫程式中重要的理念，

其實電腦的運作，說穿了就是一堆 0 和 1 的轉換，

而 0 和 1 的轉換也只是高低電位的差別。

## 1.1 The elements of Programming

我們在談論程式語言時，常常只注重「程式」，

關注於跟電腦的互動，忽略了程式語言，

跟我們的語言一樣，都是一種表達的方式。

而組成一個強大、具有表達力的語言據有以下三個特性：

- `primitive expressions`

```
> (1)
1
```

- `means of combination`

    - 可以組合幾個簡單的想法，達到一個複雜的目的

- `means of abstraction`
    
    - alias XD

在寫程式時，我們其實就是在處理兩種元素：

- `data` : 我們要操作、改變（manipulate）的東西

`procedures`: 描述我們如何去改變(manipulate)  `data`

不過這兩者並不是被嚴格的區分開來，

> 像在 lisp 中，everything's data。

第一章會專注在數值 data 上，

讓我們熟悉怎樣去描述一個 procedure。

## 1.1.1 Expressions

這裡要先裝好 scheme 的解釋器（interpreter ）

> 解釋器其實就是一個函數，你輸入一個 expression，

> 它會輸出一個值

> `(expression) => value`

Expression 可以是單一個 primitive 值，

也可以是數個簡單的 procedures 所組合而成，

重點在於最後**得到了一個值**。

像是：

```
> 123
123
> (+ 2 12)
14
```

Scheme 中的 operator（+, -, *) 都是 prefix 的，

需要習慣一下，不過以一個習慣於寫程式的人來說，

operator 放在前面就像是在 call 一個 function 一樣那麼自然。

而括號決定了求職的優先順序，

我們能得到一個 「No ambiguity can arise」的計算式。

當然，Lisp 也能夠支援好一點的 format 來求值，

畢竟程式碼是給人看的。

比起這樣：

```
> (+ (* 3 (+ (* 2 4) (+ 3 5))) (+ (- 10 7) 6))
57
```

下面這樣的形式是更加優雅，且一樣能得到正確結果的：

```
> (+ (* 3
      (+ (* 2 4)
         (+ 3 5)))
   (+ (- 10 7)
      6))
57
```

而且也更加接近一個樹狀圖，

讓每一次的計算都是有跡可循的。

## 1.1.2  Naming and the Environment

提供一個名字對應到某個 procedure 上也是相當重要的。

在 Scheme (Lisp 的方言)中，我們用 `define`

```
> (define size 2)
> size
2
```

這就是抽象化的概念，上述的例子可能還不代表什麼。

來看看下述例子如何靠著 naming 讓程序變得更有意義：

```
> (define pi 3.14159)
> (define radius 10)
> (define circumference ( * 2 pi radius))
> circumference
62.8318
```

> 稍微熟悉 programming 的人能想到更通用的方法，

> 不過這裡主要是在介紹藉由最簡單的抽象化：Define

> 能幫助我們做到什麼事情

我們在這一段中拆分了一個 computational object 的複雜度，

step by step 的去完成每件小事，

最後再得到一個較為複雜的結果。


我雖然喜愛 functional programming，

但我並不會一味的追求不去 define 一個變數，

事實上，任何能讓程式語言更能 mapping 到這個實體世界的寫法，

我們都該敞開心胸去嘗試。

而我們 define 的這些值，

會由 interpreter 幫我們存在記憶體中，

這些記憶體位置就叫做「environment」，

更精確一點的說應該叫做「global environment」。

> 在稍後的章節中我們才會看到什麼是 local 的 environment

## 1.1.3  Evaluating Combinations

Evaluate，中文翻作求值。

書上把求值這件事情用很精練的語言分成了兩步驟：

> 1. Evaluate the subexpressions of the combination.

> 2. Apply the procedure that is the value of the leftmost subexpression (the operator) to the arguments that are the values of the other subexpressions (the operands).

看來還是有點抽象，但如果化成樹的形式，

就會好理解很多：

![tree](https://mitpress.mit.edu/sicp/full-text/book/ch1-Z-G-1.gif)

我們會先求出最下方節點的值（subexpression），

再讓 operators 作用在 operands 上。

## 1.1.4  Compound Procedures

這一小節要來「組合」 procedures，

以完成更複雜的任務。

> 有寫程式的經驗的人，通常都已經習慣宣告一個函數、方法

> 而底下的概念便是創造了一個 compound procedure

```txt
> (define (square x) (* x x))
> (square 6)
36
``` 

`square` 當然也能拿來組裝

```
> (define (sum-of-squares x y)
    (+ (square x)(square y)))
> (sum-of-squares 3 4)
25

```

## 1.1.5  The Substitution Model for Procedure Application

這裡要來解釋一下前面較複雜的 procedure 是怎麼運作的。

先假設我們的 interpreter 可以直接求出 primitive procedure 的值

> 像是 `+`, `-`, `*` 以及數值

會先找出對應的 compound procedure，

再將 body 中的參數替代成我們在最外層呼叫的 argument，

講起來太抽象，直接看例子會比較具體

```bash
> (define (f a)
    (sum-of-squares (+ a 1) (* a 2)))
```

我們對 f 這個 compound procedure 傳入 2 這個 parameter

```scm
(f 2)
```

他會先將 f 換成 body 中對應的形式：

```scm
(sum-of-squares (+ a 1) (* a 2))
```

接下來把 a 替換成我們一開始傳進來的 parameter

```scm
(sum-of-squares (+ 2 1) (* 2 2))
```

```scm
(sum-of-squares 3 4)
```

而 scm 也是一個 compound procedure，

所以替換的 process 會遞迴地進行下去

```scm
(+ (square x)(square y))
```

```scm
(+ (square 3)(square 4))
```

```scm
(+ (* 3 3)(* 4 4))
```

```scm
(+ 9 16)
```

```scm
25
```

這個過程叫做 substitution model，

了解這個過程注重在對 evaluate 的 process 有更全面的思考，

而不是 interpreter 到底如何做到的。

且上述的求值過程並不是唯一的。

> 上述的過程中，我們邊展開邊求值
> 另外也有先把 procedure 全展開以後
> 再開始求值的方法
> 稱作 normal-order evaluation
> 而剛剛的方式則是 application-order evaluation
> Lisp uses applicative-order evaluation

## 1.1.6  Conditional Expressions and Predicates

```scm
(define (abs x)
    (cond ((> x 0) x)
          ((= x 0) 0)
          ((< x 0) (- x))))
```