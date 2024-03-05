# 创建Spring Boot项目
## 官网创建（了解）

这里面的创建方式不做过多说明，只需要在 [官网](https://start.spring.io/) 里面创建好了，然后下载解压，就可以了，我这里直接使用编辑器创建



## IDEA编辑器创建

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302174956348-803751172.png)



> **提示：**
>
> 有时会遇到使用上面的spring官网进行Spring Boot项目创建出现“链接不上“(没用魔法上网的原因)，此时将上面的 start.spring.io custom自定义为阿里云的 https://start.aliyun.com 即可。



后面就是选择相应依赖，根据需要自行选择即可，然后就会自动拉取选择的依赖。









# 小彩蛋：banner

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175005756-1580797122.png)


上面这玩意儿，我不想看到它。推荐一个网址：https://www.bootschool.net/ascii-art

 在项目的resources资源目录下，新建一个banner.txt文件粘贴上述网址中复制的内容即可。






# 了解yml语法

这玩意儿的语法就像如下图的类与属性的关系一样，层层递进的。

> **注意：**
>
> 1. 使用yml语法时，每句的结尾别有空格，容易出问题。
> 2. IDEA中采用tab缩进没问题，但是：在其他地方，如：linux中，使用yml语法时，别用tab缩进，也容易导致程序启动不起来。





## 读取YAML自定义配置内容

> **提示：**
>
> 自定义配置（读取YAML文件中内容）时可以用到这种小知识点，当然还可以使用`@value`注解，看场景来使用即可。



准备工作：导入依赖
```xml
<!--这个jar包就是为了实体类中使用@ConfigurationProperties(prefix = "xxxx")这个注解而不报红-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>

<!--  也可选择不导入此依赖，而是在启动类中加入 @EnableConfigurationProperties(value = {上面那个注解所在类名.class})
	  这个注解是 org.springframework.boot.context.properties.EnableConfigurationProperties; 中的
-->
```

使用`@ConfigurationProperties`注解实现给实体类属性赋值

![image](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175008415-619976598.png)





![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302180327605-1614749436.png)








# jsr303检验

jsr303这是数据检验的规范，基于这个的实现方式有好几个，自行百度一下，然后注解含义都是和下面列出来的差不多。



1. 依赖

```xml
<!--JSR303校验的依赖 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```



2. 使用jsr303检验

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175057891-1829882460.png)



可以搭配的注解如下：

```java
空检查
	@Null					验证对象是否为null
	@NotNull				验证对象是否不为null, 无法查检长度为0的字符串
	@NotBlank				检查约束字符串是不是Null还有被Trim的长度是否大于0,只对字符串,且会去掉前后空格.
	@NotEmpty				检查约束元素是否为NULL或者是EMPTY.


Booelan检查
	@AssertTrue				验证 Boolean 对象是否为 true
	@AssertFalse			验证 Boolean 对象是否为 false


长度检查
	@Size(min=, max=)		验证对象（Array,Collection,Map,String）长度是否在给定的范围之内
	@Length(min=, max=)		Validates that the annotated string is between min and max included.


日期检查
	@Past					验证 Date 和 Calendar 对象是否在当前时间之前，验证成立的话被注释的元素一定是一个过去的日期
	@Future					验证 Date 和 Calendar 对象是否在当前时间之后 ，验证成立的话被注释的元素一定是一个将来的日期
	@Pattern				验证 String 对象是否符合正则表达式的规则，被注释的元素符合制定的正则表达式，
							regexp:正则表达式
							flags: 指定 Pattern.Flag 的数组，表示正则表达式的相关选项。


数值检查
	建议使用在Stirng,Integer类型，不建议使用在int类型上，因为表单值为“”时无法转换为int，
	但可以转换为Stirng为”“,Integer为null

	@Min					验证 Number 和 String 对象是否大等于指定的值
	@Max					验证 Number 和 String 对象是否小等于指定的值
	@DecimalMax				被标注的值必须不大于约束中指定的最大值.
							这个约束的参数是一个通过BigDecimal定义的最大值的字符串表示.小数存在精度

	@DecimalMin				被标注的值必须不小于约束中指定的最小值.
							这个约束的参数是一个通过BigDecimal定义的最小值的字符串表示.小数存在精度

	@Digits					验证 Number 和 String 的构成是否合法
	@Digits(integer=,fraction=)
							验证字符串是否是符合指定格式的数字，interger指定整数精度，fraction指定小数精度

	@Range(min=, max=)		被指定的元素必须在合适的范围内
	@Range(min=10000,max=50000,message=”range.bean.wage”)
	@Valid					递归的对关联对象进行校验, 如果关联对象是个集合或者数组,那么对其中的元素进行递归校验,
							如果是一个map,则对其中的值部分进行校验.(是否进行递归验证)

	@CreditCardNumber		信用卡验证
	@Email					验证是否是邮件地址，如果为null,不进行验证，算通过验证。
	@ScriptAssert(lang= ,script=, alias=)
	@URL(protocol=,host=, port=,regexp=, flags=)
```





# yml多环境配置

1. 不推荐使用

![image](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175008717-1872484781.png)



配置内容如下：

```yaml
# 端口号	即启动之后不再是8080
server:
	port: 8081

# 想切换（使用）那套环境配置
spring:
	profiles:
		active: dev
	# 给当前程序起个名字
    application:
		name: application-name

# 通过 --- 把多套环境隔开
---

server:
	port: 8082

# 给配置环境起一个名字	测试环境 test	开发环境 dev	线上环境 prod
spring:
	profiles: dev

---

server:
	port: 8083
```



2. 推荐使用：采用多个yml文件。

- `application.yml` 公用配置，且定义使用那套环境配置（`spring.profiles.active`）。
- `application-test.yml` 就是测试环境的配置。
- `appilication-dev.yml` 就是开发环境的配置。
- `appilication-prod.yml` 就是生产环境配置。







# 设置默认首页

这是Spring Boot + thmeleaf响应式编程的技术，现在前后端分离，这种东西其实没什么鸟用。

## 页面在static目录中时

直接在controller中编写跳转地址即可。

![image-20240302182025901](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302181941937-87962381.png)



## 页面在templates模板引擎中时

![image-20240302182054258](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302182011037-1057293264.png)




1. 这种需要导入相应的启动器

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

2. 编写controller

![image-20240302182127163](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302182043360-1821138867.png)





# 简单认识thymeleaf

> 这是 Spring Boot + thymeleaf 响应式编程的技术，现在前后端分离，这种东西其实没什么鸟用。
>
> 官网学习地址：https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html



## 什么是thymeleaf？

一张图看明白：

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175000836-622436900.png)


解读：
- 前端交给我们的页面，是html页面。如果是我们以前开发，我们需要把他们转成jsp页面，jsp好处就是当我们查出一些数据转发到JSP页面以后，我们可以用jsp轻松实现数据的显示，及交互等。

- jsp支持非常强大的功能，包括能写Java代码，但是，Spring Boot是以jar的方式，不是war，第二，我们用的还是嵌入式的Tomcat，所以，**Spring Boot现在默认是不支持jsp的**。

- 那不支持jsp，如果我们直接用纯静态页面的方式，那给我们开发会带来非常大的麻烦，那怎么办？



**Spring Boot推荐使用模板引擎：**

- 模板引擎，jsp就是一个模板引擎，还有用的比较多的FreeMaker、Velocity，再多的模板引擎，他们的思想都是一样的，Spring Boot推荐使用thymeleaf。

  - 模板引擎的作用就是我们来写一个页面模板，比如有些值，是动态的，我们写一些表达式。而这些值从哪来？就是我们在后台封装一些数据。然后把这个模板和这个数据交给模板引擎，模板引擎按照我们封装的数据把这表达式解析出来、填充到我们指定的位置，然后把这个数据最终生成一个我们想要的内容从而最后显示出来，这就是模板引擎。

  - 不管是jsp还是其他模板引擎，都是这个思想。只不过，不同模板引擎之间，他们可能语法有点不一样。其他的就不介绍了，这里主要介绍一下Spring Boot给我们推荐的Thymeleaf模板引擎，这模板引擎，是一个高级语言的模板引擎，他的这个语法更简单。而且功能更强大。



## thymeleaf的取数据方式

官网中有说明

![image-20240302182330364](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302182246166-1123492903.png)

提取出来看一下，从而在Spring Boot中演示一下。
```txt
简单的表达：
	变量表达式： ${...}
	选择变量表达式： *{...}
	消息表达： #{...}
	链接 URL 表达式： @{...}
	片段表达式： ~{...}
```



## Spring Boot中使用thymeleaf

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```




**怎么使用thymeleaf?**

这个问题换言之就是：html文件应该放到什么目录下。

前面我们已经导入了依赖，那么按照Spring Boot的原理（自行百度），底层会帮我们导入相应的东西，并做了相应的配置，那么就去看一下源码，从而知道我们应该把文件放在什么地方。

> **提示：**
>
> Spring Boot中和配置相关的都在 `xxxxxProperties `文件中、

因此：去看一下thymeleaf对应的thymeleafProperties文件。

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175003967-1766791787.png)


那就来建一个。

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175017912-171050566.png)


编写controller，让其跳到templates目录的页面中去。

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175007241-1580966564.png)


测试

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175001585-1795430014.png)


成功跳过去了






## 延伸：传输数据
### 开胃菜

参照官网来。这里只演示 变量表达式： `${...}` ，其他的都是一样的原理。

```txt
简单的表达：
	变量表达式： ${...}
	选择变量表达式： *{...}
	消息表达： #{...}
	链接 URL 表达式： @{...}
	片段表达式： ~{...}
```



1. 编写后台，存入数据

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175031794-1367110087.png)


2. 在前台获取数据

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175008757-945985051.png)


表空间约束链接如下，这个在thymeleaf官网中有。

```xml
xmlns:th="http://www.thymeleaf.org"
```


3. 测试

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175001636-1477911296.png)





### 开整

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175002954-598686399.png)



1. 后台


![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175050839-17972432.png)

2. 前台

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175053697-1419555787.png)


3. 测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175002258-2037318457.png)






# 静态资源处理方式

在前面玩了thymeleaf，在resources中还有一个目录是static

![image-20240302182754048](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302182710254-1217446704.png)



那么就来研究一下静态资源：Spring Boot底层是怎么去装配静态资源的？都在`WebMvcAutoConfiguration`有答案，去看一下。

![image-20240302182851921](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302182808250-179658090.png)

通过上述的源码发现两个东西：`webjars` 和 `getStaticLocations()`。




## webjars的方式处理静态资源

webjars的官网：https://www.webjars.org/all

进去之后里面就是各种各样的jar包。




**使用jQuery做演示：** 导入jQuery的依赖

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.4.1</version>
</dependency>
```

![image-20240302182947726](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302182903863-1306783975.png)



导入之后：发现多了这么一个jar包，现在我们去直接访问一下。

![image-20240302183012722](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302182928777-632012800.png)



是可以直接访问的，为什么？

![image-20240302183047142](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302183003482-90318942.png)



`getStaticLocations()`点进去看一下。发现是如下这么一个方法。

```java
public String[] getStaticLocations() {
	return this.staticLocations;
}
```

查看`staticLocations`。

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175016217-144752930.png)


![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175004741-904470235.png)



```java
// 这个就不多说明，指的就是再建一个META-INF文件夹，里面再建一个resources目录
// 参照Java基础中的web项目目录
"classpath:/META-INF/resources/",

"classpath:/resources/",

"classpath:/static/",

"classpath:/public/"
```

发现有四种方式可以放静态资源，那就来测试一下。




### resources / static / public的优先级

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175046891-1781366529.png)



测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175002734-932834382.png)


发现resources下的优先级最高。



删掉resources中的资源文件，继续测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175002700-457202988.png)


发现static目录其次。



### 总结：resources、static、public优先级

优先级为：resources > static > public



**资源放置建议：**
1. public：放置公有的资源，如：img、js、css....
2. static：放置静态访问的页面，如：登录、注册....
3. resources：应该说是templates，放置动态资源，如：用户管理.....






# 整合JDBC、Druid、Druid实现日志监控
## 整合JDBC、Druid

1. 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>


<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

2. 编写`application.yml`

```yml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis_spring?useUnicode=true&characterEncoding=utf-8
    username: root
# 注意：在yml中，这种自己写的内容最好用字符串写法，以后玩Redis也是一样，不然有时出现坑，
# 即：密码无效 / 这里面填入的值没有解析出来，不匹配
    password: "072413"
```

3. 测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175058056-1534675927.png)




## 整合Druid

1. 依赖

```xml
<!--要玩druid的话，需要导入下面这个依赖 -->
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid-spring-boot-starter</artifactId>
	<version>1.1.10</version>
</dependency>
```


2. 修改yml文件

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175012476-1748499888.png)


3. 测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175055269-1131268208.png)





## Druid实现日志监控

> **重要提示：**
> 需要web启动器支持。

```xml
<!--
	玩druid实现监控日志，需要web启动器支持，
	因为：druid的statViewServlet本质是继承了servlet
	因此：需要web的依赖支持 / servlet支持
-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

1. 编写配置

```java
import com.alibaba.druid.support.http.StatViewServlet;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;

/**
 * 这个类是为了延伸druid的强大功能，监控后台
 * 注意：这个需要spring的web启动器支持，即：这个监控后台的本质StatViewServlet就是servlet，所以需要servlet支持
*/

