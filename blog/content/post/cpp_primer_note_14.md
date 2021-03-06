+++
Categories = ["Development"]
Description = "cpp primer 第十四章笔记"
Tags = ["Development","c++"]
date = "2016-07-21T09:18:59+08:00"
title = "cpp primer 第十四章笔记"

+++

第十四章主要讲运算符重载和类型转换（感觉每次第一句话都是废话）：

1. 不应该被重载的运算符：`&& || , `因为无法保留其运算顺序（还有逻辑运算符的短路运算属性），而对于逗号和取址运算，语言已经定义了其对于类的特殊含义
1. 必须定义为成员函数的运算符：`= [] () ->`（语言也禁止这样做）
1. 对于需要对称性的运算符（＋），可能会转换任一端参数，定义为普通函数
1. 对于输入输出运算符的重载，注意：
    1. 减少格式化输出，不应该添加换行符
    1. 输入输出运算符必须是非成员函数
    1. 输入运算符中应处理出错情况，负责从错误中恢复；输出运算符则不需要考虑错误
1. 同时定义了算术运算符和相关的复合赋值运算，应优先使用复合赋值（开销小）
1. 递增递减运算的原型：

    ```cpp
    class X{
    public:
        X& operator++();    //前置
        X operator++(int);  //后置，其中参数不会用到
    };
    ```
1. 下表运算一般定义两个版本，普通的返回引用，加const限定符的返回常量引用
1. lambda表达式产生的类不含默认构造函数、赋值运算符、默认析构函数
1. cpp的可调用对象有：函数、函数指针、lambda表达式、bind对象、函数对象
1. std::function模版可以用来存放或者说是接受可调用对象，但是不能放重载函数，解决方案是使用函数指针活着lambda给重载函数做一个“别名”。
1. 类型转换函数，可以定义为转换成任何能作为函数返回类型的类型（数组、函数类型就不行，但是数组指针和函数指针可以）；必须是类的成员函数；无返回类型参数列表；通常const限定：
    ```cpp
    operator type() const;
    ```
1. 显示的类型转换运算符，就是在前面加explicit，使用时必须用cast。例外：
    1. if，while，do语句条件部分
    1. for语句条件部分
    1. `! || &&`运算对象
    1. 条件运算符表达式（?:）的条件部分

    这些语句中会隐式的调用显示类型转换运算符中转换为bool的那类。比如io库中，我们经常用：
    ```cpp
    while(cin>>n){
    ...
    }
    ```
    所以，一般向bool型的类型转换我们可以大胆的写成explicit的（一般也只定义向bool的转换）
