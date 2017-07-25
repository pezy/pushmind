---
layout: post
title: 关于Unicode编码
categories: [windows]
description: 闲时翻译
keywords: C++, Windows
---

很多Windows\C++的初学者对于`TCHAR`, `LPCTSTR`这些怪异的符号很头疼. (我有一个能力很强的同事便是如此, 甚至因为这个丧失了对Windows开发的兴趣, 转去做Java了.)今天就把这些玩意一探究竟吧.

## 简述ANSI与Unicode

显示某个字符, 可以用单字节存储, 也可以用双字节存储. 前者便是我们常见的ANSI编码策略, 适用于大多数的英系字符；后者则是Unicode编码策略, 几乎可以用以表示世界上所有的文字.

在VC++编译器中, 对应于以上两种编码方式, 分别给出了`char`与`wchar_t`两种数据类型. *Unicode*还有诸多好处, 但现在仅需知道它作为一种双字节存储方式可以更好的支持Windows程序的国际化.

> 不止是Unicode, Windows还会用到更多别的双字节编码, 如默认使用的UTF-16字符编码.

作为程序员, 一定希望自己的C/C++代码适用于所有的字符编码吧?

**建议**：使用通用的数据类型来表示字符与字符串. 举个例子吧, 请把

    char cResponse; // 'Y' or 'N'
    char sUsername[64];
    // str* functions
和
    wchar_t cResponse; // 'Y' or 'N'
    wchar_t sUsername[64];
    // wcs* functions
替换成：
    #include<TCHAR.H> // Implicit or explicit include
    TCHAR cResponse; // 'Y' or 'N'
    TCHAR sUsername[64];
    // _tcs* functions

这样就照顾了多语言环境的需求(就像Unicode), 是更加通用的一种形式.

在实际编码的时候, 如在VS2010的环境下, 可以这样设置编译时所用的字符集：(*常规->字符集*)

![setUnicode](/images/setUnicode.png)

如上图所示, 若设置为Unicode, `TCHAR`将视为`wchar_t`；若设置为多字节, `TCHAR`将被视为`char`. 项目的具体设置不会影响到`wchar_t`或是`char`类型的使用了. 为啥会这样呢, 请看`TCHAR`的定义：

    #ifdef _UNICODE
    typedef wchar_t TCHAR;
    #else
    typedef char TCHAR;
    #endif

`_UNICODE`宏的作用就是当项目"*设置为Unicode字符集*"时, `TCHAR`含义是`wchar_t`；"*设置为多字节字符集*"时, `TCHAR`含义是`char`.

与上述类似, 为了支持各字符集都能使用最基本的函数, 最好使用`_tcscpy, _tcslen, _tcscat`函数来代替`strcpy, strlen, strcat`(出于安全的考虑, 常会在后面加上`_s`)或`wcscpy,wcslen,wcscat`(已经考虑了安全性).

strlen的原型如下：

    size_t strlen(const char*);

wcslen的原型如下：

    size_t wcslen(const wchar_t* );

最好使用_tcslen, 从*逻辑*上可将其原型表述为：

    size_t _tcslen(const TCHAR* );

**wc**表示宽字符, 可知`wcs`表示宽字符串, 同理可知`_tcs`表示_T字符串, 能猜到_T在逻辑上既代表`char`又代表`wchar_t`了吧?

不过, 实际上, `_tcslen`(还有其他`_tcs`前缀的函数)并**不是**一个拥有完整定义的真实函数, 而仅仅是一个**宏**. 其定义应该长这样子：

    #ifdef _UNICODE
    #define _tcslen wcslen
    #else
    #define _tcslen strlen
    #endif

在**`TCHAR.H`**中, 会找到与之相似的准确定义.

> 为何定义成宏, 而不是直接定义为函数接口呢?

原因很简单, lib或是dll只能导出单一的函数名与参数类型(这里不考虑C++的重载). 例如, 你导出的是：

    void _TPrintChar(char);

而应用程序正好需要用到

    void _TPrintChar(wchar_t);

