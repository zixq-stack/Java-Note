# 简单认识通用mapper

## 了解mapper

**作用：**就是为了帮助我们自动的生成sql语句

通用mapper是MyBatis的一个插件，是pageHelper的同一个作者进行开发的

**通用mapper官网地址：**https://gitee.com/free/Mapper

**文档地址：** https://mapper.mybatis.io/



# 玩通用mapper

## 准备工作

1、建测试表

```sql
CREATE TABLE `user` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `user_name` varchar(20) DEFAULT NULL COMMENT '用户名',
  `user_sex` varchar(2) DEFAULT NULL COMMENT '用户性别',
  `user_salary` decimal(5,2) DEFAULT NULL COMMENT '用户薪资',
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='用户表';


insert  into `user`(`user_id`,`user_name`,`user_sex`,`user_salary`) values 
(1,'紫邪情','女',100.00),
(2,'紫玲','女',50.00),
(3,'张三','男',999.99);
```

2、创建Spring项目 并 导入依赖

```xml
<!--  spring整合mybatis的依赖  -->
<!--  1、spring需要的依赖  -->
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aspects</artifactId>
        <version>5.1.9.RELEASE</version>
    </dependency>

    <!-- 2、mybatis的依赖 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.6</version>
    </dependency>

    <!-- spring整合mybatis的第三方依赖 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis-spring</artifactId>
        <version>2.0.2</version>
    </dependency>

    <!--    1、数据库驱动    -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.47</version>
    </dependency>

    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.6</version>
    </dependency>

    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.2.9.RELEASE</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.22</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>

    <!-- 通用mapper的依赖 -->
    <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper</artifactId>
        <version>3.4.2</version>
    </dependency>

</dependencies>
```

3、编写SSM框架整合的xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"

       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
				https://www.springframework.org/schema/beans/spring-beans.xsd
				http://www.springframework.org/schema/aop
				https://www.springframework.org/schema/aop/spring-aop.xsd
				http://www.springframework.org/schema/context
				https://www.springframework.org/schema/context/spring-context.xsd
				http://www.springframework.org/schema/tx
				https://www.springframework.org/schema/tx/spring-tx.xsd
			">

    <!--  1、获取数据源 —— 使用druid -->
    <context:property-placeholder location="classpath:db.properties"/>
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${druid.driverClassName}"/>
        <property name="url" value="${druid.url}"/>
        <property name="username" value="${druid.username}"/>
        <property name="password" value="${druid.password}"/>
    </bean>

    <!--  2、获取SQLSessionFactory工厂-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!-- 把mybatis集成进来 -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/> <!-- 集成mybatis-config.xml -->
    </bean>

    <!--  3、配置事务管理  -->
    <!--  声明事务托管  -->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--  说明哪些方法要进行事务托管 —— 即：通知类 -->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="*" rollback-for="Exception"/>
        </tx:attributes>
    </tx:advice>
    <!--  编写切面  -->
    <aop:config>
        <!-- 切点 -->
        <aop:pointcut id="pointCut" expression="execution( * cn.xiegongzi.mapper.*.*(..) )"/>
        <!-- 组装切面 ——— 切点和通知类组装 -->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pointCut"/>
    </aop:config>

    <!-- 4、扫描mapper层，整合通用mapper的唯一一个注意点
        原始SSM整合写法是：org.mybatis.spring.mapper.MapperScannerConfigurer
        现在用通用mapper替换：tk.mybatis.spring.mapper.MapperScannerConfigurer
        为什么通用mapper可以替换掉mybatis？
            因为：通用mapper的MapperScannerConfigurer在底层继承了mybatis的MapperScannerConfigurer，可以点源码
    -->
    <bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="cn.zixieqing.mapper"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    </bean>
    
    <!--扫描service层-->
    <context:component-scan base-package="cn.zixieqing.service"/>


</beans>
```

> **注意点：在扫描mapper层时，使用通用mapper覆盖mybatis，写法不太一样**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220413130519939-1442795967.png" alt="image" style="zoom:50%;" />



**建对应的实体类**

> **注意点**：数据类型用包装类，因为包装类可以判断null值，这个涉及到通用mapper的原理，数据类型用包装类在MaBatis中就已经知道的事情

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
public class UserEntity implements Serializable {

    private Integer userId;
    private String userName;
    private String userSex;
    private Double userSalary;
}
```

