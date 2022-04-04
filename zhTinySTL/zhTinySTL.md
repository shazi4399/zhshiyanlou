# 一 C++ STL 简介

## 1 STL 介绍

本次课程主要面对有一定 c++ 基础（了解基本语法，熟悉常用特性）的 ，想要学习 c++ 更深入特性 ，掌握 c++ 强大标准库的同学 。通过本次课程，你将学习到 c++ template ，异常处理 ，并回顾数据库的部分知识 ，初步掌握 STL 开发 ，避免重复制造轮子。

- 将学习到的知识点：
  - 模板编程
  - 泛型编程
  - STL 常用组件
  - lambda 表达式
  - 异常处理
  - 内存处理
  - 部分数据结构
  - 部分算法

> 提示：本课程所有代码至少需要开启 -std=c++11 选项来支持 C++11 相关特性，在介绍 C++14 特性时的相关代码需要开启 -std=c++14 的编译选项，例如：

- g++ main.cpp -std=c++11
- g++ main.cpp -std=c++14

**如果你没有使用过 STL，那么你是不爱 c++ 的**，STL 的原名是“Standard Template Library”，翻译过来就是标准模板库。STL 是 C++ 标准库的一个重要组成部分，STL 实现了常用的数据结构和算法 ，蕴含其间的泛型编程和代码复用的思想深刻的影响了编程习惯，像微积分延长天文学家寿命一样，STL 延长了程序员的寿命。 STL 由**算法**，**容器**，**迭代器**，**适配器**，**仿函数（函数对象）**，**空间适配器**六大部件组成 。我们将主要讲解容器，迭代器，算法和仿函数。适配器的部分会交给学员来实现，而空间适配器不会太过于深入。从上往下学习 STL，学习曲线不再那么陡峭。

## 2 容器



鱼缸是容器，瓶子是容器，饭碗也是容器，STL 的容器也不列外。这里的容器首先是一个模板类，在类中实现对数据的操作，而包含这样的类的实现就叫一个容器。STL 有许多这样的容器，它们包括：向量（vector），列表（list），队列（queue），双端队列（deque），优先队列（Priority queue），集合（set），多种集合（multiset），映射（map），多重映射（multimap）。



## 3 算法

数据结构加算法等于程序，如果说容器实现了数据结构的话，那么算法就是 STL 的灵魂 ，STL 的算法是一种通用的算法，并不依赖于特定的数据结构和对象 。这样的好处是不用针对每种情况编写特定的代码，而是给出一种通用的做法，是代码复用的一种实现方法，模板编程则是泛型编程的基础。



## 4 迭代器

让我们演示一个简单的函数： `add（int &a ,int &b）` ，它传入两个引用，然后执行加法操作，可以看到它依赖于 int 这个特定的类型，而且暴露了这个函数的内部结构不利于对底层的隔离和封装。那么 STL 是怎么解决这个问题的呢？他们使用了迭代器（**对指针的一种泛化**）。迭代器底层是由指针实现的，是容器和算法的桥梁。STL 里大多数容器都实现了自己的迭代器，我们可以使用迭代器来完成对容器的访问。后面我们会详细讲到迭代器的种类，性质，使用，实现。



## 5 适配器

学习过数据结构的同学大都知道，数据结构不是独立的，部分数据结构是可以相互转换的。比如栈和队列可以互相实现。当我们需要一个碗的时候我们不一定重新制造，我们可以把瓶子的上部去掉。同样的道理，当我们需要队列（queue）的时候，也可以用双端队列（deque）去实现。而 queue 就叫做适配器。STL 有三种基本容器 vector，deque，list。有用基本容器扩展的适配器 queue，stack 等。适配器主要有容器适配器，迭代器适配器，函数适配器，它们的作用范围不同，意思大致一致。后面我们也会详细讲到。



## 6 仿函数

仿函数又叫做函数对象，**其本质是类的对象，一种可回调机制**，在类中重载了**（）**运算符，使对象在用（）时呈现出函数的特性，所以叫做仿函数。叫仿函数体现了它的作用，叫函数对象体现其本质，大家喜欢叫什么都可以。而为什么需要仿函数呢？因为 STL 没有也不可能将所有东西都包含到函数中，而程序是对现实的模拟，现实又是最复杂的，一个 `sort（）`，你要 `<`，我要 `>` 。如何协调呢？我们可以定义自己需要的仿函数，定制自己的操作。具体的内容我们后面会讲。这儿只做说明。



## 7 空间配置器

