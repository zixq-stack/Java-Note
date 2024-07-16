#  tkMapper的概述

> 基于mybatis提供了很多第三方插件，这些插件通常可以帮助我们完成数据操作方法的封装、数据库逆向工程工作的工作

数据库逆向工程：根据数据表生成实体类、映射文件

**常用的第三方插件：**

*   MyBatis-plus
*   tkMapper



##  tkMapper的介绍

tkMapper是一个Mybatis插件，是在Mybatis的基础上提供了许多工具，可以帮助我们完成对应数据库的相关操作，提高开发效率

## tkMapper的作用

*   提供了针对表通用的数据库操作方法
*   逆向工程–根据数据表生成实体类、dao接口、映射文件

# SpringBoot集成tkMapper

1、依赖

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>
```



2、修改启动类`@MapperScan`的包

> 修改tk.mybatis的包为`tk.mybatis.spring.annotation.MapperScan`，不修改会报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/85a7ddcab51442688f0a2d8e19a7bb19.png)  



```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import tk.mybatis.spring.annotation.MapperScan;

@SpringBootApplication
//@MapperScan("com.xiaoqing.springboottkmapperdemo.dao")
@MapperScan("com.xiaoqing.springboottkmapperdemo.dao")
public class SpringbootTkmapperDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootTkmapperDemoApplication.class, args);
    }
}
```

# tkMapper的使用

## 依赖

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <scope>4.13.2</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

## MySQL表

```sql
DROP TABLE IF EXISTS `users`;
CREATE TABLE `users` (
  `user_id` int(11) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(20) NOT NULL,
  `user_pwd` varchar(20) NOT NULL,
  `user_realname` varchar(20) NOT NULL,
  `user_img` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=17 DEFAULT CHARSET=utf8;
```

## 实体类

> tkMapper，在映射的时候，实体类名要与数据库表名保持一致（驼峰命名）
> 
> 如果表名和实体类名不对应会报错，一共有两种解决方式：
> 
> *   修改实体类名，与表名对应
> *   使用`@Table()`注解指定表名

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "users")	 //指定数据库中的表名
public class User {
    private int userId;
    private String userName;
    private String userPwd;
    private String userRealname;
    private String userImg;
}
```

## DAO接口

> tkMapper已经完成了对单表的通用操作的封装，主要封装在Mapper接口和MysqlMapper接口中，因此我们如果要完成对单表的操作，只需要自定义dao接口继承这两个接口即可 
> 
> 在测试类的时候userdao会报错，提示“找不到userdao”，这里可以忽略(因为是可以获取到的)，可以在接口中使用`@Repository`注解就不会提示错误

```java
@Repository
public interface UserDAO extends Mapper<User>, MySqlMapper<User> {
}
```

## 测试

> `@RunWith`声明单元测试的驱动
> 
> `@SpringBootTest`声明启动类的路径

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = SpringbootTkmapperDemoApplication.class)
public class UserDAOTest {
    @Autowired
    private UserDAO userDAO;
    
    @Test
    public void insert(){
        User user = new User();
        user.setUserName("hong");
        user.setUserPwd("123456");
        user.setUserRealname("小红");
        user.setUserImg("12.png");
        int insert = userDAO.insert(user);
        System.out.println(insert);
    }
}
```



# tkMapper的常用方法

## 添加操作

> 添加操作的常用方法有两个：
> 
> *   `insert()` 添加一个对象
>     
> *   `insertUseGeneratedKeys()` 添加一个对象并进行主键回填
>     
>     在进行主键回填的时候，实体类中id必须要用`@Id`指定，并且主键的数据类型改为Integer（通过反射）
>     

### 添加一个对象

```java
@Test
public void insert(){

    User user = new User();
    user.setUserName("ming");
    user.setUserPwd("123456");
    user.setUserRealname("小明");
    user.setUserImg("13.png");

    int i = userDAO.insert(user);
    
    assertEquals(1,i);
}
```



### 主键回填的添加

> 在进行主键回填的时候,，**实体类中id必须要用`@Id`指定，并且主键的数据类型改为Integer**，不然获取不到回填的主键，因为tkMapper底层是通过反射实现的

```java
@Test
public void insertUseGeneratedKeys(){
    
    User user = new User();
    user.setUserName("xiao");
    user.setUserPwd("888888");
    user.setUserRealname("笑笑");
    user.setUserImg("16.png");

    int i = userDAO.insertUseGeneratedKeys(user);
    
    assertEquals(1,i);
    
    System.out.println("回填的主键：" + user.getUserId());
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/283ec0fdad6644489cd3d28bfa35e372.png)





