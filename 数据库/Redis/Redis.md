# 认识Redis

Redis官网：https://redis.io/

Redis诞生于2009年全称是**Re**mote  **D**ictionary **S**erver 远程词典服务器，是一个基于内存的键值型NoSQL数据库

**特征**：

- 键值（key-value）型，value支持多种不同数据结构，功能丰富
- 单线程，每个命令具备原子性
- 低延迟，速度快（基于内存.IO多路复用.良好的编码）
- 支持数据持久化
- 支持主从集群、分片集群



**NoSQL**可以翻译做Not Only SQL（不仅仅是SQL），或者是No SQL（非SQL的）数据库。是相对于传统关系型数据库而言，有很大差异的一种特殊的数据库，因此也称之为**非关系型数据库**

关系型数据是结构化的，即有严格要求，而NoSQL则对数据库格式没有严格约束，往往形式松散，自由

可以是键值型：

![](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723225400287-213396476.png)



也可以是文档型：

![](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723225400957-1290961281.png)





甚至可以是图格式：

![](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723225401127-1039329556.png)





在事务方面：

1. 传统关系型数据库能满足事务ACID的原则
2. 非关系型数据库往往不支持事务，或者不能严格保证ACID的特性，只能实现基本的一致性



除了上面说的，在存储方式.扩展性.查询性能上关系型与非关系型也都有着显著差异，总结如下：

![](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723225908958-2048137091.png)

- 存储方式
  - 关系型数据库基于磁盘进行存储，会有大量的磁盘IO，对性能有一定影响
  - 非关系型数据库，他们的操作更多的是依赖于内存来操作，内存的读写速度会非常快，性能自然会好一些

* 扩展性
  * 关系型数据库集群模式一般是主从，主从数据一致，起到数据备份的作用，称为垂直扩展。
  * 非关系型数据库可以将数据拆分，存储在不同机器上，可以保存海量数据，解决内存大小有限的问题。称为水平扩展。
  * 关系型数据库因为表之间存在关联关系，如果做水平扩展会给数据查询带来很多麻烦





# 安装Redis

企业都是基于Linux服务器来部署项目，而且Redis官方也没有提供Windows版本的安装包

本文选择的Linux版本为CentOS 7



## 单机安装

1. 安装需要的依赖

```shell
yum install -y gcc tcl
```

2. 上传压缩包并解压

```shell
tar -zxf redis-7.0.12.tar.gz
```

3. 进入解压的redis目录

```shell
cd redis-7.0.12
```

4. 编译并安装

```shell
make && make install
```

默认的安装路径是在 `/usr/local/bin`目录下

![image-20230723232738946](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723232740010-1566728255.png)



该目录已经默认配置到环境变量，因此可以在任意目录下运行这些命令。其中：

- redis-cli：是redis提供的命令行客户端
- redis-server：是redis的服务端启动脚本
- redis-sentinel：是redis的哨兵启动脚本





## 启动Redis

redis的启动方式有很多种，例如：

- 默认启动
- 指定配置启动
- 开机自启





### 默认启动

安装完成后，在任意目录输入redis-server命令即可启动Redis：

```shell
redis-server
```

这种启动属于“前台启动”，会阻塞整个会话窗口，窗口关闭或者按下`CTRL + C`则Redis停止





### 指定配置启动

如果要让Redis以“后台”方式启动，则必须修改Redis配置文件，就在之前解压的redis安装包下（`/usr/local/src/redis-6.2.6`），名字叫redis.conf

修改redis.conf文件中的一些配置：可以先拷贝一份再修改

```properties
# 允许访问的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
# 守护进程，修改为yes后即可后台运行
daemonize yes
# 密码，设置后访问Redis必须输入密码
requirepass 072413
```



Redis的其它常见配置：

```properties
# 监听的端口
port 6379
# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志、持久化等文件会保存在这个目录
dir .
# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.log"
```



启动Redis：

```sh
# 进入redis安装目录 
cd /opt/redis-6.2.13
# 启动
redis-server redis.conf
```



停止服务：

```sh
# 利用redis-cli来执行 shutdown 命令，即可停止 Redis 服务，
# 因为之前配置了密码，因此需要通过 -u 来指定密码
redis-cli -u 072413 shutdown
```





### 开机自启

可以通过配置来实现开机自启。

首先，新建一个系统服务文件：

```sh
vim /etc/systemd/system/redis.service
```

内容如下：

```conf
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /opt/redis-6.2.13/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

然后重载系统服务：

```sh
systemctl daemon-reload
```

现在，我们可以用下面这组命令来操作redis了：

```sh
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```

执行下面的命令，可以让redis开机自启：

```sh
systemctl enable redis
```







# 卸载Redis

1. 查看redis是否启动

```shell
ps aux | grep redis
```

2. 若启动，则杀死进程

```shell
kill -9 PID
```

![image-20230724175911257](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230724175913106-987380126.png)

3. 停止服务

```shell
redis-cli shutdown
```

4. 查看`/usr/local/lib`目录中是否有与Redis相关的文件

```shell
ll /usr/local/bin/redis-*