@Configuration
public class DruidConfig {

    @Bean
    public ServletRegistrationBean StatViewServlet() {

        ServletRegistrationBean bean = new ServletRegistrationBean<>(new StatViewServlet(), "/druid/*");

        HashMap<String, String> initParameters = new HashMap<>();

        /* 
        * 下面这些参数可以在 com.alibaba.druid.support.http.StatViewServlet的父类
        * com.alibaba.druid.support.http.ResourceServlet 中找到
        * */
        
        // 登录日志监控的用户名	这些数据可以配置在YAML中读取，也可以从数据查到
        initParameters.put("loginUsername", "zixieqing");
        // 登录密码
        initParameters.put("loginPassword", "072413");
        // 允许谁可以访问日志监控	根据情况自行配置
        initParameters.put("allow", "`localhost`");

        bean.setInitParameters(initParameters);
        return bean;
    }
}
```

2. 测试

输入用户名、密码即可进入。

![image-20240302183733938](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302183650402-953709846.png)








# 整合Mybatis

## xml版

1. 导入依赖

```xml
<!--
	mybatis-spring-boot-starter是第三方（mybatis）jar包，不是Spring的
	Spring自己的生态是：spring-boot-stater-xxxx
-->
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.2</version>
</dependency>

<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid-spring-boot-starter</artifactId>
	<version>1.1.10</version>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.47</version>
</dependency>

<dependency>
	<groupId>org.projectlombok</groupId>
	<artifactId>lombok</artifactId>
</dependency>
```


2. 编写实体

```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class User implements Serializable {

    private Integer id;
    private String username;
    private String password;
}
```


3. 编写dao / mapper层

```java
import cn.xiegongzi.entity.User;
import org.apache.ibatis.annotations.Mapper;

import java.util.List;

/*
*   @Mapper 这个注解是mybati-spring提供的 就是自动装配（交给Spring托管）
*   还可以用：
*       @Repository   是spring本身提供的
* 
*   以及：在启动类（main）中使用 @mapperScan("xxx.xxx.mapper") 扫包	这样就不用在每个mapper中加@Mapper注解了
* */

@Mapper
public interface IUserMapper {
    List<User> findALLUser();
}
```



4. 编写xml的sql语句

![image-20240302184059173](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302184015499-1211712076.png)


> **注意点：**
> dao层/mapper和xml的同包同名问题。



5. 编写yml

```yml
# 编写连接池
spring:
  datasource:
  	# MySQL驱动，注意5.7 和 8.x的区别（com.mysql.cj.jdbc.Driver）
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis_spring?useUnicode=true&characterEncoding=utf-8
    username: root
    password: "072413"
    # 使用DruidDataSource
    type: com.alibaba.druid.pool.DruidDataSource

mybatis:
# mybatis配置了额外的信息，则集成进这个配置中来	即在resources下还有一个mybatis-config.xml
	config-location: classpath:/mybatis-config.xml
# 把实现类xml文件添加进来
  	mapper-locations: classpath:mapper/*.xml
# 给实体类配置别名
  	type-aliases-package: cn.xiegongzi.entity
  	configuration:
#   开启驼峰命名映射
    	map-underscore-to-camel-case: true

# 给日志设置级别
logging:
	level:
		cn.xiegongzi.mapper: debug
```

关于日志去这里：[Java日志框架体系整理](https://www.cnblogs.com/xiegongzi/p/16293103.html)



6. 测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175054649-362225526.png)




## 注解版

> 直接在dao层 / mapper的接口方法头上用`@insert("sql语句")` 、 `@delete("sql语句")` 、 `@update("sql语句") `、` @select("sql语句")`注解。




![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175041348-397426329.png)







# 整合PageHelper分页插件

> PageHelper分页的本质：
>
> 将数据库中满足条件的所有数据查出来，然后通过配置的 `startPage(page, limit)` 条件将数据截取出来，最后装入PageInfo对象中。



1. 依赖

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper-spring-boot-starter</artifactId>
	<version>1.2.5</version>
</dependency>
```

2. 测试

```java
@Test
void pageHelperTest(){
    // 设置分页		第1页 显示3条数据
    PageHelper.startPage(1, 3);
    
    // 查库	注意点：PageHelper.startPage() 和 Pagein 之间“只能”有一条SQL操作语句
    // 若有多条SQL操作语句，那抱歉，只有第一条有效
    List<User> users = userMapper.findAllUser();
    
    // 封装PageInfo对象
    PageInfo<User> pageInfo = new PageInfo<>(users);
    
    // .................根据需要进行操作：直接返回等等.....
}
```






# 集成Swagger

理论知识滤过，自行百度百科swagger是什么。

**swagger的常见注解和解读网址：** https://blog.csdn.net/loli_kong/article/details/108103746



**常识**

- [OpenAPI](https://www.openapis.org/)

是一个组织（OpenAPI Initiative），他们指定了一个如何描述HTTP API的规范（OpenAPI Specification）。既然是规范，那么谁想实现都可以，只要符合规范即可。

- [Swagger](https://swagger.io/)

它是[SmartBear](https://smartbear.com/) 这个公司的一个开源项目，里面提供了一系列工具，包括著名的 `swagger-ui`。`swagger`是早于OpenApi的，某一天`swagger`将自己的API设计贡献给了OpenApi，然后由其标准化了。





1. 导入依赖

```xml
<!--swagger所需要的依赖-->
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.8.0</version>
</dependency>
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.8.0</version>
</dependency>
<!--这个依赖是为了渲染swagger文档页面的（ 为了好看一点罢了 ） ，swagger真正的依赖是上面两个-->
<dependency>
	<groupId>com.github.xiaoymin</groupId>
	<artifactId>swagger-bootstrap-ui</artifactId>
	<version>1.8.5</version>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>

<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.2</version>
</dependency>

<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>1.2.75</version>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```


2. 编写swagger配置文件

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.service.Contact;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration      // 表明当前类是一个配置类，并把当前类丢到Spring容器中去
@EnableSwagger2     // 开启swagger2功能
public class SwaggerConfig {

  @Bean
  public Docket createRestApi() {
    // http://ip地址:端口/项目名/swagger-ui.html#/
    ApiInfo apiInfo = new ApiInfoBuilder()
                    // 网站标题    即：生成的文档标题
					.title("悠忽有限公司")
					 // 网站描述     即：对生成文档的描述
					.description("这是一个很nice的接口文档")
					 // 版本
					.version("1.0")
					 // 联系人
					.contact(new Contact("紫邪情","https://www.cnblogs.com/xiegongzi/", "110"))
					 // 协议
					.license("tcp")
					// 协议url 即：进入到swagger文档页面的地址
					.licenseUrl("http://localhost:8080/")
					.build();

    // swagger版本
	return new Docket(DocumentationType.SWAGGER_2)
					// 请求映射路径	就是：controller中有一个接口，然后前台访问的那个接口路径
        			// 这个可以在生成的文档中进行调试时看到
					.pathMapping("/")
					 // 根据pathMapping去进行查询（做相应的操作）
					.select()
					// 扫描包   即：哪些地方可以根据我们的注解配置帮我们生成文档
					.apis(RequestHandlerSelectors.basePackage("cn.xiegongzi"))
					.paths(PathSelectors.any())
					.build()
					.apiInfo(apiInfo);
  }
}
```


3. 编写yml文件

```yml
spring:
  datasource:
    # 注意这里MySQL驱动用的是8.x的
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis_spring?useUnicode=true&characterEncoding=utf-8
    username: root
    password: "072413"
```

4. 编写实体类

```java
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;


/**
 * <p>@ApiModel(description = "描述")	表明这个实体类就是需要的数据名和类型
 * 
 * 后台接收前端的参数是一个对象时使用（controller写的是@RequestBody OrderPaidDTO orderPaid）
 * 即：后端接收参数封装成了一个xxxxDTO（PO、BO、Entity、DTO、DAO含义和关系是什么，自行百度）
 * 
 * 这个东西可以先不加，在做增加、修改时可以用这个测试一下，从而去swagger中看效果
 * </p>
*/
@Data
@AllArgsConstructor
@NoArgsConstructor
@ApiModel(description = "用户信息")
public class User implements Serializable {

    // 数据属性配置，
    // 这里面可以跟几个属性，常见的是value、required、dataType、hidden，在前面注解解读连接中有解释
    @ApiModelProperty
    private Integer id;

    @ApiModelProperty
    private String username;

    @ApiModelProperty
    private String phone;
}
```


5. 编写mapper


```java
import cn.xiegongzi.entity.User;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;

import java.util.List;

@Mapper
public interface IUserMapper {

    @Select("select * from user")
    List<User> findAllUser();
}
```


6. 编写service接口和实现类

![image-20240302191600548](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302191517443-1050007233.png)


7. 编写controller

```java
package cn.xiegongzi.controller;

import cn.xiegongzi.service.IUserService;
import com.alibaba.fastjson.JSON;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


/*
 * @Api
 *   表示当前类可以被生成一个swagger文档
 *   可以跟参数tags，参数表示：这整个接口的名字（前端是接口，后端是controller控制层）
 */
@Api(tags = "用户管理接口集")
@RestController
public class UserController {

    @Autowired
    private IUserService userService;


    /*
    * @ApiImplicitParam 这个注解是对请求参数做限制用的
    *     如：请求时要求前台传递一个id，那么：在这个注解里面：就可以声明这个参数的类型
    *        对象类型中要求属性限制，可以使用 @ApiModelProperty 也可以使用 符合jsr303规范的数据检验方式
    * */
	// value这个接口的名字; notes 对这个接口的描述
    @ApiOperation(value = "获取全部用户接口" , notes = "获取全部的用户")
    // 遵循restful风格  要是使用 @RequestMapping 的话，会生成多个接口swagger文档，即：对应post、get....
    @GetMapping("/swaggger/doc")
    public String findAllUser() {
        
        return JSON.toJSONString(userService.findAllUser());
    }
}
```


8. 测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175009102-1640396763.png)

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175004947-1571802343.png)







# 集成Swagger增强版：Knife4j

[官方文档](https://doc.xiaominfo.com/docs/quick-start)

> **注意：**
>
> Spring Boot版本不一样使用上有区别，本文中采用的是Spring Boot 3.x，至于Spring Boot 2.x的使用和注意事项请看上述官方文档。



对于Spring Boot 3：

- Spring Boot 3 只支持OpenAPI3规范。
- Knife4j提供的starter已经引用 [springdoc-openapi](https://springdoc.org/) 的jar，需注意避免jar包冲突。
- **JDK版本必须 >= 17**。
- Demo请参考：[knife4j-spring-boot3-demo](https://gitee.com/xiaoym/swagger-bootstrap-ui-demo/tree/master/knife4j-spring-boot3-demo)



## 安装

1. Maven

```xml
<dependency>
	<groupId>com.github.xiaoymin</groupId>
	<artifactId>knife4j-openapi3-jakarta-spring-boot-starter</artifactId>
	<!-- 可以用 4.4.0 -->
	<version>4.1.0</version>
</dependency>
```

2. Gradle

```gradle
implementation("com.github.xiaoymin:knife4j-openapi3-jakarta-spring-boot-starter:4.4.0")
```

> 引入之后，其余的配置可完全参考springdoc-openapi的项目说明，Knife4j只提供了增强部分，如果要启用Knife4j的增强功能，可以在配置文件中进行开启.


## 配置

编写配置类

```java
import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Contact;
import io.swagger.v3.oas.models.info.Info;
import org.springdoc.core.models.GroupedOpenApi;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * <p>
 * Swagger增强版Knife4j配置类
 * </p>
 *
 * <p>@author : ZiXieqing</p>
 */

@Configuration
public class Knife4jConfig {
    @Bean
    public GroupedOpenApi adminApi() {      // 创建了一个api接口的分组
        return GroupedOpenApi.builder()
                .group("admin-api")         // 分组名称
                .pathsToMatch("/admin/**")  // 接口请求路径规则
                .build();
    }

    /**
     * 自定义接口信息
     */
    @Bean
    public OpenAPI customOpenAPI() {

        return new OpenAPI()
                .info(new Info()
                        .title("悠忽有限公司皮包骨 API接口文档")
                        .version("1.0")
                        .description("皮包骨API接口文档")
                        .contact(new Contact().name("zixieqing"))); // 设定作者
    }
}
```


### 增强功能配置
上面配置类配好之后，可以通过YAML配置文件进行Knife4j增强功能配置。


> **重要提示**
>
> Knife4j 自4.0版本，配置属性元数据全部改由`spring-boot-configuration-processor`自动生成，因此之前版本的驼峰命名全部修改成了横杠(`-`)代替。

```yaml
knife4j:
# 是否开启Knife4j增强模式	默认false		要使用Knife4j提供的增强则此值必须为true
  enable: true
# 是否开启一个默认的跨域配置,该功能配合自定义Host使用
  cors: false
# 是否开启生产环境保护策略		说明去看：https://doc.xiaominfo.com/docs/features/accessControl
  production: false
# 对Knife4j提供的资源提供BasicHttp校验,保护文档
  basic:
# 开启 或 关闭BasicHttp功能	即是否配置用户名、密码登录文档
    enable: false
# basic用户名
    username: test
# basic密码
    password: 12313
