{{{
"title": "Change Logs",
"date": "2016/1/1",
"intro": "紀錄一下這個小網站",
"tags": ["JavaScript"]
}}}

```ss
2016/1/11 完成初版，加上 disqus
```

底下是邊開發編寫的筆記
sss
TL;DR

- goal：生出一個可以拿 github page 當host，以後要搬家也很好搬的 blog

- 規劃：
    - document
        - readme
    - auto release 
        - git hook
        - test
        - compile if needed
        - export 到其他平台上, ex: medium
    - react router
    - 能夠讀取每個文章里的標頭檔案
    - 能夠 parse 標頭檔案來生成
    - 讀取 `posts/` 底下的文章（.md）
    - code highlighted => marked


#Document

# Auto Release

- Build head:
    - 使用 fs watch 和 

- 首先是讀取 markdown 檔案
    - 因為 webpack 的 hmr基本上會綁架 public path
    （）

# Fetch API

- 使用 webpack 的 shimming module 來做

```js
function doSomething(res) {
    ...
}
fetch(url)
    .then(doSomething)
    .then(result => result)

```

- 如何把資料拿出來呢？

- `then` 的語法看起來很像是我們熟悉的 promise，不過有沒有更好、更 high level 的語法來完成這件事情呢？

- [repo of fetch](https://github.com/github/fetch)

- Redux
    
    - 每個頁面都需要 connect 才能傳 props(至少目前這樣看起來最直觀)

- Markdown handling:
    
    - 讓 anchor tag 連到外部時能開一個新分頁

    - https://github.com/chjj/marked/pull/451