# 有的话就删掉

rm -rf /usr/local/bin/redis-*
```

5. 删除解压的Redis文件

```shell
rm -rf redis解压之后的路径
```









# Redis客户端工具

## 命令行客户端

Redis安装完成后就自带了命令行客户端：redis-cli，使用方式如下：

```sh
redis-cli [options] [commonds]
```

其中常见的options有：

- `-h 127.0.0.1`：指定要连接的redis节点的IP地址，默认是127.0.0.1
- `-p 6379`：指定要连接的redis节点的端口，默认是6379
- `-a 072413`：指定redis的访问密码 

其中的commonds就是Redis的操作命令，例如：

- `ping`：与redis服务端做心跳测试，服务端正常会返回`pong`

不指定commond时，会进入`redis-cli`的交互控制台：

![image-20230724180838657](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230724180840237-578785595.png)









## 图形化客户端

地址：https://github.com/uglide/RedisDesktopManager

不过该仓库提供的是RedisDesktopManager的源码，并未提供windows安装包。

在下面这个仓库可以找到安装包：https://github.com/lework/RedisDesktopManager-Windows/releases

下载之后，解压、安装

![image-20230724182022549](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230724182023537-593444.png)

![image-20230724182209444](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230724182210498-823892810.png)



Redis默认有16个仓库，编号从0至15.  通过配置文件可以设置仓库数量，但是不超过16，并且不能自定义仓库名称。

如果是基于redis-cli连接Redis服务，可以通过select命令来选择数据库

```shell
# 选择 0号库
select 0
```





# Redis常见命令

Redis是一个key-value的数据库，key一般是String类型，不过value的类型多种多样：

![1652887393157](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725122402725-1490057960.png)



查命令的官网： https://redis.io/commands 

在交互界面使用 help 命令查询：

```shell
help [command]
```

![image-20230725122759107](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725122800297-1332756108.png)









## 通用命令

通用指令是部分数据类型都可以使用的指令，常见的有：

- KEYS：查看符合模板的所有key。**在生产环境下，不推荐使用keys 命令，因为这个命令在key过多的情况下，效率不高**
- DEL：删除一个指定的key
- EXISTS：判断key是否存在
- EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除。内存非常宝贵，对于一些数据，我们应当给他一些过期时间，当过期时间到了之后，他就会自动被删除
  - 当使用EXPIRE给key设置的有效期过期了，那么此时查询出来的TTL结果就是-2 
  - 如果没有设置过期时间，那么TTL返回值就是-1
- TTL：查看一个KEY的剩余有效期



![image-20230725123855605](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725123856923-464274200.png)









## String命令

**使用场景：**

1. 验证码保存



String类型，也就是字符串类型，是Redis中最简单的存储类型

其value是字符串，不过根据字符串的格式不同，又可以分为3类：

* string：普通字符串
* int：整数类型，可以做自增.自减操作
* float：浮点类型，可以做自增.自减操作

![1652890121291](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725123958271-35047923.png)

String的常见命令有：

* SET：添加或者修改已经存在的一个String类型的键值对，对于SET，若key不存在则为添加，存在则为修改
* GET：根据key获取String类型的value
* MSET：批量添加多个String类型的键值对
* MGET：根据多个key获取多个String类型的value
* INCR：让一个整型的key自增1
* INCRBY：让一个整型的key自增并指定步长
  * incrby num 2 让num值自增2
  * 也可以使用负数，是为减法，如：incrby num -2 让num值-2。此种类似 DECR 命令，而DECR是每次-1

* INCRBYFLOAT：让一个浮点类型的数字自增并指定步长
* **SETNX**：添加一个String类型的键值对(key不存在为添加，存在则不执行)
* **SETEX**：添加一个String类型的键值对，并且指定有效期

**注：**以上命令除了INCRBYFLOAT 都是常用命令







## key的层级结构

Redis没有类似MySQL中的Table的概念，我们该如何区分不同类型的key呢？

可以通过给key添加前缀加以区分，不过这个前缀不是随便加的，有一定的规范

Redis的key允许有多个单词形成层级结构，多个单词之间用`:`隔开，格式如下：

![1652941631682](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725125028590-1578712035.png)

这个格式并非固定，也可以根据自己的需求来删除或添加词条

如我们的项目名称叫 automation，有user和product两种不同类型的数据，我们可以这样定义key：

- user相关的key：automation:user:1

- product相关的key：automation:product:1







## Hash命令

**使用场景：**

1. 对象保存



这个在工作中使用频率很高

Hash类型，也叫散列，其value是一个无序字典，类似于Java中的HashMap结构。

String结构是将对象序列化为JSON字符串后存储，当需要修改对象某个字段时很不方便：

![1652941995945](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725125452689-1165601556.png)



Hash结构可以将对象中的每个字段独立存储，可以针对单个字段做CRUD：

![1652942027719](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725125452747-418819141.png)



**Hash类型的常见命令**

- HSET key field value：添加或者修改hash类型key的field的值。同理：操作不存在数据是为新增，存在则为修改

- HGET key field：获取一个hash类型key的field的值

- HMSET：批量添加多个hash类型key的field的值

- HMGET：批量获取多个hash类型key的field的值

- HGETALL：获取一个hash类型的key中的所有的field和value
- HKEYS：获取一个hash类型的key中的所有的field
- HINCRBY：让一个hash类型key的field的value值自增并指定步长
- HSETNX：添加一个hash类型的key的field值，前提是这个field不存在，否则不执行







## List命令 - 命令规律开始变化

Redis中的List类型与Java中的LinkedList类似，可以看做是一个双向链表结构。既可以支持正向检索，也可以支持反向检索。

特征也与LinkedList类似：

* 有序
* 元素可以重复
* 插入和删除快
* 查询速度一般

常用来存储一个有序数据，例如：朋友圈点赞列表，评论列表等。

**List的常见命令有：**

- LPUSH key element ... ：向列表左侧插入一个或多个元素
- LPOP key：移除并返回列表左侧的第一个元素，没有则返回nil
- RPUSH key element ... ：向列表右侧插入一个或多个元素
- RPOP key：移除并返回列表右侧的第一个元素
- LRANGE key star end：返回一段角标范围内的所有元素
- BLPOP和BRPOP：与LPOP和RPOP类似，只不过在没有元素时等待指定时间，而不是直接返回nil

![1652943604992](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725130141530-484226078.png)







## Set命令

Redis的Set结构与Java中的HashSet类似，可以看做是一个value为null的HashMap。因为也是一个hash表，因此具备与HashSet类似的特征：

* 无序
* 元素不可重复
* 查找快
* 支持交集.并集.差集等功能

**Set类型的常见命令**

* SADD key member ... ：向set中添加一个或多个元素
* SREM key member ... ：移除set中的指定元素
* SCARD key：返回set中元素的个数
* SISMEMBER key member：判断一个元素是否存在于set中
* SMEMBERS：获取set中的所有元素
* SINTER key1 key2 ... ：求key1与key2的交集
* SDIFF key1 key2 ... ：求key1与key2的差集
* SUNION key1 key2 ..：求key1和key2的并集







## SortedSet / ZSet 命令

Redis的SortedSet是一个可排序的set集合，与Java中的TreeSet有些类似，但底层数据结构却差别很大。SortedSet中的每一个元素都带有一个score属性，可以基于score属性对元素排序，底层的实现是一个跳表（SkipList）加 hash表。

SortedSet具备下列特性：

- 可排序
- 元素不重复
- 查询速度快

因为SortedSet的可排序特性，经常被用来实现排行榜这样的功能。



SortedSet的常见命令有：

- ZADD key score member：添加一个或多个元素到sorted set ，如果已经存在则更新其score值
- ZREM key member：删除sorted set中的一个指定元素
- ZSCORE key member : 获取sorted set中的指定元素的score值
- ZRANK key member：获取sorted set 中的指定元素的排名
- ZCARD key：获取sorted set中的元素个数
- ZCOUNT key min max：统计score值在给定范围内的所有元素的个数
- ZINCRBY key increment member：让sorted set中的指定元素自增，步长为指定的increment值
- ZRANGE key min max：按照score排序后，获取指定排名范围内的元素
- ZRANGEBYSCORE key min max：按照score排序后，获取指定score范围内的元素
- ZDIFF.ZINTER.ZUNION：求差集.交集.并集

注意：所有的排名默认都是升序，如果要降序则在命令的Z后面添加REV即可，例如：

- **升序**获取sorted set 中的指定元素的排名：ZRANK key member
- **降序**获取sorted set 中的指定元素的排名：ZREVRANK key memeber









# Java操作：Jedis

官网：https://redis.io/docs/clients/

其中Java客户端也包含很多：

![image-20220609102817435](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725134433153-1221779103.png)



标记为❤的就是推荐使用的Java客户端，包括：

- Jedis和Lettuce：这两个主要是提供了“Redis命令对应的API”，方便我们操作Redis，而SpringDataRedis又对这两种做了抽象和封装
- Redisson：是在Redis基础上实现了分布式的可伸缩的Java数据结构，例如Map.Queue等，而且支持跨进程的同步机制：Lock.Semaphore等待，比较适合用来实现特殊的功能需求





## 入门Jedis

创建Maven项目

1. 依赖

```xml
<!--jedis-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.7.0</version>
</dependency>
```

2. 测试：其他类型如Hash、Set、List、SortedSet和下面String是一样的用法

```java
package com.zixieqing.redis;