c++ 的一大魅力就是对底层的操作，你像一个魔法师一样，挥舞着魔杖操纵着底层的各种资源。当然一个不好，程序也崩给你看。而空间配置器就是 STL 自己的“内存池”。完成对内存的申请，释放，维护。配置器有两个部分：一级空间配置器，二级空间配置器。本次课程不会过度讲解配置器，感兴趣的同学可以去看一下实验楼另外一个课程：[c++ 实现高性能内存池](https://www.lanqiao.cn/courses/566)。



## 8 实验总结

STL 是学习 C++ 路上必须领略的美景，STL 由六个部分组成：容器，迭代器，算法，仿函数，适配器，空间配置器。各个部件相互调用，相互关联。运用泛型，模板，oop 等思想，是学习和理解 c++ 这门语言的必经之路。

# 二 template 编程和迭代器粗解

## 1 实验简介

#### 实验内容

本节内容主要讲述 c++11 模板的用法，以后的代码中会大量的用到模板的知识。同时简单讲解迭代器的相关知识，为后面容器和算法的内容作铺垫。

#### 实验知识点

- 模板编程
  - 基本语法
  - 模板函数
  - 类模板和成员模板
  - 模板类中的静态成员
  - typename 和 class
- 迭代器
  - 迭代器详解
  - 迭代器种类和使用

## 2 模板编程

#### 基本语法

模板编程是 STL 的基石，也是 c++11 的核心特性之一。模板是相对于编译器而言，顾名思义就是向编译器提供一个处理事务的模板，以后需要处理的东西，如果都是这个事务类型，那么统统用这个模板处理。

模板的基本语法如下：

```
template <typename/class T>
```

template 告诉编译器，接下来是一个模板 ，typename 和 class 都是关键字，在这里二者可以互用没有区别。在 `< >` 中 `T` 叫做模板形参，一旦模板被实例化，`T` 也会变成具体的类型。接下来，我们看一个例子。

## 3 模板函数

代码实例：

```c++
template <typename T>
T  add(const T lva ,const T rva)
{
    T a ;
    a = lva + rva ;
    return a;
}
```

这是一个模板函数的简单实例，所有模板函数在开始都需要 `template` 语句，以告诉编译器这是一个模板和参数等必要信息，当然里面的 `T` 可以取任意你喜欢的名字 ，模板参数个数也是任意更换的。 还要提醒的一点是：`template <typename T1 ,typename T2 = int>` 函数模板是支持默认参数的，T1 、T2 顺序在默认情况下是可以任意的，不用严格按照从右到左的顺序。

然后就是使用了，我们可以写出`add(1,2)` 这样的函数,也可以写出 `add(2.5,4.6)` 这样的函数，向 `add` 函数提供参数时，编译器会自动分析参数的类型，然后将所有用到 T 定义的换成相对性的类型，以上的两个函数在编译期间会生成

```c++
int add(const int lva ,const int rva)
{
    int a ;
    a = lva + rva ;
    return a;
}
double add(const double lva ,const double rva)
{
    double a ;
    a = lva + rva ;
    return a;
}
```

这样的两个具体函数。如果我们使用`add(1,2.0)`是会报错的，编译器无法找到`add(int,double)`。大家可以自己分析一下为什么。

## 4 类模板和成员模板



#### 类模版

c++11 不仅支持对函数的模板化，也支持对类的模板，下面来看基本的语法是怎样的：

```c++
template <class T>
class Myclass
{
    T a;
    public:
        T add(const T lva ,const T rva);
};

template <class T>
T Myclass<T>::add(const T lva, const T rva)
{
    a = lva + rva;
    return a;
}
```

这是一个简单并且典型的类模板，在程序中给出模板并不能使用它，还必须实例化，比如：

`Myclass<int> A；` //用 int 实例化一个类 A

`Myclass<double> B；` //用 double 实例化一个类 B

当程序编译到这里时就会按照我们给出的类型，声明两组类和两组类函数。注意，在这里我们一定要显式给出类型 T 。类模板不像是函数模板 ，函数模板会根据参数推断类型。 当然类模板也支持默认参数，但是类模板必须严格从右往左默认化。

#### 成员模板

模板的使用范围是广泛的，不仅可以用作函数模板，类模板，还可以用作 class ，struct ，template class 的成员。而要实现 STL 这是我们必须掌握和使用的特性。我们先看一个简单的例子,用上面的类改编而来：

```c++
 template <class T>
 class Myclass
 {
     public:
        T a;
        template <typename type_1 , typename type_2>
         type_1 add(const type_1 lva ,const type_2 rva);
 };

 template <class T>
     template <typename type_1,typename type_2>
 type_1 Myclass<T>::add(const type_1 lva, const type_2 rva)
 {
     a = lva + rva;
     return a;
 }
```

在类的声明中使用了一个嵌套的模板声明。且通过作用域运算符 **::** 指出 add 是类的成员，需要注意的一点，有些编译器不支持模板成员，而有些编译器不支持在类外定义。我们默认大家的编译器都支持。模板如此强大，甚至允许我们在模板类中再建立模板类：

```c++
 template <class T>
 class Myclass
 {
     public:
        T a;
        template <typename type_1 , typename type_2>
         type_1 add(const type_1 lva ,const type_2 rva);

         template <class type_3>
         class Myclass_2;         // 声明放在这里，具体定义放在类外进行。
         Myclass_2<T> C;          // 定义一个Myclass_2 类 A。使用 T 进行实例化
 };

 template <class T>
     template <typename type_1,typename type_2>
 type_1 Myclass<T>::add(const type_1 lva, const type_2 rva)
 {
     a = lva + rva;
     return a;
 }

 template <class T>
     template <class type_3>
     class Myclass<T>::Myclass_2
     {
         public:
             type_3 value;
             type_3 sub(const type_3 a , const type_3 b) {value = a - b;}
     };
```

当然我们暂时还用不到这样复杂的东西，这里只是展现了模板的部分特性。



## 5 模板类中的静态成员

我们知道，在类中定义的静态成员是存储在静态区中，被所有类对象共享，并不属于某一个类所有，同样的在模板类中的静态成员也不会被复制多份，而是被同类实例化的类对象共享，比如所有 int 和所有 double 的类对象，享有相互独立的静态变量。也可以说是编译器生成了 int 和 double 两个版本的类定义。



## 6 typename 和 class

`typename`和`class`是模板中经常使用的两个关键词 ，在模板定义的时候没有什么区别。以前用的是 class，后来 c++ 委员会加入了 typename。因为历史原因，两个是可以通用的。对有些程序员来说，在定义类模板的时候，常常使用 class 作为关键字，增加代码可读性。其它则用 typename，上面的代码大都遵循这样的标准，但是并无强制规定。但是如果二者没有差别，为什么还要加入 typename 呢？**c++标准委员会不会增加无用的特性**，让我们来看一个例子：

```c++
class Myclass{
    public:
        Myclass();
        typedef int test;  //定义类型别名
}
template <class T>
class Myclass2{
    public:
        Myclass2();
        T::test *a  // 声明一个指向T::test类型的指针。
        //   typename T::test * a
}
```

以上的代码没有全部写完，大家觉得编译器能够过吗？答案是不能,因为在 c++ 中，允许我们在类中定义一个类型别名，且使用的时候和类名访问类成员的方法一样。这样编译器在编译的时候就会产生二义性，它根本不知道这是一个类型还是别名，所以我们加上 typename 显式说明出来。当然如果这里没有二义性，比如`Myclass ::test * a` ,加上 typename 是会报错的。此外，在 class 的 STL 底层还有一个特性，用于保留模板参数，但是在 c++17 中已经舍弃，所以我们没有讲。

## 7 实验总结

模板是 c++ 最重要的特性之一，模板函数、模板类、类中的模板函数、类中的模板类、模板类中的模板类等等，可以写出太多强大的代码，这也是模板的魅力所在，而 STL 就是基于模板的，大家一定要掌握模板的基本用法。

> 引用《c++ primer》, 《STL 源码解析》

# 三 迭代器

## 1 实验内容

本节实验我们将为大家讲解迭代器，主要介绍 5 种常见迭代器：输入、输出迭代器，前向逆向迭代器，双向迭代器和随机迭代器。主要内容包括各自的构造方法和操作方法。

#### 知识点

- 输出迭代器
- 输入迭代器
- 前向迭代器
- 双向迭代器
- 随机迭代器
- 迭代器辅助函数

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件 `Iterator.h`。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip
unzip -q mySTL.zip -d ./Code/
```

### 2-1 迭代器详述

迭代器（iterator）是一种对象，它能够用来遍历标准模板库容器中的部分或全部元素，每个迭代器对象代表容器中的确定的地址。迭代器修改了常规指针的接口，所谓迭代器是一种概念上的抽象：那些行为上像迭代器的东西都可以叫做迭代器。然而迭代器有很多不同的能力，它可以把抽象容器和通用算法有机的统一起来。迭代器基本分为五种，输入输出迭代器，前向逆向迭代器，双向迭代器和随机迭代器。

**简单概括**：迭代器是一种检查容器内元素并遍历元素的可带泛型数据类型。

下面，我们新建头文件 `Iterator.h` 是头文件，用来实现我们的迭代器，这里的代码需要引用到系统头文件 `#include <cstddef>`，它主要用于定义一些类型。接下来我们定义 5 种迭代器的类型，将其写入 `Iterator.h` 文件中：

```c++
struct input_iterator_tag{};//返回输入迭代器
struct output_iterator_tag{};//返回输出迭代器
struct forward_iterator_tag :public input_iterator_tag {};//返回前向迭代器
struct bidirectional_iterator_tag :public forward_iterator_tag {};//返回双向迭代器
struct random_access_iterator_tag :public bidirectional_iterator_tag {};//返回随机迭代器
```



### 2-2 输入迭代器

通过对输入迭代器解除引用，它将引用对象，而对象可能位于集合中。通常用于传递地址。

```c++
template<class T, class Distance>
struct input_iterator {
    typedef input_iterator_tag iterator_category;//返回类型
    typedef T                  value_type;//所指对象类型
    typedef Distance           difference_type;//迭代器间距离类型
    typedef T*                 pointer;//操作结果类型
    typedef T&                 reference;//解引用操作结果类型
};
```



### 2-4 输出迭代器

该类迭代器和输入迭代器极其相似，也只能单步向前迭代元素，不同的是该类迭代器对元素只有写的权力。通常用于返回地址。

```c++
struct output_iterator{
    typedef output_iterator_tag iterator_category;
    typedef void                value_type;
    typedef void                difference_type;
    typedef void                pointer;
    typedef void                reference;
};
```

### 2-5 前向迭代器



前向迭代器可以在一个正确的区间中进行读写操作，它拥有输入迭代器的所有特性，和输出迭代器的部分特性，以及单步向前迭代元素的能力。通常用于遍历。

```c++
template <class T, class Distance> struct forward_iterator{
    typedef forward_iterator_tag    iterator_category;
    typedef T                        value_type;
    typedef Distance                difference_type;
    typedef T*                        pointer;
    typedef T&                        reference;
};
```

### 2-6 双向迭代器



该类迭代器是在前向迭代器的基础上提供了单步向后迭代元素的能力，前向迭代器的高级版。

```c++
template <class T, class Distance> struct bidirectional_iterator{
    typedef bidirectional_iterator_tag    iterator_category;
    typedef T                        value_type;
    typedef Distance                difference_type;
    typedef T*                        pointer;
    typedef T&                        reference;
};
```



### 2-7 随机迭代器

该类迭代器能完成上面所有迭代器的工作，它自己独有的特性就是可以像指针那样进行算术计算，而不是仅仅只有单步向前或向后迭代。

```c++
template <class T, class Distance> struct random_access_iterator{
    typedef random_access_iterator_tag    iterator_category;
    typedef T                        value_type;
    typedef Distance                difference_type;
    typedef T*                        pointer;
    typedef T&                        reference;
};
```



### 2-8 迭代器辅助函数

迭代器的实现，我们这里主要会使用到两个函数来辅助完成操作，分别是 advace 函数和 distance 函数。这两个函数是写到`Algorithm.h`文件中的。

- advance 函数: 用于迭代器前移，增加迭代的位置。可用于定向访问到迭代器的某个变量。

```c++
template<class InputIterator, class Distance>
void _advance(InputIterator& it, Distance n, input_iterator_tag){
    assert(n >= 0);
    while (n--){//当n大于0，迭代器前移n位
        ++it;
    }
}

void advance(InputIterator& it, Distance n){
    typedef typename iterator_traits<InputIterator>::iterator_category iterator_category;
    _advance(it, n, iterator_category());
}
```

- distance 函数: 用于计算迭代器间距离

```c++
template<class InputIterator>
typename iterator_traits<InputIterator>::difference_type_distance(InputIterator first, InputIterator last, input_iterator_tag){
    typename iterator_traits<InputIterator>::difference_type dist = 0;//初始化距离
    while (first++ != last){//当首地址不等于尾地址，距离增加
      ++dist;
    }
    return dist;//返回迭代器间距离
}

template<class Iterator>
typename iterator_traits<Iterator>::difference_type_distance(Iterator first, Iterator last){
    typedef typename iterator_traits<Iterator>::iterator_category iterator_category;
    return _distance(first, last, iterator_category());
}
```

### 2-9 实例测试

完成上面迭代器的实现后，我们现在在 `Test` 目录下新建文件 `iteratortest.cpp`,用于测试 Iterator 的功能。这里的测试我们需要借助到 vector 容器。关于`Vector` 容器的内容，我们将在后面的实验中给大家讲解，这里我们直接使用 `Vector.h`的内容。该文件可以通过下载课程源码找到。

```c++
#include <iostream>
#include "Iterator.h"
#include "Vector.h"

int main()
{
    mySTL::vector<int> vec;
    for(int i = 0;i < 10;i++)
        vec.push_back(i);

    mySTL::vector<int>::iterator it = vec.begin();
    mySTL::vector<int>::iterator end = vec.end();
    std::cout<<"The value of vector:";
    for(;it != vec.end();it++)
    std::cout<<*it<<" ";
    std::cout<<std::endl;

    //advance使用
    it = vec.begin();
    std::cout<<"After advance 3:";
    mySTL::advance(it,3);
    std::cout << *it << " "<<std::endl;

    //distance使用
    std::cout<<"The distance of position 3 to the end:";
    std::cout<<mySTL::distance(it,end)<<std::endl;

    return 0;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539852238529.png)

## 2 实验总结



通过本节实验，我们一步一步的实现了迭代器的具体内容，并通过一个简单的实例对相关函数进行了相应的测试。相信大家对于常见的迭代器的构造方法以及操作方法有了一定的了解。这里再给大家补充几点注意事项：

- 绝对不要对无效迭代器执行解引用操作；
- 当对容器进行插入操作后，先前获得的迭代器已经无效；
- 两个迭代器必须能构成有效的范围；
- 还有不要试图去修改内置内型的临时变量。

# 四 函数对象（仿函数）



## 1 实验内容



#### 知识点

- 函数对象概述
- 预定义函数对象
- 辅助函数对象
- 适配器
- 函数对象使用方法

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件`Functional.h`。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip

# 解压文件到 Code 目录
unzip -q mySTL.zip -d ./Code/
```

## 2 函数对象概述



函数对象是重载函数调用操作符的类的对象。即函数对象是行为类似函数的对象，又称仿函数，是一个能被当做普通函数来调用的对象。

函数对象与函数指针相比，有两个优点：第一是编译器可以内联执行函数对象的调用；第二是函数对象内部可以保持状态。

STL 中的众多算法，非常依赖于函数对象处理容器的元素。所以 STL 预定义了许多函数对象、谓词和适配器。



### 2-1 预定义和辅助函数对象

我们可以在 `Functional.h` 中预定义一些函数对象，以方便在以后的实验中直接调用。

首先在 `include` 目录下创建 `Functional.h`。

- unary_function: 作为一元函数对象的基类，只定义了参数和返回值的类型

```c++
template<class T>
struct unary_function {
    typedef T argument_type;
    typedef T result_type;
};
```

- binary_function:作为二元函数基类，只定义了参数和返回值的类型

```c++
template<class T>
struct binary_function {
    typedef T first_argument_type;
    typedef T second_argument_type;
    typedef T result_type;
};
```

- less:用于返回较小值

```c++
template<class T>
struct less{
    typedef T first_argument_type;
    typedef T second_argument_type;
    typedef bool result_type;

    result_type operator()(const first_argument_type& x, const second_argument_type& y){
        return x < y;
    }
};
```

- equal_to: 判断是否相等

```c++
template<class T>
struct equal_to{
    typedef T first_argument_type;
    typedef T second_argument_type;
    typedef bool result_type;
    result_type operator()(const first_argument_type& x, const second_argument_type& y){
        return x == y;
    }
};
```

- identity: 验证同一性

```c++
template <class T>
struct identity : public unary_function<T> {
    const T& operator()(const T& x) const {return x;}  //函数调用操作符
};
```

- select1st: 返回键值，在 map 中会用到

```c++
template <class T>
struct select1st : public unary_function<T, typename T::first_type> {
    const typename T::first_type& operator()(const T& x) const {return x.first;}
};
```

## 3 适配器

函数对象适配器本质上任然是一个函数；函数对象适配器提供了对函数对象或者普通函数的操作，使其能够根据我们的需求来修改函数对象或者普通函数的功能。

使用函数对象适配器的步骤:

（1）首先让自定义的函数对象 public 继承一个父类。这里有两个选择：binary_function 和 unary_function。如果有两个参数选择前者。

（2）定义一个函数对象作为参数传入函数对象适配器。常见的函数对象适配器有:

- 绑定适配器 bind1st bind2nd (bind1st 绑定第一个参数, bind2nd 绑定第二个参数)
- 取反适配器 not1 not2 (not1 作用于一元函数对象，not2 作用于二元函数对象)
- 普通函数适配器 ptr_fun
- 作用于类中方法的适配器 mem_fun mem_fun_ref

（3）加 const



### 3-1 实例测试

我们在上面讲了预定义函数对象和适配器，在下面我们就来演示预定义函数和适配器的搭配。我们在 `Test` 文件夹下建立 `functionaltest.cpp`，在这里需要用到 vector 容器来测试，关于`Vector` 容器的内容，我们将在后面的实验中给大家讲解，这里我们直接使用 `Vector.h`的内容。该文件可以通过下载课程源码找到。

```c++
#include <iostream>
#include "Vector.h"
#include "Functional.h"

using namespace std;

class compare:public binary_function<int,int,bool>{//用于接收两个参数
public:
    bool operator()(int i, int num) const {
        return i > num;
    }
};

class comparetonum:public unary_function<int,bool>{//用于接收一个参数
public:
    bool operator()(int i) const {
        return i > 5;
    }
};

void print(int i,int j)//普通函数对象
{
    if (i > j){
        cout << i << " ";
    }
}

int main(){

    mySTL::vector <int> vec;
    for (int i = 0; i < 10; i++)
    {
        vec.push_back(i + 1);
    }

    mySTL::vector<int>::iterator it = find_if(vec.begin(), vec.end(), bind2nd(compare(),6));//找出大于6的第一个数
    if (it == vec.end())
    {
        cout << "cannot find the number!" << endl;
    }
    else
    {
        cout << "find num: " << *it << endl;
    }

    mySTL::vector<int>::iterator rit = find_if(vec.begin(), vec.end(), not1(comparetonum()));  //取反适配器的用法，找出小于5的第一个数
    if (rit == vec.end())
    {
        cout << "cannot find the number!" << endl;
    }
    else
    {
        cout << "find num: " << *rit << endl;
    }

    mySTL::vector<int> vec1;
    for (int i = 0; i < 10; i++)
    {
        vec1.push_back(i);
    }

    cout<<"The num larger than 5: ";
    mySTL::for_each(vec1.begin(), vec1.end(), bind2nd(ptr_fun(print),5)); //使用ptr_fun将普通函数转换为函数对象，然后给函数对象绑定参数。
    cout << endl;

    return 0;
}
```

执行命令：

```bash
g++ functionaltest.cpp -std=c++11 -o functionaltest -I ../include
```

-I 命令是说，先到../include文件夹里找头文件，找不到再用系统的头文件。

![](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539843160037.png)



## 4 总结

通过本次实验，我们已经实现了预定义函数对象，并且通过了测试。接下来我们总结一下：

- 函数对象是重载了“()”操作符的普通类对象，因此从语法上讲，函数对象与普通的函数行为类似。
- 函数对象的优点是可以在内部修改而不改动外部接口，所以非常方便，节省了不少开发时间。

# 五 算法

## 1 实验内容

本次实验主要讲述 STL 常用算法和基本算法，算法在日常的开发使用中能够提高不少的开发效率。

#### 知识点

- lambda 表达式
- 常见基本算法

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件 `Algorithm.h`。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip
# 解压文件到 Code 目录
unzip -q mySTL.zip -d ./Code/
```

## 2 lambda 表达式定制操作



#### 泛型算法中的定制操作

很多算法都会比较输入序列中的元素以达到排序的效果，通过定制比较动作，可以控制算法按照编程者的意图工作。 这里我们用排序算法举例。

- 普通排序算法，这样的排序算法只能按照从小到大排序，很多时候是不适用的。

```c++
template<class RandomIterator>
void sort(RandomIterator first, RandomIterator last){
    if (first >= last || first + 1 == last)
        return;
    if (last - first <= 20)//区间长度小于等于20的采用冒泡排序更快
        return bubble_sort(first, last, pred);
        auto mid = mid3(first, last - 1, pred);
        auto p1 = first, p2 = last - 2;
        while (p1 < p2){
        while (pred(*p1, mid) && (p1 < p2)) ++p1;
               while (!pred(*p2, mid) && (p1 < p2)) --p2;
            if (p1 < p2){
                swap(*p1, *p2);
            }
        }
    swap(*p1, *(last - 2));//将作为哨兵的mid item换回原来的位置
    sort(first, p1, pred);
    sort(p1 + 1, last, pred);
}
```

- 排序算法的定制操作,普通排序算法只能由小到大排序，并不智能。二排序算法的定制操作，次函数多了一个类型 BinaryPredicate，可以用来定制规则（如从大到小），增加了实用性。

```c++
template<class RandomIterator, class BinaryPredicate>
void sort(RandomIterator first, RandomIterator last, BinaryPredicate pred){
    if (first >= last || first + 1 == last)
        return;
    if (last - first <= 20)//区间长度小于等于20的采用冒泡排序更快
        return bubble_sort(first, last, pred);
    auto mid = mid3(first, last - 1, pred);
    auto p1 = first, p2 = last - 2;
    while (p1 < p2){
        while (pred(*p1, mid) && (p1 < p2)) ++p1;
        while (!pred(*p2, mid) && (p1 < p2)) --p2;
        if (p1 < p2){
            swap(*p1, *p2);
        }
    }
    swap(*p1, *(last - 2));//将mid item换回原来的位置
    sort(first, p1, pred);
    sort(p1 + 1, last, pred);
}
```

#### 谓词

- 谓词相当于一个动作（将要干什么），比如有一个需求，希望从大到小排序，则可以先定义一个谓词（函数）。

```c++
bool comp(const int& v1,const int& v2)
{
    return v1 > v2;
}
```

- 将这个函数传递给 sort 算法，就可以按照从大到小排序。

```c++
sort(v.begin(),v.end(),comp);
```

#### lambda 表达式

前面的例子中，定义了一个函数传递给 sort 算法。这个函数可以重复使用还好，如果只是使用一次的话就显得比较麻烦，而且浪费了空间。这种情况下就可以使用 lambda 表达式。lambda 表达式相较于谓词，它没有定义函数（没有函数名）。

```c++
sort(v.begin(),v.end(),[]comp(const int& v1,const int& v2){return v1 > v2;});//这种没有定义函数的指定动作（谓词）的方式就是lambda表达式。
```

## 3 STL 算法

算法部分主要包含在头文件 "Algorithm.h" 、 "Functional” 和 “numeric” 中。

- "Algorithm.h"是所有 STL 头文件中最大的一个，其中常用到的功能范围涉及到比较、 交换、查找、遍历操作、复制、修改、反转、排序、合并等等。
- "Functional”中则定义了一些模板类，用以声明函数对象，前面实验已经讲到。
- “Numeric ” 则包含一些数值运算操作，这个会在 STL 数值算法中讲到。

STL 算法大大小小差不多有 70+，下面列举一些我们实验需要常用的。

首先在 `include` 目录下创建 `Algorithm.h`。

- find: 利用底层元素的等于操作符,对指定范围内的元素与输入值进行比较,当匹配时,结束搜索,返回该元素的一个迭代器。用于查找。

函数原型:

```c++
template <class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& val){
    for (; first != last; ++first){
        if (*first == val)
            break;
    }
    return first;
}
```

- sort: 以升序重新排列指定范围内的元素,重载版本使用自定义的比较操作。用于排序。

函数原型:

```c++
template<class RandomIterator>
void sort(RandomIterator first, RandomIterator last){
        return sort(first, last, less<typename iterator_traits<RandomIterator>::value_type>());
}
//重载版本
template<class RandomIterator, class BinaryPredicate>
void sort(RandomIterator first, RandomIterator last, BinaryPredicate pred){
    if (first >= last || first + 1 == last)
        return;
    if (last - first <= 20)//区间长度小于等于20的采用冒泡排序更快
        return bubble_sort(first, last, pred);
    auto mid = mid3(first, last - 1, pred);
    auto p1 = first, p2 = last - 2;
    while (p1 < p2){//快速排序
        while (pred(*p1, mid) && (p1 < p2)) ++p1;
        while (!pred(*p2, mid) && (p1 < p2)) --p2;
        if (p1 < p2){
            swap(*p1, *p2);
        }
    }
    swap(*p1, *(last - 2));//将作为哨兵的mid item换回原来的位置
    sort(first, p1, pred);
    sort(p1 + 1, last, pred);
}
```

- swap: 交换存储在两个对象中的值。

函数原型:

```c++
template <class RandomAccessIterator, class Compare>
void pop_heap(RandomAccessIterator first, RandomAccessIterator last, Compare comp){
    mySTL::swap(*first, *(last - 1));
    if (last - first >= 2)
        mySTL::down(first, last - 2, first, comp);
}
```

- up:上溯算法，用于从下向上遍历

```c++
template<class RandomAccessIterator, class Compare>
//heap上溯算法
static void up(RandomAccessIterator first, RandomAccessIterator last,
RandomAccessIterator head, Compare comp){
    if (first != last){
        int index = last - head;
        auto parentIndex = (index - 1) / 2;
        for (auto cur = last; parentIndex >= 0 && cur != head; parentIndex = (index - 1) / 2){
            auto parent = head + parentIndex;//get parent
            if (comp(*parent, *cur))
                mySTL::swap(*parent, *cur);
            cur = parent;
            index = cur - head;
        }
    }
}
```

- down:下降算法，用于从上向下遍历

```c++
template<class RandomAccessIterator, class Compare>
//heap下降算法
static void down(RandomAccessIterator first, RandomAccessIterator last,
RandomAccessIterator head, Compare comp){
    if (first != last){
    auto index = first - head;
    auto leftChildIndex = index * 2 + 1;
    for (auto cur = first; leftChildIndex < (last - head + 1) && cur < last; leftChildIndex = index * 2 + 1){
        auto child = head + leftChildIndex;//get the left child
        if ((child + 1) <= last && *(child + 1) > *child)//cur has a right child
            child = child + 1;
        if (comp(*cur, *child))
            mySTL::swap(*cur, *child);
        cur = child;
        index = cur - head;
        }
    }
}
```

- copy: 复制算法，常用于赋值

```c++
template<>
inline char *copy(char *first, char *last, char *result){
    auto dist = last - first;
    memcpy(result, first, sizeof(*first) * dist);
    return result + dist;
}
```

## 4 set 算法



STL 中有关 set 给出了四种算法，分别是交集，并集， 差集，对称差集。但是此处的 set 不同于数学中的集合。数学中的集合允许元素以任意次数、任意次序出现，但此处的不允许元素重复出现，而且所有元素按序出现。这四种算法处理的结构也是有序的。

- 交集 set_intersection

交集用于求出两个不同集合中的相同元素。

```c++
template<class InputIterator,class OutputIterator>
OutputIt set_intersection(InputIterator first1, InputIterator last1,
                          InputIterator first2, InputIterator last2,
                          OutputIterator d_first){
    while (first1 != last1 && first2 != last2) {
            if (*first1 < *first2) {
                ++first1;
            }
            else  {
                if (!(*first2 < *first1)) {
                    *d_first++ = *first1++;
                }
                ++first2;
            }
        }
        return d_first;
    }
