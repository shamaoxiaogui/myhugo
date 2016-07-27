+++
Categories = ["Development"]
Description = "convariant return type and override"
Tags = ["Development","c++"]
date = "2016-07-27T13:23:23+08:00"
title = "convariant return type and override"

+++

我们都知道在类的继承体系中，想要override一个虚方法，要求新方法的函数返回类型以及函数签名同被覆盖的方法相同，也就是说，下面的代码：
```cpp
#include <iostream>
using namespace std;
struct base{
    virtual int func(){cout<<"base func"<<endl;return 0;}
    virtual ~base(){}
};
struct derived:public base{
    virtual double func() override {cout<<"derivied func"<<endl;return 3.14;}
};
int main(){
    base *bp=new derived;
    bp->func();
    delete bp;
    return 0;
}
```

是不能通过编译的。然而有一种例外情况，就是当新方法相对于被覆盖的方法而言，有convariant返回值类型的话，允许返回值类型不一致：
```cpp
/* Inheritance hierarchies

         NetServer
             |
             ^
            / \
NetServerTCP   NetServerSCTP


         NetClient
             |
             ^
            / \
NetClientTCP   NetClientSCTP

*/

class NetServer {
public:
    virtual NetClient* acceptConnection() = 0;
};

class NetServerTCP : public NetServer {
public:
    virtual NetClientTCP* acceptConnection();
};

class NetServerSCTP : public NetServer {
public:
    virtual NetClientSCTP* acceptConnection();
};
```

这样的override是允许的。c++标准中：

> The return type of an overriding function shall be either identical to the return type of the overridden function or covariant with the classes of the functions. If a function D::f overrides a function B::f, the return type of the functions are covariant if the satisfy the following criteria:
> 
> both are pointers to classes or references to classes98)

> the class in the return type of B::f is the same class as the class in the return type of D::f or, is an unambiguous direct or indirect base class of the class in the return type of D::f and is accessible in D

> both pointers or references have the same cv-qualification and the class type in the return type of D::f has the same cv-qualification as or less cv-qualification than the class type in the return type of B::f.

注意第三条关于cv-qualification，举个例子就是可以这样：

```cpp
struct base{
    const base* func(){return this;};
};
struct derivied: public base{
    derivied* func(){return this;}
};
```
但是不能反过来。

总结一下，covariant返回值的情况简单来说就是如果在代码的某个位置可以用D代替B（D派生自B），那么就可以使用返回值为D的指针或引用的同签名函数override B中的虚函数。

最后，记住const限定符修饰的方法与无const限定符修饰的是不同的函数，不会互相override，只会够成重载（overload）关系。
