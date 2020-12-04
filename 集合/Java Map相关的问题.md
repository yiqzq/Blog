

# HashMap的相关问题

[toc]

## 解决哈希冲突的一些方法

- 开放定址法：
  - 开放定址法就是一旦发生了冲突，就去寻找下一个空的散列地址，只要散列表足够大，空的散列地址总能找到，并将记录存入。
- 链地址法
  - 将哈希表的每个单元作为链表的头结点，所有哈希地址为i的元素构成一个同义词链表。即发生冲突时就把该关键字链在以该单元为头结点的链表的尾部。
- 再哈希法
  - 当哈希地址发生冲突用其他的函数计算另一个哈希函数地址，直到冲突不再产生为止。
- 建立公共溢出区
  - 将哈希表分为基本表和溢出表两部分，发生冲突的元素都放入溢出表中。

## HashMap的简单介绍

HashMap的底层采用的是数组+链表的形式。

简单来说，数组就是hash桶，然后每一个数组都存放一个链表。

然后在jdk1.8中进行了底层的优化，如果hash桶的数量大于64且链表长度大于8，那么就会将链表进行树化，构成一个红黑树，将查询的复杂度从O(n)将为了O(lgn)。

## 为什么不将链表全部换成红黑树

很明显，红黑树从时间复杂度上是要优于链表的，但是为什么没有全部替换成红黑树，原因大概有一下几点：

1. 链表的结构简单，红黑树的结构复杂
2. 在小范围的数据上，红黑树的复杂度并没有优于链表很多
3. 在HashMap扩容的时候，会对红黑树进行拆分和重组，这其中的操作比较耗时。

## 为什么长度一定是2的幂次

在jdk1.7中,计算某个哈希值对应的数组索引采用原理是取模。

举个例子：如果当前计算出的哈希值是9，哈希桶的数量是8，那么9对应的数组下标就是9%8=1

但是，取模的运算速度是很慢的。因此有了一种快速计算索引的方法。那就是9&(8-1)=1。(位运算)

hash%length = hash&(length-1)

就是用位运算代替取模，但是这种方法只有在模数是2的幂的时候才起作用，所以要求容量大小是2的幂次，就是为了加快速度，提高性能。同时在另外一方面，也可以减少冲突的可能性。

## HashMap中的一些类常量介绍

```java
 //默认hash桶初始长度16
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 

  //hash表最大容量2的30次幂
  static final int MAXIMUM_CAPACITY = 1 << 30;

  //默认负载因子 0.75,主要是用于计算阈值
  static final float DEFAULT_LOAD_FACTOR = 0.75f;

  //链表的数量大于等于8个并且桶的数量大于等于64时链表树化 
  static final int TREEIFY_THRESHOLD = 8;

  //hash表某个节点链表的数量小于等于6时树拆分
  static final int UNTREEIFY_THRESHOLD = 6;

  //树化时最小桶的数量
  static final int MIN_TREEIFY_CAPACITY = 64;
```

## 代码问题，关于哈希算法

