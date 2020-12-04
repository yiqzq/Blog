# Java的lamda表达式和闭包

[toc]

之前只是了解了什么是lamda表达式，并没有仔细地去了解什么是闭包，直到面试遇到了，那就gg了，很尴尬。

这里假设各位都已经知道什么是lamda表达式了

## 先谈一下闭包地概念

> 定义一个闭包的要点如下:
> - 一个依赖于外部环境的`自由变量`的函数.
> - 这个函数能够访问外部环境的`自由变量`.
> - 也就是说,**外部环境持有内部函数所依赖的`自由变量`,由此对内部函数形成了闭包.**

在java中，使用闭包很明显的例子就是使用**内部类+接口**的形式来实现闭包了。

当然，既然内部类可以实现，那么lamda表达式在一定程度上也是能够实现闭包的。

下面是一个很简单的例子

```java
interface Calculator {
    int calculate(int a);
}
public class TestOther {
    public static void main(String[] args) {
        final int y = 2;
		//引用自由变量y
        Calculator cal = (x) -> x + y;
        System.out.println(cal.calculate(5));
    }
}

```

但是java的闭包又被称为伪闭包，其原因在于使用的自由变量必须由final修饰，也就是说不能被修改。在1.8中进行了优化，允许不加final修饰变量，但是不能修改，否则编译报错。

## JVM的实现

可以通过反编译查看字节码文件，下面是上面代码的字节码文件，省去了常量池部分

```java
"C:\Program Files\Java\jdk1.8.0_201\bin\javap.exe" -c -verbose -p TestOther
Classfile /F:/Users/39268/IdeaProjects/NettyTest/target/classes/TestOther.class
  Last modified 2020-4-21; size 1166 bytes
  MD5 checksum 4bbd9afb74558f14958e8ec5e30ac8fa
  Compiled from "TestOther.java"
public class TestOther
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
//常量池
{
  public TestOther();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 5: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   LTestOther;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=3, args_size=1
         0: iconst_2
         1: istore_1
         2: invokedynamic #2,  0              // InvokeDynamic #0:calculate:()LCalculator;
         7: astore_2
         8: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        11: aload_2
        12: iconst_5
        13: invokeinterface #4,  2            // InterfaceMethod Calculator.calculate:(I)I
        18: invokevirtual #5                  // Method java/io/PrintStream.println:(I)V
        21: return
      LineNumberTable:
        line 7: 0
        line 8: 2
        line 9: 8
        line 10: 21
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      22     0  args   [Ljava/lang/String;
            2      20     1     y   I
            8      14     2   cal   LCalculator;

  private static int lambda$main$0(int);
    descriptor: (I)I
    flags: ACC_PRIVATE, ACC_STATIC, ACC_SYNTHETIC
    Code:
      stack=2, locals=1, args_size=1
         0: iload_0
         1: iconst_2
         2: iadd
         3: ireturn
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       4     0     x   I
}
SourceFile: "TestOther.java"
InnerClasses:
     public static final #59= #58 of #62; //Lookup=class java/lang/invoke/MethodHandles$Lookup of class java/lang/invoke/MethodHandles
BootstrapMethods:
  0: #30 invokestatic java/lang/invoke/LambdaMetafactory.metafactory:(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodType;Ljava/lang/invoke/MethodHandle;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;
    Method arguments:
      #31 (I)I
      #32 invokestatic TestOther.lambda$main$0:(I)I
      #31 (I)I

Process finished with exit code 0

```

说回正题，lamda表达式的JVM实现主要依赖于在jdk7中新添加的字节码指令**invokedynamic** 。

**invokedynamic** 这个操作码的执行方法会关联到一个动态调用点对象（Call Site object），这个call site 对象会指向一个具体的bootstrap 方法（方法的二进制字节流信息在BootstrapMethods属性表中）的执行invokedynamic指令的调用会有一个独特的调用链，不像其他四个指令会直接调用方法，在实际的运行过程也相对前四个更加复杂。



**lambda表达式对应一个incokedynamic 指令，通过指令在常量池的符号引用，可以得到BootstrapMethods 属性表对应的引导方法。在运行时，JVM会通过调用这个引导方法生成一个含有MethodHandle（CallSite的target属性）对象的CallSite作为一个Lambda的回调点。Lambda的表达式信息在JVM中通过字节码生成技术转换成一个内部类，这个内部类被绑定到MethodHandle对象中。每次执行lambda的时候，都会找到表达式对应的回调点CallSite执行。**



## 参考内容

[Java中的闭包之争](https://sylvanassun.github.io/2017/07/30/2017-07-30-JavaClosure/)

[Java 8 动态类型语言Lambda表达式实现原理分析](https://blog.csdn.net/raintungli/article/details/54910152)

[通过字节码分析JDK8中Lambda表达式编译及执行机制](https://blog.csdn.net/lijingyao8206/article/details/51225839)