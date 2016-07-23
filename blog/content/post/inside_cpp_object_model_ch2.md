+++
Categories = ["Development"]
Description = "深入理解cpp对象模型 第二章笔记"
Tags = ["Development","c++"]
date = "2016-07-23T09:44:58+08:00"
title = "深入理解cpp对象模型 第二章笔记"

+++

构造函数部分：

1. 先复习下primer对合成默认构造函数的叙述：
    1. 构造函数不能使用const限定符，一个const对象直到构造函数完成时才获得其“常量”属性。
    1. 编译器只有发现类不包含任何构造函数的情况下才会生成一个默认构造函数。
    1. 如果类中包含没有默认构造函数的类成员，就无法生成合成默认构造函数。
1. inside中，一个没有任何构造函数的类会有一个隐式声明的trivial（没啥用的）构造函数，在以下情况则会有nontrivial（有用的，编译器所需要的，我的理解是，就是我们通常说的合成默认构造函数）
    1. 如果一个类没有任何构造函数且所有的类成员都有默认构造函数，那么编译器会合成一个默认构造函数（在需要的时候，也就如果在代码中实际定义类的实体才会生成），按类成员的声明顺序来调用它们的默认构造函数。而内置变量成员的初始化则是程序员的责任。如果程序员提供了一个构造函数，来进行内置成员变量的初始化，则编译器会按照上述规则扩充这个函数。

        ```cpp
        #include <iostream>
        using namespace std;
        class c1{
        public:
            c1(){cout<<"c1 ctor"<<endl;}
            c1(const c1&){cout<<"c1 copy ctor"<<endl;}
        };
        class c2{
        public:
            c2(){cout<<"c2 ctor"<<endl;}
            c2(const c2&){cout<<"c2 copy ctor"<<endl;}
            virtual ~c2(){}
        };
        class c3{
        public:
            c2 cc2;
            c1 cc1;
            char *str;
        };
        class c4{
        public:
            c4(){cout<<"c4 ctor"<<endl;n=1;}
            c4(const c4&){cout<<"c4 copy ctor"<<endl;}
            c2 cc2;
            c1 cc1;
            int n;
        };
        int main(){
            c3 cc3;
            cout<<"==========================="<<endl;
            c4 cc4;
            cout<<"==========================="<<endl;
            c4 cc5(cc4);
            cout<<"sizeof c1 "<<sizeof(c1)<<endl;
            cout<<"sizeof c2 "<<sizeof(c2)<<endl;
            return 0;
        }
        ```
        ```shell
        $ ./classctor.out
        c2 ctor
        c1 ctor
        ===========================
        c2 ctor
        c1 ctor
        c4 ctor
        ===========================
        c2 ctor
        c1 ctor
        c4 copy ctor
        sizeof c1 1
        sizeof c2 8
        ```
    可以发现（调皮的我顺路测试了下空类的大小），编译器确实扩展了程序员编写的构造函数(即使是拷贝构造函数，编译器扩展时活着生成时也是调用类成员的默认构造函数)。另外，为了防止在多个文件（编译模块）中生成多个默认构造函数，编译器***把合成的默认构造函数、拷贝构造函数、析构函数，拷贝赋值运算符都以inline的方式完成***，因为inline函数只在当前文件作用域有效，如果函数太复杂，就合成为static非inline函数。
    1. 如果一个没有任何构造函数的子类的父类有默认构造函数，那么编译器会合成一个调用父类默认构造函数的合成默认构造函数。如果程序员提供了构造函数，但是没有提供默认构造函数，编译器不会合成一个默认构造函数，但是会扩展所有的构造函数，调用父类的默认构造函数。

        ```cpp
        #include <iostream>
        using namespace std;
        class father{
        public:
            father(){cout<<"father ctor"<<endl;}
        };
        class son:public father{
        public:
            int n;
        };
        class son2:public father{
        public:
            son2(int x){n=x;cout<<"son2 ctor"<<endl;}
            int n;
        };
        int main(){
            son s;
            cout<<"===================="<<endl;
            // son2 s2; //error
            cout<<"===================="<<endl;
            son2 s3(3);
            return 0;
        }
        ```
        ```shell
        $ ./fatherctor.out
        father ctor
        ====================
        ====================
        father ctor
        son2 ctor
        ```
    1. 如果一个类中有virtual方法，不论是声明还是继承来的，因为需要虚表，所以必然得有构造函数来初始化vptr的值令其指向虚表，这种情况下，没有任何构造函数的类编译器会合成一个默认构造函数（为了初始化虚表），有构造函数的编译器会进行相应的扩展。
    1. 相应的，如果类在继承链上有一个虚继承的基类，那么编译器很可能需要在类中插入一个指向该基类的指针（或者其它机制，总之需要进行一翻操作），这就要求一个构造函数来操作。所以这种情况下，如果类没有任何构造函数编译器就合成一个完成该操作的合成默认构造函数；若有程序员提供的构造函数，编译器就扩展它们。

    总结：以上四种情况说明的是 ，***对编译器而言，有必要合成默认构造函数***的情况。也就是编译器合成默认构造函数或者扩展构造函数的目的与作用：

    1. 调用类成员的默认构造函数
    1. 调用父类的默认构造函数
    1. 为类初始化虚表相关的操作
    1. 为类初始化虚基类机制

    一旦上面四条对于一个类来说都不需要，那么合成的默认构造函数被就是trivial（没有用）的，而实际上编译器不会将这种构造函数生成出来。
    还记得primer中有个聚合类（aggregate class），当聚合类不含类成员时，上面四个状态就都不满足，那编译器就不会给他生成合成默认构造函数了。所以：

    1. 任何class没有定义构造函数，编译器会生成一个合成默认构造函数
    1. 编译器合成的默认构造函数会初始化所有类的成员

    这两个观点都不正确