```java
//通过移位和异或运算，可以让 hash 变得更复杂，进而影响 hash 的分布性。
/*
JDK 1.8 中，是通过 hashCode() 的高 16 位异或低 16 位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度，功效和质量来考虑的，减少系统的开销，也不会造成因为高位没有参与下标的计算，从而引起的碰撞。
*/
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

## 为什么要用异或运算符

保证了对象的 hashCode 的 32 位值只要有一位发生改变，整个 hash() 返回值就会改变。尽可能的减少碰撞。

## 关于comparableClass

For的作用解释

参考https://blog.csdn.net/weixin_42340670/article/details/80673127

```java
/**
* 如果对象x的类是C，如果C实现了Comparable<C>接口，那么返回C，否则返回null
*/
static Class<?> comparableClassFor(Object x) {
    if (x instanceof Comparable) {
        Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
        if ((c = x.getClass()) == String.class) // 如果x是个字符串对象
            return c; // 返回String.class
        /*
         * 为什么如果x是个字符串就直接返回c了呢 ? 因为String  实现了 Comparable 接口，可参考如下String类的定义
         * public final class String implements java.io.Serializable, Comparable<String>, CharSequence
         */ 
 
        // 如果 c 不是字符串类，获取c直接实现的接口（如果是泛型接口则附带泛型信息）    
        if ((ts = c.getGenericInterfaces()) != null) {
            for (int i = 0; i < ts.length; ++i) { // 遍历接口数组
                // 如果当前接口t是个泛型接口 
                // 如果该泛型接口t的原始类型p 是 Comparable 接口
                // 如果该Comparable接口p只定义了一个泛型参数
                // 如果这一个泛型参数的类型就是c，那么返回c
                if (((t = ts[i]) instanceof ParameterizedType) &&
                    ((p = (ParameterizedType)t).getRawType() ==
                        Comparable.class) &&
                    (as = p.getActualTypeArguments()) != null &&
                    as.length == 1 && as[0] == c) // type arg is c
                    return c;
            }
            // 上面for循环的目的就是为了看看x的class是否 implements  Comparable<x的class>
        }
    }
    return null; // 如果c并没有实现 Comparable<c> 那么返回空
}
```
## 关于loadFactor 负载因子大小的调整

在源码种,loadFactor 的默认值是0.75,但是有一个构造器也提供了修改loadFactor的。但是源码中并没有对loadFactor 进行任何的限制，也就是说允许很小，也允许很大。

那么这就可以牵扯到不同的场景，如果loadFactor 很小，那么阈值就会很小，也就是稍微添加几个元素就会触发扩容机制，将容量和阈值扩大两倍，这么做带来的好处在于减小哈希冲突的可能性，但是增大了空间的消耗，是典型的空间换时间。

那么loadFactor 也可以大于1，那么这时候阈值就会大于cap，也就是可以理解就算是把每一个哈希桶装满，也还是能够容纳元素，而不触发扩容机制。也就是典型的时间换空间的做法。

## tableSizeFor方法解析

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

这个方法的作用就是在初始化HashMap的时候，因为要求容量是2的幂次，但是如果你传入的不是一个2的幂次的数，代码底层就会把你传入的那个数字优化为一个2的幂次。

首先是n = cap-1,减一的目的是如果传入的cap恰好为2的幂次，那么通过下面的算法的时候会多扩大一倍，模拟一下就知道效果了。

下面操作的原理大概是这样的，对于一个数，它的二进制最高为为1，那么将这个数右移一位再或起来，那么高两位就一定为1。然后将新得到的数或上右移两位，那么高4为就一定为1。以此类推。

最后得到的就是一个2进制上全为1的数了，最后再加1，那么返回的就是一个2的幂次数。

至于为什么没有 n |= n >>> 32，那是因为容量最大是2的32次。

## 初始化threshold的地方

1. 构造器传入一个初始容量的时候，会设置阈值`this.threshold = tableSizeFor(initialCapacity);`
2. 构造器传入一个Map的时候，会设置阈值`threshold = tableSizeFor(t);`



## 核心方法putVal

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //这就是所谓的延迟创建，在第一次put元素的时候，初始化底层数组
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //如果当前元素对应的数组位置是空的，那么就直接把对应的Node节点加进去
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //发现数组的对应位置有值，但是发现要put的键值对的key和当前位置的相同
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //当前节点是红黑树节点了，那就要调用添加红黑树节点的方法
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //不然就遍历整个链表，看有没有和要put的键值对一样的key
            for (int binCount = 0; ; ++binCount) {
                //发现链表到底了还没有找到一样的key，那就直接添加新的链表节点元素
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //发现链表长度到达阈值了，尝试树化链表
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //链表中找到了一样的key
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //如果上面找到了一的key，那就用传进来的value值更新value属性
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    //长度到达了阈值，扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

## 核心方法resize

首先有一点，1.8中，把初始化的操作延迟到了第一次put的时候。

这个resize方法的作用很多，不仅在每个HashMap首次添加元素的时候会调用，而且在容量超过阈值的时候，都会调用，基本是每次扩大2倍。

### 说明一下构造器

便于解释一下下方的注释

总共有4种，但是大体可以分为3类

```java
public HashMap(int initialCapacity, float loadFactor) {}
public HashMap(int initialCapacity) {}
public HashMap() {}
public HashMap(Map<? extends K, ? extends V> m) {}
```

第一类：带初始容量的，设置了阈值

第二类：无参构造器，什么都没有

第三类：传入集合，既设置阈值，同时也会调用resize()

### e.hash & oldCap

这个表达式的作用是这样的,我们知道oldCap一定是一个2的幂次,所以它的二进制上只有一个1,也就是形如000010000,这种形式。所以e.hash & oldCap这个表达式的结果只可能有2种，要么是0，要么是oldCap，而且从概率上来说，两者的概率应该是五五开的。那么这样子就就可以用于下方链表的分裂操作了，也即是把一个链表拆分成两个链表，拆分的依据就是这两种不同的结果。

### 还有一点，关于怎么扩容

由于容器是2的幂次的，扩容也是每次扩大两倍，那么就会有一个很好的特点。

假设旧容器大小时oldcap，新容器newcap的大小为newcap=oldcap*2

那么，对于old数组下标为index上的内容，扩容到新的数组上就只有2种取值，要么是原位置，要么是原位置+oldcap

举个例子：

如果之前的oldcap是4，那么原来哈希值为1 % 4 = 1，5 %4  = 1，9 % 4 = 1，13 % 4 = 1的就都会映射到index为1的位置。

现在经过扩容之后，newcap = 8，那么1 % 8 = 1，5 % 8 = 5，9 %8  = 1，13 % 8 = 5，结果就很明显了。

### 关于增加链表长度的三步

1. 首先要记录起始节点和尾节点

2. 每当增加一个元素，那么就要更新尾节点.next 属性，同时要将尾节点往后移
3. 所有元素添加完毕之后，将尾节点的next属性置为null，不然就死循环了。

### 源码注释

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    //不是初始化的调用，因为有值
    if (oldCap > 0) {
  		  //这个判断是在容量到达上限的时候
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //正常的2倍扩容
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; 
    }
    //初始化调用，设置新容量为就阈值
    else if (oldThr > 0) 
        newCap = oldThr;
   //初始化调用，且没有设置新阈值，就是无参构造器
    else {           
        //新容量就是默认的16
        newCap = DEFAULT_INITIAL_CAPACITY;
        //新阈就是16*0.75=12
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    //这个就是对另外两类构造器的完善,设置新阈值
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    //给阈值赋值
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //给底层的链表数组赋值
    table = newTab;
    //下面都是针对扩容而言,而不是初始化
    if (oldTab != null) {
        //遍历旧数组
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            //旧数组对应的索引位置有值
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                //如果是单个的，直接赋值
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                //如果是红黑树的节点，那么就进行树的分割，下面会详细讲述
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    //loHead和loTail可以理解为第一个链表的首节点和尾节点
                    //hiHead和hiTail可以理解为第二个链表的首节点和尾节点
                    do {
                        next = e.next;
                        //这就是上面所说的，根据表达式的值，分两类
                        if ((e.hash & oldCap) == 0) {
                            //第一个链表记录首节点
                            if (loTail == null)
                                loHead = e;
                            else
                                //相当于链表长度+1
                                loTail.next = e;
                            //更新尾节点
                            loTail = e;
                        }
                        //下面和上面同理
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    //将第一个链表接到新数组的j位置
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    //将第二个链表接到新数组的j+oldcap位置
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

## treeifyBin(树化)的部分细节

### 如何判断节点在红黑树的左边还是右边

1. 先判断hash值，小的往左，大的往右，相等就看第二步
2. 判断能不能通过comparable接口去判断，如果不能或者还是相等，就第三步
3. 调用tieBreakOrder方法，最后一定能够区分是往左还是往右

### 源码分析

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    //必须满足链表长度大于8（进入该函数已满足），并且哈希桶的数量大于64才能进行树化操作
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    //当前对应下标的数组中有值
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            //将链表节点准换为树节点，其实是一个建立双向链表的过程
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
            //真正的树化
            hd.treeify(tab);
    }
}
```

