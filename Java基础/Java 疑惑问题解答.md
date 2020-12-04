# Java 疑惑问题解答

[toc]

## 一个类所实现的两个不同的接口中，有同名的方法，怎么知道实现的是哪个接口中的方法

答:如果这个类实现了这个同名方法,那么这个方法可以理解为属于这两个接口。

**注意事项：**但是这个两个接口的同名方法的返回值是必须相同的，否则编译器会报错。

同时如果接口会抛异常，那么该类只能抛出两个同名方法异常的交集，且只针对Checked Exception

## Java 数组==运算符的判断原理

```java
		int[] a = {1, 2, 3};
        int[] b = {1, 2, 3};
        System.out.println(a == b);
```

答案是false,所以很明显，还是判断引用。

## GC工作地点

1. 在ArrayList的remove方法

```java
 public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

## 函数式接口的定义

只有**一个抽象方法的接口**

## Java 1.8 新增的特性

1. lamda表达式
2. （1）功能性接口：Function，传入一个值，返回一个值
   （2）断言性接口：Predicate，理解为判断，传入一个值，返回boolean
   （3）供给性接口：Supplier，理解为提供，无参数，但返回一个值
   （4）消费性接口：Consumer，理解为消耗，传入一个值，但返回void
3. 访问局部变量不需要手动添加final参数，但是还是不能修改
4. 接口的静态方法和默认方法
5. 新增并行接口Spliterator

##  Function<T, R>接口解析

```java
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

```

可以看到，这里面有4个方法，1个是抽象的，符号函数式接口的定义。

apply方法，规定传入一个t值，然后返回一个R类型的对象。

compose方法，会先调用before的apply方法，然后将返回值传给自己的apply方法，然后调用这个apply方法

andThen方法，和compose方法相反，先调用自己的apply，然后调用传入的after的apply方法

identity方法，相当于y=x的用法

提供一个理解andThen方法的代码

```java
public static Integer modifyTheValue2(int value, Function<Integer, Integer> function1, Function<Integer, Integer> function2) {
        Function<Integer, Integer> function3 =function1.andThen(function2);
        return function3.apply(value);
    // return function1.andThen(function2).apply(value);//等价于这句话
    }

    public static void main(String[] args) {
        System.out.println(modifyTheValue2(3, val -> {
            System.out.println("f1");
            return val + 2;
        }, val -> {
            System.out.println("f2");
            return val + 3;
        }));
    }
```

一开始对注释的整段不是很理解，但是把这个拆分开来之后，就容易理解多了。

 function1.andThen(function2)返回的同样是一个function类型的，也就是等价于function3，相当于定义了一个新的function，然后传入参数value。

## 关于Java1.8接口新增的static和default

接口中的变量被默认指定为 public static final 类型

接口中的方法则默认是 public abstract 类型的抽象方法



在1.8中，java给接口新增了两个关键字，其中static 方法是可以在接口中内部实现的，但是对于这个方法，只能通过接口名调用，不可以通过实现类的类名或者实现类的对象调用。

另外说明一点，如果定义的是静态

然后是default关键字，同样也是允许实现这个方法，而且这个方法是会被子类继承到的，允许进行重写的操作。只能通过接口实现类的对象来调用。

由于java支持一个实现类可以实现多个接口，如果多个接口中存在同样的static和default方法会怎么样呢？如果有两个接口中的静态方法一模一样，并且一个实现类同时实现了这两个接口，此时并不会产生错误，因为jdk8只能通过接口类调用接口中的静态方法，所以对编译器来说是可以区分的。但是如果两个接口中定义了一模一样的默认方法，并且一个实现类同时实现了这两个接口，那么必须在实现类中重写默认方法，否则编译失败。

## Type接口

来源：https://www.cnblogs.com/baiqiantao/p/7460580.html

- raw type：原始类型，对应Class 。这里的Class不仅仅指平常所指的类，还包括数组、接口、注解、枚举等结构。
- primitive types：基本类型，仍然对应Class
- parameterized types：参数化类型，对应**ParameterizedType**，带有类型参数的类型，即常说的泛型，如：List<T>、Map<Integer, String>、List<? extends Number>。
- type variables：类型变量，对应TypeVariable<D>，如参数化类型中的E、K等类型变量，表示泛指任何类。
- array types：(泛型)数组类型，对应GenericArrayType，比如List<T>[]，T[]这种。注意，这不是我们说的一般数组，而是表示一种【元素类型是参数化类型或者类型变量的】数组类型。

## String的intern方法

```java
public static void main(String[] args) throws Exception {
    String aString = new String("0102");
    String bString = aString.intern();
    String cString = "0102";
    System.out.println(aString == bString);//false
    System.out.println(aString == cString);//false
    System.out.println(bString == cString);//true
}
```

intern方法是一个native类型的,作用是从常量池中寻找所要的String,如果没有,则会将String加入到常量池中。

这也就是为什么第三个返回true，因为cString直接声明字符串是在常量池中，bString调用intern方法也获得了常量池中的“0102”，所以对应地址相等，返回true。

## Java的四种对象引用类型

| 引用类型 | 被垃圾回收时间 | 用途               | 生存时间          |
| -------- | -------------- | ------------------ | ----------------- |
| 强引用   | 从来不会       | 对象的一般状态     | JVM停止运行时终止 |
| 软引用   | 当内存不足时   | 对象缓存           | 内存不足时终止    |
| 弱引用   | 正常垃圾回收时 | 对象缓存           | 垃圾回收后终止    |
| 虚引用   | 正常垃圾回收时 | 跟踪对象的垃圾回收 | 垃圾回收后终止    |

## 	Java的运行时异常和非运行时异常

1. 运行时异常都是RuntimeException类及其子类异常，如NullPointerException、IndexOutOfBoundsException等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

2. 非运行时异常是RuntimeException以外的异常，类型上都属于Exception类及其子类。如IOException、SQLException等以及用户自定义的Exception异常。对于这种异常，JAVA编译器强制要求我们必需对出现的这些异常进行catch并处理，否则程序就不能编译通过。所以，面对这种异常不管我们是否愿意，只能自己去写一大堆catch块去处理可能的异常。

## Java 创建一个对象的方法

1. new
2. 反射
3. clone
4. 反序列化

## Java使用new创建对象和instance创建对象的区别

new的对象在编译环境中要必须在类路径中有，class.forName()在编译时可以不在类路径中，所class.forName()指定了ClassLoader后，就可以在指定的环境中查找某些类，即：new一个对象允许class文件还没加载进来，jvm虚拟机会自动检查内存中是否有这个class对象，若没有就通过类加载器加载进来，而newInstance()必须要确保class文件已经加载进内存中才能产生一个对象，这时需通过class.foName()方法加载class文件。

newInstance()实际上是把new这个方式分解为两步，即首先调用Class加载方法加载某个类，然后实例化。