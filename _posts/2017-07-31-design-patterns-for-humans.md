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
- [工厂方法](#-工厂方法)
- [抽象工厂](#-抽象工厂)
- [生成器](#-生成器)
- 原型
- 单例

### 🏠 简单工厂

真实案例:

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

**使用时机**:

当你创建一个对象, 并非简单拷贝赋值, 牵扯很多其他逻辑时, 就应该把它放到一个专门的工厂中, 而不是每次都重复. 这个体现在 C++ 中, 主要就是将 new 语句的逻辑抽象到一个单例, 或者如上述例子一样, 扔到一个统一的静态函数中去.

**本质**:

其实就是"抽象"在**创建对象**时的一个具体体现.

### 🏭 工厂方法

真实案例:

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

class CommunityExecutive : public IInterviewer
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

template <typename Interviewer>
class OtherManager : public HiringManager {
protected:
    IInterviewer* makeInterviewer() override {
        return new Interviewer();
    }
};

int main()
{
    HiringManager* devManager = new OtherManager<Developer>();
    devManager->takeInterview();

    HiringManager* marketingManager = new OtherManager<CommunityExecutive>();
    marketingManager->takeInterview();
}
```

**使用时机**:

当一个类中有一些通用的处理, 但要等到运行时决定用哪个子类的实现. 换句话说, 当客户并不知道他需要哪一个子类时.

**解释**:

- 客户并不知道他需要哪一个子类? 他分明指定了委托人: 开发部主管和市场部主管!

要注意, 这里我们抽象的对象是面试这个过程, 作为用户, 它只知道 Interview 这个接口, 而不清楚这个 Interview 的派生关系. 也就是说作为招聘主管(用户), 他只是找来了开发主管, 市场主管. 然后他说:"面试一下". 他并不知道会面试些什么内容, 什么形式. 这些就是被封装的部分. 这就是上述"不知道需要哪一个子类"的具体解释.

- 这和简单工厂有啥区别?

平心而论, 这篇文章里, 简单工厂的例子并不恰当, 而简单工厂和工厂方法居然用了两个实例, 真是扰乱视听, 让人糊涂. 但即便如此, 还是可以看出, 两者最大的区别: 抽象的维度. 简单工厂的抽象, 是一维的, 它抽象的仅仅是所创建"**类型**"的接口; 而工厂方法的抽象, 是二维的, 它不仅抽象了所创建"类型"的接口, 而且抽象了"**方法**"的接口.

具体到例子里, 简单工厂实例中, 客户就是要一个门, 而不关心创建过程, 最后实际创造的是一个木门. 这个颇为讽刺, 如果客户要的是个铁门呢? 那就事与愿违了. 所以在这个例子里, 也是存在两维抽象的.一是"门"这个类型的抽象, 二是"造门"这个方法的抽象. 简单工厂只做到了前者, 而没有给出后者的解决方案, 这才造成了客户可能吃了哑巴亏. 如果我们按照工厂方法的思路, 将门工厂造门这件事进行细分, 木门交给木门工厂, 铁门交给铁门工厂. 这就和工厂方法里的例子别无二致了. 客户需要先指定委托对象, 而不关心具体怎么造门:

```cpp
DoorFactory* woodenDoorFactory = new WoodenDoorFactory();
woodenDoorFactory->MakeDoor(100, 200);

DoorFactory* ironDoorFactory = new IronDoorFactory();
ironDoorFactory->MakeDoor(100, 200);
```

这就成了工厂方法了. 所谓二维: "DoorFactory" 和 "MakeDoor", 前者是无论委托给谁, 反正它是一个门工厂; 后者是管它造什么门, 反正它会给我造出一个 100*200 的门来. 客户不关心细节, 只关心结果. 这就使抽象的本质了. 第二个例子也是一样, "Interview" 和 "HiringManager" 是两维抽象, 客户要的是完成面试过程, 要的就是一个面试官. 至于找了谁来充当面试官, 进行的是什么方面的面试. 最终都是看不到的. 最后抽象成的结果就是: "面试官完成了面试"这么件事.

### 🔨 抽象工厂

真实案例:

> 接着简单工厂里的案例. 根据实际需要, 您可以从木门商店获得木门, 从铁门商店获得铁门, 从相关商店获得 PVC 门. 除此之外, 你还需要不同专业的人, 来帮你装门, 木门需要木匠, 铁门需要焊工. 可以看到, 门之间存在着一定的对应依赖关系. 木门-木匠, 铁门-焊工, 等等.

大白话:

> 工厂们的工厂, 一家管理各自独立却又互相依赖的一批工厂, 而不关心各自的细节.

Wikipedia:

> The abstract factory pattern provides a way to encapsulate a group of individual factories that have a common theme without specifying their concrete classes

**示例代码**:

```cpp
#include <iostream>

class IDoor {
public:
    virtual void GetDescription() = 0;
};

class WoodenDoor : public IDoor {
public:
    void GetDescription() override {
        std::cout << "I am a wooden door" << std::endl;
    }
};

class IronDoor : public IDoor {
public:
    void GetDescription() override {
        std::cout << "I am a iron door" << std::endl;
    }
};

class IDoorFittingExpert {
public:
    virtual void GetDescription() = 0;
};

class Carpenter : public IDoorFittingExpert {
    void GetDescription() override {
        std::cout << "I can only fit wooden doors" << std::endl;
    }
};

class Welder : public IDoorFittingExpert {
    void GetDescription() override {
        std::cout << "I can only fit iron doors" << std::endl;
    }
};

class IDoorFactory {
public:
    virtual IDoor* MakeDoor() = 0;
    virtual IDoorFittingExpert* MakeFittingExpert() = 0;
};

class WoodenDoorFactory : public IDoorFactory {
public:
    IDoor* MakeDoor() override {
        return new WoodenDoor();
    }
    IDoorFittingExpert* MakeFittingExpert() override {
        return new Carpenter();
    }
};

class IronDoorFactory : public IDoorFactory {
public:
    IDoor* MakeDoor() override {
        return new IronDoor();
    }
    IDoorFittingExpert* MakeFittingExpert() override {
        return new Welder();
    }
};

int main()
{
    IDoorFactory* woodenFactory = new WoodenDoorFactory();
    {
        IDoor* door = woodenFactory->MakeDoor();
        IDoorFittingExpert* expert = woodenFactory->MakeFittingExpert();
        door->GetDescription();
        expert->GetDescription();
    }

    IDoorFactory* ironFactory = new IronDoorFactory();
    {
        IDoor* door = ironFactory->MakeDoor();
        IDoorFittingExpert* expert = ironFactory->MakeFittingExpert();
        door->GetDescription();
        expert->GetDescription();
    }
}
```

**使用时机**:

当遇到不那么简单的创建逻辑, 伴随着与之相关的依赖关系时.

**本质**:

依然可以用维度来理解抽象工厂. 抽象工厂比工厂方法又多了一维. 我们再把三个工厂理一遍: 简单工厂, 是针对一种"类型"的抽象; 工厂方法, 是针对一种"类型", 以及一种"创建方法"的抽象; 抽象工厂, 是针对**一组**"类型"与"创建方法"的抽象, 组内每一套类型与创建方法一一对应. 用造门这个例子来说: 简单工厂, 是封装了"造门"的操作, 输出的是一种门; 工厂方法, 是封装了"多种造门"的操作, 并委托"多家工厂", 输出的是"各种门". 抽象工厂, 是封装了"多种造门"的操作, "提供多种专业人员"的操作, 并委托给"多家工厂", 输出的是"各种门", 以及"各种专业人员", 且"门"与"专业人员"一一对应.

例子中, 抽象工厂提供了两套"类型 - 创建操作"(分别是"门 - 造门", "专业人员 - 提供专业人员"), 其实这个个数是无限的. 你可以提供 n 套这样的对应关系. 然后委托给相关的工厂. 这就是"工厂们的工厂"的具体含义.

### 👷 生成器

真实案例:

> 假设你在哈迪斯(美国连锁快餐集团), 正想下单. 如果你说, 要一个 "大哈迪", 他们很快就能交给你, 而不多问一句. 这是简单工厂的例子. 但, 当创建逻辑涉及更多步骤时, 譬如, 你在 Subway 买汉堡, 那么你可能需要做出更多选择, 想要哪种面包? 哪种酱汁? 哪种奶酪? 这种情况下, 就需要用到生成器模式了.

大白话:

> 允许你创建不同风格的对象, 同时避免构造器污染. 当对象有好几种口味的时候尤其有用. 或者是创建对象的过程涉及很多步骤时.

Wikipedia:

> The builder pattern is an object creation software design pattern with the intentions of finding a solution to the telescoping constructor anti-pattern.

简单说下"the telescoping constructor anti-pattern"(可伸缩构造器的反模式)是什么. 你总会看到下面这种构造函数:

```cpp
Burger(int size, bool cheese = true, bool peperoni = true, bool tomato = false, bool lettuce = true);
```

你应该已经察觉了, 构造函数的参数数量可能会迅速失控, 而且其参数安排会越来越难理解. 将来想要增加更多选项, 这个列表会一直增长下去. 这就被称为"the telescoping constructor anti-pattern"(可伸缩构造器的反模式).

**示例代码**:

```cpp
#include <iostream>

class Burger {
public:
    class BurgerBuilder;
    void showFlavors() {
        std::cout << size_;
        if (cheese_) std::cout << "-cheese";
        if (peperoni_) std::cout << "-peperoni";
        if (lettuce_) std::cout << "-lettuce";
        if (tomato_) std::cout << "-tomato";
        std::cout << std::endl;
    }
private:
    Burger(int size): size_(size) {}

    int size_ = 7;
    bool cheese_ = false;
    bool peperoni_ = false;
    bool lettuce_ = false;
    bool tomato_ = false;
};

class Burger::BurgerBuilder {
public:
    BurgerBuilder(int size) { burger_ = new Burger(size); }
    BurgerBuilder& AddCheese() { burger_->cheese_ = true; return *this; }
    BurgerBuilder& AddPepperoni() { burger_->peperoni_ = true; return *this; }
    BurgerBuilder& AddLettuce() { burger_->lettuce_ = true; return *this; }
    BurgerBuilder& AddTomato() { burger_->tomato_ = true; return *this; }
    Burger* Build() { return burger_; }
private:
    Burger* burger_;
};

int main()
{
    Burger* burger = Burger::BurgerBuilder(14).AddPepperoni().AddLettuce().AddTomato().Build();
    burger->showFlavors();
}
```

上述代码与原文代码略有不同, 但表达的主旨, 以及最终用法完全一致. (额外添加了输出函数, 方便运行查看)

**使用时机**:

当对象拥有好几种口味, 且需要避免构造器伸缩时使用. 与工厂模式运用场景不同之处在于: 当创建过程仅仅一步到位, 使用工厂模式. 如果需要分步进行, 则考虑使用生成器模式.

**本质**:

生成器模式的本质, 就是将构造函数中的参数列表**方法化**. 长长的参数列表, 无论是面向对象还是函数式编程, 都是大忌. 该模式主要就是为了解决该问题. 函数式编程中对该问题的解决方式是: [柯里化](https://zh.wikipedia.org/wiki/%E6%9F%AF%E9%87%8C%E5%8C%96), 其本质与生成器模式是一样的.