import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import redis.clients.jedis.Jedis;

/**
 * jedis操作redis：redis的命令就是jedis对应的API
 *
 * @author : ZiXieqing
 */

public class QuickStartTest {
    private Jedis jedis;

    @Before
    public void setUp() throws Exception {
        jedis = new Jedis("host", 6379);
        // 设置密码
        jedis.auth("072413");
        // 设置库
        jedis.select(0);
    }

    @After
    public void tearDown() throws Exception {
        if (null != jedis) jedis.close();
    }

    /**
     * String类型
     */
    @Test
    public void stringTest() {
        // 添加key-value
        String result = jedis.set("name", "zixieqing");
        System.out.println("result = " + result);

        // 通过key获取value
        String value = jedis.get("name");
        System.out.println("value = " + value);

        // 批量添加或修改
        String mset = jedis.mset("age", "18", "sex", "girl");
        System.out.println("mset = " + mset);
        System.out.println("jedis.keys() = " + jedis.keys("*"));

        // 给key自增并指定步长
        long incrBy = jedis.incrBy("age", 5L);
        System.out.println("incrBy = " + incrBy);

        // 若key不存在，则添加，存在则不执行
        long setnx = jedis.setnx("city", "hangzhou");
        System.out.println("setnx = " + setnx);

        // 添加key-value，并指定有效期
        String setex = jedis.setex("job", 10L, "Java");
        System.out.println("setex = " + setex);

        // 获取key的有效期
        long ttl = jedis.ttl("job");
        System.out.println("ttl = " + ttl);
    }
}
```







## 连接池

Jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，推荐使用Jedis连接池代替Jedis的直连方式

```java
package com.zixieqing.redis.util;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.time.Duration;

