# Java List相关我学习疑惑以及问题

[toc]

## iterator的set和add的区别

Set方法：用指定元素替换 next 或 previous 返回的最后一个元素

add方法将指定的元素插入列表，该元素直接插入到 next 返回的元素的后面

```JAVA
List<Integer> list = new ArrayList<>();
        list.add(0);
        list.add(1);
        list.add(2);
        ListIterator it = list.listIterator();
        it.next();
  		//it.add(10);
        //it.set(10);
        System.out.println(list);
```

如果取消注释掉的add方法,结果就是`[0, 10, 1, 2]`

如果取消注释掉的set方法,结果就是`[10, 1, 2]`

##  expectedModCount = modCount操作,关于快速失败和安全失败

https://www.cnblogs.com/hasse/p/5024193.html

https://juejin.im/post/5be62527f265da617369cdc8

## 快速失败机制的一个漏洞

https://www.cnblogs.com/Xieyang-blog/p/9320943.html

简单来说,就是在用迭代器删除一个list的倒数第2个元素的时候,调用list的remove()方法并不会报错,原因在于remove方法修改了size(),导致下一次的hasNext()返回false,使得不会进入checkForComodification()方法,也就不会报错。

## lastRet变量作用

官方解释：最近一次调用next或 previous返回的元素的索引，没找到作用在哪

next方法：修改lastRet

previous方法：修改lastRet

remove方法：重置为-1，且不允许在-1下操作

add方法：重置为-1

set方法：不允许在-1下操作

## protected transient int modCount = 0

这中间有一个transient 关键字,这个关键字	的用处在于**当某个字段被声明为transient后，默认序列化机制就会忽略该字段。**顺带一提,在java序列化的时候,静态变量是不能被序列化的。这里的不能序列化的意思，是序列化信息中不包含这个静态成员域。

这个`modCount `变量代表着当前集合对象的结构性修改的次数，每次进行修改都会进行加1的操作，而`expectedModCount`代表的是迭代器对对象进行结构性修改的次数，这样的话每次进行结构性修改的时候都会将`expectedModCount`和`modCount`进行对比，如果相等的话，说明没有别的迭代器对对对象进行修改。如果不相等，说明发生了并发的操作，就会抛出一个异常。

## 迭代器模式的好处

**迭代器模式是为容器而生。**我们知道，对容器对象的访问必然涉及到遍历算法。你可以一股脑的将遍历方法塞到容器对象中去，或者，根本不去提供什么遍历算法，让使用容器的人自己去实现。这两种情况好像都能够解决问题。然而，对于前一种情况，容器承受了过多的功能，它不仅要负责自己“容器”内的元素维护（增、删、改、查 等），而且还要提供遍历自身的接口；而且最重要的是， **由于遍历状态保存的问题，不能对同一个容器对象同时进行多个遍历,并且还需增加 reset 操作**。**第二种方式倒是省事，却又将容器的内部细节暴露无遗。**

## 为什么AbstractList已经实现了List接口，而Arraylist还要去实现List接口

参考链接：https://stackoverflow.com/questions/4387419/why-does-arraylist-have-implements-list

根据stackflow所说，虽然在实不实现List都能够使得代码正常工作，但是为了方便观察继承结构，帮助理解，所以还是把List接口加上了。

## Arraylist和Vector的区别

1、Vector是线程安全的，ArrayList不是线程安全的。
2、ArrayList在底层数组不够用时在原来的基础上扩展0.5倍，Vector是扩展1倍。

在代码的底层实现中，**只要是关键性的操作，Vector方法前面都加了synchronized关键字，来保证线程的安全性**。

## ArrayList传入集合的有参构造器的问题

