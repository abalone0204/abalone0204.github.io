---
title: "Text Mining & Analysis： week 1"
date: 2016-07-15 16:20:51
tags: [textmining, NLP, coursera, tf-idf]
---

最近對於 text mining 的知識頗有有需求，

所以開始上一門 coursera 上的課程，

主要在探討 text mining 和 分析，

會停留在比較 general purpose 的理論和演算法上，

乍看之下其實有點無聊，

但我覺得原因是課堂的教授預設我們有許多預備知識和對數學的靈敏度，

實際上它的內容非常的扎實，而且講解的也很深入，

經過我整理之後應該會變得好懂很多，

> 因為我不像那些教授一樣是天才 XDD 

> 大學也沒修過這類型的課程，只是因為興趣所以研究這個。

> 可能就如同每個在網路上看到這篇文章的人一樣。

如果是跟我一樣的 NLP 新手，這一系列筆記應該會對你很有幫助，

並且成為你往更深層分析研究的基石。

<!-- more-->

# Introduction

> 這裡推薦一下大家去看吳軍教授的「數學之美」，

> 裡面有提到相當多關於「語言」以及電腦科學在自然語言處理上的介紹。

語言其實就是承載知識的載體，

仔細想一想我們其實能從大量的文本中整合出各式各樣的主題（topic），

甚至能從中汲取出作者的觀點來，

但是該怎麼做呢？

原本的自然語言處理領域認為，電腦要學會看文字，

應該要跟人類一樣從文法和字母開始學起，

但後來發現地球上有這麼多語言，每種語言又有特殊的文法，

而文法更是會隨著時代改變，

漸漸的，大家發現這條路好像不是這麼行得通。

> 更別說有 context dependancy 的語法分析的時間複雜度高達 O(n^6)

經過數十年在學術上的努力後，

終於發現用統計模型來讓電腦理解語言才是正途，

這也是現在在搜尋引擎中所用的技術之一，

而且無關什麼語言或是什麼樣的句法，

都能夠用統計語言模型的方法來表示。

> 我想如果大學教授有跟我提這件事的話，

> 我現在可能就不會是個寫程式的人，會是個研究統計學的狂熱者了 XD

這堂課主要就是做初步的去探討怎麼樣把「文本資料」，

轉換成我們能夠做運算的模型，

進而讓電腦理解。

有了這些基礎後，才能更簡單的運用電腦擅於運算的特性，

讓我們更快速的運用這些從文本上挖掘到的知識。

> 這是一門專注在 shallow techniques 上面的課，

> 但內容可一點都不膚淺，後面會再提到何謂 shallow，何謂 deep。

# Overview

- 流程：

  1. Natural language processing & text representation

  2. Word association mining & analysis

  3. Topic mining & anaylysis

  4. Opinioon mining & sentiment analysis

  5. Text-based prediction

來看一下流程圖：

