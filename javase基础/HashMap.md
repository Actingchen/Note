#HashMap

# 【一】从「基础结构」到「1.7 到 1.8 版本变化」
## 1-1 基础结构
hashmap的基础结构当然是散列表啦，也叫哈希表，当然hashmap里面还有一个node结构，其中key字段、value字段、next字段、hash字段
补充一下1.8是叫node，1.7是叫entry。

> 简单理解它的特性为：key与value一一对应，唯一映射。key不可重复、value可以重复。
> hashmap就是采用hash算法来实现唯一映射——存储下标= （n-1）&hash(关键字key) 
> 存储位置：Node<K,V>[存储下标] table（按情况更新key字段、value字段、next字段、hash字段）
> （当然，理论上是无法避免哈希冲突的，即唯一映射，后面会说到）


## 1-2 1.7到1.8的版本变化
|区别|1.7和1.8的版本变化
|--------|:--------|
一|存储结构方面：1.7是数组+链表、1.8是数组+链表+红黑树
二|插值方面：1.7用的是头插法、1.8之后用的是尾插法  
三|计算下标的方式：扩容后数据存储位置的计算方式不一样
这个深究下去说不完，等后续另开一篇，本文只讨论1.8。
# 【二】从「与其他 Map 结构对比和区别」到「HashMap 具体的 put/resize/hash 等的具体过程」

## 2-1 与LinkedHashMap、TreeMap、HashTable区别
斯，偷懒了，这个真不会
> 1、LinkedHashMap内部维持了一个双向列表，可以保持插入顺序。
> 2、TreeMap采用红黑树算法实现，可以保持插入顺序，且支持排序。
> 3、Hashtable与HashMap类似，不同的是：它不允许记录的键或者值为空；它支持线程的同步，即任一时刻只有一个线       程能写Hashtable，因此也导致了Hashtable在写入时会比较慢。
> 4、Hashmap 是一个最常用的Map,它根据键的HashCode值存储数据,根据键可以直接获取它的值，具有很快的访问速度，遍历时，取得数据的顺序是完全随机的。 HashMap最多只允许一条记录的键为Null;允许多条记录的值为 Null;HashMap不支持线程的同步，即任一时刻可以有多个线程同时写HashMap;可能会导致数据的不一致。如果需要同步，可以用 Collections的synchronizedMap方法使HashMap具有同步的能力，或者使用ConcurrentHashMap。
> 一般情况下，我们用的最多的是HashMap,在Map 中插入、删除和定位元素，HashMap 是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。如果需要输出的顺序和输入的相同,那么用LinkedHashMap 可以实现,它还可以按读取顺序来排列.
> ——[HashMap实现原理以及与其他Map实现类的区别](https://blog.csdn.net/wwd0501/article/details/46812637)

## 2-2 HashMap 具体的 put/resize/hash 等的具体过程
### hash操作
直接先上个图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200727212514283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1ZHVoaA==,size_16,color_FFFFFF,t_70)
上源码

```java
static final int hash(Object key) {
    int h;
    //扰动函数：高低位异或
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

```
==key.hashCode()==
CTRL+鼠标点击，发现这里的hashcode是来自Object的方法（Object类提供的保证每个对象的hash码不同，可以复写）。得到的哈希码一般是整性int，2进制32位带符号的范围，也就是说大概有40亿个映射位置。

==hash=(h  ^ (h >>> 16)==

> 扰动函数：右位移16位，正好是32bit的一半，自己的高半区和低半区做异或，就是为了混合原始哈希码的高位和低位，以此来加大低位的随机性。而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来。

大白话理解就是大多数场景下我们的数据一般低16位（近14亿）就已经够用了，它的高16位几乎完全被浪费了。为了更好地保证低位的随机性，所以设计者综合考虑了速度、作用、质量，决定把高16bit和低16bit进行异或运算。
这里设计者用的是异或更能保留原数据的特征，因为&运算会去倾向0，|运算会更倾向于1.

==(n-1)&hash==
此时你可能明白了，前面做的努力都是为了hash算法尽力地去保证散列分布均匀避免哈希冲突并且高效计算，就剩最后一步了，因为我们的数组长度是n，所以我们得把这个存储位置限制在我们的n才行，（n-1)&hash，OK，问题来了计算下标的时候使用&位操作，为什么不是取余？取余不是直接获取后几位吗？
**取余(%)操作中如果除数是 2 的幂次则等价于与其除数减一的与(&)操作（也就是说 hash%length==hash&(length-1)**
> table长度n为2的幂，比如n=16,即n - 1为15(0000000000000000 0000000000001111)，其实散列真正生效的只是低4bit的有效位，也就是特征都集中在低4位了。。


