+++
Categories = ["Development"]
Description = "cpp primer 第七章笔记"
Tags = ["Development","c++"]
date = "2016-07-20T16:36:02+08:00"
title = "cpp primer 第七章笔记"

+++

第七章，终于到正题了，类：

1. 只有在类中没有任何构造函数时，编译器才会生产合成默认构造函数；只有当类内部的内置类型被赋予类内初始值时，合成默认构造函数才有可能合适；如果类内包含其它的没有默认构造函数的类时，编译器不能为其生成合成默认构造函数；可以明确指定使用合成默认构造函数：`xxx()=default;`
1. 继承的三种方式，参见[文章](http://blog.csdn.net/complety/article/details/7493194)
1. 友元也会被继承，但是友元类不具有传递性；友元声明仅仅是友元声明，一般需要在类定义外加一个普通函数声明（一般类定义是放在头文件的，所以一般也需要这个普通函数声明），虽然有的编译器不这么要求
1. mutable关键字是说，即使类的实体是个const实体，其mutable成员也是可以改变的
1. 类内初始值可以用＝和花括号
1. 对于类中的const、引用或者未提供默认构造函数的类成员，必须用类内初始值或者构造函数的初始值列表来初始化，而不能直接在构造函数中“初始化”（原因很简单，构造函数中的“初始化”实际上是先对成员进行默认初始化，然后赋值，上面这三种都不能默认初始化）
1. 关于构造顺序，我们可以做个实验：

    ```cpp
    #include <iostream>
    #include <string>
    using std::string;
    using std::cout;
    using std::endl;
    class member{
    public:
        member(int n):m(n){cout<<"member "<<m<<" ctor"<<endl;}
        ~member(){cout<<"member "<<m<<" dctor"<<endl;}
    private:
        int m; 
    };
    class grandpa{
    public:
        grandpa(const string& n="G"):name(n){cout<<"grand "<<name<<" ctor"<<endl;}
        ~grandpa(){cout<<"grand "<<name<<" dctor"<<endl;}
    private:
        string name;
    };
    class father:public grandpa{
    public:
        father(const string& n="F"):name(n){cout<<"father "<<name<<" ctor"<<endl;}
        ~father(){cout<<"father "<<name<<" dctor"<<endl;}
    private:
        string name;
    };
    class child:public father{
    public:
        child(const string& n="C"):m1(1),m2(2),m3(3),name(n){cout<<"child "<<name<<" ctor"<<endl;}
        ~child(){cout<<"child "<<name<<" dctor"<<endl;}
    private:
        string name;
        static member m4;
        member m1,m2,m3;
    };
    member child::m4(4);
    int main(){
        child ch;
        return 0;
    }
    ```
    ```shell
    $ ./initorder.out
    member 4 ctor
    grand G ctor
    father F ctor
    member 1 ctor
    member 2 ctor
    member 3 ctor
    child C ctor
    child C dctor
    member 3 dctor
    member 2 dctor
    member 1 dctor
    father F dctor
    grand G dctor
    member 4 dctor
    ```
    总结一下：首先，类的静态变量和程序的全局变量一起初始化，然后类的父类初始化，然后类自身初始化，其中成员按照定义的顺序初始化，最后执行类自身的构造函数。析构的时候，完全反过来：先调用自己的析构，然后逆着成员的定义顺序析构成员，最后调用父类的析构函数。
1. 委托构造函数，嗯，好东西，有了这个就不需要之前的init辅助函数
1. explicit禁止隐式转换，切不能当成拷贝初始化，但可以用构造函数显示转换为临时对象，或者static_cast也可以
1. 聚合类的条件：
    1. 所有成员public
    1. 没定义任何构造函数
    1. 没有类内初始值
    1. 没有基类，没有虚函数
