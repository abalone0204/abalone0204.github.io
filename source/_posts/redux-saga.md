---
title: Saga Pattern 在前端的應用
tags:
  - redux
  - saga
  - redux-saga
date: 2016-05-14 19:02:18
desc: redux, JavaScript, redux-saga, saga
---

> 這是篇長文，你可以直接跳到[你想看的地方](/2016/05/14/redux-saga/#Catalogue)就好
> 或是直接在 github 上面看我 step by step 的教學
> [redux-thunk-to-saga-tutorial](https://github.com/abalone0204/redux-thunk-to-saga-tutorial/commit/1a4455b23ce6bc434d17a8c2ebcbf9e80e922be5)

先把結論講在一開始，這並不只是一個 library 的使用方法介紹而已，

因為學習 saga pattern 對於前端工程師是有幫助的，

主要不出以下三個概念：

- 好的 UI/UX 該是一個畫面的 transaction

- User 隨時能夠取消 transaction

- 滿足上述條件實作出來的資料流是要容易被測試的

那`redux-saga`到底是在解決什麼問題呢？

答案：

- 讓我們的非同步 action 能夠更好被開發、維護、測試。

- 讓我們用不同的方式來思考非同步的前端資料流

<!--more-->

![Adventure time](https://upload.wikimedia.org/wikipedia/zh/3/37/Adventure_Time_-_Title_card.png)

> saga 的中文翻譯是冒險故事 

> 這裡來舉個例子：我們要登入

```
送出登入 request =>
畫面進入 loading 畫面 =>
if (登入成功) {
    取得並把 token 快取起來 => 
    拿到`username`以及對應的`token` => 
    done
} else {
    顯示錯誤訊息在首頁上
    done
}
```

你會怎樣去設計這個資料流呢？

畫面要有什麼 state ？

假如登入要可以取消，你要怎樣改變畫面的 state 呢？

這個流程看似簡單，

但要處理的乾淨、又好測試，

是不是事情就沒有那麼直覺了？

目前看起來好像很抽象，但瞭解後，

`redux-saga` 並沒有什麼神奇的黑魔法。

我不認為 `redux-saga` 的只是拿來取代 `redux-thunk`的工具，

重要的應該是 saga 這個 pattern 背後的概念，

給了你新的方式去思考前端資料流。

`送出資料 => loading 動畫 => 完成`

其實前端的畫面也隱含著 transaction 的概念在裡面。

我認為如果有出現以下幾個現象，

那 `redux-saga` 值得你一試：

- 學會 generator function 卻無處可應用

- 處理非同步的 action 時，總覺得哪裡怪怪的 => 回傳 promise 時要怎麼測試

- 純粹好奇 `redux-saga`能幫助你什麼



# Catalogue

- [Introduction](#Introduction)

- [什麼是 Saga](#什麼是-Saga)
    
    - [Long lived transaction (LLT)有什麼問題](#Long-lived-transaction-LLT-有什麼問題)
    
    - [Saga 是一種特殊的 LLT](#Saga-是一種特殊的-LLT)

- [Front-end perspective (如果你懶得看理論的話可以直接從這裡開始看)](#Front-end-perspective)
    
    - [example: Login flow](#Login-flow)

    - [Redux thunk 的解法與問題](#Redux-thunk-的解法與問題)

    - [Front-end 中的 saga](#Front-end-中的-saga)

- [Refactor with `redux-saga`](#Refactor-with-redux-saga)

    - [Setup](#Setup)

    - [Effect](#Effect)

    - [Watch action](#Watch-action)

    - [Migrate Login Flow to saga](#Migrate-Login-Flow-to-saga)

    - [Combine `loginFlow` saga](#Combine-loginFlow-saga)

- [Abortable flow(compensating transaction)](#Abortable-flow-compensating-transaction)

    - [`fork` and `cancel`](#fork-and-cancel)

    - [Test for cancelable flow](#Test-for-cancelable-flow)

    - [Combine cancelable `loginFlow`](#Combine-cancelable-loginFlow)

    - [Conclusion](#Conclusion)



# Introduction

有些人會說 `redux-saga` 的學習曲線比較陡峭，

其實並不盡然。

會覺得 `redux-saga` 太過困難，

通常就是因為一次就想直接學會、並應用，

忽略有些預先知識必須要一步一步學習，

而且有些情況，必須拉高一點視角會比較好看清楚，

從概念的角度去看，而不是只關注在前端的實作。

我認為這裡只有三件事情要掌握

- 什麼是 saga？

- saga 跟前端開發有什麼關係？

- redux-saga 的基礎用法

# 什麼是 Saga

要學一個東西，把名詞搞懂是很重要的。

像 router 就是個很直覺又常見的名詞，

saga 是什麼呢？

`redux-saga` 有提供一些資源供參考，

包括了最原始提出 saga 這個 pattern 的[論文](http://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf)。

一共 11 頁，不過扣掉 acknowledgment 跟 References ，

就只有 9 頁半啦！

不過論文中是從 Database 的角度看，

另一個影片，是從應用在分散式系統的角度去解釋，

提高了不少複雜度。

基於以前端的角度，這篇講解 saga 主要會以 paper 上為主。

saga 其實是個很簡單的概念，

要應用它也並不困難，

這篇論文在 DBMS 上實作的原因，

主要只是要闡明如何實做一個簡潔、有效率的 sagas，

所以不要擔心接下來講的例子看起來跟 redux 或前端開發沒有關係，

稍後會提到要怎樣在前端開發中應用 saga 這個 pattern。

所以看個幾分鐘之後，腦袋裡會冒出許多的問號：「所以 saga 是⋯⋯？」。

這裡我試著用最簡單的語言解釋 saga 是什麼。

**Saga**，就是個滿足特殊條件的 **LLT**(Long lived transaction)。

> 待會會說是什麼特殊條件。

> 如果你不知道什麼是 Transaction：

> 是 Database 上常會用到（但不僅止侷限於 Database）的名詞，

> 即是「交易」。

> 「交易」聽起來很抽象，

> 其實他要敘述的就是銀貨兩訖後，

> 一個交易才算是完成，

> 假如銀貨不兩訖的話，那要退回最一開始的時候，

> 買賣雙方的狀態會退回交易前的狀態，不會有任何改變。

## Long lived transaction (LLT)有什麼問題

Long lived transaction 是什麼呢？

而 LLT 就是一個長時間的 transaction，

就算沒有受到其他影響，

整個完成可能也需要數小時或數天。

聽起來，似乎是很糟糕的概念對吧？

因為為了實現 transaction，我們通常會把正在 transaction 中的 object lock 住，

讓其他人沒辦法更動它。

（維持資料的 consistency）

所以這麼長時間的 transaction，

會造成兩個問題：

- 較高的失敗率

- dead lock 造成的長時間 delay

> 舉個很實際的例子，就是江蕙演唱會的訂票。

> 購票的時間可能會是某一段時間，

> 而我們最終要確認訂票的數，這就會是一個 LLT。


為解決這個問題，

我們這裡可以假設這個 LLT：`T`

可以被拆成許多相互獨立的 subtransaction的集合:
`t_1`~`t_n`。

但如果我們不會希望`t_1`~`t_n`分別被送進 DB 並且記錄下來。

> 以上述江蕙演唱會的例子，
> 每個小`t`就會是每筆訂票紀錄

如下圖：

![first state](http://i.imgur.com/sUodUqB.jpg)

假如每個 transaction 都一次就成功，

而且沒有人退票的話，那個 transaction 就會正常的被執行：

![all success](http://i.imgur.com/2P9E1wP.jpg)



因為假如有一個失敗的話，

那 `T` 就不算是完成的 transaction。

儘管如此，這樣做也比一般的 transaction 帶來了一些彈性，

我們可以隨意的插入 subtransaction。

接著就來解釋 saga 運用什麼樣的設計方式來解決這些問題。

## Saga 是一種特殊的 LLT

第一件要注意到的事就是 saga 仍然是個 LLT。

> `saga`: LLT that can be broken up into a collection of subtransactions that can be iterleaved in any way with other transactlons 

作為一個 LLT，

假如任何一個 saga 中的 subtransaction: `t_i` 單獨執行了，

我們應該要有一個 compensating transaction `c_i` 可以將它 undo。

這裡的 compensating transaction，

指的是從語意上的觀點來看，

而不是整個系統都得還原到 `t_i` 發生的那個時間點。

再看一次上面這段話，魔鬼就藏在細節裡，

這正是 saga 為什麼可以解決 LLT 問題的關鍵。

> 你可能會覺得這兩件事不是差不多嗎？

> 舉個例子：

> 如果有個 LLT : `T` 是要記住所有買江蕙票的座位數，

> 底下每個訂票都是一個 subtransaction: `t` 。

> 假設 `t_i` 要被買票的人取消，

> 我們執行 `c_i`時，

> 只是把買的座位數從 database 裡面減掉

> 而不是讓 database 回到 `t_i`發生前的時間點

所以我們可以得到一個簡單的公式，

Saga's gurantee：

- 如果全部都執行成功(Successful saga)：
    
    - `t_1`, `t_2`...., `t_n`

示意圖：

![success gif](http://i.imgur.com/RNCrTe0.gif)

- 失敗的話(Unsuccessful saga)：

    - `t_1`, `t_2`...., `t_n`, `c_n`..., `c_1`

![failed](http://i.imgur.com/thzgNNg.gif)

> 這裡可以注意到其實 `c4` 是沒有做任何事情的，

> 在實作時候如果是最後一個 transaction failed 掉的話，可以忽略 `c4`

> 不過就算執行了也不應該會出錯

> 因為每個執行應該都是 idempotent（冪等）的


如此一來我們就掌握了對 saga 的基本知識了！

在進入`redux-saga`前，先來看看我們會遇到什麼問題

# Front-end perspective

## Login flow

講了這麼多抽象概念的事情，

讓我們回到實務上來看，

來看最開始的這個例子：

```
送出登入 request =>
畫面進入 loading 畫面 =>
if (登入成功) {
    取得並把 token 快取起來 => 
    拿到`username`以及對應的`token` => 
    done
} else {
    顯示錯誤訊息在首頁上
    done
}
```

畫面出來大概是這樣：

![login flow](http://i.imgur.com/aWm0IqG.gif)

> 以下部分你可能必須要熟悉 `redux` ，

> 或是任何單向資料流的架構，

> 我盡量不預設讀者有任何預備知識來寫以下的文章 XD

> 不過真的不行的時候，會放上參考資料

在 redux 中，如果要改變畫面的狀態(state)，

我們必須 dispatch 一個 action 到 store 去，

而對應的 reducer 會根據 action 幫我們生出下一個 state，

並且將 store 中的 state 更新成對應的新 state。

> `reducer(state , action) => nextState`

> 假如還是很模糊的話，可以看看 redux 優秀的文件：

> [redux](http://redux.js.org/)

來看一下 `login` 的 reducer 會長什麼樣子：

> 這裡為了簡化，有刪去一些東西


```js
function login(state = {
    status: 'init'
}, action) {
    switch (action.type) {
        case LOGIN_REQUEST:
            return {status: 'loading'}
        case LOGIN_SUCCESS:
            return {
                status: 'logined',
                username: action.response.username,
                token: action.response.token
            }
        case LOGIN_ERROR:
            return {
                status: 'error',
                error: action.error
            }
        default:
            return state
    }
}
```

歸類成以下幾個結果：

- `LOGIN_REQUEST`：當我們送出`LOGIN_REQUEST`這個 action 時，會進入 loading 狀態

- `LOGIN_SUCCESS`：登入成功，會拿到 `username` 以及對應的 `token`

- `LOGIN_ERROR`：登入失敗，會拿到錯誤訊息

那真正執行的時候該如何執行呢？

## Redux thunk 的解法與問題

Thunk？Is it good to drink?

來看一下維基百科的解釋：

> In computer programming, **a thunk is a subroutine that is created, often automatically, to assist a call to another subroutine.**

只截錄一小段，剩下的多看也只是搞混。

簡單說就是我們為了把一個 subroutine A 的工作，

帶到另一個 subroutine B 做完，

中間需要一個橋樑：subroutine C，

這個 C 就是 thunk 啦！

在 redux 中，我們如果要讓一個 action 能夠更新，

必須要 dispatch 它。

所以上述的`login`流程大概會長這個樣子：

```js
function loginFlow({username, password}) {
    return (dispatch) => {
        dispatch(loginRequest())
        loginAPI({username, password})
            .then(response => {
                dispatch(loginSucess(response))    
            })
            .catch(error => {
                dispatch(loginError(error))
            })
    }
}
```

> `loginRequest` 是一個 action creator，
> 會回傳 `{type: LOGIN_REQUEST}`這個 object。

這裡回傳的就是一個 thunk，

因為我們在這個 action 裡面同時得完成：

- 送request

- 收到 response data

- 處理錯誤

所以我們必須把 dispatch 給傳進來，

完成原本只靠單個 subroutine(一般的 action creator) 無法做到的事情。

這裡有什麼問題呢？

- 你要如何去測試這個一連串的動作？

- 這裡回傳的是一個 promise，它無法被 abort，如果我們今天想加上取消按鈕呢？
    
    - 更 low level 一點的問法：你要在哪裡 dispatch `loginCancel`這個 action 呢？

當然， login 是一個相對簡易的流程，

假如遇到有更多 state 要處理，

無法寫出測試以及不那麼直覺的語法，

將會為我們的開發帶來一些問題。

## Front-end 中的 saga

這裡的一整個 `loginFlow`，其實就是一個 LLT(長時間的 transaction)，

> [Long lived transaction 是什麼？](#Long-lived-transaction-LLT-有什麼問題)

> 可以看完這一段再回到這裡 XD

底下的 subtransaction 就是各個 action(request, success, error)。

有了這樣的概念之後，剩下來的事就簡單多了。

而且 saga 就是底下每個 transaction 都附帶 compensating transaction 的 LLT，

也就是說上述的 abort ，在 saga pattern 之下是內建的。

# Refactor with `redux-saga`

## Setup

> 這裡跟概念比較沒關係，

> 但環境設定絕對是許多人卡關的第一步。

首先要建立一個 sagas 資料夾，

底下有一個 rootSaga，它會是一個 generator function：

```js
export default function* rootSaga() {
    yield [
        // to be done
    ]
}
```

接著在 middleware 中將它跑起來。

```js
import rootSaga from './sagas'

const sagaMiddleware = createSagaMiddleware()

const store = createStore(rootReducer,
  applyMiddleware(thunkMiddleware, sagaMiddleware)
)

sagaMiddleware.run(rootSaga)
```

這裡的基本設定，其實每次都大同小異，

所以就不再多著墨底下發生什麼事情。

## Effect

前面有提到的 subtransaction，可以很粗略的對應到這裡的 `effect`。

saga 不出以下幾種情形：

- 監聽 action 發生 -> take, takeEvery

- 執行 transaction -> put 

- 取消 transaction -> cancel

右邊的就是我們在 redux-saga 中對應到的 helper function，

他們就是 action creactor 一樣，會回傳一個物件，

不過這一次是回傳一個 effect ，而不是 action，

e.q: `take({type: LOGIN_REQUEST})` 就是產生一個拿到 loginRequest 的 effect。

接著就來把 code 改寫吧！

## Watch action

```js
import {
    takeEvery
} from 'redux-saga/effects'
import {
    LOGIN_REQUEST,
    LOGIN_SUCCESS,
    LOGIN_ERROR
} from '../actions/login.js'

export function* watchRequestLogin() {
    yield takeEvery(LOGIN_REQUEST, loginFlow)
}

export function* loginFlow() {
    // to be done
}
```

值得注意的是這裡都是 generator function，

假如你完全對 generator function 沒有概念的話，

推薦你看[這篇文章](http://abalone0204.github.io/2016/05/08/es6-generator-func/)。

> 是我寫的 XD

這裡的 code 還蠻語義化的，

就是當我們遇到一個 `LOGIN_REQUEST` 的 action ，

就會執行 `loginFlow` 這個 function。

接著是前面提到的好測試，

我們來測試這個 saga 吧！

```js
describe('Sagas/ login', () => {
    describe('watchRequestLogin', () => {
        const iterator = watchRequestLogin()
        it('should take every login request', () => {
            const expected = takeEvery(LOGIN_REQUEST, loginFlow)
            const actual = iterator.next().value
            assert.equal(expected.name, actual.name)
        })
    })
})
```

這裡比較 tricky 是我們測試的是 effect 的名字，

為什麼不是直接 deepEqual 兩個 effect？

我們回傳的 effect 其實就是個 object，長相是下面這樣：

```js
{ name: 'takeEvery(LOGIN_REQUEST, loginFlow)',
  next: [Function: next],
  throw: [Function] }
```

只要 name 是對的，我們就知道他在對應的 `LOGIN_REQUEST`進來時，

會執行`loginFlow` 這個 function。

> 而且在JavaScript中會判斷這兩個 next 是不同 function XD

> 直接測試名字，是我現在想到比較直觀的方法

## Migrate Login Flow to saga

Talk is cheap:

```js
export function* loginFlow(action) {
    try {
        const response = yield call(loginAPI, {
            username: action.username,
            password: action.password
        })
        yield put({type: LOGIN_SUCCESS})
    }
    catch(error) {
        yield put({type: LOGIN_ERROR, error})
    }
}
```

call 跟我們熟悉的 `Function.prototype.call` 很像！

不一樣的是，這裡的 call 會回傳的是一個 `effect`，

這代表什麼？代表我們能夠很好的測試它，

而不是真的去 call loginAPI，帶來了無止盡的 mock。

我們把 loginFlow 的 test 拆成四個部分來看

- Initialize

- Call loginAPI

- Handle login success

- Handle login error


前面的 watch function 會把 request 這個 action 丟進來這裡，

所以我們要先製造出一個待會會用到的 iterator：

> 執行 Generator function 會返回一個 iterator，
> 然後我們去對這個 iterator 呼叫 `next` function
> 感謝 CT 的指正。


```js
const iterator = loginFlow({
    type: LOGIN_REQUEST,
    username: 'denny',
    password: '12345678'
})
```

再來則是 call API，注意我們測試的是 call effect，

而不是真的去呼叫這個 API：

```js
it('should call loginAPI', () => {
    const expected = call(loginAPI, {
        username: 'denny',
        password: '12345678'
    })
    const actual = iterator.next().value
    assert.deepEqual(expected, actual)
})
```

```js
it('should handle login success', () => {
    const getResponse = () => ({
        username: 'denny',
        token: 'fake token'
    })
    const expected = put({
        type: LOGIN_SUCCESS,
        response: {
            username: 'denny',
            token: 'fake token'
        }
    })
    const actual = iterator.next(getResponse()).value
    assert.deepEqual(expected, actual)
})
```

這裡我們可以運用 generator 的特性來把假 error 丟進去XD

裡面的 catch 接到 error 之後，就會執行 login error 的流程了。

```js
it('should handle login error', () => {
    const error = 'error message'
    const expected = put({
        type: LOGIN_ERROR,
        error: 'error message'
    })
    const actual = generator.throw(error).value
    assert.deepEqual(expected, actual)
})
```

## Combine loginFlow saga

首先要把 login 的 saga 接到 root saga 去

接著我們要來把原本 dispatch 的 loginFlow action 換成 loginFlowSaga 了。

```js
import {watchRequestLogin} from './login.js'

export default function* rootSaga() {
    yield [
        watchRequestLogin()
    ]
}
```

再來我們只要把原本放 loginFlow action 的地方，

換成 `loginRequest` 這個相對簡單的 action creator 就行了。

這樣也更符合實際在運作的方式，

他按下這個按鈕做的 action 就只是送出 request 而已，

剩下的部分就是讓 saga 中的 generator 去管理，

而且經由這樣的拆分，我們發現接下來能夠實作 `cancel` 。

> 就是 saga 中的 compensating

這裡的 code 就請到 github 上面去看了 XD

總之我們得到了一樣的效果，但是更容易測試以及維護：

![login flow](http://i.imgur.com/aWm0IqG.gif)

# Abortable flow(compensating transaction) 

前面有說到要實作取消這個功能，

在 promise 中是很困難的，因為 promise 沒有辦法 abort。

不過我們活用 generator 的，就有辦法很直觀的實作出這個功能來。

首先當然是先做出 cancel 這個 action，

以及讓 reducer 根據這個 action 作出對應的改變。

完成了之後，接下來就是 saga 的重頭戲了。


## `fork` and `cancel`

首先我們要將原本的 loginFlow 拆分成兩部分，

第一部分是原本的 login 流程：

```js
function* authorize({username, password}){
    try {
        const response = yield call(loginAPI, {
            username,
            password
        })
        yield put({
            type: LOGIN_SUCCESS,
            response
        })
    } catch (error) {
        yield put({
            type: LOGIN_ERROR,
            error
        })
    }
}
```

第二部分則是取消 login：

```js
export function* loginFlow(action) {
    const task = yield fork(authorize,{username:action.username, password: action.password})
    yield take(LOGIN_CANCEL)
    yield cancel(task)
}
```

這裡我們看到兩個新的 effect，第一個是 fork，

語法基本上跟 call 相同，

不同的部分是 fork 跟我們在 git 上面的 fork 一樣會開一支 branch出來處理，

當 yield fork effect 之後，

就會自動開一條 branch 執行下去，這裡有個 @kuy 做的圖：

![process](https://pbs.twimg.com/media/CidrNh4UUAAJuSt.jpg)

而如果我們在上述 task 完成之前，就接收到了 `loginCancel` 這個 action，

那所有在 `task` 裡面的動作就會被 abort 掉！

> 是不是覺得有 race condition 的概念在裡面，
> 沒錯，`redux-saga`也提供了 `race` 這個 effect

## Test for cancelable flow

這裡一樣也測試以下幾件事情

- 是否有 fork 一個新的 task

- 是否能處理 cancel 這個 function

- 拆分出來的 authorize 是否正常運作

首先當然是先看進入 loginFlow 之後有沒有 fork ：

```js
it('should fork to authorize', () => {
    const expected = fork(authorize, {
        username: 'denny',
        password: '12345678'
    })
    const actual = iterator.next().value
    assert.deepEqual(expected, actual)
})
```

接下來是是否能處裡 cancel，

這裡我們就需要用到 mock 了，

在最外層的地方從 `redux-saga/utils` 引用 `createMockTask`：

```js
const task = createMockTask()
it('should take cancel login action',  () => {
    const expected = take(LOGIN_CANCEL)
    const actual = iterator.next(task).value
    assert.deepEqual(expected, actual)            
})

it('should cancel the login task',  () => {
    const expected = cancel(task)
    const actual = iterator.next().value
    assert.deepEqual(expected, actual)            
})
```

這裡仍然是運用了 generator 的特性來做 mock，

因為我們再隔一個動作才能取消 task，

所以在這之前我們要先把 mock 起來的 task 丟進去。

最後則是確認原本的 authorize 流程還是能正常運作，

基本上只是把原本的 test case 丟進另一個 describe 的 block 而已，

詳情可以去看 repo 裡的 code。

## Combine cancelable loginFlow

其實這裡蠻簡單的，

只是新增一個按鈕，按了會 dispatch`cancelLogin`這個action，

一切就結束了。 

像是底下這個樣子：

![cancel](http://i.imgur.com/CWhs8xi.gif)

# Conclusion

結論就是我們現在終於將 saga pattern 應用在前端了，

每一個好的 UX 都會是一個 transaction，

而且比起原本的論文中，我們多了一些彈性，

可以選擇要不要加上 compensating transiction。

如此一來我們的非同步 action 變得更好測試，

而且也不用擔心在每次處理過度複雜的資料流時，

沒有依據可找了，因為我們都是在組合各種 effect 而已XD

# 參考資料

- [sagas original paper by Hector Garcia-Molina & Kenneth Salem](http://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf)

- [redux-thunk-to-saga-tutorial](https://github.com/abalone0204/redux-thunk-to-saga-tutorial)
