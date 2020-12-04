# 深入理解Java虚拟机读书笔记(四）

[toc]

**类加载流程**

一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期将会经历加载（Loading）、验证（Verification）、准备（Preparation）、解析（Resolution）、初始化 （Initialization）、使用（Using）和卸载（Unloading）七个阶段，其中验证、准备、解析三个部分统称 为连接（Linking）。

**必须立即初始化的情况**

1. 遇到new、getstatic、putstatic或invokestatic这四条字节码指令时，如果类型没有进行过初始化，则需要先触发其初始化阶段。

   能够生成这四条指令的典型Java代码场景有： 

   使用new关键字实例化对象的时候。 

   读取或设置一个类型的静态字段（**被final修饰、已在编译期把结果放入常量池的静态字段除外**）的时候。 

   调用一个类型的静态方法的时候。 

2. 使用java.lang.reflect包的方法对类型进行反射调用的时候，如果类型没有进行过初始化，则需要先触发其初始化。
3. 当初始化类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。 
4. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。 
5. 当使用JDK 7新加入的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果为REF_getStatic、REF_putStatic、REF_invokeStatic、REF_newInvokeSpecial四种类型的方法句 柄，并且这个方法句柄对应的类没有进行过初始化，则需要先触发其初始化。
6. 当一个接口中定义了JDK 8新加入的默认方法（被default关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。

**关于类初始化和接口初始化的区别**

当一个类在初始化时，要求其父类全部都已经初始化过了，但是一个接口在初始化时，并不要求其父 接口全部都完成了初始化，只有在真正使用到父接口的时候（如引用接口中定义的常量）才会初始化

## 加载

1. 通过一个类的全限定名来获取定义此类的二进制字节流。 

   并**没有指明**二进制字节流必须得从某个Class文件中获取,也就意味着可以从各处获得这个二进制字节流。

   - **从ZIP压缩包中读取**，这很常见，最终成为日后JAR、EAR、WAR格式的基础。 
   - **从网络中获取**，这种场景最典型的应用就是Web Applet。 
   - **运行时计算生成**，这种场景使用得最多的就是动态代理技术，在java.lang.reflect.Proxy中，就是用
     了ProxyGenerator.generateProxyClass()来为特定接口生成形式为“*$Proxy”的代理类的二进制字节流
   - **由其他文件生成**，典型场景是JSP应用，由JSP文件生成对应的Class文件。

2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。 

3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据的访问入口

**关于加载和连接阶段的顺序**

加载阶段与连接阶段的部分动作（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段
尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的一部 分，这两个阶段的开始时间仍然保持着固定的先后顺序

## 验证

由于加载阶段获得的字节流不单单通过java代码编译获得，还可以从别的地方获得，因此为了防止载入对虚拟机有危害的代码，应该对输入的字节流进行验证。

验证阶段大致有四类：**文件格式验证、元数据验证、字节 码验证和符号引用验证**。

## 准备

准备阶段是正式为类中定义的变量（即静态变量，被static修饰的变量）分配内存并设置类变量初始值的阶段，从概念上讲，这些变量所使用的内存都应当在方法区中进行分配，但必须注意到方法区 本身是一个逻辑上的区域，在JDK 7及之前，HotSpot使用永久代来实现方法区时，实现是完全符合这 种逻辑概念的；而在JDK 8及之后，类变量则会随着Class对象一起存放在Java堆中，这时候“类变量在 方法区”就完全是一种对逻辑概念的表述了。

<b><font color=red >这对这段话，解决了我的一个误区，注意：类的元数据！= Class对象</font></b>

<b><font color=red >在jdk8以后，类变量，class对象实际存储在java堆中。在jdk7及以前，采用的是永久代实现方法区，那么类变量和class对象的确是存储在方法区中，但是自从采用了metaspace之后，就改变了存储方式，只有元数据存储在metaspace，其余的都在堆中。</font></b>