## 修改操作

> 修改操作的常用方法：
> 
> *   `updateByPrimaryKey()` 根据主键修改记录
> *   `updateByExample()` 根据条件删除记录

### 根据主键修改记录

```java
@Test
public void update(){

    User user = new User();
    user.setUserId(17);
    user.setUserName("pipi");
    user.setUserPwd("666666");
    user.setUserRealname("pipi");
    user.setUserImg("18.png");

    int i = userDAO.updateByPrimaryKey(user);
    
    assertEquals(1,i);
}
```



## 删除操作

> 删除操作常用方法：
> 
> *   `deleteByPrimaryKey()` 根据主键删除记录
> *   `deleteByExample()` 根据条件删除记录

### 根据主键删除记录

```java
@Test
public void delete(){

    int i = userDAO.deleteByPrimaryKey(17);
    assertEquals(1,i);
}
```



## 查询操作

> 查询操作常用方法：
> 
> *   `selectAll()` 查询所有记录
> *   `selectByPrimaryKey()` 根据主键查询
> *   `selectByExample()` 根据条件查询
> *   `selectByRowBounds()` 分页查询
> *   `selectCount()` 查询总记录数
> *   `selectByExampleAndRowBounds()` 带条件的分页查询
> *   `selectCountByExample()`查询满足条件的总记录数

### 查询所有记录

```java
@Test
public void select(){
    List<User> users = userDAO.selectAll();
    for (User user : users) {
        System.out.println(user);
    }
}
```



### 根据主键查询记录

```java
@Test
public void select(){

    User user = userDAO.selectByPrimaryKey(18);
    System.out.println(user);
}
```





### 根据条件查询

1、查询用户名为xiao和hong的记录

```java
@Test
public void select(){

    // 根据条件查询
    // 创建一个EXample对象用于封装User的查询条件
    Example example = new Example(User.class);
    Example.Criteria criteria = example.createCriteria();

    // 封装条件：userName为xiao或者hong
    criteria.andEqualTo("userName","xiao");
    criteria.orEqualTo("userName","hong");

    List<User> users = userDAO.selectByExample(example);
    for (User user:users) {
    	System.out.println(user);
    }
}
```



2、查询用户名带“x”的记录

```java
@Test
public void select(){
    // 根据条件查询
    // 创建一个EXample对象用于封装User的查询条件
    Example example = new Example(User.class);
    Example.Criteria criteria = example.createCriteria();
    // 查询用户名带“x”的记录
    criteria.andLike("userName","%x%");

    List<User> users = userDAO.selectByExample(example);
    for (User user:
         users) {
        System.out.println(user);
    }
}
```





### 分页查询

查询第二页，每页五条记录,总记录数

```java
@Test
public void select(){
    // 分页查询
    int pageNum = 2;
    int pageSize = 5;
    int start = (pageNum - 1) * pageSize;

    RowBounds rowBounds = new RowBounds(start,pageSize);
    List<User> users = userDAO.selectByRowBounds(new User(), rowBounds);
    for (User user:
         users) {
        System.out.println(user);
    }
    // 查询总记录数
    int count = userDAO.selectCount(new User());
    System.out.println(count);
}
```





### 带条件的分页查询

查询密码为123456的第二页，每页五条记录，满足条件的总记录数

```java
@Test
public void select2(){
    // 条件
    Example example = new Example(User.class);
    Example.Criteria criteria = example.createCriteria();
    criteria.andEqualTo("userPwd","123456");

    // 分页查询
    int pageNum = 2;	// 第几页
    int pageSize = 5;	// 每页的条数
    int start = (pageNum - 1) * pageSize;
    RowBounds rowBounds = new RowBounds(start,pageSize);

    List<User> users = userDAO.selectByExampleAndRowBounds(example, rowBounds);
    for (User user:
            users) {
        System.out.println(user);
    }
    // 查询满足条件的总记录数
    int count = userDAO.selectCountByExample(example);
    System.out.println(count);
}
```





