+++
Categories = ["Development"]
Description = "cpp primer 第四章笔记"
Tags = ["Development","c++"]
date = "2016-07-19T21:18:38+08:00"
title = "cpp primer 第四章笔记"

+++

第四章主要是表达式相关的内容：

1. 除了`&& || ?: ,`四种操作符外，其他操作符都没有定义运算顺序（重要）
2. sizeof是一种运算符，它不会对表达式进行求值，c++11允许用作用域运算符(::)来获取成员大小而不必提供一个实体（前提是，该成员是public的，活着调用方法是static的），对数组返回数组大小而不是指针大小，对vector和string这种返回其固定部分大小。

    ```cpp
    #include <iostream>
    #include <string>
    class testc{
    public:
        char pwc;
        void update(){}
    private:
        char wc;
    };
    int main(){
        int a=44;
        int ar[5];
        std::string str("heheda!");
        std::cout<<"int : "<<sizeof(a)<<" "<<sizeof(int)<<std::endl;
        std::cout<<"int [5]: "<<sizeof(ar)<<" "<<sizeof(int [5])<<std::endl;
        std::cout<<"string :"<<sizeof(str)<<" "<<sizeof(std::string)<<std::endl;
        std::cout<<"class member(char): "<<sizeof(testc::pwc)<<std::endl;
        return 0;
    }
    ```
    ```shell
    $ ./sizeoftest.out
    int : 4 4
    int [5]: 20 20
    string :24 24
    class member(char): 1
    ```
1. 逗号运算符先算左边再算右边，最后返回右边，若右边结果为左值则返回的也是左值
1. 大多数情况下数组退化为指针，有四种例外：decltype、取址运算符（&），sizeof，typeid：

    ```cpp
    #include <iostream>
    int main(){
        int a[4];
        std::cout<<"int size: "<<sizeof(int)<<std::endl;
        std::cout<<"arr size: "<<sizeof(a)<<std::endl;
        std::cout<<"a[0] addr: "<<&a[0]<<std::endl;
        std::cout<<"a addr: "<<a<<std::endl;
        std::cout<<"&a :"<<&a<<std::endl;
        std::cout<<"&a[0]+1: "<<&a[0]+1<<std::endl;
        std::cout<<"a+1: "<<a+1<<std::endl;
        std::cout<<"&a+1: "<<&a+1<<std::endl;
        return 0;
    }
    ```
    ```shell
    $ ./arrayaddr.out
    int size: 4
    arr size: 16
    a[0] addr: 0x7fff566947e0
    a addr: 0x7fff566947e0
    &a :0x7fff566947e0
    &a[0]+1: 0x7fff566947e4
    a+1: 0x7fff566947e4
    &a+1: 0x7fff566947f0
    ```
    可以发现当采取`&a+1`这种形式时，与数组首地址相比，并不是单纯加4，而是加4*4，故而大胆推测，`&a`的类型为`int [4]`（类似二维数组中的一维元素）
1. 指针的转换规则：
    - 0和nullptr可以转换成任意指针（空指针）
    - 任意非常量指针可以转换成`void*`
    - 任意指针可以转换成`const void*`
1. 显示转换：
    - `static_cast`就相当于c的强制类型转换
    - `const_cast`去除底层const属性，但是对转换后的对象（一般是指针，对其指向的对象）写操作是未定义的，而且不能改变变量类型
    - `reinterpret_cast`最危险的转换，相当于对同一个内存的地址做不同意义的解释（比如把一个int＊解释为一个char＊），依赖于机器
    - `dynamic_cast`将指向子类的父类型的指针转化为子类型，这个功能也可以由`static_cast`来实现，但是dy会在运行时检查父子类关系，而static只会在编译时检查
