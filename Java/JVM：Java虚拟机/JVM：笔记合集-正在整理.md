# JVM整体结构

> 本文主要说的是HotSpot虚拟机，



JVM 全称是 Java Virtual Machine，中文译名：Java虚拟机

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031181910930-41332754.jpg" alt="JVM-framework" style="zoom:80%;" />



简化一下：

![image-20231110204142543](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231110204142530-1227278687.png)









# Java字节码文件

> Class文件本质上是一个以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在Class文件中，JVM根据其特定的规则解析该二进制数据，从而得到相关信息
>
> Class文件采用一种伪结构来存储数据，它有两种类型：无符号数和表

首先从整体上看一下Java字节码文件所包含的内容：

![image-20231031215759601](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031215800890-1616844151.png)









## 初识Class文件、基础信息

```java
package com.zixieqing;

public class KnowClass {
    static int a = 0;

    public static void main(String[] args) {
        int b = a++;

        System.out.println("b = " + b);
    }
}
```

通过以下命令, 可以在当前所在路径下生成一个 .Class 文件

```java
javac KnowClass.java
```

使用NotePad++的十六进制插件（HEX-Editor）打开编译后的Class文件，部分截图如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031223853827-38591583.png" alt="image-20231031223852875" style="zoom:80%;" />



其中：

- 左边Address这一列：是当前文件中的地址
- 中间部分：是整个十六进制数据
- 右边Dump这一列：是编码之后的结果

对于中间部分数据：

1. 文件开头的4个字节（“cafe babe”）就是所谓的“magic魔数”。唯有以"cafe babe"开头的Class文件方可被虚拟机所接受，这4个字节就是字节码文件的身份识别

文件是无法通过文件扩展名来确定文件类型的，文件扩展名可以随意修改，不影响文件的内容

软件使用文件的头几个字节（文件头）去校验文件的类型，如果软件不支持该种类型就会出错

**Java字节码文件中，将文件头称为magic魔数**

![image-20231031214631078](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031214632056-1402615546.png)



2. 0000是编译器JDK版本的次版本号0，0034转化为十进制是52，是主版本号

主次版本号指的是编译字节码文件的JDK版本号

主版本号用来标识大版本号

JDK1.0-1.1使用了45.0-45.3，JDK1.2是46之后每升级一个大版本就加1；副版本号是当主版本号相同时作为区分不同

版本的标识，一般只需要关心主版本号

```txt
1.2之后大版本号计算方法就是:
主版本号 – 44
比如主版本号52就是52 - 44 = 8，即JDK8

以前用的 Java -version 命令也就可以验证
PS C:\Users\zixq\Desktop> java -version
java version "1.8.0_221"
Java(TM) SE Runtime Environment (build 1.8.0_221-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.221-b11, mixed mode)
```

**版本号的作用主要是判断当前字节码的版本和运行时的JDK是否兼容**

```txt
主版本号不兼容导致的错误的两种解决方案：
1.升级JDK版本													（容易引发其他的兼容性问题，并且需要大量的测试）
2.将第三方依赖的版本号降低或者更换依赖，以满足JDK版本的要求			√ 建议采用
```







## 反编译Class文件

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

输入命令`Javap -verbose -p KnowClass.Class`查看输出内容:

```java
Classfile /E:/Study/JVM-Demo/out/production/JVM-Demo/com/zixieqing/KnowClass.class			// Class文件当前所在位置
  Last modified 2023-10-31; size 862 bytes													// 最后修改时间、文件大小
  MD5 checksum 1b6100d02bb70d920adceac139839609												// MD5值
  Compiled from "KnowClass.java"															// 编译自哪个文件
public class com.zixieqing.KnowClass														// 类全限定名
  minor version: 0																			// 次版本号
  major version: 52																			// 主版本号
  flags: ACC_PUBLIC, ACC_SUPER																// 该类的访问标志	一会儿单独说明有哪些
Constant pool:																				// 常量池
   #1 = Methodref          #12.#30        // java/lang/Object."<init>":()V
   #2 = Fieldref           #11.#31        // com/zixieqing/KnowClass.a:I
   #3 = Fieldref           #32.#33        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Class              #34            // java/lang/StringBuilder
   #5 = Methodref          #4.#30         // java/lang/StringBuilder."<init>":()V
   #6 = String             #35            // b =
   #7 = Methodref          #4.#36         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #4.#37         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   #9 = Methodref          #4.#38         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #10 = Methodref          #39.#40        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #11 = Class              #41            // com/zixieqing/KnowClass
  #12 = Class              #42            // java/lang/Object
  #13 = Utf8               a
  #14 = Utf8               I
  #15 = Utf8               <init>
  #16 = Utf8               ()V
  #17 = Utf8               Code
  #18 = Utf8               LineNumberTable
  #19 = Utf8               LocalVariableTable
  #20 = Utf8               this
  #21 = Utf8               Lcom/zixieqing/KnowClass;
  #22 = Utf8               main
  #23 = Utf8               ([Ljava/lang/String;)V
  #24 = Utf8               args
  #25 = Utf8               [Ljava/lang/String;
  #26 = Utf8               b
  #27 = Utf8               <clinit>
  #28 = Utf8               SourceFile
  #29 = Utf8               KnowClass.java
  #30 = NameAndType        #15:#16        // "<init>":()V
  #31 = NameAndType        #13:#14        // a:I
  #32 = Class              #43            // java/lang/System
  #33 = NameAndType        #44:#45        // out:Ljava/io/PrintStream;
  #34 = Utf8               java/lang/StringBuilder
  #35 = Utf8               b =
  #36 = NameAndType        #46:#47        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #37 = NameAndType        #46:#48        // append:(I)Ljava/lang/StringBuilder;
  #38 = NameAndType        #49:#50        // toString:()Ljava/lang/String;
  #39 = Class              #51            // java/io/PrintStream
  #40 = NameAndType        #52:#53        // println:(Ljava/lang/String;)V
  #41 = Utf8               com/zixieqing/KnowClass
  #42 = Utf8               java/lang/Object
  #43 = Utf8               java/lang/System
  #44 = Utf8               out
  #45 = Utf8               Ljava/io/PrintStream;
  #46 = Utf8               append
  #47 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #48 = Utf8               (I)Ljava/lang/StringBuilder;
  #49 = Utf8               toString
  #50 = Utf8               ()Ljava/lang/String;
  #51 = Utf8               java/io/PrintStream
  #52 = Utf8               println
  #53 = Utf8               (Ljava/lang/String;)V
{
  static int a;
    descriptor: I
    flags: ACC_STATIC

  public com.zixieqing.KnowClass();				// 方法表集合，就是前面看Class整体中说的“方法”
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/zixieqing/KnowClass;

  public static void main(java.lang.String[]);	// 方法表集合		一会儿说明
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=2, args_size=1
         0: getstatic     #2                  // Field a:I
         3: dup
         4: iconst_1
         5: iadd
         6: putstatic     #2                  // Field a:I
         9: istore_1
        10: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
        13: new           #4                  // class java/lang/StringBuilder
        16: dup
        17: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
        20: ldc           #6                  // String b =
        22: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        25: iload_1
        26: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        29: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        32: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        35: return
      LineNumberTable:
        line 16: 0
        line 18: 10
        line 19: 35
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      36     0  args   [Ljava/lang/String;
           10      26     1     b   I

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: iconst_0
         1: putstatic     #2                  // Field a:I
         4: return
      LineNumberTable:
        line 13: 0
}
SourceFile: "KnowClass.java"				// 资源文件名	即源代码的文件名
```

上面提到了类的访问标志：ACC_PUBLIC, ACC_SUPER，访问标志的含义如下:

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







## 常量池

`Constant pool`意为常量池

常量池中的数据都有一个编号，编号从1开始。在字段或者字节码指令中通过编号可以快速的找到对应的数据

字节码指令中通过编号引用到常量池的过程称之为“符号引用”

常量池可以理解成Class文件中的资源仓库，主要存放的是两大类常量：字面量(Literal)和符号引用(Symbolic References)，字面量类似于Java中的常量概念，如文本字符串，final常量等，而符号引用则属于编译原理方面的概念，包括以下三种:

- 类和接口的全限定名(Fully Qualified Name)
- 字段的名称和描述符号(Descriptor)
- 方法的名称和描述符

不同于C/C++,，JVM是在加载Class文件的时候才进行的动态链接，也就是说这些字段和方法符号引用只有在运行期转换后才能获得真正的内存入口地址，当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建或运行时解析并翻译到具体的内存地址中。如上一节反编译的文件中的常量池：

```java
Constant pool:							  // 常量池
   #1 = Methodref          #12.#30        // java/lang/Object."<init>":()V
   #2 = Fieldref           #11.#31        // com/zixieqing/KnowClass.a:I
   #3 = Fieldref           #32.#33        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Class              #34            // java/lang/StringBuilder
   #5 = Methodref          #4.#30         // java/lang/StringBuilder."<init>":()V
   #6 = String             #35            // b =
   #7 = Methodref          #4.#36         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #4.#37         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   #9 = Methodref          #4.#38         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #10 = Methodref          #39.#40        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #11 = Class              #41            // com/zixieqing/KnowClass
  #12 = Class              #42            // java/lang/Object
  #13 = Utf8               a
  #14 = Utf8               I
  #15 = Utf8               <init>
  #16 = Utf8               ()V
  #17 = Utf8               Code
  #18 = Utf8               LineNumberTable
  #19 = Utf8               LocalVariableTable
  #20 = Utf8               this
  #21 = Utf8               Lcom/zixieqing/KnowClass;
  #22 = Utf8               main
  #23 = Utf8               ([Ljava/lang/String;)V
  #24 = Utf8               args
  #25 = Utf8               [Ljava/lang/String;
  #26 = Utf8               b
  #27 = Utf8               <clinit>
  #28 = Utf8               SourceFile
  #29 = Utf8               KnowClass.java
  #30 = NameAndType        #15:#16        // "<init>":()V
  #31 = NameAndType        #13:#14        // a:I
  #32 = Class              #43            // java/lang/System
  #33 = NameAndType        #44:#45        // out:Ljava/io/PrintStream;
  #34 = Utf8               java/lang/StringBuilder
  #35 = Utf8               b =
  #36 = NameAndType        #46:#47        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #37 = NameAndType        #46:#48        // append:(I)Ljava/lang/StringBuilder;
  #38 = NameAndType        #49:#50        // toString:()Ljava/lang/String;
  #39 = Class              #51            // java/io/PrintStream
  #40 = NameAndType        #52:#53        // println:(Ljava/lang/String;)V
  #41 = Utf8               com/zixieqing/KnowClass
  #42 = Utf8               java/lang/Object
  #43 = Utf8               java/lang/System
  #44 = Utf8               out
  #45 = Utf8               Ljava/io/PrintStream;
  #46 = Utf8               append
  #47 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #48 = Utf8               (I)Ljava/lang/StringBuilder;
  #49 = Utf8               toString
  #50 = Utf8               ()Ljava/lang/String;
  #51 = Utf8               java/io/PrintStream
  #52 = Utf8               println
  #53 = Utf8               (Ljava/lang/String;)V
```

**第一个常量**是一个方法定义，指向了第12和第30个常量。以此类推查看第12和第30个常量最后可以拼接成第一个常量右侧的注释内容:

```java
// java/lang/Object."<init>":()V
```

这段可以理解为该类的实例构造器的声明，由于Main类没有重写构造方法，所以调用的是父类的构造方法。此处也说明了Main类的直接父类是Object，该方法默认返回值是V, 也就是void，无返回值

**第二个常量**同理可得:

```java
  #2 = Fieldref           #11.#31        // com/zixieqing/KnowClass.a:I
  #11 = Class              #41            // com/zixieqing/KnowClass
  #13 = Utf8               a
  #14 = Utf8               I
  #41 = Utf8               com/zixieqing/KnowClass
  #31 = NameAndType        #13:#14        // a:I
```

此处声明了一个字段a，类型为I，I即为int类型。关于字节码的类型对应如下：

| 标识字符 | 含义                                       | 备注   |
| -------- | ------------------------------------------ | ------ |
| B        | 基本类型byte                               |        |
| C        | 基本类型char                               |        |
| D        | 基本类型double                             |        |
| F        | 基本类型float                              |        |
| I        | 基本类型int                                |        |
| J        | 基本类型long                               | 特殊记 |
| S        | 基本类型short                              |        |
| Z        | 基本类型boolean                            | 特殊记 |
| V        | 特殊类型void                               |        |
| L        | 对象类型，以分号结尾，如LJava/lang/Object; | 特殊记 |

对于数组类型，每一位使用一个前置的`[`字符来描述，如定义一个`Java.lang.String[][]`类型的二维数组，将被记录为`[[LJava/lang/String;`













## 方法表集合

在常量池之后的是对类内部的方法描述，在字节码中以表的集合形式表现，暂且不管字节码文件的16进制文件内容如何，我们直接看反编译后的内容

```java
  static int a;
    descriptor: I
    flags: ACC_STATIC
```

此处声明了一个static的变量a，类型为int

```java
  public com.zixieqing.KnowClass();				// 方法表集合，就是前面看Class整体中说的“方法”
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lcom/zixieqing/KnowClass;
```

这里是构造方法：KnowClass()，返回值为void, 权限修饰符为public

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



上面Code中有一个小东西：`0: aload_0`，这里的 `aload_0` 叫做虚拟机字节码指令

> 关于更多虚拟机字节码指令，也可以在《深入理解Java虚拟机 ：JVM高级特性与最佳实践-附录B》中获取

这里说几个最基本的虚拟机字节码指令：

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031231841053-1227524628.png" alt="image-20231031231839977" style="zoom:80%;" />



再来个示例：

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031231920242-1520087013.png" alt="image-20231031231919319" style="zoom:80%;" />









## 字节码常用工具

### javap命令

这个命令前面已经玩过

javap是JDK自带的反编译工具，可以通过控制台查看字节码文件的内容。**适合在服务器上查看字节码文件内容**

直接输入javap查看所有参数

输入` javap -v 字节码文件名称` 查看具体的字节码信息。（如果jar包需要先使用 jar –xvf 命令解压）

![image-20231031233150816](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031233151837-1538049901.png)







### jclasslib工具

1. exe方式安装：https://github.com/ingokegel/jclasslib
2. IDEA集成插件：直接在IDEA的plugins中搜索jclasslib即可

![image-20231031233459824](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031233500561-92800441.png)



![image-20231031233844283](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031233845402-930599476.png)



**注意点：**源代码改变之后需要重编译，然后刷新jclasslib，否则看到的就是旧的class文件

![image-20231031234134747](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031234135753-23584882.png)



附加：遇到不会的虚拟机字节码指令时，可以通过jclasslib中右键对应指令，跳入官方文档查看描述

![image-20231031234427465](https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031234428335-284470540.png)





### 阿里Arthas

Arthas 是一款线上监控诊断产品，通过全局视角实时查看应用 load、内存、gc、线程的状态信息，并能在不修改应用代码的情况下，对业务问题进行诊断，大大提升线上问题排查效率

官网：https://arthas.aliyun.com/doc/

<img src="https://img2023.cnblogs.com/blog/2421736/202310/2421736-20231031234600724-1208870057.png" alt="image-20231031234559729" style="zoom:67%;" />



1. 下载jar包，运行jar包：本地查看就下载jar包到本地，Linux中查看线上代码就丢在Linux中即可

````bash
java -jar xxx.jar
````

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231104213441973-819577938.png" alt="image-20231104213439088" style="zoom:67%;" />



进入后使用示例：更多命令参考官网 https://arthas.aliyun.com/doc/commands.html

```bash
dump 类的全限定名		命令含义：dump已加载类的字节码文件到特定目录
					场景：线上查看Class文件，就可选择将此Class文件整到自己规定的目录中去
					
jad 类的全限定名		命令含义：反编译已加载类的源码			
					场景：BUG修复，然后上线，但BUG还在，就可选择此命令将Class文件反编译为源代码（即xxxx.java文件）
						 然后看部署的是修复好BUG的代码 还是 旧代码
```







# 类的生命周期

整个流程如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109171703028-1277212645.png" alt="image-20231109171701801" style="zoom:67%;" />

注意：`加载`、`验证`、`准备`和`初始化`这四个阶段发生的顺序是确定的，*而`解析`阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定(也称为动态绑定或晚期绑定)*。另外：这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段





## 加载：查找并加载类的二进制数据

加载是类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情:

1. 类加载器根据类的全限定名通过不同的渠道以二进制流的方式获取字节码信息。如下列的不同渠道：

- 从本地系统中直接加载
- 通过网络下载.Class文件
- 从zip，jar等归档文件中加载.Class文件
- 从专有数据库中提取.Class文件
- 将Java源文件动态编译为.Class文件

2. 将这个字节流所代表的静态存储结构转化为**方法区**的运行时数据结构，**生成一个InstanceKlass对象**，保存类的所有信息，里边还包含实现特定功能比如多态的信息。

![image-20231107180858869](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107180900267-132766341.png)

3. Java虚拟机会在**堆中生成**一份与方法区中数据“类似”的**java.lang.Class对象**，作为对方法区中相关数据的访问入口。

作用：在Java代码中去获取类的信息以及存储静态字段的数据（这里的静态字段是说的JDK8及之后的Java虚拟机的设计）

PS：堆中生成的java.lang.Class对象信息是方法区中生成的InstanceKlass对象信息的浓缩版，也就是将方法区InstanceKlass中程序员不需要的信息剔除就成为堆中的java.lang.Class对象信息

对于开发者来说，只需要访问堆中的Class对象而不需要访问方法区中所有信息。这样Java虚拟机就能很好地控制开发者访问数据的范围

![image-20231107181729831](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107181731071-1942446648.png)

> **注意：**类加载器并不需要等到某个类被“首次主动使用”时再加载它，JVM规范允许类加载器在预料某个类将要被使用时就预先加载它，如果在预先加载的过程中遇到了 .Class文件 缺失或存在错误，类加载器必须在程序首次主动使用该类时才报告错误(LinkageError错误)，如果这个类一直没有被程序主动使用，那么类加载器就不会报告错误



既然要加载类，那是怎么加载的就得了解了，而这就不得不了解“JVM类加载机制”了





### JVM类加载机制

#### 类加载器的分类

> 类加载器（ClassLoader）是Java虚拟机提供给应用程序去实现获取类和接口字节码数据的技术。
>
> 类加载器只参与加载过程中的字节码获取并加载到内存这一部分。

**类加载器分为两类，一类是Java代码中实现的，一类是Java虚拟机底层源码实现的**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107201852011-1580861569.png" alt="image-20231107201850757" style="zoom:67%;" />

 

类加载器的设计JDK8和8之后的版本差别较大，JDK8及之前的版本中默认的类加载器有如下几种：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107201957359-1777713058.png" alt="image-20231107201956280" style="zoom:80%;" />

##### 查找类加载器