# 自定义文档集合，该属性是数组
  documents:
    -
      group: 2.X版本	# 所属分组
      name: 接口签名	# 类似于接口中的tag,对于自定义文档的分组
      locations: classpath:sign/*	# markdown文件路径,可以是一个文件夹(classpath:markdowns/*)		也可以是单个文件(classpath:md/sign.md)
# 前端Ui的个性化配置属性
  setting:
# Ui默认显示语言,目前主要有两种:中文(zh-CN)、英文(en-US)
    language: zh-CN
# 是否显示界面中SwaggerModel功能
    enable-swagger-models: true
# 是否显示界面中"文档管理"功能
    enable-document-manage: true
# 重命名SwaggerModel名称,默认Swagger Models
    swagger-model-name: 实体类列表
# 是否开启界面中对某接口的版本控制,如果开启，后端变化后Ui界面会存在小蓝点
    enable-version: false
# 是否在每个Debug调试栏后显示刷新变量按钮,默认不显示
    enable-reload-cache-parameter: false
# 调试Tab是否显示AfterScript功能,默认开启
    enable-after-script: true
# 具体接口的过滤类型
    enable-filter-multipart-api-method-type: POST
# 针对RequestMapping的接口请求类型,在不指定参数类型的情况下,
# 如果不过滤,默认会显示7个类型的接口地址参数,如果开启此配置,默认展示一个Post类型的接口地址
    enable-filter-multipart-apis: false
# 是否开启请求参数缓存
    enable-request-cache: true
# 是否启用Host
    enable-host: false
# HOST地址
    enable-host-text: 192.168.0.193:8000
# 是否开启自定义主页内容
    enable-home-custom: true
# 主页内容Markdown文件路径
    home-custom-path: classpath:markdown/home.md
# 是否禁用Ui界面中的搜索框
    enable-search: false
# 是否显示Footer
    enable-footer: false
# 是否开启自定义Footer
    enable-footer-custom: true
# 自定义Footer内容
    footer-custom-content: Apache License 2.0 | Copyright  2019-[浙江八一菜刀股份有限公司](https://gitee.com/xiaoym/knife4j)
# 是否开启动态参数调试功能
    enable-dynamic-parameter: false
# 启用调试
    enable-debug: true
# 显示OpenAPI规范
    enable-open-api: false
# 显示服务分组
    enable-group: true
```



## 注解示例

### 实体类注解：@Schema

```java
import io.swagger.v3.oas.annotations.media.Schema;
import lombok.Data;

@Data
@Schema(description="注册对象")
public class UserRegisterDto {

    @Schema(description = "用户名")
    private String username;

    @Schema(description = "密码")
    private String password;

    @Schema(description = "昵称")
    private String nickName;

    @Schema(description = "手机验证码")
    private String code ;
}
```



### Controller层注解：@Tag 和 @Operation 及  @Parameters

```java
@Tag(name = "body参数接口")
@RestController
@RequestMapping("body")
public class BodyController {

   @Operation(summary = "普通body请求")
   @PostMapping("/body")
   public ResponseEntity<FileResp> body(@RequestBody FileResp fileResp){
       return ResponseEntity.ok(fileResp);
   }

   @Operation(summary = "普通body请求 + Param + Header + Path")
   @Parameters({
           @Parameter(name = "id",description = "文件id",in = ParameterIn.PATH),
           @Parameter(name = "token",description = "请求token",required = true,in = ParameterIn.HEADER),
           @Parameter(name = "name",description = "文件名称",required = true,in=ParameterIn.QUERY)
   })
   @PostMapping("/bodyParamHeaderPath/{id}")
   public ResponseEntity<FileResp> bodyParamHeaderPath(@PathVariable("id") String id,
   														@RequestHeader("token") String token,
														@RequestParam("name")String name,
														@RequestBody FileResp fileResp){

       fileResp.setName(fileResp.getName()+",receiveName:"+name+",token:"+token+",pathID:"+id);
       return ResponseEntity.ok(fileResp);
   }
}
```

最后，访问Knife4j的文档地址：http://ip:port/doc.html 即可查看文档。







# 集成SpringDoc

上面都整了Swagger、Knife4j了，那就顺便把SpringDoc也一起弄了吧。



- [Springfox](https://github.com/springfox/springfox)

是Spring生态的一个开源库，是Swagger与OpenApi规范的具体实现。我们使用它就可以在spring中生成API文档。以前基本上是行业标准，目前最新版本可以支持 Swagger2, Swagger3 以及 OpenAPI3 三种格式。但是其从 2020年7月14号就不再更新了，不支持springboot3，所以业界都在不断的转向我们今天要谈论的另一个库Springdoc，新项目就不要用了

- [Springdoc](https://springdoc.org/index.html#getting-started)

算是后起之秀，带着继任Springfox的使命而来。其支持OpenApi规范，支持Springboot3，新项目就可以直接用这个。



> **提示：**
>
> SpringDoc支持 Java Bean Validation API 的注解，如：`@NotNull`。





1. 依赖：其实引入依赖访问 http://server:port/context-path/swagger-ui.html  就可以使用，如http://localhost:8080/swagger-ui.html 。只是使用的都是默认值而已。

```xml
<dependency>
   <groupId>org.springdoc</groupId>
   <artifactId>springdoc-openapi-ui</artifactId>
   <version>1.7.0</version>
</dependency>
```



2. 配置文档信息和分组情况

```java
@Configuration
public class SpringDocConfig {
    @Bean
    public OpenAPI myOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("悠忽有限公司皮包骨API")
                        .description("程序员的大本营")
                        .version("v1.0.0")
                        .license(new License()
                                .name("许可协议")
                                .url("http://localhost:8080"))
                        .contact(new Contact()
                                .name("紫邪情")
                                .email("zixq8@qq.com")))
                .externalDocs(new ExternalDocumentation()	// 外部文档
                        .description("紫邪情博客")
                        .url("https://www.cnblogs.com/xiegongzi"));
    }
    
	@Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
                .group("api")	// 分组：controller以/api为前缀的这一组
                .pathsToMatch("/api/**")
                .build();
    }

    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
                .group("admin")		// 分组：controller以 /admin 为前缀的这一组
                .pathsToMatch("/admin/**")
                .build();
    }
 }
```



## 常用注解说明

| 注解            | 含义                                                         | 示例                                                         |
| :-------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| @Tag            | 用在controller类上，描述此controller的信息                   | `@Tag(name = "用户接口")`                                    |
| @Operation      | 用在controller的方法里，描述此api的信息。<br />`@Parameter`以及`@ApiResponse`都可以配置在它里面。 | `@Operation(summary = "添加用户")`                           |
| @Parameter      | 用在controller方法里的参数上，描述参数信息                   | `(@Parameter(description = "用户id")`<br />`@PathVariable Integer id)` |
| @Parameters     | 用在controller方法里的参数上。@Parameter的批量参数添加       | ![image-20240302212102631](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302212018269-1972795544.png) |
| @ApiResponse    | 用在controller方法的返回值上                                 | ![image-20240302212031377](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302211947739-84466403.png) |
| @ApiResponses   | 用在controller方法的返回值上                                 | 见@ApiResponse示例                                           |
| @Schema         | 用于Entity / VO / DTO / BO，以及其属性上                     | `@Schema(description = "搜索条件实体类")`<br />或属性<br />`@Schema(description = "结束时间")`<br />支持Bean校验注解<br />`@NotNull`    ` @Min(18)`     `@Max(35)`等 |
| @Hidden         | 用在各种地方，用于隐藏其api                                  |                                                              |
| @ResponseStatus | 统一异常处理。<br /><br />统一异常处理中，每个方法会捕捉对应的异常，只要我们使用`@ResponseStatus`来标记这些方法，springdoc就会自动生成相应的文档 | ` @ExceptionHandler(value = Exception.class)`<br />`@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)` |





## 认证

针对的是：服务需要认证后才能调用，例如使用了Spring Security，或者自己写了个Filter 来实现认证功能。

这种情况当从`swagger-ui`调用API时会返回401。

希望能够正常调用API：使用`@SecurityScheme` 定义一个安全模式。可以定义全局的，也可以针对某个controller定义类级别的。

1. 启动类添加`@SecurityScheme`注解。

```java
// 定义一个名为api_token的安全模式，并指定其使用HTTP Bearer的方式
@SecurityScheme(name = "api_token", 
                type = SecuritySchemeType.HTTP, 
                scheme ="bearer", 
                in = SecuritySchemeIn.HEADER)
@SpringBootApplication
```

2. 使此安全模式生效。

```java
@Configuration
public class SpringDocConfig {
    @Bean
    public OpenAPI myOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("悠忽有限公司皮包骨API")
                        .description("程序员的大本营")
                        .version("v1.0.0")
                        .license(new License()
                                .name("许可协议")
                                .url("http://localhost:8080"))
                        .contact(new Contact()
                                .name("紫邪情")
                                .email("zixq8@qq.com")))
                .security(List.of(new SecurityRequirement().addList("api_token")));	// 使定义的安全模式生效
        		// 注：api_token是1中定义的name
    }
}
```

3. 声明是否需要认证：使用`@SecurityRequirements()`来设置。

默认情况下按照上面两步设置后，整个应用程序的API就会生效，但是有的API是不需要认证的，例如登录。

```java
@RestController
@RequestMapping(value = "/admin", produces =  "application/json")
public class AuthController {
    ...
	/*	@SecurityRequirements()		不认证		属性是一个String数组，里面列出需要使用的 @SecurityScheme
	 *								不写就说明不需要任何的安全模式
	 * */
    @PostMapping("/login")
    public Result<String> login(@RequestBody LoginRequest request){
        return Result.ok("123");
    }
}
```
















# 集成JPA

1. 数据库信息

![](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175007274-1617914336.png)



2. 导入依赖

```xml
<!--导入jpa需要的依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>


<!--项目需要的依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
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

<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid-spring-boot-starter</artifactId>
	<version>1.1.10</version>
</dependency>
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>1.2.75</version>
</dependency>

<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.3.2</version>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```


3. 编写yml文件

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis_spring?useUnicode=true&characterEncoding=utf-8
    username: root
    password: "072413"

  jpa:
# 这里可以不用hibernate，还可以用hikari（这个在前面整合jdbc时见过，就是当时输出的那句话）
    hibernate:
# 指定为update，每次启动项目检测表结构，有变化的时候会新增字段，表不存在时会新建表
      ddl-auto: update
# 如果指定create，则每次启动项目都会清空数据并删除表，再新建
# 这里面还可以跟：create-drop、create、none
      naming:
        # 指定jpa的自动表生成策略，驼峰自动映射为下划线格式
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl  # 默认就是这个
#        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
# 注掉的这种是：不用驼峰名字，直接把实体类的大写字母变小写就完了

    show-sql: true		# 在控制台显示sql语句	不是真的sql语句，而是相当于：说明		默认是false
# 使用INNODB引擎
    properties.hibernate.dialect: org.hibernate.dialect.MySQL55Dialect
    database-platform: org.hibernate.dialect.MySQL55Dialect
# 使用JPA创建表时，默认使用的存储引擎是MyISAM，通过指定数据库版本，可以使用InnoDB
```



4. 编写实体类

```java
import lombok.Data;
import org.springframework.data.annotation.Id;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import java.io.Serializable;



/** <p>
 * @Entity	表明：当前类和数据库中的这个同类名的数据库表形成ORM映射关系
 * 			要是数据库中没有这个表，那么：根据yml配置的ddl-auto: update 就会自动帮我们生成
 * </p>
 */
@Data
@Entity
public class ZiXieQing implements Serializable {

    @javax.persistence.Id
    @Id  // 表明这个属性是数据库表中的主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)     // 表示：自增  默认是auto，即：和数据库中的auto_increment是一样的
    private int id;

    // 生成数据库中的列字段，里面的参数不止这些，还可以用其他的，对应数据库列字段的那些操作
    @Column(length = 15)
    private String name;

    // public ZiXieQing() {
    // }


    public ZiXieQing(int id, String name) {
        this.id = id;
        this.name = name;
    }
}
```


附：`@Column` 注解中可以支持的属性

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    String name() default "";

    boolean unique() default false;

    boolean nullable() default true;

    boolean insertable() default true;

    boolean updatable() default true;

    String columnDefinition() default "";

    String table() default "";

    int length() default 255;

    int precision() default 0;

    int scale() default 0;
}
```


5. 编写mapper

```java
import cn.xiegongzi.entity.ZiXieQing;
import org.springframework.data.jpa.repository.JpaRepository;


/**
 * <p>
 *	这里别用 @Mapper 这个注解，因为：@mapper是mybatis提供的注解
 * 	JpaRepository相对mybatis来说就是外部的东西。因此：并不能支持@mapper注解
 * </p>
 */
@Repository
public interface ZiXieQingMapper extends JpaRepository<ZiXieQing , Integer> {

