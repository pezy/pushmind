---
title: 给人读的设计模式
date: 2017-07-31
categories:
  - dev
tags:
  - design patterns
---

本文是针对 <https://github.com/kamranahmedse/design-patterns-for-humans> 的翻译与笔记, 会结合部分个人理解. 若您发现有明显理解有误的地方, 及疏漏之处, 麻烦留言指正, 在下不胜感激.

> 标题的解读: 设计模式与重构号称软工双雄, 在软件工程领域可谓智慧的结晶, 尤其是设计模式, 由于其高度抽象与最佳实践的特性, 导致初学者以及编程经验不足者, 读此如读天书. 所谓"给人读的", 就是将设计模式请下神坛, 用更容易理解的角度来介绍其精髓. 本人大学时期曾读过一本<大话设计模式>, 就走的通俗易懂之路, 然而, 通俗不能有失准确, 易懂不能理解偏差. 差之毫厘, 谬以千里. 闻者足戒.
>
> Explain them in the simplest way possible. --- 作者的话

## 🚀 初见

软工的江湖, 有一个原则贯穿始终, 有如剑道: DRY(don't repeat yourself). 无数先哲们, 想尽各种办法来解决这个终极问题. 所谓设计模式, 就是其中最著名的一个解决方案, 其作者有四位, ~号称"东邪, 西毒..."~. 而这种办法, 早已不是一招一式, 不是什么特定的类, 库, 代码, 你没法 include, import 一下就坐享其成. 这些方法被称之为 guidelines, 如果直译的话, 就是指导方针. 听起来比较虚一点, 但它们的确是针对具体问题的.

这里引入 Wikipedia 的描述:

> In software engineering, a software design pattern is a general reusable solution to a commonly occurring problem within a given context in software design. It is not a finished design that can be transformed directly into source or machine code. It is a description or template for how to solve a problem that can be used in many different situations.

这可能说的更加精确一些, 但意思就是上述那意思.

### ⚠️ 小心

- 设计模式**不是银弹**(废话, 根本没有银弹)
- **不要教条, 不要中二, 也不要强迫症.** 如果陷入以上三种状态, 请牢记: 设计模式是用来解决问题的, 而不是用来找茬的.
- **因地制宜**, 就是天使, 否则, 则是魔鬼.

原作者使用 PHP7 作为示例代码, 而恰好本人完全不会宇宙最好语言, 只好用老土的 C++ 来阐述.

### 设计模式的类型

- [创建型](#创建型设计模式)
- [结构型](#结构型设计模式)
- [行为型](#行为型设计模式)

## 创建型设计模式

大白话:

> 创建型模式, 是针对如何**创建对象**的解决方案

Wikipedia:

> In software engineering, creational design patterns are design patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. The basic form of object creation could result in design problems or added complexity to the design. Creational design patterns solve this problem by somehow controlling this object creation.

- [简单工厂](#-简单工厂)
- 工厂方法
- 抽象工厂
- 生成器
- 原型
- 单例

### 🏠 简单工厂

真实世界:

> 造房子时需要一个门. 你是穿上木匠服开始在你家门口锯木头, 搞得一团糟; 还是从工厂里生产一个.

大白话:

> 简单工厂为用户提供了一个实例, 而隐藏了具体的实例化逻辑.

Wikipedia:

> In object-oriented programming (OOP), a factory is an object for creating other objects – formally a factory is a function or method that returns objects of a varying prototype or class from some method call, which is assumed to be "new".

**示例代码**:

```cpp
#include <iostream>

class IDoor {
public:
    virtual float GetWidth() = 0;
    virtual float GetHeight() = 0;
};

class WoodenDoor : public IDoor {
public:
    WoodenDoor(float width, float height): m_width(width), m_height(height){}
    float GetWidth() override { return m_width; }
    float GetHeight() override { return m_height; }

protected:
    float m_width;
    float m_height;
};

class DoorFactory {
public:
    static IDoor* MakeDoor(float width, float heigh)
    {
        return new WoodenDoor(width, heigh);
    }
};

int main()
{
    IDoor* door = DoorFactory::MakeDoor(100, 200);
    std::cout << "Width: " << door->GetWidth() << std::endl;
    std::cout << "Height: " << door->GetHeight() << std::endl;
}
```

**何时使用**:

当你创建一个对象, 并非简单拷贝赋值, 牵扯很多其他逻辑时, 就应该把它放到一个专门的工厂中, 而不是每次都重复. 这个体现在 C++ 中, 主要就是将 new 语句的逻辑抽象到一个单例, 或者如上述例子一样, 扔到一个统一的静态函数中去.

**本质**:

其实就是"抽象"在**创建对象**时的一个具体体现.

### 🏭 工厂方法

真实世界:

> 如果你主管招聘, 你肯定无法做到什么职位都由你一个人来面试. 根据具体的工作性质, 你需要选择并委托不同的人来按步骤进行面试.

大白话:

> 一种将实例化逻辑委托给**子类**的方法

Wikipedia:

> In class-based programming, the factory method pattern is a creational pattern that uses factory methods to deal with the problem of creating objects without having to specify the exact class of the object that will be created. This is done by creating objects by calling a factory method—either specified in an interface and implemented by child classes, or implemented in a base class and optionally overridden by derived classes—rather than by calling a constructor.

**示例代码**:

```cpp
#include <iostream>

class IInterviewer
{
public:
    virtual void askQuestions() = 0;
};

class Developer : public IInterviewer
{
public:
    void askQuestions() override {
        std::cout << "Asking about design patterns!" << std::endl;
    }
};

class CommuityExcutive : public IInterviewer
{
public:
    void askQuestions() override {
        std::cout << "Asking about community building!" << std::endl;
    }
};

class HiringManager
{
public:
    void takeInterview() {
        IInterviewer* interviewer = this->makeInterviewer();
        interviewer->askQuestions();
    }

protected:
    virtual IInterviewer* makeInterviewer() = 0;
};

class DevelopmentManager : public HiringManager
{
protected:
    IInterviewer* makeInterviewer() {
        return new Developer();
    }
};

class MarketingManager : public HiringManager
{
protected:
    IInterviewer* makeInterviewer() {
        return new CommuityExcutive();
    }
};

int main()
{
    HiringManager* devManager = new DevelopmentManager();
    devManager->takeInterview();

    HiringManager* marketingManager = new MarketingManager();
    marketingManager->takeInterview();
}
```

**何时使用**:

当一个类中有一些通用的处理, 但要等到运行时决定用哪个子类的实现. 换句话说, 当客户并不知道他需要哪一个子类时.