```

- 并集 set_union

将两个集合合并到一起，相同的元素只在并集中出现一次。

```c++
template<class InputIterator,class OutputIterator>
OutputIt set_union(InputIterator first1, InputIterator last1,
                   InputIterator first2, InputIterator last2,
                   OutputIterator d_first)
{
    for (; first1 != last1; ++d_first) {
        if (first2 == last2)
            return std::copy(first1, last1, d_first);
        if (*first2 < *first1) {
            *d_first = *first2++;
        }
        else {
            *d_first = *first1;
            if (!(*first1 < *first2))
                ++first2;
            ++first1;
        }
    }
    return std::copy(first2, last2, d_first);
}
```

- 差集 set_difference

在集合 1 中出现而没有在集合 2 出现的元素

```c++
template<class InputIterator, class OutputIterator>
OutputIt set_difference(InputIterator first1, InputIterator last1,
                        InputIterator first2, InputIterator last2,
                        OutputIterator d_first)
{
    while (first1 != last1) {
        if (first2 == last2)
            return std::copy(first1, last1, d_first);

        if (*first1 < *first2) {
            *d_first++ = *first1++;
        }
        else {
            if (! (*first2 < *first1)) {
                ++first1;
            }
            ++first2;
        }
    }
    return d_first;
}
```

## 5 实验总结



算法是编程的效率所在，好的算法可以减少空间的浪费，并且可以极大程度的提高时间效率。但是算法更多的是理论上的东西，注重的是学习过程，如果有更优的算法或者什么新奇的算法我们都可以添加到 Algorithm.h 在以后的编程过程中可以直接调用。

# 六 基础容器之vector

## 1 实验内容



本次实验主要讲述 c++11 vector 容器的用法，vector 是 c++11 STL 中一个非常重要的成员，通过对它的使用，我们可以更加高效率的处理数据，通过本次实验你可以实现属于自己的 vector。

#### 知识点

- vector 基础
- vector 初始化
- vector 基本操作
- vector 成员函数

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件`Vector.h`，成员函数封装在 Detail 目录下 `Vector.impl.h`中。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip

# 解压文件到Code目录
unzip -q mySTL.zip -d ./Code/
```

本次实验需要用到的头文件包括： 可以在 `mySTL` 目录下查看

```c++
#include "TypeTraits.h"                    //类型库，抽象定义了一些类型
#include "Allocator.h"                    //空间分配器，已经写好直接调用
#include "Algorithm.h"                    //算法库
#include "Iterator.h"                    //迭代器
#include "ReverseIterator.h"            //逆向迭代器，重载了一些运算符，可以直接使用
#include "UninitializedFunctions.h"        //初始化函数库，直接使用
```

## 2 vector 详解



#### vector 介绍

vector 是 C++ 标准模板库中的部分内容，它是一个多功能的，能够操作多种数据结构和算法的模板类和函数库。vector 是向量类型，它可以容纳多种类型的数据，所以称之为容器。

首先在 `include` 目录下创建 `Vector.h`，在 `Detail` 目录下创建 `Vector.impl.h`,用于封装成员函数。便于理解，成员函数我们直接封装了。接口可以在 `Vector.h` 中查看。

#### vector 类基础

我们需要为 vector 定义四个私有成员:

- `*start_`:表示使用空间的头
- `*finnish_`:表示使用空间的末尾
- `*endofStorage_`:表示可用空间的末尾
- `dataAllocator`:表示分配空间

```c++
private:
    T *start_;
    T *finish_;
    T *endofStorage_;
    typedef Alloc dataAllocator;
```

同时我们还需要声明一些类型，以便于与系统类型进行区别。

```c++
public:
        typedef T                                    value_type;//值类型
        typedef T*                                    iterator;//迭代器
        typedef const T*                            const_iterator;//常量迭代器
        typedef reverse_iterator_t<T*>                reverse_iterator;//反向迭代器
        typedef reverse_iterator_t<const T*>                           const_reverse_iterator;
        typedef iterator                            pointer;//操作结果类型
        typedef T&                                    reference;//解引用操作结果类型
        typedef const T&                            const_reference;
        typedef size_t                                size_type;
        typedef ptrdiff_t                            difference_type;//表示两个迭代器之间的距离 ,c++内置定义 typedef int ptrdiff_t;
```

## 3 vector 初始化



和任何一种类类型一样，vector 模板控制着定义和初始化向量的方法。下面我们就讲讲 vector 的初始化。

#### 不带参数的构造函数初始化(默认值为 0)

```c++
vector():start_(0), finish_(0), endOfStorage_(0){}
```

#### 带参数的构造函数初始化

- 构造 n 个 0

```c++
template<class T, class Alloc>
vector<T, Alloc>::vector(const size_type n){
    allocateAndFillN(n, value_type());//调用成员函数，默认值为0
}
```

- 构造 n 个 value

```c++
template<class T, class Alloc>
vector<T, Alloc>::vector(const size_type n, const value_type& value){
    allocateAndFillN(n, value);//调用成员函数
}
```

- 把 fist 到 last 的所有元素放进 vector

```c++
template<class T, class Alloc>
template<class InputIterator>
vector<T, Alloc>::vector(InputIterator first, InputIterator last){
    //处理指针和数字间的区别的函数
    vector_aux(first, last, typename                 std::is_integral<InputIterator>::type());//需调用插入辅助函数
}

template<class T, class Alloc>
template<class Integer>
void vector<T, Alloc>::vector_aux(Integer n, const value_type& value, std::true_type){
    allocateAndFillN(n, value);//构造辅助函数
}
```

- 拷贝赋值构造一个 vector

```c++
template<class T, class Alloc>
vector<T, Alloc>::vector(const vector& v){
    allocateAndCopy(v.start_, v.finish_);//这里调用辅助成员函数完成
}
```

- 赋值操作符

```c++
template<class T, class Alloc>
    vector<T, Alloc>& vector<T, Alloc>::operator = (const vector& v){
    if (this != &v){
        allocateAndCopy(v.start_, v.finish_);
    }
    return *this;
}
```

- 析构函数

```c++
template<class T, class Alloc>
vector<T, Alloc>::~vector(){
    destroyAndDeallocateAll();
}
```

## 4 vector 常规操作（添加、删除、访问、插入）



#### 添加（添加到末尾）

vector 有自己的添加元素的方法，就是把元素添加到 vector 容器的末尾。这里我们命名为 push_back()函数。我们需要考虑到空间问题，如果空间已满的情况下，需要申请更多的空间。

```c++
template<class T, class Alloc>
void vector<T, Alloc>::push_back(const value_type& value){
    insert(end(), value);
}
```

#### 删除

删除需要根据具体情况来看，可以单纯只删除一个，也可以删除某指定元素或者某串元素，下面我们对这些问题具体分析。

- 尾部删除

尾部删除可以简单的执行删除操作。

```c++
template<class T, class Alloc>
void vector<T, Alloc>::pop_back(){
    --finish_;
    dataAllocator::destroy(finish_);//辅助成员函数
}
```

- 指定删除

简单的尾部删除肯定满足不了我们的编程需求，所以这里假设了指定位置删除。

(1)指定位置删除

```c++
template<class T, class Alloc>
typename vector<T, Alloc>::iterator vector<T, Alloc>::erase(iterator position){
    return erase(position, position + 1);
}
```

(2)指定范围删除

```c++
template<class T, class Alloc>
typename vector<T, Alloc>::iterator vector<T, Alloc>::erase(iterator first, iterator last){
    //尾部残留对象数
    difference_type lenOfTail = end() - last;
    //删去的对象数目
    difference_type lenOfRemoved = last - first;
    finish_ = finish_ - lenOfRemoved;
    for (; lenOfTail != 0; --lenOfTail){
        auto temp = (last - lenOfRemoved);
        *temp = *(last++);
    }
    return (first);
}
```

#### 访问

vector 的访问是比较简单的，只要掌握了数组的使用，就不难理解。

- 返回下标位置元素

```c++
reference operator[](const difference_type i){  //重载[],这样可以用a[n]进行访问元素
    return *(begin() + i);
}
```

- 返回倒数第 i 个元素

```c++
const_reference operator[](const difference_type i)const{
    return *(cbegin() + i);
}
```

- 返回第一个元素

```c++
reference front(){
    return *(begin());
}
```

- 返回最后一个元素

```c++
reference back(){
    return *(end() - 1);
}
```

- 返回第一个元素的指针

```c++
pointer data(){
    return start_;
}
```

#### 插入

单纯的尾部添加可能还需要我们进行排序之类的操作，这样就增加了不少的外部代码，所以我们有必要增加一个成员函数 insert，用于在指定位置插入元素。

- 单个插入

```c++
template<class T, class Alloc>
typename vector<T, Alloc>::iterator vector<T, Alloc>::insert(iterator position, const value_type& val){
    const auto index = position - begin();
    insert(position, 1, val);//插入辅助函数
    return begin() + index;
}

template<class T, class Alloc>
template<class Integer>
void vector<T, Alloc>::insert_aux(iterator position, Integer n, const value_type& value, std::true_type){
    assert(n != 0);//判断n是否为0
    difference_type locationLeft = endOfStorage_ - finish_; // 可使用空间大小
    difference_type locationNeed = n;//需要空间

    if (locationLeft >= locationNeed){//剩余空间大于需求空间
        auto tempPtr = end() - 1;
           for (; tempPtr - position >= 0; --tempPtr){//move the [position, finish_) back
            construct(tempPtr + locationNeed, *tempPtr);
        }
        mySTL::uninitialized_fill_n(position, n, value);
        finish_ += locationNeed;
    }
    else{
        reallocateAndFillN(position, n, value);
    }
}
```

- 指定范围插入

```c++
template<class T, class Alloc>
template<class InputIterator>
void vector<T, Alloc>::insert(iterator position, InputIterator first, InputIterator last){
    insert_aux(position, first, last, typename         std::is_integral<InputIterator>::type());//插入辅助函数
}

void vector<T, Alloc>::insert_aux(iterator position,
InputIterator first,InputIterator last,std::false_type){
    difference_type locationLeft = endOfStorage_ - finish_; // 计算剩余空间大小
    difference_type locationNeed = distance(first, last);//计算全部空间大小

    if (locationLeft >= locationNeed){//如果剩余空间满足需求，直接插入
        if (finish_ - position > locationNeed){
        mySTL::uninitialized_copy(finish_ - locationNeed, finish_, finish_);
        std::copy_backward(position, finish_ - locationNeed, finish_);
        std::copy(first, last, position);
        }
        else{//不满足，把指定位置后的元素先取出来，插入后在添加到末尾
            iterator temp = mySTL::uninitialized_copy(first + (finish_ - position), last, finish_);
            mySTL::uninitialized_copy(position, finish_, temp);
            std::copy(first, first + (finish_ - position), position);
               }
            finish_ += locationNeed;
        }
    else{
        reallocateAndCopy(position, first, last);
    }
}
```

## 5 vector 增长模型



我们可以把 vector 理解为动态数组，作为一个动态数组，vector 有一个指针指向一片连续的内存空间，但是这片空间不是无限的，当内存装不下数据时，系统会自动申请一片更大的空间把原来的数据拷贝过去，释放原来的内存空间。 **vector 的内存非常重要，一旦内存重新配置，与之相关的所有指针、迭代器都会失效，而且配置内存非常耗时。**

## 6 vector 成员函数



vector 是非常方便的，这离不开它的成员函数，vector 的许多操作都是通过成员函数完成的，包括添加删除。

### 6-1 迭代器相关



- 返回头部迭代器

```c++
iterator begin(){ return (start_); }
const_iterator begin()const{ return (start_); }//常量迭代器，只读属性
const_iterator cbegin()const{ return (start_); }
```

- 返回尾部迭代器

```c++
iterator end(){ return (finish_); }
const_iterator end()const{ return (finish_); }
const_iterator cend()const{ return (finish_); }
```

- 逆向，返回头部迭代器

```c++
reverse_iterator rend(){ return reverse_iterator(start_); }
const_reverse_iterator crend()const{ return const_reverse_iterator(start_); }
```

