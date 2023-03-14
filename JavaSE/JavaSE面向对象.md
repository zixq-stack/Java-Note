

# 理解面向对象编程思想

拿大象装冰箱来看，假设装得下

- **开门**：我觉得冻到我手了，不想去做，所以我想**有个人**(`此人就是一个对象`)来帮我做开门这件事

- **把大象装进去**：我感觉我身体太弱了，弱不禁风的，大象太重我装不进去，我又想**有个人**(`此人就是一个对象`)来帮我装进去

- **关门**：这个可以，我一脚就可以把门关上了，所以我可以做




像上述这种：找人(`对象`)来帮我做某件事情的思想就是面向对象编程思想，这样就不用太注重这件事应该怎么做，而是注重我应该找谁来帮我做。核心就是“对象”，这个对象就相当于是一个小秘书，这个小秘书对我言听计从的，我叫她干啥就干啥。当然这是表面而已，还是要自己领会







## Java中的面向对象有什么？

### 01 类

> **定义：**是一个抽象的东西，这个东西**具有相同特征和行为**

举个栗子：

人类。那什么叫人类？直立行走的两栖动物？但是好像不止人才可以直立行走并为两栖动物

可是：人类都有名字、有性别、都可以吃饭、可以睡觉，这个类都有这些相同的特征和行为





#### （一）在java中怎么创建一个类?

**使用class关键字**

照样用人类来看

```java
public class People{ 
/* 
  class关键字 类名{
    	类体
  }
*/
}

```



类定义中 说的**特征和行为** 是什么？
**属性**：静态描述的特征。如：人类都有名字、有年龄，这些些都是静态的，没有什么动作之类的 
**方法/函数**：动态的动作行为。如：人类可以吃饭、可以打游戏、可以说话、可以学习



#### （二）对象

> **定义：**类中的一个具体执行者（类的实例）。通过这个对象可以去操作这个类中的东西了，如：人类中有一个叫“苍井空”的

 



#### （三）创建一个类并使用

```txt
步骤：
  1、声明一个类，即：创建一个类
  2、用属性 / 方法描述这个类（ ps：方法还没学，后续会说，有点基础就可以理解，因为：就是所谓的函数 ）
  3、调用这个类的属性或方法
```




还是用人类举例

```java
public class People{

  String name；
  char sex；
  int age；

	public static void main（String[] args）{

    // 创建一个People类的具体执行者（对象）
    People person = new People()；
    // 通过 对象.属性名 调用类中的属性
    person.name = "苍井空";
    // 当然对于类中的方法的调用也是这么回事儿，即：对象.方法名（）
    person.sex = '女';
    person.age = 18;
    // + 就是拼接的意思
    System.out.println( person.name + "性别:" + person.sex + ",今年" + person.age +"岁" );
	}
}

```

 

**小结：创建类的实例/对象 及其 属性的调用**

```json
创建对象：实质就是创建某类型的空间
	类名  对象名 = new 类名();　// 注：真正的对象实质上是new 类名()，而我们起的对象名只是一个对象的引用地址

// 属性调用：用 对象. 来调用
对象名.属性名;
```



附：对象在JVM虚拟机中的原理分析图，这个图是为了好理解，在JVM中实质不是这个样子的

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213069-779198749.png)

 





#### （四） 类中成员有那些？

换句话来说：就是一个类是由哪些东西来描述它的，有如下这些：

1. 属性
2. 一般方法
3. 构造方法
4. 块 / 程序块 / 代码块





##### 类中的属性

> 其实就是前面基础篇中学的变量
> **定义：类所具有的特征**



举个例子：一样用人类来看，人类中有个苍井空（她有名字、有性别、有年龄）

```java
public class Person{

	String name;
	char sex；
	int sge；

	/*
		上面三个属性在这里不严谨，但是意思没变，只是严格来讲需要做一点调整（ 等到修饰符、还有封装思想会了就知道了 ）
		主要是为了理解这个意思
	*/
}

```



**小结：属性的定义**


```json
[修饰符] 数据类型  属性名 / 变量名 = [值]			目前不用修饰符也可以（ 后续懂了再加 ）
```

另外：属性具体写多少个？根据你需求来编写，需要哪些就编写哪些



##### 补充：类与对象的关系

**类是对象的模板，对象是类的实例**，一个类可以实例化出N个对象，来个图：

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213067-514864710.png)



 

##### 类中的一般方法

> **定义**： 做一件事 / 描述一件事

 

**方法的完整结构**：

```java
权限修饰符 [特征修饰符] 返回值类型 方法名（参数列表）[抛出异常]{

	方法执行体 / 逻辑处理
}

```

其中`[]`里面的内容不是必有，看情况来决定要不要有





##### 面向对象之方法设计

###### 无参数无返回值类型

```java
public class People{

	String name = "邪公子";
	String address = "在地球上"；

	/**
	* 这就是一个无参无返回值方法 public是一个权限修饰符 暂时死记这么写就行
	* void 是返回值类型，这是无返回值，eat（）的（）里面就是参数列表........
	*/
	public void eat（）{
		System.out.println( "吃了一碗大米饭" );
	}


	/*
	 * 测试一下
	 */
	public static void main(String[] args){

		People person = new People();
		System.out.println( person.eat() );
	}
}

```



###### 无参数有返回值类型

有这么一个需求——用现金去买东西，柜台找钱，问：老板需要找我多少money？


```java
public class Refund{

	public double money(){

		// 假如买的这个东西价格为11.3元
		double price = 11.3;
		// 我支付给店家20.5元
		double payMoney = 20.5;

		// 则老板需要给我的钱为refund
		double refund = payMoney - price;

		// 这就是老板需要找回我的钱
		return refund;
	}

	/*
	 * 测试一下
	 */
	public static void main(String[] args){

		System.out.println("老板需要找回我多少钱：");
		Refund myMoney = new Refund();
		System.out.println (myMoney.money());
	}
}

```

 

###### 有参数无返回值类型

假如有这么一个情况： 小猫偷吃鱼肉

```java
public class  Cat{

	public void eatFish（String name）{
		System.out.println(name + "猫咪偷吃了鱼"）;
	}


	/*
	 * 测试一下
	 */
	public static void main(String[] args){

		Cat smallCat = new Cat();
		System.out.println( mallCat.eatFish("二郎神"));
	}
}

```



###### 有参数有返回值类型

假如有这么一个情况：我口渴了，想让人帮忙买瓶饮料

```java
public class BuyDrink{

	public String drink（int money）{

		if(money > 5）{
			return "红牛"；
		}else{
		  return "矿泉水";
		}
	}


	/*
	 * 测试一下
	 */
	public static void main(String[] args){
		BuyDrink buyDrink = new BuyDrink();
		System.out.println(buyDrink.drink(10));
	}
}

```

 

怎么设计方法？

1. 先分析需求：需要什么数据类型的返回值（返回值类型）
2. 再看需求：需要传什么数据类型的条件进去（参数，可以有多个）

 



###### 锻炼脑袋

假如有这么一个需求：我想要一种容器

变量是一种容器，但是只能存储一个给定数据类型的数据；

数组也是一种容器，但是只能存储一组给定数据类型的数据，而且不可以自动扩容、也没有增删查改功能，虽然利用前面已经学的知识可以做到这些，可是：我想要的不是这些，我想要的是方法

**所以我需要的这种容器能够帮忙实现：1、自动扩容；2、能够提供增删查改功能**

