+++
Categories = ["Development"]
Description = "cpp primer 第六章笔记"
Tags = ["Development","c++"]
date = "2016-07-20T14:49:08+08:00"
title = "cpp primer 第六章笔记"

+++

第六章主要是函数相关的内容：

1. 函数是可以没有定义只有声明的，只要你不用它（变量声明，类的前置声明也是）。。。
1. 关于引用与指针，我觉得最重要的区别的话，主要是
    1. 引用初始化后就不能再改变引用的对象了，而指针可以指向不同的对象
    1. 有空指针没有空引用
    1. 引用是对象的别名，不是一个对象，而指针是一个专门用来存放地址的对象，所以sizeof二者会有区别
1. 对于函数的参数，可以理解为c++只有传值，没有传引用，其传引用是通过对地址的传值实现的。数组传引用时会退化成指针，当使用引用来作为数组形参时，需要指定维度，不符合维度的数组不能匹配到该函数
1. 又到了可爱的initializer_list，记住注意复制开销
2. 关于函数return临时对象的问题，参见[文章](http://www.cnblogs.com/xkfz007/articles/2506022.html)，可见编译器帮我们做了很多事情，但是仍然注意后面的右值引用和完美转发
1. inline和constexpr函数可以定义多次，但是定义必须完全一样，所以一般把它们放到头文件中
2. 对于函数对象和函数的比较，发现效率似乎差距不大（clang）

    ```cpp
    #include <iostream>
    #include <chrono>
    auto func1(int t)->int{
        return ++t;
    }
    class func2{
    public:
        auto operator()(int t)->int{
            return ++t;
        }
    };
    inline int func3(int t){
        return ++t;
    }
    static inline int func4(int t){
        return ++t;
    }
    int main(){
        int n,i=0;
        std::chrono::high_resolution_clock::time_point t1,t2;
        std::cout<<"input: ";
        std::cin>>n;
        std::cout<<std::endl;
        std::cout<<"========================="<<std::endl;
        t1=std::chrono::high_resolution_clock::now();
        for(int j=0;j<n;++j){
            i=func1(j);
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"time: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
        std::cout<<"========================="<<std::endl;
        func2 func2i;
        t1=std::chrono::high_resolution_clock::now();
        for(int j=0;j<n;++j){
            i=func2i(j);
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"time: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
        std::cout<<"========================="<<std::endl;
        t1=std::chrono::high_resolution_clock::now();
        for(int j=0;j<n;++j){
            i=func3(j);
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"time: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
        std::cout<<"========================="<<std::endl;
        t1=std::chrono::high_resolution_clock::now();
        for(int j=0;j<n;++j){
            i=func4(j);
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"time: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
    }
    ```
    ```shell
    $ ./functest.out
    input: 1000000000

    =========================
    time: 3859925 microseconds
    =========================
    time: 3849522 microseconds
    =========================
    time: 3647694 microseconds
    =========================
    time: 3865106 microseconds
    ```