## 通用mapper基础API

先看一下通用mapper大概有哪些API

```java
// 这里接口直接继承通用mapper接口即可
//    注意：泛型中的信息就是实体类
public interface UserMapper extends Mapper<User> {		// 看源码，点mapper即可进入
    
}
```

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220412223350257-113535741.png" alt="image" style="zoom:50%;" />



看看BaseMapper

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220412223536994-2109219894.png" alt="image" style="zoom:50%;" />



其他的都是差不多的套路，归纳起来其实就是增删查改的封装，然后做了不同的细分，需要继续查看的，那就往后挨个去点击



### 和select相关

#### selectOne方法 和 @Table及@Column注解

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220412224058727-1585991322.png" alt="image" style="zoom:50%;" />



编写测试类 并启动

```java
import cn.zixieqing.entity.*;
import cn.zixieqing.mapper.*;
import org.junit.*;
import org.springframework.context.*;
import org.springframework.context.support.*;


public class MyTest {

    @Test
    public void selectOneTest() {

        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        UserMapper userMapper = context.getBean("userMapper", UserMapper.class);

        UserEntity userEntity = new UserEntity();
        userEntity.setUserId(1)
                .setUserName("紫邪情")
                .setUserSex("女")
                .setUserSalary(new Double(100));
        System.out.println(userMapper.selectOne(userEntity));
    }
}
```

**出现报错：Table 'mapper_study.user_entity' doesn't exist**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416171653584-1421493233.png" alt="image" style="zoom:50%;" />



**原因就是编写的实体类名叫做UserEntity，而数据库的表名叫做user，解决方式就是在实体类中加入@Table注释，注意此注解是import javax.persistence.*;包下的**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416171948106-2075104792.png" alt="image" style="zoom:50%;" />





另外的selectxxx方法直接一点就出来了，基本上都是见名知意，就算不知道的源码中也有解释，通用mapper就是国内人写的



**至于`@Column`注解就是见名知意，用来处理实体类的字段和数据库中的字段不一致的问题**

- 默认规则： 
  - 实体类字段：驼峰式命名 
  - 数据库表字段：使用`_`区分各个单词

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416173826269-1269992706.png" alt="image" style="zoom:50%;" />



#### 观察日志总结selectOne方法

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416172326778-492922714.png" alt="image" style="zoom:50%;" />



**`selectOne()`是将封装的实体类作为了WHERE子句的条件**

- 这里是使用了非空的值作为的WHERE子句

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416172613501-1483535000.png" alt="image" style="zoom:50%;" />

- **在条件表达式中使用“=”进行比较**



> **注意点：**要求必须返回<span style="color:blue">一个</span>实体类结果，如果有多个，则会抛出异常



#### xxxByPrimaryKey方法 和 @Id注解

测试

```java
    @Test
    public void selectByPrimaryKey() {
        System.out.println(userMapper.selectByPrimaryKey(3));
    }
```

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416174558536-1265840354.png" alt="image" style="zoom:50%;" />



**结果：发现将实体类的所有字段属性都作为WHERE子句的条件了**

**解决办法：给实体类中对应的数据库表的主键字段加上@Id注解**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416174814745-2112149310.png" alt="image" style="zoom:50%;" />



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416174849756-1397767306.png" alt="image" style="zoom:50%;" />





#### xxxByPrimaryKey方法 和 @Id注解总结

> **@Id注解**

为了将实体类字段属性和对应的数据库表主键做匹配

原因：通用mapper在执行xxxByPrimaryKey方法时会出现两种情况：

1）、没有加`@Id`注解时，通用mapper会将实体类的所有属性作为联合主键来匹配数据库表的主键，故而会出现将实体类中的字段属性全部作为WHERE子句后面的条件字段

```sql
SELECT user_id,user_name,user_sex,user_salary 
FROM user 
WHERE user_id = ? AND user_name = ? AND user_sex = ? AND user_salary = ?
```

