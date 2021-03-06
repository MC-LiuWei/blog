---
title: 对js使用双指针实现循环队列里的理解
date: 2020-03-19 01:01:43
cover: /imgs/queuebgone.jpeg
categories:
    - LeetCode
tags:
    - 编程基础
---
前两天从新的再走一遍leetcode的数据结构基础，又仔细看了看循环队列的实现。对于示例解答的基础代码仔细的看了看，发现有和我最开始的实现方式的操作，都是用push和shift之类的api去做的。对照着双指针的方案想，感觉还是别人的思路好，可以充分的考虑js数组的操作特性。重新回写一遍别人的方案，验证一下自己的理解。

```js
class Circularqueue {
    constructor(len) {
        this.head = -1; // 出队列计数指针
        this.footer = -1; // 入队列计数指针
        this.queue = new Array(len);
        this.length = this.queue.length;
    }

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
#### 队列的操作方式
>队列（queue）是只允许在一端进行插入操作，而在另一端进行删除操作的线性表

这个特点估计看一些面试题或者一些介绍已经听的耳熟能详的，不过有些时候是没办法理解好。就自己来看对于有些实现代码的理解可能是本末倒置的。对这段代码的理解我之前的思维方式就是，这样写的代码就是队列，这个应该不能算学习一些数据结构的基本知识，只记代码逻辑还是容易混淆特点的。
>顺序队列结构必须为其静态分配或动态申请一片连续的存储空间，并设置两个指针进行管理。一个是队头指针front，它指向队头元素

从上面描述的顺序队列的特点，也就知道样例代码里构造函数里这么写就是根据特点出发的。下次再写相关的就会知道要怎么写，而不是先回忆看过的代码，这样不会受他人的输出约束。再想熟练只能看在各种场景下对特点的应用了。回到循环队列上面，代码其实很基础，主要看优势和劣势的区别。

#### push和shift的方式
从存储方式看，数组有索引和元素两点。在js里数组空间其实是动态的，越界的情况如果不设置控制逻辑可能只有在内存消耗完才会有并不是越界情况的内存溢出。用push和shift操作会直接的操作数据，删除增加元素，它的存储大小（length）会一直的变动。同时也会操作索引和元素的指向，了解过v8的垃圾处理其实知道在新老引用交替的过程中会出现非连续性的存储情况，最后会对这段存储空间重新做连续性处理。这个和对数组头尾操作是相似的。
```js
let a = [1,2,3,4,5,6];
console.log(a[1]); // 现在访问索引为1的元素是2
a.shift();
console.log(a[1]); // 现在访问索引为1的元素是3
```
用shift操作中的一点是会引起数组的索引偏移，这个操作成本有点划不来，为了操作一个元素，要引起后面一连串的操作。用push虽然不会有偏移出现，但是数组的存储大小会改变。顺序队列没问题它可以是静态也可以动态，循环队列也可以，只是有点不太符合循环队列使队列空间重复利用的要求，重复利用是空间有限的情况。

#### 指针的利用
再来说在这个代码里对指针的最大化使用，感觉对编码里变量声名也是有一些帮助，有时候我们写代码会声名很多的变量，可是未必需要这么多也许一两个就能把功能做完，所以声名的时候也还是得考虑整个功能。这个里面头尾指针一共有三种作用，分别是：

##### 1.充当访问索引

```js
this.queue[this.footer] = item;

if (!this.isEmpty()) return this.queue[this.footer];

if (!this.isEmpty)) return this.queue[this.head];

this.head = (this.head + 1)%this.length;

this.footer = (this.footer + 1)%this.length;
``` 
这个非常舒服，用头尾指针的移动模拟出队列的入队和出队，做到了只有在插入的时候替换原数组对应索引下元素最小化对数组的操作，稳定还高效。

##### 2.循环逻辑实现
```js
    insert(item) {
        if (this.isFull()) return false;
        if (this.isEmpty()) this.head = 0;
        // 循环控制
        this.footer = (this.footer + 1)%this.length;
        this.queue[this.footer] = item;
        return true;
    }

    delete() {
        if (!this.isEmpty()) {
            if (this.head === this.footer) {
                // 循环控制
                this.head = -1;
                this.footer = -1;
            } else {
                // 循环控制
                this.head = (this.head + 1)%this.length;
            }
            return true;
        }
        return false;
    }
```
在队列里能不能做插入和删除的前提是存储空间是不是满的或者是不是空的，既然用双指针去模拟删除插入也就不是真的去删除数组元素添加元素（替换）。那非空或者满的条件只有通过指针重合去实现。循环的处理主要利用取模的方式去处理头尾指针。

![队列循环逻辑](/imgs/queuethree.png)

## 循环队列的使用
循环队列一般被用作环形缓冲区。从业以来听的比较多的就是消息队列，单从消息队列来看，队列的使用主要为了减小服务端处理的压力。两个服务之间调用，每次调用的参数就好像是数组里的一个元素，只有真正被响应了才会从中清除。在前端处理一些高频率函数调用的时候有时也会需要这样的操作，本质上就是将本来在短期内需要处理的操作将它依次延后操作。循环缓冲区主要的区别在于定长的队列空间重复利用，这其实是对队列系统的内存使用空间控制。再形象点的联想这个循环缓冲还和前端的首页滚动横幅还是有那么点像呢。不过这个首页滚动横幅这个地方其实更像兵乓队列。