1. 同样，拷贝构造函数也有trivial和nontrivial之分，编译器只会生成nontrivial的拷贝构造函数。而判断trivial的标准就在于类是否展现出位逐次拷贝（Bitwise Copy Semantics）特性。首先，看下primer是怎么说的：

    > 与合成默认构造函数不同，即使我们定义了其它构造函数，编译器也会为我们合成一个拷贝构造函数。

    > 而一般情况，合成的拷贝构造函数会将其参数的成员逐个拷贝到正在创建的对象中。

    下面是inside的分析：
    1. 当一个类的成员中有一个类，该类成员有拷贝构造函数（不论是它显示声明的，还是编译器因为非位逐次拷贝而生成的。第二个说法有点绕口，其实就是说，如果该类成员也有一个有拷贝构造函数的类成员。。。。递归中。。。。）
    1. 当类的继承链上有一个有拷贝构造函数（和上面括号中描述的一样，两种情况），这两种情况下类是无法进行位逐次拷贝的，此时编译器会生成一个默认拷贝构造函数来调用类成员／基类的拷贝构造函数：

        ```cpp
        #include <iostream>
        using namespace std;
        struct bitwise{
            int n;
            char *hehe;
        };
        struct goodcopy{
            goodcopy(){}
            goodcopy(const goodcopy&){cout<<"good copy!!"<<endl;}
        };
        struct nonbitwise{
            goodcopy gc;
        };
        struct nonbitwise2:public goodcopy{

        };
        struct nonbitwise3{
            nonbitwise3(){}
            nonbitwise3(const nonbitwise3&){}
            goodcopy gc;
        };
        int main(){
            bitwise bw1;
            char * str="blabla!";
            bw1.hehe=str;
            bw1.n=1;
            bitwise bw2(bw1);
            cout<<"bw1 n "<<bw1.n<<" hehe "<<bw1.hehe<<endl;
            cout<<"bw2 n "<<bw2.n<<" hehe "<<bw2.hehe<<endl;
            cout<<"============================="<<endl;
            nonbitwise nbw1;
            nonbitwise nbw12(nbw1);
            cout<<"============================="<<endl;
            nonbitwise2 nbw2;
            nonbitwise2 nbw22(nbw2);
            cout<<"============================="<<endl;
            nonbitwise3 nbw3;
            nonbitwise3 nbw32(nbw3);
            return 0;
        }
        ```
        ```shell
        $ ./bitwisecopy.out
        bw1 n 1 hehe blabla!
        bw2 n 1 hehe blabla!
        =============================
        good copy!!
        =============================
        good copy!!
        =============================

        ```
    上面的代码中，不能进行位逐次拷贝的nonbitwise2和nonbitwise都由编译器生成了nontrivial的拷贝构造函数来调用goodcopy的拷贝构造函数；而nonbitwise3虽然也不能位逐次拷贝，但是由于程序员定义了拷贝构造函数，所以编译器没有为其生成，而且也没有对其进行扩展，这一点与默认构造函数不同
    1.
    如果一个类中含有虚函数，那这个类中必然会有虚表。这样的话，如果这个类之间进行拷贝（子类到子类，类型相同），使用位逐次拷贝是可以的，因为两个类类型相同，虚表指针指向相同的虚表。但是考虑这种情况，当一个子类拷贝给一个父类时，仍然使用位逐次拷贝，可以么？这种情况下的拷贝会发生子类的截断，即只拷贝其基类部分。而位逐次拷贝会使基类的虚表指针指向子类的虚表，而其子类部分又不存在（因为这货是基类啊）所以不能用位逐次拷贝，需要对虚表指针进行处理。综合一下就是，类中含有虚函数，就不能位逐次拷贝。
    1. 同样的，如果一个类的继承链上存在虚继承关系，也不能单纯的用位逐次拷贝处理子类拷贝给父类的情况，而需要编译器合成一个拷贝构造函数。
