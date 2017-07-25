---
layout: post
title: VC处理XML的方法
categories: [dev]
description: some word here
keywords: cpp, xml
---

目前调研到两种比较好的方法: `CMarkup` 和 `TinyXML-2`.

以下是两者分别的文档: [CMarkup](http://www.firstobject.com/dn_markupmethods.htm), [TinyXML-2](http://www.grinninglizard.com/tinyxml2docs/index.html)

从上手的难易程度来看, 目前偏向于使用 `CMarkup`, 不过有机会一定要研究一下`TinyXML-2`的使用, 目前关于`TinyXML-2`的中文资料较少, 正好花时间翻译整理一下. 另外, 它不仅开源, 而且功能较`CMarkup`要全面, 文档更加齐全.

下面首先说一下`CMarkup`的基本使用方法:

首先在[这里]下载`CMarkup`, 里面包含一个`Markup.h`和`Markup.cpp`. 分别引入自己的项目下.

**NOTE:** 貌似`CMarkup`仅支持MFC, 在控制台程序中似乎无法使用.

在引入之后, 需在在`Markup.cpp`的头部加上`#include "stdafx.h"`, 在下述`testXML`工程里, 不再赘述这一过程.

[这里]: http://www.firstobject.com/cmarkup-11.5-release-notes.htm

## 创建XML

1. 新建一个MFC工程(`testXML`), 选择基于对话框的就好, 其余一切默认.
1. 在对话框上新建一个按钮(`Create`), 添加`BN_CLICKED`控件事件.
1. 在事件函数中写入一下代码:

```cpp
void CtestXmlDlg::OnBnClickedBtmCreate()
{
    // 创建XML
    CMarkup xml;
    xml.AddElem( L"ORDER" );
    xml.IntoElem();
    xml.AddElem( L"ITEM" );
    xml.IntoElem();
    xml.AddElem( L"SN", L"132487A-J" );
    xml.AddElem( L"NAME", L"crank casing" );
    xml.AddElem( L"QTY", L"1" );

    xml.Save( L"F:\\Sample.xml" );
    MessageBox(L"Success!");
}
```

我们仅运用了两种方法: [AddElem]和[IntoElem], 前者可以理解为增加一个标签, 后者则是进入此标签, 很简单的逻辑.

以下就是生成的XML示例:

```xml
<ORDER>
<ITEM>
<SN>132487A-J</SN>
<NAME>crank casing</NAME>
<QTY>1</QTY>
</ITEM>
</ORDER>
```

千万不要忘记了保存. 即调用`XML.Save();`.

**NOTE:** 这里有一点很囧, 就是保存路径无法写**相对路径**. 如果有知道如何写入相对路径的, 请留言告知, 感激不尽.

[AddElem]: http://www.firstobject.com/dn_markAddElem.htm

## 读取XML

我们就来读上面写好的`Sample.xml`吧. 如果在同一函数中, 完全可以用[GetDoc]方法重新读取该文档:

    MCD_STR strXML = xml.GetDoc();

`Markup.h`中定义了[MCD_STR]作为默认的字符串类型, 你也可以使用`std::string`或者`CString`. 然后可调用[ResetPos]方法来返回文档头部. 或者咱们重新写个函数, 专门来读:

*再在对话框上新建一个按钮(`Read`), 并添加`BN_CLICKED`事件. 写下代码: *

```cpp
void CtestXmlDlg::OnBnClickedBtmRead()
{
    // 读取XML
    CMarkup xml;
    xml.Load(L"F:\\Sample.xml");
    //MCD_STR strXML = xml.GetDoc();
    //xml.SetDoc(strXML);

    xml.FindElem(); // root ORDER element
    xml.IntoElem(); // inside ORDER
    CString outPut;
    while ( xml.FindElem( L"ITEM") )
    {
        xml.IntoElem();
        xml.FindElem( L"SN" );
        MCD_STR strSN = xml.GetData();
        xml.FindElem( L"QTY" );
        int nQty = _wtoi( MCD_2PCSZ(xml.GetData()) );
        outPut.Format(L"SN:%s\nQTY:%d",strSN,nQty);
        xml.OutOfElem();
    }

    MessageBox(outPut);
}
```

首先, 需要声明一个`CMarkup`对象, 然后用[Load]方法引入需要解析的XML文档. 或者使用[SetDoc]方法(见被注释的两行代码).

然后再循环调用[FindElem]与[GetData]来定位并获取标签内容. 这里如果需要的数字, 可以通过`atoi`来转化字符串, 如果在`Unicode`环境下, 需要用`_wtoi`来转换. [MCD_2PCSZ]在`Markup.h`中有定义, 返回一个字符串的const指针.

最后要注意的就是, 每一次定位到某标签, 需要调用[IntoElem]来进入子标签, 并继而通过[OutOfElem]跳出. 每一次循环一定由一对[IntoElem]和[OutOfElem]来包夹.

运行结果如下:

![result](/images/ReadXML.png)

## 添加标签及属性

上面创建的XML仅仅包含一个ITEM标签. 下面再举一个例子, 通过已有的数据源创建多个ITEM. 然后通过[SetAttrib]来为`SHIPMENT`标签设置一个属性.

在对话框上新建一个按钮(`Add`), 并添加`BN_CLICKED`事件. 写下代码:

```cpp
struct Items
{
    CString strSN;
    CString strName;
    int nQty;
};

void CtestXmlDlg::OnBnClickedBtnAdd()
{
    // 向XML中添加标签与属性
    CArray<Items,Items&> aItems;
    Items item1 = {L"132487A-J",L"crank casing",1};
    Items item2 = {L"4238764-A",L"bearing",15};
    aItems.Add(item1);
    aItems.Add(item2);

    CString strPOCType(L"non-emergency");
    CString strPOCName(L"John Smith");
    CString strPOCTel(L"555-1234");

    CMarkup xml;
    xml.AddElem(L"ORDER");
    xml.IntoElem(); // inside ORDER
    for(int nItem=0; nItem<aItems.GetSize(); ++nItem)
    {
        xml.AddElem( L"ITEM" );
        xml.IntoElem(); // inside ITEM
        xml.AddElem( L"SN", aItems[nItem].strSN );
        xml.AddElem( L"NAME", aItems[nItem].strName );
        xml.AddElem( L"QTY", aItems[nItem].nQty );
        xml.OutOfElem(); // back out to ITEM level
    }
    xml.AddElem( L"SHIPMENT" );
    xml.IntoElem(); // inside SHIPMENT
    xml.AddElem( L"POC" );
    xml.SetAttrib( L"type", strPOCType );
    xml.IntoElem(); // inside POC
    xml.AddElem( L"NAME", strPOCName );
    xml.AddElem( L"TEL", strPOCTel );

    xml.Save( L"F:\\Sample.xml" );
    MessageBox(L"Success!");
}
```

这段代码生成如下XML: 根标签`ORDER`包含了两个`ITEM`子标签和一个`SHIPMENT`标签. 该`ITEM`标签包括了`SN`,`NAME`和`QTY`子标签. `SHIPMENT`标签包含了一个`POC`子标签, `POC`拥有一个`type`属性, 和一个`NAME`、`TEL`子标签.

```xml
<ORDER>
<ITEM>
<SN>132487A-J</SN>
<NAME>crank casing</NAME>
<QTY>1</QTY>
</ITEM>
<ITEM>
<SN>4238764-A</SN>
<NAME>bearing</NAME>
<QTY>15</QTY>
</ITEM>
<SHIPMENT>
<POC type="non-emergency">
<NAME>John Smith</NAME>
<TEL>555-1234</TEL>
</POC>
</SHIPMENT>
</ORDER>
```

## 查找标签

`FindItem`方法默认查找**下一个**相邻的标签. 如果其参数指定了标签名称, 则一直向下查找, 直到得到匹配的标签. 找到标签的位置将被视为当前位置, 下一次调用`FindItem`方法时, 将以当前位置为开端, 继续向下查找能够匹配的标签位置.

在FindItem循环查找过程中, 如果不想继续顺序查找下去, 而是希望返回到第一个标签的位置, 可以用[ResetMainPos]. 例如上述例子中, 如果你无法确定`SN`标签是否在`QTY`标签之前(而你恰好又是先找的`QTY`), 那就可以用`ResetMainPos`方法返回重新查找. 代码如下:

```cpp
void CtestXmlDlg::OnBnClickedBtnFind()
{
    // 检索XML
    CMarkup xml;
    xml.Load(L"F:\\Sample.xml");
    //MCD_STR strXML = xml.GetDoc();
    //xml.SetDoc(strXML);

    xml.FindElem(); // root ORDER element
    xml.IntoElem(); // inside ORDER

    CString outXml;
    while ( xml.FindElem(L"ITEM") )
    {
        xml.IntoElem();
        xml.FindElem( L"QTY" );
        int nQty = _wtoi( MCD_2PCSZ(xml.GetData()) );
        xml.ResetMainPos();
        xml.FindElem( L"SN" );
        MCD_STR strSN = xml.GetData();
        xml.OutOfElem();

        outXml.Format(L"strSN:%s\nnQty:%d",strSN,nQty);
    }

    CString strFindSN(L"87890310-A");
    MCD_STR strSN;
    xml.ResetPos(); // top of document
    xml.FindElem(); // ORDER element is root
    xml.IntoElem(); // inside ORDER
    while ( xml.FindElem(L"ITEM") )
    {
        xml.FindChildElem( L"SN" );
        if ( xml.GetChildData() == strFindSN )
        {
            strSN = xml.GetChildData();
            break; // found
        }
    }

    xml.Save(L"F:\\Sample.xml");
    MessageBox(outXml+"\n"+strSN);
}
```

如果只是想找一个特殊的值(如上述代码中值为"87890310-A"的SN), 可以循环遍历`ITEM`标签, 并比对其子标签`SN`的值. 如果指定查找`ITEM`标签, 如上述代码, 那就不会去理其他标签, 如`SHIPMENT`等.

另外, 进出`ITEM`标签去对其子标签`SN`进行查找, 使用`FindChildElem`和`GetChildData`方法要便捷的多.

## 这就完了么？

额, 准确的说, 上述这点玩意儿, 只是最最最基本的操作. 不过完全可以看出`CMarkup`类的便捷简单了吧？利用`CMarkup`操作XML虽说已经是很成熟的技术了, 但它的功能并不是太全面. 这只是一种轻量级的工具, 合适与否完全看个人需求了. 更多的方法API请见[这里].

## 参考资料

- [1] [Fast start to XML in C++](http://www.firstobject.com/fast-start-to-xml-in-c++.htm)
- [2] [CMarkup Methods](http://www.firstobject.com/dn_markupmethods.htm)

[ResetPos]: http://www.firstobject.com/dn_markResetPos.htm
[Load]: http://www.firstobject.com/dn_markLoad.htm
[GetDoc]: http://www.firstobject.com/dn_markGetDoc.htm
[MCD_STR]: http://www.firstobject.com/dn_markmcdstr.htm
[SetDoc]: http://www.firstobject.com/dn_markSetDoc.htm
[FindElem]: http://www.firstobject.com/dn_markFindElem.htm
[GetData]: http://www.firstobject.com/dn_markGetData.htm
[MCD_2PCSZ]: http://www.firstobject.com/dn_markunifiedstlmfc.htm
[IntoElem]: http://www.firstobject.com/dn_markIntoElem.htm
[OutOfElem]: http://www.firstobject.com/dn_markOutOfElem.htm
[SetAttrib]: http://www.firstobject.com/dn_markSetAttrib.htm
[ResetMainPos]: http://www.firstobject.com/dn_markResetMainPos.htm
[这里]: http://www.firstobject.com/dn_markupmethods.htm