该怎么办呢?单字节参数不会无缘无故的变成双字节. 其实那是两个不同的函数：

    void PrintCharA(char); // A = ANSI
    void PrintCharW(wchar_t); // W = Wide character

如果定义一个宏, 就轻而易举的解决了这个问题：

    #ifdef _UNICODE
    void _TPrintChar(wchar_t);
    #else
    void _TPrintChar(char);
    #endif

届时, 应用程序只需如此调用：

    TCHAR cChar;
    _TPrintChar(cChar);

宏避免了两种字符集的共存, 并允许我们可以运用ANSI或Unicode编码的函数来处理各类字符或字符串. 大多数windows函数都采取了这样的措施, 为了简化程序员的工作, 只写一个函数(用宏去转换)是非常棒的解决方法. `SetWindowText`便是一个典型的例子：

    // WinUser.H
    #ifdef UNICODE
    #define SetWindowText  SetWindowTextW
    #else
    #define SetWindowText  SetWindowTextA
    #endif // !UNICODE

极少数的函数没有采用宏, 而是仅仅以**W**或**A**后缀作为区分. 例如`ReadDirectoryChangesW`, 它并没有ANSI编码的等价函数.

## ANSI与Unicode的转换

我们一般习惯用双引号来标记字符串. 其实那是ANSI字符串的用法, 其中每个字符均为单字节存储, 例如：

    "This is ANSI String. Each letter takes 1 byte"

上述字符串**不支持**Unicode, 对多语言环境的支持有限. 想要表现为Unicode编码, 你需要用到`L`前缀. 例如：

    L"This is Unicode string. Each letter would take 2 bytes, including spaces."

注意, **L**在字符串的前面, 表明这是Unicode字符串. 其中**所有**的字符都是双字节存储, 包括英文字符、空格、数字、甚至是空字符. 因此, Unicode字符串占用的空间将永远是2-字节的倍数. 一个长度为7的Unicode字符串需要14字节, 诸如此例. 所以, 永远不存在一个占用15个字节的Unicode字符串.

更通用的说法是, 字符串所占空间都是`sizeof(TCHAR)`个字节的倍数.

当需要表示一个硬编码的字符串时, 可以这样：

    "ANSI String"; // ANSI
    L"Unicode String"; // Unicode

    _T("Either string, depending on compilation"); // ANSI or Unicode
    // or use TEXT macro, if you need more readability

其中, 无前缀的是ANSI字符串, **L**前缀的是Unicode字符串, 以`_T`或`TEXT`标识的, 将根据宏定义而定. 还是一样, `_T`和`TEXT`只是宏而已, 它们定义如下：

    // SIMPLIFIED
    #ifdef _UNICODE
     #define _T(c) L##c
     #define TEXT(c) L##c
    #else
     #define _T(c) c
     #define TEXT(c) c
    #endif

这`##`符号是[符号连接操作符], 它将`_T("Unicode")`替换为`L"Unicode"`, 该替换取决于宏参数——`_UNICODE`是否被定义. 如果没有被定义, `_T("Unicode")`意味着`"Unicode"`. *符号连接操作符*并不仅仅是VC或字符编码中的特定符号, 它甚至存在于C语言中.

注意, 该宏既适用于字符串, 也适用于字符. 例如`_T("R")`将被转换为`L"R"`或`"R"`. 前者为Unicode字符, 后者为ANSI字符.

**但, 这无法转换变量(字符或字符串)**. 以下代码就是非法的：

    char c = 'C';
    char str[16] = "MyProject";

    _T(c);
    _T(str);

最后两句代码可以在ANSI(多字节)下编译成功, 因为`_T(x)`就是`x`, 因此`_T(c)`和`_T(str)`将分别输出`c`和`str`. 但在Unicode字符集下编译, 则会报错：

    error C2065: 'Lc' : undeclared identifier
    error C2065: 'Lstr' : undeclared identifier

我们需要注意所有的字符、字符串操作的函数, 尤其是windows API提供的那些, 基本都是MSDN推荐的典范. 拿`SetWindowsTextA/W`来说吧：

    BOOL SetWindowText(HWND, const TCHAR*);

