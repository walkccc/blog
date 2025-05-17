---
title: Testing with Jasmine
date: 2018-07-17 19:11:13
tags:
  - JavaScript
---

在此附上 Elie Schoppik 講師的
[slides](http://webdev.slides.com/eschoppik/testing-with-jasmine#/)

# 為什麼要用 Jasmine？

- Jasming 提供了各式各樣不同的測試方式
- 在各種 JavaScript 環境下都能順利運行！

# Getting Started

1. 新增 html 檔。
1. 連結 CSS、JavaScript 檔。
1. 開始寫測試程式碼！

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Jasmine Tests</title>
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.6.2/jasmine.css"
    />
  </head>
  <body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.6.2/jasmine.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.6.2/jasmine-html.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jasmine/2.6.2/boot.js"></script>
  </body>
</html>
```

## 一些 keywords

- `describe` - "let me describt \_\_ to you."
- `it` - "let me tell you about \_\_."
- `expect` - "here's what I expect."

## 概念化

- `describe('Earth')`
  - `it('is round')`
    - `expect(earth.isRound.toBe(true))`
  - `it('is the third planet from the sun')`
    - `expect(earth.numberFromSun).toBe(3)`

## 寫成程式碼的話

在這裡，我一律使用 ES2015 的 arrow function 來使程式碼更加簡潔！

```javascript
let earth = {
  isRound: true,
  numberFromSun: 3,
};

describe("Earth", () => {
  it("is round", () => expect(earth.isRound).toBe(true));
  it("is the third planet from the sun", () =>
    expect(earth.numberFromSun).toBe(3));
});
```

## Matchers

Jasmine 提供了很多種 matchers，
在[這裡](https://jasmine.github.io/2.0/introduction)可以看到官方詳細的介紹

- `toBe` / `not.toBe`
- `toBeCloseTo`
- `toBeDefined`
- `toBeFalsey / toBeTruthy`
- `toBeGreaterThan / toBeLessThan`
- `toContain`
- `toEqual`
- `jasmine.any()`

## beforeEach 和 afterEach

- `beforeEach` 會在每次 `it` callback 前先執行，如此可以大符降低程式碼的重複性

舉例來說：

```javascript
describe("Arrays", () => {
  let arr;
  beforeEach(() => (arr = [1, 3, 5])); // run before each 'it' callback
  it("adds elements to an array", () => {
    arr.push(7);
    expect(arr).toEqual([1, 3, 5, 7]);
  });
  it("returns the new length of the array", () => expect(arr.push(7)).toBe(4));
  it("adds anything into the array", () => expect(arr.push({})).toBe(4));
});
```

- 類似的概念，`afterEach` 則是在每次 `it` callback 後執行

一樣來看個例子：

```javascript
describe("Counting", () => {
  let count = 0;
  beforeEach(() => count++);
  afterEach(() => (count = 0)); // run after each 'it' callback
  it("has a counter that increments", () => expect(count).toBe(1));
  it("gets reset", () => expect(count).toBe(1));
});
```

- `beforeAll` / `afterAll`

```javascript
let arr = [];
beforeAll(() => (arr = [1, 3, 5]));
describe("Counting", () => {
  it("starts with an array", () => {
    arr.push(4);
    expect(1).toBe(1);
  });
  it("keeps mutating that array", () => {
    console.log(arr); // [1, 2, 3, 4]
    arr.push(5);
    expect(1).toBe(1);
  });
});
describe("Again", () => {
  it("keeps mutating the array...again", () => {
    console.log(arr); // [1, 2, 3, 4, 5]
    expect(1).toBe(1);
  });
});
```
