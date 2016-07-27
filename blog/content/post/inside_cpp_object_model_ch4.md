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
    c++保证，非静态成员函数至少应该和一般的非成员函数有相同的效率。
    对函数名的处理，一般在函数名中添加所属类、函数签名（不包括函数返回值）信息，这样来区分重载函数以及父子类间的虚函数等。这样在编译链接不同的文件时，如果函数参数类型不同链接会出错，但是如果函数返回值类型声明错了就没法检查出来了（不同的编译器都不太一样）

1. 对于指针或者引用调用的虚函数，编译器会将其扩展为由虚表指针指向的虚表中的相应函数指针，效率会损失。而由对象调用的虚函数，则复合一般成员函数调用规则，效率高如果虚函数是inline的，使用对象调用时编译器可以将其展开，效率会更高。
1. 对于静态成员函数：
    1. 不能直接存取类中的非静态成员
    1. 不能被声明为const、volatile、virtual（原因在于，静态成员函数被编译器当作非成员函数对待，不会传给他类的this指针，而const和volatile修饰的函数需要传入一个`const A * const this`或`volatile A * const this`指针，而virtual函数也需要this指针来查找虚表）
    1. 不需要经由对象才能调用

    ```cpp
    #include <iostream>
    struct statictest{
        // static void hehe()const{std::cout<<"static func"<<std::endl;}
        // static void hehe()volatile{std::cout<<"static func"<<std::endl;}
        // static virtual void hehe(){std::cout<<"static func"<<std::endl;}
        static void hehe(){std::cout<<"static func"<<std::endl;}
        statictest& func(){std::cout<<"func run"<<std::endl;return *this;}
    private:
        int n;
    };
    int main(){
        statictest st;
        st.func().hehe();
        return 0;
    }
    ```
    ```shell
    $ ./static.out
    func run
    static func
    ```
这个例子说明，虽然调用静态函数不需要对象，但是有的编译器还是会对获得对象的表达式进行运算还有，对静态成员函数取地址，得到是是一个非成员函数指针
