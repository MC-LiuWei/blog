---
title: 从双指针在循环队列里的使用
date: 2020-03-19 01:01:43
categories:
    - LeetCode
tags:
    - 编程基础
---
前两天从新的再走一遍leetcode的数据结构基础，又仔细看了看循环队列的实现。对于示例解答的基础代码仔细的看了看，发现有和我最开始的实现方式的操作，都是用push和shift之类的api去做的。对照着双指针的方案想，感觉还是别人的思路好，可以充分的考虑js数组的操作特性。重新回写一遍别人的方案，验证一下自己的理解。

```js
class Circularqueue {
    constructor(len) {
        this.length = len;
        this.head = -1; // 出队列计数指针
        this.footer = -1; // 入队列计数指针
        this.queue = [];
    }

    // 插入
    insert(item) {
        if (this.isFull()) return false;
        if (this.isEmpty()) this.head = 0;
        this.footer = (this.footer + 1)%this.length;
        this.queue[this.footer] = item;
        return true;
    }

    delete() {
        if (!this.isEmpty()) {
            if (this.head === this.footer) {
                this.head = -1;
                this.footer = -1;
            } else {
                this.head = (this.head + 1)%this.length;
            }
            return true;
        }
        return false;
    }

    read() {
        if (!this.isEmpty()) return this.queue[this.footer];
        return -1;
    }

    front() {
        if (!this.isEmpty)) return this.queue[this.head];
        return -1;
    }

    isFull() {
        return (this.footer + 1)%this.length === this.head;
    }

    isEmpty() {
        return this.head === -1;
    }
}
```

## 代码理解