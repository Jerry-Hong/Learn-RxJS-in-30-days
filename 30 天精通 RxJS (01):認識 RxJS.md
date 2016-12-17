30 天精通 RxJS (01)：認識 RxJS 
======

> RxJS 是筆者認為未來幾年內會非常紅的 Library，RxJS 提供了一套完整的非同步解決方案，讓我們在面對各種非同步行為，不管是 Event, AJAX, 還是 Animation 等，我們都可以使用相同的 API (Application Programming Interface) 做開發。

![RxJS Logo](https://raw.githubusercontent.com/Reactive-Extensions/RxJS/master/logos/logo.png)


這是【30天精通 RxJS】的 01 篇，如果還沒看過 00 篇可以往這邊走： 
[30 天精通 RxJS (00)： 關於本系列文章](http://ithelp.ithome.com.tw/articles/10186103)

在網頁的世界存取任何資源都是非同步(Async)的，比如說我們希望拿到一個檔案，要先發送一個請求，然後必須等到檔案回來，再執行對這個檔案的操作。這就是一個非同步的行為，而隨著網頁需求的複雜化，我們所寫的 JavaScript 就有各種針對非同步行為的寫法，例如使用 callback 或是 Promise 物件甚至是新的語法糖 async/await —— 但隨著應用需求愈來愈複雜，撰寫非同步的程式碼仍然非常困難。

非同步常見的問題
-------

- 競態條件 (Race Condition)
- 記憶體洩漏 (Memory Leak)
- 複雜的狀態 (Complex State)
- 例外處理 (Exception Handling)

### [Race Condition](https://goo.gl/GlNLYl)

每當我們對同一個資源同時做多次的非同步存取時，就可能發生 Race Condition 的問題。比如說我們發了一個 Request 更新使用者資料，然後我們又立即發送另一個 Request 取得使用者資料，這時第一個 Request 和第二個 Request 先後順序就會影響到最終接收到的結果不同，這就是 Race Condition。

### [Memory Leak](https://en.wikipedia.org/wiki/Memory_leak)

Memory Leak 是最常被大家忽略的一點。原因是在傳統網站的行為，我們每次換頁都是整頁重刷，並重新執行 JavaScript，所以不太需要理會記憶體的問題！但是當我們希望將網站做得像應用程式時，這件事就變得很重要。例如做 [SPA](https://en.wikipedia.org/wiki/Single-page_application) (Single Page Application) 網站時，我們是透過 JavaScript 來達到切換頁面的內容，這時如果有對 DOM 註冊監聽事件，而沒有在適當的時機點把監聽的事件移除，就有可能造成 Memory Leak。比如說在 A 頁面監聽 body 的 scroll 事件，但頁面切換時，沒有把 scroll 的監聽事件移除。

### Complex State

當有非同步行為時，應用程式的狀態就會變得非常複雜！比如說我們有一支付費用戶才能播放的影片，首先可能要先抓取這部影片的資訊，接著我們要在播放時去驗證使用者是否有權限播放，而使用者也有可能再按下播放後又立即按了取消，而這些都是非同步執行，這時就會各種複雜的狀態需要處理。

### Exception Handling

JavaScript 的 try/catch 可以捕捉同步的例外，但非同步的程式就沒這麼容易，尤其當我們的非同步行為很複雜時，這個問題就愈加明顯。

## 各種不同的 API

我們除了要面對非同步會遇到的各種問題外，還需要煩惱很多不同的 API

- DOM Events
- XMLHttpRequest
- fetch
- WebSockets
- Server Send Events
- Service Worker
- Node Stream
- Timer

上面列的 API 都是非同步的，但他們都有各自的 API 及寫法！如果我們使用 RxJS，上面所有的 API 都可以透過 RxJS 來處理，就能用同樣的 API 操作 (RxJS 的 API)。

這裡我們舉一個例子，假如我們想要監聽點擊事件(click event)，但點擊一次之後不再監聽。

**原生 JavaScript**

```javascript
var handler = (e) => {
	console.log(e);
	document.body.removeEventListener('click', handler); // 結束監聽
}

// 註冊監聽
document.body.addEventListener('click', handler);
```

**使用 Rx 大概的樣子**

```javascript
Rx.Observable
	.fromEvent(document.body, 'click') // 註冊監聽
	.take(1) // 只取一次
	.subscribe(console.log);
```

(點擊畫面後會在 console 顯示，記得打開 console 來看)

[JSbin](https://jsbin.com/vofaluv/4/edit?console,output) | [JSFiddle](https://jsfiddle.net/s6323859/d95a8peo/1/) 

大致上能看得出來我們在使用 RxJS 後，不管是針對 DOM Event 還是上面列的各種 API 我們都可以透過 RxJS 的 API 來做資料操作，像是範例中用 `take(n)` 來設定只取一次，之後就釋放記憶體。

說了這麼多，其實就是簡單一句話

**在面對日益複雜的問題，我們需要一個更好的解決方法。**

RxJS 基本介紹
------

RxJS 是一套藉由 **Observable sequences** 來組合**非同步行為**和**事件基礎**程序的 Library！

> 可以把 RxJS 想成處理 非同步行為 的 Lodash。

這也被稱為 Functional Reactive Programming，更切確地說是指 Functional Programming 及 Reactive Programming 兩個編程思想的結合。

> RxJS 確實是 Functional Programming 跟 Reactive Programming 的結合，但能不能稱為 Functional Reactive Programming (FRP) 一直有爭議。

> Rx 在[官網](http://reactivex.io/intro.html)上特別指出，有時這會被稱為 FRP 但這其實是個“誤稱”。

> 簡單說 FRP 是操作隨著時間**連續性改變的數值** 而 Rx 則比較像是操作隨著時間發出的**離散數值**，這個部份讀者不用分得太細，因為 FRP 的定義及解釋一直存在著歧異，也有眾多大神為此爭論，如下

> [André Staltz](https://medium.com/@andrestaltz/why-i-cannot-say-frp-but-i-just-did-d5ffaa23973b#.dhmsyic9w)：Rx 著名的推廣者，也是 RxJS 5 主要貢獻者之一，同時是 Cycle.js 的作者。Staltz 特別寫了一篇[文章](https://medium.com/@andrestaltz/why-i-cannot-say-frp-but-i-just-did-d5ffaa23973b#.dhmsyic9w)解釋為什麼 Rx 不能說是 FRP 但他仍然稱其為 FRP。

> [Juan Gomez](https://twitter.com/_juandg)：曾在 Netflix 工作，目前任職於 Fitbit，經常出現在國外演討會，主要寫 Android。Juan Gomez 在 [Droidcon NYC 2015 的演講](https://realm.io/news/droidcon-gomez-functional-reactive-programming/)中特別提出他堅持稱 Rx 為 FRP。

> [Evan Czaplicki](https://twitter.com/czaplic)：任職於 NoRedInk，Elm 的作者。Evan 在 [StrangeLoop 2014 的演講](https://www.youtube.com/watch?v=Agu6jipKfYw)中，特別為現在各種 FRP 的不同解釋做分類。

> 筆者自己的看法是比較偏向直接稱 Rx 為 FRP，原因是這較為直覺(FP + RP = FRP)，也比較不會對新手造成困惑，另外就是其他各種編程範式(包含 OOP, FP)其實都是**想法的集合，而非嚴格的指南(Guideline)**，我們應該更寬鬆的看待 FRP 而不是給他一個嚴格的定義。

### 關於 Reactive Extension (Rx)

Rx 最早是由微軟開發的 LinQ 擴展出來的開源專案，之後主要由社群的工程師貢獻，有多種語言支援，也被許多科技公司所採用，如 Netflix, Trello, Github, Airbnb...等。

#### Rx 的相關資訊

- 開源專案 (Apache 2.0 License)
- 多種語言支持
	- JavaScript
	- Java
	- C#
	- Python
	- Ruby
	- ...(太多了列不完)
- [官網](http://reactivex.io/)
- ~~微軟目前最成功的開源專案~~

> LinQ 唸做 Link，全名是 Language-Integrated Query，其功能很多元也非常強大；學 RxJS 可以不用會。

### Functional Reactive Programming

Functional Reactive Programming 是一種編程範式(programming paradigm)，白話就是一種**寫程式的方法論**！舉個例子，像 OOP 就是一種編程範式，OOP 告訴我們要使用物件的方式來思考問題，以及撰寫程式。而 Functional Reactive Programming 其實涵蓋了 Reactive Programming 及 Functional Programming 兩種編程思想。

**Functional Programming**

Functional Programming 大部分的人應該多少都有接觸過，這也是 Rx 學習過程中的重點之一，我們之後會花兩天的篇幅來細講 Functional Programming。
如果要用一句話來總結 Functional Programming，那就是 **用 function 來思考我們的問題，以及撰寫程式**

> 在下一篇文章會更深入的講解 Functional Programming

**Reactive Programming**

> 很多人一談到 Reactive Programming 就會直接聯想到是在講 RxJS，但實際上 Reactive Programming 仍是一種編程範式，在不同的場景都有機會遇到，而非只存在於 RxJS，尤雨溪(Vue 的作者)就曾在 twitter 對此表達不滿！

![Evan You 的推文](https://res.cloudinary.com/dohtkyi84/image/upload/v1480531809/%E8%9E%A2%E5%B9%95%E5%BF%AB%E7%85%A7_2016-11-30_%E4%B8%8A%E5%8D%886.23.49_zdniva.png)

Reactive Programming 簡單來說就是 **當變數或資源發生變動時，由變數或資源自動告訴我發生變動了**

這句話看似簡單，其實背後隱含兩件事

- 當發生變動 => 非同步：不知道什麼時候會發生變動，反正變動時要跟我說
- 由變數自動告知我 => 我不用寫通知我的每一步程式碼

由於最近很紅的 Vue.js 底層就是用 Reactive Programming 的概念實作，讓我能很好的舉例，讓大家理解什麼是 Reactive Programming！

當我們在使用 vue 開發時，只要一有綁定的變數發生改變，相關的變數及畫面也會跟著變動，而開發者不需要寫這其中如何**通知**發生變化的每一步程式碼，只需要**專注在發生變化時要做什麼事**，這就是典型的 Reactive Programming (記得必須是由變數或資源主動告知！)

> Vue.js 在做 two-ways data binding 是透過 ES5 definedProperty 的 getter/setter。每當變數發生變動時，就會執行 getter/setter 從而收集有改動的變數，這也被稱為**依賴收集**。

Rx 基本上就是上述的兩個觀念的結合，這個部份讀者在看完之後的文章，會有更深的體悟。


今日小結
------

今天這篇文章主要是帶大家了解為什麼我們需要 RxJS，以及 RxJS 的基本介紹。若讀者還不太能吸收本文的內容，可以過一段時間後再回來看這篇文章會有更深的體會，或是在下方留言給我！