```java
import java.util.Arrays;

/**
 *  自己封装一种容器————这个容器拥有：1、自动扩容；2、这个容器提供对存储的元素进行增删查改功能
 *  本例子中用这个容器存储的是int类型举例
 */
public class ArrayBox {

	/*
		给一个数组的初始大小
		private是权限修饰符，static是特征修饰符，后续马上就可以接触到了，不懂也没事，甚至不加这两个也可以
	*/
	private static int[] array = new int[10];

	// 保证这个计数器全局唯一，不加static也行
	static int index = 0;

	/** 功能一：自动扩容功能：就是一个方法(在基础篇中已经弄过了，只是到了这里设计成一个方法了而已
	 * 1、考虑传什么条件进去（参数）：得把你原来的数组给我呀，不然我帮谁扩容？
	 * 2、考虑返回值是什么：得把扩容之后的数组返回去啊
	 */
	public int[] grow(int[] array) {

		/*
			扩容二绝技
			1、数据拷贝：需要一个新数组（当然直接通过传进来的这个数组也可以操作，只是为了好理解，熟练了就可以直接用当前传进来的数组
		*/
		int[] newArray;
		if (index == array.length) {
			newArray = new int[array.length * 2];

			for (int i = 0; i < array.length; i++) {
				newArray[i] = array[i];
			}

			// 2、把数据拷贝并扩容的数组的地址 指回 原数组
			array = newArray;
		}

		// 返回扩容之后的数组
		return array;
	}


	/**
	 * 功能二：增加元素功能
	 * 1、考虑要传什么条件进来：老衲需要知道你要添加的是爪子东西呀
	 * 2、考虑返回值是什么：就添加元素，不需要返回值嘛
	 */
		public void add(int element){

			// 扩容，如果没满，在扩容中自然会进行判断
			grow();

			// 把元素添加进去
			array[index] = element;

			// 计数器指向下一个准备添加元素的位置
			index ++;
		}

		/**
		 * 功能三：删除元素功能
		 * 1、考虑传什么条件进去：得告诉老夫要删除哪个位置的元素呀
		 * 2、需要返回什么东西：只是删除指定的元素，也不需要返回什么
		 */
		public void remove(int removeIndex){

			// 可以利用删除元素的位置 = 删除元素的后一个位置
			for (int i = removeIndex; i < index - 1; i++) {
				// 利用值覆盖
				array[removeIndex] = array[removeIndex + 1];
			}

			// 把数组的最后一个元素置0，这只是小测试，实际看到的效果不是这样的，可以做优化
			array[index] = 0;
			// 同时让计数器的值 -1，即：让其指向原来位置的上一个位置
			index -- ;
		}

		/**
		 * 功能四：查询数组元素功能
		 */
		public void quary() {
			for (int i = 0; i < array.length; i++) {
				System.out.print( array[i] + " ");
			}
		}

		/**
		 * 功能五：修改元素
		 */
		public void set(int setIndex,int newElement) {
			if ( setIndex < array.length ){
				array[setIndex] = newElement;
			}
		}


		/**
		 * 测试
		 */
		public static void main(String[] args) {

			ArrayBox arrayBox = new ArrayBox();

			// 扩容数组
			int[] newArray = arrayBox.grow();
			System.out.println(newArray.length);

			// 换行
			System.out.println();

			// 添加元素
			arrayBox.add(9);
			arrayBox.add(5);
			arrayBox.add(2);
			arrayBox.add(3);
			arrayBox.add(8);

			// 打印出来看一下
			System.out.println(Arrays.toString(newArray));
			/* Arrays是一个工具类，在工具类篇中会做解读，总之：Arrays.toString()方法的作用就是：
			把newArray转成人能够看懂的字符串，不然直接输出newArray是一个地址码
			*/
			System.out.println();

			// 删除元素
			arrayBox.remove(2);
			System.out.println(Arrays.toString(newArray));
			System.out.println();

			// 查询元素
			arrayBox.quary();
			System.out.println();

			// 修改元素
			arrayBox.set(1,7);
			System.out.println(Arrays.toString(newArray));
		}
}

```

 



##### 方法重载

> **定义**：类中的一组方法，它们具有相同的名字，不同的参数列表
>
> 参数列表的不同体现在那里？
>
> 1. 参数的顺序
> 2. 参数的个数
> 3. 参数的类型
>
> **注意**：与返回值类型无关
>
> **作用**：便于操作者记忆与使用（只需要记得一个方法的名字，就可以实现不同的操作）




用人类来举例

```java
public class OverLoadMethod{

	public void sleep（）{
		System.out.println( "张三是可以睡觉的" );
	}

	/**
	* 形参name
	* 形参：就是没有值的参数，这个方法执行完之后，这个形参就没了（不是本身没了，是调用时产生的空间没了）
	*/
	public void sleep(String name){
		System.out.println( name + "也可以睡觉" );
	}

	public void sleep(String name,double time){
		System.out.println( name + "睡了" + time + "小时");
	}

	public void sleep(double time）{
		System.out.println("李白是不是睡过头了？");

		if(time > 8){
			return "true" ;
		}else{
			return "false" ;
		}
	}


	/**
	 * 测试
	 */
	public static void main(String[] args){

		OverLoadMethod invokeSleep = new OverLoadMethod();

		invokeSleep.sleep();

		// “邪公子” 就是实参，调用这个方法的时候，把"邪公子"这个值传给了形参
		invokeSleep.sleep("邪公子");

		invokeSleep.sleep("山口香子",7.5);

		// ...........后续的不写了
	}
}

```



##### 方法的执行原理

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213132-939095033.png)



调用方法的原理：实质就是调用方法里面的执行语句，并把实参赋值给形参的过程

 



##### 方法之动态类型参数

> 形参在JDK1.5之后有一个新玩法：**动态类型参数**，即：`数据类型...x`
> 这种玩法：数据类型固定了，但是参数的个数却没有固定，传0到N个都行

 

```java
public class NewPlayMethod{

	public void test(int...x){
		System.out.println("这是一个动态参数列表测试");
	}

	public static void main(String[] args){

		NewPlayMethod p = new NewPlayMethod();
		p.test();　　　　　　/*
		p.test(1);　　　　　　这三种传参都可以
		p.test( 1,2,3,4 );　　*/
	}
}

```

**注意**：

- 这个动态参数列表的使用，如果有自己定义的另外参数，则这个动态参数列表只能放在参数列表的最后
- 这个参数列表的本质是数组，所以有length，有下标索引
- 它不能和具有相同意义的数组类型方法构成重载，因为本质都是数组
- 它在方法的“参数列表中只能存在一份，且必须放在参数列表的最后一个”



 



##### 构造方法 + 块 + this关键字

###### 构造方法

**定义方式**：就是只有如下的结构

```json
权限修饰符 和类名一样的方法名（参数列表）{

	做一件事 / 创建一个对象;
}

```