```java
public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

其中， c.toArray might (incorrectly) not return Object[] (see 6260652)，意思是存在一个bug，使得c.toArray 返回的可能不是一个Object。

经过实测，也的确是这样的，有问题。

```java
System.out.println(Arrays.asList("element1", "element2").toArray().getClass() == Object[].class);
```

在1.8及以前这是false，但是这个bug已经在1.9的时候已经修复了。

## EMPTY_ELEMENTDATA和DEFAULTCAPACITY_EMPTY_ELEMENTDATA的区别（雾）

EMPTY_ELEMENTDATA在ArrayList中是一个static修饰的字段，然后我们也会发现，ArrayList的构造器总共有种。

其中如果传入了容量，且容量为0，就会把EMPTY_ELEMENTDATA数组的引用赋给当前数组。

如果传入的是一个Collection，并且传入的集合为空，那么也会把EMPTY_ELEMENTDATA数组的引用赋给当前数组。

所以可以看出EMPTY_ELEMENTDATA的最大作用可以理解为减少内存消耗。因为在jdk1.8以前的版本，对于传入容量为0的构造器，内部实现都是new了一个空数组，一定程度上增加了内存的消耗。

---

DEFAULTCAPACITY_EMPTY_ELEMENTDATA则用在了无参构造器上的初始化，又或者是在扩容的时候用于比较。当然这也是一个static字段，也有着减少内存消耗的作用。它与EMPTY_ELEMENTDATA区分开来，也许在某种程度上来说也是为了区分度。

## new 和Clone的区别

1. 使用new操作符创建一个对象

2. 使用clone方法复制一个对象

关于clone的步骤：

分配内存，调用clone方法时，分配的内存和源对象（即调用clone方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域， 填充完成之后，clone方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部。

另外关于深拷贝和浅拷贝，当前clone的对象引用是不同的，属于深拷贝。但是如果对象里面还有子对象，那么子对象是属于浅拷贝，也就是拷贝对象和被拷贝对象中的子对象是同一个引用。

因此，要想实现彻底的深拷贝，需要对对象的引用链上所有子对象都实现Cloneable接口。

## Arrays.copyOf和System.arraycopy的联系

**联系：** 看两者源代码可以发现`copyOf()`内部调用了`System.arraycopy()`方法 **区别：**

1. arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里，而且可以选择拷贝的起点和长度以及放入新数组中的位置
2. copyOf()是系统自动在内部新建一个数组，并返回该数组。

## grow方法不会修改size的值吗

grow又不增加size大小

## batchRemove()方法的一些解答

```java
 private boolean batchRemove(Collection<?> c, boolean complement) {
        final Object[] elementData = this.elementData;
        int r = 0, w = 0;
        boolean modified = false;
        try {
            for (; r < size; r++)
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                modified = true;
            }
        }
        return modified;
    }
```

可以看到,里面有一段(r != size)的判断,就觉得很奇怪。

注释上解释为`保持兼容一致性因为contains是抽象方法，有可能为发生异常`,也就意味着如果发生异常，后面的arraycopy会把异常位置后面的代码全部拷贝到新的数组，不再判断这部分代码的contain()。

可以发现，这里写的是比较严谨的。

## 为什么elementData要修饰为transient

elementData数组相当于容器，当容器不足时就会再扩充容量，但是容器的容量往往都是大于或者等于ArrayList所存元素的个数。
比如，现在实际有了8个元素，那么elementData数组的容量可能是8x1.5=12，如果直接序列化elementData数组，那么就会浪费4个元素的空间，特别是当元素个数非常多时，这种浪费是非常不合算的。所以ArrayList的设计者将elementData设计为transient，然后在writeObject方法中手动将其序列化，并且只序列化了实际存储的那些元素，而不是整个数组。

总的来说就是节省空间。

## readObject和writeobject

```java
 private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

因为elementData是由transient修饰的，也就是不能进行序列化操作，因为极端情况下会很浪费空间。那么我们就需要手动序列化。这就要依靠readObject方法。首先会执行defaultWriteObject方法，用于序列化所有非静态和非transient修饰的字段。然后获得size，之后将长度为size的数组写入，而不是数组的所有长度。

而readObject同理，先获得size，然后只读取size个元素。

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    elementData = EMPTY_ELEMENTDATA;

    // Read in size, and any hidden stuff
    s.defaultReadObject();

    // Read in capacity
    s.readInt(); // ignored

    if (size > 0) {
        // be like clone(), allocate array based upon size not capacity
        int capacity = calculateCapacity(elementData, size);
        SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
        ensureCapacityInternal(size);

        Object[] a = elementData;
        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            a[i] = s.readObject();
        }
    }
}
```



## forEach()方法与forEachRemaining()方法的区别和联系

相似之处：

- 都可以遍历集合
- 都是接口的默认方法
- 都是1.8版本引入的

不同之处：

- forEach()方法位于的是Iterable接口，forEachRemaining()方法位于的是Iterator接口

- 可以看下具体的默认实现

```java
  default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