想必应该知道了, `SetWindowText`仅仅是一个宏, 它取决于你的编译配置, 其含义可以是以下其中一个：

    BOOL SetWindowTextA(HWND, const char*);
    BOOL SetWindowTextW(HWND, const wchar_t*);

因此, 不要困惑于下面这段取地址函数为啥调用失败了！

    HMODULE hDLLHandle;
    FARPROC pFuncPtr;
    hDLLHandle = LoadLibrary(L"user32.dll");
    pFuncPtr = GetProcAddress(hDLLHandle, "SetWindowText");
    //pFuncPtr will be null, since there doesn't exist any function with name SetWindowText !

`SetWindowTextA`和`SetWindowTextW`都由`User32.dll`所导出的. 并没有通用的函数名.

有趣的是, 在.Net 框架下总算有了一个通用的函数：

    [DllImport("user32.dll")]
    extern public static int SetWindowText(IntPtr hWnd, string lpString);

如果没有技术的改进, `GetProcAddress`怕是还会被一群*if..else*语句包围吧.

所有的函数都有ANSI和Unicode两个版本, 但**实际上却都是由Unicode版本来实现**的. 这意味着：当你调用`SetWindowTextA`时, 传入一个ANSI字符串, 编译器会先将该字符串转换为Unicode字符串, 然后调用`SetWindowTextW`. 类似这样的实际操作(设置窗口的标题、内容及名称等)将都由Unicode版本的函数来执行！

再举一个例子, 获取窗口内容将用到`GetWindowText`. 你调用`GetWindowTextA`时, 若目的是得到一个ANSI的buffer. `GetWindowTextA`将先调用`GetWindowTextW`, 并得到一个Unicode字符串(一个`wchar_t`数组). 然后为你把Unicode字符串转换为ANSI字符串.

ANSI与Unicode互转的操作并不局限于GUI函数, 而涵盖了整个Windows API系列中拥有两套方案的字符串处理函数. 例如：

* `CreateProcess`
* `GetUserName`
* `OpenDesktop`
* `DeleteFile`
* etc

这就是为何都推荐直接使用Unicode版本函数的原因了. 不要仅仅因为多年的习惯, 而固守ANSI不放, 试着用Unicode来编译吧. 但, 我们可能还是会去存储或读取ANSI字符串, 尤其在某些文件操作和消息传递中. 这些转换函数还是有其存在的意义的.

**注意**：还有一种常见的定义类型：**`WCHAR`**, 它等价于`wchar_t`.

## 更好的指针表示

`TCHAR`往往修饰单个字符, 当然可以声明一个`TCHAR`的数组. 如果你想表示一个*character-pointer*或一个*const-character-pointer*, 会用下列哪一个呢?

    // ANSI characters
    foo_ansi(char*);
    foo_ansi(const char*);
    /*const*/ char* pString;

    // Unicode/wide-string
    foo_uni(WCHAR*);
    wchar_t* foo_uni(const WCHAR*);
    /*const*/ WCHAR* pString;

    // Independent
    foo_char(TCHAR*);
    foo_char(const TCHAR*);
    /*const*/ TCHAR* pString;

看过`TCHAR`的解释, 应该会很明确的选择最后一种了吧. 那样表示字符串最有效. 别忘了引入windows.h头文件. **注意**：如果你的项目直接或间接的引入了windows.h, 就不需要引入`TCHAR.H`了.

首先, 再来更好的理解一下老式的字符操作函数, 看`strlen`：

    size_t strlen(const char*);

可以表示为：

    size_t strlen(LPCSTR);

**`LPCSTR`**符号的定义类型是：

    // Simplified
    typedef const char* LPCSTR;

其含义拆解如下：

* **LP** – Long Pointer
* **C** – Constant
* **STR** – String

`LPCSTR`的本质含义是指向固定字符串的(长)指针.

用新的形式来表述`strcpy`就是：

    LPSTR strcpy(LPSTR szTarget, LPCSTR szSource);