## tkMapper的关联查询

> 关联查询有两种策略：
> 
> *   进行多次单表查询
> *   自定义查询



### 多次单表查询

查询用户同时查询订单

> ①在User实体类private List orderList属性
> 
> ②根据用户名查询出用户记录
> 
> ③获取到用户记录的userId
> 
> ④再根据userId去订单表查询符合条件订单记录
> 
> ⑤最后将订单列表设置到用户orderList属性

```java
@Test
public void select3(){
    // 查询用户的同时查询订单

    // 根据用户名查询用户
    Example example = new Example(User.class);
    Example.Criteria criteria = example.createCriteria();
    criteria.andEqualTo("userName","zhangsan");

    List<User> users = userDAO.selectByExample(example);
    User user = users.get(0);

    // 根据用户id查询订单
    Example example1 = new Example(Order.class);
    Example.Criteria criteria1 = example1.createCriteria();
    criteria1.andEqualTo("userId",user.getUserId());
    List<Order> orders = orderDAO.selectByExample(example1);

    // 将顶顶设置到user中
    user.setOrderList(orders);
    System.out.println(user);
}
```





### 自定义查询

> 自定义查询是UserMapper.xml文件查询



# 基于tkMapper的逆向工程

> 逆向工程，就是根据创建好的数据表，生成实体类、dao、映射文件

## 依赖

> 此依赖是一个mybatis的maven插件，在build标签里添加

```xml
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.5</version>

    <!--插件需要的依赖-->
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper</artifactId>
            <version>4.1.5</version>
        </dependency>
    </dependencies>
</plugin>
```



## 逆向工程配置

> 在com/xiaoqing/general目录下创建GeneralDao接口
> 
> **注意：‘GeneralDao’,不能在dao包里**

```java
import tk.mybatis.mapper.common.Mapper;
import tk.mybatis.mapper.common.MySqlMapper;

public interface GeneralDao<T> extends Mapper<T>, MySqlMapper<T> {
}
```

> 在resources目录下创建`generatorConfig.xml`，在里面定义了需要生成的表，以及生成文件的路径等信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!-- 引入数据库连接配置 -->
    <!--    <properties resource="jdbc.properties"/>-->

    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="beginningDelimiter" value="`"/>
        <property name="endingDelimiter" value="`"/>

        <!-- 配置 GeneralDAO -->
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="com.xiaoqing.general.GeneralDao"/>
        </plugin>

        <!-- 配置数据库连接 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/fmmail"
                        userId="root" password="123456">
        </jdbcConnection>

        <!-- 配置实体类存放路径 -->
        <javaModelGenerator targetPackage="com.xiaoqing.springboottkmapperdemo.beans" targetProject="src/main/java"/>

        <!-- 配置 XML 存放路径 -->
        <sqlMapGenerator targetPackage="/" targetProject="src/main/resources/mappers"/>

        <!-- 配置 DAO 存放路径 -->
        <javaClientGenerator targetPackage="com.xiaoqing.springboottkmapperdemo.dao" targetProject="src/main/java" type="XMLMAPPER"/>

        <!-- 配置需要指定生成的数据库和表，% 代表所有表 -->
        <table tableName="%"></table>
    </context>
</generatorConfiguration>
```



## 将配置文件设置到maven插件中

![在这里插入图片描述](https://img-blog.csdnimg.cn/5acfee00d49f4dc29a7c20539a9a3c60.png)

```xml
<configuration>
    <configurationFile>${basedir}/src/main/resources/generatorConfig.xml</configurationFile>
</configuration>
```



## 执行逆向生成

双击mybatis:generator:generate

![在这里插入图片描述](https://img-blog.csdnimg.cn/dfc0edd5b138414696066a51c2acabea.png)

出现BUILD SUCCESS则逆向成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/06f503d222fe45aca5e2465087b47fa6.png)