2）、使用`@Id`主键将实体类中的字段属性和数据库表中的主键做明确匹配

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416180054825-1736554194.png" alt="image" style="zoom:67%;" />



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416180209148-1254437192.png" alt="image" style="zoom:50%;" />



> **xxxByPrimaryKey方法**

需要使用`@Id`注解来让实体类中的字段属性和数据库表中的主键做明确匹配，否则：通用mapper默认将实体类的所有字段属性作为联合主键来进行匹配





#### select方法

> **传什么，就用什么来拼接WHERE子句的条件**

测试

```java
    @Test
    public void select() {
        UserEntity userEntity = new UserEntity();
        userEntity.setUserName("紫邪情");
        System.out.println(userMapper.select(userEntity));
    }
```

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416181854269-143823458.png" alt="image" style="zoom:50%;" />



### 和insert相关

#### insert方法

测试

```java
    @Test
    public void insertTest() {

        UserEntity userEntity = new UserEntity();
        userEntity.setUserId(4)
                .setUserName("不知火舞");

        // 这个API会将null也拼接到SQL语句中
        System.out.println(userMapper.insert(userEntity));
    }
```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416233315357-1713515164.png" alt="image" style="zoom:50%;" />

#### insertSelective方法

```java
    @Test
    public void insertSelective() {
        UserEntity userEntity = new UserEntity();
        userEntity.setUserName("百里守约")
                .setSex("老六");

        // 这个API会将非null的字段拼接说起来语句中
        userMapper.insertSelective(userEntity);
    }
```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220416234236481-433364130.png" alt="image" style="zoom:50%;" />



#### @GeneratedValue注解

**这个注解是为了让通用mapper再执行insert语句之后，把数据库中自动生成的主键值回填到实体类对象中**

官网文档介绍：https://gitee.com/free/Mapper/wikis/2.orm/2.3-generatedvalue

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220417142812499-435107595.png" alt="image" style="zoom:67%;" />





### 和update相关

#### updateByPrimaryKeySelective方法

这个其实看一眼就知道了，**也就是：根据主键把不为null值的字段修改掉，即：set后面的字段就是实体类中不为null的字段**

```java
    @Test
    public void updateByPrimaryKeySelectiveTest() {

        System.out.println( userMapper.updateByPrimaryKeySelective( new UserEntity().setUserId(1).setUserName("小紫") ) );
    }

```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220417132901654-1598246757.png" alt="image" style="zoom:50%;" />





### 和delete相关

#### delete方法

**切记：使用时，记得把实体类值传进去，否则：若是null的实体类，则：SQL语句就没有WHERE条件了，继而：变成全表的逻辑删除了**

**原理：还是一样的，使用非null的字段作为WHERE子句条件**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220417134105619-1607172028.png" alt="image" style="zoom:50%;" />



#### deleteByPrimaryKey方法

见名知意，直接通过主键来删

```java
    @Test
    public void deleteByPrimaryKeyTest() {
        userMapper.deleteByPrimaryKey(2);
    }
```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220417134325706-1118632996.png" alt="image" style="zoom:50%;" />



### @Transient注解

一般情况下，实体中的字段和数据库表中的字段是一一对应的，但是也有很多情况我们会在实体中增加一些额外的属性，这种情况下，就需要使用 `@Transient` 注解来告诉通用 Mapper 这不是表中的字段

```java
@Transient
private String otherThings; //非数据库表中字段
```





## QBC查询

> QBC全称：query by criteria 也就是通过规则( criteria )来查询



### Criteria对象

