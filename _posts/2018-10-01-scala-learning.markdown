---
layout: post
title:  "Scala操作符学习笔记"
date:   2015-03-08 22:21:49
categories: Scala
tags: Scala
---
> `Scala`中很多的集合操作符，学会了他们在平时的编程中可以节约很多的时间。这其中的一些操作和`RxJava`中的一些操作符十分相似。

## 1.foreach
`foreach`可以直接遍历一个集合中的所有元素，没有返回值。
栗子：
```scala
//先定义个List
val names=List("a","b","c")
names.foreach(name =>{
      println(name)
})
/*
输出结果
a
b
c
*/
```

## 2.map
`map`的功能有点和`foreach`相似，但是`map`有返回值，他可以将一个集合中的元素逐一取出，在处理过后再重新组合成原来集合的类型并返回。
举个栗子：
```scala
//先定义个List
val names=List("a","b","c")
println(names.map(_.toUpperCase))
//打印出结果 List(A, B, C)
//可见map将函数中的每个字母取出去进行大小写转换，然后又重新组合成原有的数据类型List集合并返回。
```

## 3.flatten

`flatten`的作用是讲一个嵌套的结构展开。
话不多说，老板来个栗子！
```scala
//我们分别定义了List,Set,Map集合，将其嵌套在一个List中，在使用flatten函数将其展开
val aNames = List("a","b")
val bNames = Set("c","d")
val cNames = Map(("d"->"dn"),("e"->"en"))
val names = List(aNames,bNames,cNames)
println(names.flatten)

//打印结果
//List(a, b, c, d, (d,dn), (e,en))
```

## 4.zip
看到`zip`心里想着这货不是压缩包的后缀嘛，顾名思义就是讲两个集合打包，就是合并起来的意思，这个的合并是指每个对应的元素合并起来，形成一个包含两个元素的新元素，可能有点难理解，直接看栗子。
```scala
List('a,'b,'c,'d).zip(List(1,2,3,4))
//打印结果
//List(('a,1), ('b,2), ('c,3), ('d,4))
```

## 5.zipWithIndex
`zipWithIndex`是指和`zip`有些类似，是讲一个列表遍历出来，与当前元素在集合中的位置（即Index）合并在一起，形成一个新的元素。
```scala
List(2,3,4,5).zipWithIndex
//打印结果
//List((2,0), (3,1), (4,2), (5,3))
```

## 6.flatmap
`flatmap`函数的功能是`map`和`flatten`的结合体，先将嵌套的结构，接收一个可以处理嵌套列表的函数（我的理解是这个函数需要有返回值，例如前面介绍的`map`），返回一个经过处理的原类型的集合。
```scala
//这里我将两个List嵌套起来，然后先经过flatMap取到x（x即为names中嵌套的集合：aNames和bNames），然后经过map函数，遍历拿到的x集合，将其转换为大写字母，返回给flatMap函数，进行合并。
val aNames = List("a","b")
val bNames = List("c","d")
val names = List(aNames,bNames)
val result = names.flatMap(x =>{
  x.map(y=>{
    y.toUpperCase()
  })
})
println(result)
//打印结果如下
//List(A, B, C, D)
```
## 7.filter
`filter`顾名思义就是过滤器的意思，接收一个返回Boolean的函数，true和保留，false过滤掉。
```scala
//我们定义了一个几个，filter函数中返回的是各个元素是否为偶数，是偶数返回true，则保留，反之过滤。
val ages = List(1,2,3,4,5,6,7)
println(ages.filter(age =>{age%2 == 0}))
//打印结果
//List(2, 4, 6)
```


> 以上是我暂时总结的7种集合操作符，理解之后灵活运用可以节省很多时间，但是这些函数的使用都需要有lambda表达式的基础，今天写到这里，如果你发现了一些错误还望海涵，并严厉指出，不胜感激！
