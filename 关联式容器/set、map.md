# set

## 一.set概述

set的特性是set容器中的所有元素都会根据元素的键值自动排序，且set元素的键值就是实值，即只有一种值，set不允许两个元素有相同的键值。

因为set中的元素都是严格排好序的，所以不允许使用迭代器对set的元素进行更改，set的迭代器是一种constant iterator（使用红黑树的const_iterator实现），红黑树本身可以排序，且set的操作红黑树都有，所以几乎所有的set操作行为都只是调用红黑树的操作行为。

## 二.set具体实现

set的具体实现没什么好说的，都是调用RB-tree的操作。排序也主要是利用RB-tree的内部自有结构。



# map

## 一.map概述

map的特性是所有元素根据键值自动排序，且map的所有元素都是pair，同时拥有实值和键值，pair的第一元素为键值，第二元素为实值，map不允许两个元素拥有相同键值。

我们可以通过map的迭代器更改pair的实值，但是不能更改键值。（迭代器是一个普通的迭代器，不过map定义value_type的时候把pair的first弄成const了）

map的底层依旧由RB-tree实现。

## 二.map具体实现

map的实现基本也是靠调用RB-tree的操作，不过要注意map重载了[]，可以通过[]添加元素，修改实值，查询元素。



# multiset，multimap

这两个和set，map差不多，不过允许键值重复，所以insert的时候用的是RB-tree的insert_equal