```java
final void treeify(Node<K,V>[] tab) {
    TreeNode<K,V> root = null;
    //遍历双向链表
    for (TreeNode<K,V> x = this, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;//初始化左右儿子
        //设置根节点
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        }
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            //遍历已经存在的红黑树的结构，判断当前的节点应该插在哪里
            for (TreeNode<K,V> p = root;;) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);
				//dir的过程标题开头已经介绍过了
                TreeNode<K,V> xp = p;
                //插入节点
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }
    /*
    这个方法里做的事情，就是保证树的根节点一定也要成为链表的首节点
    这是为了红黑树和链表互相转化方便，下面就不分析这个方法的源码了	
    */
    moveRootToFront(tab, root);
}
```

可以看一下源码中红黑树平衡是怎么写的

关于红黑树的概念以及理解，可以看我的这篇[博客](https://blog.csdn.net/yiqzq/article/details/104714375)

```java
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                           TreeNode<K,V> x) {
    //插入的节点默认是红色
    x.red = true;
    /*
    	关于这些变量：
    	xp：x节点的父亲节点
    	xpp：xp节点的父亲节点，也就是x节点的祖父节点
    	xppl：x的祖父节点的左儿子
    	xppr：x的祖父节点的右儿子
    */
    for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
		//发现是根节点，设为黑色，返回        
        if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }
        //如果父亲节点是黑色或者祖父为空，其实也就是父亲节点是根节点
        else if (!xp.red || (xpp = xp.parent) == null)
            return root;
        //x的父亲节点是x的祖父节点的左儿子，当然了x的父节点是红色的
        if (xp == (xppl = xpp.left)) {
            //x的叔叔节点存在并且是红色
            if ((xppr = xpp.right) != null && xppr.red) {
                xppr.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            //x的叔叔节点不存在或者是为黑色
            else {
                //x是x的父节点的右儿子
                if (x == xp.right) {
                    //左旋，同时修改x节点为x的父节点（x=xp）
                    root = rotateLeft(root, x = xp);
                    //不知道为什么要再次确认xpp，在我的认知里，应该不会修改xpp的值
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                //修改xp
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        //修改xpp
                        xpp.red = true;
                        //对xpp进行右旋
                        root = rotateRight(root, xpp);
                    }
                }
            }
        }
        //下面和上面是差不多的
        else {
            if (xppl != null && xppl.red) {
                xppl.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            else {
                if (x == xp.left) {
                    root = rotateRight(root, x = xp);
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        root = rotateLeft(root, xpp);
                    }
                }
            }
        }
    }
}
```

## removeNode方法分析

```java
/*
matchValue 如果为true，则当key对应的键值对的值equals(value)为true时才删除；否则不关心value的值
movable 删除后是否移动节点，如果为false，则不移动
*/
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    //数组有值且对应的下标内的元素不为空
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
       //node节点的作用就是获得相匹配的要删除的节点
           if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
           //树节点，一个新的逻辑
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                //遍历链表，找到一个相匹配的节点，p的作用是记录要删除节点的前驱节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
           //红黑树节点删除
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
           //要删除的节点是链表头
            else if (node == p)
                tab[index] = node.next;
            //删除的节点是链表中间的节点，用下面的表达式表示删除
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

## 内部迭代器的分析

```java
//这是抽象类，内部没有实现Next方法，因为HashMap的迭代器可以有多种，遍历key的，value的，entry的，所以交给子类实现
abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot

    HashIterator() {
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        //关键代码，因为底层数组不一定每个下标都有内容，所以通过遍历找到第一个有值的地方
        if (t != null && size > 0) { // advance to first entry
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }
	//关键方法
    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        //要么找到链表的下一个位置，要么是找到下一个数组index，这个是不会用到红黑树节点的，因为没必要
        if ((next = (current = e).next) == null && (t = table) != null) {
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }

    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        //删除节点
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
}
//三种具体的实现类，分别迭代key，value,entry
final class KeyIterator extends HashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}