/*********************************************************************/
  default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

可以看到，作用基本都是遍历集合，对每一个遍历到的元素都调用Consumer的accept方法。

但是，forEachRemaining是通过当前迭代器遍历的，而且遍历到最后之后，再次调用就会没有任何作用。

而forEach方法，是可以多次调用的，而且都是遍历全部内容。

除此之外，forEachRemaining在ArrayList中是被重写了的，这就会导致一个问题,看如下代码

```java
 public static void main(String[] args) {

        ArrayList<String> list = new ArrayList();
        for (int i = 0; i < 10; i++) {
            list.add(String.valueOf(i));
        }

        Iterator iterator = list.iterator();
        iterator.forEachRemaining(new Consumer() {
            @Override
            public void accept(Object o) {
                System.out.println(o);
                if (o.equals("3")) {
                    System.out.println("remove");
                    iterator.remove();
                }
             }
        });
    }
```

如果是默认的代码，这显然没有问题的，但是实际上是有问题的，这段代码会抛出`IllegalStateException`异常，原因在于迭代器的remove方法基于lastRet的值，只有在这个值！=-1 的前提下，才能够正常执行。

而重写过后的forEachRemaining

```java
ublic void forEachRemaining(Consumer<? super E> consumer) {
            Objects.requireNonNull(consumer);
            final int size = ArrayList.this.size;
            int i = cursor;
            if (i >= size) {
                return;
            }
            final Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length) {
                throw new ConcurrentModificationException();
            }
            while (i != size && modCount == expectedModCount) {
                consumer.accept((E) elementData[i++]);
            }
            // update once at end of iteration to reduce heap write traffic
    		// 注意这里
            cursor = i;
            lastRet = i - 1;
            checkForComodification();
        }
```

它是通过for的主动遍历，而不再通过next方法，也就导致没有更新lastRet，所以在forEachRemaining过程中调用remove方法，就会报错。



## 为什么一定要去实现Iterable这个接口呢？为什么不直接实现Iterator接口呢？

​    因为Iterator接口的核心方法next()或者hasNext() 是**依赖于迭代器的当前迭代位置**的。 如果Collection直接实现Iterator接口，势必导致集合对象中包含当前迭代位置的数据(指针)。 当集合在不同方法间被传递时，由于当前迭代位置不可预置，那么next()方法的结果会变成不可预知。 除非再为Iterator接口添加一个reset()方法，用来重置当前迭代位置。 但即时这样，Collection也只能**同时存在一个当前迭代位置**。 而Iterable则不然，每次调用都会返回一个从头开始计数的迭代器。 多个迭代器是互不干扰的。

## spliterator方法

这个方法也是来自于 Collection 接口，ArrayList 对此方法进行了重写。该方法会返回 ListSpliterator 实例，该实例用于遍历和分离容器所存储的元素。

它的主要操作方法有下面三种：

- `tryAdvance` 迭代单个元素，类似于 `iterator.next()`
- `forEachRemaining` 迭代剩余元素
- `trySplit` 将元素切分成两部分并行处理,但需要注意的 Spliterator 并不是线程安全的。

下面是例子：

```java
public static void main(String[] args) {
        ArrayList<Integer> numberss = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5, 6));
        Spliterator<Integer> numbers1 = numberss.spliterator();
        Spliterator<Integer> numbers2 = numberss.spliterator();

        numbers1.forEachRemaining(e -> System.out.println(e + " num1"));//1 2 3 4 5 6
        numbers2.tryAdvance(e -> System.out.println(e + " num2"));//1
        Spliterator<Integer> numbers3 = numbers2.trySplit();//返回的是左半部分,如果是技术,则是较少的
        numbers2.forEachRemaining(e -> System.out.println(e + " num2"));//4 5 6
        numbers3.forEachRemaining(e -> System.out.println(e + " num3"));//2 3

    }
```



## ArrayList的replaceAll方法

源码时这样的

