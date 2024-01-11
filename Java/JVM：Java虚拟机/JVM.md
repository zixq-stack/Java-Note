# 版权声明

> **内容说明：** 本篇内容不属于原创，只是当做笔记的作用，方便需要时查看而已
>
> **原作者：** pdai
>
> **原文链接：** https://www.pdai.tech/md/Java/JVM/Java-JVM-x-overview.html
>
> **额外说明：** 若要用于谋利，请联系原作者





# JVM 基础 - 类字节码详解

> 源代码通过编译器编译为字节码，再通过类加载子系统进行加载到JVM中运行

##  多语言编译为字节码在JVM运行

计算机是不能直接运行Java代码的，必须要先运行Java虚拟机，再由Java虚拟机运行编译后的Java代码，这个编译后的Java代码，就是本文要介绍的Java字节码

为什么JVM不能直接运行Java代码呢，这是因为在CPU层面看来计算机中所有的操作都是一个个指令的运行汇集而成的，Java是高级语言，只有人类才能理解其逻辑，计算机是无法识别的，所以Java代码必须要先编译成字节码文件，JVM才能正确识别代码转换后的指令并将其运行

- Java代码间接翻译成字节码，储存字节码的文件再交由运行于不同平台上的JVM虚拟机去读取执行，从而实现一次编写，到处运行的目的
- JVM也不再只支持Java，由此衍生出了许多基于JVM的编程语言，如Groovy, Scala, Koltin等等

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511122541014-680316648.png)

## Java字节码文件

Class文件本质上是一个以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在Class文件中，JVM根据其特定的规则解析该二进制数据，从而得到相关信息

Class文件采用一种伪结构来存储数据，它有两种类型：无符号数和表，这里暂不详细的讲

本文将通过简单的Java例子编译后的文件来理解

### Class文件的结构属性

在理解之前先从整体看下Java字节码文件包含了哪些类型的数据：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511122605009-2000862489.png)

### 从一个例子开始

下面以一个简单的例子来逐步讲解字节码

```java
// Main.Java
public Class Main {
    
    private int m;
    
    public int inc() {
        return m + 1;
    }
}
```

通过以下命令, 可以在当前所在路径下生成一个Main.Class文件

```bash
Javac Main.Java
```

以文本的形式打开生成的Class文件，内容如下:

```bash
cafe babe 0000 0034 0013 0a00 0400 0f09
0003 0010 0700 1107 0012 0100 016d 0100
0149 0100 063c 696e 6974 3e01 0003 2829
5601 0004 436f 6465 0100 0f4c 696e 654e
756d 6265 7254 6162 6c65 0100 0369 6e63
0100 0328 2949 0100 0a53 6f75 7263 6546
696c 6501 0009 4d61 696e 2e6a 6176 610c
0007 0008 0c00 0500 0601 0010 636f 6d2f
7268 7974 686d 372f 4d61 696e 0100 106a
6176 612f 6c61 6e67 2f4f 626a 6563 7400
2100 0300 0400 0000 0100 0200 0500 0600
0000 0200 0100 0700 0800 0100 0900 0000
1d00 0100 0100 0000 052a b700 01b1 0000
0001 000a 0000 0006 0001 0000 0003 0001
000b 000c 0001 0009 0000 001f 0002 0001
0000 0007 2ab4 0002 0460 ac00 0000 0100
0a00 0000 0600 0100 0000 0800 0100 0d00
0000 0200 0e
```

- 文件开头的4个字节("cafe babe")称之为 `魔数`，唯有以"cafe babe"开头的Class文件方可被虚拟机所接受，这4个字节就是字节码文件的身份识别
- 0000是编译器JDK版本的次版本号0，0034转化为十进制是52，是主版本号，Java的版本号从45开始，除1.0和1.1都是使用45.x外,以后每升一个大版本，版本号加一，也就是说，编译生成该Class文件的JDK版本为1.8.0

通过Java -version命令稍加验证, 可得结果

```java
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```

继续往下是常量池... 知道是这么分析的就可以了，然后我们通过工具反编译字节码文件继续去看

### 反编译字节码文件

> 使用Java内置的一个反编译工具Javap可以反编译字节码文件, 用法: `Javap <options> <Classes>`

其中`<options>`选项包括:

```bash
-help  --help  -?        输出此用法消息
-version                 版本信息
-v  -verbose             输出附加信息
-l                       输出行号和本地变量表
-public                  仅显示公共类和成员
-protected               显示受保护的/公共类和成员
-package                 显示程序包/受保护的/公共类和成员 (默认)
-p  -private             显示所有类和成员
-c                       对代码进行反汇编
-s                       输出内部类型签名
-sysinfo                 显示正在处理的类的系统信息 (路径, 大小, 日期, MD5 散列)
-constants               显示最终常量
-Classpath <path>        指定查找用户类文件的位置
-cp <path>               指定查找用户类文件的位置
-bootClasspath <path>    覆盖引导类文件的位置
```

输入命令`Javap -verbose -p Main.Class`查看输出内容:

```java
Classfile /E:/JavaCode/TestProj/out/production/TestProj/com/rhythm7/Main.Class
  Last modified 2018-4-7; size 362 bytes
  MD5 checksum 4aed8540b098992663b7ba08c65312de
  Compiled from "Main.Java"
public Class com.rhythm7.Main
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#18         // Java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#19         // com/rhythm7/Main.m:I
   #3 = Class              #20            // com/rhythm7/Main
   #4 = Class              #21            // Java/lang/Object
   #5 = Utf8               m
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lcom/rhythm7/Main;
  #14 = Utf8               inc
  #15 = Utf8               ()I
  #16 = Utf8               SourceFile
  #17 = Utf8               Main.Java
  #18 = NameAndType        #7:#8          // "<init>":()V
  #19 = NameAndType        #5:#6          // m:I
  #20 = Utf8               com/rhythm7/Main
  #21 = Utf8               Java/lang/Object
{
  private int m;
    descriptor: I
    flags: ACC_PRIVATE

  public com.rhythm7.Main();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method Java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/rhythm7/Main;

  public int inc();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field m:I
         4: iconst_1
         5: iadd
         6: ireturn
      LineNumberTable:
        line 8: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       7     0  this   Lcom/rhythm7/Main;
}
SourceFile: "Main.Java"
```

### 字节码文件信息

开头的7行信息包括：Class文件当前所在位置，最后修改时间，文件大小，MD5值，编译自哪个文件，类的全限定名，JDK次版本号，主版本号

然后紧接着的是该类的访问标志：ACC_PUBLIC, ACC_SUPER，访问标志的含义如下:

| 标志名称       | 标志值 | 含义                                                         |
| -------------- | ------ | ------------------------------------------------------------ |
| ACC_PUBLIC     | 0x0001 | 是否为Public类型                                             |
| ACC_FINAL      | 0x0010 | 是否被声明为final，只有类可以设置                            |
| ACC_SUPER      | 0x0020 | 是否允许使用invokespecial字节码指令的新语义．                |
| ACC_INTERFACE  | 0x0200 | 标志这是一个接口                                             |
| ACC_ABSTRACT   | 0x0400 | 是否为abstract类型，对于接口或者抽象类来说，次标志值为真，其他类型为假 |
| ACC_SYNTHETIC  | 0x1000 | 标志这个类并非由用户代码产生                                 |
| ACC_ANNOTATION | 0x2000 | 标志这是一个注解                                             |
| ACC_ENUM       | 0x4000 | 标志这是一个枚举                                             |

### 常量池

`Constant pool`意为常量池

常量池可以理解成Class文件中的资源仓库，主要存放的是两大类常量：字面量(Literal)和符号引用(Symbolic References)，字面量类似于Java中的常量概念，如文本字符串，final常量等，而符号引用则属于编译原理方面的概念，包括以下三种:

- 类和接口的全限定名(Fully Qualified Name)
- 字段的名称和描述符号(Descriptor)
- 方法的名称和描述符

不同于C/C++,，JVM是在加载Class文件的时候才进行的动态链接，也就是说这些字段和方法符号引用只有在运行期转换后才能获得真正的内存入口地址，当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建或运行时解析并翻译到具体的内存地址中，直接通过反编译文件来查看字节码内容：

```java
#1 = Methodref          #4.#18         // Java/lang/Object."<init>":()V
#4 = Class              #21            // Java/lang/Object
#7 = Utf8               <init>
#8 = Utf8               ()V
#18 = NameAndType        #7:#8          // "<init>":()V
#21 = Utf8               Java/lang/Object
```

**第一个常量**是一个方法定义，指向了第4和第18个常量。以此类推查看第4和第18个常量最后可以拼接成第一个常量右侧的注释内容:

```java
Java/lang/Object."<init>":()V
```

这段可以理解为该类的实例构造器的声明，由于Main类没有重写构造方法，所以调用的是父类的构造方法。此处也说明了Main类的直接父类是Object，该方法默认返回值是V, 也就是void，无返回值

**第二个常量**同理可得:

```java
#2 = Fieldref           #3.#19         // com/rhythm7/Main.m:I
#3 = Class              #20            // com/rhythm7/Main
#5 = Utf8               m
#6 = Utf8               I
#19 = NameAndType        #5:#6          // m:I
#20 = Utf8               com/rhythm7/Main
```

此处声明了一个字段m，类型为I, I即是int类型。关于字节码的类型对应如下：

| 标识字符 | 含义                                       |
| -------- | ------------------------------------------ |
| B        | 基本类型byte                               |
| C        | 基本类型char                               |
| D        | 基本类型double                             |
| F        | 基本类型float                              |
| I        | 基本类型int                                |
| J        | 基本类型long                               |
| S        | 基本类型short                              |
| Z        | 基本类型boolean                            |
| V        | 特殊类型void                               |
| L        | 对象类型，以分号结尾，如LJava/lang/Object; |

对于数组类型，每一位使用一个前置的`[`字符来描述，如定义一个`Java.lang.String[][]`类型的二维数组，将被记录为`[[LJava/lang/String;`

### 方法表集合

在常量池之后的是对类内部的方法描述，在字节码中以表的集合形式表现，暂且不管字节码文件的16进制文件内容如何，我们直接看反编译后的内容

```java
private int m;
  descriptor: I
  flags: ACC_PRIVATE
```

此处声明了一个私有变量m，类型为int，返回值为int

```java
public com.rhythm7.Main();
   descriptor: ()V
   flags: ACC_PUBLIC
   Code:
     stack=1, locals=1, args_size=1
        0: aload_0
        1: invokespecial #1                  // Method Java/lang/Object."<init>":()V
        4: return
     LineNumberTable:
       line 3: 0
     LocalVariableTable:
       Start  Length  Slot  Name   Signature
           0       5     0  this   Lcom/rhythm7/Main;
```

这里是构造方法：Main()，返回值为void, 公开方法

code内的主要属性为:

- **stack**:：最大操作数栈，JVM运行时会根据这个值来分配栈帧(Frame)中的操作栈深度,此处为1
- **locals**:：局部变量所需的存储空间，单位为Slot, Slot是虚拟机为局部变量分配内存时所使用的最小单位，为4个字节大小，方法参数(包括实例方法中的隐藏参数this)，显示异常处理器的参数(try catch中的catch块所定义的异常)，方法体中定义的局部变量都需要使用局部变量表来存放。值得一提的是，locals的大小并不一定等于所有局部变量所占的Slot之和，因为局部变量中的Slot是可以重用的
- **args_size**:：方法参数的个数，这里是1，因为每个实例方法都会有一个隐藏参数this
- **attribute_info**:：方法体内容，0,1,4为字节码"行号"，该段代码的意思是将第一个引用类型本地变量推送至栈顶，然后执行该类型的实例方法，也就是常量池存放的第一个变量，也就是注释里的`Java/lang/Object."":()V`, 然后执行返回语句，结束方法
- **LineNumberTable**:：该属性的作用是描述源码行号与字节码行号(字节码偏移量)之间的对应关系。可以使用 `-g:none` 或`-g:lines` 选项来取消或要求生成这项信息，如果选择不生成LineNumberTable，当程序运行异常时将无法获取到发生异常的源码行号，也无法按照源码的行数来调试程序
- **LocalVariableTable**：该属性的作用是描述帧栈中局部变量与源码中定义的变量之间的关系。可以使用 `-g:none` 或` -g:vars` 来取消或生成这项信息，如果没有生成这项信息，那么当别人引用这个方法时，将无法获取到参数名称，取而代之的是arg0, arg1这样的占位符 

```txt
start		    表示该局部变量在哪一行开始可见
length		    表示可见行数	start和length也可以称之为变量的作用域。从字节码的start 到 length行这个作用域里该变量一直有效/可见
Slot		    代表所在帧栈位置/变量槽的索引
Name		    变量名称
Signature		类型签名/变量类型		就是前面“常量池”中说的字节码的类型
```

同理可以分析Main类中的另一个方法"inc()":

方法体内的内容是：将this入栈，获取字段#2并置于栈顶, 将int类型的1入栈，将栈内顶部的两个数值相加，返回一个int类型的值

### 类名

最后很显然是源码文件：

```java
SourceFile: "Main.Java"
```

## 再看两个示例

### 分析try-catch-finally

通过以上一个最简单的例子，可以大致了解源码被编译成字节码后是什么样子。下面利用所学的知识点来分析一些Java问题:

```java
public Class TestCode {
    public int foo() {
        int x;
        try {
            x = 1;
            return x;
        } catch (Exception e) {
            x = 2;
            return x;
        } finally {
            x = 3;
        }
    }
}
```

试问当不发生异常和发生异常的情况下，foo()的返回值分别是多少

```java
Javac TestCode.Java
Javap -verbose TestCode.Class
```

查看字节码的foo方法内容:

```java
public int foo();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=5, args_size=1
         0: iconst_1 // int型1入栈 ->栈顶=1
         1: istore_1 // 将栈顶的int型数值存入第二个局部变量 ->局部2=1
         2: iload_1 // 将第二个int型局部变量推送至栈顶 ->栈顶=1
         3: istore_2 // !!将栈顶int型数值存入第三个局部变量 ->局部3=1

         4: iconst_3 // int型3入栈 ->栈顶=3
         5: istore_1 // 将栈顶的int型数值存入第二个局部变量 ->局部2=3
         6: iload_2 // !!将第三个int型局部变量推送至栈顶 ->栈顶=1
         7: ireturn // 从当前方法返回栈顶int数值 ->1

         8: astore_2 // ->局部3=Exception
         9: iconst_2 // ->栈顶=2
        10: istore_1 // ->局部2=2
        11: iload_1 //->栈顶=2
        12: istore_3 //!! ->局部4=2

        13: iconst_3 // ->栈顶=3
        14: istore_1 // ->局部1=3
        15: iload_3 //!! ->栈顶=2
        16: ireturn // -> 2

        17: astore        4 // 将栈顶引用型数值存入第五个局部变量=any
        19: iconst_3 // 将int型数值3入栈 -> 栈顶3
        20: istore_1 // 将栈顶第一个int数值存入第二个局部变量 -> 局部2=3
        21: aload         4 // 将局部第五个局部变量(引用型)推送至栈顶
        23: athrow // 将栈顶的异常抛出
      Exception table:
         from    to  target type
             0     4     8   Class Java/lang/Exception // 0到4行对应的异常，对应#8中储存的异常
             0     4    17   any // Exeption之外的其他异常
             8    13    17   any
            17    19    17   any
```

在字节码的4,5，以及13,14中执行的是同一个操作，就是将int型的3入操作数栈顶，并存入第二个局部变量，这正是我们源码在finally语句块中内容。也就是说，JVM在处理异常时，会在每个可能的分支都将finally语句重复执行一遍

通过一步步分析字节码，可以得出最后的运行结果是：

- 不发生异常时: return 1
- 发生异常时: return 2
- 发生非Exception及其子类的异常，抛出异常，不返回值

> 以上例子来自于《深入理解Java虚拟机 JVM高级特性与最佳实践》, 关于虚拟机字节码指令表，也可以在《深入理解Java虚拟机 JVM高级特性与最佳实践-附录B》中获取

### kotlin 函数扩展的实现

kotlin提供了扩展函数的语言特性，借助这个特性，我们可以给任意对象添加自定义方法

以下示例为Object添加"sayHello"方法

```java
// SayHello.kt
package com.rhythm7

fun Any.sayHello() {
    println("Hello")
}
```

编译后，使用Javap查看生成SayHelloKt.Class文件的字节码

```java
Classfile /E:/JavaCode/TestProj/out/production/TestProj/com/rhythm7/SayHelloKt.Class
Last modified 2018-4-8; size 958 bytes
 MD5 checksum 780a04b75a91be7605cac4655b499f19
 Compiled from "SayHello.kt"
public final Class com.rhythm7.SayHelloKt
 minor version: 0
 major version: 52
 flags: ACC_PUBLIC, ACC_FINAL, ACC_SUPER
Constant pool:
    //省略常量池部分字节码
{
 public static final void sayHello(Java.lang.Object);
   descriptor: (LJava/lang/Object;)V
   flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
   Code:
     stack=2, locals=2, args_size=1
        0: aload_0
        1: ldc           #9                  // String $receiver
        3: invokestatic  #15                 // Method kotlin/JVM/internal/Intrinsics.checkParameterIsNotNull:(LJava/lang/Object;LJava/lang/String;)V
        6: ldc           #17                 // String Hello
        8: astore_1
        9: getstatic     #23                 // Field Java/lang/System.out:LJava/io/PrintStream;
       12: aload_1
       13: invokevirtual #28                 // Method Java/io/PrintStream.println:(LJava/lang/Object;)V
       16: return
     LocalVariableTable:
       Start  Length  Slot  Name   Signature
           0      17     0 $receiver   LJava/lang/Object;
     LineNumberTable:
       line 4: 6
       line 5: 16
   RuntimeInvisibleParameterAnnotations:
     0:
       0: #7()
}
SourceFile: "SayHello.kt"
```

观察头部发现,koltin为文件SayHello生成了一个类，类名"com.rhythm7.SayHelloKt".

由于我们一开始编写SayHello.kt时并不希望SayHello是一个可实例化的对象类，所以，SayHelloKt是无法被实例化的，SayHelloKt并没有任何一个构造器

再观察唯一的一个方法：发现Any.sayHello()的具体实现是静态不可变方法的形式:

```java
public static final void sayHello(Java.lang.Object);
```

所以当我们在其他地方使用Any.sayHello()时，事实上等同于调用Java的SayHelloKt.sayHello(Object)方法

顺便一提的是，当扩展的方法为Any时，意味着Any是non-null的，这时，编译器会在方法体的开头检查参数的非空，即调用 `kotlin.JVM.internal.Intrinsics.checkParameterIsNotNull(Object value, String paramName)` 方法来检查传入的Any类型对象是否为空如果我们扩展的函数为`Any?.sayHello()`，那么在编译后的文件中则不会有这段字节码的出现

## 参考文章

- https://www.cnblogs.com/paddix/p/5282004.html
- https://blog.csdn.net/sinat_37191123/article/details/84582438
- https://blog.csdn.net/tyyj90/article/details/78472986
- https://blog.csdn.net/a15089415104/article/details/83215598
- 咸鱼不思议 https://juejin.im/post/5aca2c366fb9a028c97a5609

------







# JVM 基础 - 字节码的增强技术

## 字节码增强技术

在上文中，着重介绍了字节码的结构，这为我们了解字节码增强技术的实现打下了基础。字节码增强技术就是一类对现有字节码进行修改或者动态生成全新字节码文件的技术。接下来，我们将从最直接操纵字节码的实现方式开始深入进行剖析

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511122938700-1810862780.png)

### ASM

对于需要手动操纵字节码的需求，可以使用ASM，它可以直接生产 .Class字节码文件，也可以在类被加载入JVM之前动态修改类行为（如下图所示）ASM的应用场景有AOP（Cglib就是基于ASM）、热部署、修改其他jar包中的类等。当然，涉及到如此底层的步骤，实现起来也比较麻烦。接下来，本文将介绍ASM的两种API，并用ASM来实现一个比较粗糙的AOP。但在此之前，为了让大家更快地理解ASM的处理流程，强烈建议读者先对访问者模式进行了解。简单来说，访问者模式主要用于修改或操作一些数据结构比较稳定的数据，而通过第一章，我们知道字节码文件的结构是由JVM固定的，所以很适合利用访问者模式对字节码文件进行修改

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511122953285-894496650.png)

#### ASM API

##### 核心API

ASM Core API可以类比解析XML文件中的SAX方式，不需要把这个类的整个结构读取进来，就可以用流式的方法来处理字节码文件，好处是非常节约内存，但是编程难度较大。然而出于性能考虑，一般情况下编程都使用Core AP。I在Core API中有以下几个关键类：

- ClassReader：用于读取已经编译好的.Class文件
- ClassWriter：用于重新构建编译后的类，如修改类名、属性以及方法，也可以生成新的类的字节码文件
- 各种Visitor类：如上所述，Core API根据字节码从上到下依次处理，对于字节码文件中不同的区域有不同的Visitor，比如用于访问方法的MethodVisitor、用于访问类变量的FieldVisitor、用于访问注解的AnnotationVisitor等。为了实现AOP，重点要使用的是MethodVisitor



##### 树形API

ASM Tree API可以类比解析XML文件中的DOM方式，把整个类的结构读取到内存中，缺点是消耗内存多，但是编程比较简单。Tree Api不同于Core API，Tree API通过各种Node类来映射字节码的各个区域，类比DOM节点，就可以很好地理解这种编程方式

#### 直接利用ASM实现AOP

利用ASM的Core API来增强类，这里不纠结于AOP的专业名词如切片、通知，只实现在方法调用前、后增加逻辑，通俗易懂且方便理解，首先定义需要被增强的Base类：其中只包含一个process()方法，方法内输出一行“process”。增强后，我们期望的是，方法执行前输出“start”，之后输出”end”

```java
public Class Base {
    public void process(){
        System.out.println("process");
    }
}
```

为了利用ASM实现AOP，需要定义两个类：一个是MyClassVisitor类，用于对字节码的visit以及修改；另一个是Generator类，在这个类中定义ClassReader和ClassWriter，其中的逻辑是，ClassReader读取字节码，然后交给MyClassVisitor类处理，处理完成后由ClassWriter写字节码并将旧的字节码替换掉，Generator类较简单，我们先看一下它的实现，如下所示，然后重点解释MyClassVisitor类

```java
import org.objectweb.asm.ClassReader;
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.ClassWriter;

public Class Generator {
    public static void main(String[] args) throws Exception {
		// 读取
        ClassReader ClassReader = new ClassReader("meituan/bytecode/asm/Base");
        ClassWriter ClassWriter = new ClassWriter(ClassWriter.COMPUTE_MAXS);
        // 处理
        ClassVisitor ClassVisitor = new MyClassVisitor(ClassWriter);
        ClassReader.accept(ClassVisitor, ClassReader.SKIP_DEBUG);
        byte[] data = ClassWriter.toByteArray();
        // 输出
        File f = new File("operation-server/target/Classes/meituan/bytecode/asm/Base.Class");
        FileOutputStream fout = new FileOutputStream(f);
        fout.write(data);
        fout.close();
        System.out.println("now generator cc success!!!!!");
    }
}
```

MyClassVisitor继承自ClassVisitor，用于对字节码的观察，它还包含一个内部类MyMethodVisitor，继承自MethodVisitor，用于对类内方法的观察，它的整体代码如下：

```java
import org.objectweb.asm.ClassVisitor;
import org.objectweb.asm.MethodVisitor;
import org.objectweb.asm.Opcodes;

public Class MyClassVisitor extends ClassVisitor implements Opcodes {
    
    public MyClassVisitor(ClassVisitor cv) {
        super(ASM5, cv);
    }
    
    @Override
    public void visit(int version, int access, String name, String signature,
                      String superName, String[] interfaces) {
        cv.visit(version, access, name, signature, superName, interfaces);
    }
    
    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        
        MethodVisitor mv = cv.visitMethod(access, name, desc, signature,exceptions);
        // Base类中有两个方法：无参构造以及process方法，这里不增强构造方法
        if (!name.equals("<init>") && mv != null) {
            mv = new MyMethodVisitor(mv);
        }
        return mv;
    }
    
    Class MyMethodVisitor extends MethodVisitor implements Opcodes {
        public MyMethodVisitor(MethodVisitor mv) {
            super(Opcodes.ASM5, mv);
        }

        @Override
        public void visitCode() {
            super.visitCode();
            mv.visitFieldInsn(GETSTATIC, "Java/lang/System", "out", "LJava/io/PrintStream;");
            mv.visitLdcInsn("start");
            mv.visitMethodInsn(INVOKEVIRTUAL, "Java/io/PrintStream", "println", "(LJava/lang/String;)V", false);
        }
        @Override
        public void visitInsn(int opcode) {
            if ((opcode >= Opcodes.IRETURN && opcode <= Opcodes.RETURN)
                    || opcode == Opcodes.ATHROW) {
                // 方法在返回之前，打印"end"
                mv.visitFieldInsn(GETSTATIC, "Java/lang/System", "out", "LJava/io/PrintStream;");
                mv.visitLdcInsn("end");
                mv.visitMethodInsn(INVOKEVIRTUAL, "Java/io/PrintStream", "println", "(LJava/lang/String;)V", false);
            }
            mv.visitInsn(opcode);
        }
    }
}
```

利用这个类就可以实现对字节码的修改详细解读其中的代码，对字节码做修改的步骤是：

- 首先通过MyClassVisitor类中的visitMethod方法，判断当前字节码读到哪一个方法了跳过构造方法 `<init>` 后，将需要被增强的方法交给内部类MyMethodVisitor来进行处理
- 接下来，进入内部类MyMethodVisitor中的visitCode方法，它会在ASM开始访问某一个方法的Code区时被调用，重写visitCode方法，将AOP中的前置逻辑就放在这里，MyMethodVisitor继续读取字节码指令，每当ASM访问到无参数指令时，都会调用MyMethodVisitor中的visitInsn方法，我们判断了当前指令是否为无参数的“return”指令，如果是就在它的前面添加一些指令，也就是将AOP的后置逻辑放在该方法中
- 综上，重写MyMethodVisitor中的两个方法，就可以实现AOP了，而重写方法时就需要用ASM的写法，手动写入或者修改字节码，通过调用methodVisitor的visitXXXXInsn()方法就可以实现字节码的插入，XXXX对应相应的操作码助记符类型，比如mv.visitLdcInsn(“end”)对应的操作码就是ldc “end”，即将字符串“end”压入栈 完成这两个visitor类后，运行Generator中的main方法完成对Base类的字节码增强，增强后的结果可以在编译后的target文件夹中找到Base.Class文件进行查看，可以看到反编译后的代码已经改变了然后写一个测试类MyTest，在其中new Base()，并调用base.process()方法，可以看到下图右侧所示的AOP实现效果：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123049539-1608513746.png)

#### ASM工具

利用ASM手写字节码时，需要利用一系列visitXXXXInsn()方法来写对应的助记符，所以需要先将每一行源代码转化为一个个的助记符，然后通过ASM的语法转换为visitXXXXInsn()，这种写法第一步将源码转化为助记符就已经够麻烦了，不熟悉字节码操作集合的话，需要我们将代码编译后再反编译，才能得到源代码对应的助记符；第二步利用ASM写字节码时，如何传参也很令人头疼，ASM社区也知道这两个问题，所以提供了工具[ASM Byte Code Outline](https://plugins.jetbrains.com/plugin/5918-asm-bytecode-outline)

安装后，右键选择“Show Bytecode Outline”，在新标签页中选择“ASMified”这个tab，如下图所示，就可以看到这个类中的代码对应的ASM写法了，图中上下两个红框分别对应AOP中的前置逻辑与后置逻辑，将这两块直接复制到visitor中的visitMethod()以及visitInsn()方法中，就可以了

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123110707-1707923633.png)

### Javassist

ASM是在指令层次上操作字节码的，阅读上文后，我们的直观感受是在指令层次上操作字节码的框架实现起来比较晦涩，故除此之外，我们再简单介绍另外一类框架：强调源代码层次操作字节码的框架Javassist

利用Javassist实现字节码增强时，可以无须关注字节码刻板的结构，其优点就在于编程简单直接使用Java编码的形式，而不需要了解虚拟机指令，就能动态改变类的结构或者动态生成类，其中最重要的是ClassPool、CtClass、CtMethod、CtField这四个类：

- CtClass（compile-time Class）：编译时类信息，它是一个Class文件在代码中的抽象表现形式，可以通过一个类的全限定名来获取一个CtClass对象，用来表示这个类文件
- ClassPool：从开发视角来看，ClassPool是一张保存CtClass信息的HashTable，key为类名，value为类名对应的CtClass对象，当我们需要对某个类进行修改时，就是通过pool.getCtClass(“ClassName”)方法从pool中获取到相应的CtClass
- CtMethod、CtField：这两个比较好理解，对应的是类中的方法和属性

了解这四个类后，我们可以写一个小Demo来展示Javassist简单、快速的特点，我们依然是对Base中的process()方法做增强，在方法调用前后分别输出”start”和”end”，实现代码如下，我们需要做的就是从pool中获取到相应的CtClass对象和其中的方法，然后执行method.insertBefore和insertAfter方法，参数为要插入的Java代码，再以字符串的形式传入即可，实现起来也极为简单

```java
import com.meituan.mtrace.agent.Javassist.*;

public Class JavassistTest {
    public static void main(String[] args) throws NotFoundException, CannotCompileException, IllegalAccessException, InstantiationException, IOException {
        ClassPool cp = ClassPool.getDefault();
        CtClass cc = cp.get("meituan.bytecode.Javassist.Base");
        CtMethod m = cc.getDeclaredMethod("process");
        m.insertBefore("{ System.out.println(\"start\"); }");
        m.insertAfter("{ System.out.println(\"end\"); }");
        Class c = cc.toClass();
        cc.writeFile("/Users/zen/projects");
        Base h = (Base)c.newInstance();
        h.process();
    }
}
```

## 运行时类的重载

### 问题引出

上一章重点介绍了两种不同类型的字节码操作框架，且都利用它们实现了较为粗糙的AOP。其实，为了方便大家理解字节码增强技术，在上文中我们避重就轻将ASM实现AOP的过程分为了两个main方法：第一个是利用MyClassVisitor对已编译好的Class文件进行修改，第二个是new对象并调用，这期间并不涉及到JVM运行时对类的重加载，而是在第一个main方法中，通过ASM对已编译类的字节码进行替换，在第二个main方法中，直接使用已替换好的新类信息，另外在Javassist的实现中，我们也只加载了一次Base类，也不涉及到运行时重加载类

如果我们在一个JVM中，先加载了一个类，然后又对其进行字节码增强并重新加载会发生什么呢？模拟这种情况，只需要我们在上文中Javassist的Demo中main()方法的第一行添加Base b=new Base()，即在增强前就先让JVM加载Base类，然后在执行到c.toClass()方法时会抛出错误，如下图20所示跟进c.toClass()方法中，我们会发现它是在最后调用了ClassLoader的native方法defineClass()时报错，也就是说，JVM是不允许在运行时动态重载一个类的

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123143951-474784759.png)

显然，如果只能在类加载前对类进行强化，那字节码增强技术的使用场景就变得很窄了。我们期望的效果是：在一个持续运行并已经加载了所有类的JVM中，还能利用字节码增强技术对其中的类行为做替换并重新加载，为了模拟这种情况，我们将Base类做改写，在其中编写main方法，每五秒调用一次process()方法，在process()方法中输出一行“process”

我们的目的就是，在JVM运行中的时候，将process()方法做替换，在其前后分别打印“start”和“end”，也就是在运行中时，每五秒打印的内容由”process”变为打印”start process end”。那如何解决JVM不允许运行时重加载类信息的问题呢？为了达到这个目的，我们接下来一一来介绍需要借助的Java类库

```java
import Java.lang.management.ManagementFactory;

public Class Base {
    public static void main(String[] args) {
        String name = ManagementFactory.getRuntimeMXBean().getName();
        String s = name.split("@")[0];
        // 打印当前Pid
        System.out.println("pid:"+s);
        while (true) {
            try {
                Thread.sleep(5000L);
            } catch (Exception e) {
                break;
            }
            process();
        }
    }

    public static void process() {
        System.out.println("process");
    }
}
```

### Instrument

instrument是JVM提供的一个可以修改已加载类的类库，专门为Java语言编写的插桩服务，提供支持它需要依赖JVMTI的Attach API机制实现，JVMTI这一部分，我们将在下一小节进行介绍，在JDK 1.6以前，instrument只能在JVM刚启动开始加载类时生效，而在JDK 1.6之后，instrument支持了在运行时对类定义的修改，要使用instrument的类修改功能，我们需要实现它提供的ClassFileTransformer接口，定义一个类文件转换器，接口中的transform()方法会在类文件被加载时调用，而在transform方法里，我们可以利用上文中的ASM或Javassist对传入的字节码进行改写或替换，生成新的字节码数组后返回

我们定义一个实现了ClassFileTransformer接口的类TestTransformer，依然在其中利用Javassist对Base类中的process()方法进行增强，在前后分别打印“start”和“end”，代码如下：

```java
import Java.lang.instrument.ClassFileTransformer;

public Class TestTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String ClassName, Class<?> ClassBeingRedefined, ProtectionDomain protectionDomain, byte[] ClassfileBuffer) {
        System.out.println("Transforming " + ClassName);
        try {
            ClassPool cp = ClassPool.getDefault();
            CtClass cc = cp.get("meituan.bytecode.JVMti.Base");
            CtMethod m = cc.getDeclaredMethod("process");
            m.insertBefore("{ System.out.println(\"start\"); }");
            m.insertAfter("{ System.out.println(\"end\"); }");
            return cc.toBytecode();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

现在有了Transformer，那么它要如何注入到正在运行的JVM呢？还需要定义一个Agent，借助Agent的能力将Instrument注入到JVM中，我们将在下一小节介绍Agent，现在要介绍的是Agent中用到的另一个类Instrumentation，在JDK 1.6之后，Instrumentation可以做启动后的Instrument、本地代码（Native Code）的Instrument，以及动态改变Classpath等等。我们可以向Instrumentation中添加上文中定义的Transformer，并指定要被重加载的类，代码如下所示这样，当Agent被Attach到一个JVM中时，就会执行类字节码替换并重载入JVM的操作

```java
import Java.lang.instrument.Instrumentation;

public Class TestAgent {
    public static void agentmain(String args, Instrumentation inst) {
        //指定我们自己定义的Transformer，在其中利用Javassist做字节码替换
        inst.addTransformer(new TestTransformer(), true);
        try {
            //重定义类并载入新的字节码
            inst.retransformClasses(Base.Class);
            System.out.println("Agent Load Done.");
        } catch (Exception e) {
            System.out.println("agent load failed!");
        }
    }
}
```

### JVMTI & Agent & Attach API

上一小节中，我们给出了Agent类的代码，追根溯源需要先介绍JPDA（Java Platform Debugger Architecture）如果JVM启动时开启了JPDA，那么类是允许被重新加载的，在这种情况下，已被加载的旧版本类信息可以被卸载，然后重新加载新版本的类。正如JDPA名称中的Debugger，JDPA其实是一套用于调试Java程序的标准，任何JDK都必须实现该标准

JPDA定义了一整套完整的体系，它将调试体系分为三部分，并规定了三者之间的通信接口，三部分由低到高分别是Java 虚拟机工具接口（JVMTI），Java 调试协议（JDWP）以及 Java 调试接口（JDI），三者之间的关系如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123211335-980002630.png)

现在回到正题，我们可以借助JVMTI的一部分能力，帮助动态重载类信息JVM TI（JVM TOOL INTERFACE，JVM工具接口）是JVM提供的一套对JVM进行操作的工具接口通过JVMTI，可以实现对JVM的多种操作，它通过接口注册各种事件勾子，在JVM事件触发时，同时触发预定义的勾子，以实现对各个JVM事件的响应，事件包括类文件加载、异常产生与捕获、线程启动和结束、进入和退出临界区、成员变量修改、GC开始和结束、方法调用进入和退出、临界区竞争与等待、VM启动与退出等等

而Agent就是JVMTI的一种实现，Agent有两种启动方式，一是随Java进程启动而启动，经常见到的Java -agentlib就是这种方式；二是运行时载入，通过attach API，将模块（jar包）动态地Attach到指定进程id的Java进程内

Attach API 的作用是提供JVM进程间通信的能力，比如说我们为了让另外一个JVM进程把线上服务的线程Dump出来，会运行jstack或jmap的进程，并传递pid的参数，告诉它要对哪个进程进行线程Dump，这就是Attach API做的事情在下面，我们将通过Attach API的loadAgent()方法，将打包好的Agent jar包动态Attach到目标JVM上具体实现起来的步骤如下：

- 定义Agent，并在其中实现AgentMain方法，如上一小节中定义的代码块7中的TestAgent类；
- 然后将TestAgent类打成一个包含MANIFEST.MF的jar包，其中MANIFEST.MF文件中将Agent-Class属性指定为TestAgent的全限定名，如下图所示；

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123224970-1235488534.png)

- 最后利用Attach API，将我们打包好的jar包Attach到指定的JVM pid上，代码如下：

```java
import com.sun.tools.attach.VirtualMachine;

public Class Attacher {
    public static void main(String[] args) throws AttachNotSupportedException, IOException, AgentLoadException, AgentInitializationException {
        // 传入目标 JVM pid
        VirtualMachine vm = VirtualMachine.attach("39333");
        vm.loadAgent("/Users/zen/operation_server_jar/operation-server.jar");
    }
}
```

- 由于在MANIFEST.MF中指定了Agent-Class，所以在Attach后，目标JVM在运行时会走到TestAgent类中定义的agentmain()方法，而在这个方法中，我们利用Instrumentation，将指定类的字节码通过定义的类转化器TestTransformer做了Base类的字节码替换（通过Javassist），并完成了类的重新加载由此，我们达成了“在JVM运行时，改变类的字节码并重新载入类信息”的目的

以下为运行时重新载入类的效果：先运行Base中的main()方法，启动一个JVM，可以在控制台看到每隔五秒输出一次”process”接着执行Attacher中的main()方法，并将上一个JVM的pid传入此时回到上一个main()方法的控制台，可以看到现在每隔五秒输出”process”前后会分别输出”start”和”end”，也就是说完成了运行时的字节码增强，并重新载入了这个类

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123237414-2118602051.png)

### 使用场景

至此，字节码增强技术的可使用范围就不再局限于JVM加载类前了。通过上述几个类库，我们可以在运行时对JVM中的类进行修改并重载了。通过这种手段，可以做的事情就变得很多了：

- 热部署：不部署服务而对线上服务做修改，可以做打点、增加日志等操作
- Mock：测试时候对某些服务做Mock
- 性能诊断工具：比如bTrace就是利用Instrument，实现无侵入地跟踪一个正在运行的JVM，监控到类和方法级别的状态信息

## 总结

字节码增强技术相当于是一把打开运行时JVM的钥匙，利用它可以动态地对运行中的程序做修改，也可以跟踪JVM运行中程序的状态。此外，我们平时使用的动态代理、AOP也与字节码增强密切相关，它们实质上还是利用各种手段生成符合规范的字节码文件。综上所述，掌握字节码增强后可以高效地定位并快速修复一些棘手的问题（如线上性能问题、方法出现不可控的出入参需要紧急加日志等问题），也可以在开发中减少冗余代码，大大提高开发效率

## 参考文献

- 《ASM4-Guide》
- Oracle:The Class File Format
- Oracle:The Java Virtual Machine Instruction Set
- Javassist tutorial
- JVM Tool Interface - Version 1.2







# JVM 基础 - Java 类加载机制

## 类的生命周期

类加载的过程包括了`加载`、`验证`、`准备`、`解析`、`初始化`五个阶段。在这五个阶段中，`加载`、`验证`、`准备`和`初始化`这四个阶段发生的顺序是确定的，*而`解析`阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定(也称为动态绑定或晚期绑定)*。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123402969-231616582.png)

### 类的加载: 查找并加载类的二进制数据

加载是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情:

- 通过一个类的全限定名来获取其定义的二进制字节流
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
- 在Java堆中生成一个代表这个类的Java.lang.Class对象，作为对方法区中这些数据的访问入口

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123418694-159018568.png)



相对于类加载的其他阶段而言，*加载阶段(准确地说，是加载阶段获取类的二进制字节流的动作)是可控性最强的阶段*，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载

加载阶段完成后，虚拟机外部的 二进制字节流就按照虚拟机所需的格式存储在方法区之中，而且在Java堆中也创建一个`Java.lang.Class`类的对象，这样便可以通过该对象访问方法区中的这些数据

类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了.Class文件缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误(LinkageError错误)，如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误

> 加载.Class文件的方式

- 从本地系统中直接加载
- 通过网络下载.Class文件
- 从zip，jar等归档文件中加载.Class文件
- 从专有数据库中提取.Class文件
- 将Java源文件动态编译为.Class文件

### 连接

#### 验证: 确保被加载的类的正确性

验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作:

- `文件格式验证`: 验证字节流是否符合Class文件格式的规范；例如: 是否以`0xCAFEBABE`开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型
- `元数据验证`: 对字节码描述的信息进行语义分析(注意: 对比`Javac`编译阶段的语义分析)，以保证其描述的信息符合Java语言规范的要求；例如: 这个类是否有父类，除了`Java.lang.Object`之外
- `字节码验证`: 通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的
- `符号引用验证`: 确保解析动作能正确执行

> 验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，*如果所引用的类经过反复验证，那么可以考虑采用`-Xverifynone`参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间*

#### 准备: 为类的静态变量分配内存，并将其初始化为默认值

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，**这些内存都将在方法区中分配**，对于该阶段有以下几点需要注意:

- 这时候进行内存分配的仅包括类变量(`static`)，而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中

- 这里所设置的初始值通常情况下是数据类型默认的零值(如`0`、`0L`、`null`、`false`等)，而不是在Java代码中被显式地赋予的值。

	假设一个类变量的定义为: `public static int value = 3`；那么变量value在准备阶段过后的初始值为`0`，而不是`3`，因为这时候尚未开始执行任何Java方法，而把value赋值为3的`put static`指令是在程序编译后，存放于类构造器`<clinit>()`方法之中的，所以把value赋值为3的动作将在初始化阶段才会执行

> 这里还需要注意如下几点

- 对基本数据类型来说，对于类变量(static)和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值，而对于局部变量来说，在使用前必须显式地为其赋值，否则编译时不通过
- 对于同时被`static`和`final`修饰的常量，必须在声明的时候就为其显式地赋值，否则编译时不通过；而只被final修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值
- 对于引用数据类型`reference`来说，如数组引用、对象引用等，如果没有对其进行显式地赋值而直接使用，系统都会为其赋予默认的零值，即`null`
- 如果在数组初始化时没有对数组中的各元素赋值，那么其中的元素将根据对应的数据类型而被赋予默认的零值
- 如果类字段的字段属性表中存在ConstantValue属性，即同时被final和static修饰，那么在准备阶段变量value就会被初始化为ConstValue属性所指定的值，假设上面的类变量value被定义为: ` public static final int value = 3；`编译时Javac将会为value生成ConstantValue属性，在准备阶段虚拟机就会根据ConstantValue的设置将value赋值为3，我们可以理解为`static final`常量在编译期就将其结果放入了调用它的类的常量池中

#### 解析: 把类中的符号引用转换为直接引用

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程，解析动作主要针对`类`或`接口`、`字段`、`类方法`、`接口方法`、`方法类型`、`方法句柄`和`调用点`限定符7类符号引用，进行符号引用就是一组符号来描述目标，可以是任何字面量

`直接引用`就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄

### 初始化

初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化，在Java中对类变量进行初始值设定有两种方式:

- 声明类变量时指定初始值
- 使用静态代码块为类变量指定初始值

**JVM初始化步骤**

- 假如这个类还没有被加载和连接，则程序先加载并连接该类
- 假如该类的直接父类还没有被初始化，则先初始化其直接父类
- 假如类中有初始化语句，则系统依次执行这些初始化语句

**类初始化时机**: 只有当对类的主动使用的时候才会导致类的初始化，类的主动使用包括以下六种:

- 创建类的实例，也就是new的方式。
- 访问某个类或接口的静态变量，或者对该静态变量赋值。注：变量是`final`修饰且等号右边是常量不会触发初始化。
- 调用类的静态方法。
- 反射。如：`Class.forName("com.zixieqing.JVM.Test"))` 。
- 初始化某个类的子类，则其父类也会被初始化。
- Java虚拟机启动时被标明为启动类的类，直接使用`Java.exe`命令来运行某个主类。

### 使用

类访问方法区内的数据结构的接口， 对象是Heap区的数据

### 卸载

**Java虚拟机将结束生命周期的几种情况**

- 执行了System.exit()方法
- 程序正常执行结束
- 程序在执行过程中遇到了异常或错误而异常终止
- 由于操作系统出现错误而导致Java虚拟机进程终止

## 类加载器， JVM类加载机制

### 类加载器的层次

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123513696-2022708891.png)

> 注意: 这里父类加载器并不是通过继承关系来实现的，而是采用组合实现的

> 站在Java虚拟机的角度来讲，只存在两种不同的类加载器: 
>
> 1. 启动类加载器: 它使用C++实现(这里仅限于`Hotspot`，也就是JDK1.5之后默认的虚拟机，有很多其他的虚拟机是用Java语言实现的)，是虚拟机自身的一部分；
> 2. 所有其他的类加载器: 这些类加载器都由Java语言实现，独立于虚拟机之外，并且全部继承自抽象类`Java.lang.ClassLoader`，这些类加载器需要由启动类加载器加载到内存中之后才能去加载其他的类

**站在Java开发人员的角度来看，类加载器可以大致划分为以下三类** :

1. `启动类加载器`： Bootstrap ClassLoader，负责加载存放在 `JDK\jre\lib` (JDK代表JDK的安装目录，下同)下，或被`-XbootClasspath` 参数指定的路径中的，并且能被虚拟机识别的类库(如rt.jar，所有的Java.*开头的类均被Bootstrap ClassLoader加载)，启动类加载器是无法被Java程序直接引用的

2. `扩展类加载器`：Extension ClassLoader，该加载器由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载 `JDK\jre\lib\ext` 目录中，或者由 `Java.ext.dirs` 系统变量指定的路径中的所有类库(如Javax.*开头的类)，开发者可以直接使用扩展类加载器

3. `应用程序类加载器`：Application ClassLoader，该类加载器由`sun.misc.Launcher$AppClassLoader`来实现，它负责加载用户类路径(ClassPath)所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器

应用程序都是由这三种类加载器互相配合进行加载的，如果有必要，我们还可以加入自定义的类加载器。因为JVM自带的ClassLoader只是懂得从本地文件系统加载标准的Java Class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点:

- 在执行非置信代码之前，自动验证数字签名
- 动态地创建符合用户特定需要的定制化构建类
- 从特定的场所取得Java Class，例如数据库中和网络中

### 寻找类加载器

寻找类加载器小例子如下:

```java
package com.pdai.JVM.Classloader;
public Class ClassLoaderTest {
     public static void main(String[] args) {
        ClassLoader loader = Thread.currentThread().getContextClassLoader();
        System.out.println(loader);
        System.out.println(loader.getParent());
        System.out.println(loader.getParent().getParent());
    }
}
```

结果如下:

```java
sun.misc.Launcher$AppClassLoader@64fef26a
sun.misc.Launcher$ExtClassLoader@1ddd40f3
null
```

从上面的结果可以看出，并没有获取到`ExtClassLoader`的父Loader，原因是`BootstrapLoader`(引导类加载器)是用C语言实现的，找不到一个确定的返回父Loader的方式，于是就返回`null`

### 类的加载

类加载有三种方式:

1. 命令行启动应用时由JVM初始化加载

2. 通过 `Class.forName()` 方法动态加载

3. 通过 `ClassLoader.loadClass()` 方法动态加载

```java
public Class loaderTest { 
        public static void main(String[] args) throws ClassNotFoundException { 
                ClassLoader loader = HelloWorld.Class.getClassLoader(); 
                System.out.println(loader); 
                // 使用ClassLoader.loadClass()来加载类，不会执行初始化块 
                loader.loadClass("Test2"); 
                // 使用Class.forName()来加载类，默认会执行初始化块 
			   // Class.forName("Test2"); 
                // 使用Class.forName()来加载类，并指定ClassLoader，初始化时不执行静态块 
			   // Class.forName("Test2", false, loader); 
        } 
}

public Class Test2 { 
        static { 
                System.out.println("静态初始化块执行了！"); 
        } 
}
```

分别切换加载方式，会有不同的输出结果

> Class.forName()和ClassLoader.loadClass()区别?

- Class.forName(): 将类的.Class文件加载到JVM中之外，还会对类进行解释，执行类中的static块；
- ClassLoader.loadClass(): 只干一件事情，就是将.Class文件加载到JVM中，不会执行static中的内容,只有在newInstance才会去执行static块
- Class.forName(name, initialize, loader)带参函数也可控制是否加载static块并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 

## JVM类加载机制

- `全盘负责`，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
- `父类委托`，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
- `缓存机制`，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效
- `双亲委派机制`, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类

**双亲委派机制过程？**

1. 当AppClassLoader加载一个Class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成
2. 当ExtClassLoader加载一个Class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成
3. 如果BootStrapClassLoader加载失败(例如在$Java_HOME/jre/lib里未查找到该Class)，会使用ExtClassLoader来尝试加载；
4. 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException

**双亲委派代码实现**

```java
public Class<?> loadClass(String name)throws ClassNotFoundException {
            return loadClass(name, false);
    }
    protected synchronized Class<?> loadClass(String name, boolean resolve)throws ClassNotFoundException {
            // 首先判断该类型是否已经被加载
            Class c = findLoadedClass(name);
            if (c == null) {
                //如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
                try {
                    if (parent != null) {
                         //如果存在父类加载器，就委派给父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {
                    //如果不存在父类加载器，就检查是否是由启动类加载器加载的类，通过调用本地方法native Class findBootstrapClass(String name)
                        c = findBootstrapClass0(name);
                    }
                } catch (ClassNotFoundException e) {
                 // 如果父类加载器和启动类加载器都不能完成加载任务，才调用自身的加载功能
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
```

**双亲委派优势**

- 系统类防止内存中出现多份同样的字节码
- 保证Java程序安全稳定运行

## 自定义类加载器

通常情况下，我们都是直接使用系统类加载器，但是，有的时候，我们也需要自定义类加载器，比如应用是通过网络来传输 Java 类的字节码，为保证安全性，这些字节码经过了加密处理，这时系统类加载器就无法对其进行加载，这样则需要自定义类加载器来实现。自定义类加载器一般都是继承自 ClassLoader 类，从上面对 loadClass 方法来分析来看，我们只需要重写 findClass 方法即可。下面我们通过一个示例来演示自定义类加载器的流程:

```java
import Java.io.*;

public Class MyClassLoader extends ClassLoader {

    private String root;

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] ClassData = loadClassData(name);
        if (ClassData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, ClassData, 0, ClassData.length);
        }
    }

    private byte[] loadClassData(String ClassName) {
        String fileName = root + File.separatorChar + ClassName.replace('.', File.separatorChar) + ".Class";
        try {
            InputStream ins = new FileInputStream(fileName);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length = 0;
            while ((length = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, length);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args)  {

        MyClassLoader ClassLoader = new MyClassLoader();
        ClassLoader.setRoot("D:\\temp");

        Class<?> testClass = null;
        try {
            testClass = ClassLoader.loadClass("com.pdai.JVM.Classloader.Test2");
            Object object = testClass.newInstance();
            System.out.println(object.getClass().getClassLoader());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```

自定义类加载器的核心在于对字节码文件的获取，如果是加密的字节码则需要在该类中对文件进行解密。由于这里只是演示，我并未对Class文件进行加密，因此没有解密的过程

**这里有几点需要注意** :

1. 这里传递的文件名需要是类的全限定性名称，即`com.pdai.JVM.Classloader.Test2`格式的，因为 defineClass 方法是按这种格式进行处理的

2. 最好不要重写loadClass方法，因为这样容易破坏双亲委托模式

3. Test 类本身可以被 AppClassLoader 类加载，因此我们不能把com/pdai/JVM/Classloader/Test2.Class 放在类路径下，否则，由于双亲委托机制的存在，会直接导致该类由 AppClassLoader 加载，而不会通过我们自定义类加载器来加载

## 参考文章

- http://www.cnblogs.com/ityouknow/p/5603287.html
- http://blog.csdn.net/ns_code/article/details/17881581
- https://segmentfault.com/a/1190000005608960
- http://www.importnew.com/18548.html
- http://zyjustin9.iteye.com/blog/2092131
- http://www.codeceo.com/article/Java-Class-loader-learn.html









# JVM 基础 - JVM 内存结构

> 本文主要对JVM 内存结构进行讲解，注意不要和Java内存模型混淆了。原先这里放了一篇我自己整理的文章，最近看到**海星的Javakeeper公众号**整理的文章，整理的很好，所以替换为他的文章，以方便你构筑JVM内存结构的知识体系

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123624719-788813842.jpg)



## 运行时数据区

内存是非常重要的系统资源，是硬盘和 CPU 的中间仓库及桥梁，承载着操作系统和应用程序的实时运行。JVM 内存布局规定了 Java 在运行过程中内存申请、分配、管理的策略，保证了 JVM 的高效稳定。运行不同的 JVM 对于内存的划分方式和管理机制存在着部分差异

下图是 JVM 整体架构，中间部分就是 Java 虚拟机定义的各种运行时数据区域

![JVM-framework](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123647387-1705367970.jpg)



Java 虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁，另外一些则是与线程一一对应的，这些与线程一一对应的数据区域会随着线程开始和结束而创建和销毁

- **线程私有**：程序计数器、虚拟机栈、本地方法区
- **线程共享**：堆、方法区, 堆外内存（Java7的永久代或JDK8的元空间、代码缓存）

> 下面我们就来一一解读下这些内存区域，先从最简单的入手

## 程序计数器

程序计数寄存器（**Program Counter Register**），Register 的命名源于 CPU 的寄存器，寄存器存储指令相关的线程信息，CPU 只有把数据装载到寄存器才能够运行

这里，并非是广义上所指的物理寄存器，叫程序计数器（或PC计数器或指令计数器）会更加贴切，并且也不容易引起一些不必要的误会。**JVM 中的 PC 寄存器是对物理 PC 寄存器的一种抽象模拟**

程序计数器是一块较小的内存空间，可以看作是**当前线程所执行的字节码的行号指示器**

### 作用

PC 寄存器用来存储指向下一条指令的地址，即将要执行的指令代码由执行引擎读取下一条指令

![JVM-pc-counter](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123710481-1114142040.jpg)



（分析：进入Class文件所在目录，执行 `Javap -v xx.Class` 反解析（或者通过 IDEA 插件 `JClasslib` 直接查看，上图），可以看到当前类对应的Code区（汇编指令）、本地变量表、异常表和代码行偏移量映射表、常量池等信息）

关于类的字节码相关可以看：JVM基础 - 类字节码详解

### 概述

> 通过下面两个问题，理解下PC计数器

1. **使用PC寄存器存储字节码指令地址有什么用呢？为什么使用PC寄存器记录当前线程的执行地址呢？**

因为CPU需要不停的切换各个线程，这时候切换回来以后，就得知道接着从哪开始继续执行，JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令

2. **PC寄存器为什么会被设定为线程私有的？**

多线程在一个特定的时间段内只会执行其中某一个线程方法，CPU会不停的做任务切换，这样必然会导致经常中断或恢复。为了能够准确的记录各个线程正在执行的当前字节码指令地址，所以为每个线程都分配了一个PC寄存器，每个线程都独立计算，不会互相影响

> 相关总结如下：

- 它是一块很小的内存空间，几乎可以忽略不计，也是运行速度最快的存储区域
- 在 JVM 规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期一致
- 任何时间一个线程都只有一个方法在执行，也就是所谓的**当前方法**。如果当前线程正在执行的是 Java 方法，程序计数器记录的是 JVM 字节码指令地址，如果是执行 native 方法，则是未指定值（undefined）
- 它是程序控制流的指示器，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成
- 字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令
- **它是唯一一个在 JVM 规范中没有规定任何 `OutOfMemoryError` 情况的区域**



## 虚拟机栈

### 概述

> Java 虚拟机栈(Java Virtual Machine Stacks)，早期也叫 Java 栈。每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame），对应着一次次 Java 方法调用，是线程私有的，生命周期和线程一致

**作用**：主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回

**特点**：

- 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器
- JVM 直接对虚拟机栈的操作只有两个：每个方法执行，伴随着**入栈**（进栈/压栈），方法执行结束**出栈**
- **栈不存在垃圾回收问题**



**栈中可能出现的异常**：Java 虚拟机规范允许 **Java虚拟机栈的大小是动态的或者是固定不变的**

- 如果采用固定大小的 Java 虚拟机栈，那每个线程的 Java 虚拟机栈容量可以在线程创建的时候独立选定，如果线程请求分配的栈容量超过 Java 虚拟机栈允许的最大容量，Java 虚拟机将会抛出一个 **StackOverflowError** 异常
- 如果 Java 虚拟机栈采用动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那 Java 虚拟机将会抛出一个**OutOfMemoryError**异常

可以通过参数`-Xss`来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度

官方提供的参考工具，可查一些参数和操作：https://docs.oracle.com/Javase/8/docs/technotes/tools/windows/Java.html#BGBCIEFC



### 栈的存储单位

栈中存储什么？

- 每个线程都有自己的栈，栈中的数据都是以**栈帧（Stack Frame）的格式存在**
- 在这个线程上正在执行的每个方法都各自有对应的一个栈帧
- 栈帧是一个内存区块，是一个数据集，维系着方法执行过程中的各种数据信息



### 栈运行原理

- JVM 直接对 Java 栈的操作只有两个，对栈帧的**压栈**和**出栈**，遵循“先进后出/后进先出”原则
- 在一条活动线程中，一个时间点上，只会有一个活动的栈帧，即只有当前正在执行的方法的栈帧（**栈顶栈帧**）是有效的，这个栈帧被称为**当前栈帧**（Current Frame），与当前栈帧对应的方法就是**当前方法**（Current Method），定义这个方法的类就是**当前类**（Current Class）
- 执行引擎运行的所有字节码指令只针对当前栈帧进行操作
- 如果在该方法中调用了其他方法，对应的新的栈帧会被创建出来，放在栈的顶端，称为新的当前栈帧
- 不同线程中所包含的栈帧是不允许存在相互引用的，即不可能在一个栈帧中引用另外一个线程的栈帧
- 如果当前方法调用了其他方法，方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，接着，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧
- Java 方法有两种返回函数的方式，**一种是正常的函数返回，使用 return 指令，另一种是抛出异常，不管用哪种方式，都会导致栈帧被弹出**

IDEA 在 debug 时候，可以在 debug 窗口看到 Frames 中各种方法的压栈和出栈情况

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123759867-476158682.jpg)





### 栈帧的内部结构

每个**栈帧**（Stack Frame）中存储着：

- 局部变量表（Local Variables）
- 操作数栈（Operand Stack）(或称为表达式栈)
- 动态链接（Dynamic Linking）：指向运行时常量池的方法引用
- 方法返回地址（Return Address）：方法正常退出或异常退出的地址
- 一些附加信息

![JVM-stack-frame](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123815916-1384080329.jpg)



继续深抛栈帧中的五部分~~



#### 局部变量表

- 局部变量表也被称为局部变量数组或者本地变量表
- 是一组变量值存储空间，**主要用于存储方法参数和定义在方法体内的局部变量**，包括编译器可知的各种 Java 虚拟机**基本数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference类型，它并不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此相关的位置）和 **returnAddress** 类型（指向了一条字节码指令的地址，已被异常表取代）
- 由于局部变量表是建立在线程的栈上，是线程的私有数据，因此**不存在数据安全问题**
- **局部变量表所需要的容量大小是编译期确定下来的**，并保存在方法的 Code 属性的 `maximum local variables` 数据项中。在方法运行期间是不会改变局部变量表的大小的
- 方法嵌套调用的次数由栈的大小决定，一般来说，**栈越大，方法嵌套调用次数越多**。对一个函数而言，它的参数和局部变量越多，使得局部变量表膨胀，它的栈帧就越大，以满足方法调用所需传递的信息增大的需求，进而函数调用就会占用更多的栈空间，导致其嵌套调用次数就会减少
- **局部变量表中的变量只在当前方法调用中有效**。在方法执行时，虚拟机通过使用局部变量表完成参数值到参数变量列表的传递过程，当方法调用结束后，随着方法栈帧的销毁，局部变量表也会随之销毁
- 参数值的存放总是在局部变量数组的 index0 开始，到数组长度 -1 的索引结束



##### 槽 Slot

- 局部变量表最基本的存储单元是 Slot（变量槽）
- 在局部变量表中，32 位以内的类型只占用一个 Slot(包括returnAddress类型)，64 位的类型（long和double）占用两个连续的 Slot
	- byte、short、char 在存储前被转换为int，boolean也被转换为int，0 表示 false，非 0 表示 true
	- long 和 double 则占据两个 Slot
- JVM 会为局部变量表中的每一个 Slot 都分配一个访问索引，通过这个索引即可成功访问到局部变量表中指定的局部变量值，索引值的范围从 0 开始到局部变量表最大的 Slot 数量
- 当一个实例方法被调用的时候，它的方法参数和方法体内部定义的局部变量将会**按照顺序被复制**到局部变量表中的每一个 Slot 上
- **如果需要访问局部变量表中一个 64bit 的局部变量值时，只需要使用前一个索引即可**（比如：访问 long 或 double 类型变量，不允许采用任何方式单独访问其中的某一个 Slot）
- 如果当前帧是由构造方法或实例方法创建的，那么该对象引用 this 将会存放在 index 为 0 的 Slot 处，其余的参数按照参数表顺序继续排列（这里就引出一个问题：静态方法中为什么不可以引用 this，就是因为this 变量不存在于当前方法的局部变量表中）
- **栈帧中的局部变量表中的槽位是可以重用的**，如果一个局部变量过了其作用域，那么在其作用域之后申明的新的局部变量就很有可能会复用过期局部变量的槽位，从而**达到节省资源的目的**（下图中，this、a、b、c 理论上应该有 4 个变量，c 复用了 b 的槽）

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123839227-962398178.jpg)

- 在栈帧中，与性能调优关系最为密切的就是局部变量表在方法执行时，虚拟机使用局部变量表完成方法的传递
- **局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象都不会被回收**



#### 操作数栈

- 每个独立的栈帧中除了包含局部变量表之外，还包含一个**后进先出**（Last-In-First-Out）的操作数栈，也可以称为**表达式栈**（Expression Stack）
- **操作数栈，在方法执行过程中，根据字节码指令，往操作数栈中写入数据或提取数据，即入栈（push）、出栈（pop）**
- 某些字节码指令将值压入操作数栈，其余的字节码指令将操作数取出栈使用它们后再把结果压入栈。比如，执行复制、交换、求和等操作



##### 概述

- 操作数栈，**主要用于保存计算过程的中间结果，同时作为计算过程中变量临时的存储空间**
- 操作数栈就是 JVM 执行引擎的一个工作区，当一个方法刚开始执行的时候，一个新的栈帧也会随之被创建出来，**此时这个方法的操作数栈是空的**
- 每一个操作数栈都会拥有一个明确的栈深度用于存储数值，其所需的最大深度在编译期就定义好了，保存在方法的 Code 属性的 `max_stack` 数据项中
- 栈中的任何一个元素都可以是任意的 Java 数据类型 
	- 32bit 的类型占用一个栈单位深度
	- 64bit 的类型占用两个栈单位深度
- 操作数栈并非采用访问索引的方式来进行数据访问的，而是只能通过标准的入栈和出栈操作来完成一次数据访问
- **如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新 PC 寄存器中下一条需要执行的字节码指令
- 操作数栈中元素的数据类型必须与字节码指令的序列严格匹配，这由编译器在编译期间进行验证，同时在类加载过程中的类检验阶段的数据流分析阶段要再次验证
- 另外，我们说**Java虚拟机的解释引擎是基于栈的执行引擎**，其中的栈指的就是操作数栈



##### 栈顶缓存（Top-of-stack-Cashing）

HotSpot 的执行引擎采用的并非是基于寄存器的架构，但这并不代表 HotSpot VM 的实现并没有间接利用到寄存器资源，寄存器是物理 CPU 中的组成部分之一，它同时也是 CPU 中非常重要的高速存储资源。一般来说，寄存器的读/写速度非常迅速，甚至可以比内存的读/写速度快上几十倍不止，不过寄存器资源却非常有限，不同平台下的CPU 寄存器数量是不同和不规律的。寄存器主要用于缓存本地机器指令、数值和下一条需要被执行的指令地址等数据

基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要更多的指令分派（instruction dispatch）次数和内存读/写次数，由于操作数是存储在内存中的，因此频繁的执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM 设计者们提出了栈顶缓存技术，**将栈顶元素全部缓存在物理 CPU 的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率**



#### 动态链接（指向运行时常量池的方法引用）

- **每一个栈帧内部都包含一个指向运行时常量池中该栈帧所属方法的引用**。包含这个引用的目的就是为了支持当前方法的代码能够实现动态链接(Dynamic Linking)
- 在 Java 源文件被编译到字节码文件中时，所有的变量和方法引用都作为**符号引用**（Symbolic Reference），保存在 Class 文件的常量池中。比如：描述一个方法调用了另外的其他方法时，就是通过常量池中指向方法的符号引用来表示的，那么**动态链接的作用就是为了将这些符号引用转换为调用方法的直接引用**

![JVM-dynamic-linking](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511123915085-674734910.jpg)



##### JVM 是如何执行方法调用的

方法调用不同于方法执行，方法调用阶段的唯一任务就是确定被调用方法的版本（即调用哪一个方法），暂时还不涉及方法内部的具体运行过程，Class 文件的编译过程中不包括传统编译器中的连接步骤，一切方法调用在 Class文件里面存储的都是**符号引用**，而不是方法在实际运行时内存布局中的入口地址（**直接引用**），也就是需要在类加载阶段，甚至到运行期才能确定目标方法的直接引用

> 【这一块内容，除了方法调用，还包括解析、分派（静态分派、动态分派、单分派与多分派），这里先不介绍，后续再挖】

在 JVM 中，将符号引用转换为调用方法的直接引用与方法的绑定机制有关

- **静态链接**：当一个字节码文件被装载进 JVM 内部时，如果被调用的**目标方法在编译期可知**，且运行期保持不变时，这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接
- **动态链接**：如果被调用的方法在编译期无法被确定下来，也就是说，只能在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接

对应的方法的绑定机制为：早期绑定（Early Binding）和晚期绑定（Late Binding）。**绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生一次**

- 早期绑定：**早期绑定就是指被调用的目标方法如果在编译期可知，且运行期保持不变时**，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用
- 晚期绑定：如果被调用的方法在编译器无法被确定下来，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式就被称为晚期绑定



##### 虚方法和非虚方法

- 如果方法在编译器就确定了具体的调用版本，这个版本在运行时是不可变的，这样的方法称为非虚方法，比如静态方法、私有方法、final 方法、实例构造器、父类方法都是非虚方法
- 其他方法称为虚方法



##### 虚方法表

在面向对象编程中，会频繁的使用到动态分派，如果每次动态分派都要重新在类的方法元数据中搜索合适的目标，有可能会影响到执行效率，为了提高性能，JVM 采用在类的方法区建立一个虚方法表（virtual method table），使用索引表来代替查找非虚方法会不会出现在表中

每个类中都有一个虚方法表，表中存放着各个方法的实际入口

虚方法表会在类加载的连接阶段被创建并开始初始化，类的变量初始值准备完成之后，JVM 会把该类的方法表也初始化完毕



#### 方法返回地址（return address）

用来存放调用该方法的 PC 寄存器的值

一个方法的结束，有两种方式

- 正常执行完成
- 出现未处理的异常，非正常退出

无论通过哪种方式退出，在方法退出后都返回到该方法被调用的位置。方法正常退出时，调用者的 PC 计数器的值作为返回地址，即调用该方法的指令的下一条指令的地址；而通过异常退出的，返回地址是要通过异常表来确定的，栈帧中一般不会保存这部分信息

当一个方法开始执行后，只有两种方式可以退出这个方法：

1. 执行引擎遇到任意一个方法返回的字节码指令，会有返回值传递给上层的方法调用者，简称**正常完成出口**

	一个方法的正常调用完成之后，究竟需要使用哪一个返回指令，还需要根据方法返回值的实际数据类型而定

	在字节码指令中，返回指令包含 ireturn(当返回值是 boolean、byte、char、short 和 int 类型时使用)、lreturn、freturn、dreturn 以及 areturn，另外还有一个 return 指令供声明为 void 的方法、实例初始化方法、类和接口的初始化方法使用

2. 在方法执行的过程中遇到了异常，并且这个异常没有在方法内进行处理，也就是只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，简称**异常完成出口**

	方法执行过程中抛出异常时的异常处理，存储在一个异常处理表，方便在发生异常的时候找到处理异常的代码

本质上，**方法的退出就是当前栈帧出栈的过程**，此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去

正常完成出口和异常完成出口的区别在于：**通过异常完成出口退出的，不会给他的上层调用者产生任何的返回值**



#### 附加信息

栈帧中还允许携带与 Java 虚拟机实现相关的一些附加信息，例如，对程序调试提供支持的信息，但这些信息取决于具体的虚拟机实现



## 本地方法栈

### 本地方法接口

简单的讲，一个 Native Method 就是一个 Java 调用非 Java 代码的接口。我们知道的 Unsafe 类就有很多本地方法

> 为什么要使用本地方法（Native Method）?

Java 使用起来非常方便，然而有些层次的任务用 Java 实现起来也不容易，或者我们对程序的效率很在意时，问题就来了

- 与 Java 环境外交互：有时 Java 应用需要与 Java 外面的环境交互，这就是本地方法存在的原因
- 与操作系统交互：JVM 支持 Java 语言本身和运行时库，但是有时仍需要依赖一些底层系统的支持。通过本地方法，我们可以实现用 Java 与实现了 jre 的底层系统交互， JVM 的一些部分就是 C 语言写的
- Sun's Java：Sun的解释器就是C实现的，这使得它能像一些普通的C一样与外部交互，jre大部分都是用 Java 实现的，它也通过一些本地方法与外界交互。比如，类 `Java.lang.Thread` 的 `setPriority()` 的方法是用Java 实现的，但它实现调用的是该类的本地方法 `setPrioruty()`，该方法是C实现的，并被植入 JVM 内部



### 本地方法栈（Native Method Stack）

- Java 虚拟机栈用于管理 Java 方法的调用，而本地方法栈用于管理本地方法的调用
- 本地方法栈也是线程私有的
- 允许线程固定或者可动态扩展的内存大小
	- 如果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java 虚拟机将会抛出一个 `StackOverflowError` 异常
	- 如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么 Java虚拟机将会抛出一个`OutofMemoryError`异常
- 本地方法是使用 C 语言实现的
- 它的具体做法是 `Native Method Stack` 中登记 native 方法，在 `Execution Engine` 执行时加载本地方法库，当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界，它和虚拟机拥有同样的权限
- 本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区，它甚至可以直接使用本地处理器中的寄存器，直接从本地内存的堆中分配任意数量的内存
- 并不是所有 JVM 都支持本地方法，因为 Java 虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等，如果 JVM 产品不打算支持 native 方法，也可以无需实现本地方法栈
- 在 Hotspot JVM 中，直接将本地方法栈和虚拟机栈合二为一



> **栈是运行时的单位，而堆是存储的单位**
>
> 栈解决程序的运行问题，即程序如何执行，或者说如何处理数据；堆解决的是数据存储的问题，即数据怎么放、放在哪



## 堆内存

### 内存划分

对于大多数应用，Java 堆是 Java 虚拟机管理的内存中最大的一块，被所有线程共享。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数据都在这里分配内存

为了进行高效的垃圾回收，虚拟机把堆内存**逻辑上**划分成三块区域（分代的唯一理由就是优化 GC 性能）：

- 新生代（年轻代）：新对象和没达到一定年龄的对象都在新生代
- 老年代（养老区）：被长时间使用的对象，老年代的内存空间应该要比年轻代更大
- 元空间（JDK1.8 之前叫永久代）：像一些方法中的操作临时对象等，JDK1.8 之前是占用 JVM 内存，JDK1.8 之后直接使用物理内存

![JDK7](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511124023208-2044167795.jpg)

Java 虚拟机规范规定，Java 堆可以是处于物理上不连续的内存空间中，只要逻辑上是连续的即可，像磁盘空间一样实现时，既可以是固定大小，也可以是可扩展的，主流虚拟机都是可扩展的（通过 `-Xmx` 和 `-Xms` 控制），如果堆中没有完成实例分配，并且堆无法再扩展时，就会抛出 `OutOfMemoryError` 异常



#### 年轻代 (Young Generation)

年轻代是所有新对象创建的地方。当填充年轻代时，执行垃圾收集，这种垃圾收集称为 **Minor GC**。年轻一代被分为三个部分——伊甸园（**Eden Memory**）和两个幸存区（**Survivor Memory**，被称为from/to或s0/s1），默认比例是`8:1:1`

- 大多数新创建的对象都位于 Eden 内存空间中。
- 当 Eden 空间被对象填充时，执行**Minor GC**，并将所有幸存者对象移动到一个幸存者空间中。
- Minor GC 检查幸存者对象，并将它们移动到另一个幸存者空间。所以每次，一个幸存者空间总是空的。
- 经过多次 GC 循环后存活下来的对象被移动到老年代。通常，这是通过设置年轻一代对象的年龄阈值来实现的，然后他们才有资格提升到老一代。



#### 老年代(Old Generation)

旧的一代内存包含那些经过许多轮小型 GC 后仍然存活的对象。通常，垃圾收集是在老年代内存满时执行的。老年代垃圾收集称为 主GC（Major GC），通常需要更长的时间

大对象直接进入老年代（大对象是指需要大量连续内存空间的对象），这样做的目的是避免在 Eden 区和两个Survivor 区之间发生大量的内存拷贝

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511124052837-1345226006.jpg)



#### 元空间

不管是 JDK8 之前的永久代，还是 JDK8 及以后的元空间，都可以看作是 Java 虚拟机规范中方法区的实现

虽然 Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫 Non-Heap（非堆），目的应该是与 Java 堆区分开，所以元空间放在后边的方法区再说



### 设置堆内存大小和 OOM

Java 堆用于存储 Java 对象实例，那么堆的大小在 JVM 启动的时候就确定了，我们可以通过 `-Xmx` 和 `-Xms` 来设定

- `-Xms` 用来表示堆的起始内存，等价于 `-XX:InitialHeapSize`
- `-Xmx` 用来表示堆的最大内存，等价于 `-XX:MaxHeapSize`

如果堆的内存大小超过 `-Xmx` 设定的最大内存， 就会抛出 `OutOfMemoryError` 异常

我们通常会将 `-Xmx` 和 `-Xms` 两个参数配置为相同的值，其目的是为了能够在垃圾回收机制清理完堆区后不再需要重新分隔计算堆的大小，从而提高性能

- 默认情况下，初始堆内存大小为：电脑内存大小/64
- 默认情况下，最大堆内存大小为：电脑内存大小/4

可以通过代码获取到我们的设置值，当然也可以模拟 OOM：

```java
public static void main(String[] args) {

  // 返回 JVM 堆大小
  long initalMemory = Runtime.getRuntime().totalMemory() / 1024 /1024;
  // 返回 JVM 堆的最大内存
  long maxMemory = Runtime.getRuntime().maxMemory() / 1024 /1024;

  System.out.println("-Xms : "+initalMemory + "M");
  System.out.println("-Xmx : "+maxMemory + "M");

  System.out.println("系统内存大小：" + initalMemory * 64 / 1024 + "G");
  System.out.println("系统内存大小：" + maxMemory * 4 / 1024 + "G");
}
```



#### 查看 JVM 堆内存分配

1. 在默认不配置 JVM 堆内存大小的情况下，JVM 根据默认值来配置当前内存大小

2. 默认情况下新生代和老年代的比例是 1:2，可以通过 `–XX:NewRatio` 来配置

	- 新生代中的 **Eden**:**From Survivor**:**To Survivor** 的比例是 **8:1:1**，可以通过 `-XX:SurvivorRatio` 来配置

3. 若在 JDK 7 中开启了 `-XX:+UseAdaptiveSizePolicy`，JVM 会动态调整 JVM 堆中各个区域的大小以及进入老年代的年龄

	此时 `–XX:NewRatio` 和 `-XX:SurvivorRatio` 将会失效，而 JDK 8 是默认开启`-XX:+UseAdaptiveSizePolicy`

	在 JDK 8中，**不要随意关闭**`-XX:+UseAdaptiveSizePolicy`，除非对堆内存的划分有明确的规划

每次 GC 后都会重新计算 Eden、From Survivor、To Survivor 的大小

计算依据是**GC过程**中统计的**GC时间**、**吞吐量**、**内存占用量**

```text
Java -XX:+PrintFlagsFinal -version | grep HeapSize
    uintx ErgoHeapSizeLimit                         = 0                                   {product}
    uintx HeapSizePerGCThread                       = 87241520                            {product}
    uintx InitialHeapSize                          := 134217728                           {product}
    uintx LargePageHeapSizeThreshold                = 134217728                           {product}
    uintx MaxHeapSize                              := 2147483648                          {product}
Java version "1.8.0_211"
Java(TM) SE Runtime Environment (build 1.8.0_211-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.211-b12, mixed mode)
$ jmap -heap 进程号
```



### 对象在堆中的生命周期

1. 在 JVM 内存模型的堆中，堆被划分为新生代和老年代 
	- 新生代又被进一步划分为 **Eden区** 和 **Survivor区**，Survivor 区由 **From Survivor** 和 **To Survivor** 组成
2. 当创建一个对象时，对象会被优先分配到新生代的 Eden 区 
	- 此时 JVM 会给对象定义一个**对象年轻计数器**（`-XX:MaxTenuringThreshold`）
3. 当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC） 
	- JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1
	- 对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会+1
4. 如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代**



### 对象的分配过程

为对象分配内存是一件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法和内存回收算法密切相关，所以还需要考虑 GC 执行完内存回收后是否会在内存空间中产生内存碎片。

1. new 的对象先放在伊甸园区，此区有大小限制
2. 当伊甸园的空间填满时，程序又需要创建对象，JVM 的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC），将伊甸园区中不再被其他对象所引用的对象进行销毁，再加载新的对象放到伊甸园区
3. 然后将伊甸园中的剩余对象移动到幸存者 0 区
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者 0 区，如果没有回收，就会放到幸存者 1 区
5. 如果再次经历垃圾回收，此时会重新放回幸存者 0 区，接着再去幸存者 1 区
6. 什么时候才会去养老区呢？ 默认是 15 次回收标记
7. 在养老区，相对悠闲，当养老区内存不足时，再次触发 Major GC，进行养老区的内存清理
8. 若养老区执行了 Major GC 之后发现依然无法进行对象的保存，就会产生 OOM 异常



### GC 垃圾回收简介

#### Minor GC、Major GC、Full GC

JVM 在进行 GC 时，并非每次都对堆内存（新生代、老年代；方法区）区域一起回收的，大部分时候回收的都是指新生代

针对 HotSpot VM 的实现，它里面的 GC 按照回收区域又分为两大类：部分收集（Partial GC），整堆收集（Full GC）

- 部分收集：不是完整收集整个 Java 堆的垃圾收集。其中又分为： 
	- 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
	- 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集 
		- 目前，只有 CMS GC 会有单独收集老年代的行为
		- 很多时候 Major GC 会和 Full GC 混合使用，需要具体分辨是老年代回收还是整堆回收
	- 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集 
		- 目前只有 G1 GC 会有这种行为
- 整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾



### TLAB

#### 什么是 TLAB （Thread Local Allocation Buffer）?

- 从内存模型而不是垃圾回收的角度，对 Eden 区域继续进行划分，JVM 为每个线程分配了一个私有缓存区域，它包含在 Eden 空间内
- 多线程同时分配内存时，使用 TLAB 可以避免一系列的非线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**
- OpenJDK 衍生出来的 JVM 大都提供了 TLAB 设计



#### 为什么要有 TLAB ?

- 堆区是线程共享的，任何线程都可以访问到堆区中的共享数据
- 由于对象实例的创建在 JVM 中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度

尽管不是所有的对象实例都能够在 TLAB 中成功分配内存，但 JVM 确实是将 TLAB 作为内存分配的首选

在程序中，可以通过 `-XX:UseTLAB` 设置是否开启 TLAB 空间

默认情况下，TLAB 空间的内存非常小，仅占有整个 Eden 空间的 1%，我们可以通过 `-XX:TLABWasteTargetPercent` 设置 TLAB 空间所占用 Eden 空间的百分比大小

一旦对象在 TLAB 空间分配内存失败时，JVM 就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在 Eden 空间中分配内存



### 堆是分配对象存储的唯一选择吗

> 随着 JIT 编译期的发展和逃逸分析技术的逐渐成熟，栈上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆上也渐渐变得不那么“绝对”了 ——《深入理解 Java 虚拟机》



#### 逃逸分析

**逃逸分析(Escape Analysis)\是目前 Java 虚拟机中比较前沿的优化技术\。这是一种可以有效减少 Java 程序中同步负载和内存堆分配压力的跨函数全局数据流分析算法**。通过逃逸分析，Java Hotspot 编译器能够分析出一个新的对象的引用的使用范围，从而决定是否要将这个对象分配到堆上

逃逸分析的基本行为就是分析对象动态作用域：

- 当一个对象在方法中被定义后，对象只在方法内部使用，则认为没有发生逃逸
- 当一个对象在方法中被定义后，它被外部方法所引用，则认为发生逃逸，例如作为调用参数传递到其他地方中，称为方法逃逸

```java
public static StringBuffer craeteStringBuffer(String s1, String s2) {
   StringBuffer sb = new StringBuffer();
   sb.append(s1);
   sb.append(s2);
   return sb;
}
```

`StringBuffer sb`是一个方法内部变量，上述代码中直接将sb返回，这样这个 StringBuffer 有可能被其他方法所改变，这样它的作用域就不只是在方法内部，虽然它是一个局部变量，但是其逃逸到了方法外部甚至还有可能被外部线程访问到，譬如赋值给类变量或可以在其他线程中访问的实例变量，称为线程逃逸

上述代码如果想要 `StringBuffer sb`不逃出方法，可以这样写：

```java
public static String createStringBuffer(String s1, String s2) {
   StringBuffer sb = new StringBuffer();
   sb.append(s1);
   sb.append(s2);
   return sb.toString();
}
```

不直接返回 StringBuffer，那么 StringBuffer 将不会逃逸出方法

**参数设置：**

- 在 JDK 6u23 版本之后，HotSpot 中默认就已经开启了逃逸分析
- 如果使用较早版本，可以通过`-XX"+DoEscapeAnalysis`显式开启

开发中使用局部变量，就不要在方法外定义

使用逃逸分析，编译器可以对代码做优化：

- **栈上分配**：将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配
- **同步省略**：如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步
- **分离对象或标量替换**：有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而存储在 CPU 寄存器

JIT 编译器在编译期间根据逃逸分析的结果，发现如果一个对象并没有逃逸出方法的话，就可能被优化成栈上分配分配。完成后，继续在调用栈内执行，最后线程结束，栈空间被回收，局部变量对象也被回收，这样就无需进行垃圾回收了

常见栈上分配的场景：成员变量赋值、方法返回值、实例引用传递



##### 代码优化之同步省略（消除）

- 线程同步的代价是相当高的，同步的后果是降低并发性和性能
- 在动态编译同步块的时候，JIT 编译器可以借助逃逸分析来判断同步块所使用的锁对象是否能够被一个线程访问而没有被发布到其他线程，如果没有，那么 JIT 编译器在编译这个同步块的时候就会取消对这个代码的同步。这样就能大大提高并发性和性能，这个取消同步的过程就叫做**同步省略，也叫锁消除**

```java
public void keep() {
  Object keeper = new Object();
  synchronized(keeper) {
    System.out.println(keeper);
  }
}
```

如上代码，代码中对 keeper 这个对象进行加锁，但是 keeper 对象的生命周期只在 `keep()`方法中，并不会被其他线程所访问到，所以在 JIT编译阶段就会被优化掉优化成：

```java
public void keep() {
  Object keeper = new Object();
  System.out.println(keeper);
}
```



##### 代码优化之标量替换

**标量**（Scalar）是指一个无法再分解成更小的数据的数据。Java 中的基本数据类型就是标量

相对的，那些的还可以分解的数据叫做**聚合量**（Aggregate），Java 中的对象就是聚合量，因为其还可以分解成其他聚合量和标量

在 JIT 阶段，通过逃逸分析确定该对象不会被外部访问，并且对象可以被进一步分解时，JVM 不会创建该对象，而会将该对象成员变量分解成若干个被使用的方法的成员变量所代替。这些代替的成员变量在栈帧或寄存器上分配空间。这个过程就是**标量替换**

通过 `-XX:+EliminateAllocations` 可以开启标量替换，`-XX:+PrintEliminateAllocations` 查看标量替换情况

```java
public static void main(String[] args) {
   alloc();
}

private static void alloc() {
   Point point = new Point（1,2）;
   System.out.println("point.x="+point.x+"; point.y="+point.y);
}

Class Point{
    private int x;
    private int y;
}
```

以上代码中，point 对象并没有逃逸出 `alloc()` 方法，并且 point 对象是可以拆解成标量的。那么，JIT 就不会直接创建 Point 对象，而是直接使用两个标量 int x ，int y 来替代 Point 对象

```java
private static void alloc() {
   int x = 1;
   int y = 2;
   System.out.println("point.x="+x+"; point.y="+y);
}
```



##### 代码优化之栈上分配

我们通过 JVM 内存分配可以知道 Java中的对象都是在堆上进行分配，当对象没有被引用的时候，需要依靠 GC 进行回收内存，如果对象数量较多的时候，会给 GC 带来较大压力，也间接影响了应用的性能。为了减少临时对象在堆内分配的数量，JVM 通过逃逸分析确定该对象不会被外部访问。那就通过标量替换将该对象分解在栈上分配内存，这样该对象所占用的内存空间就可以随栈帧出栈而销毁，就减轻了垃圾回收的压力



**总结：**

关于逃逸分析的论文在1999年就已经发表了，但直到JDK 1.6才有实现，而且这项技术到如今也并不是十分成熟

**其根本原因就是无法保证逃逸分析的性能消耗一定能高于他的消耗。虽然经过逃逸分析可以做标量替换、栈上分配、和锁消除。但是逃逸分析自身也是需要进行一系列复杂的分析的，这其实也是一个相对耗时的过程**

一个极端的例子，就是经过逃逸分析之后，发现没有一个对象是不逃逸的。那这个逃逸分析的过程就白白浪费掉了

虽然这项技术并不十分成熟，但是他也是即时编译器优化技术中一个十分重要的手段





## 方法区

- 方法区（Method Area）与 Java 堆一样，是所有线程共享的内存区域
- 虽然 Java 虚拟机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫 Non-Heap（非堆），目的应该是与 Java 堆区分开
- 运行时常量池（Runtime Constant Pool）是方法区的一部分。Class 文件中除了有类的版本/字段/方法/接口等描述信息外，还有一项信息是常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的是 `String.intern()`方法。受方法区内存的限制，当常量池无法再申请到内存时会抛出 `OutOfMemoryErro`r 异常
- 方法区的大小和堆空间一样，可以选择固定大小也可选择可扩展，方法区的大小决定了系统可以放多少个类，如果系统类太多，导致方法区溢出，虚拟机同样会抛出内存溢出错误
- JVM 关闭后方法区即被释放



### 解惑

你是否也有看不同的参考资料，有的内存结构图有方法区，有的又是永久代，元数据区，一脸懵逼的时候？

- **方法区（method area）只是 JVM 规范中定义的一个概念**，用于存储类信息、常量池、静态变量、JIT编译后的代码等数据，并没有规定如何去实现它，不同的厂商有不同的实现。而**永久代（PermGen）是 Hotspot 虚拟机特有的概念， Java8 的时候又被 元空间**取代了，永久代和元空间都可以理解为方法区的落地实现
- 永久代物理是堆的一部分，和新生代，老年代地址是连续的（受垃圾回收器管理），而元空间存在于本地内存（我们常说的堆外内存，不受垃圾回收器管理），这样就不受 JVM 限制了，也比较难发生OOM（都会有溢出异常）
- Java7 中我们通过`-XX:PermSize` 和 `-xx:MaxPermSize` 来设置永久代参数，Java8 之后，随着永久代的取消，这些参数也就随之失效了，改为通过`-XX:MetaspaceSize` 和 `-XX:MaxMetaspaceSize` 用来设置元空间参数
- 存储内容不同，元空间存储类的元信息，静态变量和常量池等并入堆中。相当于永久代的数据被分到了堆和元空间中
- 如果方法区域中的内存不能用于满足分配请求，则 Java 虚拟机抛出 `OutOfMemoryError`
- JVM 规范说方法区在逻辑上是堆的一部分，但目前实际上是与 Java 堆分开的（Non-Heap）

所以对于方法区，Java8 之后的变化：

- 移除了永久代（PermGen），替换为元空间（Metaspace）；
- 永久代中的 class metadata 转移到了 native memory（本地内存，而不是虚拟机）；
- 永久代中的 interned Strings 和 class static variables 转移到了 Java heap；
- 永久代参数 （PermSize MaxPermSize） -> 元空间参数（MetaspaceSize MaxMetaspaceSize）



### 设置方法区内存的大小

JDK8 及以后：

- 元数据区大小可以使用参数 `-XX:MetaspaceSize` 和 `-XX:MaxMetaspaceSize` 指定，替代上述原有的两个参数
- 默认值依赖于平台。Windows 下，`-XX:MetaspaceSize` 是 21M，`-XX:MaxMetaspacaSize` 的值是 -1，即没有限制
- 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据发生溢出，虚拟机一样会抛出异常 `OutOfMemoryError:Metaspace`
- `-XX:MetaspaceSize` ：设置初始的元空间大小。对于一个 64 位的服务器端 JVM 来说，其默认的 `-XX:MetaspaceSize` 的值为20.75MB，这就是初始的高水位线，一旦触及这个水位线，Full GC 将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个高水位线将会重置，新的高水位线的值取决于 GC 后释放了多少元空间。如果释放的空间不足，那么在不超过 `MaxMetaspaceSize`时，适当提高该值。如果释放空间过多，则适当降低该值
- 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次，通过垃圾回收的日志可观察到 Full GC 多次调用。为了避免频繁 GC，建议将 `-XX:MetaspaceSize` 设置为一个相对较高的值





### 方法区内部结构

方法区用于存储已被虚拟机加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存等



#### 类型信息

对每个加载的类型（类 Class、接口 interface、枚举 enum、注解 annotation），JVM 必须在方法区中存储以下类型信息

- 这个类型的完整有效名称（全名=包名.类名）
- 这个类型直接父类的完整有效名（对于 interface或是 Java.lang.Object，都没有父类）
- 这个类型的修饰符（public，abstract，final 的某个子集）
- 这个类型直接接口的一个有序列表



#### 域（Field）信息

- JVM 必须在方法区中保存类型的所有域的相关信息以及域的声明顺序
- 域的相关信息包括：域名称、域类型、域修饰符（public、private、protected、static、final、volatile、transient 的某个子集）



#### 方法（Method）信息

JVM 必须保存所有方法的

- 方法的修饰符（public，private，protected，static，final，synchronized，native，abstract 的一个子集）
- 方法的返回类型
- 方法名称
- 方法参数的数量和类型
- 方法的字符码（bytecodes）、操作数栈、局部变量表及大小（abstract 和 native 方法除外）
- 异常表（abstract 和 native 方法除外） 
	- 每个异常处理的开始位置、结束位置、代码处理在程序计数器中的偏移地址、被捕获的异常类的常量池索引

**栈、堆、方法区的交互关系**

![image-20230511125142334](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125141871-2060339676.png)





### 运行时常量池

> 运行时常量池（Runtime Constant Pool）是方法区的一部分，理解运行时常量池的话，我们先来说说字节码文件（Class 文件）中的常量池（常量池表）



#### 常量池

一个有效的字节码文件中除了包含类的版本信息、字段、方法以及接口等描述信息外，还包含一项信息那就是常量池表（Constant Pool Table），包含各种字面量和对类型、域和方法的符号引用



##### 为什么需要常量池？

一个 Java 源文件中的类、接口，编译后产生一个字节码文件。而 Java 中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候用到的就是运行时常量池

如下，我们通过 jClasslib 查看一个只有 Main 方法的简单类，字节码中的 #2 指向的就是 Constant Pool

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125242307-1032086854.jpg)

常量池可以看作是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等类型



#### 运行时常量池

- 在加载类和结构到虚拟机后，就会创建对应的运行时常量池
- 常量池表（Constant Pool Table）是 Class 文件的一部分，用于存储编译期生成的各种字面量和符号引用，**这部分内容将在类加载后存放到方法区的运行时常量池中**
- JVM 为每个已加载的类型（类或接口）都维护一个常量池。池中的数据项像数组项一样，是通过索引访问的
- 运行时常量池中包含各种不同的常量，包括编译器就已经明确的数值字面量，也包括到运行期解析后才能够获得的方法或字段引用。此时不再是常量池中的符号地址了，这里换为真实地址 
  - 运行时常量池，相对于 Class 文件常量池的另一个重要特征是：**动态性**，Java 语言并不要求常量一定只有编译期间才能产生，运行期间也可以将新的常量放入池中，String 类的 `intern()` 方法就是这样的
- 当创建类或接口的运行时常量池时，如果构造运行时常量池所需的内存空间超过了方法区所能提供的最大值，则 JVM 会抛出 OutOfMemoryError 异常





### 方法区在 JDK6、7、8中的演进细节

只有 HotSpot 才有永久代的概念

| JDK1.6及之前 | 有永久代，静态变量存放在永久代上                             |
| ------------ | ------------------------------------------------------------ |
| JDK1.7       | 有永久代，但已经逐步“去永久代”，字符串常量池、静态变量移除，保存在堆中 |
| JDK1.8及之后 | 取消永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆中 |

**@pdai: HotSpot中字符串常量池保存在哪里？永久代？方法区还是堆区**？

1. 运行时常量池（Runtime Constant Pool）是虚拟机规范中是方法区的一部分，在加载类和结构到虚拟机后，就会创建对应的运行时常量池；而字符串常量池是这个过程中常量字符串的存放位置。所以从这个角度，字符串常量池属于虚拟机规范中的方法区，它是一个**逻辑上的概念**；而堆区，永久代以及元空间是实际的存放位置
2. 不同的虚拟机对虚拟机的规范（比如方法区）是不一样的，只有 HotSpot 才有永久代的概念
3. HotSpot也是发展的，由于[一些问题](http://openJDK.Java.net/jeps/122)的存在，HotSpot考虑逐渐去永久代，对于不同版本的JDK，**实际的存储位置**是有差异的，具体看如下表格：

| JDK版本      | 是否有永久代，字符串常量池放在哪里？                         | 方法区逻辑上规范，由哪些实际的部分实现的？                   |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| JDK1.6及之前 | 有永久代，运行时常量池（包括字符串常量池），静态变量存放在永久代上 | 这个时期方法区在HotSpot中是由永久代来实现的，以至于**这个时期说方法区就是指永久代** |
| JDK1.7       | 有永久代，但已经逐步“去永久代”，字符串常量池、静态变量移除，保存在堆中； | 这个时期方法区在HotSpot中由**永久代**（类型信息、字段、方法、常量）和**堆**（字符串常量池、静态变量）共同实现 |
| JDK1.8及之后 | 取消永久代，类型信息、字段、方法、常量保存在本地内存的元空间，但字符串常量池、静态变量仍在堆中 | 这个时期方法区在HotSpot中由本地内存的**元空间**（类型信息、字段、方法、常量）和**堆**（字符串常量池、静态变量）共同实现 |

#### 移除永久代原因

http://openJDK.Java.net/jeps/122

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125311289-861847999.jpg)

- 为永久代设置空间大小是很难确定的。

  在某些场景下，如果动态加载类过多，容易产生 Perm 区的 OOM。如果某个实际 Web 工程中，因为功能点比较多，在运行过程中，要不断动态加载很多类，经常出现 OOM。而元空间和永久代最大的区别在于，元空间不在虚拟机中，而是使用本地内存，所以默认情况下，元空间的大小仅受本地内存限制

- 对永久代进行调优较困难





### 方法区的垃圾回收

方法区的垃圾收集主要回收两部分内容：**常量池中废弃的常量和不再使用的类型**

先来说说方法区内常量池之中主要存放的两大类常量：字面量和符号引用。字面量比较接近 Java 语言层次的常量概念，如文本字符串、被声明为 final 的常量值等。而符号引用则属于编译原理方面的概念，包括下面三类常量：

- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

HotSpot 虚拟机对常量池的回收策略是很明确的，只要常量池中的常量没有被任何地方引用，就可以被回收

判定一个类型是否属于“不再被使用的类”，需要同时满足三个条件：

- 该类所有的实例都已经被回收，也就是 Java 堆中不存在该类及其任何派生子类的实例
- 加载该类的类加载器已经被回收，这个条件除非是经过精心设计的可替换类加载器的场景，如 OSGi、JSP 的重加载等，否则通常很难达成
- 该类对应的 Java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

Java 虚拟机被允许堆满足上述三个条件的无用类进行回收，这里说的仅仅是“被允许”，而并不是和对象一样，不使用了就必然会回收。是否对类进行回收，HotSpot 虚拟机提供了 `-Xnoclassgc` 参数进行控制，还可以使用 `-verbose:class` 以及 `-XX:+TraceClassLoading` 、`-XX:+TraceClassUnLoading` 查看类加载和卸载信息

在大量使用反射、动态代理、CGLib 等 ByteCode 框架、动态生成 JSP 以及 OSGi 这类频繁自定义 ClassLoader 的场景都需要虚拟机具备类卸载的功能，以保证永久代不会溢出





## 参考与感谢

- 作者：海星
- 来源于：JavaKeeper

原作者参考内容如下

算是一篇学习笔记，共勉，主要来源：

《深入理解 Java 虚拟机 第三版》

宋红康老师的 JVM 教程

https://docs.oracle.com/Javase/specs/index.html

https://www.cnblogs.com/wicfhwffg/p/9382677.html

https://www.cnblogs.com/hollischuang/p/12501950.html







# JVM 基础 - Java 内存模型引入

> 很多人都无法区分Java内存模型和JVM内存结构，以及Java内存模型与物理内存之间的关系。本文从堆栈角度引入JMM，然后介绍JMM和物理内存之间的关系, 为后面`JMM详解`, `JVM 内存结构详解`, `Java 对象模型详解`等铺垫

## JMM引入

### 从堆栈说起

JVM内部使用的Java内存模型在线程栈和堆之间划分内存 此图从逻辑角度说明了Java内存模型：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125540474-1002273258.png)

### 堆栈里面放了什么?

线程堆栈还包含正在执行的每个方法的所有局部变量(调用堆栈上的所有方法)。 线程只能访问它自己的线程堆栈。 由线程创建的局部变量对于创建它的线程以外的所有其他线程是不可见的。 即使两个线程正在执行完全相同的代码，两个线程仍将在每个自己的线程堆栈中创建该代码的局部变量。 因此，每个线程都有自己的每个局部变量的版本。

基本类型的所有局部变量(boolean，byte，short，char，int，long，float，double)完全存储在线程堆栈中，因此对其他线程不可见。 一个线程可以将一个基本类型变量的副本传递给另一个线程，但它不能共享原始局部变量本身。

堆包含了在Java应用程序中创建的所有对象，无论创建该对象的线程是什么。 这包括基本类型的包装类(例如Byte，Integer，Long等)。 无论是创建对象并将其分配给局部变量，还是创建为另一个对象的成员变量，该对象仍然存储在堆上

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125559314-531291164.png)

局部变量可以是基本类型，在这种情况下，它完全保留在线程堆栈上

局部变量也可以是对象的引用，在这种情况下，引用(局部变量)存储在线程堆栈中，但是对象本身存储在堆(Heap)上

对象的成员变量与对象本身一起存储在堆上，当成员变量是基本类型时，以及它是对象的引用时都是如此

静态类变量也与类定义一起存储在堆上

### 线程栈如何访问堆上对象?

所有具有对象引用的线程都可以访问堆上的对象，当一个线程有权访问一个对象时，它也可以访问该对象的成员变量，如果两个线程同时在同一个对象上调用一个方法，它们都可以访问该对象的成员变量，但每个线程都有自己的局部变量副本

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125614853-833023087.png)

两个线程有一组局部变量，其中一个局部变量(局部变量2)指向堆上的共享对象(对象3) 。两个线程各自对同一对象具有不同的引用，它们的引用是局部变量，因此存储在每个线程的线程堆栈中(在每个线程堆栈上) 。但是，这两个不同的引用指向堆上的同一个对象

注意：共享对象(对象3)如何将对象2和对象4作为成员变量引用(由对象3到对象2和对象4的箭头所示) ，通过对象3中的这些成员变量引用，两个线程可以访问对象2和对象4.

该图还显示了一个局部变量，该变量指向堆上的两个不同对象，在这种情况下，引用指向两个不同的对象(对象1和对象5)，而不是同一个对象，理论上，如果两个线程都引用了两个对象，则两个线程都可以访问对象1和对象5，但是在上图中，每个线程只引用了两个对象中的一个

### 线程栈访问堆示例

那么，什么样的Java代码可以导致上面的内存图? 好吧，代码就像下面的代码一样简单：

```java
public Class MyRunnable implements Runnable() {

    public void run() {
        methodOne();
    }

    public void methodOne() {
        int localVariable1 = 45;

        MySharedObject localVariable2 =
            MySharedObject.sharedInstance;

        //... do more with local variables.

        methodTwo();
    }

    public void methodTwo() {
        Integer localVariable1 = new Integer(99);

        //... do more with local variable.
    }
}

public Class MySharedObject {

    //static variable pointing to instance of MySharedObject

    public static final MySharedObject sharedInstance =
        new MySharedObject();


    //member variables pointing to two objects on the heap

    public Integer object2 = new Integer(22);
    public Integer object4 = new Integer(44);

    public long member1 = 12345;
    public long member1 = 67890;
}
```

如果两个线程正在执行run()方法，则前面显示的图表将是结果 run()方法调用methodOne()，methodOne()调用methodTwo()

methodOne()声明一个局部基本类型变量(类型为int的localVariable1)和一个局部变量，它是一个对象引用(localVariable2)

执行methodOne()的每个线程将在各自的线程堆栈上创建自己的localVariable1和localVariable2副本 localVariable1变量将完全相互分离，只存在于每个线程的线程堆栈中 一个线程无法看到另一个线程对其localVariable1副本所做的更改

执行methodOne()的每个线程也将创建自己的localVariable2副本 但是，localVariable2的两个不同副本最终都指向堆上的同一个对象 代码将localVariable2设置为指向静态变量引用的对象 静态变量只有一个副本，此副本存储在堆上 因此，localVariable2的两个副本最终都指向静态变量指向的MySharedObject的同一个实例 MySharedObject实例也存储在堆上 它对应于上图中的对象3

注意MySharedObject类还包含两个成员变量 成员变量本身与对象一起存储在堆上 两个成员变量指向另外两个Integer对象 这些Integer对象对应于上图中的Object 2和Object 4

另请注意methodTwo()如何创建名为localVariable1的局部变量 此局部变量是对Integer对象的对象引用 该方法将localVariable1引用设置为指向新的Integer实例 localVariable1引用将存储在执行methodTwo()的每个线程的一个副本中 实例化的两个Integer对象将存储在堆上，但由于该方法每次执行该方法时都会创建一个新的Integer对象，因此执行此方法的两个线程将创建单独的Integer实例 在methodTwo()中创建的Integer对象对应于上图中的Object 1和Object 5

另请注意类型为long的MySharedObject类中的两个成员变量，它们是基本类型 由于这些变量是成员变量，因此它们仍与对象一起存储在堆上 只有局部变量存储在线程堆栈中

## JMM与硬件内存结构关系

### 硬件内存结构简介

现代硬件内存架构与内部Java内存模型略有不同，了解硬件内存架构也很重要，以了解Java内存模型如何与其一起工作。本节介绍了常见的硬件内存架构，后面的部分将介绍Java内存模型如何与其配合使用

这是现代计算机硬件架构的简化图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125642855-1399013160.png)

现代计算机通常有2个或更多CPU。其中一些CPU也可能有多个内核，关键是，在具有2个或更多CPU的现代计算机上，可以同时运行多个线程，每个CPU都能够在任何给定时间运行一个线程，这意味着如果您的Java应用程序是多线程的，线程真的在可能同时运行.

每个CPU基本上都包含一组在CPU内存中的寄存器。CPU可以在这些寄存器上执行的操作比在主存储器中对变量执行的操作快得多，这是因为CPU可以比访问主存储器更快地访问这些寄存器

每个CPU还可以具有CPU高速缓存存储器层。事实上，大多数现代CPU都有一些大小的缓存存储层，CPU可以比主存储器更快地访问其高速缓存存储器，但通常不会像访问其内部寄存器那样快。因此，CPU高速缓存存储器介于内部寄存器和主存储器的速度之间。某些CPU可能有多个缓存层(级别1和级别2)，但要了解Java内存模型如何与内存交互，这一点并不重要，重要的是要知道CPU可以有某种缓存存储层

计算机还包含主存储区(RAM) 。所有CPU都可以访问主内存，主存储区通常比CPU的高速缓存存储器大得多同时访问速度也就较慢.

通常，当CPU需要访问主存储器时，它会将部分主存储器读入其CPU缓存 它甚至可以将部分缓存读入其内部寄存器，然后对其执行操作，当CPU需要将结果写回主存储器时，它会将值从其内部寄存器刷新到高速缓冲存储器，并在某些时候将值刷新回主存储器

### JMM与硬件内存连接 - 引入

如前所述，Java内存模型和硬件内存架构是不同的。硬件内存架构不区分线程堆栈和堆，在硬件上，线程堆栈和堆都位于主存储器中，线程堆栈和堆的一部分有时可能存在于CPU高速缓存和内部CPU寄存器中。这在图中说明：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125659691-1481146703.png)

当对象和变量可以存储在计算机的各种不同存储区域中时，可能会出现某些问题，两个主要问题是：

- Visibility of thread updates (writes) to shared variables.
- Race conditions when reading, checking and writing shared variables. 以下各节将解释这两个问题

### JMM与硬件内存连接 - 对象共享后的可见性

如果两个或多个线程共享一个对象，而没有正确使用volatile声明或同步，则一个线程对共享对象的更新可能对其他线程不可见

想象一下，共享对象最初存储在主存储器中，然后，在CPU上运行的线程将共享对象读入其CPU缓存中，它在那里对共享对象进行了更改，只要CPU缓存尚未刷新回主内存，共享对象的更改版本对于在其他CPU上运行的线程是不可见的。这样，每个线程最终都可能拥有自己的共享对象副本，每个副本都位于不同的CPU缓存中

下图描绘了该情况，在左CPU上运行的一个线程将共享对象复制到其CPU缓存中，并将其count变量更改为2，对于在右边的CPU上运行的其他线程，此更改不可见，因为计数更新尚未刷新回主内存中.

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125715941-1639891955.png)

要解决此问题，您可以使用Java的volatile关键字 volatile关键字可以确保直接从主内存读取给定变量，并在更新时始终写回主内存

### JMM与硬件内存连接 - 竞态条件

如果两个或多个线程共享一个对象，并且多个线程更新该共享对象中的变量，则可能会出现竞态

想象一下，如果线程A将共享对象的变量计数读入其CPU缓存中，想象一下，线程B也做同样的事情，但是进入不同的CPU缓存，现在，线程A将一个添加到count，而线程B执行相同的操作，现在var1已经增加了两次，每个CPU缓存一次

如果这些增量是按先后顺序执行的，则变量计数将增加两次并将原始值+ 2写回主存储器

但是，两个增量同时执行而没有适当的同步 无论线程A和B中哪一个将其更新后的计数版本写回主存储器，更新的值将仅比原始值高1，尽管有两个增量

该图说明了如上所述的竞争条件问题的发生：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125733008-658920295.png)

要解决此问题，您可以使用Java synchronized块 同步块保证在任何给定时间只有一个线程可以进入代码的给定关键部分，同步块还保证在同步块内访问的所有变量都将从主存储器中读入，当线程退出同步块时，所有更新的变量将再次刷新回主存储器，无论变量是不是声明为volatile

## 参考文章

- http://tutorials.jenkov.com/Java-concurrency/Java-memory-model.html
- https://blog.csdn.net/weixin_39596082/article/details/88093274







# JVM 基础 - Java 内存模型详解

> 本文主要转载自 Info 上[深入理解Java内存模型](https://www.infoq.cn/article/Java_memory_model/), 作者程晓明，这篇文章对JMM讲的很清楚了，大致分三部分：重排序与顺序一致性；三个同步原语（lock，volatile，final）的内存语义，重排序规则及在处理器中的实现；Java 内存模型的设计，及其与处理器内存模型和顺序一致性内存模型的关系



## 基础

### 并发编程模型的分类

在并发编程中，我们需要处理两个关键问题：线程之间如何通信及线程之间如何同步（这里的线程是指并发执行的活动实体）通信是指线程之间以何种机制来交换信息。在命令式编程中，线程之间的通信机制有两种：共享内存和消息传递

在共享内存的并发模型里，线程之间共享程序的公共状态，线程之间通过写 - 读内存中的公共状态来隐式进行通信；在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过明确的发送消息来显式进行通信

同步是指程序用于控制不同线程之间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的，程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行；在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的

Java 的并发采用的是共享内存模型，Java 线程之间的通信总是隐式进行，整个通信过程对程序员完全透明。如果编写多线程程序的 Java 程序员不理解隐式进行的线程之间通信的工作机制，很可能会遇到各种奇怪的内存可见性问题

### Java 内存模型的抽象

在 Java 中，所有实例域、静态域和数组元素存储在堆内存中，堆内存在线程之间共享（本文使用“共享变量”这个术语代指实例域，静态域和数组元素）局部变量（Local variables），方法定义参数（Java 语言规范称之为 formal method parameters）和异常处理器参数（exception handler parameters）不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响

Java 线程之间的通信由 Java 内存模型（本文简称为 JMM）控制，JMM 决定一个线程对共享变量的写入何时对另一个线程可见。从抽象的角度来看，JMM 定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读 / 写共享变量的副本，本地内存是 JMM 的一个抽象概念，并不真实存在，它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。Java 内存模型的抽象示意图如下：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125851116-1264932945.png)

从上图来看，线程 A 与线程 B 之间如要通信的话，必须要经历下面 2 个步骤：

- 首先，线程 A 把本地内存 A 中更新过的共享变量刷新到主内存中去
- 然后，线程 B 到主内存中去读取线程 A 之前已更新过的共享变量

下面通过示意图来说明这两个步骤：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125902213-1882865947.png)

如上图所示，本地内存 A 和 B 有主内存中共享变量 x 的副本，假设初始时，这三个内存中的 x 值都为 0，线程 A 在执行时，把更新后的 x 值（假设值为 1）临时存放在自己的本地内存 A 中，当线程 A 和线程 B 需要通信时，线程 A 首先会把自己本地内存中修改后的 x 值刷新到主内存中，此时主内存中的 x 值变为了 1，随后，线程 B 到主内存中去读取线程 A 更新后的 x 值，此时线程 B 的本地内存的 x 值也变为了 1

从整体来看，这两个步骤实质上是线程 A 在向线程 B 发送消息，而且这个通信过程必须要经过主内存，JMM 通过控制主内存与每个线程的本地内存之间的交互，来为 Java 程序员提供内存可见性保证

### 重排序

在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

- 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序
- 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行，如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序
- 内存系统的重排序。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行

从 Java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125915303-1975089555.png)

上述的 1 属于编译器重排序，2 和 3 属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性问题。对于编译器，JMM 的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）；对于处理器重排序，JMM 的处理器重排序规则会要求 Java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel 称之为 memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）

JMM 属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证

### 处理器重排序与内存屏障指令

现代的处理器使用写缓冲区来临时保存向内存写入的数据。写缓冲区可以保证指令流水线持续运行，它可以避免由于处理器停顿下来，等待向内存写入数据而产生的延迟。同时，通过以批处理的方式刷新写缓冲区，以及合并写缓冲区中对同一内存地址的多次写，可以减少对内存总线的占用。虽然写缓冲区有这么多好处，但每个处理器上的写缓冲区，仅仅对它所在的处理器可见，这个特性会对内存操作的执行顺序产生重要的影响：处理器对内存的读 / 写操作的执行顺序，不一定与内存实际发生的读 / 写操作顺序一致！为了具体说明，请看下面示例：

```java
// Processor A
a = 1; //A1  
x = b; //A2

// Processor B
b = 2; //B1  
y = a; //B2

// 初始状态：a = b = 0；处理器允许执行后得到结果：x = y = 0
```

假设处理器 A 和处理器 B 按程序的顺序并行执行内存访问，最终却可能得到 x = y = 0 的结果具体的原因如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125938776-399650423.png)

这里处理器 A 和处理器 B 可以同时把共享变量写入自己的写缓冲区（A1，B1），然后从内存中读取另一个共享变量（A2，B2），最后才把自己写缓存区中保存的脏数据刷新到内存中（A3，B3），当以这种时序执行时，程序就可以得到 x = y = 0 的结果

从内存操作实际发生的顺序来看，直到处理器 A 执行 A3 来刷新自己的写缓存区，写操作 A1 才算真正执行了。虽然处理器 A 执行内存操作的顺序为：A1->A2，但内存操作实际发生的顺序却是：A2->A1。此时，处理器 A 的内存操作顺序被重排序了（处理器 B 的情况和处理器 A 一样，这里就不赘述了）

这里的关键是，由于写缓冲区仅对自己的处理器可见，它会导致处理器执行内存操作的顺序可能会与内存实际的操作执行顺序不一致。由于现代的处理器都会使用写缓冲区，因此现代的处理器都会允许对写 - 读操作做重排序

下面是常见处理器允许的重排序类型的列表：

|           | Load-Load | Load-Store | Store-Store | Store-Load | 数据依赖 |
| --------- | --------- | ---------- | ----------- | ---------- | -------- |
| sparc-TSO | N         | N          | N           | Y          | N        |
| x86       | N         | N          | N           | Y          | N        |
| ia64      | Y         | Y          | Y           | Y          | N        |
| PowerPC   | Y         | Y          | Y           | Y          | N        |

上表单元格中的“N”表示处理器不允许两个操作重排序，“Y”表示允许重排序

从上表我们可以看出：常见的处理器都允许 Store-Load 重排序；常见的处理器都不允许对存在数据依赖的操作做重排序。sparc-TSO 和 x86 拥有相对较强的处理器内存模型，它们仅允许对写 - 读操作做重排序（因为它们都使用了写缓冲区）

- ※注 1：sparc-TSO 是指以 TSO(Total Store Order) 内存模型运行时，sparc 处理器的特性
- ※注 2：上表中的 x86 包括 x64 及 AMD64
- ※注 3：由于 ARM 处理器的内存模型与 PowerPC 处理器的内存模型非常类似，本文将忽略它
- ※注 4：数据依赖性后文会专门说明

为了保证内存可见性，Java 编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序。JMM 把内存屏障指令分为下列四类：

| 屏障类型            | 指令示例                   | 说明                                                         |
| ------------------- | -------------------------- | ------------------------------------------------------------ |
| LoadLoad Barriers   | Load1; LoadLoad; Load2     | 确保 Load1 数据的装载，之前于 Load2 及所有后续装载指令的装载 |
| StoreStore Barriers | Store1; StoreStore; Store2 | 确保 Store1 数据对其他处理器可见（刷新到内存），之前于 Store2 及所有后续存储指令的存储 |
| LoadStore Barriers  | Load1; LoadStore; Store2   | 确保 Load1 数据装载，之前于 Store2 及所有后续的存储指令刷新到内存 |
| StoreLoad Barriers  | Store1; StoreLoad; Load2   | 确保 Store1 数据对其他处理器变得可见（指刷新到内存），之前于 Load2 及所有后续装载指令的装载 |

StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令

StoreLoad Barriers 是一个“全能型”的屏障，它同时具有其他三个屏障的效果。现代大多处理器都支持该屏障（其他类型的屏障不一定被所有处理器支持），执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中（buffer fully flush）

### happens-before

从 JDK5 开始，Java 使用新的 JSR -133 内存模型（本文除非特别说明，针对的都是 JSR- 133 内存模型）。JSR-133 提出了 happens-before 的概念，通过这个概念来阐述操作之间的内存可见性。如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在 happens-before 关系，这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。与程序员密切相关的 happens-before 规则如下：

- 程序顺序规则：一个线程中的每个操作，happens- before 于该线程中的任意后续操作
- 监视器锁规则：对一个监视器锁的解锁，happens- before 于随后对这个监视器锁的加锁
- volatile 变量规则：对一个 volatile 域的写，happens- before 于任意后续对这个 volatile 域的读
- 传递性：如果 A happens- before B，且 B happens- before C，那么 A happens- before C

注意，两个操作之间具有 happens-before 关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before 仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前（the first is visible to and ordered before the second）happens- before 的定义很微妙，后文会具体说明 happens-before 为什么要这么定义

happens-before 与 JMM 的关系如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511125957018-1988889757.png)

如上图所示，一个 happens-before 规则通常对应于多个编译器重排序规则和处理器重排序规则。对于 Java 程序员来说，happens-before 规则简单易懂，它避免程序员为了理解 JMM 提供的内存可见性保证而去学习复杂的重排序规则以及这些规则的具体实现

## 重排序

### 数据依赖性

如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。数据依赖分下列三种类型：

| 名称   | 代码示例     | 说明                         |
| ------ | ------------ | ---------------------------- |
| 写后读 | a = 1;b = a; | 写一个变量之后，再读这个位置 |
| 写后写 | a = 1;a = 2; | 写一个变量之后，再写这个变量 |
| 读后写 | a = b;b = 1; | 读一个变量之后，再写这个变量 |

上面三种情况，只要重排序两个操作的执行顺序，程序的执行结果将会被改变

前面提到过，编译器和处理器可能会对操作做重排序，编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序

注意，这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑

### as-if-serial 语义

as-if-serial 语义的意思指：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。编译器，runtime 和处理器都必须遵守 as-if-serial 语义

为了遵守 as-if-serial 语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作可能被编译器和处理器重排序。为了具体说明，请看下面计算圆面积的代码示例：

```java
double pi  = 3.14;    //A
double r   = 1.0;     //B
double area = pi * r * r; //C
```

上面三个操作的数据依赖关系如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130025035-1094094194.png)

如上图所示，A 和 C 之间存在数据依赖关系，同时 B 和 C 之间也存在数据依赖关系，因此在最终执行的指令序列中，C 不能被重排序到 A 和 B 的前面（C 排到 A 和 B 的前面，程序的结果将会被改变），但 A 和 B 之间没有数据依赖关系，编译器和处理器可以重排序 A 和 B 之间的执行顺序。下图是该程序的两种执行顺序：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130034829-2081555176.png)

as-if-serial 语义把单线程程序保护了起来，遵守 as-if-serial 语义的编译器，runtime 和处理器共同为编写单线程程序的程序员创建了一个幻觉：单线程程序是按程序的顺序来执行的，as-if-serial 语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题

### 程序顺序规则

根据 happens- before 的程序顺序规则，上面计算圆的面积的示例代码存在三个 happens- before 关系：

- A happens- before B；
- B happens- before C；
- A happens- before C；

这里的第 3 个 happens- before 关系，是根据 happens- before 的传递性推导出来的

这里 A happens- before B，但实际执行时 B 却可以排在 A 之前执行（看上面的重排序后的执行顺序）。在第一章提到过，如果 A happens- before B，JMM 并不要求 A 一定要在 B 之前执行，JMM 仅仅要求前一个操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前。这里操作 A 的执行结果不需要对操作 B 可见；而且重排序操作 A 和操作 B 后的执行结果，与操作 A 和操作 B 按 happens- before 顺序执行的结果一致，在这种情况下，JMM 会认为这种重排序并不非法（not illegal），JMM 允许这种重排序

在计算机中，软件技术和硬件技术有一个共同的目标：在不改变程序执行结果的前提下，尽可能的开发并行度编译器和处理器遵从这一目标，从 happens- before 的定义我们可以看出，JMM 同样遵从这一目标

### 重排序对多线程的影响

现在让我们来看看，重排序是否会改变多线程程序的执行结果。请看下面的示例代码：

```java
Class ReorderExample {
    int a = 0;
    boolean flag = false;

    public void writer() {
        a = 1;                   //1
        flag = true;             //2
    }

    Public void reader() {
        if (flag) {                //3
            int i =  a * a;        //4
            ……
        }
    }
}
```

flag 变量是个标记，用来标识变量 a 是否已被写入。这里假设有两个线程 A 和 B，A 首先执行 writer() 方法，随后 B 线程接着执行 reader() 方法，线程 B 在执行操作 4 时，能否看到线程 A 在操作 1 对共享变量 a 的写入?

答案是：不一定能看到

由于操作 1 和操作 2 没有数据依赖关系，编译器和处理器可以对这两个操作重排序；同样，操作 3 和操作 4 没有数据依赖关系，编译器和处理器也可以对这两个操作重排序。让我们先来看看，当操作 1 和操作 2 重排序时，可能会产生什么效果? 请看下面的程序执行时序图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130057455-741238443.png)

如上图所示，操作 1 和操作 2 做了重排序。程序执行时，线程 A 首先写标记变量 flag，随后线程 B 读这个变量，由于条件判断为真，线程 B 将读取变量 a。此时，变量 a 还根本没有被线程 A 写入，在这里多线程程序的语义被重排序破坏了！

※注：本文统一用红色的虚箭线表示错误的读操作，用绿色的虚箭线表示正确的读操作

下面再让我们看看，当操作 3 和操作 4 重排序时会产生什么效果（借助这个重排序，可以顺便说明控制依赖性）。下面是操作 3 和操作 4 重排序后，程序的执行时序图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130109868-1895649963.png)

在程序中，操作 3 和操作 4 存在控制依赖关系。当代码中存在控制依赖性时，会影响指令序列执行的并行度。为此，编译器和处理器会采用猜测（Speculation）执行来克服控制相关性对并行度的影响。以处理器的猜测执行为例，执行线程 B 的处理器可以提前读取并计算 a*a，然后把计算结果临时保存到一个名为重排序缓冲（reorder buffer ROB）的硬件缓存中。当接下来操作 3 的条件判断为真时，就把该计算结果写入变量 i 中

从图中我们可以看出，猜测执行实质上对操作 3 和 4 做了重排序，重排序在这里破坏了多线程程序的语义！

在单线程程序中，对存在控制依赖的操作重排序，不会改变执行结果（这也是 as-if-serial 语义允许对存在控制依赖的操作做重排序的原因）；但在多线程程序中，对存在控制依赖的操作重排序，可能会改变程序的执行结果

## 顺序一致性

### 数据竞争与顺序一致性保证

当程序未正确同步时，就会存在数据竞争。Java 内存模型规范对数据竞争的定义如下：

- 在一个线程中写一个变量，
- 在另一个线程读同一个变量，
- 而且写和读没有通过同步来排序

当代码中包含数据竞争时，程序的执行往往产生违反直觉的结果（前一章的示例正是如此）。如果一个多线程程序能正确同步，这个程序将是一个没有数据竞争的程序

JMM 对正确同步的多线程程序的内存一致性做了如下保证：

- 如果程序是正确同步的，程序的执行将具有顺序一致性（sequentially consistent）-- 即程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同（马上我们将会看到，这对于程序员来说是一个极强的保证）。这里的同步是指广义上的同步，包括对常用同步原语（lock，volatile 和 final）的正确使用

### 顺序一致性内存模型

顺序一致性内存模型是一个被计算机科学家理想化了的理论参考模型，它为程序员提供了极强的内存可见性保证。顺序一致性内存模型有两大特性：

- 一个线程中的所有操作必须按照程序的顺序来执行 +（不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见。顺序一致性内存模型为程序员提供的视图如下：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130149693-236654967.png)

在概念上，顺序一致性模型有一个单一的全局内存，这个内存通过一个左右摆动的开关可以连接到任意一个线程。同时，每一个线程必须按程序的顺序来执行内存读 / 写操作。从上图我们可以看出，在任意时间点最多只能有一个线程可以连接到内存。当多个线程并发执行时，图中的开关装置能把所有线程的所有内存读 / 写操作串行化

为了更好的理解，下面我们通过两个示意图来对顺序一致性模型的特性做进一步的说明

假设有两个线程 A 和 B 并发执行，其中 A 线程有三个操作，它们在程序中的顺序是：A1->A2->A3；B 线程也有三个操作，它们在程序中的顺序是：B1->B2->B3

假设这两个线程使用监视器来正确同步：A 线程的三个操作执行后释放监视器，随后 B 线程获取同一个监视器。那么程序在顺序一致性模型中的执行效果将如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130208332-434193292.png)

现在我们再假设这两个线程没有做同步，下面是这个未同步程序在顺序一致性模型中的执行示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130217652-116159883.png)

未同步程序在顺序一致性模型中虽然整体执行顺序是无序的，但所有线程都只能看到一个一致的整体执行顺序。以上图为例，线程 A 和 B 看到的执行顺序都是：B1->A1->A2->B2->A3->B3。之所以能得到这个保证是因为顺序一致性内存模型中的每个操作必须立即对任意线程可见

但是，在 JMM 中就没有这个保证。未同步程序在 JMM 中不但整体的执行顺序是无序的，而且所有线程看到的操作执行顺序也可能不一致。比如，在当前线程把写过的数据缓存在本地内存中，且还没有刷新到主内存之前，这个写操作仅对当前线程可见；从其他线程的角度来观察，会认为这个写操作根本还没有被当前线程执行，只有当前线程把本地内存中写过的数据刷新到主内存之后，这个写操作才能对其他线程可见。在这种情况下，当前线程和其它线程看到的操作执行顺序将不一致

### 同步程序的顺序一致性效果

下面我们对前面的示例程序 ReorderExample 用监视器来同步，看看正确同步的程序如何具有顺序一致性

请看下面的示例代码：

```java
Class SynchronizedExample {
    int a = 0;
    boolean flag = false;

    public synchronized void writer() {
        a = 1;
        flag = true;
    }

    public synchronized void reader() {
        if (flag) {
            int i = a;
            ……
        }
    }
}
```

上面示例代码中，假设 A 线程执行 writer() 方法后，B 线程执行 reader() 方法，这是一个正确同步的多线程程序。根据 JMM 规范，该程序的执行结果将与该程序在顺序一致性模型中的执行结果相同。下面是该程序在两个内存模型中的执行时序对比图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130237522-1311757059.png)

在顺序一致性模型中，所有操作完全按程序的顺序串行执行；而在 JMM 中，临界区内的代码可以重排序（但 JMM 不允许临界区内的代码“逸出”到临界区之外，那样会破坏监视器的语义），JMM 会在退出监视器和进入监视器这两个关键时间点做一些特别处理，使得线程在这两个时间点具有与顺序一致性模型相同的内存视图（具体细节后文会说明）。虽然线程 A 在临界区内做了重排序，但由于监视器的互斥执行的特性，这里的线程 B 根本无法“观察”到线程 A 在临界区内的重排序，这种重排序既提高了执行效率，又没有改变程序的执行结果

从这里我们可以看到， JMM 在具体实现上的基本方针：在不改变（正确同步的）程序执行结果的前提下，尽可能的为编译器和处理器的优化打开方便之门

### 未同步程序的执行特性

对于未同步或未正确同步的多线程程序，JMM 只提供最小安全性：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0，null，false），JMM 保证线程读操作读取到的值不会无中生有（out of thin air）的冒出来。为了实现最小安全性，JVM 在堆上分配对象时，首先会清零内存空间，然后才会在上面分配对象（JVM 内部会同步这两个操作）。因此，在已清零的内存空间（pre-zeroed memory）分配对象时，域的默认初始化已经完成了

JMM 不保证未同步程序的执行结果与该程序在顺序一致性模型中的执行结果一致。因为未同步程序在顺序一致性模型中执行时，整体上是无序的，其执行结果无法预知，保证未同步程序在两个模型中的执行结果一致毫无意义

和顺序一致性模型一样，未同步程序在 JMM 中执行时，整体上也是无序的，其执行结果也无法预知。同时，未同步程序在这两个模型中的执行特性有下面几个差异：

- 顺序一致性模型保证单线程内的操作会按程序的顺序执行，而 JMM 不保证单线程内的操作会按程序的顺序执行（比如上面正确同步的多线程程序在临界区内的重排序）。这一点前面已经讲过了，这里就不再赘述
- 顺序一致性模型保证所有线程只能看到一致的操作执行顺序，而 JMM 不保证所有线程能看到一致的操作执行顺序。这一点前面也已经讲过，这里就不再赘述
- JMM 不保证对 64 位的 long 型和 double 型变量的读 / 写操作具有原子性，而顺序一致性模型保证对所有的内存读 / 写操作都具有原子性

第 3 个差异与处理器总线的工作机制密切相关。在计算机中，数据通过总线在处理器和内存之间传递。每次处理器和内存之间的数据传递都是通过一系列步骤来完成的，这一系列步骤称之为总线事务（bus transaction）。总线事务包括读事务（read transaction）和写事务（write transaction）。读事务从内存传送数据到处理器，写事务从处理器传送数据到内存，每个事务会读 / 写内存中一个或多个物理上连续的字，这里的关键是，总线会同步试图并发使用总线的事务。在一个处理器执行总线事务期间，总线会禁止其它所有的处理器和 I/O 设备执行内存的读 / 写。下面让我们通过一个示意图来说明总线的工作机制：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130302997-631871983.png)

如上图所示，假设处理器 A，B 和 C 同时向总线发起总线事务，这时总线仲裁（bus arbitration）会对竞争作出裁决，这里我们假设总线在仲裁后判定处理器 A 在竞争中获胜（总线仲裁会确保所有处理器都能公平的访问内存）。此时处理器 A 继续它的总线事务，而其它两个处理器则要等待处理器 A 的总线事务完成后才能开始再次执行内存访问。假设在处理器 A 执行总线事务期间（不管这个总线事务是读事务还是写事务），处理器 D 向总线发起了总线事务，此时处理器 D 的这个请求会被总线禁止

总线的这些工作机制可以把所有处理器对内存的访问以串行化的方式来执行；在任意时间点，最多只能有一个处理器能访问内存。这个特性确保了单个总线事务之中的内存读 / 写操作具有原子性

在一些 32 位的处理器上，如果要求对 64 位数据的读 / 写操作具有原子性，会有比较大的开销。为了照顾这种处理器，Java 语言规范鼓励但不强求 JVM 对 64 位的 long 型变量和 double 型变量的读 / 写具有原子性。当 JVM 在这种处理器上运行时，会把一个 64 位 long/ double 型变量的读 / 写操作拆分为两个 32 位的读 / 写操作来执行。这两个 32 位的读 / 写操作可能会被分配到不同的总线事务中执行，此时对这个 64 位变量的读 / 写将不具有原子性

当单个内存操作不具有原子性，将可能会产生意想不到的后果。请看下面示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130314495-2052594806.png)

如上图所示，假设处理器 A 写一个 long 型变量，同时处理器 B 要读这个 long 型变量、处理器 A 中 64 位的写操作被拆分为两个 32 位的写操作，且这两个 32 位的写操作被分配到不同的写事务中执行。同时处理器 B 中 64 位的读操作被拆分为两个 32 位的读操作，且这两个 32 位的读操作被分配到同一个的读事务中执行。当处理器 A 和 B 按上图的时序来执行时，处理器 B 将看到仅仅被处理器 A“写了一半“的无效值

## 总结

### 处理器内存模型

顺序一致性内存模型是一个理论参考模型，JMM 和处理器内存模型在设计时通常会把顺序一致性内存模型作为参照。JMM 和处理器内存模型在设计时会对顺序一致性模型做一些放松，因为如果完全按照顺序一致性模型来实现处理器和 JMM，那么很多的处理器和编译器优化都要被禁止，这对执行性能将会有很大的影响

根据对不同类型读 / 写操作组合的执行顺序的放松，可以把常见处理器的内存模型划分为下面几种类型：

- 放松程序中写 - 读操作的顺序，由此产生了 total store ordering 内存模型（简称为 TSO）
- 在前面 1 的基础上，继续放松程序中写 - 写操作的顺序，由此产生了 partial store order 内存模型（简称为 PSO）
- 在前面 1 和 2 的基础上，继续放松程序中读 - 写和读 - 读操作的顺序，由此产生了 relaxed memory order 内存模型（简称为 RMO）和 PowerPC 内存模型

注意，这里处理器对读 / 写操作的放松，是以两个操作之间不存在数据依赖性为前提的（因为处理器要遵守 as-if-serial 语义，处理器不会对存在数据依赖性的两个内存操作做重排序）

下面的表格展示了常见处理器内存模型的细节特征：

| 内存模型名称 | 对应的处理器  | Store-Load 重排序 | Store-Store 重排序 | Load-Load 和 Load-Store 重排序 | 可以更早读取到其它处理器的写 | 可以更早读取到当前处理器的写 |
| ------------ | ------------- | ----------------- | ------------------ | ------------------------------ | ---------------------------- | ---------------------------- |
| TSO          | sparc-TSO X64 | Y                 |                    |                                |                              | Y                            |
| PSO          | sparc-PSO     | Y                 | Y                  |                                |                              | Y                            |
| RMO          | ia64          | Y                 | Y                  | Y                              |                              | Y                            |
| PowerPC      | PowerPC       | Y                 | Y                  | Y                              | Y                            | Y                            |

在这个表格中，我们可以看到所有处理器内存模型都允许写 - 读重排序，原因在第一章以说明过：它们都使用了写缓存区，写缓存区可能导致写 - 读操作重排序。同时，我们可以看到这些处理器内存模型都允许更早读到当前处理器的写，原因同样是因为写缓存区：由于写缓存区仅对当前处理器可见，这个特性导致当前处理器可以比其他处理器先看到临时保存在自己的写缓存区中的写

上面表格中的各种处理器内存模型，从上到下，模型由强变弱。越是追求性能的处理器，内存模型设计得会越弱。因为这些处理器希望内存模型对它们的束缚越少越好，这样它们就可以做尽可能多的优化来提高性能

由于常见的处理器内存模型比 JMM 要弱，Java 编译器在生成字节码时，会在执行指令序列的适当位置插入内存屏障来限制处理器的重排序。同时，由于各种处理器内存模型的强弱并不相同，为了在不同的处理器平台向程序员展示一个一致的内存模型，JMM 在不同的处理器中需要插入的内存屏障的数量和种类也不相同。下图展示了 JMM 在不同处理器内存模型中需要插入的内存屏障的示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130338500-1250170003.png)

如上图所示，JMM 屏蔽了不同处理器内存模型的差异，它在不同的处理器平台之上为 Java 程序员呈现了一个一致的内存模型

### JMM，处理器内存模型与顺序一致性内存模型之间的关系

JMM 是一个语言级的内存模型，处理器内存模型是硬件级的内存模型，顺序一致性内存模型是一个理论参考模型。下面是语言内存模型，处理器内存模型和顺序一致性内存模型的强弱对比示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130353981-807312795.png)

从上图我们可以看出：常见的 4 种处理器内存模型比常用的 3 种语言内存模型要弱，处理器内存模型和语言内存模型都比顺序一致性内存模型要弱。同处理器内存模型一样，越是追求执行性能的语言，内存模型设计的会越弱

### JMM 的设计

从 JMM 设计者的角度来说，在设计 JMM 时，需要考虑两个关键因素：

- 程序员对内存模型的使用。程序员希望内存模型易于理解，易于编程。程序员希望基于一个强内存模型来编写代码
- 编译器和处理器对内存模型的实现。编译器和处理器希望内存模型对它们的束缚越少越好，这样它们就可以做尽可能多的优化来提高性能。编译器和处理器希望实现一个弱内存模型

由于这两个因素互相矛盾，所以 JSR-133 专家组在设计 JMM 时的核心目标就是找到一个好的平衡点：一方面要为程序员提供足够强的内存可见性保证；另一方面，对编译器和处理器的限制要尽可能的放松。下面让我们看看 JSR-133 是如何实现这一目标的

为了具体说明，请看前面提到过的计算圆面积的示例代码：

```java
double pi  = 3.14;    //A
double r   = 1.0;     //B
double area = pi * r * r; //C
```

上面计算圆的面积的示例代码存在三个 happens- before 关系：

- A happens- before B；
- B happens- before C；
- A happens- before C；

由于 A happens- before B，happens- before 的定义会要求：A 操作执行的结果要对 B 可见，且 A 操作的执行顺序排在 B 操作之前。但是从程序语义的角度来说，对 A 和 B 做重排序即不会改变程序的执行结果，也还能提高程序的执行性能（允许这种重排序减少了对编译器和处理器优化的束缚）。也就是说，上面这 3 个 happens- before 关系中，虽然 2 和 3 是必需要的，但 1 是不必要的。因此，JMM 把 happens- before 要求禁止的重排序分为了下面两类：

- 会改变程序执行结果的重排序
- 不会改变程序执行结果的重排序

JMM 对这两种不同性质的重排序，采取了不同的策略：

- 对于会改变程序执行结果的重排序，JMM 要求编译器和处理器必须禁止这种重排序
- 对于不会改变程序执行结果的重排序，JMM 对编译器和处理器不作要求（JMM 允许这种重排序）

下面是 JMM 的设计示意图：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130413146-259084478.png)

从上图可以看出两点：

- JMM 向程序员提供的 happens- before 规则能满足程序员的需求。JMM 的 happens- before 规则不但简单易懂，而且也向程序员提供了足够强的内存可见性保证（有些内存可见性保证其实并不一定真实存在，比如上面的 A happens- before B）
- JMM 对编译器和处理器的束缚已经尽可能的少。从上面的分析我们可以看出，JMM 其实是在遵循一个基本原则：只要不改变程序的执行结果（指的是单线程程序和正确同步的多线程程序），编译器和处理器怎么优化都行比如，如果编译器经过细致的分析后，认定一个锁只会被单个线程访问，那么这个锁可以被消除再比如，如果编译器经过细致的分析后，认定一个 volatile 变量仅仅只会被单个线程访问，那么编译器可以把这个 volatile 变量当作一个普通变量来对待。这些优化既不会改变程序的执行结果，又能提高程序的执行效率

### JMM 的内存可见性保证

Java 程序的内存可见性保证按程序类型可以分为下列三类：

- 单线程程序。单线程程序不会出现内存可见性问题。编译器，runtime 和处理器会共同确保单线程程序的执行结果与该程序在顺序一致性模型中的执行结果相同
- 正确同步的多线程程序。正确同步的多线程程序的执行将具有顺序一致性（程序的执行结果与该程序在顺序一致性内存模型中的执行结果相同）。这是 JMM 关注的重点，JMM 通过限制编译器和处理器的重排序来为程序员提供内存可见性保证
- 未同步 / 未正确同步的多线程程序。JMM 为它们提供了最小安全性保障：线程执行时读取到的值，要么是之前某个线程写入的值，要么是默认值（0，null，false）

下图展示了这三类程序在 JMM 中与在顺序一致性内存模型中的执行结果的异同：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230511130428477-19815475.png)****

只要多线程程序是正确同步的，JMM 保证该程序在任意的处理器平台上的执行结果，与该程序在顺序一致性内存模型中的执行结果一致

### JSR-133 对旧内存模型的修补

JSR-133 对 JDK5 之前的旧内存模型的修补主要有两个：

- 增强 volatile 的内存语义。旧内存模型允许 volatile 变量与普通变量重排序。JSR-133 严格限制 volatile 变量与普通变量的重排序，使 volatile 的写 - 读和锁的释放 - 获取具有相同的内存语义
- 增强 final 的内存语义。在旧内存模型中，多次读取同一个 final 变量的值可能会不相同。为此，JSR-133 为 final 增加了两个重排序规则。现在，final 具有了初始化安全性









# GC - Java 垃圾回收基础知识

> 垃圾收集主要是针对堆和方法区进行；程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后也会消失，因此不需要对这三个区域进行垃圾回收

## 判断一个对象是否可被回收

### 1. 引用计数算法

给对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1，引用计数为 0 的对象可被回收

两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收

正因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法

```java
public Class ReferenceCountingGC {

    public Object instance = null;

    public static void main(String[] args) {
        ReferenceCountingGC objectA = new ReferenceCountingGC();
        ReferenceCountingGC objectB = new ReferenceCountingGC();
        objectA.instance = objectB;
        objectB.instance = objectA;
    }
}
```

### 2. 可达性分析算法

通过 GC Roots 作为起始点进行搜索，能够到达到的对象都是存活的，不可达的对象可被回收

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133551077-1697233072.png)

Java 虚拟机使用该算法来判断对象是否可被回收，在 Java 中 GC Roots 一般包含以下内容:

- 虚拟机栈中引用的对象
- 本地方法栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象

### 3. 方法区的回收

因为方法区主要存放永久代对象，而永久代对象的回收率比新生代低很多，因此在方法区上进行回收性价比不高

主要是对常量池的回收和对类的卸载

在大量使用反射、动态代理、CGLib 等 ByteCode 框架、动态生成 JSP 以及 OSGi 这类频繁自定义 ClassLoader 的场景都需要虚拟机具备类卸载功能，以保证不会出现内存溢出

类的卸载条件很多，需要满足以下三个条件，并且满足了也不一定会被卸载:

- 该类所有的实例都已经被回收，也就是堆中不存在该类的任何实例
- 加载该类的 ClassLoader 已经被回收
- 该类对应的 Class 对象没有在任何地方被引用，也就无法在任何地方通过反射访问该类方法

可以通过 -XnoClassgc 参数来控制是否对类进行卸载

### 4. finalize()

finalize() 类似 C++ 的析构函数，用来做关闭外部资源等工作。但是 try-finally 等方式可以做的更好，并且该方法运行代价高昂，不确定性大，无法保证各个对象的调用顺序，因此最好不要使用

当一个对象可被回收时，如果需要执行该对象的 finalize() 方法，那么就有可能通过在该方法中让对象重新被引用，从而实现自救，自救只能进行一次，如果回收的对象之前调用了 finalize() 方法自救，后面回收时不会调用 finalize() 方法

## 引用类型

无论是通过引用计算算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关

Java 具有四种强度不同的引用类型

### 1. 强引用

被强引用关联的对象不会被回收

使用 new 一个新对象的方式来创建强引用

```java
Object obj = new Object();
```

### 2. 软引用

被软引用关联的对象只有在内存不够的情况下才会被回收

使用 SoftReference 类来创建软引用

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

### 3. 弱引用

被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前

使用 WeakReference 类来实现弱引用

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

### 4. 虚引用

又称为幽灵引用或者幻影引用。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得一个对象

为一个对象设置虚引用关联的唯一目的就是能在这个对象被回收时收到一个系统通知

使用 PhantomReference 来实现虚引用

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```

## 垃圾回收算法

### 1. 标记 - 清除

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133649269-1964599761.jpg)

将存活的对象进行标记，然后清理掉未被标记的对象

不足:

- 标记和清除过程效率都不高；
- 会产生大量不连续的内存碎片，导致无法给大对象分配内存

### 2. 标记 - 整理

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133704896-6353708.jpg)

让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存

### 3. 复制

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133718928-2140426049.jpg)

将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理

主要不足是只使用了内存的一半

现在的商业虚拟机都采用这种收集算法来回收新生代，但是并不是将新生代划分为大小相等的两块，而是分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 空间和其中一块 Survivor在回收时，将 Eden 和 Survivor 中还存活着的对象一次性复制到另一块 Survivor 空间上，最后清理 Eden 和使用过的那一块 Survivor

HotSpot 虚拟机的 Eden 和 Survivor 的大小比例默认为 8:1，保证了内存的利用率达到 90%。如果每次回收有多于 10% 的对象存活，那么一块 Survivor 空间就不够用了，此时需要依赖于老年代进行分配担保，也就是借用老年代的空间存储放不下的对象

### 4. 分代收集

现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法

一般将堆分为新生代和老年代

- 新生代使用: 复制算法
- 老年代使用: 标记 - 清除 或者 标记 - 整理 算法

## 垃圾收集器

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133740940-821185983.jpg)

以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用

- 单线程与多线程: 单线程指的是垃圾收集器只使用一个线程进行收集，而多线程使用多个线程；
- 串行与并行: 串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并行指的是垃圾收集器和用户程序同时执行除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行

### 1. Serial 收集器

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133758718-1031349757.jpg)

Serial 翻译为串行，也就是说它以串行的方式执行

它是单线程的收集器，只会使用一个线程进行垃圾收集工作

它的优点是简单高效，对于单个 CPU 环境来说，由于没有线程交互的开销，因此拥有最高的单线程收集效率

它是 Client 模式下的默认新生代收集器，因为在用户的桌面应用场景下，分配给虚拟机管理的内存一般来说不会很大。Serial 收集器收集几十兆甚至一两百兆的新生代停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿是可以接受的。



### 2. ParNew 收集器

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133823312-674052050.jpg)

它是 Serial 收集器的多线程版本

是 Server 模式下的虚拟机首选新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合工作

默认开启的线程数量与 CPU 数量相同，可以使用 -XX:ParallelGCThreads 参数来设置线程数

### 3. Parallel Scavenge 收集器

与 ParNew 一样是多线程收集器

其它收集器关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，它被称为“吞吐量优先”收集器。这里的吞吐量指 CPU 用于运行用户代码的时间占总时间的比值

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的: 新生代空间变小，垃圾回收变得频繁，导致吞吐量下降

可以通过一个开关参数打开 GC 自适应的调节策略(GC Ergonomics)，就不需要手动指定新生代的大小(-Xmn)、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量

### 4. Serial Old 收集器

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133844826-131067361.jpg)

是 Serial 收集器的老年代版本，也是给 Client 模式下的虚拟机使用如果用在 Server 模式下，它有两大用途:

- 在 JDK 1.5 以及之前版本(Parallel Old 诞生以前)中与 Parallel Scavenge 收集器搭配使用
- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用

### 5. Parallel Old 收集器

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133900340-42335374.jpg)

是 Parallel Scavenge 收集器的老年代版本

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器

### 6. CMS 收集器

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133916547-444153777.jpg)

CMS(Concurrent Mark Sweep)，Mark Sweep 指的是标记 - 清除算法

分为以下四个流程:

- 初始标记: 仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿
- 并发标记: 进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿
- 重新标记: 为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿
- 并发清除: 不需要停顿

在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿

具有以下缺点:

- 吞吐量低: 低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高
- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS
- 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC

### 7. G1 收集器

G1(Garbage-First)，它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 G1 可以直接对新生代和老年代一起回收

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133936367-2136315669.png)

G1 把堆划分成多个大小相等的独立区域(Region)，新生代和老年代不再物理隔离

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133947076-574632200.png)

通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收，这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能通过记录每个 Region 垃圾回收时间以及回收所获得的空间(这两个值是通过过去回收的经验获得)，并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region

每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region，通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描

![image](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230516133958346-626892146.jpg)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤:

- 初始标记
- 并发标记
- 最终标记: 为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行
- 筛选回收: 首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率

具备如下特点:

- 空间整合: 整体来看是基于“标记 - 整理”算法实现的收集器，从局部(两个 Region 之间)上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片
- 可预测的停顿: 能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒

更详细内容请参考: [Getting Started with the G1 Garbage Collector](http://www.oracle.com/webfolder/technetwork/tutorials/obe/Java/G1GettingStarted/index.html)

## 内存分配与回收策略

### Minor GC、Major GC、Full GC

JVM 在进行 GC 时，并非每次都对堆内存（新生代、老年代；方法区）区域一起回收的，大部分时候回收的都是指新生代

针对 HotSpot VM 的实现，它里面的 GC 按照回收区域又分为两大类：部分收集（Partial GC），整堆收集（Full GC）

- 部分收集：不是完整收集整个 Java 堆的垃圾收集其中又分为： 
	- 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
	- 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集 
		- 目前，只有 CMS GC 会有单独收集老年代的行为
		- 很多时候 Major GC 会和 Full GC 混合使用，需要具体分辨是老年代回收还是整堆回收
	- 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集 
		- 目前只有 G1 GC 会有这种行为
- 整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾

### 内存分配策略

#### 1. 对象优先在 Eden 分配

大多数情况下，对象在新生代 Eden 区分配，当 Eden 区空间不够时，发起 Minor GC

#### 2. 大对象直接进入老年代

大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组

经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象

-XX:PretenureSizeThreshold，大于此值的对象直接在老年代分配，避免在 Eden 区和 Survivor 区之间的大量内存复制

#### 3. 长期存活的对象进入老年代

为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄就增加 1 岁，增加到一定年龄则移动到老年代中

-XX:MaxTenuringThreshold 用来定义年龄的阈值

#### 4. 动态对象年龄判定

虚拟机并不是永远地要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄所有对象大小的总和大于 Survivor 空间的一半，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄

#### 5. 空间分配担保

在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的

如果不成立的话虚拟机会查看 HandlePromotionFailure 设置值是否允许担保失败，如果允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC；如果小于，或者 HandlePromotionFailure 设置不允许冒险，那么就要进行一次 Full GC

### Full GC 的触发条件

对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下条件:

#### 1. 调用 System.gc()

只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存

#### 2. 老年代空间不足

老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等

为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间

#### 3. 空间分配担保失败

使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。具体内容请参考上面的第五小节

#### 4. JDK 1.7 及以前的永久代空间不足

在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据

当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 Java.lang.OutOfMemoryError

为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC

#### 5. Concurrent Mode Failure

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足(可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足)，便会报 Concurrent Mode Failure 错误，并触发 Full GC

## 参考

- [GC算法 垃圾收集器](https://mp.weixin.qq.com/s/olNXcRAT3PTK-hV_ehtmtw)
- [Java GC 分析](https://mp.weixin.qq.com/s/S3PcA2KIzCVB2hJmsbVzyQ)
- [Java应用频繁FullGC分析](https://yq.aliyun.com/articles/94557?spm=5176.100239.blogcont217385.73.SaQb9l)









# GC - Java 垃圾回收器之G1详解

> G1垃圾回收器是在Java7 update 4之后引入的一个新的垃圾回收器。
>
> 同优秀的CMS垃圾回收器一样，G1也是关注最小时延的垃圾回收器，也同样适合大尺寸堆内存的垃圾收集。
>
> 官方在ZGC还没有出现时也推荐使用G1来代替选择CMS。
>
> G1最大的特点是引入分区的思路，弱化了分代的概念，合理利用垃圾收集各个周期的资源，解决了其他收集器甚至CMS的众多缺陷

## 1. 概述

G1垃圾回收器是在Java7 update 4之后引入的一个新的垃圾回收器，G1是一个分代的，增量的，并行与并发的标记-复制垃圾回收器。它的设计目标是为了适应现在不断扩大的内存和不断增加的处理器数量，进一步降低暂停时间（pause time），同时兼顾良好的吞吐量。G1回收器和CMS比起来，有以下不同：

- G1垃圾回收器是**compacting**的，因此其回收得到的空间是连续的。这避免了CMS回收器因为不连续空间所造成的问题。如需要更大的堆空间，更多的floating garbage。连续空间意味着G1垃圾回收器可以不必采用空闲链表的内存分配方式，而可以直接采用bump-the-pointer的方式；
- G1回收器的内存与CMS回收器要求的内存模型有极大的不同。G1将内存划分一个个固定大小的region，每个region可以是年轻代、老年代的一个，**内存的回收是以region作为基本单位的**；
- G1还有一个及其重要的特性：**软实时**（soft real-time）。所谓的实时垃圾回收，是指在要求的时间内完成垃圾回收；“软实时”则是指，用户可以指定垃圾回收时间的限时，G1会努力在这个时限内完成垃圾回收，但是G1并不担保每次都能在这个时限内完成垃圾回收。通过设定一个合理的目标，可以让达到90%以上的垃圾回收时间都在这个时限内；

## 2. G1的内存模型

### [#](#_2-1-分区概念) 2.1 分区概念

G1分区示意图

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519162952604-1565966626.jpg)

#### 2.1.1 分区Region

G1采用了分区(Region)的思路，将整个堆空间分成若干个大小相等的内存区域，每次分配对象空间将逐段地使用内存。因此，在堆的使用上，G1并不要求对象的存储一定是物理上连续的，只要逻辑上连续即可；每个分区也不会确定地为某个代服务，可以按需在年轻代和老年代之间切换。启动时可以通过参数-XX:G1HeapRegionSize=n可指定分区大小(1MB~32MB，且必须是2的幂)，默认将整堆划分为2048个分区

#### 2.1.2 卡片Card

在每个分区内部又被分成了若干个大小为512 Byte卡片(Card)，标识堆内存最小可用粒度，所有分区的卡片将会记录在全局卡片表(Global Card Table)中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过记录卡片来查找该引用对象(见RSet)。每次对内存的回收，都是对指定分区的卡片进行处理

#### 2.1.3 堆Heap

G1同样可以通过-Xms/-Xmx来指定堆空间大小。当发生年轻代收集或混合收集时，通过计算GC与应用的耗费时间比，自动调整堆空间大小。如果GC频率太高，则通过增加堆尺寸，来减少GC频率，相应地GC占用的时间也随之降低；目标参数-XX:GCTimeRatio即为GC与应用的耗费时间比，G1默认为9，而CMS默认为99，因为CMS的设计原则是耗费在GC上的时间尽可能的少另外，当空间不足，如对象空间分配或转移失败时，G1会首先尝试增加堆空间，如果扩容失败，则发起担保的Full GC。Full GC后，堆尺寸计算结果也会调整堆空间

### 2.2 分代模型

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163020996-602244689.jpg)

#### 2.2.1 分代垃圾收集

分代垃圾收集可以将关注点集中在最近被分配的对象上，而无需整堆扫描，避免长命对象的拷贝，同时独立收集有助于降低响应时间。虽然分区使得内存分配不再要求紧凑的内存空间，但G1依然使用了分代的思想。与其他垃圾收集器类似，G1将内存在逻辑上划分为年轻代和老年代，其中年轻代又划分为Eden空间和Survivor空间。但年轻代空间并不是固定不变的，当现有年轻代分区占满时，JVM会分配新的空闲分区加入到年轻代空间

整个年轻代内存会在初始空间`-XX:G1NewSizePercent`(默认整堆5%)与最大空间(默认60%)之间动态变化，且由参数目标暂停时间`-XX:MaxGCPauseMillis`(默认200ms)、需要扩缩容的大小以`-XX:G1MaxNewSizePercent`及分区的已记忆集合(RSet)计算得到。当然，G1依然可以设置固定的年轻代大小(参数-XX:NewRatio、-Xmn)，但同时暂停目标将失去意义

#### 2.2.2 本地分配缓冲 Local allocation buffer (Lab)

值得注意的是，由于分区的思想，每个线程均可以"认领"某个分区用于线程本地的内存分配，而不需要顾及分区是否连续。因此，每个应用线程和GC线程都会独立的使用分区，进而减少同步时间，提升GC效率，这个分区称为本地分配缓冲区(Lab)

其中，应用线程可以独占一个本地缓冲区(TLAB)来创建对象，而大部分都会落入Eden区域(巨型对象或分配失败除外)，因此TLAB的分区属于Eden空间；而每次垃圾收集时，每个GC线程同样可以独占一个本地缓冲区(GCLAB)用来转移对象，每次回收会将对象复制到Suvivor空间或老年代空间；对于从Eden/Survivor空间晋升(Promotion)到Survivor/老年代空间的对象，同样有GC独占的本地缓冲区进行操作，该部分称为晋升本地缓冲区(PLAB)

### 2.3 分区模型

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163048229-2020650943.jpg)

G1对内存的使用以分区(Region)为单位，而对对象的分配则以卡片(Card)为单位

#### 2.3.1 巨形对象Humongous Region

一个大小达到甚至超过分区大小一半的对象称为巨型对象(Humongous Object)。当线程为巨型分配空间时，不能简单在TLAB进行分配，因为巨型对象的移动成本很高，而且有可能一个分区不能容纳巨型对象。因此，巨型对象会直接在老年代分配，所占用的连续空间称为巨型分区(Humongous Region)。G1内部做了一个优化，一旦发现没有引用指向巨型对象，则可直接在年轻代收集周期中被回收

巨型对象会独占一个、或多个连续分区，其中第一个分区被标记为开始巨型(StartsHumongous)，相邻连续分区被标记为连续巨型(ContinuesHumongous)。由于无法享受Lab带来的优化，并且确定一片连续的内存空间需要扫描整堆，因此确定巨型对象开始位置的成本非常高，如果可以，应用程序应避免生成巨型对象

#### 2.3.2 已记忆集合Remember Set (RSet)

在串行和并行收集器中，GC通过整堆扫描，来确定对象是否处于可达路径中。然而G1为了避免STW式的整堆扫描，在每个分区记录了一个已记忆集合(RSet)，内部类似一个反向指针，记录引用分区内对象的卡片索引。当要回收该分区时，通过扫描分区的RSet，来确定引用本分区内的对象是否存活，进而确定本分区内的对象存活情况

事实上，并非所有的引用都需要记录在RSet中，如果一个分区确定需要扫描，那么无需RSet也可以无遗漏的得到引用关系，那么引用源自本分区的对象，当然不用落入RSet中；同时，G1 GC每次都会对年轻代进行整体收集，因此引用源自年轻代的对象，也不需要在RSet中记录。最后只有老年代的分区可能会有RSet记录，这些分区称为拥有RSet分区(an RSet’s owning region)

#### 2.3.3 Per Region Table (PRT)

RSet在内部使用Per Region Table(PRT)记录分区的引用情况。由于RSet的记录要占用分区的空间，如果一个分区非常"受欢迎"，那么RSet占用的空间会上升，从而降低分区的可用空间。G1应对这个问题采用了改变RSet的密度的方式，在PRT中将会以三种模式记录引用：

- 稀少：直接记录引用对象的卡片索引。
- 细粒度：记录引用对象的分区索引。
- 粗粒度：只记录引用情况，每个分区对应一个比特位。

由上可知，粗粒度的PRT只是记录了引用数量，需要通过整堆扫描才能找出所有引用，因此扫描速度也是最慢的

### 2.4 收集集合 (CSet)

CSet收集示意图

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163115136-1259801222.jpg)

收集集合(CSet)代表每次GC暂停时回收的一系列目标分区。在任意一次收集暂停中，CSet所有分区都会被释放，内部存活的对象都会被转移到分配的空闲分区中。因此无论是年轻代收集，还是混合收集，工作的机制都是一致的。年轻代收集CSet只容纳年轻代分区，而混合收集会通过启发式算法，在老年代候选回收分区中，筛选出回收收益最高的分区添加到CSet中

候选老年代分区的CSet准入条件，可以通过活跃度阈值-XX:G1MixedGCLiveThresholdPercent(默认85%)进行设置，从而拦截那些回收开销巨大的对象；同时，每次混合收集可以包含候选老年代分区，可根据CSet对堆的总大小占比-XX:G1OldCSetRegionThresholdPercent(默认10%)设置数量上限

由上述可知，G1的收集都是根据CSet进行操作的，年轻代收集与混合收集没有明显的不同，最大的区别在于两种收集的触发条件

#### 2.4.1 年轻代收集集合 CSet of Young Collection

应用线程不断活动后，年轻代空间会被逐渐填满。当JVM分配对象到Eden区域失败(Eden区已满)时，便会触发一次STW式的年轻代收集在年轻代收集中，Eden分区存活的对象将被拷贝到Survivor分区；原有Survivor分区存活的对象，将根据任期阈值(tenuring threshold)分别晋升到PLAB中、新的survivor分区和老年代分区，而原有的年轻代分区将被整体回收掉

同时，年轻代收集还负责维护对象的年龄(存活次数)，辅助判断老化(tenuring)对象晋升的时候是到Survivor分区还是到老年代分区。年轻代收集首先先将晋升对象尺寸总和、对象年龄信息维护到年龄表中，再根据年龄表、Survivor尺寸、Survivor填充容量-XX:TargetSurvivorRatio(默认50%)、最大任期阈值-XX:MaxTenuringThreshold(默认15)，计算出一个恰当的任期阈值，凡是超过任期阈值的对象都会被晋升到老年代

#### 2.4.2 混合收集集合 CSet of Mixed Collection

年轻代收集不断活动后，老年代的空间也会被逐渐填充。当老年代占用空间超过整堆比IHOP阈值-XX:InitiatingHeapOccupancyPercent(默认45%)时，G1就会启动一次混合垃圾收集周期。为了满足暂停目标，G1可能不能一口气将所有的候选分区收集掉，因此G1可能会产生连续多次的混合收集与应用线程交替执行，每次STW的混合收集与年轻代收集过程相类似

为了确定包含到年轻代收集集合CSet的老年代分区，JVM通过参数混合周期的最大总次数-XX:G1MixedGCCountTarget(默认8)、堆废物百分比-XX:G1HeapWastePercent(默认5%)。通过候选老年代分区总数与混合周期最大总次数，确定每次包含到CSet的最小分区数量；根据堆废物百分比，当收集达到参数时，不再启动新的混合收集。而每次添加到CSet的分区，则通过计算得到的GC效率进行安排

#### 2.4.3 并发标记算法（三色标记法）

CMS和G1在并发标记时使用的是同一个算法：三色标记法，使用白灰黑三种颜色标记对象。白色是未标记；灰色自身被标记，引用的对象未标记；黑色自身与引用对象都已标记

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163141975-1871971726.png)

GC 开始前所有对象都是白色，GC 一开始所有根能够直达的对象被压到栈中，待搜索，此时颜色是灰色然后灰色对象依次从栈中取出搜索子对象，子对象也会被涂为灰色，入栈当其所有的子对象都涂为灰色之后该对象被涂为黑色当 GC 结束之后灰色对象将全部没了，剩下黑色的为存活对象，白色的为垃圾

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163154907-2134563224.gif)

#### 2.4.4 漏标问题

在remark过程中，黑色指向了白色，如果不对黑色重新扫描，则会漏标。会把白色D对象当作没有新引用指向从而回收掉

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163217294-817984886.png)

并发标记过程中，Mutator删除了所有从灰色到白色的引用，会产生漏标。此时白色对象应该被回收

产生漏标问题的条件有两个：

- 黑色对象指向了白色对象
- 灰色对象指向白色对象的引用消失

所以要解决漏标问题，打破两个条件之一即可：

- **跟踪黑指向白的增加** incremental update：增量更新，关注引用的增加，把黑色重新标记为灰色，下次重新扫描属性CMS采用该方法
- **记录灰指向白的消失** SATB snapshot at the beginning：关注引用的删除，当灰–>白消失时，要把这个 引用 推到GC的堆栈，保证白还能被GC扫描到G1采用该方法

**为什么G1采用SATB而不用incremental update**？

因为采用incremental update把黑色重新标记为灰色后，之前扫描过的还要再扫描一遍，效率太低。G1有RSet与SATB相配合。Card Table里记录了RSet，RSet里记录了其他对象指向自己的引用，这样就不需要再扫描其他区域，只要扫描RSet就可以了。

也就是说 灰色–>白色 引用消失时，如果没有 黑色–>白色，引用会被push到堆栈，下次扫描时拿到这个引用，由于有RSet的存在，不需要扫描整个堆去查找指向白色的引用，效率比较高SATB配合RSet浑然天成。

## 3. G1的活动周期

### 3.1 G1垃圾收集活动汇总

G1垃圾收集活动周期图

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163239785-134527329.jpg)

### 3.2 RSet的维护

由于不能整堆扫描，又需要计算分区确切的活跃度，因此，G1需要一个增量式的完全标记并发算法，通过维护RSet，得到准确的分区引用信息。在G1中，RSet的维护主要来源两个方面：写栅栏(Write Barrier)和并发优化线程(Concurrence Refinement Threads)

#### 3.2.1 栅栏Barrier

栅栏代码示意

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163308996-1950002265.jpg)

我们首先介绍一下栅栏(Barrier)的概念。栅栏是指在原生代码片段中，当某些语句被执行时，栅栏代码也会被执行。而G1主要在赋值语句中，使用写前栅栏(Pre-Write Barrrier)和写后栅栏(Post-Write Barrrier)。事实上，写栅栏的指令序列开销非常昂贵，应用吞吐量也会根据栅栏复杂度而降低

**写前栅栏 Pre-Write Barrrier**

即将执行一段赋值语句时，等式左侧对象将修改引用到另一个对象，那么等式左侧对象原先引用的对象所在分区将因此丧失一个引用，那么JVM就需要在赋值语句生效之前，记录丧失引用的对象。JVM并不会立即维护RSet，而是通过批量处理，在将来RSet更新(见SATB)

**写后栅栏 Post-Write Barrrier**

当执行一段赋值语句后，等式右侧对象获取了左侧对象的引用，那么等式右侧对象所在分区的RSet也应该得到更新。同样为了降低开销，写后栅栏发生后，RSet也不会立即更新，同样只是记录此次更新日志，在将来批量处理(见Concurrence Refinement Threads)

#### 3.2.2 起始快照算法Snapshot at the beginning (SATB)

Taiichi Tuasa贡献的增量式完全并发标记算法：起始快照算法(SATB)，主要针对标记-清除垃圾收集器的并发标记阶段（即CMS和G1的并发标记），非常适合G1的分区块的堆结构，同时解决了CMS的主要烦恼：重新标记暂停时间长带来的潜在风险

SATB会创建一个对象图，相当于堆的逻辑快照，从而确保并发标记阶段所有的垃圾对象都能通过快照被鉴别出来。当赋值语句发生时，应用将会改变了它的对象图，那么JVM需要记录被覆盖的对象。因此写前栅栏会在引用变更前，将值记录在SATB日志或缓冲区中。每个线程都会独占一个SATB缓冲区，初始有256条记录空间，当空间用尽时，线程会分配新的SATB缓冲区继续使用，而原有的缓冲区则加入全局列表中。最终在并发标记阶段，并发标记线程(Concurrent Marking Threads)在标记的同时，还会定期检查和处理全局缓冲区列表的记录，然后根据标记位图分片的标记位，扫描引用字段来更新RSet。此过程又称为并发标记 / SATB写前栅栏

#### 3.2.3 并发优化线程Concurrence Refinement Threads

G1中使用基于Urs Hölzle的快速写栅栏，将栅栏开销缩减到2个额外的指令。栅栏将会更新一个card table type的结构来跟踪代间引用

当赋值语句发生后，写后栅栏会先通过G1的过滤技术判断是否是跨分区的引用更新，并将跨分区更新对象的卡片加入缓冲区序列，即更新日志缓冲区或脏卡片队列与SATB类似，一旦日志缓冲区用尽，则分配一个新的日志缓冲区，并将原来的缓冲区加入全局列表中

并发优化线程(Concurrence Refinement Threads)，只专注扫描日志缓冲区记录的卡片来维护更新RSet，线程最大数目可通过`-XX:G1ConcRefinementThreads`(默认等于`-XX:ParellelGCThreads`)设置并发优化线程永远是活跃的，一旦发现全局列表有记录存在，就开始并发处理如果记录增长很快或者来不及处理，那么通过阈值`-X:G1ConcRefinementGreenZone/-XX:G1ConcRefinementYellowZone/-XX:G1ConcRefinementRedZone`，G1会用分层的方式调度，使更多的线程处理全局列表如果并发优化线程也不能跟上缓冲区数量，则Mutator线程(Java应用线程)会挂起应用并被加进来帮助处理，直到全部处理完因此，必须避免此类场景出现

### 3.3 并发标记周期 Concurrent Marking Cycle

并发标记周期是G1中非常重要的阶段，这个阶段将会为混合收集周期识别垃圾最多的老年代分区整个周期完成根标记、识别所有(可能)存活对象，并计算每个分区的活跃度，从而确定GC效率等级

当达到IHOP阈值`-XX:InitiatingHeapOccupancyPercent`(老年代占整堆比，默认45%)时，便会触发并发标记周期整个并发标记周期将由初始标记(Initial Mark)、根分区扫描(Root Region Scanning)、并发标记(Concurrent Marking)、重新标记(Remark)、清除(Cleanup)几个阶段组成其中，初始标记(随年轻代收集一起活动)、重新标记、清除是STW的，而并发标记如果来不及标记存活对象，则可能在并发标记过程中，G1又触发了几次年轻代收集

#### 3.3.1 并发标记线程 Concurrent Marking Threads

并发标记位图过程

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519163342097-1659495856.png)

要标记存活的对象，每个分区都需要创建位图(Bitmap)信息来存储标记数据，来确定标记周期内被分配的对象G1采用了两个位图Previous Bitmap、Next Bitmap，来存储标记数据，Previous位图存储上次的标记数据，Next位图在标记周期内不断变化更新，同时Previous位图的标记数据也越来越过时，当标记周期结束后Next位图便替换Previous位图，成为上次标记的位图同时，每个分区通过顶部开始标记(TAMS)，来记录已标记过的内存范围同样的，G1使用了两个顶部开始标记Previous TAMS(PTAMS)、Next TAMS(NTAMS)，记录已标记的范围

在并发标记阶段，G1会根据参数`-XX:ConcGCThreads`(默认GC线程数的1/4，即`-XX:ParallelGCThreads/4`)，分配并发标记线程(Concurrent Marking Threads)，进行标记活动每个并发线程一次只扫描一个分区，并通过"手指"指针的方式优化获取分区并发标记线程是爆发式的，在给定的时间段拼命干活，然后休息一段时间，再拼命干活

每个并发标记周期，在初始标记STW的最后，G1会分配一个空的Next位图和一个指向分区顶部(Top)的NTAMS标记Previous位图记录的上次标记数据，上次的标记位置，即PTAMS，在PTAMS与分区底部(Bottom)的范围内，所有的存活对象都已被标记那么，在PTAMS与Top之间的对象都将是隐式存活(Implicitly Live)对象在并发标记阶段，Next位图吸收了Previous位图的标记数据，同时每个分区都会有新的对象分配，则Top与NTAMS分离，前往更高的地址空间在并发标记的一次标记中，并发标记线程将找出NTAMS与PTAMS之间的所有存活对象，将标记数据存储在Next位图中同时，在NTAMS与Top之间的对象即成为已标记对象如此不断地更新Next位图信息，并在清除阶段与Previous位图交换角色

#### 3.3.2 初始标记 Initial Mark

初始标记(Initial Mark)负责标记所有能被直接可达的根对象(原生栈对象、全局对象、JNI对象)，根是对象图的起点，因此初始标记需要将Mutator线程(Java应用线程)暂停掉，也就是需要一个STW的时间段事实上，当达到IHOP阈值时，G1并不会立即发起并发标记周期，而是等待下一次年轻代收集，利用年轻代收集的STW时间段，完成初始标记，这种方式称为借道(Piggybacking)在初始标记暂停中，分区的NTAMS都被设置到分区顶部Top，初始标记是并发执行，直到所有的分区处理完

#### 3.3.3 根分区扫描 Root Region Scanning

在初始标记暂停结束后，年轻代收集也完成的对象复制到Survivor的工作，应用线程开始活跃起来此时为了保证标记算法的正确性，所有新复制到Survivor分区的对象，都需要被扫描并标记成根，这个过程称为根分区扫描(Root Region Scanning)，同时扫描的Suvivor分区也被称为根分区(Root Region)根分区扫描必须在下一次年轻代垃圾收集启动前完成(并发标记的过程中，可能会被若干次年轻代垃圾收集打断)，因为每次GC会产生新的存活对象集合

#### 3.3.4 并发标记 Concurrent Marking

和应用线程并发执行，并发标记线程在并发标记阶段启动，由参数`-XX:ConcGCThreads`(默认GC线程数的1/4，即`-XX:ParallelGCThreads/4`)控制启动数量，每个线程每次只扫描一个分区，从而标记出存活对象图在这一阶段会处理Previous/Next标记位图，扫描标记对象的引用字段同时，并发标记线程还会定期检查和处理STAB全局缓冲区列表的记录，更新对象引用信息参数`-XX:+ClassUnloadingWithConcurrentMark`会开启一个优化，如果一个类不可达(不是对象不可达)，则在重新标记阶段，这个类就会被直接卸载所有的标记任务必须在堆满前就完成扫描，如果并发标记耗时很长，那么有可能在并发标记过程中，又经历了几次年轻代收集如果堆满前没有完成标记任务，则会触发担保机制，经历一次长时间的串行Full GC

#### 3.3.5 存活数据计算 Live Data Accounting

存活数据计算(Live Data Accounting)是标记操作的附加产物，只要一个对象被标记，同时会被计算字节数，并计入分区空间只有NTAMS以下的对象会被标记和计算，在标记周期的最后，Next位图将被清空，等待下次标记周期

#### 3.3.6 重新标记 Remark

重新标记(Remark)是最后一个标记阶段在该阶段中，G1需要一个暂停的时间，去处理剩下的SATB日志缓冲区和所有更新，找出所有未被访问的存活对象，同时安全完成存活数据计算这个阶段也是并行执行的，通过参数-XX:ParallelGCThread可设置GC暂停时可用的GC线程数同时，引用处理也是重新标记阶段的一部分，所有重度使用引用对象(弱引用、软引用、虚引用、最终引用)的应用都会在引用处理上产生开销

#### 3.3.7 清除 Cleanup

紧挨着重新标记阶段的清除(Clean)阶段也是STW的Previous/Next标记位图、以及PTAMS/NTAMS，都会在清除阶段交换角色清除阶段主要执行以下操作：

- **RSet梳理**，启发式算法会根据活跃度和RSet尺寸对分区定义不同等级，同时RSet数理也有助于发现无用的引用参数`-XX:+PrintAdaptiveSizePolicy`可以开启打印启发式算法决策细节；
- **整理堆分区**，为混合收集周期识别回收收益高(基于释放空间和暂停目标)的老年代分区集合；
- **识别所有空闲分区**，即发现无存活对象的分区该分区可在清除阶段直接回收，无需等待下次收集周期

### 3.4 年轻代收集/混合收集周期

年轻代收集和混合收集周期，是G1回收空间的主要活动当应用运行开始时，堆内存可用空间还比较大，只会在年轻代满时，触发年轻代收集；随着老年代内存增长，当到达IHOP阈值`-XX:InitiatingHeapOccupancyPercent`(老年代占整堆比，默认45%)时，G1开始着手准备收集老年代空间首先经历并发标记周期，识别出高收益的老年代分区，前文已述但随后G1并不会马上开始一次混合收集，而是让应用线程先运行一段时间，等待触发一次年轻代收集在这次STW中，G1将保准整理混合收集周期接着再次让应用线程运行，当接下来的几次年轻代收集时，将会有老年代分区加入到CSet中，即触发混合收集，这些连续多次的混合收集称为混合收集周期(Mixed Collection Cycle)

#### 3.4.1 GC工作线程数

GC工作线程数 `-XX:ParallelGCThreads`

JVM可以通过参数`-XX:ParallelGCThreads`进行指定GC工作的线程数量参数`-XX:ParallelGCThreads`默认值并不是固定的，而是根据当前的CPU资源进行计算如果用户没有指定，且CPU小于等于8，则默认与CPU核数相等；若CPU大于8，则默认JVM会经过计算得到一个小于CPU核数的线程数；当然也可以人工指定与CPU核数相等

#### 3.4.2 年轻代收集 Young Collection

每次收集过程中，既有并行执行的活动，也有串行执行的活动，但都可以是多线程的在并行执行的任务中，如果某个任务过重，会导致其他线程在等待某项任务的处理，需要对这些地方进行优化

**并行活动**

- `外部根分区扫描 Ext Root Scanning`：此活动对堆外的根(JVM系统目录、VM数据结构、JNI线程句柄、硬件寄存器、全局变量、线程对栈根)进行扫描，发现那些没有加入到暂停收集集合CSet中的对象如果系统目录(单根)拥有大量加载的类，最终可能其他并行活动结束后，该活动依然没有结束而带来的等待时间
- `更新已记忆集合 Update RS`：并发优化线程会对脏卡片的分区进行扫描更新日志缓冲区来更新RSet，但只会处理全局缓冲列表作为补充，所有被记录但是还没有被优化线程处理的剩余缓冲区，会在该阶段处理，变成已处理缓冲区(Processed Buffers)为了限制花在更新RSet的时间，可以设置暂停占用百分比-XX:G1RSetUpdatingPauseTimePercent(默认10%，即-XX:MaxGCPauseMills/10)值得注意的是，如果更新日志缓冲区更新的任务不降低，单纯地减少RSet的更新时间，会导致暂停中被处理的缓冲区减少，将日志缓冲区更新工作推到并发优化线程上，从而增加对Java应用线程资源的争夺
- `RSet扫描 Scan RS`：在收集当前CSet之前，考虑到分区外的引用，必须扫描CSet分区的RSet如果RSet发生粗化，则会增加RSet的扫描时间开启诊断模式-XX:UnlockDiagnosticVMOptions后，通过参数-XX:+G1SummarizeRSetStats可以确定并发优化线程是否能够及时处理更新日志缓冲区，并提供更多的信息，来帮助为RSet粗化总数提供窗口参数-XX：G1SummarizeRSetStatsPeriod=n可设置RSet的统计周期，即经历多少此GC后进行一次统计
- `代码根扫描 Code Root Scanning`：对代码根集合进行扫描，扫描JVM编译后代码Native Method的引用信息(nmethod扫描)，进行RSet扫描事实上，只有CSet分区中的RSet有强代码根时，才会做nmethod扫描，查找对CSet的引用
- `转移和回收 Object Copy`：通过选定的CSet以及CSet分区完整的引用集，将执行暂停时间的主要部分：CSet分区存活对象的转移、CSet分区空间的回收通过工作窃取机制来负载均衡地选定复制对象的线程，并且复制和扫描对象被转移的存活对象将拷贝到每个GC线程分配缓冲区GCLABG1会通过计算，预测分区复制所花费的时间，从而调整年轻代的尺寸
- `终止 Termination`：完成上述任务后，如果任务队列已空，则工作线程会发起终止要求如果还有其他线程继续工作，空闲的线程会通过工作窃取机制尝试帮助其他线程处理而单独执行根分区扫描的线程，如果任务过重，最终会晚于终止
- `GC外部的并行活动 GC Worker Other`：该部分并非GC的活动，而是JVM的活动导致占用了GC暂停时间(例如JNI编译)

**串行活动**

- `代码根更新 Code Root Fixup`：根据转移对象更新代码根
- `代码根清理 Code Root Purge`：清理代码根集合表
- `清除全局卡片标记 Clear CT`：在任意收集周期会扫描CSet与RSet记录的PRT，扫描时会在全局卡片表中进行标记，防止重复扫描在收集周期的最后将会清除全局卡片表中的已扫描标志
- `选择下次收集集合 Choose CSet`：该部分主要用于并发标记周期后的年轻代收集、以及混合收集中，在这些收集过程中，由于有老年代候选分区的加入，往往需要对下次收集的范围做出界定；但单纯的年轻代收集中，所有收集的分区都会被收集，不存在选择
- `引用处理 Ref Proc`：主要针对软引用、弱引用、虚引用、final引用、JNI引用当Ref Proc占用时间过多时，可选择使用参数`-XX:ParallelRefProcEnabled`激活多线程引用处理G1希望应用能小心使用软引用，因为软引用会一直占据内存空间直到空间耗尽时被Full GC回收掉；即使未发生Full GC，软引用对内存的占用，也会导致GC次数的增加
- `引用排队 Ref Enq`：此项活动可能会导致RSet的更新，此时会通过记录日志，将关联的卡片标记为脏卡片
- `卡片重新脏化 Redirty Cards`：重新脏化卡片
- `回收空闲巨型分区 Humongous Reclaim`：G1做了一个优化：通过查看所有根对象以及年轻代分区的RSet，如果确定RSet中巨型对象没有任何引用，则说明G1发现了一个不可达的巨型对象，该对象分区会被回收
- `释放分区 Free CSet`：回收CSet分区的所有空间，并加入到空闲分区中
- `其他活动 Other`：GC中可能还会经历其他耗时很小的活动，如修复JNI句柄等

### 3.5 并发标记周期后的年轻代收集 Young Collection Following Concurrent Marking Cycle

当G1发起并发标记周期之后，并不会马上开始混合收集G1会先等待下一次年轻代收集，然后在该收集阶段中，确定下次混合收集的CSet(Choose CSet)

#### 3.5.1 混合收集周期 Mixed Collection Cycle

单次的混合收集与年轻代收集并无二致根据暂停目标，老年代的分区可能不能一次暂停收集中被处理完，G1会发起连续多次的混合收集，称为混合收集周期(Mixed Collection Cycle)G1会计算每次加入到CSet中的分区数量、混合收集进行次数，并且在上次的年轻代收集、以及接下来的混合收集中，G1会确定下次加入CSet的分区集(Choose CSet)，并且确定是否结束混合收集周期

#### 3.5.2 转移失败的担保机制 Full GC

转移失败(Evacuation Failure)是指当G1无法在堆空间中申请新的分区时，G1便会触发担保机制，执行一次STW式的、单线程的Full GCFull GC会对整堆做标记清除和压缩，最后将只包含纯粹的存活对象参数-XX:G1ReservePercent(默认10%)可以保留空间，来应对晋升模式下的异常情况，最大占用整堆50%，更大也无意义

G1在以下场景中会触发Full GC，同时会在日志中记录to-space-exhausted以及Evacuation Failure：

- 从年轻代分区拷贝存活对象时，无法找到可用的空闲分区
- 从老年代分区转移存活对象时，无法找到可用的空闲分区
- 分配巨型对象时在老年代无法找到足够的连续分区

由于G1的应用场合往往堆内存都比较大，所以Full GC的收集代价非常昂贵，应该避免Full GC的发生

## 4. 总结

G1是一款非常优秀的垃圾收集器，不仅适合堆内存大的应用，同时也简化了调优的工作通过主要的参数初始和最大堆空间、以及最大容忍的GC暂停目标，就能得到不错的性能；同时，我们也看到G1对内存空间的浪费较高，但通过**首先收集尽可能多的垃圾**(Garbage First)的设计原则，可以及时发现过期对象，从而让内存占用处于合理的水平

虽然G1也有类似CMS的收集动作：初始标记、并发标记、重新标记、清除、转移回收，并且也以一个串行收集器做担保机制，但单纯地以类似前三种的过程描述显得并不是很妥当

- G1的设计原则是"**首先收集尽可能多的垃圾**(Garbage First)"因此，G1并不会等内存耗尽(串行、并行)或者快耗尽(CMS)的时候开始垃圾收集，而是在内部采用了启发式算法，在老年代找出具有高收集收益的分区进行收集同时G1可以根据用户设置的暂停时间目标自动调整年轻代和总堆大小，暂停目标越短年轻代空间越小、总空间就越大；
- G1采用内存分区(Region)的思路，将内存划分为一个个相等大小的内存分区，回收时则以分区为单位进行回收，存活的对象复制到另一个空闲分区中由于都是以相等大小的分区为单位进行操作，因此G1天然就是一种压缩方案(局部压缩)；
- G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代与老年代的区别，也不需要完全独立的survivor(to space)堆做复制准备G1只有逻辑上的分代概念，或者说每个分区都可能随G1的运行在不同代之间前后切换；
- G1的收集都是STW的，但年轻代和老年代的收集界限比较模糊，采用了混合(mixed)收集的方式即每次收集既可能只收集年轻代分区(年轻代收集)，也可能在收集年轻代的同时，包含部分老年代分区(混合收集)，这样即使堆内存很大时，也可以限制收集范围，从而降低停顿

## 参考资料

- Charlie H, Monica B, Poonam P, Bengt R. Java Performance Companion
- 周志明. 深入理解JVM虚拟机







# GC - Java 垃圾回收器之ZGC详解

> ZGC（The Z Garbage Collector）是JDK 11中推出的一款低延迟垃圾回收器, 是JDK 11+ 最为重要的更新之一，适用于**大内存低延迟**服务的内存管理和回收在梳理相关知识点时，发现美团技术团队分享的文章[新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)比较完善（包含G1收集器停顿时间瓶颈，原理，优化等）, 这里分享给你，帮你构建ZGC相关的知识体系

## ZGC概述

> ZGC（The Z Garbage Collector）是JDK 11中推出的一款低延迟垃圾回收器，它的设计目标包括：

- 停顿时间不超过10ms；
- 停顿时间不会随着堆的大小，或者活跃对象的大小而增加（对程序吞吐量影响小于15%）；
- 支持8MB~4TB级别的堆（未来支持16TB）

从设计目标来看，我们知道ZGC适用于**大内存低延迟**服务的内存管理和回收本文主要介绍ZGC在低延时场景中的应用和卓越表现，文章内容主要分为四部分：

1. **GC之痛**：介绍实际业务中遇到的GC痛点，并分析CMS收集器和G1收集器停顿时间瓶颈；
2. **ZGC原理**：分析ZGC停顿时间比G1或CMS更短的本质原因，以及背后的技术原理；
3. **ZGC调优实践**：重点分享对ZGC调优的理解，并分析若干个实际调优案例；
4. **升级ZGC效果**：展示在生产环境应用ZGC取得的效果

## GC之痛

> 很多低延迟高可用Java服务的系统可用性经常受GC停顿的困扰GC停顿指垃圾回收期间STW（Stop The World），当STW时，所有应用线程停止活动，等待GC停顿结束

以美团风控服务为例，部分上游业务要求风控服务65ms内返回结果，并且可用性要达到99.99%但因为GC停顿，我们未能达到上述可用性目标当时使用的是CMS垃圾回收器，单次Young GC 40ms，一分钟10次，接口平均响应时间30ms通过计算可知，有（40ms + 30ms) * 10次 / 60000ms = 1.12%的请求的响应时间会增加0 ~ 40ms不等，其中30ms * 10次 / 60000ms = 0.5%的请求响应时间会增加40ms可见，GC停顿对响应时间的影响较大为了降低GC停顿对系统可用性的影响，我们从降低单次GC时间和降低GC频率两个角度出发进行了调优，还测试过G1垃圾回收器，但这三项措施均未能降低GC对服务可用性的影响

### CMS与G1停顿时间瓶颈

> 在介绍ZGC之前，首先回顾一下CMS和G1的GC过程以及停顿时间的瓶颈CMS新生代的Young GC、G1和ZGC都基于标记-复制算法，但算法具体实现的不同就导致了巨大的性能差异

标记-复制算法应用在CMS新生代（ParNew是CMS默认的新生代垃圾回收器）和G1垃圾回收器中标记-复制算法可以分为三个阶段：

- **标记阶段**，即从GC Roots集合开始，标记活跃对象；
- **转移阶段**，即把活跃对象复制到新的内存地址上；
- **重定位阶段**，因为转移导致对象的地址发生了变化，在重定位阶段，所有指向对象旧地址的指针都要调整到对象新的地址上

下面以G1为例，通过G1中标记-复制算法过程（G1的Young GC和Mixed GC均采用该算法），分析G1停顿耗时的主要瓶颈G1垃圾回收周期如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193159749-1807312500.png)

G1的混合回收过程可以分为标记阶段、清理阶段和复制阶段

#### 标记阶段停顿分析

- **初始标记阶段**：初始标记阶段是指从GC Roots出发标记全部直接子节点的过程，该阶段是STW的由于GC Roots数量不多，通常该阶段耗时非常短
- **并发标记阶段**：并发标记阶段是指从GC Roots开始对堆中对象进行可达性分析，找出存活对象该阶段是并发的，即应用线程和GC线程可以同时活动并发标记耗时相对长很多，但因为不是STW，所以我们不太关心该阶段耗时的长短
- **再标记阶段**：重新标记那些在并发标记阶段发生变化的对象该阶段是STW的

#### 清理阶段停顿分析

- **清理阶段**清点出有存活对象的分区和没有存活对象的分区，该阶段不会清理垃圾对象，也不会执行存活对象的复制该阶段是STW的

#### 复制阶段停顿分析

- **复制算法**中的转移阶段需要分配新内存和复制对象的成员变量转移阶段是STW的，其中内存分配通常耗时非常短，但对象成员变量的复制耗时有可能较长，这是因为复制耗时与存活对象数量与对象复杂度成正比对象越复杂，复制耗时越长

四个STW过程中，初始标记因为只标记GC Roots，耗时较短再标记因为对象数少，耗时也较短清理阶段因为内存分区数量少，耗时也较短转移阶段要处理所有存活的对象，耗时会较长因此，**G1停顿时间的瓶颈主要是标记-复制中的转移阶段STW**为什么转移阶段不能和标记阶段一样并发执行呢？主要是G1未能解决转移过程中准确定位对象地址的问题

G1的Young GC和CMS的Young GC，其标记-复制全过程STW，这里不再详细阐述

## ZGC原理

### 全并发的ZGC

> 与CMS中的ParNew和G1类似，**ZGC也采用标记-复制算法**，不过ZGC对该算法做了重大改进：**ZGC在标记、转移和重定位阶段几乎都是并发**的，这是ZGC实现停顿时间小于10ms目标的最关键原因

ZGC垃圾回收周期如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193241007-1758742276.png)

ZGC只有三个STW阶段：初始标记，再标记，初始转移其中，初始标记和初始转移分别都只需要扫描所有GC Roots，其处理时间和GC Roots的数量成正比，一般情况耗时非常短；再标记阶段STW时间很短，最多1ms，超过1ms则再次进入并发标记阶段即，ZGC几乎所有暂停都只依赖于GC Roots集合大小，停顿时间不会随着堆的大小或者活跃对象的大小而增加与ZGC对比，G1的转移阶段完全STW的，且停顿时间随存活对象的大小增加而增加

### ZGC关键技术

ZGC通过着色指针和读屏障技术，解决了转移过程中准确访问对象的问题，实现了并发转移大致原理描述如下：并发转移中“并发”意味着GC线程在转移对象的过程中，应用线程也在不停地访问对象假设对象发生转移，但对象地址未及时更新，那么应用线程可能访问到旧地址，从而造成错误而在ZGC中，应用线程访问对象将触发“读屏障”，如果发现对象被移动了，那么“读屏障”会把读出来的指针更新到对象的新地址上，这样应用线程始终访问的都是对象的新地址那么，JVM是如何判断对象被移动过呢？就是利用对象引用的地址，即着色指针下面介绍着色指针和读屏障技术细节

#### 着色指针

> 着色指针是一种将信息存储在指针中的技术

ZGC仅支持64位系统，它把64位虚拟地址空间划分为多个子空间，如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193302710-961555534.png)

其中，[0~4TB) 对应Java堆，[4TB ~ 8TB) 称为M0地址空间，[8TB ~ 12TB) 称为M1地址空间，[12TB ~ 16TB) 预留未使用，[16TB ~ 20TB) 称为Remapped空间

当应用程序创建对象时，首先在堆空间申请一个虚拟地址，但该虚拟地址并不会映射到真正的物理地址ZGC同时会为该对象在M0、M1和Remapped地址空间分别申请一个虚拟地址，且这三个虚拟地址对应同一个物理地址，但这三个空间在同一时间有且只有一个空间有效ZGC之所以设置三个虚拟地址空间，是因为它使用“空间换时间”思想，去降低GC停顿时间“空间换时间”中的空间是虚拟空间，而不是真正的物理空间后续章节将详细介绍这三个空间的切换过程

与上述地址空间划分相对应，ZGC实际仅使用64位地址空间的第041位，而第4245位存储元数据，第47~63位固定为0

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193319151-158480141.png)

ZGC将对象存活信息存储在42~45位中，这与传统的垃圾回收并将对象存活信息放在对象头中完全不同

#### 读屏障

> 读屏障是JVM向应用代码插入一小段代码的技术当应用线程从堆中读取对象引用时，就会执行这段代码需要注意的是，仅“从堆中读取对象引用”才会触发这段代码

读屏障示例：

```java
Object o = obj.FieldA   // 从堆中读取引用，需要加入屏障
<Load barrier>
Object p = o  // 无需加入屏障，因为不是从堆中读取引用
o.dosomething() // 无需加入屏障，因为不是从堆中读取引用
int i =  obj.FieldB  //无需加入屏障，因为不是对象引用
```

ZGC中读屏障的代码作用：在对象标记和转移过程中，用于确定对象的引用地址是否满足条件，并作出相应动作

### ZGC并发处理演示

接下来详细介绍ZGC一次垃圾回收周期中地址视图的切换过程：

- **初始化**：ZGC初始化之后，整个内存空间的地址视图被设置为Remapped程序正常运行，在内存中分配对象，满足一定条件后垃圾回收启动，此时进入标记阶段
- **并发标记阶段**：第一次进入标记阶段时视图为M0，如果对象被GC标记线程或者应用线程访问过，那么就将对象的地址视图从Remapped调整为M0所以，在标记阶段结束之后，对象的地址要么是M0视图，要么是Remapped如果对象的地址是M0视图，那么说明对象是活跃的；如果对象的地址是Remapped视图，说明对象是不活跃的
- **并发转移阶段**：标记结束后就进入转移阶段，此时地址视图再次被设置为Remapped如果对象被GC转移线程或者应用线程访问过，那么就将对象的地址视图从M0调整为Remapped

其实，在标记阶段存在两个地址视图M0和M1，上面的过程显示只用了一个地址视图**之所以设计成两个，是为了区别前一次标记和当前标记**也即，第二次进入并发标记阶段后，地址视图调整为M1，而非M0

着色指针和读屏障技术不仅应用在并发转移阶段，还应用在并发标记阶段：将对象设置为已标记，传统的垃圾回收器需要进行一次内存访问，并将对象存活信息放在对象头中；而在ZGC中，只需要设置指针地址的第42~45位即可，并且因为是寄存器访问，所以速度比访问内存更快

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193340898-1843921570.png)

## ZGC调优实践

> ZGC不是“银弹”，需要根据服务的具体特点进行调优网络上能搜索到实战经验较少，调优理论需自行摸索，我们在此阶段也耗费了不少时间，最终才达到理想的性能本文的一个目的是列举一些使用ZGC时常见的问题，帮助大家使用ZGC提高服务可用性

### 调优基础知识

#### 理解ZGC重要配置参数

> 以我们服务在生产环境中ZGC参数配置为例，说明各个参数的作用：

重要参数配置样例：

```java
-Xms10G -Xmx10G 
-XX:ReservedCodeCacheSize=256m -XX:InitialCodeCacheSize=256m 
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC 
-XX:ConcGCThreads=2 -XX:ParallelGCThreads=6 
-XX:ZCollectionInterval=120 -XX:ZAllocationSpikeTolerance=5 
-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive 
-Xlog:safepoint,Classhisto*=trace,age*,gc*=info:file=/opt/logs/logs/gc-%t.log:time,tid,tags:filecount=5,filesize=50m 
```

- `-Xms -Xmx`：堆的最大内存和最小内存，这里都设置为10G，程序的堆内存将保持10G不变
- `-XX:ReservedCodeCacheSize -XX:InitialCodeCacheSize`：设置CodeCache的大小， JIT编译的代码都放在CodeCache中，一般服务64m或128m就已经足够我们的服务因为有一定特殊性，所以设置的较大，后面会详细介绍
- `-XX:+UnlockExperimentalVMOptions -XX:+UseZGC`：启用ZGC的配置
- `-XX:ConcGCThreads`：并发回收垃圾的线程默认是总核数的12.5%，8核CPU默认是1调大后GC变快，但会占用程序运行时的CPU资源，吞吐会受到影响
- `-XX:ParallelGCThreads`：STW阶段使用线程数，默认是总核数的60%
- `-XX:ZCollectionInterval`：ZGC发生的最小时间间隔，单位秒
- `-XX:ZAllocationSpikeTolerance`：ZGC触发自适应算法的修正系数，默认2，数值越大，越早的触发ZGC
- `-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive`：是否启用主动回收，默认开启，这里的配置表示关闭
- `-Xlog`：设置GC日志中的内容、格式、位置以及每个日志的大小

#### 理解ZGC触发时机

> 相比于CMS和G1的GC触发机制，ZGC的GC触发机制有很大不同ZGC的核心特点是并发，GC过程中一直有新的对象产生如何保证在GC完成之前，新产生的对象不会将堆占满，是ZGC参数调优的第一大目标因为在ZGC中，当垃圾来不及回收将堆占满时，会导致正在运行的线程停顿，持续时间可能长达秒级之久

ZGC有多种GC触发机制，总结如下：

- **阻塞内存分配请求触发**：当垃圾来不及回收，垃圾将堆占满时，会导致部分线程阻塞我们应当避免出现这种触发方式日志中关键字是“Allocation Stall”
- **基于分配速率的自适应算法**：最主要的GC触发方式，其算法原理可简单描述为”ZGC根据近期的对象分配速率以及GC时间，计算出当内存占用达到什么阈值时触发下一次GC”自适应算法的详细理论可参考彭成寒《新一代垃圾回收器ZGC设计与实现》一书中的内容通过ZAllocationSpikeTolerance参数控制阈值大小，该参数默认2，数值越大，越早的触发GC我们通过调整此参数解决了一些问题日志中关键字是“Allocation Rate”
- **基于固定时间间隔**：通过ZCollectionInterval控制，适合应对突增流量场景流量平稳变化时，自适应算法可能在堆使用率达到95%以上才触发GC流量突增时，自适应算法触发的时机可能会过晚，导致部分线程阻塞我们通过调整此参数解决流量突增场景的问题，比如定时活动、秒杀等场景日志中关键字是“Timer”
- **主动触发规则**：类似于固定间隔规则，但时间间隔不固定，是ZGC自行算出来的时机，我们的服务因为已经加了基于固定时间间隔的触发机制，所以通过-ZProactive参数将该功能关闭，以免GC频繁，影响服务可用性 日志中关键字是“Proactive”
- **预热规则**：服务刚启动时出现，一般不需要关注日志中关键字是“Warmup”
- **外部触发**：代码中显式调用System.gc()触发 日志中关键字是“System.gc()”
- **元数据分配触发**：元数据区不足时导致，一般不需要关注 日志中关键字是“Metadata GC Threshold”

#### 理解ZGC日志

一次完整的GC过程，需要注意的点已在图中标出

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193419389-759244267.png)

注意：该日志过滤了进入安全点的信息正常情况，在一次GC过程中还穿插着进入安全点的操作

GC日志中每一行都注明了GC过程中的信息，关键信息如下：

- **Start**：开始GC，并标明的GC触发的原因上图中触发原因是自适应算法
- **Phase-Pause Mark Start**：初始标记，会STW
- **Phase-Pause Mark End**：再次标记，会STW
- **Phase-Pause Relocate Start**：初始转移，会STW
- **Heap信息**：记录了GC过程中Mark、Relocate前后的堆大小变化状况High和Low记录了其中的最大值和最小值，我们一般关注High中Used的值，如果达到100%，在GC过程中一定存在内存分配不足的情况，需要调整GC的触发时机，更早或者更快地进行GC
- **GC信息统计**：可以定时的打印垃圾收集信息，观察10秒内、10分钟内、10个小时内，从启动到现在的所有统计信息利用这些统计信息，可以排查定位一些异常点

日志中内容较多，关键点已用红线标出，含义较好理解，更详细的解释大家可以自行在网上查阅资料

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193431941-1029405740.png)

#### 理解ZGC停顿原因

我们在实战过程中共发现了6种使程序停顿的场景，分别如下：

- **GC时，初始标记**：日志中Pause Mark Start
- **GC时，再标记**：日志中Pause Mark End
- **GC时，初始转移**：日志中Pause Relocate Start
- **内存分配阻塞**：当内存不足时线程会阻塞等待GC完成，关键字是”Allocation Stall”

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193448417-314887730.png)

- **安全点**：所有线程进入到安全点后才能进行GC，ZGC定期进入安全点判断是否需要GC先进入安全点的线程需要等待后进入安全点的线程直到所有线程挂起
- **dump线程、内存**：比如jstack、jmap命令

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193458908-1286881725.png)

![img](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230519193506953-1288434960.png)

### 调优案例

我们维护的服务名叫Zeus，它是美团的规则平台，常用于风控场景中的规则管理规则运行是基于开源的表达式执行引擎AviatorAviator内部将每一条表达式转化成Java的一个类，通过调用该类的接口实现表达式逻辑

Zeus服务内的规则数量超过万条，且每台机器每天的请求量几百万这些客观条件导致Aviator生成的类和方法会产生很多的ClassLoader和CodeCache，这些在使用ZGC时都成为过GC的性能瓶颈接下来介绍两类调优案例

> **第一类：内存分配阻塞，系统停顿可达到秒级**

#### 案例一：秒杀活动中流量突增，出现性能毛刺

日志信息：对比出现性能毛刺时间点的GC日志和业务日志，发现JVM停顿了较长时间，且停顿时GC日志中有大量的“Allocation Stall”日志

分析：这种案例多出现在“自适应算法”为主要GC触发机制的场景中ZGC是一款并发的垃圾回收器，GC线程和应用线程同时活动，在GC过程中，还会产生新的对象GC完成之前，新产生的对象将堆占满，那么应用线程可能因为申请内存失败而导致线程阻塞当秒杀活动开始，大量请求打入系统，但自适应算法计算的GC触发间隔较长，导致GC触发不及时，引起了内存分配阻塞，导致停顿

解决方法：

（1）开启”基于固定时间间隔“的GC触发机制：-XX:ZCollectionInterval比如调整为5秒，甚至更短 （2）增大修正系数-XX:ZAllocationSpikeTolerance，更早触发GCZGC采用正态分布模型预测内存分配速率，模型修正系数ZAllocationSpikeTolerance默认值为2，值越大，越早的触发GC，Zeus中所有集群设置的是5

#### 案例二：压测时，流量逐渐增大到一定程度后，出现性能毛刺

日志信息：平均1秒GC一次，两次GC之间几乎没有间隔

分析：GC触发及时，但内存标记和回收速度过慢，引起内存分配阻塞，导致停顿

解决方法：增大-XX:ConcGCThreads， 加快并发标记和回收速度ConcGCThreads默认值是核数的1/8，8核机器，默认值是1该参数影响系统吞吐，如果GC间隔时间大于GC周期，不建议调整该参数

> **第二类：GC Roots 数量大，单次GC停顿时间长**

#### 案例三： 单次GC停顿时间30ms，与预期停顿10ms左右有较大差距

日志信息：观察ZGC日志信息统计，“Pause Roots ClassLoaderDataGraph”一项耗时较长

分析：dump内存文件，发现系统中有上万个ClassLoader实例我们知道ClassLoader属于GC Roots一部分，且ZGC停顿时间与GC Roots成正比，GC Roots数量越大，停顿时间越久再进一步分析，ClassLoader的类名表明，这些ClassLoader均由Aviator组件生成分析Aviator源码，发现Aviator对每一个表达式新生成类时，会创建一个ClassLoader，这导致了ClassLoader数量巨大的问题在更高Aviator版本中，该问题已经被修复，即仅创建一个ClassLoader为所有表达式生成类

解决方法：升级Aviator组件版本，避免生成多余的ClassLoader

#### 案例四：服务启动后，运行时间越长，单次GC时间越长，重启后恢复

日志信息：观察ZGC日志信息统计，“Pause Roots CodeCache”的耗时会随着服务运行时间逐渐增长

分析：CodeCache空间用于存放Java热点代码的JIT编译结果，而CodeCache也属于GC Roots一部分通过添加-XX:+PrintCodeCacheOnCompilation参数，打印CodeCache中的被优化的方法，发现大量的Aviator表达式代码定位到根本原因，每个表达式都是一个类中一个方法随着运行时间越长，执行次数增加，这些方法会被JIT优化编译进入到Code Cache中，导致CodeCache越来越大

解决方法：JIT有一些参数配置可以调整JIT编译的条件，但对于我们的问题都不太适用我们最终通过业务优化解决，删除不需要执行的Aviator表达式，从而避免了大量Aviator方法进入CodeCache中

值得一提的是，我们并不是在所有这些问题都解决后才全量部署所有集群即使开始有各种各样的毛刺，但计算后发现，有各种问题的ZGC也比之前的CMS对服务可用性影响小所以从开始准备使用ZGC到全量部署，大概用了2周的时间在之后的3个月时间里，我们边做业务需求，边跟进这些问题，最终逐个解决了上述问题，从而使ZGC在各个集群上达到了一个更好表现

## 升级ZGC效果

### 延迟降低

TP(Top Percentile)是一项衡量系统延迟的指标：TP999表示99.9%请求都能被响应的最小耗时；TP99表示99%请求都能被响应的最小耗时

在Zeus服务不同集群中，ZGC在低延迟（TP999 < 200ms）场景中收益较大：

- TP999：下降12142ms，下降幅度18%74%
- TP99：下降528ms，下降幅度10%47%

超低延迟（TP999 < 20ms）和高延迟（TP999 > 200ms）服务收益不大，原因是这些服务的响应时间瓶颈不是GC，而是外部依赖的性能

### 吞吐下降

对吞吐量优先的场景，ZGC可能并不适合例如，Zeus某离线集群原先使用CMS，升级ZGC后，系统吞吐量明显降低究其原因有二：

- 第一，ZGC是单代垃圾回收器，而CMS是分代垃圾回收器单代垃圾回收器每次处理的对象更多，更耗费CPU资源；
- 第二，ZGC使用读屏障，读屏障操作需耗费额外的计算资源

## 总结

ZGC作为下一代垃圾回收器，性能非常优秀ZGC垃圾回收过程几乎全部是并发，实际STW停顿时间极短，不到10ms这得益于其采用的着色指针和读屏障技术

Zeus在升级JDK 11+ZGC中，通过将风险和问题分类，然后各个击破，最终顺利实现了升级目标，GC停顿也几乎不再影响系统可用性

最后推荐大家升级ZGC，Zeus系统因为业务特点，遇到了较多问题，而风控其他团队在升级时都非常顺利

## 参考文献

- [ZGC官网](https://wiki.openJDK.Java.net/display/zgc/Main)
- 彭成寒.《新一代垃圾回收器ZGC设计与实现》. 机械工业出版社, 2019.
- [从实际案例聊聊Java应用的GC优化](https://tech.meituan.com/2017/12/29/JVM-optimize.html)
- [Java Hotspot G1 GC的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)

作者简介

- 王东，美团信息安全资深工程师
- 王伟，美团信息安全技术专家

更多优秀文章推荐

- https://www.jianshu.com/p/664e4da05b2c











# 调试排错 - JVM 调优参数

> 本文对JVM涉及的常见的调优参数和垃圾回收参数进行阐述

- 调试排错 - JVM 调优参数
  - JVM参数
  - 垃圾回收





## JVM参数

| 参数                        | 含义                                                         |
| --------------------------- | ------------------------------------------------------------ |
| -Xms                        | 堆最小值                                                     |
| -Xmx                        | 堆最大堆值-Xms与-Xmx 的单位默认字节都是以k、m做单位的<br />通常这两个配置参数相等，避免每次空间不足，动态扩容带来的影响 |
| -Xmn                        | 新生代大小                                                   |
| -Xss                        | 每个线程池的堆栈大小在JDK5以上的版本，每个线程堆栈大小为1m，JDK5以前的版本是每个线程池大小为256k一般在相同物理内存下，如果减少－xss值会产生更大的线程数，但不同的操作系统对进程内线程数是有限制的，是不能无限生成 |
| -XX:NewRatio                | 设置新生代与老年代比值，-XX:NewRatio=4 表示新生代与老年代所占比例为1:4 ，新生代占比整个堆的五分之一如果设置了-Xmn的情况下，该参数是不需要在设置的 |
| -XX:PermSize                | 设置持久代初始值，默认是物理内存的六十四分之一               |
| -XX:MaxPermSize             | 设置持久代最大值，默认是物理内存的四分之一                   |
| -XX:MaxTenuringThreshold    | 新生代中对象存活次数，默认15(若对象在eden区，经历一次MinorGC后还活着，则被移动到Survior区，年龄加1以后，对象每次经历MinorGC，年龄都加1达到阀值，则移入老年代) |
| -XX:SurvivorRatio           | Eden区与Subrvivor区大小的比值，如果设置为8，两个Subrvivor区与一个Eden区的比值为2:8，一个Survivor区占整个新生代的十分之一 |
| -XX:+UseFastAccessorMethods | 原始类型快速优化                                             |
| -XX:+AggressiveOpts         | 编译速度加快                                                 |
| -XX:PretenureSizeThreshold  | 对象超过多大值时直接在老年代中分配                           |



```text
说明: 
整个堆大小的计算公式: JVM 堆大小 ＝ 年轻代大小＋年老代大小＋持久代大小
增大新生代大小就会减少对应的年老代大小，设置-Xmn值对系统性能影响较大，所以如果设置新生代大小的调整，则需要严格的测试调整而新生代是用来存放新创建的对象，大小是随着堆大小增大和减少而有相应的变化，默认值是保持堆大小的十五分之一，-Xmn参数就是设置新生代的大小，也可以通过-XX:NewRatio来设置新生代与年老代的比例，Java 官方推荐配置为3:8

新生代的特点就是内存中的对象更新速度快，在短时间内容易产生大量的无用对象，如果在这个参数时就需要考虑垃圾回收器设置参数也需要调整推荐使用: 复制清除算法和并行收集器进行垃圾回收，而新生代的垃圾回收叫做初级回收
StackOverflowError和OutOfMemoryException当线程中的请求的栈的深度大于最大可用深度，就会抛出前者；若内存空间不够，无法创建新的线程，则会抛出后者栈的大小直接决定了函数的调用最大深度，栈越大，函数嵌套可调用次数就越多
```

**经验** :

1. Xmn用于设置新生代的大小过小会增加Minor GC频率，过大会减小老年代的大小一般设为整个堆空间的1/4或1/3.
2. XX:SurvivorRatio用于设置新生代中survivor空间(from/to)和eden空间的大小比例； XX:TargetSurvivorRatio表示，当经历Minor GC后，survivor空间占有量(百分比)超过它的时候，就会压缩进入老年代(当然，如果survivor空间不够，则直接进入老年代)默认值为50%
3. 为了性能考虑，一开始尽量将新生代对象留在新生代，避免新生的大对象直接进入老年代因为新生对象大部分都是短期的，这就造成了老年代的内存浪费，并且回收代价也高(Full GC发生在老年代和方法区Perm).
4. 当Xms=Xmx，可以使得堆相对稳定，避免不停震荡
5. 一般来说，MaxPermSize设为64MB可以满足绝大多数的应用了若依然出现方法区溢出，则可以设为128MB若128MB还不能满足需求，那么就应该考虑程序优化了，减少**动态类**的产生







## 垃圾回收

**垃圾回收算法** :

- 引用计数法: 会有循环引用的问题，古老的方法；
- Mark-Sweep: 标记清除根可达判断，最大的问题是空间碎片(清除垃圾之后剩下不连续的内存空间)；
- Copying: 复制算法对于短命对象来说有用，否则需要复制大量的对象，效率低**如Java的新生代堆空间中就是使用了它(survivor空间的from和to区)；**
- Mark-Compact: 标记整理对于老年对象来说有用，无需复制，不会产生内存碎片

**GC考虑的指标**

- 吞吐量: 应用耗时和实际耗时的比值；
- 停顿时间: 垃圾回收的时候，由于Stop the World，应用程序的所有线程会挂起，造成应用停顿

```text
吞吐量和停顿时间是互斥的
对于后端服务(比如后台计算任务)，吞吐量优先考虑(并行垃圾回收)；
对于前端应用，RT响应时间优先考虑，减少垃圾收集时的停顿时间，适用场景是Web系统(并发垃圾回收)
```

**回收器的JVM参数**

- -XX:+UseSerialGC

串行垃圾回收，现在基本很少使用

- -XX:+UseParNewGC

新生代使用并行，老年代使用串行；

- -XX:+UseConcMarkSweepGC

新生代使用并行，老年代使用CMS(一般都是使用这种方式)，CMS是Concurrent Mark Sweep的缩写，并发标记清除，一看就是老年代的算法，所以，它可以作为老年代的垃圾回收器CMS不是独占式的，它关注停顿时间

- -XX:ParallelGCThreads

指定并行的垃圾回收线程的数量，最好等于CPU数量

- -XX:+DisableExplicitGC

禁用System.gc()，因为它会触发Full GC，这是很浪费性能的，JVM会在需要GC的时候自己触发GC

- -XX:CMSFullGCsBeforeCompaction

在多少次GC后进行内存压缩，这个是因为并行收集器不对内存空间进行压缩的，所以运行一段时间后会产生很多碎片，使得运行效率降低

- -XX:+CMSParallelRemarkEnabled

降低标记停顿

- -XX:+UseCMSCompactAtFullCollection

在每一次Full GC时对老年代区域碎片整理，因为CMS是不会移动内存的，因此会非常容易出现碎片导致内存不够用的

- -XX:+UseCmsInitiatingOccupancyOnly

使用手动触发或者自定义触发cms 收集，同时也会禁止hostspot 自行触发CMS GC

- -XX:CMSInitiatingOccupancyFraction

使用CMS作为垃圾回收，使用70%后开始CMS收集

- -XX:CMSInitiatingPermOccupancyFraction

设置perm gen使用达到多少％比时触发垃圾回收，默认是92%

- -XX:+CMSIncrementalMode

设置为增量模式

- -XX:+CmsClassUnloadingEnabled

CMS是不会默认对永久代进行垃圾回收的，设置此参数则是开启

- -XX:+PrintGCDetails

开启详细GC日志模式，日志的格式是和所使用的算法有关

- -XX:+PrintGCDateStamps

将时间和日期也加入到GC日志中











# 调试排错 - Java 内存分析之堆内存和MetaSpace内存

> 本文以两个简单的例子(`堆内存溢出`和`MetaSpace (元数据) 内存溢出`）解释Java 内存溢出的分析过程

调试排错 - Java 内存分析之堆内存和MetaSpace内存

- 常见的内存溢出问题(内存和MetaSpace内存)
  - Java 堆内存溢出
    - OutOfMemoryError: Java heap space
    - OutOfMemoryError: GC overhead limit exceeded
  - MetaSpace (元数据) 内存溢出



分析案例

- 堆内存dump
- 使用MAT分析内存







## 常见的内存溢出问题(内存和MetaSpace内存)

> 常见的内存溢出问题(内存和MetaSpace内存)



### Java 堆内存溢出

Java 堆内存（Heap Memory)主要有两种形式的错误：

1. OutOfMemoryError: Java heap space
2. OutOfMemoryError: GC overhead limit exceeded



#### OutOfMemoryError: Java heap space

在 Java 堆中只要不断的创建对象，并且 `GC-Roots` 到对象之间存在引用链，这样 `JVM` 就不会回收对象

只要将`-Xms(最小堆)`,`-Xmx(最大堆)` 设置为一样禁止自动扩展堆内存

当使用一个 `while(true)` 循环来不断创建对象就会发生 `OutOfMemory`，还可以使用 `-XX:+HeapDumpOutofMemoryErorr` 。当发生 OOM 时会自动 dump 堆栈到文件中

伪代码:

```java
public static void main(String[] args) {
	List<String> list = new ArrayList<>(10) ;
	while (true){
		list.add("1") ;
	}
}
```

当出现 OOM 时可以通过工具来分析 `GC-Roots` [引用链](https://github.com/crossoverJie/Java-Interview/blob/master/MD/GarbageCollection.md#可达性分析算法) ，查看对象和 `GC-Roots` 是如何进行关联的，是否存在对象的生命周期过长，或者是这些对象确实该存在的，那就要考虑将堆内存调大了

```text
Exception in thread "main" Java.lang.OutOfMemoryError: Java heap space
	at Java.util.Arrays.copyOf(Arrays.Java:3210)
	at Java.util.Arrays.copyOf(Arrays.Java:3181)
	at Java.util.ArrayList.grow(ArrayList.Java:261)
	at Java.util.ArrayList.ensureExplicitCapacity(ArrayList.Java:235)
	at Java.util.ArrayList.ensureCapacityInternal(ArrayList.Java:227)
	at Java.util.ArrayList.add(ArrayList.Java:458)
	at com.crossoverjie.oom.HeapOOM.main(HeapOOM.Java:18)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.Java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.Java:43)
	at Java.lang.reflect.Method.invoke(Method.Java:498)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.Java:147)

Process finished with exit code 1
```

`Java.lang.OutOfMemoryError: Java heap space`表示堆内存溢出







#### OutOfMemoryError: GC overhead limit exceeded

GC overhead limt exceed检查是Hotspot VM 1.6定义的一个策略，通过统计GC时间来预测是否要OOM了，提前抛出异常，防止OOM发生。Sun 官方对此的定义是：“并行/并发回收器在GC回收时间过长时会抛出OutOfMemroyError。过长的定义是，超过98%的时间用来做GC并且回收了不到2%的堆内存，用来避免内存过小造成应用不能正常工作“

PS：-Xmx最大内存配置2GB

```java
public void testOom1() {
	List<Map<String, Object>> mapList = new ArrayList<>();
	for (int i = 0; i < 1000000; i++) {
		Map<String, Object> map = new HashMap<>();
		for (int j = 0; j < i; j++) {
				map.put(String.valueOf(j), j);
		}
		mapList.add(map);
	}
}
```

上述的代码执行会：old区占用过多导致频繁Full GC，最终导致GC overhead limit exceed

```java
Java.lang.OutOfMemoryError: GC overhead limit exceeded
	at Java.util.HashMap.newNode(HashMap.Java:1747) ~[na:1.8.0_181]
	at Java.util.HashMap.putVal(HashMap.Java:642) ~[na:1.8.0_181]
	at Java.util.HashMap.put(HashMap.Java:612) ~[na:1.8.0_181]
	at tech.pdai.test.oom.controller.TestOomController.testOom1(TestOomController.Java:33) ~[Classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_181]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.Java:62) ~[na:1.8.0_181]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.Java:43) ~[na:1.8.0_181]
	at Java.lang.reflect.Method.invoke(Method.Java:498) ~[na:1.8.0_181]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.Java:197) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.Java:141) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.Java:106) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.Java:895) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.Java:808) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.Java:87) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.Java:1064) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.Java:963) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.Java:1006) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.Java:898) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at Javax.servlet.http.HttpServlet.service(HttpServlet.Java:655) ~[tomcat-embed-core-9.0.50.jar:4.0.FR]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.Java:883) ~[spring-webmvc-5.3.9.jar:5.3.9]
	at Javax.servlet.http.HttpServlet.service(HttpServlet.Java:764) ~[tomcat-embed-core-9.0.50.jar:4.0.FR]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:228) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:163) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.Java:53) ~[tomcat-embed-websocket-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:190) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:163) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.Java:100) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.Java:119) ~[spring-web-5.3.9.jar:5.3.9]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:190) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:163) ~[tomcat-embed-core-9.0.50.jar:9.0.50]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.Java:93) ~[spring-web-5.3.9.jar:5.3.9]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.Java:119) ~[spring-web-5.3.9.jar:5.3.9]
```

还可以使用 `-XX:+HeapDumpOutofMemoryErorr` 。当发生 OOM 时会自动 dump 堆栈到文件中

JVM还有这样一个参数：`-XX:-UseGCOverheadLimit` 设置为false可以禁用这个检查。其实这个参数解决不了内存问题，只是把错误的信息延后，替换成 Java.lang.OutOfMemoryError: Java heap space







### MetaSpace (元数据) 内存溢出

> `JDK8` 中将永久代移除，使用 `MetaSpace` 来保存类加载之后的类信息，字符串常量池也被移动到 Java 堆

`PermSize` 和 `MaxPermSize` 已经不能使用了，在 JDK8 中配置这两个参数将会发出警告

JDK 8 中将类信息移到到了本地堆内存(Native Heap)中，将原有的永久代移动到了本地堆中成为 `MetaSpace` ,如果不指定该区域的大小，JVM 将会动态的调整

可以使用 `-XX:MaxMetaspaceSize=10M` 来限制最大元数据。这样当不停的创建类时将会占满该区域并出现 `OOM`

```java
public static void main(String[] args) {
	while (true){
		Enhancer  enhancer = new Enhancer() ;
		enhancer.setSuperClass(HeapOOM.Class);
		enhancer.setUseCache(false) ;
		enhancer.setCallback(new MethodInterceptor() {
			@Override
			public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
				return methodProxy.invoke(o,objects) ;
			}
		});
		enhancer.create() ;

	}
}
```

使用 `cglib` 不停的创建新类，最终会抛出:

```text
Caused by: Java.lang.reflect.InvocationTargetException
	at sun.reflect.GeneratedMethodAccessor1.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.Java:43)
	at Java.lang.reflect.Method.invoke(Method.Java:498)
	at net.sf.cglib.core.ReflectUtils.defineClass(ReflectUtils.Java:459)
	at net.sf.cglib.core.AbstractClassGenerator.generate(AbstractClassGenerator.Java:336)
	... 11 more
Caused by: Java.lang.OutOfMemoryError: Metaspace
	at Java.lang.ClassLoader.defineClass1(Native Method)
	at Java.lang.ClassLoader.defineClass(ClassLoader.Java:763)
	... 16 more
```

注意: 这里的 OOM 伴随的是 `Java.lang.OutOfMemoryError: Metaspace` 也就是元数据溢出









## 分析案例

> 在实际工作中，如何去定位内存泄漏问题呢？





### 堆内存dump

1. **通过OOM获取**

即在OutOfMemoryError后获取一份HPROF二进制Heap Dump文件，在JVM中添加参数：

```bash
-XX:+HeapDumpOnOutOfMemoryError
```

2. **主动获取**

在虚拟机添加参数如下，然后在Ctrl+Break组合键即可获取一份Heap Dump

```bash
-XX:+HeapDumpOnCtrlBreak
```

3. **使用HPROF agent**

使用Agent可以在程序执行结束时或受到SIGOUT信号时生成Dump文件

配置在虚拟机的参数如下：

```bash
-agentlib:hprof=heap=dump,format=b
```

4. **jmap获取** (常用)

jmap可以在cmd里执行，命令如下：

```bash
jmap -dump:format=b file=<文件名XX.hprof> <pid>
```

5. **使用JConsole**

Acquire Heap Dump

6. **使用JProfile**

Acquire Heap Dump





### 使用MAT分析内存

MAT 等工具可以看后面的：Java 问题排查之JVM可视化工具 - MAT 一节









# 调试排错 - Java 内存分析之堆外内存

> Java 堆外内存分析相对来说是复杂的，美团技术团队的[Spring Boot引起的“堆外内存泄漏”排查及经验总结](https://tech.meituan.com/2019/01/03/spring-boot-native-memory-leak.html) 可以为很多Native Code内存泄漏/占用提供方向性指引



排查过程

- 使用Java层面的工具定位内存区域
- 使用系统层面的工具定位堆外内存
  - 首先，使用了gperftools去定位问题
  - 然后，使用strace去追踪系统调用
  - 接着，使用GDB去dump可疑内存
  - 再次，项目启动时使用strace去追踪系统调用
  - 最后，使用jstack去查看对应的线程





## 背景

为了更好地实现对项目的管理，我们将组内一个项目迁移到MDP框架（基于Spring Boot），随后我们就发现系统会频繁报出Swap区域使用量过高的异常笔者被叫去帮忙查看原因，发现配置了4G堆内内存，但是实际使用的物理内存竟然高达7G，确实不正常JVM参数配置是“-XX:MetaspaceSize=256M -XX:MaxMetaspaceSize=256M -XX:+AlwaysPreTouch -XX:ReservedCodeCacheSize=128m -XX:InitialCodeCacheSize=128m, -Xss512k -Xmx4g -Xms4g,-XX:+UseG1GC -XX:G1HeapRegionSize=4M”，实际使用的物理内存如下图所示：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808094404585-1516195765.png)





## 排查过程

### 使用Java层面的工具定位内存区域

> 使用Java层面的工具可以定位出堆内内存、Code区域或者使用unsafe.allocateMemory和DirectByteBuffer申请的堆外内存

笔者在项目中添加`-XX:NativeMemoryTracking=detailJVM`参数重启项目，使用命令`jcmd pid VM.native_memory detail`查看到的内存分布如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808094500738-1455092746.png)



发现命令显示的committed的内存小于物理内存，因为jcmd命令显示的内存包含堆内内存、Code区域、通过unsafe.allocateMemory和DirectByteBuffer申请的内存，但是不包含其他Native Code（C代码）申请的堆外内存所以猜测是使用Native Code申请内存所导致的问题

为了防止误判，笔者使用了pmap查看内存分布，发现大量的64M的地址；而这些地址空间不在jcmd命令所给出的地址空间里面，基本上就断定就是这些64M的内存所导致

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808094518688-1492181013.png)





### 使用系统层面的工具定位堆外内存

因为笔者已经基本上确定是Native Code所引起，而Java层面的工具不便于排查此类问题，只能使用系统层面的工具去定位问题



#### 首先，使用了gperftools去定位问题

gperftools的使用方法可以参考gperftools，gperftools的监控如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808094936938-1670660541.png)

从上图可以看出：使用malloc申请的的内存最高到3G之后就释放了，之后始终维持在700M-800M笔者第一反应是：难道Native Code中没有使用malloc申请，直接使用mmap/brk申请的？（gperftools原理就使用动态链接的方式替换了操作系统默认的内存分配器（glibc））





#### 然后，使用strace去追踪系统调用

因为使用gperftools没有追踪到这些内存，于是直接使用命令“strace -f -e”brk,mmap,munmap” -p pid”追踪向OS申请内存请求，但是并没有发现有可疑内存申请strace监控如下图所示:

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095008011-1860672220.jpg)





#### 接着，使用GDB去dump可疑内存

因为使用strace没有追踪到可疑内存申请；于是想着看看内存中的情况就是直接使用命令gdp -pid pid进入GDB之后，然后使用命令dump memory mem.bin startAddress endAddressdump内存，其中startAddress和endAddress可以从/proc/pid/smaps中查找然后使用strings mem.bin查看dump的内容，如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095031274-305312245.jpg)

从内容上来看，像是解压后的JAR包信息读取JAR包信息应该是在项目启动的时候，那么在项目启动之后使用strace作用就不是很大了所以应该在项目启动的时候使用strace，而不是启动完成之后





#### 再次，项目启动时使用strace去追踪系统调用

项目启动使用strace追踪系统调用，发现确实申请了很多64M的内存空间，截图如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095053968-176237807.png)

使用该mmap申请的地址空间在pmap对应如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095106613-575396555.png)





#### 最后，使用jstack去查看对应的线程

因为strace命令中已经显示申请内存的线程ID直接使用命令jstack pid去查看线程栈，找到对应的线程栈（注意10进制和16进制转换）如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095126639-857266485.png)

这里基本上就可以看出问题来了：MCC（美团统一配置中心）使用了Reflections进行扫包，底层使用了Spring Boot去加载JAR因为解压JAR使用Inflater类，需要用到堆外内存，然后使用Btrace去追踪这个类，栈如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095138239-1883047332.png)

然后查看使用MCC的地方，发现没有配置扫包路径，默认是扫描所有的包于是修改代码，配置扫包路径，发布上线后内存问题解决





### 为什么堆外内存没有释放掉呢？

虽然问题已经解决了，但是有几个疑问：

- 为什么使用旧的框架没有问题？
- 为什么堆外内存没有释放？
- 为什么内存大小都是64M，JAR大小不可能这么大，而且都是一样大？
- 为什么gperftools最终显示使用的的内存大小是700M左右，解压包真的没有使用malloc申请内存吗？

带着疑问，笔者直接看了一下[Spring Boot Loader](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-tools/spring-boot-loader/src/main/Java/org/springframework/boot/loader)那一块的源码发现Spring Boot对Java JDK的InflaterInputStream进行了包装并且使用了Inflater，而Inflater本身用于解压JAR包的需要用到堆外内存而包装之后的类ZipInflaterInputStream没有释放Inflater持有的堆外内存于是笔者以为找到了原因，立马向Spring Boot社区反馈了[这个bug](https://github.com/spring-projects/spring-boot/issues/13935)但是反馈之后，笔者就发现Inflater这个对象本身实现了finalize方法，在这个方法中有调用释放堆外内存的逻辑也就是说Spring Boot依赖于GC释放堆外内存

笔者使用jmap查看堆内对象时，发现已经基本上没有Inflater这个对象了于是就怀疑GC的时候，没有调用finalize带着这样的怀疑，笔者把Inflater进行包装在Spring Boot Loader里面替换成自己包装的Inflater，在finalize进行打点监控，结果finalize方法确实被调用了于是笔者又去看了Inflater对应的C代码，发现初始化的使用了malloc申请内存，end的时候也调用了free去释放内存

此刻，笔者只能怀疑free的时候没有真正释放内存，便把Spring Boot包装的InflaterInputStream替换成Java JDK自带的，发现替换之后，内存问题也得以解决了

这时，再返过来看gperftools的内存分布情况，发现使用Spring Boot时，内存使用一直在增加，突然某个点内存使用下降了好多（使用量直接由3G降为700M左右）这个点应该就是GC引起的，内存应该释放了，但是在操作系统层面并没有看到内存变化，那是不是没有释放到操作系统，被内存分配器持有了呢？

继续探究，发现系统默认的内存分配器（glibc 2.12版本）和使用gperftools内存地址分布差别很明显，2.5G地址使用smaps发现它是属于Native Stack内存地址分布如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095204725-81859413.png)

到此，基本上可以确定是内存分配器在捣鬼；搜索了一下glibc 64M，发现glibc从2.11开始对每个线程引入内存池（64位机器大小就是64M内存），原文如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095218005-1758207980.jpg)

按照文中所说去修改MALLOC_ARENA_MAX环境变量，发现没什么效果查看tcmalloc（gperftools使用的内存分配器）也使用了内存池方式

为了验证是内存池搞的鬼，笔者就简单写个不带内存池的内存分配器使用命令`gcc zjbmalloc.c -fPIC -shared -o zjbmalloc.so`生成动态库，然后使用`export LD_PRELOAD=zjbmalloc.so`替换掉glibc的内存分配器其中代码Demo如下：

```c
#include<sys/mman.h>
#include<stdlib.h>
#include<string.h>
#include<stdio.h>
//作者使用的64位机器，sizeof(size_t)也就是sizeof(long) 
void* malloc ( size_t size )
{
   long* ptr = mmap( 0, size + sizeof(long), PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, 0, 0 );
   if (ptr == MAP_FAILED) {
  	return NULL;
   }
   *ptr = size;                     // First 8 bytes contain length.
   return (void*)(&ptr[1]);        // Memory that is after length variable
}

void *calloc(size_t n, size_t size) {
 void* ptr = malloc(n * size);
 if (ptr == NULL) {
	return NULL;
 }
 memset(ptr, 0, n * size);
 return ptr;
}
void *realloc(void *ptr, size_t size)
{
 if (size == 0) {
	free(ptr);
	return NULL;
 }
 if (ptr == NULL) {
	return malloc(size);
 }
 long *plen = (long*)ptr;
 plen--;                          // Reach top of memory
 long len = *plen;
 if (size <= len) {
	return ptr;
 }
 void* rptr = malloc(size);
 if (rptr == NULL) {
	free(ptr);
	return NULL;
 }
 rptr = memcpy(rptr, ptr, len);
 free(ptr);
 return rptr;
}

void free (void* ptr )
{
   if (ptr == NULL) {
	 return;
   }
   long *plen = (long*)ptr;
   plen--;                          // Reach top of memory
   long len = *plen;               // Read length
   munmap((void*)plen, len + sizeof(long));
}
```

通过在自定义分配器当中埋点可以发现其实程序启动之后应用实际申请的堆外内存始终在700M-800M之间，gperftools监控显示内存使用量也是在700M-800M左右但是从操作系统角度来看进程占用的内存差别很大（这里只是监控堆外内存）

笔者做了一下测试，使用不同分配器进行不同程度的扫包，占用的内存如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095233051-140720846.jpg)

**为什么自定义的malloc申请800M，最终占用的物理内存在1.7G呢**？

因为自定义内存分配器采用的是mmap分配内存，mmap分配内存按需向上取整到整数个页，所以存在着巨大的空间浪费通过监控发现最终申请的页面数目在536k个左右，那实际上向系统申请的内存等于512k * 4k（pagesize） = 2G

**为什么这个数据大于1.7G呢**？

因为操作系统采取的是延迟分配的方式，通过mmap向系统申请内存的时候，系统仅仅返回内存地址并没有分配真实的物理内存只有在真正使用的时候，系统产生一个缺页中断，然后再分配实际的物理Page





## 总结

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095254597-1520485178.jpg)

整个内存分配的流程如上图所示MCC扫包的默认配置是扫描所有的JAR包在扫描包的时候，Spring Boot不会主动去释放堆外内存，导致在扫描阶段，堆外内存占用量一直持续飙升当发生GC的时候，Spring Boot依赖于finalize机制去释放了堆外内存；但是glibc为了性能考虑，并没有真正把内存归返到操作系统，而是留下来放入内存池了，导致应用层以为发生了“内存泄漏”所以修改MCC的配置路径为特定的JAR包，问题解决笔者在发表这篇文章时，发现**Spring Boot的最新版本（2.0.5.RELEASE）已经做了修改，在ZipInflaterInputStream主动释放了堆外内存不再依赖GC**；所以Spring Boot升级到最新版本，这个问题也可以得到解决





## 参考资料

- GNU C Library (glibc)
- [Native Memory Tracking](https://docs.oracle.com/Javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html)
- [gperftools](https://github.com/gperftools/gperftools)
- [Btrace](https://github.com/btraceio/btrace)



作者简介

- 纪兵，2015年加入美团，目前主要从事酒店C端相关的工作







# 调试排错 - Java 线程分析之线程Dump分析

> Thread Dump是非常有用的诊断Java应用问题的工具





## Thread Dump介绍

### 什么是Thread Dump

Thread Dump是非常有用的诊断Java应用问题的工具每一个Java虚拟机都有及时生成所有线程在某一点状态的thread-dump的能力，虽然各个 Java虚拟机打印的thread dump略有不同，但是 大多都提供了当前活动线程的快照，及JVM中所有Java线程的堆栈跟踪信息，堆栈信息一般包含完整的类名及所执行的方法，如果可能的话还有源代码的行数



### Thread Dump特点

- 能在各种操作系统下使用；
- 能在各种Java应用服务器下使用；
- 能在生产环境下使用而不影响系统的性能；
- 能将问题直接定位到应用程序的代码行上；



### Thread Dump抓取

一般当服务器挂起，崩溃或者性能低下时，就需要抓取服务器的线程堆栈（Thread Dump）用于后续的分析在实际运行中，往往一次 dump的信息，还不足以确认问题为了反映线程状态的动态变化，需要接连多次做thread dump，每次间隔10-20s，建议至少产生三次 dump信息，如果每次 dump都指向同一个问题，我们才确定问题的典型性

- 操作系统命令获取ThreadDump

```bash
ps –ef | grep Java
kill -3 <pid>
```

注意：

> 一定要谨慎, 一步不慎就可能让服务器进程被杀死kill -9 命令会杀死进程

- JVM 自带的工具获取线程堆栈

```bash
jps 或 ps –ef | grep Java （获取PID）
jstack [-l ] <pid> | tee -a jstack.log（获取ThreadDump）
```



## Thread Dump分析

### Thread Dump信息

- 头部信息：时间，JVM信息

```bash
2011-11-02 19:05:06  
Full thread dump Java HotSpot(TM) Server VM (16.3-b01 mixed mode): 
```

- 线程INFO信息块：

```bash
1. "Timer-0" daemon prio=10 tid=0xac190c00 nid=0xaef in Object.wait() [0xae77d000] 
# 线程名称：Timer-0；线程类型：daemon；优先级: 10，默认是5；
# JVM线程id：tid=0xac190c00，JVM内部线程的唯一标识（通过Java.lang.Thread.getId()获取，通常用自增方式实现）
# 对应系统线程id（NativeThread ID）：nid=0xaef，和top命令查看的线程pid对应，不过一个是10进制，一个是16进制（通过命令：top -H -p pid，可以查看该进程的所有线程信息）
# 线程状态：in Object.wait()；
# 起始栈地址：[0xae77d000]，对象的内存地址，通过JVM内存查看工具，能够看出线程是在哪儿个对象上等待；
2.  Java.lang.Thread.State: TIMED_WAITING (on object monitor)
3.  at Java.lang.Object.wait(Native Method)
4.  -waiting on <0xb3885f60> (a Java.util.TaskQueue)     # 继续wait 
5.  at Java.util.TimerThread.mainLoop(Timer.Java:509)
6.  -locked <0xb3885f60> (a Java.util.TaskQueue)         # 已经locked
7.  at Java.util.TimerThread.run(Timer.Java:462)
Java thread statck trace：是上面2-7行的信息到目前为止这是最重要的数据，Java stack trace提供了大部分信息来精确定位问题根源
```

- Java thread statck trace详解：

**堆栈信息应该逆向解读**：程序先执行的是第7行，然后是第6行，依次类推

```bash
- locked <0xb3885f60> (a Java.util.ArrayList)
- waiting on <0xb3885f60> (a Java.util.ArrayList) 
```

**也就是说对象先上锁，锁住对象0xb3885f60，然后释放该对象锁，进入waiting状态**为啥会出现这样的情况呢？看看下面的Java代码示例，就会明白：

```java
synchronized(obj) {  
   .........  
   obj.wait();  
   .........  
}
```

如上，线程的执行过程，先用 `synchronized` 获得了这个对象的 Monitor（对应于 `locked <0xb3885f60>` ）当执行到 `obj.wait()`，线程即放弃了 Monitor的所有权，进入 “wait set”队列（对应于 `waiting on <0xb3885f60>` ）

**在堆栈的第一行信息中，进一步标明了线程在代码级的状态**，例如：

```bash
Java.lang.Thread.State: TIMED_WAITING (parking)
```

解释如下：

```bash
|blocked|

> This thread tried to enter asynchronized block, but the lock was taken by another thread. This thread isblocked until the lock gets released.

|blocked (on thin lock)|

> This is the same state asblocked, but the lock in question is a thin lock.

|waiting|

> This thread calledObject.wait() on an object. The thread will remain there until some otherthread sends a notification to that object.

|sleeping|

> This thread calledJava.lang.Thread.sleep().

|parked|

> This thread calledJava.util.concurrent.locks.LockSupport.park().

|suspended|

> The thread's execution wassuspended by Java.lang.Thread.suspend() or a JVMTI agent call.
```





### Thread状态分析

线程的状态是一个很重要的东西，因此thread dump中会显示这些状态，通过对这些状态的分析，能够得出线程的运行状况，进而发现可能存在的问题**线程的状态在Thread.State这个枚举类型中定义**：

```java
public enum State   
{  
       /** 
        * Thread state for a thread which has not yet started. 
        */  
       NEW,  
         
       /** 
        * Thread state for a runnable thread.  A thread in the runnable 
        * state is executing in the Java virtual machine but it may 
        * be waiting for other resources from the operating system 
        * such as processor. 
        */  
       RUNNABLE,  
         
       /** 
        * Thread state for a thread blocked waiting for a monitor lock. 
        * A thread in the blocked state is waiting for a monitor lock 
        * to enter a synchronized block/method or  
        * reenter a synchronized block/method after calling 
        * {@link Object#wait() Object.wait}. 
        */  
       BLOCKED,  
     
       /** 
        * Thread state for a waiting thread. 
        * A thread is in the waiting state due to calling one of the  
        * following methods: 
        * <ul> 
        *   <li>{@link Object#wait() Object.wait} with no timeout</li> 
        *   <li>{@link #join() Thread.join} with no timeout</li> 
        *   <li>{@link LockSupport#park() LockSupport.park}</li> 
        * </ul> 
        *  
        * <p>A thread in the waiting state is waiting for another thread to 
        * perform a particular action.   
        * 
        * For example, a thread that has called <tt>Object.wait()</tt> 
        * on an object is waiting for another thread to call  
        * <tt>Object.notify()</tt> or <tt>Object.notifyAll()</tt> on  
        * that object. A thread that has called <tt>Thread.join()</tt>  
        * is waiting for a specified thread to terminate. 
        */  
       WAITING,  
         
       /** 
        * Thread state for a waiting thread with a specified waiting time. 
        * A thread is in the timed waiting state due to calling one of  
        * the following methods with a specified positive waiting time: 
        * <ul> 
        *   <li>{@link #sleep Thread.sleep}</li> 
        *   <li>{@link Object#wait(long) Object.wait} with timeout</li> 
        *   <li>{@link #join(long) Thread.join} with timeout</li> 
        *   <li>{@link LockSupport#parkNanos LockSupport.parkNanos}</li>  
        *   <li>{@link LockSupport#parkUntil LockSupport.parkUntil}</li> 
        * </ul> 
        */  
       TIMED_WAITING,  
  
       /** 
        * Thread state for a terminated thread. 
        * The thread has completed execution. 
        */  
       TERMINATED;  
}
```

- NEW：

每一个线程，在堆内存中都有一个对应的Thread对象Thread t = new Thread();当刚刚在堆内存中创建Thread对象，还没有调用t.start()方法之前，线程就处在NEW状态在这个状态上，线程与普通的Java对象没有什么区别，就仅仅是一个堆内存中的对象

- RUNNABLE：

该状态表示线程具备所有运行条件，在运行队列中准备操作系统的调度，或者正在运行 这个状态的线程比较正常，但如果线程长时间停留在在这个状态就不正常了，这说明线程运行的时间很长（存在性能问题），或者是线程一直得不得执行的机会（存在线程饥饿的问题）

- BLOCKED：

线程正在等待获取Java对象的监视器(也叫内置锁)，即线程正在等待进入由synchronized保护的方法或者代码块synchronized用来保证原子性，任意时刻最多只能由一个线程进入该临界区域，其他线程只能排队等待

- WAITING：

处在该线程的状态，正在等待某个事件的发生，只有特定的条件满足，才能获得执行机会而产生这个特定的事件，通常都是另一个线程也就是说，如果不发生特定的事件，那么处在该状态的线程一直等待，不能获取执行的机会比如：

A线程调用了obj对象的obj.wait()方法，如果没有线程调用obj.notify或obj.notifyAll，那么A线程就没有办法恢复运行； 如果A线程调用了LockSupport.park()，没有别的线程调用LockSupport.unpark(A)，那么A没有办法恢复运行 TIMED_WAITING：

J.U.C中很多与线程相关类，都提供了限时版本和不限时版本的APITIMED_WAITING意味着线程调用了限时版本的API，正在等待时间流逝当等待时间过去后，线程一样可以恢复运行如果线程进入了WAITING状态，一定要特定的事件发生才能恢复运行；而处在TIMED_WAITING的线程，如果特定的事件发生或者是时间流逝完毕，都会恢复运行

- TERMINATED：

线程执行完毕，执行完run方法正常返回，或者抛出了运行时异常而结束，线程都会停留在这个状态这个时候线程只剩下Thread对象了，没有什么用了





### 关键状态分析

- **Wait on condition**：The thread is either sleeping or waiting to be notified by another thread.

该状态说明它在等待另一个条件的发生，来把自己唤醒，或者干脆它是调用了 sleep(n)

此时线程状态大致为以下几种：

```bash
Java.lang.Thread.State: WAITING (parking)：一直等那个条件发生；
Java.lang.Thread.State: TIMED_WAITING (parking或sleeping)：定时的，那个条件不到来，也将定时唤醒自己
```

- **Waiting for Monitor Entry 和 in Object.wait()**：The thread is waiting to get the lock for an object (some other thread may be holding the lock). This happens if two or more threads try to execute synchronized code. Note that the lock is always for an object and not for individual methods.

在多线程的Java程序中，实现线程之间的同步，就要说说 Monitor**Monitor是Java中用以实现线程之间的互斥与协作的主要手段，它可以看成是对象或者Class的锁每一个对象都有，也仅有一个 Monitor** 下面这个图，描述了线程和 Monitor之间关系，以及线程的状态转换图：

![image](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230808095633290-196184083.png)

如上图，每个Monitor在某个时刻，只能被一个线程拥有，**该线程就是 “ActiveThread”，而其它线程都是 “Waiting Thread”，分别在两个队列“Entry Set”和“Wait Set”里等候**在“Entry Set”中等待的线程状态是“Waiting for monitor entry”，而在“Wait Set”中等待的线程状态是“in Object.wait()”

先看“Entry Set”里面的线程我们称被 synchronized保护起来的代码段为临界区**当一个线程申请进入临界区时，它就进入了“Entry Set”队列**对应的 code就像：

```java
synchronized(obj) {
   .........
}
```

这时有两种可能性：

- 该 monitor不被其它线程拥有， Entry Set里面也没有其它等待线程本线程即成为相应类或者对象的 Monitor的 Owner，执行临界区的代码
- 该 monitor被其它线程拥有，本线程在 Entry Set队列中等待

在第一种情况下，线程将处于 “Runnable”的状态，而第二种情况下，线程 DUMP会显示处于 “waiting for monitor entry”如下：

```bash
"Thread-0" prio=10 tid=0x08222eb0 nid=0x9 waiting for monitor entry [0xf927b000..0xf927bdb8] 
at testthread.WaitThread.run(WaitThread.Java:39) 
- waiting to lock <0xef63bf08> (a Java.lang.Object) 
- locked <0xef63beb8> (a Java.util.ArrayList) 
at Java.lang.Thread.run(Thread.Java:595) 
```

**临界区的设置，是为了保证其内部的代码执行的原子性和完整性**但是因为临界区在任何时间只允许线程串行通过，这和我们多线程的程序的初衷是相反的**如果在多线程的程序中，大量使用 synchronized，或者不适当的使用了它，会造成大量线程在临界区的入口等待，造成系统的性能大幅下降**如果在线程 DUMP中发现了这个情况，应该审查源码，改进程序

再看“Wait Set”里面的线程**当线程获得了 Monitor，进入了临界区之后，如果发现线程继续运行的条件没有满足，它则调用对象（一般就是被 synchronized 的对象）的 wait() 方法，放弃 Monitor，进入 “Wait Set”队列只有当别的线程在该对象上调用了 notify() 或者 notifyAll()，“Wait Set”队列中线程才得到机会去竞争**，但是只有一个线程获得对象的Monitor，恢复到运行态在 “Wait Set”中的线程， DUMP中表现为： in Object.wait()如下：

```bash
"Thread-1" prio=10 tid=0x08223250 nid=0xa in Object.wait() [0xef47a000..0xef47aa38] 
 at Java.lang.Object.wait(Native Method) 
 - waiting on <0xef63beb8> (a Java.util.ArrayList) 
 at Java.lang.Object.wait(Object.Java:474) 
 at testthread.MyWaitThread.run(MyWaitThread.Java:40) 
 - locked <0xef63beb8> (a Java.util.ArrayList) 
 at Java.lang.Thread.run(Thread.Java:595) 
综上，一般CPU很忙时，则关注runnable的线程，CPU很闲时，则关注waiting for monitor entry的线程
```

- **JDK 5.0 的 Lock**

上面提到如果 synchronized和 monitor机制运用不当，可能会造成多线程程序的性能问题在 JDK 5.0中，引入了 Lock机制，从而使开发者能更灵活的开发高性能的并发多线程程序，可以替代以往 JDK中的 synchronized和 Monitor的 机制但是，**要注意的是，因为 Lock类只是一个普通类，JVM无从得知 Lock对象的占用情况，所以在线程 DUMP中，也不会包含关于 Lock的信息**， 关于死锁等问题，就不如用 synchronized的编程方式容易识别





### 关键状态示例

- **显示BLOCKED状态**

```java
package jstack;  

public Class BlockedState  
{  
    private static Object object = new Object();  
    
    public static void main(String[] args)  
    {  
        Runnable task = new Runnable() {  

            @Override  
            public void run()  
            {  
                synchronized (object)  
                {  
                    long begin = System.currentTimeMillis();  
  
                    long end = System.currentTimeMillis();  

                    // 让线程运行5分钟,会一直持有object的监视器  
                    while ((end - begin) <= 5 * 60 * 1000)  
                    {  
  
                    }  
                }  
            }  
        };  

        new Thread(task, "t1").start();  
        new Thread(task, "t2").start();  
    }  
} 
```

先获取object的线程会执行5分钟，**这5分钟内会一直持有object的监视器，另一个线程无法执行处在BLOCKED状态**：

```java
Full thread dump Java HotSpot(TM) Server VM (20.12-b01 mixed mode):  
  
"DestroyJavaVM" prio=6 tid=0x00856c00 nid=0x1314 waiting on condition [0x00000000]  
Java.lang.Thread.State: RUNNABLE  

"t2" prio=6 tid=0x27d7a800 nid=0x1350 waiting for monitor entry [0x2833f000]  
Java.lang.Thread.State: BLOCKED (on object monitor)  
     at jstack.BlockedState$1.run(BlockedState.Java:17)  
     - waiting to lock <0x1cfcdc00> (a Java.lang.Object)  
     at Java.lang.Thread.run(Thread.Java:662)  

"t1" prio=6 tid=0x27d79400 nid=0x1338 runnable [0x282ef000]  
 Java.lang.Thread.State: RUNNABLE  
     at jstack.BlockedState$1.run(BlockedState.Java:22)  
     - locked <0x1cfcdc00> (a Java.lang.Object)  
     at Java.lang.Thread.run(Thread.Java:662)
```

通过thread dump可以看到：**t2线程确实处在BLOCKED (on object monitor)waiting for monitor entry 等待进入synchronized保护的区域**

- **显示WAITING状态**

```java
package jstack;  
  
public Class WaitingState  
{  
    private static Object object = new Object();  

    public static void main(String[] args)  
    {  
        Runnable task = new Runnable() {  

            @Override  
            public void run()  
            {  
                synchronized (object)  
                {  
                    long begin = System.currentTimeMillis();  
                    long end = System.currentTimeMillis();  

                    // 让线程运行5分钟,会一直持有object的监视器  
                    while ((end - begin) <= 5 * 60 * 1000)  
                    {  
                        try  
                        {  
                            // 进入等待的同时,会进入释放监视器  
                            object.wait();  
                        } catch (InterruptedException e)  
                        {  
                            e.printStackTrace();  
                        }  
                    }  
                }  
            }  
        };  

        new Thread(task, "t1").start();  
        new Thread(task, "t2").start();  
    }  
}  
Full thread dump Java HotSpot(TM) Server VM (20.12-b01 mixed mode):  

"DestroyJavaVM" prio=6 tid=0x00856c00 nid=0x1734 waiting on condition [0x00000000]  
Java.lang.Thread.State: RUNNABLE  

"t2" prio=6 tid=0x27d7e000 nid=0x17f4 in Object.wait() [0x2833f000]  
Java.lang.Thread.State: WAITING (on object monitor)  
     at Java.lang.Object.wait(Native Method)  
     - waiting on <0x1cfcdc00> (a Java.lang.Object)  
     at Java.lang.Object.wait(Object.Java:485)  
     at jstack.WaitingState$1.run(WaitingState.Java:26)  
     - locked <0x1cfcdc00> (a Java.lang.Object)  
     at Java.lang.Thread.run(Thread.Java:662)  

"t1" prio=6 tid=0x27d7d400 nid=0x17f0 in Object.wait() [0x282ef000]  
Java.lang.Thread.State: WAITING (on object monitor)  
     at Java.lang.Object.wait(Native Method)  
     - waiting on <0x1cfcdc00> (a Java.lang.Object)  
     at Java.lang.Object.wait(Object.Java:485)  
     at jstack.WaitingState$1.run(WaitingState.Java:26)  
     - locked <0x1cfcdc00> (a Java.lang.Object)  
     at Java.lang.Thread.run(Thread.Java:662)  
```

可以发现t1和t2都处在WAITING (on object monitor)，进入等待状态的原因是调用了in Object.wait()通过J.U.C包下的锁和条件队列，也是这个效果，大家可以自己实践下

- **显示TIMED_WAITING状态**

```java
package jstack;  

import Java.util.concurrent.TimeUnit;  
import Java.util.concurrent.locks.Condition;  
import Java.util.concurrent.locks.Lock;  
import Java.util.concurrent.locks.ReentrantLock;  
  
public Class TimedWaitingState  
{  
    // Java的显示锁,类似Java对象内置的监视器  
    private static Lock lock = new ReentrantLock();  
  
    // 锁关联的条件队列(类似于object.wait)  
    private static Condition condition = lock.newCondition();  

    public static void main(String[] args)  
    {  
        Runnable task = new Runnable() {  

            @Override  
            public void run()  
            {  
                // 加锁,进入临界区  
                lock.lock();  
  
                try  
                {  
                    condition.await(5, TimeUnit.MINUTES);  
                } catch (InterruptedException e)  
                {  
                    e.printStackTrace();  
                }  
  
                // 解锁,退出临界区  
                lock.unlock();  
            }  
        };  
  
        new Thread(task, "t1").start();  
        new Thread(task, "t2").start();  
    }  
} 
Full thread dump Java HotSpot(TM) Server VM (20.12-b01 mixed mode):  

"DestroyJavaVM" prio=6 tid=0x00856c00 nid=0x169c waiting on condition [0x00000000]  
Java.lang.Thread.State: RUNNABLE  

"t2" prio=6 tid=0x27d7d800 nid=0xc30 waiting on condition [0x2833f000]  
Java.lang.Thread.State: TIMED_WAITING (parking)  
     at sun.misc.Unsafe.park(Native Method)  
     - parking to wait for  <0x1cfce5b8> (a Java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)  
     at Java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.Java:196)  
     at Java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.Java:2116)  
     at jstack.TimedWaitingState$1.run(TimedWaitingState.Java:28)  
     at Java.lang.Thread.run(Thread.Java:662)  

"t1" prio=6 tid=0x280d0c00 nid=0x16e0 waiting on condition [0x282ef000]  
Java.lang.Thread.State: TIMED_WAITING (parking)  
     at sun.misc.Unsafe.park(Native Method)  
     - parking to wait for  <0x1cfce5b8> (a Java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)  
     at Java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.Java:196)  
     at Java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.Java:2116)  
     at jstack.TimedWaitingState$1.run(TimedWaitingState.Java:28)  
     at Java.lang.Thread.run(Thread.Java:662)  
```

可以看到t1和t2线程都处在Java.lang.Thread.State: TIMED_WAITING (parking)，这个parking代表是调用的JUC下的工具类，而不是Java默认的监视器





## 案例分析

### 问题场景

1. **CPU飙高，load高，响应很慢**

一个请求过程中多次dump；

对比多次dump文件的runnable线程，如果执行的方法有比较大变化，说明比较正常如果在执行同一个方法，就有一些问题了；



2. **查找占用CPU最多的线程**

使用命令：top -H -p pid（pid为被测系统的进程号），找到导致CPU高的线程ID，对应thread dump信息中线程的nid，只不过一个是十进制，一个是十六进制；

在thread dump中，根据top命令查找的线程id，查找对应的线程堆栈信息；



3. **CPU使用率不高但是响应很慢**

进行dump，查看是否有很多thread struck在了i/o、数据库等地方，定位瓶颈原因；



4. **请求无法响应**

多次dump，对比是否所有的runnable线程都一直在执行相同的方法，如果是的，恭喜你，锁住了！





### 死锁

死锁经常表现为程序的停顿，或者不再响应用户的请求从操作系统上观察，对应进程的CPU占用率为零，很快会从top或prstat的输出中消失

比如在下面这个示例中，是个较为典型的死锁情况：

```java
"Thread-1" prio=5 tid=0x00acc490 nid=0xe50 waiting for monitor entry [0x02d3f000 
..0x02d3fd68] 
at deadlockthreads.TestThread.run(TestThread.Java:31) 
- waiting to lock <0x22c19f18> (a Java.lang.Object) 
- locked <0x22c19f20> (a Java.lang.Object) 

"Thread-0" prio=5 tid=0x00accdb0 nid=0xdec waiting for monitor entry [0x02cff000 
..0x02cff9e8] 
at deadlockthreads.TestThread.run(TestThread.Java:31) 
- waiting to lock <0x22c19f20> (a Java.lang.Object) 
- locked <0x22c19f18> (a Java.lang.Object) 
```

在 Java 5中加强了对死锁的检测**线程 Dump中可以直接报告出 Java级别的死锁**，如下所示：

```java
Found one Java-level deadlock: 
============================= 
"Thread-1": 
waiting to lock monitor 0x0003f334 (object 0x22c19f18, a Java.lang.Object), 
which is held by "Thread-0" 

"Thread-0": 
waiting to lock monitor 0x0003f314 (object 0x22c19f20, a Java.lang.Object), 
which is held by "Thread-1"
```





### 热锁

热锁，也往往是导致系统性能瓶颈的主要因素其表现特征为：**由于多个线程对临界区，或者锁的竞争**，可能出现：

- **频繁的线程的上下文切换**：从操作系统对线程的调度来看，当线程在等待资源而阻塞的时候，操作系统会将之切换出来，放到等待的队列，当线程获得资源之后，调度算法会将这个线程切换进去，放到执行队列中
- **大量的系统调用**：因为线程的上下文切换，以及热锁的竞争，或者临界区的频繁的进出，都可能导致大量的系统调用
- **大部分CPU开销用在“系统态”**：线程上下文切换，和系统调用，都会导致 CPU在 “系统态 ”运行，换而言之，虽然系统很忙碌，但是CPU用在 “用户态 ”的比例较小，应用程序得不到充分的 CPU资源
- **随着CPU数目的增多，系统的性能反而下降**因为CPU数目多，同时运行的线程就越多，可能就会造成更频繁的线程上下文切换和系统态的CPU开销，从而导致更糟糕的性能

上面的描述，都是一个 scalability（可扩展性）很差的系统的表现从整体的性能指标看，由于线程热锁的存在，程序的响应时间会变长，吞吐量会降低

**那么，怎么去了解 “热锁 ”出现在什么地方呢**？

一个重要的方法是 结合操作系统的各种工具观察系统资源使用状况，以及收集Java线程的DUMP信息，看线程都阻塞在什么方法上，了解原因，才能找到对应的解决方法







## JVM重要线程

JVM运行过程中产生的一些比较重要的线程罗列如下：

| 线程名称                        | 解释说明                                                     |
| ------------------------------- | ------------------------------------------------------------ |
| Attach Listener                 | Attach Listener 线程是负责接收到外部的命令，而对该命令进行执行的并把结果返回给发送者通常我们会用一些命令去要求JVM给我们一些反馈信息，如：Java -version、jmap、jstack等等 如果该线程在JVM启动的时候没有初始化，那么，则会在用户第一次执行JVM命令时，得到启动 |
| Signal Dispatcher               | 前面提到Attach Listener线程的职责是接收外部JVM命令，当命令接收成功后，会交给signal dispather线程去进行分发到各个不同的模块处理命令，并且返回处理结果signal dispather线程也是在第一次接收外部JVM命令时，进行初始化工作 |
| CompilerThread0                 | 用来调用JITing，实时编译装卸Class  通常，JVM会启动多个线程来处理这部分工作，线程名称后面的数字也会累加，例如：CompilerThread1 |
| Concurrent Mark-Sweep GC Thread | 并发标记清除垃圾回收器（就是通常所说的CMS GC）线程， 该线程主要针对于老年代垃圾回收ps：启用该垃圾回收器，需要在JVM启动参数中加上：-XX:+UseConcMarkSweepGC |
| DestroyJavaVM                   | 执行main()的线程，在main执行完后调用JNI中的 jni_DestroyJavaVM() 方法唤起DestroyJavaVM 线程，处于等待状态，等待其它线程（Java线程和Native线程）退出时通知它卸载JVM每个线程退出时，都会判断自己当前是否是整个JVM中最后一个非deamon线程，如果是，则通知DestroyJavaVM 线程卸载JVM |
| Finalizer Thread                | 这个线程也是在main线程之后创建的，其优先级为10，主要用于在垃圾收集前，调用对象的finalize()方法；关于Finalizer线程的几点：1) 只有当开始一轮垃圾收集时，才会开始调用finalize()方法；因此并不是所有对象的finalize()方法都会被执行；2) 该线程也是daemon线程，因此如果虚拟机中没有其他非daemon线程，不管该线程有没有执行完finalize()方法，JVM也会退出；3) JVM在垃圾收集时会将失去引用的对象包装成Finalizer对象（Reference的实现），并放入ReferenceQueue，由Finalizer线程来处理；最后将该Finalizer对象的引用置为null，由垃圾收集器来回收；4) JVM为什么要单独用一个线程来执行finalize()方法呢？如果JVM的垃圾收集线程自己来做，很有可能由于在finalize()方法中误操作导致GC线程停止或不可控，这对GC线程来说是一种灾难； |
| Low Memory Detector             | 这个线程是负责对可使用内存进行检测，如果发现可用内存低，分配新的内存空间 |
| Reference Handler               | JVM在创建main线程后就创建Reference Handler线程，其优先级最高，为10，它主要用于处理引用对象本身（软引用、弱引用、虚引用）的垃圾回收问题 |
| VM Thread                       | 这个线程就比较牛b了，是JVM里面的线程母体，根据hotspot源码（vmThread.hpp）里面的注释，它是一个单个的对象（最原始的线程）会产生或触发所有其他的线程，这个单个的VM线程是会被其他线程所使用来做一些VM操作（如：清扫垃圾等） |





## 参考文章

- 作者：猿码架构
- 链接：https://www.jianshu.com/p/f1db856022de
- 来源：简书
- 著作权归作者所有商业转载请联系作者获得授权，非商业转载请注明出处









# 调试排错 - Java 问题排查之Linux命令

> Java 在线问题排查主要分两篇：本文是第一篇，通过linux常用命令排查



## 文本操作

### 文本查找 - grep

grep常用命令：

```bash
# 基本使用
grep yoursearchkeyword f.txt     #文件查找
grep 'KeyWord otherKeyWord' f.txt cpf.txt #多文件查找, 含空格加引号
grep 'KeyWord' /home/admin -r -n #目录下查找所有符合关键字的文件
grep 'keyword' /home/admin -r -n -i # -i 忽略大小写
grep 'KeyWord' /home/admin -r -n --include *.{vm,Java} #指定文件后缀
grep 'KeyWord' /home/admin -r -n --exclude *.{vm,Java} #反匹配

# cat + grep
cat f.txt | grep -i keyword # 查找所有keyword且不分大小写  
cat f.txt | grep -c 'KeyWord' # 统计Keyword次数

# seq + grep
seq 10 | grep 5 -A 3    #上匹配
seq 10 | grep 5 -B 3    #下匹配
seq 10 | grep 5 -C 3    #上下匹配，平时用这个就妥了
```

Grep的参数：

```bash
--color=auto：显示颜色;
-i, --ignore-case：忽略字符大小写;
-o, --only-matching：只显示匹配到的部分;
-n, --line-number：显示行号;
-v, --invert-match：反向显示,显示未匹配到的行;
-E, --extended-regexp：支持使用扩展的正则表达式;
-q, --quiet, --silent：静默模式,即不输出任何信息;
-w, --word-regexp：整行匹配整个单词;
-c, --count：统计匹配到的行数; print a count of matching lines;

-B, --before-context=NUM：print NUM lines of leading context   后#行 
-A, --after-context=NUM：print NUM lines of trailing context   前#行 
-C, --context=NUM：print NUM lines of output context           前后各#行 
```



### 文本分析 - awk

awk基本命令：

```bash
# 基本使用
awk '{print $4,$6}' f.txt
awk '{print NR,$0}' f.txt cpf.txt    
awk '{print FNR,$0}' f.txt cpf.txt
awk '{print FNR,FILENAME,$0}' f.txt cpf.txt
awk '{print FILENAME,"NR="NR,"FNR="FNR,"$"NF"="$NF}' f.txt cpf.txt
echo 1:2:3:4 | awk -F: '{print $1,$2,$3,$4}'

# 匹配
awk '/ldb/ {print}' f.txt   #匹配ldb
awk '!/ldb/ {print}' f.txt  #不匹配ldb
awk '/ldb/ && /LISTEN/ {print}' f.txt   #匹配ldb和LISTEN
awk '$5 ~ /ldb/ {print}' f.txt #第五列匹配ldb
```

内建变量

```bash
`NR`: NR表示从awk开始执行后，按照记录分隔符读取的数据次数，默认的记录分隔符为换行符，因此默认的就是读取的数据行数，NR可以理解为Number of Record的缩写

`FNR`: 在awk处理多个输入文件的时候，在处理完第一个文件后，NR并不会从1开始，而是继续累加，因此就出现了FNR，每当处理一个新文件的时候，FNR就从1开始计数，FNR可以理解为File Number of Record

`NF`: NF表示目前的记录被分割的字段的数目，NF可以理解为Number of Field
```

更多请参考：[Linux awk 命令](https://www.runoob.com/linux/linux-comm-awk.html)

### [#](#文本处理-sed) 文本处理 - sed

sed常用：

```bash
# 文本打印
sed -n '3p' xxx.log #只打印第三行
sed -n '$p' xxx.log #只打印最后一行
sed -n '3,9p' xxx.log #只查看文件的第3行到第9行
sed -n -e '3,9p' -e '=' xxx.log #打印3-9行，并显示行号
sed -n '/root/p' xxx.log #显示包含root的行
sed -n '/hhh/,/omc/p' xxx.log # 显示包含"hhh"的行到包含"omc"的行之间的行

# 文本替换
sed -i 's/root/world/g' xxx.log # 用world 替换xxx.log文件中的root; s==search  查找并替换, g==global  全部替换, -i: implace

# 文本插入
sed '1,4i hahaha' xxx.log # 在文件第一行和第四行的每行下面添加hahaha
sed -e '1i happy' -e '$a new year' xxx.log  #【界面显示】在文件第一行添加happy,文件结尾添加new year
sed -i -e '1i happy' -e '$a new year' xxx.log #【真实写入文件】在文件第一行添加happy,文件结尾添加new year

# 文本删除
sed  '3,9d' xxx.log # 删除第3到第9行,只是不显示而已
sed '/hhh/,/omc/d' xxx.log # 删除包含"hhh"的行到包含"omc"的行之间的行
sed '/omc/,10d' xxx.log # 删除包含"omc"的行到第十行的内容

# 与find结合
find . -name  "*.txt" |xargs   sed -i 's/hhhh/\hHHh/g'
find . -name  "*.txt" |xargs   sed -i 's#hhhh#hHHh#g'
find . -name  "*.txt" -exec sed -i 's/hhhh/\hHHh/g' {} \;
find . -name  "*.txt" |xargs cat
```

更多请参考：[Linux sed 命令](https://www.runoob.com/linux/linux-comm-sed.html) 或者 [Linux sed命令详解](https://www.cnblogs.com/ftl1012/p/9250171.html)





## 文件操作

### 文件监听 - tail

最常用的`tail -f filename`

```bash
# 基本使用
tail -f xxx.log # 循环监听文件
tail -300f xxx.log #倒数300行并追踪文件
tail +20 xxx.log #从第 20 行至文件末尾显示文件内容

# tailf使用
tailf xxx.log #等同于tail -f -n 10 打印最后10行，然后追踪文件
```

tail -f 与tail F 与tailf三者区别

```bash
`tail  -f `  等于--follow=descriptor，根据文件描述进行追踪，当文件改名或删除后，停止追踪

`tail -F` 等于 --follow=name ==retry，根据文件名字进行追踪，当文件改名或删除后，保持重试，当有新的文件和他同名时，继续追踪

`tailf` 等于tail -f -n 10（tail -f或-F默认也是打印最后10行，然后追踪文件），与tail -f不同的是，如果文件不增长，它不会去访问磁盘文件，所以tailf特别适合那些便携机上跟踪日志文件，因为它减少了磁盘访问，可以省电
```

tail的参数

```bash
-f 循环读取
-q 不显示处理信息
-v 显示详细的处理信息
-c<数目> 显示的字节数
-n<行数> 显示文件的尾部 n 行内容
--pid=PID 与-f合用,表示在进程ID,PID死掉之后结束
-q, --quiet, --silent 从不输出给出文件名的首部
-s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒
```





### 文件查找 - find

```bash
sudo -u admin find /home/admin /tmp /usr -name \*.log(多个目录去找)
find . -iname \*.txt(大小写都匹配)
find . -type d(当前目录下的所有子目录)
find /usr -type l(当前目录下所有的符号链接)
find /usr -type l -name "z*" -ls(符号链接的详细信息 eg:inode,目录)
find /home/admin -size +250000k(超过250000k的文件，当然+改成-就是小于了)
find /home/admin f -perm 777 -exec ls -l {} \; (按照权限查询文件)
find /home/admin -atime -1  1天内访问过的文件
find /home/admin -ctime -1  1天内状态改变过的文件    
find /home/admin -mtime -1  1天内修改过的文件
find /home/admin -amin -1  1分钟内访问过的文件
find /home/admin -cmin -1  1分钟内状态改变过的文件    
find /home/admin -mmin -1  1分钟内修改过的文件
```





### pgm

批量查询vm-shopbase满足条件的日志

```bash
pgm -A -f vm-shopbase 'cat /home/admin/shopbase/logs/shopbase.log.2017-01-17|grep 2069861630'
```





## 查看网络和进程

### 查看所有网络接口的属性

```bash
[root@pdai.tech ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.31.165.194  netmask 255.255.240.0  broadcast 172.31.175.255
        ether 00:16:3e:08:c1:ea  txqueuelen 1000  (Ethernet)
        RX packets 21213152  bytes 2812084823 (2.6 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 25264438  bytes 46566724676 (43.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 502  bytes 86350 (84.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 502  bytes 86350 (84.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```



### 查看防火墙设置

```bash
[root@pdai.tech ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```



### 查看路由表

```bash
[root@pdai.tech ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.175.253  0.0.0.0         UG    0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.31.160.0    0.0.0.0         255.255.240.0   U     0      0        0 eth0
```



### netstat

查看所有监听端口

```bash
[root@pdai.tech ~]# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name  
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:9999            0.0.0.0:*               LISTEN      1249/Java         
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1547/sshd         
tcp6       0      0 :::3306                 :::*                    LISTEN      1894/mysqld       
```

查看所有已经建立的连接

```bash
[root@pdai.tech ~]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:9999            0.0.0.0:*               LISTEN      1249/Java
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      970/nginx: master p
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1547/sshd
tcp        0      0 172.31.165.194:53874    100.100.30.25:80        ESTABLISHED 18041/AliYunDun
tcp        0     64 172.31.165.194:22       xxx.194.1.200:2649      ESTABLISHED 32516/sshd: root@pt
tcp6       0      0 :::3306                 :::*                    LISTEN      1894/m
```

查看当前连接

```bash
[root@pdai.tech ~]# netstat -nat|awk  '{print $6}'|sort|uniq -c|sort -rn
      5 LISTEN
      2 ESTABLISHED
      1 Foreign
      1 established)
```

查看网络统计信息进程

```bash
[root@pdai.tech ~]# netstat -s
Ip:
    21017132 total packets received
    0 forwarded
    0 incoming packets discarded
    21017131 incoming packets delivered
    25114367 requests sent out
    324 dropped because of missing route
Icmp:
    18088 ICMP messages received
    692 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 4241
        timeout in transit: 19
        echo requests: 13791
        echo replies: 4
        timestamp request: 33
    13825 ICMP messages sent
    0 ICMP messages failed
    ICMP output histogram:
        destination unreachable: 1
        echo replies: 13791
        timestamp replies: 33
IcmpMsg:
        InType0: 4
        InType3: 4241
        InType8: 13791
        InType11: 19
        InType13: 33
        OutType0: 13791
        OutType3: 1
        OutType14: 33
Tcp:
    12210 active connections openings
    208820 passive connection openings
    54198 failed connection attempts
    9805 connection resets received
...
```

netstat 请参考这篇文章: [Linux netstat命令详解](https://www.cnblogs.com/ftl1012/p/netstat.html)



### 查看所有进程

```bash
[root@pdai.tech ~]# ps -ef | grep Java
root      1249     1  0 Nov04 ?        00:58:05 Java -jar /opt/tech_doc/bin/tech_arch-0.0.1-RELEASE.jar --server.port=9999
root     32718 32518  0 08:36 pts/0    00:00:00 grep --color=auto Java
```



### top

top除了看一些基本信息之外，剩下的就是配合来查询vm的各种问题了

```bash
# top -H -p pid
top - 08:37:51 up 45 days, 18:45,  1 user,  load average: 0.01, 0.03, 0.05
Threads:  28 total,   0 running,  28 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882088 total,    74608 free,   202228 used,  1605252 buff/cache
KiB Swap:  2097148 total,  1835392 free,   261756 used.  1502036 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
 1347 root      20   0 2553808 113752   1024 S  0.3  6.0  48:46.74 VM Periodic Tas
 1249 root      20   0 2553808 113752   1024 S  0.0  6.0   0:00.00 Java
 1289 root      20   0 2553808 113752   1024 S  0.0  6.0   0:03.74 Java
...
```



## 查看磁盘和内存相关

### 查看内存使用 - free -m

```bash
[root@pdai.tech ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1837         196         824           0         816        1469
Swap:          2047         255        1792
```



### 查看各分区使用情况

```bash
[root@pdai.tech ~]# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        909M     0  909M   0% /dev
tmpfs           919M     0  919M   0% /dev/shm
tmpfs           919M  452K  919M   1% /run
tmpfs           919M     0  919M   0% /sys/fs/cgroup
/dev/vda1        40G   15G   23G  40% /
tmpfs           184M     0  184M   0% /run/user/0
```



### 查看指定目录的大小

```bash
[root@pdai.tech ~]# du -sh
803M
```



### 查看内存总量

```bash
[root@pdai.tech ~]# grep MemTotal /proc/meminfo
MemTotal:        1882088 kB
```



### 查看空闲内存量

```bash
[root@pdai.tech ~]# grep MemFree /proc/meminfo
MemFree:           74120 kB
```



### 查看系统负载磁盘和分区

```bash
[root@pdai.tech ~]# grep MemFree /proc/meminfo
MemFree:           74120 kB
```



### 查看系统负载磁盘和分区

```bash
[root@pdai.tech ~]# cat /proc/loadavg
0.01 0.04 0.05 2/174 32751
```



### 查看挂接的分区状态

```bash
[root@pdai.tech ~]# mount | column -t
sysfs       on  /sys                             type  sysfs       (rw,nosuid,nodev,noexec,relatime)
proc        on  /proc                            type  proc        (rw,nosuid,nodev,noexec,relatime)
devtmpfs    on  /dev                             type  devtmpfs    (rw,nosuid,size=930732k,nr_inodes=232683,mode=755)
securityfs  on  /sys/kernel/security             type  securityfs  (rw,nosuid,nodev,noexec,relatime)
...
```



### 查看所有分区

```bash
[root@pdai.tech ~]# fdisk -l

Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x0008d73a

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83884031    41940992   83  Linux
```



### 查看所有交换分区

```bash
[root@pdai.tech ~]# swapon -s
Filename                                Type            Size    Used    Priority
/etc/swap                               file    2097148 261756  -2
```



### 查看硬盘大小

```bash
[root@pdai.tech ~]# fdisk -l |grep Disk
Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Disk label type: dos
Disk identifier: 0x0008d73a
```





## 查看用户和组相关

### 查看活动用户

```shell
[root@pdai.tech ~]# w
 08:47:20 up 45 days, 18:54,  1 user,  load average: 0.01, 0.03, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    xxx.194.1.200    08:32    0.00s  0.32s  0.32s -bash
```



### 查看指定用户信息

```bash
[root@pdai.tech ~]# id
uid=0(root) gid=0(root) groups=0(root)
```



### 查看用户登录日志

```bash
[root@pdai.tech ~]# last
root     pts/0        xxx.194.1.200    Fri Dec 20 08:32   still logged in
root     pts/0        xxx.73.164.60     Thu Dec 19 21:47 - 00:28  (02:41)
root     pts/0        xxx.106.236.255  Thu Dec 19 16:00 - 18:24  (02:23)
root     pts/1        xxx.194.3.173    Tue Dec 17 13:35 - 17:37  (04:01)
root     pts/0        xxx.194.3.173    Tue Dec 17 13:35 - 17:37  (04:02)
...
```



### 查看系统所有用户

```bash
[root@pdai.tech ~]# cut -d: -f1 /etc/passwd
root
bin
daemon
adm
...
```



### 查看系统所有组

```bash
cut -d: -f1 /etc/group
```



## 查看服务，模块和包相关

```bash
# 查看当前用户的计划任务服务
crontab -l 

# 列出所有系统服务
chkconfig –list 

# 列出所有启动的系统服务程序
chkconfig –list | grep on 

# 查看所有安装的软件包
rpm -qa 

# 列出加载的内核模块
lsmod 
```



## 查看系统，设备，环境信息

```bash
# 常用
env # 查看环境变量资源
uptime # 查看系统运行时间、用户数、负载
lsusb -tv # 列出所有USB设备的linux系统信息命令
lspci -tv # 列出所有PCI设备
head -n 1 /etc/issue # 查看操作系统版本，是数字1不是字母L
uname -a # 查看内核/操作系统/CPU信息的linux系统信息命令

# /proc/
cat /proc/cpuinfo ：查看CPU相关参数的linux系统命令
cat /proc/partitions ：查看linux硬盘和分区信息的系统信息命令
cat /proc/meminfo ：查看linux系统内存信息的linux系统命令
cat /proc/version ：查看版本，类似uname -r
cat /proc/ioports ：查看设备io端口
cat /proc/interrupts ：查看中断
cat /proc/pci ：查看pci设备的信息
cat /proc/swaps ：查看所有swap分区的信息
cat /proc/cpuinfo |grep "model name" && cat /proc/cpuinfo |grep "physical id"
```



## tsar

tsar是淘宝开源的的采集工具很好用, 将历史收集到的数据持久化在磁盘上，所以我们快速来查询历史的系统数据当然实时的应用情况也是可以查询的啦大部分机器上都有安装

```bash
tsar  ##可以查看最近一天的各项指标
tsar --live ##可以查看实时指标，默认五秒一刷
tsar -d 20161218 ##指定查看某天的数据，貌似最多只能看四个月的数据
tsar --mem
tsar --load
tsar --cpu ##当然这个也可以和-d参数配合来查询某天的单个指标的情况 
```

具体可以看这篇文章：[linux 淘宝开源监控工具tsa](https://www.jianshu.com/p/5562854ed901)









# 调试排错 - Java 问题排查之工具单

> Java 在线问题排查主要分两篇：本文是第二篇，通过Java调试/排查工具进行问题定位





## Java 调试入门工具

### jps

> jps是JDK提供的一个查看当前Java进程的小工具， 可以看做是JavaVirtual Machine Process Status Tool的缩写

jps常用命令

```bash
jps # 显示进程的ID 和 类的名称
jps –l # 输出输出完全的包名，应用主类名，jar的完全路径名 
jps –v # 输出JVM参数
jps –q # 显示Java进程号
jps -m # main 方法
jps -l xxx.xxx.xx.xx # 远程查看 
```

jps参数

```bash
-q：仅输出VM标识符，不包括Classname,jar name,arguments in main method 
-m：输出main method的参数 
-l：输出完全的包名，应用主类名，jar的完全路径名 
-v：输出JVM参数 
-V：输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件 
-Joption：传递参数到vm,例如:-J-Xms512m
```

jps原理

> Java程序在启动以后，会在Java.io.tmpdir指定的目录下，就是临时文件夹里，生成一个类似于hsperfdata_User的文件夹，这个文件夹里（在Linux中为/tmp/hsperfdata_{userName}/），有几个文件，名字就是Java进程的pid，因此列出当前运行的Java进程，只是把这个目录里的文件名列一下而已 至于系统的参数什么，就可以解析这几个文件获得

更多请参考 [jps - Java Virtual Machine Process Status Tool](https://docs.oracle.com/Javase/1.5.0/docs/tooldocs/share/jps.html)





### jstack

> jstack是JDK自带的线程堆栈分析工具，使用该命令可以查看或导出 Java 应用程序中线程堆栈信息

jstack常用命令:

```bash
# 基本
jstack 2815

# Java和native c/c++框架的所有栈信息
jstack -m 2815

# 额外的锁信息列表，查看是否死锁
jstack -l 2815
```

jstack参数：

```bash
-l 长列表. 打印关于锁的附加信息,例如属于Java.util.concurrent 的 ownable synchronizers列表.

-F 当’jstack [-l] pid’没有相应的时候强制打印栈信息

-m 打印Java和native c/c++框架的所有栈信息.

-h | -help 打印帮助信息
```

更多请参考: [JVM 性能调优工具之 jstack](https://www.jianshu.com/p/025cb069cb69)

### jinfo

> jinfo 是 JDK 自带的命令，可以用来查看正在运行的 Java 应用程序的扩展参数，包括Java System属性和JVM命令行参数；也可以动态的修改正在运行的 JVM 一些参数当系统崩溃时，jinfo可以从core文件里面知道崩溃的Java应用程序的配置信息

jinfo常用命令:

```bash
# 输出当前 JVM 进程的全部参数和系统属性
jinfo 2815

# 输出所有的参数
jinfo -flags 2815

# 查看指定的 JVM 参数的值
jinfo -flag PrintGC 2815

# 开启/关闭指定的JVM参数
jinfo -flag +PrintGC 2815

# 设置flag的参数
jinfo -flag name=value 2815

# 输出当前 JVM 进行的全部的系统属性
jinfo -sysprops 2815
```

jinfo参数：

```bash
no option 输出全部的参数和系统属性
-flag name 输出对应名称的参数
-flag [+|-]name 开启或者关闭对应名称的参数
-flag name=value 设定对应名称的参数
-flags 输出全部的参数
-sysprops 输出系统属性
```

更多请参考：[JVM 性能调优工具之 jinfo](https://www.jianshu.com/p/8d8aef212b25)





### jmap

> 命令jmap是一个多功能的命令它可以生成 Java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列

两个用途

```bash
# 查看堆的情况
jmap -heap 2815

# dump
jmap -dump:live,format=b,file=/tmp/heap2.bin 2815
jmap -dump:format=b,file=/tmp/heap3.bin 2815

# 查看堆的占用
jmap -histo 2815 | head -10
```

jmap参数

```bash
no option： 查看进程的内存映像信息,类似 Solaris pmap 命令
heap： 显示Java堆详细信息
histo[:live]： 显示堆中对象的统计信息
clstats：打印类加载器信息
finalizerinfo： 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
dump:<dump-options>：生成堆转储快照
F： 当-dump没有响应时，使用-dump或者-histo参数. 在这个模式下,live子参数无效.
help：打印帮助信息
J<flag>：指定传递给运行jmap的JVM的参数
```

更多请参考：

1. [JVM 性能调优工具之 jmap](https://www.jianshu.com/p/a4ad53179df3) 
2. [jmap - Memory Map](https://docs.oracle.com/Javase/1.5.0/docs/tooldocs/share/jmap.html)





### jstat

jstat参数众多，但是使用一个就够了

```bash
jstat -gcutil 2815 1000 
```





### jdb

jdb可以用来预发debug,假设你预发的Java_home是/opt/Java/，远程调试端口是8000.那么

```bash
jdb -attach 8000
```

出现以上代表jdb启动成功后续可以进行设置断点进行调试

具体参数可见oracle官方说明[jdb - The Java Debugger](http://docs.oracle.com/Javase/7/docs/technotes/tools/windows/jdb.html)





###  CHLSDB

CHLSDB感觉很多情况下可以看到更好玩的东西，不详细叙述了。查询资料听说jstack和jmap等工具就是基于它的

```bash
Java -Classpath /opt/taobao/Java/lib/sa-jdi.jar sun.JVM.hotspot.CLHSDB
```

更详细的可见R大此贴 http://rednaxelafx.iteye.com/blog/1847971





## Java 调试进阶工具

### btrace

首当其冲的要说的是btrace真是生产环境&预发的排查问题大杀器 简介什么的就不说了直接上代码干

- 查看当前谁调用了ArrayList的add方法，同时只打印当前ArrayList的size大于500的线程调用栈

```java
@OnMethod(clazz = "Java.util.ArrayList", method="add", location = @Location(value = Kind.CALL, clazz = "/./", method = "/./"))
public static void m(@ProbeClassName String probeClass, @ProbeMethodName String probeMethod, @TargetInstance Object instance, @TargetMethodOrField String method) {

    if(getInt(field("Java.util.ArrayList", "size"), instance) > 479){
        println("check who ArrayList.add method:" + probeClass + "#" + probeMethod  + ", method:" + method + ", size:" + getInt(field("Java.util.ArrayList", "size"), instance));
        jstack();
        println();
        println("===========================");
        println();
    }
}
```

- 监控当前服务方法被调用时返回的值以及请求的参数

```java
@OnMethod(clazz = "com.taobao.sellerhome.transfer.biz.impl.C2CApplyerServiceImpl", method="nav", location = @Location(value = Kind.RETURN))
public static void mt(long userId, int current, int relation, String check, String redirectUrl, @Return AnyType result) {

    println("parameter# userId:" + userId + ", current:" + current + ", relation:" + relation + ", check:" + check + ", redirectUrl:" + redirectUrl + ", result:" + result);
}
```

btrace 具体可以参考这里：https://github.com/btraceio/btrace

注意:

- 经过观察，1.3.9的release输出不稳定，要多触发几次才能看到正确的结果
- 正则表达式匹配trace类时范围一定要控制，否则极有可能出现跑满CPU导致应用卡死的情况
- 由于是字节码注入的原理，想要应用恢复到正常情况，需要重启应用





### Greys

Greys是@杜琨的大作吧。说几个挺棒的功能(部分功能和btrace重合):

- `sc -df xxx`: 输出当前类的详情,包括源码位置和Classloader结构
- `trace Class method`: 打印出当前方法调用的耗时情况，细分到每个方法, 对排查方法性能时很有帮助





### Arthas

> Arthas是基于Greys

具体请参考：[调试排错 - Java应用在线调试Arthas]()





### javOSize

就说一个功能:

- `Classes`：通过修改了字节码，改变了类的内容，即时生效 所以可以做到快速的在某个地方打个日志看看输出，缺点是对代码的侵入性太大但是如果自己知道自己在干嘛，的确是不错的玩意儿

其他功能Greys和btrace都能很轻易做的到，不说了

更多请参考：[官网](http://www.javosize.com/)





### JProfiler

之前判断许多问题要通过JProfiler，但是现在Greys和btrace基本都能搞定了再加上出问题的基本上都是生产环境(网络隔离)，所以基本不怎么使用了，但是还是要标记一下

更多请参考：[官网](https://www.ej-technologies.com/products/jprofiler/overview.html)





## 其它工具

### dmesg

如果发现自己的Java进程悄无声息的消失了，几乎没有留下任何线索，那么dmesg一发，很有可能有你想要的

sudo dmesg|grep -i kill|less 去找关键字oom_killer找到的结果类似如下:

```bash
[6710782.021013] Java invoked oom-killer: gfp_mask=0xd0, order=0, oom_adj=0, oom_scoe_adj=0
[6710782.070639] [<ffffffff81118898>] ? oom_kill_process+0x68/0x140 
[6710782.257588] Task in /LXC011175068174 killed as a result of limit of /LXC011175068174 
[6710784.698347] Memory cgroup out of memory: Kill process 215701 (Java) score 854 or sacrifice child 
[6710784.707978] Killed process 215701, UID 679, (Java) total-vm:11017300kB, anon-rss:7152432kB, file-rss:1232kB
```

以上表明，对应的Java进程被系统的OOM Killer给干掉了，得分为854. 解释一下OOM killer（Out-Of-Memory killer），该机制会监控机器的内存资源消耗当机器内存耗尽前，该机制会扫描所有的进程（按照一定规则计算，内存占用，时间等），挑选出得分最高的进程，然后杀死，从而保护机器

dmesg日志时间转换公式: log实际时间=格林威治1970-01-01+(当前时间秒数-系统启动至今的秒数+dmesg打印的log时间)秒数：

date -d "1970-01-01 UTC `echo "$(date +%s)-$(cat /proc/uptime|cut -f 1 -d' ')+12288812.926194"|bc ` seconds" 剩下的，就是看看为什么内存这么大，触发了OOM-Killer了





## 参考文章

- 文章主要参考了如下， 在此基础上重新整理，和添加新内容
- 作者：红魔七号
- 文章来源：https://yq.aliyun.com/articles/69520
- https://www.cnblogs.com/xuchunlin/p/5671572.html











# 调试排错 - Java 问题排查之JVM可视化工具

> 本文主要梳理常见的JVM可视化的分析工具，主要包括JConsole, Visual VM, Vusial GC, JProfile 和 MAT等





## JConsole

> Jconsole （Java Monitoring and Management Console），JDK自带的基于JMX的可视化监视、管理工具 官方文档可以参考[这里](https://docs.oracle.com/Javase/8/docs/technotes/guides/management/jconsole.html)

- **找到jconsole工具**

```bash
pdai@MacBook-Pro bin % ls
jaotc		jcmd		jinfo		jshell		rmid
jar		jconsole(这里)	jjs		jstack		rmiregistry
jarsigner	jdb		jlink		jstat		serialver
Java		jdeprscan	jmap		jstatd		unpack200
Javac		jdeps		jmod		keytool
Javadoc		jhsdb		jps		pack200
Javap		jimage		jrunscript	rmic
```

- **打开jconsole**

选择

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809165929422-452809910.png)



![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809165944890-353800184.png)

- **查看概述、内存、线程、类、VM概要、MBean**

概述

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809165959227-659472953.png)

内存

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170011610-786330985.png)

线程

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170023961-261927868.png)

类

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170035057-288784653.png)

VM概要

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170047128-190097670.png)

MBean

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170100090-184050158.png)





## Visual VM

> VisualVM 是一款免费的，集成了多个 JDK 命令行工具的可视化工具，它能为您提供强大的分析能力，对 Java 应用程序做性能分析和调优这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时它还支持在 MBeans 上进行浏览和操作

Overview

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170123376-1311679297.png)

Monitor

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170133378-1983826849.png)

线程

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170149129-297565892.png)

Sampler

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170156135-1916264479.png)





## Visual GC

> visual gc 是 visualvm 中的图形化查看 gc 状况的插件。官方文档可以参考[这里](https://www.oracle.com/Java/technologies/visual-garbage-collection-monitoring-tool.html)

比如我在IDEA中使用visual GC 插件来看GC状况

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170253861-183607199.png)





## JProfile

> Profiler 是一个商业的主要用于检查和跟踪系统（限于Java开发的）的性能的工具。JProfiler可以通过时时的监控系统的内存使用情况，随时监视垃圾回收，线程运行状况等手段，从而很好的监视JVM运行情况及其性能

JProfiler 是一个全功能的Java剖析工具（profiler），专用于分析J2SE和J2EE应用程序。它把CPU、执行绪和内存的剖析组合在一个强大的应用中。 JProfiler可提供许多IDE整合和应用服务器整合用途。JProfiler直觉式的GUI让你可以找到效能瓶颈、抓出内存漏失(memory leaks)、并解决执行绪的问题。它让你得以对heap walker作资源回收器的root analysis，可以轻易找出内存漏失；heap快照（snapshot）模式让未被参照（reference）的对象、稍微被参照的对象、或在终结（finalization）队列的对象都会被移除；整合精灵以便剖析浏览器的Java外挂功能





### 核心组件

JProfiler 包含用于采集目标 JVM 分析数据的 JProfiler agent、用于可视化分析数据的 JProfiler UI、提供各种功能的命令行工具，它们之间的关系如下图所示

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170343851-1651443341.png)

- **JProfiler agent**

JProfiler agent 是一个本地库，它可以在 JVM 启动时通过参数`-agentpath:<path to native library>`进行加载或者在程序运行时通过[JVM Attach 机制](http://lovestblog.cn/blog/2014/06/18/JVM-attach/)进行加载。Agent 被成功加载后，会设置 JVMTI 环境，监听虚拟机产生的事件，如类加载、线程创建等例如，当它监听到类加载事件后，会给这些类注入用于执行度量操作的字节码

- **JProfiler UI**

JProfiler UI 是一个可独立部署的组件，它通过 socket 和 agent 建立连接。这意味着不论目标 JVM 运行在本地还是远端，JProfiler UI 和 agent 间的通信机制都是一样的

JProfiler UI 的主要功能是展示通过 agent 采集上来的分析数据，此外还可以通过它控制 agent 的采集行为，将快照保存至磁盘，展示保存的快照

- **命令行工具**

JProfiler 提供了一系列命令行工具以实现不同的功能

1. jpcontroller - 用于控制 agent 的采集行为。它通过 agent 注册的 JProfiler MBean 向 agent 传递命令
2. jpenable - 用于将 agent 加载到一个正在运行的 JVM 上
3. jpdump - 用于获取正在运行的 JVM 的堆快照
4. jpexport & jpcompare - 用于从保存的快照中提取数据并创建 HTML 报告





### 运行测试

**我们运行一个SpringBoot测试工程，选择attach到JVM**

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170408735-1955370467.png)

选择指定的进程

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170421749-157185647.png)

**设置数据采集模式**

JProfier 提供两种数据采集模式 Sampling 和 Instrumentation

- Sampling - 适合于不要求数据完全精确的场景优点是对系统性能的影响较小，缺点是某些特性不支持（如方法级别的统计信息）
- Instrumentation - 完整功能模式，统计信息也是精确的缺点是如果需要分析的类比较多，对应用性能影响较大为了降低影响，往往需要和 Filter 一起使用

由于我们需要获取方法级别的统计信息，这里选择了 Instrumentation 模式

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170434896-2142954706.png)

概览

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170449421-1699537174.png)

内存

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170459464-1420194444.png)

实时内存分布（类对象）

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170513694-593546012.png)

dump 堆内存

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170524425-923252768.png)

dump完会直接打开显示

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170535111-294274639.png)

线程存储

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170543869-1376552044.png)

导出HTML报告

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170554303-1393753520.png)

CPU 调用树

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170606801-1678949968.png)

线程历史

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170615536-1310210689.png)

JEE & 探针

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170626319-895710042.png)

MBeans

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170638785-209518557.png)





## Eclipse Memory Analyzer (MAT)

> MAT 是一种快速且功能丰富的 Java 堆分析器，可帮助你发现内存泄漏并减少内存消耗。MAT在的堆内存分析问题使用极为广泛，需要重点掌握

可以在[这里](https://www.eclipse.org/mat/)下载， 官方文档可以看[这里](http://help.eclipse.org/latest/index.jsp?topic=/org.eclipse.mat.ui.help/welcome.html)

- **Overview**

包含内存分布，以及潜在的问题推测

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170713462-1903053300.png)

- **Histogram**

可以列出内存中的对象，对象的个数以及大小

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170730044-781653374.png)

具体需要重点理解如下两个概念，可参考[官网文档](http://help.eclipse.org/latest/index.jsp?topic=/org.eclipse.mat.ui.help/welcome.html)的解释

1. Shallow Heap ：一个对象内存的消耗大小，不包含对其他对象的引用
2. Retained Heap ：是shallow Heap的总和，也就是该对象被GC之后所能回收到内存的总和

- **Dominator Tree**

可以列出那个线程，以及线程下面的那些对象占用的空间

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170745176-1587984477.png)

- **Top consumers**

通过图形列出最大的object

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170756841-142803188.png)

- **Leak Suspects**

自动分析潜在可能的泄漏

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809170809044-870334149.png)









# 调试排错 - Java 问题排查之应用在线调试Arthas

> 本文主要介绍Alibaba开源的Java诊断工具，开源到现在已经几万个点赞了，深受开发者喜爱





## Arthas简介

> 在学习Arthas之前，推荐先看上一篇美团技术团队的"Java 动态调试技术原理及实践"，这样你会对它最底层技术有个了解。可以看下文中最后有个对比图：Greys(Arthas也是基于它做的二次开发)和Java-debug-tool



### Arthas是什么

`Arthas` 是Alibaba开源的Java诊断工具，深受开发者喜爱



### Arthas能解决什么问题

当你遇到以下类似问题而束手无策时，`Arthas`可以帮助你解决：

- 这个类从哪个 jar 包加载的? 为什么会报各种类相关的 Exception?
- 我改的代码为什么没有执行到? 难道是我没 commit? 分支搞错了?
- 遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗?
- 线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！
- 是否有一个全局视角来查看系统的运行状况?
- 有什么办法可以监控到JVM的实时运行状态?

`Arthas`支持JDK 6+，支持Linux/Mac/Windows，采用命令行交互模式，同时提供丰富的 `Tab` 自动补全功能，进一步方便进行问题的定位和诊断





### Arthas资源推荐

- [用户文档](https://alibaba.github.io/arthas/)
- [官方在线教程(推荐)](https://alibaba.github.io/arthas/arthas-tutorials?language=cn)
- [快速入门](https://alibaba.github.io/arthas/quick-start.html)
- [进阶使用](https://alibaba.github.io/arthas/advanced-use.html)
- [命令列表](https://alibaba.github.io/arthas/commands.html)
- [WebConsole](https://alibaba.github.io/arthas/web-console.html)
- [Docker](https://alibaba.github.io/arthas/docker.html)
- [用户案例](https://github.com/alibaba/arthas/issues?q=label%3Auser-case)
- [常见问题](https://github.com/alibaba/arthas/issues?utf8=✓&q=label%3Aquestion-answered+)





### Arthas基于了哪些工具上发展而来

- [greys-anatomy](https://github.com/oldmanpushcart/greys-anatomy): Arthas代码基于Greys二次开发而来
- [termd](https://github.com/termd/termd): Arthas的命令行实现基于termd开发，是一款优秀的命令行程序开发框架
- [crash](https://github.com/crashub/crash): Arthas的文本渲染功能基于crash中的文本渲染功能开发，可以从[这里](https://github.com/crashub/crash/tree/1.3.2/shell)看到源码
- [cli](https://github.com/eclipse-vertx/vert.x/tree/master/src/main/Java/io/vertx/core/cli): Arthas的命令行界面基于vert.x提供的cli库进行开发
- [compiler](https://github.com/skalogs/SkaETL/tree/master/compiler) Arthas里的内存编绎器代码来源
- [Apache Commons Net](https://commons.apache.org/proper/commons-net/) Arthas里的Telnet Client代码来源
- `JavaAgent`：运行在 main方法之前的拦截器，它内定的方法名叫 premain ，也就是说先执行 premain 方法然后再执行 main 方法
- `ASM`：一个通用的Java字节码操作和分析框架。它可以用于修改现有的类或直接以二进制形式动态生成类。ASM提供了一些常见的字节码转换和分析算法，可以从它们构建定制的复杂转换和代码分析工具。ASM提供了与其他Java字节码框架类似的功能，但是主要关注性能。因为它被设计和实现得尽可能小和快，所以非常适合在动态系统中使用(当然也可以以静态方式使用，例如在编译器中)





### 同类工具有哪些

- BTrace
- 美团 Java-debug-tool
- [去哪儿Bistoury: 一个集成了Arthas的项目](https://github.com/qunarcorp/bistoury)
- [一个使用MVEL脚本的fork](https://github.com/XhinLiang/arthas)





## Arthas入门

### Arthas 上手前

推荐先在线使用下arthas：[官方在线教程(推荐)](https://alibaba.github.io/arthas/arthas-tutorials?language=cn)





### Arthas 安装

下载`arthas-boot.jar`，然后用`Java -jar`的方式启动：

```bash
curl -O https://alibaba.github.io/arthas/arthas-boot.jar
Java -jar arthas-boot.jar
```



### Arthas 官方案例展示

#### Dashboard

- https://alibaba.github.io/arthas/dashboard

![dashboard](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809171257340-559696411.png)





#### Thread

一目了然的了解系统的状态，哪些线程比较占cpu? 他们到底在做什么?

```bash
$ thread -n 3

"as-command-execute-daemon" Id=29 cpuUsage=75% RUNNABLE
    at sun.management.ThreadImpl.dumpThreads0(Native Method)
    at sun.management.ThreadImpl.getThreadInfo(ThreadImpl.Java:440)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand$1.action(ThreadCommand.Java:58)
    at com.taobao.arthas.core.command.handler.AbstractCommandHandler.execute(AbstractCommandHandler.Java:238)
    at com.taobao.arthas.core.command.handler.DefaultCommandHandler.handleCommand(DefaultCommandHandler.Java:67)
    at com.taobao.arthas.core.server.ArthasServer$4.run(ArthasServer.Java:276)
    at Java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.Java:1145)
    at Java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.Java:615)
    at Java.lang.Thread.run(Thread.Java:745)

    Number of locked synchronizers = 1
    - Java.util.concurrent.ThreadPoolExecutor$Worker@6cd0b6f8

"as-session-expire-daemon" Id=25 cpuUsage=24% TIMED_WAITING
    at Java.lang.Thread.sleep(Native Method)
    at com.taobao.arthas.core.server.DefaultSessionManager$2.run(DefaultSessionManager.Java:85)

"Reference Handler" Id=2 cpuUsage=0% WAITING on Java.lang.ref.Reference$Lock@69ba0f27
    at Java.lang.Object.wait(Native Method)
    -  waiting on Java.lang.ref.Reference$Lock@69ba0f27
    at Java.lang.Object.wait(Object.Java:503)
    at Java.lang.ref.Reference$ReferenceHandler.run(Reference.Java:133)
```





#### jad

对类进行反编译:

```java
$ jad Javax.servlet.Servlet

ClassLoader:
+-Java.net.URLClassLoader@6108b2d7
  +-sun.misc.Launcher$AppClassLoader@18b4aac2
    +-sun.misc.Launcher$ExtClassLoader@1ddf84b8

Location:
/Users/xxx/work/test/lib/servlet-api.jar

/*
 * Decompiled with CFR 0_122.
 */
package Javax.servlet;

import Java.io.IOException;
import Javax.servlet.ServletConfig;
import Javax.servlet.ServletException;
import Javax.servlet.ServletRequest;
import Javax.servlet.ServletResponse;

public interface Servlet {
    public void init(ServletConfig var1) throws ServletException;

    public ServletConfig getServletConfig();

    public void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    public String getServletInfo();

    public void destroy();
}
```





#### mc

Memory Compiler/内存编译器，编译`.Java`文件生成`.Class`

```bash
mc /tmp/Test.Java
```





#### redefine

加载外部的`.Class`文件，redefine JVM已加载的类

```bash
redefine /tmp/Test.Class
redefine -c 327a647b /tmp/Test.Class /tmp/Test\$Inner.Class
```





#### sc

查找JVM中已经加载的类

```bash
$ sc -d org.springframework.web.context.support.XmlWebApplicationContext

 Class-info        org.springframework.web.context.support.XmlWebApplicationContext
 code-source       /Users/xxx/work/test/WEB-INF/lib/spring-web-3.2.11.RELEASE.jar
 name              org.springframework.web.context.support.XmlWebApplicationContext
 isInterface       false
 isAnnotation      false
 isEnum            false
 isAnonymousClass  false
 isArray           false
 isLocalClass      false
 isMemberClass     false
 isPrimitive       false
 isSynthetic       false
 simple-name       XmlWebApplicationContext
 modifier          public
 annotation
 interfaces
 super-Class       +-org.springframework.web.context.support.AbstractRefreshableWebApplicationContext
                     +-org.springframework.context.support.AbstractRefreshableConfigApplicationContext
                       +-org.springframework.context.support.AbstractRefreshableApplicationContext
                         +-org.springframework.context.support.AbstractApplicationContext
                           +-org.springframework.core.io.DefaultResourceLoader
                             +-Java.lang.Object
 Class-loader      +-org.apache.catalina.loader.ParallelWebappClassLoader
                     +-Java.net.URLClassLoader@6108b2d7
                       +-sun.misc.Launcher$AppClassLoader@18b4aac2
                         +-sun.misc.Launcher$ExtClassLoader@1ddf84b8
 ClassLoaderHash   25131501
```





#### stack

查看方法 `test.arthas.TestStack#doGet` 的调用堆栈：

```bash
$ stack test.arthas.TestStack doGet

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 286 ms.
ts=2018-09-18 10:11:45;thread_name=http-bio-8080-exec-10;id=d9;is_daemon=true;priority=5;TCCL=org.apache.catalina.loader.ParallelWebappClassLoader@25131501
    @test.arthas.TestStack.doGet()
        at Javax.servlet.http.HttpServlet.service(HttpServlet.Java:624)
        at Javax.servlet.http.HttpServlet.service(HttpServlet.Java:731)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:303)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:208)
        at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.Java:52)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:208)
        at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.Java:241)
        at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.Java:208)
        at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.Java:220)
        at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.Java:110)
        ...
        at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.Java:169)
        at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.Java:103)
        at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.Java:116)
        at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.Java:451)
        at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.Java:1121)
        at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.Java:637)
        at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.Java:316)
        at Java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.Java:1142)
        at Java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.Java:617)
        at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.Java:61)
        at Java.lang.Thread.run(Thread.Java:745)
```





#### Trace

观察方法执行的时候哪个子调用比较慢:

![trace](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809171429800-628724190.png)





#### Watch

观察方法 `test.arthas.TestWatch#doGet` 执行的入参，仅当方法抛出异常时才输出

```bash
$ watch test.arthas.TestWatch doGet {params[0], throwExp} -e

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 65 ms.
ts=2018-09-18 10:26:28;result=@ArrayList[
    @RequestFacade[org.apache.catalina.connector.RequestFacade@79f922b2],
    @NullPointerException[Java.lang.NullPointerException],
]
```





#### Monitor

监控某个特殊方法的调用统计数据，包括总调用次数，平均rt，成功率等信息，每隔5秒输出一次

```bash
$ monitor -c 5 org.apache.dubbo.demo.provider.DemoServiceImpl sayHello

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 109 ms.
 timestamp            Class                                           method    total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------------------------------------
 2018-09-20 09:45:32  org.apache.dubbo.demo.provider.DemoServiceImpl  sayHello  5      5        0     0.67        0.00%

 timestamp            Class                                           method    total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------------------------------------
 2018-09-20 09:45:37  org.apache.dubbo.demo.provider.DemoServiceImpl  sayHello  5      5        0     1.00        0.00%

 timestamp            Class                                           method    total  success  fail  avg-rt(ms)  fail-rate
----------------------------------------------------------------------------------------------------------------------------
 2018-09-20 09:45:42  org.apache.dubbo.demo.provider.DemoServiceImpl  sayHello  5      5        0     0.43        0.00%
```





#### Time Tunnel(tt)

记录方法调用信息，支持事后查看方法调用的参数，返回值，抛出的异常等信息，仿佛穿越时空隧道回到调用现场一般

```bash
$ tt -t org.apache.dubbo.demo.provider.DemoServiceImpl sayHello

Press Ctrl+C to abort.
Affect(Class-cnt:1 , method-cnt:1) cost in 75 ms.
 INDEX   TIMESTAMP            COST(ms)  IS-RET  IS-EXP   OBJECT         Class                          METHOD
-------------------------------------------------------------------------------------------------------------------------------------
 1000    2018-09-20 09:54:10  1.971195  true    false    0x55965cca     DemoServiceImpl                sayHello
 1001    2018-09-20 09:54:11  0.215685  true    false    0x55965cca     DemoServiceImpl                sayHello
 1002    2018-09-20 09:54:12  0.236303  true    false    0x55965cca     DemoServiceImpl                sayHello
 1003    2018-09-20 09:54:13  0.159598  true    false    0x55965cca     DemoServiceImpl                sayHello
 1004    2018-09-20 09:54:14  0.201982  true    false    0x55965cca     DemoServiceImpl                sayHello
 1005    2018-09-20 09:54:15  0.214205  true    false    0x55965cca     DemoServiceImpl                sayHello
 1006    2018-09-20 09:54:16  0.241863  true    false    0x55965cca     DemoServiceImpl                sayHello
 1007    2018-09-20 09:54:17  0.305747  true    false    0x55965cca     DemoServiceImpl                sayHello
 1008    2018-09-20 09:54:18  0.18468   true    false    0x55965cca     DemoServiceImpl                sayHello
```





#### Classloader

了解当前系统中有多少类加载器，以及每个加载器加载的类数量，帮助您判断是否有类加载器泄露

```bash
$ Classloader

 name                                                  numberOfInstances  loadedCountTotal
 BootstrapClassLoader                                  1                  3346
 com.taobao.arthas.agent.ArthasClassloader             1                  1262
 Java.net.URLClassLoader                               2                  1033
 org.apache.catalina.loader.ParallelWebappClassLoader  1                  628
 sun.reflect.DelegatingClassLoader                     166                166
 sun.misc.Launcher$AppClassLoader                      1                  31
 com.alibaba.fastjson.util.ASMClassLoader              6                  15
 sun.misc.Launcher$ExtClassLoader                      1                  7
 org.jvnet.hk2.internal.DelegatingClassLoader          2                  2
 sun.reflect.misc.MethodUtil                           1                  1
```





#### Web Console

- https://alibaba.github.io/arthas/web-console

![web console](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809172831166-1831339616.png)





### Arthas 命令集

#### 基础命令

- help——查看命令帮助信息
- [cat](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/cat.md)——打印文件内容，和linux里的cat命令类似
- [grep]](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/grep.md)——匹配查找，和linux里的grep命令类似
- [pwd](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/pwd.md)——返回当前的工作目录，和linux命令类似
- cls——清空当前屏幕区域
- session——查看当前会话的信息
- [reset](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/reset.md)——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类
- version——输出当前目标 Java 进程所加载的 Arthas 版本号
- history——打印命令历史
- quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响
- stop/shutdown——关闭 Arthas 服务端，所有 Arthas 客户端全部退出
- [keymap](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/keymap.md)——Arthas快捷键列表及自定义快捷键





#### JVM相关

- [dashboard](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/dashboard.md)——当前系统的实时数据面板
- [thread](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/thread.md)——查看当前 JVM 的线程堆栈信息
- [JVM](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/JVM.md)——查看当前 JVM 的信息
- [sysprop](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/sysprop.md)——查看和修改JVM的系统属性
- [sysenv](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/sysenv.md)——查看JVM的环境变量
- [vmoption](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/vmoption.md)——查看和修改JVM里诊断相关的option
- [logger](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/logger.md)——查看和修改logger
- [getstatic](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/getstatic.md)——查看类的静态属性
- [ognl](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/ognl.md)——执行ognl表达式
- [mbean](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/mbean.md)——查看 Mbean 的信息
- [heapdump](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/heapdump.md)——dump Java heap, 类似jmap命令的heap dump功能





#### Class/Classloader相关

- [sc](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/sc.md)——查看JVM已加载的类信息
- [sm](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/sm.md)——查看已加载类的方法信息
- [jad](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/jad.md)——反编译指定已加载类的源码
- [mc](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/mc.md)——内存编绎器，内存编绎`.Java`文件为`.Class`文件
- [redefine](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/redefine.md)——加载外部的`.Class`文件，redefine到JVM里
- [dump](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/dump.md)——dump 已加载类的 byte code 到特定目录
- [Classloader](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/Classloader.md)——查看Classloader的继承树，urls，类加载信息，使用Classloader去getResource





#### monitor/watch/trace相关

> 请注意，这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 `shutdown` 或将增强过的类执行 `reset` 命令

- [monitor](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/monitor.md)——方法执行监控
- [watch](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/watch.md)——方法执行数据观测
- [trace](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/trace.md)——方法内部调用路径，并输出方法路径上的每个节点上耗时
- [stack](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/stack.md)——输出当前方法被调用的调用路径
- [tt](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/tt.md)——方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测





#### options

- [options](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/options.md)——查看或设置Arthas全局开关



#### 管道

Arthas支持使用管道对上述命令的结果进行进一步的处理，如`sm Java.lang.String * | grep 'index'`

- grep——搜索满足条件的结果
- plaintext——将命令的结果去除ANSI颜色
- wc——按行统计输出结果



#### 后台异步任务

当线上出现偶发的问题，比如需要watch某个条件，而这个条件一天可能才会出现一次时，异步后台任务就派上用场了，详情请参考[这里](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/async.md)

- 使用 > 将结果重写向到日志文件，使用 & 指定命令是后台运行，session断开不影响任务执行(生命周期默认为1天)
- jobs——列出所有job
- kill——强制终止任务
- fg——将暂停的任务拉到前台执行
- bg——将暂停的任务放到后台执行



#### Web Console

通过websocket连接Arthas

- [Web Console](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/web-console.md)



#### 用户数据回报

在`3.1.4`版本后，增加了用户数据回报功能，方便统一做安全或者历史数据统计

在启动时，指定`stat-url`，就会回报执行的每一行命令，比如： `./as.sh --stat-url 'http://192.168.10.11:8080/api/stat'`

在tunnel server里有一个示例的回报代码，用户可以自己在服务器上实现

[StatController.Java](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/https://github.com/alibaba/arthas/blob/master/tunnel-server/src/main/Java/com/alibaba/arthas/tunnel/server/app/web/StatController.Java)



#### 其他特性

- [异步命令支持](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/async.md)
- [执行结果存日志](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/save-log.md)
- [批处理的支持](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/batch-support.md)
- [ognl表达式的用法说明](https://github.com/alibaba/arthas/blob/master/site/src/site/sphinx/https://github.com/alibaba/arthas/issues/11)





## Arthas场景实战

### 查看最繁忙的线程，以及是否有阻塞情况发生?

> 场景：我想看下查看最繁忙的线程，以及是否有阻塞情况发生? 常规查看线程，一般我们可以通过 top 等系统命令进行查看，但是那毕竟要很多个步骤，很麻烦

```bash
thread -n 3			# 查看最繁忙的三个线程栈信息
thread				# 以直观的方式展现所有的线程情况
thread -b			#找出当前阻塞其他线程的线程
```



### 确认某个类是否已被系统加载?

> 场景：我新写了一个类或者一个方法，我想知道新写的代码是否被部署了?

```bash
# 即可以找到需要的类全路径，如果存在的话
sc *MyServlet

# 查看这个某个类所有的方法
sm pdai.tech.servlet.TestMyServlet *

# 查看某个方法的信息，如果存在的话
sm pdai.tech.servlet.TestMyServlet testMethod  
```



### 如何查看一个Class类的源码信息?

> 场景：我新修改的内容在方法内部，而上一个步骤只能看到方法，这时候可以反编译看下源码

```bash
# 直接反编译出Java 源代码，包含一此额外信息的
jad pdai.tech.servlet.TestMyServlet
```



### 重要：如何跟踪某个方法的返回值、入参.... ?

> 场景：我想看下我新加的方法在线运行的参数和返回值?

```bash
# 同时监控入参，返回值，及异常
watch pdai.tech.servlet.TestMyServlet testMethod "{params, returnObj, throwExp}" -e -x 2 
```

具体看watch命令



### 如何看方法调用栈的信息?

> 场景：我想看下某个方法的调用栈的信息?

```bash
stack pdai.tech.servlet.TestMyServlet testMethod
```

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上



### 重要：找到最耗时的方法调用?

> 场景：testMethod这个方法入口响应很慢，如何找到最耗时的子调用?

```bash
# 执行的时候每个子调用的运行时长，可以找到最耗时的子调用
trace pdai.tech.servlet.TestMyServlet testMethod
```

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上，然后一层一层看子调用



### 重要：如何临时更改代码运行?

> 场景：我找到了问题所在，能否线上直接修改测试，而不需要在本地改了代码后，重新打包部署，然后重启观察效果?

```bash
# 先反编译出Class源码
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.Java  

# 然后使用外部工具编辑内容
mc /tmp/UserController.Java -d /tmp  # 再编译成Class

# 最后，重新载入定义的类，就可以实时验证你的猜测了
redefine /tmp/com/example/demo/arthas/user/UserController.Class
```

如上，是直接更改线上代码的方式，但是一般好像是编译不成功的所以，最好是本地ide编译成 Class文件后，再上传替换为好！

总之，已经完全不用重启和发布了！这个功能真的很方便，比起重启带来的代价，真的是不可比的比如，重启时可能导致负载重分配，选主等等问题，就不是你能控制的了



### 我如何测试某个方法的性能问题?

> 场景：我想看下某个方法的性能

```bash
monitor -c 5 demo.MathGame primeFactors
```



### 更多

请参考: [官方Issue墙](https://github.com/alibaba/arthas/issues?q=label%3Auser-case)



## Arthas源码

首先我们先放出一张整体宏观的模块调用图：

![trace](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173134665-475796932.jpg)

源码理解可以看移步这两篇文章:

- [什么是 Arthas](https://www.jianshu.com/p/70c1c55f12ef)
- [Arthas阅读](https://yq.aliyun.com/articles/704228)



## 参考资料

- https://www.cnblogs.com/muxuanchan/p/10097639.html
- https://www.cnblogs.com/yougewe/p/10770690.html
- https://help.aliyun.com/document_detail/112975.html











# 调试排错 - Java 问题排查之使用IDEA本地调试和远程调试

> Debug用来追踪代码的运行流程，通常在程序运行过程中出现异常，启用Debug模式可以分析定位异常发生的位置，以及在运行过程中参数的变化；并且在实际的排错过程中，还会用到Remote DebugIDEA 。相比 Eclipse/STS效率更高，本文主要介绍基于IDEA的Debug和Remote Debug的技巧





## Debug开篇

> 首先看下IDEA中Debug模式下的界面

如下是在IDEA中启动Debug模式，进入断点后的界面，我这里是Windows，可能和Mac的图标等会有些不一样。就简单说下图中标注的8个地方：

- ① 以Debug模式启动服务，左边的一个按钮则是以Run模式启动在开发中，我一般会直接启动Debug模式，方便随时调试代码
- ② 断点：在左边行号栏单击左键，或者快捷键Ctrl+F8 打上/取消断点，断点行的颜色可自己去设置
- ③ Debug窗口：访问请求到达第一个断点后，会自动激活Debug窗口如果没有自动激活，可以去设置里设置，如图1.2
- ④ 调试按钮：一共有8个按钮，调试的主要功能就对应着这几个按钮，鼠标悬停在按钮上可以查看对应的快捷键在菜单栏Run里可以找到同样的对应的功能，如图1.4
- ⑤ 服务按钮：可以在这里关闭/启动服务，设置断点等
- ⑥ 方法调用栈：这里显示了该线程调试所经过的所有方法，勾选右上角的[Show All Frames]按钮，就不会显示其它类库的方法了，否则这里会有一大堆的方法
- ⑦ Variables：在变量区可以查看当前断点之前的当前方法内的变量
- ⑧ Watches：查看变量，可以将Variables区中的变量拖到Watches中查看

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173303730-2093407242.png)

在设置里勾选Show debug window on breakpoint，则请求进入到断点后自动激活Debug窗口

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173313389-1435836917.png)

如果你的IDEA底部没有显示工具栏或状态栏，可以在View里打开，显示出工具栏会方便我们使用可以自己去尝试下这四个选项

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173322377-1653114003.png)

在菜单栏Run里有调试对应的功能，同时可以查看对应的快捷键

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173334928-717425915.png)





## 基本用法&快捷键

Debug调试的功能主要对应着图一中4和5两组按钮：

- **首先说第一组按钮，共8个按钮**，从左到右依次如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173354445-1603072420.png)

- `Show Execution Point` (Alt + F10)：如果你的光标在其它行或其它页面，点击这个按钮可跳转到当前代码执行的行
- `Step Over` (F8)：步过，一行一行地往下走，如果这一行上有方法不会进入方法
- `Step Into` (F7)：步入，如果当前行有方法，可以进入方法内部，一般用于进入自定义方法内，不会进入官方类库的方法，如第25行的put方法
- `Force Step Into` (Alt + Shift + F7)：强制步入，能进入任何方法，查看底层源码的时候可以用这个进入官方类库的方法
- `Step Out` (Shift + F8)：步出，从步入的方法内退出到方法调用处，此时方法已执行完毕，只是还没有完成赋值
- `Drop Frame` (默认无)：回退断点，后面章节详细说明
- `Run to Cursor` (Alt + F9)：运行到光标处，你可以将光标定位到你需要查看的那一行，然后使用这个功能，代码会运行至光标行，而不需要打断点
- `Evaluate Expression` (Alt + F8)：计算表达式，后面章节详细说明
- **第二组按钮，共7个按钮**，从上到下依次如下：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173405438-87011623.png)

- `Rerun 'xxxx'`：重新运行程序，会关闭服务后重新启动程序
- `Update 'tech' application` (Ctrl + F5)：更新程序，一般在你的代码有改动后可执行这个功能而这个功能对应的操作则是在服务配置里，如图2.3
- `Resume Program` (F9)：恢复程序，比如，你在第20行和25行有两个断点，当前运行至第20行，按F9，则运行到下一个断点(即第25行)，再按F9，则运行完整个流程，因为后面已经没有断点了
- `Pause Program`：暂停程序，启用Debug目前没发现具体用法
- `Stop 'xxx'` (Ctrl + F2)：连续按两下，关闭程序有时候你会发现关闭服务再启动时，报端口被占用，这是因为没完全关闭服务的原因，你就需要查杀所有JVM进程了
- `View Breakpoints` (Ctrl + Shift + F8)：查看所有断点，后面章节会涉及到
- `Mute Breakpoints`：哑的断点，选择这个后，所有断点变为灰色，断点失效，按F9则可以直接运行完程序再次点击，断点变为红色，有效如果只想使某一个断点失效，可以在断点上右键取消Enabled，则该行断点失效
- **更新程序**

`On 'Update' actions`，执行更新操作时所做的事情，一般选择`'Update Classes and resources'`，即更新类和资源文件

一般配合热部署插件会更好用，如JRebel，这样就不用每次更改代码后还要去重新启动服务如何激活JRebel，在最后章节附上

下面的`On frame deactivation`，在IDEA窗口失去焦点时触发，即一般你从idea切换到浏览器的时候，idea会自动帮你做的事情，一般可以设置Do nothing，频繁切换会比较消耗资源的

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173419123-963075349.png)

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173429086-1483203820.png)





## 变量查看

> 在Debug过程中，跟踪查看变量的变化是非常必要的，这里就简单说下IDEA中可以查看变量的几个地方，相信大部分人都了解

- 如下，在IDEA中，参数所在行后面会显示当前变量的值

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173446303-213207605.png)

- 光标悬停到参数上，显示当前变量信息。点击打开详情如下图。我一般会使用这种方式，快捷方便

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173457399-14499864.png)

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173507048-303329551.png)

- 在Variables里查看，这里显示当前方法里的所有变量

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173515735-1127566460.png)

- 在Watches里，点击New Watch，输入需要查看的变量或者可以从Variables里拖到Watche里查看

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173530516-1852883589.png)

如果你发现你没有Watches，可能在下图所在的地方

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173538593-1930395357.png)



![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173546803-1417112037.png)





## 计算表达式

> 在前面提到的计算表达式如下图的按钮，Evaluate Expression (Alt + F8) 可以使用这个操作在调试过程中计算某个表达式的值，而不用再去打印信息

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173604261-1846366142.png)

- 按Alt + F8或按钮，或者，你可以选中某个表达式再Alt + F8，弹出计算表达式的窗口，如下，回车或点击Evaluate计算表达式的值

这个表达式不仅可以是一般变量或参数，也可以是方法，当你的一行代码中调用了几个方法时，就可以通过这种方式查看查看某个方法的返回值

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173611747-1982780110.png)

- 设置变量，在计算表达式的框里，可以改变变量的值，这样有时候就能很方便我们去调试各种值的情况了不是

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173621337-1480989519.png)





## 智能步入

> 想想，一行代码里有好几个方法，怎么只选择某一个方法进入之前提到过使用Step Into (Alt + F7) 或者 Force Step Into (Alt + Shift + F7)进入到方法内部，但这两个操作会根据方法调用顺序依次进入，这比较麻烦

那么智能步入就很方便了，智能步入，这个功能在Run里可以看到，Smart Step Into (Shift + F7)，如下图

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173635855-1901977129.png)

按Shift + F7，会自动定位到当前断点行，并列出需要进入的方法，如图5.2，点击方法进入方法内部

如果只有一个方法，则直接进入，类似Force Step Into

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173644348-993417735.png)





## 断点条件设置

> 通过设置断点条件，在满足条件时，才停在断点处，否则直接运行

通常，当我们在遍历一个比较大的集合或数组时，在循环内设置了一个断点，难道我们要一个一个去看变量的值？那肯定很累，说不定你还错过这个值得重新来一次

- 在断点上右键直接**设置当前断点的条件**，如下图设置exist为true时断点才生效

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173700954-791864667.png)

- 点击View Breakpoints (Ctrl + Shift + F8)，查看所有断点

   

  - Java Line Breakpoints 显示了所有的断点，在右边勾选Condition，设置断点的条件
  - 勾选Log message to console，则会将当前断点行输出到控制台，如图6.3
  - 勾选Evaluate and log，可以在执行这行代码是计算表达式的值，并将结果输出到控制台

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173712506-1647606179.png)



![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173721011-2015807504.png)



- 再说说右边的Filters过滤

  ，这些一般情况下不常用，简单说下意思 

  - Instance filters：实例过滤，输入实例ID(如下图中的实例ID)，但是我这里没有成功，不知道什么原因，知道的朋友留个言
  - Class filters：类过滤，根据类名过滤，同样没有成功....
  - Pass count：用于循环中，如果断点在循环中，可以设置该值，循环多少次后停在断点处，之后的循环都会停在断点处

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173740248-680702849.png)



![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173749056-2119830575.png)

- **异常断点，通过设置异常断点，在程序中出现需要拦截的异常时，会自动定位到异常行**

如下图，点击+号添加Java Exception Breakpoints，添加异常断点然后输入需要断点的异常类

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173758207-1197682245.png)

之后可以在Java Exception Breakpoints里看到添加的异常断点

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173806125-659091380.png)

这里添加了一个NullPointerException异常断点，出现空指针异常后，自动定位在空指针异常行

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173814899-1808287885.png)





## 多线程调试

> 一般情况下我们调试的时候是在一个线程中的，一步一步往下走但有时候你会发现在Debug的时候，想发起另外一个请求都无法进行了？

那是因为IDEA在Debug时默认阻塞级别是ALL，会阻塞其它线程，只有在当前调试线程走完时才会走其它线程可以在View Breakpoints里选择Thread，如图7.1，然后点击Make Default设置为默认选项

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173832331-532829006.png)

切换线程，在下图中Frames的下拉列表里，可以切换当前的线程，如下我这里有两个Debug的线程，切换另外一个则进入另一个Debug的线程

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173841405-699903384.png)





## 回退断点

> 在调试的时候，想要重新走一下流程而不用再次发起一个请求？

- 首先认识下这个**方法调用栈**，如下图，首先请求进入DemoController的insertDemo方法，然后调用insert方法，其它的invoke我们且先不管，最上面的方法是当前断点所在的方法

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173858307-262284257.png)



- **断点回退**

所谓的断点回退，其实就是回退到上一个方法调用的开始处，在IDEA里测试无法一行一行地回退或回到到上一个断点处，而是回到上一个方法

回退的方式有两种，一种是Drop Frame按钮，按调用的方法逐步回退，包括三方类库的其它方法

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173908604-187138609.png)

取消Show All Frames按钮会显示三方类库的方法

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173916110-440104522.png)

第二种方式，在调用栈方法上选择要回退的方法，右键选择Drop Frame，回退到该方法的上一个方法调用处，此时再按F9(Resume Program)，可以看到程序进入到该方法的断点处了

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173924081-735826436.png)

但有一点需要注意，断点回退只能重新走一下流程，之前的某些参数/数据的状态已经改变了的是无法回退到之前的状态的，如对象、集合、更新了数据库数据等等





## 中断Debug

> 想要在Debug的时候，中断请求，不要再走剩余的流程了？

有些时候，我们看到传入的参数有误后，不想走后面的流程了，怎么中断这次请求呢(后面的流程要删除数据库数据呢....)，难道要关闭服务重新启动程序？嗯，我以前也是这么干的

确切的说，我也没发现可以直接中断请求的方式(除了关闭服务)，但可以通过Force Return，即强制返回来避免后续的流程，如图

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173942511-1046368739.png)

点击Force Return，弹出Return Value的窗口，我这个方法的返回类型为Map，所以，我这里直接返回 results，来强制返回，从而不再进行后续的流程或者你可以`new HashMap<>()`

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809173950313-741832490.png)





## 远程调试(Remote Debug)

> 有时候，本地调试的时候没有问题，打包部署到测试环境的时候却爆出一堆莫名其妙的问题，这时该怎么办呢？



### 使用特定JVM参数运行服务端代码

要让远程服务器运行的代码支持远程调试，则启动的时候必须加上特定的JVM参数，这些参数是：

```bash
-Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=${debug_port}
```

其中的`${debug_port}`是用户自定义的，为debug端口，本例以5555端口为例

本人在这里踩过一个坑，必须要说一下：在使用公司内部的自动化部署平台NDP进行应用部署时，该平台号称支持远程调试，只需要在某个配置页面配置一下调试端口号（没有填写任何IP相关的信息），并且重新发布一下应用即可。事实上也可以发现，上述JVM参数中唯一可变的就是${debug_port}。但是实际在本地连接时发现却始终连不上5555 的调试端口，仔细排查才发现，下面截取了NDP发布的应用所有JVM参数列表中与远程调试相关的JVM启动参数如下：

```bash
-Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=127.0.0.1:5555
```

将address设置为127.0.0.1:5555，表示将调试端口限制为本地访问，远程无法访问，这个应该是NDP平台的一个bug，我们在自己设置JVM的启动参数时也需要格外注意

如果只是临时调试，在端口号前面不要加上限制访问的IP地址，调试完成之后，将上述JVM参数去除掉之后重新发布下，防范开放远程调试端口可能带来的安全风险



### 本地连接远程服务器debug端口

打开IDEA，在顶部靠右的地方选择”Edit Configurations…”，进去之后点击+号，选择”Remote”，按照下图的只是填写红框内的内容，其中Name填写名称，这里为remote webserver，host为远程代码运行的机器的ip/hostname，port为上一步指定的debug_port，本例是5555。然后点击Apply，最后点击OK即可

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174020671-1179886966.png)

现在在上一步选择”Edit Configurations…”的下拉框的位置选择上一步创建的remote webserver，然后点击右边的debug按钮(长的像臭虫那个)，看控制台日志，如果出现类似“Connected to the target VM, address: ‘xx.xx.xx.xx:5555’, transport: ‘socket’”的字样，就表示连接成功过了，我这里实际显示的内容如下：

```bash
Connected to the target VM, address: '10.185.0.192:15555', transport: 'socket'
```





### 设置断点，开始调试

远程debug模式已经开启，现在可以在需要调试的代码中打断点了，比如：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174042496-1962119510.png)

如图中所示，如果断点内有√，则表示选取的断点正确

现在在本地发送一个到远程服务器的请求，看本地控制台的bug界面，划到debugger这个标签，可以看到当前远程服务的内部状态（各种变量）已经全部显示出来了，并且在刚才设置了断点的地方，也显示了该行的变量值

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174052110-1844461207.png)

> 备注：需要注意的是，用于远程debug的代码必须与远程部署的代码完全一致，不能发生任何的修改，否则打上的断点将无法命中，切记切记





## 参考文章

- 前面9个部分，主要总结自 https://www.cnblogs.com/diaobiyong/p/10682996.html
- 远程调试，主要整理自 https://www.jianshu.com/p/302dc10217c0
- 著作权归相关作者所有商业转载请联系作者获得授权，非商业转载请注明出处









# 调试排错 - Java动态调试技术原理

> 本文转载自 美团技术团队胡健的[Java 动态调试技术原理及实践](https://tech.meituan.com/2019/11/07/Java-dynamic-debugging-technology.html), 通过学习Java agent方式进行动态调试了解目前很多大厂开源的一些基于此的调试工具





##  简介

断点调试是我们最常使用的调试手段，它可以获取到方法执行过程中的变量信息，并可以观察到方法的执行路径但断点调试会在断点位置停顿，使得整个应用停止响应在线上停顿应用是致命的，动态调试技术给了我们创造新的调试模式的想象空间。本文将研究Java语言中的动态调试技术，首先概括Java动态调试所涉及的技术基础，接着介绍我们在Java动态调试领域的思考及实践，通过结合实际业务场景，设计并实现了一种具备动态性的断点调试工具Java-debug-tool，显著提高了故障排查效率

JVMTI (JVM Tool Interface)是Java虚拟机对外提供的Native编程接口，通过JVMTI，外部进程可以获取到运行时JVM的诸多信息，比如线程、GC等。Agent是一个运行在目标JVM的特定程序，它的职责是负责从目标JVM中获取数据，然后将数据传递给外部进程。加载Agent的时机可以是目标JVM启动之时，也可以是在目标JVM运行时进行加载，而在目标JVM运行时进行Agent加载具备动态性，对于时机未知的Debug场景来说非常实用。下面将详细分析Java Agent技术的实现细节



## Agent的实现模式

JVMTI是一套Native接口，在Java SE 5之前，要实现一个Agent只能通过编写Native代码来实现。从Java SE 5开始，可以使用Java的Instrumentation接口(Java.lang.instrument)来编写Agent。无论是通过Native的方式还是通过Java Instrumentation接口的方式来编写Agent，它们的工作都是借助JVMTI来进行完成，下面介绍通过Java Instrumentation接口编写Agent的方法



### 通过Java Instrumentation API

- 实现Agent启动方法

Java Agent支持目标JVM启动时加载，也支持在目标JVM运行时加载，这两种不同的加载模式会使用不同的入口函数，如果需要在目标JVM启动的同时加载Agent，那么可以选择实现下面的方法：

```java
[1] public static void premain(String agentArgs, Instrumentation inst);
[2] public static void premain(String agentArgs);
```

JVM将首先寻找[1]，如果没有发现[1]，再寻找[2]如果希望在目标JVM运行时加载Agent，则需要实现下面的方法：

```java
[1] public static void agentmain(String agentArgs, Instrumentation inst);
[2] public static void agentmain(String agentArgs);
```

这两组方法的第一个参数AgentArgs是随同 “– Javaagent”一起传入的程序参数，如果这个字符串代表了多个参数，就需要自己解析这些参数inst是Instrumentation类型的对象，是JVM自动传入的，我们可以拿这个参数进行类增强等操作

- 指定Main-Class

Agent需要打包成一个jar包，在ManiFest属性中指定“Premain-Class”或者“Agent-Class”：

```java
Premain-Class: Class
Agent-Class: Class
```

- 挂载到目标JVM

将编写的Agent打成jar包后，就可以挂载到目标JVM上去了如果选择在目标JVM启动时加载Agent，则可以使用 “-Javaagent:[=]“，具体的使用方法可以使用“Java -Help”来查看。如果想要在运行时挂载Agent到目标JVM，就需要做一些额外的开发了

com.sun.tools.attach.VirtualMachine 这个类代表一个JVM抽象，可以通过这个类找到目标JVM，并且将Agent挂载到目标JVM上。下面是使用com.sun.tools.attach.VirtualMachine进行动态挂载Agent的一般实现：

```java
    private void attachAgentToTargetJVM() throws Exception {
        List<VirtualMachineDescriptor> virtualMachineDescriptors = VirtualMachine.list();
        VirtualMachineDescriptor targetVM = null;
        for (VirtualMachineDescriptor descriptor : virtualMachineDescriptors) {
            // 通过指定的进程ID找到目标JVM
            if (descriptor.id().equals(configure.getPid())) {
                targetVM = descriptor;
                break;
            }
        }
        if (targetVM == null) {
            throw new IllegalArgumentException("could not find the target JVM by process id:" + configure.getPid());
        }
        VirtualMachine virtualMachine = null;
        try {
            // 通过Attach挂载到目标JVM上
            virtualMachine = VirtualMachine.attach(targetVM);
            virtualMachine.loadAgent("{agent}", "{params}");
        } catch (Exception e) {
            if (virtualMachine != null) {
                // 将Agent从目标JVM卸载
                virtualMachine.detach();
            }
        }
    }
```

首先通过指定的进程ID找到目标JVM，然后通过Attach挂载到目标JVM上，执行加载Agent操作。VirtualMachine的Attach方法就是用来将Agent挂载到目标JVM上去的，而Detach则是将Agent从目标JVM卸载。关于Agent是如何挂载到目标JVM上的具体技术细节，将在下文中进行分析



## 启动时加载Agent

### 参数解析

创建JVM时，JVM会进行参数解析，即解析那些用来配置JVM启动的参数，比如堆大小、GC等；本文主要关注解析的参数为-agentlib、 -agentpath、 -Javaagent，这几个参数用来指定Agent，JVM会根据这几个参数加载Agent。下面来分析一下JVM是如何解析这几个参数的

```java
  // -agentlib and -agentpath
  if (match_option(option, "-agentlib:", &tail) ||
          (is_absolute_path = match_option(option, "-agentpath:", &tail))) {
      if(tail != NULL) {
        const char* pos = strchr(tail, '=');
        size_t len = (pos == NULL) ? strlen(tail) : pos - tail;
        char* name = strncpy(NEW_C_HEAP_ARRAY(char, len + 1, mtArguments), tail, len);
        name[len] = '\0';
        char *options = NULL;
        if(pos != NULL) {
          options = os::strdup_check_oom(pos + 1, mtArguments);
        }
#if !INCLUDE_JVMTI
        if (valid_jdwp_agent(name, is_absolute_path)) {
          jio_fprintf(defaultStream::error_stream(),
            "Debugging agents are not supported in this VM\n");
          return JNI_ERR;
        }
#endif // !INCLUDE_JVMTI
        add_init_agent(name, options, is_absolute_path);
      }
    // -Javaagent
    } else if (match_option(option, "-Javaagent:", &tail)) {
#if !INCLUDE_JVMTI
      jio_fprintf(defaultStream::error_stream(),
        "Instrumentation agents are not supported in this VM\n");
      return JNI_ERR;
#else
      if (tail != NULL) {
        size_t length = strlen(tail) + 1;
        char *options = NEW_C_HEAP_ARRAY(char, length, mtArguments);
        jio_snprintf(options, length, "%s", tail);
        add_init_agent("instrument", options, false);
        // Java agents need module Java.instrument
        if (!create_numbered_property("JDK.module.addmods", "Java.instrument", addmods_count++)) {
          return JNI_ENOMEM;
        }
      }
#endif // !INCLUDE_JVMTI
    }
```

上面的代码片段截取自hotspot/src/share/vm/runtime/arguments.cpp中的 Arguments::parse_each_vm_init_arg(const JavaVMInitArgs* args, bool* patch_mod_Javabase, Flag::Flags origin) 函数，该函数用来解析一个具体的JVM参数。这段代码的主要功能是解析出需要加载的Agent路径，然后调用add_init_agent函数进行解析结果的存储。下面先看一下add_init_agent函数的具体实现：

```java
  // -agentlib and -agentpath arguments
  static AgentLibraryList _agentList;
  static void add_init_agent(const char* name, char* options, bool absolute_path)
    { _agentList.add(new AgentLibrary(name, options, absolute_path, NULL)); }
```

AgentLibraryList是一个简单的链表结构，add_init_agent函数将解析好的、需要加载的Agent添加到这个链表中，等待后续的处理

这里需要注意，解析-Javaagent参数有一些特别之处，这个参数用来指定一个我们通过Java Instrumentation API来编写的Agent，Java Instrumentation API底层依赖的是JVMTI，对-JavaAgent的处理也说明了这一点，在调用add_init_agent函数时第一个参数是“instrument”，关于加载Agent这个问题在下一小节进行展开。到此，我们知道在启动JVM时指定的Agent已经被JVM解析完存放在了一个链表结构中。下面来分析一下JVM是如何加载这些Agent的



### 执行加载操作

在创建JVM进程的函数中，解析完JVM参数之后，下面的这段代码和加载Agent相关：

```java
  // Launch -agentlib/-agentpath and converted -Xrun agents
  if (Arguments::init_agents_at_startup()) {
    create_vm_init_agents();
  }
  static bool init_agents_at_startup() {
    return !_agentList.is_empty(); 
  }
```

当JVM判断出上一小节中解析出来的Agent不为空的时候，就要去调用函数create_vm_init_agents来加载Agent，下面来分析一下create_vm_init_agents函数是如何加载Agent的

```java
void Threads::create_vm_init_agents() {
  AgentLibrary* agent;
  for (agent = Arguments::agents(); agent != NULL; agent = agent->next()) {
    OnLoadEntry_t  on_load_entry = lookup_agent_on_load(agent);
    if (on_load_entry != NULL) {
      // Invoke the Agent_OnLoad function
      jint err = (*on_load_entry)(&main_vm, agent->options(), NULL);
    }
  }
}
```

create_vm_init_agents这个函数通过遍历Agent链表来逐个加载Agent。通过这段代码可以看出，首先通过lookup_agent_on_load来加载Agent并且找到Agent_OnLoad函数，这个函数是Agent的入口函数。如果没找到这个函数，则认为是加载了一个不合法的Agent，则什么也不做，否则调用这个函数，这样Agent的代码就开始执行起来了。对于使用Java Instrumentation API来编写Agent的方式来说，在解析阶段观察到在add_init_agent函数里面传递进去的是一个叫做”instrument”的字符串，其实这是一个动态链接库。在Linux里面，这个库叫做libinstrument.so，在BSD系统中叫做libinstrument.dylib，该动态链接库在{Java_HOME}/jre/lib/目录下



### instrument动态链接库

libinstrument用来支持使用Java Instrumentation API来编写Agent，在libinstrument中有一个非常重要的类称为：JPLISAgent(Java Programming Language Instrumentation Services Agent)，它的作用是初始化所有通过Java Instrumentation API编写的Agent，并且也承担着通过JVMTI实现Java Instrumentation中暴露API的责任

我们已经知道，在JVM启动的时候，JVM会通过-Javaagent参数加载Agent。最开始加载的是libinstrument动态链接库，然后在动态链接库里面找到JVMTI的入口方法：Agent_OnLoad。下面就来分析一下在libinstrument动态链接库中，Agent_OnLoad函数是怎么实现的

```java
JNIEXPORT jint JNICALL
DEF_Agent_OnLoad(JavaVM *vm, char *tail, void * reserved) {
    initerror = createNewJPLISAgent(vm, &agent);
    if ( initerror == JPLIS_INIT_ERROR_NONE ) {
        if (parseArgumentTail(tail, &jarfile, &options) != 0) {
            fprintf(stderr, "-Javaagent: memory allocation failure.\n");
            return JNI_ERR;
        }
        attributes = readAttributes(jarfile);
        premainClass = getAttribute(attributes, "Premain-Class");
        /* Save the jarfile name */
        agent->mJarfile = jarfile;
        /*
         * Convert JAR attributes into agent capabilities
         */
        convertCapabilityAttributes(attributes, agent);
        /*
         * Track (record) the agent Class name and options data
         */
        initerror = recordCommandLineData(agent, premainClass, options);
    }
    return result;
}
```

上述代码片段是经过精简的libinstrument中Agent_OnLoad实现，大概的流程就是：先创建一个JPLISAgent，然后将ManiFest中设定的一些参数解析出来， 比如(Premain-Class)等。创建了JPLISAgent之后，调用initializeJPLISAgent对这个Agent进行初始化操作。跟进initializeJPLISAgent看一下是如何初始化的：

```java
JPLISInitializationError initializeJPLISAgent(JPLISAgent *agent, JavaVM *vm, JVMtiEnv *JVMtienv) {
    /* check what capabilities are available */
    checkCapabilities(agent);
    /* check phase - if live phase then we don't need the VMInit event */
    JVMtierror = (*JVMtienv)->GetPhase(JVMtienv, &phase);
    /* now turn on the VMInit event */
    if ( JVMtierror == JVMTI_ERROR_NONE ) {
        JVMtiEventCallbacks callbacks;
        memset(&callbacks, 0, sizeof(callbacks));
        callbacks.VMInit = &eventHandlerVMInit;
        JVMtierror = (*JVMtienv)->SetEventCallbacks(JVMtienv,&callbacks,sizeof(callbacks));
    }
    if ( JVMtierror == JVMTI_ERROR_NONE ) {
        JVMtierror = (*JVMtienv)->SetEventNotificationMode(JVMtienv,JVMTI_ENABLE,JVMTI_EVENT_VM_INIT,NULL);
    }
    return (JVMtierror == JVMTI_ERROR_NONE)? JPLIS_INIT_ERROR_NONE : JPLIS_INIT_ERROR_FAILURE;
}
```

这里，我们关注callbacks.VMInit = &eventHandlerVMInit;这行代码，这里设置了一个VMInit事件的回调函数，表示在JVM初始化的时候会回调eventHandlerVMInit函数。下面来看一下这个函数的实现细节，猜测就是在这里调用了Premain方法：

```java
void JNICALL  eventHandlerVMInit( JVMtiEnv *JVMtienv,JNIEnv *jnienv,jthread thread) {
   // ...
   success = processJavaStart( environment->mAgent, jnienv);
  // ...
}
jboolean  processJavaStart(JPLISAgent *agent,JNIEnv *jnienv) {
    result = createInstrumentationImpl(jnienv, agent);
    /*
     *  Load the Java agent, and call the premain.
     */
    if ( result ) {
        result = startJavaAgent(agent, jnienv, agent->mAgentClassName, agent->mOptionsString, agent->mPremainCaller);
    }
    return result;
}
jboolean startJavaAgent( JPLISAgent *agent,JNIEnv *jnienv,const char *Classname,const char *optionsString,jmethodID agentMainMethod) {
  // ...  
  invokeJavaAgentMainMethod(jnienv,agent->mInstrumentationImpl,agentMainMethod, ClassNameObject,optionsStringObject);
  // ...
}
```

看到这里，Instrument已经实例化，invokeJavaAgentMainMethod这个方法将我们的premain方法执行起来了。接着，我们就可以根据Instrument实例来做我们想要做的事情了





## 运行时加载Agent

比起JVM启动时加载Agent，运行时加载Agent就比较有诱惑力了，因为运行时加载Agent的能力给我们提供了很强的动态性，我们可以在需要的时候加载Agent来进行一些工作。因为是动态的，我们可以按照需求来加载所需要的Agent，下面来分析一下动态加载Agent的相关技术细节



### AttachListener

Attach机制通过Attach Listener线程来进行相关事务的处理，下面来看一下Attach Listener线程是如何初始化的

```java
// Starts the Attach Listener thread
void AttachListener::init() {
  // 创建线程相关部分代码被去掉了
  const char thread_name[] = "Attach Listener";
  Handle string = Java_lang_String::create_from_str(thread_name, THREAD);
  { MutexLocker mu(Threads_lock);
    JavaThread* listener_thread = new JavaThread(&attach_listener_thread_entry);
    // ...
  }
}
```

我们知道，一个线程启动之后都需要指定一个入口来执行代码，Attach Listener线程的入口是attach_listener_thread_entry，下面看一下这个函数的具体实现：

```java
static void attach_listener_thread_entry(JavaThread* thread, TRAPS) {
  AttachListener::set_initialized();
  for (;;) {
      AttachOperation* op = AttachListener::dequeue();
      // find the function to dispatch too
      AttachOperationFunctionInfo* info = NULL;
      for (int i=0; funcs[i].name != NULL; i++) {
        const char* name = funcs[i].name;
        if (strcmp(op->name(), name) == 0) {
          info = &(funcs[i]); break;
        }}
       // dispatch to the function that implements this operation
        res = (info->func)(op, &st);
      //...
    }
}
```

整个函数执行逻辑，大概是这样的：

- 拉取一个需要执行的任务：AttachListener::dequeue
- 查询匹配的命令处理函数
- 执行匹配到的命令执行函数

其中第二步里面存在一个命令函数表，整个表如下：

```java
static AttachOperationFunctionInfo funcs[] = {
  { "agentProperties",  get_agent_properties },
  { "datadump",         data_dump },
  { "dumpheap",         dump_heap },
  { "load",             load_agent },
  { "properties",       get_system_properties },
  { "threaddump",       thread_dump },
  { "inspectheap",      heap_inspection },
  { "setflag",          set_flag },
  { "printflag",        print_flag },
  { "jcmd",             jcmd },
  { NULL,               NULL }
};
```

对于加载Agent来说，命令就是“load”现在，我们知道了Attach Listener大概的工作模式，但是还是不太清楚任务从哪来，这个秘密就藏在AttachListener::dequeue这行代码里面，接下来我们来分析一下dequeue这个函数：

```java
LinuxAttachOperation* LinuxAttachListener::dequeue() {
  for (;;) {
    // wait for client to connect
    struct sockaddr addr;
    socklen_t len = sizeof(addr);
    RESTARTABLE(::accept(listener(), &addr, &len), s);
    // get the credentials of the peer and check the effective uid/guid
    // - check with jeff on this.
    struct ucred cred_info;
    socklen_t optlen = sizeof(cred_info);
    if (::getsockopt(s, SOL_SOCKET, SO_PEERCRED, (void*)&cred_info, &optlen) == -1) {
      ::close(s);
      continue;
    }
    // peer credential look okay so we read the request
    LinuxAttachOperation* op = read_request(s);
    return op;
  }
}
```

这是Linux上的实现，不同的操作系统实现方式不太一样。上面的代码表面，Attach Listener在某个端口监听着，通过accept来接收一个连接，然后从这个连接里面将请求读取出来，然后将请求包装成一个AttachOperation类型的对象，之后就会从表里查询对应的处理函数，然后进行处理

Attach Listener使用一种被称为“懒加载”的策略进行初始化，也就是说，JVM启动的时候Attach Listener并不一定会启动起来。下面我们来分析一下这种“懒加载”策略的具体实现方案

```java
  // Start Attach Listener if +StartAttachListener or it can't be started lazily
  if (!DisableAttachMechanism) {
    AttachListener::vm_start();
    if (StartAttachListener || AttachListener::init_at_startup()) {
      AttachListener::init();
    }
  }
// Attach Listener is started lazily except in the case when
// +ReduseSignalUsage is used
bool AttachListener::init_at_startup() {
  if (ReduceSignalUsage) {
    return true;
  } else {
    return false;
  }
}
```

上面的代码截取自create_vm函数，DisableAttachMechanism、StartAttachListener和ReduceSignalUsage这三个变量默认都是false，所以AttachListener::init();这行代码不会在create_vm的时候执行，而vm_start会执行下面来看一下这个函数的实现细节：

```java
void AttachListener::vm_start() {
  char fn[UNIX_PATH_MAX];
  struct stat64 st;
  int ret;
  int n = snprintf(fn, UNIX_PATH_MAX, "%s/.Java_pid%d",
           os::get_temp_directory(), os::current_process_id());
  assert(n < (int)UNIX_PATH_MAX, "Java_pid file name buffer overflow");
  RESTARTABLE(::stat64(fn, &st), ret);
  if (ret == 0) {
    ret = ::unlink(fn);
    if (ret == -1) {
      log_debug(attach)("Failed to remove stale attach pid file at %s", fn);
    }
  }
}
```

这是在Linux上的实现，是将/tmp/目录下的.Java_pid{pid}文件删除，后面在创建Attach Listener线程的时候会创建出来这个文件。上面说到，AttachListener::init()这行代码不会在create_vm的时候执行，这行代码的实现已经在上文中分析了，就是创建Attach Listener线程，并监听其他JVM的命令请求。现在来分析一下这行代码是什么时候被调用的，也就是“懒加载”到底是怎么加载起来的

```java
  // Signal Dispatcher needs to be started before VMInit event is posted
  os::signal_init();
```

这是create_vm中的一段代码，看起来跟信号相关，其实Attach机制就是使用信号来实现“懒加载“的。下面我们来仔细地分析一下这个过程

```java
void os::signal_init() {
  if (!ReduceSignalUsage) {
    // Setup JavaThread for processing signals
    EXCEPTION_MARK;
    Klass* k = SystemDictionary::resolve_or_fail(vmSymbols::Java_lang_Thread(), true, CHECK);
    instanceKlassHandle klass (THREAD, k);
    instanceHandle thread_oop = klass->allocate_instance_handle(CHECK);
    const char thread_name[] = "Signal Dispatcher";
    Handle string = Java_lang_String::create_from_str(thread_name, CHECK);
    // Initialize thread_oop to put it into the system threadGroup
    Handle thread_group (THREAD, Universe::system_thread_group());
    JavaValue result(T_VOID);
    JavaCalls::call_special(&result, thread_oop,klass,vmSymbols::object_initializer_name(),vmSymbols::threadgroup_string_void_signature(),
                           thread_group,string,CHECK);
    KlassHandle group(THREAD, SystemDictionary::ThreadGroup_klass());
    JavaCalls::call_special(&result,thread_group,group,vmSymbols::add_method_name(),vmSymbols::thread_void_signature(),thread_oop,CHECK);
    os::signal_init_pd();
    { MutexLocker mu(Threads_lock);
      JavaThread* signal_thread = new JavaThread(&signal_thread_entry);
     // ...
    }
    // Handle ^BREAK
    os::signal(SIGBREAK, os::user_handler());
  }
}
```

JVM创建了一个新的进程来实现信号处理，这个线程叫“Signal Dispatcher”，一个线程创建之后需要有一个入口，“Signal Dispatcher”的入口是signal_thread_entry：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174335168-632289388.png)

这段代码截取自signal_thread_entry函数，截取中的内容是和Attach机制信号处理相关的代码。这段代码的意思是，当接收到“SIGBREAK”信号，就执行接下来的代码，这个信号是需要Attach到JVM上的信号发出来，这个后面会再分析。我们先来看一句关键的代码：AttachListener::is_init_trigger()：

```java
bool AttachListener::is_init_trigger() {
  if (init_at_startup() || is_initialized()) {
    return false;               // initialized at startup or already initialized
  }
  char fn[PATH_MAX+1];
  sprintf(fn, ".attach_pid%d", os::current_process_id());
  int ret;
  struct stat64 st;
  RESTARTABLE(::stat64(fn, &st), ret);
  if (ret == -1) {
    log_trace(attach)("Failed to find attach file: %s, trying alternate", fn);
    snprintf(fn, sizeof(fn), "%s/.attach_pid%d", os::get_temp_directory(), os::current_process_id());
    RESTARTABLE(::stat64(fn, &st), ret);
  }
  if (ret == 0) {
    // simple check to avoid starting the attach mechanism when
    // a bogus user creates the file
    if (st.st_uid == geteuid()) {
      init();
      return true;
    }
  }
  return false;
}
```

首先检查了一下是否在JVM启动时启动了Attach Listener，或者是否已经启动过。如果没有，才继续执行，在/tmp目录下创建一个叫做.attach_pid%d的文件，然后执行AttachListener的init函数，这个函数就是用来创建Attach Listener线程的函数，上面已经提到多次并进行了分析。到此，我们知道Attach机制的奥秘所在，也就是Attach Listener线程的创建依靠Signal Dispatcher线程，Signal Dispatcher是用来处理信号的线程，当Signal Dispatcher线程接收到“SIGBREAK”信号之后，就会执行初始化Attach Listener的工作



### 运行时加载Agent的实现

我们继续分析，到底是如何将一个Agent挂载到运行着的目标JVM上，在上文中提到了一段代码，用来进行运行时挂载Agent，可以参考上文中展示的关于“attachAgentToTargetJVM”方法的代码。这个方法里面的关键是调用VirtualMachine的attach方法进行Agent挂载的功能。下面我们就来分析一下VirtualMachine的attach方法具体是怎么实现的

```java
public static VirtualMachine attach(String var0) throws AttachNotSupportedException, IOException {
    if (var0 == null) {
        throw new NullPointerException("id cannot be null");
    } else {
        List var1 = AttachProvider.providers();
        if (var1.size() == 0) {
            throw new AttachNotSupportedException("no providers installed");
        } else {
            AttachNotSupportedException var2 = null;
            Iterator var3 = var1.iterator();
            while(var3.hasNext()) {
                AttachProvider var4 = (AttachProvider)var3.next();
                try {
                    return var4.attachVirtualMachine(var0);
                } catch (AttachNotSupportedException var6) {
                    var2 = var6;
                }
            }
            throw var2;
        }
    }
}
```

这个方法通过attachVirtualMachine方法进行attach操作，在MacOS系统中，AttachProvider的实现类是BsdAttachProvider。我们来看一下BsdAttachProvider的attachVirtualMachine方法是如何实现的：

```java
public VirtualMachine attachVirtualMachine(String var1) throws AttachNotSupportedException, IOException {
    this.checkAttachPermission();
    this.testAttachable(var1);
    return new BsdVirtualMachine(this, var1);
}
BsdVirtualMachine(AttachProvider var1, String var2) throws AttachNotSupportedException, IOException {
    int var3 = Integer.parseInt(var2);
    this.path = this.findSocketFile(var3);
    if (this.path == null) {
        File var4 = new File(tmpdir, ".attach_pid" + var3);
        createAttachFile(var4.getPath());
        try {
            sendQuitTo(var3);
            int var5 = 0;
            long var6 = 200L;
            int var8 = (int)(this.attachTimeout() / var6);
            do {
                try {
                    Thread.sleep(var6);
                } catch (InterruptedException var21) {
                    ;
                }
                this.path = this.findSocketFile(var3);
                ++var5;
            } while(var5 <= var8 && this.path == null);
        } finally {
            var4.delete();
        }
    }
    int var24 = socket();
    connect(var24, this.path);
}
private String findSocketFile(int var1) {
    String var2 = ".Java_pid" + var1;
    File var3 = new File(tmpdir, var2);
    return var3.exists() ? var3.getPath() : null;
}
```

findSocketFile方法用来查询目标JVM上是否已经启动了Attach Listener，它通过检查”tmp/“目录下是否存在Java_pid{pid}来进行实现。如果已经存在了，则说明Attach机制已经准备就绪，可以接受客户端的命令了，这个时候客户端就可以通过connect连接到目标JVM进行命令的发送，比如可以发送“load”命令来加载Agent；如果Java_pid{pid}文件还不存在，则需要通过sendQuitTo方法向目标JVM发送一个“SIGBREAK”信号，让它初始化Attach Listener线程并准备接受客户端连接。可以看到，发送了信号之后客户端会循环等待Java_pid{pid}这个文件，之后再通过connect连接到目标JVM上



### load命令的实现

下面来分析一下，“load”命令在JVM层面的实现：

```java
static jint load_agent(AttachOperation* op, outputStream* out) {
  // get agent name and options
  const char* agent = op->arg(0);
  const char* absParam = op->arg(1);
  const char* options = op->arg(2);
  // If loading a Java agent then need to ensure that the Java.instrument module is loaded
  if (strcmp(agent, "instrument") == 0) {
    Thread* THREAD = Thread::current();
    ResourceMark rm(THREAD);
    HandleMark hm(THREAD);
    JavaValue result(T_OBJECT);
    Handle h_module_name = Java_lang_String::create_from_str("Java.instrument", THREAD);
    JavaCalls::call_static(&result,SystemDictionary::module_Modules_klass(),vmSymbols::loadModule_name(),
                           vmSymbols::loadModule_signature(),h_module_name,THREAD);
  }
  return JVMtiExport::load_agent_library(agent, absParam, options, out);
}
```

这个函数先确保加载了Java.instrument模块，之后真正执行Agent加载的函数是 load_agent_library ,这个函数的套路就是加载Agent动态链接库，如果是通过Java instrument API实现的Agent，则加载的是libinstrument动态链接库，然后通过libinstrument里面的代码实现运行agentmain方法的逻辑，这一部分内容和libinstrument实现premain方法运行的逻辑其实差不多，这里不再做分析至此，我们对Java Agent技术已经有了一个全面而细致的了解



## 动态字节码修改的限制

上文中已经详细分析了Agent技术的实现，我们使用Java Instrumentation API来完成动态类修改的功能，在Instrumentation接口中，通过addTransformer方法来增加一个类转换器，类转换器由类ClassFileTransformer接口实现ClassFileTransformer接口中唯一的方法transform用于实现类转换，当类被加载的时候，就会调用transform方法，进行类转换在运行时，我们可以通过Instrumentation的redefineClasses方法进行类重定义，在方法上有一段注释需要特别注意：

```java
     * The redefinition may change method bodies, the constant pool and attributes.
     * The redefinition must not add, remove or rename fields or methods, change the
     * signatures of methods, or change inheritance.  These restrictions maybe be
     * lifted in future versions.  The Class file bytes are not checked, verified and installed
     * until after the transformations have been applied, if the resultant bytes are in
     * error this method will throw an exception.
```

这里面提到，我们不可以增加、删除或者重命名字段和方法，改变方法的签名或者类的继承关系。认识到这一点很重要，当我们通过ASM获取到增强的字节码之后，如果增强后的字节码没有遵守这些规则，那么调用redefineClasses方法来进行类的重定义就会失败。那redefineClasses方法具体是怎么实现类的重定义的呢? 它对运行时的JVM会造成什么样的影响呢? 下面来分析redefineClasses的实现细节





## 重定义类字节码的实现细节

上文中我们提到，libinstrument动态链接库中，JPLISAgent不仅实现了Agent入口代码执行的路由，而且还是Java代码与JVMTI之间的一道桥梁。我们在Java代码中调用Java Instrumentation API的redefineClasses，其实会调用libinstrument中的相关代码，我们来分析一下这条路径

```java
public void redefineClasses(ClassDefinition... var1) throws ClassNotFoundException {
    if (!this.isRedefineClassesSupported()) {
        throw new UnsupportedOperationException("redefineClasses is not supported in this environment");
    } else if (var1 == null) {
        throw new NullPointerException("null passed as 'definitions' in redefineClasses");
    } else {
        for(int var2 = 0; var2 < var1.length; ++var2) {
            if (var1[var2] == null) {
                throw new NullPointerException("element of 'definitions' is null in redefineClasses");
            }
        }
        if (var1.length != 0) {
            this.redefineClasses0(this.mNativeAgent, var1);
        }
    }
}
private native void redefineClasses0(long var1, ClassDefinition[] var3) throws ClassNotFoundException;
```

这是InstrumentationImpl中的redefineClasses实现，该方法的具体实现依赖一个Native方法redefineClasses()，我们可以在libinstrument中找到这个Native方法的实现：

```java
JNIEXPORT void JNICALL Java_sun_instrument_InstrumentationImpl_redefineClasses0
  (JNIEnv * jnienv, jobject implThis, jlong agent, jobjectArray ClassDefinitions) {
    redefineClasses(jnienv, (JPLISAgent*)(intptr_t)agent, ClassDefinitions);
}
```

redefineClasses这个函数的实现比较复杂，代码很长下面是一段关键的代码片段：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174516998-9038008.png)

可以看到，其实是调用了JVMTI的RetransformClasses函数来完成类的重定义细节

```java
// Class_count - pre-checked to be greater than or equal to 0
// Class_definitions - pre-checked for NULL
JVMtiError JVMtiEnv::RedefineClasses(jint Class_count, const JVMtiClassDefinition* Class_definitions) {
//TODO: add locking
  VM_RedefineClasses op(Class_count, Class_definitions, JVMti_Class_load_kind_redefine);
  VMThread::execute(&op);
  return (op.check_error());
} /* end RedefineClasses */
```

重定义类的请求会被JVM包装成一个VM_RedefineClasses类型的VM_Operation，VM_Operation是JVM内部的一些操作的基类，包括GC操作等。VM_Operation由VMThread来执行，新的VM_Operation操作会被添加到VMThread的运行队列中去，VMThread会不断从队列里面拉取VM_Operation并调用其doit等函数执行具体的操作。VM_RedefineClasses函数的流程较为复杂，下面是VM_RedefineClasses的大致流程：

- 加载新的字节码，合并常量池，并且对新的字节码进行校验工作

```java
  // Load the caller's new Class definition(s) into _scratch_Classes.
  // Constant pool merging work is done here as needed. Also calls
  // compare_and_normalize_Class_versions() to verify the Class
  // definition(s).
  JVMtiError load_new_Class_versions(TRAPS);
```

- 清除方法上的断点

```java
  // Remove all breakpoints in methods of this Class
  JVMtiBreakpoints& JVMti_breakpoints = JVMtiCurrentBreakpoints::get_JVMti_breakpoints();
  JVMti_breakpoints.clearall_in_Class_at_safepoint(the_Class());
```

- JIT逆优化

```java
  // Deoptimize all compiled code that depends on this Class
  flush_dependent_code(the_Class, THREAD);
```

- 进行字节码替换工作，需要进行更新类itable/vtable等操作
- 进行类重定义通知

```java
  SystemDictionary::notice_modification();
```

VM_RedefineClasses实现比较复杂的，详细实现可以参考 [RedefineClasses](https://github.com/pandening/openJDK/blob/0301fc792ffd3c7b506ef78887af250e0e3ae09e/src/hotspot/share/prims/JVMtiEnv.cpp#L456)的实现





## Java-debug-tool

Java-debug-tool是一个使用Java Instrument API来实现的动态调试工具，它通过在目标JVM上启动一个TcpServer来和调试客户端通信。调试客户端通过命令行来发送调试命令给TcpServer，TcpServer中有专门用来处理命令的handler，handler处理完命令之后会将结果发送回客户端，客户端通过处理将调试结果展示出来。下面将详细介绍Java-debug-tool的整体设计和实现



### Java-debug-tool整体架构

Java-debug-tool包括一个Java Agent和一个用于处理调试命令的核心API，核心API通过一个自定义的类加载器加载进来，以保证目标JVM的类不会被污染。整体上Java-debug-tool的设计是一个Client-Server的架构，命令客户端需要完整的完成一个命令之后才能继续执行下一个调试命令。Java-debug-tool支持多人同时进行调试，下面是整体架构图：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174610464-1097594053.png)

下面对每一层做简单介绍：

- 交互层：负责将程序员的输入转换成调试交互协议，并且将调试信息呈现出来
- 连接管理层：负责管理客户端连接，从连接中读调试协议数据并解码，对调试结果编码并将其写到连接中去；同时将那些超时未活动的连接关闭
- 业务逻辑层：实现调试命令处理，包括命令分发、数据收集、数据处理等过程
- 基础实现层：Java-debug-tool实现的底层依赖，通过Java Instrumentation提供的API进行类查找、类重定义等能力，Java Instrumentation底层依赖JVMTI来完成具体的功能

在Agent被挂载到目标JVM上之后，Java-debug-tool会安排一个Spy在目标JVM内活动，这个Spy负责将目标JVM内部的相关调试数据转移到命令处理模块，命令处理模块会处理这些数据，然后给客户端返回调试结果。命令处理模块会增强目标类的字节码来达到数据获取的目的，多个客户端可以共享一份增强过的字节码，无需重复增强下面从Java-debug-tool的字节码增强方案、命令设计与实现等角度详细说明





### Java-debug-tool的字节码增强方案

Java-debug-tool使用字节码增强来获取到方法运行时的信息，比如方法入参、出参等，可以在不同的字节码位置进行增强，这种行为可以称为“插桩”，每个“桩”用于获取数据并将他转储出去。Java-debug-tool具备强大的插桩能力，不同的桩负责获取不同类别的数据，下面是Java-debug-tool目前所支持的“桩”：

- 方法进入点：用于获取方法入参信息
- Fields获取点1：在方法执行前获取到对象的字段信息
- 变量存储点：获取局部变量信息
- Fields获取点2：在方法退出前获取到对象的字段信息
- 方法退出点：用于获取方法返回值
- 抛出异常点：用于获取方法抛出的异常信息
- 通过上面这些代码桩，Java-debug-tool可以收集到丰富的方法执行信息，经过处理可以返回更加可视化的调试结果



#### 字节码增强

Java-debug-tool在实现上使用了ASM工具来进行字节码增强，并且每个插桩点都可以进行配置，如果不想要什么信息，则没必要进行对应的插桩操作这种可配置的设计是非常有必要的，因为有时候我们仅仅是想要知道方法的入参和出参，但Java-debug-tool却给我们返回了所有的调试信息，这样我们就得在众多的输出中找到我们所关注的内容。如果可以进行配置，则除了入参点和出参点外其他的桩都不插，那么就可以快速看到我们想要的调试数据，这种设计的本质是为了让调试者更加专注。下面是Java-debug-tool的字节码增强工作方式：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174639872-1049167292.png)

如图所示，当调试者发出调试命令之后，Java-debug-tool会识别命令并判断是否需要进行字节码增强，如果命令需要增强字节码，则判断当前类+当前方法是否已经被增强过。上文已经提到，字节码替换是有一定损耗的，这种具有损耗的操作发生的次数越少越好，所以字节码替换操作会被记录起来，后续命令直接使用即可，不需要重复进行字节码增强，字节码增强还涉及多个调试客户端的协同工作问题，当一个客户端增强了一个类的字节码之后，这个客户端就锁定了该字节码，其他客户端变成只读，无法对该类进行字节码增强，只有当持有锁的客户端主动释放锁或者断开连接之后，其他客户端才能继续增强该类的字节码

字节码增强模块收到字节码增强请求之后，会判断每个增强点是否需要插桩，这个判断的根据就是上文提到的插桩配置，之后字节码增强模块会生成新的字节码，Java-debug-tool将执行字节码替换操作，之后就可以进行调试数据收集了

经过字节码增强之后，原来的方法中会插入收集运行时数据的代码，这些代码在方法被调用的时候执行，获取到诸如方法入参、局部变量等信息，这些信息将传递给数据收集装置进行处理。数据收集的工作通过Advice完成，每个客户端同一时间只能注册一个Advice到Java-debug-tool调试模块上，多个客户端可以同时注册自己的Advice到调试模块上。Advice负责收集数据并进行判断，如果当前数据符合调试命令的要求，Java-debug-tool就会卸载这个Advice，Advice的数据就会被转移到Java-debug-tool的命令结果处理模块进行处理，并将结果发送到客户端





#### Advice的工作方式

Advice是调试数据收集器，不同的调试策略会对应不同的Advice。Advice是工作在目标JVM的线程内部的，它需要轻量级和高效，意味着Advice不能做太过于复杂的事情，它的核心接口“match”用来判断本次收集到的调试数据是否满足调试需求。如果满足，那么Java-debug-tool就会将其卸载，否则会继续让他收集调试数据，这种“加载Advice” -> “卸载Advice”的工作模式具备很好的灵活性

关于Advice，需要说明的另外一点就是线程安全，因为它加载之后会运行在目标JVM的线程中，目标JVM的方法极有可能是多线程访问的，这也就是说，Advice需要有能力处理多个线程同时访问方法的能力，如果Advice处理不当，则可能会收集到杂乱无章的调试数据。下面的图片展示了Advice和Java-debug-tool调试分析模块、目标方法执行以及调试客户端等模块的关系

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174703200-324764638.png)

Advice的首次挂载由Java-debug-tool的命令处理器完成，当一次调试数据收集完成之后，调试数据处理模块会自动卸载Advice，然后进行判断，如果调试数据符合Advice的策略，则直接将数据交由数据处理模块进行处理，否则会清空调试数据，并再次将Advice挂载到目标方法上去，等待下一次调试数据。非首次挂载由调试数据处理模块进行，它借助Advice按需取数据，如果不符合需求，则继续挂载Advice来获取数据，否则对调试数据进行处理并返回给客户端





### Java-debug-tool的命令设计与实现

#### 命令执行

上文已经完整的描述了Java-debug-tool的设计以及核心技术方案，本小节将详细介绍Java-debug-tool的命令设计与实现。首先需要将一个调试命令的执行流程描述清楚，下面是一张用来表示命令请求处理流程的图片：

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174727030-2620359.png)

上图简单的描述了Java-debug-tool的命令处理方式，客户端连接到服务端之后，会进行一些协议解析、协议认证、协议填充等工作，之后将进行命令分发。服务端如果发现客户端的命令不合法，则会立即返回错误信息，否则再进行命令处理命令。处理属于典型的三段式处理，前置命令处理、命令处理以及后置命令处理，同时会对命令处理过程中的异常信息进行捕获处理，三段式处理的好处是命令处理被拆成了多个阶段，多个阶段负责不同的职责。前置命令处理用来做一些命令权限控制的工作，并填充一些类似命令处理开始时间戳等信息，命令处理就是通过字节码增强，挂载Advice进行数据收集，再经过数据处理来产生命令结果的过程，后置处理则用来处理一些连接关闭、字节码解锁等事项

Java-debug-tool允许客户端设置一个命令执行超时时间，超过这个时间则认为命令没有结果，如果客户端没有设置自己的超时时间，就使用默认的超时时间进行超时控制。Java-debug-tool通过设计了两阶段的超时检测机制来实现命令执行超时功能：首先，第一阶段超时触发，则Java-debug-tool会友好的警告命令处理模块处理时间已经超时，需要立即停止命令执行，这允许命令自己做一些现场清理工作，当然需要命令执行线程自己感知到这种超时警告；当第二阶段超时触发，则Java-debug-tool认为命令必须结束执行，会强行打断命令执行线程。超时机制的目的是为了不让命令执行太长时间，命令如果长时间没有收集到调试数据，则应该停止执行，并思考是否调试了一个错误的方法当然，超时机制还可以定期清理那些因为未知原因断开连接的客户端持有的调试资源，比如字节码锁





#### 获取方法执行视图

Java-debug-tool通过下面的信息来向调试者呈现出一次方法执行的视图：

- 正在调试的方法信息
- 方法调用堆栈
- 调试耗时，包括对目标JVM造成的STW时间
- 方法入参，包括入参的类型及参数值
- 方法的执行路径
- 代码执行耗时
- 局部变量信息
- 方法返回结果
- 方法抛出的异常
- 对象字段值快照

下图展示了Java-debug-tool获取到正在运行的方法的执行视图的信息

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174745797-11375522.png)





### Java-debug-tool与同类产品对比分析

Java-debug-tool的同类产品主要是greys，其他类似的工具大部分都是基于greys进行的二次开发，所以直接选择greys来和Java-debug-tool进行对比

![img](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230809174806378-1515853036.jpg)

本文详细剖析了Java动态调试关键技术的实现细节，并介绍了我们基于Java动态调试技术结合实际故障排查场景进行的一点探索实践；动态调试技术为研发人员进行线上问题排查提供了一种新的思路，我们基于动态调试技术解决了传统断点调试存在的问题，使得可以将断点调试这种技术应用在线上，以线下调试的思维来进行线上调试，提高问题排查效率





## 参考文献

- [ASM 4 guide](https://asm.ow2.io/asm4-guide.pdf)
- [Java Virtual Machine Specification](https://docs.oracle.com/Javase/specs/JVMs/se7/html/JVMs-4.html)
- [JVM Tool Interface](https://docs.oracle.com/Javase/8/docs/platform/JVMti/JVMti.html)
- [alibaba arthas](https://alibaba.github.io/arthas/)
- [openJDK](https://github.com/pandening/openJDK)