**作用**：为了创建对象、为对象赋值。new对象的时候会触发构造(new的是无参就会触发无参构造、new的是有参就会触发有参构造
构造方法是类中定义的一种特殊方法，**通过构造方法来实例化该类对象**，也就是该类实例
**注**：如果自己不写无参构造，系统会有一个默认的无参构造；要是自己写的有无参构造，则会覆盖系统原有的无参构造，从而使用我们自己定义的



```java
public class Test{

	String thisName;

	/**
	* 无参构造
	*/
	public test(){}

	/**
	* 有参构造
	*/
	public test（String name）{
		thisName = name;
	}
}

```





##### 块( 程序块 / 代码块  /  普通快 )

**定义方式**：就只有一个大括号，其他的啥子结构都没有
**意义**：在每次调用和块同类的方法之前，都会默认调用一次“块”

```java
{
	执行体;
}

```

 

**还有一种特殊块，在企业级开发中会玩：静态块**
**意义**：在整个类加载的时候，就加载静态块的内容，但是只执行一次，这是在内存中开辟了一个“公共区域”，即：前面画的图中的那个静态元素区。再次强调：在JVM虚拟机中实质不是那样的啊，是为了好理解才那么画的，这个静态块的特征在后续的特征修饰符中会解释static的特性，那时候就懂这个静态块了

```java
static{
	执行体;
}

```

 

**"块"举个例子（ps：对java一点基础都没有的人，搞不懂可以跳过，因为：涉及到后面的知识了）**：

```java
public class Test{

	{
		System.out.println( "这就是一个普通快" );
	}

	static {
		System.out.println( "这是一个静态块" );
	}
}

/**
 * 可以有多个这种类定义，但：public修饰的类只能有一个
 */
class Demo{

	public static void main(String[] args){
		Test test = new Test();
		// 可以提前玩一下，然后琢磨琢磨，在这里直接运行程序，会输出什么结果？为什么结果是那样的？
	}
}

```





##### this关键字

> this是一个指代词，指的是“一个对象”，**就是指：调用当前类的属性或方法 的 那个对象**
> **注意**：只能在当前类的内部使用，要是在其他类A中用this调用当前类B的属性和方法是不得吃的
>
> this可以调用的那些？
>
> 1. 属性
> 2. 方法
> 3. 还可以在当前类的一个构造方法中 调用 当前类的另一个构造方法




```java
public class Test{

	String name;

	public void play(){
		System.out.println( "玩个毛线" );
	}

	// 玩一下 在当前类的一个构造中 调用 另一个构造
	public test(){
		System.out.println( "这是我自己定义的无参构造“ ）；
	}

	public test(String name){

		this();
		/*
			在这里调用本类中的无参构造
			也就是：后续要是创建本类的有参对象，进入到这里时会先去调用无参构造的执行语句
			这个式子等价于：this.test(); 简写就是this()
		*/
		System.out.println( name + "调用了有参构造" );
	}


	/**
	 * 测试
	 */
	public static void main(String[] args){

		System.out.println（this.name);
		System.out.println(this.play());

		Test demo = new Test("邪公子");
	}
}

```





**小结：this关键字的作用**

1. 作为一个引用：指向当前类的属性或方法( 上述的`this.name` )
2. 指代当前类中的其他构造方法( 上述的`this()` )



至此：类中的成员就完了 [ ps：属性、一般方法、构造方法、块 ]，接下来就要玩类与类之间的关系了

 

 



### 02 修饰符

#### （一）权限修饰符

权限修饰符有哪些、可以放在什么地方？

|             | 名字         | 适用的地方                                                   |
| ----------- | ------------ | ------------------------------------------------------------ |
| **public**  | **公有的**   | **类、同包 / 异包、子类（ 只要在一个项目中，有类的相应对象，就都可以访问 ）** |
| protected   | 受保护的     | 本类、同包、异包子类（ 在子类中，通过子类的对象，“在子类范围内”访问，即：超出了子类以外就不得吃了 ） |
| 默认不写    | 就是默认不写 | 本类、同包                                                   |
| **private** | **私有的**   | **本类**                                                     |

 

 

##### 权限修饰符可以使用的范围

> **可以使用在“类本身、类中的成员（ 注：块不可以 ）**
>
> **细节**：修饰类本身的时候，只能用`public`和`protected`，以及`默认不写`
>
> - **注意**：是默认不写，不是用什么`default`（ 这个是在接口 和 注解中用的 ）
>
> **权限修饰符使用建议**
>
> - 在属性没有明确要求的情况下，使用`private`
> - 方法在没有说明的情况下，使用`public`
> - 务必要让子类继承的方法和属性，使用`protected

 





#### （二）特别说明：default关键字

> **这是Java8中的东西**
>
> **可以修饰的东西**
>
> - **修饰属性：** 在注解中会用到，给注解参数设置默认值
> - **修饰方法：** 一般在接口中会见到，是为了解决接口的缺陷，一个接口有某个类实现时，这个类不需要接口中的某一个方法，或者说接口中某个方法不是一定要让子类实现的(此方法有一套默认逻辑)，那就用`default`修饰
> 	- 但是：如果有两个接口，这二者里面有一个同名并被`default`修饰的方法，那么子类同时实现这两个类时，那么这两个接口中的这个同名且被`default`的方法就必须实现，否则：报错





#### （三）理论：类与类的关系、Java面向对象的四个特征

> **类与类的关系：** 继承、包含（ 组合、聚合、关联 ）、依赖
>
> **java面向对象的四个特征：** 继承、多态、封装、抽象
>
> 1. 封装：这是一种思想，指的是把一些数据、执行过程进行包装起来。目的：为了保护数据 或者 执行过程的安全。如：一个方法也算一种封装：封装了设计思想；一个类也算一种封装 .......
> 2. 对于属性的封装:
> 	- 用 **private 修饰属性**，成为私有属性
> 	- **提供 public公有的 方法** 让人能够获取 / 修改`private`修饰的属性
> 	- 得出结论：声明属性的时候，最好私有化，只需要提供一个公有的方法即可。因为：不私有的话，别人是可以对属性进行操作的，不安全（ 即：可以通过new这个类的对象，然后通过这个对象对属性进行修改 ）

 

#### （四）特征修饰符

> **特征修饰符有哪些？**
>
> - `static`
> - `final`
> - `native`
> - `abstract`
> - `synchronized`
> - `transient`
> - `volatile`





`synchronized`、`transient`、`volatile` 在这里不做说明，原因如下：

- `synchronized` (同步锁) 是用来做线程安全的
- `volatile`（禁止指令重排序）是用来将数据更新到主存中去，以上两个都是多线程中的，所以到了多线程再说明
- `transient`（不被序列化) 这个在对象流中，序列化时可以用到，就是让用这个修饰符修饰的对象不要序列化到指定文件中去。因此这个特征修饰符会在流技术中的对象流时会做说明



##### final特征修饰符

> **意思：不可变的、不可更改的**

 

###### 修饰类本身

> 修饰类本身时表明这个类不可更改。什么意思？太监类咯，也就是不可以被子类继承嘛



```java
public final class Test{
	// do something
}

```

 

###### 修饰类中属性

> 修饰类中属性时表明这个属性不可更改
> **注1**：如果修饰的这种属性（变量）在定义时没有存值，则系统会给一次赋值的机会（因为变量在栈内存空间内，如果不给值，则没法使用）
> **注2**：如果`final`修饰的是基本数据类型，则就是相当于常量；如果修饰的是引用数据类型，则表明这个变量的地址不可以更改

 

```java
public class Test{

	public static void main(String[] args){

		// 当然在这里直接赋值更可以
		private final int age;

		age = 18;
	}
}

```

 



###### 修饰类的一般方法

> 修饰一般方法时表示这个方法不可以被更改：即：不可以被重写

 

```java
public class Test{

	public static void main(String[] args){

		public final void demo(){
			System.out.println("你打我啊");
		}
	}
}

```

 



##### static特征修饰符

> 指的就是静态的
>
> **可以修饰什么？**
>
> 1. 属性
> 2. 方法
> 3. 块
> 4. 内部类(即：成员内部类。在本篇内容的最后会说明成员内部类)
>
>
> **修饰之后具有什么特点？**
>
> 1. **修饰之后被所有的对象和类共享** ，在静态元素区中
>
> 2. **静态元素区中的内容只加载一次，有且只有一份**
>
> 3. **修饰之后，在类加载时，修饰的东西就已经被初始化了，但此时没有创建对象**
>
> 4. **由于在类加载的时候没有创建对象，所以可以通过 类名. 直接进行访问**
>
> 5. 每一个静态类都有自己的单独区域，不会和其他的类冲突
>
> 6. 静态元素区中的属性或方法GC回收机制管不了，所以可以认为静态元素区中的内容是常驻内存的
>
> 7. 非静态元素区( 堆内存中 ) 可以访问 静态元素区中的内容
>
> 8. 静态元素中的 可以访问静态元素区中的( 隔壁、及隔壁的隔壁可以串门儿嘛 ）
>
> 9. 静态元素区中的 想要访问 非静态元素区中的内容则不可以。原因如下：
>
> 	- 因为静态元素区中的内容是属于类的，而非静态元素区中的内容是属于对象的( new出来的 ）
>
> 	- 因为静态元素区中的内容只有一份，而非静态元素区中的内容有很多份，所以静态元素区想要访问非静态元素区不可以是因为：静态元素区中的 不确定到底是访问非静态元素区中的哪一份
>
>
> **原理分析：只是为了好理解，所以是这么画的图 （JVM中实质不是这样的）**
> ![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213328-1045763493.png)

 





```java
/**
 * 面试题： 对象创建 各初始化模块执行的先后顺序
 */
public class Demo {
	// 测试
	public static void main(String[] args) {
		Son son = new Son(); // 问：输出结果为多少？
	}
}



// 爷爷类
class YeYe{

	{
		System.out.println("爷爷动态代码块");
	}

	static {
		System.out.println("爷爷静态代码块");
	}

	public YeYe() {
		System.out.println("爷爷构造器");
	}
}




// 父类
class Father extends YeYe{

	{
		System.out.println("父类动态代码块");
	}

	static {
		System.out.println("父类静态代码块");
	}

	public Father() {
		System.out.println("父类构造器");
	}
}



// 子类
class Son extends Father{

	{
		System.out.println("Son动态代码块");
	}

	static {
		System.out.println("Son静态代码块");
	}

	public Son() {
		System.out.println("Son构造器");
	}
}

```



 

##### native特征修饰符

> 指的是本地的
> 一旦在java源码中看到native修饰了，则说明后续的源码看不到了
>
> native修饰的方法只有方法的结构，以分好 `;` 结尾，没有方法执行体，如：`public native int getInt( Object var1, long var2 );`
>
> - 没有方法执行体：不是说代码都没有了，而是后面的执行过程是用其他语言写的（ 如：C、C++.... ）





##### abstract特征修饰符

> 指的是抽象的。这玩意指的就是一个不具体的东西，就是一个概念，即：只是说应该有这么一个东西存在于里面，里面到底是什么，马上就进行说明

 

###### 修饰类的一般方法

> **用`abstract`修饰的方法叫抽象方法， 它只有方法的结构，然后就分号 `;` 结尾，没有方法的执行过程，大括号`{ } `都没得**
>
> 和`native`修饰的方法有区别，`native`是后续的执行过程用其他语言写了，而abstract是直接没有后续了，需要等着子类来继承之后重写它 / 实现出它应该有的逻辑代码



###### 修饰类本身

> 用`abstract`修饰的类就叫抽象类

 

利用  `abstract` 修饰 类本身 和 方法 举个例子


```java
/**
 * 修饰类本身，即：抽象类
 */
public abstract class Demo{

	/**
	 * 修饰方法，即：抽象方法
	 */
	public abstract void test();
	/*
		注意：抽象方法 和 用native修饰的方法不一样
				 二者在表面上看起来很像，但是执行过程那是两码事：就像看起来清纯的人私底下可能整得有点嗨
	*/

	// 举个例子：就像在基础篇中提到的数组扩容的第二种方式调arraycopy()方法，它在底层中是这样的
	public static native void arraycopy(Object src,  int  srcPos,Object dest, int destPos,int length);

}

```




**由上述的代码可以联想出两个问题**

1. **抽象类中必须含有抽象方法吗？**
	- 不是的，抽象类中可以没有抽象方法
2. **抽象方法必须放在抽象类中吗？**
	- 目前来看，抽象方法必须放在抽象类（ 或者接口 —— 接口后续会进行解释 ）中的，因为普通类不允许含有抽象方法





##### Synchronized、volatile、transient特征修饰符

在这里都不做说明，只是放在这里成为一个知识体系点而已

 





### 03 预知：抽象类

前面说`abstract`修饰类本身就变成抽象类，那也来了解一下它

 

> **抽象类中有什么？**
>
> - 普通类有的它都可以有(属性、一般方法、构造方法、块)，而且还允许有抽象的方法
>
> **抽象类如何使用？** 抽象类虽然拥有普通类的东西，那也有构造方法，但是我们却不可以利用构造方法来创建对象
>
> - 抽象类想要创建对象，则必须通过子类来继承，从而来做事情(子类继承之后，有抽象方法还必须把它具体化[就是重写这个抽象方法]，因为一般类不允许有抽象方法)



#### （一）抽象类、具体类的关系

**1、一个抽象类 ——> 可以直接单继承 ——> 另一个抽象类。这种需要知道**

```java
abstract class Test1{

	public abstract void sleep();

}


abstract class Test2 extends Test1{

	public abstract void study();

}

// 以上这样操作是可行的

```

 

**2、一个抽象类 ——> 直接单继承 ——> 一个具体类**

- 这种用法好使，但是通常不会出现

 

**3、一个具体类 ——> 直接单继承 ——> 一个抽象类。这种需要知道**

- 好使，但是如果抽象类中有抽象方法，则具体类必须把它具体化(即：重写抽象类(父类）中的抽象方法)


```java
abstract class Test3{

	/**
	 * 抽象方法
	 */
	public abstract void eat();

}


public class Test4 extends Test3{

	/**
	 * 具体类必须具体化
	 */
	public void eat（）{
		System.out.println( "天王盖地虎" );
	}
}

```

 


**由这些知识又得出两个结论**

1. 抽象类可不可以只有抽象方法，没有具体成员？
	- 可以的。因为抽象到极致就是另外一个名字了：接口
2. 抽象类中可不可以只有具体成员，没有抽象方法？
	- 可以的。因为抽象类可以有属性、一般方法、构造方法、块





### 04 类与类之间的关系

##### （一）继承关系 extends

> **这是 `A is-a B` 的关系，A是一个B。就相当于是 一个类（子类）找了另一个类（父类）当干爹，所以子类可以使用父类的很多东西**
> **学这个东西的目的**：
>
> 1. 写代码迫不得已需要用它
> 2. java的源码中用到了它
> 3. 和后面的技术又息息相关
>
>
> 虽然有这个东西，但是自己在“实际开发”的时候，最好能少用就少用（除非不得不去继承），因为“继承”增加了类与类之间的耦合度
>
> - 类与类之间的耦合度大小：继承 > 包含（ 组合 > 聚合 > 关联 ） > 依赖
> - 设计类需要做到的原则之一是：**高内聚低耦合**
> 	- **高内聚**：指的是类中的一个方法最好只做一件事
> 	- **低耦合**：就是类与类之间的耦合度尽量低



 



###### 子类继承一个父类，通过extends关键字

```java
public class Animal{

	String name;
	String age;

	public void eat(){
		System.out.println( “动物可以吃饭" );
	}
}

public class Person extend Animal{
	// 在这个类里面，就具有了Animal类的所有属性和方法，只需要new一个person类的对象，就可以进行调用
}

```

**注：子类只能访问由`protected`和`public`这两个权限修饰的父类的属性与方法，从而当做自己的属性和方法来用**

 

###### 子类也可有独有的属性和方法

> 子类继承了父类，但子类也可有自己独有的属性和方法

 

就像前面Person类继承了Animal类，但是Person也可以有自己的方法，如：


```java
public class Person extend Animal{

	// 在这个类里面，就具有了Animal类的所有属性和方法，只需要new一个person类的对象，就可以进行调用

	// 人类还可以学习、可以说话啊
	public void study(){
		System.out.println("人类还可以学习");
	}

	public void talk(){
		System.out.println("人类还可以说话"）；
	}


	/**
	 * 测试
	 */
	public static void main（String[] args）{

		Person p = new Person();

		p.name = "印度阿三";  　　// 以下这几种，子类Person都具有
		p.age = 100;
		p.eat();　　　　　　　　　/*  怎么确定通过对象 调用的是方法还是属性？
		p.study();　　　　　　　　　　　　　　　调用的是属性的话，只有属性名，没有（）这个括号
							 调用的是方法的话，有方法名和()这个括号
		p.talk();　　　　　　　　*/
	}
}

```

 



###### 子类重写父类方法

> 子类从父类中继承过来的方法不能满足子类的时候，子类可以通过"重写"父类的方法，从而达到子类自己的目的
> **什么叫方法重写？**
>
> 1. 方法重写更多指的是：**内容的重写**
> 2. 因此：**方法重写就是和父类中的方法结构一样**，即：权限修饰符 特征修饰符 返回值类型 方法名都和父类的一样，只是`{ }`这个大括号里的**方法执行体不一样而已（内容）**




```java
public class Animal{

	public void sleep(){
		System.out.println("这是动物的睡觉方法");
	}

	public void eat(String name){
		System.out.println(name + "吃了一堆翔”）；
	}
}


public class Person extends Animal{

	pubilc void sleep(){
		System.out.println("这是人类的睡觉方法"）;
	}


	public void eat（String name，double time){
		System.out.println( name + "吃了“ + time "小时的大米饭" ）；
	}
}

```

 



###### 方法重写与方法重载的区别

利用方法结构组成来分析即可

|                | **方法重写**                                                 | **方法重载**                                     |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| ***类***       | *两个继承关系的类（子类重写父类的方法）*                     | *一个类中的一组方法（只是类中的名字相同而已）*   |
| ***权限修饰*** | *子类可以大于等于父类*                                       | *没有要求*                                       |
| ***特征修饰*** | *有final、static、abstract*                                  | *没有要求*                                       |
|                | *父类方法为final时，子类不可以重写*                          |                                                  |
|                | *父类方法为static时，子类不存在*                             |                                                  |
|                | *父类方法为abstract是，子类必须重写*                         |                                                  |
| ***返回值***   | *子类可以小于等于父类*                                       | *没有要求*                                       |
| ***方法名***   | *子类与父类必须一致*                                         | *一个类中好多方法名一致*                         |
| ***参数***     | *子类与父类必须一致*                                         | *每一个方法的参数必须不一致（类型、顺序、个数）* |
| ***异常***     | *分运行时和编译时*                                           | *没有要求*                                       |
|                | *若父类方法抛出运行异常，则子类可以不予理会*                 |                                                  |
|                | *若父类方法抛出编译时异常*                                   |                                                  |
|                | *则：子类抛出的异常个数少于等于父类*; *子类抛出的异常类型小于等于父类* |                                                  |
| ***方法体***   | *子类的内容可以与父类的不一致*                               | *每一个重载方法执行过程不一致*                   |








###### 每一个类都有继承类

> 若：不写extends关键字，则默认继承Object类，这是最大的类，是老祖宗级别的类
>
> - **Object类很重要，是任何一个"引用类型"的父类，直接或间接的继承Object类，Object类没有父类**
>
> 若：写了extends关键字，则子类继承的是extends关键字后面的那个类




```java
public class Animal{ }


class Person extends Animal(

	public static void main(String[] args){

		Person p = new Person();

		/*
			若：在这里利用对象调用（ p. )，可以调到方法吗？
				是可以的，可以调到Object类中的方法。因为Animal类默认继承了Object类，
				而Person类又继承了Animal类，所以Person类也继承了Object类。这也是多继承的一种实现方式（继承的传递）
		*/
	}
)

```

 



###### Object类中有哪些方法？

```json
hashCode()　						将对象在内存中经过计算机计算得到一个int整数
										  在底层中的源码是：public native int hashCode();

equals()　　					 用来比较两个对象的内容，Object类中默认是 ==
										  == 可以比较基本数据类型（比较的是值），也可以比较引用数据类型（比较的是地址）
										  注：若想要改变其比较规则，则需要重写这个方法
										  底层中的源码是：
											public boolean equals（Object ob）{
											  return （this == ob );
											}

toString()  					 将对象转化为字符串
										  底层中的源码为(这个了解即可）：
										  public String toString(){
												return getClass.getName + "@" + Integer.toHexString( hashCode() );
											}

getClass()  					 获取对象对应类的类映射（ 反射机制技术，映射在流操作原理中会说明 ）

wait()      					 让线程处于挂机等待状态（ 多线程的时候需要用到 ）—— 存在方法重载

notify()    					 把线程唤醒（ 一样在多线程的时候需要用到 ）

notifyAll() 					 把所有线程唤醒（ 还是在多线程的时候需要用到 ）

finalize()  					 权限修饰符是protected，在对象被垃圾回收机制（GC）回收的时候，系统会默认调用这个方法

clone()     					 权限修饰符是protected，为了克隆对象(ps：23种设计模式之创建型模式的原型模式中，
											其中一种克隆模式用到了这个方法[会进行重写])

```

 



###### Java中继承是单继承

**指的是：每一个类只能继承一个类。extends关键后面的那个类 或 是默认继承Object类 **



**是否可以实现多继承的效果？** 粗略来说：是可以的，严格来说不算

1. 可以通过“继承的传递”（ 前面已提到 ）
2. 另外还可以通过“多实现”（ 后续接口的时候会做详细说明 ）



怎么理解继承？找下图堆内存那里剥皮儿就可以找到一个类所拥有的属性 / 方法了，注：下面这个图只是为了好理解
![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213348-467972681.png)





 

###### 关于this和super关键字的使用

> **this**：是一个指代词，指代的是“某一个对象”，即：调用当前类的属性与方法 的 那个对象。前面已经玩过了，不玩了
> **super**：也是一个指代词，代指的也是“某一个对象”，即：调用当前类的属性与方法 的 那个对象 的 父类对象

 

**1、调用“一般属性”和“一般方法”**

> **二者都可以调用“一般属性”和“一般方法”，可以放在类成员的任意位置（ 属性、方法、构造方法、块 ）**

 

调用“一般方法”时：可以来回互相调用（ 写法和编译都好用，但是运行时会出问题，`stackOverFlowErro 栈内存溢出`）

**原因**：就是互相来回调用，这就不断的创建东西、不断地执行程序，不就成死递归了吗，就和死循环一样，栈内存都堆满了




**什么是递归？** 就是不断调用自身，自己玩自己，简称：自wei。有一个典型的故事来理解

从前山里有座庙，庙里有个老和尚跟小和尚讲故事：讲的什么故事？讲的从前山里有座庙，庙里有个老和尚跟小和尚讲故事 ..........

 



**2、调用“构造方法”**

> **调用“构造方法”时：必须放在构造方法的第一行**

 

**注意点**：`this`和`super`在一个构造方法中调用另一个构造方法时，不能同时出现在第一行（因为两个都是必须放在第一行的，哪有位置放得下）

**另外**：在构造方法中没有用`this`或者`super`时，系统会默认在第一行调用，没有继承关系时，用的是`this()`，有继承关系时，会在子类的构造中使用`super()`[不一定就非是`this()`和`super()`，也可以是使用`this`和`super`调本类中的其他方法]

**但是**：如果自己在构造方法的第一行写了`this()`或者`super()`的调用，那么就会覆盖系统默认的调用，从而用自己写的`this()`或者`super()`调用的这个



**最后**：在构造方法中，不能来回互相调用，因为编译不好使，原因和前面的调用一般方法一样



 



##### （二）包含关系

> **这是 `A has - a B`的关系。指的是 一个类的属性中用到了另一个类的引用，但细分的话就有：组合、聚合、关联**，分为这几个的原因如下：
>
>  因为中国的语义问题，博大精深呐，歪果仁的表达继承就完了，但是在我国，这个文字可以延伸出来很多东西[ ps：我说的是 这个词义在中国和国外的区别啊，不是思想、逻辑 ]
>
> 说白了：这三者就是亲密程度不一样而已






###### 预知：包 package

这个已经在前面的代码中用过了，如：`cn.zixiegongzi.studyoop`，这就是在cn包下有一个xiegongzi包，下面又有studyoop的包

当类很多的时候，不方便管理了，所以就需要一个东西来帮忙 ： 包

**包可以理解成文件夹**

**当使用包编译时，在我们类的第一行会出现`package 路径名`，若：需要同时有`package`和`import`，则`package`必须放在第一行，`import`放在其后（ **`package`只能有一个，`import`可以有多个** ）**

 



###### 组合

> **定义：一荣俱荣、一毁俱毁。** 好比：人和脑袋，这就是整体和部分的关系，不可分割（不然把脑袋砍下来试一下？）




**例子**

```java
public class A{

	// 组合，直接属性中直接new实例
	private A a = new B();
}


class B{}

```

 

###### 聚合

> **好比：电脑和主板，还是整体和部分的关系，但是创建时有可能是分隔开的**

 

**例子**

```java
public class A{

	// 聚合，属性中没有new出实例
	private A a;
}


class B{}

```

 

###### 关联

> **好比：人有汽车、人有电脑。也是整体和部分的关系，可以分割，是后来组合在一起的。关联又分为单关联、双关联**
>
> - **单关联：一个类的属性中用到了另一个类的引用，即：上面说的组合、聚合都可以是这种单关联**
> - **双关联：两个类中都用到了彼此，即：我中有你，你中有我**






##### （三）依赖关系

> **是`A need - a B` 的关系。二者不是一开始就组合在一起的，更不是整体和部分的关系，而是因为一件事让他们关联起来了，并且这件事情做完，他们的关系就解散了。**
>
> **场景理解：** 屠夫杀猪
>
> - 屠夫 ————> 需要做一件事 ————> 杀猪
>
>
> **依赖关系在java中的实现：** 一个类中的方法 用到了 另一个类的对象。具体实现方式有以下3种：
>
> - 通过给方法传参（ 最常用这种 ）
> - 直接在方法中创建对象
> - 使用聚合关系拿到另一个类的引用(有参构造让调用者传入实例对象)，然后在方法中使用这个对象

 


**建议：**

- 在编程中最好别直接用继承（ 即：`extends` ），因为继承增加了耦合度
- 因此类与类之间的关系一定要尽量考虑“高内聚（ 一个方法只做一件事 ）、低耦合（ 继承 > 包含[组合 > 聚合 > 关联] > 依赖”。包含关系和依赖关系最适合 ）





### 05 接口

#### （一）什么是接口？

> **定义：抽象类抽象到了极致就是接口。** 即：只知道有这么一个东西，但是这个东西里面还有哪些东西、以及这个东西后面是怎么做的不知道
>
> - 比如：王者荣耀。有法师、刺客、坦克....，但是具体有哪些法师、哪些刺客并不知道，甚至某一个法师 / 刺客技能是怎么样的、有什么被动都不知道（ 等着实现类来做 ）
>
> **可以这么理解：接口也是类的一种结构(这个说法不对，是为了便于理解)，只是用interface替代了原本的class关键字而已**

 

```java
/**
 * 声明一个接口
 */
public interface Demo{
}

```

 



#### （二）接口有哪些成员组成？

##### 属性

> 不能含有一般属性，必须是**公有的 静态的 常量**。即：`public static final 大写英文单词的常量名;`

 

```java
public interface Demo{

	public static fianl String MY_NAME= "紫邪情";
		/*
			这样定义就是对的 另：public static final不写也行，默认也是这个
			同时注意静态常量的命名规范，多个单词用下划线分割
	  */

	/*
		这么写也行
		如果这接口中里面只有这样的属性，那这个写法有些人也叫“特殊接口 / 常量接口（ 全是常量 ）”
	*/
	String SPEAK= "这种写也行“;
	/*
		这种常量接口写法有什么用？
			假如：需要做判断，那个值输入的是常量又是字符串的时候，那么为了以防万一别人手贱在中间打了一个空格
			或者：那人手残输错了一个字母，那就可以用到这个知识点预防了
			当然：后续会了枚举类，那么全局常量就可以放到枚举类中
	*/
}

```

 

##### 方法

> 不能含有一般方法，必须是**公有的 抽象的 方法**，即：`public abstract 返回值类型 方法名( 参数列表 );`
> **注**：接口没有构造方法（ 对于接口中对方法的条件都不满足 ），也没有块（ 因为块就是一个具体化的，接口中不允许有具体化的，接口是最抽象的）
>
> - 普通类(即`class`定义) —抽象一下—> 抽象类(即：`abstract class`定义) —抽象一下—> 接口(即：`interface`定义)，所以接口是最抽象的

 


```java
public interface Demo{

	public abstract void eat();
		/*
			这样定义就是对的
			另：public abstract这个不要也行，默认也是这个（ 开发中也是没写这两个修饰符 ）
		*/

	// 这么写也行
	void sleep();
}

```





#### （三）如何使用接口？

> 想要使用接口，就必须通过子类多实现（ 使用关键字`implements` ，和前面的继承差不多 ）来做事

 

**例子**

```java
/**
 * 定义一个接口
 */

public interface Demo1{
	void eat();
}


/**
 * 实现接口，这里只是实现了一个
 * 实现多个接口的话，直接在implements后面接上需要实现的其他接口就行，多个用 , 隔开
 */
class Demo2 implements Demo1{
	// 如：implements Demo1，Demo2，Demo3

	// 重写接口中的方法（ 具体化 / 编写相应的逻辑代码 ）
	  public void eat(){
			System.out.println("你吃了个西北风");
	}
}

```

 



#### （四）接口继承

> 就是一个接口 继承 另一个接口，但是：**可以支持多继承，中间用 逗号 隔开**

 


```java
interface A{
		void a();
}


interface B{
	void b();
}

interface C extends B,A {
	void c();
}



class X implements C {
	/*
		这里X就实现了C一个类，但重写了3个抽象方法之后，X类就有3个自己的方法了,同时得出答案：是支持多继承的( 因为C继承了A、B )。
		另外：@Override是重写的意思，这是注解，暂时先不管
	*/
	@Override
	public void a() {}

	@Override
	public void b() {}

	@Override
	public void c() {}
}

```



**这种接口继承拿来有什么用？**

- 如果自己在前面写的一个接口H中的一些方法是自己正在写的类 / 接口中也想要拥有的方法，同时接口F中一些方法也是自己想要的，那就H继承F，然后让自己正在写的类 / 接口去实现接口H 或者 继承接口H就行了。当然：这种用法是不得不需要时才玩的，这种增加了类与类之间的耦合度，所以不到万不得已坚决不用



 

##### （五）“特殊接口/常量接口”的补充

以下的这种设计思想其实就是23种设计模式中创建型模式的“简单工厂模式”

 

**接口定义：**

```java
/**
 * 常量接口
 * 车的品牌
 */
public interface Brand{

	/**
	 * 静态常量：要是有另外品牌的车，则直接在这里面加入相应的内容，最后去后面Shop类中加入相应的if语句就可以了
	 */
	String BRAND_HAVEL = "havel";
	String BRAND_BYD = "byd";
	String BRAND_CHANGAN = "changAn";
}



/**
 * 车类
 */
public abstract class Car {

	private String name;

	public abstract void run();

	public Car() {}

	public Car(String name) {
		this.name = name;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}


```

 

**实现类逻辑编写**

```java
/**
 * 车类的实现类
 * 比亚迪车
 */
public class Byd extends Car{

	public Byd(String name) {
		super(name);
	}

	@Override
	public void run() {
		System.out.println(this.getName() + "行驶了");
	}
}



/**
 * 哈佛车
 */
class Havel extends Car{

	public Havel(String name) {
		super(name);
	}

	@Override
	public void run() {
		System.out.println(this.getName() + "行驶了");
	}
}



/**
 * 长安车
 */
class ChangAn extends Car{

	public ChangAn(String name) {
		super(name);
	}

	@Override
	public void run() {
		System.out.println(this.getName() + "行驶了");
	}
}




/**
 * 销售车的类
 */
class Shop{

	public static Car salesCar(String name){

		if (name.equals(Brand.BRAND_BYD)){
		/*
			使用静态常量，原因：要是使用字符串的话，万一手贱多打了一个空格，
			或者直接多打了一个符号，那比较结果不就不同了吗，所以直接使用“静态常量”不是很保险吗
			这也就是常量接口的好处
		*/

			return new Byd("比亚迪-汉");
		}

		if (name.equals( Brand.BRAND_HAVEL)){
			return new Havel("哈佛-h6");
		}

		if (name.equals( Brand.BRAND_CHANGAN)){
			return new ChangAn("长安-乱扯一个");
		}

		return null;
	}
}




/**
 * 测试类
 */
class Test{

	public static void main(String[] args) {

		Car car = Shop.salesCar(Brand.BRAND_BYD);
		car.run();

		Car car2 = Shop.salesCar(Brand.BRAND_HAVEL);
		car2.run();

		Car car3 = Shop.salesCar(Brand.BRAND_CHANGAN);
		car3.run();
	}
}

```



 

##### （六）接口与别的类的关系

> 接口————>不能继承————>其他的类（ 普通类、抽象类 ）
> 抽象类————>可以多实现————>接口
> 接口————>可以“多继承”————接口			注意：是多继承



**1、接口————>可以“多继承”————接口例子**

```java
interface Demo{
	void eat();
}



interface Demo1{
	void sleep();
}



/**
 * 在接口中这种是可以得吃的
 */
public interface Demo2 extends Demo,Demo1{

	// 另外要写的代码

}

```

 


**2、普通类————>想要直接多实现————>接口**

- 如果接口中有抽象方法，则：必须具体化（ 重写 ）接口中的抽象方法( 对于接口，实际开发的玩法就是这种 )
- 画图分析接口、抽象类、普通类彼此怎么实现，以及分别负责的是什么

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213068-589855556.png)





### 06 多态

假如有这么一个关系图：

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213068-1041161722.png)

 

> **多态就是指：父类类型的引用  指向  子类对象 如：`Person p = new Teacher();`**
>
> - 此引用只能调用Person这个父类中的属性和方法 。 因为：虽然`new Teacher()`才是对象，但是它的引用类型实际上是`Person`这个父类，只是这个引用指向了`Teacher`这个子类
>
>
> **如果子类中有和父类一样的属性名，那么调用执行的是子类中的属性**
> **如果子类将父类的方法重写了，那么调用执行的是子类中重写的方法**
> **多态的名言：在多态中，编译看左边（ 能调用哪些东西 ），运行看右边（ 实际运行的是谁 ）**





#### （一）instanceOf 关键字

**如果想要调用子类中独有的成员，则：需要进行强制类型转换（ 也叫做造型 / 铸型 / 向上或向下转型 ）。** 但是：这种有可能会产生一种运行时异常：`classCastException`造型异常，若想要避免这种异常，则：需要使用instanceOf关键字

 

> **instanceOf关键字：为了判断当前对象 是不是 某个类的对象**




**用法**


```json
对象名  instanceOf  类
```

 


```java
/**
 * 父类
 */
public class Animal {

	private String name ;
	private String sex;

	public Animal(){}

	public Animal(String name,String sex){
		this.name = name;
		this.sex = sex;
	}


	public void eat() {
		System.out.println("动物的吃饭方法");
	}

	public void sleep() {
		System.out.println("动物的睡觉方法");
	}
}



/**
 * 子类
 */
package cn.xiegongzi.test;

public class Person extends Animal{

	public Person{}

	@Override
	public void talk() {
		System.out.println("人类的说话能力");
	}

	@Override
	public void study() {
		System.out.println("人类的学习能力");
	}
}



/**
 * 测试类
 */
class Test{

	public static void main(String[] args) {
		Animal animal = new Person();

		/*
			在这里我想要通过这个animal对象引用 去 调用Person类中的特有方法
			但是：在这里只能调用Animal类中的方法、属性( 需要父类提供get和[setter方法]，重点为了表达这个知识，所以省掉了
			在IDEA编辑器中可以通过alt+insert键选择getter和setter之后全选，然后生成 ）

			因此可以通过向下转型（ 向下转型 是因为Animal是父类，Person是子类，所以父 ———— > 子 ，就是向下转，其他的几个名字意思也是这么个逻辑
		*/
		if (animal instanceof Person) {

			Person person = (Person)animal;
			person.study();
			person.talk();
			person.eat();　　// 余下这两个都可以调到
			person.sleep()
			}
		}
	}

```

 





#### （二）多态示例

继续借用前面的图

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213112-83253487.png)

 

实操举例

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213466-581168026.png)



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213155-628506213.png)


![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213198-1117241048.png)


1、`Objdect obj = new Teacher(); `问：obj可以调用哪些属性和方法？

- 这种只能调用Object类中的9大方法

 

2、`Animal a = （Animal）obj;` 问：a可以调用哪些方法和属性？

- 这种可以调用Object类中9大方法，以及：`a.name`、`a.sleep()`、`a.eat()`[Animal类中的属性 ， 但是注意：`Teacher`重写了`Animal`中的`eat()`、`Person`重写了`Animal`中的`sleep()` ]

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213346-898476028.png)




3、`Person p = （Person) obj;`问：p可以调用哪些属性和方法？

- 这种可以调用Object类中的9大方法、Person类中的属性、Person重写、Person子类重写（ 如：输出的老师吃饭方式 ）的方法、以及自己独有的方法

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213437-1530417549.png)



 


**小结：在多态中**

- 当子类重写了父类的方法时，执行的是重写的那个方法
- 当子类有和父类相同名字的属性时，执行的是：子类中的属性





#### （三）开发中多态的应用场景

开发中不会像上述`Animal a = new Person()`这样明明白白的体现出多态

 

##### 父类类型做参数


```java
/**
 * 主人
 * 假设还有Animal动物类，然后Dog和Cat继承了Animal
 */
public class Master {

	/*
	这两个方法重载，可以使用多态优化为一个方法
	// 接收一个 Dog 类型的参数
	public void  feed(Dog obj){
		obj.eat();
	}

	// 接收一个 Cat 类型的参数
	public void  feed(Cat obj){
		obj.eat();
	}

		*/


	/**
	 * 优化
	 * 接收一个 Animal 类型的参数
	 */
	public void  feed(Animal obj){ // Animal obj = cat = new Cat(); Dog是同样的道理
		obj.eat();
	}


/**
 * 测试
 */
	public static void main(String[] args) {
		Dog dog = new Dog();
		Cat cat = new Cat();
		Master master = new Master();
		master.feed(  cat  ); // 这里可以传  cat 和  dog对象 均可。
	}
}

```




###### 父类类型做返回值

```java
/**
 * 主人
 */
public class Master {

	// 接收一个 Animal 类型的参数
	public void  feed( Animal obj  ){
		obj.eat();
	}

	// 返回值为父类类型
	public  Animal  kill(){

		Random random = new Random(); // 这是生成一个随机数，Math工具类中的方法，在工具类中会说明
		int num = random.nextInt(2);

		if( num == 0 ){ // 杀个狗
			Dog dog = new Dog();
			return dog;　　// 这种返回回去Animal是父类，也接收得了

		}else{ // 杀个猫
			Cat cat = new Cat();
			return cat;　　// 同样的Animal照样可以接收
		}
	}
}

```

 



###### 父类类型定义数组，保存子类类型对象

```java
public static void main(String[] args) {

	Animal[] arr = new Animal[4];

	arr[0] = new Cat();
	arr[1] = new Dog();
	arr[2] = new Cat();
	arr[3] = new Dog();

	for( Animal an : arr){
		an.eat();
	}
}

```

 



### 07 接口、多态、继承总结

##### （一）接口多态

> **定义：接口类型的引用 指向 实现类的对象**
>
> - 该引用只能调用接口中定义的属性和方法，但是真正执行的是实现类中重写的方法，其实和前面的多态一样

 



##### ）二）标志接口

> **定义：就是把另外一些类的一个共同点提取出来，这个共同点成为一个接口。** 这个接口里面可以啥都没有，然后用来让前面说的那些具有这个共同点的类 实现 这个标志接口，即：实现这个接口的类 就被做了一个标记

 

这个标志接口有什么用？

先来看个简单的例子：java中的`serializable(序列化)`接口，它就是一个标志接口，这个序列化接口在流技术的IO流技术中会用，目前不用了解都行，只是为了说明这个标志接口而已。这个接口底层中的源码如下：

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230209103213551-51464323.png)



标志接口实例



**1、定义标志接口**

```java
/**
 * 创建一个能飞的接口，用来给别人做标记的
 */
public interface CanFly {}

```

 

**2、标志接口的实现类**

```java
/**
 * 鸟类，标志一下，它可以飞（即：实现CanFly接口）
 */
public class Bird implements CanFly{

	public void birdFly() {
		System.out.println("小鸟儿飞.....");
	}
}

```





**3、编写两个类来识别是否能飞**

```java
/**
 * 飞机类，也标志一下，也可以飞
 */
public class Plane implements CanFly{

	public void planeFly(){
		System.out.println("飞机飞.....");
	}
}


```

 


```java
/**
* 猪类，这玩意能飞？是飞猪？
*/
public class Pig {

	public void eat() {
		System.out.println("一天就知道吃吃吃....");
	}
}

```





**4、测试**

```java
/**
 * 测试类
 */
public class Test{

	/**
	 * 这里采用Object类只是为了能够让什么类型对象都可以传进来，测试嘛
	 */
	public void fly(Object obj){

		// 这里标志接口的好处不就来了吗，在这里就可以用这个标志来判断了，多态的思想
		if (obj instanceof CanFly){
			System.out.println(obj + "这玩意儿可以飞");
		}else {
			// 这个是自定义异常，暂时先不考虑，后续在java高级篇中会做详细说明
			throw new RuntimeException("这玩意儿尼玛不能飞");
		}
	}


	public static void main(String[] args) {

		Test test = new Test();
	//        test.fly( new Bird() );
	//        test.fly( new Plane() );     // 这两个玩意儿都可以，但是重点不在这里，注意：这里传的对象是子类对象，多态隐式玩法

	test.fly(new Pig());
	/*
		在这里我new一个Bird、Plane都可以飞，但是Pig这玩意儿也可以传到fly()方法里面去
		所以如果不用标志接口整一下,表明哪些东西是可以飞的，那不得啥玩意儿都可以进去执行了
	*/
	}
}

```





### 08 内部类

> **指的是：把一个类 定义在 另一类的内部**
>
> **内部类可以定义在哪里？**
>
> - 定义在类的内部：与类的成员层次一致
> - 定义在类成员的内部：和方法的局部变量处于一个层次



 

##### （一）成员内部类

代码简化的时候会用，但是有更简化的

> **指的就是：把一个类定义在另一个类的内部，作为成员。与类的属性和方法处于一个层次**
>
> 成员内部类可以和正常类一样有属性、方法这些，同时也可以有修饰符，好处如下：
>
> - 减少了一个`.java`文件
> - 成员内部类可以访问外部类的所有成员(包括私有的)，因为成员内部类和属性以及方法处于同一个层次

 


**内部类的创建、在内部类中调用外部类、以及调用内部类里面的属性和方法**

```java
	public class Persion {

		private String name = "魔族";
		private int age = 30000;

		public void eat() {
			System.out.println("这是人类的吃饭方法");
		}

		class Teacher {

			private String sex = "女";
			private String phone = "123456";

			public void sleep() {

				System.out.println( "这是老师的睡觉方法" );
				eat();
				System.out.println( "这里调用了外部类的name属性：" + name );
				System.out.println( "这里调用了外部类的age属性: " + age );
			}
		}

		public static void main(String[] args  {

			System.out.println("这是外部类的调用");
			Persion p = new Persion();
			System.out.println(p.age);
			System.out.println(p.name);
			p.eat();
			System.out.println();

			System.out.println("=========成员内部类的调用如下：==============");
			Teacher teacher = p.new Teacher(); // 创建内部类的对象
			System.out.println(teacher.sex);
			System.out.println(teacher.phone);
			teacher.sleep();
		}
	}

```






##### （二）局部内部类

代码简化时可以用，但是有更简化的

> **指的是：把一个类 放 在另一个类的方法 / 块里面，和方法的局部变量处于一个层次**

 



**局部内部类的创建、使用**


```java
	public class Animal {

		private String name = "动物的名字";
		private int age = 100;

		public void eat() {
			System.out.println( "动物的吃饭方法" );

			class Pig{

				String name = "猪的名字";

				public void sleep() {

					System.out.println(); // 换行 为了好看效果

					System.out.println("这里面就是局部内部类的方法了：");

					System.out.println("猪的睡觉方式");

					System.out.println("这里调用了外部类的name属性： " + Animal.this.name);
					System.out.println("这里调用了外部类的age属性: " + Animal.this.age);
				}
			}

			Pig pig = new Pig();                // 想要访问局部内部类的属性和方法
			System.out.println(pig.name);     // 直接在局部内部类所放的方法中  直接创建这个局部内部类的对象
			pig.sleep();                        // 有了这个对象就可以调用这个局部内部类的属性和方法了
		}

		public static void main(String[] args) {

			System.out.println("外部类的调用如下：");
			Animal animal = new Animal();
			System.out.println(animal.name);
			System.out.println(animal.age);

			animal.eat(); // 这个方法中创建了局部内部类的对象 以及调用了它的属性和方法
		}
	}

```








##### （三）匿名内部类

通常用来写接口 / 抽象类的子类。以前那种传统的方法是重新写一个类来实现接口 / 抽象类，而匿名内部类就是现用现写，不用再单独写一个类了 （ 当然：越到最后越有更简化的 ）


**一样有成员匿名内部类 和 局部匿名内部类**



```java
	public interface Animal {

		void sleep();

		// 这里是为了方便，所以就直接写在这个接口里面了，这个实现类可以放在需要的地方写都可以（ 都是同样的道理 ）
		Animal animal = new Animal() {
			private char sex = '男';

			@Override　　// 这是注解，就是重写的意思，在注解知识中会做详细说明
			public void sleep() {
				System.out.println( "接口和抽象类的子类实现" );
			}
		};
	}

	class Test{

		public static void main(String[] args) {
			Animal.animal.sleep(); // 对成员匿名内部类的调用
		}
	}

```




**匿名内部类小结：什么时候会用匿名内部类？**

高频使用：面向接口编程原则嘛，虽然后面有更简化的方式，但是以上这些也必须会，因为什么都有好和坏，因此需要看实际来应用。 即：**发现要传递的那个参数是一个接口类型的时候，但是又不想去写一个类去实现那个接口类，那么就现写现用**