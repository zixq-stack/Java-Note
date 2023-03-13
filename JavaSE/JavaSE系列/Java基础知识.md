# Java环境搭建问题

## 下载JDK

JDK指的是： `Java Development Kits` ， 即：java开发工具包

**我用时最稳定的版本是JDK8，至于具体是JDK8点几的无所谓**，这个版本最稳定，也是应用人员最多的

直接在0racle官网中下载即可：[官网点这里](https://www.oracle.com/java/technologies/downloads/#java8)

选择自己对应的操作系统就行了，linux就选linux，window就选window

当然：上面这是官网下载，很慢，这里有一个另外的下载地址：http://www.codebaoku.com/jdk/jdk-index.html

**建议**：最好是自己去Oracle官网摸索着找一下





## 使用exe的方式安装JDK

**傻瓜式安装**，下一步下一步即可，当然选择安装路径的时候，可以根据自己的想法来

**只需**：记住自己安装的路径即可

**建议**：把所有的配置放在一个目录文件夹下

**注**：`.exe`的安装方式，在安装JDK的时候，会弹出第二个框( 即：安装JRE )，这个是一个公开的JRE，不用安装也可以，因为JDK中有JRE，JRE中有JVM，如图所示：

![image](https://img2023.cnblogs.com/blog/2421736/202303/2421736-20230310170351633-1835145861.png)





## 配置环境变量问题

把JDK安装好之后，其实开发工具包已经算是弄好了，如图所示，我现在不配置环境变量，一样可以用JDK，只需要进到JDK的安装目录的bin之中，`java、javac、java -version`这些命令照样可以用

![image](https://img2023.cnblogs.com/blog/2421736/202303/2421736-20230310170351633-1964964351.png)

 

**但是为什么需要配置环境变量？**

- 因为每次都这样去找目录不麻烦吗？所以就需要配置环境变量，自报家门，因此说明一下：网上说的以及很多认为的安装JDK就是下载好了，然后安装，以及配置环境变量。安装JDK和配置环境变量是两码事，我下载安装好了JDK，照样可以进行相关的操作，只是很麻烦而已，所以需要配置环境变量来自报家门




**需要配置的环境变量以及说明**

- 注：win10版本、win7有细微区别，以下方式是win10，win7自行百度

 



## JAVA_HOME问题

找到计算机—>右键，选择属性—>选择高级系统设置—>选择环境变量—>在系统变量（ 用户变量和系统变量的区别就是使用范围的大小，就字面意思 ）里面新建一个JAVA_HOME，然后路径就是JDK的安装路径，不用整到什么bin目录下，如我的：`D:\Install\JDK8`，这样即可

![image](https://img2023.cnblogs.com/blog/2421736/202303/2421736-20230310170351800-609341336.png)







## path问题

这个就是给系统自报家门，所以在这里面只需要新建

1. `%JAVA_HOME%\bin`
2. ``%JAVA_HOME%\jre\bin` &nbsp; &nbsp; &nbsp; 其中：
	- `% %` 就是引用的意思，即：引用了前面配置的`JAVA_HOME`的目录，所以才在前面说：不需要精确到JDK的bin目录下，这样在这里直接引用不就方便多了

![image](https://img2023.cnblogs.com/blog/2421736/202303/2421736-20230310170351689-861394473.png)







## CLASSPATH问题

按如下配置就行了

![image](https://img2023.cnblogs.com/blog/2421736/202303/2421736-20230310170351638-1319915905.png)



**注**：在这个`CLASSPATH`的目录中有一个 `“ . "`，这个符号要不要都无所谓，因为它表示的就是：从当前目录中加载class文件（是加载class文件），系统默认也是这个

**最后：弄好之后一路点确认即可**

 





# 关于Java的数据类型

**数据类型：是<span style = "color:blue">存储一个指定类型</span>的数据**

 

> **须知（强制要求）：标识符 / 起名字的命名规范**
>
> 1. 只能有数字、字母、下划线( 即： `_`) 、 `$`组成，数字不能作为开始
> 2. 起的名字不能和Java系统的关键字冲突（ 就是java中已经用了的名字 ），如：`String`、`int`......
> 3. 多个单词组成的名字采用驼峰命名法
> 	 -  类名、接口名：每个单词首字母大写
> 	 -  方法名、属性（变量）名：第一个单词首字母小写，其余单词首字母大写
> 4. 需要见名知意
> 5. 最好别用中文以及拼音来命名，但“某些特定场景”下可以用，如：alibaba国际通用的
> 6. 常量名（包括静态常量）：全部字母大写，多个单词通过`_`拼接
> 7. 包名：全部字母小写(VO、DTO、POJO、DO、PO、DO例外)，不可出现复数形式
> 	- 注：别和关键字冲突，因为关键字也是小写 
> 8. 更多开发规范，请去下载 [阿里开发规范手册](https://developer.aliyun.com/article/888697?spm=5176.21213303.J_6704733920.7.749253c9agCrkV&scm=20140722.S_community%40%40%E6%96%87%E7%AB%A0%40%40888697._.ID_community%40%40%E6%96%87%E7%AB%A0%40%40888697-RL_%E5%BC%80%E5%8F%91%E6%89%8B%E5%86%8C%E9%BB%84%E5%B1%B1%E7%89%88-LOC_main-OR_ser-V_2-RK_rerank-P0_0)

 





##  基本数据类型

**注：java中和符号相关的一定是英文输入法状态下打出来的，如：双引号、单引号、括号......**

 



### 01 整型

从小到大为如下顺序

| 关键字 | 含义   | 存储      | 范围                                                         |
| ------ | ------ | --------- | ------------------------------------------------------------ |
| byte   | 字节型 | 1字节8位  | [ -128, 127 ]，也可以这么记：-2^7=-128 到 2^7-1=127          |
| short  | 短整型 | 2字节16位 | [-32768, 32767]，也可以这么记：-2^15=-32768 到  2^15-1=32767 |
| int    | 整型   | 4字节32位 | [-21 4748 3648, 21 4748 3647]，也可以这么记：-2^31=-2147483648 到 2^31=2147483647 |
| long   | 长整型 | 8字节64位 | [-922 3372 0368 5477 5808, 922 3372 0368 5477 5805]，也可以这么记：-2^63=-9223372036854775808 到  2^63-1=9223372036854775807 |

 


其中：大小的划分是由于冯洛伊曼提出的计算机存储机制（ 即：二进制 ）所决定的


例子：假如有这么一个二进制

| 0    | 1    | 0    | 1    |
| ---- | ---- | ---- | :--- |

里面的每一个数字为一个bit位：1 byte = 8 bit

整型里面的这个大小顺序就是：按照这个类型所拥有的多少个bit位来决定的（ 具体换算关系：百度，滤过 ）

 

### 02 浮点数

按精确度从小到大

| 类型   | 名字         | 存储  |
| ------ | ------------ | ----- |
| float  | 单精度浮点数 | 4字节 |
| double | 双精度浮点数 | 8字节 |

它的大小划分标准是根据IEEE754标准来的（ 简称IE754 )，但是这个标准有bug，即：对于冯洛伊曼的这种bit位存储“不太适合”

举个栗子：假如有一个数利用冯洛伊曼的这种bit位存储来存储，理论上应该是( 这个数是假如的 ）：1.001011


这种数就会出现一种情况：就是后面那半截1011，在存储的时候，由于冯洛伊曼的bit位存储方式，位数刚好满了呢

也就是成为1.000000，那后面这个1011就没有位置可以放下了，所以就没了

所以float和double的区别就是：后面的这个尾数可以存储的更多一点

因此：要是在编程中见到那种有一个总是很接近于一个数，但是就是不等于的时候，不用意外，就是这个原因导致的

 

### 03 布尔型 boolean

**就两种true和false**：主要就是为了判断逻辑关系的，非真即假嘛





### 04 字符型  char

**对于表示单个字符：** 就是单引号`'	'`括起来的一个信息，啥都可以，如：'1'、'a‘、’$'、、、、、，只要是一个就行

 

**表示Unicode字符（了解）：** 用十六进制表示，范围从`\u0000` 到 `\uFFFF`----

| **转义序列** | **名称** | **UNICODE码** |
| ------------ | -------- | ------------- |
| `\b`         | 退格     | \u0008        |
| `\t`         | 制表     | \u0009        |
| `\n`         | 换行     | \u000a        |
| `\r`         | 回车     | \u000d        |
| `\"`         | 双引号   | \u0022        |
| `\'`         | 单引号   | \u0027        |
| `\\`         | 反斜杠   | \u005c        |

 





## 引用数据类型

**一句话：除了基本数据类型以外的数据类型都是引用数据类型**（ 后续自定义的类也是 ），如：字符串String、类class（接口interface、抽象类abstract）、注解@xxxxx、枚举enum

 



## 变量与常量

**额外知识：注释，指的是机器不会编译，只是对写的代码起解释说明的，给人看的**

```txt
	//			单行注释			使用了这个表明： // 这符号后面这一行的内容不会被编译

	/* */		多行注释			使用了这个表明：/*  这符号中间的内容不会被编译  */

	/** */		文档注释		用这注释备注的地方，后续可以把它生成出来，成为一个文档
											典型的就是：后续学工具类时的那个API文档
```


**建议**：写注释的时候最好空一格再写注释（ 因为考虑到数据库中写注释的习惯 ），如：`// 写注释内容`

 





### 01 变量

**定义**：指的是定义并赋值之后还可以再发生变化（ 至于怎么发生改变，到了构造器就知道了 ）

**声明方式 与 赋值：**
```txt
数据类型 变量名 [= 值]
```
其中：`[]` 表示不是必有，但是不赋值需要注意下述的注意1

 

举个栗子：

```java
/**
 * 这种是类的创建，先这么玩儿，到了面向对象思想编程时会做详细说明
 */
public class Test{

	/**
	 * 主方法入口，java运行的时候首先找的就是这个，这是死的，目前死记就行，后续到了类的方法时就明白了
	 */
	public static void main（ String[] args ){

		byte b = 126；

		short s = 310000；

		int i = 2000000000；
		long = 2200000000L；

		float f = 3.4F
		/*
			注：定义long 、float、double需要注意，在赋值的时候需要在数值后面加上相应数据类型的首字母
				（ 大写和小写都可以，建议大写，因为小写可能会不好区分 ）
				加这个的原因：
					因为赋的数值，系统默认是int类型的，
					所以赋的这个值原本是Long、float....类型的，但是int类型装不下，所以会造成数字异常
		*/

		double d = 9.1415D；

		boolean  b2 = true；

		// 注：char c = ''；这种是不可以的( 即：单引号中什么都没有 )，会报错，但是String这么玩就可以
		char c = 'a'
	}
}
```

 



> **注意1：变量定义在方法内部的话，必须先赋值，才能使用（ 即：局部变量 ）**

举个栗子：

```java
public class Test{

	public static void main( String[] args ){

		// int i;
		// System.out.println( i );// 这种不得吃，没赋值

		int m = 10;
		System.out.println( m );// 这种就得吃
	}
}
```

 


> **注意2：如果变量定义在方法外部，则就算没有赋值，也可以使用，因为就算没有赋值，系统也会默认的帮忙赋值（ 即：全局变量 ）**

举个栗子：

```java
public class Test{

	String name;

	public static void main( String[] args ){

		System.out.println( name );// 这种得吃

		name = "邪公子";
		System.out.println( name );// 这种还得吃
  }
}
```

 





### 02 常量

**定义：指的是定义并赋值之后不可以再发生变化** ，所有访问这个常量得到的 都是同样的值

 

**常量是怎么玩的？**

- 就是在数据类型前面加上一个特征修饰符`final`（ 目前就这么囫囵吞枣记就行，因为这是特征修饰符，后续在修饰符中会讲到 ）

举个栗子：

```java
public class Test{
	// 这表明以后谁来访问这个name变量的时候这个值都是 邪公子
	final String MY_NAME = "邪公子";
}
```







## 数据类型的转换

### 01 对于同一种大数据类型

指的是：都是属于基本数据类型 / 引用数据类型

 



### （一）小数据类型也相同

如：都是整型 / 浮点型，**秘诀就是小的可以转成大的（ 隐式转换 ），但是大的转成小的不可行，需要强制转换**


```java
public class Test{

	public static void main（ String[] args ）{

		int i = 10；
		double d = i；// 这种是可行的，因为int的bit存储位比double的小，所以小的转成大的是可以行的（大的装得下小的嘛）

		// int number = 20；
		// byte b = number；// 这种是不可行的，因为小的空间怎么可能装得下大的

		 int number = 300；
		 byte b = （ byte ）number；// 这样强制转换之后就可行了，但是：会造成精度的丢失，即：数据不一定对
	}
}
```





### 02 其他类型之间的转换

直接强制转换即可

```java
public class TypeConversion{

	public static void main( String[] args ){

		  int i = 10;
		  double d = ( double )i;
		  System.out.println( d );
	}
}
```

**注意：强制类型转换虽然好用，但是可能会造成数据的丢失，所以不是什么都可以强转的**



 



# 运算符

## 算术运算符 / 单目运算符

是几目 是看这个运算符需要几个操作数来参与才是完整的表达式，如：+1、-1，就只需要一个操作数就完了，一共有如下这些

| +    | -    | *    | /    | %    |
| ---- | ---- | ---- | ---- | ---- |





## 复合赋值运算符 / 双目运算符

| ++   | --   | +=   | -=   | *=   | /=   |
| ---- | ---- | ---- | ---- | ---- | ---- |





**注：`++` 和 `--` 运算符的问题 , 两句话：**

- 如果对于操作数自己本身来说：如a嘛，无论是`++a` 还是`--a`，最后a的结果都是`a+1`

- 如果对于一个表达式来说（ 就是有操作数与运算符组成的式子 ），**`++a`有及时性，即：先赋值再运算**；而 **`a --` 有延时性，即：先运算再赋值**





举个栗子：

```java
public class Operator{
	public static void main（ String[] args ){

		int m = 2,k = 4,L = 3;

		System.out.println( m++ + --k + L++ + --m - k++ + --L );// 问：答案为多少？

		/*
		  分析一下
			  初始值：m = 2、k = 4、L = 3
			  进行计算（注意：数值的变化）：2 + 3 + 3 + 2 - 3 + 3 = 13　　　 所以最终结果就是13
			  每一次变量运算完之后，数值的变化：
				前半截 m -> 从2变为3（延时性，先用初始值进行运算，再进行重新赋值）、k -> 从4变为3（及时性，立即赋值并参与运算）、L -> 从3变为4
				后半截（在前半截变化值的基础上进行变化） m -> 从3变为2、k -> 从4变为5、L -> 从4变为3
		*\
	}
}
```

 



## 关系运算符（ 也是双目运算符 ）


| >    | <    | >=   | <=   | ==   | !=   | %=   | /=   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |

 



## 逻辑运算符


| &&   | \|\| | &    | \|   | !    | ^    | <<   | >>   | >>>  |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |





**`&&`：与运算。有假则假**

**`||`：或运算，有真则真**

**`&`：按位与**

**&& 和 &的区别**

- java 中 && 和 & 都是表示与的逻辑运算符，都表示逻辑运输符 and，当两边的表达式都为 true 的时候，整个运算结果才为 true，否则为 false

- & 直接操作整数基本类型，而 && 不行。按位与运算符 “&” 是双目运算符。其功能是参与运算的两数各对应的二进位相与。只有对应的两个二进位都为 1 时，结果位才为 1。参与运算的两个数均以补码出现。如：0x31 & 0x0f 的结果为 0x01

- && 也叫短路与，&& 有短路效应，即：当第一个布尔运算为 false，第二个布尔运算不执行。而 & 运算符没有。如：对于 `if (str != null && !str.equals (“”))` 表达式，当 str 为 null 时，后面的表达式不会执行，所以不会出现 `NullPointerException`，如果将 && 改为 &，则会抛出 `NullPointerException` 异常。 `If (x==33 & ++y>0)` y 会增长， `If (x==33 && ++y>0)` 不会增长





**插入内容：&的细节。新手看不懂可以跳过，只记住下面的结论即可**

- 对于 `a % b` 来说，如果 b 满足2的整数次方，那么有如下的计算公式

> a % b = a & (b-1)		将 b 换为 2^n ，就变成如下的样子
> a % 2^n = a & 2^n - 1
> 这种 & 也叫取模运算，当然：%也叫取模运算，但：& 比 %更高效

此时 `&` 比 `%`更高效。因为 `b^n -1` 相当于一个“低位掩码” —— 这个掩码的低位最好全是 1，这样 & 操作才有意义，否则结果就肯定是 0，那么 & 操作就没有意义了

举个例子：a = 14，b = 8，也就是 n = 3，即：b = 2^3

```txt
14 % 8

4 的二进制为 1110		8 的二进制 1000			8-1 = 7 的二进制为 0111

110 & 0111 = 0110
即：0110 = 0*2^0 + 1*2^1 +1*2^2 + 0*2^3 = 0 + 2 + 4 + 0 = 6

而 14 % 8 刚好也等于 6
```


- 在后续学一个很重要，以后经常打交道，面试也喜欢问题的一个东西：`HashMap`，底层就用了上面的技巧

```java
// 假如有如下代码，目前看不懂没关系，复制下去走一下
HashMap<String, Integer> hashMap = new HashMap<>();
hashMap.put("紫", 18);	// 进行源码，光标放到put，按ctrl+b

// 会进入如下的方法 key = "紫"，value=18
public V put(K key, V value) {
	return putVal(hash(key), key, value, false, true);
}
// 在这里：调用putVal()之前就进入了一个很重要的方法hash()

// 继续ctrl+b 进入hash()
static final int hash(Object key) {
	int h;
	// 重点：(h = key.hashCode()) ^ (h >>> 16) 这就是计算hash值的方式，先不深究
	// 后续学习hashMap时自己追一下，这个hash 方法就是为了增加随机性，让数据元素更加均衡的分布，减少哈希碰撞
	return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

// 回到上面的putVal()，按ctrl+alt+左方向键即可，然后进入putVal()
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
		// 其他的都别看，别搭理，看不懂的，本次的重点是：p = tab[i = (n - 1) & hash]
		// 即：(n - 1) & hash 这里就应用了上面说的 & 技巧，也可称为：取模运算，这里的n就是数组长度length
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```


现在回来再看：

> a % 2^n = a & 2^n - 1
> 		  (n - 1) & hash

这就解释了后续学hashMap时，为什么它的数组长度取的是 2的整数次幂的原因

2 的整次幂刚好是偶数，偶数-1 是奇数，奇数的二进制最后一位是 1，保证了 hash &(length-1) 的最后一位可能为 0，也可能为 1（这取决于 h 的值），即 & 运算后的结果可能为偶数，也可能为奇数，这样便可以保证哈希值的均匀性。即：**hashMap的 hash() 方法就是为了增加随机性，让数据元素更加均衡的分布，减少哈希碰撞**



 



**`|` : 按位或**

- 原理和按位与一样的

- | 和 || 的区别与&、&&的区别一样，||也会产生短路

 


**`^`：异或运算**

- 只要参与运算的两个表达式是相反的就得行，否则不得行，即：若，a > 0为真，b < 0为假，两个结果异或，就为真





**`<<` : 按位左移
`>>`：按位右移**

- 这个是用的二进制，就是把数转化为二进制，然后对二进制进行左移或者右移，移动之后空的部分用0填充(**正数用0填充，负数用1填充**)

- 另：`<<` 相当于是：原来的数（ 十进制的那个 ） 乘  2 ^ 需要移动的数 次方
	- 即得到：左位移之后的十进制数是什么

- 而：`>>` 相当于是：原来的数（十进制的那个） /  2 ^ 移动的数次幂
	- 即：右位移之后的十进制数是多少

举个栗子：

```json
	6 << 2 = 24 ———— 6 * 2 ^ 2 = 6 * 4 = 24

	6在底层中的样子
		00000000 00000000 00000000 00000110

		左移两位之后
		000000 00000000 00000000 0000011000

	右位移是一样的原理
```





**`>>>` ： 无符号右位移**

- 就是不保留符号位，然后就和前面的`>>`和`<<`一样的原理了
- **注**：负数的符号位不会保留（ 正数的原码和补码一样，所以不考虑 ）


举个栗子：

```json
	-6 >>> 1
		11111111 11111111 11111111 11111010 ————往右移1位

		?11111111 11111111 11111111 1111101 ———— 这个符号位（ 即：? ）怎么填？

		011111111 11111111 11111111 11111111 ———— 无符号右移嘛，直接不保留符号位了 ，然后用0填充
```

**还是补充一下吧**：

- 计算机中不管是正数还是负数，存储形式都是以补码的形式存储的
	- 一个正数是以它的绝对值大小转化为二进制数，这就是它的原码，最高位就是符号位，正数的符号位为0，负数的符号位为1

举个栗子：

```json
	5的原码————是正数，所以它在底层中的存储形式就是下面这个
		00000000 00000000 00000000 00000101

	-5的原码————负数的补码是：反码+1，而反码就是除了符号位不动，其他位置取反
		10000000 00000000 00000000 00000101 ———— 这是-5的原码
		11111111 11111111 11111111 11111010 ———— 这就是 -5 的反码

			补码是什么？——反码+1
			11111111 11111111 11111111 11111011 ———— 这就是 -5 的补码，也就是计算机真正存的值
```





## 三目运算符 ？ ：

这个就和`if.......else.......` 一样（ 流程控制马上说明 ）

举个栗子：

```java
	return a > 0 ？"结果1，还可以支持已经定义好的变量，甚至其他数据类型" : " 结果2，但是这个运算符的最终结果返回值类型 却取决于这两个值的最大数据类型 "
```

 

可以参照if else语句

```java
public class Test{

	public static void main( String[] args ){

		int a = 2;

		if(a > 0){

		  System.out.println( "这是其中的一种结果" );
		}else{

		  System.out.println( "这又是另一种结果" );
		}
	}
}
```







# 流程控制

有三种类型：顺序、选择、循环

 

## 顺序结构

指的就是按照写的代码顺序从上到下执行


```java
public class Test{

	public static void main( String[] args ){

		System.out.println( "你来打我啊" ）;

		System.out.println( "诶，就是来气你" ）;

		System.out.println( "嘿~就是玩儿，你能咋滴" ）;
	}
}
```



 

## 选择结构

### 01 单分支选择结构if（）{}

```java
public class Test{

	public static void main（ String[] args ）{

		int score = 91；

		if（ score > 90）{

		  System.out.println（ "你厉害了" ）;
		}
	}
}
```



 



## 多分支选择结构

### 01 if  else语句 和 if语句嵌套

```java
if（条件）{
	// do something
} else{
	// do something
}


以及

if（条件1）{
	// do something
} else if（条件2）{
	// do something
} ........
else {
	// 上面的if条件都不满足时执行的逻辑
}
```

 

举个栗子：

```java
public class Test{
　　public static void main（ String[] args ）{

　　　　// 单分支选泽结构
　　　　int score = 91；

　　　　if（ score > 60 && score < 70 ）{
　　　　　　System.out.println（ "你还需要努力" ）;
　　　　}else{
　　　　　　System.out.println（ "你还需要努力" ）;
　　　　}


　　　　// 多分支选择结构
　　　　int sumScore = 100；

　　　　if（ sumScore < 60 ){
　　　　　　System.out.println( "你去出家吧" );
　　　　}else if( sumScore > 60 && sumScore < 70 ）{
　　　　　　System.out.println( "你还需要继续努力" );
　　　　}else if( sumScore > 70 && sumScore < 80 ) {
　　　　　　System.out.println( "你很牛" ）；
　　　　}else if（ sumScore > 90 && sumScore < 100 ){
　　　　　　System.out.println( "你牛逼了" );
　　　　}else if( sumScore == 100 ){
　　　　　　System.out.println( "你牛逼大了" );
　　　　}
　　}
}
```



### 02 Switch语句

```java
Switch（条件 这里面结果可以为byte、short、int及这三个的对应包装类，包装类在工具类中会说明、char、String 这个string类型是JDK1.7的时候有的 ）{

	case 1:
		代码1；
		break/continue； // break 是指：终止循环（只要是循环体中都可以用这两个关键字）

	case 2:
		代码2;
		break / continue;　　// continue 是指：终止本次循环，从下一次循环开始执行

	.........
}
```



 


举个栗子：

```java
public class Test{

	public static void main( String[] args ){

		int sumScore = 99;

		Switch（ sumScore > 60 ）{

		  case 70：
				System.out.println（ "哟西，还可以" );
				break；

		  case 80；
				System.out.println( "nice，你要不得了" ）；
				break；

		  case 90：
				............... // 不写了，手软得很！
		}
	}
}
```



 

## 循环结构

### 01 while循环

和if语句差不多

 


举个栗子：有这么一个需求：需要对输入的数 的 各个位的数求和(不确定这个数有多少位），如：输入1234 ——> `sum = 1+2+3+4 = 10`

```java
	import java.util.Scanner;
	/*
		这个import就是导包的意思，Java其实就是别人写好的工具罢了，所以这里就是引入别人(JDK)写的java.util包下的Scanner类
		这个就是为了可以键盘输入内容而已
			（ 输入的内容类型取决于int number = input.nextInt（）中的nextInt（），这个里面是int则结果就为int类型，是string则结果就是string类型
	*/


public class Test{

	public static void main( String[] args ){

		System.out.println( "请输入一个数" );
		Scanner input = new Scanner( System.in );
		int number = input.nextInt();　　// 以上对于用键盘输入内容的代码都是固定的，只是input和number的名字可以改而已

		int sum = 0；
		while（ number ！= 0 ）{

		int remainder = number % 10; // 输入的数 % 10 得到输入数的最后一个数字 如：1234 % 10————>得到4
		number /= 10;　　// 对输入的数 进行截取————如：1234 / 10————得到123，这样就可以下一次得到3这个数字了
		sum += remainder;　　// 这样就可以实现每个数相加的和 如：4+3+2+1 = 10

		}
	}
}
```



 

### 02 do while语句

这个语句会至少执行一次，也就是说：程序是先执行再判断满不满足条件

 

```java
public class Test{

	public static void main（ String[] args ){

		int score = 80;
		do{

			System.out.println（ “恭喜你成为最后一名了!" );
		}while(score > 90)
	}
}
```





### 03 for循环

```java
	for（初始值（计数器） ; 结束条件 ; 迭代表达式）{

		好多代码......

	}
```

 

举个栗子：

```java
public class Test{

	public static void main（ String[] args ）{

		for（ int i = 0 ； i < 10 ; i ++ ){

		  System.out.print(" " + i);
		  /*
			程序解读：
				i = 0，i < 10 ？ 满足则继续执行后续的程序，不满足则直接不用进入本循环了（这里是满足的）
				满足条件，执行打印输出语句
				然后：i = i + 1 = 1
				后面依次类推

				最终结果为：
					 0 1 2 3 4 5 6 7 8 9
		  */
		}
	}
}
```



### 04 while(true)和for(;;)两种写法有区别吗？

> 这是知乎上刷到的一篇R大的回答
> 作者：RednaxelaFX
> 整理者：紫邪情
> 参考链接：https://www.zhihu.com/question/52311366/answer/130090347


首先是先问是不是再问为什么系列

在JDK8u的jdk项目下做个很粗略的搜索


```txt
mymbp:/Users/me/workspace/jdk8u/jdk/src
$ egrep -nr "for \\(\\s?;\\s?;" . | wc -l
     369
mymbp:/Users/me/workspace/jdk8u/jdk/src
$ egrep -nr "while \\(true" . | wc -l
     323
```

并没有差多少。

其次，`for (;;)` 在Java中的来源。个人看法是喜欢用这种写法的人，追根溯源是受到C语言里的写法的影响。这些人不一定是自己以前写C习惯了这样写，而可能是间接受以前写C的老师、前辈的影响而习惯这样写的。

在C语言里，如果不include某些头文件或者自己声明的话，是没有内建的_Bool / bool类型，也没有TRUE / FALSE / true / false这些_Bool / bool类型值的字面量的。

所以，假定没有include那些头文件或者自己define出上述字面量，一个不把循环条件写在 while (...) 括号里的while语句，最常见的是这样



```java
while (1) {
    /* ... */
  }
```


.......但不是所有人都喜欢看到那个魔数“1”的。

而用 `for (;;)` 来表达不写循环条件（也就是循环体内不用break或goto就会是无限循环）则非常直观——这就是for语句本身的功能，而且不需要写任何魔数。所以这个写法就流传下来了。

顺带一提，在Java里我是倾向于写 `while (true)` 的，不过我也不介意别人在他们自己的项目里写 `for (;;)`



==========================================================================================



至于Java里 `while (true)` 与 `for (;;)` 哪个“效率更高”。这种规范没有规定的问题，答案都是“看实现”，毕竟实现只要保证语义符合规范就行了，而效率并不在规范管得着的范畴内。

以Oracle/Sun JDK8u / OpenJDK8u的实现来看，首先看javac对下面俩语句的编译结果：


```java
public void foo() {
    int i = 0;
    while (true) { i++; }
  }

/*
  public void foo();
    Code:
      stack=1, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iinc          1, 1
         5: goto          2
*/
```

与

```java
public void bar() {
    int i = 0;
    for (;;) { i++; }
}

/*
  public void bar();
    Code:
      stack=1, locals=2, args_size=1
         0: iconst_0
         1: istore_1
         2: iinc          1, 1
         5: goto          2
*/
```

连javac这种几乎什么优化都不做（只做了Java语言规范规定一定要做的常量折叠，和非常少量别的优化）的编译器，对上面俩版本的代码都生成了一样的字节码。后面到解释执行、JIT编译之类的就不用说了，输入都一样，输出也不会不同


当然，这种问题在我们普通人看来没什么较真的必要，但 R大认真去研究了，并且得出了非常令人信服的答案，牛掰的人自有牛掰之处

起码我们的收获就是以后可以放心大胆在代码里写 `for(;;)`、`while(true)` 这样的死循环了





# 数组

数组：是**存储一组相同数据类型**的数据



## 为什么要有数组？

要存储数据，变量就够了，干嘛还要有数组？

- 因为变量存储的是一个指定类型的数据，所以当数据很多的时候（如：50个、100个、1000个），这就要定义很多变量，既麻烦，又浪费存储空间，所以有了数组




数组的三个特性

- 一致性：数组只能保存相同数据类型元素，元素的数据类型可以是任何相同的数据类型
- 有序性：数组中的元素是有序的，通过下标访问
- 不可变性：数组一旦初始化，则长度（数组中元素的个数）不可变





## 一维数组

指的是：数组中每个元素只带有一个下标



### 01 静态创建

指的是：创建这个数组的时候就赋值，格式如下：

```txt
数据类型[] 数组名字 = new 数据类型[]{ 值1，值2，值3.... }；
```

这种方式的缺点：数组的大小已经固定了，如果后续想要改变数组的大小就无法改变了（目前是不可以，后续的技术是可以的）

 

举个栗子：

```java
public class Array{

	public static void main( String[] args ){

		// 第一种：
		int[] array1 = new int[]{ 1,2,3,4 };

		// 第二种
		int array2 = new int[3];
		array2 = 1; //数组默认是从下标0开始存值/取值
		array2 = 3;
		array2 = 4;
	  }
}
```



 

### 02 动态创建

指的是：只定义数组的数据类型和名字，但是不给赋值，后续用的时候再赋值，这种创建方式稍微比较灵活一点

 

举个栗子：

```java
public class Array{

	public static void main（ String[] args ){

		String[] array; // 这里不赋值，后续再赋值，所以这里也就不会确定数组的大小了

		// 后续用了再赋值：加入日本著名程序员
		array = {“苍井空”，“波多野结衣”}； // 但是这里一旦赋了值，则数组的空间大小就固定了，后续再想改就不得吃了，所以也是有弊端的
	}
}
```





### 03 数组怎么取值？

**通过` 数组名[下标值]`**

```java
public class Array{

	public static void main( String[] args ){

		int[] arrray = new int{}{ 1,2,3,4,5,6 };

		System.out.println( array[0] );
		System.out.println( array[1] );
		System.out.println( array[2] );
	}
}
```





### 04 数组的遍历

**注**：遍历 不等于 输出（遍历是指去访问一遍，输出是把它访问了打印出来，所以这两个是两码事儿）

 

#### （一）利用for循环

```java
public class ArrayEach{

	public static void main( String[] args ){

		int[] array = new int[]{ 5,2,0,1,3 };

		for( int i = 0 ; i < array.length ; i ++ ){

		  System.out.println( array[i] );
		}
	}
}
```



 



#### （二）利用增强for

##### 什么是增强for？

```java
	for( 数据类型 接收者名字 ： 被接收者名字 ）{

	}
```

 

举个栗子：

```java
public class ArrayTest{

	public static void main( String[] args ){

		String[] array = new String[]{ "张学友“，”张家辉“，”张学良“ };

		for( String result : array ){　　/* 注意：这个接受者的数据类型 要和 被接受者的数据类型一样 
							或者 是比 被接受者的数据类型小 */

		  System.out.println( result );
		}
	}
}
```







### 05 数组的扩容

本质就是利用数据的拷贝



#### （一）自己写扩容流程

```java
public class ExpansionArray1{

	// 这是定义了一个方法，后续面向对象思想编程会做详细说明————扩容的思想是下面的
	public static void expansion{

		 // 1、数组的初始长度
		 int[] array = {1,2,3,4};

		int[] newArray = new newArray[ array.length * 2 ];

		// 2、实现数据拷贝
		for（ int i = 0 ; i < array.length ; i ++ ){

		  newArray[i] = array[i]
		}

		// 3、实现地址交换————即：重命名，这样别人看起来就是扩容了
		array = newArray;
	  }
}
```

 



#### （二）调用System.arraycopy()方法

```java
public class ExpansionArray2{

	public static void main( String[] args ){

	// 旧数组的开始长度
	int array = { 1,2,3,4 }；

	/*
		参数说明：
			void arraycopy(Object src, int srcPos,Object dest, int destPos, int length );
				src     旧数组名字
				srcPos  从旧数组的哪个位置开始复制
				dest    新数组名字
				destPos 从新数组哪个位置开始
				length  复制旧数组的多少个元素
			注意：这个方法没有返回值，是void修饰的，目前这么记，到了面向对象层面就可以懂方法、返回值、参数这些了
        */

	// 实现数组扩容并实现数组地址交换
	int[] newArray;
	array = System.arraycopy( array , 0 , newArray , 0 , array.length );
	}
}
```





#### （三）调用Arrays.copyOf()

**注：** 这个方法有重载的，看需要来决定用哪一个。重载就是这个`copyOf()`名字都是一样的，在面向对象中会做详解

 

```java
public class Expansion3{

	public static void main( String[] args ){

	// 旧数组初始长度
	int[] array = {1,2,3,4};

	// 实现数组扩容并实现地址交换
	array = Arrays.copyOf( array , 5 );
	// 参数说明：第一个参数为——旧数组的名字　　第二个参数为——想要创建的新数组大小
						   // 这个方法底层采用的原理就是第二个方法的设计原理
	}
}
```



 

## 二维数组

二维数组的本质其实就是一维数组，是一维数组的特殊数组，可以看成是一维数组中套了一层一维数组,**实际开发中很少用（ 了解即可 ）**



 



### 01 二维数组的 创建与赋值

和一维数组一样，有着静态创建和动态创建




> 静态创建：一样的在创建的时候就把数组的大小确定了

```java
public class Array{

	public static void main（ String[] args ){

		// 第一种静态创建
		int[][] array = new int[][]{ {1,2},{3,4} }; 　　// [][]可以理解为是几行几列（虽然严格来说这么想不对，但是可以这么想，等到new 对象的原理分析时就懂了，二者一样）

		// 第二种静态创建
		short[][] array2 = new short[3][4];

		// 第三种静态创建————只给一部分的数组大小
		byte[][] array3 = new byte[3][];
		byte[][] array4 = new byte[][4]; // 注：这种是不得吃的
	}
}
```

 


> 动态创建：一样的才创建的时候不给数组的具体大小，后续需要用的时候再赋值

```java
public class Array{

	public static void main( String[] args ){

		int[][] array;

		// 后续用到了，然后就可以进行赋值————缺点：也是一旦赋值了后续再想改数组大小就不得吃了
		array = { {2,3},{5,5} };
	}
}
```



后续的和一维数组是一样的东西



 

# 锻炼脑袋：排序算法

## 冒泡排序

**思路：**

- 以数组索引的顺序 依次和后一个元素进行两两比较

- 前一个数比后一个数大就彼此换位，这样一个轮回下来 就可以让最大的那个数成为数组的最后一个了（ 一个轮回做的事情 ）

 


**实现：**

- 首先按上面一个轮回的思想来做：这样一共要进行多少轮？
	- 数组的长度是多少就是多少轮

- 然后就是：怎么知道已经进行了多少个轮回？
	- `Array.length() - 1(去除第一个轮回排出来的这一轮）- i`。i的大小就是已经进行了多少个轮回，即：已经排好了几个数



```java
public class BubbleSort{

	public static void main(String[] args) {

	 // 要求对这个数组进行升序排列
	int[] array = {3, 5, 6, 2, 7, 1, 9, 0};

	// 实现方式：

	// 控制总的趟数（轮回数）
	for (int i = 0 ; i < array.length ; i++) {

		// 控制每一趟（每一个轮回）
		for (int j = 0; j < array.length - 1 - i; j++) {

			// 每一个轮回需要做的事情————打擂台————两两比较
			if (array[j] > array[j + 1]) {　　// 要进行降序排列的话　　把这里的 > 改成 < 即可

				int temp = array[j];
				array[j] = array[j + 1];
				array[j+1] = temp;
			}
		}
	}

		System.out.println(Arrays.toString(array));
		// Array.toString（）是一个工具类 用来对数组进行转换的————转换成我们人可以看得懂的，不然直接进行输出array。那得到的就是一个地址码
	}
}
```

 


插入一个编程思想在现实中发生的，就当愉悦心情

在上面的冒泡排序中有一个变量换位的思想，源码如下：

```java
	int temp = array[j];
	array[j] = array[j + 1];
	array[j+1] = temp;
```

这是一种换位的思想嘛，所以来个面试题。假如去华为面试，有这么一个问题：

- 一头牛重15kg，这头牛准备过一座桥，而这座桥只能承重10kg，问：利用编程的思想，怎么把这头牛弄过桥？

**答案：**

- 把这头牛卖了
- 买一部等价钱的华为手机，然后把这部手机带过桥
- 因为：华为手机就是牛^ _ ^







## 选择排序

**思路：**

- 假设数组中的第一个数为最小的数（把这个数当做是已经排好了）

- 然后从后面没有排好的数之中，依数组顺序去找另外的数，和假设的这个最小数进行比较
	- 如果有比假设的这个数更小的，则标记那个更小的数 的 索引值（即：下标）
		- 另外：在找寻更小的数时，只是临时更新那个更小数的索引值，要继续找寻下去，若：有比更新的那个数 更要小的，则：记录的索引值又更新为这个更加小的索引值

- 到一轮结束之后：记录的索引值 对应的元素就是这个数组中 余下那些还未排序的数 的最小数

- 最后把找到的这个最小数 和 第一步自己假设的最小数 进行换位————这样就得到前面排好的数就是升序排列了



```java
public class SelectSort{

	public static void main( String[] args ){

		// 对这个数组进行升序排列
		int[] arr = { 3,2,7,5,4,1,9,8,0 };

		// 代码实现

		// 1、把每个数都假设一遍：让它成为最小的数
		for ( int i = 0; i < arr.length; i++ ) {

			// 先假设第i个数为最小的数(刚进入循环时i = 0 就是指数组的第一个数），记录它的索引
			int min = i;

			// 2、控制每一趟做的事情
			for ( int j = i + 1; j < arr.length; j++ ) {　　// i + 1 就是假设第一个数为已经排好的数

				// 若里面有其他数比假设的最小数还小
				if ( arr[min] > arr[j] ) {　　// 要想实现降序排列 修改这里

					// 更新最小值的索引值
					min = j;
				}
			}

			// 3、验证一下：如果有比假设的数更小的————看看假设的数 的索引值有没有产生变化————有的话两个数就进行换位
			if ( min != i ) {

				int temp = arr[min];
				arr[min] = arr[i]; // 有比假设的最小数更小的话，则：这里的min在前面的if语句中（即：min = j语句）更新了
				arr[i] = temp;
			}
		}

		// 打印输出
		System.out.println( Arrays.toString( arr ) );
	}
}
```



 

## 插入排序

**思路：**

- 假设数组中的第一个数排好的（已排序区）

- 依次用后面未排序区的数值 和 已排序区的数值进行比较

- 如果未排序区中的数值 比 已排序区中 的数值小，则：二者进行换位。一个轮回完毕

- 依此思想进行循环，直到完全排好序



```java
import java.util.Arrays;

/**
 *  插入排序学习
 */
public class InsertSortStudy {

	public static void main(String[] args){

		// 对这个数组进行升序排列
		int[] array = { 3,5,2,7,4,8,1 };

		// 实现方法————优化版的
		for( int i=1; i < array.length; i++ ){

			//保存每次需要插入的那个数
			int temp = array[i];
			int j;

			//把大于需要插入的数往后移动。最后不大于temp的数就空出来j
			for( j=i; j>0 && array[j-1]>temp; j-- ){

				array[j] = array[j-1];
			}

			array[j] = temp;//将需要插入的数放入这个位置
		}

		System.out.println( Arrays.toString(array) );
	}
}
```