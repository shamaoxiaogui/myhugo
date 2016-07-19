+++
Categories = ["Development"]
Description = "cpp primer 第二章笔记"
Tags = ["Development","c++"]
date = "2016-07-19T15:16:37+08:00"
title = "cpp primer 第二章笔记"

+++

第二章主要讲的是基本类型，需要注意的有：

1. 同一个表达式混用***有符号***和***无符号***数，会自动转换成***无符号***
1. 字面值前缀指定字符编码，后缀指定变量大小或种类
1. 字符串字面值不可修改（因为放在一个只读段中）

    ```cpp
    #include <iostream>
    int main(){
        char *str="heheda!";
        std::cout<<"before m: "<<str<<std::endl;
        str[3]='a';
        std::cout<<"after m: "<<str<<std::endl;
        return 0;
    }
    ```

    ```shell
    $ ./cstr.out
    before m: heheda!
    [2]    37400 bus error  ./cstr.out
    ```
1. `c++11`提供了一种新的*列表初始化*，但是效率么。。。

    ```cpp
    #include <iostream>
    #include <string>
    #include <vector>
    class tc{
    public:
        tc(){std::cout<<"ct a class! "<<std::endl;}
        tc(const tc &t){std::cout<<"copy a class!"<<std::endl;}
        tc& operator=(tc &t){std::cout<<"= a class!"<<std::endl; return *this;}
    };
    int main(){
        tc t1;
        std::cout<<"==============="<<std::endl;
        tc t2(t1);
        std::cout<<"==============="<<std::endl;
        tc t3=t1;
        std::cout<<"==============="<<std::endl;
        tc t4{t1};
        std::cout<<"==============="<<std::endl;
        tc t5={t1};
        std::cout<<"==============="<<std::endl;
        std::vector<tc> tcv{t1,t2,t3};
        return 0;
    }
    ```

    ```shell
    $ ./initlist.out
    ct a class!
    ===============
    copy a class!
    ===============
    copy a class!
    ===============
    copy a class!
    ===============
    copy a class!
    ===============
    copy a class!
    copy a class!
    copy a class!
    copy a class!
    copy a class!
    copy a class!
    ```
    所以在用容器时不要随便用初始化列表，详细见[这里](https://segmentfault.com/a/1190000002484690)
1. const相关
    - const变量只能初始化不能赋值（哈士奇都知道）
    - 不加extern的const变量只能在本文件访问（所以说加了呢）
    - const左值引用可以绑定：const变量，非const变量，常量（字面量，这也是std::string的设计，只能将字符串传给const std::string &，而不能传给std::string &），无论如何，常量左值引用意味着不能通过这个引用去修改变量
    - 顶层const指指针本身的值是常量，底层const则是指针指向的值是常量，故而在函数传参或者复制指针时，顶层const忽略（反正就算你是常量我也只是复制你，后面无论怎么操作也不会修改你），底层const则不能忽略（因为有可能通过复制的指针修改数据）

1. constexpr常量表达式，由编译器帮忙检查；using的别名声明功能（类似typedef—）
1. 单纯的auto的推断效果：忽略顶层const，然后把表达式的值算出来是啥类型auto就是啥类型。底层const保留。加const的auto推断出来才是顶层const的，auto &推断出来才是引用
1. decltype基本保留变量的所有属性，尤其是当变量为引用时推断出来的结果也是引用。当其表达式是解引用操作时，结果必为引用，而其跟双括号表达式（加了括号的变量），结果必为引用
1. 注意类内初始值的初始化顺序：类内初始值－》构造函数初始化列表－》构造函数体