不过，为类变量赋初值也是有意外的，如果字段的属性表上存在ConstantValue属性，那么就会在准备阶段赋值（并不是默认值，而是java代码编写的初始化值）

## 解析

解析阶段是Java虚拟机将常量池内的符号引用替换为直接引用的过程

**类或接口的解析**

**字段解析**

**方法解析**

**接口方法解析**

## 初始化

在初始化阶段，则会根据程序员通过程序编码制定的主观计划去初始化类变量和其他资源。我们也可以从另外一种更直接的形式来表 达：初始化阶段就是**执行类构造器\<clinit**>**()方法的过程**

需要注意的是接口和类的区别：

**类的话，在初始化阶段，一定会保证父类的clinit()方法先执行。而接口的话，只有当父类被使用的使用才会被初始化。**

当然还有一点要注意的是clinit方法是线程安全的，这就意味着如果clinit方法调用时间很长，会阻塞其他初始化这个类的线程。

## 类加载器

**启动类加载器**（Bootstrap Class Loader）：前面已经介绍过，这个类加载器负责加载存放在<JAVA_HOME>\lib目录，或者被-Xbootclasspath参数所指定的路径中存放的，而且是Java虚拟机能够 识别的（按照文件名识别，如rt.jar、tools.jar，名字不符合的类库即使放在lib目录中也不会被加载）类 库加载到虚拟机的内存中。采用的是C++语言编写的。

**扩展类加载器**（Extension Class Loader）：这个类加载器是在类sun.misc.Launcher$ExtClassLoader中以Java代码的形式实现的。它负责加载<JAVA_HOME>\lib\ext目录中，或者被java.ext.dirs系统变量所 指定的路径中所有的类库，采用的是Java语言编写。

**应用程序类加载器**（Application Class Loader）：这个类加载器由sun.misc.Launcher$AppClassLoader来实现。由于应用程序类加载器是ClassLoader类中的getSystemClassLoader()方法的返回值，所以有些场合中也称它为“系统类加载器”。它负责加载用户类路径 （ClassPath）上所有的类库，开发者同样可以直接在代码中使用这个类加载器。

**双亲委派模型**

双亲委派模型的工作过程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的 加载请求最终都应该传送到最顶层的启动类加载器中，只有当父加载器反馈自己无法完成这个加载请 求（它的搜索范围中没有找到所需的类）时，子加载器才会尝试自己去完成加载。

**局部变量表**

局部变量表（Local Variables Table）是一组变量值的存储空间，用于存放方法参数和方法内部定义的局部变量。在Java程序被编译为Class文件时，就在方法的Code属性的max_locals数据项中确定了该方 法所需分配的局部变量表的最大容量。

局部变量表的容量以变量槽（Variable Slot）为最小单位，每个变量槽都应该能存放一个boolean、 byte、char、short、int、float、reference或returnAddress类型的数据。

为了尽可能节省栈帧耗用的内存空间，局部变量表中的变量槽是可以重用的。

**操作数栈**

操作数栈（Operand Stack）也常被称为操作栈，它是一个后入先出（Last In First Out，LIFO）栈。同局部变量表一样，操作数栈的最大深度也在编译的时候被写入到Code属性的max_stacks数据项 之中。

**动态连接**

每个栈帧都包含一个指向运行时常量池[1]中该栈帧所属方法的引用，持有这个引用是为了支持方法调用过程中的动态连接

**方法返回地址**

一个方法存在两种返回方式：**正常调用完成，异常调用完成**

## 方法调用

**哪些符号引用在解析阶段会被解析成直接引用：**方法在程序真正运行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期是不 可改变的。换句话说，调用目标在程序代码写好、编译器进行编译那一刻就已经确定下来。

**具体来说，是以下几类**：