/**
 * Jedis连接池
 *
 * @author : ZiXieqing
 */

public class JedisConnectionFactory {
    private static JedisPool jedisPool;

    static {
        // 设置连接池
        JedisPoolConfig PoolConfig = new JedisPoolConfig();
        PoolConfig.setMaxTotal(30);
        PoolConfig.setMaxIdle(30);
        PoolConfig.setMinIdle(0);
        PoolConfig.setMaxWait(Duration.ofSeconds(1));

        /*
            设置链接对象
            JedisPool(GenericObjectPoolConfig<Jedis> poolConfig, String host, int port, int timeout, String password)
         */
        jedisPool = new JedisPool(PoolConfig, "192.168.46.128", 6379, 1000, "072413");
    }

    public static Jedis getJedis() {
        return jedisPool.getResource();
    }
}
```







# Java操作：SpringDataRedis

SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做SpringDataRedis

官网：https://spring.io/projects/spring-data-redis

* 提供了对不同Redis客户端的整合（Lettuce和Jedis）
* 提供了RedisTemplate统一API来操作Redis
* 支持Redis的发布订阅模型
* 支持Redis哨兵和Redis集群
* 支持基于JDK.JSON、字符串、Spring对象的数据序列化及反序列化
* 支持基于Redis的JDKCollection实现

SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中：

![1652976773295](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725143957710-1372787584.png)







## 入门SpringDataRedis

创建SpringBoot项目

1. pom.xml配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.9.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.zixieqing</groupId>
    <artifactId>02-spring-data-redis</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>02-spring-data-redis</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>8</java.version>
    </properties>

    <dependencies>
        <!--redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--common-pool-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
        <!--Jackson依赖-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

2. YAML文件配置

```yaml
spring:
  redis:
    host: 192.168.46.128
    port: 6379
    password: "072413"
    jedis:
      pool:
        max-active: 100 # 最大连接数
        max-idle: 100 # 最大空闲数
        min-idle: 0 # 最小空闲数
        max-wait: 5 # 最大链接等待时间 单位：ms
```

3. 测试：其他如Hash、List、Set、SortedSet的方法和下面String差不多

```java
package com.zixieqing.springdataredis;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.RedisTemplate;

import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.Objects;
import java.util.concurrent.TimeUnit;

