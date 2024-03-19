## JAVA 集合

### 接口继承关系和实现

集合类存放于Java.util包中，主要有 3 种：set(集）、list(列表包含Queue）和map(映射)。

1. Collection：Collection是集合List、Set、Queue的最基本的接口。
2. Iterator：迭代器，可以通过迭代器遍历集合中的数据
3. Map：是映射表的基础接口

![image-20240205132034914](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163611189-1450015342.png)

![image-20240205132048180](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163611159-1209475998.png)



### LIST

Java的List是非常常用的数据类型。List是有序的Collection。Java List一共三个实现类：分别是ArrayList、Vector和LinkedList。

![image-20240205132109251](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163611455-557686900.png)

#### ArrayList （数组）

ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要将已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。

#### Vector （数组实现、线程同步）

Vector与ArrayList一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一个线程能够写Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问ArrayList慢。

#### LinkList （链表）

LinkedList是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，他还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。


### SET

Set注重独一无二的性质,该体系集合用于存储无序(存入和取出的顺序不一定相同)元素，值不能重复。对象的相等性本质是对象hashCode值（java是依据对象的内存地址计算出的此序号）判断的，如果想要让两个不同的对象视为相等的，就必须覆盖Object的hashCode方法和equals方法。

![image-20240205132147756](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163611351-1350141299.png)

##### HashSet（Hash表）

哈希表边存放的是哈希值。HashSet存储元素的顺序并不是按照存入时的顺序（和List显然不同） 而是按照哈希值来存的所以取数据也是按照哈希值取得。元素的哈希值是通过元素的hashcode方法来获取的, HashSet首先判断两个元素的哈希值，如果哈希值一样，接着会比较equals方法 如果 equls结果为true ，HashSet就视为同一个元素。如果equals 为false就不是同一个元素。

哈希值相同equals为false的元素是怎么存储呢,就是在同样的哈希值下顺延（可以认为哈希值相同的元素放在一个哈希桶中）。也就是哈希一样的存一列。如图 1 表示hashCode值不相同的情况；图 2 表示hashCode值相同，但equals不相同的情况。

![image-20240205132200112](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163610648-431819097.png)


HashSet通过hashCode值来确定元素在内存中的位置。一个hashCode位置上可以存放多个元素。

##### TreeSet（二叉树）

1. TreeSet()是使用二叉树的原理对新add()的对象按照指定的顺序排序（升序、降序），每增加一个对象都会进行排序，将对象插入的二叉树指定的位置。
2. Integer和String对象都可以进行默认的TreeSet排序，而自定义类的对象是不可以的，自己定义的类必须实现Comparable接口，并且覆写相应的compareTo()函数，才可以正常使用。
3. 在覆写compare()函数时，要返回相应的值才能使TreeSet按照一定的规则来排序
4. 比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则分别返回负整数、零或正整数。

##### LinkHashSet（HashSet+LinkedHashMap）

对于 LinkedHashSet 而言，它继承与HashSet、又基于LinkedHashMap 来实现的。LinkedHashSet底层使用LinkedHashMap来保存所有元素，它继承与HashSet，其所有的方法操作上又与HashSet相同，因此LinkedHashSet 的实现上非常简单，只提供了四个构造方法，并通过传递一个标识参数，调用父类的构造器，底层构造一个 LinkedHashMap来实现，在相关操作上与父类HashSet的操作相同，直接调用父类HashSet的方法即可。

### MAP

![image-20240205132249014](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163611330-778686672.png)

#### HashMap （数组 + 链表 + 红黑树）

HashMap根据键的hashCode值存储数据，大多数情况下可以直接定位到它的值，因而具有很快的访问速度，但遍历顺序却是不确定的。 HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全，即任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致。如果需要满足线程安全，可以用 Collections 的synchronizedMap 方法使HashMap 具有线程安全的能力，或者使用ConcurrentHashMap。我们用下面这张图来介绍HashMap 的结构。

##### JAVA7实现

![image-20240205132313757](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163610887-1218735213.png)

大方向上，HashMap 里面是一个数组，然后数组中每个元素是一个单向链表。上图中，每个绿色的实体是嵌套类 Entry 的实例，Entry 包含四个属性：key, value, hash 值和用于单向链表的 next。

1. capacity：当前数组容量，始终保持 2^n，可以扩容，扩容后数组大小为当前的 2 倍。
2. loadFactor：负载因子，默认为 0.75。


3. threshold：扩容的阈值，等于 capacity * loadFactor

##### JAVA8实现

Java8 对 HashMap 进行了一些修改，最大的不同就是利用了红黑树，所以其由 数组+链表+红黑树 组成。

根据 Java7 HashMap 的介绍，我们知道，查找的时候，根据 hash 值我们能够快速定位到数组的具体下标，但是之后的话，需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度，为 O(n)。为了降低这部分的开销，在 Java8 中，当链表中的元素超过了 8 个以后，会将链表转换为红黑树，在这些位置进行查找的时候可以降低时间复杂度为 O(logN)。

![image-20240205132346673](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163610999-2020708507.png)

#### ConcurrentHashMap

##### Segment段

ConcurrentHashMap 和 HashMap 思路是差不多的，但是因为它支持并发操作，所以要复杂一些。整个 ConcurrentHashMap 由一个个 Segment 组成，Segment 代表”部分“或”一段“的意思，所以很多地方都会将其描述为分段锁。注意，行文中，我很多地方用了“槽”来代表一个segment。

##### 线程安全（Segment 继承 ReentrantLock 加锁）

简单理解就是，ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

![image-20240205132417791](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163611256-11500546.png)


##### 并行度（默认 16 ）

concurrencyLevel：并行级别、并发数、Segment 数，怎么翻译不重要，理解它。默认是 16 ，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。再具体到每个 Segment 内部，其实每个 Segment 很像之前介绍的 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。

##### Java8实现 （引入了红黑树）

Java8 对 ConcurrentHashMap 进行了比较大的改动,Java8 也引入了红黑树。

![image-20240205132446820](https://img2023.cnblogs.com/blog/2421736/202402/2421736-20240205163611018-1642482284.png)


### HashTable （线程安全）

Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写 Hashtable，并发性不如 ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

### TreeMap （可排序）

TreeMap实现SortedMap接口，能够把它保存的记录根据键排序，默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。

如果使用排序的映射，建议使用TreeMap。

在使用TreeMap时，key必须实现Comparable 接口或者在构造 TreeMap传入自定义的Comparator，否则会在运行时抛出java.lang.ClassCastException类型的异常。

参考：https://www.ibm.com/developerworks/cn/java/j-lo-tree/index.html

### LinkHashMap （记录插入顺序）

LinkedHashMap 是HashMap 的一个子类，保存了记录的插入顺序，在用Iterator 遍历LinkedHashMap时，先得到的记录肯定是先插入的，也可以在构造时带参数，按照访问次序排序。
参考 1 ：http://www.importnew.com/28263.html

参考 2 ：http://www.importnew.com/20386.html#comment- 648123