1. 对于拷贝构造在函数传参和返回中的使用，主要注意NRV。先来看一个函数是如何返回一个类的：

    ```cpp
    struct someone{
        someone(const someone &x){/*...*/}
    //...
    }
    someone func(int n){
        someone t;
        //...
        return t;
    }
    someone r=func(1);
    //实际中编译器很可能会产生如下代码
    void func(int n, someone& ret){
        someone t;
        //...
        ret.someone::someone(t);
    }
    someone r; //此处不执行someone的默认构造函数
    func(1,r);
    ```
    这里就可以发现返回类的函数被处理为通过参数来返回的函数了。这时，完全可以省去临时类t：
    ```cpp
    void func(int n, someone& ret){
        ret.someone::someone();
        //...
        return;
    }
    ```
    这就是NRV优化（Named Return Value），也就是对函数中的中间类直接用返回值的引用替换，省去一个临时类，就省去了一次默认构造和拷贝构造以及一次析构的开销。但是NRV也有缺点：
    1. 不同的编译器，其NRV的实现程度不一样，也就是编译器不一定会实现NRV
    1. 不同复杂度的函数，NRV实现程度也不一样

    这些不确定性让我们不敢依靠NRV来评估软件性能（所以c++11的右值引用是一大亮点）。另外，关于到底要不要明确的写出拷贝构造函数，总结一下就是，如果类可以位逐次拷贝，那就不要写，因为编译器会生成一个trivial的拷贝构造函数，效率很高，自己写的话很有可能使效率下降。至于书中说的explicit copy constructor与NRV的关系，我查了下，见这篇[文章](http://www.cnblogs.com/cyttina/archive/2012/11/26/2790076.html)，就是说，嗯，没关系。。。。

1. 最后，关于构造函数初始化列表：
    1. 注意初始化顺序，按成员声明的顺序来。尽量不要交织。
    1. 尤其不要用在子类的初始化列表中用子类的成员函数为父类的构造函数提供参数，比如：

    ```cpp
    struct father{
        //...
        father(int n){/*...*/}
        //...
    };
    struct son:public father{
        //...
        son(int x):_t(x),father(memberfunc(x)){/*...*/}
        //...
        int memberfunc(int n){/*...*/}
        int _t;
    };
    //...  编译器有可能把初始化列表转化为
    // son::son(/*...this pointer...*/){
        // father::father(this,this->memberfunc(x)); //尼玛，this的son部分还没构建你就以及调用它的方法了！！！！！
        // _t=x;
    // }
    ```
    最后注意，在构造活着析构函数中调用虚函数，虚函数一定是当前构造／析构函数所属类型的虚函数，因为再往下，子类部分要不还没构建，要不就已经析构了。

---
## 总结
对于构造函数，一句总结就是，如果写了任何构造函数，编译器就不会给你生成默认构造函数了。对于拷贝构造，不需要明确写的就不要写，对于初始化列表，不要“交织”！