    /*
     * JpaRepository这里面有默认的一些方法，即：增删查改...
     * JpaRepository<ZiXieQing , Integer> 本来样子是：JpaRepository<T , ID>
     *     T  表示：自己编写的实体类 类型
     *     ID  表示： 实体类中id字段的类型  注：本示例中，实体类中id是int 因为要弄自增就必须为int，不然和数据库映射时对不上
     * */
}
```

附：`JpaRepository` 中提供的方法。

![image-20240302214422172](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302214338279-1462679227.png)



6. 编写service接口和实现类

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175046099-159198996.png)


7. 编写controller


![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175042371-1913794712.png)


8. 测试

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175053144-1520015985.png)


9. 现在去看一下数据库

![截图](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175006116-286370178.png)







# 集成Mybatis-Plus

mybatis-plus官网地址：https://baomidou.com/guide/




1. 导入依赖

```xml
<!--mybatis-plus需要的依赖-->
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.3.2</version>
</dependency>
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>druid-spring-boot-starter</artifactId>
	<version>1.1.10</version>
</dependency>
```


2. 编写yml

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis_spring?useUnicode=true&characterEncoding=utf-8
    username: root
    password: "072413"
    type: com.alibaba.druid.pool.DruidDataSource

mybatis-plus:
  configuration:
# 	mybatis-plus配置日志
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
# 	开启驼峰映射 即：实体类属性名和数据库字段采用驼峰映射
    map-underscore-to-camel-case: true
# 	自动映射字段
    auto-mapping-behavior: full
# 如果使用了mybatis和mybatis-plus 那么这里就可以把mybatis的实现类xml集成进来
  mapper-locations: classpath:mapper/*.xml
# 但是：最好别这种做，用了mybatis就别用mybatis-plus，二者最好只用其一
```

> **注意点：**
> 别把mybatis和mybatis-plus一起集成到spring中，否则：很容易出问题，虽然：mybatis-plus是mybatis的增强版，既然是增强版，那么就不会抛弃它原有的东西，只会保留原有的东西，然后新增功能，但是：==mybatis和mybatis-plus集成到一起之后很容易造成版本冲突==

因此：对于单个系统模块/单个系统来说，建议二者只选其一集成。

PS：当然事情不是绝对的 我说的是万一，只是操作不当很容易触发错误而已，但是:二者一起集成也是可以的，当出现报错时可以解决掉，不延伸了 ，这不是这里该做的事情。




3. 编写实体类

```java
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName(value = "user")      // 表名注解
public class User implements Serializable {

    @TableId(type = IdType.AUTO)        // 表示主键，这个主键是一个Long类型的值（ 即：snowflake雪花算法 ）
    private Integer id;
    @TableField("username")         // 数据库字段名   就是：当实体类中的字段和数据库字段不一样时可以使用
    private String name;
    private String phone;
}
```


4. 编写mapper


```java
import cn.xiegongzi.entity.User;
import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import org.apache.ibatis.annotations.Mapper;

/**
 * 不想在每个mapper层都写这个注解，那在启动类中加入 @MapperScan("xx.xxxxxxx.mapper") 这个注解也可以实现
 */
@Mapper
public interface IUserMapper extends BaseMapper<User> {

    /*
     * BaseMapper 和 JPA一样，内部有很多方法 ， 即：CRUD.....,还有分页（ 分页就是page()这个方法 ）
     * BaseMapper原来的样子是：BaseMapper<T>  T表示实体类 类型
	 * */
}
```


附：`BaseMapper<T>` 提供的方法如下：

![image-20240302214638479](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302214555065-856783457.png)


5. 测试


![image-20240302214705872](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302214622016-1964904951.png)








# 进程本地缓存技术：Ehcache

进程本地缓存：最典型的情况 项目中整一个全局map。

还有一种比较流行的是`Caffeine`这个东西要更简单、更好一点。

Caffeine官网GitHub地址：https://github.com/ben-manes/caffeine
Caffeine快速入手网址：http://www.mydlq.club/article/56/ 这个网址中的第二种集成方式和下面玩的Ehcache注解的含义一样，而且不需要借助xml文件，而ehcache需要借助xml文件。



## Ehcache介绍

Ehacahe是一个比较成熟的Java缓存框架，最早从hibernate发展而来，是进程中的缓存系统，它提供了用内存、磁盘文件存储，以及分布式存储方式等多种灵活的cache管理方案。



## Ehcache常用注解
### @CacheConfig注解

> 用于标注在类上，可以存放该类中所有缓存的公有属性（如：设置缓存名字）。

```java
@CacheConfig(cacheNames = "users")
public class UserService{

}
```

当然：这个注解其实可以使用`@Cacheable`来代替。




### @Cacheable注解（读数据时）：用得最多

> 应用到读取数据的方法上，如：查找数据的方法，使用了之后可以做到先从本地缓存中读取数据，若是没有，则再调用此注解下的方法去数据库中读取数据，当然：还可以将数据库中读取的数据放到用此注解配置的指定缓存中。

```java
@Cacheable(value = "user", key = "#userId")
User selectUserById( Integer userId );
```



**`@Cacheable` 注解的属性：**

- `value`、`cacheNames`
  - 这两个参数其实是等同的( acheNames为Spring 4新增的，作为value的别名)。
  - **这两个属性的作用：用于指定缓存存储的集合名**。




- `key` **作用：缓存对象存储在Map集合中的key值**。




- `condition`  **作用：缓存对象的条件。** 即：只有满足这里面配置的表达式条件的内容才会被缓存，如：`@Cache( key = "#userId",condition="#userId.length() < 3"` 这个表达式表示只有当userId长度小于3的时候才会被缓存。




- `unless` **作用：另外一个缓存条件。** 它不同于condition参数的地方在于此属性的判断时机（此注解中编写的条件是在函数被`调用之后`才做判断，所以：这个属性可以通过封装的result进行判断）。




- `keyGenerator`
  - **作用：用于指定key生成器。** 若需要绑定一个自定义的key生成器，我们需要去实现`org.springframewart.cahce.intercceptor.KeyGenerator`接口，并使用该参数来绑定。
  - **注意点：该参数与上面的key属性是互斥的**。




- `cacheManager` **作用：指定使用哪个缓存管理器。** 也就是当有多个缓存器时才需要使用。




- `cacheResolver`
  - **作用：指定使用哪个缓存解析器**。
  - **需要通过`org.springframewaork.cache.interceptor.CacheResolver`接口来实现自己的缓存解析器**。





### @CachePut注解 (写数据时)

> 用在写数据的方法上，如：新增 / 修改方法，调用方法时会自动把对应的数据放入缓存，`@CachePut` 的参数和 `@Cacheable` 差不多。

```java
@CachePut(value="user", key = "#userId")
public User save(User user) {
	users.add(user);
	return user;
}
```




### @CacheEvict注解 (删除数据时)

> 用在删除数据的方法上，调用方法时会从缓存中移除相应的数据。

```java
@CacheEvict(value = "user", key = "#userId")
void delete( Integer userId);
```

这个注解除了和 `@Cacheable` 一样的参数之外，还有另外两个参数：
- `allEntries`： 默认为false，当为true时，会移除缓存中该注解该属性所在的方法的所有数据。
- `beforeInvocation`：默认为false，在调用方法之后移除数据，当为true时，会在调用方法之前移除数据。




### @Cacheing组合注解：推荐
```java
// 将userId、username、userAge放到名为user的缓存中存起来
@Caching(
	put = {
		@CachePut(value = "user", key = "#userId"),
		@CachePut(value = "user", key = "#username"),
		@CachePut(value = "user", key = "#userAge"),
	}
)
```







## Spring Boot集成Ehcache
### 配置Ehcache

1. 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
	<groupId>net.sf.ehcache</groupId>
	<artifactId>ehcache</artifactId>
</dependency>
```


2. 在`application.yml`配置文件中加入配置

```yml
cache:
	ehcache:
		# 配置ehcache.xml配置文件所在地
		config: classpath:ehcache.xml