1. **Arthas关于中类加载器的详细信息的查看方式**。[Arthas之classloader命令官网说明](https://arthas.aliyun.com/doc/classloader.html)

```bash
classloader		# 查看 classloader 的继承树，urls，类加载信息，使用 classloader 去获取资源
```

参数说明：

|              参数名称 | 参数说明                                   |
| --------------------: | :----------------------------------------- |
|                   [l] | 按类加载实例进行统计                       |
|                   [t] | 打印所有 ClassLoader 的继承树              |
|                   [a] | 列出所有 ClassLoader 加载的类，请谨慎使用  |
|                `[c:]` | ClassLoader 的 hashcode                    |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name |
|             `[c: r:]` | 用 ClassLoader 去查找 resource             |
|          `[c: load:]` | 用 ClassLoader 去加载指定的类              |

示例：

PS：下图那些数量指的是我进入的当前这个线程相关的，不是说所有的数量就是那些，根据进程会发生改变

![image-20231107203154937](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107203156691-1726406511.png)

另外：Arthas也可以查看 JVM 已加载的”某个类“的类信息，[Arthas之sc命令官网说明](https://arthas.aliyun.com/doc/sc.html)

sc命令，即“Search-Class” 的简写，这个命令能搜索出所有已经加载到 JVM 中的 Class 信息，这个命令支持的参数有 `[d]`、`[E]`、`[f]` 和 `[x:]`，参数说明如下：

|              参数名称 | 参数说明                                                     |
| --------------------: | :----------------------------------------------------------- |
|       *class-pattern* | 类名表达式匹配                                               |
|      *method-pattern* | 方法名表达式匹配                                             |
|                   [d] | 输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的 ClassLoader 等详细信息。 如果一个类被多个 ClassLoader 所加载，则会出现多次 |
|                   [E] | 开启正则表达式匹配，默认为通配符匹配                         |
|                   [f] | 输出当前类的成员变量信息（需要配合参数-d 一起使用）          |
|                  [x:] | 指定输出静态变量时属性的遍历深度，默认为 0，即直接使用 `toString` 输出 |
|                `[c:]` | 指定 class 的 ClassLoader 的 hashcode                        |
| `[classLoaderClass:]` | 指定执行表达式的 ClassLoader 的 class name                   |
|                `[n:]` | 具有详细信息的匹配类的最大数量（默认为 100）                 |
|          `[cs <arg>]` | 指定 class 的 ClassLoader#toString() 返回值。长格式`[classLoaderStr <arg>]` |

示例：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107210801646-778180858.png" alt="image-20231107210759606" style="zoom:80%;" />



2. **自己写代码查看**

```java
package com.zixieqing;

import java.io.IOException;

/**
 * <p>
 * 查找类加载器
 * </p>
 *
 * <p>@author     : ZiXieqing</p>
 */

public class FindClassLoader {
    public static void main(String[] args) throws IOException {
        // 获取当前类的类加载器   即应用程序类加载器
        ClassLoader classLoader = FindClassLoader.class.getClassLoader();
        System.out.println("classLoader = " + classLoader);
        // 获取父类加载器  即扩展类加载器
        ClassLoader parentclassLoader = classLoader.getParent();
        System.out.println("parentclassLoader = " + parentclassLoader);
        // 获取启动类加载器     C语言实现，所以结果为 null	目的：出于安安全考虑，所以不许操作此类加载器
        ClassLoader loader = parentclassLoader.getParent();
        System.out.println("loader = " + loader);

        System.in.read();
    }
}
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107204235089-2007068259.png" alt="image-20231107204233782" style="zoom:80%;" />

还有几种获取类加载器的方式：

```java
// 获取当前 ClassLoader
clazz.getClassLoader();

// 获取当前线程上下文的 ClassLoader
Thread.currentThread().getContextClassLoader();

// 获取系统的 ClassLoader
ClassLoader.getSystemClassLoader();

// 获取调用者的 ClassLoader
DriverManager.getCallerClassLoader();
```



通过这个代码查看之后，也可以说三个类加载器的层次关系长下面这个鸟样儿：

> **注：**这几个类加载器并不是继承关系，而是组合，或者可说是层级/上下级关系，只是源码中用的是ClassLoader类型的Parent字段来实现“双亲委派机制”（后续会上源码）。



<img src="C:/Users/zixq/AppData/Roaming/Typora/typora-user-images/image-20231108204352072.png" alt="image-20231108204352072" style="zoom: 67%;" />





##### 启动类加载器

启动类加载器（Bootstrap ClassLoader）是由Hotspot虚拟机提供的、使用C++编写的类加载器。

启动类加载器是无法被Java程序直接引用的。

**默认加载Java安装目录/jre/lib**下的类文件，比如rt.jar（Java的核心类库，如java.lang.String），tools.jar，resources.jar等。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107204705687-1146242012.png" alt="image-20231107204704719" style="zoom:80%;" />





###### 启动类加载器加载用户jar包的方式

1. 放入  **JDK安装目录/jre/lib** 下进行扩展。 ==不推荐==

尽可能不要去更改JDK安装目录中的内容，会出现即时放进去由于文件名不匹配的问题也不会正常地被加载

2. 使用虚拟机参数 **-Xbootclasspath/a:jar包目录/jar包名** 命令扩展。==推荐==

PS：此命令中有一个 “a” 即add的意思，意为添加一个“jar包目录/jar包名”的jar包

IDEA中示例：

```bash
-Xbootclasspath/a:E:/Study/JVM-Demo/jar/FindClassLoader.jar
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107212413880-990849318.png" alt="image-20231107212412442" style="zoom:80%;" />



##### 扩展 与 应用程序类加载器

扩展类加载器和应用程序类加载器都是JDK中提供的、使用Java编写的类加载器。

它们的源码都位于sun.misc.Launcher中，是一个静态内部类。继承自URLClassLoader。具备通过目录或者指定jar包将字节码文件加载到内存中。

**扩展类加载器**：加载 **Java安装目录/jre/lib/ext** 下的类文件

**应用程序类加载器**：加载 **classpath** 下的类文件，此classpath包括“自己项目中编写的类或接口中的文件 和 第三方jar包中的类或接口中的文件（如：maven依赖中的）”

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107213417227-2043964033.png" alt="image-20231107213415977" style="zoom:67%;" />

Arthas验证上述目录：

```bash
classloader -l						# 查看所有类加载器的hash码

classloader –c 类加载器的hash码		# 查看当前类加载器的加载路径
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107215447835-2033678429.png" alt="image-20231107215445557" style="zoom:50%;" />





###### 扩展类加载器加载用户jar包的方式

1. 放入 **JDK安装目录/jre/lib/ext** 下进行扩展。==不推荐==

尽可能不要去更改JDK安装目录中的内容

2. 使用虚拟机参数 **-Djava.ext.dirs=jar包目录** 进行扩展。==推荐==

注意：此种方式会覆盖掉扩展类加载器原始加载的目录。Windows可以使用`;`分号，macos/Linux可以使用`:`冒号隔开，从而将原始目录添加在后面

```bash
# Windows中示例
-Djava.ext.dirs=E:/Study/JVM-Demo/jar/ext;D:/Install/JDK/JDK8/jre/lib/ext
```







#### 双亲委派机制

若是前面查看了应用程序类加载器的加载路径的话，会发现一个有意思的地方：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231107224850080-1625458848.png" alt="image-20231107224848371" style="zoom:67%;" />

查看应用程序类加载器的加载路径，发现启动类加载器的加载路径也有，为什么？

要搞清楚这个问题，就需要知道“双亲委派机制”了。

>  **双亲委派机制：**
>
> ![image-20231108204546605](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231108210325720-978226333.png)
>
> 1. 当一个类加载器加载某个类的时候，会**自底向上查找是否加载过**；若加载过直接返回，若一直到最顶层的类加载器都没有加载，再**自顶向下进行加载**。
> 2. **应用程序类加载器的父类加载器是扩展类加载器，扩展类加载器的父类加载器是启动类加载器**。
>
> - PS：严谨点来说，**扩展类加载器没有父类加载器，只是会“委派”给启动类加载器**，即如果类加载的parent为null，则会提交给启动类加载器处理。
>
> 3. 双亲委派机制的好处：一是避免恶意代码替换JDK中的核心类库（如：java.lang.String），确保核心类库得到完整性和安全性；二是避免一个类重复被加载



**双亲委派机制的作用：**

1. **避免类被重复加载**：若一个类重复出现在三个类加载器的加载位置，应该由谁加载？

启动类加载器加载，根据双亲委派机制，它的优先级是最高的。

双亲委派机制可以避免同一个类被多次加载，上层的类加载器如果加载过该类，就会直接返回该类，避免重复加载。

2. **保证类加载的安全性**：如在自己的项目中创建一个java.lang.String类，会被加载吗？

不能，会交由启动类加载器加载在rt.jar包中的String类。

通过双亲委派机制，让顶层的类加载器去加载核心类，**避免恶意代码替换JDK中的核心类库**（这个也叫**沙箱安全机制**），如：上述的java.lang.String，确保核心类库的完整性和安全性。

同时底层源码中建包“禁止”以`java.`开头

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231110195924054-1517961328.png" alt="image-20231110195922678" style="zoom:50%;" />



上面定义中第一点说“类加载器加载某个类的时候”，那在Java中怎么加载一个类？这就需要知道Java中类加载的方式了。





##### Java中类加载的方式

在Java中如何使用代码的方式去主动加载一个类？Java中类加载有两种方式：

1. 通过 `Class.forName()` 方法动态加载

2. 通过 `ClassLoader.loadClass()` 方法动态加载

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

> Class.forName() 和 ClassLoader.loadClass()区别?

- Class.forName():：将类的.Class文件加载到JVM中之外，还会对类进行解释，执行类中的static块；
- ClassLoader.loadClass():：只干一件事情，就是将.Class文件加载到JVM中，不会执行static中的内容,只有在newInstance才会去执行static块
- Class.forName(name, initialize, loader)带参函数也可控制是否加载static块并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 







##### Java中查看双亲委派机制的实现

```java
package java.lang;


public abstract class ClassLoader {
    
    // 每个Java实现的类加载器中保存了一个成员变量叫“父”（Parent）类加载器，可以理解为它的上级，并不是继承关系
    private final ClassLoader parent;
    
    
	public Class<?> loadClass(String name) throws ClassNotFoundException {
        // 第二个参数 会决定着是否要开始类的生命周期的解析阶段，实质执行的是"类生命周期中的连接阶段"
        return loadClass(name, false);
    }
    
	protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // 首先判断该类型是否已经被加载
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    // 如果没有被加载，就委托给父类加载或者委派给启动类加载器加载
                    if (parent != null) {
                        // 如果存在父类加载器，就委派给父类加载器加载
                        c = parent.loadClass(name, false);
                    } else {
                        // 如果不存在父类加载器，就检查是否是由启动类加载器加载的类，
                        // 通过调用本地方法native Class findBootstrapClass(String name)
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                // 如果父类加载器和启动类加载器都不能完成加载任务
                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    // 调用自身的加载功能
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            // 是否开始解析，此处resolve = false
            if (resolve) {
                // 最终调用 private native void resolveClass0(Class<?> c);
                resolveClass(c);
            }
            return c;
        }
    }
}
```







##### 自定义类加载器

通常情况下，我们都是直接使用系统类加载器，但是，有的时候，我们也需要自定义类加载器，比如应用是通过网络来传输 Java 类的字节码，为保证安全性，这些字节码经过了加密处理，这时系统类加载器就无法对其进行加载，此时就需要我们自定义类加载器。

而需要自定义类加载器，就需要了解几个API：

```java
package java.lang;


public abstract class ClassLoader {

    /**
     * 类加载的入口，提供双亲委派机制，调用了 findClass(String name)
     */
    protected Class<?> loadClass(String name, boolean resolve) {
        // .............
    }
    
    /**
     * 由类加载器子类实现获取二进制数据，
     * 调用 defineClass(String name, byte[] b, int off, int len) 如：URLClassLoader 会根据文件路径获取类文件中的二进制数据
     */
	protected Class<?> findClass(String name) {
        throw new ClassNotFoundException(name);
    }
}


/**
 * 做一些类名的校验，然后调用虚拟机底层的方法将字节码信息加载到虚拟机内存中
 */
protected final Class<?> defineClass(String name, byte[] b, int off, int len){
    // .........
}


/**
 * 执行类生命周期中的连接阶段
 */
protected final void resolveClass(Class<?> c){
    // .........
}
```

从上对 loadClass 方法来分析来看：想要自定义类加载器，那么继承 ClassLoader 类，重写 findClass() 即可，示例如下：

> JDK1.2之前是重写loadClass方法，但JDK1.2之后就建议自定义类加载器最好重写findClass方法而不要重写loadClass方法，因为这样容易破坏双亲委托模式。



```java
package com.zixieqing;

import java.io.*;

/**
 * <p>
 * 自定义类加载器：核心在于对字节码文件的获取
 * </p>
 *
 * <p>@author     : ZiXieqing</p>
 */

public class MyClassLoader extends ClassLoader {

    private String root;

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 获取二进制数据
        byte[] ClassData = loadClassData(name);

        if (ClassData == null) {
            throw new ClassNotFoundException();
        } else {
            // 校验，将字节码信息加载进虚拟机内存
            return defineClass(name, ClassData, 0, ClassData.length);
        }
    }

    /**
     * 加载二进制数据
     * @param className Class全限定名
     */
    private byte[] loadClassData(String className) {

        // 得到Class字节码文件名
        String fileName = root + File.separatorChar + className.replace('.', File.separatorChar) + ".Class";

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

    public void setRoot(String root) {
        this.root = root;
    }

    public static void main(String[] args) throws ClassNotFoundException {

        MyClassLoader myClassLoader = new MyClassLoader();
        // 设置此类加载器的加载目录
        myClassLoader.setRoot("E:\\lib\\");

        // 要加载哪个类
        // 传递的文件名是类的全限定名，以 "." 隔开，因为 defineClass() 是按这种格式进行处理的
        Class<?> testClass = myClassLoader.loadClass("com.zixq.FindClassLoader");
        System.out.println(testClass.getClassLoader());
        System.out.println(testClass.getClassLoader().getParent());
    }
}
```

结果如下：

```java
om.zixieqing.MyClassLoader@1b6d3586
sun.misc.Launcher$AppClassLoader@18b4aac2
```

1. 问题：**自定义类加载器父类怎么是AppClassLoader**？

以Jdk8为例，ClassLoader类中提供了构造方法设置parent的内容：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109145557267-2110824427.png" alt="image-20231109145558395" style="zoom:67%;" />

这个构造方法由另外一个构造方法调用，其中父类加载器由getSystemClassLoader方法设置，该方法返回的是AppClassLoader。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109145800286-1067637581.png" alt="image-20231109145801435" style="zoom: 50%;" />



2. 问题：**两个自定义类加载器加载相同限定名的类，会不会冲突？**

不会冲突，在同一个Java虚拟机中，只有“相同类加载器+相同的类限定名”才会被认为是同一个类。

```bash
# 在Arthas中使用下列方式查看具体的情况
sc –d 类的全限定名
```







#### Java9之后的类加载器

JDK8及之前的版本中，扩展类加载器和应用程序类加载器的源码位于rt.jar包中的s`un.misc.Launcher.java`。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109202531696-2116661804.png" alt="image-20231109202530496" style="zoom: 67%;" />

DK9引入了module的概念，类加载器在设计上发生了变化：

1. 启动类加载器使用Java编写，位于`jdk.internal.loader.ClassLoaders`类中。

Java中的BootClassLoader继承自BuiltinClassLoader实现从模块中找到要加载的字节码资源文件。

启动类加载器依然无法通过Java代码获取到，返回的仍然是null，保持了统一。

2. 扩展类加载器被替换成了平台类加载器（Platform Class Loader）。

平台类加载器遵循模块化方式加载字节码文件，所以继承关系从URLClassLoader变成了BuiltinClassLoader，BuiltinClassLoader实现了从模块中加载字节码文件。平台类加载器的存在更多的是为了与老版本的设计方案兼容，自身没有特殊的逻辑。





## 连接

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109171703028-1277212645.png" alt="image-20231109171701801" style="zoom:67%;" />



### 验证：验证被加载的类是否满足Java虚拟机规范

> 验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息遵守《Java虚拟机规范》中的约束，并且不会危害虚拟机自身的安全。

==这个阶段一般不需要程序员参与==。验证阶段大致会完成4个阶段的检验动作:

- `文件格式验证`:：验证字节流是否符合Class文件格式的规范；例如: 是否以`0xCAFEBABE`开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型
- `元数据验证`:：对字节码描述的信息进行语义分析(注意: 对比`Javac`编译阶段的语义分析)，以保证其描述的信息符合Java语言规范的要求；例如: 这个类是否有父类，除了`Java.lang.Object`之外
- `字节码验证`:：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的
- `符号引用验证`:确保解析动作能正确执行

> 验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，
>
> **如果所引用的类经过反复验证，那么可以考虑采用`-Xverifynone`参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间**





### 准备：为静态变量分配内存 并 为其设置默认值

> 准备阶段是为静态变量（static）分配内存并设置默认值。**这些内存都将在方法区中分配**。

对于该阶段有以下几点需要注意：

1. 这时候进行内存分配的==仅包括静态变量(`static`)==，而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中。
2. 这里所设置的默认值通常情况下是数据类型默认的零值，而不是在Java代码中被显式地赋予的值。

| 数据类型     | 默认值   |
| ------------ | -------- |
| byte         | 0        |
| short        | 0        |
| int          | 0        |
| long         | 0L       |
| boolean      | false    |
| double       | 0.0      |
| char         | ‘\u0000’ |
| 引用数据类型 | null     |

- 对基本数据类型来说，静态变量(static)和全局变量，如果不显式地对其赋值而直接使用，则系统会为其赋予默认的零值；而对于局部变量来说，在使用前必须显式地为其赋值，否则编译时不通过。
- 只被`final`修饰的常量则既可以在声明时显式地为其赋值，也可以在类初始化时显式地为其赋值，总之，在使用前必须为其显式地赋值，系统不会为其赋予默认零值。
- 同时被`static`和`final`修饰的基本数据类型的常量，准备阶段直接会将代码中的值进行赋值（即必须在声明的时候就为其显式地赋值，否则编译时不通过）。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109181450986-1215107925.png" alt="image-20231109181449857" style="zoom: 50%;" />









### 解析：将常量池中的符号引用 替换为 指向内存的直接引用

> 解析阶段主要是将常量池中的符号引用替换为直接引用。
>
> 解析动作主要针对`类`或`接口`、`字段`、`类方法`、`接口方法`、`方法类型`、`方法句柄`和`调用点`限定符7类符号引。

符号引用就是在字节码文件中使用编号来访问常量池中的内容。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109183305002-712655662.png" alt="image-20231109183303764" style="zoom:67%;" />

直接引用不在使用编号，而是使用内存中地址进行访问具体的数据。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109183446304-200246862.png" alt="image-20231109183445217" style="zoom:80%;" />





## 初始化：为类的静态变量赋正确值，对类进行初始化

> 初始化：为类的静态变量赋予正确的初始值，JVM负责对类进行初始化（主要对类变量进行初始化）。

初始化阶段会执行字节码文件中 ==clinit== （若是有）部分的字节码指令。clinit就是class init，即类构造方法，此类构造方法不是我们说的构造方法，不是一回事。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109201249945-69018762.png" alt="image-20231109201248802" style="zoom:67%;" />

**`clinit`指令在特定情况下不会出现**：

> 注：
>
> 1. 子类的初始化clinit调用之前，会先调用父类的clinit初始化方法。
> 2. 虚拟机会保证一个类的clinit方法在多线程下被同步加锁，即一个类的clinit方法只会被加载一次。

1. 无静态代码块且无静态变量赋值语句。
2. 有静态变量的声明，但是没有赋值语句。
3. 静态变量的定义使用final关键字，这类变量会在准备阶段直接进行初始化。



**在Java中对类变量进行初始值设定有两种方式**：

- 声明类变量时指定初始值
- 使用静态代码块为类变量指定初始值



**JVM初始化步骤：**

1. 若这个类还没有被加载和连接，则程序先加载并连接该类。
2. 若该类的父类还没有被初始化，则先初始化其父类。
3. 若类中有初始化语句，则系统依次执行这些初始化语句。

关于第3点：原因是在链接阶段的准备中，已经将静态变量加载到内存中了，只是初始值是数据类型的默认值而已，而这里初始化就是重新赋正确值，而这顺序就按代码顺序赋值。

如下图，虚拟机指令是按顺序执行初始化语句

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231109205006247-1956213042.png" alt="image-20231109205004723" style="zoom: 67%;" />



**类初始化时机**：只有当对类的主动使用的时候才会导致类的初始化。类的主动使用包括以下六种：

> 添加 `-XX:+TraceClassLoading` 虚拟机参数可以打印出加载并初始化的类。

1. 创建类的对象，也就是new的方式。
2. 访问某个类或接口的静态变量，或者对该静态变量赋值。注：变量是`final`修饰且等号右边是常量不会触发初始化。

```java
/**
 * 变量是 final 修饰且等号右边是常量不会触发初始化
 */

public class Initialization {
    public static final int value = 8;

    static {
        System.out.println("Initialization类被初始化了");
    }
}

class Test {
    public static void main(String[] args) {
        System.out.println(Initialization.value);
    }
}
```

3. 调用类的静态方法。
4. 反射。如：`Class.forName("com.zixieqing.JVM.Test"))` 。若使用的是下面这个API，则是否初始化取决于程序员。

```java
public static Class<?> forName(String name, boolean initialize, ClassLoader loader) {
    
}
```

5. 初始化某个类的子类，则其父类也会被初始化。
6. Java虚拟机启动时被标明为启动类的类，直接使用`Java.exe`命令来运行某个主类。





## 使用

类访问方法区内的数据结构的接口， 对象是Heap（堆）区的数据。



## 卸载

这个玩意儿在垃圾回收还会整。



**Java虚拟机将结束生命周期的几种情况：**

- 调用 Runtime 类或 system 类的 exit 方法，或 Runtime 类的 halt 方法，并且 Java 安全管理器也允许这次 exit 或 halt 操作。
- 程序正常执行结束。
- 程序在执行过程中遇到了异常或错误而异常终止。
- 由于操作系统出现错误而导致Java虚拟机进程终止。







# JVM内存结构

> 注：不要和Java内存模型混淆了。



## 运行时数据区

> **Java虚拟机在运行Java程序过程中管理的内存区域，称之为运行时数据区**。
>
> Java 虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁，另外一些则是与线程一 一对应的，这些与线程一 一对应的数据区域会随着线程开始和结束而创建和销毁。
>
> **线程不共享：**
>
> - 程序计数器
> - 本地方法栈
> - Java虚拟机栈
>
> **线程共享：**
>
> - 方法区
> - 堆



![image-20231110204142543](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231111214356868-657372761.png)





### 程序计数器

> 程序计数器（**Program Counter Register**）也叫**PC**寄存器，**用来存储指向下一条字节码指令的地址，即将要执行的指令代码由执行引擎读取下一条指令**。

1. ==**程序计数器是唯一 一个在 JVM 规范中没有规定任何 内存溢出（`OutOfMemoryError` ）情况的区域**==。

> **内存溢出：** 指的是程序在使用某一块内存区域时，存放的数据需要占用的内存大小超过了虚拟机能提供的内存上限。

- 因为每个线程只存储一个固定长度的内存地址，所以程序计数器是不会发生内存溢出的。 因此：**程序员无需对程序计数器做任何处**。

2. **在 JVM 规范中，每个线程都有它自己的程序计数器，是线程私有的，生命周期与线程的生命周期一致**。
3. 程序计数器是一块很小的内存空间，几乎可以忽略不计，也是运行速度最快的存储区域。
4. 分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

如：分支、跳转

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112142204976-197590123.png" alt="image-20231112142205561" style="zoom: 50%;" />

5. **JVM 中的 PC 寄存器是对物理 PC 寄存器的一种抽象模拟**。



> 接下来结合图理解一下程序计数器。

在加载阶段，虚拟机将字节码文件中的指令读取到内存之后，会将原文件中的偏移量转换成内存地址。每一条字节码指令都会拥有一个内存地址。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112134532660-124307120.png" alt="image-20231112134532805" style="zoom:67%;" />

在代码执行过程中，程序计数器会记录下一行字节码指令的地址。执行完当前指令之后，虚拟机的执行引擎根据程序计数器执行下一行指令。

> **注：**如果当前线程正在执行的是 Java 方法，程序计数器记录的是 JVM 字节码指令地址，如果是执行 native 方法，则是未指定值（undefined）。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112134852332-1705218448.png" alt="image-20231112134852723" style="zoom:67%;" />



1. 问题：**使用程序计数器存储字节码指令地址有什么用？为什么使用程序计数器记录当前线程的执行地址？**

因为CPU需要不停的切换各个线程，当切换回来以后，就得知道接着从哪开始继续执行，JVM的字节码解释器就需要通过改变程序计数器的值来明确下一条应该执行什么样的字节码指令。

2. 问题：**程序计数器为什么会被设定为线程私有的？**

多线程在一个特定的时间段内只会执行其中某一个线程方法，CPU会不停的做任务切换，这样必然会导致经常中断或恢复。为了能够准确的记录各个线程正在执行的当前字节码指令地址，所以为每个线程都分配了一个程序计数器，每个线程都独立计算，不会互相影响。





### 栈内存：Java虚拟机栈 与 本地方法栈

> 栈即先进后出（First In Last Out），是一种快速有效的分配存储方式，访问速度仅次于程序计数器.
>
> ==栈不存在垃圾回收问题==。



#### Java虚拟机栈（JVM Stack）

> **Java**虚拟机栈（Java Virtual Machine Stack），早期也叫 Java 栈。采用**栈**的数据结构来管理方法调用中的基本数据。
>
> 每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的**栈帧(Stack Frame）**，每一个Java方法的调用都使用一个栈帧来保存。

1. Java虚拟机栈主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。
2. ==Java虚拟机栈线程私有的，生命周期和线程一致==。
3. JVM 直接对虚拟机栈的操作只有两个：方法的入栈(方法的执行)与出栈(方法的结束)。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112165138710-1571653769.png" alt="image-20231112165137448" style="zoom:67%;" />

4. Java虚拟机栈如果栈帧过多，占用内存超过栈内存可以分配的最大大小就**会出现内存溢出**（内存溢出会出现`StackOverflowError`错误）。

```java
package com.zixieqing.runtime_data_area.stack;

/**
 * <p>
 * JVM内存结构：运行时数据区之Java虚拟机栈
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class JVM_Stack {

    /**
     * 计数器：得到当前系统的栈帧大小
     */
    private static int count = 0;

    public static void main(String[] args) {
        stackOverFlowTest();
    }

    /**
     * <p>
     * 1、测试Java虚拟机栈是否会内存溢出
     * </p>
     */
    public static void stackOverFlowTest() {
        System.out.println(++count);
        stackOverFlowTest();
    }
}



// 结果
10710
Exception in thread "main" java.lang.StackOverflowError
```

- **Java虚拟机栈默认大小**：如果我们不指定栈的大小，JVM 将创建一个具有默认大小的栈。大小取决于操作系统和计算机的体系结构。

```txt
Linux
		x86（64位）：1MB
		
