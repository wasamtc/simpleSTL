# 迭代器概念与traits编程技法

迭代器定义：提供一种方法，使之能够依序巡防某个聚合物（容器）所含的各个元素，而又无需暴露该聚合物的内部表述方式

## 一.迭代器设计思维—STL关键所在

STL中心思想在于将容器与算法分开，彼此独立设计然后用迭代器将两者撮合在一起，算法可以通过迭代器访问容器。

## 二.迭代器是一种smart pointer

开始先介绍了一个auto_ptr（这个和智能指针好像差不多），auto_ptr可以在离开作用域的时候自动销毁从而避免造成内存泄漏，实现方式就是把指针封装在一个auto_ptr类中，析构函数为delete，迭代器也可以借鉴auto_ptr的设计方式，封装一个相应类型的指针在类中，然后设计相应的函数使这个类表现的像个指针（主要是重载），后面设计了一个用于list的find的迭代器，这个迭代器也是只能给list用，而且单独在外面设计的话暴露了list太多的信息，还不如就在list里面设计，所以我们看到的迭代器都是专门设计在每一种STL容器的里面。

我觉得要理解迭代器很关键的一点就是迭代器的表现像指针，但这种相似是通过封装了指针以及许多函数的模板类实现的。

## 三.迭代器相应类别

假如设计好了迭代器，但是有一个问题，如何“以迭代器所指对象的型别”作为型别，如果我们定义了一个容器，然后用星号去取迭代器所指内容，得到的是值而不是型别，这里我们用的是模板函数的自动推导，即

```C++
template<typename T1, typename T2>
void func_impl(T1 iter, T2 t)
{
    T2 temp;
    // ......
}

template<typename T>
void func(T iter)
{
    func_impl(iter, *iter);
}
```

func是一个使用迭代器所指型别的接口，这样传参func_impl中的T2就是迭代器所指型别了。

与迭代器有关的型别不只有迭代器所指对象的型别这一种，还有许多其他的无法用这种参数推导的方式获得，所以我们需要更加全面的解法。

## 四.Traits编程技法——STL源代码门钥

前面我们说过，迭代器有许多相关的类型，其中许多类型是不能用三中类型推导的方法得到的，例如迭代器所指类型作为函数返回类型，所以我们又想了个办法：迭代器声明中内嵌型别，即

```C++
template<typename T>
class MyIter{
    typedef T value_type; //这个不就把迭代器指向的类型内嵌了吗
    //......
}

template<typename T>
typename T::value_type//返回值类型
func(I ite)
{
    return *ite;
}
```

这样就可以直接使用迭代器相关的型别了。

但是还有一个问题，就是不是所有的迭代器都是class type的，如果不是class type，就不能定义内嵌的型别，例如指针就不是class type（但是STL以及整个泛型思维都必须把原生指针作为一种迭代器），这个时候，就要用到偏特化了。

什么是偏特化，就是有一个主模板，然后有其他同名的模板通过直接指定部分模板参数完成偏特化（指定全部参数那叫全特化），具体的偏特化定义与举例可以自行上网查看。

这里我们设计了一个很厉害的东西：iterator_traits，这个类是对迭代器特性的萃取，即把迭代器作为模板类iterator_traits的模板参数，iterator_traits可以萃取出该迭代器的各种特性（注意，这些特性是迭代器自行定义的）。

```c++
template<typename T>
struct iterator_traits
{
    typedef typename T::value_type value_type;
}
```

那这个是如何与偏特化结合解决原生指针的问题呢

```C++
template<typename T>
struct iterator_traits<T*>
{
    typedef typename T value_type;
}
```

或者

```C++
template<typename T>
struct iterator_traits<const T*>
{
    typedef typename T value_type;
}
```

所以对原生指针或指向常数对象的指针我们都需要设计偏特化版本。

即各个容器里的迭代器定义自己的相关类型，然后普通的iterator_traits萃取这些迭代器的特性，两个偏特化的iterator_traits萃取指针相关的特性。

最常用到的迭代器相关型别有五种：value type（迭代器所指对象的类型），difference type（两个迭代器的距离，迭代器的最大容量，总之是个数值型的），pointer（指向迭代器所指之物的指针），reference（迭代器所指之物的引用），iterator category（迭代器的移动特性与施行操作）。

### 1.value type

迭代器所指对象的类型，直接在迭代器内部定义即可，偏特化也比较简单。

### 2.difference type

两个迭代器的距离，迭代器的最大容量，总之是个数值型的，也比较简单，偏特化用的是C++内建的ptrdiff_t。

### 3.reference type

迭代器所指之物的引用

### 4.pointer type

指向迭代器所指之物的指针

### 5.iterator_category

前几个都比较简单，这个就有了复杂了。或者说，前几个都跟迭代器的模板参数有关，这个可能有关吧，但是有五个具体的类型供迭代器选择。

这个iterator_category是按照移动特性和施行操作分成五种（这里面input和output最弱，然后逐渐增强）：

Input iterator：迭代器所指对象只读

Output iterator：只写

Foward iterator：允许“写入型算法”对迭代器所指区间进行读写

Bidirectional iterator：可双向移动

Random Access iterator：包含所有算术能力，如p+n,p-n,p[n]

我们可以把这五种类型分别作为区分重载函数的参数以实现不同的移动和操作。

### 6.消除单纯传递调用的函数

观察书上advance的Forward iterator版本，因为其余input iterator动作相同，所以其直接就在函数体内调用了input iterator，其实这种调用也会消耗一定的资源，能不能消除这种单纯只做了传递调用的函数呢，可以的，只要两个类之间存在继承关系，那么子类在没有设计相应函数的情况下，就可以调用对父类设计的函数。

注意：这里所说的函数不是类的函数，而是把类作为参数的函数。

因为这五种类型间有各自的继承关系，例如Foward iterator继承了input iterator，有时候在两种类型行为一致的情况下就可以不设计Foward的函数了，编译器检测到Foward没有对应的函数，会自动调用input的函数的。
