+++
Categories = ["Development"]
Description = "cpp primer 读书笔记"
Tags = ["Development","c++"]
date = "2016-07-19T17:05:39+08:00"
title = "cpp primer 第三章笔记"

+++
第三章主要讲vector和string

1. string到底是不是以\0结尾的？c++11中规定***是***。但是不要依赖这个特性，乖乖的用size()；但是getline的时候换行符确实丢掉了
2. auto for循环，注意效率，有些时候auto for效率更高（迭代器最慢～～）

    ```cpp
    #include <iostream>
    #include <chrono>
    #include <string>
    int main(){
        int n;
        char ch;
        std::cout<<"input n: ";
        std::cin>>n;
        std::cout<<std::endl;
        std::string str(n,'c');
        std::chrono::high_resolution_clock::time_point t1,t2;
        t1=std::chrono::high_resolution_clock::now();
        for(int i=0;i<n;++i){
            ch=str[i];
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"for: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
        t1=std::chrono::high_resolution_clock::now();
        for(auto t:str){
            ch=t;
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"auto for: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
        t1=std::chrono::high_resolution_clock::now();
        for(auto &t:str){
            ch=t;
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"auto& for: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
        t1=std::chrono::high_resolution_clock::now();
        for(const auto &t:str){
            ch=t;
        }
        t2=std::chrono::high_resolution_clock::now();
        std::cout<<"const auto& for: "<<std::chrono::duration_cast<std::chrono::microseconds>(t2-t1).count()<<" microseconds"<<std::endl;
        return 0;
    }
    ```
    ```shell
    $ ./fortest.out
    input n: 1000000000

    for: 9104902 microseconds
    auto for: 5651329 microseconds
    auto& for: 5445633 microseconds
    const auto& for: 5508764 microseconds
    ```
    但是切记范围for不能改变循环序列的大小！！！

1. string的c_str()返回的char*字符串有可能不保证一直有效！！！
