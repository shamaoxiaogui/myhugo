+++
Categories = ["Development"]
Description = "深入理解cpp对象模型 第六章笔记"
Tags = ["Development","c++"]
date = "2016-07-29T09:05:34+08:00"
title = "深入理解cpp对象模型 第六章笔记"

+++

## 全局变量的初始化问题。
考虑在如下代码中，全局和静态变量都放在哪个段中，初始化顺序又如何

```cpp
#include <iostream>
#include <string>
using namespace std;
class nonPOD{
public:
    nonPOD(const string& s="default"):str(s){cout<<str<<" nonPOD ctor"<<endl;}
    nonPOD(const nonPOD& t):n(t.n),str(t.str+"_copy"){cout<<str<<" copy ctor"<<endl;}
    virtual ~nonPOD(){cout<<str<<" nonPOD dctor"<<endl;}
    int getn(){return n;}
private:
    int n;
    string str;
};
struct POD{
    int i;
    char ch;
};

nonPOD np;
nonPOD np1("hehe");
nonPOD np2(np1);
POD p;
POD p1={1,'c'};
int gzn;    //bss
int gn=4;   //data

int main(){
    cout<<"main start"<<endl;
    static int szn;
    static int sn=5;
    cout<<"gzn: "<<gzn<<" gn: "<<gn<<" szn: "<<szn<<" sn: "<<sn<<endl;
    static nonPOD snp;
    static nonPOD snp1("static hehe");
    static POD sp;
    static POD sp1={1,'c'};
    cout<<"np.n: "<<np.getn()<<endl;
    cout<<"np1.n: "<<np1.getn()<<endl;
    cout<<"np2.n: "<<np2.getn()<<endl;
    cout<<"snp.n: "<<snp.getn()<<endl;
    cout<<"snp1.n: "<<snp1.getn()<<endl;
    cout<<"p.i: "<<p.i<<endl;
    cout<<"sp.i: "<<sp.i<<endl;
    cout<<"main stop"<<endl;
    return 0;
}
```
```shell
$ ./segment.out
default nonPOD ctor
hehe nonPOD ctor
hehe_copy copy ctor
main start
gzn: 0 gn: 4 szn: 0 sn: 5
default nonPOD ctor
static hehe nonPOD ctor
np.n: 0
np1.n: 0
np2.n: 0
snp.n: 0
snp1.n: 0
p.i: 0
sp.i: 0
main stop
static hehe nonPOD dctor
default nonPOD dctor
hehe_copy nonPOD dctor
hehe nonPOD dctor
default nonPOD dctor
```

这个信息量，呵呵。。。好了，我们来分析一下：

1. 全局变量的构造，一定是在main函数开始之前完成，而静态变量是从第一次定义它的时候才构建，这个是没有异议的。
1. 析构顺序，全局对象与静态对象算在一起，按与它们构建相反的顺序析构。
1.
但是构建的顺序，这里就有坑了。上面的代码是在同一个文件中，全局变量似乎确实是按照定义的顺序来构造的。但是考虑这样一种情况，有一部分全局变量分散在其它文件中（比如库），本文件使用extern声明它们，那这时候构造顺序是什么样子的？好吧，我没有去实验，而且各家的编译器标准也不太一样，但是可以认为，至少在03标准中，全局变量的构建顺序是不确定的，唯一确定的就是上面说的，在main之前能构造完。这样就会产生一个问题，一旦某个全局变量的构造依赖于另一个全局变量，不能保证另一个对象在本对象构造前已经构造好，这就会造成问题。所以很多人建议，要不不要在全局对象中引入依赖（不太可能），要不使用静态变量代替（或多或少也会有这种问题），或者将这些有依赖的全局对象之间的依赖关系，选一个特定的过程来构造它们，比如说构建静态变量的单例模式。

高能预警分割线