```java
public class ExampleTest {

    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserMapper userMapper = context.getBean("userMapper", UserMapper.class);

    // 1、创建Example对象
    Example example = new Example(UserEntity.class);


    @Test
    public void exampleTest() {

        // 2、使用Example创建Criteria对象
        Example.Criteria criteria = example.createCriteria();

        // 3、添加规则 下面这些都是Criteria能够调的API，还有其他的，需要时再说
        /*       
            andGreaterThan 即：>               andGreaterThanOrEqualTo 即：>=
            andLessThan 即：<                  andLessThanOrEqualTo 即：<=
            andIn 即：就是SQL的in               andNotIn  就是SQL的not in
            andBetween  即：SQL的between       andNotBetween 即SQL中的not between
            andLike 即sql中的like              andNotLike  即SQL的not like
            要看这些比较符，直接点andGreaterThan看源码，里面都有各种介绍 
        */
        criteria.andGreaterThan("userSalary", 50).andLessThan("userSalary", 200);

        // 4、调用Example封装的API 不止selectByExample这一个，还有其他的CRUD
        List<UserEntity> userEntities = userMapper.selectByExample(example);

        for (UserEntity userEntity : userEntities) {
            System.out.println(userEntity);
        }
    }
}
```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220417180023776-1917131290.png" alt="image" style="zoom:50%;" />



### Example对象能调用的API

#### CreateCriteria()

```java
    @Test
    public void createCriteriaTest() {

        // createCriteria创建规则 - 有一个别扭的注意点
        Example.Criteria criteria01 = example.createCriteria();
        Example.Criteria criteria02 = example.createCriteria();
        // 使用Example调用or()时，传入的这个Criteria对象的参数有点别扭
        // 添加规则1
        criteria01.andGreaterThan("userId", 1).andLessThan("userId", 6);
        // 添加规则2
        criteria02.andGreaterThan("userSalary", 100).andLessThan("userSalary", 500);
        /*
        * 拼接的SQL语句：
        *   SELECT user_id,user_name,user_sex,user_salary
        *   FROM user
        *   WHERE ( user_id > ? and user_id < ? ) or ( user_salary > ? and user_salary < ? )
        * */
        // 别扭之处就在这里：是将规则2 criteria02 使用or拼接起来，理论上应该是criteria01.or(criteria02)
        // 但是：却是使用example来调的or( Criteria criteria )，所以感觉example就相当于是criteria01一样，有点别扭
        example.or(criteria02);
        List<UserEntity> userEntities = userMapper.selectByExample(example);
        userEntities.forEach(System.out::println);
    }
```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220417183011014-1160192055.png" alt="image" style="zoom:50%;" />





#### orderBy(String property) 排序

```java
    @Test
    public void orderByTest() {

        // userSalary 排序字段      desc 排序方式 - 降序desc 升序asc
        example.orderBy("userSalary").desc();

        userMapper.selectByExample(example).forEach(System.out::println);
    }
```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220417183529838-1900043799.png" alt="image" style="zoom:50%;" />



#### setDistinct(boolean isDistinct) 去重

```java
    @Test
    public void setDistinctTest() {
        example.setDistinct(true);
        userMapper.selectByExample(example).forEach(System.out::println);
    }
```



#### selectProperties(String... properties) 设置select后的字段

```java
    // 设置拼接SQL的select后面的字段
    @Test
    public void selectPropertiesTest() {
        // 拼接的SQL语句： SELECT user_id , user_name FROM user
        // 默认是* 即：实体类的所有字段都拼接上去了
        example.selectProperties("userId","userName");

        userMapper.selectByExample(example).forEach(System.out::println);
    }
```



#### excludeProperties(String... properties) 设置select后不包含的字段

```java
    // 设置select后不包含的字段
    @Test
    public void excludePropertiesTest() {

        // SQL语句 SELECT user_name , user_sex FROM user 
        example.excludeProperties("userId", "userSalary");
        
        userMapper.selectByExample(example).forEach(System.out::println);
    }
```

其他的API直接用example点就出来了，都是见名知意的



## 通用mapper逆向工程

### 依赖

```xml

<!-- 注意：别少了这个依赖 -->
<dependencies>
    <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper</artifactId>
        <version>4.0.0-beta3</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.6</version>
            <configuration>
                <!-- generatorConfig.xml逆向工程配置文件所在地 根据需要自行修改 -->
                <configurationFile>
                    ${basedir}/generatorConfig.xml
                </configurationFile>
                <overwrite>true</overwrite>
                <verbose>true</verbose>
            </configuration>
            <!-- 通用mapper逆向工程需要的两个依赖 -->
            <dependencies>
                <dependency>
                    <groupId>mysql</groupId>
                    <artifactId>mysql-connector-java</artifactId>
                    <version>5.1.47</version>
                </dependency>
                <dependency>
                    <groupId>tk.mybatis</groupId>
                    <artifactId>mapper</artifactId>
                    <version>4.0.0-beta3</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```



