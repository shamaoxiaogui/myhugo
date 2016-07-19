+++
Categories = ["Development"]
Description = "C++ Primer的读书笔记，第一章"
Tags = ["Development","c++"]
date = "2016-07-19T14:20:39+08:00"
title = "cpp primer 第一章笔记"

+++

第一章只是一个引入，要注意的有：

1. `std::cin`在读到EOF的情形
    ```cpp
    #include <iostream>
    int main(){
        int val;
        while(std::cin>>val){
            std::cout<<val<<" ";
        }
        std::cout<<std::endl;
        std::cout<<"after loop,val= "<<val<<std::endl;
        return 0;
    }
    ```
测试发现最终结束时，输入C+D(EOF)与输入非数字，最终val的值是不一样的
    ```shell

    $ ./istest
    1
    1 $

    after loop,val= 0

    $ ./istest
    1
    1 ^D
    
    after loop,val= 1

    ```

1. `std::cout,std::cerr,std::clog`的区别
    ```cpp
    #include <iostream>
    int main(){
        std::cout<<"from cout"<<std::endl;
        std::clog<<"from clog"<<std::endl;
        std::cerr<<"from cerr"<<std::endl;
        return 0;
    }
    ```
    测试结果：
    ```shell
$ ./istest && echo "======" && ./istest 2>hehe
from cout
from clog
from cerr
======
from cout
    ```
    总结：
        - cout 标准输出
        - cerr 标准错误
        - clog 标准错误（貌似带缓冲，没验证）