---
回到开篇的问题，这些对象都放在哪个段？回忆一下c语言，“全局和静态初始化变量放在全局数据区，全局和静态未初始化变量放在BSS段”，好吧，其实不是这么简单的。首先来想一下，全局数据段是干嘛的。当我们使用unix下的exec系的调用时，操作系统加载可执行文件，从文件中读入DATA段，然后DATA段之间放入内存，也就是说是一种直接加载的模式。而BSS段，它不占用可执行文件的空间，因为系统加载这段时，只是读取大小然后在内存中开辟一段这个大小的内存，然后清零就好了。所以说所谓的“全局和静态未初始化变量”是有初始值的，其内存填0。
那我们通过nm工具来查看下符号表：

```shell
...
000000010000318c d __ZZ4mainE2sn
00000001000032b8 b __ZZ4mainE2sp
0000000100003258 b __ZZ4mainE3snp
0000000100003190 d __ZZ4mainE3sp1
0000000100003250 b __ZZ4mainE3szn
0000000100003288 b __ZZ4mainE4snp1
...
0000000100000000 T __mh_execute_header
0000000100003188 D _gn
0000000100003248 S _gzn
0000000100000e00 T _main
00000001000031c8 S _np
00000001000031f0 S _np1
0000000100003218 S _np2
0000000100003240 S _p
0000000100003180 D _p1
                 U _strlen
                 U dyld_stub_binder
```
这里只贴一小部分分析。简单解释下符合，D代表DATA段，T代表TEXT段，U代表未在本文件定义，也就是外部符号，而S，手册上说的是“不在以上所有段中”，之前的man page中说的是未初始化数据段，用于small object，似乎和B代表的BSS段一样啊。在之前的代码中，我们能确定的就是gzn一定是在BSS段，gn一定是在DATA段，再观察符号表，发现gzn对应着S，所以我们就当S是BSS段好了。

回到正题。全局和静态对象放在哪，首先我们不考虑POD，关注代码中的np＊和snp＊，发现不是S就是b，居然全都是BSS段。而POD类型，p和sp对应b或S放在BSS段，它们都是默认初始化的；而p1和sp1则都放在DATA段，是显示初始化的。

好吧，跟普通内置类型不一样啊。实际上，所有的全局或者静态对象（包括内置变量）在初始化的过程中，标准定义了以下几个阶段：

1. 静态初始化阶段，这个阶段又细分为以下两个阶段：
    1. 零初始化阶段：系统为对象准备内存，清零
    1. 常量初始化阶段：在清零的内存中加载可执行文件中已定义好的数据
1. 动态初始化阶段：完成一些不能依靠简单的加载可执行文件就能做到的功能。

我们一一举例。

1. 零初始化阶段，准备并清零内存，是不是很熟悉？BSS段么，所以BSS段中的对象就只执行静态初始化中的零初始化阶段，没有常量初始化阶段。
1. 常量初始化阶段。比如定义一个全局变量：`int x=5;`，按照标准而言，系统会先在内存中为x清零一段sizeof(x)的内存（零初始化阶段），然后将可执行文件中的5加载到这个内存中（常量初始化）。蛋疼不？反正最后得加载内存，为啥还要清零？所以很多系统、编译器在这里跳过零初始化阶段，编译生成可执行文件时把这种可以直接加载到内存而不需要提前清零的全局或者静态变量放到DATA段里，系统就只需要一步常量初始化就OK了。
1. 动态初始化阶段，比如定义一个全局变量，其初始值不能在编译期决定`int x=getInt();`，因为编译器不知道你这个getInt函数会返回什么值，它的值是在运行期动态决定的，这时候就需要动态初始化了。详细来讲，对于这个x，先清零内存（零初始化），再在main之前的startup代码中执行getInt来初始化它的值（动态初始化）。所以这个x也应该放到BSS中。

好了，现在我们来看类。一个全局活着静态类，如果不是POD，那必须用其构造函数来初始化，所以这就决定了它们必然会经历动态初始化阶段，所以我们上面看到的np＊和snp＊都在BSS段中。这些类的初始化过程是：