### generatorConfig.xml配置

```xml
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 引入外部数据库配置文件 最好使用引入，否则：下面数据库配置那里奇葩要求很多 -->
    <properties resource="db.properties"/>

    <!-- MySQL基础信息 就是起始和结束分隔符  如：`` -->
    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <!--  通用mapper插件
            	type	是通用mapper插件，这个可以配置在前面引入的那个外部配置文件中，即配置成如下：
                		mapper.plugin=tk.mybatis.mapper.generator.MapperPlugin
                然后在这里使用${mapper.plugin}引入，这种方式方便管理
        -->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <!-- 这是mapper接口层中extend继承的那个类，即：public interface userMapper extends Mapper<User> -->
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <!-- 这是区别大小写 如：user 和 User -->
            <property name="caseSensitive" value="true"/>
        </plugin>

        <!-- 数据库 -->
        <jdbcConnection driverClass="${jdbc.driver}"
                        connectionURL="${jdbc.url}"
                        userId="${jdbc.username}"
                        password="${jdbc.password}">
        </jdbcConnection>

        <!-- 实体类 -->
        <javaModelGenerator targetPackage="cn.zixieqing.entity"
                            targetProject="src/main/java"/>

        <!-- xxxMapper.xml所在位置 -->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src/main/resources"/>

        <!-- mapper接口层 -->
        <javaClientGenerator targetPackage="cn.zixieqing.mapper"
                             targetProject="src/main/java"
                             type="XMLMAPPER"/>

        <!-- 数据库表名 和 实体类生成关系
            	如果感觉每个表都配置麻烦，那么直接改变tableName的值即可，即：tableName="%"
            	但是：此种方式的默认规则是采用 _ 转驼峰命名，如：table_name  ——> TableName
            	可是：有时我们并不需要这样命名，此时就需要使用tableName 和 domainObjectName两个配置项一起来配置
                tableName 数据库表名
                domainObjectName  生成的实体类名
        -->
        <table tableName="user" domainObjectName = "User">
            <!-- 主键生成策略 -->
            <generatedKey column="user_id" sqlStatement="JDBC"/>
        </table>
    </context>
</generatorConfiguration>
```

引入的外部文件db.properties的配置

```properties
# 数据库配置
jdbc.driver=com.mysql.jdbc.Driver
# 注意：建议使用db.properties配置从而在generatorConfig.xml中引入的原因就在这里
# 在这里可以在这个url后面拼接参数，如：useSSL=false
# 若是直接把这些配置写到generatorConfig.xml中，那么后面的参数配置就有几个奇葩的地方
# 			1、参数之间而不是通过&隔开，而是需要使用;分号隔开  如：useSSL=false;useUnicode=true
#			2、false / true等值需要使用``括起来，具体可以尝试，然后看报的ERROR
jdbc.url=jdbc:mysql://localhost:3306/mapper_study?useSSL=false&useUnicode=true&characterEncoding=utf-8
jdbc.username=root
jdbc.password=root

#c3p0
jdbc.maxPoolSize=50
jdbc.minPoolSize=10
jdbc.maxStatements=100
jdbc.testConnection=true

# 通用Mapper配置 若是在generatorConfig.xml的plugin配置中是通过引入的方式来做的，那么就可以在这里配置这两个信息
# 从而方便管理
# mapper.plugin=tk.mybatis.mapper.generator.MapperPlugin
# mapper.Mapper=tk.mybatis.mapper.common.Mapper
```



### 启动

**在 pom.xml 这一级目录**的命令行窗口执行 `mvn mybatis-generator:generate`即可

![image](https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220418225618856-1413625074.png)



**但是：注意生成的实体类中并没有无参和有参构造及`toString()`，可以自行加上**



### 简单测试一下API

依赖

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
</dependency>
```

log4j配置

```properties
# 将等级为DEBUG的日志信息输出到console和file这两个目的地，console和file的定义在下面的代码
log4j.rootLogger=DEBUG,console,file