Windows
		基于操作系统默认值
		
BSD
		x86（64位）：1MB
		
Solarls
		64位：1MB
```

- **自己设置栈大小**：使用虚拟机参数 `-Xss栈大小`

> **注：** HotSpot Java虚拟机对栈大小的最大值和最小值有要求
>
> - **Windows（64位）下的JDK8最小值为180k，最大值为1024m** 。
>
> 一般情况下，工作中栈的深度最多也只能到几百,不会出现栈的溢出。所以此参数可以手动指定为`-Xss256k`节省内存。

```txt
参数含义：设置线程的最大栈空间

语法：-Xss栈大小

单位：字节（默认，必须是 1024 的倍数）、k或者K(KB)、m或者M(MB)、g或者G(GB)。示例如下：
	-Xss1048576 
    -Xss1024K 
    -Xss1m
    -Xss1g



与 -Xss 类似，也可以使用 -XX:ThreadStackSize 调整标志来配置堆栈大小。

	格式为： -XX:ThreadStackSize=1024
```



由前面内容知道：每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的**栈帧(Stack Frame）**，这些栈帧对应着一个个方法的调用，那栈帧组成又是怎么样的？





##### 栈帧的组成

每个**栈帧**（Stack Frame）中存储着：

1. **局部变量表（Local Variables）**：局部变量表的作用是在运行过程中存放所有的局部变量。
2. **操作数栈（Operand Stack）**：操作数栈是栈帧中虚拟机在执行指令过程中用来存放临时数据的一块区域。
3. **帧数据（Frame Data）**：帧数据主要包含动态链接、方法出口、异常表的引用。

- **动态链接（Dynamic Linking）**：当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用（编号）转换成对应的运行时常量池中的内存地址。**动态链接就保存了编号（符号引用）到 运行时常量池的内存地址 的映射关系**。
- **方法返回地址 / 方法出口（Return Address）**：方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，需要存储此方法出口的地址。
- 一些附加信息：栈帧中还允许携带与 Java 虚拟机实现相关的一些附加信息，例如，对程序调试提供支持的信息，但这些信息取决于具体的虚拟机实现，所以在这里不说明这玩意儿。



###### 局部变量表（Local Variables）

> 局部变量表也被称为局部变量数组或者本地变量表。
>
> **局部变量表的作用是在方法执行过程中存放所有的局部变量**。编译成字节码文件时就可以确定局部变量表的内容。
>
> ==**局部变量表中保存的是：实例方法的this对象，方法的参数，方法体中声明的局部变量**==

1. **栈帧中的局部变量表是一个数组，数组中每一个位置称之为槽(slot)，long和double类型占用两个槽，其他类型占用一个槽。** 

> byte、short、char 在存储前被转换为int；
>
> boolean也被转换为int，0 表示 false，非 0 表示 true。



<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112205004543-2094783191.png" alt="image-20231112205003404" style="zoom:80%;" />

栈帧中的局部变量表是咋样的？数组咯，每个索引位置就是一个槽（Slot）

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112205502809-1418460258.png" alt="image-20231112205501841" style="zoom: 80%;" />



2. **如果当前帧是由构造方法或实例方法创建的，那么该对象引用 this 将会存放在 index 为 0 的 Slot 处，其余的参数按照参数表顺序继续排列**。

因为实例方法需要先拿到实例对象，即代码首行有个隐藏的this，它去搞对象了。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112210402806-1619948978.png" alt="image-20231112210400680" style="zoom:67%;" />



3. 为了节省空间，**局部变量表中的槽（Slot）是可以被复用的**。一旦某个局部变量不再生效，当前槽就可以再次被使用。

![image-20231112212102143](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112212104648-524392593.png)





###### 操作数栈（Operand Stack）

> **操作数栈：是栈帧中虚拟机在执行指令过程中用来存放中间数据的一块区域**。如果一条指令将一个值压入操作数栈，则后面的指令可以弹出并使用该值。
>
> - PS：这种栈也是先进后出。

1. **操作数栈，在方法执行过程中，根据字节码指令，往操作数栈中写入数据或提取数据，即入栈（push）、出栈（pop）**。

2. **如果被调用的方法带有返回值的话，其返回值将会被压入当前栈帧的操作数栈中**，并更新 PC 寄存器中下一条需要执行的字节码指令。
3. 所谓的**Java虚拟机的解释引擎是基于栈的执行引擎**，其中的栈指的就是操作数栈。

4. **每一个操作数栈在编译期就已经确定好了其所需的最大深度**，从而在执行时正确的分配内存大小。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112220642671-2003094921.png" alt="image-20231112220641722" style="zoom:67%;" />

那这个所需的最大深度是怎么确定的？

编译器模拟对应的字节码指令执行过程，在这个过程中最多可以存放几个数据，这个最多存放多少个数据就是操作数栈所需的最大深度。

如下图，共出现了0和1这两个数据，所以操作数栈的最大深度就是2。

![操作数栈最大深度的确定 + 操作数栈的入栈、出栈](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231112222052205-149655198.gif)

> 另一个小知识点：栈顶缓存（Top-of-stack-Cashing）

HotSpot 的执行引擎采用的并非是基于寄存器的架构，但这并不代表 HotSpot VM 的实现并没有间接利用到寄存器资源，寄存器是物理 CPU 中的组成部分之一，它同时也是 CPU 中非常重要的高速存储资源。一般来说，寄存器的读/写速度非常迅速，甚至可以比内存的读/写速度快上几十倍不止，不过寄存器资源却非常有限，不同平台下的CPU 寄存器数量是不同和不规律的。寄存器主要用于缓存本地机器指令、数值和下一条需要被执行的指令地址等数据

基于栈式架构的虚拟机所使用的零地址指令更加紧凑，但完成一项操作的时候必然需要使用更多的入栈和出栈指令，这同时也就意味着将需要更多的指令分派（instruction dispatch）次数和内存读/写次数，由于操作数是存储在内存中的，因此频繁的执行内存读/写操作必然会影响执行速度。为了解决这个问题，HotSpot JVM 设计者们提出了==**栈顶缓存技术，将栈顶元素全部缓存在物理 CPU 的寄存器中，以此降低对内存的读/写次数，提升执行引擎的执行效率**==





###### 帧数据（Frame Data）

1. **动态链接（Dynamic Linking）**：当前类的字节码指令引用了其他类的属性或者方法时，需要将符号引用（编号）转换成对应的运行时常量池中的内存地址。**动态链接就保存了编号（符号引用）到 运行时常量池的内存地址 的映射关系**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113144513254-1121270090.png" alt="image-20231113144512701" style="zoom:67%;" />

> JVM 是如何执行方法调用的？

方法调用不同于方法执行，方法调用阶段的唯一任务就是确定被调用方法的版本（即调用哪一个方法），暂时还不涉及方法内部的具体运行过程，Class 文件的编译过程中不包括传统编译器中的连接步骤，一切方法调用在 Class文件里面存储的都是**符号引用**，而不是方法在实际运行时内存布局中的入口地址（**直接引用**），也就是需要在类加载阶段，甚至到运行期才能确定目标方法的直接引用。

在 JVM 中，将符号引用转换为调用方法的直接引用与方法的绑定机制有关

- **静态链接**：当一个字节码文件被装载进 JVM 内部时，如果被调用的**目标方法在编译期可知，且运行期保持不变时**，这种情况下将调用方法的符号引用转换为直接引用的过程称之为静态链接。
- **动态链接**：如果**被调用的方法在编译期无法被确定下来**，也就是说，只能在程序运行期将调用方法的符号引用转换为直接引用，由于这种引用转换过程具备动态性，因此也就被称之为动态链接。

对应的方法的绑定机制为：早期绑定（Early Binding）和晚期绑定（Late Binding）。

**绑定是一个字段、方法或者类在符号引用被替换为直接引用的过程，这仅仅发生一次**。

- **早期绑定**：**指被调用的目标方法如果在编译期可知，且运行期保持不变时**，即可将这个方法与所属的类型进行绑定，这样一来，由于明确了被调用的目标方法究竟是哪一个，因此也就可以使用静态链接的方式将符号引用转换为直接引用。
- **晚期绑定**：如果**被调用的方法在编译器无法被确定下来**，只能够在程序运行期根据实际的类型绑定相关的方法，这种绑定方式就被称为晚期绑定。



> 虚方法与非虚方法

- **非虚方法：指的是方法在编译器就确定了具体的调用版本，这个版本在运行时是不可变的**，如静态方法、私有方法、final 方法、实例构造器、父类方法都是非虚方法。
- 其他方法称为虚方法。





2. **方法返回地址 / 方法出口（Return Address）：方法在正确或者异常结束时，当前栈帧会被弹出，同时程序计数器应该指向上一个栈帧中的下一条指令的地址。所以在当前栈帧中，存放的这条指令地址就是方法出口地址**。

> 本质上，**方法的结束就是当前栈帧出栈的过程**，此时，需要恢复上层方法的局部变量表、操作数栈、将返回值压入调用者栈帧的操作数栈、设置PC寄存器值等，让调用者方法继续执行下去

细节说明一下：

- **方法正常退出时**，调用该方法的指令的下一条指令的地址即为返回地址。

一个方法的正常调用完成之后，究竟需要使用哪一个返回指令，还需要根据方法返回值的实际数据类型而定。

| 返回类型                         | 返回指令 |
| -------------------------------- | -------- |
| void、类和接口的初始化方法       | return   |
| int (boolean、byte、char、short) | ireturn  |
| long                             | lreturn  |
| float                            | freturn  |
| double                           | dreturn  |
| reference                        | areturn  |

- **方法通过异常退出的**，返回地址是要通过异常表来确定的，栈帧中一般不会保存这部分信息。

在方法执行的过程中遇到了异常，并且这个异常没有在方法内进行处理，也就是只要在本方法的异常表中没有搜索到匹配的异常处理器，就会导致方法退出，简称**异常完成出口**。

方法执行过程中抛出异常时的异常处理，存储在一个异常处理表，方便在发生异常的时候找到处理异常的代码。



> 上述内容说到了“异常表”，也顺便解释一下这个东西

**异常表：存放的是代码中异常的处理信息，包含了异常捕获的生效范围以及异常发生后跳转到的字节码指令位置**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113161321529-1755139183.png" alt="image-20231113161321037" style="zoom: 50%;" />







#### 本地方法栈（Native Method Stack）

> Java虚拟机栈存储了Java方法调用时的栈帧，而**本地方法栈存储的是native本地方法的栈帧**。
>
> 它的具体做法是 `Native Method Stack` 中登记 native 方法，在 `Execution Engine` 执行时加载本地方法库，当某个线程调用一个本地方法时，它就进入了一个全新的并且不再受虚拟机限制的世界，它和虚拟机拥有同样的权限。
>
> ==本地方法栈也是线程私有的。也会发生内存溢出==。
>
> - 果线程请求分配的栈容量超过本地方法栈允许的最大容量，Java 虚拟机将会抛出一个 `StackOverflowError` 异常
> - 如果本地方法栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的本地方法栈，那么 Java虚拟机将会抛出一个`OutofMemoryError`异常

1. 在Hotspot虚拟机中，Java虚拟机栈和本地方法栈实现上使用了同一个栈空间，即**在 Hotspot JVM 中，直接将本地方法栈和虚拟机栈合二为一了**。

2. 本地方法栈会在栈内存上生成一个栈帧，临时保存方法的参数同时方便出现异常时也把本地方法的栈信息打印出来。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113164905429-1403218080.png" alt="image-20231113164905329" style="zoom:50%;" />



```java
package com.zixieqing.runtime_data_area.stack;

import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

/**
 * <p>
 * JVM -> 运行时数据区 -> 本地方法栈
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class NativeMethodStack {
    public static void main(String[] args) {
        try {
            // 这里并没有F盘
            FileOutputStream fos = new FileOutputStream("F:\\zixq.txt");
            fos.write(2);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}


// 结果
java.io.FileNotFoundException: F:\zixq.txt (系统找不到指定的路径。)
	at java.io.FileOutputStream.open0(Native Method)		// 这里就把Native方法信息也打印出来了
	at java.io.FileOutputStream.open(FileOutputStream.java:270)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:213)
	at java.io.FileOutputStream.<init>(FileOutputStream.java:101)
	at com.zixieqing.runtime_data_area.stack.NativeMethodStack.main(NativeMethodStack.java:20)
```

3. 本地方法可以通过本地方法接口来访问虚拟机内部的运行时数据区，它甚至可以直接使用本地处理器中的寄存器，直接从本地内存的堆中分配任意数量的内存。
4. 并不是所有 JVM 都支持本地方法，因为 Java 虚拟机规范并没有明确要求本地方法栈的使用语言、具体实现方式、数据结构等，如果 JVM 产品不打算支持 native 方法，也可以无需实现本地方法栈。







### 堆区（Heap Area）

> **栈是运行时的单位，而堆是存储的单位**。
>
> - 栈解决的是程序的运行问题，即程序如何执行，或者说如何处理数据。
> - 堆解决的是数据存储的问题，即数据怎么放、放在哪。



> 一般Java程序中堆内存是空间最大的一块内存区域。**创建出来的对象都存在于堆上。几乎所有的对象实例以及数据都在这里分配内存。**
>
> ==被所有线程共享，会发生内存溢出==。
>
> - 栈上的局部变量表中，可以存放堆上对象的引用。静态变量也可以存放堆对象的引用，通过静态变量就可以实
>
>   现对象在线程之间共享。
>
> <img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113231643195-2127233617.png" alt="image-20231113231643098" style="zoom:50%;" />
>
> - **Java 虚拟机规范规定，Java 堆可以是处于物理上不连续的内存空间中，只要逻辑上是连续的即可**。
> - 堆的大小可以是固定大小，也可以是可扩展的，主流虚拟机都是可扩展的（通过 `-Xmx` 和 `-Xms` 控制）。
> - **如果堆中没有完成实例分配，并且堆无法再扩展时，就会抛出 `OutOfMemoryError` 异常**。

**堆内存溢出模拟**：

```java
import java.util.ArrayList;

/**
 * <p>
 * 模拟堆内存溢出
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class OutOfMemoryTest {

    private static ArrayList<Object> list = new ArrayList<>();

    public static void main(String[] args) {
        while (true) {
            list.add(new byte[1024 * 1024 * 10]);
        }
    }
}


// 结果
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.zixieqing.runtime_data_area.heap.OutOfMemoryTest.main(OutOfMemoryTest.java:20)
```



为了进行高效的垃圾回收（及GC后续说明），虚拟机把堆内存**逻辑上**划分成三块区域（分代的唯一理由就是优化 GC 性能）：

![JDK7](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231113225142450-1115685021.jpg)

> 根据JDK版本不同，也可以说是两块区域，往下看就会了解。

1. **新生代 / 年轻代 （Young Generation）**：新对象和没达到一定年龄的对象都在新生代。

年轻代是所有新对象创建的地方。当填充年轻代时，执行垃圾收集，这种垃圾收集称为 **Minor GC**。

年轻代被分为三个部分——伊甸园（**Eden Memory**，伊甸园-上帝创造夏娃）和两个幸存区（**Survivor Memory**，被称为 from / to 或 s0 / s1），默认比例是`8:1:1`，这个比例可以通过 `-XX:SurvivorRatio` 来配置。

- 大多数新创建的对象都位于 Eden 内存空间中。
- 当 Eden 空间被对象填充时，执行**Minor GC**，并将所有幸存者对象移动到一个幸存者空间中。
- Minor GC 检查幸存者对象，并将它们移动到另一个幸存者空间。所以每次，一个幸存者空间总是空的。
- 经过多次 GC 循环后存活下来的对象被移动到老年代。通常，这是通过设置年轻一代对象的年龄阈值来实现的，然后他们才有资格提升到老一代。



2. **老年代 / 养老区 （Old Generation）**：被长时间使用的对象，老年代的内存空间比年轻代更大（因大部分对象都是创建出来之后就使用了，之后就不再使用了，就可以回收了。如：自己封装的对象-获取订单到返回订单就基本上不用了）。

旧的一代内存包含那些经过许多轮小型 GC 后仍然存活的对象。通常，垃圾收集是在老年代内存满时执行的。老年代垃圾收集称为 主GC（Major GC），通常需要更长的时间。

大对象直接进入老年代（大对象是指需要大量连续内存空间的对象），这样做的目的是避免在 Eden 区和两个Survivor 区之间发生大量的内存拷贝。



> 默认情况下新生代和老年代的比例是 1:2，可以通过 `–XX:NewRatio` 来配置。
>
> 
>
> 若在 JDK 7 中开启了 `-XX:+UseAdaptiveSizePolicy`，JVM 会动态调整 JVM 堆中各个区域的大小以及进入老年代的年龄,此时 `–XX:NewRatio` 和 `-XX:SurvivorRatio` 将会失效，而 JDK 8 是默认开启`-XX:+UseAdaptiveSizePolicy`。
>
> 在 JDK 8中，**不要随意关闭**`-XX:+UseAdaptiveSizePolicy`，除非对堆内存的划分有明确的规划。



3. **元空间（Meta Space） / 永久代（Permanent-Generation）**：JDK1.8前是永久代，JDK1.8及之后是元空间。JDK1.8 之前是占用 JVM 内存，JDK1.8 之后直接使用物理 / 直接内存。

不管是 JDK8 之前的**永久代**，还是 JDK8 及之后的**元空间**，**都是 Java 虚拟机规范中方法区的实现**。因此：这点内容在这里不多说明，到后面方法区再进行说明。





#### 对象在堆中的生命周期

1. 在 JVM 内存模型的堆中，堆被划分为新生代和老年代 
   - 新生代又被进一步划分为 **Eden区** 和 **Survivor区**，Survivor 区由 **From Survivor** 和 **To Survivor** 组成
2. 当创建一个对象时，对象会被优先分配到新生代的 Eden 区 
   - 此时 JVM 会给对象定义一个**对象年龄计数器**（`-XX:MaxTenuringThreshold`）
3. 当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC） 
   - JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1
   - 对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会+1
4. 如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代**





#### 对象的分配过程

> 涉及的内容对于初识JVM的人有点超纲，后续会慢慢了解。

为对象分配内存是一件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法和内存回收算法密切相关，所以还需要考虑 GC 执行完内存回收后是否会在内存空间中产生内存碎片。

1. new 的对象先放在伊甸园区，此区有大小限制。
2. 当伊甸园的空间填满时，程序又需要创建对象，JVM 的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC），将伊甸园区中不再被其他对象所引用的对象进行销毁，再加载新的对象放到伊甸园区。
3. 然后将伊甸园中的剩余对象移动到幸存者 0 区。
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者 0 区，如果没有被回收，就会放到幸存者 1 区。
5. 如果再次经历垃圾回收，此时会重新放回幸存者 0 区，接着再去幸存者 1 区。
6. 什么时候才会去养老区呢？ 默认是 15 次回收标记。
7. 在养老区，相对悠闲，当养老区内存不足时，触发 Major GC，进行养老区的内存清理。
8. 若养老区执行了 Major GC 之后发现依然无法进行对象的保存，就会产生 OOM 异常。





#### 堆内存三个值：used、total、max

> **used**：指的是**当前已使用的堆内存**；
>
> **total**：指的是Java虚拟机**已经分配的可用堆内存**；
>
> **max**：指的是Java虚拟机可以**分配的最大堆内存**。
>
> ![image-20231114202838913](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114202840739-1985125440.png)





#### Arthas中堆内存相关的功能

1. **堆内存used total max三个值可以通过dashboard命令看到**。

```txt
手动指定刷新频率（不指定默认5秒一次）：dashboard –i 刷新频率(毫秒)

要是只想看内存栏的数据，可以直接使用指令：memory
```



```java
package com.zixieqing.runtime_data_area.heap;

import java.io.IOException;
import java.util.ArrayList;

/**
 * <p>
 * Arthas中堆内存的相关功能
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class Arthas_Heap {

    public static void main(String[] args) throws IOException, InterruptedException {

        ArrayList<Object> list = new ArrayList<>();
        
        System.in.read();

        while (true) {
            list.add(new byte[1024 * 1024 * 100]);
        }
    }
}
```



<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114203714413-1108140369.png" alt="image-20231114203712309" style="zoom:67%;" />



2. **随着堆中的对象增多，当total可以使用的内存即将不足时，Java虚拟机会继续分配内存给堆**。

![image-20231114204015965](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114204016590-1777970374.png)

3. **如果堆内存不足，Java虚拟机就会不断的分配内存，total值会变大。==total最多只能与max相等==**。

![image-20231114204203055](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114204203609-948403832.png)



> 问题：是不是当used = max = total的时候，堆内存就溢出了？

**不是**，堆内存溢出的判断条件比较复杂（具体内容需要到后续的垃圾回收相关内容去了解）。





#### 设置堆内存大小

> **如果不设置任何的虚拟机参数，max默认是系统内存的1/4，total默认是系统内存的1/64**。==在实际应用中一般都需要设置total和max的值==
>
> Oracle官方文档：https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html
>
> **要修改堆内存大小，可以使用虚拟机参数 -Xms (初始的total) 和 –Xmx（max最大值）**。

Java 堆用于存储 Java 对象实例，那么堆的大小在 JVM 启动的时候就确定了，我们可以通过 `-Xmx` 和 `-Xms` 来设定

- `-Xms` 用来表示堆的起始内存，等价于 `-XX:InitialHeapSize`
- `-Xmx` 用来表示堆的最大内存，等价于 `-XX:MaxHeapSize`

> **在工作中，这两个值一般配置的都是相同的值**
>
> - 好处：这样在程序启动之后可使用的总内存就是最大内存，而无需向java虚拟机再次申请，减少了申请并分配内存时间上的开销，同时也不会出现内存过剩之后堆收缩的情况。

PS：如果堆的内存大小超过 `-Xmx` 设定的最大内存， 就会抛出 `OutOfMemoryError` 异常

```txt
语法：-Xmx值 -Xms值

单位：字节（默认，必须是 1024 的倍数）、k或者K(KB)、m或者M(MB)、g或者G(GB)

限制：Xmx必须大于 2 MB，Xms必须大于1MB

示例：
-Xms6291456
-Xms6144k
-Xms6m
-Xmx83886080
-Xmx81920k
-Xmx80m
```

可以通过代码获取到我们的设置值：

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



> 拓展点：为什么Arthas中显示的heap堆大小与设置的值不一样？
>
> PS：测试自行在IDEA中设置上面内容的Java虚拟机参数，然后启动Arthas，不断添加对象，在Arthas中输入`memory`参数进行对比查看。

```bash
-Xms1g -Xmx1g
```

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231114211409212-284352653.png" alt="image-20231114211407634" style="zoom:67%;" />

原因：**arthas中**的heap堆内存使用了JMX技术的内存获取方式，这种方式与垃圾回收器有关，**计算的是可以分配对象的内存**，而不是整个内存。

即：虽然设置了多少内存，但有一点内存是不会用到的，也就是JMX技术会把这部分内存去掉，不申请这部分内存，因为这点内存也放不下一个新对象，因此申请了也是浪费。







### 方法区（Method Area）

> 方法区只是 JVM 规范中定义的一个概念。并没有规定如何去实现它，不同的厂商有不同的实现。
>
> **永久代（PermGen）是 Hotspot 虚拟机特有的概念， Java8 的时候被 元空间**取代了，永久代和元空间方法区的落地实现方式。
>
> ==方法区是线程共享的，并且也有内存溢出==
>
> - PS：如果方法区域中的内存不能用于满足分配请求，则 Java 虚拟机抛出 `OutOfMemoryError`。



HotSpot虚拟机中：

- 永久代是HopSpot虚拟机中才有的概念。
- JDK1.6及之前，方法区的实现方式是永久代，是在堆区中（运行时常量池[里面的逻辑包含了字符串常量池]、静态变量就存储在这个永久代中）
- JDK1.7，方法区的实现方式还是永久代，也还在堆区中，但逐步“去永久代”，将字符串常量池、静态变量放到了堆区中（java.lang.Class对象）
- JDK1.8及之后，取消永久代，方法区的实现方式变为了堆+元空间，元空间位于操作系统维护的直接内存中，默认情况下只要不超过操作系统承受的上限就可以一直分配内存。
  - PS：此时，类型信息、字段、方法、常量保存在元空间中，字符串常量池、静态变量还存储在堆区中（java.lang.Class对象）





#### Java与直接内存

> 直接内存（Direct Memory）并不在《Java虚拟机规范》中存在，所以并不属于Java运行时的内存区域。

在 JDK 1.4 中引入了 NIO 机制，使用了直接内存，主要为了解决以下两个问题：

1. Java堆中的对象如果不再使用要回收，回收时会影响对象的创建和使用。
2. 以前，IO操作比如读文件，需要先把文件读入直接内存（缓冲区）再把数据复制到Java堆中。现在，直接放入直接内存即可，同时Java堆上维护直接内存的引用，减少了数据复制的开销。写文件也是类似的思路。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117222445137-274627799.png" alt="image-20231117222444427" style="zoom:67%;" />



> **要创建直接内存上的数据，可以使用`java.nio.ByteBuffer`。** 
>
> 语法：`ByteBuffer directBuffer = ByteBuffer.allocateDirect(size);`
>
> 注意：也会抛`OutOfMemoryError`。

```java
package com.zixieqing.runtime_data_area;

import java.io.IOException;
import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.List;

/**
 * <p>
 * Java使用直接内存Direct Memory
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class DirectMemory {

    /**
     * SIZE = 100mb
     */
    private static int SIZE = 1024 * 1025 * 100;

    private static List<ByteBuffer> LIST = new ArrayList<>();

    private static int COUNT = 0;

    public static void main(String[] args) throws IOException, InterruptedException {

        System.in.read();

        while (true) {
            ByteBuffer byteBuffer = ByteBuffer.allocateDirect(SIZE);
            LIST.add(byteBuffer);
            System.out.println("COUNT = " + (++COUNT));
            Thread.sleep(5000);
        }
    }
}
```



> Arthas的memory命令可以查看直接内存大小，属性名direct。

![image-20231117225342871](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117225344541-1658102154.png)

> 如果需要手动调整直接内存的大小，可以使用 -XX:MaxDirectMemorySize=大小

单位k或K表示千字节，m或M表示兆字节，g或G表示千兆字节。默认不设置该参数情况下，JVM 自动选择最大分配的大小。示例如下：

```bash
-XX:MaxDirectMemorySize=1m
-XX:MaxDirectMemorySize=1024k
-XX:MaxDirectMemorySize=1048576
```





#### 方法区的内部结构

> 说的是HotSpot虚拟机，且是JDK1.8及之后。

主要包含三部分内容：

1. **类的元信息**：保存所有类的基本信息。一般称之为**InstanceKlass对象。**在类的加载阶段完成。这一点就是前面一开始玩的字节码文件内容

![image-20231117170921007](https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117170922195-650785069.png)

2. **运行时常量池**：字节码文件中通过编号查表的方式找到常量，这种常量池称为静态常量池。当常量池加载到内存中之后，可以通过内存地址快速的定位到常量池中的内容，这种常量池称为运行时常量池。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117214933506-211615087.png" alt="image-20231117214931503" style="zoom:80%;" />

常量池表（Constant Pool Table）是 Class 文件的一部分，用于存储编译期生成的各种字面量和符号引用，**这部分内容将在类加载后存放到方法区的运行时常量池中**。

> 为什么需要常量池？
>
> - 一个 Java 源文件中的类、接口，编译后产生一个字节码文件。而 Java 中的字节码需要数据支持，通常这种数据会很大以至于不能直接存到字节码里，换另一种方式，可以存到常量池，这个字节码包含了指向常量池的引用。在动态链接的时候用到的就是运行时常量池



3. **字符串常量池（**StringTable**）**：JDK7后，在堆中，存储在代码中定义的常量字符串内容。比如“123” 这个123就会被放入字符串常量池。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117215324003-945968567.png" alt="image-20231117215323048" style="zoom:67%;" />

> JDK6中，`String.intern()` 方法会把第一次遇到的字符串实例复制到永久代的字符串常量池中，返回的也是永久代里面这个字符串实例的引用。
>
> JDK7及之后中，由于字符串常量池在堆上，所以 `String.intern()` 方法会把第一次遇到的字符串的引用放入字符串常量池。

如下图：上为JDK6的结果，下为JDK7及之后中的结果

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117221921037-171634504.png" alt="image-20231117221920037" style="zoom:67%;" />

早期设计时（JDK1.6及之前），字符串常量池是属于运行时常量池的一部分，他们存储的位置也是一致的（堆中的永久代空间），后续做出了调整，大意图如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231117215459171-718250975.png" alt="image-20231117215458355" style="zoom:67%;" />





#### 设置方法区大小

1. JDK1.7及之前，通过下列参数设置永久代空间大小。

```txt
-XX:PermSize=值 和 -xx:MaxPermSize=值
```

2. JDK1.8及之后，通过下列参数设置元空间大小。

```txt
-XX:MetaspaceSize=值 和 -XX:MaxMetaspaceSize=值
```

- 默认值依赖于平台。Windows 下，`-XX:MetaspaceSize` 是 21M，`-XX:MaxMetaspacaSize` 的值是 -1，即没有限制。
- 与永久代不同，如果不指定大小，默认情况下，虚拟机会耗尽所有的可用系统内存。如果元数据发生溢出，虚拟机一样会抛出异常 `OutOfMemoryError:Metaspace`。
- `-XX:MetaspaceSize` ：设置初始的元空间大小。对于一个 64 位的服务器端 JVM 来说，其默认的 `-XX:MetaspaceSize` 的值为20.75MB，这就是初始的高水位线，一旦触及这个水位线，Full GC 将会被触发并卸载没用的类（即这些类对应的类加载器不再存活），然后这个高水位线将会重置，新的高水位线的值取决于 GC 后释放了多少元空间。如果释放的空间不足，那么在不超过 `MaxMetaspaceSize`时，适当提高该值。如果释放空间过多，则适当降低该值。
- 如果初始化的高水位线设置过低，上述高水位线调整情况会发生很多次，通过垃圾回收的日志可观察到 Full GC 多次调用。为了避免频繁 GC，建议将 `-XX:MetaspaceSize` 设置为一个相对较高的值。









## 执行引擎

> 执行引擎：执行本地已经编译好的方法，如虚拟机提供的C++方法。
>
> 包括：即时编译器（JIT，即Just-in-time）、解释器、垃圾回收器等。



<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231119165759216-474366881.png" alt="image-20231110204142543" style="zoom:80%;" />

### 自动垃圾回收

> 在C/C++这类没有自动垃圾回收机制的语言中，一个对象如果不再使用，需要手动释放，否则就会出现内存泄漏。我们称这种释放对象的过程为垃圾回收，而需要程序员编写代码进行回收的方式为手动回收。 
>
> - PS：内存泄漏指的是不再使用的对象在系统中未被回收，内存泄漏的积累可能会导致内存溢出。

Java中为了简化对象的释放，引入了自动垃圾回收（Garbage Collection简称GC）机制。**虚拟机通过垃圾回收器来对不再使用的对象完成自动的回收**，**垃圾回收器主要负责对堆上的内存进行回收**。

<img src="https://img2023.cnblogs.com/blog/2421736/202311/2421736-20231119170950627-1731223678.png" alt="image-20231119170949432" style="zoom:80%;" />

> 自动垃圾回收与手动垃圾回收的优缺点。

1. **自动垃圾回收**：自动根据对象是否使用由虚拟机来回收对象。

- 优点：降低程序员实现难度、降低对象回收bug的可能性。
- 缺点：程序员无法控制内存回收的及时性。

2. **手动垃圾回收**：由程序员编程实现对象的删除。

- 优点：回收及时性高，由程序员把控回收的时机。
- 缺点：编写不当容易出现悬空指针、重复释放、内存泄漏等问题。





### 方法区的回收

> **线程不共享的部分（程序计数器、Java虚拟机栈、本地方法栈），都是伴随着线程的创建而创建，线程的销毁而销毁**。而方法的栈帧在执行完方法之后就会自动弹出栈并释放掉对应的内存，因此不需要对这部分区域进行垃圾回收。
>
> 
>
> **由前面对方法区的了解可知：方法区中能回收的内容主要就是常量池中废弃的常量和不再使用的类型**。
>
> 开发中此类场景一般很少出现，主要在如 OSGi、JSP 的热部署等应用场景中。每个jsp文件对应一个唯一的类加载器，当一个jsp文件修改了，就直接卸载这个jsp类加载器。重新创建类加载器，重新加载jsp文件。

在前面类的生命周期中，最后一步是卸载（unloading），而判定一个类可以被卸载。需要**同时满足**下面三个条件：

> 可以使用虚拟机参数 `-verbose:class` 或 `-XX:+TraceClassLoading` 、`-XX:+TraceClassUnloading` 查看类加载和卸载信息。
>
> `-XnoClassgc` 参数可以关闭类的GC / 控制是否对类进行卸载。在垃圾收集时类对象不会被回收，会被认为总是存活的，这将导致存放类对象的内存被持续占用，如果不谨慎使用，将可能导致OOM。

1. **此类所有实例对象都已经被回收**。在堆中不存在任何该类的实例对象以及子类对象。 

2. **加载该类的类加载器已经被回收**。

3. **该类对应的 `java.lang.Class` 对象没有在任何地方被引用**。

```java
package com.zixieqing.execution_engine.method_area_collection;