> 静态初始化（只执行零初始化阶段，清零内存） --> 动态初始化（执行构造函数）

对可执行文件中段的占用也就是bss（内存数据占用）＋init（或者finit、ctor等构造函数占用）段

而POD类则不一样。C++中POD类型是唯一可以进行静态初始化（特指常量初始化）的类型（这里不但指POD类，也指大多数标量类型，具体POD类型包含哪些，参照[wiki](https://zh.wikipedia.org/wiki/POD_(程序设计))）。也就是说POD可以进行内存加载直接初始化。所以上面的代码中p1和sp1都是直接放在DATA段中的，初始化过程只有静态初始化中的常量初始化。而p和sp则放到BSS段中，进行的是静态初始化中的零初始化过程。二者都不涉及（由构造函数导致的）动态初始化。（因为其它运行时因素，比如用函数返回值来初始化它们，仍然会引发动态初始化）总体上而言，我们将POD类与内置标量类型同样看待会比较好理解。

那么现在，我们就可以解释上面程序的输出了。在输出中，那些默认初始化的非POD类的int成员都是0，因为它们放在bss段中，内存被清0，而构造函数又没有修改它们，所以就是0。

最后，关于初始化，静态初始化的优势在于快，11标准的constexpr就是优化初始化的大杀器，具体的去google吧。奥，还有，c语言中我们有时这样写代码不会出错：

```c
int i;
int i;
int i=6;
//...
```
这是因为c支持“临时性定义”的概念，链接器会把这些实例给折叠，只留下一个实例，国内某些万恶的培训机构说的这是声明不是定义，呵呵。。c++是不支持“临时性定义的概念”的，这样写只会报错。还有，对于静态对象，如果第一构建出错（抛异常，但是捕捉到，没有终止程序），那么下一次程序运行到这个位置还会再次尝试构建。

我的表达能力有限，如果想看权威的文献，异步[这里](http://en.cppreference.com/w/cpp/language/initialization);

开完小差的分割线

---

1. 关于new操作符，实际包括两步操作，第一步是分配空间，第二部构造。如果请求分配空间为0，new会分配1字节的空间，因为标准要求每次new的地址不同。
1. 关于delete，该用`delete []`的地方不要用`delete`，反过来也一样，因为其行为是未定义的，会根据编译器的不同而变，还记得有些公司的面试题里非要说delete一个内置变量的数组不会有问题，因为它只是不调用析构函数而言，扯蛋，你敢在你的产品里这样写么，就tm会出些刁钻的题难为学生，还误导人家。[这篇讨论](http://stackoverflow.com/questions/1553382/is-delete-equal-to-delete)就说的很好。
1. `delete[]`一个数组时注意，不要把一个装有派生类的数组传给一个基类指针，然后delete[]，因为它会根据传入的指针判断每个数组成员的大小（也就是每个内存单元的大小）以及应该调用的析构函数，这样一定会出错（一方面调错析构函数，另一方面释放的内存大小不对）
1. placement new operator，不支持多态，简单的把它当成对给定大小内存上的操作，注意内存的分配与回收，它本身是不分配内存的，而placement delete operator也是不回收内存的，二者配合可以节省内存加快构造速度，但是也增加了内存管理的复杂性。
1. 如今大部分的编译器对于如下的表达式是不会生成临时对象的：
    ```cpp
    T c=a+b;
    ```

    基本上都会用NRV优化为对c的拷贝构造函数的调用。但是下面的式子就不行了，会产生临时对象：
    ```cpp
    c=a+b;
    ```

1. 临时对象的声明周期：
    1. 临时对象只有在完整表达式的最后一个步骤后才销毁
    1. 如果临时对象是用来初始化另一个对象的，应在其初始化完成后才销毁
    1. 如果临时对象被绑定给一个引用，则其生命周期持续到该引用的生命周期结束