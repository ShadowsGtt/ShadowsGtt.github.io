---
title: "[C++] 智能指针实现"
search: false
author: "Shadow"
---

## 1.什么是智能指针 

&emsp;&emsp;C++的智能指针其实就是对普通指针的封装(即封装成一个类)，通过重载 * 和 ->两个运算符，使得智能指针表现的就像普通指针一样。

## 2.智能指针的作用 

&emsp;&emsp;C++ 程序设计中使用堆内存是非常频繁的操作，堆内存的申请和释放都由程序员自己管理。程序员自己管理堆内存可以提高了程序的效率，但是整体来说堆内存的管理是麻烦的，C++11中引入了智能指针的概念，方便管理堆内存。使用普通指针，容易   造成堆内存泄露（忘记释放），二次释放，程序发生异常时内存泄露等问题等，使用智能指针能更好的管理堆内存。

## 3.智能指针实现原理

&emsp;&emsp;智能指针(smart pointer)的通用实现技术是使用引用计数(reference count)。智能指针类将一个计数器与类指向的对象相关联，引用计数跟踪该类有多少个对象的指针指向同一对象。每次创建类的新对象时，初始化指针就将引用计数置为1；当对象作为另一对象的副本而创建时，拷贝构造函数拷贝指针并增加与之相应的引用计数；对一个对象进行赋值时，赋值操作符减少左操作数所指对象的引用计数（如果引用计数为减至0，则删除对象），并增加右操作数所指对象的引用计数；调用析构函数时，析构函数减少引用计数（如果引用计数减至0，则删除基础对象）。

## 4.使用C++标准库中的智能指针 
&emsp;&emsp;智能指针在C++11版本之后提供，包含在头文件`#include <memory>`中，`shared_ptr、unique_ptr、weak_ptr`

#### **shared_ptr**

&emsp;&emsp;利用引用计数->每有一个指针指向相同的一片内存时，引用计数+1，每当一个指针取消指向一片内存时，引用计数减1，引用计数减为0时释放内存

#### **week_ptr**
&emsp;&emsp;弱指针 : 辅助shared_ptr解决循环引用的问题

&emsp;&emsp;weak_ptr是为了配合shared_ptr而引入的一种智能指针，因为它不具有普通指针的行为，没有重载operator*和->,它的最大作用在于协助shared_ptr工作，像旁观者那样观测资源的使用情况。weak_ptr可以从一个shared_ptr或者另一个weak_ptr对象构造，获得资源的观测权。但weak_ptr没有共享资源，它的构造不会引起指针引用计数的增加。使用`weak_ptr`的成员函数`use_count()`可以观测资源的引用计数，另一个成员函数`expired()`的功能等价于`use_count()==0`,但更快，表示被观测的资源(也就是shared_ptr的管理的资源)已经不复存在。weak_ptr可以使用一个非常重要的成员函数lock()从被观测的shared_ptr 获得一个可用的shared_ptr对象，从而操作资源。但当expired()==true的时候，lock()函数将返回一个存储空指针的shared_ptr



### unique_ptr

&emsp;&emsp;“唯一”拥有其所指对象，同一时刻只能有一个unique_ptr指向给定对象（禁止拷贝、赋值），可以释放所有权，转移所有权


