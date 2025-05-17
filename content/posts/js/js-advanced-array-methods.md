---
title: Advanced Array Methods
author: Peng-Yu Chen
date: 2018-07-20 04:26:39
tags:
  - JavaScript
---

在此附上 Elie Schoppik 講師的
[slides](http://webdev.slides.com/eschoppik/advanced-array-methods#/)

# 陣列迭代

在 JavaScript 中，callback 是一種常見的函式，這篇文章想要對 JS 中幾種常見迭代
（iterate）陣列的方式做個初步的介紹。

主要函式有以下幾種，使用它們可以使程式碼變得更加整潔、清楚！

1. `forEach`
1. `map`
1. `filter`
1. `some`
1. `every`
1. `reduce`

## forEach

- 執行步驟：

  1. 迭代整個陣列
  1. 對每一個值（value）執行一次 callback
  1. 回傳 `undefined`

- 運行機制：

  ```js
  function forEach(arr, callback) {
    for (let i = 0; i < arr.length; ++i) {
      callback(arr[i], i, arr);
    }
  }
  ```

以下讓我們透過例子來深入了解 `forEach` 的奧妙！

1. Write a function called `doubleValues` which accepts an array and returns a
   new array with all the values in the array passed to the function doubled.

   - Examples:

     ```js
     doubleValues([1, 2, 3]);
     // [2, 4, 6]

     doubleValues([5, 1, 2, 3, 10]);
     // [10, 2, 4, 6, 20]
     ```

   - Solution:

     ```js
     function doubleValues(arr) {
       let newArr = [];
       arr.forEach((val) => newArr.push(val * 2));
       return newArr;
     }
     ```

1. Write a function called `onlyEvenValues` which accepts an array and returns a
   new array with only the even values in the array passed to the function.

   - Examples:

     ```js
     onlyEvenValues([1, 2, 3]);
     // [2]

     onlyEvenValues([5, 1, 2, 3, 10]);
     // [2, 10]
     ```

   - Solution:

     ```js
     function onlyEvenValues(arr) {
       let newArr = [];
       arr.forEach((val) => {
         if (val % 2 === 0) {
           newArr.push(val);
         }
       });
       return newArr;
     }
     ```

1. Write a function called `showFirstAndLast` which accepts an array of strings
   and returns a new array with only the first and last character of each
   string.

   - Examples:

     ```js
     showFirstAndLast(["colt", "matt", "tim", "udemy"]);
     // ['ct', 'mt', 'tm', 'uy']

     showFirstAndLast(["hi", "goodbye", "smile"]);
     // ['hi', 'ge', 'se']
     ```

   - Solution:

     ```js
     function showFirstAndLast(arr) {
       let newArr = [];
       arr.forEach((val) => newArr.push(val[0] + val[val.length - 1]));
       return newArr;
     }
     ```

1. Write a function called `addKeyAndValue` which accepts an array of objects, a
   key, and a value and returns the array passed to the function with the new
   key and value added for each object.

   - Examples:

     ```js
     addKeyAndValue(
       [{ name: "Elie" }, { name: "Colt" }],
       "title",
       "instructor"
     );
     // [{ name: 'Elie', title: 'instructor' },
     //  { name: 'Colt', title: 'instructor' }]
     ```

   - Solution:

     ```js
     function addKeyAndValue(arr, key, value) {
       arr.forEach((val) => (val[key] = value));
       return arr;
     }
     ```

1. Write a function called `vowelCount` which accepts a string and returns an
   object with the keys as the vowel and the values as the number of times the
   vowel appears in the string. This function should be case insensitive so a
   lowercase letter and uppercase letter should count.

   - Examples:

     ```js
     vowelCount("Elie");
     // { e: 2, i: 1 }

     vowelCount("Colt");
     // { o: 1 }

     vowelCount("hmmm");
     // {};

     vowelCount("I Am awesome and so are you");
     // { i: 1, a: 4, e: 3, o: 3, u: 1 }
     ```

   - Solution:

     ```js
     function vowelCount(str) {
       let obj = {};
       let vowels = "aeiou";
       str
         .toLowerCase()
         .split("")
         .forEach((val) => {
           if (vowels.indexOf(val) !== -1) {
             if (obj[val]) {
               ++obj[val];
             } else {
               obj[val] = 1;
             }
           }
         });
       return obj;
     }
     ```

## map

- 執行步驟：

  1. 創建一個新的陣列（newArr）
  1. 迭代原本的陣列（arr）
  1. 對每一個值（value）執行一次 callback
  1. 將 callback 函式回傳的結果加入步驟一所建立的陣列（newArr）
  1. 回傳 newArr

- 運行機制：

  ```js
  function map(arr, callback) {
    let newArr;
    for (let i = 0; i < arr.length; ++i) {
      newArr.push(callback(arr[i], i, arr));
    }
    return newArr;
  }
  ```

我們可以發現，透過 map，我們不需要再自行建立陣列，這會使的程式碼變得非常潔簡，由
其是搭配 1-line arrow function 時！

一樣透過以下幾個例子，讓大家細細品嘗 `map` 比 `forEach` 強大的地方！

1. Write a function called `doubleValues` which accepts an array and returns a
   new array with all the values in the array passed to the function doubled.

   - Examples:

     ```js
     doubleValues([1, 2, 3]);
     // [2, 4, 6]

     doubleValues([1, -2, -3]);
     // [2, -4, -6]
     ```

   - Solution:

     ```js
     function doubleValues(arr) {
       return arr.map((val) => val * 2);
     }
     ```

1. Write a function called `valTimesIndex` which accepts an array and returns a
   new array with each value multiplied by the index it is currently at in the
   array.

   - Examples:

     ```js
     valTimesIndex([1, 2, 3]);
     // [0, 2, 6]

     valTimesIndex([1, -2, -3]);
     // [0, -2, -6]
     ```

   - Solution:

     ```js
     function valTimesIndex(arr) {
       return arr.map((val, i) => val * i);
     }
     ```

1. Write a function called `extractKey` which accepts an array of objects and
   some key and returns a new array with the value of that key in each object.

   - Examples:

     ```js
     extractKey([{ name: "Elie" }, { name: "Colt" }], "name");
     // ['Elie', 'Colt']
     ```

   - Solution:

     ```js
     function extractKey(arr, key) {
       return arr.map((val) => val[key]);
     }
     ```

1. Write a function called `extractFullName` which accepts an array of objects
   and returns a new array with the value of the key with a name of 'first' and
   the value of a key with the name of 'last' in each object, concatenated
   together with a space.

   - Examples:

     ```js
     extractFullName([
       { first: "Elie", last: "Schoppik" },
       { first: "Colt", last: "Steele" },
     ]);
     // ['Elie Schoppik', 'Colt Steele']
     ```

   - Solution:

     ```js
     function extractFullName(arr) {
       return arr.map((val) => `${val.first} ${val.last}`);
     }
     ```

## filter

- 執行步驟：

  1. 創建一個新的陣列（newArr）
  1. 迭代原本的陣列（arr）
  1. 對每一個值（value）執行一次 callback
     - 若 callback 回傳 `true`，將 value 加入步驟一所建立的陣列（newArr）
     - 若 callback 回傳 `false`，continue
  1. 回傳 newArr

- 運行機制：

  ```js
  function filter(arr, callback) {
    let newArr;
    for (let i = 0; i < arr.length; ++i) {
      if (callback(arr[i], i, arr)) {
        newArr.push(arr[i]);
      }
    }
    return newArr;
  }
  ```

例子：

1. Write a function called `filterByValue` which accepts an array of objects and
   a key and returns a new array with all the objects that contain that key.

   - Examples:

     ```js
     filterByValue(
       [
         { first: "Elie", last: "Schoppik" },
         { first: "Colt", last: "Steele", isCatOwner: true },
       ],
       "isCatOwner"
     );
     // [{ first: 'Colt', last: 'Steele', isCatOwner: true }]
     ```

   - Solution:

     ```js
     function filterByValue(arr, key) {
       return arr.filter((val) => val[key]);
     }
     ```

1. Write a function called `find` which accepts an array and a value and returns
   the first element in the array that has the same value as the second
   parameter or undefined if the value is not found in the array.

   - Examples:

     ```js
     find([1, 2, 3, 4, 5], 3);
     // 3

     find([1, 2, 3, 4, 5], 10);
     // undefined
     ```

   - Solution:

     ```js
     function find(arr, searchValue) {
       return arr.filter((val) => val === searchValue)[0];
     }
     ```

1. Write a function called `findInObj` which accepts an array of objects, a key,
   and some value to search for and returns the first found value in the array.

   - Examples:

     ```js
     findInObj(
       [
         { first: "Elie", last: "Schoppik" },
         { first: "Tim", last: "Garcia", isCatOwner: true },
         { first: "Colt", last: "Steele", isCatOwner: true },
       ],
       "isCatOwner",
       true
     );
     // { first: 'Tim', last: 'Garcia', isCatOwner: true}
     ```

   - Solution:

     ```js
     function findInObj(arr, key, searchValue) {
       return arr.filter((val) => val[key] === searchValue)[0];
     }
     ```

1. Write a function called `removeVowels` which accepts a string and returns a
   new string with all of the vowels (both uppercased and lowercased) removed.
   Every character in the new string should be lowercased.

   - Examples:

     ```js
     removeVowels("Elie");
     // 'l'

     removeVowels("TIM");
     // 'tm'

     removeVowels("ZZZZZZ");
     // 'zzzzzz'
     ```

   - Solution:

     ```js
     function removeVowels(str) {
       let vowels = "aeiou";
       return str
         .toLowerCase()
         .split("")
         .filter((val) => vowels.indexOf(val) === -1)
         .join("");
     }
     ```

1. Write a function called `doubleOddNumbers` which accepts an array and returns
   a new array with all of the odd numbers doubled (HINT - you can use map and
   fitler to double and then filter the odd numbers).

   - Examples:

     ```js
     doubleOddNumbers([1, 2, 3, 4, 5]);
     // [2, 6, 10]

     doubleOddNumbers([4, 4, 4, 4, 4]);
     // []
     ```

   - Solution:

     ```js
     function doubleOddNumbers(arr) {
       return arr.filter((val) => val % 2 === 1).map((val) => val * 2);
     }
     ```

## some

- 執行步驟：

  1. 迭代原本的陣列（arr）
  2. 對每一個值（value）執行一次 callback
     - 若至少有一個 callback 回傳 `true`，則回傳 `true`
     - 否則，回傳 `false`

- 運行機制：

  ```js
  function some(arr, callback) {
    for (let i = 0; i < arr.length; ++i) {
      if (callback(arr[i], i, arr)) {
        return true;
      }
    }
    return false;
  }
  ```

## every

- 執行步驟：

  1. 迭代原本的陣列（arr）
  1. 對每一個值（value）執行一次 callback
     - 若至少有一個 callback 回傳 `false`，則回傳 `fale`
     - 否則，回傳 `true`

- 運行機制：

  ```js
  function every(arr, callback) {
    for (let i = 0; i < arr.length; ++i) {
      if (callback(arr[i], i, arr) === false) {
        return false;
      }
    }
    return true;
  }
  ```

以下我們一樣透過例子來看如何實作 `some` 和 `every`：

1. Write a function called `hasOddNumber` which accepts an array and returns
   `true` if the array contains at least one odd number, otherwise it returns
   `false`.

   - Examples:

     ```js
     hasOddNumber([1, 2, 2, 2, 2, 2, 4]);
     // true

     hasOddNumber([2, 2, 2, 2, 2, 4]);
     // false
     ```

   - Solution:

     ```js
     function hasOddNumber(arr) {
       return arr.some((val) => val % 2 === 1);
     }
     ```

1. Write a function called `hasAZero` which accepts a number and returns `true`
   if that number contains at least one zero. Otherwise, the function should
   return `false`.

   - Examples:

     ```js
     hasAZero(3332123213101232321);
     // true

     hasAZero(1212121);
     // false
     ```

   - Solution:

     ```js
     function hasAZero(num) {
       return num
         .toString()
         .split("")
         .some((val) => val === "0");
     }
     ```

1. Write a function called `hasOnlyOddNumbers` which accepts an array and
   returns `true` if every single number in the array is odd. If any of the
   values in the array are not odd, the function should return `false`.

   - Examples:

     ```js
     hasOnlyOddNumbers([1, 3, 5, 7]);
     // false

     hasOnlyOddNumbers([1, 2, 3, 5, 7]);
     // false
     ```

   - Solution:

     ```js
     function hasOnlyOddNumbers(arr) {
       return arr.every((val) => val % 2 === 1);
     }
     ```

1. Write a function called `hasNoDuplicates` which accepts an array and returns
   `true` if there are no duplicate values (more than one element in the array
   that has the same value as another). If there are any duplicates, the
   function should return `false`.

   - Examples:

     ```js
     hasNoDuplicates([1, 2, 3, 1]);
     // false

     hasNoDuplicates([1, 2, 3]);
     // true
     ```

   - Solution:

     ```js
     function hasNoDuplicates(arr) {
       return arr.every((val) => arr.indexOf(val) === arr.lastIndexOf(val));
     }
     ```

1. Write a function called `hasCertainKey` which accepts an array of objects and
   a key, and returns `true` if every single object in the array contains that
   key. Otherwise it should return `false`.

   - Examples:

     ```js
     const arr = [
       { title: "Instructor", first: "Elie", last: "Schoppik" },
       { title: "Instructor", first: "Tim", last: "Garcia", isCatOwner: true },
       { title: "Instructor", first: "Matt", last: "Lane" },
       { title: "Instructor", first: "Colt", last: "Steele", isCatOwner: true },
     ];

     hasCertainKey(arr, "first");
     // true

     hasCertainKey(arr, "isCatOwner");
     // false
     ```

   - Solution:

     ```js
     function hasCertainKey(arr, key) {
       return arr.every((val) => key in val);
     }
     ```

1. Write a function called `hasCertainValue` which accepts an array of objects
   and a key, and a value, and returns `true` if every single object in the
   array contains that value for the specific key. Otherwise it should return
   `false`.

   - Examples:

     ```js
     let arr = [
       { title: "Instructor", first: "Elie", last: "Schoppik" },
       { title: "Instructor", first: "Tim", last: "Garcia", isCatOwner: true },
       { title: "Instructor", first: "Matt", last: "Lane" },
       { title: "Instructor", first: "Colt", last: "Steele", isCatOwner: true },
     ];

     hasCertainValue(arr, "title", "Instructor");
     // true

     hasCertainValue(arr, "first", "Elie");
     // false
     ```

   - Solution:

     ```js
     function hasCertainValue(arr, key, searchValue) {
       return arr.every((val) => val[key] === searchValue);
     }
     ```

## reduce

- 執行步驟

  1. 參數：
     - 1.（必要）callback 函式
     - 1.（可有可無）第二個參數（optional parameter）
  1. 迭代原本的陣列（`arr`）
  1. 對每一個值（value）執行一次 callback
     - 若有提供 optional parameter 的話，callback 的第一個參數便是該 optional
       parameter
     - 否則，則是 `arr[0]`，也就是第 0 個 value
     - 通常我們稱呼 callback 的第一個參數為 'accumulator'，常用縮寫為 acc
  1. 每次從 callback 回傳的值，變成為新的 accumulator！

本文最後介紹的 `reduce` 和前面任何一個函式執行方式有著很大的差異，需要透過不停的
練習才能熟能生巧，因為難度較高，讓我們先看幾個範例：

1. 無 optional parameter，最後回傳結果為 `6`。

   ```js
   const arr = [1, 2, 3];
   arr.reduce((acc, next) => acc + next);
   ```

   | accumulator | nextValue | returned value |
   | :---------: | :-------: | :------------: |
   |      1      |     2     |       3        |
   |      3      |     3     |       6        |

1. 有 optional parameter，最後回傳結果為 `16`。

   ```js
   const arr = [1, 2, 3];
   arr.reduce((acc, next) => acc + next, 10);
   ```

   | accumulator | nextValue | returned value |
   | :---------: | :-------: | :------------: |
   |     10      |     1     |       11       |
   |     11      |     2     |       13       |
   |     13      |     3     |       16       |

從範例中可以發現，透過 `reduce` 來加總整個陣列是非常方便的。

以下再多看幾題練習題吧！

1. Write a function called `extractValue` which accepts an array of objects and
   a key and returns a new array with the value of each object at the key.

   - Examples:

     ```js
     const arr = [{ name: "Elie" }, { name: "Colt" }];
     extractValue(arr, "name");
     // ['Elie', 'Colt']
     ```

   - Solution:

     ```js
     function extractValue(arr, key) {
       return arr.reduce((acc, next) => {
         acc.push(next[key]);
         return acc;
       }, []);
     }
     ```

1. Write a function called `vowelCount` which accepts a string and returns an
   object with the keys as the vowel and the values as the number of times the
   vowel appears in the string. This function should be case insensitive so a
   lowercase letter and uppercase letter should count

   - Examples:

     ```js
     vowelCount("Elie");
     // { e: 2, i: 1 }

     vowelCount("Colt");
     // { o: 1 }

     vowelCount("hmmm");
     // {};

     vowelCount("I Am awesome and so are you");
     // { i: 1, a: 4, e: 3, o: 3, u: 1 }
     ```

   - Solution:

     ```js
     function vowelCount(str) {
       const vowels = "aeiou";
       return str
         .toLowerCase()
         .split("")
         .reduce((acc, next) => {
           if (vowels.indexOf(next) !== -1) {
             if (acc[next]) {
               ++acc[next];
             } else {
               acc[next] = 1;
             }
           }
           return acc;
         }, {});
     }
     ```

1. Write a function called `addKeyAndValue` which accepts an array of objects, a
   key, and a value and returns the array passed to the function with the new
   key and value added for each object.

   - Examples:

     ```js
     addKeyAndValue(
       [{ name: "Elie" }, { name: "Colt" }],
       "title",
       "instructor"
     );
     // [{ name: 'Elie', title: 'instructor' },
     //  { name: 'Colt', title: 'instructor' }]
     ```

   - Solution:

     ```js
     function addKeyAndValue(arr, key, value) {
       return arr.reduce((acc, next, i) => {
         acc[i][key] = value;
         return acc;
       }, arr);
     }
     ```

1. Write a function called `partition` which accepts an array and a callback and
   returns an array with two arrays inside of it. The partition function should
   run the callback function on each value in the array and if the result of the
   callback function at that specific value is true, the value should be placed
   in the first subarray. If the result of the callback function at that
   specific value is false, the value should be placed in the second subarray.

   - Examples:

     ```js
     function isEven(val) {
       return val % 2 === 0;
     }
     const arr = [1, 2, 3, 4, 5, 6, 7, 8];
     partition(arr, isEven);
     // [[2, 4, 6, 8], [1, 3, 5, 7]];

     function isLongerThanThreeCharacters(val) {
       return val.length > 3;
     }
     const names = ["Elie", "Colt", "Tim", "Matt"];
     partition(names, isLongerThanThreeCharacters);
     // [['Elie', 'Colt', 'Matt'], ['Tim']]
     ```

   - Solution:

     ```js
     function partition(arr, callback) {
       return arr.reduce(
         (acc, next, i) => {
           if (callback(next)) {
             acc[0].push(next);
           } else {
             acc[1].push(next);
           }
           return acc;
         },
         [[], []]
       );
     }
     ```

以上就是幾個重要常見的 Array Methods！
