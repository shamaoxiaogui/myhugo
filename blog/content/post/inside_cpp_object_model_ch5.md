+++
Categories = ["Development"]
Description = "深入理解cpp对象模型 第五章笔记"
Tags = ["Development","c++"]
date = "2016-07-28T09:59:16+08:00"
title = "深入理解cpp对象模型 第五章笔记"

+++

1. 对于以下代码：

    ```cpp
    #include <iostream>
    struct abase{
        virtual ~abase()=0;     //Oops, pure virtual dctor must be defined, or there would be a link error
        virtual void func()=0;
        virtual void func1()=0;
    protected:
        abase(char c=0):_ch(c){}
        char _ch;
    };
    abase::~abase(){std::cout<<"abase dctor"<<std::endl;}
    abase::func(){}
    struct derivied:public abase{
        void func(){}
        void func1(){}
        void func3(){abase::func();}
    };
    int main(){
        derivied d;
        return 0;
    }
    ```
    1. 纯虚函数可以被定义，但是只能用于静态调用，所以说是否去定义实现一个纯虚函数对用户而言可选。但是析构函数例外。一个抽象类，如果定义了一个纯虚析构函数，就必须定义实现它，因为编译器会对其子类的析构函数进行扩展，如果定义在抽象类声明一个纯虚析构函数而不实现它，会产生链接错误。最好是将析构函数定义为虚函数而不是纯虚函数。
    1. 既然抽象基类中含有数据成员，我们就应该为其提供一个初始化机制，以允许派生类能初始化它
    1. 对于类中那些只操作该类的成员而不会影响后续派生类成员的接口，不要声明virtual
1. 构造顺序：
    1. 初始化virtual base class，从左到右（在继承列表中的顺序），由深到浅（在初始化列表中显示定义的就显式调用相应构造函数，否则调用默认构造函数）
    1. 初始化base class，类似上一步
    1. 设定虚表
    1. 按声明顺序初始化成员，类似第一步的规则
    1. 运行用户代码
1. 对于用户提供的拷贝赋值运算符，注意
    1. 不要忘了考虑自赋值情况
    1. 不要忘了处基类的拷贝

    如果不提供显式的拷贝运算符，编译器会提供一个合成的，它会调用父类的拷贝运算。有时候没必要提供拷贝运算就不要提供，因为这种情况下编译器大都会使用trivial的拷贝运算，也就是位逐次拷贝，效率高。但是有时我们为了触发编译器的NRV优化，确实会提供拷贝构造函数。拷贝运算符对virtual base class的处理相当复杂，所以如果没有必要不要为virtual base class提供数据成员。
1. 析构顺序：
    1. 运行用户代码
    1. 析构成员，按声明的相反顺序
    1. 设定虚表
    1. 析构基类， 按声明的相反顺序
    1. 析构虚基类，按构造的相反顺序

    同样，对于析构函数，如果没有必要提供定义就不要提供，一旦编译器使用trivial的析构方法，效率会非常高，相当于只进行内存的释放。
