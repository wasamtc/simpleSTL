# heap

## 一.heap概述

heap并不是STL容器组件，那为什么要有它呢，因为随后的priority queue是以heap为底层结构，priority queue就是一个优先队列，即无论按什么样的顺序把任何元素推入优先队列中，取出来的时候一定是先取优先级最大的元素。为了实现这一目标，heap要做的就是把存在其中的元素排好序，如果用list的话排序太慢，如果用binary search tree的话实现太复杂，所以我们这里用complete binary tree，即用二叉完全树作为heap的数据结构，那具体用什么来实现这个二叉完全树呢，可以用array，但是array空间不能拓展，所以用vector。

总而言之，为了排序和效率，使用二叉完全树作为heap的数据结构，使用vector作为存储这棵树的结构，heap要做的就是利用这样的结构在插入删除的时候保持排序（一般是max-tree）。

## 二.heap算法

#### push_heap算法

新加入的元素放在尾端（通过vector的push_back），然后执行一个所谓的上溯程序，新节点依次与父节点比较，如果比父节点大就交换，直到比父节点小或与根节点交换完。

#### pop_heap算法

要取的元素即根节点的元素，为了维护完全二叉树的结构和排序，把要取出的元素存放在尾端（提前把尾端元素拿出来），此时根节点作为一个空洞节点不断与左右子节点中较大的节点交换直到叶节点（此所谓下溯），然后把原来尾端的元素作为此时空洞节点所在节点的元素启动上溯程序，之后返回现在的尾端元素（即原来的根节点元素）即可。

#### sort_heap算法

sort_heap算法就是不断的进行pop_heap操作完成排序，这个算法只需要不断调用pop_heap即可实现，只是要注意每次调用last的值要减1。

#### make_heap算法

这个算法主要用来将现有的一段数据转为heap，根据的原理是隐式表述（即完全二叉树的父子节点在vector或array中的序号有一定关系），具体实现方法是先找到尾端的父节点，然后对这个子树用adjust_heap排序，然后依次对前面的父节点及子树进行排序，直到根节点排完序。

## 三.小结

heap不是容器（更没有迭代器），也不是配接器，准确的说它只是一组算法，这组算法结合vector的函数可以完成排序功能。

heap是以vector为存储基础，以完全二叉树为数据结构，用来排序的一组算法，vector的序号意义不大，只是[0]肯定对应着最大的数。



# priority_queue

## 一.priority概述

priority_queue是一个优先队列，首先它要满足队列的性质，即一端进一端出，先进先出，其次要满足其中的元素是按权值排列的，即每次取值都取出权值最大的，默认情况下priority_queue用max-heap完成。

## 二.priority_queue定义完整列表

priority_queue不是容器，是配接器，以vector为底层容器，函数实现中用了heap的函数，还会接受一个比较标准compare，其中的函数没什么好说的，但是感觉其中的vector的pop_back函数好像是被改了，按道理pop_back应该是弹尾端的，但是队列是一端出一端进，这里好像把pop_back改成首端了 //待完善

## 三.小结

priority_queue是个队列，不提供遍历操作，没有迭代器，底层容器是vector，函数实现和排序借助heap。