@SpringBootTest(classes = App.class)
class ApplicationTests {

    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    /**
     * SpringDataRedis操作redis：String类型  其他类型都是同理操作
     *
     * String：opsForValue
     * Hash：opsForHash
     * List：opsForList
     * Set：opsForSet
     * SortedSet：opsForZSet
     */
    @Test
    void stringTest() {
        // 添加key-value
        redisTemplate.opsForValue().set("name", "紫邪情");

        // 根据key获取value
        String getName = Objects.requireNonNull(redisTemplate.opsForValue().get("name")).toString();
        System.out.println("getName = " + getName);

        // 添加key-value 并 指定有效期
        redisTemplate.opsForValue().set("job", "Java", 10L, TimeUnit.SECONDS);
        String getJob = Objects.requireNonNull(redisTemplate.opsForValue().get("job")).toString();
        System.out.println("getJob = " + getJob);

        // 就是 setnx 命令，key不存在则添加，存在则不执行
        redisTemplate.opsForValue().setIfAbsent("city", "杭州");
        redisTemplate.opsForValue().setIfAbsent("info", "脸皮厚，欠揍", 10L, TimeUnit.SECONDS);

        ArrayList<String> keys = new ArrayList<>();
        keys.add("name");
        keys.add("job");
        keys.add("city");
        keys.add("info");
        redisTemplate.delete(keys);
    }
}
```









## 数据序列化

RedisTemplate可以接收Object类型作为值写入Redis：

![](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725220147109-2003270191.png)



只不过写入前会把Object序列化为字节形式，默认是采用JDK序列化，得到的结果是这样的：

![image-20230725220208536](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725220209733-549553778.png)



缺点：

- 可读性差
- 内存占用较大





### Jackson序列化

我们可以自定义RedisTemplate的序列化方式，代码如下：

```java
package com.zixieqing.springdataredis.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;

/**
 * redis自定义序列化方式
 *
 * @author : ZiXieqing
 */

@Configuration
public class RedisSerializeConfig {
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory){
        // 创建RedisTemplate对象
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 设置Key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setHashValueSerializer(jsonRedisSerializer);
        // 返回
        return template;
    }
}
```



这里采用了JSON序列化来代替默认的JDK序列化方式。最终结果如图：

![image-20230725221131734](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725221133118-434948430.png)



整体可读性有了很大提升，并且能将Java对象自动的序列化为JSON字符串，并且查询时能自动把JSON反序列化为Java对象

不过，其中记录了序列化时对应的class名称，目的是为了查询时实现自动反序列化。这会带来额外的内存开销。







### StringRedisTemplate

尽管JSON的序列化方式可以满足我们的需求，但依然存在一些问题

![image-20230725221320439](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725221321539-1356626600.png)



为了在反序列化时知道对象的类型，JSON序列化器会将类的class类型写入json结果中，存入Redis，会带来额外的内存开销。

为了减少内存的消耗，我们可以采用手动序列化的方式，换句话说，就是不借助默认的序列化器，而是我们自己来控制序列化的动作，同时，我们只采用String的序列化器，这样，在存储value时，我们就不需要在内存中多存储数据，从而节约我们的内存空间

![1653054744832](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725221500449-360113056.png)



这种用法比较普遍，因此SpringDataRedis就提供了RedisTemplate的子类：StringRedisTemplate，它的key和value的序列化方式默认就是String方式

![image-20230725224017233](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725224019562-1913671903.png)



![image-20230725223754743](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725223757938-372078237.png)





使用示例：

```java
package com.zixieqing.springdataredis;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.zixieqing.springdataredis.entity.User;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.StringRedisTemplate;

import javax.annotation.Resource;
import java.util.ArrayList;
import java.util.Objects;
import java.util.concurrent.TimeUnit;

@Slf4j
@SpringBootTest(classes = App.class)
class ApplicationTests {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    // 是jackson中的
    private final ObjectMapper mapper = new ObjectMapper();