- 逆向，返回尾部

```c++
reverse_iterator rbegin(){ return reverse_iterator(finish_); }
const_reverse_iterator crbegin()const{ return const_reverse_iterator(finish_);
```

### 6-2 访问元素相关



- 顺序访问

```c++
reference front(){ return *(begin()); } //返回头部迭代器
reference back(){ return *(end() - 1); }//返回最后一个元素迭代器
pointer data(){ return start_; }//返回头部指针
```

- 定向访问

```c++
reference operator[](const difference_type i){ return *(begin() + i); }//返回第 i 个元素的迭代器
const_reference operator[](const difference_type i)const{ return *(cbegin() + i); }//返回第 i 个元素的迭代器，只读
```

### 6-3 容量相关



- 返回元素个数

```c++
difference_type size() const{
    return finish_ - start_;
}
```

- 返回容量大小

```c++
difference_type capacity() const{
    return endOfStorage_ - start_;
}
```

- 判断容器是否为空

```c++
bool empty() const{
    return start_ == finish_;
}
```

- 重新定义容器大小

```c++
template<class T, class Alloc>
void vector<T, Alloc>::resize(size_type n, value_type val){
    if (n < size()){//如果重新定义空间小于元素个数，需要释放多余空间
        dataAllocator::destroy(start_ + n, finish_);
        finish_ = start_ + n;
    }
    else if (n > size() && n <= capacity()){//如果重新定义空间大于元素个数而小于本来空间，需要释放多余空间
        auto lengthOfInsert = n - size();
        finish_ = mySTL::uninitialized_fill_n(finish_, lengthOfInsert, val);
    }
    else if (n > capacity()){
        auto lengthOfInsert = n - size();
        T *newStart =             dataAllocator::allocate(getNewCapacity(lengthOfInsert));
        T *newFinish = mySTL::uninitialized_copy(begin(), end(), newStart);
        newFinish = mySTL::uninitialized_fill_n(newFinish, lengthOfInsert, val);

    destroyAndDeallocateAll();//初始化
    start_ = newStart;
    finish_ = newFinish;
    endOfStorage_ = start_ + n;
    }
}
```

- 重新分配存储区大小

```c++
template<class T, class Alloc>
void vector<T, Alloc>::reserve(size_type n){//首先释放空间，然后初始化
    if (n <= capacity())
        return;
    T *newStart = dataAllocator::allocate(n);
    T *newFinish = mySTL::uninitialized_copy(begin(), end(), newStart);
    destroyAndDeallocateAll();

    start_ = newStart;
    finish_ = newFinish;
    endOfStorage_ = start_ + n;
}
```

- 删除所有元素

```c++
void clear()
{
    erase(begin(), end());
}
```

- 释放内存

```c++
template<class T, class Alloc>
void vector<T, Alloc>::clear(){
    dataAllocator::destroy(start_, finish_);
    finish_ = start_;
}
```

### 6-4 辅助函数



- 赋值,用于构造插入元素等

```c++
template<class T, class Alloc>
template<class InputIterator>
void vector<T, Alloc>::reallocateAndCopy(iterator position, InputIterator first, InputIterator last){
    difference_type newCapacity = getNewCapacity(last - first);//需要申请空间大小

    T *newStart = dataAllocator::allocate(newCapacity);//申请空间
    T *newEndOfStorage = newStart + newCapacity;
    T *newFinish = mySTL::uninitialized_copy(begin(), position, newStart);//把begin到position赋值给新地址的头
    newFinish = mySTL::uninitialized_copy(first, last, newFinish);//插入需要插入的值到新地址的尾
    newFinish = mySTL::uninitialized_copy(position, end(), newFinish);//剩下的元素拷贝过来

    destroyAndDeallocateAll();//释放空间
    start_ = newStart;//重新定义
    finish_ = newFinish;
    endOfStorage_ = newEndOfStorage;
}
```

### 6-5 vector 使用算法



vector 的编写使用了 `#include "Algoritm.h"` 和 `#include "Allocator.h"` 中的泛函算法，具体包括:

- 分配空间

```c++
pointer allocate (size_type n, allocator<void>::const_pointer hint=0);
```

- 填充元素

```c++
void construct(T* p,const T* val);插入元素
```

- 释放空间

```c++
void deallocate (pointer p, size_type n);释放空间
```

- 填充元素到目的区间

```c++
void uninitialized_fill(ForwardIt first, ForwardIt last, const T& value);
```

- 复制填充元素到目的区间

```c++
uninitialized_copy( InputIterator first, InputIterator last, ForwardIterator result);
```

- 在指定位置插入 count 个 元素

```c++
void uninitialized_fill_n( ForwardIt first, Size count, const T& value );
```

## 7 vector 实例测试



在上面我们已经写了许多成员函数，接下来我们运用自己写的 vector 编写一个程序来测试一下效果，这里我们需要在 `Test` 目录下创建 `vectortest.cpp`。

```c++
#include "Vector.h"
#include <iostream>
#include "Algorithm.h"

int summing(mySTL::vector<int> val)
{
    int sum = 0;
    mySTL::vector<int>::iterator ix = val.begin();
    for (;ix != val.end(); ix++)
        sum += *ix;//求和
    return sum;
}

void print(mySTL::vector<int> val){
    mySTL::vector<int>::iterator ix = val.begin();
    for (;ix != val.end(); ix++)
     std::cout<<*ix<<" ";
    std::cout<<std::endl;
}

int main(int argv,char *argc[])
{
    int sum;
    int input[5];
    std::cout<<"enter 5 numbers"<<std::endl;
    for(int i = 0;i < 5;i++)
    {
        std::cin>>input[i];//输入5个参数
    }
    mySTL::vector<int> val(input,input + 5);

    if(val.size() == 0)//判断a是否为空
    {
        std::cout<<"no element?"<<std::endl;
        return -1;
    }

    std::cout<<"The vector of val:"<<std::endl;
    print(val);

    std::cout<<"After insert:"<<std::endl;
    mySTL::vector<int>::iterator it = val.begin();
    mySTL::advance(it,2);
    val.insert(it,2,3);
    print(val);

    std::cout<<"The size of val:"<<std::endl;
    std::cout<<val.size()<<std::endl;

    std::cout<<"After pushback:"<<std::endl;
    val.push_back(11);
    print(val);

    std::cout<<"After erase:"<<std::endl;
    val.erase(it);
    print(val);

    std::cout<<"After popback:"<<std::endl;
    val.pop_back();
    print(val);

    sum = summing(val);
    std::cout<<"The sum of val = "<<sum<<std::endl;


    std::cout<<"After sort:"<<std::endl;
    mySTL::vector<int>::iterator beg = val.begin();
    mySTL::vector<int>::iterator end = val.end();
    mySTL::sort(beg,end);
    print(val);
    return 0;
}
```

在命令行中执行如下代码：

```bash
g++ vectortest.cpp -std=c++11 -o vectortest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539852127549.png)

## 8 实验总结



通过以上的学习，我们了解到 vector 的构造和成员函数的编写，也间接领悟到了 STL 的便捷与强大，通过对 vector 编写我们可以更加深刻的理解 vector 的用法，在接下来的实验中，我们还会掌握更多的知识。

# 七 基础容器之 list



## 1 实验内容



本次实验主要讲解 list 的原理，怎么通过代码创建属于自己的 list，list 是 c++11 STL 中一个非常重要的成员，通过使用它，我们可以高效地进行插入删除元素。

#### 知识点

- list 介绍
- list 构造
- list 成员函数
- list 迭代器
- list 基本操作

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件 `List.h`，成员函数封装在 `Detail` 目录下 `List.impl.h` 中。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip

# 解压文件到Code目录
unzip -q mySTL.zip -d ./Code/
```

本次实验需要用到的头文件包括： 可以在 `mySTL` 目录下查看

```c++
#include "Allocator.h"                    //空间分配器，已经写好直接调用
#include "Iterator.h"                    //迭代器
#include "ReverseIterator.h"            //逆向迭代器，重载了一些运算符，可以直接使用
#include "UninitializedFunctions.h"        //初始化函数库，直接使用，用于构造或者赋值
#include "TypeTraits.h"                    //类型库，抽象定义了一些类型
```

## 2 list 的介绍和性质



list 其实就是一个双向链表，在编程语言中 List 是 c++ 标准模板库中的一部分内容，可以简单视之为双向连结串行，以线性列的方式管理物件集合。list 的特色是在集合的任何位置增加或删除元素都很快，但是不支持随机存取。list 是类库提供的众多容器（container）之一。list 以模板方式实现（即泛型），可以处理任意型别的变量，包括使用者自定义的资料型态。

## 3 list 的定义和赋值



首先在 `include` 目录下创建 `List.h` ，在 `Detail` 目录下创建 `List.impl.h` 用于封装。

### 3-1 封装对象，形成链表节点



list 的本质是链表，所以我们要赋予 list 链表的属性。在这里先定义一个结构体。

```c++
template<class T>
struct node{
    T data;
    node *prev;
    node *next;
    list<T> *container;
    node(const T& d, node *p, node *n, list<T> *c):
    data(d), prev(p), next(n), container(c){}//初始化节点
    bool operator ==(const node& n){//比较重载
    return data == n.data && prev == n.prev && next == n.next && container == n.container;
    }
};
```

### 3-2 定义



我们需要定义一些 list 的成员来表示 list 的节点以及头和尾。

```c++
typedef allocator<Detail::node<T>> nodeAllocator;//空间分配器
typedef Detail::node<T> *nodePtr;//节点
private:
    iterator head;//定义头和尾
    iterator tail;
```

同时我们还需要声明一些类型，以便于与系统类型进行区别。

```c++
public:
    typedef T value_type;//值类型
    typedef Detail::listIterator<T> iterator;//迭代器
    typedef Detail::listIterator<const T> const_iterator;//常量迭代器
    typedef reverse_iterator_t<iterator> reverse_iterator;//逆向迭代器
    typedef T& reference;//解引用操作结果类型
    typedef size_t size_type;
```

### 3-3 初始化



初始化需要用到很多重复的代码，所以我们写了一个构造辅助函数 ctorAux 用于辅助。

- 无参构造

```c++
template<class T>
list<T>::list(){
    head.p = newNode();//添加一个节点
    tail.p = head.p;
}
```

- 带参构造

list 有 n 个值的结点

```c++
template<class T>
list<T>::list(size_type n, const value_type& val){
    ctorAux(n, val, std::is_integral<value_type>());//定义一个初始化辅助函数
}
```

用 [first,last] 区间构造 list

```c++
template<class T>
template <class InputIterator>
list<T>::list(InputIterator first, InputIterator last){
    ctorAux(first, last, std::is_integral<InputIterator>());
}
```

- 拷贝复制构造

```c++
template<class T>
list<T>::list(const list& l){
    head.p = newNode();//添加一个节点
    tail.p = head.p;
    for (auto node = l.head.p; node != l.tail.p; node = node->next)
        push_back(node->data);
    }
```

- 析构函数

```c++
template<class T>
list<T>::~list(){
    for (; head != tail;){
        auto temp = head++;
        nodeAllocator::destroy(temp.p);//释放节点，调用成员函数
        nodeAllocator::deallocate(temp.p);//释放空间
    }
    nodeAllocator::deallocate(tail.p);
}
```

## 4 迭代器



所谓的迭代器就是赋予容器指针一样的功能，但是又比指针智能，接下来我们就实现这个功能。

- 构建一个迭代器，可以理解为一个结构或者一个类，迭代器拥有自己的成员以及重载了一些操作符。

```c++
template<class T>
struct listIterator :public iterator<bidirectional_iterator_tag, T>{
    template<class ELEM>//这里不能使用class T，会报重复声明
    friend class list;
    public:
    typedef node<T>* nodePtr;//节点指针
    nodePtr p;
    public:
    explicit listIterator(nodePtr ptr = nullptr) :p(ptr){}//初始化迭代器

    //重载相关
    listIterator& operator++(){//自增
        p = p->next;
        return *this;}
    listIterator operator++(int){//左值自增
        auto res = *this;
        ++*this;
        return res;
    }
    listIterator& operator --(){//自减
        p = p->prev;
        return *this;
    }
    listIterator operator --(int){//左值自减
        auto res = *this;
        --*this;
        return res;
    }
    T& operator *(){ return p->data; }//返回迭代器指向的值
    T* operator ->(){ return &(operator*()); }//指向

    template<class ELEM>
    friend bool operator ==(const listIterator<ELEM>& lhs, const listIterator<ELEM>& rhs){//比较是否相等
        return lhs.p == rhs.p;
    }
    template<class ELEM>
    friend bool operator !=(const listIterator<ELEM>& lhs, const listIterator<ELEM>& rhs){
        return !(lhs == rhs);
    }
};
```

## 5 基础成员函数



接下来，我们给大家介绍相关基础成员函数，主要是迭代器相关和容器相关以及辅助函数。

### 5-1 迭代器相关



- 返回头部

```c++
iterator begin(){ return head; }

const_iterator begin()const{
    auto temp = (list*const)this;
    return changeIteratorToConstIterator(temp->head);
}//常量迭代器，只读属性

const_iterator rbegin()const{ return reverse_iterator(tail); }
```

- 返回尾部

```c++
iterator end(){ return tail; }

const_iterator end()const{
    auto temp = (list*const)this;
    return changeIteratorToConstIterator(temp->tail);
}

const_iterator rend()const{ return reverse_iterator(head); }
```

### 5-2 容器相关



- 返回元素个数

```c++
template<class T>
typename list<T>::size_type list<T>::size()const{
    size_type length = 0;
    for (auto h = head; h != tail; ++h)
        ++length;
    return length;
}
```

- 链接,用于拼接两个 list

```c++
template<class T>
void list<T>::splice(iterator position, list& x){//在位置position链接list x
    this->insert(position, x.begin(), x.end());
    x.head.p = x.tail.p;
}

template<class T>
void list<T>::splice(iterator position, list& x, iterator first, iterator last){//在位置position 链接 list x的first 到 last区间元素
    if (first.p == last.p) return;
    auto tailNode = last.p->prev;
    if (x.head.p == first.p){//判断是否从头开始
        x.head.p = last.p;
        x.head.p->prev = nullptr;
    }
    else{//链接
        first.p->prev->next = last.p;
        last.p->prev = first.p->prev;
    }
    if (position.p == head.p){//在头部链接
        first.p->prev = nullptr;
        tailNode->next = head.p;
        head.p->prev = tailNode;
        head.p = first.p;
    }
    else{//在position位置插入
        position.p->prev->next = first.p;
        first.p->prev = position.p->prev;
        tailNode->next = position.p;
        position.p->prev = tailNode;
    }
}
```

- 清空容器

```c++
template<class T>
void list<T>::clear(){
    erase(begin(), end());//成员函数删除
}
```

### 5-3 辅助函数



- 删除节点,释放节点空间

```c++
template<class T>
void list<T>::deleteNode(nodePtr p){
    p->prev = p->next = nullptr;
    nodeAllocator::destroy(p);
    nodeAllocator::deallocate(p);
}
```

- 构造辅助函数