import com.zixieqing.class_and_classloader.FindClassLoader;

/**
 * <p>
 * 卸载（unloading）类“同时满足”的三个条件
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class UnloadingConditions {

    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, InterruptedException {

        ClassLoader classLoader = FindClassLoader.class.getClassLoader();
        Class<?> clazz = classLoader.loadClass("com.zixieqing.class_and_classloader.FindClassLoader");
        Object obj = clazz.newInstance();
        // 1、该类所有实例对象都被回收   反例证明：将obj这个对象放到其他地方去，让其被持有则不会不会被卸载了
        obj = null;

        // 2、加载该类的类加载器已经被回收     反例证明：将此类加载器其他持有就不会被卸载
        classLoader = null;

        // 3、该类对应的 java.lang.Class 对象没有在任何地方被引用     反例证明：该对象被其他持有
        clazz = null;

        // 手动触发垃圾回收
        // 不一定会立即回收垃圾，仅仅是向Java虚拟机发送一个垃圾回收的请求，具体是否需要执行垃圾回收Java虚拟机会自行判断
        System.gc();
    }
}
```



> 补充：关于 finalize() 方法

finalize() 类似 C++ 的析构函数，用来做关闭外部资源等工作。但是 try-finally 等方式可以做的更好，并且该方法运行代价高昂，不确定性大，无法保证各个对象的调用顺序，因此最好不要使用。

当一个对象可被回收时，如果需要执行该对象的 finalize() 方法，那么就有可能通过在该方法中让对象重新被引用，从而实现自救，自救只能进行一次，如果回收的对象之前调用了 finalize() 方法自救，后面回收时不会调用 finalize() 方法。





### 堆回收

#### 如何判断堆上的对象是否可被回收？

> Java中的对象是否能被回收，是根据对象是否被引用来决定的。如果对象被引用了，说明该对象还在使用，不允许被回收。

![image-20231210163809458](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210163812081-1703581880.png)

而判断对象是否有引用有两种判断方法：引用计数法和可达性分析法。



##### 引用计数法

> 引用计数法会为每个对象维护一个引用计数器，当对象被引用时加1，取消引用（或引用失效）时减1，引用计数为 0 的对象可被回收。

引用计数法的优点是实现简单，C++中的智能指针就采用了引用计数法，但是它也存在缺点，主要有两点：

1. 每次引用和取消引用都需要维护计数器，对系统性能会有一定的影响。
2. 存在循环引用问题，此时引用计数器永远不为 0，导致无法对它们进行回收。所谓循环引用就是当A引用B，B同时引用A时会出现对象无法回收的问题。

![image-20231210171559057](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210171600079-1092784022.png)



##### 可达性分析法

> 可达性分析将对象分为两类：垃圾回收的根对象（GC Root）和普通对象，对象与对象之间存在引用关系。通过 GC Roots 作为起始点进行搜索，能够到达到的对象都是存活的，不可达的对象可被回收。
>
> **Java 虚拟机使用的是可达性分析算法来判断对象是否可以被回收。**
>
> ==注：可达性算法中描述的对象引用，一般指的是强引用（另外几种引用后续会说明）。==即是GCRoot对象对普通对象有引用关系，只要这层关系存在，普通对象就不会被回收。

![image](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210172002410-6587695.png)

**哪些对象被称之为GC Root对象？**

1. 线程Thread对象，引用线程栈帧中的方法参数、局部变量等。
2. 系统类加载器加载的java.lang.Class对象，引用类中的静态变量。

![image-20231210172758467](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210172759222-1322912237.png)

3. 监视器对象，用来保存同步锁synchronized关键字持有的对象。

![image-20231210172859422](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210172900410-1547384601.png)

4. 本地方法调用时使用的全局对象。





**在 Java 中 GC Roots 一般包含哪些内容？**

1. 虚拟机栈中引用的对象。
2. 本地方法栈中引用的对象。
3. 方法区中类静态属性引用的对象。
4. 方法区中的常量引用的对象。





##### 引用类型

> 无论是通过引用计算算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。共有5种引用类型（由强到弱）：强引用、软引用、弱引用、虚引用、终结器引用



###### 强引用

> 强引用：指的是GCRoot对象对普通对象有引用关系，即由可达性分析法判断，只要这层关系存在，普通对象就不会被回收。

创建强引用的方式：“可以使用” new 的方式来创建强引用。PS：别钻牛角尖，来个 `new SoftReference<Object>(obj);`

```java
Object obj = new Object();
```



###### 软引用

> 软引用相对于强引用是一种比较弱的引用关系。
>
> 如果一个对象只有软引用关联到它，则当程序内存不足时，就会将此软引用中的数据进行回收。

在JDK 1.2版之后提供了SoftReference类来实现软引用，软引用常用于缓存中。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

**软引用的执行过程如下：**

1. 将对象使用软引用包装起来，**`new SoftReference<对象类型>(对象)`**。
2. 内存不足时，虚拟机尝试进行垃圾回收。
3. 如果垃圾回收仍不能解决内存不足的问题，回收软引用中的对象。
4. 如果依然内存不足，抛出OutOfMemory异常。



> 问题：软引用中的对象如果在内存不足时回收，SoftReference对象本身也需要被回收。如何知道哪些SoftReference对象需要回收？

SoftReference提供了一套队列机制：

1. 软引用创建时，通过构造器传入引用队列。
2. 在软引用中包含的对象被回收时，该软引用对象会被放入引用队列。
3. 通过代码遍历引用队列，将SoftReference的强引用删除。

![image-20231210215049848](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210215051853-1875548629.png)

软引用应用场景：缓存示例

![image-20231210215151456](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231210215152458-1613924896.png)





###### 弱引用

> 弱引用的整体机制和软引用基本一致，区别在于弱引用包含的对象在垃圾回收时，不管内存够不够都会直接被回收。**即：被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。**

JDK 1.2版之后提供了WeakReference类来实现弱引用，弱引用主要在ThreadLocal中使用。

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```





###### 虚引用和终结器引用

> ==这两种引用在常规开发中是不会使用的。==

1. **虚引用**：也叫幽灵引用/幻影引用，不能通过虚引用对象获取到包含的对象。虚引用唯一的用途是当对象被垃圾回收器回收时可以接收到对应的通知。Java中使用PhantomReference实现了虚引用，

直接内存中为了及时知道直接内存对象不再使用，从而回收内存，使用了虚引用来实现。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```



2. **终结器引用**：指的是在对象需要被回收时，终结器引用会关联对象并放置在Finalizer类中的引用队列中，在稍后由一条FinalizerThread线程从队列中获取对象，然后执行对象的finalize方法，在对象第二次被回收时，该对象才真正的被回收。在这个过程中可以在finalize方法中再将自身对象使用强引用关联上，但是不建议这样做。







#### 垃圾回收算法

> 垃圾回收算法核心思想：
>
> 1. 找到内存中存活的对象。
> 2. 释放不再存活对象的内存，使得程序能再次利用这部分空间。



##### 判断GC算法是否优秀的标准

> Java垃圾回收过程会通过单独的GC线程来完成，但是不管使用哪一种GC算法，都会有部分阶段需要停止所有的用户线程。这个过程被称之为Stop The World简称STW，如果STW时间过长则会影响用户的使用。

1. **吞吐量**

吞吐量指的是 CPU 用于执行用户代码的时间与 CPU 总执行时间的比值，即：吞吐量 = 执行用户代码时间 /（执行用户代码时间 + GC时间）。吞吐量数值越高，垃圾回收的效率就越高。

2. **最大暂停时间**

最大暂停时间指的是所有在垃圾回收过程中的STW时间最大值。比如如下的图中，黄色部分的STW就是最大暂停时间，显而易见上面的图比下面的图拥有更少的最大暂停时间。最大暂停时间越短，用户使用系统时受到的影响就越短。

如下图就是上优下劣：

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211213638104-737794451.png" alt="image-20231211213636743" style="zoom:67%;" />

3. **堆使用效率**

不同垃圾回收算法，对堆内存的使用方式是不同的。比如标记清除算法，可以使用完整的堆内存。而复制算法会将堆内存一分为二，每次只能使用一半内存。从堆使用效率上来说，标记清除算法要优于复制算法。



> 三种评价标准：**堆使用效率、吞吐量，以及最大暂停时间不可兼得**。
>
> 一般来说，堆内存越大，最大暂停时间就越长。想要减少最大暂停时间，就会降低吞吐量。
>
> **不同的垃圾回收算法，适用于不同的场景**。





##### GC算法：标记-清除算法

> 一句话概括就是：标记存活对象，删除未标记对象。
>
> ==标记-清除算法可以使用完整的堆内存==。



![image-20231211214713125](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211214713898-653683505.png)



标记清除算法的核心思想分为两个阶段：

1. 标记阶段：将所有存活的对象进行标记。Java中使用可达性分析算法，从GC Root开始通过引用链遍历出所有存活对象。

2. 清除阶段：从内存中删除没有被标记（也就是非存活）对象。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211214316392-219611098.png" alt="image-20231211214315968" style="zoom:67%;" />

优点：实现简单，只需要在第一阶段给每个对象维护标志位，第二阶段删除对象即可。

缺点：

1. 会产生内存碎片化问题。

由于内存是连续的，所以在对象被删除之后，内存中会出现很多细小的可用内存单元。如果我们需要的是一个比较大的空间，很有可能这些内存单元的大小过小无法进行分配。

2. 分配速度慢。

由于内存碎片的存在，需要维护一个空闲链表，极有可能发生每次需要遍历到链表的最后才能获得合适的内存空间。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211214658988-1178393094.png" alt="image-20231211214658763" style="zoom:67%;" />



##### GC算法：标记-整理算法

> 一句话概括就是：让所有存活的对象都向堆内存的一端移动，然后直接清理掉“端边界以外”的内存。
>
> 标记整理算法也叫标记压缩算法，是对标记清理算法中容易产生内存碎片问题的一种解决方案。
>
> ==标记-整理算法可以使用整个堆内存==。



![image-20231211215213871](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211215213945-1314935861.png)

核心思想分为两个阶段：

1. 标记阶段：将所有存活的对象进行标记。Java中使用可达性分析算法，从GC Root开始通过引用链遍历出所有存活对象。

2. 整理阶段：将存活对象移动到堆的一端。清理掉非存活对象的内存空间。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211215325857-1121448194.png" alt="image-20231211215325960" style="zoom:67%;" />

优点：

1. 内存使用率高：整个堆内存都可以使用。
2. 不会产生内存碎片化问题：在整理阶段可以将对象往内存的一侧进行移动，剩下的空间都是可以分配对象的有效空间。

缺点：

1. 整理阶段的效率不高：因为要去找存活和非存活对象，然后进行相应内存位置移动，这里又涉及对象引用问题，所以会造成整体性能不佳。

如：Lisp2整理算法就需要对整个堆中的对象搜索3词。当然也有优化整理阶段的算法，如Two-Finger、表格算法、ImmixFC等高效的整理算法来提升此阶段性能。





##### GC算法：复制算法

> ==复制算法每次只能使用一半堆内存==。



![image-20231211220534975](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211220535483-1449601383.png)

复制算法的核心思想：

1. 准备两块空间From空间和To空间，每次在对象分配阶段，只能使用其中一块空间（From空间）。
2. 在垃圾回收GC阶段，将From中存活对象复制到To空间。
3. 将两块空间的From和To名字互换。



优点：

1. 吞吐量高：复制算法只需要遍历一次存活对象复制到To空间即可。比标记-整理算法少了一次遍历过程，因而性能较好，但不如标记-清除算法，因标记-清除算法不需要进行对象的移动。
2. 不会发生内存碎片化问题：复制算法在复制之后就会将对象按照顺序放入To内存，所以对象以外的区域是可用空间，因此不会产生内存碎片化问题。

缺点：

1. 内存使用率低：每次只能让一半的内存空间来供创建对象使用。





##### GC算法：分代收集算法

> 主流的JVM（如：HotSpot）采用的就是此种算法。
>
> 一般将堆分为新生代和老年代：
>
> - PS：新生代分为伊甸园（Eden）区、两个幸存（Survivor ）区——被称为 from / to 或 s0 / s1。默认比例是8:1:1。
> - 新生代使用: 复制算法
> - 老年代使用: 标记 - 清除 或者 标记 - 整理 算法



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231211222724175-209132338.png" alt="image-20231211222722551" style="zoom:50%;" />

1. 分代回收时，创建出来的对象，首先会被放入Eden伊甸园区。
2. 随着对象在Eden区越来越多，如果Eden区满，新创建的对象已经无法放入，就会触发年轻代的GC，称为Minor GC或者Young GC。
3. Minor GC会把需要eden中和From需要回收的对象回收，把没有回收的对象放入To区。
4. 接下来，S0会变成To区，S1变成From区。当eden区满时再往里放入对象，依然会发生Minor GC。
5. 此时会回收eden区和S1(from)中的对象，并把eden和from区中剩余的对象放入S0。

> 注意：每次Minor GC中都会为对象记录他的年龄（或者叫回收标记次数），默认值为0（默认值和垃圾回收器有关），每次GC完加1。JVM中此值最大为15。

6. 如果Minor GC后对象的年龄达到阈值（最大值为15），对象就会被晋升至老年代。
7. 当老年代中空间不足，无法放入新的对象时，先尝试minor gc，如果还是不足，就会触发Full GC。

> 问题：为什么老年代空间不足，需要先尝试minor gc，即年轻代回收？

因为第6步中年轻代中的对象不是一定是年龄达到15才会进入老年代。年轻代空间满了，此时有些对象年龄可能是小于15的，但为了腾出可用空间，这部分对象也可能会被丢进老年代。

8. 如果Full GC依然无法回收掉老年代的对象，那么当对象继续放入老年代时，就会抛出Out Of Memory异常。



> 关于伊甸园（Eden）区有一个小知识点：TLAB （Thread Local Allocation Buffer）

- 从内存模型而不是垃圾回收的角度，对 Eden 区域继续进行划分，JVM 为每个线程分配了一个私有缓存区域，它包含在 Eden 空间内。
- 多线程同时分配内存时，使用 TLAB 可以避免一系列的非线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**。
- OpenJDK 衍生出来的 JVM 大都提供了 TLAB 设计。



> 为什么要有 TLAB ?

- 堆区是线程共享的，任何线程都可以访问到堆区中的共享数据。
- 由于对象实例的创建在 JVM 中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的。
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度。

尽管不是所有的对象实例都能够在 TLAB 中成功分配内存，但 JVM 确实是将 TLAB 作为内存分配的首选。

在程序中，可以通过 `-XX:UseTLAB` 设置是否开启 TLAB 空间。

默认情况下，TLAB 空间的内存非常小，仅占有整个 Eden 空间的 1%，我们可以通过 `-XX:TLABWasteTargetPercent` 设置 TLAB 空间所占用 Eden 空间的百分比大小。

一旦对象在 TLAB 空间分配内存失败时，JVM 就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在 Eden 空间中分配内存。





> Arthas查看分代之后的内存情况

1. 在JDK8中，添加 `-XX:+UseSerialGC` 参数使用分代回收的垃圾回收器，运行程序。
2. 使用 `memory` 命令查看内存，显示出三个区域的内存情况。

测试时需要的JVM参数参考：JDK 1.8(版本不同有些参数会无效) + 添加 `-XX:+UseSerialGC` 参数

| 参数名                                      | 参数定义                                                     | 示例                                                         |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| -Xms                                        | 设置堆的最小 / 初识 大小（相当于前面说的total）。<br />必须是1024的倍数且大于1MB。 | 设置为6MB的写法：<br />-Xms6291456<br />-Xms6144k<br />-Xms6m |
| -Xmx                                        | 设置最大堆的大小（相当于前面说的max）。<br />必须是1024倍数且大于2MB。 | 设置为80 MB的写法：<br />-Xmx83886080<br />-Xmx81920k<br />-Xmx80m |
| -Xmn                                        | 新生代的大小                                                 | 设置256 MB的写法：<br />-Xmn256m<br />-Xmn262144k<br />-Xmn268435456 |
| -XX:SurvivorRatio                           | 伊甸园区和幸存区的比例，默认为8。<br />如：新生代1g内存，伊甸园区800MB,S0和S1各100MB | 比例调整为4的写法：<br />-XX:SurvivorRatio=4                 |
| -XX:+PrintGCDetails<br />或<br />verbose:gc | 打印GC日志                                                   |                                                              |





#### 垃圾回收器

> 垃圾回收器（Garbage collector，即GC）是垃圾回收算法的具体实现。
>
> **除G1之外**，**其他**垃圾回收器**必须成对组合**进行使用（如下图）。
>
> 年轻代回收都是复制算法（包括G1），老年代回收的算法不同。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212221114589-1260192870.png" alt="image-20231212221113679" style="zoom:67%;" />



> 记忆方式：
>
> 1. JDK8及之前
>
> - 关注暂停时间：ParNew + CMS（CMS在使用中需测试，因CMS在回收老年代时可能会影响用户线程）。
> - 关注吞吐量：Parallel Scavenge + Parallel Old（此组合为JDK8默认）。
> - ~~较大堆且关注暂停时间（JDK8之前不建议用）：G1~~。PS：JDK8最新版算是成熟的G1，故其实可以直接使用。
>
> 2. JDK9之后：G1（默认）。生产环境中也建议使用。。





##### 垃圾回收器：Serial 与 Serial Old

###### 新生代：Serial

> Serial是一种**单线程串行回收**年轻代的垃圾回收器，采用的是**复制算法**。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程（即STW）。
>
> **适用场景**：Java编写的客户端程序 或者 硬件配置有限（服务器不多）的场景。或者直接说 Client 模式下的场景。
>
> - PS：Serial 收集器收集几十兆甚至一两百兆的新生代停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿是可以接受的。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212215638312-1741193841.png" alt="image-20231212215637335" style="zoom:67%;" />

**优点**：单CPU处理器下吞吐量非常出色。

**缺点**：多CPU下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长时间的等待。





###### 老年代：Serial Old

> Serial Old是Serial垃圾回收器的老年代版本，**单线程串行回收**，采用的是**标记-整理算法**。
>
> 开启的方式：使用虚拟机参数 `-XX:+UseSerialGC` 即：新生代、老年代都使用串行回收器。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：与Serial垃圾回收器搭配使用 或者 在CMS特殊情况下使用（CMS时会说明）。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212220704050-431305960.png" alt="image-20231212220703204" style="zoom:67%;" />

优缺点和Serial一样。

**优点**：单CPU处理器下吞吐量非常出色。

**缺点**：多CPU下吞吐量不如其他垃圾回收器，堆如果偏大会让用户线程处于长时间的等待。





##### 垃圾回收器：ParNew 与 CMS

###### 新生代：ParNew

> ParNew垃圾回收器本质上是对Serial在多CPU下的优化，但：JDK9之后不建议使用了。
>
> 使用**多线程进行垃圾回收**，采用的是**复制算法**。
>
> - PS：默认开启的线程数量与 CPU 数量相同，可以使用 `-XX:ParallelGCThreads` 参数来设置线程数。
>
> 开启方式：使用参数 `-XX:+UseParNewGC` 即：新生代使用ParNew回收器， 老年代使用串行回收器（即Serial Old）。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：JDK8及之前的版本中，与老年代垃圾回收器CMS搭配使用。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212221645408-144737065.png" alt="image-20231212221644577" style="zoom:67%;" />

**优点**：多CPU处理器下停顿时间较短。

**缺点**：吞吐量和停顿时间不如G1，所以在JDK9之后不建议使用。



###### 老年代：CMS(Concurrent Mark Sweep)

> CMS垃圾回收器关注的是系统的暂停时间。
>
> **允许用户线程和垃圾回收线程在某些步骤中同时执行**，减少了用户线程的等待时间。采用的是**标记-清除算法**。
>
> 开启方式：使用参数 `XX:+UseConcMarkSweepGC`。
>
> 初识标记 与 重新标记阶段 会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：大型的互联网系统中用户请求数据量大、频率高的场景。如订单接口、商品接口等。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212222406985-474363136.png" alt="image-20231212222405963" style="zoom:80%;" />

**CMS执行步骤**：

1. 初始标记（Initial Mark）：用极短的时间标记出GC Roots能直接关联到的对象（可达性分析法）。 
2. 并发标记（Concurrent Mark）：标记所有的对象，用户线程不需要暂停。

这里采用了一个并发标记算法，学名叫三色标记法，G1垃圾回收器也采用的是这个算法，所以G1时再说明这个算法。

并发阶段运行时的线程数可以通过参数 `-XX:ConcGCThreads` （默认值为0）设置，由系统计算得出。即：CMS存在的线程资源争抢问题的解决方式。

```txt
计算公式为：
		(-XX:ParallelGCThreads定义的线程数 + 3) / 4


ParallelGCThreads		是STW停顿之后的并行线程数
	
	ParallelGCThreads是由处理器核数决定的：
		1、当cpu核数小于8时，ParallelGCThreads = CPU核数
		2、否则 ParallelGCThreads = 8 + (CPU核数 – 8 ) * 5 / 8
```

3. 重新标记（Remark）：由于并发标记阶段有些对象会发生变化，故存在错标、漏标等情况，因此需要重新标记。
4. 并发清理（Clean）：清理非存活对象，用户线程不需要暂停。





> CMS存在的问题

缺点：

1. CMS使用了标记-清除算法，在垃圾收集结束之后会出现大量的内存碎片，CMS会在Full GC时进行碎片的整理。这样会导致用户线程暂停，可以使用参数 ` -XX:CMSFullGCsBeforeCompaction=N` （默认0）调整N次Full GC之后再整理。
2. 无法处理在并发清理过程中产生的“浮动垃圾”，不能做到完全的垃圾回收。

“浮动垃圾”的原因：并发清理时用户线程会产生一些很快就使用、后续不用的对象，而这些对象在这次的GC中无法回收，只能等到下次GC回收，这部分垃圾就是“浮动垃圾”。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212224114901-1389920984.png" alt="image-20231212224114153" style="zoom:67%;" />

3. 如果老年代内存不足无法分配对象，CMS就会退化成Serial Old单线程回收老年代（即：前面说的特殊情况会调用Serial Old）。









##### 垃圾回收器：Parallel Scavenge 与 Parallel Old

###### 新生代：Parallel Scavenge（PS）

> **Parallel Scavenge是JDK8默认的年轻代垃圾回收器**。
>
> **多线程并行回收**，关注的是系统的吞吐量。具备**自动调整堆内存大小**的特点，采用的是**复制算法**。
>
> - PS：自动调整堆内存——即不需要手动指定新生代的大小(`-Xmn`)、Eden 和 Survivor 区的比例（`-XX:SurvivorRatio` ）、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。
>
> - PS：可以使用参数 `-XX:+UseAdaptiveSizePolicy` 让垃圾回收器根据吞吐量和最大停顿毫秒数自动调整内存大小。
>
> 开启方式：`-XX:+UseParallelGC` 或 `-XX:+UseParallelOldGC` 就可以使用Parallel Scavenge + Parallel Old这种组合。 
>
> - PS：Oracle官方建议在使用这个组合时，**不要设置堆内存的最大值**，垃圾回收器会根据最大暂停时间和吞吐量自动调整内存大小。
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。
>
> **适用场景**：后台任务，不需要与用户交互，并且容易产生大量的对象。如大数据的处理，大文件导出。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212225111952-1559169926.png" alt="image-20231212225110859" style="zoom:67%;" />



**优点**：吞吐量高，而且手动可控。为了提高吞吐量，虚拟机会动态调整堆的参数。

- PS：可以使用参数 `-XX:GCTimeRatio=n` 设置吞吐量为n。

```txt
用户线程执行时间 = n / (n + 1)
```

**缺点**：不能保证单次的停顿时间。

- PS：可以使用参数 `-XX:MaxGCPauseMillis=n` 设置每次垃圾回收时的最大停顿毫秒数。



> 证明JDK8默认采用了Parallel Scavenge垃圾回收器

1. CMD中输入下列参数即可

```bash
C:\Users\zixq\Desktop> java -XX:+PrintCommandLineFlags -version			# 打印出启动过程中使用的所有虚拟机参数

打印出的关键结果：

-XX:InitialHeapSize=264767296
-XX:MaxHeapSize=4236276736
-XX:+PrintCommandLineFlags
-XX:+UseCompressedClassPointers 
-XX:+UseCompressedOops 
-XX:-UseLargePagesIndividualAllocation 
-XX:+UseParallelGC		# 此处：已经加了使用 Parallel Scavenge 垃圾回收器的虚拟机参数
```

2. 也可在Arthas中查看：随便写段代码，使用 `System.in.read();` 定住，不添加任何虚拟机参数启动。示例如下：

```java
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * <p>
 * 测试JDK8的垃圾回收器（GC）是什么
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

public class JDK8_GC_Test {

    public static void main(String[] args) throws IOException {
        List<Object> list = new ArrayList<>();

        int count = 0;
        while (true) {
            System.in.read();
            System.out.println("++count = " + ++count);
            // 1M
            list.add(new byte[1024 * 1024]);
        }
    }
}
```

Arthas命令：

```bash
dashboard -n 1
```

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212233212383-537452935.png" alt="image-20231212233209157" style="zoom:67%;" />





###### 老年代：Parallel Old（PO）

> Parallel Old是为Parallel Scavenge收集器设计的老年代版本，JDK8默认的老年代垃圾回收器。
>
> 利用**多线程并发收集**，采用**标记-整理算法**。
>
> 开启方式：`-XX:+UseParallelGC` 或 `-XX:+UseParallelOldGC` 就可以使用Parallel Scavenge + Parallel Old这种组合。 
>
> 垃圾回收线程进行GC时，会让用户线程进入等待，GC执行完了才会进行用户线程。





<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231212233406186-1725335491.png" alt="image-20231212233405285" style="zoom:67%;" />



**优点**：并发收集，在多核CPU下效率较高。

**缺点**：暂停时间会比较长。





##### 垃圾回收器：G1

> G1（Garbage First）是在Java7 update 4之后引入的一个新的垃圾回收器，引入分区的思路，弱化了分代的概念。
>
> JDK9之后默认的垃圾回收器是G1垃圾回收器。
>
> 堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或老年代，而 G1 可以**直接对新生代和老年代一起回收（Mixed GC）**，采用**标记-复制算法**（就是先标记，再用复制算法）。
>
> 开启方式：使用参数 `-XX:+UseG1GC` 
>
> - PS：JDK9之后默认是此垃圾回收器，故不需要打开。
>
> 适用场景：JDK8最新版本、JDK9之后建议默认使用。



Parallel Scavenge关注吞吐量，允许用户设置最大暂停时间 ，但是会减少年轻代可用空间的大小。

CMS关注暂停时间，但是吞吐量方面会下降。

而G1设计目标就是将上述两种垃圾回收器的优点融合：

1. 支持巨大的堆空间回收，并有较高的吞吐量。
2. 支持多CPU并行垃圾回收。
3. 允许用户设置最大暂停时间。



优点：对比较大的堆如超过6G的堆回收时，延迟可控不会产生内存碎片并发标记的SATB算法效率高.

缺点：JDK8之前还不够成熟。





###### G1的内存划分

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213161803495-1937292118.png" alt="image-20231213161802465" style="zoom:67%;" />

G1是将整个堆会被划分成多个大小相等的区域，称之为Region，区域不要求是物理连续的（逻辑连续即可）。分为Eden、Survivor、Old区。

> 问题1：每个Region的大小是怎么计算来的？

1. 虚拟机自行计算的方式

```txt
每个Region的大小 = 堆空间大小 / 2048m


如：堆空间大小 = 4G，则每个Region的大小 = （1024 * 4）/ 2048 = 2m
```

2. 程序员手动配置的方式

```bash
可通过参数 -XX:G1HeapRegionSize=32m 指定

	其中32m指定每个region大小为32M，Region size必须是2的指数幂，取值范围从1M到32M
```



> 问题2：Region内部又有什么？

1. **卡片Card**。==G1对内存的使用以分区(Region)为单位，而对对象的分配则以卡片(Card)为单位==。

在每个分区内部又被分成了若干个大小为**512 Byte卡片(Card)**，标识堆内存最小可用粒度。

所有分区的卡片将会记录在**全局卡片表(Global Card Table)**中，分配的对象会占用物理上连续的若干个卡片，当查找对分区内对象的引用时便可通过记录卡片来查找该引用对象。

每次对内存的回收，本质都是对指定分区的卡片进行处理。

![image-20231213202231807](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213202232632-1026534972.png)

2. **已记忆集合Remember Set (RSet)**：用来记录该 Region 对象的引用对象所在的 Region。通过使用 RSet，在做可达性分析时就可以避免全堆扫描。

在串行和并行收集器中，GC通过整堆扫描，来确定对象是否处于可达路径中。

然而G1为了避免STW式的整堆扫描，在每个分区记录了一个已记忆集合(RSet)，内部类似一个反向指针，记录引用分区内对象的卡片索引。当要回收该分区时，通过扫描分区的RSet，来确定引用本分区内的对象是否存活，进而确定本分区内的对象存活情况。

- 每次引用类型数据写操作时，都会产生一个Write Barrier【写屏障/写栅栏】暂时中断操作；
- 然后检查将要写入的引用指向的对象是否和该引用类型数据在不同的Region（其他收集器：检查老年代对象是否引用了新生代对象）；
  - 如果不同，通过cardTable把相关引用信息记录到引用指向对象的所在Region对应的Remembered Set中；当进行垃圾收集时，在GC根节点的枚举范围加入Remembered Set；就可以保证不进行全局扫描，也不会有遗漏。

**这个RSet还有个注意点**：并非所有的引用都需要记录在RSet中，【结合下列第2点】只有老年代的分区可能会有RSet记录，这些分区称为**拥有RSet分区(an RSet’s owning region)**

- 如果一个分区确定需要扫描，那么无需RSet也可以无遗漏的得到引用关系，那么引用源自本分区的对象，当然不用落入RSet中；
- G1 GC每次都会对年轻代进行整体收集，因此引用源自年轻代的对象，也不需要在RSet中记录。



> 问题3：RSet的内部又有什么？有个屁，人都问麻了
>
> - 答案：PRT（ Per Region Table ）

RSet在内部使用Per Region Table(PRT)记录分区的引用情况。由于RSet的记录要占用分区的空间，如果一个分区非常"受欢迎"，那么RSet占用的空间会上升，从而降低分区的可用空间。G1应对这个问题采用了改变RSet的密度的方式，在PRT中将会以三种模式记录引用：

- 稀少：直接记录引用对象的卡片索引。
- 细粒度：记录引用对象的分区索引。
- 粗粒度：只记录引用情况，每个分区对应一个比特位。

由上可知，粗粒度的PRT只是记录了引用数量，需要通过整堆扫描才能找出所有引用，因此扫描速度也是最慢的。



> 分区，即分Region，还会牵扯到一个小知识点：本地分配缓冲 Lab（Local allocation buffer）

由于分区的思想，每个线程均可以"认领"某个分区用于线程本地的内存分配，而不需要顾及分区是否连续。因此，每个应用线程和GC线程都会独立的使用分区，进而减少同步时间，提升GC效率，这个分区称为**本地分配缓冲区(Lab)**。

其中：

1. 应用线程可以独占一个本地缓冲区(TLAB，详情见前面的分代回收算法)来创建对象，而大部分都会落入Eden区域(巨型对象或分配失败除外)，因此TLAB的分区属于Eden空间。
2. 而每次垃圾收集时，每个GC线程同样可以独占一个本地缓冲区(GCLAB)用来转移对象，每次回收会将对象复制到Suvivor空间或老年代空间；对于从Eden / Survivor空间晋升（Promotion）到Survivor / 老年代空间的对象，同样由GC独占的本地缓冲区进行操作，该部分称为**晋升本地缓冲区(PLAB)**







###### G1垃圾回收的方式

> G1垃圾回收有两种方式：
>
> 1. 年轻代回收：Young GC（Minor GC）
>
> - Young GC：回收Eden区和Survivor区中不用的对象。会导致STW。采用复制算法。
> - G1中可以通过参数 `-XX:MaxGCPauseMillis=n` （默认200ms） 设置每次垃圾回收时的最大暂停时间毫秒数，G1垃圾回收器会“尽可能地”保证暂停时间（即软实时：尽可能在此时限内完成垃圾回收）。
>
> 
>
> 2. 混合回收（年轻代+老年代）：Mixed GC。
>
> - Mixed GC：回收所有年轻代和部分老年代的对象以及大对象区。采用标记-复制算法



> 年轻代回收 Young GC（Minor GC）：执行流程
>
> - 下列流程也有个专业名字：年轻代收集集合 CSet of Young Collection

1. **新创建的对象会存放在Eden区。当G1判断年轻代区不足（max默认60%），无法分配对象时需要回收时会执行Young GC。** 

**max指的是**：年轻代内存超过整个堆内存的60%。

**注意**：部分对象如果大小达到甚至超过Region的一半，会直接放入老年代，这类老年代被称为Humongous【巨型】区。

如堆内存是4G，那么每个Region是2M，而只要一个大对象超过了1M就被放入Humongous区，如果对象过大会横跨多个Region。

G1内部做了一个优化，一旦发现没有引用指向巨型对象，则可直接在年轻代收集周期中被回收。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213165452814-586753045.png" alt="image-20231213165452618" style="zoom:67%;" />

从上图也可知：巨型对象会独占一个、或多个连续分区，其中

- 第一个分区被标记为开始巨型(StartsHumongous)，相邻连续分区被标记为连续巨型(ContinuesHumongous)。
- 由于无法享受Lab带来的优化，并且确定一片连续的内存空间需要整堆扫描，因此确定巨型对象开始位置的成本非常高，如果可以，应用程序应避免生成巨型对象。



2. **标记出Eden和Survivor区域中的存活对象。**
3. **根据配置的最大暂停时间选择“某些Region区域”，将这些区域中存活对象复制到一个新的Survivor区中（对象年龄【存活次数】+1），然后清空这些区域。**

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213214316495-2110152344.png" alt="image-20231213171509169" style="zoom:67%;" />

> 这里的小细节是：Eden分区存活的对象将被拷贝到Survivor分区；原有Survivor分区存活的对象，将根据任期阈值(tenuring threshold)分别晋升到PLAB、新的survivor分区和老年代分区中，而原有的年轻代分区将被整体回收掉。



> 问题1：所谓的“某些Region区域“是怎么选择的？

- G1在进行Young GC的过程中会去记录每次垃圾回收时每个Eden区和Survivor区的平均耗时，以作为下次回收时的参考依据。

- 然后根据配置的最大暂停时间【`-XX:MaxGCPauseMillis=n` （默认200ms）】就能计算出本次回收时最多能回收多少个Region区域。

如：`-XX:MaxGCPauseMillis=n`（默认200ms），每个Region回收耗时40ms，那么这次回收最多只能回收200 / 40 = 5，但选5个可能会超出设置的最大暂停时间，所以只选择4个Region进行回收。



> 问题2：此步要维护对象年龄（年龄+1）的原因是什么？
>
> - 答案：辅助判断老化(tenuring)对象晋升时是到Survivor分区还是到老年代分区。

- 年轻代收集首先先将晋升对象内存大小总和、对象年龄信息维护到年龄表中。
- 再根据年龄表、Survivor内存大小、Survivor填充容量 `-XX:TargetSurvivorRatio` (默认50%)、最大任期阈值`-XX:MaxTenuringThreshold` (默认15)，计算出一个恰当的任期阈值，凡是超过任期阈值的对象都会被晋升到老年代。



4. **后续Young GC时与之前相同，只不过Survivor区中存活对象会被搬运到另一个Survivor区。**当某个存活对象的年龄到达阈值（默认15），将被放入老年代。



> 关于上述年轻代回收有一个小知识点：收集集合（CSet）
>
> - 收集集合(CSet)：代表每次GC暂停时回收的一系列目标分区。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213213240528-1259651546.jpg" alt="img" style="zoom:80%;" />



在任意一次收集暂停中，CSet所有分区都会被释放，内部存活的对象都会被转移到分配的空闲分区中。

因此无论是年轻代收集，还是混合收集，工作的机制都是一致的。年轻代收集CSet只容纳年轻代分区，而混合收集会通过启发式算法，在老年代候选回收分区中，筛选出回收收益最高的分区添加到CSet中。

> 候选老年代分区的CSet准入条件：可以通过活跃度阈值 `-XX:G1MixedGCLiveThresholdPercent` (默认85%)进行设置，从而拦截那些回收开销巨大的对象；同时，每次混合收集可以包含候选老年代分区，可根据CSet对堆的总大小占比 `-XX:G1OldCSetRegionThresholdPercent` (默认10%)设置数量上限。

由上述可知，G1的收集都是根据CSet进行操作的，年轻代收集与混合收集没有明显的不同，最大的区别在于两种收集的触发条件。





> 混合收集：Mixed GC
>
> - 回收所有年轻代和部分老年代的对象以及大对象区。采用的算法：类似标记-整理（无只可使用堆的一半的限制）。
>
> **触发时机**：经过多次的回收之后(上述Young GC执行流程)，会出现很多老年代区（Old），此时总堆占有率达到阈值时（`-XX:InitiatingHeapOccupancyPercent` 默认45%）会触发混合回收Mixed GC。

为了满足暂停目标，G1可能不能一口气将所有的候选分区收集掉，因此G1可能会产生连续多次的混合收集与应用线程交替执行，每次STW的混合收集与年轻代收集过程相类似。

为了确定包含到年轻代收集集合CSet的老年代分区，JVM通过参数混合周期的最大总次数`-XX:G1MixedGCCountTarget`(默认8)、堆废物百分比-`XX:G1HeapWastePercent`(默认5%)。通过候选老年代分区总数与混合周期最大总次数，确定每次包含到CSet的最小分区数量；根据堆废物百分比，当收集达到参数时，不再启动新的混合收集。而每次添加到CSet的分区，则通过计算得到的GC效率进行安排。



**Mixed GC的回收流程**：

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213170847310-294288465.png" alt="image-20231213170846871" style="zoom:67%;" />

1. **初始标记（initial mark）：标记Gc Roots引用的对象为存活**。
2. **并发标记（concurrent mark）：将第一步中标记的对象引用的对象，标记为存活**。这里和前面说的Region维护的Remebered Set挂钩。

> CMS和G1在并发标记时使用的是同一个算法：三色标记法
>
> 使用白灰黑三种颜色标记对象。
>
> - 白色：是未标记；
> - 灰色：是自身被标记，引用的对象未标记；
> - 黑色：是自身与引用对象都已标记。



![img](https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213220758112-1592549812.gif)



**执行流程如下：**

- GC 开始前所有对象都是白色。
- GC 一开始，所有根（GC Root）能够直达的对象被压到栈中，待搜索，此时颜色是灰色。
- 然后灰色对象依次从栈中取出搜索子对象，子对象也会被涂为灰色，入栈。当其所有的子对象都涂为灰色之后，该对象被涂为黑色。
- 当 GC 结束之后，灰色对象将全部没了，**剩下黑色的为存活对象**，白色的为垃圾。



3. **最终标记（remark或者Finalize Marking）：标记一些引用改变而漏标的对象**。

> 和CMS的区别在这步：Mixed G在这里不管新创建 和 不再关联的对象。

**小细节**：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。

> 问题：漏标问题是怎么产生的？

在最终标记（remark或者Finalize Marking）过程中，黑色指向了白色，如果不对黑色重新扫描，则会漏标。会把白色D对象当作没有新引用指向从而回收掉。

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213222630479-1600594513.png" alt="img" style="zoom:80%;" />

并发标记过程中，Mutator删除了所有从灰色到白色的引用，会产生漏标。此时白色对象应该被回收。

产生漏标问题的条件有两个：

- 黑色对象指向了白色对象。
- 灰色对象指向白色对象的引用消失。

所以要解决漏标问题，打破两个条件之一即可：

- **跟踪黑指向白的增加** incremental update：增量更新，关注引用的增加。把黑色重新标记为灰色，下次重新扫描属性。CMS采用该方法。
- **记录灰指向白的消失** SATB（snapshot at the beginning）：关注引用的删除。当灰–>白消失时，要把这个 引用 推到GC的栈中，保证白还能被GC扫描到。G1采用该方法。

> 问题：为什么G1采用SATB而不用incremental update？

因为采用incremental update把黑色重新标记为灰色后，之前扫描过的还要再扫描一遍，效率太低。G1有RSet与SATB相配合。Card Table里记录了RSet，RSet里记录了其他对象指向自己的引用，这样就不需要再扫描其他区域，只要扫描RSet就可以了。

也就是说 灰色–>白色 引用消失时，如果没有 黑色–>白色，引用会被push到堆栈，下次扫描时拿到这个引用，由于有RSet的存在，不需要扫描整个堆去查找指向白色的引用，效率比较高SATB配合RSet浑然天成。



4. **并发清理（cleanup）：将存活对象复制到别的Region**。使用复制算法的目的是为了不产生内存碎片。



<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231213172021793-1104993078.png" alt="image-20231213172021831" style="zoom: 67%;" />



> 在并发清理这里顺便提一嘴：G1对老年代的清理会选择存活度最低的区域来进行回收，这样可以保证回收效率最高，这也是G1（Garbage first）名称的由来。
>
> - G1的1（first）指的就是存活度最低的区域。





> 顺便再提一下和这里相关的：Full GC 整堆收集，是单线程+标记-整理算法，会导致用户线程的暂停。
>
> 建议：尽量保证应该用的堆内存有一定多余的空间。

**触发时机**：

1. 如果上面的清理过程中发现没有足够的空Region存放转移的对象（大对象、长期存活的对象进入老年代，导致老年代空间不足），会出现Full GC。

避免这种情况引起Full GC的方式：

一是：尽量不要创建过大的对象以及数组。

二是：通过参数 `-Xmn` 调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。

三是：通过参数 `-XX:MaxTenuringThreshold` 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

> 此处相关的小知识：Concurrent Mode Failure 并发模式失败
>
> - 执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足(可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足)，便会报 Concurrent Mode Failure 错误，并触发 Full GC。

2. 调用 `System.gc()`。不建议的方式。因为调用此方法只是建议虚拟机执行 Full GC，但是虚拟机不一定会真正去执行。
3. 使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

**内存空间分配担保**：在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间。

- 如果条件成立的话，那么 Minor GC 可以确认是安全的。

- 如果不成立的话：

JDK 6 Update 24之前规则：虚拟机会查看 `-XX:HandlePromotionFailure=true/false`设置值是否允许担保失败

- 如果允许，那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小

  - 如果大于，将尝试着进行一次 Minor GC；

  - 如果小于，或者 HandlePromotionFailure 设置不允许冒险，那么就要进行一次 Full GC。

**JDK 6 Update 24之后的规则**：`-XX:HandlePromotionFailure=true/false`配置不会再影响到虚拟机的空间分配担保策略。所以此时是只要老年代的连续空间大于新生代对象空间总大小 或者 历次晋升的平均大小，就会进行 Minor GC，否则将进行 Full GC。





##### 关于Minor GC、Major GC、Full GC的说明

> 尽量别让Major GC / Full GC触发。

JVM 在进行 GC 时，并非每次都对堆内存（新生代、老年代；方法区）区域一起回收的，大部分时候回收的都是指新生代。

针对 HotSpot VM 的实现，它里面的 GC 按照回收区域又分为两大类：部分收集（Partial GC），整堆收集（Full GC）。

1. 部分收集（Partial GC）：不是完整收集整个 Java 堆的垃圾收集其中又分为： 

- 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集。
- 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集。
  - 目前，只有 CMS GC 会有单独收集老年代的行为。
  - 很多时候 Major GC 会和 Full GC 混合使用，需要具体分辨是老年代回收还是整堆回收。

- 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集。
  - 目前只有 G1 GC 会有这种行为。

2. 整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾。



> 年轻代GC（Minor GC）触发机制

- 当年轻代空间不足时，就会触发MinorGC，这里的年轻代满指的是Eden代满，Survivor满不会引发GC。（每次Minor GC会清理年轻代的内存）
- 因为Java对象大多都具备朝生夕灭的特性.，所以Minor GC非常频繁，一般回收速度也比较快。这一定义既清晰又易于理解。
- Minor GC会引发STW，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行。



> 老年代GC（Major GC / Full GC）触发机制 

- 指发生在老年代的GC，对象从老年代消失时，我们说 “Major GC” 或 “Full GC” 发生了。

> 出现了Major Gc，经常会伴随至少一次的Minor GC（非绝对，在Paralle1 Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）。
>
> ==也就是在老年代空间不足时，会先尝试触发Minor Gc。如果之后空间还不足，则触发Full GC（Major GC）==。

- Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长。
- 如果Major GC后，内存还不足，就报OOM了。



>  Full GC触发机制：Full GC 是开发或调优中尽量要避免的。这样暂停时间会短一些。因为Full GC是单线程+标记-整理，单线程会STW会长。

触发Full GC执行的情况有如下几种：

1. 调用`System.gc()`时，系统建议执行Full GC，但是不必然执行。
2. 老年代空间不足。前面G1说的触发时机基本都可统称为这个原因。

- 补充一种情况：由Eden区、survivor space0（From Space）区向survivor space1（To Space）区复制时，对象大小 大于 To Space可用内存，则把该对象转存到老年代，且老年代的可用内存 小于 该对象大小。

3. 方法区空间不足。







# Java问题排查之Linux命令

> 声明：此章节内容整理自：[@pdai：调试排错 - Java 问题排查之Linux命令](https://www.pdai.tech/md/java/jvm/java-jvm-debug-tools-linux.html)



## 文本操作

### 文本查找 - grep

grep常用命令：

```bash
# 基本使用
grep yoursearchkeyword f.txt										# 文件查找
grep 'KeyWord otherKeyWord' f.txt cpf.txt							# 多文件查找, 含空格加引号
grep 'KeyWord' /home/admin -r -n									# 目录下查找所有符合关键字的文件
grep 'keyword' /home/admin -r -n -i									# -i 忽略大小写
grep 'KeyWord' /home/admin -r -n --include *.{vm,Java}				# 指定文件后缀
grep 'KeyWord' /home/admin -r -n --exclude *.{vm,Java}				# 反匹配

# cat + grep
cat f.txt | grep -i keyword											# 查找所有keyword且不分大小写  
cat f.txt | grep -c 'KeyWord'										# 统计Keyword次数

# seq + grep
seq 10 | grep 5 -A 3												# 上匹配
seq 10 | grep 5 -B 3												# 下匹配
seq 10 | grep 5 -C 3												# 上下匹配，平时用这个就妥了
```

Grep的参数：

```bash
--color=auto													# 显示颜色;
-i, --ignore-case												# 忽略字符大小写;
-o, --only-matching												# 只显示匹配到的部分;
-n, --line-number												# 显示行号;
-v, --invert-match												# 反向显示,显示未匹配到的行;
-E, --extended-regexp											# 支持使用扩展的正则表达式;
-q, --quiet, --silent											# 静默模式,即不输出任何信息;
-w, --word-regexp												# 整行匹配整个单词;
-c, --count														# 统计匹配到的行数; print a count of matching lines;

-B, --before-context=NUM：print NUM lines of leading context		# 后#行
-A, --after-context=NUM：print NUM lines of trailing context		# 前#行
-C, --context=NUM：print NUM lines of output context				# 前后各#行
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
awk '/ldb/ {print}' f.txt						# 匹配ldb
awk '!/ldb/ {print}' f.txt						# 不匹配ldb
awk '/ldb/ && /LISTEN/ {print}' f.txt			# 匹配ldb和LISTEN
awk '$5 ~ /ldb/ {print}' f.txt					# 第五列匹配ldb
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
sed -n '3p' xxx.log							# 只打印第三行
sed -n '$p' xxx.log							# 只打印最后一行
sed -n '3,9p' xxx.log						# 只查看文件的第3行到第9行
sed -n -e '3,9p' -e '=' xxx.log				# 打印3-9行，并显示行号
sed -n '/root/p' xxx.log					# 显示包含root的行
sed -n '/hhh/,/omc/p' xxx.log				# 显示包含"hhh"的行到包含"omc"的行之间的行

# 文本替换
sed -i 's/root/world/g' xxx.log				# 用world 替换xxx.log文件中的root; s==search  查找并替换, g==global  全部替换, -i: implace

# 文本插入
sed '1,4i hahaha' xxx.log								# 在文件第一行和第四行的每行下面添加hahaha
sed -e '1i happy' -e '$a new year' xxx.log  			#【界面显示】在文件第一行添加happy,文件结尾添加new year
sed -i -e '1i happy' -e '$a new year' xxx.log			#【真实写入文件】在文件第一行添加happy,文件结尾添加new year

# 文本删除
sed  '3,9d' xxx.log									# 删除第3到第9行,只是不显示而已
sed '/hhh/,/omc/d' xxx.log							# 删除包含"hhh"的行到包含"omc"的行之间的行
sed '/omc/,10d' xxx.log								# 删除包含"omc"的行到第十行的内容

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
tail -f xxx.log					# 循环监听文件
tail -300f xxx.log				# 倒数300行并追踪文件
tail +20 xxx.log				# 从第 20 行至文件末尾显示文件内容

# tailf使用
tailf xxx.log					# 等同于tail -f -n 10 打印最后10行，然后追踪文件
```

tail -f 与tail F 与tailf三者区别

```bash
`tail  -f `  等于--follow=descriptor，根据文件描述进行追踪，当文件改名或删除后，停止追踪

`tail -F` 等于 --follow=name ==retry，根据文件名字进行追踪，当文件改名或删除后，保持重试，当有新的文件和他同名时，继续追踪

`tailf` 等于tail -f -n 10（tail -f或-F默认也是打印最后10行，然后追踪文件），与tail -f不同的是，如果文件不增长，它不会去访问磁盘文件，所以tailf特别适合那些便携机上跟踪日志文件，因为它减少了磁盘访问，可以省电
```

tail的参数

```bash
-f				循环读取
-q				不显示处理信息
-v				显示详细的处理信息
-c<数目>		   显示的字节数
-n<行数>		   显示文件的尾部 n 行内容
--pid=PID		与-f合用,表示在进程ID,PID死掉之后结束
-q, --quiet, --silent			从不输出给出文件名的首部
-s, --sleep-interval=S			与-f合用,表示在每次反复的间隔休眠S秒
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
env							# 查看环境变量资源
uptime						# 查看系统运行时间、用户数、负载
lsusb -tv					# 列出所有USB设备的linux系统信息命令
lspci -tv					# 列出所有PCI设备
head -n 1 /etc/issue		# 查看操作系统版本，是数字1不是字母L
uname -a					# 查看内核/操作系统/CPU信息的linux系统信息命令

# /proc/
cat /proc/cpuinfo			# 查看CPU相关参数的linux系统命令
cat /proc/partitions		# 查看linux硬盘和分区信息的系统信息命令
cat /proc/meminfo			# 查看linux系统内存信息的linux系统命令
cat /proc/version			# 查看版本，类似uname -r
cat /proc/ioports			# 查看设备io端口
cat /proc/interrupts		# 查看中断
cat /proc/pci				# 查看pci设备的信息
cat /proc/swaps				# 查看所有swap分区的信息
cat /proc/cpuinfo |grep "model name" && cat /proc/cpuinfo |grep "physical id"
```



## tsar

tsar是淘宝开源的的采集工具很好用, 将历史收集到的数据持久化在磁盘上，所以我们快速来查询历史的系统数据当然实时的应用情况也是可以查询的啦大部分机器上都有安装

```bash
tsar					## 可以查看最近一天的各项指标
tsar --live				## 可以查看实时指标，默认五秒一刷
tsar -d 20161218		## 指定查看某天的数据，貌似最多只能看四个月的数据
tsar --mem
tsar --load
tsar --cpu				## 当然这个也可以和-d参数配合来查询某天的单个指标的情况 
```

具体可以看这篇文章：[linux 淘宝开源监控工具tsa](https://www.jianshu.com/p/5562854ed901)





# 排错

> **内存泄漏**（memory leak）：在Java中如果不再使用一个对象，但是该对象依然在GC ROOT的引用链上，这个对象就不会被垃圾回收器回收，这种情况就称之为内存泄漏。 
>
> - PS：==内存泄漏绝大多数情况都是由堆内存泄漏引起的==。
>
> **内存溢出**：指的是内存的使用量超过了Java虚拟机可以分配的上限，最终产生了内存溢出OutOfMemory的错误。



**解决内存溢出的思路**：

1. **发现问题**：通过监控工具尽可能尽早地发现内存慢慢变大的现象。
2. **诊断原因**：通过分析内存快照或者在线分析方法调用过程，诊断问题产生的根源，定位到出现问题的源代码。
3. **修复问题**：尝试重现问题，如借助jmeter什么鬼之类的。之后修复，如源代码中的bug问题、技术方案不合理、业务设计不合理等等。
4. **验证测试**：在测试环境验证问题是否已经解决，最后发布上线。



**内存溢出产生的原因**：

1. **持续的内存泄漏**：内存泄漏持续发生，不可被回收同时不再使用的内存越来越多，就像滚雪球雪球越滚越大，最终内存被消耗完无法分配更多的内存取使用，导致内存溢出。

这种原因一般就是代码中的内存泄漏，所以一般在测试阶段就会被测试出来，如下示例：

- **不正确的`eauals()`和`hashcode()`**：定义新类时没有重写正确的equals()和hashCode()方法。在使用HashMap的场景下，如果使用这个类对象作为key，HashMap在判断key是否已经存在时会使用这些方法，如果重写方式不正确，会导致相同的数据被保存多份。

此种情况的解决方式：定义新实体类时记得重写这两个方法，且重写时使用“唯一标识”去区分不同对象，以及在使用HashMap时key使用实体的“唯一标识”。

- **非静态内部类和匿名内部类的错误使用**：非静态的内部类默认会持有外部类，尽管代码上不再使用外部类，所以如果有地方引用了这个非静态内部类，会导致外部类也被引用，垃圾回收时无法回收这个外部类。另外就是匿名内部类对象如果在非静态方法中被创建，会持有调用者对象，垃圾回收时无法回收调用者。

此种情况的解决方式：使用静态内部类和静态方法即可。

- **ThreadLocal的错误使用**：由于线程池中的线程不被回收导致的ThreadLocal内存泄漏。

如果仅仅使用手动创建的线程，就算没有调用ThreadLocal的remove方法清理数据，也不会产生内存泄漏。因为当线程被回收时，ThreadLocal也同样被回收。但是如果使用线程池就不一定了。

此种情况的解决方式：线程方法执行完，记得调用ThreadLocal中的remove方法清理对象。

- **静态变量的错误使用（很常见哦）**：大量的数据在静态变量中被引用，但不再使用，就成为了内存泄漏。

如果大量的数据在静态变量中被长期引用，数据就不会被释放，如果这些数据不再使用，就成为了内存泄漏。

此种情况的解决方式：

一是：尽量减少将对象长时间的保存在静态变量中，如果不再使用，必须将对象删除（比如在集合中）或者将静态变量设置为null。 

二是：使用单例模式时，尽量使用懒加载，而不是立即加载。

三是：Spring的Bean中不要长期存放大对象，如果是缓存用于提升性能，尽量设置过期时间定期失效。

- **资源没有正常关闭**：由于资源没有调用`close()`方法正常关闭，”可能“导致内存泄漏。

连接和流这些资源会占用内存，如果使用完之后没有关闭，**这部分内存"不一定"会出现内存泄漏**，但是会导致close方法不被执行。

不一定的原因：如下列代码

```java
public static void lead() throws SQLException {	// 此方法执行完
    Startement stmt = null;
    Connection conn = DriverManager.getConnection(DB_URL, USER, PASS);
    // 则Connection不在GC Roots引用链上，就会被回收，从而conn关联的Startement、RestultSet这些对象也会被回收，从而不会造成内存泄漏
    
    stmt = conn.createStatement();
    String sql = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
    RestultSet rs = stmt.executeQuery(sql);
    
    while(rs.next()){
        // ...............
    }
    
    // 最终没有关闭流管道
}
```

此种情况的解决方式：在finally块中关闭不再使用的资源。另外就是从 Java 7 开始，可以使用try-with-resources语法可以用于自动关闭资源。

![image-20240106215225651](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106215224113-1647299824.png)

2. **并发请求问题**：用户通过发送请求向Java应用获取数据，正常情况下Java应用将数据返回之后，这部分数据就可以在内存中被释放掉。但是由于用户的并发请求量有可能很大，同时处理数据的时间很长，导致大量的数据存在于内存中，最终超过了内存的上限，导致内存溢出。









## 发现问题：可视化工具

### Java 调试入门工具

> 声明：此节内容主要整理自[@pdai：调试排错 - Java 问题排查之工具单](https://www.pdai.tech/md/java/jvm/java-jvm-debug-tools-list.html)，在此基础上做了改动。

#### jps：查看当前进程

> jps是JDK提供的一个查看当前Java进程的小工具， 可以看做是Java Virtual Machine Process Status Tool的缩写

jps常用命令

```bash
jps								# 显示进程的ID 和 类的名称
jps –l							# 输出输出完全的包名，应用主类名，jar的完全路径名 
jps –v							# 输出JVM参数
jps –q							# 显示Java进程号
jps -m							# main 方法
jps -l xxx.xxx.xx.xx			# 远程查看


jps参数：

    -q							# 仅输出VM标识符，不包括Classname,jar name,arguments in main method 
    -m							# 输出main method的参数 
    -l							# 输出完全的包名，应用主类名，jar的完全路径名 
    -v							# 输出JVM参数 
    -V							# 输出通过flag文件传递到JVM中的参数(.hotspotrc文件或-XX:Flags=所指定的文件 
    -Joption					# 传递参数到vm,例如:-J-Xms512m
```



jps原理

> Java程序在启动以后，会在Java.io.tmpdir指定的目录下，就是临时文件夹里，生成一个类似于hsperfdata_User的文件夹，这个文件夹里（在Linux中为/tmp/hsperfdata_{userName}/），有几个文件，名字就是Java进程的pid，因此列出当前运行的Java进程，只是把这个目录里的文件名列一下而已。至于系统的参数什么，就可以解析这几个文件获得

更多请参考 [jps - Java Virtual Machine Process Status Tool](https://docs.oracle.com/Javase/1.5.0/docs/tooldocs/share/jps.html)





#### jstack：线程的栈信息

> jstack是JDK自带的线程堆栈分析工具，使用该命令可以查看或导出 Java 应用程序中线程堆栈信息

jstack常用命令:

```bash
# 基本
jstack 2815

# Java和native c/c++框架的所有栈信息
jstack -m 2815

# 额外的锁信息列表，查看是否死锁
jstack -l 2815




jstack参数：

    -l			# 长列表. 打印关于锁的附加信息,例如属于Java.util.concurrent 的 ownable synchronizers列表.

    -F			# 当’jstack [-l] pid’没有相应的时候强制打印栈信息

    -m			# 打印Java和native c/c++框架的所有栈信息.

    -h | -help	# 打印帮助信息
```

更多请参考: [JVM 性能调优工具之 jstack](https://www.jianshu.com/p/025cb069cb69)

#### jinfo：查看参数信息

> jinfo 是 JDK 自带的命令，可以用来查看正在运行的 Java 应用程序的扩展参数，包括Java System属性和JVM命令行参数；也可以动态的修改正在运行的 JVM 一些参数。当系统崩溃时，jinfo可以从core文件里面知道崩溃的Java应用程序的配置信息

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



jinfo参数：

    no option				# 输出全部的参数和系统属性
    -flag name				# 输出对应名称的参数
    -flag [+|-]name			# 开启或者关闭对应名称的参数
    -flag name=value		# 设定对应名称的参数
    -flags					# 输出全部的参数
    -sysprops				# 输出系统属性
```

更多请参考：[JVM 性能调优工具之 jinfo](https://www.jianshu.com/p/8d8aef212b25)





#### jmap：生成dump文件 和 查看堆情况

> 命令jmap是一个多功能的命令。它可以生成 Java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。
>
> - PS：dump文件是什么去这里：https://www.cnblogs.com/toSeeMyDream/p/7151635.html

两个用途

```bash
# 查看堆的情况
jmap -heap 2815

# dump
jmap -dump:live,format=b,file=/tmp/heap2.bin 2815
jmap -dump:format=b,file=/tmp/heap3.bin 2815

# 查看堆的占用
jmap -histo 2815 | head -10



jmap参数：

    no option					# 查看进程的内存映像信息,类似 Solaris pmap 命令
    heap						# 显示Java堆详细信息
    histo[:live]				# 显示堆中对象的统计信息
    clstats						# 打印类加载器信息
    finalizerinfo				# 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
    dump:<dump-options>			# 生成堆转储快照
    F							# 当-dump没有响应时，使用-dump或者-histo参数. 在这个模式下,live子参数无效.
    help						# 打印帮助信息
    J<flag>						# 指定传递给运行jmap的JVM的参数
```

更多请参考：

1. [JVM 性能调优工具之 jmap](https://www.jianshu.com/p/a4ad53179df3) 
2. [jmap - Memory Map](https://docs.oracle.com/Javase/1.5.0/docs/tooldocs/share/jmap.html)





#### jstat：总结垃圾回收统计

jstat参数众多，但是使用一个就够了

```bash
# 命令格式：jstat -gcutil pid interval(间隔，单位ms)
jstat -gcutil 2815 1000
```

更多请参考：[Java的jstat命令使用详解](https://blog.csdn.net/heihaozi/article/details/123497656)





#### jdb：预发debug

jdb可以用来预发debug，假设你预发的Java_home是/opt/Java/，远程调试端口是8000，那么

```bash
jdb -attach 8000
```

出现以上代表jdb启动成功。后续可以进行设置断点进行调试。

具体参数可见oracle官方说明[jdb - The Java Debugger](http://docs.oracle.com/Javase/7/docs/technotes/tools/windows/jdb.html)







### Linux：Top命令

> top除了看一些基本信息之外，剩下的就是配合来查询vm的各种问题了。
>
> 缺点：只能查看最基础的进程信息，无法查看到每个部分的内存占用（堆、方法区、堆外）

top命令是Linux下用来查看系统信息的一个命令，它提供给我们去实时地去查看系统的资源，比如执行时的进程、线程和系统参数等信息。

> 关于下列两个概念的说明：
>
> 1. **常驻内存**：当前进程总的使用了多少内存。
>
> - PS：常驻内存包含了“共享内存”，所以当前进程真正使用的内存是：常驻内存 - 共享内存。
>
> 2. **共享内存**：当前进程第三方依赖需要的内存。只加载一次，其他地方就可以用了，故而称为“共享”。

```bash
# top -H -p pid

top - 08:37:51 up 45 days, 18:45,  1 user,  load average: 0.01, 0.03, 0.05
Threads:  28 total,   0 running,  28 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.7 sy,  0.0 ni, 98.6 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1882088 total（堆的总内存）,    74608 free（空闲内存，若此值极小则说明本服务器的程序有问题）,   202228 used（已使用内存）,  1605252 buff/cache（缓存）		# 关注点
KiB Swap:  2097148 total,  1835392 free,   261756 used.  1502036 avail Mem


# %CPU		当前进程对CPU的使用率				 若此值长期保持很高，则需要关注程序请求量是否过大，或出现死循环之类的
# %MEM		当前进程占总内存的比率					若上面的 free值很小，而此值很高，则可以确定系统内存不足就是当前进程所造成的
# TIME+		当前进程自启动以来所消耗的CPU累计时间
# COMMAND	启动命令
  PID USER      PR  NI    VIRT（虚拟内存）    RES（常驻内存）    SHR（共享内存） S %CPU %MEM     TIME+ COMMAND
 1347 root      20   0 2553808 				113752   		 1024 			S  0.3  6.0  48:46.74 VM Periodic Tas
 1249 root      20   0 2553808 				113752   		 1024 			S  0.0  6.0   0:00.00 Java
 1289 root      20   0 2553808 				113752   		 1024 			S  0.0  6.0   0:03.74 Java
...
```







### JConsole：本地+远程监控

> Jconsole （Java Monitoring and Management Console），JDK自带的基于JMX的可视化监视、管理工具 官方文档可以参考[这里](https://docs.oracle.com/Javase/8/docs/technotes/guides/management/jconsole.html)
>
> 路径：JDK\bin\jconsole.exe



本地连接 或 远程连接：

> 注：远程连接在“测试环境”用就可以了，别在线上环境用。

![image-20240106195445531](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106195445077-496683861.png)



![image-20240106195600896](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106195559898-1219779873.png)







### VisualVM：本地+远程监控

> VisualVM 是一款免费的，集成了多个 JDK 命令行工具的可视化工具，整合了命令行 JDK 工具和轻量级分析功能，它能为您提供强大的分析能力，对 Java 应用程序做性能分析和调优这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时它还支持在 MBeans 上进行浏览和操作。
>
> 注：这款软件在Oracle JDK 6~8 中发布（路径：JDK\bin\jvisualvm.exe），但是在 Oracle JDK 9 之后不在JDK安装目录下需要单独下载。下载地址：https://visualvm.github.io/
>
> 优点：支持Idea插件，开发过程中也可以使用。
>
> 缺点：对大量集群化部署的Java进程需要手动进行管理。



本地连接：JDK\bin\jvisualvm.exe的方式，这种是中文版

![image-20240106200430766](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106200429361-1686866202.png)

IDEA插件的方式：

![image-20240106200638281](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106200636862-1051496214.png)

以下两种方式均可启动VisualVM

![image-20240106200750834](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106200759167-968268559.png)

远程连接：

> 注：==只可用于“测试环境”，不可用于“生产环境”==。因为操作VisualVM中提供的功能时会停掉线程，从而影响用户。

1. 服务器中开启JMX远程连接

```bash
java -jar \
-Djava.rmi.server.hostname=xxxxxxxx \					# 配置主机名	就是服务器ip
-Dcom.sun.management.jmxremote \						# 开启JMX远程连接
-Dcom.sun.management.jmxremote.port=xxxx \				# 设置连接的端口号
-Dcom.sun.management.jmxremote.ssl=false \				# 关闭ssl连接
-Dcom.sun.management.jmxremote.authenticate=false \		# 关闭身份验证
xxxxx.jar												# 要启动的服务jar包
```

2. 使用VisualVM建立远程连接

![image-20240106202155366](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106202154183-1465862557.png)



### Arthas Tunnel

> 官网地址：https://arthas.aliyun.com/doc/tunnel.html
>
> 优点：
>
> 1. 功能强大，不止于监控基础的信息，还能监控单个方法的执行耗时等细节内容。
> 2.  支持应用的集群管理.



大概流程如下：

![image-20240106202700733](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106202659094-425000391.png)

大概操作流程如下：

1. 添加依赖（目前仅支持Spring Boot2.x版本），在配置文件中添加tunnel服务端的地址，便于tunnel去监控所有的程序。

```xml
<dependency>
    <groupId>com.taobao.arthas</groupId>
    <artifactId>arthas-spring-boot-starter</artifactId>
    <version>3.7.1</version>
</dependency>
```

YAML配置：

```yaml
arthas:
  # tunnel部署的地址
  tunnel-server: ws://localhost:7777/ws
  # tunnel显示的应用名称：注册到tunnel上的每个服务都要有个名称
  app-name: ${spring.application.name}
  # arthas http访问的端口 和 远程连接的端口		这两个端口不可重复
  http-port: 8888
  telnet-port: 9999
```

2. 将tunnel服务端程序部署在某台服务器上并启动

> 注：需要去官网[下载](https://github.com/alibaba/arthas/releases) tunnel的jar包丢在服务器目录中，

```bash
# 启动命令
nohup java -jar \										# nohup 即no hang up（不挂起），后台不挂断
-Darthas.detail.pages=true \							# 打开可视化页面	注：这个页面占用的端口是80
arthas-tunnel-server-下载的某版本-fatjar.jar $			# 别忘了有个 $	即：将这个任务放到后台执行
```

页面网址：部署tunnel的ip:8080/apps.html

3. 启动Java程序，然后再上一步的页面中就可以看到对应的应用名称了。打开tunnel的服务端页面，查看所有的进程列表，并选择进程（应用名称）就可进入arthas进行arthas的操作。

> 排错：在arthas-tunnel-server-下载的某版本-fatjar.jar所在的目录中有一个nohup.out文件，打开即可排错，如：有些服务没注册上来之类的。





### Eclipse Memory Analyzer (MAT)

> ==这玩意儿可以说在开发中都会接触到，所以需要好好了解一下==。
>
> MAT 是一种快速且功能丰富的 Java 堆分析器，可帮助你发现内存泄漏并减少内存消耗。MAT在的堆内存分析问题使用极为广泛，需要重点掌握。
>
> 可以在[这里](https://www.eclipse.org/mat/)下载， 官方文档可以看[这里](http://help.eclipse.org/latest/index.jsp?topic=/org.eclipse.mat.ui.help/welcome.html)
>
> 提示：启动时可能会提示某某版本的JDK不支持，需要某某版本或以上，那安装对应的JDK版本，然后将其直到bin目录的路径放到path配置中即可，但：建议将此版本配置移到最上面或比其他版本的JDK更靠上。



先来了解三个东西：也是MAT的原理

1. **支配树树（Dominator Tree）**：MAT提供了支配树的对象图。**支配树展示的是对象实例间的支配关系**。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106221447907-1599564403.png" alt="image-20240106221449391" style="zoom:67%;" />

如上图所示：

- 对于B来说：B引用了A，而B并没有再引用其他的（即：到B只有一条路，A ->B），所以就是说：A支配B（如右图所示）。
- 而C是同理，对于D和F来说（用D来举例）：D引用了B，而B引用了A；同时D引用了C，而C引用了A（所以是两条线），但归根到底就是A支配了D。
- 其他E、F也是和D是同理分析的。



2. **深堆（Retained Heap）和浅堆（Shallow Heap）**

> **浅堆(Shallow Heap）**：支配树中对象本身占用的空间。 
>
> **深堆（Retained Heap）**：支配树中对象的子树就是所有被该对象支配的内容，这些内容组成了对象的深堆（Retained Heap），也称之为保留集（ Retained Set ） 。**深堆的大小表示该对象如果可以被回收，能释放多大的内存空间**。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106222616825-1479814000.png" alt="image-20240106222618607" style="zoom:67%;" />

如上图所示：在这个支配树中，对于C这个对象来说

- 这个对象本身占用的空间就是"浅堆"。
- C这个对象 以及 它的子树 所组成的空间大小就是深堆，若C对象被回收，那能够回收的空间大小就是：C对象本身+其子树E对象 这二者的总空间大小。



> MAT内存泄漏检测的原理：MAT就是根据支配树，从叶子节点向根节点遍历，如果发现深堆的大小超过整个堆内存的一定比例阈值，就会将其标记成内存泄漏的“嫌疑对象”。





#### 使用MAT发现问题

1. 当堆内存溢出时，可以在堆内存溢出时将整个堆内存保存下来，生成内存快照(Heap Profile )文件。

> 使用内存快照的目的：找出是程序哪里引发的问题、定位到问题出现的地方。

生成内存快照的Java虚拟机参数：

```bash
-XX:+HeapDumpOnOutOfMemoryError				发生OutOfMemoryError错误时，自动生成hprof内存快照文件

-XX:HeapDumpPath=<path>						指定hprof文件的输出路径
```

2.  使用MAT打开hprof文件（file -> open head dump），并选择内存泄漏检测功能（Leak Suspects Report 即内存泄漏检测报告），MAT会自行根据内存快照中保存的数据分析内存泄漏的根源。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106231453019-531139479.png" alt="image-20240106231454300" style="zoom:67%;" />



> 服务器中导出运行中系统的内存快照的简单方式：场景为内存在持续增长，但未发生内存泄漏，所以上面的`-XX:+HeapDumpOnOutOfMemoryError`就不能用
>
> - PS：注意只需要导出标记为存活的对象即可。

1. 通过JDK自带的jmap命令导出，格式为：

```bash
jmap -dump:live,format=b,file=文件路径和文件名 进程ID

# 进程ID获取方式
ps -ef|grep java
```

2. 通过arthas的heapdump命令导出，格式为：

```bash
heapdump --live 文件路径\文件名
```





> 生成的堆内存报告很大怎么办？

机器内存范围之内的快照文件，直接使用MAT打开分析即可。

但是经常会遇到生成的快照文件很大，要下载到本地来也要很久。此时就需要下载服务器操作系统对应的MAT。下载地址：https://eclipse.dev/mat/downloads.php

然后将下载的对应版本MAT丢在某服务器中，如：Linux中。

> 注意：服务器中放MAT的目录记得将读写权限打开。

之后通过MAT中的脚本生成分析报告：生成的报告就是像上面那种静态页面

> 注意：默认MAT分析时只使用了1G的堆内存，如果快照文件超过1G，需要修改MAT目录下的MemoryAnalyzer.ini配置文件调整最大堆内存（`-Xmx值`）。

```bash
# 生成之后，在快照文件路径中会有几个压缩包，对应：内存泄漏报告、系统总览图、组件，下载自己需要的压缩包到本地即可
./ParseHeapDump.sh 快照文件路径 org.eclipse.mat.api:suspects org.eclipse.mat.api:overview org.eclipse.mat.api:top_components
```

最后将分析报告下载到本地，打开即可（一般有一个index.html）。





> 涉及到SpringMVC时，怎么定位到是哪个接口导致的问题？

1. 生成内存快照，使用MAT打开
2. 打开支配树，使用深堆（Retained Heap）排序，找到当前执行线程，如下面的taskThread，随便打开一个即可。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240106235928191-1330363652.png" alt="image-20240106235928972" style="zoom:67%;" />

3. 找到当前线程正在执行的方法是哪一个。即找handleMethod，右键选择list objects（当前对象关联的对象） -> outgoing references（当前对象引用了哪些对象）

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107000727455-679510651.png" alt="image-20240107000728753" style="zoom:67%;" />

4. 找到description，这里就可找到到底是哪个controller的哪个方法导致的问题。
5. 然后将本地代码的Java虚拟机参数弄成和服务器一样，之后重现问题（借助压测jmeter什么鬼之类的）。
6. 最后结合前面内存快照得到的原因，解决问题，验证测试即可。









### 在线定位问题：Arthas之stack命令 和 btrace工具

诊断和解决问题一般有两种方案：离线分析（即生成内存快照分析）、在线定位。

> 内存快照分析：

优点：有完整的内存快照，从而能更准确地判断出问题的原因。

缺点：

- 内存较大时，生成内存快照较慢，这个过程会影响用户的使用。
- 通过MAT分析内存快照，至少要准备 1.5 - 2倍大小的内存空间。



> 在线定位

优点：无需生成内存快照，整个过程对用户的影响“较小”。

缺点：

- 无法查看到详细的内存信息。
- 需要具备一定的经验。而且一般还需要借助另外的工具（本章节使用arthas的stack命令 和 btrace工具）。



> Arthas的stacke在线定位大致思路

1. 将内存中存活对象以直方图的形式保存到文件中，这个过程会影响用户的时间，但是时间比较短暂。

使用命令如下：

```bash
jmap -histo:live 进程ID > 文件路径/文件名		# 表示：将 > 符号左边的内容 输出到 右边这个路径中
```

2. 查看直方图，分析内存占用最多的对象（直方图是排好序的），一般这些对象就是造成内存泄漏的原因。
3. 使用arthas的 [stack](https://arthas.aliyun.com/doc/stack.html) 命令，追踪第2步中分析的对象创建的方法被调用的调用路径，找到对象创建的根源。

使用命令如下：假设2中分析出来的对象是UserEntity

> 注意：别忘了把Arthas的jar包上传到服务器目录中，不然下面的命令能用个毛线。

```bash
stack com.zixieqing.jvm.entity.UserEntity -n 1	# 意思：输出1次com.zixieqing.jvm.entity.UserEntity这个类的所有方法的调用路径
```

通过上面的方式就可以找到是：哪个类那个方法哪一行了，然后尝试重现问题，修复问题、验证测试即可。





#### btrace工具

> btrace是一个在Java 平台上执行的追踪工具，可以有效地用于线上运行系统的方法追踪，具有侵入性小、对性能的影响微乎其微等特点。是生产环境&预发的排查问题大杀器。项目中可以使用btrace工具，实现定制化，打印出方法被调用的栈信息等等。

使用方法：btrace 具体可以参考这里：https://github.com/btraceio/btrace

1. 下载btrace工具， 官方地址：https://github.com/btraceio/btrace/releases/latest
2. 编写btrace脚本，通常是一个Java文件。如下两个示例：

编写时为了有提示和提供对应注解方法且不报错，可以加入如下依赖：路径改为自己下载的本地btrace

```xml
<dependency>
    <groupId>org.openjdk.btrace</groupId>
    <artifactId>btrace-agent</artifactId>
    <version>${btrace.version}</version>
    <scope>system</scope>
    <systemPath>D:Install\btrace-v2.2.4-bin\libs\btrace-agent.jar</systemPath>
</dependency>
<dependency>
    <groupId>org.openjdk.btrace</groupId>
    <artifactId>btrace-boot</artifactId>
    <version>${btrace.version}</version>
    <scope>system</scope>
    <systemPath>D:Install\btrace-v2.2.4-bin\libs\btrace-boot.jar</systemPath>
</dependency>
<dependency>
    <groupId>org.openjdk.btrace</groupId>
    <artifactId>btrace-client</artifactId>
    <version>${btrace.version}</version>
    <scope>system</scope>
    <systemPath>D:Install\btrace-v2.2.4-bin\libs\btrace-client.jar</systemPath>
</dependency>
```

- 查看当前谁调用了ArrayList的add方法，同时只打印当前ArrayList的size大于500的线程调用栈。编写如下Java代码

```java
@BTrace
public class TracingAdd {
    
    @OnMethod(clazz = "Java.util.ArrayList", method="add", 
          location = @Location(value = Kind.CALL, clazz = "/./", method = "/./")
         )
    public static void m(@ProbeClassName String probeClass, @ProbeMethodName String probeMethod, 
                         @TargetInstance Object instance, @TargetMethodOrField String method) {

        if(getInt(field("Java.util.ArrayList", "size"), instance) > 479){
            println("check who ArrayList.add method:" + probeClass + "#" + probeMethod  + ", method:" + method + ", size:" + getInt(field("Java.util.ArrayList", "size"), instance));
            jstack();
            println();
            println("===========================");
            println();
        }
	}
}
```

- 监控当前服务方法被调用时返回的值以及请求的参数

```java
@BTrace
public class TaoBaoNav {
    @OnMethod(clazz = "com.taobao.sellerhome.transfer.biz.impl.C2CApplyerServiceImpl", method="nav", 
          location = @Location(value = Kind.RETURN)
         )
    public static void mt(long userId, int current, int relation, 
                          String check, String redirectUrl, @Return AnyType result) {

        println("parameter# userId:" + userId + ", current:" + current + ", relation:" + relation + ", check:" + check + ", redirectUrl:" + redirectUrl + ", result:" + result);
    }
}
```

3. 将btrace工具和脚本上传到服务器，在服务器上执行如下格式的命令：

> 注意：需要配置环境变量BTRACE_HOME，和配置JDK是一样的。

```bash
btrace 进程ID 脚本文件名
```

4. 观察执行结果。



> 上面示例看起来懵的话，直接去看这个示例：https://www.cnblogs.com/wei-zw/p/9502274.html













### IDEA本地调试和远程调试

> 声明：
>
> - 前面9个部分，主要总结自 https://www.cnblogs.com/diaobiyong/p/10682996.html
> - 远程调试，主要整理自 https://www.jianshu.com/p/302dc10217c0
> - 著作权归相关作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



Debug用来追踪代码的运行流程，通常在程序运行过程中出现异常，启用Debug模式可以分析定位异常发生的位置，以及在运行过程中参数的变化；并且在实际的排错过程中，还会用到Remote Debug IDEA 。相比 Eclipse/STS效率更高，本文主要介绍基于IDEA的Debug和Remote Debug的技巧。



#### Debug开篇

首先看下IDEA中Debug模式下的界面：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107161800502-1145879883.png)

如上是在IDEA中启动Debug模式，进入断点后的界面，我这里是Windows，可能和Mac的图标等会有些不一样。就简单说下图中标注的8个地方：

- ① 以Debug模式启动服务，左边的一个按钮则是以Run模式启动。在开发中，我一般会直接启动Debug模式，方便随时调试代码
- ② 断点：在左边行号栏单击左键，或者快捷键Ctrl+F8 打上/取消断点，断点行的颜色可自己去设置
- ③ Debug窗口：访问请求到达第一个断点后，会自动激活Debug窗口；如果没有自动激活，可以去设置里设置，如下图Show debug window on breakpoint设置
- ④ 调试按钮：一共有8个按钮，调试的主要功能就对应着这几个按钮，鼠标悬停在按钮上可以查看对应的快捷键。在菜单栏Run里可以找到同样的对应的功能，如下图Run的设置
- ⑤ 服务按钮：可以在这里关闭/启动服务，设置断点等
- ⑥ 方法调用栈：这里显示了该线程调试所经过的所有方法，勾选右上角的[Show All Frames]按钮，就不会显示其它类库的方法了，否则这里会有一大堆的方法
- ⑦ Variables：在变量区可以查看当前断点之前的当前方法内的变量
- ⑧ Watches：查看变量，可以将Variables区中的变量拖到Watches中查看



在设置里勾选Show debug window on breakpoint，则请求进入到断点后自动激活Debug窗口

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107162200477-30238584.png)

如果你的IDEA底部没有显示工具栏或状态栏，可以在View里打开，显示出工具栏会方便我们使用可以自己去尝试下这四个选项

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107162200606-740734901.png)

在菜单栏Run里有调试对应的功能，同时可以查看对应的快捷键

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107162200655-926651524.png)







#### 基本用法&快捷键

Debug调试的功能主要对应着上面开篇中图一的4和5两组按钮：

> **首先说第一组按钮，共8个按钮**，从左到右依次如下：

![image-20240107163131696](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107163129667-1351291426.png)

- `Show Execution Point` (Alt + F10)：如果你的光标在其它行或其它页面，点击这个按钮可跳转到当前代码执行的行
- `Step Over` (F8)：步过，一行一行地往下走，如果这一行上有方法不会进入方法
- `Step Into` (F7)：步入，如果当前行有方法，可以进入方法内部，一般用于进入自定义方法内，不会进入官方类库的方法，如第25行的put方法
- `Force Step Into` (Alt + Shift + F7)：强制步入，能进入任何方法，查看底层源码的时候可以用这个进入官方类库的方法
- `Step Out` (Shift + F8)：步出，从步入的方法内退出到方法调用处，此时方法已执行完毕，只是还没有完成赋值
- `Drop Frame` (默认无)：回退断点，后面章节详细说明
- `Run to Cursor` (Alt + F9)：运行到光标处，你可以将光标定位到你需要查看的那一行，然后使用这个功能，代码会运行至光标行，而不需要打断点
- `Evaluate Expression` (Alt + F8)：计算表达式，后面章节详细说明



> **第二组按钮，共7个按钮**，从上到下依次如下：

![image-20240107163408102](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107163406115-376357611.png)

- `Rerun 'xxxx'`：重新运行程序，会关闭服务后重新启动程序
- `Modify Run Configuration` ：更新程序，一般在你的代码有改动后可执行这个功能。而这个功能对应的操作则是在服务配置里，如下“更新程序”图一
- `Resume Program` (F9)：恢复程序，比如，你在第20行和25行有两个断点，当前运行至第20行，按F9，则运行到下一个断点(即第25行)，再按F9，则运行完整个流程，因为后面已经没有断点了
- `Pause Program`：暂停程序，启用Debug目前没发现具体用法
- `Stop 'xxx'` (Ctrl + F2)：连续按两下，关闭程序。有时候你会发现关闭服务再启动时，报端口被占用，这是因为没完全关闭服务的原因，你就需要查杀所有JVM进程了
- `View Breakpoints` (Ctrl + Shift + F8)：查看所有断点，后面章节会涉及到
- `Mute Breakpoints`：哑的断点，选择这个后，所有断点变为灰色，断点失效，按F9则可以直接运行完程序。再次点击，断点变为红色，有效。如果只想使某一个断点失效，可以在断点上右键取消Enabled，则该行断点失效



> **更新程序**

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107165123119-1447224350.png" alt="img" style="zoom:50%;" />



- `On 'Update' actions`，执行更新操作时所做的事情，一般选择`'Update Classes and resources'`，即更新类和资源文件

一般配合热部署插件会更好用，如JRebel，这样就不用每次更改代码后还要去重新启动服务。如何激活JRebe在前面声明的原文连接中。

- `On frame deactivation`，在IDEA窗口失去焦点时触发，即一般你从idea切换到浏览器的时候，idea会自动帮你做的事情，一般可以设置Do nothing，频繁切换会比较消耗资源的



![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107163518455-48695815.png)





#### 变量查看

> 在Debug过程中，跟踪查看变量的变化是非常必要的，这里就简单说下IDEA中可以查看变量的几个地方，相信大部分人都了解。

如下，在IDEA中，参数所在行后面会显示当前变量的值

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107171840346-647263163.png" alt="img" style="zoom:67%;" />

光标悬停到参数上，显示当前变量信息。点击打开详情如下图。我一般会使用这种方式，快捷方便

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107171950623-770321482.png)

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172015539-566466136.png" alt="img" style="zoom:67%;" />



在Variables里查看，这里显示当前方法里的所有变量

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172037266-2004039260.png" alt="img" style="zoom:67%;" />

在Watches里，点击New Watch，输入需要查看的变量或者可以从Variables里拖到Watche里查看

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172102529-982642536.png" alt="img" style="zoom:67%;" />

如果你发现你没有Watches，可能在下图所在的地方

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172102586-1376733589.png)

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172126254-1548144072.png" alt="img" style="zoom:67%;" />





#### 计算表达式

> 在前面提到的计算表达式如下图的按钮，Evaluate Expression (Alt + F8) 可以使用这个操作在调试过程中计算某个表达式的值，而不用再去打印信息

![image-20240107172245056](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172242945-2048193484.png)

按Alt + F8或按钮，或者，你可以选中某个表达式再Alt + F8，弹出计算表达式的窗口，如下，回车或点击Evaluate计算表达式的值

这个表达式不仅可以是一般变量或参数，也可以是方法，当你的一行代码中调用了几个方法时，就可以通过这种方式查看查看某个方法的返回值

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172325093-1505246742.png" alt="img" style="zoom:50%;" />

设置变量，在计算表达式的框里，可以改变变量的值，这样有时候就能很方便我们去调试各种值的情况了不是

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172355974-140514076.png" alt="img" style="zoom:50%;" />









#### 智能步入

> 想想，一行代码里有好几个方法，怎么只选择某一个方法进入。之前提到过使用Step Into (Alt + F7) 或者 Force Step Into (Alt + Shift + F7)进入到方法内部，但这两个操作会根据方法调用顺序依次进入，这比较麻烦

那么智能步入就很方便了，智能步入，这个功能在Run -> Debugging Action里可以看到，Smart Step Into (Shift + F7)，如下图

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172757916-225542336.png" alt="image-20240107172759629" style="zoom:67%;" />

按Shift + F7，会自动定位到当前断点行，并列出需要进入的方法，如下图，点击方法进入方法内部

如果只有一个方法，则直接进入，类似Force Step Into

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172541112-442094890.png)





#### 断点条件设置

> 通过设置断点条件，在满足条件时，才停在断点处，否则直接运行。
>
> 通常，当我们在遍历一个比较大的集合或数组时，在循环内设置了一个断点，难道我们要一个一个去看变量的值？那肯定很累，说不定你还错过这个值得重新来一次。



> 设置当前断点的条件

在断点上右键直接**设置当前断点的条件**，如下图设置exist为true时断点才生效

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107172930886-1944017030.png)

> 查看所有断点

点击View Breakpoints (Ctrl + Shift + F8)，查看所有断点

- Java Line Breakpoints 显示了所有的断点，在右边勾选Condition，设置断点的条件
- 勾选Log message to console，则会将当前断点行输出到控制台，如下图二
- 勾选Evaluate and log，可以在执行这行代码时计算表达式的值，并将结果输出到控制台

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107173033831-1568590010.png" alt="img" style="zoom:50%;" />

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107173104369-694500543.png)

> 右边的Filters过滤

再说说右边的Filters过滤：这些一般情况下不常用，简单说下意思

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107175320122-221116701.png)

- Instance filters：实例过滤，输入实例ID(如下图中的实例ID)。
- Class filters：类过滤，根据类名过滤。
- Pass count：用于循环中，如果断点在循环中，可以设置该值，循环多少次后停在断点处，之后的循环都会停在断点处

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107175341201-1277031059.png)



> 异常断点：通过设置异常断点，在程序中出现需要拦截的异常时，会自动定位到异常行

如下图，点击+号添加Java Exception Breakpoints，添加异常断点然后输入需要断点的异常类

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107175639427-50690585.png)

之后可以在Java Exception Breakpoints里看到添加的异常断点

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107175708156-1160245560.png" alt="image-20240107175709961" style="zoom:67%;" />

这里添加了一个NullPointerException异常断点，出现空指针异常后，自动定位在空指针异常行

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107175644571-97607672.png" alt="img" style="zoom:50%;" />









#### 多线程调试

> 一般情况下，我们调试的时候是在一个线程中的，一步一步往下走。但有时候你会发现在Debug的时候，想发起另外一个请求都无法进行了？

那是因为IDEA在Debug时默认阻塞级别是ALL，会阻塞其它线程，只有在当前调试线程走完时才会走其它线程。可以在View Breakpoints里选择Thread，如下图，然后点击Make Default设置为默认选项。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107175902812-1640306601.png" alt="img" style="zoom:67%;" />

切换线程，在下图中Frames的下拉列表里，可以切换当前的线程，如下我这里有两个Debug的线程，切换另外一个则进入另一个Debug的线程

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107175941692-402629755.png" alt="img" style="zoom:50%;" />





#### 回退断点

> 在调试的时候，想要重新走一下流程而不用再次发起一个请求？

首先认识下这个**方法调用栈**，如下图，首先请求进入DemoController的insertDemo方法，然后调用insert方法，其它的invoke我们且先不管，最上面的方法是当前断点所在的方法。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107180058156-1502199982.png" alt="img" style="zoom:50%;" />



> 断点回退
>
> - 注意：断点回退只是重新走一下流程，之前的某些参数/数据的状态已经改变了的是无法回退到之前的状态的，如对象、集合、更新了数据库数据等等。

所谓的断点回退，其实就是回退到上一个方法调用的开始处，在IDEA里测试无法一行一行地回退或回到上一个断点处，而是回到上一个方法。

回退的方式有两种：一种是Drop Frame按钮，按调用的方法逐步回退，包括三方类库的其它方法。

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107180201997-273799138.png)

取消Show All Frames按钮会显示三方类库的方法。

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107180207294-910659732.png" alt="img" style="zoom:67%;" />

第二种方式，在调用栈方法上选择要回退的方法，右键选择Drop Frame，回退到该方法的上一个方法调用处，此时再按F9(Resume Program)，可以看到程序进入到该方法的断点处了

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107180318978-1898038030.png)







#### 中断Debug

> 想要在Debug的时候，中断请求，不要再走剩余的流程了？

有些时候，我们看到传入的参数有误后，不想走后面的流程了，怎么中断这次请求呢(后面的流程要删除数据库数据呢....)，难道要关闭服务重新启动程序？嗯，我以前也是这么干的。

确切的说，我也没发现可以直接中断请求的方式(除了关闭服务)，但可以通过Force Return，即强制返回来避免后续的流程，如图：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107180730424-1528864257.png)

点击Force Return，弹出Return Value的窗口，我这个方法的返回类型为Map，所以，我这里直接返回 results，来强制返回，从而不再进行后续的流程或者你可以`new HashMap<>()`

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107180730686-386474665.png" alt="img" style="zoom:67%;" />







#### 远程调试(Remote Debug)

> 有时候，本地调试的时候没有问题，打包部署到测试环境的时候却爆出一堆莫名其妙的问题，这时该怎么办呢？

##### 使用特定JVM参数运行服务端代码

要让远程服务器运行的代码支持远程调试，则启动的时候必须加上特定的JVM参数，这些参数是：

```bash
-Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=${debug_port}
```

其中的`${debug_port}`是用户自定义的，为debug端口，本例以5555端口为例。

本人在这里踩过一个坑，必须要说一下：在使用公司内部的自动化部署平台NDP进行应用部署时，该平台号称支持远程调试，只需要在某个配置页面配置一下调试端口号（没有填写任何IP相关的信息），并且重新发布一下应用即可。事实上也可以发现，上述JVM参数中唯一可变的就是${debug_port}。但是实际在本地连接时发现却始终连不上5555 的调试端口，仔细排查才发现，下面截取了NDP发布的应用所有JVM参数列表中与远程调试相关的JVM启动参数如下：

```bash
-Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=127.0.0.1:5555
```

将address设置为127.0.0.1:5555，表示将调试端口限制为本地访问，远程无法访问，这个应该是NDP平台的一个bug，我们在自己设置JVM的启动参数时也需要格外注意。

如果只是临时调试，在端口号前面不要加上限制访问的IP地址，调试完成之后，将上述JVM参数去除掉之后重新发布下，防范开放远程调试端口可能带来的安全风险。





##### 本地连接远程服务器debug端口

打开IDEA，在顶部靠右的地方选择”Edit Configurations…”，进去之后点击+号，选择”Remote”，按照下图的只是填写红框内的内容，其中Name填写名称，这里为remote webserver，host为远程代码运行的机器的ip/hostname，port为上一步指定的debug_port，本例是5555。然后点击Apply，最后点击OK即可

<img src="https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107181410106-1375117683.png" alt="img" style="zoom:67%;" />

现在在上一步选择”Edit Configurations…”的下拉框的位置选择上一步创建的remote webserver，然后点击右边的debug按钮(长的像臭虫那个)，看控制台日志，如果出现类似“Connected to the target VM, address: ‘xx.xx.xx.xx:5555’, transport: ‘socket’”的字样，就表示连接成功过了，我这里实际显示的内容如下：

```bash
Connected to the target VM, address: '10.185.0.192:15555', transport: 'socket'
```





##### 设置断点，开始调试

> 特别注意：用于远程debug的代码必须与远程部署的代码完全一致，不能发生任何的修改，否则打上的断点将无法命中，切记切记！

远程debug模式已经开启，现在可以在需要调试的代码中打断点了，比如：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107202903769-1474659159.png)

如图中所示，如果断点内有√，则表示选取的断点正确

现在在本地发送一个到远程服务器的请求，看本地控制台的bug界面，划到debugger这个标签，可以看到当前远程服务的内部状态（各种变量）已经全部显示出来了，并且在刚才设置了断点的地方，也显示了该行的变量值

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107202904056-1679768182.png)

























# 附加：Java应用在线调试Arthas整理

> 参考资料：
>
> - https://www.cnblogs.com/muxuanchan/p/10097639.html
> - https://www.cnblogs.com/yougewe/p/10770690.html
> - https://help.aliyun.com/document_detail/112975.html



### Arthas简介

> 在学习Arthas之前，推荐先看后面的美团技术团队的 [Java 动态调试技术原理](#Java 动态调试技术原理)，这样你会对它最底层技术有个了解。可以看下文中最后有个对比图：Greys(Arthas也是基于它做的二次开发)和Java-debug-tool



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
curl -O https://alibaba.github.io/arthas/arthas-boot.jar	# 也可以选择去官网下载jar然后丢到服务器中

Java -jar arthas-boot.jar
```



### Arthas 案例展示

#### Dashboard

> 官方地址：https://arthas.aliyun.com/doc/dashboard.html

![dashboard](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932837-1171725582.png)





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

![trace](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932799-391739634.png)





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

> 官方地址：https://arthas.aliyun.com/doc/web-console.html

![web console](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932781-252621157.png)





### Arthas 命令集

#### 基础命令

- help——查看命令帮助信息
- [cat](https://arthas.aliyun.com/doc/cat.html)——打印文件内容，和linux里的cat命令类似
- [grep]](https://arthas.aliyun.com/doc/grep.html)——匹配查找，和linux里的grep命令类似
- [pwd](https://arthas.aliyun.com/doc/pwd.html)——返回当前的工作目录，和linux命令类似
- cls——清空当前屏幕区域
- session——查看当前会话的信息
- [reset](https://arthas.aliyun.com/doc/reset.html)——重置增强类，将被 Arthas 增强过的类全部还原，Arthas 服务端关闭时会重置所有增强过的类
- version——输出当前目标 Java 进程所加载的 Arthas 版本号
- history——打印命令历史
- quit——退出当前 Arthas 客户端，其他 Arthas 客户端不受影响
- stop/shutdown——关闭 Arthas 服务端，所有 Arthas 客户端全部退出
- [keymap](https://arthas.aliyun.com/doc/keymap.html)——Arthas快捷键列表及自定义快捷键





#### JVM相关

- [dashboard](https://arthas.aliyun.com/doc/dashboard.html)——当前系统的实时数据面板
- [thread](https://arthas.aliyun.com/doc/thread.html)——查看当前 JVM 的线程堆栈信息
- [JVM](https://arthas.aliyun.com/doc/jvm.html)——查看当前 JVM 的信息
- [sysprop](https://arthas.aliyun.com/doc/sysprop.html)——查看和修改JVM的系统属性
- [sysenv](https://arthas.aliyun.com/doc/sysenv.html)——查看JVM的环境变量
- [vmoption](https://arthas.aliyun.com/doc/vmoption.html)——查看和修改JVM里诊断相关的option
- [logger](https://arthas.aliyun.com/doc/logger.html)——查看和修改logger
- [getstatic](https://arthas.aliyun.com/doc/getstatic.html)——查看类的静态属性
- [ognl](https://arthas.aliyun.com/doc/ognl.html)——执行ognl表达式
- [mbean](https://arthas.aliyun.com/doc/mbean.html)——查看 Mbean 的信息
- [heapdump](https://arthas.aliyun.com/doc/heapdump.html)——dump Java heap, 类似jmap命令的heap dump功能





#### Class/Classloader相关

- [sc](https://arthas.aliyun.com/doc/sc.html)——查看JVM已加载的类信息
- [sm](https://arthas.aliyun.com/doc/sm.html)——查看已加载类的方法信息
- [jad](https://arthas.aliyun.com/doc/jad.html)——反编译指定已加载类的源码
- [mc](https://arthas.aliyun.com/doc/mc.html)——内存编绎器，内存编绎`.Java`文件为`.Class`文件
- [redefine](https://arthas.aliyun.com/doc/redefine.html)——加载外部的`.Class`文件，redefine到JVM里
- [dump](https://arthas.aliyun.com/doc/dump.html)——dump 已加载类的 byte code 到特定目录
- [Classloader](https://arthas.aliyun.com/doc/classloader.html)——查看Classloader的继承树，urls，类加载信息，使用Classloader去getResource





#### monitor/watch/trace相关

> 请注意，这些命令，都通过字节码增强技术来实现的，会在指定类的方法中插入一些切面来实现数据统计和观测，因此在线上、预发使用时，请尽量明确需要观测的类、方法以及条件，诊断结束要执行 `shutdown` 或将增强过的类执行 `reset` 命令

- [monitor](https://arthas.aliyun.com/doc/monitor.html)——方法执行监控
- [watch](https://arthas.aliyun.com/doc/watch.html)——方法执行数据观测
- [trace](https://arthas.aliyun.com/doc/trace.html)——方法内部调用路径，并输出方法路径上的每个节点上耗时
- [stack](https://arthas.aliyun.com/doc/stack.html)——输出当前方法被调用的调用路径
- [tt](https://arthas.aliyun.com/doc/tt.html)——方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测





#### options

- [options](https://arthas.aliyun.com/doc/options.html)——查看或设置Arthas全局开关



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

![trace](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107160932807-302497775.jpg)

源码理解可以这篇文章：https://yq.aliyun.com/articles/704228





# [#](#Java 动态调试技术原理)Java动态调试技术原理

> 本文转载自 美团技术团队胡健的[Java 动态调试技术原理及实践](https://tech.meituan.com/2019/11/07/java-dynamic-debugging-technology.html), 通过学习Java agent方式进行动态调试了解目前很多大厂开源的一些基于此的调试工具





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

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511352-1064313242.png)

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

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511391-960675052.png)

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

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511371-473664371.png)

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

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511352-304047437.png)

如图所示，当调试者发出调试命令之后，Java-debug-tool会识别命令并判断是否需要进行字节码增强，如果命令需要增强字节码，则判断当前类+当前方法是否已经被增强过。上文已经提到，字节码替换是有一定损耗的，这种具有损耗的操作发生的次数越少越好，所以字节码替换操作会被记录起来，后续命令直接使用即可，不需要重复进行字节码增强，字节码增强还涉及多个调试客户端的协同工作问题，当一个客户端增强了一个类的字节码之后，这个客户端就锁定了该字节码，其他客户端变成只读，无法对该类进行字节码增强，只有当持有锁的客户端主动释放锁或者断开连接之后，其他客户端才能继续增强该类的字节码

字节码增强模块收到字节码增强请求之后，会判断每个增强点是否需要插桩，这个判断的根据就是上文提到的插桩配置，之后字节码增强模块会生成新的字节码，Java-debug-tool将执行字节码替换操作，之后就可以进行调试数据收集了

经过字节码增强之后，原来的方法中会插入收集运行时数据的代码，这些代码在方法被调用的时候执行，获取到诸如方法入参、局部变量等信息，这些信息将传递给数据收集装置进行处理。数据收集的工作通过Advice完成，每个客户端同一时间只能注册一个Advice到Java-debug-tool调试模块上，多个客户端可以同时注册自己的Advice到调试模块上。Advice负责收集数据并进行判断，如果当前数据符合调试命令的要求，Java-debug-tool就会卸载这个Advice，Advice的数据就会被转移到Java-debug-tool的命令结果处理模块进行处理，并将结果发送到客户端





#### Advice的工作方式

Advice是调试数据收集器，不同的调试策略会对应不同的Advice。Advice是工作在目标JVM的线程内部的，它需要轻量级和高效，意味着Advice不能做太过于复杂的事情，它的核心接口“match”用来判断本次收集到的调试数据是否满足调试需求。如果满足，那么Java-debug-tool就会将其卸载，否则会继续让他收集调试数据，这种“加载Advice” -> “卸载Advice”的工作模式具备很好的灵活性

关于Advice，需要说明的另外一点就是线程安全，因为它加载之后会运行在目标JVM的线程中，目标JVM的方法极有可能是多线程访问的，这也就是说，Advice需要有能力处理多个线程同时访问方法的能力，如果Advice处理不当，则可能会收集到杂乱无章的调试数据。下面的图片展示了Advice和Java-debug-tool调试分析模块、目标方法执行以及调试客户端等模块的关系

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511371-1082875809.png)

Advice的首次挂载由Java-debug-tool的命令处理器完成，当一次调试数据收集完成之后，调试数据处理模块会自动卸载Advice，然后进行判断，如果调试数据符合Advice的策略，则直接将数据交由数据处理模块进行处理，否则会清空调试数据，并再次将Advice挂载到目标方法上去，等待下一次调试数据。非首次挂载由调试数据处理模块进行，它借助Advice按需取数据，如果不符合需求，则继续挂载Advice来获取数据，否则对调试数据进行处理并返回给客户端





### Java-debug-tool的命令设计与实现

#### 命令执行

上文已经完整的描述了Java-debug-tool的设计以及核心技术方案，本小节将详细介绍Java-debug-tool的命令设计与实现。首先需要将一个调试命令的执行流程描述清楚，下面是一张用来表示命令请求处理流程的图片：

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511335-1530405067.png)

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

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155511372-1764753977.png)





### Java-debug-tool与同类产品对比分析

Java-debug-tool的同类产品主要是greys，其他类似的工具大部分都是基于greys进行的二次开发，所以直接选择greys来和Java-debug-tool进行对比

![img](https://img2023.cnblogs.com/blog/2421736/202401/2421736-20240107155512151-635229710.jpg)

本文详细剖析了Java动态调试关键技术的实现细节，并介绍了我们基于Java动态调试技术结合实际故障排查场景进行的一点探索实践；动态调试技术为研发人员进行线上问题排查提供了一种新的思路，我们基于动态调试技术解决了传统断点调试存在的问题，使得可以将断点调试这种技术应用在线上，以线下调试的思维来进行线上调试，提高问题排查效率







# GC调优















