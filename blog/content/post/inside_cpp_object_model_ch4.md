+++
Categories = ["Development"]
Description = "深入理解cpp对象模型 第四章笔记"
Tags = ["Development","c++"]
date = "2016-07-26T10:31:23+08:00"
title = "深入理解cpp对象模型 第四章笔记"

+++
第四章主要是对成员函数的讲解：

1. 对于static成员函数：
    1. 不可以直接存取nonstatic成员
    1. 不可以是const限定的
1. 对于非静态成员函数的处理：
    1. 在函数签名（函数名，参数数量，参数类型）中添加一个新参数，this指针。
    1. 将其中对非静态成员的存取改为this指针操作

    ```cpp
    struct Someclass{
        void func(){n+=3;}
        int n;
    };
    Someclass::func();
    //expand to
    void func(Someclass *const this){ //magic here
        this->n+=3;
    }
    ```
    注意到扩展出的this是一个const的指针，也就是说在非静态成员函数中我们不能修改传入的this指针。同样，如果非静态成员函数是const限定的，那么传入的const会同时有顶层和底层const属性，也就限定了函数不得修改this和this指向的类。
