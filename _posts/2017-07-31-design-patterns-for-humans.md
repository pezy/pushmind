---
layout: post
title: 给人读的设计模式
categories: [dev]
description: 翻译加解读
keywords: design patterns
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

简言之:

> 创建型模式, 是针对如何**创建对象**的解决方案

Wikipedia:

> In software engineering, creational design patterns are design patterns that deal with object creation mechanisms, trying to create objects in a manner suitable to the situation. The basic form of object creation could result in design problems or added complexity to the design. Creational design patterns solve this problem by somehow controlling this object creation.

- [简单工厂](#-简单工厂)
- [工厂方法](#-工厂方法)
- [抽象工厂](#-抽象工厂)
- [生成器](#-生成器)
- [原型](#-原型)
- [单例](#-单例)

### 🏠 简单工厂

真实案例:

> 造房子时需要一个门. 你是穿上木匠服开始在你家门口锯木头, 搞得一团糟; 还是从工厂里生产一个.

简言之:

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

简言之:

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

简言之:

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

template <typename Door, typename DoorFittingExpert>
class DoorFactory : public IDoorFactory {
public:
    IDoor* MakeDoor() override {
        return new Door();
    }
    IDoorFittingExpert* MakeFittingExpert() override {
        return new DoorFittingExpert();
    }
};

int main()
{
    IDoorFactory* woodenFactory = new DoorFactory<WoodenDoor, Carpenter>();
    {
        IDoor* door = woodenFactory->MakeDoor();
        IDoorFittingExpert* expert = woodenFactory->MakeFittingExpert();
        door->GetDescription();
        expert->GetDescription();
    }

    IDoorFactory* ironFactory = new DoorFactory<IronDoor, Welder>();
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

简言之:

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

### 🐑 原型

真实案例:

> 还记得多利吗? 这个山羊是克隆的! 先不谈细节, 关键点都集中在于克隆上.

简言之:

> 通过克隆, 基于已有对象来创建对象.

Wikipedia:

> The prototype pattern is a creational design pattern in software development. It is used when the type of objects to create is determined by a prototypical instance, which is cloned to produce new objects.

简言之, 它允许你拷贝一份已有对象, 并根据需求改造之. 从而省去了草创的过程.

**示例代码**:

```cpp
#include <iostream>
#include <string>

class Sheep {
public:
    Sheep(const std::string& name, const std::string& category = "Mountain Sheep") : name_(name), category_(category) {}
    void SetName(const std::string& name) { name_ = name; }
    void ShowInfo() { std::cout << name_ << " : " << category_ << std::endl; }
private:
    std::string name_;
    std::string category_;
};

int main()
{
    Sheep jolly("Jolly");
    jolly.ShowInfo();

    Sheep dolly(jolly); // copy constructor
    dolly.SetName("Dolly");
    dolly.ShowInfo();

    Sheep doolly = jolly; // copy assignment
    doolly.SetName("Doolly");
    doolly.ShowInfo();
}
```

上述代码, 利用的是 `Sheep` 默认的拷贝构造和拷贝赋值函数, 当然也可也重写这两个函数实现自定义操作.

**使用时机**:

当需要的对象, 与已存在的对象非常相似, 或当创建过程比克隆一下更费时的时候.

**本质**:

原型模型, 基本已经嵌入在各种语言实现里了. 其核心就是 Copy. 其实这个策略不局限于代码架构, 当你重装完操作系统, 并安装了必备软件, 一般会打包一份 ghost, 下次给别人重装的时候, 直接一键 ghost 即可, 如有特殊需要, 可以装好后再调整. 这个过程实际上也是原型模式. 再比如, 我们开发时总会先搭建环境(脚手架), 现在可以用 docker 来搭建了, 那么你用别人的 docker 包来搭建时, 其实也时原型模式. 这样理解, 就比较通俗了.

### 💍 单例

真实案例:

> 所谓一山不容二虎, 一国不可两君. 遇到大事, 总应该由同一位老大来处理. 那么这位老大就是单例.

简言之:

> 确保一个特定类的一个对象, 只能创建一次.

Wikipedia:

> In software engineering, the singleton pattern is a software design pattern that restricts the instantiation of a class to one object. This is useful when exactly one object is needed to coordinate actions across the system.

实际上, 单利模式常被认为是反模式, 要避免过度使用. 它本质上类似全局变量, 可能会造成耦合度过高的问题, 也可能给调试带来困难. 故而一定要谨慎使用.

**示例代码**:

创造单例, 要确保构造函数私有化, 拷贝构造, 拷贝赋值应该禁用. 创建一个静态变量来存此单例.

```cpp
#include <iostream>
#include <string>
#include <cassert>

class President {
public:
    static President& GetInstance() {
        static President instance;
        return instance;
    }

    President(const President&) = delete;
    President& operator=(const President&) = delete;

private:
    President() {}
};

int main()
{
    const President& president1 = President::GetInstance();
    const President& president2 = President::GetInstance();

    assert(&president1 == &president2); // same address, point to same object.
}
```

## 结构型设计模式

简言之:

> 结构型模式重点关注对象组合, 换句话说, 实体如何互相调用. 还有另一个解释: 对"如何构建一个软件组件?"问题的回答.

Wikipedia:

> In software engineering, structural design patterns are design patterns that ease the design by identifying a simple way to realize relationships between entities.

- [适配器](#-适配器)
- [桥接](#-桥接)
- [组成](#-组成)
- [装饰](#-装饰)
- [外观](#-外观)
- [享元](#-享元)
- [代理](#-代理)

### 🔌 适配器

真实案例:

> 三个例子: 1) 你从相机存储卡传照片给电脑, 需要与此兼容的适配器, 来保证连接. 2) 电源适配器, 三脚插头转两脚插头. 3) 翻译, 看好莱坞大片, 将英文字幕转为中文.

简言之:

> 适配器模式, 包装一个对象, 让本不兼容其他类的该对象变得兼容.

Wikipedia:

> In software engineering, the adapter pattern is a software design pattern that allows the interface of an existing class to be used as another interface. It is often used to make existing classes work with others without modifying their source code.

**示例代码**:

```cpp
#include <iostream>

class ILion {
public:
    virtual void Roar() {
        std::cout << "I am a Lion" << std::endl;
    }
};

class Hunter {
public:
    void Hunt(ILion& lion) {
        lion.Roar();
    }
};

class WildDog
{
public:
    void Bark() {
        std::cout << "I am a wild dog." << std::endl;
    }
};

//! now we added a new class `WildDog`, the hunter can hunt it also.
//! But we cannot do that directly because dog has a different interface.
//! To make it compatible for our hunter, we will have to create an adapter.

class WildDogAdapter : public ILion {
public:
    WildDogAdapter(WildDog& dog): dog_(dog) {}
    void Roar() override {
        dog_.Bark();
    }
private:
    WildDog& dog_;
};

int main()
{
    WildDog dog;
    WildDogAdapter dogAdapter(dog);

    Hunter hunter;
    hunter.Hunt(dogAdapter);
}
```

**本质**:

软工领域, 很多问题都可以用**在中间加一层**的方式来解决. 适配器模式, 是这种策略的典型应用之一.

### 🚡 桥接

真实案例:

> 如果你有一个网站, 有着很多不同种类的页面. 而此刻你有一个功能是允许用户来改变主题样式. 该怎么做? 为每个不同页面都创建多个副本? 还是创建单独的主题, 并根据用户偏好加载它们? 桥接模式允许你实现第二种方案.

一图胜千言:

![bridge](https://cloud.githubusercontent.com/assets/11269635/23065293/33b7aea0-f515-11e6-983f-98823c9845ee.png)

简言之:

> 桥接模式, 优先考虑组合而非继承. 将实现细节从层次结构中, 剥离并独立成另一套层次结构.

Wikipedia:

> The bridge pattern is a design pattern used in software engineering that is meant to "decouple an abstraction from its implementation so that the two can vary independently"

**示例代码**:

```cpp
#include <iostream>
#include <string>

class ITheme {
public:
    virtual std::string GetColor() = 0;
};

class DarkTheme : public ITheme {
public:
    std::string GetColor() override { return "Dark Black"; }
};

class LightTheme : public ITheme {
public:
    std::string GetColor() override { return "Off white"; }
};

class AquaTheme : public ITheme {
public:
    std::string GetColor() override { return "Light blue"; }
};

class IWebPage {
public:
    IWebPage(ITheme& theme) : theme_(theme) {}
    virtual std::string GetContent() = 0;
protected:
    ITheme& theme_;
};

class About : public IWebPage {
public:
    About(ITheme& theme) : IWebPage(theme) {}
    std::string GetContent() override {
        return "About page in " + theme_.GetColor();
    }
};

class Careers : public IWebPage {
public:
    Careers(ITheme& theme) : IWebPage(theme) {}
    std::string GetContent() override {
        return "Careers page in " + theme_.GetColor();
    }
};

int main()
{
    DarkTheme darkTheme;
    About about(darkTheme);
    Careers careers(darkTheme);

    std::cout << about.GetContent() << std::endl;
    std::cout << careers.GetContent() << std::endl;
}
```

**本质**:

桥接模式本质上是一种拆分. 讲适配器模式的时候, 我提过一句行内名言: "很多(任何)问题都可以通过在中间加一层的方式解决". 这个可以理解成"加"操作. 那么桥接模式, 实际上是一种"减"操作. 而对应着另一句话: "面对混乱, 难以维护的代码, 第一步要做的就是拆". 怎么拆? 桥接模式讲究在一种层次结构中剥离出另一套独立的层次结构. 譬如这个例子中, "Theme"就是剥离出来的抽象概念. 玻璃之前, 页面一大堆, 但是有规律: 很多页面, 只是样式不同, 但内容相同. 怎么做? DRY 原则告诉我们, 相同的东西保留一份, 将不同的东西抽象出来. 这也与 "高内聚, 低耦合" 这个最终目的一脉相承.抽出 Theme 之后, 剩下的 Page 部分, 内聚就比较高了, 也可以称之为: 更加纯粹了, 只负责页面内容.

还要注意到适配器和桥接模式的相同点, 它们都存在一个强耦合的**关联**(UML)关系. 譬如 `WildDogAdapter` 就必然包含一个 `WildDog` (**组合**(UML)关系). `IWebPage` 就必然包含一个 `ITheme`(也是**组合**(UML)关系). 而从这个角度来看, 你发现桥接里, 这种关系发生在接口层面, 而适配器, 只是简单的发生在两个类层面. 这就好比简单工厂与工厂方法的关系. 是维度的上升, 而多出的这个维度, 就是桥接模式中"层次"的体现.

### 🌿 组成

真实案例:

> 每个公司都是由员工组成的. 每个员工, 有相同点: 如都有薪水, 都需要负责, 都可能有上级, 也都可能有下属.

简言之:

> 组合模式让客户以统一的方式对待各个独立的对象.

Wikipedia:

> In software engineering, the composite pattern is a partitioning design pattern. The composite pattern describes that a group of objects is to be treated in the same way as a single instance of an object. The intent of a composite is to "compose" objects into tree structures to represent part-whole hierarchies. Implementing the composite pattern lets clients treat individual objects and compositions uniformly.

**示例代码**:

```cpp
#include <iostream>
#include <string>
#include <vector>

class Employee {
public:
    Employee(const std::string& name, float salary): name_(name), salary_(salary) {}
    virtual std::string GetName() { return name_; }
    virtual float GetSalary() { return salary_; }

protected:
    float salary_;
    std::string name_;
};

class Developer : public Employee {
public:
    Developer(const std::string& name, float salary) : Employee(name, salary) {}
};

class Designer : public Employee {
public:
    Designer(const std::string& name, float salary) : Employee(name, salary) {}
};

class Organization {
public:
    void AddEmployee(const Employee& employee) {
        employees_.push_back(employee);
    }
    float GetNetSalaries() {
        float net_salary = 0;
        for (auto&& employee : employees_) {
            net_salary += employee.GetSalary();
        }
        return net_salary;
    }

private:
    std::vector<Employee> employees_;
};

int main()
{
    Developer john("John Doe", 12000);
    Designer jane("Jane Doe", 15000);

    Organization org;
    org.AddEmployee(john);
    org.AddEmployee(jane);

    std::cout << "Net salaries: " << org.GetNetSalaries() << std::endl;
}
```

**本质**:

组合模式在我看来甚至称不上什么模式, 其核心就是**多态**特性的体现. 另一个核心在于, 容器存储的是接口类型, 利用多态, 可以迭代处理通用操作.

### ☕ 装饰

真实案例:

> 假设你经营一家提供多种服务的汽车服务店. 你如何来计算收费账单? 通常会选择一项服务的同时, 动态更新服务的总价. 这里, 每一种服务都是装饰器.

简言之:

> 装饰模式将对象包装在装饰类对象中, 从而在运行时动态改变该对象的行为.

Wikipedia：

> In object-oriented programming, the decorator pattern is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class. The decorator pattern is often useful for adhering to the Single Responsibility Principle, as it allows functionality to be divided between classes with unique areas of concern.

**示例代码**:

```cpp
#include <iostream>
#include <string>

class ICoffee {
public:
    virtual float GetCost() = 0;
    virtual std::string GetDescription() = 0;
};

class SimpleCoffee : public ICoffee {
public:
    float GetCost() override { return 10; }
    std::string GetDescription() override { return "Simple coffee"; }
};

class CoffeePlus : public ICoffee {
public:
    CoffeePlus(ICoffee& coffee): coffee_(coffee) {}
    virtual float GetCost() = 0;
    virtual std::string GetDescription() = 0;
protected:
    ICoffee& coffee_;
};

class MilkCoffee : public CoffeePlus {
public:
    MilkCoffee(ICoffee& coffee): CoffeePlus(coffee) {}
    float GetCost() override { return coffee_.GetCost() + 2; }
    std::string GetDescription() override { return coffee_.GetDescription() + ", milk"; }
};

class WhipCoffee : public CoffeePlus {
public:
    WhipCoffee(ICoffee& coffee): CoffeePlus(coffee) {}
    float GetCost() override { return coffee_.GetCost() + 5; }
    std::string GetDescription() override { return coffee_.GetDescription() + ", whip"; }
};

class VanillaCoffee : public CoffeePlus {
public:
    VanillaCoffee(ICoffee& coffee): CoffeePlus(coffee) {}
    float GetCost() override { return coffee_.GetCost() + 3; }
    std::string GetDescription() override { return coffee_.GetDescription() + ", vanilla"; }
};

int main()
{
    ICoffee* someCoffee = new SimpleCoffee();
    std::cout << someCoffee->GetCost() << std::endl;
    std::cout << someCoffee->GetDescription() << std::endl;

    someCoffee = new MilkCoffee(*someCoffee);
    std::cout << someCoffee->GetCost() << std::endl;
    std::cout << someCoffee->GetDescription() << std::endl;

    someCoffee = new WhipCoffee(*someCoffee);
    std::cout << someCoffee->GetCost() << std::endl;
    std::cout << someCoffee->GetDescription() << std::endl;

    someCoffee = new VanillaCoffee(*someCoffee);
    std::cout << someCoffee->GetCost() << std::endl;
    std::cout << someCoffee->GetDescription() << std::endl;
}
```

**本质**:

装饰模式的形态很有意思, 像是静态语言的动态化. 其实实现上与组合模式类似, 但关键之处在于, 其**依赖的对象是其父类接口**. 顾名思义, 装饰模式, 装饰的是对象自身, 且支持重复装饰. 譬如上述实例中, 完全可以加两遍牛奶. 但最终要保证接口的一致性, 就像你的房子无论装饰成什么样子, 它依然只是你的房子.

### 📦 外观

真实案例:

> 如何开机? 你会说"按一下电源键". 你会有这样的反应, 是因为计算机提供了一个超级简单的接口, 而隐藏了一系列复杂的开机操作所致. 这个简单接口, 对于复杂操作来说, 就是外观.

简言之:

> 外观模式为复杂的子系统提供了一个简单接口.

Wikipedia:

> A facade is an object that provides a simplified interface to a larger body of code, such as a class library.

**示例代码**:

```cpp
#include <iostream>

class Computer {
public:
    void GetElectricShock() { std::cout << "Ouch!" << std::endl; }
    void MakeSound() { std::cout << "Beep beep!" << std::endl; }
    void ShowLoadingScreen() { std::cout << "Loading..." << std::endl; }
    void Bam() { std::cout << "Ready to be used!" << std::endl; }
    void CloseEverything() { std::cout << "Bup bup bup buzzzz!" << std::endl; }
    void Sooth() { std::cout << "Zzzzz" << std::endl; }
    void PullCurrent() { std::cout << "Haaah!" << std::endl; }
};

class ComputerFacade {
public:
    ComputerFacade(Computer& computer): computer_(computer) {}
    void TurnOn() {
        computer_.GetElectricShock();
        computer_.MakeSound();
        computer_.ShowLoadingScreen();
        computer_.Bam();
    }
    void TurnOff() {
        computer_.CloseEverything();
        computer_.PullCurrent();
        computer_.Sooth();
    }

private:
    Computer& computer_;
};

int main()
{
    Computer real_computer;
    ComputerFacade computer(real_computer);
    computer.TurnOn();
    computer.TurnOff();
}
```

**本质**:

同样很难称之为模式的模式. 用的依然是"多加一层"的思想, 通过封装的方式来实现. 多的这一层, 就是所谓的"外观"了.
