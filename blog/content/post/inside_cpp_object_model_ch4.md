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
1. 关于继承体系中的虚函数，书中有两点建议：
    1. 由于多继承时虚函数调用开销加大，编译器会对短的虚函数进行优化，所以虚函数尽量写的短一些。
    1. 不要在虚基类中定义非静态成员，编译器会做一些复杂的操作，影响性能。
1. 成员函数指针的讨论：
    1. 静态成员、静态成员函数、非静态成员函数的指针都是其实际内存地址，但是对于非静态成员函数，调用还需要提供this指针，所以这个地址是“不完整”的

		```cpp
		#include <iostream>
		using namespace std;
		class Someclass{
		public:
			static void sfunc(){cout<<"static func"<<endl;}
			static int smember;
			void nsfunc(){cout<<"nonstatic func"<<endl;};
			int nsmember;
		};
		int Someclass::smember=1;
		int main(){
			void (*sfp)();
			int *smp;
			void (Someclass::*nsfp)();
			int Someclass::*nsmp;
			sfp=&Someclass::sfunc;
			smp=&Someclass::smember;
			nsfp=&Someclass::nsfunc;
			nsmp=&Someclass::nsmember;
			printf("p sfp: %p\n",sfp);
			printf("p smp: %p\n",smp);
			printf("p nsfp: %p\n",nsfp);
			printf("p nsmp: %p\n",nsmp);
			return 0;
		}
		```
        ```shell
        $ ./memberpointer.out
        p sfp: 0x10cf11150
        p smp: 0x10cf120e8
        p nsfp: 0x10cf11190
        p nsmp: 0x0
        ```
    1. 而虚函数成员指针的调用则更加复杂和低效。通常编译器可以在编译期判断调用是否为虚函数，对于非虚函数，直接使用指针指向的函数地址，而对于虚函数，由于编译器无法确定最终调用的是哪个函数，往往使用一些终端手段。比如在成员虚函数指针中存放该虚函数在虚表中的索引，再间接调用。对于我们用户而言，只需要知道虚函数的指针与普通成员函数指针同样使用，也可以运行时多态，但是效率更低就好了。

        ```cpp
 		#include <iostream>
		using namespace std;
		struct base{
			virtual void func(){cout<<"base func"<<endl;}
			virtual ~base()=default;
		};
		struct derived:public base{
			void func(){cout<<"drived func"<<endl;}
		};
		int main(){
			void (base::*bpf)()=&base::func;
			base *bp=new derived;
			(bp->*bpf)();
			delete bp;
			return 0;
		}
        ```
        ```shell
        $ ./virtualpointer.out
        drived func
        ```
    1. 最后，inline函数在引发参数副作用的时候（比如形参传入了一个函数调用，本意是使用该函数的返回值做形参），通常会引入局部变量。inline内部的局部变量也会因为展开而参与name mangling。所以大量调用inline容易产生大量局部变量