只要能被invokestatic和invokespecial指令调用的方法，都可以在解析阶段中确定唯一的调用版本，Java语言里符合这个条件的方法共有静态方法、私有方法、实例构造器、父类方法4种，再加上被final 修饰的方法（尽管它使用invokevirtual指令调用），这5种方法调用会在类加载的时候就可以把符号引 用解析为该方法的直接引用。

**关于分派和解析：**

笔者讲述的解析与分派这两者之间的关系并不是二选一的排他关系，它们是在不同层次上去筛选、确定目标方法的过程。例如前面说过静态方法会在编译期确 定、在类加载期就进行解析，而静态方法显然也是可以拥有重载版本的，选择重载版本的过程也是通 过静态分派完成的

**静态分派**

所有依赖静态类型来决定方法执行版本的分派动作，都称为静态分派。静态分派的最典型应用表现就是方法**重载**。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行 的

```java
public class StaticDispatch {
static abstract class Human { }
static class Man extends Human { }
static class Woman extends Human { }
public void sayHello(Human guy) { System.out.println("hello,guy!");
}
public void sayHello(Man guy) { System.out.println("hello,gentleman!");
}
public void sayHello(Woman guy) { System.out.println("hello,lady!");
}
public static void main(String[] args) { 
    Human man = new Man();
    Human woman = new Woman();
    StaticDispatch sr = new StaticDispatch(); 
    sr.sayHello(man);
    sr.sayHello(woman);
	}
}
/*
结果；
hello,guy!
hello,guy!
原因在于虚拟机（或者准确地说是编译器）在重载时是通过参数的静态类型而不是实际类型作为判定依据的。由于静态类型在编译期可知，所以在编译阶段，Javac编译器就根据参数的静态类型决定了会使用哪个重载版本，因此选择了sayHello(Human)作为调用目标，并把这个方法的符号引用写到 main()方法里的两条invokevirtual指令的参数中。
*/
```

**动态分配**

```java
*/
public class DynamicDispatch {
static abstract class Human {
    protected abstract void sayHello();
}
static class Man extends Human { 
    @Override
    protected void sayHello() {
        System.out.println("man say hello");
    } 
}
static class Woman extends Human { 
    @Override
    protected void sayHello() {
        System.out.println("woman say hello");
    }
}
public static void main(String[] args) {
    Human man = new Man(); 
    Human woman = new Woman();
    man.sayHello(); 
    woman.sayHello();
    man = new Woman();
    man.sayHello();
	}
}
/*
结果：
man say hello
woman say hello 
woman say hello

那看来解决问题的关键还必须从invokevirtual指令本身入手，要弄清 楚它是如何确定调用方法版本、如何实现多态查找来着手分析才行。根据《Java虚拟机规范》 
invokevirtual指令的运行时解析过程大致分为以下几步：
1）找到操作数栈顶的第一个元素所指向的对象的实际类型，记作C。
2）如果在类型C中找到与常量中的描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则返回这个方法的直接引用，查找过程结束；不通过则返回java.lang.IllegalAccessError异常。
3）否则，按照继承关系从下往上依次对C的各个父类进行第二步的搜索和验证过程。
4）如果始终没有找到合适的方法，则抛出java.lang.AbstractMethodError异常

正是因为invokevirtual指令执行的第一步就是在运行期确定接收者的实际类型，所以两次调用中invokevirtual指令并不是把常量池中方法的符号引用解析到直接引用上就结束了，还会根据方法接收者 的实际类型来选择方法版本，这个过程就是Java语言中方法重写的本质。我们把这种在运行期根据实 际类型确定方法执行版本的分派过程称为动态分派。
*/
```

需要注意的是字段没有多态性，因为字段不会调用invokevirtual指令。

**单分派与多分派**

方法的接收者与方法的参数统称为方法的宗量。根据分派基于多少种宗量，可以将分派划分为单分派和多分派两种。单分派是根据一个宗量对 目标方法进行选择，多分派则是根据多于一个宗量对目标方法进行选择。

**Java语言是一门静态多分派、动态单分派的语言。**

