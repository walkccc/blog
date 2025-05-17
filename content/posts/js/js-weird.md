---
title: 克服 JS 的奇怪部分 - 2. 執行環境與詞彙環境
author: Peng-Yu Chen
date: 2019-03-10
tags:
  - JavaScript
---

# 7. 觀念小叮嚀：名稱/值配對與物件

`Conceptual Aside: Name/Value Pairs & Objects`

> **Name/Value Pair**: A name which maps to a unique value.
>
> > The name may be defined more than once, but only can have one value in any
> > given **context**. That value may be more name/value pairs.

JS 中的物件，單純的是 Key-Value 的關係，而物件中也可以再包含物件，像 `Address`
包含 `Apartment`。

```js
const Address = {
  Street: "Main",
  Number: 100,
  Apartment: {
    Floor: 3,
    Number: 301,
  },
};
```

> Object: a collection of name value pairs

# 9. 全域環境與全域變數

`The Global Environment & The Global Object`

JS 都是在「Execution Context」中執行的，而 global execution context 會為你創建兩
個東西：

1. Global Object
1. `this`

我們可以透過以下的代碼來實驗，或是直接將 js 代碼打在 Google Chrome 的 Console 中
：

- `index.html`

  ```html
  <html>
    <head> </head>
    <body>
      <script src="app.js"></script>
    </body>
  </html>
  ```

- `app.js`

  ```js

  ```

是的，我們的 `app.js` 是空的，但 JS engine 一樣會在此處創建一個 global execution
context，所以當我們在 Console 打 `this` 或 `window` 時會如下方所示：

```
> this
< Window {postMessage: f, blur: f, focus: f, close: f, parent: Window, ...}
> window
< Window {postMessage: f, blur: f, focus: f, close: f, parent: Window, ...}
```

---

若我們試著寫一些代碼在 `app.js` 中：

注意：若用 ES6 之後的 `const` 或 `let` 取代 `var` 宣告則不會有此現象。

```js
var a = "Hello World!";

function b() {}
```

```
> a
< "Hello World!"
> window.a
< "a"
```

# 10. 執行環境：創造與提升

`The Execution Context: Creation & 'Hoisting'`

在同一個執行環境中，會優先將使用到的變數存到記憶體中。

```js
b(); // Called b!
console.log(a); // undefined
var a = "Hello World!";

function b() {
  console.log("Called b!");
}
```

`b()` 的內容會先優先存進記憶體內，所以我們可以在第 1 行優先呼叫第 5 行的 `b()`

而在使用等號賦值時，只會先將變數放至記憶體，`var a = 'Hello World!;` 在這裡只會
先將 `a` 放入記憶體中，再依執行順序將等號的值填入。

可想像代碼變成這樣：

```js
var a;

function b() {
  console.log("Called b!");
}

b(); // Called b!
console.log(a); // undefined

a = "Hello World!";
```

`var a` 被移到最前面，但還未賦值，所以 JS engine 會在一開始賦於
`a = undefined`（所有值一開始都是 `undefined`），`b()` 則是連內容都移到前面去了
。

---

總結一下這節：

執行環境會分成兩個階段被創造：

1. Creation Phase
   - 在記憶體中，設定 Global Object
   - 在記憶體中，設定 `this`
   - 設定 Outer Environment
   - 設定好變數和函數的記憶體空間（**Hoisting**）
1. Execution Phase
   - Line by line 執行代碼

註：Hoisting 不代表 JS engine 真的揶動了你的代碼，而是先在 Creation Phase 時，先
設定好變數和函數的記憶體空間。

# 11. 觀念小叮嚀：JavaScript 與 'undefined'

`Conceptual Aside: JavaScript & undefined`

- `undefined` 是 JS 的特殊值，表該值有被存在記憶體中，但當未賦值
- not defined 代表該值不存在記憶體中

  ```js
  var a;

  if (a === undefined) {
    console.log("a is undefined"); // a is undefined
  } else {
    console.log("a is defined");
  }
  ```