```java
public void replaceAll(UnaryOperator<E> operator) {
        Objects.requireNonNull(operator);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            elementData[i] = operator.apply((E) elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
        modCount++;
    }
```

这个方法传入的是一个第一次见到的参数`UnaryOperator<E> operator`,点击发现父类是java1.8新增加的一个接口

```java
public interface UnaryOperator<T> extends Function<T, T> {
    static <T> UnaryOperator<T> identity() {
        return t -> t;
    }
}
/**************************************************************/

@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}

```

也就是说，想要使用replaceAll方法，只需要传入一个实现了apply方法的类就行了，当然最方便的还是使用lamda表达式。

举个例子：

```java
 public static void main(String[] args) {

        ArrayList<Integer> list = new ArrayList<>();
        for (int i = 1; i <= 10; i++) {
            list.add(i);
        }
        list.replaceAll(x -> x + 1);//里面是lamda表达式
        System.out.println(list);
    }
```

也就是说，这个replaceAll方法作用是能够统一对集合中的元素进行某种操作。

## System.arraycopy方法

这个方法的类型是native的，这就意味着这不是java的方法实现，它调用的本地方法。而且这个放个相对于其他的遍历方式，比如说for，迭代器之类的，速度优势会比较明显。

根据对底层的理解，System.arraycopy是对内存直接进行复制，减少了for循环过程中的寻址时间，从而提高了效能。

**这个方法不是线程安全的。**

## 分析一下removeIf方法

```java
public boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        // figure out which elements are to be removed
        // any exception thrown from the filter predicate at this stage
        // will leave the collection unmodified
        int removeCount = 0;
        final BitSet removeSet = new BitSet(size);
        final int expectedModCount = modCount;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            @SuppressWarnings("unchecked")
            final E element = (E) elementData[i];
            if (filter.test(element)) {
                removeSet.set(i);
                removeCount++;
            }
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }

        // shift surviving elements left over the spaces left by removed elements
        final boolean anyToRemove = removeCount > 0;
        if (anyToRemove) {
            final int newSize = size - removeCount;
            for (int i=0, j=0; (i < size) && (j < newSize); i++, j++) {
                i = removeSet.nextClearBit(i);
                elementData[j] = elementData[i];
            }
            for (int k=newSize; k < size; k++) {
                elementData[k] = null;  // Let gc do its work
            }
            this.size = newSize;
            if (modCount != expectedModCount) {
                throw new ConcurrentModificationException();
            }
            modCount++;
        }

        return anyToRemove;
    }
```

这个方法的作用是如果满足某个条件，那么在这个集合中就会移除这个值。

这个里面出现了一个新的东西BitSet。这个对象在这里的作用主要标记集合的第几个是要被移除的。

还有一个set方法，传入i，标记第i位是true

然后下面有一个方法叫nextClearBit，传入一个i，返回i这个下标（包括）后面的false的是第几位。那么再经过一遍循环，就能够完成目标了。

## 关于ArrayList以及Vector和CopyOnWriteArrayList的联系

参考自：https://juejin.im/post/5aaa2ba8f265da239530b69e

首先讲一下什么是`Copy-On-Write`，顾名思义，在计算机中就是当你想要对一块内存进行修改时，我们不在原有内存块中进行`写`操作，而是将内存拷贝一份，在新的内存中进行`写`操作，`写`完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉嘛！

其中Vector和CopyOnWriteArrayList是线程安全的，ArrayList是线程不安全的。

`CopyOnWriteArrayList`优缺点

缺点：

- 1、耗内存（集合复制）
- 2、实时性不高

优点：

- 1、数据一致性完整，为什么？因为加锁了，并发数据不会乱
- 2、解决了`像ArrayList`、`Vector`这种集合多线程遍历迭代问题，记住，`Vector`虽然线程安全，只不过是加了`synchronized`关键字，迭代问题完全没有解决！

 `CopyOnWriteArrayList`使用场景

- 1、读多写少（白名单，黑名单，商品类目的访问和更新场景），为什么？因为写的时候会复制新集合
- 2、集合不大，为什么？因为写的时候会复制新集合
- 实时性要求不高，为什么，因为有可能会读取到旧的集合数据

## 关于为什么ArrayLiset是线程不安全的解释

https://blog.csdn.net/u012859681/article/details/78206494