---
title: c++ 隐藏 重载 覆盖
date: 2013-10-24
category: program language
tags: cpp
---

这三个关键字在cpp上经常碰到且很容易混淆，所以在这里记录他们的区别
<!-- excerpt -->

<br/>

##重载 (overload)

**特征**

1. 在同一个类中
2. 函数名相同
3. 参数不同
4. 与 virtual 关键字无关

**例子**

    :::cpp
    class foo{
        public:
            void printFoo(int x){cout<<x<<endl;}
            void printFoo(float x){cout<<x<<endl;}
    };

`printFoo` 名字相同但传入参数不同，这个情况就是重载了。

<br/>

##覆盖 (override)

**特征**

1. 在子类和父类之间
2. 函数名相同
3. 参数相同
4. 父类函数必须有 virtual 修饰

**例子**

    :::cpp
    class father{
        public: 
            virtual void printHello(void){ cout<<"i am father"<<endl;}
    };

    class son: public father{
        public:
            virtual void printHello(void){ cout<<"i am son"<<endl;}
    }；

覆盖是实现多态的重要特性。

<br/>

##隐藏

**特征**

1. 子类的函数名与父类相同，但**参数不同**，无论有没有 virtual 父类的方法都会被隐藏
2. 子类的函数名与父类相同，参数相同，但是父类函数**没有 virtual 关键字**，父类的函数也会被隐藏。

函数如果被隐藏，其调用的方法不取决与实例，而取决于指向实例的指针。例如 有一个父亲的指针指向了其儿子的实例，当要调用这个指针的儿子方法的时候，其真正调用的是父方法，程序就达不到预期目的了。

**例子**

    :::cpp
    class father
    {
    public:
        void printMethod(void);
    };

    void father::printMethod(void){
        cout<<"father"<<endl;
    }

    :::cpp
    class son: public father{
    public:
        void printMethod(void);
    };

    void son::printMethod(void){
        cout<<"son"<<endl;
    }

    :::cpp
    int main(int argc, char *argv[])
    {
        son tom;
        father *jason = &tom;
        tom.printMethod();
        jason->printMethod();
        return 0;
    }

结果是

    :::cpp
    son
    father

显然失去了多态的特性了。

cpp 的坑不少，一不小心就会掉进去了。尤其是指针，这可能就是 google 要为android 写 RefBase 的原因了。