```c++
template<class T>
void list<T>::ctorAux(size_type n, const value_type& val, std::true_type){
    head.p = newNode();//添加一个节点
    tail.p = head.p;
    while (n--)//当 n 不为 0，放入元素
        push_back(val);
}

template<class T>
template <class InputIterator>
void list<T>::ctorAux(InputIterator first, InputIterator last, std::false_type){
    head.p = newNode();//添加一个节点
    tail.p = head.p;
    for (; first != last; ++first)//插入 first 到 last 元素
        push_back(*first);
}
```

## 6 list 基础操作



list 的基本操作包括插入，删除和遍历等，这些在实际运用中是经常用到的。

### 6-1 插入



- 头插

```c++
template<class T>
void list<T>::push_front(const value_type& val){
    auto node = newNode(val);
    head.p->prev = node;
    node->next = head.p;
    head.p = node;
}
```

- 尾插

```c++
template<class T>
void list<T>::push_back(const value_type& val){
    auto node = newNode(val);//新节点添加到尾
    (tail.p)->data = val;
    (tail.p)->next = node;
    node->prev = tail.p;
    tail.p = node;
}
```

- 指定位置插入一个元素

```c++
template<class T>
typename list<T>::iterator list<T>::insert(iterator position, const value_type& val){
    if (position == begin()){//是否在头插入
        push_front(val);
        return begin();
    }
    else if (position == end()){//是否在尾插入
        auto ret = position;
        push_back(val);
        return ret;
    }
    auto node = newNode(val);//插入节点
    auto prev = position.p->prev;
    node->next = position.p;
    node->prev = prev;
    prev->next = node;
    position.p->prev = node;
    return iterator(node);
}
```

- 指定位置插入 n 个元素

```c++
template<class T>
void list<T>::insert(iterator position, size_type n, const value_type& val){
    insert_aux(position, n, val, typename     std::is_integral<T>::type());//调用插入辅助函数
}

template<class T>
void list<T>::insert_aux(iterator position, size_type n, const T& val, std::true_type){
    for (auto i = n; i != 0; --i){
           position = insert(position, val);
    }
}
```

- 指定范围插入

```c++
template<class T>
template <class InputIterator>
void list<T>::insert(iterator position, InputIterator first, InputIterator last){
    insert_aux(position, first, last, typename std::is_integral<InputIterator>::type());//调用插入辅助函数
}

template<class T>
template<class InputIterator>
void list<T>::insert_aux(iterator position, InputIterator first, InputIterator last, std::false_type){
    for (--last; first != last; --last){//逐个插入
        position = insert(position, *last);
    }
    insert(position, *last);
}
```

### 6-2 删除



- 头删

```c++
template<class T>
void list<T>::pop_front(){
    auto oldNode = head.p;//定义一个替代节点
    head.p = oldNode->next;
    head.p->prev = nullptr;
    deleteNode(oldNode);//释放节点
}
```

- 尾删

```c++
template<class T>
void list<T>::pop_back(){
    auto newTail = tail.p->prev;
    newTail->next = nullptr;
    deleteNode(tail.p);
    tail.p = newTail;
}
```

- 指定位置删除

```c++
template<class T>
typename list<T>::iterator list<T>::erase(iterator position){
    if (position == head){//判断是否需要头删
        pop_front();
        return head;
    }
    else{
        auto prev = position.p->prev;
        prev->next = position.p->next;
        position.p->next->prev = prev;
        deleteNode(position.p);
        return iterator(prev->next);
    }
}
```

- 指定范围删除

```c++
template<class T>
typename list<T>::iterator list<T>::erase(iterator first, iterator last){
    typename list<T>::iterator res;
    for (; first != last;){
        auto temp = first++;
        res = erase(temp);//逐个删除
    }
    return res;
}
```

### 6-3 访问



- 返回头部元素

```c++
reference front(){ return (head.p->data); }
```

- 返回尾部元素

```c++
reference back(){ return (tail.p->prev->data); }
```

## 7 list 的重载运算符



重载操作符是具有特殊名称的函数：保留了 operator 后接需定义的操作符符号。接下来我们实现几个运算符重载，重载运算符可以简化一些操作。

- 比较运算符重载

```c++
template <class T>
bool operator== (const list<T>& lhs, const list<T>& rhs){//判断两个list是否相等
    auto node1 = lhs.head.p, node2 = rhs.head.p;
    for (; node1 != lhs.tail.p && node2 != rhs.tail.p; node1 = node1->next, node2 = node2->next){
        if (node1->data != node2->data)
        break;
    }
    if (node1 == lhs.tail.p && node2 == rhs.tail.p)
        return true;
    return false;
}

template <class T>
bool operator!= (const list<T>& lhs, const list<T>& rhs){
    return !(lhs == rhs);//返回不等
}
```

## 8 list 实例测试



在上面我们已经写了许多成员函数，接下来我们运用自己写的 list 编写一个程序来测试一下效果，这里我们需要在 `Test` 目录下创建 `listtest.cpp`。

```C++
#include <iostream>
#include "List.h"

void print(mySTL::list<int> temp){
    mySTL::list<int>::iterator it = temp.begin();
    for(; it != temp.end();it++)
        std::cout<<*it<<" ";
    std::cout<<std::endl;
}

int main(){
    mySTL::list<int> val;
    val.push_back(1);
    val.push_back(4);
    val.push_back(7);
    std::cout<<"After pushback:"<<std::endl;
    print(val);

    val.push_front(8);
    val.push_front(5);
    val.push_front(2);
    std::cout<<"After pushfront:"<<std::endl;
    print(val);

    std::cout<<"The size of val: "<<val.size()<<std::endl;

    std::cout<<"After copy:"<<std::endl;
    mySTL::list<int> val1(val);
    print(val1);

    std::cout<<"After popback:"<<std::endl;
    val.pop_back();
    print(val);

    std::cout<<"After popfront:"<<std::endl;
    val.pop_front();
    print(val);

    mySTL::list<int>::iterator ix = val.begin();
    std::cout<<"After erase position 2:"<<std::endl;
    advance(ix,1);
    val.erase(ix);
    print(val);

    ix = val.begin();
    std::cout<<"After insert to position 2:"<<std::endl;
    advance(ix,1);
    val.insert(ix,6);
    print(val);

    std::cout<<"After splice val and val1:"<<std::endl;
    val.splice(val.begin(),val1);
    print(val);

    std::cout<<"After sort:"<<std::endl;
    val.sort();
    print(val);
}
```

在命令行中执行如下代码：

```bash
g++ listtest.cpp -std=c++11 -o listtest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539852050709.png)

## 9 实验总结

在以上的学习过程中，我们已经了解到了 list 的定义（一个双向链表）还有 list 的基本操作，以及 list 的成员函数。通过几次实验，我们不难发现 STL 里面许多构造函数和成员函数非常相似，只要一个一个理解了这些成员，就会达到触类旁通的效果。

# 八 基础容器之 deque



## 1 实验内容

本次实验的主要内容是 deque，deque 又名双端队列，是一种具有队列和栈的性质的数据结构。双端队列中的元素可以从两端弹出，其限定插入和删除操作在表的两端进行。

#### 知识点

- deque 性质
- deque 定义和初始化
- deque 基本操作
- deque 迭代器

#### 实验环境

- g++
- linux 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件`Deque.h`。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip

# 解压文件到Code目录
unzip -q mySTL.zip -d ./Code/
```

本次实验需要用到的头文件包括： 可以在 `mySTL` 目录下查看

```c++
#include "Allocator.h"            //空间分配器，已经写好直接调用
#include "Iterator.h"            //迭代器
#include "Algorithm.h"            //算法库
#include "ReverseIterator.h"    //逆向迭代器，重载了一些运算符，可以直接使用
```

## 2 deque 的介绍和性质



deque 即双端队列，全名：double-ended queue，是一种具有队列和栈的性质的数据结构。双端队列中的元素可以从两端弹出，其限定插入和删除操作在表的两端进行。 在实际使用中，还可以有输出受限的双端队列（即一个端点允许插入和删除，另一个端点只允许插入的双端队列）和输入受限的双端队列（即一个端点允许插入和删除，另一个端点只允许删除的双端队列）。而如果限定双端队列从某个端点插入的元素只能从该端点删除，则该双端队列就蜕变为两个栈底相邻的栈了。（**百度百科**）

（1）优缺点：尽管双端队列看起来似乎比栈和队列更灵活，但实际上在应用程序中远不及栈和队列有用。

（2）和 vector 相比

- 相同点：
  - 在中部插入和删除比较缓慢。
  - 迭代器属于随机存取迭代器。
- 不同：
  - deque 两端都可以快速的插入和删除。
  - deque 元素的存取和迭代器操作较慢。
  - deque 的内存是可以自动缩减的，当内存空间不再使用时会自动释放。
  - deque 迭代器需要在不同区块间跳转，所以它非一般指针。
  - deque 除了头尾两端，在任何地方安插或删除元素，都将导致指向 deque 元素的所有迭代器失效。
  - deque 使用不只一块内存，vector 使用一块连续内存。

（3）c++标准建议：vector 是应该在默认情况下使用的序列。如果大多数插入和删除操作发生在序列的头部或尾部时，应该选用 deque。

### 2-1 deque 的定义和赋值



接下来，我们主要对 deque 的定义和赋值进行说明。

### 2-2 定义



我们需要给 deque 定义一些参数与方法名称，方便调用,名称最好是便于自己理解的。

- 定义队列的成员参数

```c++
private:
    iterator beg_, end_;//头和尾
    size_t mapSize_;//空间
    T **map_;//map是一个连续的空间, 其每个元素都是一个指向缓冲区的指针
private:
    typedef Alloc dataAllocator;//空间分配器
    enum class EBucksSize{BUCKSIZE = 64};//设置默认容器尺寸

public:
    typedef T value_type;//参数类型
    typedef Detail::dq_iter<T> iterator;//迭代器
    typedef Detail::dq_iter<const T> const_iterator;//常量迭代器
    typedef T& reference;//解引用操作类型
    typedef const reference const_reference;
    typedef size_t size_type;
    typedef ptrdiff_t difference_type;
    typedef Alloc allocator_type;//空间分配器
```

### 2-3  构造



- 无参构造

```c++
template<class T, class Alloc>
deque<T, Alloc>::deque():mapSize_(0), map_(0){}
```

- 带参构造

构造 n 个 val

```c++
template<class T, class Alloc>
deque<T, Alloc>::deque(size_type n, const value_type& val){
    deque();
    deque_aux(n, val, typename std::is_integral<size_type>::type());//构造辅助函数
}
```

构造范围[first,last]的值

```c++
template<class T, class Alloc>
template <class InputIterator>
deque<T, Alloc>::deque(InputIterator first, InputIterator last){
    deque();
    deque_aux(first, last, typename std::is_integral<InputIterator>::type());//构造辅助函数
}
```

## 3 deque 迭代器



迭代器是容器的最关键的部分，因此迭代器的设计也是比较复杂的，所以我们一步一步的讲。 定义一个迭代器类，我们需要在 `include` 目录下创建 `Deque.ipml.h` 文件，用于封装成员。

- 定义成员变量

```c++
private:
    template<class ELEM, class alloc>
    friend class ::mySTL::deque;//友元调用deque
private:
    typedef const ::mySTL::deque<T>* cntrPtr;
    size_t mapIndex_;
    T *cur_;
    cntrPtr container_;
```

- 构造相关

```c++
public:
    dq_iter() :mapIndex_(-1), cur_(0), container_(0){}//无参构造

    dq_iter(size_t index, T *ptr, cntrPtr container)//传参构造
        :mapIndex_(index), cur_(ptr), container_(container){}

    dq_iter(const dq_iter& it) //拷贝构造
        :mapIndex_(it.mapIndex_), cur_(it.cur_), container_(it.container_){}
```

- 重载相关

(1)赋值重载

```c++
dq_iter& operator = (const dq_iter& it){
    if (this != &it){//如果不等，则赋值
        mapIndex_ = it.mapIndex_;
        cur_ = it.cur_;
        container_ = it.container_;
    }
    return *this;
}
```

(2)操作符重载

```c++
reference operator *(){ return *cur_; }//返回值

const reference operator *()const{ return *cur_; }

pointer operator ->(){ return &(operator*()); }//返回地址

const pointer operator ->()const{ return &(operator*()); }
```

(3)自加重载

```c++
template<class T>
dq_iter<T>& dq_iter<T>::operator ++(){
    if (cur_ != getBuckTail(mapIndex_))//+1后还在同一个桶里
            ++cur_;
    else if (mapIndex_ + 1 < container_->mapSize_){//+1后还在同一个map里
            ++mapIndex_;
            cur_ = getBuckHead(mapIndex_);
        }
    else{//+1后跳出了map
            mapIndex_ = container_->mapSize_;
            //cur_ = container_->map_[mapIndex_] + getBuckSize();//指向map_[mapSize_-1]的尾的下一个位置
            cur_ = container_->map_[mapIndex_];
        }
        return *this;
    }

template<class T>
dq_iter<T> dq_iter<T>::operator ++(int){//左值自加
    auto res = *this;
    ++(*this);
    return res;
}
```

(4)自减重载

```c++
template<class T>
dq_iter<T>& dq_iter<T>::operator --(){
    if (cur_ != getBuckHead(mapIndex_))//当前不指向桶头
            --cur_;
    else if (mapIndex_ - 1 >= 0){//-1后还在map里面
            --mapIndex_;
            cur_ = getBuckTail(mapIndex_);
    }
    else{
            mapIndex_ = 0;
            cur_ = container_->map_[mapIndex_];//指向map_[0]的头
    }
    return *this;
}

template<class T>
dq_iter<T> dq_iter<T>::operator --(int){//左值自减
    auto res = *this;
    --(*this);
    return res;
}
```

(5)比较相关重载

```c++
template<class T>
bool dq_iter<T>::operator ==(const dq_iter& it)const{//重载
    return ((mapIndex_ == it.mapIndex_) &&
        (cur_ == it.cur_) && (container_ == it.container_));
}

template<class T>
bool dq_iter<T>::operator !=(const dq_iter<T>& it)const{
    return !(*this == it);
}
```

(6)成员函数相关

```c++
private:
    T *getBuckTail(size_t mapIndex)const{//获取空间的尾部元素
        return container_->map_[mapIndex] + (container_->getBuckSize() - 1);
    }

    T *getBuckHead(size_t mapIndex)const{//获取空间头部元素
        return container_->map_[mapIndex];
    }
    size_t getBuckSize()const;
```

(7)友元函数重载

"+"重载，前进 n 位

```c++
public:
    template<class Elem>
    friend dq_iter<Elem> operator + (const dq_iter<Elem>& it, typename dq_iter<Elem>::difference_type n){// n >= 0
        dq_iter<T> res(it);
        auto m = res.getBuckTail(res.mapIndex_) - res.cur_;
        if (n <= m){//前进n步仍在同一个桶中
            res.cur_ += n;
        }
        else{
            n = n - m;
            res.mapIndex_ += (n / it.getBuckSize() + 1);
            auto p = res.getBuckHead(res.mapIndex_);
            auto x = n % it.getBuckSize() - 1;
            res.cur_ = p + x;
        }
        return res;
    }

    template<class Elem>
    friend dq_iter<Elem> operator + (typename dq_iter<Elem>::difference_type n, const dq_iter<T>& it){
        return (it + n);
    }
```

