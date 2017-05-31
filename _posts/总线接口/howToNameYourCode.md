---
title: 如何合理命名你的代码
date: 2016-07-28 15:01:20
toc: true
categories: C
tags: C 软件
---
文中内容参考自Bob Nystrom的blog。[原文链接](http://journal.stuffwithstuff.com/2016/06/16/long-names-are-long/)
<!--more-->
> A name has two goals:
	- It needs to be clear: you need to know what the name refers to.
	- It needs to be precise: you need to know what it does not refer to.

## 遵循最**简洁、精确**的命名原则
### 省略那些从变量类型上就可以得知的信息
这个规则主要用在使用静态类型的语言上，用户(程序员)通常知道一个变量的类型，那么如果此时再在变量的命名中添加关于变量类型的信息无疑是冗余的(*不是很赞同，如果是嵌入式开发者，尤其是从事单片机应用开发的程序员，现如今使用的大部分IDE或代码编辑工具对变量的自动补全、提示都支持的不够友好，例如keil。在处理一些变量的时候，知道变量的类型会让程序员知道这个变量占用的内存大小、存放位置，从而更好的去使用它。*)
```cpp
// Bad:
String nameString;
DockableModelessWindow dockableModelessWindow;

// Better:
String name;
DockableModelessWindow window;
```
### 省略那些容易产生歧义的信息
这点在我以往的命名中是个普遍存在的毛病，例如，声明一个用于存储文件路径的变量，我会命名为`string currentUsedFilePath`,虽然在一定程度上达到了变量名的自解释的作用，但是会让其他阅读你的程序的人产生误解，到底是current Used File`s Path，还是current Used FilePath(@_@);
```cpp
// Bad:
finalBattleMostDangerousBossMonster;
weaklingFirstEncounterMonster;

// Better:
boss;
firstMonster;
```
### 省略那些在当前的上下文中可以得到的信息
这点很好理解，比如你的项目中只用到LCD来完成显示相关信息，用来刷新显示的函数直接命名`display`和你命名`LCDDisplay`是一个效果的，但是后者显得会有些冗余，同样，类的成员命名也是一个道理；
```cpp
// Bad:
class AnnualHolidaySale {
  int _annualSaleRebate;
  void promoteHolidaySale() { ... }
}

// Better:
class AnnualHolidaySale {
  int _rebate;
  void promote() { ... }
}
```
### 抛弃那些意义不大的信息
一个原则：试想，如果去掉这部分信息，这个命名的意义变化了吗？如果没有，果断去掉，例如声明一个变量 `int tempVariable;`,去掉variable吧！
```cpp
// bad 
class WaffleObject {
  void garnish(List<Strawberry> strawberries) { ... }
}
// good one
class Waffle {
  void garnish(List<Strawberry> strawberries) { ... }
}
```