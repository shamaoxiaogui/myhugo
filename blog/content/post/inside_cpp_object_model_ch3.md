+++
Categories = ["Development"]
Description = "深入理解cpp对象模型 第三章笔记"
Tags = ["Development","c++"]
date = "2016-07-24T11:10:27+08:00"
title = "深入理解cpp对象模型 第三章笔记"

+++

前言描述了一个很有趣的问题，下面的类X，Y，Z，和A的sizeof各是多大？

```cpp
#include <iostream>
struct X{};
struct Y:public virtual X{};
struct Z:public virtual X{};
struct A:public Y,public Z{};
int main(){
    std::cout<<"X: "<<sizeof(X)<<std::endl;
    std::cout<<"Y: "<<sizeof(Y)<<std::endl;
    std::cout<<"Z: "<<sizeof(Z)<<std::endl;
    std::cout<<"A: "<<sizeof(A)<<std::endl;
    return 0;
}
```
```shell
$ ./sizeof.out
X: 1
Y: 8
Z: 8
A: 16
```
我们来分析一下。首先，注意到上面三个类都是所谓的空类，就是没有数据成员的。

1. 对于A，确实是个空类，没有虚函数意味着没有虚表指针，没有继承关系也不需要编译器添加特别的数据部分。这时他的大小是1个字节，因为为了使A的不同实例有不同的地址，所以编译器对其进行了小填充。
    
    ```cpp
    struct A{
        char _ch;
    };
    ```
1. 对于X和Y，其实并不是完全的空类。虽然没有虚表指针，但是它们虚继承了A，所以编译器必然会给它们添加一些成员：添加一个指针，指向虚基类或一个表格，表格中存放虚基类的地址或者其偏移量。我们来分析下这种做法。我用的mac是64位机，也就是指针大小为8字节，那么Y／Z的大小为`8(virtual base pointer)+1(virtual base class)+7(alignment padding)=16bytes`。

    ```cpp
    struct Y{
        void *vbtr;     //to A, 8bytes
        A aa;   //1byte
        //padding 7bytes
    };
    ```
而结果为8byte与我们的计算不复。这里实际上编译器执行了一种优化：空虚基类（empty virtual base class ）通常用于提供一个virtual interface，那就将这种基类当作优化为派生类的数据成员好了，也就是直接将它放到派生类的开头部分，不再需要用虚基类指针指向它。这样的话，之前“空”的派生类现在“有”了一个成员，那么就没有必要给他添加1byte来区分类的不同对象，也就是将上面第一条的1个字节给省掉了，那么也就不需要最后7个字节的alignment padding了，而需要的是对虚基类占用空间的padding。所以大小为`1(virtual base class)+7(alignment padding)=8bytes`。反过来，如果X中原本就有数据成员，就不会出现这种编译器优化差异。

    ```cpp
    struct Y{
        A aa;   //1byte
        //padding 7bytes
    };
    ```
1. 最后来看A，记住在继承链中不论出现多少次，虚基类只有一个实体。假设编译器没有优化：`8(vbtr in Y)+8(vbtr in X)+1(virtual base class)+7(alignment padding)=24bytes`

    ```cpp
    struct A{
        void *vbtr_Y;   //vbtr in Y, 8bytes
        void *vbtr_X;   //vbtr in X, 8bytes
        A aa;   //virtual base class, 1byte
        //padding 7bytes
    };
    ```
而进行优化后，大小为`8(Y)+8(Z)=16bytes`。
    
    ```cpp
    struct A{
        Y yy;   // 8bytes
        Z zz;   // 8bytes
    }
    ```

---
1. 现代C++标准对类的成员函数的解析，是在对类的解析完成后才开始的。所以，将类成员声明在成员函数之后，对成员函数中调用这些成员的语义没什么影响，但是，成员函数的签名却是按顺序解析的，即在类完全解析完之前，成员函数签名就解析完了，这有可能带来如下的影响：

    ```cpp
    typedef int length;
    extern int x;
    struct someclass{
        //...
        length func(){return x;} //这里编译器对x的解读是没问题的，因为到类解析完编译器才会解析成员函数的内容。但是对于声明中的length就不是这样了，它会按顺序解析，也就会把length解析为int
        //...
    private:
        typedef double length;
        length x;
    };
    ```
所以，***永远把类要用到的typrdef放到类声明的起始处***
1. 对于类中的数据成员，标准只规定，同一access section（private。。。）中靠后的数据成员地址高，并没有说一定要连续。所以有时为了内存对齐，成员直接会有padding。静态成员不管声明在哪都不会占空间。vptr的位置视编译器而定，大多放在类的开始。
1. 类的成员的存取。
    1. 静态成员，静态成员存储在全局区，所以类的继承关系、访问权限、指针还是对象访问，用类直接访问还是用对象访问对其访问效率都没有影响。要注意的是下面亮点：
        1. 见如下代码：

            ```cpp
            foobar().chunkSize=500;
            //
            (void)foobar();
            Point3d.chunkSize=500;
            ```
            编译器会进行如上变换。
        1. 如果两个不同的类都声明了同名的静态成员，那么编译器会进行name-mangling，变换齐名称
    1. 非静态成员：
        
        ```cpp
        origin._y=0.0;
        //
        *(&origin+(&Point3d::_y-1))=0.0;
        ```
        这里编译器会进行地址的转化，转化为类的地址加上成员的偏移地址。之所以要减一，是因为类成员指针总是指向类成员加一的位置，这样就可以区分一个类成员指针没有指向任何一个成员的情况。
        对于如下代码：

        ```cpp
        origin.x=0.0;
        pt->x=0.0;  //pt=&origin
        ```
        在通常，执行效率没有任何不同，但是，如果origin是一个虚拟继承的派生类，并且x是派生类的成员，这时效率就会有重大差异。因为在编译时不知道pt中存放的具体是哪种类型，就无法用静态地址去替换它，只能依靠一些动态手段（vptr、vbtr）；而origin的类型是可以确定的，编译器会解析出它的静态地址。
1. 继承时的布局。一般是派生类扩展基类，虚表指针放在顶部。多继承时，内存上先放第一个基类，再放第二个。。。涉及到虚基类，记住虚基类只有一个实体，直接继承虚基类的派生类会有虚基类指针，指向虚基类或者是其偏移量。
1. 关于成员存取的效率，也许对于虚拟继承基类中成员的存取会慢一点，但是现代编译器的优化都做的比较好，问题不大。
1. 最后，类成员指针总是存储的是成员在类内的偏移量，书上说这个偏移量还会加一，以区分其不指向对象的情况，但是g++测试表面编译器对此进行了内部优化，使其直接表示偏移量。

    ```cpp
    #include <iostream>
    #include <cstdio>
    struct sc{
        virtual void func(){}
        char ch;
        int x;
    };
    int main(){
        sc scc;
        printf("&scc.ch %p\n",&scc.ch);
        printf("&scc.x %p\n",&scc.x);
        printf("sc::ch %p\n",&sc::ch);
        printf("sc::x %p\n",&sc::x);
        return 0;
    }
    ```
    ```shell
    $ ./classptr.out
    &scc.ch 0x7fff5d4b37f0
    &scc.x 0x7fff5d4b37f4
    sc::ch 0x8
    sc::x 0xc
    ```