==那为什么table长度n为2的幂呢？==
和上面讲到的有些相辅相成的原因。
嗯。这个和计算存储索引有关。由二进制的特性出发，为了把存储位置锁定在存储范围里面，并且在二进制计算时尽量不去更改hash值，导致增加哈希哈希碰撞的概率。设计者发现用2的幂次方可以有效的不干预低位的二进制，也就是我们要保留的数组范围的二进制，即存储位置。如果不用2的幂次方，在二进制计算的时候你会发现有些操作后，低位时没有变的，无疑增加了哈希冲突的可能。

==为什么加载因子是0.75？==
（1）HashMap默认初始容量大小是16，所以当数组容量达到一定值时会触发扩容，这个值就是16*0.75=12.就会触发扩容机制。
（2）扩容操作本身需要消耗时间，通过由泊松分布和指数分布实验得来在时间和空间上的平衡考虑选择的0.75。

### put操作
1.8是先插入后扩容

——==先插入==——
插入的开始是一样的，都是根据key#hashcode经过高低位异或之后的hash值，然后如果这个散列表是空的就创建（因为hashmap是懒加载模式，第1次调用put函数时，即调用resize() 初始化创建）！
再按位与&（table.length-1)得到一个节点下标，得到一个下标，然后根据这个槽内的状况，状况不同，然后情况也不同
如果节点==null，表示没有哈希冲突就直接插入成功。
如果节点！=null，表示就是哈希冲突，
第一种情况：首先判断如果这个元素的key与要插入的一样，那么就替换一下，也就完事了
第二种情况：然后判断当前node的数据结构是不是红黑树，如果是，就执行putTreeVal方法插入
第三种情况：是里面的node已经链化，表示有链表了，就插入链表，不过后面要判断一下是否要树化操作[ if链表长度是否>8（8 = 桶的树化阈值）]，将链表转换成红黑树。

###**这里为什么是桶内树化阈值=8呢？**
因为经过实验，理想情况下使用随机的哈希码，容器中节点分布在hash桶中的频率遵循泊松分布。
根据泊松分布，在负载因子默认为0.75的时候，按照泊松分布的计算公式计算出单个hash槽内元素个数为8的概率小于百万分之一

>而7作为一个分水岭，等于7的时候不转换，大于等于8的时候才进行转换，小于等于6的时候就退化为链表,为什么7为分水岭，因为防止恶意或者无意中发生频繁的树化链化操作


——==后扩容==——
1.7中是判断当前大小>=阈值 且 准备插入桶发生hash冲突才进行扩容，重新计算插入值的数组下标。1.8是先插入之后全部统一计算，因为已经插入了，所以(++size > 阈值)的话，直接进行扩容，并不会去管是不是这次新插入的值有没有发生过哈希冲突，它看到的是桶数大于阈值（load factor*current capacity）就扩容。


```java
public V put(K key, V value) {
        //1、先计算hash
        return putVal(hash(key), key, value, false, true);
    }


final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //2、若哈希表的数组tab为空，则 通过resize() 创建
        // 所以，初始化哈希表的时机 = 第1次调用put函数时，即调用resize() 初始化创建
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //3、判断是否hash冲突
        //不冲突，就直接插入
        if ((p = tab[i = (n - 1) & hash]) == null)//数组下标计算方式(n - 1) & hash得到索引
            tab[i] = newNode(hash, key, value, null);
        //冲突情况
        else {
            Node<K,V> e; K k;
        //【第一种情况】
        //4、这个元素的key和要插入的一样，直接更新替换一下value的值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
        //【第二种情况】
        //5、判断一下node的数据结构是否是红黑树，如果是就执行putTreeVal方法
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //【第三种情况】
        // 6、若是链表,则在链表中插入 or 更新键值对
        // a、遍历table[i]，判断Key是否已存在：
        
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
        				//新增节点后，需判断是否要树化操作，if链表长度是否>8（8 = 桶的树化阈值）：
        				//若是，则把链表转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) 
                            treeifyBin(tab, hash);
                        break;
                    }
       				//采用equals（） 对比当前遍历节点的key 与 需插入数据的key：
       				//若已存在，则直接用新value 覆盖 旧value
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;//遍历完毕后仍无发现上述情况，则直接在链表尾部插入数据
                    }
            }
        //对第一种情况的后续操作：发现key已存在，直接用新value覆盖旧value并返回旧value
            if (e != null) { // 现有键的映射key已经存在
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
		//7、扩容：桶数大于阈值（load factor*current capacity）就扩容
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
}
```
里面具体的树化操作（treeifyBin）和红黑树插入/更新（putTreeVal）