    /**
     * 使用StringRedisTemplate操作Redis 和 序列化与反序列化
     *
     * 操作redis和String类型一样的
     */
    @Test
    void serializeTest() throws JsonProcessingException {
        User user = new User();
        user.setName("zixieqing")
                .setJob("Java");

        // 序列化
        String userStr = mapper.writeValueAsString(user);
        stringRedisTemplate.opsForValue().set("com:zixieqing:springdataredis:user", userStr);

        // 反序列化
        String userStr2 = stringRedisTemplate.opsForValue().get("com:zixieqing:springdataredis:user");
        User user2 = mapper.readValue(userStr2, User.class);

        log.info("反序列化结果：{}", user2);
    }
}
```



![image-20230725225419447](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230725225420748-719163497.png)











# 缓存更新策略

缓存更新是redis为了节约内存而设计出来的一个东西，主要是因为内存数据宝贵，当我们向redis插入太多数据，此时就可能会导致缓存中的数据过多，所以redis会对部分数据进行淘汰

![image-20230729132118455](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230729132118742-1294780096.png)



**内存淘汰：**redis自动进行，当redis内存达到咱们设定的max-memery的时候，会自动触发淘汰机制，淘汰掉一些不重要的数据(可以自己设置策略方式)

**超时剔除：**当我们给redis设置了过期时间ttl之后，redis会将超时的数据进行删除，方便咱们继续使用缓存

**主动更新：**我们可以手动调用方法把缓存删掉，通常用于解决缓存和数据库不一致问题



**业务场景**：先说结论，后面分析这些结论是怎么来的

1. 低一致性需求：使用Redis自带的内存淘汰机制
2. 高一致性需求：主动更新，并以超时剔除作为兜底方案读操作：
   - 读操作：
     - 缓存命中则直接返回
     - 缓存未命中则查询数据库，并写入缓存，设定超时时间写操作
   - 写操作：
     - 先写数据库，然后再删除缓存
     - 要确保数据库与缓存操作的原子性(单体系统写库操作和删除缓存操作放入一个事务；分布式系统使用分布式事务管理这二者)







## 主动更新策略：数据库与缓存不一致问题

由于我们的**缓存的数据源来自于数据库**，而数据库的**数据是会发生变化的**。因此，如果当数据库中**数据发生变化,而缓存却没有同步**，此时就会有**一致性问题存在**，其后果是:

用户使用缓存中的过时数据,就会产生类似多线程数据安全问题,从而影响业务,产品口碑等;怎么解决呢？有如下几种方案

![image-20230729133340137](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230729133340066-2133474689.png)



Cache Aside Pattern 人工编码方式：缓存调用者在更新完数据库后再去更新缓存，也称之为双写方案。这种由我们自己编写，所以可控，因此此种方式胜出

Read/Write Through Pattern : 由系统本身完成，数据库与缓存的问题交由系统本身去处理

Write Behind Caching Pattern ：调用者只操作缓存，其他线程去异步处理数据库，实现最终一致





## Cache Aside 人工编码 解决数据库与缓存不一致

由上一节知道数据库与缓存不一致的解决方案是 Cache Aside 人工编码，但是这个玩意儿需要考虑几个问题：

1. **删除缓存还是更新缓存？**

   - 更新缓存：每次更新数据库都更新缓存，无效写操作较多

   - ==删除缓存：更新数据库时让缓存失效，查询时再更新缓存（胜出）==



2. **如何保证缓存与数据库的操作的同时成功或失败？**
   - 单体系统，将缓存与数据库操作放在一个事务
   - 分布式系统，利用TCC等分布式事务方案



3. **先操作缓存还是先操作数据库？**

   * 先删除缓存，再操作数据库

   * ==先操作数据库，再删除缓存（胜出）==





为什么是先操作数据库，再删除缓存？

操作数据库和操作缓存在“串行”情况下没什么太大区别，问题不大，但是：在“并发”情况下，二者就有区别，就会产生数据库与缓存数据不一致的问题

先看“先删除缓存，再操作数据库”：

![先删缓存再更新数据库](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230729145314344-718743340.gif)



再看“先操作数据库，再删除缓存”：redis操作几乎是微秒级，所以下图线程1会很快完成，然后线程2业务一般都慢一点，所以缓存中能极快地更新成数据库中的最新数据，因此这种方式虽也会发生数据不一致，但几率很小(数据库操作一般不会在微秒级别内完成)

![先操作数据库，再删除缓存](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230729150116221-1775821231.gif)



因此：胜出的是“先操作数据库，再删除缓存”

![1653323595206](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230729150522392-1745241201.png)









# 缓存穿透及解决方式

> **缓存穿透**：指客户端请求的数据在缓存中和数据库中都不存在。这样缓存永远不会生效，这些请求都会打到数据库

场景：如别人模仿id，然后发起大量请求，而这些id对应的数据redis中没有，然后全跑去查库，数据库压力就会增大，导致数据库扛不住而崩掉



**解决方式**：

1. 缓存空对象：就是缓存和数据库中都没有时，直接放个空对象到缓存中，并设置有效期即可

   - 优点：实现简单，维护方便

   - 缺点：
     - 额外的内存消耗
     - 可能造成短期的不一致。一开始redis和数据库都没有，后面新增了数据，而此数据的id可能恰好对上，这样redis中存的这id的数据还是空对象



2. 布隆过滤：采用的是哈希思想来解决这个问题，通过一个庞大的二进制数组，用哈希思想去判断当前这个要查询的数据是否存在，如果布隆过滤器判断存在，则放行，这个请求会去访问redis，哪怕此时redis中的数据过期了，但是数据库中一定存在这个数据，在数据库中查询出来这个数据后，再将其放入到redis中，假设布隆过滤器判断这个数据不存在，则直接返回

   - 优点：内存占用较少，没有多余key

   - 缺点：

     - 实现复杂

     - 存在误判可能。布隆过滤器判断存在，可数据库中不一定真存在，因它采用的是哈希算法，就会产生哈希冲突



3. 增加主键id的复杂度，从而提前做好基础数据校验
4. 做用户权限认证
5. 做热点参数限流



空对象和布隆过滤的架构如下：左为空对象，右为布隆过滤

![1653326156516](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230729185634611-41725608.png)





缓存空对象示例：

```java
@Service
public class ShopServiceImpl extends ServiceImpl<ShopMapper, Shop> implements IShopService {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public Result findShopById(Long id) {
        String cacheKey = CACHE_SHOP_KEY + id;
        // 查 redis
        Map<Object, Object> shopMap = stringRedisTemplate.opsForHash().entries(cacheKey);
        // 有则返回 同时需要看是否命中的是：空对象
        if (!shopMap.isEmpty()) {
            return Result.ok(JSONUtil.toJsonStr(shopMap));
        }