# 控制台输出的相关设置
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.Target = System.out
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

# 文件输出的相关设置
log4j.appender.file = org.apache.log4j.RollingFileAppender
log4j.appender.file.File=./1og/xieGongZi.log
log4j.appender.file.MaxFileSize=10mb
log4j.appender.file.Threshold=DEBUG
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]‰m%n

# 日志输出级别
log4j.logger.org.mybatis=DEBUG
log4j.logger.java.sq1=DEBUG
log4j.logger.java.sql.Statement=DEBUG
log4j.logger.java.sql.ResultSet=DEBUG
log4j.logger.java.sql.PreparedStatement=DEBUG
```

mybatis-config.xml配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- 自行配置连接池 -->
    <properties resource="db.properties"/>

    <settings>
        <setting name="logImpl" value="LOG4J"/>
    </settings>

    <typeAliases>
        <package name="cn.xiegongzi.entity"/>
    </typeAliases>

    <environments default="dev">
        <environment id="dev">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="cn.zixieqing.mapper"/>
    </mappers>

</configuration>
```

测试

```java
@Test
public void simpleTest() throws IOException {

    InputStream in = Resources.getResourceAsStream("mybatis-config.xml");

    SqlSessionFactory build = new SqlSessionFactoryBuilder().build(in);

    SqlSession sqlSession = build.openSession();
    // 以上和mybatis的流程一样，通用mapper只是获取了sqlSession之后需要多做下面的这一步操作

    MapperHelper mapperHelper = new MapperHelper();
    Configuration configuration = sqlSession.getConfiguration();
    mapperHelper.processConfiguration(configuration);
    // new MapperHelper().processConfiguration(sqlSession.getConfiguration());

    UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

    userMapper.selectAll().forEach(System.out::println);
}
```



## 自定义mapper

**自定义mapper接口的作用：**根据实际需要自行重组mapper接口，即并不是通用mapper中的所有接口和方法都需要



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220419231618082-882876794.png" alt="image" style="zoom:50%;" />





### 玩一下自定义mapper接口

1、自定义自己要的mapper接口

```java
import tk.mybatis.mapper.common.*;


public interface CustomMapper<T> extends BaseMapper<T> {
    // 这个自定义的mapper，想继承前面画的通用mapper中的哪个接口都可以
}
```

2、编写业务mapper

```java
import cn.zixieqing.common.*;
import cn.zixieqing.entity.*;


public interface UserMapper extends CustomMapper<UserEntity> {
}
```



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421153906300-59376692.png" alt="image" style="zoom:50%;" />



> **注意点：别把自定义mapper和业务mapper放到一个包中，会报错**

3、修改applicationContext.xml文件的MapperScannerConfigurer配置

```xml
<!-- 4、扫描mapper层，整合通用mapper的唯一一个注意点
        原始SSM整合写法是：org.mybatis.spring.mapper.MapperScannerConfigurer
        现在用通用mapper替换：tk.mybatis.spring.mapper.MapperScannerConfigurer
        为什么通用mapper可以替换掉mybatis？
            因为：通用mapper的MapperScannerConfigurer在底层继承了mybatis的MapperScannerConfigurer，可以点源码
-->
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="cn.zixieqing.mapper"/>
    <!--<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>-->
    <property name="properties">
        <value>
            <!--自定义接口所在的包路径-->
            mapper=cn.zixieqing.common.CustomMapper
        </value>
    </property>
</bean>
```

4、测试

```java
@Test
public void customMapperTest() {

    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    UserMapper userMapper = context.getBean("userMapper",UserMapper.class);

    userMapper.selectAll().forEach(System.out::println);

}
```

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421154617132-1268805491.png" alt="image" style="zoom:50%;" />



> **补充1、要是不想写applicationContext.xml中MapperScannerConfigurer的那个配置**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421154919820-22316183.png" alt="image" style="zoom:50%;" />



那把内容注释掉，在自定义mapper接口的地方加个注解`@RegisterMapper`就搞定了

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421155140599-1197616807.png" alt="image" style="zoom:50%;" />