### 5.智能指针的实现
```
/*************************************************************************
	> Motto : Be better!
	> Author: ShadowsGtt 
	> Mail  : shadowsgtt@gmail.com
 ************************************************************************/

#include<iostream>
#include<mutex>
using namespace std;

/*  实现一个线程安全的智能指针 */


/* 引用计数基类 */
class Sp_counter
{
    private :
        size_t *_count;
        std::mutex mt;
    public :
        Sp_counter()
        {
            cout<<"父类构造,分配counter内存"<<endl;
            _count = new size_t(0);
        }
        virtual ~Sp_counter()
        {
            if(_count && !(*_count) ){
                cout<<"父类析构"<<endl;
                cout<<"[释放counter内存]"<<endl;
                delete _count;
                _count = NULL;
            }
        }
    Sp_counter &operator=(Sp_counter &spc)
    {
        cout<<"父类重载="<<endl;
        cout<<"[释放counter内存]"<<endl;
        delete _count;
        this->_count = spc._count;
        return *this;
    }
    Sp_counter &GetCounter()
    {
        return *this;
    }
    size_t Get_Reference()
    {
        return *_count;
    }
    virtual void Increase()
    {
        mt.lock();
        (*_count)++;
        //cout<<"_count++:"<<*_count<<endl;
        mt.unlock();
    }
    virtual void Decrease()
    {
            mt.lock();
            (*_count)--;
            //cout<<"_count--:"<<*_count<<endl;
            mt.unlock();
        }
};

template<typename T>
class smart_pointer : public Sp_counter
{
    private :
        T *_ptr;
    public :
        smart_pointer(T *ptr = NULL);               
        ~smart_pointer();                               
        smart_pointer(smart_pointer<T> &);
        smart_pointer<T> &operator=(smart_pointer<T> &);
        T &operator*();
        T *operator->(void);
        size_t use_count();
        
};


/* 子类参构造函数&带参数构造函数 */
template<typename T>
inline smart_pointer<T>::smart_pointer(T *ptr)
{
    if(ptr){
        cout<<"子类默认构造"<<endl;
        _ptr = ptr;
        this->Increase();
    }
}   


/* 子类析构函数 */
template<typename T>
smart_pointer<T>::~smart_pointer()
{
    /* 指针非空才析构 */
    if(this->_ptr){
        cout<<"子类析构,计数减1"<<endl;
        if(this->Get_Reference())
            this->Decrease();
        if(!(this->Get_Reference()) ){
            cout<<"(((子类析构,主内存被释放)))"<<endl;
            delete _ptr;
            _ptr = NULL;
        }
    }
}

/* 得到引用计数值 */
template<typename T>
inline size_t smart_pointer<T>::use_count()
{
    return this->Get_Reference();
}

/* 拷贝构造 */
template<typename T>
inline smart_pointer<T>::smart_pointer(smart_pointer<T> &sp)
{
    cout<<"子类拷贝构造"<<endl;

    /* 防止自己对自己的拷贝 */
    if(this != &sp){
        this->_ptr = sp._ptr;
        this->GetCounter() = sp.GetCounter();
        this->Increase();
    }
    
}
/* 赋值构造 */
template<typename T>
inline smart_pointer<T> &smart_pointer<T>::operator=(smart_pointer<T> &sp)
{

    /* 防止自己对自己的赋值以及指向相同内存单元的赋值 */
    if(this != &sp){

        cout<<"赋值构造"<<endl;

        /* 如果不是构造一个新智能指针并且两个只能指针不是指向同一内存单元 */
        /* =左边引用计数减1,=右边引用计数加1 */
        if(this->_ptr && this->_ptr != sp._ptr){
            this->Decrease();

            /* 引用计数为0时 */
            if(!this->Get_Reference()){
                cout<<"引用计数为0,主动调用析构"<<endl;
                this->~smart_pointer();
                //this->~Sp_counter();
                cout<<"调用完毕"<<endl;
            }
        }

        this->_ptr = sp._ptr;
        this->GetCounter() = sp.GetCounter();
        this->Increase();
    }
    return *this;
}

/* 重载解引用*运算符 */
template<typename T>
inline T &smart_pointer<T>::operator*()
{
    return *(this->_ptr);
}
template<typename T>
inline T *smart_pointer<T>::operator->(void)
{
    return this->_ptr;
}

int main()
{
    int *a = new int(10);
    int *b = new int(20);
    cout<<"-------------默认构造测试----------------->"<<endl;
    cout<<"构造sp"<<endl;
    smart_pointer<int> sp(a);
    cout<<"sp.use_count:"<<sp.use_count()<<endl;
    cout<<"------------------------------------------>"<<endl<<endl;
    
    {
        cout<<"-------------拷贝构造测试----------------->"<<endl;
        cout<<"构造sp1  :sp1(sp)"<<endl;
        smart_pointer<int> sp1(sp);
        cout<<"构造sp2  :sp2(sp)"<<endl;
        smart_pointer<int> sp2(sp1);
        cout<<"sp1和sp2引用计数为3才是正确的"<<endl;
        cout<<"sp1.use_count:"<<sp1.use_count()<<endl;
        cout<<"sp2.use_count:"<<sp2.use_count()<<endl;
        cout<<"------------------------------------------>"<<endl<<endl;
        cout<<"调用析构释放sp1,sp2"<<endl;
    }
    cout<<"-------------析构函数测试----------------->"<<endl;
    cout<<"此处sp.use_count应该为1才是正确的"<<endl;
    cout<<"sp.use_count:"<<sp.use_count()<<endl;
    cout<<"------------------------------------------>"<<endl<<endl;
    
    cout<<"-------------赋值构造测试----------------->"<<endl;
    cout<<"构造sp3  :sp3(b)"<<endl;
    smart_pointer<int> sp3(b);
    cout<<"sp3.use_count:"<<sp3.use_count()<<endl;
    cout<<"sp3 = sp"<<endl;
    sp3 = sp;
    cout<<"sp3先被释放,然后sp3引用计数为2才正确,sp的引用计数为2才正确"<<endl;
    cout<<"sp3.use_count:"<<sp3.use_count()<<endl;
    cout<<"sp.use_count :"<<sp.use_count()<<endl;
    cout<<"------------------------------------------>"<<endl<<endl;
    
    cout<<"-------------解引用测试----------------->"<<endl;
    cout<<"*sp3:"<<*sp3<<endl;
    cout<<"*sp3 = 100"<<endl;
    *sp3 = 100;
    cout<<"*sp3:"<<*sp3<<endl;
    cout<<"------------------------------------------>"<<endl;

    

   // cout<<"sp3.use_count:"<<sp3.use_count()<<endl;
    //cout<<"sp.use_count:"<<sp.use_count()<<endl;


    cout<<"===================end main===================="<<endl;
    return 0;
}
```
