# 关于AbstractCollection

[toc]

## AbstractCollection的继承关系

![image-20200307195831821](C:\Users\39268\AppData\Roaming\Typora\typora-user-images\image-20200307195831821.png)

## 作用

`AbstractCollection`是`Collection`的直接继承类,它实现了一些常见的操作,也就是将一些通用方法都抽象了出来。且`AbstractList`和`AbstractSet`都直接继承于这个类，用于后续`Hashset`，`ArrayList`的实现。

在源码的注释中有一下的内容

>  To implement an unmodifiable collection, the programmer needs only to extend this class and provide implementations for the **iterator** and **size** methods. (The iterator returned by the **iterator** method must implement **hasNext** and **next**.)
>
> To implement a modifiable collection, the programmer must additionally override this class's **add** method (which otherwise throws an **UnsupportedOperationException**), and the iterator returned by the **iterator** method must additionally implement its **remove** method.

大概意思就是，如果要实现一个不可修改的集合，只需要重写`iterator`和`size`接口就可以，并且返回的`Iterator`需要实现`hasNext`和`next`。而要实现一个可以修改的集合，还必须重写`add`方法（默认会抛出异常），返回的`Iterator`还需要实现`remove`方法。

这就是为什么，在`AbstractCollection`的`add`方法实现是下面这样的了

```java
  public boolean add(E e) {
        throw new UnsupportedOperationException();
    }
```

同时，`AbstractCollection`并没有实现所有的方法，它有两个方法是抽象的

```java
    public abstract Iterator<E> iterator();

    public abstract int size();
```



还有一个细节,代码中为什么`MAX_ARRAY_SIZE`被设为`Integer`的上限`-8`

```java
 private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

根据注释的解释说明,大概意思是有些虚拟机在数组中保留了一些头信息。避免内存溢出

>The maximum size of array to allocate.
>Some VMs reserve some header words in an array.
>Attempts to allocate larger arrays may result in
>OutOfMemoryError: Requested array size exceeds VM limit