"-"重载，用于后退 n 位

```c++
template<class Elem>
friend dq_iter<Elem> operator - (const dq_iter<Elem>& it, typename dq_iter<Elem>::difference_type n){//n >= 0
    dq_iter<T> res(it);
    auto m = res.cur_ - res.getBuckHead(res.mapIndex_);
    if (n <= m)//后退n步还在同一个桶中
        res.cur_ -= n;
    else{
        n = n - m;
        res.mapIndex_ -= (n / res.getBuckSize() + 1);
        res.cur_ = res.getBuckTail(res.mapIndex_) - (n % res.getBuckSize() - 1);
    }
    return res;
}

template<class Elem>
friend dq_iter<Elem> operator - (typename dq_iter<Elem>::difference_type n, const dq_iter<T>& it){
    return (it - n);
}
```

- 计算迭代器间距离

```c++
template<class Elem>
friend typename dq_iter<Elem>::difference_type operator - (const dq_iter<Elem>& it1, const dq_iter<T>& it2){
    if (it1.container_ == it2.container_ && it1.container_ == 0)
        return 0;
    return typename dq_iter<T>::difference_type(it1.getBuckSize()) * (it1.mapIndex_ - it2.mapIndex_ - 1)
            + (it1.cur_ - it1.getBuckHead(it1.mapIndex_)) + (it2.getBuckTail(it2.mapIndex_) - it2.cur_) + 1;
}
```

## 4 deque 成员函数



成员函数是容器的特色，拥有自己的成员函数才能提高编程的效率，下面我们添加一些成员函数。

- 返回首部指针

```c++
iterator begin(){ return beg_; }
iterator begin()const{ return beg_; }//返回值不可修改
```

- 返回尾部指针

```c++
iterator end(){ return end_; }
iterator end()const{ return end_; }//返回值不可修改
```

- 返回元素个数

```c++
size_type size() const{ return end() - begin(); }
```

- 判断是否为空

```c++
bool empty() const{ return begin() == end(); }
```

- 返回首部元素

```c++
reference back(){
        return *begin();
    }
```

- 返回尾部元素

```c++
reference back(){
        return *(end() - 1);
    }
```

- 清空队列

```c++
template<class T, class Alloc>
void deque<T, Alloc>::clear(){
    for (auto i = 0; i != mapSize_; ++i){
        for (auto p = map_[i] + 0; !p && p != map_[i] + getBuckSize(); ++p)
            dataAllocator::destroy(p);//释放空间
    }
    mapSize_ = 0;//初始化
    beg_.mapIndex_ = end_.mapIndex_ = mapSize_ / 2;
    beg_.cur_ = end_.cur_ = map_[mapSize_ / 2];
}
```

- 交换队列元素

```c++
template<class T, class Alloc>
void deque<T, Alloc>::swap(deque<T, Alloc>& x){
    mySTL::swap(mapSize_, x.mapSize_);
    mySTL::swap(map_, x.map_);
    beg_.swap(x.beg_);
    end_.swap(x.end_);
}
```

- 辅助函数，用于辅助构造或者成员函数

(1)构造辅助函数

```c++
template<class T, class Alloc>
void deque<T, Alloc>::deque_aux(size_t n, const value_type& val, std::true_type){//插入n个val
    int i = 0;
    for (; i != n / 2; ++i)
        (*this).push_front(val);
    for (; i != n; ++i)
           (*this).push_back(val);
}

template<class T, class Alloc>
template<class Iterator>
void deque<T, Alloc>::deque_aux(Iterator first, Iterator last, std::false_type){//插入区间[first,last]的值
    difference_type mid = (last - first) / 2;
    for (auto it = first + mid; it != first - 1; --it)
        (*this).push_front(*it);
    for (auto it = first + mid + 1; it != last; ++it)
        (*this).push_back(*it);
}
```

(2)分配空间

```c++
template<class T, class Alloc>
T *deque<T, Alloc>::getANewBuck(){
    return dataAllocator::allocate(getBuckSize());
}

template<class T, class Alloc>
T** deque<T, Alloc>::getANewMap(const size_t size){
    T **map = new T*[size];
    for (int i = 0; i != size; ++i)
        map[i] = getANewBuck();
    return map;
}
```

(3)申请空间和赋值

```c++
template<class T, class Alloc>
void deque<T, Alloc>::reallocateAndCopy(){
    auto newMapSize = getNewMapSize(mapSize_);
    T** newMap = getANewMap(newMapSize);//获取空间
    size_t startIndex = newMapSize / 4;//增加1/4的空间
    for (int i = 0; i + beg_.mapIndex_ != mapSize_; ++i)
        for (int j = 0; j != getBuckSize(); ++j)//把map 转移到 newmap
               newMap[startIndex + i][j] = map_[beg_.mapIndex_ + i][j];

    size_t n = beg_.cur_ - map_[beg_.mapIndex_];
    auto size = this->size();
    auto b = beg_, e = end_;
    clear();//释放map
    mapSize_ = newMapSize;//newmap 赋值给 map
    map_ = newMap;
    beg_ = iterator(startIndex, newMap[startIndex] + n, this);
    end_ = beg_ + size;

}
```

## 5 deque 基本操作



下面主要实现 deque 的一些常规的基本操作。

### 5-1 插入



- 添加到尾部

```c++
template<class T, class Alloc>
void deque<T, Alloc>::push_back(const value_type& val){
    if (empty()){//如果空，初始化
        init();
    }
    else if (back_full()){//如果满了，申请空间
           reallocateAndCopy();
    }
    mySTL::construct(end_.cur_, val);
    ++end_;
}
```

- 添加到头部

```c++
template<class T, class Alloc>
void deque<T, Alloc>::push_front(const value_type& val){
    if (empty()){
        init();
    }
    else if (front_full()){
        reallocateAndCopy();
    }
    --beg_;
    construct(beg_.cur_, val);
}
```

### 5-2 删除



- 删除尾部

```c++
template<class T, class Alloc>
void deque<T, Alloc>::pop_front(){
    --end_;//尾部前移以为
    dataAllocator::destroy(end_.cur_);//释放末尾空间
}
```

- 删除头部

```c++
template<class T, class Alloc>
void deque<T, Alloc>::pop_back(){
    dataAllocator::destroy(beg_.cur_);
    ++beg_;
}
```

### 5-3 遍历



- 返回头部元素

```c++
template<class T, class Alloc>
typename deque<T, Alloc>::iterator deque<T, Alloc>::begin(){ return beg_; }

template<class T, class Alloc>
typename deque<T, Alloc>::iterator deque<T, Alloc>::begin()const{ return beg_; }
```

- 返回尾部元素

```c++
template<class T, class Alloc>
typename deque<T, Alloc>::iterator deque<T, Alloc>::end()const{ return end_; }

template<class T, class Alloc>
typename deque<T, Alloc>::iterator deque<T, Alloc>::end(){ return end_; }
```

## 6 实例测试



在上面我们已经完成了构造和成员函数，接下来我们运用自己写的 deque 编写一个程序来测试一下效果，这里我们需要在 `Test` 目录下创建 `dequetest.cpp`文件。

```c++
#include <iostream>
#include "Deque.h"

void print(mySTL::deque<int> temp){
    mySTL::deque<int>::iterator it = temp.begin();
    for(; it != temp.end();it++)
        std::cout<<*it<<" ";
    std::cout<<std::endl;
}

int main(){
    mySTL::deque<int> val;
    val.push_back(1);
    val.push_back(4);
    val.push_back(7);
    std::cout<<"After pushback:"<<std::endl;
    print(val);

    val.push_front(8);
    val.push_front(5);
    val.push_front(2);
    std::cout<<"After pushfront:"<<std::endl;
    print(val);

    std::cout<<"The size of val: "<<val.size()<<std::endl;

    std::cout<<"After copy:"<<std::endl;
    mySTL::deque<int> val1(val);
    print(val1);

    std::cout<<"After popback:"<<std::endl;
    val.pop_back();
    print(val);

    std::cout<<"After popfront:"<<std::endl;
    val.pop_front();
    print(val);


}
```

在命令行中执行如下代码：

```bash
g++ dequetest.cpp -std=c++11 -o dequetest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539851955167.png)

## 7 实验总结



在以上的学习过程中，我们已经了解到了 deque 的定义（一个双端队列）以及 deque 的基本操作，赋值遍历等等。想必大家已经理解到 deque，剩下的则是需要自己去思考，总结自己所遇到的一些问题，这样才会更加深刻的理解 deque。

# 九 容器适配器



## 1 实验内容



本次实验主要讲述容器适配器 bitset 的介绍和用法。

#### 知识点

- bitset
- stack
- queue
- priority_queue

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip

# 解压文件到Code目录
unzip -q mySTL.zip -d ./Code/
```

## 2 bitset



bitset 是 STL 的一部分，准确地说，bitset 是一个模板类，它的模板参数不是类型，而是整形的数，有了它我们可以像使用数组一样使用类。在 `include` 目录下创建 `Bitset.h`。

### 2-1 初始化

- 无参构造

```c++
bitset()
try:head(0)   {  //计算分配的字节数，如SIZE 为10则分配2个字节就可以了，剩余6位。（16-10）
    // 以下同理可得
    head = new UCHAR[get_pos(size()) + 1];
    std::memset(head,0,get_pos(size()) + 1); //
}
catch(...){
    #ifdef _DEBUG
    std::cerr<<"out of memory"<<std::endl;
    #endif
}
```

- 带参构造(无符号数)

```c++
explicit bitset(ULONG val)    // unsigned long 版本
try:head(0){
    head = new UCHAR[get_pos(size()) + 1];
    std::memset(head,0,get_pos(size()) + 1);
    for(size_type i = 0; i < size() && i < sizeof(ULONG) * 8; ++i)
    set(i,read(&val,i));
}
catch(...){
    #ifdef _DEBUG
    std::cerr<<"out of memory"<<std::endl;
    #endif
}
```

- 带参构造(字符型)

```c++
explicit bitset(const std::string& str,size_type pos = 0,size_type n = std::string::npos)
try:head(0){
    head = new UCHAR[get_pos(size()) + 1];        //分配字节数
    std::memset(head,0,get_pos(size()) + 1);    //清0
    assert(n == std::string::npos || pos + n < str.size());
    if(pos + n > str.size())
        n = str.size() - pos;
    for(size_type i = 0,j = pos + n; i < size() && j >= pos + 1; ){
        assert(str[j-1] == '0' || str[j-1] == '1');
        #ifdef NDEBUG
        if(str[j-1] != '0' && str[j-1] != '1')
        throw mySTL::invalid_argument_1();      //非法参数，抛出异常
        #endif
        set(head,i++,str[--j] == '1');
    }
}
catch(...){
    #ifdef _DEBUG
    std::cerr<<"out of memory"<<std::endl;
    #endif
}
```

- 拷贝构造

```c++
bitset(const self& temp):head(0){
    head = new UCHAR[get_pos(size()) + 1];
    std::memcpy(head,temp.head,get_pos(size()) + 1);
}
```

- 析构

```c++
~bitset(){
    delete [] head;
}
```

### 2-3 成员函数



bitset 所需要的成员函数较多，这里不一一例举，讲解几个常用函数。更多内容请参考附件 `Bitset.h`。

- 重载 `[]`, 以数组方式访问

```c++
const_reference operator [] (size_type pos) const {
    assert(pos < size());        //判断是否越界
    return reference(head,pos);
}
```

- count 返回 1 的个数

```c++
size_type count() const {
    size_type m_count = 0;
    for(size_type i = 0; i < size(); ++i){
        size_type cur_pos = get_pos(i);//获取位置
        size_type cur_sub = get_sub(i);//获取字节数
        if(read(&head[cur_pos],cur_sub))
            ++m_count;
    }
    return m_count;
}
```

- size 返回集合大小

```c++
size_type size() const {
    return SIZE;
}
```

- test 测试某一位是否是 1

```c++
bool test(size_t pos) const{
    assert(pos < size());
    size_type cur_pos = get_pos(pos);
    size_type cur_sub = get_sub(pos);
    return read(&head[cur_pos],cur_sub);
}
```

- set 将一个 bit 设为 1

```c++
self& set(size_type pos, bool val = true){
    assert(pos < size());
    size_type cur_pos = get_pos(pos);
    size_type cur_sub = get_sub(pos);
    set(&head[cur_pos],cur_sub,val);
    return *this;
}
```

- reset 将一个 bit 设为 0

```c++
self& reset(size_type pos){
    assert(pos < size());
    size_type cur_pos = get_pos(pos);
    size_type cur_sub = get_sub(pos);
    set(&head[cur_pos],cur_sub,false);
    return *this;
}
```

- flip 翻转该 bit 的值（求反）

```c++
self& flip(){
    size_type pos = get_pos(size());        //获取所有地址
    for(size_type i = 0; i <= pos; ++i)
        head[i] = ~head[i];                    //逐个取反
    zero_last();
    return *this;
}
```

- 辅助成员函数

read: 判断 1 或 0,要和 self& read 区分开

```c++
bool read(void *ptr,size_type pos) {
    assert(ptr != 0);
    unsigned char *pointer = (unsigned char*)ptr;
    size_type subpos = (pos + 7)/ 8 - 1;
    size_type index = (pos  + 7) % 8 + 1;
    char tmp_val = (pointer[subpos] >> (index - 1) ) & char(1) ;
    return tmp_val > 0;
}
```

set: 置 1，与 self& set 区分

```c++
void* set(void* ptr,size_type pos,bool val = true){   //一般性的函数，处理数组中单个值的单个bit的设定
    assert(ptr != 0);
    unsigned char *pointer = (unsigned char*)ptr;
    size_type subpos = (pos + 7)/ 8 - 1;
    size_type index = (pos + 7) % 8  + 1;
    if(val)
        set_true(pointer[subpos],index);
    else
        set_false(pointer[subpos],index);
    return ptr;
}
```

get_pos: 获取字节

```c++
size_type get_pos(size_type pos) const   // 从0开始
{     // 举例说明，低位到高位假如有10位1000 0001 11 则最后一位getpos的结果为1,
    // 是第二个字节，getsub则是1
    return pos / 8;
}
```

get_sub: 获取数的位置

```C++
size_type get_sub(size_type pos) const   // 从0开始
{
    return pos % 8;
}
```

## 3 实例测试



完成上述操作后，我们在 `Test` 目录下创建 `bitsettest.cpp`，测试一下代码。