        // 无则查库
        Shop shop = getById(id);
        // 库中无
        if (null == shop) {
            // 向 redis 中放入 空对象，且设置有效期
            Map<String, String> hashMap = new HashMap<>(16);
            hashMap.put("", "");
            stringRedisTemplate.opsForHash().putAll(cacheKey, hashMap);
            // CACHE_NULL_TTL = 2L
            stringRedisTemplate.expire(cacheKey, CACHE_NULL_TTL, TimeUnit.MINUTES);
            
            return Result.fail("商铺不存在");
        }
        // 库中有	BeanUtil 使用的是hutool工具
        // 这步意思：因为Shop实例类中字段类型不是均为String，因此需要将字段值转成String，否则存入Redis时会发生 造型异常
        Map<String, Object> shopMapData = BeanUtil.beanToMap(shop, new HashMap<>(16),
                CopyOptions.create()
                        .ignoreNullValue()
                        .setIgnoreError(false)
                        .setFieldValueEditor((filedKey, filedValue) -> filedValue = filedValue + "")
        );
        // 写入 redis
        stringRedisTemplate.opsForHash().putAll(cacheKey, shopMapData);
        // 设置有效期    CACHE_SHOP_TTL = 30L
        stringRedisTemplate.expire(cacheKey, CACHE_SHOP_TTL, TimeUnit.MINUTES);
        // 返回客户端
        return Result.ok(JSONUtil.toJsonStr(shop));
    }
}
```











# 缓存雪崩及解决方式

> 缓存雪崩：指在同一时段大量的缓存key同时失效 或 Redis服务宕机，导致大量请求到达数据库，带来巨大压力
>
> ![1653327884526](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230729224032065-366638755.png)



**解决方案**：

1. 给不同的Key的TTL添加随机值
2. 利用Redis集群提高服务的可用性
3. 给缓存业务添加降级限流策略
4. 给业务添加多级缓存











# 缓存击穿及解决方式

> 缓存击穿问题也叫热点Key问题，就是一个被**高并发访问**并且**缓存重建业务较复杂**的key突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击
>
> ![1653328022622](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230805174159873-1089270998.png)

常见的解决方案有两种：

1. 互斥锁
2. 逻辑过期

![1653357522914](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230805174429476-662892920.png)





## 互斥锁 - 保一致

> 互斥锁：保一致性，会让线程阻塞，有死锁风险
>
> 本质：利用了String的setnx指令；key不存在则添加，存在则不操作
>
> ![1653328288627](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230805175226036-587427717.png)



示例：下列逻辑该封装则封装即可

```java
public class ShopServiceImpl extends ServiceImpl<ShopMapper, Shop> implements IShopService {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    @Override
    public Result queryShopById(Long id) {
        String cacheKey = CACHE_SHOP_KEY + id;
        // 查 redis
        Map<Object, Object> shopMap = stringRedisTemplate.opsForHash().entries(cacheKey);

        // redis 中有责返回
        if (!shopMap.isEmpty()) {
            Shop shop = BeanUtil.fillBeanWithMap(shopMap, new Shop(), false);
            return Result.ok(shop);
        }

        Shop shop = null;

        try {
            // 无则获取 互斥锁
            Boolean res = stringRedisTemplate
                    .opsForValue()
                    .setIfAbsent(LOCK_SHOP_KEY + id, UUID.randomUUID().toString(true), LOCK_SHOP_TTL, TimeUnit.MINUTES);
            boolean flag = BooleanUtil.isTrue(res);

            // 获取失败则等一会儿再试
            if (!flag) {
                Thread.sleep(20);
                return queryShopById(id);
            }

            // 获取锁成功则查 redis 此时有没有，从而减少缓存重建
            Map<Object, Object> shopMa = stringRedisTemplate.opsForHash().entries(cacheKey);

            // redis 中有责返回
            if (!shopMa.isEmpty()) {
                shop = BeanUtil.fillBeanWithMap(shopMa, new Shop(), false);
                return Result.ok(shop);
            }
            // 有则返回，无则查库
            shop = getById(id);

            // 库中无
            if (null == shop) {
                // 向 redis放入 空值，并设置有效期
                Map<String, String> hashMap = new HashMap<>(16);
                hashMap.put("", "");
                stringRedisTemplate.opsForHash().putAll(cacheKey, hashMap);
                stringRedisTemplate.expire(cacheKey, 2L, TimeUnit.MINUTES);

                return Result.fail("无此数据");
            }

            // 库中有则写入 redis，并设置有效期
            Map<String, Object> sMap = BeanUtil.beanToMap(shop, new HashMap<>(16),
                    CopyOptions.create()
                            .ignoreNullValue()
                            .setIgnoreError(false)
                            .setFieldValueEditor((filedKey, filedValue) -> filedValue = filedValue + "")
            );
            stringRedisTemplate.opsForHash().putAll(cacheKey, sMap);
            stringRedisTemplate.expire(cacheKey, CACHE_SHOP_TTL, TimeUnit.MINUTES);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            // 释放锁
            stringRedisTemplate.delete(LOCK_SHOP_KEY + id);
        }

        //返回客户端
        return Result.ok(shop);
    }
}
```





## 逻辑过期 - 保性能

这玩意儿在互斥锁的基础上再变动一下即可



逻辑过期：不保一致性，性能好，有额外内存消耗，会造成短暂的数据不一致

本质：数据不过期，一直在Redis中，只是程序员自己使用过期字段和当前时间来判定是否过期，过期则获取“互斥锁”，获取锁成功(此时可以再判断一下Redis中的数据是否过期，减少缓存重建)，则开线程重建缓存即可

![image-20230806173104369](https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230806173109028-754493451.png)





示例：

```java
@Data
@Accessors(chain = true)
public class RedisData {
    private LocalDateTime expireTime;
    private Object data;
}
```



```java
@Service
public class ShopServiceImpl extends ServiceImpl<ShopMapper, Shop> implements IShopService {
    @Resource
    private StringRedisTemplate stringRedisTemplate;