**szTarget**的类型是`LPSTR`,少了**C**, 定义如下：

    typedef char* LPSTR;

注意**szSource**的类型是`LPCSTR`,`strcpy`不能改变原字符串, 所以要用`const`限定. 返回类型是无const限定的`LPSTR`.

这些`str-`函数都是处理ANSI字符串的. 但通常我们需要操作双字节字符串, 等价的宽字符str函数同样给出了, 例如, 若计算一个宽字符数组(Unicode字符串)的长度, 可以用**`wcslen`**：

    size_t nLength;
    nLength = wcslen(L"Unicode");

`wcslen`函数原型：

    size_t wcslen(const wchar_t* szString); // Or WCHAR*

也可以替换为

    size_t wcslen(LPCWSTR szString);

这里的**`LPCWSTR`**定义为：

    typedef const WCHAR* LPCWSTR;
    // const wchar_t*

拆开来看：

* **LP** - Pointer
* **C** - Constant
* **WSTR** - Wide character String

类似地, **`wcscpy`**与`strcpy`意义相同, 只不过针对的是Unicode字符串：

    wchar_t* wcscpy(wchar_t* szTarget, const wchar_t* szSource)

也可以表现为：

    LPWSTR wcscpy(LPWSTR szTarget, LPCWSTR szSource);

其目标字符串为非常量宽字符串(`LPWSTR`), 源字符串为常量宽字符串.

存在`wcs-`函数对应`str-`函数, 前者处理Unicode字符串, 后者处理ANSI字符串.

虽然我已经建议过, 直接用Unicode的函数替代ANSI与TCHAR的函数. 原因很简单, 应用程序是必须使用Unicode的, 且无须考虑ANSI的兼容性. 但出于完整性的考虑, 还是继续说明对应的通TCHAR函数吧.

计算字符串长度, 可以用**`_tcslen`**函数(一个宏). 通常定义如下：

    size_t _tcslen(const TCHAR* szString);
或
    size_t _tcslen(LPCTSTR szString);

这里的**`LPCTSTR`**可分解为：

* LP - Pointer
* C - Constant
* **T = TCHAR**
* STR = String

根据项目设置, `LPCTSTR`可以映射为`LPCSTR`(ANSI)或是`LPCWSTR`(Unicode).

**注意**: `strlen`, `wcslen`或`_tcslen`返回的是**字符**的个数, 而不是所占字节数.

仍是拷贝字符串函数, 这里为**`_tcscpy`**, 定义如下：

    size_t _tcscpy(TCHAR* pTarget, const TCHAR* pSource);
或, 更加通用的形式：
    size_t _tcscpy(LPTSTR pTarget, LPCTSTR pSource);

可以推断出**`LPTSTR`**的含义了吧！

## 用法举例：

首先, 看一个代码片段：

```cpp
int main()
{
    TCHAR name[] = "Saturn";
    int nLen; // Or size_t

    lLen = strlen(name);
}
```

在ANSI字符集下编译完美, 因为`TCHAR`转换为`char`,因此参数`name`恰好是`char`字符数组.
如果在Unicode/_UNICODE被定义(或项目设置了Unicode字符集)的情况下, 就会报错：

    error C2440: 'initializing' : cannot convert from 'const char [7]' to 'TCHAR []'
    error C2664: 'strlen' : cannot convert parameter 1 from 'TCHAR []' to 'const char *'

对于第一个错误, 程序员们开始如此修正了：

    TCHAR name[] = (TCHAR*)"Saturn";

这不可能好使, 因为不可能从`TCHAR*`转换为`TCHAR[7]`. 同样的错误还会出现在, 试图将ANSI字符串作为参数传递给Unicode函数：

    nLen = wcslen("Saturn");
    // ERROR: cannot convert parameter 1 from 'const char [7]' to 'const wchar_t *'

不幸的是(或谓之幸运?), 错误可以通过下面这般C风格的类型转换而修正：

    nLen = wcslen((const wchar_t*)"Saturn");