![flow](http://i.imgur.com/jobmiIH.png)

上述的流程中，

你會發現越前面的步驟其實需要越少人工介入，但沒辦法挖掘更深層的資訊，

越趨近於 shallow techniques。（不深嘛）

越後面則是越趨近於 deep techniques。（不淺嘛）

- Shallow techniques ，雖然得到的知識量比較少，但比較 general，不會因為領域不同被侷限，而且通常不需要或只需要相當少的人工介入（這門課裡面講的就是 shallow techniques）。

- Deep techniques 需要人工介入，而且有更多局限性，但能獲取到更多的知識。

- 結合前兩者加上機器學習才能得到 actionable information（知道這些知識後，我們可以採取某些行動或決策）

目前還沒有什麼兩全其美的方法，

在開始文本挖掘之前，建立好這個 trade-off 的概念是相當重要的。

# Text Representation

## Sequence of characters

```
A dog is chasing a boy on the playground
```

> 就是一連串的字元組合起來的字串，但對分析並沒有意義，

> 因為它沒有提供一個電腦能夠解析的結構。

## Sequence of words + POS(Part-Of-Speech) tags

> 這是這一門課主要探討的範圍，其他可以稍微看看就好 XD

> 這一層是所有 text mining 的基石，後續的分析和 mining 都能與此有所關連

為了得到更結構化的資料，我們可以試看看 POS(Part Of Speech) tags，

來為每個詞貼上一個 tag 看看，這個 tag 就是這個詞的詞性：

```
A: Det

Dog: N

is: Aux

chasing: Verb

a: Det

boy: N

on: PPrep

the: Det

playground: N
```


POS tag 的優劣分析

- Pros:

  - word 是人們溝通的基本單位

  - 有了這個結構之後我們能統計每個字出現的次數

  - 能夠被應用到 topic analysis 上

- Cons:
  
  - 不夠 general，像是中文就不是這樣認字。（中文字和中文字中間不會用空格分開）

  - 解法：中文會需要自己的斷詞（e.q: 結巴）

## Syntatic structures

- 一個表示語法的樹狀圖，這裡不會探討太多這部分，將會以統計模型表示語言的方式為主

## Entities and relations

- 拆出這個句子裡有哪些「實體」與「關係」

- e.q：我吃香蕉

這裡的實體就是`我`、`香蕉`，

而我們的關係是 `吃`與`被吃`。

## Logic predicates

- 寫成 logic predicate：

```
Dog(d1). Boy(b1). Playground(p1). Chasing(d1, b1, p1)
```

- 經由 logic predicates 我們能做到一些更智能的判斷，而且電腦也開始懂我們在說什麼了。

## Speech acts

- 瞭解這句話的「意圖」（intent）是什麼，光用想的就非常困難

- e.q：

  - 「假的！」這句話在海濤法師出來以前可能真的是指「這個東西不真實」，但現在卻不然

## Recap

- 越往下是越深層的分析和挖掘

  - 需要更多人工判斷

  - 精確度其實是降低的

  - 但是也越接近人類知識的表達

- 繼續研究下去之前我們得了解會有這樣的 trade-off

  - 所以要想辦法在人的參與和機器學習間做優化

    - 想想要怎樣讓人工的部分更簡單、省事

    - 利用上述的結果來得到更精確的學習結果、讓人工參與部分減少

這堂課主要就是以 shallow techniques 為主，

Shallow techniques 有以下幾個特點：

- General and robust

- No/little manual effort

- "Surprisingly" powerful

- Can be combined with more sophisticated representations

# Word Association Mining and Analysis

有了基本的文字探勘概念以後，

要先了解的就是詞和詞之間的關係，

並且把它們用數學的方式來表達。

## Outline

- 什麼是 word association

- 為什麼要挖掘 word association

- 如何挖掘 word association

## 什麼是 word association

簡單說就是詞跟詞之間的關係，

有以下兩種：

- Paradigmatic Relation（聚合關係）

  - `A`,`B`兩個詞可以互相替換，那兩者就有 paradigmatic relation

  - 從 similar context 去找: high context similarity => high paradigmatic relation

  - e.q: "cat" and "dog"

- Syntagmatic Relation（組合關係）

  - `A`,`B`兩個詞可以互相結合，那兩者就有 Syntagmatic relation

  - 從 correlated occurence 去找: high co-occurences but relatively low individual occurences => high syntagmatic relation

  - e.q: "cat" nad "sit", "car" and "drive"

## 為什麼要挖掘 word association

才能了解各個 document 間的關係，

容易去做 topic analysis，

要做更深層的分析前都得先做這一步。

## 挖掘 Paradigmatic Relation（聚合關係）

前面我們稍微理解了 Paradigmatic 以及 Syntagmatic 兩者的定義，

現在要來更深入的了解 Paradigmatic Relation。

首先要了解的概念是 Pseudo document，

讓我們用不同的方式來理解 context（上下文）。

Pseudo document，其實就是各式各樣的 bag of words，

裡面裝著各種「字」。

```
cat: 
  My ___ eats fish on Saturday.
  My ___ eats turkey on Sunday.
```

Pseudo document 的表示方式就是像下面敘述的這樣：

```
Left1("cat") = {"my", "his", "big", "a", "the",...}

Right1("cat") = {"eats", "ate", "is", "has", ....}

Window10("cat") = {"my", "his", "big", "eats", "fish", "Saturday", "Sunday"...}
```

第一項 `Left1("cat")` 就是 cat 左邊的一個字能放什麼，

`Right1` 則依此類推，

甚至我們也可以用 `Window10("cat")`來表示 10 個字裡面出現 cat 的 pseudo document，

> 沒錯，window 10 

所以 pseudo document 裡的字，

跟我們的目標字（這裡是 cat）相鄰或不相鄰都是可以的。

有了這些裝在袋子裡的字，我們就有了更堅實的基礎去比較 context 間的相似度。

以下用 Sim 這個 function 來表示 "Cat" 跟 "Dog" 的 context 相似度：

```
Sim("cat", "dog") =
  Sim(Left1("cat"), Left1("dog"))
    + Sim(Right1("cat"), Right1("dog")) +
    ...
    + Sim(Window8("cat"), Window8("dog"))
```

context 有著越高的相似度，代表 "cat" 跟 "dog" 兩個字越有聚合關係。

> 這裡並不是說一定要全部都算進去，我們也可以拿其中幾個袋子就好

而要選擇用哪些袋子來分析相似度，更是大大影響了我們分析的結果，

假如我們使用的是 `Window10("cat")`，這個相對較寬鬆的條件，

我們得到的是更 general 的訊息，可能能有更廣泛的應用；

假如我們使用的是 `Left1("cat")` 這樣的 context 時，

只會知道 "cat" 左邊會出現什麼詞

得到的可能會是更趨近語法分析上的訊息，相較更侷限一些。

### Vector space model

![vsm](http://i.imgur.com/7Pw5kex.png)

數學的美妙在於能把複雜的東西簡單化，

這裡我們很驚訝地發現能夠用 vector space 的方式來表現 context similarity

但問題來了，我們要怎樣去計算 `d1` 跟 `d2` 這兩個向量呢？

### Expected Overlap of Words in Context (EOWC)

其中一個相當直觀的方法是 Expected Overlap of Words in Context (EOWC)

> 我實在不會把這個翻譯成中文 XD，以下簡稱它為 EOWC，

> 名字中有著 expected，假如你還記得高三的統計學，

> 就可以猜的到它可能隱含著期望值的概念在裡面


```
c(w_i ,d1) = Count of word w_i in d1

|d1| = Total counts of words in d1 (d2 的部分就不重複了)

```

$$
d1=(x_1, ...x_N), \quad x_i = \frac{c(w_i ,d1)}{|d1|}
$$

$$
d2=(y_1, ...y_N), \quad y_i = \frac{c(w_i ,d2)}{|d2|}
$$


$$
sim(d1,d2) = d1.d2 = \sum_{i=1}^{N}(x_i \times y_i)
$$


x_i 會是每個字(w_i)在各自的 pseudo document 裡面出的次數（count），

再去除以 document 裡面的總次數。

我們得到的值就會是 w_i 這個字，出現在這個 pseudo document 中的頻率。

> 這是個 normalize 過的數值，所以將 x_1 加到 x_N 會是 1

兩個向量的 dot product 就會是 similarity。

> dot product 的算法是這樣子， a = (1,2,1), b = (3, 3, 3)

> a.b = 1*3+2*3+1*3 = 12

聽起來是蠻直觀的，

這代表兩個 document 中出現的詞相同頻率越高，

兩個 context 就越相近。

儘管 EOWC 給了我們對相似度相當直觀的感受，

還是要小心這裡面有兩個問題：

- 假如有其中一個詞的頻率在各個 document 出現次數超級高，那就會判定這兩個 context 有很高的相似度，就算其他詞重複頻率都很低也是一樣，這可能不會是我們想要的結果。 

  - 舉個很簡單的例子，假設藍綠兩派支持者互相叫囂，內文可能充斥著大量個幹幹幹幹，但彼此針對其攻擊對象的角度可能會很不同，但因為幹出現的頻率實在太高，所以 EOWC 會判定這兩個 context 有著高相似度。

- 對所有字都一視同仁。

  - 舉例來說 "the" 跟 "cat"， "cat"能帶給我們的資訊會比 "the" 來的多，但他們的權重卻是一樣的。

- 鑑於以上原因，我們再來看看有沒有什麼更好的方法。

### Improving EOWC with Retrieval Heuristics

兩個問題的解法：

- It favors matching one frequent term very well over matching more distinct terms.

  - Sublinear transformation of Term Frequency (TF)

- It treats every word equally (overlap on “the” isn’t as
so meaningful as overlap on “eats”).

  - Reward matching a rare word: IDF term weighting


先來看怎麼解決單一個詞擁有超高重複次數的問題，

TF transformation 其實就是把原本對應到的次數，轉換成另一個更合理的次數而已。

舉例來說：

我們可以把超過 1 的都 mapping 到 1 去， 0 的就是 0。

光用想的都知道上面這一個會丟失掉許多資料給我們的訊息，

所以接下來我們找到一個更合理的方式來做 TF transformation ，



叫做： BM25 Transformation。

> 小小科普一下，雖然 tf-idf 應該是大一就會的東西 XD

> BM25 通常指的是 Okapi BM25，是一個搜尋引擎中的 ranking function，

> BM 指的是 best match，它是由 BM11 以及 BM15 結合起來的，

> 不過起始點由 0 移至 1，所以從 26 變成了 25，

> 這就是 BM25 的由來。

> 還想更瞭解 BM11 和 BM15 的話可以看下方的補充資料：

> [The BM25 Weighting Scheme](http://xapian.org/docs/bm25.html)

假設輸入為 x，輸出為 y：

$$
y = BM25(x)
$$

更進一步看 BM25 這個算法的話：

$$
y = \frac{(k+1)x}{x+k}
$$

k 是我們可以自訂的數字，

可以簡單的看出來這個 y 最多無法超過 k+1，

如此便有效的將太多次數的詞都降到 k+1 了，

那問題就來了，k 要訂多少才合理呢？

暫且擱置這個問題，先來看原本 EOWC 的第二個問題：

> It treats every word equally (overlap on “the” isn’t as
so meaningful as overlap on “eats”).

太常在所有文章中都出現的字，它的重要性應該要降低，

而鮮少出現的字應該要被提高，我們用的方法則是 IDF。

```
W 是我們要分析的詞
M 為 documents 的總數
k 為 W 出現在 documents 中的次數
```

> 舉個小例子：

> 假設全部有 A, B, C 三個 documents，

> 而 W 出現在 A, B 兩個 documents，

情況會是 ：

```
W = W
M = 3
k = 2
```

再來看 IDF 的公式：

$$
IDF(W) = log(\frac{M+1}{k})
$$

> IDf = Inverse Document Frequency

> 原本是 字在所有 document 中出現的次數/ document 的總數

> IDF 是將其反過來看，所以稱為 inverse document frequency

可以看到，當 k 越大時， IDF 就會越小，

當 k 到達最大值（M），代表每個 document 都有出現 W ，

IDF(W)會趨近於零，讓 "the" "的" 這種每個 document 都可能會出現的字權重降低，

對於第二個問題而言，是一個相當簡單卻有效的方法。


最後，我們要將這兩個方法加進我們的 EOWC 中來優化它。

一口氣看這些公式會有點嚇人，

我偏好一步一步來理解它。

首先要來定義幾個符號：

$$d1=(x_1, ...x_N)$$

> d1 就是 pseudo document，

> 裡面的 x 是一個個的 word

$$b \in [0,1]$$

$$k \in [0,+\infty)$$

$$|d1| = length\,of\,d1$$

有了上述的定義後，先不要去深究他們有什麼意義，

因為要放在下面這個公式中，他們的定義才有用：

$$
BM25(w_i, d1) = \frac{(k+1)c(w_i, d1)}{c(w_i,d1)+k(1-b+b\times\frac{|d1|}{average(|d|)})}
$$

看似好像多了很多奇怪的符號，

但只要把上面那個 BM25 拿來對照一下：

$$
y = BM25() = \frac{(k+1)x}{x+k}
$$

就會發現其實它只是把 `c(w_i, d1)` 帶入 `x` 中，

`k` 仍然是拿來控制整個 tf 轉換過後的上界；

你唯一需要注意的新東西是 `b`，

$$
k(1-b+b\times\frac{|d1|}{average(|d|)})
$$

`average(|d|)`是為了 normalize 過長的 document，

我們希望不要因為不同長度的 document ，

得到差異過大的結果，

我的經驗是理解到他在做 normalize 之後，

就不要在糾結在上面太多，

就像很多時候我們取 log 就是真的讓圖比較平滑或好看而已（或者是在輸入是 1 的時候能得到 0 值）。

再來就看我們怎麼對 doument 裡面的每個 word 做 normalize 以及轉化：

$$
SUM =\sum^{N}_{j=1}(BM25(w_j, d1))
$$

$$
x_i = \frac{BM25(w_i, d1)}{SUM}
$$

$$
\sum^{N}_{i=1}x_i = 1
$$

分母是每個字的得分，分子則是該字的得分，

這樣做的好處是相加起來會等於 1，

裡面隱含著就是機率模型的概念。

> d2 的話就把 x 代換成 y，這裡不再重複寫一次

最後就可以來算 similarity 了：

$$
sim(d1,d2) = \sum_{i=1}^{N}(IDF\(w_i\) \times x_i \times y_i)
$$

新的算法中在一定程度上矯正了前面敘述到的兩個問題:

- tf => 將在單一個 doucment 中重複次數過高的字給 normalize

- idf => 將在每個 document 中都出現的字的權重給降低

因為算是中的三個元素都介於 0 到 1，

所以結果也會落在 0 跟 1 之間。

### One more thing: Syntagmatic Relations(optional)

這一小節會更加的抽象一點，最後來看個有趣的小知識，

BM25 其實也能應用在 syntagmatic relation 上面，

前面有說到組合關係能夠從 correlated occurence 去找：

$$
IDF-weighted \quad d_1=(x_1 \times IDF(w_1), ..., x_N \times IDF(w_N))
$$

原本的 x_i 只能表現他在同一個 docoument 裡面出現的頻率有多高，

（這裡是指經過 normalized BM25  transformation 的值）

但是我們不能說這個 x_i 與其對應的 w_i 是相關的（correlated），

因為有許多 common words（e.q: "the", "的") 也被包含在裡面，

但是我們前面有學過 idf 這個轉換法，

假如我們對每個 `x_i` 去乘上對應的 `idf(w_i)`，

就能得到去掉常用出現的字之後，真正與 w_i 相關的值了。

我第一時間看到這裡有點疑惑， 

Syntagmatic relation 不是從 word 同時出現的機率去挖掘的嗎？

假設我們要看 a 出現的話，b 同時出現的機率是多少，

其實我們得到每個 word 在每個 document 中出現的機率後，

就能算出在 a 字出現的條件下，b 也同時有出現的機率了，

就只是要算個條件機率而已 XD，相當的直觀！

> 這裡也只是我自己的推論，有錯的話也請不吝告知了

# Conclusion

總計六個禮拜的課程，目前是第一個禮拜。

其實寫筆記想要寫到其他人看得懂比上課來的吃力一些，

但同時帶給我的好處就是更深入了解自己所研究的內容，

雖然 tf-idf 是這樣簡單的東西，

但實際上了解它背後的概念，會發現它能運用的場合非常的廣泛。

Text mining 並不是一門太空人般的學問，

畢竟我們生活離不開文字的溝通、訊息的傳遞，

當你有能力去知識的載體上挖更多知識時，

何樂而不為呢？


# References

- [The BM25 Weighting Scheme](http://xapian.org/docs/bm25.html)

- 數學之美 - 吳軍

- [Coursera: Text Mining and Analytics 來自於 伊利諾伊大學香檳分校](https://www.coursera.org/learn/text-mining/home/welcome)
