+++
Categories = ["Development"]
Description = "something about aggregates and pod"
Tags = ["Development","c++"]
date = "2016-07-26T14:32:39+08:00"
title = "aggregates and pod"

+++
这篇文章大部分整理自stackoverflow上的一篇[讨论](http://stackoverflow.com/questions/4178175/what-are-aggregates-and-pods-and-how-why-are-they-special)

---

还记得数组的花括号赋值么？还记得c语言中简单的结构体么？还记得c语言初始化结构体用的memset和bzero么？还记得通过socket发送结构体是怎么实现的么？这些就是cpp中aggregate和pod想要做到的。

## Aggregate

要解释POD(Plain Old Data)，就要首先来说一下aggregate。再进入细节之前先说一下aggregate能做什么。aggregate可以用花括号初始化～～，是的，就这么简单。再深一点，它翻译成中文就是聚合，就是说把一堆成员放到一起，并没有面向对象思想中对象的概念，它只是一个数据集合。
接下来看下它的定义，首先看下primer5th中的定义：

> 1. 所有成员都是public的
> 1. 没有定义任何构造函数
> 1. 没有类内初始值
> 1. 没有基类和virtual函数

## 03标准的定义

这些定义实际上柔和类c++11标准在里面，我们先看03标准的表述：

> An aggregate is an array or a class (clause 9) with no user-declared constructors (12.1), no private or protected non-static data members (clause 11), no base classes (clause 10), and no virtual functions (10.3).

然后来把这段话拆开：

1. 数组一定是aggregate的（哪怕里面放的数据成员不是aggregate的）
1. aggregate类（标准中的class可以指class和struct以及union）不能有用户声明的构造函数（包括默认、拷贝和移动，以及其它任何构造函数）
1. aggregate类的非静态成员必须都是public的（静态成员就无所谓了）
1. aggregate类没有基类和虚函数。

好嘛，这么多，原谅我的脑容量。其实很好理解，之前说过，aggregate这个概念就是为了表明数据的聚集体，那么数组自然就是数据的聚集体，而对于一个类来说，它的数据只有非静态数据成员，而静态数据成员、成员函数等等实际上都可以看作是对这个类定义的操作。有点绕口也有些抽象，那么我们就来逐条看下这些要求。

1. 不能有任何用户定义的构造函数。如果有会怎么样？如果有的话编译器就会认为你这个类不能“简单的”初始化了,就是说编译器会觉得你这个类可能还会需要分配空间意外的其它操作，这就跟“聚合”这个概念不符了。（即使是定义了一个鸡毛都不干的默认构造函数都不行！！！虽然感觉这样也不破坏聚合的概念）
1. 非静态成员都是public的，如果是private和protected的话，亲，你要访问数据成员还得定义接口啊。。。虽然也可以，但是，编译器不知道啊，后面还会讨论花括号赋值的内容会继续说这点。
1. 不能有基类和虚函数。基类和虚函数都会在一定程度上破坏聚合的概念，更重要的，它们的存在会迫使编译器生成“不简单”（实际上是notrivial）的合成默认构造函数，详见《深入理解c++对象模型》

ok，这三条大致分析了一下，接下来我们看一下aggregate类的使用。还记得我们说数组一定是aggregate的对吧，那就先想想数组怎么初始化：
```cpp
type array[m]={a1,a2,a3...an};
```
那么，针对m和n，我们有如下分析：
```shell
if m == n
    array数组中完美的存放a1到an这些元素
elseif m > n
    array数组中的前n个空间存放a1到an，后面的元素执行值初始化（也就是会调用它们的默认构造函数）
else （这时候就是a[]={a1...an}的情形）
    这时候没有显示声明array的大小，它会按照元素的数量（n）来定义一个数组存放a1到an
```

那么我们说aggregate类也是数据的聚集，也可以使用花括号来赋值，类比数组，看一下下面这个例子：
```cpp
struct X
{
  int i1;
  int i2;
};
struct Y
{
  char c;
  X x;
  int i[2];
  float f; 
protected:
  static double d;
private:
  void g(){}      
}; 

Y y = {'a', {10, 20}, {20, 30}};
```
上面的例子当中，对于Y，其数据元素除了最后一个float f，其它都有初始值，而f则进行了默认初始化。其子类x也是一个aggregate类，所以可以由嵌套的花括号初始化。现在你想，如果成员不是public的，这种初始化能行么。。。。
例子：
```cpp
#include <iostream>
class noaggregate{
    int x;
};
struct aggregate{};
struct noaggregate1{
    noaggregate1(){};
    int n;
};
struct aggregate1{
    aggregate1& operator=(const aggregate&){return *this;}
    ~aggregate1(){}
    double d;
private:
    static int hehe;
};
int main(){
    // noaggregate ng={1}; //err: no matching ctor
    noaggregate ng; //ok
    // noaggregate1 ng1={2};//err: no matching ctor
    noaggregate1 ng1; //ok
    aggregate1 ag1={3.14}; //ok
    return 0;
}
```
最后，我们来对标准定义进行小小的探索。

1. 不能有任何***显示定义的***构造函数，但是可以有***显式声明的拷贝赋值运算符***，以及***显示的析构函数***，同时，如果一个类中有引用类型的成员，或者const成员，那必然需要一个显式的构造函数，所以不能有这两种成员。（而且注意到，对于含有非静态成员的类而言，如果不提供显示的默认构造函数，是不能定义一个该类的const对象的，因为所有const变量都需要在定义时初始化）
1. 非静态成员必须是public的，但是你可以有各种访问控制（public、private、protected）的成员函数和静态成员
1. 注意了，对数组而言，其元素是不是aggregate不影响数组的aggregate属性，同样，类的成员（不管静态还是非静态）是不是aggregate也不会影响它的aggregate属性。(非aggregate没有“感染性”)

这就是03标准对aggregate的定义了，aggregate实现了c语言中对结构体的花括号赋值，更重要的是它提供了一种数据集合的概念。

### 11标准对aggregate定义的改进

先来看下11标准中的定义：

> An aggregate is an array or a class (Clause 9) with no user-provided constructors (12.1), no brace-or-equal-initializers for non-static data members (9.2), no private or protected non-static data members (Clause 11), no base classes (Clause 10), and no virtual functions (10.3).
不一样的地方主要有两点：

1. 现在不能有user-provided constructors，之前是不能有user-declared constructors。有什么区别呢？主要是因为11标准引入了主动声明使用编译器合成的构造函数机制。所以现在可以这样用了：

    ```cpp
    struct X{
        X()=default;
    };
    ```
这个声明是说使用编译器合成的默认构造函数，但是它确实是user-declared constructors。。所以新标准在这里进行了改动，允许这种用法在aggregate类中存在。
1. 非静态成员不能有brace-or-equal-initializers。什么叫brace-or-equal-initializers呢，看下面这个例子：

    ```cpp
    struct Y{
        int x=2;
        double d=3.14;
        char ch{'x'};
    };
    ``` 
    懂了吧。。。11标准引入了类内初始值的概念，可以对类成员使用＝或大括号进行类内初始化，实际上最终编译器也会将其作为构造函数的一部分，所以不能有这些。

现在，你可以回过头去对照primer5th提供的定义再好好的研究下aggregate类了，它将是我们下面所要探索的POD类型的基础。

---

### 2016-07-28添加：

事实上，不只有aggregate变量可以用花括号进行初始化。早在11标准之前，就允许对部分类进行explicit initialization。但是有个限制就是花括号列表中的值必须为编译期可确定的常量。它对类的要求就是，成员都是public的。

## POD

还记得我们在开篇时提到的那几个问题么，关于c的结构体的那几个。好吧，现在假设我们需要用c++来实现一个功能，这个功能作为某个大型软件工程的扩展，而这个工程又是c写的，它传给我们的数据是一个c的结构体，要求我们传回的也是一个c结构体怎么办？好吧，我知道你们有更好更聪明的办法，但是这里我们采用c++的POD类型来解决。
假设我们认可POD类的这个功能，就是可以用来作为导出给c的接口，我们来试着逆推一下POD的特性：

1. 应该可以实现像c的结构体一样的复制能力，比如同类型不同实例结构体之间的复制（不考虑指针复制带来的内存管理问题）这种复制应该是位逐次复制（bitwise copy，相对于memberwise copy）
1. 应该有和“同样声明”的c结构体一样的内存布局，这样c才能用memcpy来拷贝它。（就好像将一个结构体序列化写入一个文件，再从这个文件中序列化读出一样，主角必须要有相同的内存分布，否则对数据的解读就会出错）。

那么，看一下aggregate类是否满足这些条件，因为如果它满足了，我们还要POD干嘛。

1. 同类型的复制。还记得我们在之前说过aggregate类中可以定义赋值运算符么。。。。
1. 内存布局。还记得我们说aggregate类的成员可以是非aggregate的么。。。。

所以aggregate类是不满足这些条件的，但是我们发现满足这些条件的一定是aggregate类（这里大家不要纠结非静态成员变量的访问权限是不是public这个问题，后面11标准也进行了相应的改进。至于其它条件，c的结构体本身就是符合位逐次复制机制的，所以相对应满足上面第一个条件的类也应该是位逐次复制机制，这种类只有用户不显示的定义任何构造函数编译器才会为其产生相应机制的默认构造函数和赋值构造函数。而对于虚函数与基类，研究过c++对象模型的同学应该会清楚它们对类内存布局的一些影响，所以也不详谈），也就是说POD类也是aggregate类。
这里列出的两点只是为了让大家有一个感性上的认识，接下来才是严格的定义。

## 03标准的定义

> A POD-struct is an aggregate class that has no non-static data members of type non-POD-struct, non-POD-union (or array of such types) or reference, and has no user-defined copy assignment operator and no user-defined destructor. Similarly, a POD-union is an aggregate union that has no non-static data members of type non-POD-struct, non-POD-union (or array of such types) or reference, and has no user-defined copy assignment operator and no user-defined destructor. A POD class is a class that is either a POD-struct or a POD-union.

blablabla...呵呵，读的我尴尬症都犯了。同样，还是来拆开看：

1. POD类是一种aggregate类
1. POD类的非静态成员不能有非POD类型，不能有含有非POD类型的数组，不能有引用
1. POD类不能有用户定义的赋值运算符和析构函数

哦，又到了发散思维抠字眼的时候了，首先我们很高兴，因为我们的推断是正确的：POD类都是aggregate的。接下来看后面两条：

1.
不能有非POD类型的成员。之前我们说非aggregate是没有“感染性”的，那么非POD就是有感染性的。就是说如果一个类的某个成员非POD，那么这个类也必然非POD。为什么要做这样的要求呢？我们可以这样理解，非POD类，要么没有一个兼容c结构体的内存布局（这会导致它“组成”的类的布局也与传统的c结构体不兼容，比如虚表指针、虚基类都是不兼容的因素），要么就不能执行位逐次拷贝或者notrivial（这会导致编译器生成trivial的构造函数失败，影响位逐次拷贝机制，详见《深入理解c++对象模型》），要么两者兼而有之，所以才有这条限制。
1. 不能有用户提供的赋值运算符和析构函数。用户提供了这些，就意味着类中的成员需要资源管理，传统的c结构体可是没有这个概念的。或者仍然可以从位逐次拷贝机制来理解。

例子：

```cpp
struct POD
{
  int x;
  char y;
  void f() {} //no harm if there's a function
  static std::vector<char> v; //static members do not matter
};

struct AggregateButNotPOD1
{
  int x;
  ~AggregateButNotPOD1() {} //user-defined destructor
};

struct AggregateButNotPOD2
{
  AggregateButNotPOD1 arrOfNonPod[3]; //array of non-POD class
};
```

那么POD类型的特性：

1. POD可以用做向c活着.net导出数据的数据结构
1. POD的生命周期始于空间分配，终于空间回收；而非POD则始于构造完成，终于析构开始。（POD没有显示定义的dctor的原因）
1. POD可以使用memcpy、memset等函数，保证memcpy拷贝给一块足够大的内存再拷贝回来其数据不变
1. c++不允许goto从一个还未定义某个变量的作用域跳过一个变量的定义而进入它的作用域，但是POD可以：

    ```cpp
    int f()
    {
      struct NonPOD {NonPOD() {}};
      goto label;
      NonPOD x;     //bad
    label:
      return 0;
    }

    int g()
    {
      struct POD {int i; char c;};
      goto label;
      POD x;    //ok
    label:
      return 0;
    }
    ```
1. POD保证其内存开始处不会有padding（因为没有基类虚函数而且成员都是POD的，而aggregate虽然也没有基类虚函数，但是有可能第一个成员是非aggregate的带虚函数的类，那仍然会有padding），这样对于一个POD类A，它的第一个成员为T，可以使用reinterpret_cast完成A*与T*的转化。

### 11标准对POD类定义的改进

好吧，终于到最后了。11标准对POD类的定义改动还是比较大的，但整体上来说是放宽了限制。
11标准中，POD类被定义为需要满足以下两条特性：

1. 支持静态初始化（静态初始化，可以简单理解为，一个放在data段的（全局或着静态）数据，在进程运行前，其初始化的值可以由操作系统直接从可执行文件镜像中加载到内存，虽然可能不太准确但是可以这样粗略理解）。
1. POD类在c++中编译出的内存布局与在c中编译出的内存布局相同

所以相对应的POD类被拆分为trivial和Standard-layout两部分。

1. trivial类支持静态初始化，其要求有：

> A trivially copyable class is a class that:
> 
> — has no non-trivial copy constructors (12.8),
> 
> — has no non-trivial move constructors (12.8),
> 
> — has no non-trivial copy assignment operators (13.5.3, 12.8),
> 
> — has no non-trivial move assignment operators (13.5.3, 12.8), and
> 
> — has a trivial destructor (12.4).
> 
> A trivial class is a class that has a trivial default constructor (12.1) and is trivially copyable.
> 
> [ Note: In particular, a trivially copyable or trivial class does not have virtual functions or virtual base classes.—end note ]

那么trivial与nontrivial的区别：

> A copy/move constructor for class X is trivial if it is not user-provided and if
> 
> — class X has no virtual functions (10.3) and no virtual base classes (10.1), and
> 
> — the constructor selected to copy/move each direct base class subobject is trivial, and
> 
> — for each non-static data member of X that is of class type (or array thereof), the constructor selected to copy/move that member is trivial;
> 
> otherwise the copy/move constructor is non-trivial.

2. 而Standard-layout则满足第二个特性

> A standard-layout class is a class that:
> 
> — has no non-static data members of type non-standard-layout class (or array of such types) or reference,
> 
> — has no virtual functions (10.3) and no virtual base classes (10.1),
> 
> — has the same access control (Clause 11) for all non-static data members,
> 
> — has no non-standard-layout base classes,
> 
> — either has no non-static data members in the most derived class and at most one base class with non-static data members, or has no base classes with non-static data members, and
> 
> — has no base classes of the same type as the first non-static data member.
> 
> A standard-layout struct is a standard-layout class defined with the class-key struct or the class-key class.
> 
> A standard-layout union is a standard-layout class defined with the class-key union.
> 
> [ Note: Standard-layout classes are useful for communicating with code written in other programming languages. Their layout is specified in 9.2.—end note ]

详细的解释见文章开头的链接。主要注意是，11标准放宽类限制，现在所有成员只要有相同的访问控制就好了（因为之前的标准对不同访问控制的成员内存分布没有做限定，这样可以防止成员的不连续存放）。再者，禁止第一个非静态成员和基类类型相同，是因为如果类的第一个非静态成员与基类类型相同，它们会共享相同的地址。