此刻, 你会不会觉得在自己对指针的理解又深入了一层?大错特错！这代码将会返回一个错误的结果, 并且大多数情况下, 会造成访问冲突. 这样的套路, 就像是本来需要一个80个字节的结构, 却传入了一个`float`变量(从逻辑上而言).

**`"Saturn"`**是一个7字节的顺序序列：

![stringTable](/images/stringTable.png)

但当你把这一套传递给`wcslen`时, 它会将每两个字节看做一个字符, 因此, 前两位[97,83]将被当做一个值为24915(`97<<8 | 83`)的字符, 在Unicode里是:`?`. 下一个字符则是[117,116], 以此类推.

你的确没有打算把一串中文字符作为参数, 但错误的类型转换却干了这事！因此, 要深刻的明白强制转换是**祸根**！正确的做法是将第一行改为：

    TCHAR name[] = _T("Saturn");

这样源字符串就会根据编译设置, 转换为7字节或是14字节. 而`wcslen`函数应该这样使用：

    wcslen(L"Saturn");

在上述示例代码中, 用的是`strlen`, 那在Unicode环境下无法通过编译. 错误的解决方法又来了：

    lLen = strlen ((const char*)name);

在Unicode环境下编译, name将占用14字节(7个Unicode字符, 包括null). 因为字符串**"Saturn"**仅包含英文字符, 所以用的是原生的ASCII来表现. Unicode字符`'S'`将表现为[83,0],其他字符也都会接上一个零. 注意, 现在`'S'`表现为一个值为83的**双字节**字符, 字符串的末端也将表现为一个值为`0`的**双字节**字符.

所以当把这样的一个字符串传递给`strlen`时, 第一个字符(第一个字节)将是正确的(如"Saturn"中的`'S'`). 但第二个字符/字节将导致字符串的结束. 因此, `strlen`将返回一个错误的值(`1`)作为字符串的长度.

Unicode字符串是可以包含非英文字符的, 那样的结果就是无法预测的了.

简言之, 强制类型转换不总是好使的. 或者选择用其本身的类型来正确表示, 或者对Unicode与ANSI进行常规的转换.

## 字符数与字节数

现在, 应该可以理解下面的语句了：

    BOOL SetCurrentDirectory( LPCTSTR lpPathName );
    DWORD GetCurrentDirectory(DWORD nBufferLength,LPTSTR lpBuffer);

继续说, 你肯定见过某些函数/方法中需要传入**字符数**, 或是返回字符数吧. 譬如`GetCurrentDirectory`, 你需要传入的是字符数, 而**不是**字节数. 例如：

    TCHAR sCurrentDir[255];

    // Pass 255 and not 255*2
    GetCurrentDirectory(sCurrentDir, 255);

而另一方面, 如果你需要为数字或字符数组分配空间, 则一定要分配整的字节数. 在C++中, 常使用`new`关键字：

    LPTSTR pBuffer; // TCHAR*

    pBuffer = new TCHAR[128]; // Allocates 128 or 256 BYTES, depending on compilation.

但如果用的是内存分配函数如`malloc`,`LocalAlloc`,`GlobalAlloc`等等, 你必须指定其字节数！

    pBuffer = (TCHAR*) malloc (128 * sizeof(TCHAR) );

这里将返回值强制转换是很有必要的, 原因自己领悟. `malloc`语句内的表达式保证分配了其需要的字节数, 并为所要求的字符数分配了空间.

**这篇文章翻译自[What are TCHAR, WCHAR, LPSTR, LPWSTR, LPCTSTR (etc.)?], 由于时间仓促以及水平有限, 必然存在某些错误, 还请高手指正!**

[符号连接操作符]: http://msdn.microsoft.com/en-us/library/09dwwt6y(v=vs.80).aspx
[What are TCHAR, WCHAR, LPSTR, LPWSTR, LPCTSTR (etc.)?]: http://www.codeproject.com/Articles/76252/What-are-TCHAR-WCHAR-LPSTR-LPWSTR-LPCTSTR-etc