final class ValueIterator extends HashIterator
    implements Iterator<V> {
    public final V next() { return nextNode().value; }
}

final class EntryIterator extends HashIterator
    implements Iterator<Map.Entry<K,V>> {
    public final Map.Entry<K,V> next() { return nextNode(); }
}
```

## 红黑树什么时候退化

链表长度小与6的时候

## JDK1.7和JDK1.8在HashMap上的区别

1. 1.7使用头插法，1.8使用尾插法
2. 1.7没有红黑树，1.8引入了红黑树

## HashMap在多线程下的问题

在1.7中，由于扩容是头插法，容易造成环链，导致死循环的问题。

1.8中由于改进了扩容的方式，已经没有这个问题了，但是依然会有元素丢失的问题。

**昨天面试官的一个问题，问一个线程put，一个线程get，会有什么问题**

今天仔细想来，主要问题还是在于无法预知是否可以正确读到数据。

如果两个线程put和get不是同一个key，那么就不会存在任何问题。

但是如果是同一个key，那么就会有一个先后问题，如果是先get，那么可能就读不到put的内容，如果是先put，那么get就可以读到内容。

我昨天回答的是说，在扩容的时候会有问题，导致链表一分为二，数据消失，其实不是这个样子的，再次看了一下源码，发现扩容的第一步是先申请新空间，之后的所有操作都会在新空间上进行，那么是不会存在我说的问题的，因为get操作是会先复制数组引用的，这也就导致了即使get被挂起，但是原来的旧数组引用还是存在的，依然能够得到内容。