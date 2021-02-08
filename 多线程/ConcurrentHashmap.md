#ConcurrentHashmap1.8

https://www.cnblogs.com/lujiango/p/7580558.html

`sizeCtl`含义解释

> **注意：以上这些构造方法中，都涉及到一个变量`sizeCtl`，这个变量是一个非常重要的变量，而且具有非常丰富的含义，它的值不同，对应的含义也不一样，这里我们先对这个变量不同的值的含义做一下说明，后续源码分析过程中，进一步解释**
>
> `sizeCtl`为0，代表数组未初始化， 且数组的初始容量为16
>
> `sizeCtl`为正数，如果数组未初始化，那么其记录的是数组的初始容量，如果数组已经初始化，那么其记录的是数组的扩容阈值
>
> `sizeCtl`为-1，表示数组正在进行初始化
>
> `sizeCtl`小于0，并且不是-1，表示数组正在扩容， -(1+n)，表示此时有n个线程正在共同完成数组的扩容操作,注意之后怎么变为扩容后的新扩容阈值呢？0.75*2n

##put方法：
步骤：
1、检查Key或者Value是否为null，
2、得到Kye的hash值
3、如果Node数组是空的，此时才初始化 initTable()，
4、如果找的对应的下标的位置为空，直接new一个Node节点并放入， break；
5、
6、如果对应头结点不为空， 进入同步代码块
判断此头结点的hash值，是否大于零，大于零则说明是链表的头结点在链表中寻找，
如果有相同hash值并且key相同，就直接覆盖，返回旧值 结束
如果没有则就直接放置在链表的尾部
此头节点的Hash值小于零，则说明此节点是红黑二叉树的根节点
调用树的添加元素方法
判断当前数组是否要转变为红黑树

## get

首先获取到Key的hash值，
然后找到对应的数组下标处的元素
如果次元素是我们要找的，直接返回，
如果次元素是null 返回null
如果Key的值< 0 ,说明是红黑树，

##ConcurrentHashMap 1.7和1.8的区别

1️⃣整体结构

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190426100401737.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODg0OTc2,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190426101108134.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxODg0OTc2,size_16,color_FFFFFF,t_70)

1.7：Segment + HashEntry + Unsafe
 1.8: 移除 Segment，使锁的粒度更小，Synchronized + CAS + Node + Unsafe

2️⃣put()

1.7：先定位 Segment，再定位桶，put 全程加锁，没有获取锁的线程提前找桶的位置，并最多自旋 64 次获取锁，超过则挂起。
 1.8：由于移除了 Segment，类似 [HashMap](https://www.jianshu.com/p/6c70d265aa7b)，可以直接定位到桶，拿到 first 节点后进行判断：①为空则 [CAS](https://www.jianshu.com/p/98220486426a) 插入；②为 -1 则说明在扩容，则跟着一起扩容；③ else 则加锁 put(类似1.7)

3️⃣get()

基本类似，由于 value 声明为 [volatile](https://www.jianshu.com/p/6c96719bba04)，保证了修改的可见性，因此不需要加锁。

4️⃣resize()

1.7：跟 [HashMap](https://www.jianshu.com/p/6c70d265aa7b) 步骤一样，只不过是搬到单线程中执行，避免了 [HashMap](https://www.jianshu.com/p/6c70d265aa7b) 在 1.7 中扩容时死循环的问题，保证线程安全。
 1.8：支持并发扩容，[HashMap](https://www.jianshu.com/p/6c70d265aa7b) 扩容在1.8中由头插改为尾插(为了避免死循环问题)，ConcurrentHashmap 也是，迁移也是从尾部开始，扩容前在桶的头部放置一个 hash 值为 -1 的节点，这样别的线程访问时就能判断是否该桶已经被其他线程处理过了。

5️⃣size()

1.7：很经典的思路：计算两次，如果不变则返回计算结果，若不一致，则锁住所有的 Segment 求和。
 1.8：用 baseCount 来存储当前的节点个数，这就设计到 baseCount 并发环境下修改的问题。