```c++
#include <iostream>
#include "Bitset.h"
int main(){
    mySTL::bitset<32> bit(0x0153cefa);
    for(int i = 0;i < 32;i++)
    std::cout << "bit" << i <<": " << bit[i] <<" \t ";
    std::cout<<std::endl;
    std::cout << "bit: " << bit << std::endl;

    std::cout << "The num of 1: " << bit.count() << std::endl;

    std::string tag = "";
    if(bit.test(6))
    tag = "true";
    else
    tag = "fasle";
    std::cout << "IS the position 6 1 ?: " << tag << std::endl;

    std::cout << "After flip bit: " << bit.flip() << std::endl;

    bit.set(4);
    std::cout << "Set position 4 to 1: " << bit << std::endl;

    bit.reset(4);
    std::cout << "Reset position 4 to 0: " << bit << std::endl;


}
```

在命令行中执行如下代码：

```bash
g++ bitsettest.cpp -std=c++11 -o bitsettest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539851867480.png)



## 4 stack

stack 类允许在底层数据结构的一端执行插入和删除操作（先入后出）。堆栈能够用任何序列容器实现：vector、list、deque。 首先在 `include` 目录下创建 `Stack.h`.

### 4-1 初始化

- 构造

```c++
explicit stack(const container_type& ctnr = container_type()) :container_(ctnr){}
```

### 4-2 成员函数



- empty 判断是否为空

```c++
bool empty() const{ return container_.empty(); }
```

- size 返回元素个数

```c++
size_type size() const{ return container_.size(); }
```

- 返回栈顶

```c++
value_type& top(){ return (container_.back()); }
const value_type& top() const{ return (container_.back()); }
void push(const value_type& val){ container_.push_back(val); }
```

- 入栈

```c++
void push(const value_type& val){ container_.push_back(val); }
```

- 出栈

```c++
void pop(){ container_.pop_back(); }
```

- 交换

```c++
void swap(stack& x){ mySTL::swap(container_, x.container_); }
```

## 5 实例测试



完成上面的构造后，我们在 `Test` 目录下创建 `stacktest.cpp`，来测试一下。

```c++
#include <iostream>
#include "Stack.h"

void print(mySTL::stack<int> temp){

    while(!temp.empty()){
        std::cout<<temp.top()<<" ";
        temp.pop();
    }
    std::cout<<std::endl;
}

int main(){
    mySTL::stack<int> stc;
    stc.push(3);
    stc.push(4);
    stc.push(2);
    stc.push(5);
    stc.push(7);
    std::cout<<"After push: ";
    print(stc);

    std::cout<<"The size of stack: "<<stc.size()<<std::endl;

    stc.pop();
    stc.pop();
    std::cout<<"After pop: ";
    print(stc);

    std::cout<<"The size of stack after pop: "<<stc.size()<<std::endl;
    return 0;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539851800512.png)

## 6 queue



queue 类允许在底层数据结构的末尾插入元素，也允许从前面插入元素（先入先出）。 队列能够用 STL 数据结构的 list 和 deque 实现，默认情况下是用 deque 实现的。

首先需要在 `include` 目录下创建 `Queue.h`.

### 6.1 初始化



- 无参构造

```c++
queue(){}
```

- 传参构造

```c++
explicit queue(const container_type& ctnr) :container_(ctnr){}
```

### 6.2 成员函数



- empty: 判断是否为空

```c++
bool empty() const{ return container_.empty(); }
```

- size: 返回元素个数

```c++
size_type size() const{ return container_.size(); }
```

- front: 返回头部

```c++
reference& front(){ return container_.front(); }
const_reference& front() const{ return container_.front(); }
```

- 返回尾部

```c++
reference& back(){ return container_.back(); }
const_reference& back() const{ return container_.back(); }
```

- 进队

```c++
void push(const value_type& val){ container_.push_back(val); }
```

- 出队

```c++
void pop(){ container_.pop_front(); }
```

- 交换队列元素

```c++
void swap(queue& x){ container_.swap(x.container_); }
```

## 7 实例测试



完成了上述操作之后，我们在 `Test` 目录下创建 `queuetest.cpp`,来测试一下功能。

```c++
#include <iostream>
#include "Queue.h"

void print(mySTL::queue<int> temp){

    while(!temp.empty()){
        std::cout<<temp.front()<<" ";
        temp.pop();
    }
    std::cout<<std::endl;
}

int main(){
    mySTL::queue<int> stc;
    stc.push(3);
    stc.push(4);
    stc.push(2);
    stc.push(5);
    stc.push(7);
    std::cout<<"After push: ";
    print(stc);

    std::cout<<"The size of queue: "<<stc.size()<<std::endl;

    stc.pop();
    stc.pop();
    std::cout<<"After pop: ";
    print(stc);

    std::cout<<"The size of queue after pop: "<<stc.size()<<std::endl;

    std::cout<<"The front of queue: "<<stc.front()<<std::endl;
    std::cout<<"The back of queue: "<<stc.back()<<std::endl;
    return 0;
}
```

在命令行中执行如下代码：

```bash
g++ queuetest.cpp -std=c++11 -o queuetest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539851698336.png)

## 8 priority_queues



优先级队列 priority_queue 是允许用户以任意顺序将元素放入容器，但是取出元素时一定是从最高优先级的元素开始取出。默认值大的优先级高。 priority_queues 和 queue 非常相似，所以我们不要重新创建头文件，直接写在 `Queue.h` 中就可以。

### 8-1 初始化



- 无参构造,优先级以函数对象方式

```c++
explicit priority_queue(const Compare& comp = Compare(),
const Container& ctnr = Container())
: container_(ctnr), compare_(comp){}
```

- 传参构造

```c++
template <class InputIterator>
priority_queue(InputIterator first, InputIterator last,
const Compare& comp = Compare(),
const Container& ctnr = Container())
: container_(ctnr), compare_(comp){
    container_.insert(container_.end(), first, last);//插入元素
    mySTL::make_heap(container_.begin(), container_.end());//创建堆
}
```

### 8-2 成员函数



- empty:判断是否为空

```c++
bool empty() const{ return container_.empty(); }
```

- size:返回元素个数

```c++
size_type size() const{ return container_.size(); }
```

- top:返回头

```c++
reference top() {
    return container_.front();
}
```

- push:进队

```c++
void push(const value_type& val){ container_.push_back(val); }
```

- pop:出队

```c++
void pop(){ container_.pop_front(); }
```

- swap+交换

```c++
void swap(queue& x){ container_.swap(x.container_); }
```

## 9 实例测试



完成了上述操作，我们在 `Test` 目录下创建 `priority_queuestest.cpp`,来测试。

```c++
#include <iostream>
#include "Queue.h"

void print(mySTL::priority_queue<int> temp){    //打印
    while(!temp.empty()){
        std::cout<<temp.top()<<" ";
        temp.pop();
    }
    std::cout<<std::endl;
}

int main(){
    mySTL::priority_queue<int> stc;                //定义一个优先队列
    stc.push(3);
    stc.push(4);
    stc.push(2);
    stc.push(5);
    stc.push(7);
    std::cout<<"After push: ";
    print(stc);

    std::cout<<"The size of priority_queue: "<<stc.size()<<std::endl;

    stc.pop();
    stc.pop();
    std::cout<<"After pop: ";
    print(stc);

    std::cout<<"The size of priority_queue after pop: "<<stc.size()<<std::endl;

    std::cout<<"The top of priority_queue: "<<stc.top()<<std::endl;

    return 0;
}
```

在命令行中执行如下代码：

```bash
g++ priority_queuetest.cpp -std=c++11 -o priority_queuetest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539851546915.png)

## 10 实验总结



通过本次实验我们讲解了位运算、栈和队列，也应当知道容器适配器是干什么的，容器适配器其实相当于一个接口，我们把已有的容器重新封装一下，改变它的接口，就能把它当做栈或队列使用了，这样使我们的程序更加结构化，非常方便。

# 十 容器之 set 和 multiset

## 1 实验内容



本次实验主要讲述 set（集合） 和 multiset（多重集） 的区别和构造。

#### 知识点

- 关联容器介绍
- 两种容器的构造
- 成员函数
- 两种容器的基本操作

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件`Set.h` 和 `Multiset.h`中。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip

# 解压文件到Code目录
unzip -q mySTL.zip -d ./Code/
```

本次实验需要用到的头文件包括：

```c++
#include "Utility.h"            //定义了一种数据结构用于传参
#include "RBtree.h"             //红黑树，一种数据结构，方便与存储和查找
#include "Functional.h"            //一些函数对象
```

## 2 关联容器介绍



C++ 的容器类型可以分为顺序容器和关联容器两大类。顺序容器我们前面的使用已经介绍了 vector、list 和 deque，现在我们介绍一下关联容器。关联容器支持通过键值来高效的查找和读取元素，这是它和顺序容器最大的区别。两种基本的关联容器类型是 map 和 set,衍生类型有 multimap 和 multiset 下面给出四种类型差别：

```c++
map                      关联数组；元素通过键来存储和读取
set                      大小可变的集合，支持通过键实现快速读取
multimap                 支持同一个键多次出现的map类型
multiset                 支持同一个键多次出现的set类型
```

本次实验主要讲述 set 和 multiset。

### 2-2 两种容器的构造



#### set 的构造

因为 multiset 和 set 基本相似，所以我们这里只讲 set，在第六小节我们会讲 multiset 的个别区别，我们先在 `include` 目录选创建 `Set.h`。

- 成员变量

```c++
public:
          typedef Key     key_type;            //键值
          typedef Key     value_type;            //数值
          typedef Compare key_compare;        //键值比较
          typedef Compare value_compare;        //数值比较
private:
        typedef RBtree<key_type, value_type,identity<value_type>,                 key_compare> RBtree_type;            //利用红黑树传参
        RBtree_type tree;                    //定义红黑树

public:
          typedef typename RBtree_type::const_iterator         iterator;                                                //迭代器
          typedef typename RBtree_type::size_type             size_type;
          typedef typename RBtree_type::iterator                             tree_iterator;
```

- 无参构造

```c++
set(){}
```

- 传参构造

```c++
set(const set& x):tree(x.tree){}
```

- 区间构造,把 first 到 last 去加元素传递给 set

```c++
template <class InputIterator>
set(InputIterator first,InputIterator last){     tree.insert_unique(first,last);}
```

## 3 基本操作



set 容器中的键值是 const，在获取 set 容器中的某个元素迭代器后，只能对其做读操作，而不能做写操作。

- 返回头部迭代器

```c++
iterator begin()const {return tree.begin();}
```

- 返回尾部迭代器

```c++
iterator end()const {return tree.end();}
```

- 判断是否为空

```c++
bool empty()const {return tree.empty();}
```

- 返回元素个数

```c++
size_type size()const {return tree.size();}
```

- 交换 set 元素

```c++
void swap(set& x) {tree.swap(x.tree);}
```

- 插入

通过 pair 插入一个值

```c++
pair<iterator,bool> insert(const value_type& x)            //pair 类型定义在 utility.h 中
{
    pair<tree_iterator,bool> p = tree.insert_unique(x);    //set 插入使用  insert_unique
    return pair<iterator,bool>(p.first,p.second);//返回数值
}
```

插入某个区间的值

```c++
template<class InputIterator>
void insert(InputIterator first,InputIterator last){
    tree.insert_unique(first,last);//利用红黑树插入
}
```

- 删除

删除某个位置的值

```c++
void erase(iterator position){
    tree.erase((tree_iterator&)position);
}
```

删除某个确认的值

```c++
size_type erase(const key_type& x){
    return tree.erase(x);
}
```

- 清空

```c++
void clear() {tree.clear();}
```

## 4 迭代器相关



- 查找具体数值

```c++
iterator find(const key_type& x) { return tree.find(x);}
```

- 记数

```c++
size_type count(const key_type& x) { return tree.count(x);}
```

- 返回一个 iterator

从数组的 begin 位置到 end-1 位置二分查找第一个大于或等于 x 的数字

```c++
iterator lower_bound(const key_type& x) {return tree.lower_bound(x);}
```

从数组的 begin 位置到 end-1 位置二分查找第一个大于 num 的数字

```c++
iterator upper_bound(const key_type& x) {return tree.upper_bound(x);}
```

- 返回一对迭代器

返回 lower_bound 和 upper_bound

```c++
pair<iterator,iterator> equal_range(const key_type& x) {return tree.equal_range(x);}
```

## 5 Multiset



在 `include` 目录下创建 `Multiset.h` 。multiset 和 set 是极其相似的，只有个别部分不同，这里就讲一下 不同的地方。

- 构造方式

set 使用的 insert_unique 函数，这样 set 的键值就不允许重复。

multiset 使用的 insert_equal 函数，这样允许 multiset 的键值重复。

- 插入方式

set 使用 `pair<iterator,bool> insert(const value_type& x)`返回的是键值

multiset 使用 `iterator insert(const value_type& x)` 返回的是迭代器

## 6 实例测试



通过上面的构造和成员函数的编写我们的 set 和 multiset 已经初具规模，现在就来测试一下。

- set

我们在 `Test` 目录下创建 `settest.cpp`

```c++
#include <iostream>
#include "Set.h"

int main(){
    int i;
    int arr[6] = {4,2,7,1,9,8};                    //定义一个数组
    std::cout<<"Before putting in set: ";
    for(int i = 0;i < 6;i++)
    std::cout<<arr[i]<<" ";                    //给数组赋值
    std::cout<<std::endl;

    mySTL::set<int> iset(arr,arr+6);            //创建一个set容器
    std::cout<< "lower_bound 2: " <<*iset.lower_bound(2)<<std::endl;//找到第一个小于等于2的数
    std::cout<< "upper_bound 2: " <<*iset.upper_bound(2)<<std::endl;//找到第一个大于2的数

    std::cout<<"After insert and put 3: ";
    iset.insert(3);                                //插入
    mySTL::set<int>::iterator it;
    for(it = iset.begin ();it != iset.end ();it++)    //打印set
    {
        std::cout<<*it<<" ";
    }
    std::cout<<std::endl;
    it = iset.find(8);                            //查找
    std::cout <<"Can we find 8?: ";
    if(it == iset.end())                         //判断是否存在
        std::cout<< "NO!" <<std::endl;
    else
        std::cout<< "YES!" <<std::endl;
    std::cout<<std::endl;

    return 0;
}
```

在命令行中执行如下命令：

```bash
g++ settest.cpp -std=c++11 -o settest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539851461871.png)

- multiset

在 `Test` 目录下创建 `multisettest.cpp`。

```c++
#include <iostream>
#include "Multiset.h"

int main(){
    mySTL::multiset<int> imultiset;                        //定义一个multiset
    imultiset.insert(1);                                //插入5个值
    imultiset.insert(2);
    imultiset.insert(1);
    imultiset.insert(4);
    imultiset.insert(4);
    std::cout<<"After insert ";

    auto it = imultiset.begin();
    for(;it != imultiset.end();it++)                    //输出
        std::cout<<*it<<" ";
    std::cout<<std::endl;
    std::cout<<"size:"<<imultiset.size()<<std::endl;    //返回元素个数
    std::cout<< "lower_bound 2: " <<*imultiset.lower_bound(2)<<std::endl;                                            //找到第一个小于等于2的数
    std::cout<< "upper_bound 2: " <<*imultiset.upper_bound(2)<<std::endl;                                            //找到第一个大于2的数
    it = imultiset.find(2);                                //查找
    std::cout <<"Can we find 2?: ";
    if(it == imultiset.end())
        std::cout<< "NO!" <<std::endl;
    else
        std::cout<< "YES!" <<std::endl;

    std::cout<<"After erase 2"<<std::endl;
    imultiset.erase(2);                                    //删除
    it = imultiset.find(2);
    std::cout <<"Can we find 2?: ";
    if(it == imultiset.end())
        std::cout<< "NO!" <<std::endl;
    else
        std::cout<< "YES!" <<std::endl;
    std::cout<<std::endl;
    return 0;
}
```