> **补充2、如果将自定义mapper接口 和 业务mapper接口放到一个包中了**

![image](https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421155237204-834657686.png)

一运行就会报错

```json
tk.mybatis.mapper.MapperException: java.lang.ClassCastException: sun.reflect.generics.reflectiveObjects.TypeVariableImpl cannot be cast to java.lang.Class
```

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421155415899-1059970156.png" alt="image" style="zoom:50%;" />



原因就是利用反射，获取类对象时失败，即：原因如下

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421155640872-968692757.png" alt="image" style="zoom:50%;" />



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220421155817779-1405692927.png" alt="image" style="zoom:50%;" />







## 了解通用mapper的二级缓存

一二级缓存的概念哪些就跳过了，MyBatis中已经见过了，这里玩通用mapper的配置

测试：自行编写一个业务mapper，然后去实现mapper<T>，从而进行多次执行，会发现SQL执行了多次

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425134328149-1580152425.png" alt="image" style="zoom:50%;" />



注意：需要让`mapper<T>`中的实体类T实现Serializable接口，从而有用序列号，否则：会报错的

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425134756463-541871070.png" alt="image" style="zoom:50%;" />





1、修改mybatis-config.xml文件

```xml
<settings>
    <!--显示开启缓存-->
    <setting name="cacheEnabled" value="true"/>
</settings>
```

2、在业务mapper接口中添加`@CacheNamespace`注解

```java
@CacheNamespace
public interface UserMapper extends Mapper<UserEntity> {
}
```

3、再次测试

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425135220678-1997230277.png" alt="image" style="zoom:50%;" />



## 类型处理器 typeHandler

这里说的类型是简单类型和复杂类型，注意：和Java中说的基本类型和引用类型不是一回事，不是说基本类型就一定是简单类型，这里不用去考虑基本和引用的问题

**简单类型和复杂类型可以参考一对一和一对多这两种**

- **简单类型：**只有一个值
- **复杂类型：**有多个值

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425145334011-2056011746.png" alt="image" style="zoom:50%;" />



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425150530667-258459663.png" alt="image" style="zoom:50%;" />



而上面这种，对于userName来说，是无法进行CRUD的

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425150839366-1771529095.png" alt="image" style="zoom:50%;" />



**这种情况就是复杂类型，而通用mapper默认是没处理的，就有点类似于在上述例子的userName上加了一个@Transient注解，从而忽略了该字段，从而造成的效果就是：去数据库中找对应的字段值时没找到，从数据库中找到数据，然后返还给对象时没有相应的对象可以接受**



<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425151053463-1178427386.png" alt="image" style="zoom:50%;" />







> **解决办法：自定义类型处理器**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425152727977-140859912.png" alt="image" style="zoom:50%;" />



**具体操作流程如下：**

1、**创建一个类型处理器的类，然后实现TypeHandler<T>接口**，其中：T就是要处理的那个类型，如：上述例子的NameEntity

2、**实现里面的四个方法**

```java
    @Override
    public void setParameter(PreparedStatement preparedStatement, int i, NameEntity nameEntity, JdbcType jdbcType) throws SQLException {
        
    }

    @Override
    public NameEntity getResult(ResultSet resultSet, String s) throws SQLException {
        return null;
    }

    @Override
    public NameEntity getResult(ResultSet resultSet, int i) throws SQLException {
        return null;
    }

    @Override
    public NameEntity getResult(CallableStatement callableStatement, int i) throws SQLException {
        return null;
    }
```

实例逻辑编写如下

