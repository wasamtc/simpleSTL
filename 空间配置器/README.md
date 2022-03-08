# 空间配置器

## 一.空间配置器概述

STL的操作对象都放在容器内，而容器需要一定的配置空间来存放数据资料，而空间配置器就负责给容器分配空间，SGI STL分配的空间是内存（其实基本都是内存吧）。

## 二.空间配置器的标准接口

### 1.必要接口

根据容器的需要和STL的规范，一个空间配置器必须有一些必要接口，如值类型，值指针，size_type，rebind，还有一些函数，如分配空间，构造，取地址，获得最大空间等。

### 2.一个简单的空间配置器

书中提供了一个很简单的空间配置器，这个空间配置器就提供了rebind，分配空间，销毁空间，构造析构，取地址，获得最大空间这些功能，但是具体容器怎么使用，怎么分配空间怎么构造自身，这个还要看后面的知识和容器的源码。

这段代码不难，这里添加一点知识辅助看懂：

[C++的一些基本知识](https://github.com/wasamtc/C-Basics)

## 三.具备次配置力的SGI空间配置器

虽然SGI STL有一个std::allocator，但是效率不咋地，一般不用。

SGI STL真正用的空间配置器主要有以下部分：
<stl_construct.h> ：定义了全局函数中的construct和destroy函数，负责对象的构造和析构，这个文件中的类是符合STL标准规范的。

<stl_alloc.h>：这个文件里面定义了一二级配置器，用于空间的分配和释放。

<stl_uninitialized.h>：定义了一些全局函数，用来填充或复制大块内存数据，符合STL标准规范。

### 1.<stl_construct.h>

这个文件的结构是这样的，首先是构造函数construct，这个设计的很平常就那样，没什么好说的。

然后是析构函数destroy：

destroy设计了两个基本版本，一个是单纯的析构一个对象，另一个是析构两个迭代器中的所有对象，第一个没啥好说的，关键是第二个，因为有的类型的对象析不析构都无所谓（例如有的析构函数啥也没写），在这种无所谓的情况下如果我们析构了很多对象就会降低效率，所以我们用value_type()得到对象的类别，进一步设计_destroy函数，这个函数中用 __type_traits判断该类型的析构是否无关紧要，然后根据得到的类型用 _destroy_aux函数（有两个，函数重载，分别处理两种情况）处理，如果有所谓就一个一个析构，无所谓就空，啥也不干。

还有针对第二个基本版本的destroy的两个特化，char*和 wchar_t *的

### 2.<stl_alloc.h>

为什么SGI STL要采取两级配置呢，主要是考虑到内存碎片（我觉得主要是外部碎片），第一级配置器直接用malloc和free分配和释放内存，第二级配置根据情况采取策略，当要配置的区块超过128bytes时，视之为足够大，用第一级配置器，否则用第二级配置器，至于是开放设计第一级还是同时开放两级，看 _ _USE_MALLOC常量，若为真则只有第一级。（什么叫开放第一级第二级，就是能不能使用的意思，例如SGI STL没有定义 __USE_MALLOC，则内存配置定义在第二级上，这个时候配置空间是先到第二级配置器上，如果不满足第二级的条件再转到第一级配置器上）

之前说过alloc并不符合STL 标准规范，所以我们会为其包装一个simple_alloc的接口，这个接口符合STL规格，只是单纯的转调用而已。

PS：其实我还有一个问题，就是怎么调用那个构造和析构函数的，不过这个可能要看后面的代码才看得出来 //待完善

### 3.第一级配置器剖析

第一级配置器没啥好说的，直接用malloc()和realloc()以及free()函数来进行内存的分配和释放。值得注意的是当内存不够的处理办法，如果我们用的是::operator new和::operator delete的话可以直接用C++的set_new_handler()来设定处理函数，但是我们这里用的是C的malloc和realloc，所以需要自己设计一个处理函数，这里用__malloc_alloc_oom_handler来指向处理函数，用set_malloc_handler来配置函数。

还要注意inst一开始在typedef的时候就已经被设置为0了，即内存不够的处理函数需要客端设定，具体处理时客端要设计的东西，且设计这个函数要特别小心，因为在有处理函数的情况下，我们是通过一个循环不停的用处理函数释放空间，然后再在alloc里面配置空间，如果处理函数有问题，很有可能陷入死循环。

### 4.第二级配置器剖析

如果用malloc分配内存，每块被分配的内存会带有额外信息(操作系统里面应该也讲过)，对于很多小型内存区块来说这种额外信息就是很大的浪费，第二级配置器用freelist来解决这个问题。

若要分配的内存大于128字节，交由第一级配置器处理，若小于等于128字节，将其增大到8的倍数方便分配，先检查freelist有无合适的，若有直接分配更改相应的freelist，若无从内存池中分配出一大块内存一部分返回，另一部分分配给相应的freelist。若有释放的内存，直接插入到相应的freelist即可。

freelist的节点结构很特殊，因为freelist总共有16个，每一个的每个区块大小不同，每一个区块应该有一个节点来指向下一个节点，但是如果我们用单纯的指针的话这个空间就浪费了（即只用作指向下一个节点），所以这里节点用了union类型，即可看作指针，在具体分配出去后又可以存储实际的数据。freelist的接口大致如下：

![Screenshot_20220308_191741_com.newskyer.draw](C:\Users\q1369\Documents\Tencent Files\2930848926\FileRecv\MobileFile\Screenshot_20220308_191741_com.newskyer.draw.png)

（画得有点丑）

然后第二级配置器__default_alloc_template本身代码没啥好说的，释放内存不用说了，分配内存就是找到一个freelist就行，找不到就调用refill，refill找chunk_alloc。

内存池就是本身拿数据，能拿多少拿多少，要是没有就先把剩下的放到合适的freelist去，再从堆里面拿数据，要是堆里面也拿不出数据了（注意这里其实从堆里面要拿的数据是远多于要的数据的），那就看大的freelist里面还有没有东西，有的话就拿出来急用，连大的freelist都没数据了，那就调用第一级配置器吧，看看第一级配置器的处理内存不够的处理机制能不能有点作用。

参考：
[SGI STL中的空间配置器（Allocator）图文详解](https://blog.csdn.net/weixin_44277699/article/details/105799463?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-2.queryctrv4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~BlogCommendFromBaidu~Rate-2.queryctrv4&utm_relevant_index=5)

[STL 空间配置器篇](https://wonanut.blog.csdn.net/article/details/82915529?spm=1001.2101.3001.6650.16&utm_medium=distribute.pc_relevant.none-task-blog-2~default~OPENSEARCH~Rate-16.queryctrv4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~OPENSEARCH~Rate-16.queryctrv4&utm_relevant_index=22)

[STL源码剖析 — 空间配置器(allocator)](https://www.cnblogs.com/mangoyuan/p/6481556.html)

## 四.内存基本处理工具

STL定义有五个全局函数作用在没有初始化的空间上，其中有两个就是我们前面介绍过的构造函数析构函数，容器的思想中会用到它们（好像解决了之前留的那个疑问诶）。

还有另外三个，uninitialized_copy()，uninitialized_fill()，uninitialized_fill_n()，这三个函数对应三个更高阶的算法。

### 1.uninitialized_copy

作用：输入一个区间的某种类型对象，用这些对象初始化(用前面的构造函数)另一个区间的对象。

### 2.uninitialized_fill

作用：输入一个区间的某种类型对象，用一个值初始化这些对象(还是用那个构造函数)

### 3.uninitialized_fill_n

作用：输入一个区间的头指针和区间的对象个数，用一个值初始化这些对象(构造函数)

### 4.三个函数的具体实现

都是先判断是否为POD型别，如果是则调用对应的更高阶算法，如果不是则一个一个构造。

注意：这三个函数都是“commit or rollback”，要么一个都不构造，要么全部构造，如果构造到中途产生异常，则之前构造的必须全部析构掉。还有，对uninitialized_copy有两个特化，这两个特化可以用更快的memmove实现。

## 五.空间配置器小结

SGI STL空间配置器分为一二级结构，主要是解决小额区块的额外负担问题，大于128字节的交由第一级配置器直接调用malloc或free实现，小于等于128字节交由第二级配置器用freelist和memory pool实现，最后还用了simple_alloc进一步封装以符合STL的接口规范。

另外还有五个初始化内存空间的函数，这五个主要用于后面容器的实现中。

（贴代码太麻烦了，就没怎么贴过代码，反正书上也有嘛）