```java

final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        //如果扩容当前散列表数组tab为空或者当前散列表数组长度<最小树化容量64就只会发生一次resize
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab);
        }
    }
```

```java
/**
     * putTreeVal(this, tab, hash, key, value)
     * 作用：向红黑树插入/更新数据（键值对）
     * 过程：遍历红黑树判断该节点的key是否与需插入的key 相同：
     *      a. 若相同，则新value覆盖旧value
     *      b. 若不相同，则插入
     */

     final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }
        //至于深究左旋右旋，等后续另开一篇
```

### get操作

```java
//1.8
//bucket里的第一个节点，直接命中；
//如果有冲突，则通过key.equals(k)去查找对应的entry
//若为树，则在树中通过key.equals(k)查找，O(logn)；
//若为链表，则在链表中通过key.equals(k)查找，O(n)。 
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    /**
     * 实现Map.get和相关方法
     */
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // 总是检查第一个节点
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

```

### resize操作
大致意思就是，当超过限制的时候会resize，然而又因为我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。
```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
	        // 超过最大值就不再扩充了
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 没超过最大值，就扩充为原来的2倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; 
        }
        else if (oldThr > 0) 
            newCap = oldThr;
        else {               
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 计算新的resize上限
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        if (oldTab != null) {
            // 把每个bucket都移动到新的buckets中
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { 
	                     // 链表优化重hash的代码块
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 原索引
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            // 原索引+oldCap
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                         // 原索引放到bucket里
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        // 原索引+oldCap放到bucket里
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

```
### 红黑树退化成链表

```java
final Node<K,V> untreeify(HashMap<K,V> map) {
            Node<K,V> hd = null, tl = null;
            for (Node<K,V> q = this; q != null; q = q.next) {
                Node<K,V> p = map.replacementNode(q, null);
                if (tl == null)
                    hd = p;
                else
                    tl.next = p;
                tl = p;
            }
            return hd;
        }
```

1、remove时退化

// 在 removeTreeNode的方法中
// 在红黑树的root节点为空 或者root的右节点、root的左节点、root左节点的左节点为空时 说明树都比较小了

```java
if (root == null
		|| (movable
		&& (root.right == null
		|| (rl = root.left) == null
		|| rl.left == null))) {
			tab[index] = first.untreeify(map);  // too small
	return;
}
```

2、在resize中((TreeNode<K,V>)e).split(this, newTab, j, oldCap)时 lc、hc 两个TreeNode 长度小于6时 会退化为树。


```java
if (loHead != null) {
		if (lc <= UNTREEIFY_THRESHOLD)
			tab[index] = loHead.untreeify(map);
		else {
              tab[index] = loHead;
              if (hiHead != null) // (else is already treeified)
              loHead.treeify(tab);
        }
}
if (hiHead != null) {
        if (hc <= UNTREEIFY_THRESHOLD)
             tab[index + bit] = hiHead.untreeify(map);
        else {
             tab[index + bit] = hiHead;
             if (loHead != null)
                   hiHead.treeify(tab);
        }
}
```
### 桶变成红黑树
桶长度大于等于8，链表会变成红黑树。
因为哈希码的频率表遵循泊松分布，桶长度为8 的可能性极其低，几乎不可能。
同时，树节点花费的空间大概是普通节点的两倍。从时间和空间上考虑，所以一开始不使用红黑树。而是桶长度为8时。
>那为什么树退化链表是6呢？因为如果是7那有恶意或偶然插入删除可能出现频繁的链表和树的转换，特别消耗性能，所以7作为一个分水岭，6、8作为关键转变条件。

# 【三】从「为什么 String/Integer 适合做 HashMap 的 Key」到「HashMap 为什么不直接使用 hashCode() 处理后的哈希值直接作为 table 下标」
* 为什么 String/Integer 适合做 HashMap 的 Key？



