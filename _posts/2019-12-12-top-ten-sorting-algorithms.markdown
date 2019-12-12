---
layout: post
title:  "Java实现基本排序算法"
date:   2019-12-12 22:21:49
tags: Algorithm
---
说一个真实的事情，有一天下班我收到以为考研童鞋的消息，问我下面的代码是什么意思。
```java
int tmp = a;
a = b;
b = tmp;
```
我当场懵逼，这都不知道，当初是怎么上课的，我跟他说这是交换a和b的值。接着他又回我，交换两个数为什么不这么写？
```java
a = b;
b = a;
```
fuck,内心真的是无言以对。

> 排序算法是每个程序员都必备的技能，如果你还没有掌握跟我一起回顾一下吧。

## 注意
> * 本文中所有代码都是按照从小到大的顺序进行排列
> * 待比较的类型都是int类型，例子嘛，怎么简单怎么来。

## 1.冒泡排序
个人感觉冒泡排序是写起来最简单的排序了，直接上代码：
```java
private static int[] selectionSort(int[] toSort) {
    for (int i = 0; i < toSort.length; i++) {
        for (int j = i; j < toSort.length; j++) {
            if (toSort[i] > toSort[j]) {
                int tmp = toSort[j];
                toSort[j] = toSort[i];
                toSort[i] = tmp;
            }
        }
    }
    return toSort;
}
```
冒泡排序的思想就是依次拿数组中的每个元素去和他后面的每个元素进行对比，如果当前元素大于他后面的元素就置换一下。这样就可以保证第一遍运行的时候最大的那个数置换到了数组的最后面，第二遍运行的时候第二大的数字被换到了倒数第二的位置，依次类推，当执行完整个遍历的时候，数组就是按照从小到大排序的了。

![bubbleSort](/asset/img/sort/bubbleSort.png)

## 2.选择排序

```java
private static int[] selectionSort(int[] toSort) {
    int min;
    for (int i = 0; i < toSort.length; i++) {
        min = i;
        for (int j = i; j < toSort.length; j++) {
            if (toSort[min] > toSort[j]) {
                min = j;
            }
        }
        if (toSort[min] < toSort[i]) {
            int tmp = toSort[i];
            toSort[i] = toSort[min];
            toSort[min] = tmp;
        }
    }
    return toSort;
}
```
## 3.插入排序
## 4.希尔排序
## 5.快速排序
## 6.堆排序
## 7.归并排序
## 8.基数排序
## 9.计数排序
## 10.桶排序