在命令行中执行如下命令：

```bash
g++ multisettest.cpp -std=c++11 -o multisettest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539851193936.png)

## 7 总结



通过本实验我们了解到了 set 和 multiset 的基本构造与使用方法，在这里再强调几点：

- set/multiset 会根据待定的排序准则，自动将元素排序。
- 两者不同在于前者不允许元素重复，而后者允许；
- 不能直接改变元素值；
- 元素比较只能发生在类型相同的容器。

# 十一 容器之 map 和 multimap



## 1 实验内容



本次实验主要讲述 map 和 multimap 的区别和用法。

#### 知识点

- 容器作用和介绍
- 容器的构造
- 成员函数
- 基本操作

#### 实验环境

- g++
- ubuntu 16.04

#### 代码获取

可以通过以下链接获取本课程的源码内容，本次实验内容主要包含在文件`Map.h` 和 `Multimap.h`’中。

```bash
# 获取代码
wget https://labfile.oss.aliyuncs.com/courses/1166/mySTL.zip

# 解压文件到Code目录
unzip -q mySTL.zip -d ./Code/
```

本次实验需要用到的头文件包括：(可以在 `include` 目录下查看)

```c++
#include "Utility.h"            //定义了一种数据结构用于传参
#include "RBtree.h"             //红黑树，一种数据结构，方便与存储和查找
#include "Functional.h"            //一些函数对象
```

## 2 作用和介绍



Map 是 STL 的一个关联容器，它提供一对一（其中第一个可以称为关键字，每个关键字 key 只能在 map 中出现一次，第二个可能称为该关键字的值 data）的数据处理能力，由于这个特性，它完全有可能在我们处理一对一数据的时候，在编程上提供快速通道。

和 set/multiset 类似，map/multimap key 不能改，但是 data 可以改,因此 map 仍然具有自动排序的功能。

map 的 key 必须独一无二,而 multimap 的 key 可以重复。

## 3 容器的构造



因为 multimap 和 map 基本相似，所以我们这里只讲 map，在第六小节我们会将 multimap 的个别区别，我们先在 `include` 目录创建 `Map.h`。

- 成员变量

```c++
public:
          typedef Key     key_type;            //键值
          typedef Key     value_type;            //数值
        typedef T       mapped_type;        //map 类型
          typedef Compare key_compare;        //键值比较
          typedef Compare value_compare;        //数值比较
private:
        typedef RBtree<key_type, value_type,select1st<value_type>,                 key_compare> RBtree_type;            //利用红黑树传参
        RBtree_type tree;                    //定义红黑树

public:
          typedef typename RBtree_type::const_iterator         iterator;                                                //迭代器
          typedef typename RBtree_type::size_type             size_type;
          typedef typename RBtree_type::iterator                             tree_iterator;
```

- 无参构造

```c++
map(){}
```

- 传参构造

```c++
map(const map& x):tree(x.tree){}
```

- 区间构造,把 first 到 last 去加元素传递给 map

```c++
template <class InputIterator>
          map(InputIterator first,InputIterator last){ tree.insert_unique(first,last);}
```

## 4 成员函数



- 返回头部迭代器

```c++
iterator begin()const {return tree.begin();}
```

- 返回尾部迭代器

```c++
iterator end()const {return tree.end();}
```

- 判断是否为空

```c++
bool empty()const {return tree.empty();}
```

- 返回元素个数

```c++
size_type size()const {return tree.size();}
```

- 交换 map 元素

```c++
void swap(map& x) {tree.swap(x.tree);}
```

- 访问符重载,允许下标访问

```c++
T& operator[](const key_type& k){
    return (*((insert(value_type(k,T())).first))).second;
}
```

- 清空

```c++
void clear() {tree.clear();}
```

## 5 基本操作



- 插入

通过 pair 插入一对数据

```c++
pair<iterator,bool> insert(const value_type& x){
    return tree.insert_unique(x);
}
```

插入某个区间的值

```c++
template<class InputIterator>
void insert(InputIterator first,InputIterator last){
    tree.insert_unique(first,last);
}
```

- 删除

删除某个位置的值

```c++
void erase(iterator position){
    tree.erase((tree_iterator&)position);
}
```

删除某个确认的值

```c++
size_type erase(const key_type& x){
    return tree.erase(x);
}
```

- 查找

```c++
iterator find(const key_type& x) { return tree.find(x);}
```

## 6 Multimap



在 `include` 目录下创建 `Multimap.h` 。multimap 和 map 是极其相似的，只有个别部分不同，这里就讲一下 不同的地方。

- 构造方式

map 使用的 insert_unique 函数，这样 map 的键值就不允许重复。

multimap 使用的 insert_equal 函数，这样允许 multimap 的键值重复。

- 插入方式

map 使用 `pair<iterator,bool> insert(const value_type& x)` 返回的是键值和数据

multimap 使用 `iterator insert(const value_type& x)` 返回的是迭代器

## 7 实例测试



通过上面的构造和成员函数的编写我们的 map 和 multimap 已经初具规模，现在就来测试一下。

- map

首先在 `Test` 目录下创建 `maptest.cpp`

```c++
#include <iostream>
#include "Map.h"

int main(){
    mySTL::map<std::string, int> map;                            //定义一个map
    map.insert(mySTL::pair<std::string,int>("zhangsan",110));    //插入
    map.insert(mySTL::pair<std::string,int>("lisi",120));
    map.insert(mySTL::pair<std::string,int>("wangwu",130));
    map.insert(mySTL::pair<std::string,int>("mazi",140));
    const mySTL::pair<const std::string, int> value(std::string("pangdun"),150);//定义一个pair
    map.insert(value);

    std::cout<<"After insert: "<<std::endl;
    auto iter = map.begin();
    for(;iter != map.end();++iter)
         std::cout<<"name: "<<iter->first <<' ' <<"id: "<<iter->second<<std::endl;  //打印键值和数值
    std::cout <<"Can we find lisi?: ";
    iter = map.find(std::string("lisi"));                //查找
    if(iter == map.end())                                 //判断是否存在
        std::cout<< "NO!" <<std::endl;
    else
        std::cout<< "YES!" <<std::endl;
    std::cout <<"His id is "<<iter->second<< std::endl;    //如果存在打印数值
    std::cout<<std::endl;

       std::cout<< "After erase lisi" <<std::endl;
    map.erase(iter);                                    //删除
    iter = map.begin();
    for(;iter != map.end();++iter)
         std::cout<<"name: "<<iter->first <<' ' <<"id: "<<iter->second<<std::endl;
    return 0;
}
```

在命令行执行如下命令：

```bash
g++ maptest.cpp -std=c++11 -o maptest -I ../include
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539854196829.png)

- multimap

在 `Test` 目录下创建 `multimap.cpp`

```c++
#include <iostream>
#include "Multimap.h"

int main(){
    mySTL::multimap<std::string, int> mtmap;                    //定义一个multimap
    mtmap.insert(mySTL::pair<std::string,int>("zhangsan",110));    //插入键值和数值
    mtmap.insert(mySTL::pair<std::string,int>("lisi",120));
    mtmap.insert(mySTL::pair<std::string,int>("wangwu",130));
    mtmap.insert(mySTL::pair<std::string,int>("mazi",140));
    const mySTL::pair<const std::string, int> value(std::string("zhangsan"),150);//定义一个pair
    mtmap.insert(value);                                        //插入pair

    std::cout<<"After insert: "<<std::endl;
    auto iter = mtmap.begin();
    for(;iter != mtmap.end();++iter)
         std::cout<<"name: "<<iter->first <<' ' <<"id: "<<iter->second<<std::endl;  //打印map
    std::cout <<"Can we find lisi?: ";
    iter = mtmap.find(std::string("lisi"));                        //查找
    if(iter == mtmap.end())                                     //判断是否存在
        std::cout<< "NO!" <<std::endl;
    else
        std::cout<< "YES!" <<std::endl;
    std::cout <<"His id is "<<iter->second<< std::endl;            //如果存在打印数值
    std::cout<<std::endl;

       std::cout<< "After erase lisi" <<std::endl;
    mtmap.erase(iter);                                            //删除
    iter = mtmap.begin();
    for(;iter != mtmap.end();++iter)
         std::cout<<"name: "<<iter->first <<' ' <<"id: "<<iter->second<<std::endl;
    return 0;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539854389326.png)

## 8 实验总结



通过本次实验我们掌握了 map 和 multimap 的构造和常规操作，肯定也更加深刻的理解了关联容器。也清楚了 set 和 map 的区别。set 和 map 是用来方便访问的，键值都不允许修改，但 multimap 的数据可以修改，因为键值都是常量。到此容器部分已经讲完了，在以后的编程中我们就可以直接使用了。

# 十二 异常处理



## 1 实验内容



本次实验将详细的介绍什么是异常，异常的语法。通过本次实验会加深对异常的理解。

#### 知识点

- 异常介绍

#### 实验环境

- g++
- Linux16.04

## 2 介绍



异常是程序在执行期间产生的问题，C++ 异常是指在程序运行时发生的特殊情况(如被零除情况或内存不足警告)。异常提供了一种转移程序控制权的方式。C++ 异常处理涉及到三个关键字：try、catch、throw。

- try: try 块中的代码标识将被激活的特定异常。它后面通常跟着一个或多个 catch 块。
- catch: 在您想要处理问题的地方，通过异常处理程序捕获异常。catch 关键字用于捕获异常。
- throw: 当问题出现时，程序会抛出一个异常。这是通过使用 throw 关键字来完成的。

举一个例子，将下面的代码写入 `/home/shiyanlou/Code/exception.cpp` 文件中：

```c++
#include <iostream>

using namespace std;

int division(int a,int b)
{
    if(b == 0)
        throw "除数不能为0";        //抛出异常
    return a/b;
}

int main()
{
    int a,b;
    cout<<"输入被除数和除数:"<<endl;
    cin>>a>>b;
    try// 保护代码
    {
        cout<<"a / b = "<<division(a,b);
    }
    catch(const char* msg)        // 能处理任何异常的代码
    {
        cerr<<msg<<endl;
    }
    return 0;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539855112561.png)

## 3 语法



从上面的例子中我们不难看出异常操作的语法。

抛出异常:

```c++
throw  Exception
```

捕获异常:

```c++
try
{
       // 保护代码
}catch(...)
{
      // 能处理任何异常的代码
}
```

## 4 异常处理进阶



- 堆栈解退

当抛出了异常，但还没在特定的作用域中被捕获时，函数调用堆栈便被“解退”，并试图在下一个外层 try...catch 代码中捕获这个异常。解退函数调用堆栈意味着抛出未捕获异常的那个函数将终止，这个函数中的所有局部变量都将销毁，控制会返回到原先调用这个函数的语句。

实例, 将下面的代码写入 `/home/shiyanlou/Code/test.cpp` 文件中：

```c++
#include <iostream>

using namespace std;

void fun3()
{
    cout<<"In fun 3"<<endl;
    throw"runtime_error in fun3";        //抛出异常,fun1,fun2终止
}

void fun2()
{
    cout<<"fun3 is called inside fun2"<<endl;
    fun3();                                //调用fun3()
}

void fun1()
{
    cout<<"fun2 is called inside fun1"<<endl;
    fun2();                                //调用fun2()
}

int main()
{
    try
    {
        cout<<"fun1 is called inside main"<<endl;
        fun1();
    }
    catch(const char* msg)                //捕获异常
    {
        cout<<"exception occurred: "<< msg<<endl;
        cout<<"exception handled in main"<<endl;
    }

    return 0;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539855336140.png)

- 迷失

异常什么时候会迷失方向呢？

- 意外异常：如果是在带异常规范的函数中引发的，则必须与规范列表里的某个异常匹配，若没有匹配的，则为意外异常，默认情况下，会导致程序异常终止
- 未捕获异常：如果不是在函数中引发的（或者函数没有异常规范），则它必须被捕获。如果没被捕获（没有 try 块或没有匹配的 catch 块），则为未捕获异常。默认情况下，将导致程序异常终止

将下面的代码写入 `/home/shiyanlou/Code/test1.cpp` 文件中：

```c++
#include <iostream>
#include <exception>
#include <stdexcept>
using namespace std;

int main()
{
    int c;
    logic_error a("logic_error");                //定义一个逻辑异常
    range_error b("runtime_error");                //定义一个作用域异常
    try
    {
        cout<<"输入一个数:";
        cin>>c;
        switch(c)
        {
            case 1:
                throw a;
                break;
            default:
                cout << "throw a runtime_error" << endl;//抛出一个作用域异常
                throw b;
                break;
        }
    }
    catch(const logic_error & e)
    {
        cout << e.what() << endl;
    }
    catch(...)                                    //捕获其它所有可能的异常，程序终止
    {
        cout << "OK" << endl;
        terminate();
    }

    return 0;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539856078294.png)

- 局限性

全局对象在程序开始运行前进行构造，如果构造函数抛出异常，将永远无法捕获，析构也是如此，因为它们在程序结束后才会被调用，这些异常只有操作系统才可以捕获，应用程序没有办法。 对于局部对象，异常处理机制是所有从 try 到 throw 语句之间构造起来的局部对象的析构函数将被自动调用，然后清退栈堆（就像 main 函数退出那样），如果一直回溯到 main 函数后还是没有匹配到 catch 块，那么系统会调用 terminate() 终止整个程序，在这种情况下就不能保证局部对象会被正确的销毁了。

## 5 实现自己的异常处理



想要实现自己的异常处理函数库，需要派生基类 exception，然后自己定义虚函数 what()。

将下面的代码写入 `/home/shiyanlou/Code/test2.cpp` 文件中：

```c++
#include <iostream>
#include <exception>

using namespace std;

class myexception : public exception        //自定义一个异常类
{
public:
    myexception(){}
    const char* what()
    {
        return "There has an exception";
    }

};

int division(int a,int b)
{
    if(b == 0)
        throw myexception();                //抛出异常
    return a/b;
}

int main()
{
    int a,b;
    cout<<"输入被除数和除数:"<<endl;
    cin>>a>>b;
    try// 保护代码
    {
        cout<<"a / b = "<<division(a,b);
    }
    catch(myexception &e)                    // 捕获异常
    {
        cout<< e.what()<<endl;
    }


    return 0;
}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid600404labid7463timestamp1539856448233.png)

## 6 实验总结



异常处理是一个非常优秀的机制，利用这个机制程序员可直接或者间接处理程序错误。但是，使用异常处理还需注意几点:虽然异常处理利于找出错误，但也会打断执行流，也会增加代码量，所以合理的运用才能发挥它的效率。