* 1、因为String/Integer是final类型，保证了hash值的不可变性，所以保证了key的不可更改
* 2、为了成功地在HashMap里面存储、获取对象，作为key的对象必须实现equals、hashCode方法，确保不容易出现hash值计算错误，而String/Integer内部已经重写了equals、hashCode方法。
### HashMap 为什么不直接使用 hashCode() 处理后的哈希值直接作为 table 下标？
* 因为直接使用会导致超出数组长度n。


# 【四】从「什么是哈希，什么是哈希冲突」到「HashMap 如何解决哈希冲突」；
* 哈希的基本概念就是把任意长度的输入通过一个hash算法之后，映射成固定长度的输出。
哈希冲突理论上是无法避免，比如9个抽屉10个苹果，最终一定会有至少一个抽屉的苹果数量是大于等于2 的
* HashMap 如何解决哈希冲突，这个在前面hash操作已经说过了。


# 【五】再从 HashMap 延伸的其他 Java 常用集合
* 简单说一下，ArrayList、Vector、还有LinkedList
* ArrayList的数据结构是数组，内存地址是连续的，它的随机查询相对链表较快捷，因为链表要移动指针，但面对频繁插入删除效率较慢。
LinkenList的数据结构是链表，内存地址是不连续的，由于是节点指针所以频繁随机插入删除效率较快，查询较慢（必须一个节点一个节点寻找）。
* ArrayList它的插入删除主要花费在后移（插入）删除（前移）。LinkenList它的插入删除主要花费在从第一个节点开始遍历。但LinkenList开销主要在new节点
* ArrayList是非线程安全的，Vector是线程安全的。Vector它的各个方法性能较差，很少使用了。

# 扩展
## 为什么HashMap是线程不安全的？

> 多线程put-哈希碰撞问题：如果多个线程同时使用put方法添加元素，而且假设正好存在两个 put 的 key 发生了碰撞(根据 hash 值计算的 bucket 一样)，那么根据 HashMap 的实现，这两个 key 会添加到数组的同一个位置，这样最终就会发生其中一个线程的 put 的数据被覆盖；
> 多线程put-死循环问题：ashMap 在并发执行 put 操作时会引起死循环，导致 CPU 利用率接近100%。因为多线程会导致 HashMap 的 Node 链表形成环形数据结构，一旦形成环形数据结构，Node 的 next 节点永远不为空，就会在获取 Node 时产生死循环。-《Java并发编程的艺术》
> 多线程resize-数据丢失问题：如果多个线程同时检测到元素个数超过table size* loadFactor ，这样就会发生多个线程同时对 Node 数组进行扩容，都在重新计算元素位置以及复制数据，但是最终只有一个线程扩容后的数组会赋给 table，也就是说其他线程的都会丢失，并且各自线程 put 的数据也丢失；
> --[JDK之HashMap原理解析](https://juejin.im/post/5dfde87e6fb9a0162e5df3d7)
> 恍然大悟。。。
>
> 最好的死循环例子就是1.7的头插法导致并发下容易因为链表顺序，生成一个环形链表



# 名词解释
==桶==
英文名就是叫bucket，也有一种说法叫槽。
1.8 是指hashmap的Node<K,V>[] table里的node，1.7又叫entry。
在1.8中，如果发生hash冲突，这个桶还会可能变成链表或者红黑树。

==^ 异或运算==
a^b,双方二进制按照“不同则为1、同则为0”按得出新的二进制。

==& 与运算==
a&b,双方二进制按照“都为1则为1、否则为0”按得出新的二进制。

==| 或运算==
a|b,双方二进制按照“有1则为1、全0则为0”按得出新的二进制。

# 参考文档
[Java源码分析：HashMap 1.8 相对于1.7 到底更新了什么？](https://www.jianshu.com/p/8324a34577a0?utm_source=oschina-app)

[美团面试题：Hashmap的结构，1.7和1.8有哪些区别，史上最深入的分析](https://blog.csdn.net/qq_36520235/article/details/82417949)

[Java中Map接口HashMap与HashTable的区别及HashMap深入理解](https://blog.csdn.net/u012050154/article/details/50905364)

[JDK之HashMap原理解析](https://juejin.im/post/5dfde87e6fb9a0162e5df3d7)

[HashMap实现原理以及与其他Map实现类的区别](https://blog.csdn.net/wwd0501/article/details/46812637)

如果有错误之处，还望指出。