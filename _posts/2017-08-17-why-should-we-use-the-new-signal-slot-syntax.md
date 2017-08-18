---
layout: post
title: 为什么我们该用新的 Qt signal slot 语法了?
categories: [dev, qt]
description: Qt new signal slot Qt5
keywords: qt
---

你的 Qt 项目还充斥着各种 `SIGNAL` 和 `SLOT` 吗? 是不是还有各种 `foreach`?

如果恰好你也正在使用 Visual Studio 2015 或 2017 的话, 是不是还会看到头文件里, 很正常的函数, 下面也标出了警告线?

有时候我们并非为了赶时髦, 新出来什么就用什么. 而更多的会考虑新特性是否能为生产力, 效率带来一些提升.

譬如 C++11, 已经出现了十年, 我已经完全无法忍受没有 range for, lambda, auto, 右值语义等特性的 C++ 代码了. 这些东西是真的能为你的程序带来诸多好处. 而 Qt5 也有些年头了. 我们是否也应该将开发的脚步稍作停歇, 看看有哪些细节可以改进呢?

## 新的 Signal Slot 语法

在 2017 年才说新 Signal, Slot, 可能有翻陈年旧事的嫌疑. 但这也许是我个人的一种技术态度: 如果一项新技术, 我并未彻底的实践过, 绝不应该盲目推崇, 或盲目宣传. 就如 SICP 前言所说:

> Above all, **I hope we don't become missionaries.** Don't feel as if you're Bible salesmen. The world has too many of those already. What you know about computing other people will learn. Don't feel as if the key to successful computing is only in your hands. What's in your hands, I think and hope, is intelligence: **the ability to see the machine as more than when you were first led up to it, that you can make it more.**

**说说新语法在实践中的优势**:

1. 编译期检查类型匹配, 以及 signal 和 slot 是否存在.
1. 参数既可以是 typedef, 也可以是不同的命名空间修饰符.
1. signal 和 slot 的类型不必完全一致, 兼容隐式转换的情况.
1. 可以 connect 任何成员函数, 没必要非要是 slot.

其中, 我最看重的是第一点, 如果你维护一个 Qt 项目有些年头, 重构过多次, 发布过多次. 那么在使用老式语法的情况下, 经常会出现的一种情况: 总存在一些"僵尸连接". 何谓僵尸连接? 顾名思义, 连接里的信号, 和槽函数都压根不存在了, 还 connect 着呢? 为什么还能编译通过? 因为老式语法, 是用宏来实现的信号槽, moc 里记录的仅仅是字符串, 所以编译器无法对此进行检查, 直到运行时出现崩溃, 方可见端倪.

为什么新语法可以避免? 因为用**函数指针**代替了**宏**. 这样一来, 起码 Qt 可以用 `static_assert` 在编译期检查一下. 这也解释了第三条的便利, 因为是函数, 所以必然可以接受参数的隐式转换.

对于成员函数的函数指针, 可能从写法上比较啰嗦, 因为你总需要带着类名, 命名空间等等. 但这也带来了相应的好处: slot 被解放了. 怎么讲呢, 我们看一下新式语法为 connect 函数增加的三项新的静态重载:

```cpp
QObject::connect(const QObject *sender, PointerToMemberFunction signal,
                 const QObject *receiver, PointerToMemberFunction slot,
                 Qt::ConnectionType type)
QObject::connect(const QObject *sender, PointerToMemberFunction signal,
                 PointerToFunction method)
QObject::connect(const QObject *sender, PointerToMemberFunction signal,
                 Functor method)
```

除了第一项与原来语法有意相似以外, 后两者是纯纯的函数指针与函子. 也就是压根不受 moc 的限制了. 那么直接带来两项重大解放:

1. 你可以用普通成员函数当作 slot 来连接.
1. 你可以用 lambda 当作 slot 来连接.

这两者改变才是革命性的, 远比你换个更长的写法来写信号槽, 有意义得多.

**官方提到的劣势**:

1. 更复杂的语法.(上面提到了, 因为要带着类型, 所以没有老式那么短小)
1. overload 不好解决, 需要用很丑陋的解决方案.
1. 默认参数没法在 slot 函数中用了.

其实这几条里, 比较要命的是第二条, 毕竟太多的信号, 都是 overload 的. [官方](https://wiki.qt.io/New_Signal_Slot_Syntax#Overload) 给出的解决方案长这样子:

```cpp
connect(
    mySpinBox, static_cast<void (QSpinBox::*)(int)>(&QSpinBox::valueChanged),
    mySlider, &QSlider::setValue
);
```

这的确让人比较无法接受, 太糟心了. 好在 Qt5.7 之后, 提供了一个 [`qOverload`](http://doc.qt.io/qt-5/qtglobal.html#qOverload)(需要 C++14 支持). 你可以这样解决这个问题:

```cpp
connect(mySpinBox, qOverload<int>(&QSpinBox::valueChanged), mySlider, &QSlider::setValue);
```

是不是好多了, 而且更明确了?

默认参数本就不是很好的编程习惯, 所以倒也并不可惜.

综上分析, 新式语法, 绝对是利处多多的.

## 要避免 Qt 里的一些宏

Qt 在 Visual Studio 中, 就像是外地人来到了北上广, 容易被 VS 排外, 动不动就给莫名其妙的警告. 譬如我们开头提到的 `foreach`, 这货明显是个宏, 还非要小写伪装成函数的样子. VS 不知怎么就觉得这个不对劲, 如下图中：

![image](https://user-images.githubusercontent.com/1147451/29443117-7d758102-8408-11e7-806e-8a50ad0b5054.png)

GetAllCandidates 函数中因为包含了 `foreach`, 就莫名其妙的找不到实现, 且无法通过 IDE 跳转了.

当然, 这是 VS 的锅, 如果改用 QCreator 便不会有这样的问题. 但是谁让 VS 是宇宙第一好用的 IDE 呢.

更何况我们完全可以用 C++11 之后的 range for 来替代 `foreach`. 这样的替换, 不仅保证了语言的纯粹性, 还极大提高了 IDE 的使用体验. 何乐而不为呢?