# STL概论

## 一.STL简介

标准模板库（Standard Template Library，STL）是[惠普实验室](https://baike.baidu.com/item/惠普实验室/489303)开发的一系列软件的统称。它是由Alexander Stepanov、Meng Lee和David R Musser在[惠普实验室](https://baike.baidu.com/item/惠普实验室/489303)工作时所开发出来的。虽说它主要出现在C++中，但在被引入C++之前该技术就已经存在了很长时间。STL的代码从广义上讲分为三类：algorithm（算法）、container（容器）和iterator（[迭代器](https://baike.baidu.com/item/迭代器/3803342)），几乎所有的代码都采用了[模板类](https://baike.baidu.com/item/模板类/6768650)和模板函数的方式，这相比于传统的由函数和类组成的库来说提供了更好的代码重用机会。

## 二.STL六大组件

容器：存放数据的各种数据结构，如vector，list等

算法：如排序，查找等常用算法

迭代器：算法与容器之间的连接器件，算法可通过迭代器操作容器

仿函数：算法的某种策略  //待完善

配接器：修饰容器或仿函数或迭代器接口的东西 //待完善

配置器：负责空间配置与管理。

六大组件的关系：容器通过配置器取得数据存储的空间，算法通过迭代器操作容器的内容，仿函数协助算法完成不同的策略变化（待完善），配接器可以修饰或套接仿函数（待完善）

## 三.实现版本

STL有各种实现版本，这里参考的是SGI STL

## 四.文件分布与简介

C++大致有五种头文件，一种是以C开头的标准规范文件，如<cmath>,<cstdio>，一种是非STL的标准程序库的文件，如<string>,<stream>，一种是STL的标准程序库的文件，如<vector>,<algorithm>,<functional>（有一说一，之前真不知道后面两个是STL里的文件），一种是HP（惠普）规范的头文件，如<vector.h>，最后一种是真正实现SGISTL的内部文件，如<stl_vector.h>,<stl_algo.h>.

## 五.SGISTL的编译器组态设置(configuration)

因为不同编译器对C++的支持程度不一样，所以SGI STL有一个组态文件<stl_config.h>，这个文件标识了某些组态的成立与否。

对于<stl_config.h>：文件起始处是一份常量定义说明，这些常量(对应的东西)是SGI STL所必须的东西，如果现在没定义之后也要定义。

然后是针对各家编译器的各种版本的判断，各家编译器的各种版本定义自己的常量，对各个常量来说，按对C++的支持程度从高到低各家编译器的各个版本有三种定义情况：没定义（说明自身就定义了这种功能）；定义为可以（自身没有这种功能，但是可以通过后面的代码定义来提供这种功能）；定义为不行（自身没有这种功能，并且后面的代码也不能定义这种功能，即这种编译器就不能提供这种功能）。

参考：

[STL源码剖析 [配置文件]（stl_config.h](https://blog.csdn.net/langb2014/article/details/48003693)

[sgi stl_config.h 各种宏的含义](https://blog.csdn.net/kongbu0622/article/details/3479344?locationNum=5&fps=1)

[stl_config.h基本宏](https://www.cnblogs.com/xiaohaige/p/6784588.html)

PS：其实我对这个配置文件的理解程度也不深，之后再慢慢看吧，待完善。

## 六.容易感到困惑的一些C++语法

### 1.<stl_config.h>中的各种组态

__STL_STATIC_TEMPLATE_MEMBER_BUG：如果无法定义类静态成员就定义这个

__STL_CLASS_PARTIAL_SPECIALIZATION：定义类模板偏特化

关于什么是特化：[C++模板特化](https://www.cnblogs.com/invisible2/p/8001192.html)

__STL_FUNCTION_TMPL_PARTIAL_ORDER：可以看做对函数模板重载的支持

__STL_MEMBER_TEMPLATES：支持模板中再有模板

__STL_LIMITED_DEFAULT_TEMPLATES：支持根据前一个模板参数设定默认值

__STL_NON_TYPE_TMPL_PARAM_BUG：类模板是否使用非类型模板参数

什么是非类型模板参数：[非类型模板参数](https://blog.csdn.net/zhangxiao93/article/details/50752752)

__STL_NULL_TMPL_ARGS：这个可以直接理解为<>，说是为了绑定友元类型准备的，这个绑定友元类型大致就是类是什么类型，让友元也是什么类型 //待完善

__STL_TEMPLATE_NULL：定义了template<>，也就是定义了一种特化的格式（感觉这个有点偏向于全特化）

PS：这几个组态蛮重要的，主要后面经常用到这几个组态相关的东西

参考：

[STL学习笔记（0）可能困惑的C++语法](https://blog.csdn.net/RaKiRaKiRa/article/details/82997891)

[STL笔记（1）——STL的一些组态](https://blog.csdn.net/zhangxiao93/article/details/50752752)



### 2.临时对象的产生与运用

临时对象会引发copy操作，常常造成效率上的负担（所以后来有了右值引用和完美转发？），但有时候使用临时对象可以使得代码变得精简，刻意制造临时对象就是在型别名称之后直接加一对小括号，如int(8).

### 3.静态常量整数成员在class内部直接初始化

如果class中含有const static integral data member（静态常量整数成员，所谓整数成员是指int,long,char等），可以直接在类里面初始化。

### 4.increment/decrement/dereference 操作符

迭代器的前进取值功能和后退取值功能，这两种又分为前置式和后置式（即先前进后取值还是先取值后前进），通过对++或--的两种操作符重载实现。

### 5.前闭后开区间表示法

任何一个STL算法都要获得一对迭代器指示的区间来表示操作范围，而这一对迭代器指示的范围通常是一个前闭后开区间，即值从first到last-1，last所指的是最后一个元素的下一个位置，这种设计方便了如循环等的操作。

### 6.function call操作符

有些地方我们需要用到一整组操作作为参数，这一整组操作可以是函数指针或lambda表达式，但是这两个都不具备可适配性（待完善），所以这里我们引入了仿函数，即设定一个类，在类里面重载()，这样这个类就作为了一种仿函数的存在并且具有可适配性。
