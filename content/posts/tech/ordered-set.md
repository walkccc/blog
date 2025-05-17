---
title: "lower_bound and upper_bound vs floorEntry and ceilingEntry"
date: 2021-07-14
tags:
  - C++
  - Java
toc: true
---

# Example

## C++

- `lower_bound`: iterator that points to the first (minimum) number >= target.
- `upper_bound`: iterator that points to the first (minimum) number > target.

```cpp
map<int, string> map;
map[1] = "one";
map[2] = "two";
map[3] = "three";
map[5] = "five";
map[6] = "six";
                    // 1 2 3 5 6
map.lower_bound(0); // ^
map.upper_bound(0); // ^

                    // 1 2 3 5 6
map.lower_bound(4); //       ^
map.upper_bound(4); //       ^

                    // 1 2 3 5 6
map.lower_bound(5); //       ^
map.upper_bound(5); //         ^

                    // 1 2 3 5 6
map.lower_bound(6); //         ^
map.upper_bound(6); //           ^

                    // 1 2 3 5 6
map.lower_bound(7); //           ^ (null)
map.upper_bound(7); //           ^
```

## Java

- `ceilingEntry`: return a (key, value), where key is the minimum number >=
  target.

  Similar to `lower_bound` in C++.

- `floorEntry`: return a (key, value), where key is the maximum number <=
  target.

  Similar to `prev(map.upper_bound(val))` in C++.

```Java
Map<Integer, String> map = new TreeMap<>();
map.put(1, "one");
map.put(2, "two");
map.put(3, "three");
map.put(5, "five");
map.put(6, "six");

                     // 1 2 3 5 6
map.ceilingEntry(0); // ^
map.floorEntry(0);   //^ (null)

                     // 1 2 3 5 6
map.ceilingEntry(4); //       ^
map.floorEntry(4);   //     ^

                     // 1 2 3 5 6
map.ceilingEntry(5); //       ^
map.floorEntry(5);   //       ^

                     // 1 2 3 5 6
map.ceilingEntry(6); //         ^
map.floorEntry(6);   //         ^

                     // 1 2 3 5 6
map.ceilingEntry(7); //           ^ (null)
map.floorEntry(7);   //         ^
```
