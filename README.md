# RP入門

作者：[@andrestaltz](https://twitter.com/andrestaltz)

翻譯：[@benjycui](https://github.com/benjycui)、[@jsenjoy](https://github.com/jsenjoy)

正體中文轉換:[@gogobook](https://github.com/gogobook)

> 作者在[原文](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)後回答了不少人的疑惑，推薦一看。

> 在翻譯時，術語我儘量不翻譯，就算翻譯了也會給出原文以作對照。因為就個人觀察的情況而言，術語翻譯難以統一，不同的譯者會把同一個概念翻譯成不同的版本，最終只會讓讀者困惑。而且術語往往就幾個單詞，記起來也不難。

> 作者在回覆中提了一下FRP與RP是不同的，同時建議把這份教程中的FRP都替換為RP，所以譯文就把FRP都替換為RP了。

> gogobook>把英文附件更新，轉為正體中文並附上影片的連結。
### This tutorial as a series of videos

**If you prefer to watch video tutorials with live-coding, then check out this series I recorded with the same contents as in this article: [Egghead.io - Introduction to Reactive Programming](https://egghead.io/series/introduction-to-reactive-programming).**


很明顯你有興趣學習這種被稱作RP(Reactive Programming)的新技術，特別是它對應的實現如：Rx、Bacon.js、RAC等。

學習RP是很困難的一個過程，特別是在缺乏優秀資料的前提下。剛開始學習時，我試過去找一些教程，並找到了為數不多的實用教程，但是它們都流於表面，從沒有圍繞RP構建起一個完整的知識體系。庫的文檔往往也無法幫助你去瞭解它的函數。不信的話可以看一下這個：

> **Rx.Observable.prototype.flatMapLatest(selector, [thisArg])**

> ！@#￥%……&*

尼瑪。

我看過兩本書，一本只是講述了一些概念，而另一本則糾結於如何使用RP庫。我最終放棄了這種痛苦的學習方式，決定在開發中一邊使用RP，一邊理解它。在[Futurice](https://www.futurice.com)工作期間，我嘗試在真實項目中使用RP，並且當我遇到困難時，得到了[同事們的幫助](http://blog.futurice.com/top-7-tips-for-rxjava-on-android)。

在學習過程中最困難的一部分是 **以RP的方式思考**。這意味著要放棄命令式且帶狀態的(Imperative and stateful)編程習慣，並且要強迫你的大腦以一種不同的方式去工作。在互聯網上我找不到任何關於這方面的教程，而我覺得這世界需要一份關於怎麼以RP的方式思考的實用教程，這樣你就有足夠的資料去起步。庫的文檔無法為你的學習提供指引，而我希望這篇文章可以。

## 什麼是RP?

在互聯網上有著一大堆糟糕的解釋與定義。[維基百科](https://en.wikipedia.org/wiki/Functional_reactive_programming)一如既往的空泛與理論化。[Stackoverflow](http://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming)的權威答案明顯不適合初學者。[Reactive Manifesto](http://www.reactivemanifesto.org/)看起來是你展示給你公司的項目經理或者老闆們看的東西。微軟的[Rx terminology](https://rx.codeplex.com/) "Rx = Observables + LINQ + Schedulers" 過於重量級且微軟味十足，只會讓大部分人困惑。相對於你所使用的MV*框架以及鍾愛的編程語言，"Reactive"和"Propagation of change"這些術語並沒有傳達任何有意義的概念。框架的Views層當然要對Models層作出反應，改變當然會傳播(分別對應上文的"Reactive"與"Propagation of change"，意思是這一大堆術語和廢話差不多，翻譯不好，只能靠備註了)。如果沒有這些，就沒有東西會被渲染了。

所以不要再扯這些廢話了。

### RP是使用異步數據流進行編程

一方面，這並不是什麼新東西。Event buses或者Click events本質上就是異步事件流(Asynchronous event stream)，你可以監聽並處理這些事件。RP的思路大概如下：你可以用包括Click和Hover事件在內的任何東西創建Data stream(原文："FRP is that idea on steroids. You are able to create data streams of anything, not just from click and hover events.")。Stream廉價且常見，任何東西都可以是一個Stream：變量、用戶輸入、屬性、Cache、數據結構等等。舉個例子，想像一下你的Twitter feed就像是Click events那樣的Data stream，你可以監聽它並相應的作出響應。

**在這個基礎上，你還有令人驚豔的函數去combine、create、filter這些Stream。**這就是函數式(Functional)魔法的用武之地。Stream能接受一個，甚至多個Stream為輸入。你可以_merge_兩個Stream，也可以從一個Stream中_filter_出你感興趣的Events以生成一個新的Stream，還可以把一個Stream中的Data values _map_到一個新的Stream中。

既然Stream在RP中如此重要，那麼我們就應該好好的瞭解它們，就從我們熟悉的"Clicks on a button" Event stream開始。

![Click event stream](http://i.imgur.com/cL4MOsS.png)

Stream就是一個 **按時間排序的Events(Ongoing events ordered in time)序列** ，它可以emit三種不同的Events：(某種類型的)Value、Error或者一個"Completed" Signal。考慮一下"Completed"發生的時機，例如，當包含這個Button(指上面Clicks on a button"例子中的Button)的Window或者View被關閉時。

通過分別為Value、Error、"Completed"定義事件處理函數，我們將會異步地捕獲這些Events。有時可以忽略Error與"Completed"，你只需要定義Value的事件處理函數就行。監聽一個Stream也被稱作是 **訂閱(Subscribing)**，而我們所定義的函數就是觀察者(Observer)，Stream則是被觀察者(Observable)，其實就是[觀察者模式(Observer Design Pattern)](https://en.wikipedia.org/wiki/Observer_pattern)。

上面的示意圖也可以使用ASCII重畫為下圖，在下面的部分教程中我們會使用這幅圖：

```
--a---b-c---d---X---|->

a, b, c, d are emitted values
X is an error
| is the 'completed' signal
---> is the timeline
```

既然已經開始對RP感到熟悉，為了不讓你覺得無聊，我們可以嘗試做一些新東西：我們將會把一個Click event stream轉為新的Click event stream。

首先，讓我們做一個能記錄一個按鈕點擊了多少次的計數器Stream。在常見的RP庫中，每個Stream都會有多個方法，`map`、`filter`、`scan`等等。當你調用其中一個方法時，例如`clickStream.map(f)`，它就會基於原來的Click stream返回一個 **新的Stream**。它不會對原來的Click steam作任何修改。這個特性就是 **不可變性(Immutability)**，它之於RP Stream，就如果汁之於薄煎餅。我們也可以對方法進行鏈式調用如`clickStream.map(f).scan(g)`：

```
  clickStream: ---c----c--c----c------c-->
               vvvvv map(c becomes 1) vvvv
               ---1----1--1----1------1-->
               vvvvvvvvv scan(+) vvvvvvvvv
counterStream: ---1----2--3----4------5-->
```

`map(f)`會根據你提供的`f`函數把原Stream中的Value分別映射到新的Stream中。在我們的例子中，我們把每一次Click都映射為數字1。`scan(g)`會根據你提供的`g`函數把Stream中的所有Value聚合成一個Value -- `x = g(accumulated, current)`，這個示例中`g`只是一個簡單的add函數。然後，每Click一次，`counterStream`就會把點擊的總次數發給它的觀察者。

為了展示RP真正的實力，讓我們假設你想得到一個包含雙擊事件的Stream。為了讓它更加有趣，假設我們想要的這個Stream要同時考慮三擊(Triple clicks)，或者更加寬泛，連擊(Multiple clicks)。深呼吸一下，然後想像一下在傳統的命令式且帶狀態的方式中你會怎麼實現。我敢打賭代碼會像一堆亂麻，並且會使用一些的變量保存狀態，同時也有一些計算時間間隔的代碼。

而在RP中，這個功能的實現就非常簡單。事實上，這邏輯只有[4行代碼](http://jsfiddle.net/staltz/4gGgs/27/)。但現在我們先不管那些代碼。用圖表的方式思考是理解怎樣構建Stream的最好方法，無論你是初學者還是專家。

![Multiple clicks stream](http://i.imgur.com/HMGWNO5.png)

灰色的方框是用來轉換Stream的函數。首先，我們把時間間隔大於250ms的Click都放進一個列表(原文："First we accumulate clicks in lists, whenever 250 milliseconds of "event silence" has happened." ) -- 簡單來說就是`buffer(stream.throttle(250ms))`做的事，不要在意這些細節，我們只是展示一下RP而已。結果是一個列表的Stream，然後我們使用`map()`把每個列表映射為一個整數，即它的長度。最終，我們使用`filter(x >= 2)`把整數`1`給過濾掉。就這樣，3個操作就生成了我們想要的Stream。然後我們就可以訂閱(監聽)這個Stream，並以我們所希望的方式作出反應。

我希望你能感受到這個示例的優美之處。這個示例只是冰山一角：你可以把同樣的操作應用到不同種類的Stream上，例如，一個API響應的Stream；另一方面，還有很多其它可用的函數。

## 為什麼我要使用RP

RP提高了代碼的抽象層級，所以你可以只關注定義了業務邏輯的那些相互依賴的事件，而非糾纏於大量的實現細節。RP的代碼往往會更加簡明。

特別是在開發現在這些有著大量與Data events相關的UI events的高互動性Webapps、Mobile apps的時候，RP的優勢將更加明顯。10年前，網頁的交互就只是提交一個很長的表單到後端，而前端只有簡單的渲染。Apps就表現得更加的實時了：修改一個表單域就能自動地把修改後的值保存到後端，為一些內容"點贊"時，會實時的反應到其它在線用戶那裡等等。

現在的Apps有著大量各種各樣的實時Events，以給用戶提供一個交互性較高的體驗。我們需要工具去應對這個變化，而RP就是一個答案。

## 以RP方式思考的例子

讓我們做一些實踐。一個真實的例子一步一步的指導我們以RP的方式思考。不是虛構的例子，也沒有只解釋了一半的概念。學完教程之後，我們將寫出真實可用的代碼，並做到知其然，知其所以然。

在這個教程中，我將會使用 **JavaScript** 和 **[RxJS](https://github.com/Reactive-Extensions/RxJS)**，因為JavaScript是現在最多人會的語言，而[Rx* 庫](https://rx.codeplex.com/)有多種語言版本，並支持多種平台([.NET](https://rx.codeplex.com/), [Java](https://github.com/Netflix/RxJava), [Scala](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-scala), [Clojure](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-clojure),  [JavaScript](https://github.com/Reactive-Extensions/RxJS), [Ruby](https://github.com/Reactive-Extensions/Rx.rb), [Python](https://github.com/Reactive-Extensions/RxPy), [C++](https://github.com/Reactive-Extensions/RxCpp), [Objective-C/Cocoa](https://github.com/ReactiveCocoa/ReactiveCocoa), [Groovy](https://github.com/Netflix/RxJava/tree/master/language-adaptors/rxjava-groovy), 等等)。所以，無論你用的是什麼語言、庫，你都能從下面這個教程中學到東西。

## 實現"Who to follow"推薦界面

在Twitter上，這個界面看起來是這樣的：

![Twitter Who to follow suggestions box](http://i.imgur.com/eAlNb0j.png)

我們將會重點模擬它的核心功能，如下：

* 啟動時從API那裡加載帳戶數據，並顯示3個推薦
* 點擊"Refresh"時，加載另外3個推薦用戶到這三行中
* 點擊帳號所在行的'x'按鈕時，清除那個推薦然後顯示一個新的推薦
* 每行都會顯示帳號的頭像，以及他們主頁的鏈接

我們可以忽略其它的特性和按鈕，因為它們是次要的。同時，因為Twitter最近關閉了對非授權用戶的API，我們將會為Github實現這個推薦界面，而非Twitter。這是[Github獲取用戶的API](https://developer.github.com/v3/users/#get-all-users)。

如果你想先看一下最終效果，這裡有完成後的代碼 http://jsfiddle.net/staltz/8jFJH/48/ 。

## Request與Response

**在Rx中你該怎麼處理這個問題呢？** 好吧，首先，(幾乎)_所有的東西都可以轉為一個Stream_。這就是Rx的咒語。讓我們先從最簡單的特性開始："在啟動時，從API加載3個帳號的數據"。這並沒有什麼特別，就只是簡單的(1)發出一個請求，(2)收到一個響應，(3)渲染這個響應。所以，讓我們繼續，並用Stream代表我們的請求。一開始可能會覺得殺雞用牛刀，但我們應當從最基本的開始，是吧？

在啟動的時候，我們只需要發出一個請求，所以如果我們把它轉為一個Data stream的話，那就是一個只有一個Value的Stream。稍後，我們知道將會有多個請求發生，但現在，就只有一個請求。

```
--a------|->

a是一個String 'https://api.github.com/users'
```

這是一個包含了我們想向其發出請求的URL的Stream。每當一個請求事件發生時，它會告訴我們兩件事："什麼時候"與"什麼東西"。"什麼時候"這個請求會被執行，就是什麼時候這個Event會被emit。"什麼東西"會被請求，就是這個emit出來的Value：一個包含URL的String。

在RX*中，創建只有一個Value的Stream是非常簡單的。官方把一個Stream稱作Observable，因為它可以被觀察(can be observed => observable)，但是我發現那是個很傻逼的名子，所以我把它叫做_Stream_。

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');
```

但是現在，那只是一個包含了String的Stream，並沒有什麼特別，所以我們需要以某種方式使Value被emit。就是通過[訂閱(Subscribing)](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypesubscribeobserver--onnext-onerror-oncompleted)這個Stream。

```javascript
requestStream.subscribe(function(requestUrl) {
  // execute the request
  jQuery.getJSON(requestUrl, function(responseData) {
    // ...
  });
}
```

留意一下我們使用了jQuery的Ajax函數(我們假設你已經知道[它的用途](http://devdocs.io/jquery/jquery.getjson))去發出異步請求。但先等等，Rx可以用來處理 **異步** Data stream。那這個請求的響應就不能當作一個包含了將會到達的數據的Stream麼？當然，從理論上來講，應該是可以的，所以我們嘗試一下。

```javascript
requestStream.subscribe(function(requestUrl) {
  // execute the request
  var responseStream = Rx.Observable.create(function (observer) {
    jQuery.getJSON(requestUrl)
    .done(function(response) { observer.onNext(response); })
    .fail(function(jqXHR, status, error) { observer.onError(error); })
    .always(function() { observer.onCompleted(); });
  });

  responseStream.subscribe(function(response) {
    // do something with the response
  });
}
```

[`Rx.Observable.create()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservablecreatesubscribe)所做的事就是通過顯式的通知每一個Observer(或者說是Subscriber) Data events(`onNext()`)或者Errors (`onError()`)來創建你自己的Stream。而我們所做的就只是把jQuery Ajax Promise包裝起來而已。**打擾一下，這意味者Promise本質上就是一個Observable？**

&nbsp;
&nbsp;
&nbsp;
&nbsp;
&nbsp;

![Amazed](http://www.myfacewhen.net/uploads/3324-amazed-face.gif)

Yes.

Observable就是Promise++。在Rx中，你可以用`var stream = Rx.Observable.fromPromise(promise)`輕易的把一個Promise轉為Observable，所以我們就這樣子做吧。唯一的不同就是Observable並不遵循[Promises/A+](http://promises-aplus.github.io/promises-spec/)，但概念上沒有衝突。Promise就是只有一個Value的Observable。Rx Stream比Promise更進一步的是允許返回多個Value。

這樣非常不錯，並展現了Observables至少有Promise那麼強大。所以如果你相信Promise宣傳的那些東西，那麼也請留意一下Rx Observables能勝任些什麼。

現在回到我們的例子，如果你已經注意到了我們在`subscribe()`內又調用了另外一個`subscribe()`，這類似於Callback hell。同樣，你應該也注意到`responseStream`是建立在`requestStream`之上的。就像你之前瞭解到的那樣，在Rx內有簡單的機制可以從其它Stream中轉換並創建出新的Stream，所以我們也應該這樣子做。

你現在需要知道的一個基本的函數是[`map(f)`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemapselector-thisarg)，它分別把`f()`應用到Stream A中的每一個Value，並把返回的Value放進Stream B裡。如果我們也對Request Stream與Response Stream進行同樣的處理，我們可以把Request URL映射(map)為Response Promise(而Promise可以轉為Streams)。

```javascript
var responseMetastream = requestStream
  .map(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

然後，我們將會創造一個叫做"_Metastream_"的怪物：包含Stream的Stream。暫時不需要害怕。Metastream就是emit的每個Value都是Stream的Stream。你可以把它想像為[指針(Pointer)](https://en.wikipedia.org/wiki/Pointer_(computer_programming))：每個Value都是一個指向其它Stream的指針。在我們的例子裡，每個Request URL都會被映射(map)為一個指向包含響應Promise stream的指針。

![Response metastream](http://i.imgur.com/HHnmlac.png)

Response的Metastream看起來會讓人困惑，並且看起來也沒有幫到我們什麼。我們只想要一個簡單的Response stream，它返回的Value應該是JSON而不是一個JSON物件的'Promise'。是時候介紹[Mr. Flatmap](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypeflatmapselector-resultselector)了：它是`map()`的一個版本，通過把應用到"trunk" Stream上的所有操作都應用到"branch" Stream上，可以"flatten" Metastream。Flatmap並不是用來"fix" Metastream的，因為Metastream也不是一個Bug，這只是一些用來處理Rx中的異步響應(Asynchronous response)的工具。

```javascript
var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });
```

![Response stream](http://i.imgur.com/Hi3zNzJ.png)

很好。因為Response stream是根據Request stream定義的，所以**如果**我們後面在Request stream上發起更多的請求的話，在Response stream上我們將會得到相應的Response event，就像預期的那樣：

```
requestStream:  --a-----b--c------------|->
responseStream: -----A--------B-----C---|->

(小寫字母是一個Request，大寫字母是對應的Response)
```

現在，我們終於有了一個Response stream，所以可以把收到的數據渲染出來了：

```javascript
responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

把目前為止所有的代碼放到一起就是這樣：

```javascript
var requestStream = Rx.Observable.just('https://api.github.com/users');

var responseStream = requestStream
  .flatMap(function(requestUrl) {
    return Rx.Observable.fromPromise(jQuery.getJSON(requestUrl));
  });

responseStream.subscribe(function(response) {
  // render `response` to the DOM however you wish
});
```

## Refresh按鈕

我之前並沒有提到返回的JSON是一個有著100個用戶數據的列表。因為這個API只允許我們設置偏移量(Offset)，而無法設置返回的用戶數，所以我們現在是只用了3個用戶的數據而浪費了另外97個的數據。這個問題暫時可以忽略，稍後我們會學習怎麼緩存這些數據。

每點擊一次Refresh按鈕，Request stream就會emit一個新的URL，同時也會返回一個新的Response。我們需要兩樣東西：一個是Refresh按鈕上Click events組成的Stream(咒語：一切皆Stream)，而Request stream將改為隨Refresh click stream作出反應。幸運的是，RxJS提供了從Event listener生成Observable的函數。

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');
```

既然Refresh click event本身並沒有提供任何要請求的API URL，我們需要把每一次的Click都映射為一個URL。現在，我們把Refresh click stream映射為新的Request stream，其中每一個Click都分別映射為對API請求一個隨機偏移量的URL。

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

因為我比較笨並且也沒有使用自動化測試，所以我剛把之前做好的一個特性搞爛了。現在在啟動時不會再發出任何的Request，而只有在點擊Refresh按鈕時才會。額...這兩個行為我都需要：無論是點擊Refresh按鈕時還是剛打開頁面時都該發出一個Request。

我們知道怎麼分別為這兩種情況生成Stream：

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var startupRequestStream = Rx.Observable.just('https://api.github.com/users');
```

但我們怎樣才能把這兩個"合成(merge)"一個呢？好吧，有[`merge()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypemergemaxconcurrent--other)函數。這就是它做的事的圖解：

```
stream A: ---a--------e-----o----->
stream B: -----B---C-----D-------->
          vvvvvvvvv merge vvvvvvvvv
          ---a-B---C--e--D--o----->
```

這樣就簡單了：

```javascript
var requestOnRefreshStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var startupRequestStream = Rx.Observable.just('https://api.github.com/users');

var requestStream = Rx.Observable.merge(
  requestOnRefreshStream, startupRequestStream
);
```

還有一個更加乾淨的可選方案，不需要使用中間變量。

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .merge(Rx.Observable.just('https://api.github.com/users'));
```

甚至可以更短，更具有可讀性：

```javascript
var requestStream = refreshClickStream
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  })
  .startWith('https://api.github.com/users');
```

[`startWith()`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypestartwithscheduler-args)函數做的事和你預期的完全一樣。無論你輸入的Stream是怎樣，`startWith(x)`輸出的Stream一開始都是`x`。但是還不夠[DRY](https://en.wikipedia.org/wiki/Don't_repeat_yourself)，我重複了API URL。一個改進的方法是移掉`refreshClickStream`最後的`startWith()`，並在一開始的時候"emulate"一次Click。

```javascript
var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

很好。如果你把之前我"搞爛了的版本"的代碼和現在的相比，就會發現唯一的不同是加了`startWith()`函數。

## 用Stream構建三個推薦

到現在為止，我們只是談及了這個 _推薦_ UI元素在responeStream的`subscribe()`內執行的渲染步驟。對於Refresh按鈕，我們還有一個問題：當你點擊`Refresh`時，當前存在的三個推薦並不會被清除。新的推薦會在Response到達後出現，為了讓UI看起來舒服一些，當點擊刷新時，我們需要清理掉當前的推薦。

```javascript
refreshClickStream.subscribe(function() {
  // clear the 3 suggestion DOM elements
});
```

不，別那麼快，朋友。這樣不好，我們現在有 **兩個** Subscriber會影響到推薦的DOM元素(另外一個是`responseStream.subscribe()`)，而且這樣完全不符合[Separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns)。還記得RP的咒語麼？

&nbsp;
&nbsp;
&nbsp;
&nbsp;

![Mantra](http://i.imgur.com/AIimQ8C.jpg)

所以讓我們把顯示的推薦設計成emit的值為一個包含了推薦內容的JSON物件的Stream。我們以此把三個推薦內容分開來。現在第一個推薦看起來是這樣子的：

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  });
```

其他的，`suggestion2Stream`和`suggestion3Stream`可以簡單的拷貝`suggestion·Stream`的代碼來使用。這不是DRY，它會讓我們的例子變得更加簡單一些，加之我覺得這是一個可以幫助考慮如何減少重複的良好實踐。

我們不在responseStream的subscribe()中處理渲染了，我們這麼處理：

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  // render the 1st suggestion to the DOM
});
```
回到"當刷新時，清理掉當前的推薦"，我們可以很簡單的把刷新點擊映射為`null`(即沒有推薦數據)，並且在`suggestion1Stream`中包含進來，如下：

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  );
```

當渲染時，`null`解釋為"沒有數據"，所以把UI元素隱藏起來。

```javascript
suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
  }
});
```

現在的示意圖：

```
refreshClickStream: ----------o--------o---->
     requestStream: -r--------r--------r---->
    responseStream: ----R---------R------R-->
 suggestion1Stream: ----s-----N---s----N-s-->
 suggestion2Stream: ----q-----N---q----N-q-->
 suggestion3Stream: ----t-----N---t----N-t-->
```

`N`即代表了`null`

作為一種補充，我們也可以在一開始的時候就渲染空的推薦內容。這通過把`startWith(null)`添加到Suggestion stream就完成了：

```javascript
var suggestion1Stream = responseStream
  .map(function(listUsers) {
    // get one random user from the list
    return listUsers[Math.floor(Math.random()*listUsers.length)];
  })
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

現在結果是：

```
refreshClickStream: ----------o---------o---->
     requestStream: -r--------r---------r---->
    responseStream: ----R----------R------R-->
 suggestion1Stream: -N--s-----N----s----N-s-->
 suggestion2Stream: -N--q-----N----q----N-q-->
 suggestion3Stream: -N--t-----N----t----N-t-->
```

## 關閉推薦並緩存Response

還有一個功能需要實現。每一個推薦，都該有自己的"X"按鈕以關閉它，然後在該位置加載另一個推薦。最初的想法，是點擊任何關閉按鈕時都需要發起一個新的請求：

```javascript
var close1Button = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(close1Button, 'click');
// and the same for close2Button and close3Button

var requestStream = refreshClickStream.startWith('startup click')
  .merge(close1ClickStream) // we added this
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });
```

這個沒有效果。這將會關閉並且重新加載_所有_的推薦，而不是僅僅處理我們點擊的那一個。有一些不一樣的方法可以解決，並且讓它變得更加有趣，我們可以通過復用之前的請求來解決它。API的Response有100個用戶，而我們僅僅使用其中的三個，所以還有很多的新數據可以使用，無須重新發起請求。

同樣的，我們用Stream的方式來思考。當點擊'close1'時，我們想要從responseStream _最近emit的_ Response列表中獲取一個隨機的用戶，如：

```
    requestStream: --r--------------->
   responseStream: ------R----------->
close1ClickStream: ------------c----->
suggestion1Stream: ------s-----s----->
```

在Rx*中，[`combineLatest`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md#rxobservableprototypecombinelatestargs-resultselector)似乎實現了我們想要的功能。它接受兩個Stream，A和B作為輸入，當其中一個Stream emit一個值時，`combineLatest`把最近兩個emit的值`a`和`b`從各自的Stream中取出並且返回一個`c = f(x,y)`，`f`為你定義的函數。用圖來表示更好：

```
stream A: --a-----------e--------i-------->
stream B: -----b----c--------d-------q---->
          vvvvvvvv combineLatest(f) vvvvvvv
          ----AB---AC--EC---ED--ID--IQ---->

f是把值轉化成大寫字母的函數
```

我們可以在`close1ClickStream`和`responseStream`上使用combineLatest()，所以無論什麼時候當一個按鈕被點擊時，我們可以拿到Response最新emit的值，並且在`suggestion1Stream`上產生一個新的值。另一方面，combineLatest()是對稱的，當一個新的Response 在`responseStream` emit時，它將會把最後的'關閉 1'的點擊事件一起合併來產生一個新的推薦。這是有趣的，因為它允許我們把之前的`suggestion1Stream`代碼簡化成下邊這個樣子：

```javascript
var suggestion1Stream = close1ClickStream
  .combineLatest(responseStream,
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

還有一個問題需要解決。combineLatest()使用最近的兩個數據源，但是當其中一個來源沒發起任何事件時，combineLatest()無法在Output stream中產生一個Data event。從上邊的ASCII圖中，你可以看到，在第一個Stream emit `a`這個值時並沒有任何輸出產生，只有當第二個Stream emit `b`時才有值輸出。

有多種方法可以解決這個問題，我們選擇最簡單的一種，一開始在'close 1'按鈕上模擬一個點擊事件：

```javascript
var suggestion1Stream = close1ClickStream.startWith('startup click') // we added this
  .combineLatest(responseStream,
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
```

## 總結

終於完成了，所有的代碼合在一起是這樣子：

```javascript
var refreshButton = document.querySelector('.refresh');
var refreshClickStream = Rx.Observable.fromEvent(refreshButton, 'click');

var closeButton1 = document.querySelector('.close1');
var close1ClickStream = Rx.Observable.fromEvent(closeButton1, 'click');
// and the same logic for close2 and close3

var requestStream = refreshClickStream.startWith('startup click')
  .map(function() {
    var randomOffset = Math.floor(Math.random()*500);
    return 'https://api.github.com/users?since=' + randomOffset;
  });

var responseStream = requestStream
  .flatMap(function (requestUrl) {
    return Rx.Observable.fromPromise($.ajax({url: requestUrl}));
  });

var suggestion1Stream = close1ClickStream.startWith('startup click')
  .combineLatest(responseStream,
    function(click, listUsers) {
      return listUsers[Math.floor(Math.random()*listUsers.length)];
    }
  )
  .merge(
    refreshClickStream.map(function(){ return null; })
  )
  .startWith(null);
// and the same logic for suggestion2Stream and suggestion3Stream

suggestion1Stream.subscribe(function(suggestion) {
  if (suggestion === null) {
    // hide the first suggestion DOM element
  }
  else {
    // show the first suggestion DOM element
    // and render the data
  }
});
```

**你可以查看這個最終效果 http://jsfiddle.net/staltz/8jFJH/48/**

這段代碼雖然短小，但實現了不少功能：它適當的使用Separation of concerns實現了對Multiple events的管理，甚至緩存了響應。函數式的風格讓代碼看起來更加Declarative而非Imperative：我們並非給出一組指令去執行，而是通過定義Stream之間的關係 **定義這是什麼**。舉個例子，我們使用Rx告訴計算機 _`suggestion1Stream` **是** 由 'close 1' Stream與最新響應中的一個用戶合併(combine)而來，在程序剛運行或者刷新時則是`null`_。

留意一下代碼中並沒有出現如`if`、`for`、`while`這樣的控制語句，或者一般JavaScript應用中典型的基於回調的控制流。如果你想使用`filter()`，上面的`subscribe()`中甚至可以不用`if`、`else`(實現細節留給讀者作為練習)。在Rx中，我們有著像`map`、`filter`、`scan`、`merge`、`combineLatest`、`startWith`這樣的Stream函數，甚至更多類似的函數去控制一個事件驅動(Event-driven)的程序。這個工具集讓你可以用更少的代碼實現更多的功能。

## 下一步

如果你覺得Rx*會成為你首選的RP庫，花點時間去熟悉這個[函數列表](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/core/observable.md)，包括了如何轉換(transform)、合併(combine)、以及創建Observable。如果你想通過圖表去理解這些函數，看一下這份[RxJava's very useful documentation with marble diagrams](https://github.com/Netflix/RxJava/wiki/Creating-Observables)。無論什麼時候你遇到問題，畫一下這些圖，思考一下，看一下這一大串函數，然後繼續思考。以我個人經驗，這樣效果很明顯。

一旦你開始使用Rx*去編程，很有必要去理解[Cold vs Hot Observables](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/gettingstarted/creating.md#cold-vs-hot-observables)中的概念。如果忽略了這些，你一不小心就會被它坑了。我提醒過你了。通過學習真正的函數式編程(Funational programming)去提升自己的技能，並熟悉那些會影響到Rx*的問題，比如副作用(Side effect)。

但是RP不僅僅有Rx*。還有相對容易理解的[Bacon.js](http://baconjs.github.io/)，它沒有Rx*那些怪癖。[Elm Language](http://elm-lang.org/)則以它自己的方式支持RP：它是一門會編譯成Javascript + HTML + CSS的FRP **語言**，並有一個[Time travelling debugger](http://debug.elm-lang.org/)。非常NB。

Rx在需要處理大量事件的Frontend和Apps中非常有用。但它不僅僅能用在客戶端，在Backend或者與Database交互時也非常有用。事實上，[RxJava是實現Netflix's API服務器端並發的一個重要組件](http://techblog.netflix.com/2013/02/rxjava-netflix-api.html)。Rx並不是一個只能在某種應用或者語言中使用的Framework。它本質上是一個在開發任何Event-driven軟件中都能使用的編程範式(Paradigm)。

如果這份教程能幫到你，[請與更多人分享](https://twitter.com/intent/tweet?original_referer=https%3A%2F%2Fgist.github.com%2Fstaltz%2F868e7e9bc2a7b8c1f754%2F&amp;text=The%20introduction%20to%20Reactive%20Programming%20you%27ve%20been%20missing&amp;tw_p=tweetbutton&amp;url=https%3A%2F%2Fgist.github.com%2Fstaltz%2F868e7e9bc2a7b8c1f754&amp;via=andrestaltz)。