```java
public class NameHandler implements TypeHandler<NameEntity> {

    /**
     * 这个方法就是：对象NameEntity ——> 数据库的流程规则，可以将其理解为序列化流程 但是完全不一样啊
     * 只是说：像序列化一样把数据转成一个样
     * @param ps
     * @param i
     * @param nameEntity
     * @param jdbcType
     * @throws SQLException
     */
    @Override
    public void setParameter(PreparedStatement ps, int i, NameEntity nameEntity, JdbcType jdbcType) throws SQLException {

        // 1、验证NameEntity
        if ( null == nameEntity) {
            return;
        }

        // 2、取出nameEntity中的值
        String firstName = nameEntity.getFirstName();
        String lastName = nameEntity.getLastName();

        // 3、把取出的值 拼接成 一个字符串
        // 自定义规则：使用 - 进行隔开
        StringBuilder builder = new StringBuilder();
        builder.append(firstName)
                .append("-")
                .append(lastName);

        // 4、拼接SQL的参数
        ps.setString(i,builder.toString() );
    }

    /**
     * 这下面三个是重载，是为了解决：数据库 ——> 对象NameEntity的流程，类似于反序列化，把另一个东西转成正常需要的样子
     * @param resultSet
     * @param columnName
     * @return
     * @throws SQLException
     */
    @Override
    public NameEntity getResult(ResultSet resultSet, String columnName ) throws SQLException {
        // 1、从结果集ResultSet根据字段名取出字段值
        String columnValue = resultSet.getString(columnName);

        // 2、验证columnValue
        if ( null == columnValue || columnValue.length() == 0 || !columnValue.contains("-") ) {
            return null;
        }
        
        // 3、根据“-”对columnValue进行拆分
        String[] column = columnValue.split("-");
        
        // 4、把拆分之后的值 给到 对象的对应值
        return new NameEntity().setFirstName( column[0] ).setLastName( column[1] );
    }

    @Override
    public NameEntity getResult(ResultSet resultSet, int i) throws SQLException {

        // 1、从结果集ResultSet根据字段名取出字段值
        String columnValue = resultSet.getString(i);

        // 2、验证columnValue
        if ( null == columnValue || columnValue.length() == 0 || !columnValue.contains("-") ) {
            return null;
        }

        // 3、根据“-”对columnValue进行拆分
        String[] column = columnValue.split("-");

        // 4、把拆分之后的值 给到 对象的对应值
        return new NameEntity().setFirstName( column[0] ).setLastName( column[1] );
    }

    @Override
    public NameEntity getResult(CallableStatement cs, int i) throws SQLException {


        // 1、从CallableStatement 根据 索引取出字段值
        String columnValue = cs.getString(i);

        // 2、验证columnValue
        if ( null == columnValue || columnValue.length() == 0 || !columnValue.contains("-") ) {
            return null;
        }

        // 3、根据“-”对columnValue进行拆分
        String[] column = columnValue.split("-");

        // 4、把拆分之后的值 给到 对象的对应值
        return new NameEntity().setFirstName( column[0] ).setLastName( column[1] );
    }
}
```



3、**注册类型处理器**

**第一种( 字段级别 )：使用`@ColumnType(typeHandler = xxxx.class)`注解**

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425160506676-2042913166.png" alt="image" style="zoom:50%;" />

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425160846632-1978440946.png" alt="image" style="zoom:50%;" />



注意啊：我这里是改数据库了的，这是做的查询嘛，要是数据库中的数据没符合规范，那还是查不到

<img src="https://img2022.cnblogs.com/blog/2421736/202204/2421736-20220425160955463-1305722015.png" alt="image" style="zoom:50%;" />



**第二种( 全局配置 ）：在mybatis-config.xml中进行配置**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <settings>
        <setting name="logImpl" value="LOG4J"/>
        <!--显示开启缓存-->
        <setting name="cacheEnabled" value="true"/>
    </settings>

    <typeAliases>
        <package name="cn.xiegongzi.entity"/>
    </typeAliases>

    <typeHandlers>
        <!-- 
            handler 处理器位置
            javaType 要处理的是哪个对象
        -->
        <typeHandler handler="cn.zixieqing.handler.NameHandler" 
                     javaType="cn.zixieqing.entity.NameEntity"/>
    </typeHandlers>

</configuration>
```

给用到该类型的地方添加`@Column`注解

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Accessors(chain = true)
@Table(name = "user")
@ToString
public class UserEntity implements Serializable {

    private static final long serialVersionUID = -5580827379143778431L;

    private Integer userId;

    /**
     * @Transient
     *
     * @ColumnType(typeHandler = NameHandler.class)
     */
    @Column
    private NameEntity userName;

    private String userSex;

    private Double userSalary;
}
```





