---
title: Text Mining & Analysis： week 2
tags:
  - textmining
  - NLP
  - coursera
  - topic-mining
  - PLSA
date: 2016-07-21 16:09:00
---


這是一篇關於 Coursera 上：[Text Mining and Analytics (伊利諾伊大學香檳分校)](https://www.coursera.org/learn/text-mining/home/welcome)的筆記。

主要在探討 text mining 和 分析，

經過我整理之後應該會變得好懂很多。

這是第二個禮拜課程的筆記，

假如你還沒看過第一個禮拜，

可以先看這裡：

- [Text Mining & Analysis： week 1](http://abalone0204.github.io/2016/07/15/illinois-text-mining-week-1/)

<!--more -->

# Outline

這一週裡面會包含三個主題：

- Syntagmatic relation discovery

- Topic Mining and analysis

- Probabilistic Topic Models

# 前情提要

在開始之前先快速複習一下上禮拜學習的東西，

試著去解釋和回答以下幾個問題，

如果還會遲疑的話可能要回到[第一篇](http://abalone0204.github.io/2016/07/15/illinois-text-mining-week-1/)稍微看一下：

- deep techniques 和 shallow techniques 的 trade-off 是什麼

- 了解為什麼 NLP 是困難的

- Paradigmatic relation 以及 Syntagmatic relation

- Paradigmatic 以及 BM25 加上 tf-idf 


這一章節可能會有比較多特別的名詞或公式，

不過拆開來一步步理解，

他們其實都是蠻直觀的概念，

我以前也是因為這堆符號嚇到被老師當掉過，

實際去學才發現，

~~嗯老師考卷還是出太難馬的幹~~

其實背後的本質就是簡單的數學而已。

# Syntagmatic relation discovery

要找到 Syntagmatic relation，

要觀察的就是 co-occurence，（同時出現的機率）。

## Intuition

為了培養一下對於挖掘組合性關係（Syntagmatic relation）的直覺，

直接來看下面這個問題，

有以下三個字，

你覺得哪個字最好去預測它在文本中會不會出現呢？

> 這裡的文本指的可能是 pseudo document、句子、段落

$$ W_1 = "meat"$$

$$ W_2 = "the"$$

$$ W_3 = "unicorn"$$

答案會是 "the"，因為它到處都是嘛！

再來可能會是 unicorn，因為這不是個常用的單字，出現機率可能很低。

> 矽谷表示： __

最後則是 meat，因為他出現的機率最為「難預測」，

你很難精確說明怎樣的情形「一定會」出現 meat，

也很難解釋什麼條件下「一定不會」出現 meat。

上述的情境其實可以當作一個 function 來看，

輸入是某個字（meat, the, unicorn），

輸出則是它會不會出現在文本中，

會就是 1，不會就是 0。

而上述的這個概念就是「隨機變數（Random Variable）」，

> 沒錯，我一開始也很疑惑，

> 但隨機變數就是一個函數，

> 數學家才不管什麼可維護性，naming 這種東西就是他們說了算，懂？

將一個機率空間中的值（子集合）對應到一個實數上（這裡是 0 或 1），

這就是 random variable。

$$
X_w = 0 \, or \, 1 
$$

如果 w 存在文本中就等於 1，反之則為 0。

所以 `X_w` 如果越隨機，就越難預測，

問題來了，該怎麼去量化誰「比較隨機」呢？

## Entropy

要靠的是 Entropy(熵)，不要被這個名詞嚇到了，

其實它也只是一個值，算出來越高，

代表這個 random variable 越隨機，這也就代表越難預測。

來看它的公式怎麼寫：

$$ 
\sum_{v = 1 \, or \, 0} -p(X_w=v)log_2 p(X_w=v)
$$

> 這裡為了方便計算，要假設 log_2(0) 為 0。

接著拿執硬幣的機率來舉例，

假設有個公正的硬幣，兩面出現的機會相等，都是 0.5；

另一個只會擲出反面的硬幣，

哪一個會算出較高的熵？

首先把公式列出來看：

$$
entropy = -p(x=正面)log_2(p(x=正面))-p(x=反面)log_2(p(x=反面))
$$

再來先帶入公正的硬幣：

```
p(x=正面) = 0.5

p(x=反面) = 0.5
```

$$
H(X) = -\frac{1}{2} log_2(\frac{1}{2})-\frac{1}{2} log_2(\frac{1}{2}) = 1
$$

而帶入

```
p(x=正面) = 0

p(x=反面) = 1
```

$$
H(X) = -0 \times log_2(0)- 1 \times log_2(1) = 0
$$

看的出來公平硬幣的熵較高，

而在數學上證明了這個直覺是對的，

公平硬幣比起只會出現一面的硬幣來的隨機多了！

## Conditional Entropy

很快地會發現，前面講的只是單一個字會不會出現的隨機性，

換言之這裡只計算了 occurence；

但我們真正想要知道的是跟其他字同時出現的 co-occurence 啊！

這時候就是 conditional entropy 上場的時候。

當知道字句中有 "eats" 這個字存在時，

出現 "meat" 的機率可以表示為下面這樣

$$
P(X_m = 1 \mid X_e = 1)
$$

所以直接把 entropy 公式中的`P(X_m)`換成 `P(X_m | X_e)`即可算出 conditional entropy：

$$
H(X_m \mid X_e = 1) = -p(X_m \mid X_e =1 )log_2(p(X_m \mid X_e =1 ))
$$

而 `H(X_m)` 會是 `H(X_m | X_e=1)` 的上界

> 因為 `H(X_m)` = `H(X_m | X_e=1)` + `H(X_m | X_e=0)`

這裡隱含著的道理就是「做的任何事情都是要降低不確定性」，

conditional entropy 不會超過原本的 entropy，

先有這個數學上的直覺，清楚我們目標在哪後再繼續。

# Mutual Information

## Concept

上述知道使用 conditional entropy 可能會降低不可預測性後，

就要來問：要怎麼量化減少的程度為多少？

這時候就會用到 mutual information，

它代表的意義就是當我們知道其中一個詞時，

到底降低了多少的不確定性。

mutual information 用數學符號表示：

$$
I(X;Y) = H(X) - H(X|Y) = H(Y) - H(Y|X)
$$

圖大概是長這個樣子：

![mutual info](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d4/Entropy-mutual-information-relative-entropy-relation-diagram.svg/512px-Entropy-mutual-information-relative-entropy-relation-diagram.svg.png)

而 `I(X;Y)` 有三個特性：

- 大於等於 0

- 對稱性：

$$
I(X;Y) = I(Y;X)
$$

- 只有 X 與 Y 互相獨立時，`I(X;Y)`會等於 0

前面是從熵的角度去切入，

更 practical 一點的做法是用機率的角度去看，

這裡介紹 KL-divergence 這個公式，

（或稱作 relative entropy 相對熵）


![kl-div](http://i.imgur.com/BOQuu10.png)


> 格式請見諒，mathjax 在同一行要使用 X_{..}系列時會出錯

kl-divergence 最值得注意的是 log2 裡面的這一項：

$$\frac{p(X_w1 = u,\, X_w2 = v)}{p(X_w1 = u) p(X_w2 = v)}$$

如果`X_w1` 跟 `X_w2`是相互獨立的話，

分子分母會相同，

因此這一項會趨近於 1，

再取 log 後就會變成 0，

這樣在計算 divergence 時就會把這一項相互獨立的給去掉。

算 mutual information 的意義在於：

「因為我們想了解如果知道其中一個字，

到底能下降多少的不確定性？」

而兩個字出現的機率如果是完全獨立，（意思即 w2 出不出現都跟 w1沒差）

那 mutual information 自然要等於零，

因為知道其中一個字完全不會影響另一個出現的機率，

自然也不會降低任何不確定性了。

## Joint 和 Independent(Optional)

> 這裡是對獨立事件的小小補充，

> 講的並不嚴謹，

> 如果你對於 joint probability 和獨立事件沒有問題的話，可以跳過這一段

> 或是指正一下我哪裡的說明有誤 XD

如果你沒有修過機率論的話可能會覺得：

「欸？ `p(X_w1 = u, X_w2 = v)` 不就等於`p(X_w1 = u) * p(X_w2 = v)`嗎？」

但很可惜事實並非如此，用符號來表示並不直觀，

幸運的是機率這種東西用舉例的來觀察就會很直覺，

我們舉下面這個例子來說：

我們有四個 document，

X 則是 document 中有出現 w1或 w2 的隨機變數，

有出現 = 1，沒出現 = 0。

|document|X_w1|X_w2|
|--------|----|----|
|  d1    | 1  | 0  |
|  d2    | 1  | 1  |
|  d3    | 1  | 1  |
|  d4    | 0  | 1  |
 
`p(X_w1 = 1)` = 0.75
`p(X_w1 = 0)` = 0.25
`p(X_w2 = 1)` = 0.75
`p(X_w2 = 1)` = 0.25

而 `p(X_w1 = 1, X_w2 = 1)`則為 0.5，

但 `p(X_w1 = 1)` 與 `p(X_w2 = 1)`直接相乘卻是 0.1875，

由此可見這兩個在有相依性存在的情況下，

可能會不相等 XD。

但假設今天是擲一枚公正硬幣兩次的話：

`p(x1=正面)` * `p(x2=正面)` = 0.25

`p(x1=正面, x2= 正面)` = 0.25

因為投擲一枚硬幣兩次是獨立事件，

而在更上面的例子中，

`p(X_w1)`與 `p(X_w2)`並沒有完全獨立。

> 基本上有這個概念就已經夠了，

> 如果你還是很有興趣可以去翻翻機率論，

> 我大學時候修過，覺得真的⋯⋯很快樂 ^_^。

## Computing for syntagmatic relation

### Maximum Likelihood Estimation

前面對 Mutual information 有了概念以後，

下來就是真的去計算它，

並且利用它來找出 syntagmatic relation 了。

我們要用的方法是 Maximum Likelihood Estimation（最大概似估計）。

這名字聽起來一樣很炫砲，

但它的想法其實超簡單！

> 只要不是出現在數理統計裡的話

舉一個常見的例子：擲硬幣。

擲硬幣可以表示成一個 random variable: `x`，

假設這個 `x` 的機率分佈是 `p(x)`。

但其實我們不知道這個機率是不是公正的，

先進行了一次觀察，

而在這次觀察中擲了一萬次，~~然後就媽媽手~~

結果發現很剛好的有 5000次正面、5000次反面，

因此觀察後得到了一個 `p'(x)`，

假設觀察後的 `p'(x)` 跟真正的 `p(x)` 分佈一致，

那 `p(x)` 產生 `p'(x)` 的「可能性」就是「最大」的。


用「觀察出來的機率分佈」去推論「真正的機率分佈」，

並最大化觀察結果等於真實機率分布的可能性，

這就是「Maximum Likelihood Estimation」的精神啦！

有了上述直覺後，再看看更嚴謹的數學定義，

我們會寫出一個 likelihood function：

$$
arg \, max_p p(x) = argmax \prod_i p1(x_i)
$$

我們要找可以最大化這個 function 的機率分佈： p。

> `argmax`：

> 這個符號的意思就是要找出輸入後能夠最大化後方這個算式值的 arguments，

> 有時候很佩服數學家想得到這麼奇葩的符號表示法 QQ

再來看最後一個簡單例子，是從[陳鍾誠老師的網站上引用的](http://ccckmit.wikidot.com/st:maximumlikelihood)。

可以表現如何套用這個公式並且指出 Maximum likelihood estimation 的問題。

假設我們擲了一枚硬幣十次，有六次為正面，四次為反面。（這個擲十次硬幣其實就是一個 random variable: X）

我們看看下面三個機率模型可能產生 X 的機率有多少。

```
p1(正) = 0.5 ; p1(反) = 0.5
p2(正) = 0.2 ; p2(反) = 0.8
p3(正) = 0.4 ; p3(反) = 0.6
```

把上面的機率模型帶進去 likelihood function：

```
p1(x)= 0.5^4 ∗ 0.5^6 = 0.00097656
p2(x)= 0.2^4 ∗ 0.8^6 = 0.00041943
p3(x)= 0.4^4 ∗ 0.6^6 = 0.00119439
```

會發現 `p3` 這個得到的值最高，

可是對於投硬幣來說，其實 `p1` 才是更正確的模型才對，

這也就是 MLE(maximum likelihood estimation）的缺點，

如果樣本數太小或是有很嚴重偏誤的話，我們很容易見樹不見林 XD。

總而言之，簡單地去推論 `P(X_w1=1)`、`P(X_w2=1)`、`P(X_w1=1, X_w2=1)`出現的機率：

![Esi of p](http://i.imgur.com/j0nTpXS.png)

> 為什麼找這三個是因為找出來後，

> 其他都可以從這三個機率去推導

我們就從這些觀察值中去假設機率是這樣，

就是用到了前面敘述的「最大概似估計方法」。

不過這樣的做法有個顯而易見的小缺點，

當我們觀察的樣本中，有些字可能從來沒出現過時，

它的機率會被估計成 0，

但我們的樣本可能相對很小，

直接就這樣斷定這個字不可能出現絕非好事。

所以要來介紹一下一個小技巧：**Smoothing**

### Smoothing

smoothing 的做法很簡單，

就是對於每個情況發生的次數，

都加上一個小小的 constant。

![smoothing](http://i.imgur.com/SF3lVWQ.png)

如此就算我們觀察的樣本中完全沒有出現過某個情況，

這個情況也會有一個小小的次數在，

而不是直接斷定其機率為 0。

> 雖然機率為 0 也不代表不可能發生，

> 但這就不在這篇文章討論範圍內了。

## Summary of Syntagmatic mining and Analysis

複習一下三個從訊息論來的概念

- Entropy(`H(X)`)：量化隨機變數 X 的不確定性

- Conditional Entropy(`H(X|Y)`)：在給定 Y 提件下隨機變數 X 的不確定性

- Mutual Information（`I(X;Y)`）：描述 知道 Y 或 X 其中一個後，會降低多少預測的不確定性

在這階段得到的 word association 是相當 general 的，

不管後續要做什麼應用，

都有辦法跟其他算法結合。

掌握了這些基石後，

就可以來應用在稍微 deep 一點的技巧上了。

# Topic Mining & Analysis

前面一個禮拜多有了對詞的基本了解後，

要來做進階一點的分析和挖掘：Topic mining。

## What is topic?

topic 其實就是一堆文字中的「主旨」(main idea)。

隨著資料顆粒度不同，會有所改變。

> 句子、文章、書籍的 topic 萃取會大大不同

## How to discover topics

### Intuition

![topic tasks](http://i.imgur.com/oac33PW.png)

1. 先找到有哪些 topics

2. 將文本資料（document）涵蓋各個 topic 的機率找出來。

有了這個直覺後來更嚴謹的定義一下要做什麼，

- Input:

  - text documents 的集合 `C={d_1, ..., d_N}`

  - Number of topics

- Output

  - k topics: `{theta_1, ..., theta_k}`

  - Coverage of topcis in each `d_i`: `{pi_i1, ..., pi_ik}`

$$
j = 1, 2 ... k 
$$

$$
i = 1, 2 ... N
$$

$$
\sum \pi_{ij} = 1 
$$

再來的問題就是怎麼找出 theta 了

### Term as Topic

最直觀的方法就是從所有文本中，

挑出可以當作 Topic 的「詞」(word = term)。

- Parse 所有的 text 拿到 candidate terms

- 設計一個 scoring function 來判定各個 candidate term 是不是適合當作 topic

- 選擇 k 個有最高分的 terms，但盡可能最小化冗餘

  - 冗餘的意思有點太抽象，意思就是有幾個 topic 如果都長得很像，比如說「車子」、「汽車」、「車車」，那只要選擇一個當作 topic 就好

接著就直接來看看 term as topic 的做法。

最簡單的方式就是直接計算次數，觀察其佔的比例：

![comp by freq](http://i.imgur.com/m66yQ2f.png)

簡單的說 `pi` 就是每個 term 在每個 document(`d_i`) 中所佔的比率，

而 `theta` 就是各個 term 了。

最後算出來的 `pi_ij` 就會是`theta_j`/(`document_i`中所有 term 出現的次數)，

其實看的就是選的這個字在某份文件中跟其他 topics（這裡是 terms）所佔的比率，

最高的話，就判定它可能會是屬於 topic。

但是直接拿實際例子來檢驗，

馬上會發現一個大問題：

以下節錄自一則 NBA 球賽的報導：


> Cavaliers vs. Golden State Warriors: NBA playoff finals ... basketbal ... `travel` to Cleveland ... `star` ...


$$
\theta_1 = "sports"
$$

$$
\theta_2 = "travel"
$$

$$
\theta_3 = "science"
$$


出現 "sports"的次數為 0，

而出現 "travel" 的次數為 1。

所以這個運動新聞的 topic 在上述方法中反而會被歸類到 sports 去。

很明顯的，不能只看 "Sports" 這個字，

應該要把相關的字也加進來才對。

但如果把 related word 也加進來，

那 `star` 這個字就會有點 tricky 了，

因為 `star` 有模糊的空間可以被解釋，

除了表示明星之外，它也可能是指真正的星星，

會與 `science` 產生聯繫（天文物理學）。

總結一下 term as topic 會遇到的問題

- Lack of expressive power
  
  - 只能找出簡單的 topic，太過複雜就無法（前面都是以單一個詞來切分 topic）

- Incompleteness in vocabulary coverage

  - 對於找出相關的字這件事表現不佳

- Word sense ambiguity
  
  - 無法處理詞彙間的 ambiguity （最後說到的 `star` 問題）

# Probabilistic Topic Models

假如 term as topic 行不通就兩手一攤說沒辦法那也太遜，

所以馬上要來優化前面的 topic mining 方法：

回顧一下上面提到 term as topic 會遇到的問題，

以及概念上應該要如何解決：

- Lack of expressive power
  
  - 只能找出簡單的 topic，太過複雜就無法

> 解決方法： `Topic: {Multiple Words}`

> 一個字當 topic 不夠，你有試過好多個一起嗎？

- Incompleteness in vocabulary coverage

  - 對於找出相關的字這件事表現不佳

> 解決方法： 對於 word 加上權重

- Word sense ambiguity
  
  - 無法處理詞彙間的 ambiguity （最後說到的 `star` 問題）

> 解決方法： Split an ambiguous word

首先要做的事情就是小小的改變一下選取的 term: `theta`，

它不再是一個個的單詞，

而是一個詞的機率分配，這絕對是概念上的一個大進步，

先記著 `theta` 代表的是機率分配這件事，

直接看下方這張圖：

![prob](http://i.imgur.com/fA4tAEF.png)

舉 "Sports" 為例子，這代表 "Sports"這個分配底下，

裡面會出現詞的機率就是長這個樣子。

> 這裡的機率分配已經做過 smoothing 的處理，所以不會有 0

$$
V = set(w_1, w_2, ...)
$$

這其實就是字典檔，裡面裝滿了各種 words(`w`)。

而令人好奇的是 `p(w | theta)`代表的意義是什麼，

舉例子可能會清楚一點，

`theta_1` = topic 為 "Sports" 的機率分配，

`p("game" | theta_1)`的意思就是在知道屬於 `theta_1` 的條件下，

"game" 這個字出現的機率。

也因此把每個 topic 當條件下出現的機率加總起來就會是 1。

上面那張圖中也把跟 topic 較不相關的字標成黑色，

可以看到它們在所屬 topic 下出現的機率明顯是較低的。

看一下這種把 `theta`當分配來看待的做法有沒有解決上述問題：

- Lack of expressive power
  
> 解決方法： 一個 topic 底下已經可以有多個字，這些字組合起來可以變成一個相當複雜的 topic

- Incompleteness in vocabulary coverage

> 解決方法： 現在每個字在所屬 topic 中都有其自己的權重

- Word sense ambiguity
  
> 解決方法：現在在不同 topic 中出現的同樣字會有不同的機率

其實我也蠻驚訝只是換個表現的方式，

就能夠帶來這樣的成效，

下面就正式用機率模型的方式來表現 topics：

![prob model](http://i.imgur.com/Ji9yzux.png)

> 注意：theta 是一個 term 的「分布」

而 `pij` 就是該 topic 在這個 `document_j` 中的「覆蓋率」，

假如 `pij` 的覆蓋率越高，代表它越可能是屬於 `theta_i` 這個 topic。

## Generative Model

雖然前面已經提到用機率模型的方式來呈現 topics，

但是要被分析的原始文本資料（text data），本身是沒有機率分布的，

所以要自己「生成」一個有機率分佈的 model 給它們，

這也就是我待會要介紹的概念： Generative model。

![generative model](http://i.imgur.com/f0eu5c1.png)

概念就是如上圖這個樣子，有以下幾個步驟：

- Modeling of Data Generation: `P(Data |Model, lambda)`

- lambda 是所有會產生 text data 的參數：

  - 包括`{theta_1,...theta_k}, {pi_11, ..., pi_1k}... {pi_N1, ... , pi_Nk}`

  - 要求得的答案是：知道哪些參數後，能夠讓我們在知道這些參數的條件下，讓產生出 `Data`的機率最大化。

  - 如果是上面圖來說的話，就是 `lambda*` 這個 parameter

> 上述資料的 lambda 是一維的，只是為了方便圖像化，

> 事實上的 model 不太可能長這麼簡單 XD

## Statistical Language Models(以下都以 LM 簡稱)

- 其實就是一連串字的機率分配

  - `p("Today is Wednesday")` ~= 0.001

  - `p("Today Wednesday is ")` ~= 0.00000000001

- 上下文相關（ context dependent）

- 就是一種用機率方法來生成文字的 model，所以也可以稱作是 "Generative model"

 介紹完概念，就直接來看有哪些 model 可以用。

## Unigram LM

這是一個最簡單的 model，

它假設產生的每個 word 都是相互獨立的。

> 假如你學過 naive Bayes 就會發現這比 naive 還要更 naive

例子：

$$ 
p("today \, is \, Wed") = p("today")p("is")p("Wed")
$$

再來看關於 Unigram LM 更實際的例子：

![unigram sampling](http://i.imgur.com/RHhv2lr.png)

topic 1 是 text mining，

下方的 model 就是已知在這個 topic 的條件下，

各個字出現的機率分配。

從這個分配中去取樣本組成一個 data，

那很自然就會是屬於 text mining 的 document。

(Topic 2 就不再重複敘述)

但反過來從看到 data ，去判定它屬於哪個 topic，

才是我們真正想做的事情。

最直觀的方法就是前面提到的最大概似估計（Mamimum Likelihood Estimation）：

![estimation of unigram lm](http://i.imgur.com/Rxog1gI.png)

但最大概似估計有它的限制，

在上面這張圖中的下方是我們去做分析和探勘時，

要一再問自己的兩個問題：

- 這就是最好的方法了嗎？

- 何謂「最好」？

接著繼續探討怎麼去更優化這個估計方式。

## Maximum Likelihood v.s Bayesian

Maximum Likelihood 的問題：

- 樣本數過小時，估計會很不準

> 舉例：

> 投擲硬幣兩次都正面，就估計這硬幣出現正面的機率為 1，

> 可見當觀察的資料過少時，很容易得到有極大偏誤的結果

在這裡要介紹另一個簡單，但是相當高效的方法：

**Bayes Rule**，也就是大家常在說的貝氏定理。

那 Bayes Rule 的核心概念是什麼呢？

直接舉個例子來說明：

假設現在有一個袋子裡面有 5 個球，

其中 4 顆黑球，1 顆白球，

很明顯地，拿出黑球的機率就是 4/5 = 0.8。

注意，這是在「知道所有情況下」才能這樣簡單的就拿到機率，

但是在現實生活的分析中，

常常會不知道所有的情況，

而貝氏機率的哲學就是從結果逆著去推論現實生活中的情況。

舉上述例子就是從袋子裡拿球三次，其中出現一次黑球、兩次白球，

再從這些觀察的結果去推論拿出球顏色的機率分布是什麼，

這就是 Bayes Rule。

這聽起來很像 Maximum likelihood estimation，

不過他們本來就不是相斥的存在，

先耐著性子繼續看下去要怎樣應用它。

假如你不懂條件機率和貝氏定理的話，強烈建議你看一下這篇文章：

> [数学之美番外篇：平凡而又神奇的贝叶斯方法](http://mindhacks.cn/2008/09/21/the-magical-bayesian-method/)

> 看完這篇補充資料前面的例子之後，一定就懂貝氏的概念是什麼了。

> 這裡不花篇幅去解釋太多，專注在 Bayes 在統計語言模型上的應用

![bayes](http://i.imgur.com/OHaWuVi.png)

從最大化 `P( X | theta)` 變成最大化`P( theta | X)` 或是 `P(X | theta) * P(theta)`，

> `P( theta| X)` 的意思是：

> 在符合 X 條件下，是屬於 `theta`這個 topic的機率。

> 而它也不會跟 `P(X | theta) * P(theta)`相等，

> 而是成正比的關係。


所以關鍵問題就在於如何定義 prior （ `p(theta)`）。

看完例子後，

我想直接拿課堂簡報中的圖來說明比起直接用最大概似估計，

使用 Bayesian estimation 有什麼樣子的進步：

![illustration of bayesian](http://i.imgur.com/NNUJgRL.png)

你會發現最右邊的那個分配就是 MLE 得到的結果，

我們要找的那個是介於 prior 跟 MLE 中間的 posterior mode，

所以這個 estimator 也叫做 MAP(Maximum posteriori estimate)。

除了比直接用 MLE 考慮得更多一些之外，

也更加 general。

前面也有提到貝氏和 MLE 的關係，

這裡其實也有個很直覺得解釋，

那就是如果 `P(theta)` 是個 uniform 分配（也就是說每個出現的機率都一樣時），

那基本上就等於是沒辦法提供任何訊息，

因此等於就是在求 `P(X|theta)`，

即原先使用 MLE 的方法。

## Mining One Topic

接著來運用上述 Bayes rule 的模型，

不過為了專注在概念的解釋上，先用了個簡化的版本，

Mining one topic 的意思是假設只有一個 topic，

所以 input 從原先的: `C={d}, V, k` ，

變成 `C={d}, V`（k 固定為 1），

因此也不會有 `pi` 這個 output，

畢竟在這個例子裡不用去算在每個 document 中的 coverage 。

> 只有一個 topic 的話，那代表不管在哪個 d 裡面，

> coverage 都會是 100%。

首先要來定義建立 Language Model 前該有的設定：

![language model setup](http://i.imgur.com/523dHuM.png)

比較需要說明的是 Likelihood function，

後續會相當頻繁的遇到這個 function ，

所以理解它背後的含義是相當重要的，更別說它其實是個很直觀的函數 XD

這只是數學符號比較多，但沒有多出什麼新的東西來。

前面有說到我們用的模型是 Unigram LM，

所以假設每個詞出現在 text 中的機率都是相互獨立，

因此才能直接用相乘的來取得。

接著看到 `p(x_1 | theta)`.. => `p(w_1|theta)^c(w_1, d)...` 的轉化，

看起來很嚇人，實際上只是從 random variable 轉化成一般機率的形式而已：

`x_i` 其實是指 `w_i` 出現在 document: `d` 中的隨機變數，

是「機率」。

在右上角的 `c(w_i, d)` 指的是 `w_i` 真正出現在 data `d`中的次數，

因為在 Unigram LM 中，每個字出現的機率是相互獨立的，

也就是說同一個字重複出現的機率也是獨立的，

把 `p(w_i | theta)` 相乘起來就會是這個字出現的機率，

也就成功從 random variable 轉化成一般機率的形式。

弄懂 likelihood 後，

就能直接來找哪些參數 `(theta_1,...  theta_M)` 可以最大化 `p(d| theta)`。


$$
\prod_{i=1}^M \, \theta_i^{c(w_i, \, d)}
$$

為了方便計算，將連乘的部分取對數，

這樣做的好處就是直接化成相加的形式：

$$
\sum_{i=1}^M c(w_i, d) log \theta_i
$$

> 這樣做的意義完全就是為了數學上好算 XD

記得前面 `theta` 滿足這個限制後：

$$
\sum_{i=1}^M \theta_i = 1
$$

再回到原本要解決的問題，

怎麼找到有辦法最大化下面這個函數輸出值的 `theta` 呢？

$$
\sum_{i=1}^M c(w_i, d) log \theta_i
$$



答案是—— Lagrange Multiplier Method。

> 想瞭解更多關於 Lagrange 的原理可以看這邊

>  [Lagrange 乘數法](http://episte.math.ntu.edu.tw/entries/en_lagrange_mul/)

> 我大學最討厭的就是天才少年一號尤拉，以及天才少年二號 Lagrange。

你可以點上方的參考資料去深入瞭解，

不過你也可以在不知道原理的情況下繼續直接套用公式，

只要懂得簡單的微積分就好 XD

要找參數有辦法使一個凹函數出現最大值，

只要取個微分等於 0，再去解開參數等於多少就可以。

Lagrange 就是將原本的 function 轉化成以下這個形式：

$$
\sum_{i=1}^M c(w_i, d) log \theta_i + \lambda( sum \, of \, (\theta_i) - 1)
$$

前面有提到：

$$
\sum_{i=1}^M \theta_i = 1
$$

所以加上 lambda 那一項是不會影響 function 本身的，

但是加上 lambda 之後，我們在微分後可以多做許多事情：

![cal](http://i.imgur.com/mHpRT2P.png)

這時候可能你會有個感覺：這只是個數學解題而已嘛！

最後可以得到 `theta`與 `lambda` 間的關係，

再度回到上面 theta 加總起來是 1 的等式，

然後把 

$$
\theta_i =  - \frac{c(w_i, \, d)}{\lambda}
$$

代入

$$
\sum_{i=1}^M \theta_i = 1
$$

> 吃我 der lagrange 喇

最後再將得到的結果 

$$
\lambda = -\sum_{i=1}^N c(w_i, \,d)
$$

代入

$$
\hat{\theta_i} = \frac{c(w_i,d)}{\lambda} 
$$

![cal result](http://i.imgur.com/kHB5jYe.png)

> 上面加了一個小帽子唸作 hat，在統計學裡面這樣標記，

> 通常表示它是代表「估計」出來的機率，

> 而非是真正的機率。（真實的機率其實不得而知）

這個結果相當相當的直覺，

得到的結果其實就是個 normalized count。

> 直接拿 word 在 document 中出現的次數除以 document 的總字數。

## Summary of one topic mining

前面敘述的模型有著我們從第一個禮拜就在討論的問題，

就是它沒辦法過濾掉 "the", "a" 這種每個文章都有的常見字，

這些常見字其實沒有跟 topic 有很高的相關性，

但在 `p(w|theta)` 時卻會有著不低的機率。

![result](http://i.imgur.com/TonnmLC.png)

不過下一個禮拜的課程才會進入要怎麼解決這個問題。

# Conclusion

這一週蠻像以前在學機率論的，不過是簡化版的。

而且相較於以前學習的純理論，到考試再被證明題屌虐一番，

這一次有種 divide and conquer 的感覺，

我們有個很明確的目標和問題要解決，

至少我自己在經過這禮拜課程後，

對於使用統計語言模型以及 topic mining 有了初步的理解，

雖然看到 Lagrange 的時候罵了幾句髒話，

但從基石開始慢慢學起的感覺就是這個樣子，

多了點思考和研究，少了很多很多的浮躁。

也推薦看完後可以去 Coursera 上做做練習題，

看看自己是不是真的懂了。

# References

- [Coursera: Text Mining and Analytics 來自於 伊利諾伊大學香檳分校](https://www.coursera.org/learn/text-mining/home/welcome)

- [熵 (Entropy) - 李天岩](http://episte.math.ntu.edu.tw/articles/mm/mm_13_3_01/)

- [Lagrange 乘數法](http://episte.math.ntu.edu.tw/entries/en_lagrange_mul/)

- [数学之美番外篇：平凡而又神奇的贝叶斯方法](http://mindhacks.cn/2008/09/21/the-magical-bayesian-method/)