我們可以刻意讓 `a = undefined`，但這是一個 bad practice，應該讓 `undefined` 就代
表還沒賦值的變數。

# 12. 執行環境：程式執行

`The Execution Context: Code Execution`

在 10. 我們提到了執行環境的第一步驟是 creation，第二步驟很簡單，JS engine 就一行
一行的執行你的代碼。

# 13. 觀念小叮嚀：單執行緒、同步執行

`Conceptual Aside: Single Threaded, Synchronous Execution`

> **Single Threaded**: one command at a time.
>
> > Under the hood of the browser, maybe not.

> **Synchronous**: one at a time and in order.

# 14. 函數呼叫與執行堆

`Function Invocation & The Execution Stack`

這程式的普世價值，就不多做贅述。

# 15. 函數、環境與變數環境

`Functions, Context, & Variable Environments`

> **Variable Environment**: where the variables live.
>
> > and how they related to each other in memory.

```js
function b() {
  var myVar;
  console.log(myVar);
}

function a() {
  var myVar = 2;
  console.log(myVar);
  b();
}

var myVar = 1;
console.log(myVar); // 1
a(); // 2 \n undefined
console.log(myVar); // 1
```

此例子和大多數的程式語言一樣，每個變數會有他的「域」。

# 16. 範圍鏈

`The Scope Chain`

接下來這個例子，我們將 `b()` 裡面的 `var myVar` 註解掉，這時第 2 行會是 not
defined 嗎？

```js
function b() {
  // lexically (phsically) sits at the global level
  console.log(myVar);
}

function a() {
  var myVar = 2;
  b();
}

var myVar = 1;
a(); // 1
```

不是的，由於 lexical environment 的關係，而 `b()` 「lexically sits at the global
level」，因此 `b()` 的 reference to outer environment 會指向 Global Execution
Context，所以在執行 `b()` 時，當我們在 `b()` 的執行環境中找不到 `myVar` 定義時，
就會到外部環境找（`b()` 的外部環境是 global），這就是 Scope Chain 的概念。

---

```js
function a() {
  function b() {
    console.log(myVar);
  }

  var myVar = 2;
  b();
}

var myVar = 1;
a(); // 2
```

# 17. 範圍、ES6 與 `let`

`Scope, ES6, & let`

> **Scope**: where a variable is available in your code.
>
> > and if it's truly the same variable, or a new copy.

# 18. 關於異步回呼

`What About Asynchronous Callbacks?`

> **Asynchronous**: more than one at a time.

JS engine 是單執行緒、同步的，那是如何達到異步執行的效果呢？

在瀏覽器中除了執行 JS engine 外，還會執行 rendering enginer、HTTP request 等。

雖然 JS engine 本身是同步的，但其他事件是可以異步執行的（這並不是真的異步，而是
瀏覽器異步的把東西放到事件佇列（Event Queue），但原本的程式仍然繼續一行一行執行
。

**當執行完後，執行堆空了，才會處理事件。**

---

來看看以下的例子：

```js
function waitThreeSeconds() {
  const ms = 3000 + new Date().getTime();
  while (new Date() < ms) {}
  console.log("finished function");
}

function clickHandler() {
  console.log("click event!");
}

document.addEventListener("click", clickHandler);

waitThreeSeconds();
console.log("finished execution");
```

當在執行 `waitThreeSeconds()` 時，然後點頁面（`document`），試著去觸發
`clickHandler()` 會發生什麼事？

```
> fninished function
> finished execution
> click event!
```

我看發現 click event! 在最後才會印出來，因為 JS engine 直到執行堆是空的才會看事
件佇列，這表示長時間函數可以干擾事件，這就是 JS 如何同步處理在瀏覽器別處異步的事
件發生。

JS 不斷執行原先的程式，當全部完成後，它會到事件佇列看看，並不停的檢查（continous
check）。