```


3. 在主启动类开启缓存功能

```java
@SpringBootAllication
@EnableCaching
public class Starter {
	public static void main(String[] args) {
		SpringApplication.run(Starter.class);
	}
}
```

4. 编写`ehcache.xml`配置文件。在`resources`目录下新建`ehcache.xml`，并编写如下内容：

```xml
<ehcache name="myCache">
    <!--缓存磁盘保存路径-->
    <diskStore path = "D:/test/cache"/>

    <!--默认的缓存配置
        maxElementsInMemory 缓存最大数目
        eternal 对象是否永久有效 一旦设置了，那么timeout将不再起作用
        timeToIdleSeconds 设置对象在失效前能允许的闲置时间（ 单位：秒 ），默认值是0，即：可闲置时间无穷大
                            仅当eternal=“false"对象不是永久有效时使用
        timeToLiveSeconds 设置对象失效前能允许的存活时间（ 单位：秒 ）
                             最大时间介于创建时间和失效时间之间
        maxElementsOnDisk 磁盘最大缓存个数
        diskExpiryThreadIntervalSeconds 磁盘失效时，线程运行时间间隔，默认是120秒
        memoryStoreEvictionPolicy 当达到设定的maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存
                                    默认策略是LRU( 即：最近最少使用策略 ）
                                    还可以设定的策略：
                                        FIFO    先进先出策略
                                        LFU     最近最少被访问策略
                                        LRU     最近最少使用策略
                                                    缓存的元素有一个时间戳，当缓存容量满了，同时又需要腾出地方来缓存新的元素时，
                                                    那么现有缓存元素中的时间戳 离 当前时间最远的元素将被清出缓存

    -->
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            maxElementsOnDisk="10000000"
            diskExpiryThreadIntervalSeconds="120"
            memoryStoreEvictionPolicy="LRU"/>

    <!--下面的配置是自定义缓存配置，可以复制粘贴，用多套
        name 起的缓存名
        overflowToDisk 当系统宕机时，数据是否保存到上面配置的<diskStore path = "D:/test/cache"/>磁盘中
        diskPersistent 是否缓存虚拟机重启期数据

        另外的配置项：
            clearOnFlush  内存数量最大时是否清除
            diskSpoolBufferSizeMB 设置diskStore( 即：磁盘缓存 )的缓冲区大小，默认是30MB
                                    每个Cache都应该有自己的一个缓冲区
    -->
    <cache
        name="users"
        eternal="false"
        maxElementsInMemory="100"
        overflowToDisk="false"
        diskPersistent="false"
        timeToIdleSeconds="0"
        timeToLiveSeconds="300"
        memoryStoreEvictionPolicy="LRU"
    />
</ehcache>
```





### 在项目中使用Ehcache

使用常用的`@Cacheable`注解举例。



1. 查询条件是单个时（service实现类中直接开注解）

```java
// 这里的value值就是前面xml中配置的哪个cache name值
@Cacheable(value="users", key = "#username")
public User queryUserByUsername(String username) {
	return userMapper.selectUserByUsername(username);
}
```


2. 查询条件是多个时（service实现类中直接开注解）

> 本质：字符串的拼接

```java
// 这里的UserDAO.username+就是封装的UserDAO，里面的属性有username、userage、userPhone
@Cache(value="users", 
       key = "#UserDAO.username + '-' + #UserDAO.userage + '-' + #UserDAO.userPhone")
public User queryUserByUsername(UserDAO userDAO) {
	return userMapper.selectUserByUserDAO(userDAO);
}
```







# 定时任务
## 小顶堆数据结构

就是一个完全二叉树，同时这个二叉树遵循一个规则，根节点存的值永远小于两个子节点存的值。

<img src="https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175008989-138560699.png" alt="image" style="zoom:67%;" />



树结构只是一种逻辑结构，因此：数据还是要存起来的，而这种小顶堆就是采用了数组。

![image](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175003484-73068869.png)



即：数组下标为0的位置不放值，然后把树结构的数据放在对应位置。

- **树结构数据转成数组数据的规律：从上到下、从左到右**，即：根节点、左孩子节点、右孩子节点（对照上面两个图来看）。
- 这种存储方式找父节点也好找，就是数组中( 当前数值的下标值 % 2  ) ，这种算法的原理：就是利用二叉树的深度 和 存放数据个数的关系（ 数列 ），即：顶层最多可以放多少个数据？2的0次方；第二层最多可以存放多少个数据？2的1次方...........





**这种小顶堆需要明白三个点：**

- **存数据的方式： **上述提到了。
- **取数据的方式：从底向上。**即：从最底层开始，若比找寻的值小，那就找父节点，父节点也比所找寻数值小，继续找父节点的父节点.，要是比父节点大，那就找相邻兄弟节点嘛.........依次类推，最后就可以找到所找寻的数据了。
- **存数据的方式：自底向上、逐渐上浮**。即：从最底层开始，存的值 和 父节点相比，比父节点小的话，那存的值就是父节点存的值进行换位.....以此类推。





## 时间轮算法

<img src="https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175007953-1532439253.png" alt="image" style="zoom:67%;" />



### 基础型时间轮

- **模仿时钟，24个刻度( 数组，每一个刻度作为数组的下标 ），每一个刻度后面就是一个链表，这个链表中放对应的定时任务，到了指定时间点就把后面链表中的任务全部遍历出来执行**。
- 缺点：当要弄年、月、秒这种就又要再加轮子，这样就很复杂了，因此：此种方式只适合记一天24小时的定时任务，涉及到年月秒就不行了。



### round型时间轮

- **在前面基础型时间轮的基础上，在每一个刻度的位置再加一个round值（ 每个刻度后面还是一个链表存定时任务 ），round值记录的就是实际需求的值，如：一周，那round值就为7，当然这个round值可以是1，也可以是30....，每一次遍历时钟数组的那24个刻度时，遍历到某一个刻度，那么就让round值减1，知道round值为0时，就表示24数组中当前这个刻度存的定时任务该执行了**。
- 缺点：需要让round值减1，那么就是需要对时间轮进行遍历，如：定时任务应该是4号执行，但是3号遍历时间轮时，定时任务并不执行，而此时也需要遍历时间轮从而让round值减1，这浪费了性能。





### 分量时间轮

- **后续的定时任务框架就是基于这个做的，如：Spring中有一个`@Scheduleed( cron = "x x x x ...." )`注解，它的这个cron时间表达式就是基于这种分量时间轮**。

- 使用多个轮子：
  - 如：一个时间轮记录小时0 - 24，而另一个轮子记录天数0 - 30天。
  - 先遍历天伦中的刻度，若今天是0 -30中要执行定时任务的那一天，那么天轮的刻度指向的就是时轮。
  - 然后再去遍历时轮中对应的那个刻度，从而找到这个刻度后面的链表，将链表遍历出来，执行定时任务。







## Timer

- 底层原理就是：小顶堆，只是它的底层用了一个taskQueue任务队列来充当小顶堆中的哪个数组，存取找的逻辑都是和小顶堆一样的。

- **有着弊端：**
  - **`schedule()`**  API真正的执行时间 取决 上一个任务的结束时间。会出现：少执行了次数。
  - **`scheduleAtFixedRate()`**  API想要的是严格按照预设时间 12:00:00   12:00:02  12:00:04，但是最终结果是：执行时间会乱。
  - **底层调的是`run()`，也就是单线程。缺点：任务阻塞( 阻塞原因：任务超时 )**。



<img src="https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175016763-168850390.png" alt="image" style="zoom:67%;" />



```java
package com.tuling.timer;

import java.util.Date;
import java.util.Timer;
import java.util.TimerTask;

public class TimerTest {

    public static void main(String[] args) {
        
        Timer t = new Timer();	// 任务启动
        
        for (int i=0; i<2; i++){
            TimerTask task = new FooTimerTask("foo"+i);
            t.scheduleAtFixedRate(task,new Date(),2000);	// 任务添加   10s 5次   4 3
            /*
             * 预设的执行时间nextExecutorTime 12:00:00   12:00:02  12:00:04
             * schedule  真正的执行时间 取决上一个任务的结束时间  ExecutorTime   03  05  08  丢任务（少执行了次数）
             * scheduleAtFixedRate  严格按照预设时间 12:00:00   12:00:02  12:00:04（执行时间会乱）
             * 单线程  任务阻塞  任务超时
             * */
        }
    }
}


class FooTimerTask extends TimerTask {

    private String name;

    public FooTimerTask(String name) {
        this.name = name;
    }

    public void run() {
        
        try {
            System.out.println("name="+name+",startTime="+new Date());
            Thread.sleep(3000);
            System.out.println("name="+name+",endTime="+new Date());

            // 因为是单线程，所以解决办法：使用线程池执行
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```





## 定时任务线程池

- **原理：timer + 线程池执行来做到的**。

- **如下的`Executors.newScheduledThreadPool(5);`创建线程池的方法在高并发情况下，最好别用**。



<img src="https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175010877-861716498.png" alt="image" style="zoom:67%;" />



```java
package com.tuling.pool;

import java.util.Date;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class ScheduleThreadPoolTest {

    public static void main(String[] args) {
        
        /*
         * 这种线程池叫做垃圾（开个玩笑）	了解即可
         * 缺点：允许的请求队列长度为 Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM
         * */
        ScheduledExecutorService scheduledThreadPool = Executors.newScheduledThreadPool(5);
        
        for (int i=0; i<2; i++){
            scheduledThreadPool.scheduleAtFixedRate(new Task("task-" + i), 0, 2, TimeUnit.SECONDS);
        }
    }
}


class Task implements Runnable{

    private String name;

    public Task(String name) {
        this.name = name;
    }

    public void run() {
        
        try {
            System.out.println("name="+name+",startTime="+new Date());
            
            Thread.sleep(3000);
            
            System.out.println("name="+name+",endTime="+new Date());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```





## Spring Task：@Scheduled注解实现

- 这玩意儿是Spring提供的，即Spring Task。官网：https://docs.spring.io/spring-framework/reference/6.1-SNAPSHOT/integration/scheduling.html
- **缺点就是其定时时间不能动态更改，它适用于具有固定任务周期的任务**。



> **注意点：**
>
> 要在相应的代码中使用`@Scheduled`注解来进行任务配置，那么就需要在主启动类上加上`@EnableScheduling // 开启定时任务`注解。



> `@Scheduled` 这个注解的几个属性

- `fixedRate`：表示任务执行之间的时间间隔，具体是指两次任务的开始时间间隔，即第二次任务开始时，第一次任务可能还没结束。
- `fixedDelay`：表示任务执行之间的时间间隔，具体是指本次任务结束到下次任务开始之间的时间间隔。
- `initialDelay`：表示首次任务启动的延迟时间。
- `cron 表达式`：秒 分 小时 日 月 周 年 。



> cron表达式说明：可以直接浏览器搜索“cron表达式在线工具”，生成表达式复制粘贴。

<img src="https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175005341-442719966.png" alt="image" style="zoom:67%;" />



上图通配符含义

| **通配符**         | **意义**                                                     |
| ------------------ | ------------------------------------------------------------ |
| **`?`**            | 表示不指定值，即不关心某个字段的取值时使用<br />需要注意的是，月份中的日期和星期可能会起冲突，因此在配置时这两个得有一个是`?` |
| **`*`**            | 表示所有值，例如:在秒的字段上设置 * ,表示每一秒都会触发      |
| **`,`**            | 用来分开多个值，例如在周字段上设置 "MON,WED,FRI" 表示周一，周三和周五触发 |
| **`-`**            | 表示区间，例如在秒上设置 "10-12",表示 10,11,12秒都会触发     |
| **`/`**            | 用于递增触发，如在秒上面设置"5/15" 表示从5秒开始，每增15秒触发(5,20,35,50) |
| **`#`**            | 序号(表示每月的第几个周几)，例如在周字段上设置"6#3"表示在每月的第三个周六，(用<br/>在母亲节和父亲节再合适不过了) |
| **`L`**            | 表示最后的意思<br />在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会自动判断是否是润年<br />在周字段上表示星期六，相当于"7"或"SAT"（注意周日算是第一天）<br />如果在"L"前加上数字，则表示该数据的最后一个。例如在周字段上设置"6L"这样的格式,则表<br/>示"本月最后一个星期五" |
| **`W`**            | 表示离指定日期的最近工作日(周一至周五)<br />例如在日字段上设置"15W"，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发，如果15号正好在工作日(周一至周五)，则就在该天触发<br />如果指定格式为 "1W",它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，"W"前只能设置具体的数字,不允许区间"-") |
| **`L` 和 `W`组合** | 如果在日字段上设置"LW",则表示在本月的最后一个工作日触发(一般指发工资 ) |
| **`周字段的设置`** | 若使用英文字母是不区分大小写的 ，即 MON 与mon相同            |





> cron表达式举例

```json
“0 0 12 * * ?”				每天中午12点触发

“0 15 10 ? * *”				每天上午10:15触发
“0 15 10 * * ?”
“0 15 10 * * ? *”

“0 15 10 * * ? 2005”		2005年的每天上午10:15 触发

“0 0/5 14 * * ?”			在每天下午2点到下午2:55期间的每5分钟触发

“0 0-5 14 * * ?”			在每天下午2点到下午2:05期间的每1分钟触发

“0 10,44 14 ? 3 WED”		每年三月的星期三的下午2:10和2:44触发

“0 15 10 ? * MON-FRI”		周一至周五的上午10:15触发

“0 15 10 ? * 6L”			每月的最后一个星期五上午10:15触发

“0 15 10 ? * 6L 2002-2005”	2002年至2005年的每月的最后一个星期五上午10:15触发

“0 15 10 ? * 6#3”			每月的第三个星期五上午10:15触发

0 23-7/2，8 * * *			晚上11点到早上8点之间每两个小时，早上八点

0 11 4 * 1-3				每个月的4号和每个礼拜的礼拜一到礼拜三的早上11点
```

> **注意点：**
>
> cron表达式中“年”不可以跨年，默认是当前年执行（即：想每2年执行一次做不到），所以一般情况下“年”可以不指定，即：cron表达式只写6位即可。



## Redis实现：分布式定时任务

前面的方式都是单机的。



### zset实现

> **逻辑**

1. 将定时任务存放到 ZSet 集合中，并且将过期时间存储到 ZSet 的 Score 字段中。
2. 通过一个无线循环来判断当前时间内是否有需要执行的定时任务，如果有则进行执行。

```java
import redis.clients.jedis.Jedis;
import utils.JedisUtils;
import java.time.Instant;
import java.util.Set;

public class DelayQueueExample {
    // zset key
    private static final String _KEY = "myTaskQueue";

    public static void main(String[] args) throws InterruptedException {
        Jedis jedis = JedisUtils.getJedis();
        // 30s 后执行
        long delayTime = Instant.now().plusSeconds(30).getEpochSecond();
        jedis.zadd(_KEY, delayTime, "order_1");
        // 继续添加测试数据
        jedis.zadd(_KEY, Instant.now().plusSeconds(2).getEpochSecond(), "order_2");
        jedis.zadd(_KEY, Instant.now().plusSeconds(2).getEpochSecond(), "order_3");
        jedis.zadd(_KEY, Instant.now().plusSeconds(7).getEpochSecond(), "order_4");
        jedis.zadd(_KEY, Instant.now().plusSeconds(10).getEpochSecond(), "order_5");
        // 开启定时任务队列
        doDelayQueue(jedis);
    }

    /**
     * 定时任务队列消费
     * @param jedis Redis 客户端
     */
    public static void doDelayQueue(Jedis jedis) throws InterruptedException {
        while (true) {
            // 当前时间
            Instant nowInstant = Instant.now();
            long lastSecond = nowInstant.plusSeconds(-1).getEpochSecond(); // 上一秒时间
            long nowSecond = nowInstant.getEpochSecond();
            // 查询当前时间的所有任务
            Set<String> data = jedis.zrangeByScore(_KEY, lastSecond, nowSecond);
            for (String item : data) {
                // 消费任务
                System.out.println("消费：" + item);
            }
            // 删除已经执行的任务
            jedis.zremrangeByScore(_KEY, lastSecond, nowSecond);
            Thread.sleep(1000); // 每秒查询一次
        }
    }
}
```





### 键空间实现

> **逻辑**

1. **给所有的定时任务设置一个过期时间**。

2. **等到了过期之后，我们通过订阅过期消息就能感知到定时任务需要被执行了，此时我们执行定时任务即可**。



> **注意点：**
>
> 默认情况下 Redis 是不开启键空间通知的，需要我们通过 `config set notify-keyspace-events Ex` 的命令手动开启。

开启之后定时任务的代码如下：

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPubSub;
import utils.JedisUtils;

public class TaskExample {
    public static final String _TOPIC = "__keyevent@0__:expired"; // 订阅频道名称
    public static void main(String[] args) {
        Jedis jedis = JedisUtils.getJedis();
        // 执行定时任务
        doTask(jedis);
    }

    /**
     * 订阅过期消息，执行定时任务
     * @param jedis Redis 客户端
     */
    public static void doTask(Jedis jedis) {
        // 订阅过期消息
        jedis.psubscribe(new JedisPubSub() {
            @Override
            public void onPMessage(String pattern, String channel, String message) {
                // 接收到消息，执行定时任务
                System.out.println("收到消息：" + message);
            }
        }, _TOPIC);
    }
}
```







## Quartz 任务调度

组成结构图如下：需要时自行摸索即可。

<img src="https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302175010243-618701512.png" alt="image" style="zoom:67%;" />







### 简单示例

1. 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

2. 定义job

```java
public class MyJob implements Job {
    private Logger log = LoggerFactory.getLogger(MyJob.class);

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {

        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        TriggerKey triggerKey = jobExecutionContext.getTrigger().getKey();

        log.info("触发器：{},所属组：{},执行时间：{}，执行任务：{}",
                triggerKey.getName(), 
                triggerKey.getGroup(), 
                dateFormat.format(new Date()), 
                "hello Spring Boot Quartz...");
    }
}
```

3. 编写QuartzConfig

```java
public class QuartzConfig {

    @Bean
    public JobDetail jobDetail() {
        return JobBuilder.newJob(MyJob.class)
                .storeDurably()
                .build();
    }

    @Bean
    public Trigger trigger01() {
        SimpleScheduleBuilder scheduleBuilder = SimpleScheduleBuilder.simpleSchedule()
                // 每一秒执行一次
                .withIntervalInSeconds(1)
                // 永久重复，一直执行下去
                .repeatForever();

        return TriggerBuilder.newTrigger()
                // 参数1、trigger名字；参数2、当前这个trigger所属的组 - 参考时间轮存储任务，那个刻度后面是怎么存的任务
                .withIdentity("trigger01", "group1")
                .withSchedule(scheduleBuilder)
                // 哪一个job，上一个方法中bean注入
                .forJob("jobDetail")
                .build();
    }

    /**
     * 每两秒触发一次任务
     */
    @Bean
    public Trigger trigger02() {
        return TriggerBuilder
                .newTrigger()
                .withIdentity("triiger02", "group1")
                // cron时间表达式
                .withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ? *"))
                .forJob("jobDetail")
                .build();
    }
}
```







# 集成支付宝支付

官网地址：https://open.alipay.com/api

选择自己需要的方式：本文示例选择“手机网站支付”，其他的都是差不多的。

![image-20240305180322826](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240305180236240-1765340961.png)





## 申请与其条件

支持的账号类型：[支付宝企业账号](https://opendocs.alipay.com/common/02kkum)、[支付宝个人账号](https://opendocs.alipay.com/common/02kg61)。

签约申请提交材料要求：

- 提供网站地址，网站能正常访问且页面显示完整，网站需要明确经营内容且有完整的商品信息。
- 网站必须通过 ICP 备案，且备案主体需与支付宝账号主体一致。若网站备案主体与当前账号主体不同时需上传授权函。
- 个人账号申请，需提供营业执照，且支付宝账号名称需与营业执照主体一致。



> **提示**
>
> 需按照要求提交材料，若部分材料不合格，收款额度将受到限制（单笔收款 ≤ 2000 元，单日收款 ≤ 20000 元）。若签约时未能提供相关材料（如营业执照），请在合约生效后的 30 天内补全，否则会影响正常收款。



**费率**

| **收费模式** | **费率**  |
| ------------ | --------- |
| 单笔收费     | 0.6%-1.0% |



特殊行业费率为 1.0%，非特殊行业费率为 0.6%。特殊行业包含：休闲游戏、网络游戏点卡、游戏渠道代理、游戏系统商、网游周边服务、交易平台、网游运营商（含网页游戏）等。





## 接入准备

官方文档：https://opendocs.alipay.com/open/203/107084?pathHash=a33de091

整体流程：

![image-20240305180818081](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240305180731504-1117620927.png) 



为了提供数据传输的安全性，在进行传输的时候需要对数据进行加密：

常见的加密方式： 

1、不可逆加密：只能会数据进行加密不能解密

2、可逆加密：可以对数据加密也可以解密

可逆加密可以再细分为：

1、对称加密： 加密和解密使用同一个秘钥

![image-20240305180848219](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240305180801176-2027417495.png)

2、非对称加密：加密和解密使用的是不同的秘钥

![image-20240305180910399](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240305180823373-775263862.png)



支付宝为了提供数据传输的安全性使用了两个秘钥对：

 ![image-20240305180933808](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240305180846759-1543124324.png)







## 手机端网站支付接入流程

官方文档：https://opendocs.alipay.com/open/203/105285?pathHash=ada1de5b

系统交互流程图：

![image-20230709164753985](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240305181925562-536306967.png)









## 示例

支付宝支付一般都是下面这样，本文做的是简单示例

![image-20240305201729868](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240305201643283-1953480071.png)



1. 依赖

```xml
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
</dependency>
```

2. YAML配置：resources目录下新建 `application-alipay.yml`

```yaml
com:
  alipay:
    alipay_url: https://openapi.alipay.com/gateway.do	# 接入的地址
    app_id: xxxxxxxxxxxxxxx	# 这个id在前面申请后进入网页的控制台可以看到
    app_private_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx		# 私钥，前面申请后会有这个
    alipay_public_key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx	# 密钥，前面申请后会有这个
    return_payment_url: http://zixieqing.demo.com/#/pages/money/paySuccess	# 支付宝支付成功或失败后，我们自己的页面地址，也叫同步回调
    notify_payment_url: http://zixieqing.demo.com/api/order/alipay/callback/notify	# 支付宝调用的我们自己服务接口的地址		也叫异步回调
```

> 关于 return_payment_url 和 notify_payment_url：这两个东西都是支付宝支付来调用我们自己的东西，这都需要使用到域名，这样支付宝支付才可以调用到，公司中使用的就是公司的域名，自己要的话要么买个域名，要么就去搞内网穿透之类的。



提示：记得在在`application-dev.yml`文件中导入2中的配置

```yaml
spring:
  config:
    import: application-alipay.yml
```

3. 读取 `application-alipay.yml`中的配置

```java
@Data
@ConfigurationProperties(prefix = "com.alipay")
public class AlipayProperties {

    private String alipayUrl;
    private String appPrivateKey;
    public  String alipayPublicKey;
    private String appId;
    public  String returnPaymentUrl;
    public  String notifyPaymentUrl;

    public final static String format="json";
    public final static String charset="utf-8";
    public final static String sign_type="RSA2";

}
```

记得在启动类上加上 `@EnableConfigurationProperties` 注解

```java
@EnableConfigurationProperties(value = { AlipayProperties.class })
```



4. 配置AlipayClient

```java
@Configuration
public class AlipayConfiguration {

    @Autowired
    private AlipayProperties alipayProperties ;

    /**
     * 配置发送请求的核心对象：AlipayClient
     */
    @Bean
    public AlipayClient alipayClient(){
        AlipayClient alipayClient = new DefaultAlipayClient(alipayProperties.getAlipayUrl() ,
                alipayProperties.getAppId() ,
                alipayProperties.getAppPrivateKey() ,
                AlipayProperties.format ,
                AlipayProperties.charset ,
                alipayProperties.getAlipayPublicKey() ,
                AlipayProperties.sign_type );
        
        return alipayClient;
    }
}
```



5. 下单支付服务

```java
/**
 * 支付服务
 */
public interface AlipayService {
    String submitAlipay(String orderNo);
}


/**
 * 支付服务实现类
 */
@Slf4j
@Service
public class AlipayServiceImpl implements AlipayService {

    @Autowired
    private AlipayClient alipayClient;

    @Autowired
    private PaymentInfoService paymentInfoService;

    @Autowired
    private AlipayProperties alipayProperties ;

	@Override
    @SneakyThrows  // lombok的注解，对外声明异常
    public String submitAlipay(String orderNo) {

        // 保存支付记录
        PaymentInfo paymentInfo = paymentInfoService.savePaymentInfo(orderNo);

        // 创建API对应的request
        AlipayTradeWapPayRequest alipayRequest = new AlipayTradeWapPayRequest();

        // 同步回调		同步调用我们自己的页面，支付成功或失败的页面路径
        alipayRequest.setReturnUrl(alipayProperties.getReturnPaymentUrl());

        /*
         * 异步回调		异步调用我们自己的服务"接口"路径
         * 			   如：更新支付记录状态（已支付 / 未支付）、更新订单状态、更新商品销量
         * */
        alipayRequest.setNotifyUrl(alipayProperties.getNotifyPaymentUrl());

        // 准备请求参数 ，声明一个map 集合
        HashMap<String, Object> map = new HashMap<>();
        // 订单编号
        map.put("out_trade_no",paymentInfo.getOrderNo());
        map.put("product_code","QUICK_WAP_WAY");
        // 支付金额
        map.put("total_amount",paymentInfo.getAmount());
        map.put("subject",paymentInfo.getContent());
        alipayRequest.setBizContent(JSON.toJSONString(map));

        // 发送请求
        AlipayTradeWapPayResponse response = alipayClient.pageExecute(alipayRequest);
        if(response.isSuccess()){		// 支付成功
            log.info("调用成功");
            // 返回form表单数据		就是平时支付宝支付成功返回的那个页面的内容
            return response.getBody();
        } else {	// 支付失败
            log.info("调用失败");
            throw new CustomException(ResultCodeEnum.DATA_ERROR);
        }
    }
}
```

上述涉及的PaymentInfoService：

```java
/**
 * 支付信息服务
 */
public interface PaymentInfoService {
    PaymentInfo savePaymentInfo(String orderNo);
}


/**
 * 支付信息服务实现类
 */
@Service
public class PaymentInfoServiceImpl implements PaymentInfoService {

    @Autowired
    private PaymentInfoMapper paymentInfoMapper ;

    @Autowired
    private OrderFeignClient orderFeignClient ;

    @Override
    public PaymentInfo savePaymentInfo(String orderNo) {

        // 查询支付信息
        PaymentInfo paymentInfo = paymentInfoMapper.getByOrderNo(orderNo);
        // 若无支付信息则保存支付信息，若已经已经存在了就不用进行保存(一个订单支付失败以后可以继续支付)
        if(null == paymentInfo) {
            // OpenFeign远程（这是Spring Cloud中的内容）调用根据订单编号查询订单信息
            OrderInfo orderInfo = orderFeignClient.getOrderInfoByOrderNo(orderNo).getData();
            // 封装付款信息
            paymentInfo = new PaymentInfo();
            paymentInfo.setUserId(orderInfo.getUserId());
            paymentInfo.setPayType(orderInfo.getPayType());
            String content = "";
            for(OrderItem item : orderInfo.getOrderItemList()) {
                content += item.getSkuName() + " ";
            }
            paymentInfo.setContent(content);
            paymentInfo.setAmount(orderInfo.getTotalAmount());
            paymentInfo.setOrderNo(orderNo);
            paymentInfo.setPaymentStatus(0);

            // 保存支付信息
            paymentInfoMapper.save(paymentInfo);
        }
        
        // 返回支付信息
        return paymentInfo;
    }
}
```



> 要了解Spring Cloud去这里：https://www.cnblogs.com/xiegongzi/p/17858107.html

PaymentInfo实体类：

```java
@Data
@Schema(description = "支付信息实体类")
public class PaymentInfo extends BaseEntity {	// BaseEntity是基本的id、创建时间、删除时间等

   private static final long serialVersionUID = 1L;

   @Schema(description = "用户id")
   private Long userId;

   @Schema(description = "订单号")
   private String orderNo;

   @Schema(description = "付款方式：1-微信 2-支付宝")
   private Integer payType;

   @Schema(description = "交易编号（微信或支付）")
   private String outTradeNo;

   @Schema(description = "支付金额")
   private BigDecimal amount;

   @Schema(description = "交易内容")
   private String content;

   @Schema(description = "支付状态：0-未支付 1-已支付")
   private Integer paymentStatus;

   @Schema(description = "回调时间")
   private Date callbackTime;

   @Schema(description = "回调信息")
   private String callbackContent;

}
```



6. 在网关中记得配置支付的路由

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service-pay
          uri: lb://service-pay
          predicates:
            - Path=/api/order/alipay/**
```



7. notify_payment_url 异步回调我们自己的服务接口

```java
@Controller
@RequestMapping("/api/order/alipay")
public class AlipayController {

    @Autowired
    private AlipayService alipayService;

    @Autowired
    private AlipayProperties alipayProperties;

    @Autowired
    private PaymentInfoService paymentInfoService;

    /**
     * 签名校验
     *
     * @param paramMap 支付宝支付给我们返回的数据
     * @return 执行状态
     */
    @Operation(summary="支付宝异步回调")
    @RequestMapping("callback/notify")
    @ResponseBody
    public String alipayNotify(@RequestParam Map<String, String> paramMap, HttpServletRequest request) {
        
        log.info("AlipayController...alipayNotify方法执行了...");

        // 调用SDK验证签名
        boolean signVerified = false;
        try {
            // 让支付宝支付来校验进入当前的请求是否是支付宝支付发过来的而不是伪造的请求
            signVerified = AlipaySignature.rsaCheckV1(paramMap, 
                                                      alipayProperties.getAlipayPublicKey(),
                                                      AlipayProperties.charset, 
                                                      AlipayProperties.sign_type);
        } catch (AlipayApiException e) {
            e.printStackTrace();
        }

        // 交易状态
        String trade_status = paramMap.get("trade_status");

        // 合法请求
        if (signVerified) {

            /* 
             * 验签成功后，按照支付结果异步通知中的描述，对支付结果中的业务内容进行二次校验
             * 校验成功后在response中返回success并继续商户自身业务处理，校验失败返回failure
             * */
            if ("TRADE_SUCCESS".equals(trade_status) || "TRADE_FINISHED".equals(trade_status)) {
                // 正常的支付成功，那么可以进行更新支付记录状态（已支付 / 未支付）、更新订单状态、更新商品销量等操作
                paymentInfoService.updatePaymentStatus(paramMap);
                // .....................
                // paramMap.get("out_trade_no")		可以获取当前支付订单的编号
                return "success";
            }

        } else {
            // 验签失败则记录异常日志，并在response中返回failure.
            return "failure";
        }

        return "failure";
    }
}
```









# 附加：Spring 事务失效场景

这个玩意儿是Spring的，想了想在这里也放一章节吧。

需要同时写入多张表的数据。为了保证操作的原子性（要么同时成功，要么同时失败），避免数据不一致的情况，我们一般都会用到Spring事务（也会选择其他事务框架）。

spring事务用起来贼爽，就用一个简单的注解：`@Transactional`，就能轻松搞定事务。而且一直用一直爽。

但如果使用不当，它也会坑人于无形。

![img](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302220705922-1362921420.jpg)





## 事务不生效

### 访问权限问题

Java的访问权限主要有四种：private、default、protected、public，它们的权限从左到右，依次变大。

在开发过程中，把某些事务方法，定义了错误的访问权限，就会导致事务功能出问题。

```java
@Service
public class UserService {
    
    @Transactional
    private void add(UserModel userModel) {
         saveData(userModel);
         updateData(userModel);
    }
}
```

上述代码就会导致事务失效，**因为Spring要求被代理方法必须是`public`的**。

在 `AbstractFallbackTransactionAttributeSource` 类的 `computeTransactionAttribute` 方法中有个判断，如果目标方法不是public，则`TransactionAttribute`返回null，即不支持事务。

```java
protected TransactionAttribute computeTransactionAttribute(Method method, @Nullable Class<?> targetClass) {
    // Don't allow no-public methods as required.
    if (allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
      return null;
    }

    // The method may be on an interface, but we need attributes from the target class.
    // If the target class is null, the method will be unchanged.
    Method specificMethod = AopUtils.getMostSpecificMethod(method, targetClass);

    // First try is the method in the target class.
    TransactionAttribute txAttr = findTransactionAttribute(specificMethod);
    if (txAttr != null) {
      return txAttr;
    }

    // Second try is the transaction attribute on the target class.
    txAttr = findTransactionAttribute(specificMethod.getDeclaringClass());
    if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
      return txAttr;
    }

    if (specificMethod != method) {
      // Fallback is to look at the original method.
      txAttr = findTransactionAttribute(method);
      if (txAttr != null) {
        return txAttr;
      }
      // Last fallback is the class of the original method.
      txAttr = findTransactionAttribute(method.getDeclaringClass());
      if (txAttr != null && ClassUtils.isUserLevelMethod(method)) {
        return txAttr;
      }
    }
    return null;
  }
```





### 方法用final修饰

有时候，某个方法不想被子类重新，这时可以将该方法定义成final的。普通方法这样定义是没问题的，但如果将事务方法定义成final就会导致事务失效。

```java
@Service
public class UserService {

    @Transactional
    public final void add(UserModel userModel){
        saveData(userModel);
        updateData(userModel);
    }
}
```

因为Spring事务底层使用了AOP帮我们生成代理类，在代理类中实现的事务功能。**如果某个方法用final修饰了，那么在它的代理类中，就无法重写该方法，而添加事务功能**。

> **重要提示**
>
> 如果某个方法是static的，同样无法通过动态代理，变成事务方法。





### 方法内部调用

有时需要在某个Service类的某个事务方法中调用另外一个事务方法。

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    @Transactional
    public void add(UserModel userModel) {
        userMapper.insertUser(userModel);
        updateStatus(userModel);
    }

    @Transactional
    public void updateStatus(UserModel userModel) {
        doSameThing();
    }
}
```

上述代码就会导致事务失效，因为updateStatus方法拥有事务的能力是Spring AOP生成代理对象，但是updateStatus这种方法直接调用了this对象的方法，所以updateStatus方法不会生成事务。

如果有些场景，确实想在同一个类的某个方法中，调用它自己的另外一个方法，该怎么办？

1. 第一种方式：新加一个Service方法。把`@Transactional`注解加到新Service方法上，把需要事务执行的代码移到新方法中。

```java
@Servcie
public class ServiceA {
   @Autowired
   prvate ServiceB serviceB;

   public void save(User user) {
         queryData1();
         queryData2();
         serviceB.doSave(user);
   }
 }




 @Servcie
 public class ServiceB {

    @Transactional(rollbackFor=Exception.class)
    public void doSave(User user) {
       addData1();
       updateData2();
    }
 }
```

2. 第二种方式：在该Service类中注入自己。如果不想再新加一个Service类，在该Service类中注入自己也是一种选择。

```java
@Servcie
public class ServiceA {
   @Autowired
   prvate ServiceA serviceA;

   public void save(User user) {
         queryData1();
         queryData2();
         serviceA.doSave(user);
   }

   @Transactional(rollbackFor=Exception.class)
   public void doSave(User user) {
       addData1();
       updateData2();
    }
 }
```

第二种做法会不会出现循环依赖问题？

不会。Spring IOC内部的三级缓存保证了它，不会出现循环依赖问题。但有些坑，解放方式去参考：[Spring：如何解决循环依赖](https://mp.weixin.qq.com/s?__biz=MzkwNjMwMTgzMQ==&mid=2247490271&idx=1&sn=e4476b631c48882392bd4cd06d579ae9&source=41#wechat_redirect)



> 循环依赖：就是一个或多个对象实例之间存在直接或间接的依赖关系，这种依赖关系构成了构成一个环形调用。

第一种情况：自己依赖自己的直接依赖。

![图片](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302223551265-333574175.png)





第二种情况：两个对象之间的直接依赖。

![图片](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302223620257-597068237.png)



第三种情况：多个对象之间的间接依赖。

![图片](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302223639782-1608783316.png)



前面两种情况的直接循环依赖比较直观，非常好识别，但是第三种间接循环依赖的情况有时候因为业务代码调用层级很深，不容易识别出来。



> 循环依赖的N种场景

![图片](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302223739347-1215406426.png)





3. 第三种方式：通过AopContent类。在该Service类中使用`AopContext.currentProxy()`获取代理对象。

上面第二种方式确实可以解决问题，但是代码看起来并不直观，还可以通过在该Service类中使用AOPProxy获取代理对象，实现相同的功能。

```java
@Servcie
public class ServiceA {

   public void save(User user) {
         queryData1();
         queryData2();
         ((ServiceA)AopContext.currentProxy()).doSave(user);
   }

   @Transactional(rollbackFor=Exception.class)
   public void doSave(User user) {
       addData1();
       updateData2();
    }
 }
```





### 未被Spring托管

**使用Spring事务的前提是：对象要被Spring管理，需要创建bean实例**。

通常情况下，我们通过`@Controller`、`@Service`、`@Component`、`@Repository`等注解，可以自动实现bean实例化和依赖注入的功能。

但要是噼里啪啦敲完Service类，忘了加 `@Service` 注解呢？

那么该类不会交给Spring管理，它的方法也不会生成事务。





### 多线程调用

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;
    
    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {
        
        userMapper.insertUser(userModel);
        
        new Thread(() -> {
            roleService.doOtherThing();
        }).start();
    }
}



@Service
public class RoleService {

    @Transactional
    public void doOtherThing() {
        System.out.println("保存role表数据");
    }
}
```

上述代码事务方法add中是另外一个线程调用的事务方法doOtherThing。

这样会导致两个方法不在同一个线程中，获取到的数据库连接不一样，从而是两个不同的事务。如果想doOtherThing方法中抛了异常，add方法也回滚是不可能的。

**Spring事务其实是通过数据库连接来实现的。当前线程中保存了一个map，key是数据源，value是数据库连接**。

```java
private static final ThreadLocal<Map<Object, Object>> resources = 
    new NamedThreadLocal<>("Transactional resources");
```

我们说的同一个事务，其实是指同一个数据库连接，只有拥有同一个数据库连接才能同时提交和回滚。如果在不同的线程，拿到的数据库连接肯定是不一样的，所以是不同的事务。





### 表不支持事务

MySQL 5之前，默认的数据库引擎是`myisam`。好处是：索引文件和数据文件是分开存储的，对于查多写少的单表操作，性能比innodb更好。

但有个很致命的问题是：`不支持事务`。如果需要跨多张表操作，由于其不支持事务，数据极有可能会出现不完整的情况。

> **提示**
>
> 有时候我们在开发的过程中，发现某张表的事务一直都没有生效，那不一定是spring事务的锅，最好确认一下你使用的那张表，是否支持事务。





### 未开启事务

有时候，事务没有生效的根本原因是没有开启事务。

看到这句话可能会觉得好笑。因为开启事务不是一个项目中，最最最基本的功能吗？为什么还会没有开启事务？

如果使用的是Spring Boot项目，那很幸运。因为Spring Boot通过 `DataSourceTransactionManagerAutoConfiguration` 类，已经默默的帮忙开启了事务。自己所要做的事情很简单，只需要配置`spring.datasource`相关参数即可。

但如果使用的还是传统的Spring项目，则需要在`applicationContext.xml`文件中，手动配置事务相关参数。如果忘了配置，事务肯定是不会生效的。

```xml
<!-- 配置事务管理器 --> 
<bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager"> 
    <property name="dataSource" ref="dataSource"></property> 
</bean> 

<tx:advice id="advice" transaction-manager="transactionManager"> 
    <tx:attributes> 
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes> 
</tx:advice> 

<!-- 用切点把事务切进去 --> 
<aop:config> 
    <aop:pointcut expression="execution(* com.zixieqing.*.*(..))" id="pointcut"/> 
    <aop:advisor advice-ref="advice" pointcut-ref="pointcut"/> 
</aop:config> 
```

> **注意**
>
> 如果在pointcut标签中的切入点匹配规则配错了的话，有些类的事务也不会生效。







## 事务不回滚

### 错误的传播特性

在使用`@Transactional`注解时，是可以指定`propagation`参数的。

该参数的作用是指定事务的传播特性，Spring目前支持7种传播特性：

- `REQUIRED` 如果当前上下文中存在事务，那么加入该事务，如果不存在事务，创建一个事务，这是默认的传播属性值。
- `REQUIRES_NEW` 每次都会新建一个事务，并且同时将上下文中的事务挂起，执行当前新建事务完成以后，上下文事务恢复再执行。
- `NESTED` 如果当前上下文中存在事务，则嵌套事务执行，如果不存在事务，则新建事务。
- `SUPPORTS` 如果当前上下文存在事务，则支持事务加入事务，如果不存在事务，则使用非事务的方式执行。
- `MANDATORY` 如果当前上下文中存在事务，否则抛出异常。
- `NOT_SUPPORTED` 如果当前上下文中存在事务，则挂起当前事务，然后新的方法在没有事务的环境中执行。
- `NEVER` 如果当前上下文中存在事务，则抛出异常，否则在无事务环境上执行代码。



如果我们在手动设置propagation参数的时候，把传播特性设置错了就会出问题。

```java
@Service
public class UserService {

    // Propagation.NEVER	这种类型的传播特性不支持事务，如果有事务则会抛异常
    @Transactional(propagation = Propagation.NEVER)
    public void add(UserModel userModel) {
        saveData(userModel);
        updateData(userModel);
    }
}
```

目前只有这三种传播特性才会创建新事务：REQUIRED，REQUIRES_NEW，NESTED。







### 自己吞了异常

事务不会回滚，最常见的问题是：开发者在代码中手动try...catch了异常。

```java
@Slf4j
@Service
public class UserService {
    
    @Transactional
    public void add(UserModel userModel) {
        try {
            saveData(userModel);
            updateData(userModel);
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }
}
```

这种情况下Spring事务当然不会回滚，因为开发者自己捕获了异常，又没有手动抛出，换句话说就是把异常吞掉了。

**如果想要Spring事务能够正常回滚，必须抛出它能够处理的异常。如果没有抛异常，则Spring认为程序是正常的**。





### 手动抛了别的异常

即使开发者没有手动捕获异常，但如果抛的异常不正确，Spring事务也不会回滚。

```java
@Slf4j
@Service
public class UserService {
    
    @Transactional
    public void add(UserModel userModel) throws Exception {
        try {
             saveData(userModel);
             updateData(userModel);
        } catch (Exception e) {
            log.error(e.getMessage(), e);
            throw new Exception(e);
        }
    }
}
```

手动抛出了异常：Exception，事务同样不会回滚。

**因为Spring事务，默认情况下只会回滚`RuntimeException`（运行时异常）和`Error`（错误），对于普通的Exception（非运行时异常），它不会回滚**。





### 自定义了回滚异常

在使用`@Transactional`注解声明事务时，有时我们想自定义回滚的异常，Spring也是支持的。可以通过设置`rollbackFor`参数，来完成这个功能。

但如果这个参数的值设置错了，就会引出一些莫名其妙的问题，

```java
@Service
public class UserService {
    
    @Transactional(rollbackFor = BusinessException.class)
    public void add(UserModel userModel) throws Exception {
       saveData(userModel);
       updateData(userModel);
    }
}
```

如果在执行上面这段代码，保存和更新数据时，程序报错了，抛了SqlException、DuplicateKeyException等异常。而BusinessException是我们自定义的异常，报错的异常不属于BusinessException，所以事务也不会回滚。

即使rollbackFor有默认值，但阿里巴巴开发者规范中，还是要求开发者重新指定该参数。why？

**因为如果使用默认值，一旦程序抛出了Exception，事务不会回滚，这会出现很大的bug。所以，建议一般情况下，将该参数设置成：Exception或Throwable**。





### 嵌套事务回滚多了

```java
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {
        userMapper.insertUser(userModel);
        roleService.doOtherThing();
    }
}



@Service
public class RoleService {

    @Transactional(propagation = Propagation.NESTED)
    public void doOtherThing() {
        System.out.println("保存role表数据");
    }
}
```

这种情况使用了嵌套的内部事务，原本是希望调用`roleService.doOtherThing()`方法时，如果出现了异常，只回滚doOtherThing方法里的内容，不回滚 userMapper.insertUser里的内容，即回滚保存点。。但事实是，insertUser也回滚了。why？

因为doOtherThing方法出现了异常，没有手动捕获，会继续往上抛，到外层add方法的代理方法中捕获了异常。所以，这种情况是直接回滚了整个事务，不只回滚单个保存点。

怎么样才能只回滚保存点？

将内部嵌套事务放在try/catch中，并且不继续往上抛异常。这样就能保证，如果内部嵌套事务中出现异常，只回滚内部事务，而不影响外部事务。

```java
@Slf4j
@Service
public class UserService {

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private RoleService roleService;

    @Transactional
    public void add(UserModel userModel) throws Exception {

        userMapper.insertUser(userModel);
        try {
            roleService.doOtherThing();
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }
}
```





## 大事务问题

在使用Spring事务时，有个让人非常头疼的问题，就是大事务问题。

通常情况下，我们会在方法上`@Transactional`注解，填加事务功能，

但`@Transactional`注解，如果被加到方法上，有个缺点就是整个方法都包含在事务当中了。

```java
@Service
public class UserService {
    
    @Autowired 
    private RoleService roleService;
    
    @Transactional
    public void add(UserModel userModel) throws Exception {
       query1();
       query2();
       query3();
       roleService.save(userModel);
       update(userModel);
    }
}


@Service
public class RoleService {
    
    @Autowired 
    private RoleService roleService;
    
    @Transactional
    public void save(UserModel userModel) throws Exception {
       query4();
       query5();
       query6();
       saveData(userModel);
    }
}
```

上述代码，在UserService类中，其实只有这两行才需要事务：

```java
roleService.save(userModel);
update(userModel);
```

在RoleService类中，只有这一行需要事务：

```java
saveData(userModel);
```

而上面的写法会导致所有的query方法也被包含在同一个事务当中。

如果query方法非常多，调用层级很深，而且有部分查询方法比较耗时的话，会造成整个事务非常耗时，从而造成大事务问题。

![img](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240302234336095-218612978.jpg)





## 编程式事务

上面这些内容都是基于`@Transactional`注解的，主要说的是它的事务问题，我们把这种事务叫做：`声明式事务`。

其实，Spring还提供了另外一种创建事务的方式，即通过手动编写代码实现的事务，我们把这种事务叫做：`编程式事务`。

在Spring中为了支持编程式事务，专门提供了一个类：`TransactionTemplate`，在它的`execute()`方法中，就实现了事务的功能。

```java
   @Autowired
   private TransactionTemplate transactionTemplate;
   
   ...
   
   public void save(final User user) {
       
         queryData1();
         queryData2();
       
         transactionTemplate.execute((status) => {
            addData1();
            updateData2();
            return Boolean.TRUE;
         })
   }
```

相较于`@Transactional`注解声明式事务，我更建议大家使用，基于`TransactionTemplate`的编程式事务。主要原因如下：

1. 避免由于Spring AOP问题，导致事务失效的问题。
2. 能够更小粒度的控制事务的范围，更直观。

> **提示**
>
> 建议在项目中少使用`@Transactional`注解开启事务。但并不是说一定不能用它，如果项目中有些业务逻辑比较简单，而且不经常变动，使用`@Transactional`注解开启事务开启事务也无妨，因为它更简单，开发效率更高，但是千万要小心事务失效的问题。





## 参考

- https://mp.weixin.qq.com/s/D4q8pHa4Avv9wzr9wuVW_A
- https://mp.weixin.qq.com/s/TM5TXVH6cQ42M-UikvlNgg









# 附加：自定义注解

场景：记录日志。在方法执行前 / 后 / 环绕，将一些记录插入数据库。

使用Spring AOP做增强：前置增强、后置增强、环绕增强。



1. 依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

2. 自定义注解

```java
/**
 * 自定义操作日志记录注解
 */

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {

    // 模块名称
    String title();
    // 操作人类别
    OperatorType operatorType() default OperatorType.MANAGE;
    // 业务类型（0其它 1新增 2修改 3删除）
    int businessType();
    // 是否保存请求的参数
    boolean isSaveRequestData() default true;
    // 是否保存响应的参数
    boolean isSaveResponseData() default true;
    
}
```

OperatorType枚举类：

```java
/**
 * 操作人类别
 */
public enum OperatorType {
    OTHER,		// 其他
    MANAGE,		// 后台用户
    MOBILE		// 手机端用户
}
```



3. 增强逻辑：使用环绕增强做示例

```java
import com.zixieqing.spzx.log.annotation.Log;
import lombok.extern.slf4j.Slf4j;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

/**
 * <p>
 * 自定义注解 @log 增强逻辑
 * </p>
 *
 * <p>@author : Zixq</p>
 */

@Aspect
@Component
@Slf4j
public class LogAspect {
    /**
     * 环绕增强
     *      前置增强：@Before()
     *      后置增强：@After()
     *
     * @param joinPoint 切点  可以获取目标方法的参数，返回值，异常信息等
     * @param sysLog 自定义注解
     * @return 业务方法执行状态
     */
    @Around(value = "@annotation(sysLog)")
    public Object doAroundAdvice(ProceedingJoinPoint joinPoint , Log sysLog) {
        
        Object proceed = null;
        try {
            // 执行业务方法
            proceed = joinPoint.proceed();
        } catch (Throwable e) {
            // 防止事务失效（自己吞了异常，导致Spring事务不能正常回滚）：
            // 					手动抛出事务能处理的异常 RuntimeException，因Spring事务只能处理此种异常
            // Spring事务不能处理exception、error
            throw new RuntimeException(e);
        }

        // 返回执行结果
        return proceed;
    }
}
```



> **提示**
>
> 有时需要在如上面LogAspect中操作数据库，但又不可能在LogAspect所在模块引入数据库相关的东西，那么可以采用：LogAspect所在模块定义操作数据库的service接口，然后在真正操作数据库的模块（如：xxx-manager）实现该接口操作数据库，最后在LogAspect中`装配`（`@Autowired` 或 `@resource`）其service接口类即可。



4. 让增强逻辑能在其他业务服务中使用

想让LogAspect这个切面类在其他的业务服务中进行使用，那么就需要该切面类纳入到Spring容器中。Spring Boot默认会扫描和启动类所在包相同包中的bean以及子包中的bean

本示例中，LogAspect切面类不满足扫描条件，因此无法直接在业务服务中进行使用。那么此时可以通过自定义注解进行实现，

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Import(value = LogAspect.class)	// 通过Import注解导入日志切面类到Spring容器中
public @interface EnableLogAspect {
    
}
```



5. 使用

1）、在需要使用的业务服务中引入前面1 - 4模块所在。

```xml
 <dependency>
     <groupId>com.zixieqing</groupId>
     <artifactId>common-log</artifactId>
     <version>1.0-SNAPSHOT</version>
 </dependency>
```

2）、启动类加上4中自定义的`@EnableLogAspect  // 开启日志增强功能`

```java
@EnableLogAspect
@SpringBootApplication
public class ManagerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ManagerApplication.class , args) ;
    }
}
```

3）、测试

```java
@Log(title = "角色添加", businessType = 0)	// 添加自定义Log注解，设置属性
@PostMapping(value = "/saveSysRole")
public Result saveSysRole(@RequestBody SysRole SysRole) {
    sysRoleService.saveSysRole(SysRole) ;
    return Result.build(null , ResultCodeEnum.SUCCESS) ;
}


// 结果
2023-07-19 14:09:32 [INFO ] com.zixieqing.spzx.common.aspect.LogAspect LogAspect...doAroundAdvice方法执行了角色添加
```







# 附加：Spring Boot线程池
## 场景

提高一下插入表的性能优化，两张表，先插旧的表，紧接着插新的表，若是一万多条数据就有点慢了。




## 使用步骤

用Spring提供的 `ThreadPoolExecutor` 封装的线程池 `ThreadPoolTaskExecutor` 直接使用注解启用




1. 配置

```java
@Configuration
@EnableAsync
public class ExecutorConfig {

    private static final Logger logger = LoggerFactory.getLogger(ExecutorConfig.class);

    @Value("${async.executor.thread.core_pool_size}")
    private int corePoolSize;
    @Value("${async.executor.thread.max_pool_size}")
    private int maxPoolSize;
    @Value("${async.executor.thread.queue_capacity}")
    private int queueCapacity;
    @Value("${async.executor.thread.name.prefix}")
    private String namePrefix;

    @Bean(name = "asyncServiceExecutor")
    public Executor asyncServiceExecutor() {
        logger.info("start asyncServiceExecutor");
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 配置核心线程数
        executor.setCorePoolSize(corePoolSize);
        // 配置最大线程数
        executor.setMaxPoolSize(maxPoolSize);
        // 配置队列大小
        executor.setQueueCapacity(queueCapacity);
        // 配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix(namePrefix);

        // rejection-policy：当pool已经达到max size的时候，如何处理新任务
        // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        //执行初始化
        executor.initialize();
        return executor;
    }
}
```


`@Value` 取值配置是在 `application.properties` 或 `application.yml` 中的，如：application.properties配置的

```properties
# 异步线程配置
# 配置核心线程数
async.executor.thread.core_pool_size = 5
# 配置最大线程数
async.executor.thread.max_pool_size = 5
# 配置队列大小
async.executor.thread.queue_capacity = 99999
# 配置线程池中的线程的名称前缀
async.executor.thread.name.prefix = async-service
```





2. Demo测试


- Service接口

```java
public interface AsyncService {

    /**
     * 执行异步任务
     * 可以根据需求，自己加参数拟定
     */
    void executeAsync();
}
```



- Service实现类

```java
@Service
public class AsyncServiceImpl implements AsyncService {

    private static final Logger logger = LoggerFactory.getLogger(AsyncServiceImpl.class);

    @Override
    @Async("asyncServiceExecutor")
    public void executeAsync() {
        logger.info("start executeAsync");

        System.out.println("异步线程要做的事情");
        System.out.println("可以在这里执行批量插入等耗时的事情");

        logger.info("end executeAsync");
    }
}
```


3. 在Controller层注入刚刚的Service即可

```java
@Autowired
private AsyncService asyncService;

@GetMapping("/async")
public void async(){
    asyncService.executeAsync();
}
```

使用测试工具测试即可看到相应的打印结果





## 摸索一下

弄清楚线程池当时的情况，有多少线程在执行，多少在队列中等待？

创建一个 `ThreadPoolTaskExecutor` 的子类，在每次提交线程的时候都将当前线程池的运行状况打印出来

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import org.springframework.util.concurrent.ListenableFuture;

import java.util.concurrent.Callable;
import java.util.concurrent.Future;
import java.util.concurrent.ThreadPoolExecutor;

public class VisiableThreadPoolTaskExecutor extends ThreadPoolTaskExecutor {


    private static final Logger logger = LoggerFactory.getLogger(VisiableThreadPoolTaskExecutor.class);

    private void showThreadPoolInfo(String prefix) {
        ThreadPoolExecutor threadPoolExecutor = getThreadPoolExecutor();

        if (null == threadPoolExecutor) {
            return;
        }

        logger.info("{}, {},taskCount [{}], completedTaskCount [{}], activeCount [{}], queueSize [{}]",
                this.getThreadNamePrefix(),
                prefix,
                threadPoolExecutor.getTaskCount(),
                threadPoolExecutor.getCompletedTaskCount(),
                threadPoolExecutor.getActiveCount(),
                threadPoolExecutor.getQueue().size());
    }

    @Override
    public void execute(Runnable task) {
        showThreadPoolInfo("1. do execute");
        super.execute(task);
    }

    @Override
    public void execute(Runnable task, long startTimeout) {
        showThreadPoolInfo("2. do execute");
        super.execute(task, startTimeout);
    }

    @Override
    public Future<?> submit(Runnable task) {
        showThreadPoolInfo("1. do submit");
        return super.submit(task);
    }

    @Override
    public <T> Future<T> submit(Callable<T> task) {
        showThreadPoolInfo("2. do submit");
        return super.submit(task);
    }

    @Override
    public ListenableFuture<?> submitListenable(Runnable task) {
        showThreadPoolInfo("1. do submitListenable");
        return super.submitListenable(task);
    }

    @Override
    public <T> ListenableFuture<T> submitListenable(Callable<T> task) {
        showThreadPoolInfo("2. do submitListenable");
        return super.submitListenable(task);
    }
}
```


进过测试发现： `showThreadPoolInfo` 方法中将任务总数、已完成数、活跃线程数，队列大小都打印出来了，然后Override了父类的execute、submit等方法，在里面调用showThreadPoolInfo方法，这样每次有任务被提交到线程池的时候，都会将当前线程池的基本情况打印到日志中


现在修改 `ExecutorConfig.java` 的 `asyncServiceExecutor` 方法，将 `ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor()` 改为 `ThreadPoolTaskExecutor executor = new VisiableThreadPoolTaskExecutor()`

```java
@Bean(name = "asyncServiceExecutor")
    public Executor asyncServiceExecutor() {
        logger.info("start asyncServiceExecutor");
        // 在这里进行修改
        ThreadPoolTaskExecutor executor = new VisiableThreadPoolTaskExecutor();
        // 配置核心线程数
        executor.setCorePoolSize(corePoolSize);
        // 配置最大线程数
        executor.setMaxPoolSize(maxPoolSize);
        // 配置队列大小
        executor.setQueueCapacity(queueCapacity);
        // 配置线程池中的线程的名称前缀
        executor.setThreadNamePrefix(namePrefix);

        // rejection-policy：当pool已经达到max size的时候，如何处理新任务
        // CALLER_RUNS：不在新线程中执行任务，而是有调用者所在的线程来执行
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        //执行初始化
        executor.initialize();
        return executor;
    }
```

经最后测试得到的结果：提交任务到线程池的时候，调用的是 `submit(Callable task)` 这个方法，当前已经提交了3个任务，完成了3个，当前有0个线程在处理任务，还剩0个任务在队列中等待













