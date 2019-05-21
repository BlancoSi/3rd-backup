# windows下的C++链接库：动态链接库与静态链接库

标签： C++ 库

---
[引用](https://blog.csdn.net/sinat_22991367/article/details/79436028)的一个例子（具体调用实例可以参考配置opencv环境的文章）：
```
//.cpp文件
int __stdcall Add(int numa, int numb){return (numa + numb);}
int __stdcall Sub(int numa, int numb){return (numa - numb);}
```
```
;.def文件
;DllTestDef.lib : 导出DLL函数
LIBRARY DllTestDef
EXPORTS 
    Add @ 1
    Sub @ 2
```
```
//.cpp文件
#include <iostream>
#include <windows.h>
using namespace std;
typedef int (__stdcall *FUN)(int, int);
//定义一个名叫FUN的函数指针，返回值为int，参数为两个int
HINSTANCE hInstance;
//HINSTANCE为句柄类型，用于标示（记录）一个程序的实例
FUN   fun;
int main()
{
       hInstance = LoadLibrary("DLLTestDef.dll");
       //1 LoadLibrary函数
       if(!hInstance)
           cout << "Not Find this Dll" << endl;
       fun = (FUN)GetProcAddress(hInstance, MAKEINTRESOURCE(1));
       //这里用了一个宏MAKEINTRESOURCE()
       //2 GetProcAddress函数
       if (!fun)
       {
              cout << "not find this fun" << endl;
       }
       cout << fun(1, 2) << endl;
       FreeLibrary(hInstance);
       //3 FreeLibrary函数
       return 0;
}
```

> 注：这里include windows.h是为了用LoadLibrary等三个函数显式调用dll文件

## 前言：
由于人的智力水平的限制，当一个函数中包含了太多的语句时，便不太容易被理解，这时候开始需要子函数。同样的道理，一个源文件中包含了太多的函数，同样不好理解，人们开始分多个源文件了。
这时就要用到分别编译的库文件，也即lib文件（实际上是任意个obj文件的集合），等到最后才被链接到一起形成应用程序。
而另一种方式就是动态链接库，即dll。类似于exe程序，dll文件的入口函数是DllMain，但dll中的入口函数用途一般只是导出函数，所以推荐手动清理内存。但是，即使不写出入口函数DllMain，编译也可以成功，是因为编译器自动补全了此入口函数。
C和C++都可以生成链接库，链接库分为两种：动态链接库与静态链接库。
windows下与生成、使用链接库相关的文件有.dll .lib .h文件，linux下对应.dll .lib为.so .a等。此处不讨论mfc的dll规则。

## 重点：
1. 使用动态链接库生成文件时，需要对应的.lib文件与.h文件，其中lib文件会被链接到你的文件中，运行时只需要.dll文件。
2. 动态链接库的.lib文件是【引入库函数】，包含了.dll文件中函数的地址，用于用引入库的方式加载动态链接库。.dll文件包含【实际的函数内容】，.dll文件可以由程序在运行时装载和卸载。而且单个文件在运行时【按情况可以实例化多次】。
3. 静态链接库的.lib文件是把【实际的函数内容】也封装入了的【引入库函数】，会被直接链接到程序中，仅是方便了编写程序。并没有【按情况可以实例化多次】节约空间和内存的功能。
4. 关于头文件，既是用于说明输出的类或符号原型或数据结构的.h文件。当其它应用程序调用dll时，需要将该文件包含入应用程序的源文件中。
5. 链接库的源文件可以不包含入口函数，编译时会自动补全。.def文件是生成的时候用的，可以替代_declspec(dllexport)关键字。

## 正文：

## 1. 动态链接库
### 1.1 动态链接库的生成：
dll文件内有两种函数，一种为导出函数，供外部调用，另一种不导出，仅供dll内其他函数调用。只有将函数表示为导出才能被外部调用。
声明导出函数方式：

> **1.编写.def文件**【**.def文件是生成的时候用的，会生成.lib文件**】【可以防止导出的函数名被编译器改变导致无法链接】【可以手动隐藏函数名】【因为C++中的函数重载机制把函数名字重新编码了，所以这样可以限定函数名】[详解参考](http://www.cppblog.com/FateNo13/archive/2009/08/24/94224.html)


.def格式例如：
```
;这里是注释
LIBRARY Dllname
;上面对应dll文件Dllname.dll
EXPORTS
    fun1 =f1 @1 NONAME
    fun2 @2
;上面fun1和fun2是dll文件可以被其他应用调用的函数的源码内名称
;f1为导入hdll后使用的名称
;【可选】@后数字必须为从1到N且顺序，N为dll导出函数数量【说法不一】
;【可选】再加NONAME关键字，可以隐藏函数名用序号代替
;【可选】PRIVATE关键字，禁止将导出的函数名或变量名放到导入库中
;【可选】DATA关键字，指定导出的是数据，而不是代码
```
> **2.在函数声明前增加_declspec(dllexport)关键字**【另外注意目标函数声明时返回类型与函数名间可以加的类似__stdcall参数不加会被编译器设为默认值】【不需要重载的话再在前面增加extern "C"可以防止导出的函数名被编译器改变】【_declspec(dllimport)表示函数为导入的的在dll中的函数】【此种方法导出会暴露函数名】
**其中：**__stdcall等参数的解释见[度娘](https://baike.baidu.com/item/__stdcall/9466040?fr=aladdin)，一般windows API用的都是__stdcall，所以最好用这个保持一致性减少报错。


### 1.2 动态链接库的调用：

注意：windows下有一种错误叫“一个模块中分配的内存在另一个模块中释放”。原因是DLL与EXE分属两个模块，分配和释放内存不是由相同的堆管理程序完成的，如果主程序未释放dll函数分配的内存直接return，就会导致崩溃。[详情见此](https://blog.csdn.net/kendyhj9999/article/details/9746371)。所以应当注意在设计DLL接口时遵循谁分配，谁释放的原则。考虑：显示加载时，调用析构函数、在main函数return前用FreeLibrary释放dll函数分配的内存。

**有两种调用方法：**

 一、显式调用：显式引入库

**动态调用**，通过Windows API（windows.h）的LoadLibrary和GetProcAddress和FreeLibrary函数，实现动态载入，即在编译之前并不知道将会调用哪些 DLL 函数， 完全在运行过程中根据需要决定调用哪些函数。
 > **方法是：**
用 LoadLibrary 函数加载动态链接库到内存，用 GetProcAddress函数动态获得 DLL 函数的入口地址。当一个 DLL 文件用 LoadLibrary 显式加载后，在任何时刻均可以通过调用 FreeLibrary 函数显式地从内存中把它给卸载。
[用法详见于此](https://www.cnblogs.com/westsoft/p/5936092.html)或参考开头实例。
 > 
**注意：**如果是没有def文件的c++dll（导出函数没有extern“C”）的话，显式调用的函数名一般并不是源文件定义的函数名而是编译器重载过的函数名。但是用模块定义文件 (.def)的方式调用，由于定义的函数名称，则没有问题。名称问题（例如?MyFunction@@YGHH@Z）可以用预处理指示符#pragma comment(linker, "/EXPORT:MyFunction=?MyFunction@@YGHH@Z")解决
**两种引出方法注意：**
1. dll中需要引出的函数前才加上 __declspec(dllexport) 关键字。
2. 使用传统的模块定义文件 (.def)，函数名为.def中定义过的函数名。


 
二、隐式调用：直接包含头文件，用引入库的方法加载

**静态调用**，不使用Windows API（windows.h），例如用#pragma comment(lib,"xxx.lib")【或者说链接引入库】，然后用__declspec(dllimport)【声明】导入函数【这里声明需要与dll工程中.h文件中的函数声明一致】
**注意：**这里有两种选择：
1） 将dll工程中的头文件include进来：此时库中函数名可直接使用

 > 如果用引入库(*.LIB)的方式调用，则编译器会自动处理转换函数名，include对应头文件后可以直接使用库函数。
**方法：**
1. 首先你的程序中要include链接库的.h头文件
2. 然后在ide设置找到链接库.lib或者加上预处理指示#pragma comment(lib,xxx.lib)
3. 再后把.dll文件放到编译器变量或者环境变量里
4. 最后启动生成，这样隐式调用就完成了

2） 不include头文件：在引入函数前写extern “C”，或者根据重载规则更改引入函数名

> 引用：静态调用不需要使用Win32API函数来加载和卸载Dll以及获取Dll中导出函数的地址，这是因为当通过静态链接方式编译生成程序时，编译器会将.lib文件中导出函数的函数符号链接到生成的exe文件中，.lib文件中包含的与之对应的dll文件的文件名也被编译存储在exe文件内部，当应用程序运行过程中需要加载dll文件时，windows将根据这些信息查找并加载dll，然后通过符号名实现对dll函数的动态链接，这样，exe将能直接通过函数名调用dll 的输出函数，就像调用程序内部的其他函数一样。

注意：只有声明过为导出函数的才能导入
[引用参照此处](https://blog.csdn.net/w_y2010/article/details/80428067)

## 2.静态链接库
### 创建：
和动态链接库类似，只是不需要入口函数和__declspec(dllexport)。
直接创建静态库工程或者空项目配置为生成静态库写好函数直接生成就可以。
### 使用：
和dll的隐式调用没区别，先把lib加到链接的选项里或者#pragma comment() 然后直接声明导入函数或者include头文件就可以用了。

### [linux下的内容可以参考这里](https://www.cnblogs.com/nufangrensheng/p/3578784.html)