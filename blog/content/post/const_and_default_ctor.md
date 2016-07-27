+++
Categories = ["Development"]
Description = "const类和default constructor"
Tags = ["Development","c++"]
date = "2016-07-27T09:33:07+08:00"
title = "const类和default constructor"

+++

今天实验程序的时候，出现这样一个问题：

```cpp
struct statictest{
    // virtual void func(){}
    // virtual ~statictest(){}
private:
    int n;
};
int main(){
     const statictest st;
    return 0;
}
```
```shell
$ make
g++ -std=c++11 -Wall static.cpp -o static.out
static.cpp:14:23: error: default initialization of an object of const type
      'const statictest' without a user-provided default constructor
     const statictest st;
                      ^
static.cpp:14:25: note: add an explicit initializer to initialize 'st'
     const statictest st;
                        ^
                        {}
static.cpp:10:9: warning: private field 'n' is not used [-Wunused-private-field]
    int n;
        ^
1 warning and 1 error generated.
make: *** [static] Error 1
```
const类不能使用编译器提供的默认初始化？
去stackoverflow查了一下，发现标准上说：

> If a program calls for the default initialization of an object of a const-qualified type T, T shall be a class type with a user-provided default constructor.

大写的囧，让我来猜测下标准的意图。

当我们写程序，写一个const变量时，其值是在定义时初始化后不能更改的，所以对类而言，定义一个const类又使用了默认初始化，那我们就必须告诉编译器我们希望默认初始化给这个类以初值，因为以后也不能给它赋值。所以需要一个用户定义的默认初始化函数。
囧囧的是，有大神提出了这种玩法：

```cpp
struct statictest{
    statictest();
    // virtual void func(){}
    // virtual ~statictest(){}
private:
    int n;
};
inline statictest::statictest()=default;
int main(){
     const statictest st;
    return 0;
}
```

是的，可以编译通过，因为标准说：

> ... A special member function is user-provided if it is user-declared and not explicitly defaulted or deleted on its first declaration. ...

言归正传，似乎实际当中像开头这样，直接定义一个const对象的不多，但是如果一定要这样，有这么几个办法：

1. 定义一个默认构造函数（好吧。。。）
1. `const A a=A();`相当于`const int a=3;`
1. `const A a{};`值初始化