    private static final ExecutorService EXECUTORS = Executors.newFixedThreadPool(10);

    @Override
    public Result queryShopById(Long id) {
        // 使用互斥锁解决 缓存击穿
        // return cacheBreakDownWithMutex(id);

        String cacheKey = CACHE_SHOP_KEY + id;

        // 查 redis
        String shopJson = stringRedisTemplate.opsForValue().get(cacheKey);

        // redis 中没有则报错(理论上是一直存在redis中的，逻辑过期而已，所以这一步不用判断都可以)
        if (StrUtil.isBlank(shopJson)) {
            return Result.fail("无此数据");
        }

        // redis 中有，则看是否过期
        RedisData redisData = JSONUtil.toBean(shopJson, RedisData.class);
        LocalDateTime expireTime = redisData.getExpireTime();
        Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);
        // 没过期，直接返回数据
        if (expireTime.isAfter(LocalDateTime.now())) {
            return Result.ok(shop);
        }

        try {
            // 获取互斥锁    LOCK_SHOP_TTL = 10L
            Boolean res = stringRedisTemplate
                    .opsForValue()
                    .setIfAbsent(LOCK_SHOP_KEY + id, UUID.randomUUID().toString(true),
                            LOCK_SHOP_TTL, TimeUnit.SECONDS);
            boolean flag = BooleanUtil.isTrue(res);
            // 获取锁失败则眯一会儿再尝试
            if (!flag) {
                Thread.sleep(20);
                return queryShopById(id);
            }
            // 获取锁成功
            // 再看 redis 中的数据是否过期，减少缓存重建
            shopJson = stringRedisTemplate.opsForValue().get(cacheKey);
            redisData = JSONUtil.toBean(shopJson, RedisData.class);
            expireTime = redisData.getExpireTime();
            shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class);
            // 已过期
            if (expireTime.isBefore(LocalDateTime.now())) {
                EXECUTORS.submit(() -> {
                    // 重建缓存
                    this.buildCache(id, 20L);
                });
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } finally {
            // 释放锁
            stringRedisTemplate.delete(LOCK_SHOP_KEY + id);
        }

        // 返回客户端
        return Result.ok(shop);
    }

    
    /**
     * 重建缓存
     */
    public void buildCache(Long id, Long expireTime) {
        String key = LOCK_SHOP_KEY + id;
        // 重建缓存
        Shop shop = getById(id);
        if (null == shop) {
            // 库中没有则放入 空对象
            stringRedisTemplate.opsForValue().set(key, "", 10L, TimeUnit.SECONDS);
        }

        RedisData redisData = new RedisData();
        redisData.setExpireTime(LocalDateTime.now().plusSeconds(expireTime))
                .setData(shop);

        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(redisData));
    }
}
```



























