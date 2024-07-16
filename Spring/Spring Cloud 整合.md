# 前言

玩SpringCloud之前最好懂SpringBoot，别搞撑死骆驼的事。Servlet整一下变成Spring；SSM封装、加入东西就变为SpringBoot；SpringBoot再封装、加入东西就变为SpringCloud



# 架构的演进

## 单体应用架构

**单体架构**：表示层、业务逻辑层和数据访问层即所有功能都在一个工程里，打成一个jar包、war包进行部署，例如：GitHub 是基于 Ruby on Rails 的单体架构，直到 2021 年，为了让超过一半的开发人员在单体代码库之外富有成效地开展工作，GitHub 以赋能为出发点开始了向微服务架构的迁移

下图服务器用Tomcat举例

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521164029004-1514387794.png" alt="image-20230521164028933" />



**优点：**

1. 单体架构开发简单，容易上手，开发人员只要集中精力开发当前工程
2. 容易修改，只需要修改对应功能模块的代码，且容易找到相关联的其他业务代码
3. 部署简单，由于是完整的结构体，编译打包成jar包或者war包，直接部署在一个服务器上即可
4. 容易扩展，可以将某些业务抽出一个新的单体架构，用于独立分担压力，也可以方便部署集群
5. 性能最高，对于单台服务器而言，单体架构独享内存和cpu，不需要api远程调用，性能损耗最小

**缺点：**

1. 灵活度不高，随着代码量增加，代码整体编译效率下降
2. 规模化，无法满足团队规模化开发，因为共同修改一个项目
3. 应用扩展性比较差，只能横向扩展，不能深度扩展，扩容只能只对这个应用进行扩容，不能做到对某个功能点进行扩容，关键性的代码改动一处多处会受影响
4. 健壮性不高，任何一个模块的错误均可能造成整个系统的宕机
5. 技术升级，如果想对技术更新换代，代价很大





## 演进：增加本地缓存和分布式缓存

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521164632922-534104745.png" alt="image-20230521164632822" />



缓存能够将经常访问的页面或信息存起来，从而不让其去直接访问数据库，从而减小数据库压力，但是：这就会把压力变成单机Tomcat来承受了，因此缺点就是：此时单机的tomcat又不足以支撑起高并发的请求





## 垂直应用架构：引入Nginx

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521170148624-1735985478.png" alt="image-20230521170148649" />



搭配N个tomcat，从而对请求"均衡处理"，如：如果Nginx可以处理10000条请求，假设一个 tomcat可以处理100个请求，那么：就需要100个tomcat从而实现每个tomcat处理100个请求(假设每个tomcat的性能都一样 )

缺点就是数据库不足以支撑压力

后面就是将数据库做读写分离

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521170535267-191396252.png" alt="image-20230521170535184" />



后面还有数据库大表拆小表、大业务拆为小业务、复用功能抽离..............





## 面向服务架构：SOA

SOA指的是Service-OrientedArchitecture，即面向服务架构

随着业务越来越多，代码越来越多，按照业务功能将本来一整块的系统拆分为各个不同的子系统分别提供不同的服务，服务之间会彼此调用，错综复杂

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521175435390-508194047.png" alt="image-20230521175435052" />



而SOA的思想就是基于前面拆成不同的服务之后，继续再抽离一层，搞一个和事佬，即下图的“统一接口”

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521224344850-262357819.png" alt="image-20230521224344141" />



这样不同服务之间调用就可以通过统一接口进行调用了，如：用户服务需要调用订单服务，那么用户服务去找统一接口，然后由统一接口去调用订单服务，从而将订单服务中需要的结果通过统一接口的http+json或其他两种格式返回给用户服务，这样订单服务就是服务提供者，用户服务就是服务消费者，而统一接口就相当于是服务的注册与发现

> 注意：上面这段话很重要，和后面要玩的微服务框架SpringCloud技术栈有关

学过设计模式的话，上面这种不就类似行为型设计模式的“中介者模式”吗

上面这种若是反应不过来，那拆回单体架构就懂了

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521230448878-888627647.png" alt="image-20230521230448349" />





## 微服务架构

微服务架构是分布式架构的具体实现方式，和Spring的IOC控制反转和DI依赖注入的关系一样，一种是理论，一种是具体实现方案

微服务架构和前面的SOA架构是孪生兄弟，即：微服务架构是在SOA架构的基础上，通过前人不断实践、不断踩坑、不断总结，添加了一些东西之后(如：链路追踪、配置管理、负载均衡............)，从而变出来的一种经过良好架构设计的**分布式架构方案**

而广泛应用的方案框架之一就是 [SpringCloud](https://spring.io/projects/spring-cloud)

其中常见的组件包括：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521231654377-404634780.png" alt="image-20210713204155887" />



另外，SpringCloud底层是依赖于SpringBoot的，并且有版本的兼容关系，如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521231719947-1707829950.png" alt="image-20210713205003790" />



因此。现在系统架构就变成了下面这样，当然不是一定是下面这样架构设计，还得看看架构师，看领导

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521232052531-1243047828.png" alt="image-20230521232051716" />



因此，微服务技术知识如下

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521232537360-774880611.png" alt="image-20230521232536647" />









# Eureka注册中心

SpringCloud中文官网：https://www.springcloud.cc/spring-cloud-greenwich.html#netflix-ribbon-starter

SpringCloud英文网：https://docs.spring.io/spring-cloud-netflix/docs/current/reference/html/#service-discovery-eureka-clients

## Eureka是什么？

Eureka是Netflix开发的服务发现框架，本身是一个基于REST的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。

SpringCloud将它集成在其子项目spring-cloud-netflix中，以实现SpringCloud的服务发现功能

偷张图更直观地了解一下：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230522222600452-117614713.png" alt="image-20210713220104956" />



如上图所示，服务提供方会将自己注册到EurekaServer中，这样EurekaServer就会存储各种服务信息，而服务消费方想要调用服务提供方的服务时，直接找EurekaServer拉取服务列表，然后根据特定地算法(轮询、随机......)，选择一个服务从而进行远程调用

- 服务提供方：一次业务中，被其它微服务调用的服务。（提供接口给其它微服务）
- 服务消费方：一次业务中，调用其它微服务的服务。（调用其它微服务提供的接口）

服务提供者与服务消费者的角色并不是绝对的，而是相对于业务而言

如果服务A调用了服务B，而服务B又调用了服务C，服务B的角色是什么？

- 对于A调用B的业务而言：A是服务消费者，B是服务提供者
- 对于B调用C的业务而言：B是服务消费者，C是服务提供者

因此，服务B既可以是服务提供者，也可以是服务消费者





## Eureka的自我保护机制

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230522223650571-1467351950.png" alt="image-20210713220104956" />



这张图中EurekaServer和服务提供方有一个心跳检测机制，这是EurekaServer为了确定这些服务是否还在正常工作，所以进行的心跳检测

eureka-client启动时， 会开启一个心跳任务，向Eureka Server发送心跳，默认周期为30秒/次，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除(默认90秒)

eureka-server维护了每个实例的最后一次心跳时间，客户端发送心跳包过来后，会更新这个心跳时间

eureka-server启动时，开启了一个定时任务，该任务每60s/次，检查每个实例的最后一次心跳时间是否超过90s，如果超过则认为过期，需要剔除



但是EurekaClient也会因为网络等原因导致没有及时向EurekaServer发送心跳，因此EurekaServer为了保证误删服务就会有一个“自我保护机制”，俗称“好死不如赖活着”

如果在短时间内EurekaServer丢失过多客户端时 (可能断网了，**低于85%的客户端节点都没有正常的心跳** )，那么Eureka Server就认为客户端与注册中心出现了网络故障，Eureka Server自动进入自我保护状态 。Eureka的这样设计更加精准地控制是网络通信延迟，而不是服务挂掉了，一旦进入自我保护模式，那么 EurekaServer就会保留这个节点的属性，不会删除，直到这个节点恢复正常心跳 

- 85% 这个阈值，可以通过如下配置来设置：

```yaml
eureka:
  server:
    renewal-percent-threshold: 0.85
```

这里存在一个问题，这个85%是超过谁呢？这里有一个预期的续约数量，计算公式如下:

```txt
自我保护阀值 = 服务总数 * 每分钟续约数(60S/客户端续约间隔) * 自我保护续约百分比阀值因子
```

在自我保护模式中，EurekaServer会保留注册表中的信息，不再注销任何服务信息，当它收到正常心跳时，才会退出自我保护模式，也就是：**宁可保留错误的服务注册信息，也不会盲目注销任何可能健康的服务实例，即：好死不如赖活着** 



因此Eureka进入自我保护状态后，会出现以下几种情况:

- Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上，保证当前节点依然可用。Eureka的自我保护机制可以通过如下的方式开启或关闭

```yaml
eureka:
  server:
#   开启Eureka自我保护机制，默认为true
    enable-self-preservation: true
```

- Eureka Server不再从注册列表中移除因为长时间没有收到心跳而应该剔除的过期服务，如果在保护期内这个服务提供者刚好非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等！











## Eureka常用配置

```yaml
eureka:
  client: # eureka客户端配置
    register-with-eureka: true # 是否将自己注册到eureka服务端上去
    fetch-registry: true # 是否获取eureka服务端上注册的服务列表
    service-url:
      defaultZone: http://localhost:8001/eureka/ # 指定注册中心地址。若是集群可以写多个，中间用 逗号 隔开
    enabled: true # 启用eureka客户端
    registry-fetch-interval-seconds: 30 # 定义去eureka服务端获取服务列表的时间间隔
  instance: # eureka客户端实例配置
    lease-renewal-interval-in-seconds: 30 # 定义服务多久去注册中心续约
    lease-expiration-duration-in-seconds: 90 # 定义服务多久不去续约认为服务失效
    metadata-map:
      zone: hangzhou # 所在区域
    hostname: localhost # 服务主机名称
    prefer-ip-address: false # 是否优先使用ip来作为主机名
  server: # eureka服务端配置
    enable-self-preservation: false #关 闭eureka服务端的自我保护机制
```









## 使用Eureka

实现如下的逻辑：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523105026659-571765101.png" alt="image-20230523105025549" />



### 搭建Eureka Server

自行单独创建一个Maven项目，导入依赖如下：

```xml
<!--Eureka Server-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```



在YAML文件中一般可配置内容如下：

```yaml
server:
  port: 10086
spring:
  application:
    name: EUREKA-SERVER
eureka:
  instance:
    # Eureka的主机名，是为了eureka集群服务器之间好区分
    hostname: 127.0.0.1
    # 最后一次心跳后，间隔多久认定微服务不可用，默认90
    lease-expiration-duration-in-seconds: 90
  client:
    # 不向注册中心注册自己。应用为单个注册中心设置为false，代表不向注册中心注册自己，默认true	注册中心不需要开启
    # registerWithEureka: false
    # 不从注册中心拉取自身注册信息。单个注册中心则不拉取自身信息，默认true	注册中心不需要开启
    # fetchRegistry: false
    service-url:
      # Eureka Server的地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
#    server:
#      # 开启Eureka自我保护机制，默认为true
#      enable-self-preservation: true
```



> 注：在SpringCloud中配置文件YAML有两种方式，一种是 `application.yml ` 另一种是 `bootstrap.yml `，这个知识后续Nacos注册中心会用到，区别去这里：https://www.cnblogs.com/sharpest/p/13678443.html





启动类编写内容如下：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * <p>@description  : 该类功能  eureka server启动类
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

/*@EnableEurekaServer 开启Eureka Server功能*/
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```







### 服务提供者

新建一个Maven模块项目，依赖如下：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
<!--eureka client-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



YAML配置内容如下：

```yaml
server:
  port: 8081
spring:
  application:
    name: USER-SERVICE
eureka:
  client:
    service-url:
      # 将服务注册到哪个eureka server
      defaultZone: http://localhost:10086/eureka
```



启动类内容如下：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```



#### 关于开启Eureka Client的问题

上一节中启动类里面有些人会看到是如下的方式：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient // 多了这么一个操作：开启eureka client功能
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```

在eureka client启动类中，有些人会加 `@EnableEurekaClient` 注解，而有些人不会加上，为什么？除了显示声明还有其他原因吗？？

要弄这个问题，首先看yml中的配置，有些是在yml中做了一个操作：

```yaml
eureka:
  client:
    service-url:
      # 向哪个eureka server进行服务注册
      defaultZone: http://localhost:10086/eureka
    # 开启eureka client功能，默认就是true，差不多等价于启动类中加 @EnableEurekaClient 注解
    enabled: true
```

既然上面配置默认值都是true，那还有必要在启动类中加入 `@EnableEurekaClient` 注解吗？

答案是根本不用加，加了也是多此一举(前提：yml配置中没有手动地把值改为false)，具体原因看源码：答案就在Eureka client对应的自动配置类 `EurekaClientAutoConfiguration` 中

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523140658133-980195700.png" alt="image-20230523140656713" />

上图中这一行的意思是只有当application.yaml（或者环境变量，或者系统变量）里，`eureka.client.enabled`这个属性的值为`true`才会初始化这个类（如果手动赋值为false，就不会初始化这个类了）



另外再加上另一个原因，同样在 `EurekaClientAutoConfiguration` 类中还有一个 `eurekaAutoServiceRegistration()` 方法

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523141137735-1222105186.png" alt="image-20230523141136544" />



在这里使用 `EurekaAutoServiceRegistration类+@Bean注解` 意思就是通过 `@Bean` 注解，装配一个 EurekaAutoServiceRegistration 对象作为Spring的bean，而我们从名字就可以看出来EurekaClient的注册就是 EurekaAutoServiceRegistration 对象所进行的操作

同时，在这个方法上，也有这么一行 `@ConditionalOnProperty(value = "spring.cloud.service-registry.auto-registration.enabled", matchIfMissing = true)` 



综上所述：我们可以看出来，EurekaClient的注册和两个配置项有关的，一个是 `eureka.client.enabled` ，另一个是 `spring.cloud.service-registry.auto-registration.enabled` ，只不过这两个配置默认都是true。这两个配置无论哪个我们手动配置成false，我们的服务都无法进行注册，测试自行做





另外还有一个原因：上图中不是提到了 `EurekaAutoServiceRegistration类+@Bean注解` 吗，那去看一下

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523142607917-175927233.png" alt="image-20230523142606183" />



可以看到 `EurekaAutoServiceRegistration `类实现了Spring的 `SmartLifecycle `接口，这个接口的作用是帮助一个类在作为Spring的Bean的时候，由Spring帮助我们自动进行一些和生命周期有关的工作，比如在初始化或者停止的时候进行一些操作。而我们最关心的 `注册(register)` 这个动作，就是在SmartLifecycle接口的 `start()` 方法实现里完成的



而上一步讲到，`EurekaAutoServiceRegistration `类在 `EurekaClientAutoConfiguration `类里恰好被配置成Spring的Bean，所以这里的 `start()` 方法是会自动被Spring调用的，我们不需要进行任何操作





##### 总结

当我们引用了EurekaClient的依赖后，并且  `eureka.client.enabled` 和 `spring.cloud.service-registry.auto-registration.enabled` 两个开关不手动置为false，Spring就会自动帮助我们执行 `EurekaAutoServiceRegistration` 类里的 `start()` 方法，而注册的动作就是在该方法里完成的



**所以，我们的EurekaClient工程，并不需要显式地在SpringBoot的启动类上标注 `@EnableEurekaClient` 注解**









### 服务消费者

创建Maven模块，依赖如下：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



YAML配置如下：

```yaml
server:
  port: 8080
spring:
  application:
    name: ORDER-SERVICE
eureka:
  client:
    service-url:
      # 向哪个eureka server进行服务拉取
      defaultZone: http://localhost:10086/eureka
```





启动类如下：

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class OrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }

    /**
     * RestTemplate 用来进行远程调用服务提供方的服务
     * LoadBalanced 注解 是SpringCloud中的
     *              此处作用：赋予RestTemplate负载均衡的能力 也就是在依赖注入时，只注入实例化时被@LoadBalanced修饰的实例
     *              底层是 Spring的Qualifier注解，即为spring的原生操作
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
```



`@Qualifier` 注解很重要：

> @Autowired 默认是根据类型进行注入的，因此如果有多个类型一样的Bean候选者，则需要限定其中一个候选者，否则将抛出异常
>
> @Qualifier 限定描述符除了能根据名字进行注入，更能进行更细粒度的控制如何选择候选者



`@LoadBalanced`很明显，"继承"了注解`@Qualifier`，`RestTemplates`通过`@Autowired`注入，同时被`@LoadBalanced`修饰，所以只会注入`@LoadBalanced`修饰的`RestTemplate`，也就是我们的目标`RestTemplate`







通过 RestTemplate +eureka 远程调用服务提供方中的服务

```java
import com.zixieqing.order.mapper.OrderMapper;
import com.zixieqing.order.pojo.Order;
import com.zixieqing.order.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;

    @Autowired
    private RestTemplate restTemplate;

    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        // 2、远程调用服务的url 此处直接使用服务名，不用ip+port
        // 原因是底层有一个LoadBalancerInterceptor，里面有一个intercept()，后续玩负载均衡Ribbon会看到
        String url = "http://USER-SERVICE/user/" + order.getUserId();
        // 2.1、利用restTemplate调用远程服务，封装成user对象
        User user = restTemplate.getForObject(url, User.class);
        // 3、给oder设置user对象值
        order.setUser(user);
        // 4.返回
        return order;
    }
}
```



不会玩 RestTemplate 用法的 [戳这里](https://blog.csdn.net/weixin_43888891/article/details/125649613?spm=1001.2014.3001.5501)







### 测试

依次启动eureka-server、user-service、order-service，然后将user-service做一下模拟集群即可，将user-service弄为模拟集群操作方式如下：不同版本IDEA操作有点区别，出入不大

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523113543880-720911336.png" alt="image-20230523113542449" />





<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523113729320-191314494.png" alt="image-20230523113728396" />



再将复刻的use-service2也启动即可，启动之后点一下eureka-server的端口就可以在浏览器看到服务情况

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523114005904-1151512200.png" alt="image-20230523114005087" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523114155475-1176148165.png" alt="image-20230523114153992" />



可以自行在服务提供方和服务消费方编写逻辑，去链接数据库，然后在服务消费方调用服务提供方的业务，最后访问自己controller中定义的路径和参数即可

















# Ribbon负载均衡

## 负载均衡的作用范围

负载均衡是作用在网络通信上，来实现请求的分发。 
而在网络架构中，基于 OSI 模型，又分为 7 层网络模型 

![image-20240322180159227](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240322180100467-1442743838.png)



也就是意味着我们可以在网络的某些分层上做请求分发处理，因此根据这样一个特性，对于负载均衡的作用范围又可以分为： 

1. 二层负载 
2. 三层负载 
3. 四层负载 
4. 七层负载 

 

**二层负载**：基于 Mac 地址来实现请求分发，一般采用虚拟 Mac 的方式实现，服务器收到请求后，通过动态分配后端服务的实际 Mac 地址进行响应，从而实现负载均衡 
**三层负载**：基于 IP 层负载，一般通过虚拟 IP 的方式实现，外部请求访问虚拟 IP，服务器收到请求后根据后端实际 IP 地址进行转发。 
**四层负载**：通过请求报文中的目标地址和端口进行负载，Nginx、F5、LVS 等都可以实现四层负载。

![image-20240322180302972](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240322180203426-1235270774.png)



**七层负载**：七层负载是基于应用层负载，也就是服务器端可以根据 http 协议中请求的报文信息来决定把请求分发到哪个目标服务器上，比如 Cookie、消息体、RequestHeader 等。

![image-20240322180332919](https://img2023.cnblogs.com/blog/2421736/202403/2421736-20240322180233386-2085177871.png)







## Ribbon是什么？

Ribbon是Netflix发布的开源项目，`Spring Cloud Ribbon`是基于`Netflix Ribbon`实现的一套客户端`负载均衡`的框架



## Ribbon属于哪种负载均衡？

**LB负载均衡(Load Balance)是什么？**

- 简单地说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）
- 常见的负载均衡有软件Nginx，硬件 F5等





**什么情况下需要负载均衡？**

- 现在Java非常流行微服务，也就是所谓的面向服务开发，将一个项目拆分成了多个项目，其优点有很多，其中一个优点就是：将服务拆分成一个一个微服务后，我们很容易地来针对性的进行集群部署。例如订单模块用的人比较多，那就可以将这个模块多部署几台机器，来分担单个服务器的压力

- 这时候有个问题来了，前端页面请求的时候到底请求集群当中的哪一台？既然是降低单个服务器的压力，所以肯定全部机器都要利用起来，而不是说一台用着，其他空余着。这时候就需要用负载均衡了，像这种前端页面调用后端请求的，要做负载均衡的话，常用的就是Nginx





**Ribbon和Nginx负载均衡的区别**

- 当后端服务是集群的情况下，前端页面调用后端请求，要做负载均衡的话，常用的就是Nginx
- Ribbon主要是在“服务端内”做负载均衡，举例：订单后端服务 要调用 支付后端服务，这属于后端之间的服务调用，压根根本不经过页面，而后端支付服务是集群，这时候订单服务就需要做负载均衡来调用支付服务，记住是订单服务做负载均衡 “来调用” 支付服务





**负载均衡分类**

-  **集中式LB**：即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx)，由该设施负责把访问请求通过某种策略转发至服务的提供方
- **进程内LB**：将LB逻辑集成到“消费方”，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器





**Ribbon负载均衡**

- Ribbon就属于进程内LB，它只是一个类库，集成于**服务消费方进程**







## Ribbon的流程

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523150221319-1099794917.png" alt="image-20230523150220629" />



通过上图一定要明白一点：**Ribbon一定是用在消费方，而不是服务的提供方！**





**Ribbon在工作时分成两步（这里以Eureka为例，consul和zk同样道理）：**

- 第一步先选择 EurekaServer ，它优先选择在同一个区域内负载较少的server
- 第二步再根据用户指定的策略(轮询、随机、响应时间加权.....)，从server取到的服务注册列表中选择一个地址







## 请求怎么从服务名地址变为真实地址的？

只要引入了注册中心(Eureka、consul、zookeeper)，那Ribbon的依赖就在注册中心里面了，证明如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523150714289-322784310.png" alt="image-20230523150713088" />







回到正题：为什么下面这样使用服务名就可以调到服务提供方的服务，即：请求 http://userservice/user/101 怎么变成的 http://localhost:8081 ？？因为它长得好看？

```java
import com.zixieqing.order.mapper.OrderMapper;
import com.zixieqing.order.pojo.Order;
import com.zixieqing.order.pojo.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;

    @Autowired
    private RestTemplate restTemplate;

    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        // 2、远程调用服务的url 此处直接使用服务名，不用ip+port
        // 原因是底层有一个LoadBalancerInterceptor，里面有一个intercept()，后续玩负载均衡Ribbon会看到
        String url = "http://USER-SERVICE/user/" + order.getUserId();
        // 2.1、利用restTemplate调用远程服务，封装成user对象
        User user = restTemplate.getForObject(url, User.class);
        // 3、给oder设置user对象值
        order.setUser(user);
        // 4.返回
        return order;
    }
}


// RestTemplate做了下面操作，使用了 @Bean+@LoadBalanced


    /**
     * RestTemplate 用来进行远程调用服务提供方
     * LoadBalanced 注解 是SpringCloud中的
     *              此处作用：赋予RestTemplate负载均衡的能力 也就是在依赖注入时，只注入实例化时被@LoadBalanced修饰的实例
     *              底层是 Spring的Qualifier注解，即为spring的原生操作
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
```

想知道答案就得Debug了，而要Debug，就得找到 `LoadBalancerInterceptor` 类





### LoadBalancerInterceptor类

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523164303496-89314463.png" alt="image-20230523164301233" />



然后对服务消费者进行Debug

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523164749307-84514600.png" alt="image-20230523164748273" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523164906453-461257891.png" alt="image-20230523164905276" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523165134892-1027704730.png" alt="image-20230523165133615" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523165333560-193867768.png" alt="image-20230523165332402" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523170045156-1643577053.png" alt="image-20230523170043376" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523170133632-1871497916.png" alt="image-20230523170132894" />





<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523171130565-852624319.png" alt="image-20230523171129379" />





<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523171314721-632166842.png" alt="image-20230523171313688" />







<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523171517502-1987407255.png" alt="image-20230523171516222" />







问题的答案已经出来了：**为什么使用服务名就可以调到服务提供方的服务，即：请求 http://userservice/user/101 怎么变成的 http://localhost:8081 ？？**

- 原因就是使用了RibbonLoadBalancerClient+loadBalancer(默认是 ZoneAwareLoadBalance 从服务列表中选取服务)+IRule(默认是 RoundRobinRule 轮询策略选择某个服务)

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523172627992-1458093408.png" alt="image-20230523172623741" />









### 总结

SpringCloudRibbon的底层采用了一个拦截器LoadBalancerInterceptor，拦截了RestTemplate发出的请求，对地址做了修改

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523183515709-898697749.png" alt="image-20230523183514694" />









## 负载均衡策略有哪些？

根据前面的铺垫，也知道了负载均衡策略就在 `IRule` 中，那就去看一下

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523183831369-932003270.png" alt="image-20230523183830372" />



转换一下：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523183903273-919770286.png" alt="image-20210713225653000" />



`ClientConfigEnabledRoundRobinRule`：该策略较为特殊，我们一般不直接使用它。因为它本身并没有实现什么特殊的处理逻辑。一般都是可以通过继承他重写一些自己的策略，默认的choose()就实现了线性轮询机制

- `BestAvailableRule`：继承自ClientConfigEnabledRoundRobinRule，会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务，该策略的特性是可选出最空闲的实例



`PredicateBasedRule`：继承自ClientConfigEnabledRoundRobinRule，抽象策略，需要重写方法，然后自定义过滤规则

- `AvailabilityFilteringRule`：继承PredicateBasedRule，先过滤掉故障实例,再选择并发较小的实例。过滤掉的故障服务器是以下两种：
	1. 在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加
	2. 并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的`<clientName>.<clientConfigNameSpace>.ActiveConnectionsLimit` 属性进行配置
- `ZoneAvoidanceRule`：继承PredicateBasedRule，默认规则，复合判断server所在区域的性能和server的可用性选择服务器



`com.netflix.loadbalancer.RoundRobinRule`：轮询 Ribbon的默认规则

- `WeightedResponseTimeRule`：对RoundRobinRule的扩展。为每一个服务器赋予一个权重值，服务器响应时间越长，其权重值越小，这个权重值会影响服务器的选择，即：响应速度越快的实例选择权重越大,越容易被选择
- `ResponseTimeWeightedRule`：对RoundRobinRule的扩展。响应时间加权



`com.netflix.loadbalancer.RandomRule`：随机

`com.netflix.loadbalancer.StickyRule`：这个基本也没人用

`com.netflix.loadbalancer.RetryRule`：先按照RoundRobinRule的策略获取服务,如果获取服务失败则在指定时间内会进行重试，从而获取可用的服务

`ZoneAvoidanceRule`：先复合判断server所在区域的性能和server的可用性选择服务器，再使用Zone对服务器进行分类，最后对Zone内的服务器进行轮询











## 自定义负载均衡策略

在前面已经知道了策略是 `IRule` ，所以就是改变了这个玩意而已



**1、代码方式** ：服务消费者的启动类或重开config模块编写如下内容即可

```java
@Bean
public IRule randomRule(){
    // new前面提到的那些rule对象即可，当然这里面也可以自行篡改策略逻辑返回
    return new RandomRule();
}
```

**注：** 此种方式是全局策略，即所有服务均采用这里定义的负载均衡策略



**2、@RibbonClient注解**：用法如下

```java
/**
 * 在服务消费者的启动类中加入如下注解即可 如下注解指的是：调用 USER-SERVICE 服务时 使用MySelfRule负载均衡规则
 *
 * 这里的MySelfRule可以弄为自定义逻辑的策略，也可以是前面提到的那些rule策略
 */
@RibbonClient(name = "USER-SERVICE",configuration=MySelfRule.class)
```

这种方式可以达到只针对某服务做负载均衡策略，但是：**官方给出了明确警告** `configuration=MySelfRule.class` 自定义配置类一定不能放到@ComponentScan 所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的`Ribbon`客户端所共享，达不到特殊化定制的目的了（也就是一旦被扫描到，RestTemplate直接不管调用哪个服务都会用指定的算法）

> springboot项目当中的启动类使用了@SpringBootApplication注解，这个注解内部就有@ComponentScan注解，默认是扫描启动类包下所有的包，所以我们要达到定制化一定不要放在它能扫描到的地方



cloud中文官网：https://www.springcloud.cc/spring-cloud-greenwich.html#netflix-ribbon-starter



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523193845845-309027790.png" alt="image-20230523193844609" />





**3、使用YAML配置文件方式** 在服务消费方的yml配置文件中加入如下格式的内容即可

```yml
# 给某个微服务配置负载均衡规则，这里是user-service服务
user-service: 
  ribbon:
    # 负载均衡规则
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```



> **注意**，一般用默认的负载均衡规则，不做修改









## Ribbon饿汉加载

Ribbon默认是采用懒加载，即第一次访问时才会去创建LoadBalanceClient，请求时间会很长。

而饿汉加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载：

```yaml
ribbon:
  eager-load:
    # 开启负载均衡饿汉加载模式
    enabled: true
    # clients是一个String类型的List数组，多个时采用下面的 - xxxx服务 的形式，单个时直接使用 clients: 服务名 即可
    clients:
      - USER-SERVICE
```











# Nacos注册中心

国内公司一般都推崇阿里巴巴的技术，比如注册中心，SpringCloudAlibaba也推出了一个名为Nacos的注册中心

[Nacos](https://nacos.io/) 是阿里巴巴的产品，现在是 [SpringCloud](https://spring.io/projects/spring-cloud) 中的一个组件。相比 [Eureka](https://github.com/Netflix/eureka) 功能更加丰富，在国内受欢迎程度较高







## 安装Nacos

### windows安装

GitHub中下载：https://github.com/alibaba/nacos/releases

下载好之后直接解压即可，但：别解压到有“中文路径”的地方

Nacos的默认端口是8848，若该端口被占用则关闭该进程 或 修改nacos中的默认端口(conf/application.properties)



启动Nacos：密码和账号均是 nacos

```txt
startup.cmd -m standalone


-m 				mode 模式
standalone		单机
```







### Linux安装

Nacos是基于Java开发的，所以需要JDK支持，因此Linux中需要有JDK环境

上传Linux版的JDK

```shell
# 解压
tar -xvf jdk-8u144-linux-x64.tar.gz

# 配置环境变量
export JAVA_HOME=/usr/local/java			# =JDK解压后的路径
export PATH=$PATH:$JAVA_HOME/bin

# 刷新环境变量
source /etc/profile
```

上传Linux版的Nacos

```shell
# 解压
tar -xvf nacos-server-1.4.1.tar.gz

# 进入 nacos/bin 目录中，输入命令启动Nacos
sh startup.sh -m standalone

# 有8848端口冲突和windows中一样方式解决
```





## 注册服务到Nacos中

拉取Nacos的依赖管理，服务端加入如下依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
  <version>2.2.5.RELEASE</version>
  <type>pom</type>
  <scope>import</scope>
</dependency>
```

客户端依赖如下：

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

**注：**不要有其他注册中心的依赖，如前面玩的Eureka，有的话注释掉





修改客户端的yml配置文件：

```yaml
server:
  port: 8081
spring:
  application:
    name: USER-SERVICE
  cloud:
    nacos:
      # Nacos服务器地址
      server-addr: localhost:8848
#eureka:
#  client:
#    # 去哪里拉取服务列表
#    service-url:
#      defaultZone: http://localhost:10086/eureka
```



启动之后，在 ip+port/nacos 就在Nacos控制台看到信息了

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524172641663-1724265961.png" alt="image-20230524172640484" />





## Nacos集群配置与负载均衡策略调整

**1、集群配置**：Nacos的服务多级存储模型和其他的不一样

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524173247256-1640638181.png" alt="image-20230524173246752" />



就多了一个集群，不像其他的是 服务-----> 实例

好处：微服务互相访问时，应该**尽可能访问同集群实例，因为本地访问速度更快。当本集群内不可用时，才访问其它集群**





配置服务集群：想要对哪个服务配置集群则在其yml配置文件中加入即可

```yaml
server:
  port: 8081
  application:
    name: USER-SERVICE
  cloud:
    nacos:
      # Nacos服务器地址
      server-addr: localhost:8848
      # 配置集群名称，如：HZ，杭州
      cluster-name: HZ
```

测试则直接将“服务提供者”复刻多份，共用同一集群名启动，然后再复刻修改集群名启动即可，如下面的：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524174420790-807433895.png" alt="image-20230524174419882" />







**2、负载均衡策略调整**：前面玩Ribbon时已经知道了默认是轮询策略，而想要达到Nacos的 **尽可能访问同集群实例，因为本地访问速度更快。当本集群内不可用时，才访问其它集群** 的功能，则就需要调整负载均衡策略，配置如下：

```yaml
USER-SERVICE:
  ribbon:
    # 单独对某个服务设置负载均衡策略
#    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RoundRobinRule
    # 改为Naocs的负载均衡策略
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule
```

**注：** 再次说明前面提到的 ------> 负载均衡策略调整放在“服务消费方”

经过上面的配置之后，服务消费方去调用服务提供方的服务时，会优先选择和服务消费方同集群下的服务提供方的服务，若无法访问才跨集群访问其他集群下的服务提供方得到服务

- **小细节：** 服务消费方访问同集群下服务提供方的服务时(提供方是集群，多实例)，选择这些实例中的哪一个服务时并不是采用轮询了，而是随机



另外的负载均衡策略就是Ribbon中的：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524184810939-1621404096.png" alt="image-20230524184809397" />





**3、加权策略** ：服务器权重值越高，越容易被选择，所以能者多劳，性能好的服务器被访问的次数应该越多

权重值一般在 [0,10000] 之间。直接去Nacos的控制台中选择想要修改权重值的服务，点击“详情”即可修改



> **注：** 当权重值为0时，代表此服务实例不会再被访问，类似于停机迭代



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524200355093-646170701.png" alt="image-20230524200353921" />





## Nacos环境隔离

前面一节见到了Nacos的集群结构，但那只是较内的一层，**Nacos不止是注册中心，也可以是数据中心**

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525115608477-2097480414.png" alt="image-20230525115608614" />



- **namespace** ：就是环境隔离，如 dev开发环境、test测试环境、prod生产环境。若没配置，则默认是public，在没有指定命名空间时都会默认从`public`这个命名空间拉取配置以及注册到该命名空间下的注册表中。什么是注册表在后续看源码时会说明
- **group** ：就是在namespace的基础上，再进行分组，就是平时理解的分组，如 将服务相关性强的分在一个组
- **service ----> clusters -----> instances** ：就是前面说的集群，服务 ----> 集群 ------> 实例







**配置namespace：** 注意事项如下

1.  同名的命名空间只能创建一个
2. 微服务间如果没有注册到一个命名空间下，无法使用OpenFeign指定服务名负载通信（服务拉取的配置文件不同命名空间不影响）。Feign是后面要玩的

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525120133562-1501134563.png" alt="image-20230525120134073" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525120229108-2011422258.png" alt="image-20230525120229740" />





<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525120255381-567965134.png" alt="image-20230525120255821" />





**在yml配置文件中进行环境隔离配置**

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      cluster-name: HZ
      # 环境隔离：即当前这个服务要注册到哪个命名空间环境去
      # 值为在Nacos控制台创建命名空间时的id值，如下面的dev环境
      namespace: e7144264-0bf4-4caa-a17d-0af8e81eac3a
```











## Nacos临时与非临时实例

**1、Nacos和Eureka的不同：**不同在下图字体加粗的部分，加粗是Nacos具备而Eureka不具备的

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525141447179-1876206647.png" alt="image-20230525141447350" />



**临时实例：** 由服务提供者主动给Nacos发送心跳情况，在规定时间内要是没有发送，则Nacos认为此服务挂了，就会从服务列表中踢掉（非亲儿子）

**非临时实例/永久实例：**由Nacos主动来询问服务是否还健康、活着(此种实例会让服务器压力变大)，若非临时实例挂了，Naocs并不会将其踢掉（亲儿子）

1. 临时实例：Nacos官网 https://nacos.io/zh-cn/docs/open-api.html 中的“服务发现”的“发送实例心跳”中可以看到源码是在什么地方找

   - **适合**：流量激增时使用(高并发故增加更多实例)，后续流量下降了这些实例就可以不要了
   - 采用客户端心跳检测模式，心跳周期5秒

   - 心跳间隔超过15秒则标记为不健康

   - 心跳间隔超过30秒则从服务列表删除

2. 永久实例：

   - **适合**：常备实例
   - 采用服务端主动健康检测方式

   - 周期为2000 + 5000，即[2000, 7000]毫秒内的随机数

   - 检测异常只会标记为不健康，不会删除



**push：**若是Nacos检测到有服务提供者挂了，就会主动给消费者发送服务变更的消息，然后服务消费者更新自己的服务缓存列表。这一步就会让服务列表更新很及时

- 此方式是Nacos具备而Eureka不具备的，Eureka只有pull操作，因此Eureka的缺点就是服务更新可能会不及时(在30s内，服务提供者变动了，个别挂了，而消费者中的服务缓存列表还是旧的，只能等到30s到了才去重新pull)



Nacos的服务发现分为两种模式：

- 模式一：主动拉取模式(push模式)，消费者定期主动从Nacos拉取服务列表并缓存起来，再服务调用时优先读取本地缓存中的服务列表
- 模式二：订阅模式(pull模式)，消费者订阅Nacos中的服务列表，并基于UDP协议来接收服务变更通知。当Nacos中的服务列表更新时，会发送UDP广播给所有订阅者

查看服务发现源码的地方：后续也会介绍

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230712233546759-1646042323.png" alt="image-20230712233544072" />





**Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式**



**补充：CAP定理** 这是分布式事务中的一个方法论

1. **C**	即：Consistency **数据一致性**。指的是：**用户访问分布式系统中的任意节点，得到的数据必须一致**
2. **A**	即：Availability **可用性**。指的是：**用户访问集群中的任意健康节点，必须能得到响应，而不是超时或拒绝**
3. **P**	即：Partition Tolerance **分区容错性**。指的是：**由于某种原因导致系统中任意信息的丢失或失败都不能影响系统的继续独立运作**



**注：** 分区容错性是必须满足的，数据一致性( C ）和 可用性（ A ）只满足其一即可，一般的搭配是如下的**（即：取舍策略）：**

1. CP			保证数据的准确性
2. AP			保证数据的及时性



既然CAP定理都整了，那就再加一个**Base理论**吧，这个理论是对CAP中C和A这两个矛盾点的调和和选择

1. **BA**	 即：Basically Available **基本可用性**。指的是：**在发生故障的时候，可以允许损失“非核心部分”的可用性，保证系统正常运行，即保证核心部分可用**
2. **S**	   即：Soft State **软状态**。指的是：**允许系统的数据存在中间状态，只要不影响整个系统的运行就行**
3. **E**	   即：Eventual Consistency **最终一致性**。指的是：**无论以何种方式写入数据库 / 显示出来，都要保证系统最终的数据是一致的**





**2、配置临时实例与非临时实例：**在需要的一方的yml配置文件中配置如下开关即可

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848
      cluster-name: HZ
      # 默认为true，即临时实例
      ephemeral: false
```

改完之后可以在Nacos控制台看到服务是否为临时实例

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525142657387-321996528.png" alt="image-20230525142657931" />







## Nacos统一配置管理

**统一配置管理：** 将容易发生改变的配置单独弄出来，然后在后续需要变更时，直接去统一配置管理处进行更改，这样凡是依赖于这些配置的服务就可以统一被更新，而不用挨个服务更改配置，同时更改配置之后不用重启服务，直接实现热更新

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525194142989-983680569.png" alt="image-20230525194143607" />



Nacos和SpringCloud原生的config不一样，Nacos是将 注册中心+config 结合在一起了，而SpringCloud原生的是Eureka+config









**1、设置Nacos配置管理**

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525200157228-518484269.png" alt="image-20230525200157809" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525205325093-460315993.png" alt="image-20230525205325049" />



<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525200923433-1263014836.png" alt="image-20230525200924326" />



以上便是在Nacos中设置了统一配置。但是：项目/服务想要得到这些配置，那就得获取到这些配置，怎么办？

在前面说过SpringCloud中有两种yml的配置方式，一种是 `application.yml` ，一种是 `bootstrap.yml` ，这里就需要借助后者了，它是引导文件，优先级比前者高，会优先被加载，这样就可以先使用它加载到Nacos中的配置文件，然后再读取 `application.yml` ，从而完成Spring的那一套注册实例的事情

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525201257229-1149037588.png" alt="image-20230525201257171" />



**2、在需要读取Nacos统一配置的服务中引入如下依赖：**

```xml
<!--nacos配置管理依赖-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```





**3、resources下新建 bootstrap.yml，里面的配置内容如下**

```yml
spring:
  application:
    # 服务名，对应在nacos中进行配置管理的data id的服务名
    name: userservice
  profiles:
    # 环境，对应在nacos中进行配置管理的data id的环境
    active: dev
  cloud:
    nacos:
      # nacos服务器地址，需要知道去哪里拉取配置信息
      server-addr: localhost:8848
    config:
      # 文件后缀，对应在nacos中进行配置管理的data id的后缀名
      file-extension: yaml
```



<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230630172916153-1344503629.png" alt="image-20230630172914571" />



经过上面的操作之后，以前需要单独在 `application.yml` 改的事情就不需要了，`bootstrap.yml` 配置的东西会去拉取nacos中的配置

还有一种引入nacos统一配置的方式：

```yaml
spring:
  cloud:
    nacos:
      config:
        server-addr: localhost:8848
  config:
    import:
      - nacos:userservice-dev.yaml
```





**4、设置热更新：** 假如业务代码中有需要用到nacos中的配置信息，那nacos中的配置改变之后，不需要重启服务，自动更新。一共有两种方式

1. **`@RefreshScope+@Value`注解：** 在 @Value 注入的变量**所在类上**添加注解 @RefreshScope

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525205534211-29938183.png" alt="image-20230525205534523" />

2. `@ConfigurationProperties` 注解

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525210115994-1555810348.png" alt="image-20230525210116200" />



然后在需要的地方直接注入对象即可

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525210203864-778691054.png" alt="image-20230525210204143" />









## Nacos多环境共享配置

有时会遇到这样的情况：生产环境、开发环境、测试环境有些配置是相同的，这种应该不需要在每个环境中都配置，因此需要让这些相同的配置单独弄出来，然后实行共享

在前面一节中已经说到了一种Nacos的配置文件格式 即 `服务名-环境.后缀`，除了这种还有一种格式 即 `服务名.后缀`

因此：想要让环境配置共享，那么直接在Nacos控制台的配置中再加一个以` 服务名.后缀名` 格式命名的配置即可，如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525214929647-1348957116.png" alt="image-20230525214926182" />

其他的都不用动，要只是针对于项目中的yml，如 `appilication.yml`，那前面已经说了，会先读取Nacos中配置，然后和 `application.yml` 进行合并

但是：若项目本地的yml中、服务名.后缀、服务名-环境.后缀 中有相同的属性/配置时，优先级不一样，如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525215738196-645671337.png" alt="image-20230525215737066" />









## Nacos集群部署

windows和Linux都是一样的思路，集群部署的逻辑如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525230904184-1376989748.png" alt="image-20210409211355037" />



**1、解压压缩包**

**2、进入nacos的conf目录，修改配置文件cluster.conf.example，重命名为cluster.conf，并添加要部署的集群ip+port，如下：**

```txt
ip1:port1
ip2:port2
ip3:port3
```

**3、然后修改conf/application.properties文件，添加数据库配置**

```properties
# 告诉nacos数据库集群是MySQL，根据需要自定义
spring.datasource.platform=mysql
# 数据库的数量
db.num=1
# 数据库url
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
# 数据库用户名
db.user.0=root
# 数据库密码
db.password.0=88888
```

**4、复制解压包，部署到不同服务器，然后改变每个解压包的端口，路径：conf/application.properties文件，例如：**

```properties
# 第一个nacos节点
server.port=8845

# 第二个nacos节点
server.port=8846

# 第三个nacos节点
server.port=8847
```

**5、挨个启动nacos即可，进入到解压的nacos的bin目录中，执行如下命令即可**

```txt
startup.cmd

此命令告知：nacos默认就是集群启动，前面玩时加了 -m standalone 就是单机启动
```

**5、使用Nginx做反向代理** ：修改conf/nginx.conf文件，配置如下：

```nginx
upstream nacos-cluster {
  server ip1:port1;
  server ip2:port2;
  server ip3:port3;
}

server {
  listen       80;
  server_name  localhost;

  location /nacos {
    proxy_pass http://nacos-cluster;
  }
}
```

**6、代码中application.yml文件配置如下：**

```yaml
spring:
  cloud:
    nacos:
      # Nacos地址，上一步Nginx中的 server_name+listen监听的端口
      server-addr: localhost:80
```

**7、访问 http://localhost/nacos 即可**

- 注：浏览器默认就是80端口，而上面Nginx中监听的就是80，所以根据情况自行修改这里的访问路径









## Nacos服务注册表结构是怎样的？

分析源码就在nacos官网下载的source.code：nacos-naming/controller/InstanceController#register(HttpServletRequest request)

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230710230701827-1761667250.png" alt="image-20230710230658555" />

Java代码中是使用了`Map<String, Map<String, Service>>`：每一个服务去注册到Nacos时，就会把信息组织并存入这个Map中

- key：是namespace_id，用于环境隔离
- value：是Map<String, Service>
  - key：是group，但是是使用group+serviceName组成的key
  - value：表示Service服务，这个Service又套了一个Map<String, Cluster>
    - key：就是集群名
    - value：就是Cluster对象，这里面又套了一个Set<Instance>，这里面就是实例了









## Nacos为何能抗住数十万服务注册压力？

源码在：nacos-naming/controller/InstanceController#register(HttpServletRequest request)中的serviceManager.registerInstance(namespaceId, serviceName, instance)里面



先看抗住压力的原因的结论：

- 在Nacos集群下，对于临时实例，服务注册时是将其丢给了一个ArrayBlockingQueue阻塞队列，然后就返回客户端，最后通过一个死循环利用线程池去执行阻塞队列中的任务(注册服务)，这就做到了异步操作
- 将服务更新情况同步给集群中的其他节点也是同样的原理，底层还是用了阻塞队列+线程池



具体的逻辑在 DistroConsistencyServiceImpl.put()中

```java
public class DistroConsistencyServiceImpl {
    
	@Override
    public void put(String key, Record value) throws NacosException {
		// 异步服务注册 key是服务唯一id，value就是instances
        onPut(key, value);
        // 服务更新情况异步更新给集群下的另外节点
        distroProtocol.sync(new DistroKey(key, KeyBuilder.INSTANCE_LIST_KEY_PREFIX), DataOperation.CHANGE,
                globalConfig.getTaskDispatchPeriod() / 2);
    }


    /**
     * Put a new record.
     *
     * @param key   key of record
     * @param value record
     */
    public void onPut(String key, Record value) {

        // 判断是否是临时实例
        if (KeyBuilder.matchEphemeralInstanceListKey(key)) {
            // 封装 Instances 信息到 数据集：Datum
            Datum<Instances> datum = new Datum<>();
            // value就是instances
            datum.value = (Instances) value;
            // key是服务的唯一id
            datum.key = key;
            // 加入当前修改时间
            datum.timestamp.incrementAndGet();
            // 数据存储 放入dataStore中
            dataStore.put(key, datum);
        }

        if (!listeners.containsKey(key)) {
            return;
        }

        // notifier这玩意儿 implements Runnable
        notifier.addTask(key, DataOperation.CHANGE);
    }




    public class Notifier implements Runnable {

        private ConcurrentHashMap<String, String> services = new ConcurrentHashMap<>(10 * 1024);

        /**
         * 维护了一个阻塞队列
         */
        private BlockingQueue<Pair<String, DataOperation>> tasks = new ArrayBlockingQueue<>(1024 * 1024);

        /**
         * Add new notify task to queue.
         *
         * @param datumKey data key
         * @param action   action for data
         */
        public void addTask(String datumKey, DataOperation action) {

            if (services.containsKey(datumKey) && action == DataOperation.CHANGE) {
                return;
            }
            if (action == DataOperation.CHANGE) {
                services.put(datumKey, StringUtils.EMPTY);
            }
            // 将服务唯一id + 事件类型(CHANGE)放入了阻塞队列
            tasks.offer(Pair.with(datumKey, action));
        }
        
        
        
        @Override
        public void run() {
            Loggers.DISTRO.info("distro notifier started");

            for (; ; ) { // 死循环
                try {
                    // 去阻塞队列中获取任务
                    Pair<String, DataOperation> pair = tasks.take();
                    // 有任务就处理任务，更新服务列表；无任务就进入wait，所以此死循环不会导致CPU负载过高
                    handle(pair);
                } catch (Throwable e) {
                    Loggers.DISTRO.error("[NACOS-DISTRO] Error while handling notifying task", e);
                }
            }
        }
        
        
        
		/**
		 * DistroConsistencyServiceImpl.Notifier类的 handle 方法：即 handle(pair) 中的逻辑
		 */
		private void handle(Pair<String, DataOperation> pair) {
            try {
                String datumKey = pair.getValue0();
                DataOperation action = pair.getValue1();

                services.remove(datumKey);

                int count = 0;

                if (!listeners.containsKey(datumKey)) {
                    return;
                }

                // 遍历，找到变化的service，这里的 RecordListener 就是 Service
                for (RecordListener listener : listeners.get(datumKey)) {

                    count++;

                    try {
                        // 如果是 CHANGE 事件
                        if (action == DataOperation.CHANGE) {
                            // 就更新服务列表
                            listener.onChange(datumKey, dataStore.get(datumKey).value);
                            continue;
                        }

                        // 如果是 DELETE 事件
                        if (action == DataOperation.DELETE) {
                            // 就根据服务ID从服务列表中删除服务
                            listener.onDelete(datumKey);
                            continue;
                        }
                    } catch (Throwable e) {
                        Loggers.DISTRO.error("[NACOS-DISTRO] error while notifying listener of key: {}", datumKey, e);
                    }
                }

                if (Loggers.DISTRO.isDebugEnabled()) {
                    Loggers.DISTRO
                            .debug("[NACOS-DISTRO] datum change notified, key: {}, listener count: {}, action: {}",
                                    datumKey, count, action.name());
                }
            } catch (Throwable e) {
                Loggers.DISTRO.error("[NACOS-DISTRO] Error while handling notifying task", e);
            }
        }
    }
}
```

因此能抗住压力的原因：

- 在Nacos集群下，对于临时实例，服务注册时是将其丢给了一个ArrayBlockingQueue阻塞队列，然后就返回客户端，最后通过一个死循环利用线程池去执行阻塞队列中的任务(注册服务)，这就做到了异步操作
- 将服务更新情况同步给集群中的其他节点也是同样的原理，底层还是用了阻塞队列+线程池









## Nacos实例的并发读写问题

源码还是在：nacos-naming/controller/InstanceController#register(HttpServletRequest request)中的serviceManager.registerInstance(namespaceId, serviceName, instance)里面



具体思路：采用了同步锁+CopyOnWrite思想

- 并发读的解决方式 - CopyOnWrite思想：将原来的实例列表Map拷贝给了一个新的Map，然后对新的实例列表Map进行增删，最后将新的实例列表Map的引用给旧的实例列表Map
- 并发写的解决方式：
  - 在注册实例时，会使用synchronized同步锁对service进行加锁，不同service不影响，相同service通过锁排斥
  - 另外还有一个原因是：更新实例列表时，底层使用了线程池异步更新实例列表，但是线程池的线程数量为“1”




```java
@Component
public class ServiceManager {

    public void addInstance(String namespaceId, String serviceName, boolean ephemeral, Instance... ips)
            throws NacosException {
        // 监听服务列表用到的key，服务唯一标识
        // 如：com.alibaba.nacos.naming.iplist.ephemeral.public##DEFAULT_GROUP@@order-service
        String key = KeyBuilder.buildInstanceListKey(namespaceId, serviceName, ephemeral);

        // 获取服务
        Service service = getService(namespaceId, serviceName);

        // 同步锁：解决并发写的问题
        synchronized (service) {
            // 1、获取要更新的实例列表
            // addIPAddress中，会拷贝旧的实例列表，添加新实例到列表中 即：COPY
            List<Instance> instanceList = addIpAddresses(service, ephemeral, ips);

            // 2、将更新后的数据封装到Instances对象
            Instances instances = new Instances();
            instances.setInstanceList(instanceList);

            // 3、完成 注册表更新 以及 Nacos集群的数据同步(保证集群一致性)
            // 在这里面 完成对实例状态更新后，会用新列表直接覆盖旧实例列表。而在更新过程中，旧实例列表不受影响，用户依然可以读取
            consistencyService.put(key, instances);
        }
    }




    private List<Instance> addIpAddresses(Service service, boolean ephemeral, Instance... ips) throws NacosException {
        return updateIpAddresses(service, UtilsAndCommons.UPDATE_INSTANCE_ACTION_ADD, ephemeral, ips);
    }




    /**
     * 拷贝旧的实例列表，添加新实例到列表中
     */
    public List<Instance> updateIpAddresses(Service service, String action, boolean ephemeral, Instance... ips)
            throws NacosException {

        // 根据namespaceId、serviceName获取当前服务的实例列表，返回值是Datum
        // 第一次来，肯定是null
        Datum datum = consistencyService
                .get(KeyBuilder.buildInstanceListKey(service.getNamespaceId(), service.getName(), ephemeral));

        // 得到服务中旧的实例列表
        List<Instance> currentIPs = service.allIPs(ephemeral);
        // 保存实例列表，key为ip地址，value是Instance对象
        Map<String, Instance> currentInstances = new HashMap<>(currentIPs.size());
        // 创建Set集合，保存实例的instanceId
        Set<String> currentInstanceIds = Sets.newHashSet();

        // 遍历旧实例列表
        for (Instance instance : currentIPs) {
            // 保存实例列表
            currentInstances.put(instance.toIpAddr(), instance);
            // 添加instanceId到set中
            currentInstanceIds.add(instance.getInstanceId());
        }

        // 用来保存更新后的实例列表
        Map<String, Instance> instanceMap;
        // 如果服务中已经有旧的数据
        if (datum != null && null != datum.value) {
            // 将旧实例列表与新实例列表进行比对、合并
            instanceMap = setValid(((Instances) datum.value).getInstanceList(), currentInstances);
        } else {
            // 若服务中没有数据，则直接创建新的map
            instanceMap = new HashMap<>(ips.length);
        }

        // 遍历新实例列表ips
        for (Instance instance : ips) {
            // 判断服务中是否包含要注册的实例的cluster信息
            if (!service.getClusterMap().containsKey(instance.getClusterName())) {
                // 如果不包含，创建新的cluster
                Cluster cluster = new Cluster(instance.getClusterName(), service);
                cluster.init();
                // 将集群放入service的注册表
                service.getClusterMap().put(instance.getClusterName(), cluster);
                // ......记录日志
            }

            // 删除实例 or 新增实例
            // 若是Remove删除事件类型
            if (UtilsAndCommons.UPDATE_INSTANCE_ACTION_REMOVE.equals(action)) {
                // 则通过实例ID删除实例
                instanceMap.remove(instance.getDatumKey());
            } else {
                // 通过实例ID从旧实例列表中获取实例
                Instance oldInstance = instanceMap.get(instance.getDatumKey());
                if (oldInstance != null) {
                    // 若旧实例列表中有这个实例 则将旧实例ID赋值给新实例ID
                    instance.setInstanceId(oldInstance.getInstanceId());
                } else {
                    // 若旧实例列表中没有这个实例 则给新实例生成一个实例ID
                    instance.setInstanceId(instance.generateInstanceId(currentInstanceIds));
                }
                // 实例ID为key、实例为value存入新实例列表
                instanceMap.put(instance.getDatumKey(), instance);
            }

        }

        if (instanceMap.size() <= 0 && UtilsAndCommons.UPDATE_INSTANCE_ACTION_ADD.equals(action)) {
            throw new IllegalArgumentException(
                    "ip list can not be empty, service: " + service.getName() + ", ip list: " + JacksonUtils
                            .toJson(instanceMap.values()));
        }

        // 将instanceMap中的所有实例转为List返回
        return new ArrayList<>(instanceMap.values());
    }
}
```







## 服务注册源码

Nacos的注册表结构是什么样的？

- Nacos是多级存储模型，最外层通过namespace来实现环境隔离，然后是group分组，分组下就是service服务，一个服务又可以分为不同的cluster集群，集群中包含多个instance实例。因此其注册表结构为一个Map，类型是：`Map<String, Map<String, Service>>`，

  外层key是`namespace_id`，内层key是`group+serviceName`.

  Service内部维护一个Map，结构是：`Map<String,Cluster>`，key是clusterName，值是集群信息

  Cluster内部维护一个Set集合，元素是Instance类型，代表集群中的多个实例。



Nacos如何保证并发写的安全性？

- 在注册实例时，会对service加锁，不同service之间本身就不存在并发写问题，互不影响。相同service时通过锁来互斥。并且，在更新实例列表时，是基于异步的线程池来完成，而线程池的线程数量为1.

问题延伸：Nacos是如何应对数十万服务的并发写请求？

- Nacos内部会将服务注册的任务放入阻塞队列，采用线程池异步来完成实例更新，从而提高并发写能力



Nacos如何避免并发读写的冲突？

- Nacos在更新实例列表时，会采用CopyOnWrite技术，首先将Old实例列表拷贝一份，然后更新拷贝的实例列表，再用更新后的实例列表来覆盖旧的实例列表。







### 客户端

流程如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714213956296-1817140687.png" alt="image-20210923185331470" />



#### NacosServiceRegistryAutoConfiguration

Nacos的客户端是基于SpringBoot的自动装配实现的，我们可以在nacos-discovery依赖：

`spring-cloud-starter-alibaba-nacos-discovery-2.2.6.RELEASE.jar`

这个包中找到Nacos自动装配信息：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714214158836-1129250375.png" alt="image-20210907201333049" />

可以看到，在NacosServiceRegistryAutoConfiguration这个类中，包含一个跟自动注册有关的Bean：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714214159163-1974064508.png" alt="image-20210907201612322" />



#### NacosAutoServiceRegistration

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714214423933-1155508228.png" alt="image-20210907213647145" />

可以看到在初始化时，其父类`AbstractAutoServiceRegistration`也被初始化了



`AbstractAutoServiceRegistration`如图：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714214423637-1253704703.png" alt="image-20210907214111801" />

可以看到它实现了`ApplicationListener`接口，监听Spring容器启动过程中的事件

在监听到`WebServerInitializedEvent`（web服务初始化完成）的事件后，执行了`bind` 方法。

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714214423828-1719975633.png" alt="image-20210907214411267" />

其中的bind方法如下：

```java
public void bind(WebServerInitializedEvent event) {
    // 获取 ApplicationContext
    ApplicationContext context = event.getApplicationContext();
    // 判断服务的 namespace,一般都是null
    if (context instanceof ConfigurableWebServerApplicationContext) {
        if ("management".equals(((ConfigurableWebServerApplicationContext) context)
                                .getServerNamespace())) {
            return;
        }
    }
    // 记录当前 web 服务的端口
    this.port.compareAndSet(0, event.getWebServer().getPort());
    // 启动当前服务注册流程
    this.start();
}
```

其中的start方法流程：

```java
public void start() {
		if (!isEnabled()) {
			if (logger.isDebugEnabled()) {
				logger.debug("Discovery Lifecycle disabled. Not starting");
			}
			return;
		}

		// 当前服务处于未运行状态时，才进行初始化
		if (!this.running.get()) {
            // 发布服务开始注册的事件
			this.context.publishEvent(
					new InstancePreRegisteredEvent(this, getRegistration()));
            // ☆☆☆☆开始注册☆☆☆☆
			register();
			if (shouldRegisterManagement()) {
				registerManagement();
			}
            // 发布注册完成事件
			this.context.publishEvent(
					new InstanceRegisteredEvent<>(this, getConfiguration()));
            // 服务状态设置为运行状态，基于AtomicBoolean
			this.running.compareAndSet(false, true);
		}

	}
```

其中最关键的register()方法就是完成服务注册的关键，代码如下：

```java
protected void register() {
    this.serviceRegistry.register(getRegistration());
}
```

此处的this.serviceRegistry就是NacosServiceRegistry：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714214423800-1666539666.png" alt="image-20210907215903335" />





#### NacosServiceRegistry

`NacosServiceRegistry`是Spring的`ServiceRegistry`接口的实现类，而ServiceRegistry接口是服务注册、发现的规约接口，定义了register、deregister等方法的声明。



而`NacosServiceRegistry`对`register`的实现如下：

```java
@Override
public void register(Registration registration) {
	// 判断serviceId是否为空，也就是spring.application.name不能为空
    if (StringUtils.isEmpty(registration.getServiceId())) {
        log.warn("No service to register for nacos client...");
        return;
    }
    // 获取Nacos的命名服务，其实就是注册中心服务
    NamingService namingService = namingService();
    // 获取 serviceId 和 Group
    String serviceId = registration.getServiceId();
    String group = nacosDiscoveryProperties.getGroup();
	// 封装服务实例的基本信息，如 cluster-name、是否为临时实例、权重、IP、端口等
    Instance instance = getNacosInstanceFromRegistration(registration);

    try {
        // 开始注册服务
        namingService.registerInstance(serviceId, group, instance);
        log.info("nacos registry, {} {} {}:{} register finished", group, serviceId,
                 instance.getIp(), instance.getPort());
    }
    catch (Exception e) {
        if (nacosDiscoveryProperties.isFailFast()) {
            log.error("nacos registry, {} register failed...{},", serviceId,
                      registration.toString(), e);
            rethrowRuntimeException(e);
        }
        else {
            log.warn("Failfast is false. {} register failed...{},", serviceId,
                     registration.toString(), e);
        }
    }
}
```

可以看到方法中最终是调用NamingService的registerInstance方法实现注册的

而NamingService接口的默认实现就是NacosNamingService





#### NacosNamingService

NacosNamingService提供了服务注册、订阅等功能

其中registerInstance就是注册服务实例，源码如下：

```java
@Override
public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {
    // 检查超时参数是否异常。心跳超时时间(默认15秒)必须大于心跳周期(默认5秒)
    NamingUtils.checkInstanceIsLegal(instance);
    // 拼接得到新的服务名，格式为：groupName@@serviceId
    String groupedServiceName = NamingUtils.getGroupedName(serviceName, groupName);
    // 判断是否为临时实例，默认为 true。
    if (instance.isEphemeral()) { // 这里面的两行代码很关键
        // 如果是临时实例，需要定时向 Nacos 服务发送心跳 ---------- 涉及临时实例的心跳检测
        BeatInfo beatInfo = beatReactor.buildBeatInfo(groupedServiceName, instance);
        // 添加心跳任务
        beatReactor.addBeatInfo(groupedServiceName, beatInfo);
    }
    // 发送注册服务实例的请求
    serverProxy.registerService(groupedServiceName, groupName, instance);
}
```

最终，由NacosProxy的registerService方法，完成服务注册

```java
public void registerService(String serviceName, String groupName, Instance instance) throws NacosException {

	// 组织请求参数
    final Map<String, String> params = new HashMap<String, String>(16);
    params.put(CommonParams.NAMESPACE_ID, namespaceId);
    params.put(CommonParams.SERVICE_NAME, serviceName);
    params.put(CommonParams.GROUP_NAME, groupName);
    params.put(CommonParams.CLUSTER_NAME, instance.getClusterName());
    params.put("ip", instance.getIp());
    params.put("port", String.valueOf(instance.getPort()));
    params.put("weight", String.valueOf(instance.getWeight()));
    params.put("enable", String.valueOf(instance.isEnabled()));
    params.put("healthy", String.valueOf(instance.isHealthy()));
    params.put("ephemeral", String.valueOf(instance.isEphemeral()));
    params.put("metadata", JacksonUtils.toJson(instance.getMetadata()));
	// 通过POST请求将上述参数，发送到 /nacos/v1/ns/instance
    reqApi(UtilAndComs.nacosUrlInstance, params, HttpMethod.POST);

}
```

这里提交的信息就是Nacos服务注册接口需要的完整参数，核心参数有：

- namespace_id：环境
- service_name：服务名称
- group_name：组名称
- cluster_name：集群名称
- ip: 当前实例的ip地址
- port: 当前实例的端口







### 服务端

服务端流程图：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230715001442649-717310257.png" alt="image-20210923214042926" />



官网下载源码：进入 naming-nacos/com/alibaba/nacos/naming/controllers/InstanceController#register(HttpServletRequest request)

```java
@CanDistro
@PostMapping
@Secured(parser = NamingResourceParser.class, action = ActionTypes.WRITE)
public String register(HttpServletRequest request) throws Exception {
    
	// 尝试获取namespaceId
    final String namespaceId = WebUtils
        .optional(request, CommonParams.NAMESPACE_ID, Constants.DEFAULT_NAMESPACE_ID);
    // 尝试获取serviceName，其格式为 group_name@@service_name
    final String serviceName = WebUtils.required(request, CommonParams.SERVICE_NAME);
    NamingUtils.checkServiceNameFormat(serviceName);
	// 解析出实例信息，封装为Instance对象
    final Instance instance = parseInstance(request);
	// 注册实例
    serviceManager.registerInstance(namespaceId, serviceName, instance);
    return "ok";
}
```

进入serviceManager.registerInstance(namespaceId, serviceName, instance)





#### ServiceManager

这里面的东西在前面并发读写的解决方式中见过了

> 这里面的流程一句话来说就是：先获取旧的实例列表，然后把新的实例信息与旧的做对比、合并，新的实例就添加，老的实例同步ID。然后返回最新的实例列表



registerInstance方法就是注册服务实例的方法：

```java
/**
 * 注册服务实例
 *
 * Register an instance to a service in AP mode.
 *
 * <p>This method creates service or cluster silently if they don't exist.
 *
 * @param namespaceId id of namespace
 * @param serviceName service name
 * @param instance    instance to register
 * @throws Exception any error occurred in the process
 */
public void registerInstance(String namespaceId, String serviceName, Instance instance) throws NacosException {
    
	// 创建一个空的service（如果是第一次来注册实例，要先创建一个空service出来，放入注册表）
    // 此时不包含实例信息
    createEmptyService(namespaceId, serviceName, instance.isEphemeral());
    // 拿到创建好的service
    Service service = getService(namespaceId, serviceName);
    // 拿不到则抛异常
    if (service == null) {
        throw new NacosException(NacosException.INVALID_PARAM,
                                 "service not found, namespace: " + namespaceId + ", service: " + serviceName);
    }
    // 添加要注册的实例到service中
    addInstance(namespaceId, serviceName, instance.isEphemeral(), instance);
}
```

创建好了服务，接下来就要添加实例到服务中：

```java
/**
 * 添加实例到服务中
 * 
 * Add instance to service.
 *
 * @param namespaceId namespace
 * @param serviceName service name
 * @param ephemeral   whether instance is ephemeral
 * @param ips         instances
 * @throws NacosException nacos exception
 */
public void addInstance(String namespaceId, String serviceName, boolean ephemeral, Instance... ips)
    throws NacosException {
    
	// 监听服务列表用到的key
    // 服务唯一标识，例如：com.alibaba.nacos.naming.iplist.ephemeral.public##DEFAULT_GROUP@@order-service
    String key = KeyBuilder.buildInstanceListKey(namespaceId, serviceName, ephemeral);
    // 获取服务
    Service service = getService(namespaceId, serviceName);
    // 同步锁，避免并发修改的安全问题
    synchronized (service) {
        // 1、获取要更新的实例列表
        // addIPAddress中，会拷贝旧的实例列表，添加新实例到列表中 即：COPY
        List<Instance> instanceList = addIpAddresses(service, ephemeral, ips);

        // 2、将更新后的数据封装到Instances对象
        Instances instances = new Instances();
        instances.setInstanceList(instanceList);

        // 3、完成 注册表更新 以及 Nacos集群的数据同步(保证集群一致性)
        // 在这里面 完成对实例状态更新后，会用新列表直接覆盖旧实例列表。而在更新过程中，旧实例列表不受影响，用户依然可以读取
        consistencyService.put(key, instances);
    }
}
```

最后就要更新服务的实例 列表了

```java
private List<Instance> addIpAddresses(Service service, boolean ephemeral, Instance... ips) throws NacosException {
    return updateIpAddresses(service, UtilsAndCommons.UPDATE_INSTANCE_ACTION_ADD, ephemeral, ips);
}



/**
 * 拷贝旧的实例列表，添加新实例到列表中
 */
public List<Instance> updateIpAddresses(Service service, String action, boolean ephemeral, Instance... ips)
    throws NacosException {

    // 根据namespaceId、serviceName获取当前服务的实例列表，返回值是Datum
    // 第一次来，肯定是null
    Datum datum = consistencyService
        .get(KeyBuilder.buildInstanceListKey(service.getNamespaceId(), service.getName(), ephemeral));

    // 得到服务中旧的实例列表
    List<Instance> currentIPs = service.allIPs(ephemeral);
    // 保存实例列表，key为ip地址，value是Instance对象
    Map<String, Instance> currentInstances = new HashMap<>(currentIPs.size());
    // 创建Set集合，保存实例的instanceId
    Set<String> currentInstanceIds = Sets.newHashSet();

    // 遍历旧的实例列表
    for (Instance instance : currentIPs) {
        // 保存实例列表
        currentInstances.put(instance.toIpAddr(), instance);
        // 添加instanceId到set中
        currentInstanceIds.add(instance.getInstanceId());
    }

    // 用来保存更新后的实例列表
    Map<String, Instance> instanceMap;
    // 如果服务中已经有旧的数据
    if (datum != null && null != datum.value) {
        // 将旧的实例列表与新的实例列表进行比对
        instanceMap = setValid(((Instances) datum.value).getInstanceList(), currentInstances);
    } else {
        // 若服务中没有数据，则直接创建新的map
        instanceMap = new HashMap<>(ips.length);
    }

    // 遍历新的实例列表ips
    for (Instance instance : ips) {
        // 判断服务中是否包含要注册的实例的cluster信息
        if (!service.getClusterMap().containsKey(instance.getClusterName())) {
            // 如果不包含，创建新的cluster
            Cluster cluster = new Cluster(instance.getClusterName(), service);
            cluster.init();
            // 将集群放入service的注册表
            service.getClusterMap().put(instance.getClusterName(), cluster);
            Loggers.SRV_LOG
                .warn("cluster: {} not found, ip: {}, will create new cluster with default configuration.",
                      instance.getClusterName(), instance.toJson());
        }

        // 删除实例 or 新增实例
        // 若是Remove删除事件类型
        if (UtilsAndCommons.UPDATE_INSTANCE_ACTION_REMOVE.equals(action)) {
            // 则通过实例ID删除实例
            instanceMap.remove(instance.getDatumKey());
        } else {
            // 通过实例ID从旧实例列表中获取实例
            Instance oldInstance = instanceMap.get(instance.getDatumKey());
            if (oldInstance != null) {
                // 若旧实例列表中有这个实例 则将旧实例ID赋值给新实例ID
                instance.setInstanceId(oldInstance.getInstanceId());
            } else {
                // 若旧实例列表中没有这个实例 则给新实例生成一个实例ID
                instance.setInstanceId(instance.generateInstanceId(currentInstanceIds));
            }
            // 实例ID为key、实例为value存入新实例列表
            instanceMap.put(instance.getDatumKey(), instance);
        }

    }

    if (instanceMap.size() <= 0 && UtilsAndCommons.UPDATE_INSTANCE_ACTION_ADD.equals(action)) {
        throw new IllegalArgumentException(
            "ip list can not be empty, service: " + service.getName() + ", ip list: " + JacksonUtils
            .toJson(instanceMap.values()));
    }

    // 将instanceMap中的所有实例转为List返回
    return new ArrayList<>(instanceMap.values());
}
```





### Nacos集群一致性

在上一节中，在完成本地服务列表更新后，Nacos又实现了集群一致性更新，调用的是:

`consistencyService.put(key, instances);`

```java
/**
 * 添加实例到服务中
 * 
 * Add instance to service.
 *
 * @param namespaceId namespace
 * @param serviceName service name
 * @param ephemeral   whether instance is ephemeral
 * @param ips         instances
 * @throws NacosException nacos exception
 */
public void addInstance(String namespaceId, String serviceName, boolean ephemeral, Instance... ips)
    throws NacosException {
    
	// 监听服务列表用到的key
    // 服务唯一标识，例如：com.alibaba.nacos.naming.iplist.ephemeral.public##DEFAULT_GROUP@@order-service
    String key = KeyBuilder.buildInstanceListKey(namespaceId, serviceName, ephemeral);
    // 获取服务
    Service service = getService(namespaceId, serviceName);
    // 同步锁，避免并发修改的安全问题
    synchronized (service) {
        // 1、获取要更新的实例列表
        // addIPAddress中，会拷贝旧的实例列表，添加新实例到列表中 即：COPY
        List<Instance> instanceList = addIpAddresses(service, ephemeral, ips);

        // 2、将更新后的数据封装到Instances对象
        Instances instances = new Instances();
        instances.setInstanceList(instanceList);

        // 3、完成 注册表更新 以及 Nacos集群的数据同步(保证集群一致性)
        // 在这里面 完成对实例状态更新后，会用新列表直接覆盖旧实例列表。而在更新过程中，旧实例列表不受影响，用户依然可以读取
        consistencyService.put(key, instances);
    }
}
```

这里的ConsistencyService接口，代表集群一致性的接口，有很多种不同实现：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714231915065-226812848.png" alt="image-20210922161705573" />

进入DelegateConsistencyServiceImpl来看：

```java
@Override
public void put(String key, Record value) throws NacosException {
    // 根据实例是否是临时实例，判断委托对象
    mapConsistencyService(key).put(key, value);
}
```

其中的`mapConsistencyService(key)`方法就是选择委托方式：

```java
private ConsistencyService mapConsistencyService(String key) {
    // 判断是否是临时实例：
    // 是，选择 ephemeralConsistencyService，也就是 DistroConsistencyServiceImpl
    // 否，选择 persistentConsistencyService，也就是 PersistentConsistencyServiceDelegateImpl
    return KeyBuilder.matchEphemeralKey(key) ? ephemeralConsistencyService : persistentConsistencyService;
}
```

默认情况下，所有实例都是临时实例，因此关注DistroConsistencyServiceImpl即可





#### DistroConsistencyServiceImpl

这里面的逻辑在前面“Nacos如何抗住数十万服务注册压力”中见过了的，但是没弄全

```java
@Override
public void put(String key, Record value) throws NacosException {
    // 异步服务注册 key是服务的唯一id，value就是instances
    onPut(key, value);
    // 服务更强情况异步更新给集群下的另外节点
    distroProtocol.sync(new DistroKey(key, KeyBuilder.INSTANCE_LIST_KEY_PREFIX), DataOperation.CHANGE,
                        globalConfig.getTaskDispatchPeriod() / 2);
}
```





##### onPut 更新本地实例列表

```java
@DependsOn("ProtocolManager")
@org.springframework.stereotype.Service("distroConsistencyService")
public class DistroConsistencyServiceImpl implements EphemeralConsistencyService, DistroDataProcessor {

	public void onPut(String key, Record value) {

        // 判断是否是临时实例
        if (KeyBuilder.matchEphemeralInstanceListKey(key)) {
            // 封装 Instances 信息到 数据集：Datum
            Datum<Instances> datum = new Datum<>();
            // value就是instances
            datum.value = (Instances) value;
            // key是服务的唯一id
            datum.key = key;
            // 加入当前修改时间
            datum.timestamp.incrementAndGet();
            // 数据存储 放入dataStore中
            dataStore.put(key, datum);
        }

        if (!listeners.containsKey(key)) {
            return;
        }

        // notifier这玩意儿 implements Runnable
        notifier.addTask(key, DataOperation.CHANGE);
    }
    
    
    
	public class Notifier implements Runnable {

        private ConcurrentHashMap<String, String> services = new ConcurrentHashMap<>(10 * 1024);

        /**
         * 维护了一个阻塞队列
         */
        private BlockingQueue<Pair<String, DataOperation>> tasks = new ArrayBlockingQueue<>(1024 * 1024);

        /**
         * Add new notify task to queue.
         *
         * @param datumKey data key
         * @param action   action for data
         */
        public void addTask(String datumKey, DataOperation action) {

            if (services.containsKey(datumKey) && action == DataOperation.CHANGE) {
                return;
            }
            if (action == DataOperation.CHANGE) {
                services.put(datumKey, StringUtils.EMPTY);
            }
            // 将服务唯一id + 事件类型(CHANGE)放入了阻塞队列
            tasks.offer(Pair.with(datumKey, action));
        }
    }
}
```





##### Notifier异步更新

Notifier是一个Runnable，通过一个单线程的线程池来不断从阻塞队列中获取任务，执行服务列表的更新

```java
@DependsOn("ProtocolManager")
@org.springframework.stereotype.Service("distroConsistencyService")
public class DistroConsistencyServiceImpl implements EphemeralConsistencyService, DistroDataProcessor {
    
	public class Notifier implements Runnable {

        private ConcurrentHashMap<String, String> services = new ConcurrentHashMap<>(10 * 1024);

        /**
         * 维护了一个阻塞队列
         */
        private BlockingQueue<Pair<String, DataOperation>> tasks = new ArrayBlockingQueue<>(1024 * 1024);
        
        
		@Override
        public void run() {
            Loggers.DISTRO.info("distro notifier started");

            for (; ; ) { // 死循环
                try {
                    // 去阻塞队列中获取任务
                    Pair<String, DataOperation> pair = tasks.take();
                    // 有任务就处理任务，更新服务列表；无任务就进入wait，所以此死循环不会导致CPU负载过高
                    handle(pair);
                } catch (Throwable e) {
                    Loggers.DISTRO.error("[NACOS-DISTRO] Error while handling notifying task", e);
                }
            }
        }

        private void handle(Pair<String, DataOperation> pair) {
            try {
                String datumKey = pair.getValue0();
                DataOperation action = pair.getValue1();

                services.remove(datumKey);

                int count = 0;

                if (!listeners.containsKey(datumKey)) {
                    return;
                }

                // 遍历，找到变化的service，这里的 RecordListener就是 Service
                for (RecordListener listener : listeners.get(datumKey)) {

                    count++;

                    try {
                        // 如果是 CHANGE 事件
                        if (action == DataOperation.CHANGE) {
                            // 就更新服务列表
                            listener.onChange(datumKey, dataStore.get(datumKey).value);
                            continue;
                        }

                        // 如果是 DELETE 事件
                        if (action == DataOperation.DELETE) {
                            // 就根据服务ID删除从服务列表中删除服务
                            listener.onDelete(datumKey);
                            continue;
                        }
                    } catch (Throwable e) {
                        Loggers.DISTRO.error("[NACOS-DISTRO] error while notifying listener of key: {}", datumKey, e);
                    }
                }

                if (Loggers.DISTRO.isDebugEnabled()) {
                    Loggers.DISTRO
                            .debug("[NACOS-DISTRO] datum change notified, key: {}, listener count: {}, action: {}",
                                    datumKey, count, action.name());
                }
            } catch (Throwable e) {
                Loggers.DISTRO.error("[NACOS-DISTRO] Error while handling notifying task", e);
            }
        }
    }
}
```



##### onChange 覆盖实例列表

上一节中 `listener.onChange(datumKey, dataStore.get(datumKey).value);` 进去，选择Service的onChange()

```java
@JsonInclude(Include.NON_NULL)
public class Service extends com.alibaba.nacos.api.naming.pojo.Service implements Record, RecordListener<Instances> {

    @Override
    public void onChange(String key, Instances value) throws Exception {
        
        Loggers.SRV_LOG.info("[NACOS-RAFT] datum is changed, key: {}, value: {}", key, value);
        
        for (Instance instance : value.getInstanceList()) {
            
            if (instance == null) {
                // Reject this abnormal instance list:
                throw new RuntimeException("got null instance " + key);
            }
            
            if (instance.getWeight() > 10000.0D) {
                instance.setWeight(10000.0D);
            }
            
            if (instance.getWeight() < 0.01D && instance.getWeight() > 0.0D) {
                instance.setWeight(0.01D);
            }
        }
        
        // 更新实例列表
        updateIPs(value.getInstanceList(), KeyBuilder.matchEphemeralInstanceListKey(key));
        
        recalculateChecksum();
    }
}
```

updateIPs 的逻辑如下：

```java
@JsonInclude(Include.NON_NULL)
public class Service extends com.alibaba.nacos.api.naming.pojo.Service implements Record, RecordListener<Instances> {
    
	/**
	 * 更新实例列表
	 *
     * Update instances.
     *
     * @param instances instances
     * @param ephemeral whether is ephemeral instance
     */
    public void updateIPs(Collection<Instance> instances, boolean ephemeral) {
        // key是cluster，值是集群下的Instance集合
        Map<String, List<Instance>> ipMap = new HashMap<>(clusterMap.size());
        // 获取服务的所有cluster名称
        for (String clusterName : clusterMap.keySet()) {
            ipMap.put(clusterName, new ArrayList<>());
        }

        // 遍历要更新的实例
        for (Instance instance : instances) {
            try {
                if (instance == null) {
                    Loggers.SRV_LOG.error("[NACOS-DOM] received malformed ip: null");
                    continue;
                }

                // 判断实例是否包含clusterName，没有的话用默认cluster
                if (StringUtils.isEmpty(instance.getClusterName())) {
                    // DEFAULT_CLUSTER_NAME = "DEFAULT"
                    instance.setClusterName(UtilsAndCommons.DEFAULT_CLUSTER_NAME);
                }

                // 判断cluster是否存在，不存在则创建新的cluster
                if (!clusterMap.containsKey(instance.getClusterName())) {
                    Loggers.SRV_LOG
                            .warn("cluster: {} not found, ip: {}, will create new cluster with default configuration.",
                                    instance.getClusterName(), instance.toJson());
                    Cluster cluster = new Cluster(instance.getClusterName(), this);
                    cluster.init();
                    getClusterMap().put(instance.getClusterName(), cluster);
                }

                // 获取当前cluster实例的集合，不存在则创建新的
                List<Instance> clusterIPs = ipMap.get(instance.getClusterName());
                if (clusterIPs == null) {
                    clusterIPs = new LinkedList<>();
                    ipMap.put(instance.getClusterName(), clusterIPs);
                }

                // 添加新的实例到 Instance 集合
                clusterIPs.add(instance);
            } catch (Exception e) {
                Loggers.SRV_LOG.error("[NACOS-DOM] failed to process ip: " + instance, e);
            }
        }

        for (Map.Entry<String, List<Instance>> entry : ipMap.entrySet()) {
            //make every ip mine
            List<Instance> entryIPs = entry.getValue();
            // 将实例集合更新到 clusterMap（注册表）
            clusterMap.get(entry.getKey()).updateIps(entryIPs, ephemeral);
        }

        setLastModifiedMillis(System.currentTimeMillis());
        // 发布服务变更的通知消息
        getPushService().serviceChanged(this);
        StringBuilder stringBuilder = new StringBuilder();

        for (Instance instance : allIPs()) {
            stringBuilder.append(instance.toIpAddr()).append("_").append(instance.isHealthy()).append(",");
        }

        Loggers.EVT_LOG.info("[IP-UPDATED] namespace: {}, service: {}, ips: {}", getNamespaceId(), getName(),
                stringBuilder.toString());

    }
}
```

上面的  `clusterMap.get(entry.getKey()).updateIps(entryIPs, ephemeral);` 就是在更新实例列表，进入 updateIps(entryIPs, ephemeral) 即可看到逻辑

```java
public class Cluster extends com.alibaba.nacos.api.naming.pojo.Cluster implements Cloneable {
    
	/**
	 * 更新实例列表
	 *
     * Update instance list.
     *
     * @param ips       instance list
     * @param ephemeral whether these instances are ephemeral
     */
    public void updateIps(List<Instance> ips, boolean ephemeral) {

        // 获取旧实例列表
        Set<Instance> toUpdateInstances = ephemeral ? ephemeralInstances : persistentInstances;

        HashMap<String, Instance> oldIpMap = new HashMap<>(toUpdateInstances.size());

        for (Instance ip : toUpdateInstances) {
            oldIpMap.put(ip.getDatumKey(), ip);
        }

        // 更新实例列表
        List<Instance> updatedIPs = updatedIps(ips, oldIpMap.values());
        if (updatedIPs.size() > 0) {
            for (Instance ip : updatedIPs) {
                Instance oldIP = oldIpMap.get(ip.getDatumKey());

                // do not update the ip validation status of updated ips
                // because the checker has the most precise result
                // Only when ip is not marked, don't we update the health status of IP:
                if (!ip.isMarked()) {
                    ip.setHealthy(oldIP.isHealthy());
                }

                if (ip.isHealthy() != oldIP.isHealthy()) {
                    // ip validation status updated
                    Loggers.EVT_LOG.info("{} {SYNC} IP-{} {}:{}@{}", getService().getName(),
                            (ip.isHealthy() ? "ENABLED" : "DISABLED"), ip.getIp(), ip.getPort(), getName());
                }

                if (ip.getWeight() != oldIP.getWeight()) {
                    // ip validation status updated
                    Loggers.EVT_LOG.info("{} {SYNC} {IP-UPDATED} {}->{}", getService().getName(), oldIP.toString(),
                            ip.toString());
                }
            }
        }

        // 检查新加入实例的状态
        List<Instance> newIPs = subtract(ips, oldIpMap.values());
        if (newIPs.size() > 0) {
            Loggers.EVT_LOG
                    .info("{} {SYNC} {IP-NEW} cluster: {}, new ips size: {}, content: {}", getService().getName(),
                            getName(), newIPs.size(), newIPs.toString());

            for (Instance ip : newIPs) {
                HealthCheckStatus.reset(ip);
            }
        }

        // 移除要删除的实例
        List<Instance> deadIPs = subtract(oldIpMap.values(), ips);

        if (deadIPs.size() > 0) {
            Loggers.EVT_LOG
                    .info("{} {SYNC} {IP-DEAD} cluster: {}, dead ips size: {}, content: {}", getService().getName(),
                            getName(), deadIPs.size(), deadIPs.toString());

            for (Instance ip : deadIPs) {
                HealthCheckStatus.remv(ip);
            }
        }

        toUpdateInstances = new HashSet<>(ips);

        // 直接覆盖旧实例列表
        if (ephemeral) {
            ephemeralInstances = toUpdateInstances;
        } else {
            persistentInstances = toUpdateInstances;
        }
    }
}
```





##### Nacos集群一致性

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230715001648829-189629344.png" alt="image-20230715001647191" />

```java
@Component
public class DistroProtocol {
    
	/**
	 * 同步数据到其他远程服务器
	 *
     * Start to sync data to all remote server.
     *
     * @param distroKey distro key of sync data
     * @param action    the action of data operation
     */
    public void sync(DistroKey distroKey, DataOperation action, long delay) {
        // 遍历 Nacos 集群中除自己以外的其它节点
        for (Member each : memberManager.allMembersWithoutSelf()) {
            DistroKey distroKeyWithTarget = new DistroKey(distroKey.getResourceKey(), distroKey.getResourceType(),
                    each.getAddress());
            // Distro同步任务
            DistroDelayTask distroDelayTask = new DistroDelayTask(distroKeyWithTarget, action, delay);
            // 交给线程池去执行
            distroTaskEngineHolder.getDelayTaskExecuteEngine().addTask(distroKeyWithTarget, distroDelayTask);
            if (Loggers.DISTRO.isDebugEnabled()) {
                Loggers.DISTRO.debug("[DISTRO-SCHEDULE] {} to {}", distroKey, each.getAddress());
            }
        }
    }
}
```

distroTaskEngineHolder.getDelayTaskExecuteEngine() 的返回值是 NacosDelayTaskExecuteEngine，它维护了一个线程池，并且接收任务，执行任务。执行任务的方法为processTasks()方法

````java
public class NacosDelayTaskExecuteEngine extends AbstractNacosTaskExecuteEngine<AbstractDelayTask> {

    protected void processTasks() {
        Collection<Object> keys = getAllTaskKeys();
        for (Object taskKey : keys) {
            AbstractDelayTask task = removeTask(taskKey);
            if (null == task) {
                continue;
            }
            NacosTaskProcessor processor = getProcessor(taskKey);
            if (null == processor) {
                getEngineLog().error("processor not found for task, so discarded. " + task);
                continue;
            }
            try {
                // ReAdd task if process failed
                // 尝试执行同步任务，如果失败会将任务重新入队重试
                if (!processor.process(task)) {
                    retryFailedTask(taskKey, task);
                }
            } catch (Throwable e) {
                getEngineLog().error("Nacos task execute error : " + e.toString(), e);
                retryFailedTask(taskKey, task);
            }
        }
    }
}
````

Distro模式的同步是异步进行的，并且失败时会将任务重新入队并重试，因此不保证同步结果的强一致性，属于AP模式的一致性策略









## 心跳检测源码

Nacos的健康检测有两种模式：

- 临时实例：适合增加更多实例来应对高并发
  - 采用客户端心跳检测模式，心跳周期5秒
  - 心跳间隔超过15秒则标记为不健康
  - 心跳间隔超过30秒则从服务列表删除
- 永久实例：适合常备实例
  - 采用服务端主动健康检测方式
  - 周期为2000 + 5000毫秒内的随机数
  - 检测异常只会标记为不健康，不会删除







### 客户端

在前面看nacos服务注册的客户端源码时，看到过一段代码：

```java
@Override
public void registerInstance(String serviceName, String groupName, Instance instance) throws NacosException {
    
    // 检查超时参数是否异常。心跳超时时间(默认15秒)必须大于心跳周期(默认5秒)
    NamingUtils.checkInstanceIsLegal(instance);
    // 拼接得到新的服务名，格式为：groupName@@serviceId
    String groupedServiceName = NamingUtils.getGroupedName(serviceName, groupName);
    // 判断是否为临时实例，默认为 true。
    if (instance.isEphemeral()) { // 这里面的两行代码很关键
        // 如果是临时实例，需要定时向 Nacos 服务发送心跳 ---------- 涉及临时实例的心跳检测
        BeatInfo beatInfo = beatReactor.buildBeatInfo(groupedServiceName, instance);
        // 添加心跳任务
        beatReactor.addBeatInfo(groupedServiceName, beatInfo);
    }
    // 发送注册服务实例的请求
    serverProxy.registerService(groupedServiceName, groupName, instance);
}
```

这个IF中就涉及的是心跳检测



#### BeatInfo

就包含心跳需要的各种信息

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230715004628878-853553063.png" alt="image-20210922213313677" />





#### BeatReactor

维护了一个线程池

```java
public class BeatReactor implements Closeable {

    public BeatReactor(NamingProxy serverProxy, int threadCount) {
        this.lightBeatEnabled = false;
        this.dom2Beat = new ConcurrentHashMap();
        this.serverProxy = serverProxy;
        this.executorService = new ScheduledThreadPoolExecutor(threadCount, new ThreadFactory() {
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setDaemon(true);
                thread.setName("com.alibaba.nacos.naming.beat.sender");
                return thread;
            }
        });
    }
}
```

当调用`BeatReactor`的`addBeatInfo(groupedServiceName, beatInfo)`方法时，就会执行心跳

```java
public class BeatReactor implements Closeable {

    public void addBeatInfo(String serviceName, BeatInfo beatInfo) {
        LogUtils.NAMING_LOGGER.info("[BEAT] adding beat: {} to beat map.", beatInfo);
        String key = this.buildKey(serviceName, beatInfo.getIp(), beatInfo.getPort());
        BeatInfo existBeat = null;
        if ((existBeat = (BeatInfo)this.dom2Beat.remove(key)) != null) {
            existBeat.setStopped(true);
        }

        this.dom2Beat.put(key, beatInfo);
        // 利用线程池，定期执行心跳任务，周期为 beatInfo.getPeriod()
        this.executorService.schedule(new BeatReactor.BeatTask(beatInfo), beatInfo.getPeriod(), TimeUnit.MILLISECONDS);
        MetricsMonitor.getDom2BeatSizeMonitor().set((double)this.dom2Beat.size());
    }
}
```

心跳周期的默认值在`com.alibaba.nacos.api.common.Constants`类中：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230715005132896-909701255.png" alt="image-20210922213829632" />

默认5秒一次心跳





#### BeatTask

上一节中 `this.executorService.schedule(new BeatReactor.BeatTask(beatInfo), beatInfo.getPeriod(), TimeUnit.MILLISECONDS)`

心跳的任务封装就在`BeatTask`这个类中，是一个Runnable

```java
@Override
public void run() {
    if (beatInfo.isStopped()) {
        return;
    }
    // 获取心跳周期
    long nextTime = beatInfo.getPeriod();
    try {
        // 发送心跳
        JsonNode result = serverProxy.sendBeat(beatInfo, BeatReactor.this.lightBeatEnabled);
        long interval = result.get("clientBeatInterval").asLong();
        boolean lightBeatEnabled = false;
        if (result.has(CommonParams.LIGHT_BEAT_ENABLED)) {
            lightBeatEnabled = result.get(CommonParams.LIGHT_BEAT_ENABLED).asBoolean();
        }
        BeatReactor.this.lightBeatEnabled = lightBeatEnabled;
        if (interval > 0) {
            nextTime = interval;
        }
        // 判断心跳结果
        int code = NamingResponseCode.OK;
        if (result.has(CommonParams.CODE)) {
            code = result.get(CommonParams.CODE).asInt();
        }
        if (code == NamingResponseCode.RESOURCE_NOT_FOUND) {
            // 如果失败，则需要 重新注册实例
            Instance instance = new Instance();
            instance.setPort(beatInfo.getPort());
            instance.setIp(beatInfo.getIp());
            instance.setWeight(beatInfo.getWeight());
            instance.setMetadata(beatInfo.getMetadata());
            instance.setClusterName(beatInfo.getCluster());
            instance.setServiceName(beatInfo.getServiceName());
            instance.setInstanceId(instance.getInstanceId());
            instance.setEphemeral(true);
            try {
                serverProxy.registerService(beatInfo.getServiceName(),
                                            NamingUtils.getGroupName(beatInfo.getServiceName()), instance);
            } catch (Exception ignore) {
            }
        }
    } catch (NacosException ex) {
        // ...... 记录日志

    } catch (Exception unknownEx) {
        // ...... 记录日志
    } finally {
        executorService.schedule(new BeatTask(beatInfo), nextTime, TimeUnit.MILLISECONDS);
    }
}
```





#### 发送心跳

`JsonNode result = serverProxy.sendBeat(beatInfo, BeatReactor.this.lightBeatEnabled)` ,最终心跳的发送还是通过`NamingProxy`的`sendBeat`方法来实现

```java
public JsonNode sendBeat(BeatInfo beatInfo, boolean lightBeatEnabled) throws NacosException {

    if (NAMING_LOGGER.isDebugEnabled()) {
        NAMING_LOGGER.debug("[BEAT] {} sending beat to server: {}", namespaceId, beatInfo.toString());
    }
    // 组织请求参数
    Map<String, String> params = new HashMap<String, String>(8);
    Map<String, String> bodyMap = new HashMap<String, String>(2);
    if (!lightBeatEnabled) {
        bodyMap.put("beat", JacksonUtils.toJson(beatInfo));
    }
    params.put(CommonParams.NAMESPACE_ID, namespaceId);
    params.put(CommonParams.SERVICE_NAME, beatInfo.getServiceName());
    params.put(CommonParams.CLUSTER_NAME, beatInfo.getCluster());
    params.put("ip", beatInfo.getIp());
    params.put("port", String.valueOf(beatInfo.getPort()));
    // 发送请求，这个地址就是：/v1/ns/instance/beat
    String result = reqApi(UtilAndComs.nacosUrlBase + "/instance/beat", params, bodyMap, HttpMethod.PUT);
    return JacksonUtils.toObj(result);
}
```





### 服务端

对于临时实例，服务端代码分两部分：

1. InstanceController提供了一个接口，处理客户端的心跳请求
2. 定时检测实例心跳是否按期执行



#### InstanceController

在nacos-naming模块中的InstanceController类中，定义了一个方法用来处理心跳请求

```java
@RestController
@RequestMapping(UtilsAndCommons.NACOS_NAMING_CONTEXT + "/instance")
public class InstanceController {

    /** 
     * 为实例创建心跳
     * 
     * Create a beat for instance.
     *
     * @param request http request
     * @return detail information of instance
     * @throws Exception any error during handle
     */
    @CanDistro
    @PutMapping("/beat")
    @Secured(parser = NamingResourceParser.class, action = ActionTypes.WRITE)
    public ObjectNode beat(HttpServletRequest request) throws Exception {

        // 解析心跳的请求参数
        ObjectNode result = JacksonUtils.createEmptyJsonNode();
        result.put(SwitchEntry.CLIENT_BEAT_INTERVAL, switchDomain.getClientBeatInterval());

        String beat = WebUtils.optional(request, "beat", StringUtils.EMPTY);
        RsInfo clientBeat = null;
        if (StringUtils.isNotBlank(beat)) {
            clientBeat = JacksonUtils.toObj(beat, RsInfo.class);
        }
        String clusterName = WebUtils
                .optional(request, CommonParams.CLUSTER_NAME, UtilsAndCommons.DEFAULT_CLUSTER_NAME);
        String ip = WebUtils.optional(request, "ip", StringUtils.EMPTY);
        int port = Integer.parseInt(WebUtils.optional(request, "port", "0"));
        if (clientBeat != null) {
            if (StringUtils.isNotBlank(clientBeat.getCluster())) {
                clusterName = clientBeat.getCluster();
            } else {
                // fix #2533
                clientBeat.setCluster(clusterName);
            }
            ip = clientBeat.getIp();
            port = clientBeat.getPort();
        }
        String namespaceId = WebUtils.optional(request, CommonParams.NAMESPACE_ID, Constants.DEFAULT_NAMESPACE_ID);
        String serviceName = WebUtils.required(request, CommonParams.SERVICE_NAME);
        NamingUtils.checkServiceNameFormat(serviceName);
        Loggers.SRV_LOG.debug("[CLIENT-BEAT] full arguments: beat: {}, serviceName: {}", clientBeat, serviceName);
        // 尝试根据参数中的namespaceId、serviceName、clusterName、ip、port等信息从Nacos的注册表中 获取实例
        Instance instance = serviceManager.getInstance(namespaceId, serviceName, clusterName, ip, port);

        // 如果获取失败，说明心跳失败，实例尚未注册
        if (instance == null) {
            if (clientBeat == null) {
                result.put(CommonParams.CODE, NamingResponseCode.RESOURCE_NOT_FOUND);
                return result;
            }

            // ...... 记录日志

            // 重新注册一个实例
            instance = new Instance();
            instance.setPort(clientBeat.getPort());
            instance.setIp(clientBeat.getIp());
            instance.setWeight(clientBeat.getWeight());
            instance.setMetadata(clientBeat.getMetadata());
            instance.setClusterName(clusterName);
            instance.setServiceName(serviceName);
            instance.setInstanceId(instance.getInstanceId());
            instance.setEphemeral(clientBeat.isEphemeral());

            serviceManager.registerInstance(namespaceId, serviceName, instance);
        }

        // 尝试基于 namespaceId + serviceName 从 注册表 中获取Service服务
        Service service = serviceManager.getService(namespaceId, serviceName);

        // 如果不存在，说明服务不存在，返回SERVER_ERROR = 500
        if (service == null) {
            throw new NacosException(NacosException.SERVER_ERROR,
                    "service not found: " + serviceName + "@" + namespaceId);
        }
        if (clientBeat == null) {
            clientBeat = new RsInfo();
            clientBeat.setIp(ip);
            clientBeat.setPort(port);
            clientBeat.setCluster(clusterName);
        }
        // 如果心跳没问题(在确认心跳请求对应的服务、实例都在的情况下)，开始处理心跳结果
        service.processClientBeat(clientBeat);

        result.put(CommonParams.CODE, NamingResponseCode.OK);
        if (instance.containsMetadata(PreservedMetadataKeys.HEART_BEAT_INTERVAL)) {
            result.put(SwitchEntry.CLIENT_BEAT_INTERVAL, instance.getInstanceHeartBeatInterval());
        }
        result.put(SwitchEntry.LIGHT_BEAT_ENABLED, switchDomain.isLightBeatEnabled());
        return result;
    }
}
```







#### processClientBeat() 处理心跳请求

在上一节中有如下方法

```java
// 如果心跳没问题(在确认心跳请求对应的服务、实例都在的情况下)，开始处理心跳结果
service.processClientBeat(clientBeat);
```

这个方法的逻辑如下：

```java
@JsonInclude(Include.NON_NULL)
public class Service extends com.alibaba.nacos.api.naming.pojo.Service 
    implements Record, RecordListener<Instances> {

	/**
     * Process client beat.
     *
     * @param rsInfo metrics info of server
     */
    public void processClientBeat(final RsInfo rsInfo) {
        
        // 创建线程：ClientBeatProcessor implements Runnable
        ClientBeatProcessor clientBeatProcessor = new ClientBeatProcessor();
        clientBeatProcessor.setService(this);
        clientBeatProcessor.setRsInfo(rsInfo);
        // HealthCheckReactor：线程池的封装
        HealthCheckReactor.scheduleNow(clientBeatProcessor);
    }
}
```

所以关键业务逻辑就在ClientBeatProcessor的run()方法中

```java
public class ClientBeatProcessor implements Runnable {
 
	@Override
    public void run() {
        // 获取service、ip、clusterName、port、Cluster对象
        Service service = this.service;
        if (Loggers.EVT_LOG.isDebugEnabled()) {
            Loggers.EVT_LOG.debug("[CLIENT-BEAT] processing beat: {}", rsInfo.toString());
        }

        String ip = rsInfo.getIp();
        String clusterName = rsInfo.getCluster();
        int port = rsInfo.getPort();
        // 获取Cluster对象
        Cluster cluster = service.getClusterMap().get(clusterName);
        // 获取集群中的所有实例信息
        List<Instance> instances = cluster.allIPs(true);

        for (Instance instance : instances) {
            // 找到心跳的这个实例
            if (instance.getIp().equals(ip) && instance.getPort() == port) {
                if (Loggers.EVT_LOG.isDebugEnabled()) {
                    Loggers.EVT_LOG.debug("[CLIENT-BEAT] refresh beat: {}", rsInfo.toString());
                }
                // 更新最新的实例心跳时间,LastBeat就是用来判断心跳是否过期的
                instance.setLastBeat(System.currentTimeMillis());
                if (!instance.isMarked()) {
                    // 若实例已被标记为：不健康
                    if (!instance.isHealthy()) {
                        // 则将实例状态改为健康状态
                        instance.setHealthy(true);
                        Loggers.EVT_LOG
                                .info("service: {} {POS} {IP-ENABLED} valid: {}:{}@{}, region: {}, msg: client beat ok",
                                        cluster.getService().getName(), ip, port, cluster.getName(),
                                        UtilsAndCommons.LOCALHOST_SITE);
                        // 进行服务变更推送，即：push操作
                        getPushService().serviceChanged(service);
                    }
                }
            }
        }
    }
}
```





#### Service#init() 开启心跳检测任务

```java
@JsonInclude(Include.NON_NULL)
public class Service extends com.alibaba.nacos.api.naming.pojo.Service 
    implements Record, RecordListener<Instances> {

	/**
     * Init service.
     */
    public void init() {
        
        // 开启心跳检测任务
        HealthCheckReactor.scheduleCheck(clientBeatCheckTask);
        // 遍历注册表中的集群
        for (Map.Entry<String, Cluster> entry : clusterMap.entrySet()) {
            entry.getValue().setService(this);
             // 完成集群初始化：非临时实例的主动健康检测的逻辑就可以在这里面找到
            entry.getValue().init();
        }
    }
}
```

心跳检测任务的逻辑如下：

```java
public class HealthCheckReactor {

	/**
     * Schedule client beat check task with a delay.
     *
     * @param task client beat check task
     */
    public static void scheduleCheck(ClientBeatCheckTask task) {
        // ClientBeatCheckTask task 还是一个 Runnable
        // computeIfAbsent(key, mappingFunction) 与指定key关联的当前(现有的或function计算的)值，
        // 										若计算的(mappingFunction)为null则为null
        //      key：服务唯一ID，即 com.alibaba.nacos.naming.domains.meta. + NamespaceId + ## + serviceName
        //      value：mappingFunction 计算值的函数
        futureMap.computeIfAbsent(task.taskKey(),
                // scheduleNamingHealth() 第3个参数 delay 就是心跳检测任务执行时间，即：5s执行一次心跳检测任务
                k -> GlobalExecutor.scheduleNamingHealth(task, 5000, 5000, TimeUnit.MILLISECONDS));
    }
}
```

ClientBeatCheckTask的run()方法逻辑如下：

```java
public class ClientBeatCheckTask implements Runnable {

    public void run() {
        try {

            // 找到所有临时实例的列表
            List<Instance> instances = service.allIPs(true);

            // first set health status of instances:
            // 给临时实例设置健康状态
            for (Instance instance : instances) {
                // 判断 心跳间隔（当前时间 - 最后一次心跳时间） 是否大于 心跳超时时间，默认15s
                if (System.currentTimeMillis() - instance.getLastBeat() > instance.getInstanceHeartBeatTimeOut()) {
                    if (!instance.isMarked()) {
                        if (instance.isHealthy()) {
                            // 如果超时，标记实例为不健康 healthy = false
                            instance.setHealthy(false);
                            Loggers.EVT_LOG
                                    .info("{POS} {IP-DISABLED} valid: {}:{}@{}@{}, region: {}, msg: client timeout after {}, last beat: {}",
                                            instance.getIp(), instance.getPort(), instance.getClusterName(),
                                            service.getName(), UtilsAndCommons.LOCALHOST_SITE,
                                            instance.getInstanceHeartBeatTimeOut(), instance.getLastBeat());
                            // 发布实例状态变更的事件
                            getPushService().serviceChanged(service);
                            ApplicationUtils.publishEvent(new InstanceHeartbeatTimeoutEvent(this, instance));
                        }
                    }
                }
            }

            if (!getGlobalConfig().isExpireInstance()) {
                return;
            }

            // then remove obsolete instances:
            for (Instance instance : instances) {

                if (instance.isMarked()) {
                    continue;
                }
                
                // 判断心跳间隔（当前时间 - 最后一次心跳时间）是否大于 实例被删除的最长超时时间，默认30s
                if (System.currentTimeMillis() - instance.getLastBeat() > instance.getIpDeleteTimeout()) {
                    // delete instance
                    Loggers.SRV_LOG.info("[AUTO-DELETE-IP] service: {}, ip: {}", service.getName(),
                            JacksonUtils.toJson(instance));
                    // 若超过超时时间，则删除该实例
                    deleteIp(instance);
                }
            }

        } catch (Exception e) {
            Loggers.SRV_LOG.warn("Exception while processing client beat time out.", e);
        }
    }
}
```







#### 非临时实例：主动健康检测

对于非临时实例（ephemeral=false)，Nacos会采用主动的健康检测，定时向实例发送请求，根据响应来判断实例健康状态



在前面看服务注册的代码：InstanceController/re/register(HttpServletRequest request)#serviceManager.registerInstance(namespaceId, serviceName, instance)中有如下的代码

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719200941184-924014230.png" alt="image-20230719200935795" />

创建空服务时：

```java
@Component
public class ServiceManager implements RecordListener<Service> {

    public void createEmptyService(String namespaceId, String serviceName, boolean local) throws NacosException {
        // 若服务不存在，则创建新服务
        createServiceIfAbsent(namespaceId, serviceName, local, null);
    }
    
    
    /**
     * 若服务不存在，则创建新服务
     */
	public void createServiceIfAbsent(String namespaceId, String serviceName, boolean local, Cluster cluster)
            throws NacosException {
        // 通过 namespaceId + serviceName 获取服务
        Service service = getService(namespaceId, serviceName);
        if (service == null) {

            Loggers.SRV_LOG.info("creating empty service {}:{}", namespaceId, serviceName);
            // 若服务不存在则创建新服务
            service = new Service();
            service.setName(serviceName);
            service.setNamespaceId(namespaceId);
            service.setGroupName(NamingUtils.getGroupName(serviceName));
            // now validate the service. if failed, exception will be thrown
            service.setLastModifiedMillis(System.currentTimeMillis());
            service.recalculateChecksum();
            if (cluster != null) {
                cluster.setService(service);
                service.getClusterMap().put(cluster.getName(), cluster);
            }
            service.validate();

            // 写入注册表 并 初始化
            putServiceAndInit(service);
            if (!local) {
                addOrReplaceService(service);
            }
        }
    }
    
    
    
    /**
     * 将服务写入注册表 并 初始化服务
     */
	private void putServiceAndInit(Service service) throws NacosException {
        // 将服务添加到注册表
        putService(service);
        // 通过 NamespaceId + serviceName 尝试获取服务
        service = getService(service.getNamespaceId(), service.getName());
        // 初始化服务 这里就是进入Servicec,init()方法，即：开启心跳检测任务
        service.init();
        consistencyService
                .listen(KeyBuilder.buildInstanceListKey(service.getNamespaceId(), service.getName(), true), service);
        consistencyService
                .listen(KeyBuilder.buildInstanceListKey(service.getNamespaceId(), service.getName(), false), service);
        Loggers.SRV_LOG.info("[NEW-SERVICE] {}", service.toJson());
    }
}
```

service.init();的逻辑如下：

```java
@JsonInclude(Include.NON_NULL)
public class Service extends com.alibaba.nacos.api.naming.pojo.Service implements Record, RecordListener<Instances> {

    /**
     * Init service.
     */
    public void init() {
        // 开启心跳检测任务
        HealthCheckReactor.scheduleCheck(clientBeatCheckTask);
        // 遍历注册表中的集群
        for (Map.Entry<String, Cluster> entry : clusterMap.entrySet()) {
            entry.getValue().setService(this);
            // 完成集群初始化
            entry.getValue().init();
        }
    }
}
```

entry.getValue().init();的逻辑如下：

```java
public class Cluster extends com.alibaba.nacos.api.naming.pojo.Cluster implements Cloneable {

    /**
     * Init cluster.
     */
    public void init() {
        if (inited) {
            return;
        }
        checkTask = new HealthCheckTask(this);
        // 这里会开启对 非临时实例的 定时健康检测
        HealthCheckReactor.scheduleCheck(checkTask);
        inited = true;
    }
}
```

HealthCheckTask还是一个Runnable，其run()方法逻辑如下：

```java
public class HealthCheckTask implements Runnable {

    public void run() {

        try {
            if (distroMapper.responsible(cluster.getService().getName()) && 
                	switchDomain.isHealthCheckEnabled(cluster.getService().getName())) {
                // 进行健康检测
                healthCheckProcessor.process(this);
                // ......记录日志
            }
        } catch (Throwable e) {
            // ......记录日志
        } finally {
            if (!cancelled) {
                // 结束后，再次进行任务调度，一定延迟后执行
                HealthCheckReactor.scheduleCheck(this);
                // ..........
            }
        }
    }
```

健康检测逻辑定义在`healthCheckProcessor.process(this);`方法中，在HealthCheckProcessor接口中，这个接口也有很多实现，默认是`TcpSuperSenseProcessor`：

```java
@Component
@SuppressWarnings("PMD.ThreadPoolCreationRule")
public class TcpSuperSenseProcessor implements HealthCheckProcessor, Runnable {
 
        public void process(HealthCheckTask task) {
        // 从注册表中获取所有的 非临时实例
        List<Instance> ips = task.getCluster().allIPs(false);

        if (CollectionUtils.isEmpty(ips)) {
            return;
        }

        // 遍历非临时实例
        for (Instance ip : ips) {
            // 若没被标记为 不健康 则找下一个非临时实例
            if (ip.isMarked()) {
                if (SRV_LOG.isDebugEnabled()) {
                    SRV_LOG.debug("tcp check, ip is marked as to skip health check, ip:" + ip.getIp());
                }
                continue;
            }

            // 若此非临时实例不是正在被标记
            if (!ip.markChecking()) {
                // ......记录日志

                // 重新计算响应时间 并 找下一个实例
                healthCheckCommon
                        // 默认CheckRtNormalized = -1
                        // 默认TcpHealthParams：max=5000、min=1000、factor=0.75F
                        .reEvaluateCheckRT(task.getCheckRtNormalized() * 2, task, switchDomain.getTcpHealthParams());
                continue;
            }

            // 封装健康检测信息到 Beat
            Beat beat = new Beat(ip, task);
            // 异步执行：放入一个阻塞队列中
            taskQueue.add(beat);
            MetricsMonitor.getTcpHealthCheckMonitor().incrementAndGet();
        }
    }
}
```

可以看到，所有的健康检测任务都被放入一个阻塞队列，而不是立即执行了。这里又采用了异步执行的策略

而`TcpSuperSenseProcessor`本身就是一个Runnable，在它的构造函数中会把自己放入线程池中去执行，其run方法如下

```java
@Component
@SuppressWarnings("PMD.ThreadPoolCreationRule")
public class TcpSuperSenseProcessor implements HealthCheckProcessor, Runnable {

    /** 
     * 构造
     */
	public TcpSuperSenseProcessor() {
        try {
            selector = Selector.open();

            // 将自己放入线程池
            GlobalExecutor.submitTcpCheck(this);

        } catch (Exception e) {
            throw new IllegalStateException("Error while initializing SuperSense(TM).");
        }
    }
    
	
    
    public void run() {
        while (true) {
            try {
                // 处理任务
                processTask();
                // ......
            } catch (Throwable e) {
                SRV_LOG.error("[HEALTH-CHECK] error while processing NIO task", e);
            }
        }
    }
    
    
    
	private void processTask() throws Exception {
        // 将任务封装为一个 TaskProcessor，并放入集合
        Collection<Callable<Void>> tasks = new LinkedList<>();
        do {
            Beat beat = taskQueue.poll(CONNECT_TIMEOUT_MS / 2, TimeUnit.MILLISECONDS);
            if (beat == null) {
                return;
            }

            // 将任务丢给 TaskProcessor 去执行，TaskProcessor implements Callable<Void>
            tasks.add(new TaskProcessor(beat));
        } while (taskQueue.size() > 0 && tasks.size() < NIO_THREAD_COUNT * 64);

        // 批量处理集合中的任务
        for (Future<?> f : GlobalExecutor.invokeAllTcpSuperSenseTask(tasks)) {
            f.get();
        }
    }
}
```

TaskProcessor的cail()方法逻辑如下：

```java
@Component
@SuppressWarnings("PMD.ThreadPoolCreationRule")
public class TcpSuperSenseProcessor implements HealthCheckProcessor, Runnable {

	private class TaskProcessor implements Callable<Void> {

        @Override
        public Void call() {
            // 获取检测任务已经等待的时长
            long waited = System.currentTimeMillis() - beat.getStartTime();
            if (waited > MAX_WAIT_TIME_MILLISECONDS) {
                Loggers.SRV_LOG.warn("beat task waited too long: " + waited + "ms");
            }

            SocketChannel channel = null;
            try {
                // 获取实例信息
                Instance instance = beat.getIp();

                BeatKey beatKey = keyMap.get(beat.toString());
                if (beatKey != null && beatKey.key.isValid()) {
                    if (System.currentTimeMillis() - beatKey.birthTime < TCP_KEEP_ALIVE_MILLIS) {
                        instance.setBeingChecked(false);
                        return null;
                    }

                    beatKey.key.cancel();
                    beatKey.key.channel().close();
                }

                // 通过NIO建立TCP连接
                channel = SocketChannel.open();
                channel.configureBlocking(false);
                // only by setting this can we make the socket close event asynchronous
                channel.socket().setSoLinger(false, -1);
                channel.socket().setReuseAddress(true);
                channel.socket().setKeepAlive(true);
                channel.socket().setTcpNoDelay(true);

                Cluster cluster = beat.getTask().getCluster();
                int port = cluster.isUseIPPort4Check() ? instance.getPort() : cluster.getDefCkport();
                channel.connect(new InetSocketAddress(instance.getIp(), port));

                // 注册连接、读取事件
                SelectionKey key = channel.register(selector, SelectionKey.OP_CONNECT | SelectionKey.OP_READ);
                key.attach(beat);
                keyMap.put(beat.toString(), new BeatKey(key));

                beat.setStartTime(System.currentTimeMillis());

                GlobalExecutor
                        .scheduleTcpSuperSenseTask(new TimeOutTask(key), CONNECT_TIMEOUT_MS, TimeUnit.MILLISECONDS);
            } catch (Exception e) {
                beat.finishCheck(false, false, switchDomain.getTcpHealthParams().getMax(),
                        "tcp:error:" + e.getMessage());

                if (channel != null) {
                    try {
                        channel.close();
                    } catch (Exception ignore) {
                    }
                }
            }

            return null;
        }
    }
}
```







## 服务发现源码

Nacos的服务发现分为两种模式：

1. 主动拉取模式(push模式)：消费者定期主动从Nacos拉取服务列表并缓存起来，再服务调用时优先读取本地缓存中的服务列表
2. 订阅模式(pull模式)：消费者订阅Nacos中的服务列表，并基于UDP协议来接收服务变更通知。当Nacos中的服务列表更新时，会发送UDP广播给所有订阅者





### 客户端

#### 定时更新服务列表

在前面看服务注册的源码时有一个类`NacosNamingService`，这个类不仅仅提供了服务注册功能，同样提供了服务发现的功能

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719221041344-1066304952.png" alt="image-20230719221039340" />



通过下面的思路去找也行

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719230144158-472332622.png" alt="image-20230719230141716" />



所有的getAllInstances重载方法都进入了下面的方法：

```java
public class NacosNamingService implements NamingService {

	@Override
    public List<Instance> getAllInstances(String serviceName, String groupName, List<String> clusters,
            boolean subscribe) throws NacosException {
        
        ServiceInfo serviceInfo;
        // 是否需要订阅服务信息 默认true
        if (subscribe) {
            // 订阅服务信息
            serviceInfo = hostReactor.getServiceInfo(NamingUtils.getGroupedName(serviceName, groupName),
                    StringUtils.join(clusters, ","));
        } else {
           // 直接去Nacos中拉取服务信息
            serviceInfo = hostReactor
                    .getServiceInfoDirectlyFromServer(NamingUtils.getGroupedName(serviceName, groupName),
                            StringUtils.join(clusters, ","));
        }
        List<Instance> list;
        // 从服务信息中获取实例列表并返回
        if (serviceInfo == null || CollectionUtils.isEmpty(list = serviceInfo.getHosts())) {
            return new ArrayList<Instance>();
        }
        return list;
    }
}
```



##### HostReactor#getServiceInfo() 订阅服务信息

进入上一节的hostReactor.getServiceInfo()

```java
public class HostReactor implements Closeable {

	public ServiceInfo getServiceInfo(final String serviceName, final String clusters) {

        // key = name + "@@" + clusters
        String key = ServiceInfo.getKey(serviceName, clusters);
        
        // 读取本地服务列表的缓存，缓存是一个Map，格式：Map<String, ServiceInfo>
        ServiceInfo serviceObj = getServiceInfo0(serviceName, clusters);
        
		// 判断本地缓存是否存在
        if (null == serviceObj) {
            // 不存在，直接创建新的ServiceInfo 放入缓存
            serviceObj = new ServiceInfo(serviceName, clusters);
            serviceInfoMap.put(serviceObj.getKey(), serviceObj);
            
            // 放入待更新的服务列表（updatingMap）中
            updatingMap.put(serviceName, new Object());
            // 立即更新服务列表：此方法中的逻辑就是立刻从Nacos中获取
            updateServiceNow(serviceName, clusters);
            // 从待更新服务列表中删除已更新的服务
            updatingMap.remove(serviceName);
            
        } else if (updatingMap.containsKey(serviceName)) { // 缓存中有，但是需要更新
            
            if (UPDATE_HOLD_INTERVAL > 0) {
                // hold a moment waiting for update finish 等待5秒，待更新完成
                synchronized (serviceObj) {
                    try {
                        serviceObj.wait(UPDATE_HOLD_INTERVAL);
                    } catch (InterruptedException e) {
                        NAMING_LOGGER
                                .error("[getServiceInfo] serviceName:" + serviceName + ", clusters:" + clusters, e);
                    }
                }
            }
        }
        
        // 本地缓存中有，则开启定时更新服务列表的功能
        scheduleUpdateIfAbsent(serviceName, clusters);
        // 返回缓存中的服务信息
        return serviceInfoMap.get(serviceObj.getKey());
    }
}
```

基本逻辑就是先从本地缓存读，根据结果来选择：

1. 如果本地缓存没有，立即去nacos读取，`updateServiceNow(serviceName, clusters)`

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719223405680-1932336321.png" alt="image-20210923161528710" />

2. 如果本地缓存有，则开启定时更新功能，并返回缓存结果：`scheduleUpdateIfAbsent(serviceName, clusters)`

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719223433368-803063075.png" alt="image-20210923161630575" />

在UpdateTask中，最终还是调用updateService方法：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719223516929-648608640.png" alt="image-20210923161752521" />

不管是立即更新服务列表，还是定时更新服务列表，最终都会执行HostReactor中的updateService()方法：

```java
public class HostReactor implements Closeable {
    
    public void updateService(String serviceName, String clusters) throws NacosException {
        ServiceInfo oldService = getServiceInfo0(serviceName, clusters);
        try {
            // 基于ServerProxy发起远程调用，查询服务列表
            String result = serverProxy.queryList(serviceName, clusters, pushReceiver.getUdpPort(), false);

            if (StringUtils.isNotEmpty(result)) {
                // 处理查询结果
                processServiceJson(result);
            }
        } finally {
            if (oldService != null) {
                synchronized (oldService) {
                    oldService.notifyAll();
                }
            }
        }
    }
```





##### NamingProxy#queryList() 发起查询服务下的实例列表的请求

进入上一节的serverProxy.queryList()

```java
public class NamingProxy implements Closeable {
    
    public String queryList(String serviceName, String clusters, int udpPort, boolean healthyOnly)
        throws NacosException {
        // 准备请求参数
        final Map<String, String> params = new HashMap<String, String>(8);
        params.put(CommonParams.NAMESPACE_ID, namespaceId);
        params.put(CommonParams.SERVICE_NAME, serviceName);
        params.put("clusters", clusters);
        params.put("udpPort", String.valueOf(udpPort));
        params.put("clientIP", NetUtils.localIP());
        params.put("healthyOnly", String.valueOf(healthyOnly));
        // 给服务端发起请求，接口地址就是：/nacos/v1/ns/instance/list
        return reqApi(UtilAndComs.nacosUrlBase + "/instance/list", params, HttpMethod.GET);
    }
```







#### 处理服务变更通知

除了定时更新服务列表的功能外，Nacos还支持服务列表变更时的主动推送功能



基本思路是：

1. 通过PushReceiver监听服务端推送的变更数据
2. 解析数据后，通过NotifyCenter发布服务变更的事件
3. InstanceChangeNotifier监听变更事件，完成对服务列表的更新



在HostReactor类的构造函数中，有非常重要的几个步骤：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719224549075-225911888.png" alt="image-20210923164145915" />





##### PushReceiver 服务端推送变更的接收器

这个类会以UDP方式接收Nacos服务端推送的服务变更数据

先看构造函数：

```java
public PushReceiver(HostReactor hostReactor) {
    try {
        this.hostReactor = hostReactor;
        // 创建 UDP客户端
        String udpPort = getPushReceiverUdpPort();
        if (StringUtils.isEmpty(udpPort)) {
            this.udpSocket = new DatagramSocket();
        } else {
            this.udpSocket = new DatagramSocket(new InetSocketAddress(Integer.parseInt(udpPort)));
        }
        // 准备线程池
        this.executorService = new ScheduledThreadPoolExecutor(1, new ThreadFactory() {
            @Override
            public Thread newThread(Runnable r) {
                Thread thread = new Thread(r);
                thread.setDaemon(true);
                thread.setName("com.alibaba.nacos.naming.push.receiver");
                return thread;
            }
        });
		// 开启线程任务，准备接收变更数据
        this.executorService.execute(this);
    } catch (Exception e) {
        NAMING_LOGGER.error("[NA] init udp socket failed", e);
    }
}
```

PushReceiver构造函数中基于线程池来运行任务。这是因为PushReceiver本身也是一个Runnable，其中的run方法业务逻辑如下：

```java
@Override
public void run() {
    while (!closed) {
        try {
            // byte[] is initialized with 0 full filled by default
            byte[] buffer = new byte[UDP_MSS];
            DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
			// 接收推送数据
            udpSocket.receive(packet);
			// 解析为json字符串
            String json = new String(IoUtils.tryDecompress(packet.getData()), UTF_8).trim();
            NAMING_LOGGER.info("received push data: " + json + " from " + packet.getAddress().toString());
			// 反序列化为对象
            PushPacket pushPacket = JacksonUtils.toObj(json, PushPacket.class);
            String ack;
            if ("dom".equals(pushPacket.type) || "service".equals(pushPacket.type)) {
                // 交给 HostReactor去处理
                hostReactor.processServiceJson(pushPacket.data);

                // send ack to server 发送ACK回执，略。。
        } catch (Exception e) {
            if (closed) {
                return;
            }
            NAMING_LOGGER.error("[NA] error while receiving push data", e);
        }
    }
}
```





##### HostReactor#processServiceJson() 通知数据的处理

通知数据的处理交给了`HostReactor`的`processServiceJson`方法：

```java
public class HostReactor implements Closeable {
    
	public ServiceInfo processServiceJson(String json) {
        // 解析出ServiceInfo信息
        ServiceInfo serviceInfo = JacksonUtils.toObj(json, ServiceInfo.class);
        String serviceKey = serviceInfo.getKey();
        if (serviceKey == null) {
            return null;
        }
        // 查询缓存中的 ServiceInfo
        ServiceInfo oldService = serviceInfoMap.get(serviceKey);

        // 如果缓存存在，则需要校验哪些数据要更新
        boolean changed = false;
        if (oldService != null) {
            // 拉取的数据是否已经过期
            if (oldService.getLastRefTime() > serviceInfo.getLastRefTime()) {
                NAMING_LOGGER.warn("out of date data received, old-t: " + oldService.getLastRefTime() + ", new-t: "
                                   + serviceInfo.getLastRefTime());
            }
            // 放入缓存
            serviceInfoMap.put(serviceInfo.getKey(), serviceInfo);

            // 中间是缓存与新数据的对比，得到newHosts：新增的实例；remvHosts：待移除的实例;
            // modHosts：需要修改的实例
            if (newHosts.size() > 0 || remvHosts.size() > 0 || modHosts.size() > 0) {
                // 发布实例变更的事件
                NotifyCenter.publishEvent(new InstancesChangeEvent(
                    serviceInfo.getName(), serviceInfo.getGroupName(),
                    serviceInfo.getClusters(), serviceInfo.getHosts()));
                DiskCache.write(serviceInfo, cacheDir);
            }

        } else {
            // 本地缓存不存在
            changed = true;
            // 放入缓存
            serviceInfoMap.put(serviceInfo.getKey(), serviceInfo);
            // 直接发布实例变更的事件
            NotifyCenter.publishEvent(new InstancesChangeEvent(
                serviceInfo.getName(), serviceInfo.getGroupName(),
                serviceInfo.getClusters(), serviceInfo.getHosts()));
            serviceInfo.setJsonFromServer(json);
            DiskCache.write(serviceInfo, cacheDir);
        }
        // 。。。
        return serviceInfo;
    }
}
```





### 服务端

#### 拉取服务列表

进入前面说的 /nacos/v1/ns/instance/list 接口中，也就是naming-nacos/controller/InstanceController#list(HttpServletRequest request)

```java
@RestController
@RequestMapping(UtilsAndCommons.NACOS_NAMING_CONTEXT + "/instance")
public class InstanceController {
    
    @GetMapping("/list")
    @Secured(parser = NamingResourceParser.class, action = ActionTypes.READ)
    public ObjectNode list(HttpServletRequest request) throws Exception {

        // 从request中获取 namespaceId、serviceName
        String namespaceId = WebUtils.optional(request, CommonParams.NAMESPACE_ID, Constants.DEFAULT_NAMESPACE_ID);
        String serviceName = WebUtils.required(request, CommonParams.SERVICE_NAME);
        NamingUtils.checkServiceNameFormat(serviceName);

        String agent = WebUtils.getUserAgent(request);
        String clusters = WebUtils.optional(request, "clusters", StringUtils.EMPTY);
        String clientIP = WebUtils.optional(request, "clientIP", StringUtils.EMPTY);
        // 获取客户端的UDP端口
        int udpPort = Integer.parseInt(WebUtils.optional(request, "udpPort", "0"));
        String env = WebUtils.optional(request, "env", StringUtils.EMPTY);
        boolean isCheck = Boolean.parseBoolean(WebUtils.optional(request, "isCheck", "false"));

        String app = WebUtils.optional(request, "app", StringUtils.EMPTY);

        String tenant = WebUtils.optional(request, "tid", StringUtils.EMPTY);

        boolean healthyOnly = Boolean.parseBoolean(WebUtils.optional(request, "healthyOnly", "false"));

        // 获取服务列表
        return doSrvIpxt(namespaceId, serviceName, agent, clusters, clientIP, udpPort, env, isCheck, app, tenant,
                healthyOnly);
    }
}
```

doSrvIpxt()的逻辑如下：

```java
@RestController
@RequestMapping(UtilsAndCommons.NACOS_NAMING_CONTEXT + "/instance")
public class InstanceController {
    /**
     * 获取服务列表
     */
    public ObjectNode doSrvIpxt(String namespaceId, String serviceName, String agent,
                                String clusters, String clientIP,
                                int udpPort, String env, boolean isCheck,
                                String app, String tid, boolean healthyOnly) throws Exception {
        ClientInfo clientInfo = new ClientInfo(agent);
        ObjectNode result = JacksonUtils.createEmptyJsonNode();
        // 获取服务列表信息
        Service service = serviceManager.getService(namespaceId, serviceName);
        long cacheMillis = switchDomain.getDefaultCacheMillis();

        // now try to enable the push
        try {
            if (udpPort > 0 && pushService.canEnablePush(agent)) {
                // 添加当前客户端 IP、UDP端口到 PushService 中
                pushService
                    .addClient(namespaceId, serviceName, clusters, agent, new InetSocketAddress(clientIP, udpPort),
                               pushDataSource, tid, app);
                cacheMillis = switchDomain.getPushCacheMillis(serviceName);
            }
        } catch (Exception e) {
            Loggers.SRV_LOG
                .error("[NACOS-API] failed to added push client {}, {}:{}", clientInfo, clientIP, udpPort, e);
            cacheMillis = switchDomain.getDefaultCacheMillis();
        }

        if (service == null) {
            // 如果没找到，返回空
            if (Loggers.SRV_LOG.isDebugEnabled()) {
                Loggers.SRV_LOG.debug("no instance to serve for service: {}", serviceName);
            }
            result.put("name", serviceName);
            result.put("clusters", clusters);
            result.put("cacheMillis", cacheMillis);
            result.replace("hosts", JacksonUtils.createEmptyArrayNode());
            return result;
        }
        // 结果的检测，异常实例的剔除等逻辑省略
        // 最终封装结果并返回 。。。

        result.replace("hosts", hosts);
        if (clientInfo.type == ClientInfo.ClientType.JAVA
            && clientInfo.version.compareTo(VersionUtil.parseVersion("1.0.0")) >= 0) {
            result.put("dom", serviceName);
        } else {
            result.put("dom", NamingUtils.getServiceName(serviceName));
        }
        result.put("name", serviceName);
        result.put("cacheMillis", cacheMillis);
        result.put("lastRefTime", System.currentTimeMillis());
        result.put("checksum", service.getChecksum());
        result.put("useSpecifiedURL", false);
        result.put("clusters", clusters);
        result.put("env", env);
        result.replace("metadata", JacksonUtils.transferToJsonNode(service.getMetadata()));
        return result;
    }
```







#### 发布服务变更的UDP通知

在上一节中，`InstanceController`中的`doSrvIpxt()`方法中，有这样一行代码：

```java
// 添加当前客户端 IP、UDP端口到 PushService 中
pushService.addClient(namespaceId, serviceName, clusters, agent,
                      new InetSocketAddress(clientIP, udpPort),
                           pushDataSource, tid, app);
```

就是把消费者的UDP端口、IP等信息封装为一个PushClient对象，存储PushService中。方便以后服务变更后推送消息

PushService类本身实现了`ApplicationListener`接口：这个是事件监听器接口，监听的是ServiceChangeEvent（服务变更事件）

当服务列表变化时，就会通知我们：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230719231717066-1655705443.png" alt="image-20210923183017424" />

















# Feign远程调用

## Feign与OpenFeign是什么？

Feign是`Netflix`开发的`声明式、模板化`的HTTP客户端， 在 RestTemplate 的基础上做了进一步的封装，Feign可以帮助我们更快捷、优雅地调用HTTP API。具有可插入注解支持，包括Feign注解和JAX-RS注解，通过 Feign，我们只需要声明一个接口并通过注解进行简单的配置（类似于 Dao 接口上面的 Mapper 注解一样）即可实现对 HTTP 接口的绑定；通过 Feign，我们可以像调用本地方法一样来调用远程服务，而完全感觉不到这是在进行远程调用



OpenFeign全称Spring Cloud OpenFeign，2019 年 Netflix 公司宣布 Feign 组件正式进入停更维护状态，于是 Spring 官方便推出了一个名为 OpenFeign 的组件作为 Feign 的替代方案。基于Netflix feign实现，是一个**声明式**的http客户端，整合了`Spring Cloud Ribbon`，除了支持netflix的feign注解之外，增加了对Spring MVC注释的支持，OpenFeign 的 @FeignClient 可以解析SpringMVC的 @RequestMapping 注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务

- **声明式·：** 即只需要将调用服务需要的东西声明出来，剩下就不用管了，交给feign即可



> Spring Cloud Finchley 及以上版本一般使用 OpenFeign 作为其服务调用组件。由于 OpenFeign 是在 2019 年 Feign 停更进入维护后推出的，因此大多数 2019 年及以后的新项目使用的都是 OpenFeign，而 2018 年以前的项目一般使用 Feign





## OpenFeign 常用注解

使用 OpenFegin 进行远程服务调用时，常用注解如下表：

| 注解                    | 说明                                                         |
| ----------------------- | ------------------------------------------------------------ |
| **@FeignClient**        | 该注解用于通知 OpenFeign 组件对 @RequestMapping 注解下的接口进行解析，并通过动态代理的方式产生实现类，实现负载均衡和服务调用。 |
| **@EnableFeignClients** | 该注解用于开启 OpenFeign 功能，当 Spring Cloud 应用启动时，OpenFeign 会扫描标有 @FeignClient 注解的接口，生成代理并注册到 Spring 容器中。 |
| @RequestMapping         | Spring MVC 注解，在 Spring MVC 中使用该注解映射请求，通过它来指定控制器（Controller）可以处理哪些 URL 请求，相当于 Servlet 中 web.xml 的配置。 |
| @GetMapping             | Spring MVC 注解，用来映射 GET 请求，它是一个组合注解，相当于 @RequestMapping(method = RequestMethod.GET) 。 |
| @PostMapping            | Spring MVC 注解，用来映射 POST 请求，它是一个组合注解，相当于 @RequestMapping(method = RequestMethod.POST) 。 |







## Feign VS OpenFeign 

### 相同点

Feign 和 OpenFegin 具有以下相同点：

1. Feign 和 OpenFeign 都是 Spring Cloud 下的远程调用和负载均衡组件
2. Feign 和 OpenFeign 作用一样，都可以实现服务的远程调用和负载均衡
3. Feign 和 OpenFeign 都对 Ribbon 进行了集成，都利用 Ribbon 维护了可用服务清单，并通过 Ribbon 实现了客户端的负载均衡
4. Feign 和 OpenFeign 都是在服务消费者（客户端）定义服务绑定接口并通过注解的方式进行配置，以实现远程服务的调用



### 不同点

Feign 和 OpenFeign 具有以下不同：

1. Feign 和 OpenFeign 的依赖项不同，Feign 的依赖为 spring-cloud-starter-feign，而 OpenFeign 的依赖为 spring-cloud-starter-openfeign
2. Feign 和 OpenFeign 支持的注解不同，Feign 支持 Feign 注解和 JAX-RS 注解，但不支持 Spring MVC 注解；OpenFeign 除了支持 Feign 注解和 JAX-RS 注解外，还支持 Spring MVC 注解



## 入手OpenFeign

OpenFeign是Feign的增强版，使用时将依赖换一下，然后注意一下二者能支持的注解的区别即可



**1、依赖**：在“服务消费方”添加如下依赖

```xml
<!--openfeign的依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>



<!--Feign的依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

**2、启动类加入如下注解：**在“服务消费方”启动类添加

```java
@EnableFeignClients     /*开启feign客户端功能*/
```

**3、创建接口，并使用 `@org.springframework.cloud.openfeign.FeignClient` 注解：**这种方式相当于 `DAO`

```java
/**
 * @FeignClient("USER-SERVICE")
 * 
 * Spring Cloud 应用在启动时，OpenFeign 会扫描标有 @FeignClient 注解的接口生成代理，并注人到 Spring 容器中
 *
 * 参数为要调用的服务名，这里的服务名区分大小写
 */

@FeignClient("USER-SERVICE")
public interface FeignClient {
    /**
     * 支持SpringMVC的所有注解
     */
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") long id);
}
```

在编写服务绑定接口时，需要注意以下 2 点：

1. 在 @FeignClient 注解中，value 属性的取值为：服务提供者的服务名，即服务提供者配置文件(application.yml）中 spring.application.name 的值
2. 接口中定义的每个方法都与 服务提供者 中 Controller 定义的服务方法对应



**4、在需要调用3中服务与方法的地方进行调用**

```java
import com.zixieqing.order.client.FeignClient;
import com.zixieqing.order.entity.Order;
import com.zixieqing.order.entity.User;
import com.zixieqing.order.mapper.OrderMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * <p>@description  : order服务
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Service
public class OrderService {
   /* @Autowired
    private RestTemplate restTemplate;*/

    @Autowired
    private FeignClient feignClient;

    @Autowired
    private OrderMapper orderMapper;

    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        
       /* // 2、远程调用服务的url 此处直接使用服务名，不用ip+port
        // 原因是底层有一个LoadBalancerInterceptor，里面有一个intercept()，后续玩负载均衡Ribbon会看到
        String url = "http://USER-SERVICE/user/" + order.getUserId();
        // 2.1、利用restTemplate调用远程服务，封装成user对象
        User user = restTemplate.getForObject(url, User.class); */

        // 2、使用feign来进行远程调研
        User user = feignClient.findById(order.getUserId());
        // 3、给oder设置user对象值
        order.setUser(user);
        // 4.返回
        return order;
    }
}

```









## OpenFeign自定义配置

Feign可以支持很多的自定义配置，如下表所示：

| 类型                   | 作用             | 说明                                                         |
| ---------------------- | ---------------- | ------------------------------------------------------------ |
| **feign.Logger.Level** | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL<br />1、NONE：默认的，不显示任何日志<br />2、BASIC：仅记录请求方法、URL、响应状态码及执行时间<br />3、HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息<br />4、FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据 |
| feign.codec.Decoder    | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为Java对象       |
| feign.codec.Encoder    | 请求参数编码     | 将请求参数编码，便于通过http请求发送                         |
| feign. Contract        | 支持的注解格式   | 默认是SpringMVC的注解                                        |
| feign. Retryer         | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试       |

一般情况下，默认值就能满足我们使用，如果要自定义时，只需要创建自定义的 `@Bean` 覆盖默认Bean即可







### 配置日志增强

这个有4种配置方式，局部配置（2种=YAML+代码实现）、全局配置（2种=YAML+代码实现）



**1、YAML实现**

1. 基于YAML文件修改Feign的日志级别可以针对单个服务：即局部配置

```yaml
feign:  
  client:
    config: 
      userservice: # 针对某个微服务的配置
        loggerLevel: FULL #  日志级别
```

2. 也可以针对所有服务：即全局配置

```yaml
feign:  
  client:
    config: 
      default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```

**2、代码实现**

也可以基于Java代码来修改日志级别，先声明一个类，然后声明一个Logger.Level的对象：

```java
/** 
 * 注：这里可以不用加 @Configuration 注解
 * 因为要么在启动类 @EnableFeignClients 注解中进行声明这个配置类
 * 要么在远程服务调用的接口的 @FeignClient 注解中声明该配置
 */
public class DefaultFeignConfiguration  {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC; // 日志级别为BASIC
    }
}
```

1. 如果要**全局生效**，将其放到启动类的 `@EnableFeignClients` 这个注解中：

```java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration .class) 
```

2. 如果是**局部生效**，则把它放到对应的 `@FeignClient` 这个注解中：

```java
@FeignClient(value = "userservice", configuration = DefaultFeignConfiguration .class) 
```







### 配置客户端

Feign底层发起http请求，依赖于其它的框架。其底层客户端实现包括：

1. URLConnection：默认实现，不支持连接池
2. Apache HttpClient ：支持连接池
3. OKHttp：支持连接池





#### 替换为Apache HttpClient

**1、在服务消费方添加依赖**

```xml
<!--httpClient的依赖 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

**2、在YAML中开启客户端和配置连接池**

```yaml
feign:
  httpclient:
    # 开启feign对HttpClient的支持  默认值就是true，即 导入对应客户端依赖之后就开启了，但为了提高代码可读性，还是显示声明比较好
    enabled: true
    # 最大的连接数
    max-connections: 200
    # 每个路径最大连接数
    max-connections-per-route: 50
    # 链接超时时间
    connection-timeout: 2000
    # 存活时间
    time-to-live: 900
```



验证：在FeignClientFactoryBean中的loadBalance方法中打断点：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230603221112385-452208089.png" alt="image-20210714185925910" />

Debug方式启动服务消费者，可以看到这里的client底层就是Apache HttpClient：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230603221111983-216336226.png" alt="image-20210714190041542" />







## Feign的失败处理

业务失败后，不能直接报错，而应该返回用户一个友好提示或者默认结果，这个就是失败降级逻辑

给FeignClient编写失败后的降级逻辑

1. 方式一：FallbackClass，无法对远程调用的异常做处理
2. 方式二：FallbackFactory，可以对远程调用的异常做处理。一般选择这种





### 使用FallbackFactory进行失败降级

1. 在定义Feign-Client的地方创建失败逻辑处理

   ```java
   package com.zixieqing.feign.fallback;
   
   import com.zixieqing.feign.clients.UserClient;
   import com.zixieqing.feign.pojo.User;
   import feign.hystrix.FallbackFactory;
   import lombok.extern.slf4j.Slf4j;
   
   /**
    * userClient失败时的降级处理
    *
    * <p>@author       : ZiXieqing</p>
    */
   
   @Slf4j
   public class UserClientFallBackFactory implements FallbackFactory<UserClient> {
       @Override
       public UserClient create(Throwable throwable) {
           return new UserClient() {
               /**
                * 重写userClient中的方法，编写失败时的降级逻辑
                */
               @Override
               public User findById(Long id) {
                   log.info("userClient的findById()在进行 id = {} 时失败", id);
                   return new User();
               }
           };
       }
   }
   ```

2. 将定义的失败逻辑类丢给Spring容器托管

   ```java
   @Bean
   public UserClientFallBackFactory userClientFallBackFactory() {
       return new UserClientFallBackFactory();
   }
   ```

3. 在对应的Feign-Client中使用fallbackFactory回调函数

   ```java
   package com.zixieqing.feign.clients;
   
   
   import com.zixieqing.feign.fallback.UserClientFallBackFactory;
   import com.zixieqing.feign.pojo.User;
   import org.springframework.cloud.openfeign.FeignClient;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.PathVariable;
   
   @FeignClient(value = "userservice",fallbackFactory = UserClientFallBackFactory.class)
   public interface UserClient {
   
       @GetMapping("/user/{id}")
       User findById(@PathVariable("id") Long id);
   }
   ```

4. 调用，失败时就会进入自定义的失败逻辑中

   ```java
   package com.zixieqing.order.service;
   
   import com.zixieqing.feign.clients.UserClient;
   import com.zixieqing.feign.pojo.User;
   import com.zixieqing.order.mapper.OrderMapper;
   import com.zixieqing.order.pojo.Order;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Service;
   
   @Service
   public class OrderService {
   
       @Autowired
       private OrderMapper orderMapper;
   
       @Autowired
       private UserClient userClient;
   
       public Order queryOrderById(Long orderId) {
           // 1.查询订单
           Order order = orderMapper.findById(orderId);
           // 2.用Feign远程调用
           User user = userClient.findById(14321432143L);	// 传入错误 id=14321432143L 模拟错误
           // 3.封装user到Order
           order.setUser(user);
           // 4.返回
           return order;
       }
   }
   ```

   <img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701213938464-410457792.png" alt="image-20230701213914563" />









# Gateway 网关

在微服务架构中，一个系统往往由多个微服务组成，而这些服务可能部署在不同机房、不同地区、不同域名下。这种情况下，客户端（例如浏览器、手机、软件工具等）想要直接请求这些服务，就需要知道它们具体的地址信息，如 IP 地址、端口号等



这种客户端直接请求服务的方式存在以下问题：

1. 当服务数量众多时，客户端需要维护大量的服务地址，这对于客户端来说，是非常繁琐复杂的
2. 在某些场景下可能会存在跨域请求的问题
3. 身份认证的难度大，每个微服务需要独立认证



我们可以通过 API 网关来解决这些问题，下面就让我们来看看什么是 API 网关





## API 网关

API 网关是一个搭建在客户端和微服务之间的服务，我们可以在 API 网关中处理一些非业务功能的逻辑，例如权限验证、监控、缓存、请求路由等

API 网关就像整个微服务系统的门面一样，是系统对外的唯一入口。有了它，客户端会先将请求发送到 API 网关，然后由 API 网关根据请求的标识信息将请求转发到微服务实例



<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604173637373-1731467261.png" alt="img" />


对于服务数量众多、复杂度较高、规模比较大的系统来说，使用 API 网关具有以下好处：

1. 客户端通过 API 网关与微服务交互时，客户端只需要知道 API 网关地址即可，而不需要维护大量的服务地址，简化了客户端的开发
2. 客户端直接与 API 网关通信，能够减少客户端与各个服务的交互次数
3. 客户端与后端的服务耦合度降低
4. 节省流量，提高性能，提升用户体验
5. API 网关还提供了安全、流控、过滤、缓存、计费以及监控等 API 管理功能





常见的 API 网关实现方案主要有以下 5 种：

1. Spring Cloud Gateway
2. Spring Cloud Netflix Zuul
3. Kong
4. Nginx+Lua
5. Traefik







## 认识Spring Cloud Gateway

Spring Cloud Gateway 是 Spring Cloud 团队基于 Spring 5.0、Spring Boot 2.0 和 Project Reactor 等技术开发的高性能 API 网关组件

Spring Cloud Gateway 旨在提供一种简单而有效的途径来发送 API，并为它们提供横切关注点，例如：安全性，监控/指标和弹性



> Spring Cloud Gateway 是基于 WebFlux 框架实现的，而 WebFlux 框架底层则使用了高性能的 Reactor 模式通信框架 Netty





## Spring Cloud Gateway 核心概念

**Spring Cloud Gateway 最主要的功能就是路由转发**，而在定义转发规则时主要涉及了以下三个核心概念，如下表：

| 核心概念       | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| Route 路由     | 网关最基本的模块。它由一个 ID、一个目标 URI、一组断言（Predicate）和一组过滤器（Filter）组成 |
| Predicate 断言 | 路由转发的判断条件，我们可以通过 Predicate 对 HTTP 请求进行匹配，如请求方式、请求路径、请求头、参数等，如果请求与断言匹配成功，则将请求转发到相应的服务 |
| Filter 过滤器  | 过滤器，我们可以使用它对请求进行拦截和修改，还可以使用它对上文的响应进行再处理 |

> 注意：其中 Route 和 Predicate 必须同时声明





网关的**核心功能特性**：

1. 请求路由
2. 权限控制
3. 限流



架构图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604174001899-867105716.png" alt="image-20210714210131152" />



**权限控制**：网关作为微服务入口，需要校验用户是否有请求资格，如果没有则进行拦截

**路由和负载均衡**：一切请求都必须先经过gateway，但网关不处理业务，而是根据指定规则，把请求转发到某个微服务，这个过程叫做路由。当然路由的目标服务有多个时，还需要做负载均衡

**限流**：当请求流量过高时，在网关中按照下游的微服务能够接受的速度来放行请求，避免服务压力过大









## Gateway 的工作流程

Spring Cloud Gateway 工作流程如下图:



<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604174027529-1077440941.png" alt="Spring Cloud Gateway 工作流程" />


Spring Cloud Gateway 工作流程说明如下：

1. 客户端将请求发送到 Spring Cloud Gateway 上
2. Spring Cloud Gateway 通过 Gateway Handler Mapping 找到与请求相匹配的路由，将其发送给 Gateway Web Handler
3. Gateway Web Handler 通过指定的过滤器链（Filter Chain），将请求转发到实际的服务节点中，执行业务逻辑返回响应结果
4. 过滤器之间用虚线分开是因为过滤器可能会在转发请求之前（pre）或之后（post）执行业务逻辑
5. 过滤器（Filter）可以在请求被转发到服务端前，对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等
6. 过滤器可以在响应返回客户端之前，对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量监控等
7. 响应原路返回给客户端



总而言之，客户端发送到 Spring Cloud Gateway 的请求需要通过一定的匹配条件，才能到达真正的服务节点。在将请求转发到服务进行处理的过程前后（pre 和 post），我们还可以对请求和响应进行一些精细化控制。

Predicate 就是路由的匹配条件，而 Filter 就是对请求和响应进行精细化控制的工具。有了这两个元素，再加上目标 URI，就可以实现一个具体的路由了



当然，要是再加上前面已经玩过的东西的流程就变成下面的样子了：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604175754741-755292829.png" alt="image-20210714211742956" />





## Predicate 断言

Spring Cloud Gateway 通过 Predicate 断言来实现 Route 路由的匹配规则。简单点说，Predicate 是路由转发的判断条件，请求只有满足了 Predicate 的条件，才会被转发到指定的服务上进行处理。

使用 Predicate 断言需要注意以下 3 点：

1. Route 路由与 Predicate 断言的对应关系为“一对多”，一个路由可以包含多个不同断言条件
2. 一个请求想要转发到指定的路由上，就必须同时匹配路由上的所有断言
3. 当一个请求同时满足多个路由的断言条件时，请求只会被首个成功匹配的路由转发



<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604174131390-660098036.png" alt="img" />







常见的 Predicate 断言如下表：假设转发的 URI 为 http://localhost:8001

| 断言       | 示例                                                         | 说明                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `Path`     | - Path=/dept/list/**                                         | 当请求路径与 /dept/list/ 匹配时，该请求才能被转发到 http://localhost:8001 上 |
| `Before`   | - Before=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]        | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之前的请求，才会被转发到 http://localhost:8001 上 |
| `After`    | - After=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]         | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之后的请求，才会被转发到 http://localhost:8001 上 |
| `Between`  | - Between=2021-10-20T15:18:33.226+08:00[Asia/Shanghai],2021-10-20T15:23:33.226+08:00[Asia/Shanghai] | 在 2021 年 10 月 20 日 15 时 18 分 33.226 秒 到 2021 年 10 月 20 日 15 时 23 分 33.226 秒之间的请求，才会被转发到 http://localhost:8001 服务器上 |
| Cookie     | - Cookie=name,www.cnblogs.com/xiegongzi                      | 携带 Cookie 且 Cookie 的内容为 name=www.cnblogs.com/xiegongzi 的请求，才会被转发到 http://localhost:8001 上 |
| `Header`   | - Header=X-Request-Id,\d+                                    | 请求头上携带属性 X-Request-Id 且属性值为整数的请求，才会被转发到 http://localhost:8001 上 |
| `Method`   | - Method=GET                                                 | 只有 GET 请求才会被转发到 http://localhost:8001 上           |
| `Host`     | -  Host=.somehost.org,.anotherhost.org                       | 请求必须是访问.somehost.org和.anotherhost.org这两个host（域名）才会被转发到 http://localhost:8001 上 |
| Query      | - Query=name                                                 | 请求参数必须包含指定参数(name)，才会被转发到 http://localhost:8001 上 |
| RemoteAddr | - RemoteAddr=192.168.1.1/24                                  | 请求者的ip必须是指定范围（192.168.1.1 到 192.168.1.24)       |
| Weight     | <img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230605120548651-1280651580.png" alt="image-20230605120547194" /> | 权重处理weight,有两个参数：group和weight(一个整数)<br />如示例中表示：分80%的流量给weihthigh.org |

上表中这些也叫“**Predicate断言工厂**”，我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断的条件

例如 Path=/user/** 是按照路径匹配，这个规则是由`org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory`类来处理的





## 入手Gateway

新建一个Maven项目，依赖如下：

```xml
<!--Nacos服务发现-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!--网关-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```



YAML配置文件内容如下：

```yaml
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: userservice # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址，这是一种写法，常用的是下面这种
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 按路径匹配，只要以 /user/ 开头就符合要求
        - id: orderservice
          uri: lb://orderservice
          predicates:
            - Path=/order/**
```

经过如上方式，就简单搭建了Gateway网关，启动、访问 localhost:10010/user/id 或 localhost:10010/order/id 即可







## filter 过滤器

通常情况下，出于安全方面的考虑，服务端提供的服务往往都会有一定的校验逻辑，例如用户登陆状态校验、签名校验等

在微服务架构中，系统由多个微服务组成，所以这些服务都需要这些校验逻辑，此时我们就可以将这些校验逻辑写到 Spring Cloud Gateway 的 Filter 过滤器中



Filter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230607230146990-155503942.png" alt="image-20210714212312871" />



Spring Cloud Gateway 提供了以下两种类型的过滤器，可以对请求和响应进行精细化控制

| 过滤器类型 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Pre 类型   | 这种过滤器在请求被转发到微服务“之前”可以对请求进行拦截和修改，如参数校验、权限校验、流量监控、日志输出以及协议转换等操作 |
| Post 类型  | 这种过滤器在微服务对请求做出响应“之后”可以对响应进行拦截和再处理，如修改响应内容或响应头、日志输出、流量监控等 |


按照作用范围划分，Spring Cloud gateway 的 Filter 可以分为 2 类：

1. GatewayFilter：应用在“单个路由”或者“一组路由”上的过滤器
2. GlobalFilter：应用在“所有的路由”上的过滤器







### GatewayFilter 网关过滤器

>  GatewayFilter 是 Spring Cloud Gateway 网关中提供的一种应用在“单个路由”或“一组路由”上的过滤器
>
> 它可以对单个路由或者一组路由上传入的请求和传出响应进行拦截，并实现一些与业务无关的功能，如登陆状态校验、签名校验、权限校验、日志输出、流量监控等



GatewayFilter 在配置文件(如 application.yml)中的写法与 Predicate 类似，格式如下：

```yaml
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: userservice # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址，常用写法是下面这种
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 按路径匹配，只要以/user/开头就符合要求
          filters: # gateway过滤器
            - AddRequestHeader=name, zixieqing # 添加请求头name=zixieqing
        - id: orderservice
          uri: lb://orderservice
          predicates:
            - Path=/order/**
```

想要验证的话，可以在添加路由的服务中进行获取，如上面加在了userservice中，那么验证方式如下：

```java
package com.zixieqing.user.web;

import com.zixieqing.user.entity.User;
import com.zixieqing.user.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * <p>@description  : 该类功能  user控制层
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@RestController
@RequestMapping("/user")
public class UserController {
    @Autowired
    private UserService userService;

    /**
     * 路径： /user/110
     *
     * @param id 用户id
     * @return 用户
     */
    @GetMapping("/{id}")
    public User queryById(@PathVariable("id") Long id,
                          @RequestHeader(value = "name",required = false) String name) {
        System.out.println("name = " + name);
        return userService.queryById(id);
    }
}
```



> 此种过滤器一共有37种（常用就几种），它们的用法和上面的差不多，可以多个过滤器共同使用
>
> 详细去看链接：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories



下表中列举了几种比较常用的网关过滤器：

| 路由过滤器             | 描述                                                         | 参数                                                         | 使用示例                                               |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| AddRequestHeader       | 拦截传入的请求，并在请求上添加一个指定的请求头参数           | name：需要添加的请求头参数的 key<br />value：需要添加的请求头参数的 value | - AddRequestHeader=my-request-header,1024              |
| AddRequestParameter    | 拦截传入的请求，并在请求上添加一个指定的请求参数             | name：需要添加的请求参数的 key<br />value：需要添加的请求参数的 value | - AddRequestParameter=my-request-param,c.biancheng.net |
| AddResponseHeader      | 拦截响应，并在响应上添加一个指定的响应头参数                 | name：需要添加的响应头的 key<br />value：需要添加的响应头的 value | - AddResponseHeader=my-response-header,c.biancheng.net |
| PrefixPath             | 拦截传入的请求，并在请求路径增加一个指定的前缀               | prefix：需要增加的路径前缀                                   | - PrefixPath=/consumer                                 |
| PreserveHostHeader     | 转发请求时，保持客户端的 Host 信息不变，然后将它传递到提供具体服务的微服务中 | 无                                                           | - PreserveHostHeader                                   |
| RemoveRequestHeader    | 移除请求头中指定的参数                                       | name：需要移除的请求头的 key                                 | - RemoveRequestHeader=my-request-header                |
| RemoveResponseHeader   | 移除响应头中指定的参数                                       | name：需要移除的响应头                                       | - RemoveResponseHeader=my-response-header              |
| RemoveRequestParameter | 移除指定的请求参数                                           | name：需要移除的请求参数                                     | - RemoveRequestParameter=my-request-param              |
| RequestSize            | 配置请求体的大小，当请求体过大时，将会返回 413 Payload Too Large | maxSize：请求体的大小                                        | - name: RequestSize   args:    maxSize: 5000000        |







### GlobalFilter 全局过滤器

全局过滤器的作用也是处理一切进入网关的请求和微服务响应



1. 方式一：像上面一样直接在YAML文件中配置

缺点：要是需要编写复杂的业务逻辑时会非常不方便，但是：**这种过滤器的优先级比下面一种要高**

```yaml
server:
  port: 10010 # 网关端口
spring:
  application:
    name: gateway # 服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 # nacos地址
    gateway:
      routes: # 网关路由配置
        - id: userservice # 路由id，自定义，只要唯一即可
          # uri: http://127.0.0.1:8081 # 路由的目标地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 按路径匹配，只要以 /user/ 开头就符合要求
#          filters:
#            - AddRequestHeader=name, zixieqing
        - id: orderservice
          uri: lb://orderservice
          predicates:
            - Path=/order/**
      default-filters:
        # 全局过滤器
        - AddRequestHeader=name, zixieqing
```

2. 方式二：使用代码实现，定义方式是 `implements GlobalFilter` 接口：

```java
public interface GlobalFilter {
    /**
     * 处理当前请求，有必要的话通过 GatewayFilterChain 将请求交给下一个过滤器处理
     *
     * @param exchange 请求上下文，里面可以获取Request、Response等信息
     * @param chain 用来把请求委托给下一个过滤器 
     * @return Mono<Void> 返回标示当前过滤器业务结束
     */
    Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain);
}
```

在filter中编写自定义逻辑，可以实现下列功能：

- 登录状态判断
- 权限校验
- 请求限流等



举例如下：获取和比较的就是刚刚前面在YAML中使用的  `- AddRequestHeader=name, zixieqing`

```java
package com.zixieqing.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.annotation.Order;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.List;

/**
 * <p>@description  : 自定义gateway全局路由器：请求头中有 name=zixieqing 才放行
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Order(-1)  // 这个注解和本类 implements Ordered 是一样的效果，都是返回一个整数
            // 这个整数表示当前过滤器的执行优先级，值越小优先级越高，取值范围就是 int的范围
@Component
public class MyGlobalFilter implements GlobalFilter /* , Ordered */ {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取请求头中的name
        List<String> name = exchange.getRequest().getHeaders().get("name");
        for (String value : name) {
            if ("zixieqing".equals(value))
                // 放行
                return chain.filter(exchange);

        }

        // 设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);

        // 不再执行下去，到此结束 setComplete即设置完成的意思
        return exchange.getResponse().setComplete();
    }
}
```





### 过滤器执行顺序

请求进入网关会碰到三类过滤器：当前路由的过滤器、DefaultFilter、GlobalFilter

请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链（集合）中，排序后依次执行每个过滤器：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230610170715610-1341628357.png" alt="image-20210714214228409" />



排序的规则是什么呢？

1. 每一个过滤器都必须指定一个int类型的order值，**order值越小，优先级越高，执行顺序越靠前**
2. GlobalFilter通过实现Ordered接口，或者添加 @Order 注解由我们自己指定order值
3. 路由过滤器和defaultFilter的order值由Spring指定，默认是按照声明顺序从1递增
4. 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter 的顺序执行



详细内容，可以查看源码：

1. `org.springframework.cloud.gateway.route.RouteDefinitionRouteLocator#getFilters()`方法是先加载defaultFilters，然后再加载某个route的filters，最后合并
2. `org.springframework.cloud.gateway.handler.FilteringWebHandler#handle()`方法会加载全局过滤器，与前面的过滤器合并后根据order排序，组织过滤器链









## 网关跨域问题

跨域：域名不一致就是跨域，主要包括：

- 域名不同： www.taobao.com 和 www.taobao.org 和 www.jd.com 和 miaosha.jd.com

- 域名相同，端口不同：localhost:8080 和 localhost8081



跨域问题：浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题

解决方案：CORS，了解CORS可以去这里 https://www.ruanyifeng.com/blog/2016/04/cors.html





### 全局跨域

解决方式：在gateway服务的 application.yml 文件中，添加下面的配置：

```yaml
spring:
  cloud:
    gateway:
      globalcors: # 全局的跨域处理
        # 解决options请求被拦截问题。CORS跨域浏览器会问服务器可不可以跨域，而这种请求是options，网关默认会拦截这种请求
        add-to-simple-url-handler-mapping: true
        corsConfigurations:
          '[/**]':	# 拦截哪些请求，此处为拦截所有请求
            allowedOrigins: # 允许哪些网站的跨域请求 
              - "http://localhost:8090"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期是多少秒。每次跨域都要询问一次服务器，这会浪费一定性能，因此加入有效期
```





### 局部跨域

route配置允许将 CORS 作为元数据直接应用于路由，例如下面的配置：

```yaml
spring:
  cloud:
    gateway:
      routes:
      - id: cors_route
        uri: https://example.org
        predicates:
        - Path=/service/**
        metadata:
          cors
            allowedOrigins: '*'
            allowedMethods:
              - GET
              - POST
            allowedHeaders: '*'
            maxAge: 30
```



> **注意：**若是 `predicates` 中的 `Path` 没有的话，那么默认使用 `/**`







# Docker

## 安装docker

**1、安装yum工具**

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2 --skip-broken
```

**2、更新本地镜像源为阿里镜像源**：docker官方镜像仓库网速较差，我们需要设置国内镜像服务

```shell
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
```

> 参考阿里云的镜像加速文档：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors



**3、安装docker**

```shell
yum install -y docker-ce
```

**4、关闭防火墙**

Docker应用需要用到各种端口，逐一去修改防火墙设置。非常麻烦，因此可以选择直接关闭防火墙，也可以开放需要的端口号，这里采用直接关闭防火墙

```shell
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
```

**5、启动docker服务**

```shell
systemctl start docker
```

**6、开启开机自启**

```shell
systemctl enable docker
```

**7、测试是否成功**

```shell
docker ps
```

![截图](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230612181822623-407818994.png)

出现这个页面，则：说明安装成功

或者是：
```shell
docker -v		# 出现docker版本号也表示成功
```







## 镜像名称

首先来看下镜像的名称组成：

- 镜名称一般分两部分组成：[repository]:[tag]。
- 在没有指定tag时，默认是latest，代表最新版本的镜像

如图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230612184039214-132749515.png" alt="image-20210731155141362" />

这里的mysql就是repository，5.7就是tag，合一起就是镜像名称，代表5.7版本的MySQL镜像。



## Docker命令

Docker仓库地址(即dockerHub)：https://hub.docker.com

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230612185012119-380491969.png" />







常见的镜像操作命令如图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230612184112569-603279439.png" alt="image-20210731155649535" />



```shell
# 拉取镜像
docker pull 镜像名称

# 查看全部镜像
docker images

# 删除镜像
docker rmi 镜像ID

# 将本地的镜像导出 
docker save -o 导出的路径 镜像id

# 加载本地的镜像文件 
docker load -i 镜像文件

# 修改镜像名称 
docker tag 镜像id 新镜像名称:版本




# 简单运行操作 
docker run 镜像ID | 镜像名称
# docker run	指的是创建一个容器并运行

# 跟参数的运行
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像ID | 镜像名称
# 如：docker run -d -p 8081:8080 --name tomcat b8
# -d：代表后台运行容器 
# -p 宿主机端口:容器端口		为了映射当前Linux的端口和容器的端口 
# --name 容器名称：指定容器的名称

# 查看运行的容器
docker ps [-qa]
# -a：查看全部的容器，包括没有运行 
# -q：只查看容器的标识

# 查看日志
docker logs -f 容器id 
# -f：可以滚动查看日志的最后几行

# 进入容器内部
docker exec -it 容器id bash 
# docker exec 进入容器内部，执行一个命令
# -it	给当前进入的容器创建一个标准输入、输出终端，允许我们与容器交互
# bash	进入容器后执行的命令，bash是一个Linux终端交互命令
# 退出容器：exit

# 将宿主机的文件复制到容器内部的指定目录
docker cp 文件名称 容器id:容器内部路径 
docker cp index.html 982:/usr/local/tomcat/webapps/ROOT

# 重新启动容器
docker restart 容器id

# 启动停止运行的容器
docker start 容器id

# 停止指定的容器（删除容器前，需要先停止容器）
docker stop 容器id

# 停止全部容器
docker stop $(docker ps -qa)

# 删除指定容器
docker rm 容器id

# 删除全部容器
docker rm $(docker ps -qa)




# ==================数据卷volume========================

# 创建数据卷
docker volume create 数据卷名称
# 创建数据卷之后，默认会存放在一个目录下 /var/lib/docker/volumes/数据卷名称/_data

# 查看数据卷详情
docker volume inspect 数据卷名称

# 查看全部数据卷
docker volume ls

# 删除指定数据卷
docker volume rm 数据卷名称



# Docker容器映射数据卷==========>有两种方式：
# 1、通过数据卷名称映射，如果数据卷不存在。Docker会帮你自动创建，会将容器内部自带的文件，存储在默认的存放路径中

# 通过数据卷名称映射
docker run -v 数据卷名称:容器内部的路径 镜像id

# 2、通过路径映射数据卷，直接指定一个路径作为数据卷的存放位置。但是这个路径不能是空的 - 重点掌握的一种
# 通过路径映射数据卷 
docker run -v 宿主机中自己创建的路径:容器内部的路径 镜像id

# 如：docker run -d -p 8081:8080 --name tomcat -v[volume] /opt/tocmat/usr/local/tocmat/webapps b8
```

数据卷挂载和目录直接挂载的区别：

1. 数据卷挂载耦合度低，由docker来管理目录且目录较深，所以不好找
2. 目录挂载耦合度高，需要我们自己管理目录，不过很容易查看





> 更多命令通过 `docker -help` 或 `docker 某指令 --help` 来学习





## 虚悬镜像

> 虚悬镜像：指的是仓库名、标签都是 `<none>` ，即俗称dangling image
>
> 出现的原因：在构建镜像或删除镜像时出现了某些错误，从而导致仓库名和标签都是 `<none>`



事故重现：

```shell
# 1、创建Dockerfile文件，注：必须是大写的D
vim Dockerfile

# 2、编写如下内容，下面这两条指令看不懂没关系，下一节会解释
FROM ubuntu
CMD echo "执行完成"

# 3、构建镜像
docker build .

# 4、查看镜像
docker images
```

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230613112921076-1538350151.png" alt="image-20230613112920030" />



这种东西就是“虚悬镜像”，就是个残次品，不是一定会出事，也不是一定不会出事，但一旦有，就很可能会导致项目出问题，因此绝不可以出现这种镜像，一旦有就最好删掉

```shell
# 查看虚悬镜像有哪些
docker image ls -f dangling=true

# 删除所有的虚悬镜像
docker image prune
```





## Dockerfile 自定义镜像

玩这个玩的就是三步骤，重现虚悬镜像时已经见了一下：

1. 编辑Dockerfile文件          注：必须是大写D
2. docker build构建成Docker镜像
3. 启动构建的Docker镜像





### Dockerfile文件中的关键字

**官网：** https://docs.docker.com/engine/reference/builder/





| 指令       | 含义                                                         | 解读                                                         | 示例                                                         |
| :--------- | :----------------------------------------------------------- | ------------------------------------------------------------ | :----------------------------------------------------------- |
| #          | 注释                                                         | 字面意思                                                     | # 注释内容                                                   |
| FROM       | 指定当前新镜像是基于哪个基础镜像，即：基于哪个镜像继续升级<br />“必须放在第一行” | 类似于对“某系统”进行升级，添加新功能<br />这里的“某系统”就是基础镜像 | FROM centos:7                                                |
| MAINTAINER | 镜像的作者和邮箱                                             | 和IDEA中写一个类或方法时留下自己姓名和邮箱类似               | MAINTAINER  zixq<zixq8@qq.com>                               |
| RUN        | 容器“运行时”需要执行的命令<br />RUN是在进行docker build时执行 | 在进行docker build时会安装一些命令或插件，亦或输出一句话用来提示进行到哪一步了/当前这一步是否成功了 | 有两种格式：<br />1、shell格式：RUN <命令行命令>  如：RUN echo “Successfully built xxxx” 或者是 RUN yum -y imstall vim<br />这种等价于在终端中执行shell命令<br /><br />2、exec格式：RUN {“可执行文件”,”参数1”,”参数2”} <br />如：RUN {“./startup.cmd”,”-m”,”standalone”} <br />等价于 startup.cmd -m standalone |
| EXPOSE     | 当前容器对外暴露出的端口                                     | 字面意思。容器自己想设定的端口，docker要做宿主机和容器内端口映射咯 | EXPOSE 80                                                    |
| WORKDIR    | 指定在容器创建后，终端默认登录进来时的工作目录               | 虚拟机进入时默认不就是 `~` 或者 Redis中使用Redis -cli登录进去之后不是也有默认路径吗 | WORKDIR /usr/local<br /><br />或<br />WORKDIR /              |
| USER       | 指定该镜像以什么样的用户去执行，若不进行指定，则默认用 root 用户<br /><br />这玩意儿一般都不会特意去设置 | 时空见惯了，略过                                             | USER root                                                    |
| ENV        | 是environment的缩写，即：用来在镜像构建过程中设置环境变量    | 可以粗略理解为定义了一个 key=value 形式的常量，这个常量方便后续某些地方直接进行引用 | ENV MY_NAME="John Doe"<br /><br />或形象点<br />ENV JAVA_HOME=/usr/local/java |
| VOLUME     | 数据卷，进行数据保存和持久化                                 | 和前面docker中使用 `-v` 数据卷是一样的                       | VOLUME /myvol                                                |
| COPY       | 复制，拷贝目录和文件到镜像中                                 |                                                              | COPY test.txt relativeDir/<br /><br />注：这里的目标路径或目标文件relativeDir 不用事先创建，会自动创建 |
| ADD        | 将宿主机目录下的文件拷贝进镜像 且 会自动处理URL和解压tar压缩包 | 和COPY类似，就是COPY+tar文件解压这两个功能组合               | ADD test.txt /mydir/<br /><br />或形象点<br />ADD target/tomcat-stuffed-1.0.jar /deployments/app.jar |
| CMD        | 指定容器“启动后”要干的事情<br /><br />Dockerfile中可以有多个CMD指令，“但是：只有最后一个有效”<br /><br />“但可是：若Dockerfile文件中有CMD，而在执行docker run时后面跟了参数，那么就会替换掉Dockerfile中CMD的指令”，如：<br />docker run -d -p 80:80 —name tomcat 容器ID /bin/bash<br />这里用了/bin/bash参数，那就会替换掉自定义的Dockerfile中的CMD指令 |                                                              | 和RUN一样也是支持两种格式<br /><br />1、shell格式：CMD <命令> 如 CMD echo "wc，This is a test"<br /><br /><br />2、exec格式：CMD {“可执行文件”,”参数1”,”参数2”}<br /><br /><br />**和RUN的区别：**<br />CMD是docker run时运行<br />RUN是docker build时运行 |
| ENTRYPOINT | 也是用来指定一个容器“启动时”要运行的命令                     | 类似于CMD指令，但：ENRTYPOINT不会被docker run后面的命令覆盖，且这些命令行会被当做参数送给ENTRYPOINT指令指定的程序<br />![image-20230613022604766](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230613022606426-298443569.png) | 和CMD一样，支持两种格式<br /><br />1、shell格式：ENTRYPOINT<命令><br /><br />2、exec格式：ENTRYPOINT {“可执行文件”,”参数1”,”参数2”} |



> **注意：** 上表中指令必须是大写
>
> 再理解Dockerfile语法，直接参考Tomcat：https://github.com/apache/tomcat/blob/main/modules/stuffed/Dockerfile







### 将微服务构建为镜像部署

这个玩意儿属于云原生技术里面的，因为前面都玩了Dockerfile，所以就顺便弄一下这个



思路：

1. 创建一个微服务项目，编写自己的逻辑，通过Maven的package工具打成jar包

2. 将打成的jar包上传到自己的虚拟机中，目录自己随意

3. 创建Dockerfile文件，并编写内容，参考如下：

   ```shell
   # 基础镜像
   FROM java:8
   # 作者
   MAINTAINER zixq
   # 数据卷 在宿主机/var/lib/docker目录下创建了一个临时文件并映射到容器的/tmp
   VOLUME /tmp
   # 将jar包添加到容器中 并 更名为 zixq_dokcer.jar
   ADD docker_boot-0.0.1.jar zixq_docker.jar
   # 运行jar包
   RUN bash -c "touch /zixq_docker.jar"
   ENTRYPOINT {"java","-jar","/zixq_docker.jar"}
   # 暴露端口
   EXPOSE 8888
   ```

   **注**：Dockerfile文件和jar包最好在同一目录

4. 构建成docker镜像

   ```shell
   # docker build -t 仓库名字(REPOSITORY):标签(TAG)
   docker build -t zixq_docker:0.1 .
   # 最后有一个	点.	表示：当前目录，jar包和Dockerfile不都在当前目录吗
   ```

5. 运行镜像

   ```shell
   docker run -d -p 8888:8888 镜像ID
   
   # 注意防火墙的问题，端口是否开放或防火墙是否关闭，否则关闭/开放，然后重启docker，重现运行镜像.........
   ```

6. 浏览器访问

   ```shell
   自己虚拟机ip + 5中暴露的port + 自己微服务中的controller路径
   ```











## Docker-Compose

Docker Compose可以基于Compose文件帮我们快速的部署分布式应用，而无需手动一个个创建和运行容器！





### 安装Docker-Compose

**1、下载Docker-Compose**

```shell
# 1、安装
# 1.1、选择在线，直接官网拉取
curl -L https://github.com/docker/compose/releases/download/1.23.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 要是嫌慢的话，也可以去这个网址
curl -L https://get.daocloud.io/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 1.2、也可以选择离线安装，直接下载到本地后，上传到虚拟机 /usr/local/bin/ 路径中即可



# 2、修改文件权限,因为 /usr/local/bin/docker-compose 文件还没有执行权
chmod +x /usr/local/bin/docker-compose

# 3、检测是否成功，出现命令文档说明就表示成功了
docker-compose
```

可以再加上一个东西：Base自动补全命令

```shell
# 补全命令
curl -L https://raw.githubusercontent.com/docker/compose/1.29.1/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose

# 若是出现错误，这是因为上面这个网址域名的问题，这需要修改hosts文件
# 可以先修改hosts，然后再拉取Base自动补全命令
echo "199.232.68.133 raw.githubusercontent.com" >> /etc/hosts
```







### Docker-Compose语法

DockerCompose的详细语法参考官网：https://docs.docker.com/compose/compose-file/



其实DockerCompose文件可以看做是将多个docker run命令写到一个文件，只是语法稍有差异



Compose文件是一个文本文件(YAML格式)，通过指令定义集群中的每个容器如何运行。格式如下：

> **注：** 这YAML里面的格式要求很严格
>
> 1. 每行末尾别有空格
> 2. 别用tab缩进(在IDEA中编辑好除外，这种会自动进行转换，但偶尔会例外)，容易导致启动不起来
> 3. 注释最好像下面这样写在上面，不要像在IDEA中写在行尾，这样容易解析出错成为空格(偶尔会莫名其妙启动不起来，把注释位置改为上面又可以了)



```yaml
# docker-compose的版本，目前的版本有1.x、2.x、3.x
version: "3.2"

services:
# 就是docker run中 --name 后面的名字
  nacos:
    image: nacos/nacos-server
    environment:
# 前面玩nacos的单例模式启动
      MODE: standalone
    ports:
      - "8848:8848"
  mysql:
    image: mysql:5.7.25
    environment:
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - "$PWD/mysql/data:/var/lib/mysql"
      - "$PWD/mysql/conf:/etc/mysql/conf.d/"
# 对某微服务的配置，一般不要暴露端口，网关会协调，微服务之间是内部访问，对于用户只需暴露一个入口就行，即：网关
  xxxservice:
    build: ./xxx-service
  yyyservice:
    build: ./yyy-service
# 网关微服务配置
  gateway:
    build: ./gateway
    ports:
      - "10010:10010"
```

上面的Compose文件就描述一个项目，其中包含两个容器(对照使用 docker run -d -p 映射出来的宿主机端口:容器内暴露的端口 –name 某名字……… 命令跑某个镜像，这文件内容就是多个容器配置都在一起，最后一起跑起来而已)：

- mysql：一个基于`mysql:5.7.25`镜像构建的容器，并且挂载了两个目录
- web：一个基于`docker build`临时构建的镜像容器，映射端口是8090







### Docker-Compose的基本命令

> 在使用docker-compose的命令时，默认会在当前目录下找 docker-compose.yml 文件(这个文件里面的内容就是上一节中YAML格式的内容写法)，所以：需要让自己在创建的 docker-compose.yml 文件的当前目录中，从而来执行docker-compose相关的命令




```shell
# 1. 基于docker-compose.yml启动管理的容器
docker-compose up -d

# 2. 关闭并删除容器
docker-compose down

# 3. 开启|关闭|重启已经存在的由docker-compose维护的容器
docker-compose start|stop|restart

# 4. 查看由docker-compose管理的容器
docker-compose ps

# 5. 查看日志
docker-compose logs -f [服务名1] [服务名2]
```



> 更多命令使用 `docker-compose -help` 或 `docker-compose 某指令 --help` 查看即可



## Docker私有仓库搭建

公共仓库：像什么前面的DockerHub、DaoCloud、阿里云镜像仓库…………..



### 简化版仓库

Docker官方的Docker Registry是一个基础版本的Docker镜像仓库，具备仓库管理的完整功能，但是没有图形化界面。

搭建方式如下：

```sh
# 直接在虚拟机中执行命令即可
docker run -d \
    --restart=always \
    --name registry	\
    -p 5000:5000 \
    -v registry-data:/var/lib/registry \
    registry
```

命令中挂载了一个数据卷registry-data到容器内的 /var/lib/registry 目录，这是私有镜像库存放数据的目录

访问http://YourIp:5000/v2/_catalog 可以查看当前私有镜像服务中包含的镜像





### 图形化仓库

**1、在自己的目录中创建 docker-compose.yml 文件**

```shell
vim docker-compose.yml
```

**2、配置Docker信任地址**：Docker私服采用的是http协议，默认不被Docker信任，所以需要做一个配置

```shell
# 打开要修改的文件
vim /etc/docker/daemon.json
# 添加内容：registry-mirrors 是前面已经配置过的阿里云加速，放在这里是为了注意整个json怎么配置的，以及注意多个是用 逗号 隔开的
# 真正要加的内容是 "insecure-registries":["http://192.168.150.101:8080"]
{
  "registry-mirrors": ["https://838ztoaf.mirror.aliyuncs.com"],
  "insecure-registries":["http://192.168.150.101:8080"]
}
# 重加载
systemctl daemon-reload
# 重启docker
systemctl restart docker
```

**3、在docekr-compose.yml文件中编写如下内容**

```yaml
version: '3.0'

services:
  registry:
    image: registry
    volumes:
      - ./registry-data:/var/lib/registry
# ui界面搭建，用的是别人的
  ui:
    image: joxit/docker-registry-ui:static
    ports:
      - 8080:80
    environment:
      - REGISTRY_TITLE=悠忽有限公司私有仓库
      - REGISTRY_URL=http://registry:5000
    depends_on:
      - registry
```

**4、使用docker-compose启动容器**

```shell
docekr-compsoe up -d
```

**5、浏览器访问**

```txt
虚拟机IP:上面ui中配置的ports
```





### 推送和拉取镜像

推送镜像到私有镜像服务必须先tag，步骤如下：

1. 重新tag本地镜像，名称前缀为私有仓库的地址：192.168.xxx.yyy:8080/

 ```shell
# docker tag 仓库名(REPOSITORY):标签(TAG) YourIp:ui中配置的port/新仓库名:标签
docker tag nginx:latest 192.168.xxx.yyy:8080/nginx:1.0
 ```

2. 推送镜像

```shell
docker push 192.168.xxx.yyy:8080/nginx:1.0 
```

3. 拉取镜像

```shell
docker pull 192.168.xxx.yyy:8080/nginx:1.0 
```





# RabbitMQ 消息队列

官网：https://www.rabbitmq.com/

这里只说明一部分，当然针对开发也够了。全系列的RabbitMQ理论与实操知识去这个旮旯地方：https://www.cnblogs.com/xiegongzi/p/16242291.html



## 几种常见MQ的对比

|            | **RabbitMQ**            | **ActiveMQ**                   | **RocketMQ** | **Kafka**  |
| ---------- | ----------------------- | ------------------------------ | ------------ | ---------- |
| 公司/社区  | Rabbit                  | Apache                         | 阿里         | Apache     |
| 开发语言   | Erlang                  | Java                           | Java         | Scala&Java |
| 协议支持   | AMQP，XMPP，SMTP，STOMP | OpenWire,STOMP，REST,XMPP,AMQP | 自定义协议   | 自定义协议 |
| 可用性     | 高                      | 一般                           | 高           | 高         |
| 单机吞吐量 | 一般                    | 差                             | 高           | 非常高     |
| 消息延迟   | 微秒级                  | 毫秒级                         | 毫秒级       | 毫秒以内   |
| 消息可靠性 | 高                      | 一般                           | 高           | 一般       |

追求可用性：Kafka、 RocketMQ 、RabbitMQ

追求可靠性：RabbitMQ、RocketMQ

追求吞吐能力：RocketMQ、Kafka

追求消息低延迟：RabbitMQ、Kafka



## 5种常用消息模型

当然只需要记住Topic Exchange类型就可以转成其他任何一种模型，无非就是少这里少那里、适用不同场景的区别

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230616142728049-266151266.png" alt="image-20210717163332646" />





## Spring AMQP

Spring AMQP是基于RabbitMQ封装的一套模板，并且还利用SpringBoot对其实现了自动装配，使用起来非常方便

Spring AMQP官网：https://spring.io/projects/spring-amqp

<img src="https://img2023.cnblogs.com/blog/2421736/202312/2421736-20231221133316029-1576197112.png" alt="image-20231221133314091" />



Spring AMQP提供了三个功能：

- 自动声明队列、交换机及其绑定关系
- 基于注解的监听器模式，异步接收消息
- 封装了RabbitTemplate工具，用于发送消息 



依赖：

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```





## Hello word 基本消息队列模型

官网中的结构图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230616142005369-1795079636.png" alt="image-20230616142003688" />



即：1个publisher生产者、1个默认交换机、1个队列、1个consumer消费者

此种模型：做最简单的事情，一个生产者对应一个消费者，RabbitMQ相当于一个消息代理，负责将A的消息转发给B

**应用场景：**将发送的电子邮件放到消息队列，然后邮件服务在队列中获取邮件并发送给收件人





### 生产者

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConnectionFactory;
import org.junit.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * <p>@description  : 该类功能  hello word 基本消息队列模型 生产者测试
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o1HelloWordTest {
    private String host = "自己部署rabbitmq的虚拟机ip";
    private int port = 5672;
    private String username = "zixieqing";
    private String password = "072413";
    private String queueName = "hello-word";

    @Test
    public void helloWordTest() throws IOException, TimeoutException {
        // 1、设置链接信息
        ConnectionFactory conFactory = new ConnectionFactory();
        conFactory.setHost(host);
        conFactory.setPort(port);
        conFactory.setUsername(username);
        conFactory.setPassword(password);
		// 当然：这里还可以设置vhost虚拟机
		// factory.setVirtualHost();

        // 2、获取管道
        Channel channel = conFactory.newConnection().createChannel();
        
        /*
         * 3、队列声明
         * queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments);
		 * 	参数1、队列名字
		 * 	参数2、是否持久化，默认是在内存中
		 * 	参数3、是否共享，即：是否让多个消费者共享这个队列中的信息
		 * 	参数4、是否自动删除，即：最后一个消费者获取信息之后，这个队列是否自动删除
		 * 	参数5、其他配置项，这涉及到后面的知识，目前选择null
         * */
        channel.queueDeclare(queueName, false, false, false, null);

        // 4、消息推送
        String msg = "this is hello word";
        /*
        * basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body)
        * 参数1   交换机名
        * 参数2   路由键，是hello word 基础消息队列模型，所以此处使用队列名即可
        * 参数3   消息其他配置项
        * 参数4   要发送的消息内容
        * */
        channel.basicPublish("", queueName, null, msg.getBytes());

        // 5、释放资源
        channel.close();
        conFactory.clone();
    }
}
```



使用Spring AMQP就是如下的方式：

1. 配置application.yml

```yaml
spring:
  rabbitmq:
    host: 自己的ip
    port: 5672
#    集群的链接方式
#    addresses: ip:5672,ip:5673,ip:5674...................
    username: "zixieqing"
    password: "072413"
#    要是mq设置得有独立的虚拟机空间，则在此处设置虚拟机
#    virtual-host: /
```

2. 发送消息的代码：

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * <p>@description  : 该类功能  SpringAMQP测试
 * </p>
 * <p>@author       : ZiXieqing</p>
 */


@RunWith(SpringRunner.class)
@SpringBootTest
public class WorkModeTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 使用Spring AMQP实现 hello word 简单队列模式
     */
    @Test
    public void springAMQP2HelloWordTest() {
        // 1、引入spring-boot-starter-springamqp依赖

        // 2、编写application.uml文件

        // 3、发送消息
        String queueName = "hello-word";
        String message = "hello，this is springAMQP";
        rabbitTemplate.convertAndSend(queueName, message);
    }
}
```







### 消费者

```java
import com.rabbitmq.client.*;
import org.junit.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * <p>@description  : 该类功能  hello word 简单工作队列模型 消费者测试
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class HelloWordTest {
    private String host = "自己部署rabbitmq的虚拟机ip";
    private int port = 5672;
    private String username = "zixieqing";
    private String password = "072413";
    private String queueName = "hello-word";

    @Test
    public void consumerTest() throws IOException, TimeoutException {
        // 1、设置链接信息
        ConnectionFactory conFactory = new ConnectionFactory();
        conFactory.setHost(host);
        conFactory.setPort(port);
        conFactory.setUsername(username);
        conFactory.setPassword(password);

        // 2、获取管道
        Channel channel = conFactory.newConnection().createChannel();

        /*
        * 3、队列声明
        * queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments);
        * 参数1   队列名
        * 参数2   此队列是否持久化
        * 参数3   此队列是否共享，即：是否让多个消费者共享这个队列中的信息
        * 参数4   此队列是否自动删除，即：最后一个消费者获取信息之后，这个队列是否自动删除
        * 参数5   其他配置项
        *
        * */
        channel.queueDeclare(queueName, false, false, false, null);

        /*
        * 4、订阅消息
        * basicConsume(String queue, boolean autoAck, Consumer callback)
        * 参数1   队列名
        * 参数2   是否自动应答
        * 参数3   回调函数
        * */
        channel.basicConsume(queueName, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("consumerTag = " + consumerTag);
                /*
                * 可以获取到交换机、routingkey、deliveryTag
                * */
                System.out.println("envelope = " + envelope);
                System.out.println("properties = " + properties);
                System.out.println("处理了消息：" + new String(body));
            }
        });

        // 这是另外一种接收消息的方式
        /*DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("接收到了消息：" + new String(message.getBody(), StandardCharsets.UTF_8));
        };

        CancelCallback cancelCallback = consumerTag -> System.out.println("消费者取消了消费信息行为");

        channel.basicConsume(queueName, true, deliverCallback, cancelCallback);*/
    }
}
```



使用Spring AMQP就是如下的方式：

1. 配置application.yml

```yaml
spring:
  rabbitmq:
    host: 自己的ip
    port: 5672
    username: "zixieqing"
    password: "072413"
    # 要是mq设置的有独立的虚拟机空间，则在此处设置虚拟机
#    virtual-host: /
```

2. 接收消息的代码：

```java
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.time.LocalTime;

/**
 * <p>@description  : 该类功能  rabbitmq监听
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class RabbitmqListener {
    // 1、导入spring-boot-starter-springamqp依赖

    // 2、配置application.yml

    // 3、编写接受消息逻辑

    /**
     * <p>@description  : 该方法功能 监听 hello-word 队列
     * </p>
     * <p>@methodName   : listenQueue2HelloWord</p>
     * <p>@author: ZiXieqing</p>
     *
     * @param msg 接收到的消息
     */
    @RabbitListener(queues = "hello-word")
    public void listenQueue2HelloWord(String msg) {
        System.out.println("收到的消息 msg = " + msg);
    }
}
```





## Work Queue 工作消息队列模型

官网中的结构图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230616145123911-1191538262.png" alt="image-20230616145122494" />



即：1个publisher生产者、1个默认交换机、1个queue队列、多个consumer消费者

在多个消费者之间分配任务（竞争的消费者模式），一个生产者对应多个消费者，一般适用于执行资源密集型任务，单个消费者处理不过来，需要多个消费者进行处理

**应用场景：** 一个订单的处理需要10s，有多个订单可以同时放到消息队列，然后让多个消费者同时处理



### 生产者

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * <p>@description  : 该类功能  SpringAMQP测试
 * </p>
 * <p>@author       : ZiXieqing</p>
 */


@RunWith(SpringRunner.class)
@SpringBootTest
public class WorkModeTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 使用SpringAMQP实现 work queue 工作队列模式
     */
    @Test
    public void springAMQP2WorkQueueTest() {
        // 1、引入spring-boot-starter-springamqp依赖

        // 2、编写application.uml文件

        // 3、发送消息
        String queueName = "hello-word";
        String message = "hello，this is springAMQP + ";
        for (int i = 1; i <= 50; i++) {
            rabbitTemplate.convertAndSend(queueName, message + i);
        }
    }
}
```



### 消费者

application.yml配置：

```yaml
spring:
  rabbitmq:
    host: 自己的ip
    port: 5672
    username: "zixieqing"
    password: "072413"
    # 要是mq设置的有独立的虚拟机空间，则在此处设置虚拟机
#    virtual-host: /
    listener:
      simple:
        # 不公平分发，预取值 消费者每次从队列获取的消息数量 默认一次250个  通过查看后台管理器中queue的unacked数量
        prefetch: 1
```

接收消息

```java
package com.zixieqing.consumer.listener;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.time.LocalTime;

/**
 * <p>@description  : 该类功能  rabbitmq监听
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class RabbitmqListener {
    // 1、导入spring-boot-starter-springamqp依赖

    // 2、配置application.yml

    // 3、编写接受消息逻辑

    /**
     * <p>@description  : 该方法功能 监听 hello-word 队列
     * </p>
     * <p>@author: ZiXieqing</p>
     *
     * @param msg 接收到的消息
     */
    @RabbitListener(queues = "hello-word")
    public void listenQueue2WorkQueue1(String msg) throws InterruptedException {
        System.out.println("消费者1收到的消息 msg = " + msg + " + " + LocalTime.now());
        // 模拟性能，假设此消费者性能好
        Thread.sleep(20);
    }

    /**
     * <p>@description  : 该方法功能 监听 hello-word 队列
     * </p>
     * <p>@author: ZiXieqing</p>
     *
     * @param msg 接收到的消息
     */
    @RabbitListener(queues = "hello-word")
    public void listenQueue2WorkQueue2(String msg) throws InterruptedException {
        System.err.println("消费者2.............收到的消息 msg = " + msg + " + " + LocalTime.now());
        // 模拟性能，假设此消费者性差点
        Thread.sleep(200);
    }
}
```





## 交换机

>  **交换机的作用就是为了接收生产者发送的消息 并 将消息发送到队列中去**
>
>  注意：前面玩的那些模式，虽然没有写交换机，但并不是说RabbitMQ就没用交换机
>
>  ps：使用的是""空串，也就是使用了RabbitMQ的默认交换机，生产者发送的消息只能发到交换机中，从而由交换机来把消息发给队列





**交换机的分类**

1. 直接(direct) / routing 模式
2. 主题(topic)
3. 标题 (heanders)- 这个已经很少用了
4. 扇出(fancut) / 广播





### Fanout Exchange 广播模型 / 发布订阅模式

官网结构图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230616151103939-296865311.png" alt="image-20230616151102284" />



即：1个生产者、1个交换机、多个队列、多个消费者

广播消息到所有队列，没有任何处理，速度最快。**类似群发，一人发，很多人收到消息**

一次向许多消费者发送消息，一个生产者发送的消息会被多个消费者获取，也就是将消息广播到所有的消费者中

**应用场景：** 更新商品库存后需要通知多个缓存和多个数据库，这里的结构应该是：

1. 一个fanout类型交换机扇出两个消息队列，分别为缓存消息队列、数据库消息队列
2. 一个缓存消息队列对应着多个缓存消费者
3. 一个数据库消息队列对应着多个数据库消费者





#### 生产者

```java
package com.zixieqing.publisher;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * <p> fanout exchange 扇形/广播模型测试
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@RunWith(SpringRunner.class)
@SpringBootTest
public class o3FanoutExchangeTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void fanoutExchange4SendMsgTest() {
        String exchangeName = "fanout.exchange";
        String message = "this is fanout exchange";
        rabbitTemplate.convertAndSend(exchangeName,"",message);
    }
}
```





#### 消费者

创建交换机和队列 并 进行绑定

```java
package com.zixieqing.consumer.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.FanoutExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * <p> rabbitMQ配置
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Configuration
public class RabbitmqConfig {
    /**
     * 定义交换机类型 fanout.exchange
     */
    @Bean
    public FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanout.exchange");
    }

    /**
     * 定义队列 fanout.queue1
     */
    @Bean
    public Queue fanoutExchange4Queue1() {
        return new Queue("fanout.queue1");
    }

    /**
     * 将 fanout.exchange 和 fanout.queue1 两个进行绑定
     */
    @Bean
    public Binding fanoutExchangeBindQueue1(Queue fanoutExchange4Queue1, FanoutExchange fanoutExchange) {
        return BindingBuilder
            .bind(fanoutExchange4Queue1)
            .to(fanoutExchange);
    }

    /**
     * 定义队列 fanout.queue2
     */
    @Bean
    public Queue fanoutExchange4Queue2() {
        return new Queue("fanout.queue2");
    }

    /**
     * 将 fanout.exchange 和 fanout.queue2 两个进行绑定
     */
    @Bean
    public Binding fanoutExchangeBindQueue2(Queue fanoutExchange4Queue2, FanoutExchange fanoutExchange) {
        return BindingBuilder
            .bind(fanoutExchange4Queue2)
            .to(fanoutExchange);
    }
}
```

监听队列中的消息：

```java
package com.zixieqing.consumer.listener;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.time.LocalTime;

/**
 * <p>@description  : 该类功能  rabbitmq监听
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class RabbitmqListener {
    // 1、导入spring-boot-starter-springamqp依赖

    // 2、配置application.yml

    // 3、编写接受消息逻辑

    /**
     * fanoutExchange模型 监听fanout.queue1 队列的消息
     * @param msg 收到的消息
     */
    @RabbitListener(queues = "fanout.queue1")
    public void listenQueue14FanoutExchange(String msg) {
        System.out.println("消费者1收到 fanout.queue1 的消息 msg = " + msg );
    }

    /**
     * fanoutExchange模型 监听fanout.queue1 队列的消息
     * @param msg 收到的消息
     */
    @RabbitListener(queues = "fanout.queue2")
    public void listenQueue24FanoutExchange(String msg) {
        System.err.println("消费者2收到 fanout.queue2 的消息 msg = " + msg );
    }
}
```







### Direct Exchange 路由消息队列模型

官网中的结构图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230616190732193-808854073.png" alt="image-20230616190730692" />



即：1个消息发送者、1个交换机、routing key路由键、多个队列、多个消息消费者

这个玩意儿吧，才是真正的发布订阅模式，fanout类型交换机的变样板，即：多了一个routing key的配置而已，**也就是说：生产者和消费者传输消息就通过routing key进行关联起来**，**因此：现在就变成了生产者想把消息发给谁就发给谁**

有选择地（Routing key）接收消息，发送消息到交换机并指定路由key ，消费者将队列绑定到交换机时需要指定路由key，仅消费指定路由key的消息

**应用场景：** 如在商品库存中增加了1台iphone12，iphone12促销活动消费者指定routing key为iphone12，只有此促销活动会接收到消息，其它促销活动不关心也不会消费此routing key的消息





#### 生产者

```java
package com.zixieqing.publisher;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * <p> DirectEXchange 路由模式测试
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@RunWith(SpringRunner.class)
@SpringBootTest
public class o4DirectExchangeTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void sendMsg4DirectExchangeTest() {
        String exchangeNmae = "direct.exchange";
        String message = "this is direct exchange";
        // 把消息发给 routingkey 为 zixieqing 的队列中
        rabbitTemplate.convertAndSend(exchangeNmae, "zixieqing", message);
    }
}
```



#### 消费者

```java
package com.zixieqing.consumer.listener;

import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.time.LocalTime;

/**
 * <p>@description  : 该类功能  rabbitmq监听
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class RabbitmqListener {
    // 1、导入spring-boot-starter-springamqp依赖

    // 2、配置application.yml

    // 3、编写接受消息逻辑

    /**
     * 使用纯注解的方式声明队列、交换机及二者绑定、以及监听此队列的消息
     *
     * @param msg 监听到的消息
     */
    @RabbitListener(bindings = @QueueBinding(
            // 队列声明
            value = @Queue(name = "direct.queue1"),
            // 交换机声明
            exchange = @Exchange(name = "direct.exchange", type = ExchangeTypes.DIRECT),
            // 队列和交换机的绑定键值，是一个数组
            key = {"zixieqing"}
    ))
    public void listenQueue14DirectExchange(String msg) {
        System.err.println("消费者1收到 direct.queue1 的消息 msg = " + msg);
    }

    /**
     * 使用纯注解的方式声明队列、交换机及二者绑定、以及监听此队列的消息
     *
     * @param msg 监听到的消息
     */
    @RabbitListener(bindings = @QueueBinding(
            // 队列声明
            value = @Queue(name = "direct.queue2"),
            // 交换机声明
            exchange = @Exchange(name = "direct.exchange", type = ExchangeTypes.DIRECT),
            // 队列和交换机的绑定键值，是一个数组
            key = {"zimingxuan"}
    ))
    public void listenQueue24DirectExchange(String msg) {
        System.err.println("消费者2收到 direct.queue2 的消息 msg = " + msg);
    }
}
```

从此处代码可以得知：将每个队列与交换机的routing key改为一样的值，则变成Fanout Exchange了



Fanout Exchange与Direct Exchange的区别：

1. Fanout交换机将消息路由给**每一个**与之绑定的队列
2. Direct交换机**根据Routing Key判断**路由给哪个队列





### Topic Exchange 主题模型

官网结构图：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230616221652109-1311293641.png" alt="image-20230616221649375" />



前面玩的fanout扇出类型的交换机是一个生产者发布，多个消费者共享消息，和qq群类似；而direct 路由模式是消费者只能消费和消费者相同routing key的消息

而上述这两种还有局限性，如：现在生产者的routing key为zi.xie.qing，而一个消费者只消费含xie的消息，一个消费者只消费含qing的消息，另一个消费者只消费第一个为zi的零个或无数个单词的消息，甚至还有一个消费者只消费最后一个单词为qing，前面有三个单词的routing key的消息呢？

这样一看，发布订阅模式和路由模式都不能友好地解决，更别说前面玩的简单模式、工作队列模式了，因此：就来了这个topic主题模式



**应用场景：** iphone促销活动可以接收主题为iphone的消息，如iphone12、iphone13等



**topic中routing key的要求**：只要交换机类型是topic类型的，那么其routing key就不能乱写

1. routing key只能是一个“单词列表”，多个单词之间采用 点 隔开，如：com.zixieqing.rabbit
2. 单词列表的长度不能超过255个字节



**在routing key的规则列表中有两个替换符可以用**

1. `*` 代表一个单词
2. `#` 代表零或无数个单词







#### 生产者

```java
package com.zixieqing.publisher;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

/**
 * <p> Topic Exchange 话题模式测试
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@RunWith(SpringRunner.class)
@SpringBootTest
public class o5TopicExchangeTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void sendMSg2TopicExchangeTest() {
        String exchangeNmae = "topic.exchange";
        String msg = "贫道又升迁了，离目标越来越近了";
        // routing key变为 话题模式 com.zixieqing.blog
        rabbitTemplate.convertAndSend(exchangeNmae, "com.zixieqing.blog", msg);
    }
}
```





#### 消费者

```java
package com.zixieqing.consumer.listener;

import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.time.LocalTime;

/**
 * <p>@description  : 该类功能  rabbitmq监听
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class RabbitmqListener {
    // 1、导入spring-boot-starter-springamqp依赖

    // 2、配置application.yml

    // 3、编写接受消息逻辑

    /**
     * 使用纯注解的方式声明队列、交换机及二者绑定、以及监听此队列的消息
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "topic.queue1"),
            exchange = @Exchange(name = "topic.exchange", type = ExchangeTypes.TOPIC),
            // 只接收routing key 前面是一个词 且 含有 zixieiqng 发布的消息
            key = {"*.zixieqing.#"}
    ))
    public void listenQueue14TopicExchange(String msg) {
        System.out.println("消费者1收到 topic.queue1 的消息 msg = " + msg);
    }

    /**
     * 使用纯注解的方式声明队列、交换机及二者绑定、以及监听此队列的消息
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "topic.queue2"),
            exchange = @Exchange(name = "topic.exchange", type = ExchangeTypes.TOPIC),
            // 只接收routing key含有 blog 发布的消息
            key = {"#.blog"}
    ))
    public void listenQueue24TopicExchange(String msg) {
        System.err.println("消费者1收到 topic.queue1 的消息 msg = " + msg);
    }
}
```





## 消息转换器

查看Spring中默认的MessageConverter消息转换器

生产者：

```java
package com.zixieqing.publisher;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.HashMap;
import java.util.Map;

/**
 * mq消息转换器测试
 *
 * <p>@author       : ZiXieqing</p>
 */


@RunWith(SpringRunner.class)
@SpringBootTest
public class o7MessageConverterTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void mqMSgConverterTest() {
        // 准备消息
        Map<String,Object> msgMap = new HashMap<>();
        msgMap.put("name", "紫邪情");
        msgMap.put("age", 18);
        msgMap.put("profession", "java");
        
        // 发送消息
        rabbitTemplate.convertAndSend("msg.converter.queue",msgMap);
    }
}
```



```java
package com.zixieqing.publisher.config;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 注册bean
 *
 * <p>@author       : ZiXieqing</p>
 */

@Configuration
public class BeanConfig {
    @Bean
    public Queue msgConverterQueue() {
        return new Queue("msg.converter.queue");
    }
}
```



查看mq后台管理界面：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230617150354639-226036161.png" alt="image-20230617150353048" />



可知：spring中使用的消息转换器是 JDK序列化方式，即：ObjectOutputStream





### 配置Jackson序列化

生产者：

```java
package com.zixieqing.publisher.config;

import org.springframework.amqp.core.Queue;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 注册bean
 *
 * <p>@author       : ZiXieqing</p>
 */

@Configuration
public class BeanConfig {
    /**
     * 将消息转换器改为jackson序列化方式
     */
    @Bean
    public MessageConverter jacksonMsgConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```



消息发送：

```java
package com.zixieqing.publisher;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.HashMap;
import java.util.Map;

/**
 * mq消息转换器测试
 *
 * <p>@author       : ZiXieqing</p>
 */


@RunWith(SpringRunner.class)
@SpringBootTest
public class o7MessageConverterTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void mqMSgConverterTest() {
        // 准备消息
        Map<String,Object> msgMap = new HashMap<>();
        msgMap.put("name", "紫邪情");
        msgMap.put("age", 18);
        msgMap.put("profession", "java");

        // 发送消息		注意：这里的msg消息类型是map
        rabbitTemplate.convertAndSend("msg.converter.queue",msgMap);
    }
}
```



消费者：

```java
package com.zixieqing.consumer.config;

import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * bean注册
 *
 * <p>@author       : ZiXieqing</p>
 */


@Configuration
public class BeanConfig {
    /**
     * 将消息转换器改为jackson序列化方式
     */
    @Bean
    public MessageConverter jacksonMsgConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```



```java
package com.zixieqing.consumer.listener;

import org.springframework.amqp.core.ExchangeTypes;
import org.springframework.amqp.rabbit.annotation.Exchange;
import org.springframework.amqp.rabbit.annotation.Queue;
import org.springframework.amqp.rabbit.annotation.QueueBinding;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.time.LocalTime;
import java.util.Map;

/**
 * <p>@description  : 该类功能  rabbitmq监听
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class RabbitmqListener {
    // 1、导入spring-boot-starter-springamqp依赖

    // 2、配置application.yml

    // 3、编写接受消息逻辑

    /**
     * 使用jackson的方式对消息进行接收
     * @param msg   接收到的消息      注：这里的类型需要和生产者发送消息时的类型保持一致
     */
    @RabbitListener(queues = "msg.converter.queue")
    public void listenQueue4Jackson(Map<String,Object> msg) {
        System.out.println("消费者收到消息 msg = " + msg);
    }
}
```









## publisher-confirms 发布确认模型

**如何确保RabbitMQ消息的可靠性？**

1. 生产者方：
   1. 开启生产者确认机制，确保生产者的消息能到达队列
   2. 开启持久化功能，确保消息未消费前在队列中不会丢失
2. 消费者方：
   1. 开启消费者确认机制为auto，由spring确认消息处理成功后完成ack
   2. 开启消费者失败重试机制，并设置MessageRecoverer，多次重试失败后将消息投递到异常交换机，交由人工处理





正常的流程应该是下面的样子

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230617005335080-1584963969.png" alt="image" />

但是：如果交换机出问题了呢，总之就是交换机没有接收到生产者发布的消息(如：发消息时，交换机名字搞错了)，那消息就直接丢了吗？

同理：要是队列出问题了呢，总之也就是交换机没有成功地把消息推到队列中(如：routing key搞错了)，咋办？

那就需要第一个条件 **发送消息确认：用来确认消息从 producer发送到 exchange， exchange 到 queue过程中，消息是否成功投递**

**应用场景：** 对于消息可靠性要求较高，比如钱包扣款

**流程**

1. 若消息未到达exchange，则confirm回调，ack=false
2. 若消息到达exchange，则confirm回调，ack=true
3. exchange到queue成功，则不回调return
4. exchange到queue失败，则回调return(需设置mandatory=true，否则不会回调，这样消息就丢了)



**生产者方需要开启两个配置：**

```yaml
spring:
  rabbitmq:
    # 发布确认类型  生产者开启 confirm 确认机制	等价于旧版本的publisher-confirms=true
    # 有3种属性配置   correlated    none    simple
    #     none  禁用发布确认模式，是默认值
    #     correlated  异步回调  发布消息成功到exchange后会触发 rabbitTemplate.setConfirmCallback 回调方法
    #     simple 同步等待confirm结果，直到超时
    publisher-confirm-type: correlated
    # 生产者开启 return 确认机制   如果消息未能投递到目标queue中，触发returnCallback
    publisher-returns: true
```



### ConfirmCallback 回调

在前面 `publisher-confirm-type: correlated` 配置开启的前提下，发布消息成功到exchange后会进行  ConfirmCallback#confirm 异步回调，示例如下：

```java
@Component
public class ConfirmCallbackService implements RabbitTemplate.ConfirmCallback {
    /** 
     * correlationData：对象内部有id （消息的唯一性）和 Message	
     * 				    若ack为false，则Message不为null，可将Message数据 重新投递；
     * 				    若ack是true，则correlationData为nul
     *
     * ack：消息投递到exchange 的状态，true表示成功
     *
     * cause：表示投递失败的原因
     * 			若ack为false，则cause不为null
     * 			若ack是true，则cause为null
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
		if(ack){
			System.out.println("消息送达到Exchange");
		}else{
			System.out.println("消息未送达到Exchange");
		}
    }
}
```

在生产者发送消息时，可以给每一条信息添加一个dataId，放在CorrelationData，这样在RabbitConfirmCallback返回失败时可以知道哪条消息失败

```java
public void send(String dataId, String exchangeName, String rountingKey, String message){
  CorrelationData correlationData = new CorrelationData();
  // 可以给每条消息设置唯一id  在RabbitConfirmCallback返回失败时可以知道哪个消息失败
  correlationData.setId(dataId);

  rabbitTemplate.convertAndSend(exchangeName, rountingKey, message, correlationData);
}

public String receive(String queueName){
  return String.valueOf(rabbitTemplate.receiveAndConvert(queueName));
}
```

2.1版本之后，CorrelationData对象具有getFuture，可用于获取结果，而不用在rabbitTemplate上使用ConfirmCallback

```java
CorrelationData correlationData = new CorrelationData();
// 可以给每条消息设置唯一id  在RabbitConfirmCallback返回失败时可以知道哪个消息失败
correlationData.setId(dataId);

// 在新版中correlationData具有getFuture，可获取结果，而不用在rabbitTemplate上使用ConfirmCallback
correlationData.getFuture().addCallback( // 对照Ajax
    // 成功：收到MQ发的回执
    result -> {
        // 成功发送到exchange
        if (result.isAck()) {
            // 消息发送成功 ack回执
            System.out.println(correlationData.getId() + " 消息发送成功");
        } else {	// 未成功发送到exchange
            // 消息发送失败 nack回执
            System.out.println(correlationData.getId() + " 消息发送失败，原因：" + result.getReason());
        }
    }, ex -> { // ex 即 exception   不知道什么原因，抛了异常，没收到MQ的回执
        System.out.println(correlationData.getId() + " 消息发送失败，原因：" + ex.getMessage());
    }
);

rabbitTemplate.convertAndSend(exchangeName, rountingKey, message, correlationData);
```





### ReturnCallback 回调

**如果消息未能投递到目标queue中，触发returnCallback#returnedMessage**

==注意点：每个RabbitTemplate只能配置一个ReturnCallback==。 即Spring全局只有这一个Return回调，不能说想写多少个就写多少个

若向 queue 投递消息未成功，可记录下当前消息的详细投递数据，方便后续做重发或者补偿等操作



但是这玩意儿又要涉及到另外一个配置：消息路由失败策略

```yaml
spring:
  rabbitmq:
    template:
      # 生产者方消息路由失败策略
      #   true：调用ReturnCallback
      #   false：直接丢弃消息
      mandatory: true
```

ReturnCallBack回调的玩法：

```java
@Component
public class ReturnCallbackService implements RabbitTemplate.ReturnCallback {
    /**
     * 保证 spring.rabbitmq.template.mandatory = true 和 publisher-returns: true 的前提下
     * 		如果消息未能投递到目标queue中，触发本方法
     *
     * 参数1、消息 new String(message.getBody())
     * 参数2、消息退回的状态码
     * 参数3、消息退回的原因
     * 参数4、交换机名字
     * 参数5、路由键
    */
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        System.out.println("消息没有送达到Queue");
    }
}
```



### ConfirmCallback 和 ReturnCallback 整合的写法

消息发送者编写代码：

```java
package com.zixieqing.publisher.config;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;

/**
 * <p> mq的confirmCallback和ReturnCallback
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Configuration
public class PublisherConfirmAndReturnConfig implements RabbitTemplate.ConfirmCallback, 
        RabbitTemplate.ReturnCallback {
            
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 初始化方法
     * 目的：因为ConfirmCallback 和 ReturnCallback这两个接口是RabbitTemplate的内部类
     * 因此：想要让当前编写的PublisherConfirmAndReturnConfig能够访问到这两个接口
     * 那么：就需要把当前类PublisherConfirmAndReturnConfig的confirmCallback 和 returnCallback
     *      注入到RabbitTemplate中去 即：init的作用
     */
    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnCallback(this);
    }

    /**
     * 在前面 publisher-confirm-type: correlated 配置开启的前提下，发布消息成功到exchange后
     *       会进行 ConfirmCallback#confirm 异步回调
     * 参数1、发送消息的ID - correlationData.getID()  和 消息的相关信息
     * 参数2、是否成功发送消息给exchange  true成功；false失败
     * 参数3、失败原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if(ack){
            System.out.println("消息送达到Exchange");
        }else{
            System.out.println("消息未送达到Exchange");
        }
    }

    /**
     * 保证 spring.rabbitmq.template.mandatory = true 和 publisher-returns: true 的前提下
     * 		如果消息未能投递到目标queue中，触发returnCallback#returnedMessage
     * 
     * 参数1、消息 new String(message.getBody())
     * 参数2、消息退回的状态码
     * 参数3、消息退回的原因
     * 参数4、交换机名字
     * 参数5、路由键
     */
    @Override
    public void returnedMessage(Message message, int replyCode, 
                                String replyText, String exchange, String routingKey) {
        System.out.println("消息没有送达到Queue");
    }
}
```

生产者调用的方法是：

```java
// 可以给每条消息设置唯一id
CorrelationData correlationData = new CorrelationData();
correlationData.setId(dataId);

// 发送消息
rabbitTemplate.convertAndSend(String exchange, String routingKey, Object message, correlationData);
```







## 消息持久化

生产者确认可以确保消息投递到RabbitMQ的队列中，但是消息发送到RabbitMQ以后，如果突然宕机，也可能导致消息丢失

要想确保消息在RabbitMQ中安全保存，必须开启消息持久化机制：

1. **交换机持久化**：RabbitMQ中交换机默认是非持久化的，mq重启后就丢失。Spring AMQP中可以通过代码指定交换机持久化。**默认情况下，由Spring AMQP声明的交换机都是持久化的**

```java
@Bean
public DirectExchange simpleExchange(){
    // 三个参数：交换机名称、是否持久化、当没有queue与其绑定时是否自动删除
    return new DirectExchange(exchangeName, true, false);
}
```

2. **队列持久化**：RabbitMQ中队列默认是非持久化的，mq重启后就丢失。SpringAMQP中可以通过代码指定交换机持久化。**默认情况下，由Spring AMQP声明的队列都是持久化的**

```java
@Bean
public Queue simpleQueue(){
    // 使用QueueBuilder构建队列，durable就是持久化的
    return QueueBuilder.durable(queueName).build();
}
```

3. **消息持久化**：利用Spring AMQP发送消息时，可以设置消息的属性（MessageProperties），指定delivery-mode：非持久化 / 持久化。**默认情况下，Spring AMQP发出的任何消息都是持久化的**

```java
// 构建消息
Message msg = MessageBuilder.
    // 消息体
    withBody(message.getBytes(StandardCharsets.UTF_8))
    // 持久化
    .setDeliveryMode(MessageDeliveryMode.PERSISTENT)
    .build();
```





## 消费者消息确认

RabbitMQ是**阅后即焚**机制，RabbitMQ确认消息被消费者消费后会立刻删除

而RabbitMQ是通过消费者回执来确认消费者是否成功处理了消息：消费者获取消息后，应该向RabbitMQ发送ACK回执，表明自己已经处理消息

设想这样的场景：

1. RabbitMQ投递消息给消费者
2. 消费者获取消息后，返回ACK给RabbitMQ
3. RabbitMQ删除消息
4. 消费者宕机，消息尚未处理

这样，消息就丢失了。因此消费者返回ACK的时机非常重要



而Spring AMQP则允许配置三种确认模式：

1. **manual**：手动ack，需要在业务代码结束后，调用api发送ack，所以要自己根据业务情况，判断什么时候该ack
2. **auto**：自动ack，由spring监测Listener代码是否出现异常，没有异常则返回ack；抛出异常则返回nack。一般要用就用此种方式即可
3. **none**：关闭ack，MQ假定消费者获取消息后会成功处理，因此消息投递后立即被删除。不可靠，消息可能丢失



使用确认模式：在**消费者方**的YAML文件中配置如下内容：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        acknowledge-mode: auto # 自动应答模式
```





## 失败重试机制

经过前面的 发布确认模式+消息持久化+消费者消息确认 之后，还会有问题，如下面的代码：

```java
@RabbitListener(queues = "simple.queue")
public void listenSimpleQueue(String msg) {
    log.info("消费者接收到simple.queue的消息：【{}】", msg);
    // 模拟异常
    System.out.println(1 / 0);
    log.debug("消息处理完成！");
}
```

会死循环：当消费者出现异常后，消息会不断requeue（重入队）到队列，再重新发送给消费者，然后再次异常，再次requeue，无限循环，导致mq的消息处理飙升，带来不必要的压力

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230709002845387-819379886.png" alt="image-20230709002843115" />

要解决就得引入下一节的内容





### 本地重试机制

可以利用Spring的retry机制，在消费者出现异常时利用本地重试，而不是无限制的requeue到mq队列

在**消费者方**的YAML文件中添加如下内容即可：

```yaml
spring:
  rabbitmq:
    listener:
      simple:
        retry:
          enabled: true # 开启消费者失败重试
          interval-interval: 1000 # 初始的失败等待时长为1秒
          multiplier: 1 # 失败的等待时长倍数，下次等待时长 = multiplier * interval-interval
          max-attempts: 3 # 最大重试次数
          stateless: true # true无状态；false有状态。如果业务中包含事务，这里改为false
```

开启本地重试时，消息处理过程中抛出异常，不会requeue到队列，而是在消费者本地重试

**重试达到最大次数后，Spring会返回ack，消息会被丢弃**。这不可取，对于不重要的消息可以采用这种方式，但是有时的开发场景中有些消息很重要，达到重试上限后，不能丢弃，得使用另外的方式：**失败策略**





### 失败策略

达到最大重试次数后，消息会被丢弃，这是由Spring内部机制决定的

在开启重试模式后，重试次数耗尽，如果消息依然失败，则需要由MessageRecovery接口来处理，它包含三种不同的实现：

1. RejectAndDontRequeueRecoverer：重试耗尽后，直接reject，丢弃消息。默认就是这种方式
2. ImmediateRequeueMessageRecoverer：重试耗尽后，返回nack，消息重新入队
3. **RepublishMessageRecoverer**：重试耗尽后，将失败消息投递到指定的交换机



使用RepublisherMessageRecoverer失败策略：在**消费者方**定义失败之后要丢去的exchange+queue

```java
package com.zixieqing.mq.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.DirectExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.retry.MessageRecoverer;
import org.springframework.amqp.rabbit.retry.RepublishMessageRecoverer;
import org.springframework.context.annotation.Bean;

@Configuration
public class ErrorMessageConfig {
    @Bean
    public DirectExchange errorMessageExchange(){
        return new DirectExchange("error.direct.exchange");
    }
    @Bean
    public Queue errorQueue(){
        return new Queue("error.queue", true);
    }
    @Bean
    public Binding errorBinding(Queue errorQueue, DirectExchange errorMessageExchange){
        return BindingBuilder
            .bind(errorQueue)
            .to(errorMessageExchange)
            .with("error");
    }

    /**
     * 定义RepublishMessageRecoverer，关联队列和交换机
     */
    @Bean
    public MessageRecoverer republishMessageRecoverer(RabbitTemplate rabbitTemplate){
        return new RepublishMessageRecoverer(rabbitTemplate, "error.direct.exchange", "error");
    }
}
```







## 死信队列

> **死信队列：指的是“死了”的消息。** 换言之就是：生产者把消息发送到交换机中，再由交换机推到队列中，但由于某些原因，队列中的消息没有被正常消费，从而就让这些消息变成了死信，而专门用来放这种消息的队列就是死信队列，同理接收这种消息的交换机就是死信交换机
>
> 
>
> **让消息成为死信的三大因素：**
>
> 1. 消息过期 即：TTL(time to live)过期
> 2. 超过队列长度
> 3. 消息被消费者绝收了





### TTL消息过期

超时分为两种情况：若下面两个都设置了，那么先触发时间短的那个

- 消息本身设置了超时时间
- 消息所在队列设置了超时时间



实现下图逻辑：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230709230805203-633364493.png" alt="image-20230709230801915" />

1. 生产者：给消息设置超时时间

```java
package com.zixieqing.publisher;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageBuilder;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.nio.charset.StandardCharsets;

/**
 * 死信队列测试
 *
 * <p>@author       : ZiXieqing</p>
 */

@Slf4j
@SpringBootTest(classes = PublisherApp.class)
public class o8DelayedQueueTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 发消息给TTL正常交换机
     */
    @Test
    void TTLMessageTest() {
        Message message = MessageBuilder
                .withBody("hello,dead-letter-exchange".getBytes(StandardCharsets.UTF_8))
                // 给消息设置失效时间，单位ms
                .setExpiration("5000")
                .build();

        rabbitTemplate.convertAndSend("ttl.direct", "ttl", message);

        log.info("消息发送成功");
    }
}
```

2. 消费者：声明死信交换机+死信队列+二者的绑定+接收消息并处理

```java
/**
 * TTL正常队列，同时绑定死信交换机
 */
@Bean
public Queue ttlQueue() {
    return QueueBuilder
        .durable("ttl.queue")
        // 设置队列的超时时间
        .ttl(10000)
        // 绑定死信交换机
        .deadLetterExchange("dl.direct")
        // 死信交换机与死信队列的routing key
        .deadLetterRoutingKey("dl")
        .build();
}

/**
 * 将正常交换机和正常队列进行绑定
 */
@Bean
public Binding ttlBinding() {
    return BindingBuilder
        .bind(ttlQueue())
        .to(ttlExchange())
        .with("ttl");
}





/**
 * 监听死信队列：死信交换机+死信队列进行绑定
 */
@RabbitListener(bindings = @QueueBinding(
    value = @Queue(name = "dl.queue", durable = "true"),
    exchange = @Exchange(name = "dl.direct"),
    key = "dl"
))
public void listenDlQueue(String msg) {
    log.info("消费者收到了dl.queue的消息：{}", msg);
}
```







### 超过队列最大长度

分为两种情况：

1. 队列中只能放“多少条”消息
2. 队列中只能放“多少字节”的消息

```java
@Bean
public Queue queueLength() {
    return QueueBuilder
        .durable("length.queue")
        // 队列只能放多少条消息
        .maxLength(100)
        // 队列中只能放多少字节的消息
        .maxLengthBytes(10240)
        .build();

    // 或下面的方式声明

    Map<String, Object> params = new HashMap<>();
    // 队列最大长度，即队列中只能放这么多个消息
    params.put("x-max-length", 100);
    // 队列中最大的字节数
    params.put("x-max-length=bytes", 10240);
    return new Queue("length.queue", false, false, false, params);
}
```

另外一种被消费者拒收就是nack了，早已熟悉









## 惰性队列

> **解决的问题：** 消息堆积问题。当生产者发送消息的速度超过了消费者处理消息的速度，就会导致队列中的消息堆积，直到队列存储消息达到上限。之后发送的消息就会成为死信，可能会被丢弃，这就是消息堆积问题



**惰性队列：** RabbitMQ 3.6加入的，名为lazy queue

1. 接收到消息后直接存入磁盘而非内存
2. 消费者要消费消息时才会从磁盘中读取并加载到内存
3. 支持数百万条的消息存储

解决消息堆积有两种思路：

1. 增加更多消费者，提高消费速度。也就是之前说的work queue模式
2. 扩大队列容积，提高堆积上限(惰性队列要采用的方式)



1. Linux中声明

````shell
rabbitmqctl set_policy Lazy "^lazy-queue$" '{"queue-mode":"lazy"}' --apply-to queues  


rabbitmqctl						RabbitMQ的命令行工具
set_policy						添加一个策略
Lazy							策略名称，可以自定义
"^lazy-queue$" 					用正则表达式匹配队列的名字
'{"queue-mode":"lazy"}'			设置队列模式为lazy模式
--apply-to queues				策略的作用对象，是所有的队列
````

2. Java代码声明：消费者方定义即可

```java
/**
 * 惰性队列声明：Bean注解的方式
 */
@Bean
public Queue lazyQueue() {
    Map<String, Object> params = new HashMap();
    params.put("x-queue-mode", "lazy");
    return new Queue("lazy.queue", true, true, false, params);

    // 或使用下面更方便的方式

    return QueueBuilder
        .durable("lazy.queue")
        // 声明为惰性队列
        .lazy()
        .build();
}





/**
 * 惰性队列：RabbitListener注解的方式 这种就是new一个Map里面放参数的方式
 * @param msg 接收到的消息
 */
@RabbitListener(queuesToDeclare = @org.springframework.amqp.rabbit.annotation.Queue(
    name = "lazy.queue",
    durable = "true",
    arguments = @Argument(name = "x-queue-mode", value = "lazy")
))
public void lazyQueue(String msg) {
    System.out.println("消费者接收到了消息：" + msg);
}
```







## RabbitMQ如何保证消息的有序性？

RabbitMQ是队列存储，天然具备先进先出的特点，只要消息的发送是有序的，那么理论上接收也是有序的

不过当一个队列绑定了多个消费者时，可能出现消息轮询投递给消费者的情况，而消费者的处理顺序就无法保证了

因此，要保证消息的有序性，需要做到下面几点：

1. 保证消息发送的有序性
2. 保证一组有序的消息都发送到同一个队列
3. 保证一个队列只包含一个消费者









## 如何防止MQ消息被重复消费？

消息重复消费的原因多种多样，不可避免。所以只能从消费者端入手，只要能保证消息处理的幂等性就可以确保消息不被重复消费

而幂等性的保证又有很多方案：

1. 给每一条消息都添加一个唯一id，在本地记录消息表及消息状态，处理消息时基于数据库表的id唯一性做判断

2. 同样是记录消息表，利用消息状态字段实现基于乐观锁的判断，保证幂等

3. 基于业务本身的幂等性。比如根据id的删除、查询业务天生幂等；新增、修改等业务可以考虑基于数据库id唯一性、或者乐观锁机制确保幂等。本质与消息表方案类似





## 如何保证RabbitMQ的高可用？

要实现RabbitMQ的高可用无外乎下面两点：

1. 做好交换机、队列、消息的持久化
2. 搭建RabbitMQ的镜像集群，做好主从备份。当然也可以使用仲裁队列代替镜像集群













# ElasticSearch 分布式搜索引擎

此处只是浓缩内容，没基础的可能看不懂，全系列知识去下列链接：

1. 基础理论和DSL语法(可看可不看)：https://www.cnblogs.com/xiegongzi/p/15684307.html
2. Java操作：https://www.cnblogs.com/xiegongzi/p/15690534.html
4. 续篇：https://www.cnblogs.com/xiegongzi/p/15770665.html





## 关系型数据库与ES的对应关系

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230617212656749-968465782.png" alt="img" />



**注**：ES 7.x之后，type已经被淘汰了，其他的没变





## 基础理论

### 正向索引和倒排索引

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230617213116322-1555655317.png" alt="image" />

elasticsearch使用的就是倒排索引



倒排索引中又有3个小东西：

1. **词条**：**是指索引中的最小存储或查询单元**。这个其实很好理解，白话文来讲就是：字或者词组，英文就是一个单词，中文就是字或词组嘛，比如：你要查询的内容中具备含义的某一个字或词组，这就是词条呗，如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条
2. **词典**：就是词条的集合嘛。**字或者词组组成的内容呗**
3. **倒排表**：**就是指 关键字 / 关键词 在索引中的位置。** 有点类似于数组，你查询数组中某个元素的位置，但是区别很大啊，我只是为了好理解，所以才这么举例子的



### type 类型

**这玩意儿就相当于关系型数据库中的表，注意啊：关系型中表是在数据库下，那么ES中也相应的 类型是在索引之下建立的**

表是个什么玩意呢？行和列嘛，这行和列有多少？N多行和N多列嘛，所以：ES中的类型也一样，可以定义N种类型。
同时：每张表要存储的数据都不一样吧，所以表是用来干嘛的？分类 / 分区嘛，所以ES中的类型的作用也来了：就是为了分类嘛。
另外：关系型中可以定义N张表，那么在ES中，也可以定义N种类型

**因此：ES中的类型类似于关系型中的表，作用：为了分类 / 分区，同时：可以定义N种类型，但是：类型必须是在索引之下建立的（ 是索引的逻辑体现嘛 ）**

**但是：不同版本的ES，类型也发生了变化，上面的解读不是全通用的**

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230617215117345-685221570.png" alt="image" />



### field 字段

**这也就类似于关系型中的列。 对文档数据根据不同属性（列字段）进行的分类标识**

字段常见的简单类型：注意：id的类型在ES中id是字符串，这点需要注意

- 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）
- 数值：long、integer、short、byte、double、float、
- 布尔：boolean
- 日期：date
- 对象：object
- 地图类型：geo_point 和 geo_shape
  - geo_point：由纬度(latitude) 和经度(longitude)确定的一个点，如：“32.54325453, 120.453254”
  - geo_shape：由多个geo_point组成的复杂集合图形，如一条直线 “LINESTRING (-77.03653 38.897676, -77.009051 38.889939)”
- 自动补全类型：completion



**注意：**没有数组类型，但是可以实现出数组，因为每种类型可以有“多个值”，即可实现出类似于数组类型，例如下面的格式：

```json
{
    "age": 21,	// Integer类型
    "weight": 52.1,		// float类型
    "isMarried": false,		// boolean类型
    "info": "这就是一个屌丝女",		// 字符串类型 可能为test，也可能为keyword 需要看mapping定义时对文档的约束是什么
    "email": "zixq8@slafjkl.com",	// 字符串类型 可能为test，也可能为keyword 需要看mapping定义时对文档的约束是什么
    "score": [99.1, 99.5, 98.9],	// 类似数组	就是利用了一个类型可以有多个值
    "name": {		// object对象类型
        "firstName": "紫",
        "lastName": "邪情"
    }
}
```



**还有一个字段的拷贝：** 可以使用copy_to属性将当前字段拷贝到指定字段

**使用场景：** 多个字段放在一起搜索的时候

**注意：** 定义的要拷贝的那个字段在ES中看不到，但是确实是存在的，就像个虚拟的一样

```json
// 定义了一个字段
"all": {
    "type": "text",
    "analyzer": "ik_max_word"
}


"name": {
    "type": "text",
    "analyzer": "ik_max_word",
    "copy_to": "all"		// 将当前字段 name 拷贝到 all字段中去
}
```





### document 文档

**这玩意儿类似于关系型中的行。 一个文档是一个可被索引的基础信息单元，也就是一条数据嘛**

即：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息



**新增文档：**

```json
// 这是kibana中进行的操作，要是使用如postman风格的东西发请求，则在 /索引库名/_doc/文档id 前加上es主机地址即可
POST /索引库名/_doc/文档id		// 指定了文档id，若不指定则es自动创建
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    },
    // ...
}
```

**查看指定文档id的文档：**

```json
GET /{索引库名称}/_doc/{id}
```

**删除指定文档id的文档：**

```json
DELETE /{索引库名}/_doc/id值
```

**修改文档：**有两种方式

- **全量修改**：直接覆盖原来的文档。其本质是：
  - 根据指定的id删除文档
  - 新增一个相同id的文档
  - **注意**：如果根据id删除时，id不存在，第二步的新增也会执行，也就从修改变成了新增操作了

```json
// 语法格式
PUT /{索引库名}/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    // ... 略
}
```

- **增量/局部修改**：是只修改指定id匹配的文档中的部分字段

```json
// 语法格式
POST /{索引库名}/_update/文档id
{
    "doc": {
         "字段名": "新的值",
    }
}
```





#### 文档分析

试想：我们在浏览器中，输入一条信息，如：搜索“博客园紫邪情”，为什么连“博客园也搜索出来了？我要的不是这个结果啊
<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621160903705-933424963.png" alt="image-20230621160902915" />

这就是全文检索，就是ES干的事情（ 过滤数据、检索嘛 ），但是：它做了哪些操作呢？

在ES中有一个**文档分析的过程**，文档分析的过程也很简单：

1. 将文本拆成适合于倒排索引的独立的词条，然后把这些词条统一变为一个标准格式，从而使文本具有“可搜索性”。


而这个文档分析的过程在ES中是由一个叫做“分析器 analyzer”的东西来做的，这个分析器里面做了三个步骤

1. 字符过滤器：就是用来处理一些字符的嘛，像什么将 & 变为 and 啊、去掉HTML元素啊之类的。它是文本字符串在经过分词之前的一个步骤，文本字符串是按文本顺序经过每个字符串过滤器从而处理字符串
2. 分词器：见名知意，就是用来分词的，也就是将字符串拆分成词条（ 字 / 词组 ），这一步和Java中String.split()一样的，通过指定的要求，把内容进行拆分，如：空格、标点符号
3. Token过滤器：这个玩意儿的作用就是 词条经过每个Token过滤器，从而对数据再次进行筛选，如：字母大写变小写、去掉一些不重要的词条内容、添加一些词条（如：同义词）



#### 内置分析器

##### standard 标准分析器

这是根据Unicode定义的单词边界来划分文本，将字母转成小写，去掉大部分的标点符号，从而得到的各种语言的最常用文本选择，另外：这是ES的默认分析器





##### simple 简单分析器

按非字母的字符分词，例如：数字、标点符号、特殊字符等，会去掉非字母的词，大写字母统一转换成小写



##### whitespace 空格分析器

是简单按照空格进行分词，相当于按照空格split了一下，大写字母不会转换成小写



##### stop 去词分析器

会去掉无意义的词

此无意义是指语气助词等修饰性词，补语文：语气词是疑问语气、祈使语气、感叹语气、肯定语气和停顿语气。例如：the、a、an 、this等，大写字母统一转换成小写



##### keyword 不拆分分析器

就是将整个文本当作一个词







#### 文档搜索

##### 不可变的倒排索引

以前的全文检索是将整个文档集合弄成一个倒排索引，然后存入磁盘中，当要建立新的索引时，只要新的索引准备就绪之后，旧的索引就会被替换掉，这样最近的文档数据变化就可以被检索到

而索引一旦被存入到磁盘就是不可变的，而这样做有如下的好处：

1. 只要索引被读入到内存中了，由于其不变性，所以就会一直留在内存中（ 只要空间足够 ），从而当我们做“读操作”时，请求就会进入内存中去，而不会去磁盘中，这样就减小开销，提高效率了
2. 索引放到内存中之后，是可以进行压缩的，这样做之后，也就可以节约空间了
3. 放到内存中后，是不需要锁的，如果自己的索引是长期不用更新的，那么就不用怕多进程同时修改它的情况了

当然：这种不可变的倒排索引有好处，那就肯定有坏处了

- 不可变，不可修改嘛，这就是最大的坏处，当要重定一个索引能够被检索时，就需要重新把整个索引构建一下，这样的话，就会导致索引的数据量很大（ 数据量大小有限制了 ），同时要更新索引，那么这频率就会降低了
- 这就好比是什么呢？关系型中的表，一张大表检索数据、更新数据效率高不高？肯定不高，所以延伸出了：可变索引





##### 可变的倒排索引

**又想保留不可变性，又想能够实现倒排索引的更新，咋办？**

- 就搞出了`补充索引`，**所谓的补充索引：有点类似于日志这个玩意儿，就是重建一个索引，然后用来记录最近指定一段时间内的索引中文档数据的更新。**这样更新的索引数据就记录在补充索引中了，然后检索数据时，直接找补充索引即可，这样检索时不再重写整个倒排索引了，这有点类似于关系型中的拆表，大表拆小表嘛，**但是啊：每一份补充索引都是一份单独的索引啊，这又和分片很像，可是：查询时是对这些补充索引进行轮询，然后再对结果进行合并，从而得到最终的结果，这和读流程中说明的协调节点挂上钩了**

**这里还需要了解一个配套的`按段搜索`，玩过 Lucene 的可能听过。按段，每段也就可以理解为：补充索引，它的流程其实也很简单：**

1. 新文档被收集到内存索引缓存
2. 不时地提交缓存
   1. 一个新的段，一个追加的倒排索引，被写入磁盘
   2. 一个新的包含新段名字的提交点被写入磁盘
3. 磁盘进行同步，所有在文件系统缓存中等待的写入都刷新到磁盘，以确保它们被写入物理文件
4. 内存缓存被清空，等待接收新的文档
5. 新的段被开启，让它包含的文档可见，以被搜索

一样的，段在查询的时候，也是轮询的啊，然后把查询结果合并从而得到的最终结果

另外就是涉及到删除的事情，**段本身也是不可变的， 既不能把文档从旧的段中移除，也不能修改旧的段来进行文档的更新，而删除是因为：是段在每个提交点时有一个.del文件，这个文件就是一个删除的标志文件，要删除哪些数据，就对该数据做了一个标记，从而下一次查询的时候就过滤掉被标记的这些段，从而就无法查到了，这叫逻辑删除（当然：这就会导致倒排索引越积越多，再查询时。轮询来查数据也会影响效率），所以也有物理删除，它是把段进行合并，这样就舍弃掉被删除标记的段了，从而最后刷新到磁盘中去的就是最新的数据（就是去掉删除之后的 ，别忘了前面整的段的流程啊，不是白写的）**







### mapping 映射

**指的就是：结构信息 / 限制条件**

还是对照关系型来看，在关系型中表有哪些字段、该字段是否为null、默认值是什么........诸如此的限制条件，所以**ES中的映射就是：数据的使用规则设置**



mapping是对索引库中文档的约束，常见的mapping属性包括：

- index：是否创建索引，默认为true
- analyzer：使用哪种分词器
- properties：该字段的子字段

更多类型去官网查看：https://www.elastic.co/guide/en/elasticsearch/reference/8.8/mapping-params.html





创建索引库，最关键的是mapping映射，而mapping映射要考虑的信息包括：

- 字段名
- 字段数据类型
- 是否参与搜索
- 是否需要分词
- 如果分词，分词器是什么？

其中：

- 字段名、字段数据类型，可以参考数据表结构的名称和类型
- 是否参与搜索要分析业务来判断，例如图片地址，就无需参与搜索
- 是否分词呢要看内容，内容如果是一个整体就无需分词，反之则要分词
- 分词器，我们可以统一使用ik_max_word





```json
{
  "mappings": {
    "properties": {		// 子字段
      "字段名1":{		// 定义字段名
        "type": "text",		// 该字段的类型
        "analyzer": "ik_smart"		// 该字段采用的分词器类型 这是ik分词器中的，一种为ik_smart 一种为ik_max_word，具体看一开始给的系列知识链接
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"		// 该字段是否可以被索引，默认值为trus，即：不想被搜索的字段就可以显示声明为false
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}
```

**创建索引库的同时，创建数据结构约束：**

```json
// 格式
PUT /索引库名称				// 创建索引库
{						// 同时创建数据结构约束信息
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}



// 示例
PUT /user
{
  "mappings": {
    "properties": {
      "info":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email":{
        "type": "keyword",
        "index": "falsae"
      },
      "name":{
        "properties": {
          "firstName": {
            "type": "keyword"
          },
		 "lastName": {
			"type": "keyword"
          }
        }
      },
      // ... 略
    }
  }
}
```





### index 索引库

**所谓索引：类似于关系型数据库中的数据库**

但是索引这个东西在ES中又有点东西，它的作用和关系型数据库中的索引是一样的，相当于门牌号，一个标识，旨在：提高查询效率，当然，不是说只针对查询，CRUD都可以弄索引，所以这么一说ES中的索引和关系型数据库中的索引是一样的，就不太类似于关系型中的数据库了，此言差矣！在关系型中有了数据库，才有表结构（ 行、列、类型...... ）

而在ES中就是有了索引，才有doc、field.....，因此：这就类似于关系型中的数据库，只是作用和关系型中的索引一样罢了

**因此：ES中索引类似于关系型中的数据库，作用：类似于关系型中的索引，旨在：提高查询效率，当然：在一个集群中可以定义N多个索引，同时：索引名字必须采用全小写字母**

当然：也别忘了有一个倒排索引

- 关系型数据库通过增加一个**B+树**索引到指定的列上，以便提升数据检索速度。而ElasticSearch 使用了一个叫做 `倒排索引` 的结构来达到相同的目的



**创建索引：** 相当于在创建数据库

```json
# 在kibana中进行的操作
PUT /索引库名称

# 在postman之类的地方创建
http://ip:port/indexName     如：http://127.0.0.1:9200/createIndex    	请求方式：put
```

**注：put请求具有幂等性**，幂等性指的是： 不管进行多少次重复操作，都是得到相同的结果。可以采用把下面的请求多执行几次，然后：观察返回的结果

**具有幂等性的有：put、delete、get**



**查看索引库：**

```json
# 查看指定的索引库
GET /索引库名

# 查看所有的索引库
GET /_cat/indices?v 
```



**修改索引库：**

- 倒排索引结构虽然不复杂，但是一旦数据结构改变（比如改变了分词器），就需要重新创建倒排索引，这简直是灾难。因此索引库**一旦创建，无法修改mapping**。



虽然无法修改mapping中已有的字段，但是却允许添加新的字段到mapping中，因为不会对倒排索引产生影响。

**语法说明**：

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
        // ............
    }
  }
}
```



**删除索引库：**

```json
DELETE /索引库名
```







### 分词器

#### 内置分词器

**1、标准分析器 standard：** 根据Unicode定义的单词边界来划分文本，将字母转成小写，去掉大部分的标点符号，从而得到的各种语言的最常用文本选择，==另外：这是ES的默认分析器==



**2、简单分析器 simple：** 按非字母的字符分词，例如：数字、标点符号、特殊字符等，会去掉非字母的词，大写字母统一转换成小写



**3、空格分析器 whitespace：** 简单按照空格进行分词，相当于按照空格split了一下，大写字母不会转换成小写



**4、去词分析器 stop：**会去掉无意义的词（此无意义是指语气助词等修饰性词，补语文：语气词是疑问语气、祈使语气、感叹语气、肯定语气和停顿语气），例如：the、a、an 、this等，大写字母统一转换成小写



**5、不拆分分析器 keyword：** 就是将整个文本当作一个词





#### IK分词器

官网：https://github.com/medcl/elasticsearch-analysis-ik/releases

步骤：

1. 下载ik分词器。注意：版本对应问题，版本关系在这里查看 https://github.com/medcl/elasticsearch-analysis-ik
2. 上传解压，放到es的plugins插件目录
3. 重启es



此种分词器的分词器类型：

1. **ik_max_word**        是细粒度的分词，就是：穷尽词汇的各种组成。如4个字是一个词，继续看3个字是不是一个词，再看2个字又是不是一个词，以此穷尽..........
2. **ik_smart**                 是粗粒度的分词。如：那个吊毛也是一个程序员，就先看整句话是不是一个词(length = 11)，不是的话，就看length-1是不是一个词.....，如果某个长度是一个词了，那么这长度内的内容就不看了，继续看其他的是不是一个词，如“那个"是一个词，那就看后面的内容，继续length、length-1、length-2........



在ik分词器的 config/IKAnalyzer.cfg.xml 中可以配置扩展词典和停用词典(即：敏感词)





#### 拼音分词器

官网：https://github.com/medcl/elasticsearch-analysis-pinyin

安装和IK分词器一样

- 下载
- 上传解压
- 重启es



测试拼音分词器

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627215749039-2092610734.png" alt="image-20230627210119445" />

由上可知，伴随2个问题：

1. 只进行了拼音分词，汉字分词不见了
2. 只采用拼音分词会出现一种情况：同音字，如“狮子”，“虱子”，这样的话明明想搜索的是“狮子”，结果“虱子”也出来了，所以这种搜索效果不好



因此：需要定制，让汉字分词出现，同时搜索时使用的汉字是什么就是什么，别弄同音字



要完成上面的需求，就需要结合文档分析的过程

在ES中有一个**文档分析的过程**，文档分析的过程也很简单：

1. **将文本拆成适合于倒排索引的独立的词条，然后把这些词条统一变为一个标准格式，从而使文本具有“可搜索性”。** 

而这个文档分析的过程在ES是由一个叫做“分析器 analyzer”的东西来做的，这个分析器里面做了三个步骤

1. 字符过滤器(character filters)：就是用来处理一些字符的嘛，像什么将 & 变为 and 啊、去掉HTML元素啊之类的。它是文本字符串在经过分词之前的一个步骤，文本字符串是按文本顺序经过每个字符串过滤器从而处理字符串
2. 分词器(tokenizer)：见名知意，就是用来分词的，也就是将字符串拆分成词条（ 字 / 词组 ），这一步和Java中String的split()一样的，通过指定的要求，把内容进行拆分，如：空格、标点符号
3. Token过滤器(tokenizer filter)：这个玩意儿的作用就是 词条经过每个Token过滤器，从而对数据再次进行筛选，如：字母大写变小写、去掉一些不重要的词条内容、添加一些词条（ 如：同义词 ）





举例理解：character filters、tokenizer、tokenizer filter

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627215749039-764720909.png" alt="image-20210723210427878" />

因此现在自定义分词器就变成如下的样子：

**注：** 是建立索引时自定义分词器，即自定义的分词器只对当前索引库有效

```json
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": { // 自定义分词器
        "my_analyzer": {  // 分词器名称
          "tokenizer": "ik_max_word",
          "filter": "py"
        }
      },
      "filter": { // 自定义tokenizer filter
        "py": { // 过滤器名称
          "type": "pinyin", // 过滤器类型，这里是pinyin，这些参数都在 拼音分词器官网有
		  "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer",	// 指明在索引时使用的分词器
        "search_analyzer": "ik_smart"	// 指明搜索时使用的分词器
      }
    }
  }
}
```



使用自定义分词器：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627215749040-412706024.png" alt="image-20230627212610200" />





### shards 分片

**这玩意儿就类似于关系型中的分表**

在关系型中如果一个表的数据太大了，查询效率很低、响应很慢，所以就会采用大表拆小表，如：用户表，不可能和用户相关的啥子东西都放在一张表吧，这不是找事吗？因此：需要分表

相应的在ES中，也需要像上面这么干，如：存储100亿文档数据的索引，在单节点中没办法存储这么多的文档数据，所以需要进行切割，就是将这整个100亿文档数据切几刀，然后每一刀切分出来的每份数据就是一个分片 （ 索引 ），然后将切开的每份数据单独放在一个节点中，这样切开的所有文档数据合在一起就是一份完整的100亿数据，因此：这个的作用也是为了提高效率

**创建一个索引的时候，可以指定想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上**

**分片有两方面的原因：**

- 允许水平分割 / 扩展内容容量，水平扩充，负载均衡嘛
- 允许在分片之上进行分布式的、并行的操作，进而提高性能 / 吞吐量

**注意： 当 Elasticsearch 在索引中搜索的时候， 它发送查询到每一个属于索引的分片，然后合并每个分片的结果到一个全局的结果集中**



### replicas 副本

**这不是游戏中的刷副本的那个副本啊。是指：分片的复制品**

失败是常有的事嘛，所以：在ES中也会失败呀，可能因为网络、也可能因此其他鬼原因就导致失败了，此时不就需要一种故障转移机制吗，也就是 **创建分片的一份或多份拷贝，这些拷贝就叫做复制分片( 副本 )**

**副本（ 复制分片 ）之所以重要，有两个原因：**

- 在分片 / 节点失败的情况下，**提供了高可用性。因为这个原因，复制分片不与原 / 主（ original / primary ）分片置于同一节点上是非常重要的**
- 扩展搜索量 / 吞吐量，因为搜索可以在所有的副本上并行运行

多说一嘴，分片和副本这两个不就是配套了吗，分片是切割数据，放在不同的节点中（ 服务中 ）；副本是以防服务宕掉了，从而丢失数据，进而把分片拷贝了任意份。这个像什么？不就是主备吗（ 我说的是主备，不是主从啊 ，这两个有区别的，主从是主机具有写操作，从机具有读操作；而主备是主机具有读写操作，而备机只有读操作，不一样的啊 ）

**有个细节需要注意，在ES中，分片和副本不是在同一台服务器中，是分开的，如：分片P1在节点1中，那么副本R1就不能在节点1中，而是其他服务中，不然服务宕掉了，那数据不就全丢了吗**



### allocation 分配

前面讲到了分片和副本，对照Redis中的主备来看了，那么对照Redis的主从来看呢？主机宕掉了怎么重新选一个主机？Redis中是加了一个哨兵模式，从而达到的。那么在ES中哪个是主节点、哪个是从节点、分片怎么去分的？就是利用了分配

**所谓的分配是指： 将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。注意：这个过程是由 master 节点完成的，和Redis还是有点不一样的啊**

既然都说了这么多，那就再来一个ES的系统架构吧

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230617215725410-1123704279.png" alt="image" />

其中，**P表示分片、R表示副本**

**默认情况下，分片和副本都是1，根据需要可以改变**







## Java操作ES

### 操作索引

```java
import org.apache.http.HttpHost;
import org.elasticsearch.action.admin.indices.delete.DeleteIndexRequest;
import org.elasticsearch.action.admin.indices.flush.FlushRequest;
import org.elasticsearch.action.admin.indices.flush.FlushResponse;
import org.elasticsearch.action.support.master.AcknowledgedResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.client.indices.CreateIndexRequest;
import org.elasticsearch.client.indices.CreateIndexResponse;
import org.elasticsearch.client.indices.GetIndexRequest;
import org.elasticsearch.client.indices.GetIndexResponse;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

import static com.zixieqing.hotel.constant.MappingConstant.mappingContext;

/**
 * elasticsearch的索引库测试
 * 规律：esClient.indices().xxx(xxxIndexRequest(IndexName), RequestOptions.DEFAULT)
 *      其中 xxx 表示要对索引进行的操作，如：create、delete、get、flush、exists.............
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o1IndexTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(RestClient.builder(HttpHost.create("http://ip:9200")));
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 创建索引 并 创建字段的mapping映射关系
     */
    @Test
    void createIndexAndMapping() throws IOException {
        // 1、创建索引
        CreateIndexRequest request = new CreateIndexRequest("indexName");
        // 2、创建字段的mapping映射关系   参数1：编写的mapping json字符串  参数2：采用的文本类型
        request.source(mappingContext, XContentType.JSON);
        // 3、发送请求 正式创建索引库与mapping映射关系
        CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);
        // 查看是否创建成功
        System.out.println("response.isAcknowledged() = " + response.isAcknowledged());
        // 判断指定索引库是否存在
        boolean result = client.indices().exists(new GetIndexRequest("indexName"), RequestOptions.DEFAULT);
        System.out.println(result ? "hotel索引库存在" : "hotel索引库不存在");
    }

    /**
     * 删除指定索引库
     */
    @Test
    void deleteIndexTest() throws IOException {
        // 删除指定的索引库
        AcknowledgedResponse response = client.indices()
                .delete(new DeleteIndexRequest("indexName"), RequestOptions.DEFAULT);
        // 查看是否成功
        System.out.println("response.isAcknowledged() = " + response.isAcknowledged());
    }

    // 索引库一旦创建，则不可修改，但可以添加mapping映射

    /**
     * 获取指定索引库
     */
    @Test
    void getIndexTest() throws IOException {
        // 获取指定索引
        GetIndexResponse response = client.indices()
                .get(new GetIndexRequest("indexName"), RequestOptions.DEFAULT);
    }

    /**
     * 刷新索引库
     */
    @Test
    void flushIndexTest() throws IOException {
        // 刷新索引库
        FlushResponse response = client.indices().flush(new FlushRequest("indexName"), RequestOptions.DEFAULT);
        // 检查是否成功
        System.out.println("response.getStatus() = " + response.getStatus());
    }
}
```





### 操作文档

#### 基本的CRUD操作

```java
import com.alibaba.fastjson.JSON;
import com.zixieqing.hotel.pojo.Hotel;
import com.zixieqing.hotel.pojo.HotelDoc;
import com.zixieqing.hotel.service.IHotelService;
import org.apache.http.HttpHost;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * elasticsearch的文档测试
 * 规律：esClient.xxx(xxxRequest(IndexName, docId), RequestOptions.DEFAULT)
 *      其中 xxx 表示要进行的文档操作，如：
 *          index   新增文档
 *          delete  删除指定id文档
 *          get     获取指定id文档
 *          update  修改指定id文档的局部数据
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o2DocumentTest {
    @Autowired
    private IHotelService service;

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 添加文档
     */
    @Test
    void addDocumentTest() throws IOException {

        // 1、准备要添加的文档json数据
        // 通过id去数据库获取数据
        Hotel hotel = service.getById(36934L);
        // 当数据库中定义的表结构和es中定义的字段mapping映射不一致时：将从数据库中获取的数据转成 es 中定义的mapping映射关系对象
        HotelDoc hotelDoc = new HotelDoc(hotel);

        // 2、准备request对象    指定 indexName+文档id
        IndexRequest request = new IndexRequest("hotel").id(hotel.getId().toString());

        // 3、把数据转成json
        request.source(JSON.toJSONString(hotelDoc), XContentType.JSON);

        // 4、发起请求，正式在ES中添加文档    就是根据数据建立倒排索引，所以这里调用了index()
        IndexResponse response = client.index(request, RequestOptions.DEFAULT);

        // 5、检查是否成功     使用下列任何一个API均可   若成功二者返回的结果均是 CREATED
        System.out.println("response.getResult() = " + response.getResult());
        System.out.println("response.status() = " + response.status());
    }

    /**
     * 根据id删除指定文档
     */
    @Test
    void deleteDocumentTest() throws IOException {
        // 1、准备request对象
        DeleteRequest request = new DeleteRequest("indexName", "docId");

        // 2、发起请求
        DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
        // 查看是否成功   成功则返回 OK
        System.out.println("response.status() = " + response.status());
    }

    /**
     * 获取指定id的文档
     */
    @Test
    void getDocumentTest() throws IOException {
        // 1、获取request
        GetRequest request = new GetRequest"indexName", "docId");

        // 2、发起请求，获取响应对象
        GetResponse response = client.get(request, RequestOptions.DEFAULT);

        // 3、解析结果
        HotelDoc hotelDoc = JSON.parseObject(response.getSourceAsString(), HotelDoc.class);
        System.out.println("hotelDoc = " + hotelDoc);
    }

    /**
     * 修改指定索引库 和 文档id的局部字段数据
     * 全量修改是直接删除指定索引库下的指定id文档，然后重新添加相同文档id的文档即可
     */
    @Test
    void updateDocumentTest() throws IOException {
        // 1、准备request对象
        UpdateRequest request = new UpdateRequest("indexName", "docId");

        // 2、要修改那个字段和值      注：参数是 key, value 形式 中间是 逗号
        request.doc(
                "price",500
        );

        // 3、发起请求
        UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
        // 查看结果 成功则返回 OK
        System.out.println("response.status() = " + response.status());
    }
}
```



#### 批量操作

> 本质：把请求封装了而已，从而让这个请求可以传递各种类型参数，如：删除的、修改的、新增的，这样就可以搭配for循环



```java
package com.zixieqing.hotel;

import com.alibaba.fastjson.JSON;
import com.zixieqing.hotel.pojo.Hotel;
import com.zixieqing.hotel.pojo.HotelDoc;
import com.zixieqing.hotel.service.IHotelService;
import org.apache.http.HttpHost;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.get.MultiGetItemResponse;
import org.elasticsearch.action.get.MultiGetRequest;
import org.elasticsearch.action.get.MultiGetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.List;

/**
 * elasticsearch 批量操作文档测试
 * 规律：EsClient.bulk(new BulkRequest()
 *                    .add(xxxRequest("indexName").id().source())
 *                    , RequestOptions.DEFAULT)
 * 其中：xxx 表示要进行的操作，如
 *      index   添加
 *      delete  删除
 *      get     查询
 *      update  修改
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o3BulkDocumentTest {
    @Autowired
    private IHotelService service;

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 批量添加文档数据到es中
     */
    @Test
    void bulkAddDocumentTest() throws IOException {
        // 1、去数据库批量查询数据
        List<Hotel> hotels = service.list();

        // 2、将数据库中查询的数据转成 es 的mapping需要的对象
        BulkRequest request = new BulkRequest();
        for (Hotel hotel : hotels) {
            HotelDoc hotelDoc = new HotelDoc(hotel);
            // 批量添加文档数据到es中
            request.add(new IndexRequest("hotel")
                    .id(hotelDoc.getId().toString())
                    .source(JSON.toJSONString(hotelDoc), XContentType.JSON));
        }

        // 3、发起请求
        BulkResponse response = client.bulk(request, RequestOptions.DEFAULT);
        // 检查是否成功   成功则返回OK
        System.out.println("response.status() = " + response.status());
    }

    /**
     * 批量删除es中的文档数据
     */
    @Test
    void bulkDeleteDocumentTest() throws IOException {
        // 1、准备要删除数据的id
        List<Hotel> hotels = service.list();

        // 2、准备request对象
        BulkRequest request = new BulkRequest();
        for (Hotel hotel : hotels) {
            // 根据数据id 批量删除es中的文档
            request.add(new DeleteRequest("hotel").id(hotel.getId().toString()));
        }

        // 3、发起请求
        BulkResponse response = client.bulk(request, RequestOptions.DEFAULT);
        // 检查是否成功       成功则返回 OK
        System.out.println("response.status() = " + response.status());
    }

    
    // 批量获取和批量修改是同样的套路  批量获取还可以使用 mget 这个API


    /**
     * mget批量获取
     */
    @Test
    void mgetTest() throws IOException {
        List<Hotel> hotels = service.list();

        // 1、准备request对象
        MultiGetRequest request = new MultiGetRequest();
        for (Hotel hotel : hotels) {
            // 添加get数据    必须指定index 和 文档id，可以根据不同index查询
            request.add("hotel", hotel.getId().toString());
        }

        // 2、发起请求，获取响应
        MultiGetResponse responses = client.mget(request, RequestOptions.DEFAULT);
        for (MultiGetItemResponse response : responses) {
            GetResponse resp = response.getResponse();
            // 如果存在则打印响应信息
            if (resp.isExists()) {
                System.out.println("获取到的数据= " +
                        JSON.toJSONString(resp.getSourceAsString()));
            }
        }
    }
}
```





### 近实时搜索、文档刷新、文档刷写、文档合并

> **ES的最大好处就是实时数据全文检索**
>
> 但是：ES这个玩意儿并不是真的实时的，而是近实时 / 准实时
>
> 原因就是：ES的数据搜索是分段搜索，最新的数据在最新的段中(每一个段又是一个倒排索引)，只有最新的段刷新到磁盘中之后，ES才可以进行数据检索，这样的话，磁盘的IO性能就会极大的影响ES的查询效率，而ES的目的就是为了：快速的、准确的获取到我们想要的数据，因此：降低数据查询处理的延迟就very 重要了，而ES对这方面做了什么操作？
>
> - 就是搞的**一主多副的方式**(一个主分片，多个副本分片)，这虽然就是一句话概括了，但是：里面的门道却不是那么简单的

**首先来看一下主副操作**
<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621160841027-1326216498.png" alt="image" />



但是：这种去找寻节点的过程想都想得到会造成延时，而**延时 = 主分片延时 + 主分片拷贝数据给副本的延时**

而且并不是这样就算完了，前面提到的分段、刷新到磁盘还没上堂呢，所以接着看
<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621160841001-432709753.png" alt="image" />



但是：在flush到磁盘中的时候，万一断电了呢？或者其他原因导致出问题了，那最后数据不就没有flush到磁盘吗

因此：其实还有一步操作，把数据保存到另外一个文件中去
<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621160840966-1495951055.png" alt="image" />



数据放到磁盘中之后，translog中的数据就会清空

同时更新到磁盘之后，用户就可以进行搜索数据了

**注意：**这里要区分一下，数据库中是先更新到log中，然后再更新到内存中，而ES是反着的，是先更新到Segment（可以直接认为是内存，因它本身就在内存中），再更新到log中

可是啊，还是有问题，flush刷写到磁盘是很耗性能的，假如：不断进行更新呢？这样不断进行IO操作，性能好吗？也不行，因此：继续改造(没有什么是加一层解决不了的，一层不够，那就再来一层)

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621160841099-1736093438.png" alt="image" />



加入了缓存之后，这缓存里面的数据是可以直接用来搜索的，这样就不用等到flush到磁盘之后，才可以搜索了，这大大的提高了性能，而flush到磁盘，只要时间到了，让它自个儿慢慢flush就可以了，**上面这个流程也叫：持久化 / 持久化变更**

**写入和打开一个新段的轻量的过程叫做refresh。默认情况下每个分片会每秒自动刷新一次。这就是为什么我们说 ES是近实时搜索：文档的变化并不是立即对搜索可见，但会在一秒之内变为可见**

刷新是1s以内完成的，这是有时间间隙的，所以会造成：搜索一个文档时，可能并没有搜索到，因此：解决办法就是使用refresh API刷新一下即可

**但是这样也伴随一个问题：虽然这种从内存刷新到缓存中看起来不错，但是还是有性能开销的。并不是所有的情况都需要refresh的，** 假如：是在索引日志文件呢？去refresh干嘛，浪费性能而已，所以此时：你要的是查询速度，而不是近实时搜索，因此：可以通过一个配置来进行改动，从而降低每个索引的刷新频率

```json
http://ip:port/index_name/_settings		// 请求方式：put

// 请求体内容
{
    "settings": {
        "refresh_interval": "60s"
    }
}
```

refresh_interval 可以在既存索引上进行动态更新。在生产环境中，当你正在建立一个大的新索引时，可以先关闭自动刷新，待开始使用该索引时，再把它们调回来。虽然有点麻烦，但是按照ES这个玩意儿来说，确实需要这么做比较好

```json
// 关闭自动刷新
http://ip:port/users/_settings		// 请求方式：put

// 请求体内容
{ 
    "refresh_interval": -1 
}

// 每一秒刷新
http://ip:port/users/_settings		// 请求方式：put
// 请求体内容
{ 
    "refresh_interval": "1s" 
}
```

另外：不断进行更新就会导致很多的段出现(在内存刷写到磁盘那里，会造成很多的磁盘文件 )，因此：在那里利用了文档合并的功能(也就是段的能力，合并文档，从而让刷写到磁盘中的文档变成一份)







### 路由计算

路由、路由，这个东西太熟悉了，在Vue中就见过路由router了(用来转发和重定向的嘛)

那在ES中的路由计算又是怎么回事？**这个主要针对的是ES集群中的存数据，试想：你知道你存的数据是在哪个节点 / 哪个主分片中吗（ 副本是拷贝的主分片，所以主分片才是核心 ）？**

当然知道啊，就是那几个节点中的任意一个嘛。娘希匹~这样的骚回答好吗？其实这是由一个公式来决定的

```txt
shard = hash(routing) % number_of_primary_shards

routing							 是一个任意值，默认是文档的_id，也可以自定义
number_of_primary_shards 		 表示主分片的数量
hash()							 是一个hash函数
```

这就解释了为什么我们要在创建索引的时候就确定好主分片的数量并且永远不会改变这个数量：因为如果数量变化了，那么之前所有路由的值都会无效，文档也再也找不到了



分片是将索引切分成任意份，然后得到的每一份数据都是一个单独的索引

分片完成后，我们存数据时，存到哪个节点上，就是通过`shard = hash(routing) % number_of_primary_shards`得到的

而我们查询数据时，ES怎么知道我们要找的数据在哪个节点上，就是通过`协调节点`做到的，它会去找到和数据相关的所有节点，从而轮询。所以最后的结果可能是从主分片上得到的，也可能是从副本上得到的，就看最后轮询到的是哪个节点罢了



### 分片控制

既然有了存数据的问题，那当然就有取数据的问题了。

**请问：在ES集群中，取数据时，ES怎么知道去哪个节点中取数据(假如在3节点中，你去1节点中，可以取到吗？)，因此：来了分片控制**

其实ES不知道数据在哪个节点中，但是：你自己却可以取到数据，为什么？

负载均衡，轮询嘛。所以这里有个小知识点，就是：**协调节点 `coordinating node`**，**我们可以发送请求到集群中的任一节点，每个节点都有能力处理任意请求，每个节点都知道集群中任一文档位置，这就是分片控制，而我们发送请求的那个节点就是：协调节点，它会去帮我们找到我们要的数据在哪里**

**因此：当发送请求的时候， 为了扩展负载，更好的做法是轮询集群中所有的节点**



### 集群下的数据写流程

新建、删除请求都是写操作， 必须在主分片上面完成之后才能被复制到相关的副本分片

**整个流程也很简单**

1. 客户端请求任意节点（协调节点）
2. 通过路由计算，协调节点把请求转向指定的节点
3. 转向的节点的主分片保存数据
4. 主节点再将数据转发给副本保存
5. 副本给主节点反馈保存结果
6. 主节点给客户端反馈保存结果
7. 客户端收到反馈结果

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621160557204-974310452.png" alt="image-20230621160556854" />



但是：从图中就可以看出来，这套流程完了，才可以做其他事（ 如：才可以去查询数据 ），那我为什么不可以异步呢？就是我只要保证到了哪一个步骤之后，就可以进行数据查询，所以：这里有两个小东西需要了解

在进行写数据时，我们做个小小的配置，这就是接下来的两个小节内容





#### consistency 一致性

这玩意就是为了和读数据搭配起来，写入和读取保证数据的一致性呗

**这玩意儿可以设定的值如下：**

- one ：只要主分片状态 ok 就允许执行读操作，这种写入速度快，但不能保证读到最新的更改
- all：这是强一致性，必须要主分片和所有副本分片的状态没问题才允许执行写操作
- quorum：这是ES的默认值。即大多数的分片副本状态没问题就允许执行写操作。这是折中的方法，write的时候，W>N/2，即参与写入操作的节点数W，必须超过副本节点数N的一半，在这个默认情况下，ES是怎么判定你的分片数量的，就一个公式：

```txt
int((primary + number_of_replicas) / 2) + 1

primary						指的是创建的索引数量
number_of_replicas			是指的在索引设置中设定的副本分片数
							如果你的索引设置中指定了当前索引拥有3个副本分片
							那规定数量的计算结果为：int(1 primary + 3 replicas) / 2) + 1 = 3，
							如果此时你只启动两个节点，那么处于活跃状态的分片副本数量就达不到规定数量，
							也因此你将无法索引和删除任何文档
```

- realtime request：就是从translog里头读，可以保证是最新的。**但是注意：get是最新的，但是检索等其他方法不是( 如果需要搜索出来也是最新的，需要refresh，这个会刷新该shard但不是整个index，因此如果read请求分发到repliac shard，那么可能读到的不是最新的数据，这个时候就需要指定preference=_primary)**





#### timeout 超时

如果没有足够的副本分片会发生什么？Elasticsearch 会等待，希望更多的分片出现。默认情况下，它最多等待 1 分钟。 如果你需要，你可以使用timeout参数使它更早终止，单位是毫秒，如：100就是100毫秒

新索引默认有1个副本分片，这意味着为满足规定数量应该需要两个活动的分片副本。 但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当number_of_replicas 大于1的时候，规定数量才会执行







### 集群下的数据读流程

有写流程，那肯定也要说一下读流程嘛，其实和写流程很像，只是变了那么一丢丢而已

**流程如下：**

1. 客户端发送请求到任意节点(协调节点)
2. 这里不同，此时协调节点会做两件事：1、通过路由计算得到分片位置，2、还会把当前查询的数据所在的另外节点也找到(如：副本)
3. 为了负载均衡(可能某个节点中的访问量很大嘛，减少一下压力咯)，所以就会对查出来的所有节点做轮询操作，从而找到想要的数据. 因此：你想要的数据在主节点中有、副本中也有，但是：给你的数据可能是主节点中的，也可能是副本中的 ，看轮询到的是哪个节点中的
4. 节点反馈结果
5. 客户端收到反馈结果

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621164044989-2027781030.png" alt="image-20230619202223102" />



**这里有个注意点：** 在文档( 数据 ）被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。 在这种情况下，副本分片可能会报文档不存在，但是主分片可能成功返回文档。 一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的







### 集群下的更新操作流程

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621165145137-220191768.png" alt="image-20230619202310833" />



1. 客户端向node 1发送更新请求
2. 它将请求转发到主分片所在的node 3
3. node 3从主分片检索文档，修改_source字段中的JSON，并且尝试重新索引主分片的文档。如果文档已经被另一个进程修改,它会重试步骤3 ,超过retry_on_conflict次后放弃
4. 如果 node 3成功地更新文档，它将新版本的文档并行转发到node 1和 node 2上的副本分片，重新建立索引。一旦所有副本分片都返回成功，node 3向协调节点也返回成功，协调节点向客户端返回成功



当然：上面有个漏洞，就是万一在另一个进程修改之后，当前修改进程又去修改了，那要是把原有的数据修改了呢？这不就成关系型数据库中的“不可重复读”了吗？

- 不会的。因为当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。注意点：这些更改将会“异步转发”到副本分片，并且不能保证它们以相同的顺序到达。 如果 ES 仅转发更改请求，则可能以错误的顺序应用更改，导致得到的是损坏的文档





### 集群下的批量更新操作流程

这个其实更容易理解，单文档更新懂了，那多文档更新就懂了嘛，多文档就请求拆分呗

**所谓的多文档更新就是：将整个多文档请求分解成每个分片的文档请求，并且将这些请求并行转发到每个参与节点。协调节点一旦收到来自每个节点的应答，就将每个节点的响应收集整理成单个响应，返回给客户端**



原理图的话：我就在网上偷一张了
<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621165301710-335819263.png" alt="image" />



其实mget 和 bulk API的模式就类似于单文档模式。区别在于协调节点知道每个文档存在哪个分片中



**用单个 mget 请求取回多个文档所需的步骤顺序:**

1. 客户端向 Node 1 发送 mget 请求
2. Node 1为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复，Node 1 构建响应并将其返回给客户端。可以对docs数组中每个文档设置routing参数

- bulk API， 允许在单个批量请求中执行多个创建、索引、删除和更新请求

 <img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230621165301646-2122224632.png" alt="img" /> 





**bulk API 按如下步骤顺序执行：**

1. 客户端向Node 1 发送 bulk请求
2. Node 1为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机
3. 主分片一个接一个按顺序执行每个操作。当每个操作成功时,主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端











### Java进行DSL文档查询

其实这种查询都是套路而已，一看前面玩的DSL查询的json形式是怎么写的，二看你要做的是什么查询，然后就是用 queryBuilds  将对应的查询构建出来，其他都是相同套路了



#### 查询所有 match all

> match all：查询出所有数据



```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * es的dsl文档查询之match all查询所有，也可以称之为 全量查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o1MatchAll {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }


    /**
     * 全量查询：查询所有数据
     */
    @Test
    void matchAllTest() throws IOException {
        // 1、准备request
        SearchRequest request = new SearchRequest("indexName");
        // 2、指定哪种查询/构建DSL语句
        request.source().query(QueryBuilders.matchAllQuery());
        // 3、发起请求 获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 4、处理响应结果
        // 4.1、获取结果中的Hits
        SearchHits searchHits = response.getHits();
        // 4.2、获取Hits中的total
        long total = searchHits.getTotalHits().value;
        System.out.println("总共获取了 " + total + " 条数据");
        // 4.3、获取Hits中的hits
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            // 4.3.1、获取hits中的source 也就是真正的数据，获取到之后就可以用来处理自己要的逻辑了
            String source = hit.getSourceAsString();
            System.out.println("source = " + source);
        }
    }
}
```

Java代码和前面玩的DSL语法的对应情况：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230625153725882-834216864.png" alt="image-20230623213506444" />





#### 全文检索查询

##### match 单字段查询 与 multi match多字段查询

下面的代码根据情境需要，可自行将响应结果处理进行抽取

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DLS之全文检索查询：利用分词器对用户输入内容分词，然后去倒排索引库中匹配
 * match_query 单字段查询 和 multi_match_query 多字段查询
 *
 * <p>@author       : ZiXieqing</p>
 */


@SpringBootTest
public class o2FullTextTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * match_query  单字段查询
     */
    @Test
    void matchQueryTest() throws IOException {
        // 1、准备request
        SearchRequest request = new SearchRequest("indexName");
        // 2、准备DSL
        request.source().query(QueryBuilders.matchQuery("city", "上海"));
        // 3、发送请求，获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }

    /**
     * multi match 多字段查询 任意一个字段符合条件就算符合查询条件
     */
    @Test
    void multiMatchTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source().query(QueryBuilders.multiMatchQuery("成人用品", "name", "business"));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```





#### 精确查询

> **精确查询**：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段，所以**不会**对搜索条件分词



##### range 范围查询 和 term精准查询

> term：根据词条精确值查询
>
> range：根据值的范围查询



```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之精确查询：根据精确词条值查找数据，一般是查找keyword、数值、日期、boolean等类型字段，所以 不会 对搜索条件分词
 * range 范围查询 和 term 精准查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o3ExactTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * term 精准查询 根据词条精确值查询
     * 和 match 单字段查询有区别，term要求内容完全匹配
     */
    @Test
    void termTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source().query(QueryBuilders.termQuery("city", "深圳"));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }

    /**
     * range 范围查询
     */
    @Test
    void rangeTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source().query(QueryBuilders.rangeQuery("price").lte(250));
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```





#### 地理坐标查询

##### geo_distance 附近查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之地理位置查询
 * geo_bounding_box 矩形范围查询 和 geo_distance 附近查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o4GeoTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * geo_distance 附近查询
     */
    @Test
    void geoDistanceTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source()
                .query(QueryBuilders
                       .geoDistanceQuery("location")
                       // 方圆多少距离
                       .distance("15km")
                       // 圆中心点坐标
                       .point(31.21,121.5)
                );
        
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```





#### 复合查询

function_score 算分函数查询 是差不多的道理



##### bool 布尔查询之must、should、must not、filter查询

布尔查询是一个或多个查询子句的组合，每一个子句就是一个**子查询**。子查询的组合方式有：

- must：必须匹配每个子查询，类似“与”
- should：选择性匹配子查询，类似“或”
- must_not：必须不匹配，**不参与算分**，类似“非”
- filter：必须匹配，**不参与算分**

**注意：** 搜索时，参与**打分的字段越多，查询的性能也越差**。因此这种多条件查询时，建议这样做：

- 搜索框的关键字搜索，是全文检索查询，使用must查询，参与算分
- 其它过滤条件，采用filter查询。不参与算分



```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之复合查询：基础DSL查询进行组合，从而得到实现更复杂逻辑的复合查询
 * function_score 算分函数查询
 *
 * bool 布尔查询
 *  must     必须匹配每个子查询   即：and “与”   参与score算分
 *  should   选择性匹配子查询    即：or “或”    参与score算分
 *  must not 必须不匹配         即：“非"       不参与score算分
 *  filter   必须匹配           即：过滤        不参与score算分
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o5Compound {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }


    /**
     * bool布尔查询
     *  must     必须匹配每个子查询   即：and “与”   参与score算分
     *  should   选择性匹配子查询    即：or “或”    参与score算分
     *  must not 必须不匹配         即：“非"       不参与score算分
     *  filter   必须匹配           即：过滤        不参与score算分
     */
    @Test
    void boolTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        // 构建must   即：and 与
        boolQueryBuilder.must(QueryBuilders.termQuery("city", "北京"));
        // 构建should   即：or 或
        boolQueryBuilder.should(QueryBuilders.multiMatchQuery("速8", "brand", "name"));
        // 构建must not   即：非
        boolQueryBuilder.mustNot(QueryBuilders.rangeQuery("price").gte(250));
        // 构建filter   即：过滤
        boolQueryBuilder.filter(QueryBuilders.termQuery("starName", "二钻"));

        request.source().query(boolQueryBuilder);
        
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```

Java代码和前面玩的DSL语法对应关系：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230625153725882-1774469174.png" alt="image-20230624131548461" />





#### fuzzy 模糊查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.unit.Fuzziness;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之模糊查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest
public class o6FuzzyTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

	/**
     * 模糊查询
     */
    @Test
    void fuzzyTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        // fuzziness(Fuzziness.ONE)     表示的是：字符误差数  取值有：zero、one、two、auto
        // 误差数  指的是：fuzzyQuery("name","深圳")这里面匹配的字符的误差    可以有几个字符不一样，多/少几个字符？
        request.source().query(QueryBuilders
                               .fuzzyQuery("name", "深圳")
                               .fuzziness(Fuzziness.ONE)
                         );
        
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```







#### 排序和分页查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.sort.SortOrder;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * DSL之排序和分页
 *
 * <p>@author       : ZiXieqing</p>
 */


@SpringBootTest
public class o7SortAndPageTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * sort 排序查询
     */
    @Test
    void sortTest() throws IOException {
        SearchRequest request = new SearchRequest("indexName");
        request.source()
                .query(QueryBuilders.matchAllQuery())
                .sort("price", SortOrder.ASC);
        
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }

    /**
     * page 分页查询
     */
    @Test
    void pageTest() throws IOException {
        int page = 2, size = 20;
        
        SearchRequest request = new SearchRequest("indexName");
        request.source()
            .query(QueryBuilders.matchAllQuery())
            .from((page - 1) * size)
            .size(size);

        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        
        // 处理响应结果，后面都是一样的流程 都是解析json结果而已
        SearchHits searchHits = response.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("获取了 " + total + " 条数据");
        for (SearchHit hit : searchHits.getHits()) {
            String dataJson = hit.getSourceAsString();
            System.out.println("dataJson = " + dataJson);
        }
    }
}
```







#### 高亮查询

返回结果处理的逻辑有点区别，但思路都是一样的



```java
package com.zixieqing.hotel.dsl_query_document;

import com.alibaba.fastjson.JSON;
import com.zixieqing.hotel.HotelApp;
import com.zixieqing.hotel.pojo.HotelDoc;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.builder.SearchSourceBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.util.CollectionUtils;

import java.io.IOException;
import java.util.Map;

/**
 * DSL之高亮查询
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o8HighLightTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    /**
     * 高亮查询
     * 返回结果处理不太一样
     */
    @Test
    void highLightTest() throws IOException {
        SearchRequest request = new SearchRequest("hotel");
        request.source()
                .query(QueryBuilders.matchQuery("city", "北京"))
                .highlighter(SearchSourceBuilder
                             .highlight()
                             .field("name")  // 要高亮的字段
                             .preTags("<em>")    // 前置HTML标签 默认就是em
                        	 .postTags("</em>")  // 后置标签
                         	 .requireFieldMatch(false)	 // 是否进行查询字段和高亮字段匹配
                );

        // 发起请求，获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        
        // 处理响应结果
        for (SearchHit hit : response.getHits()) {
            String originalData = hit.getSourceAsString();
            HotelDoc hotelDoc = JSON.parseObject(originalData, HotelDoc.class);
            System.out.println("原始数据为：" + originalData);

            // 获取高亮之后的结果
            // key 为要进行高亮的字段，如上为field("name")   value 为添加了标签之后的高亮内容
            Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            if (!CollectionUtils.isEmpty(highlightFields)) {
                // 根据高亮字段，获取对应的高亮内容
                HighlightField name = highlightFields.get("name");
                if (name != null) {
                    // 获取高亮内容   是一个数组
                    String highLightStr = name.getFragments()[0].string();
                    hotelDoc.setName(highLightStr);
                }
            }

            System.out.println("hotelDoc = " + hotelDoc);
        }
    }
}
```

代码和DSL语法对应关系： request.source()  获取到的就是返回结果的整个json文档

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230625153726096-645662272.png" alt="image-20230624175348848" />









#### 聚合查询

**[聚合（](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)[aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)[）](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)**可以让我们极其方便地实现对数据的统计、分析、运算



聚合常见的有三类：

- **桶（Bucket）**聚合：用来对文档做分组
  - TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
  - Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组

- **度量（Metric）**聚合：用以计算一些值，比如：最大值、最小值、平均值等
  - Avg：求平均值
  - Max：求最大值
  - Min：求最小值
  - Stats：同时求max、min、avg、sum等
- **管道（pipeline）**聚合：其它聚合的结果为基础做聚合



> **注意：**参加聚合的字段必须是keyword、日期、数值、布尔类型，即：可以说只要不是 text 类型即可，因为text类型会进行分词，而聚合不能进行分词





```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.Aggregations;
import org.elasticsearch.search.aggregations.BucketOrder;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;
import java.util.List;

/**
 * 数据聚合 aggregation 可以让我们极其方便的实现对数据的统计、分析、运算
 * 桶（Bucket）聚合：用来对文档做分组
 *      TermAggregation：按照文档字段值分组，例如按照品牌值分组、按照国家分组
 *      Date Histogram：按照日期阶梯分组，例如一周为一组，或者一月为一组
 *
 *  度量（Metric）聚合：用以计算一些值，比如：最大值、最小值、平均值等
 *      Avg：求平均值
 *      Max：求最大值
 *      Min：求最小值
 *      Stats：同时求max、min、avg、sum等
 *
 *  管道（pipeline）聚合：其它聚合的结果为基础做聚合
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o9AggregationTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    @Test
    void aggregationTest() throws IOException {
        // 获取request
        SearchRequest request = new SearchRequest("indexName");
        // 组装DSL
        request.source()
                .size(0)
                .query(QueryBuilders
                        .rangeQuery("price")
                        .lte(250)
                )
                .aggregation(AggregationBuilders
                        .terms("brandAgg")
                        .field("brand")
                        .order(BucketOrder.aggregation("scoreAgg.avg",true))
                        .subAggregation(AggregationBuilders
                                .stats("scoreAgg")
                                .field("score")
                        )
                );

        // 发送请求，获取响应
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        
        // 处理响应结果
        System.out.println("response = " + response);
        // 获取全部聚合结果对象 getAggregations
        Aggregations aggregations = response.getAggregations();
        // 根据聚合名 获取其聚合对象
        Terms brandAgg = aggregations.get("brandAgg");
        // 根据聚合类型 获取对应聚合对象
        List<? extends Terms.Bucket> buckets = brandAgg.getBuckets();
        for (Terms.Bucket bucket : buckets) {
            // 根据key获取其value
            String value = bucket.getKeyAsString();
            // 将value根据需求做处理
            System.out.println("value = " + value);
        }
    }
}
```

请求组装对应关系：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627213312189-633112157.png" alt="image-20230627140843561" />

响应结果对应关系：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627213312183-1009559133.png" alt="image-20230627141303392" />





#### 自动补全查询

```java
package com.zixieqing.hotel.dsl_query_document;

import com.zixieqing.hotel.HotelApp;
import org.apache.http.HttpHost;
import org.elasticsearch.action.search.SearchRequest;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.search.suggest.Suggest;
import org.elasticsearch.search.suggest.SuggestBuilder;
import org.elasticsearch.search.suggest.SuggestBuilders;
import org.elasticsearch.search.suggest.completion.CompletionSuggestion;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

/**
 * 自动补全 completion类型： 这个查询会匹配以用户输入内容开头的词条并返回
 *  参与补全查询的字段 必须 是completion类型
 *  字段的内容一般是用来补全的多个词条形成的数组
 *
 * <p>@author       : ZiXieqing</p>
 */

@SpringBootTest(classes = HotelApp.class)
public class o10Suggest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(
                RestClient.builder(HttpHost.create("http://ip:9200"))
        );
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }

    @Test
    void completionTest() throws IOException {
        // 准备request
        SearchRequest request = new SearchRequest("hotel");
        // 构建DSL
        request.source()
            .suggest(new SuggestBuilder().addSuggestion(
                "title_suggest",
                SuggestBuilders
                	.completionSuggestion("title")
                	.prefix("s")
                	.skipDuplicates(true)
                	.size(10)
             ));

        // 发起请求，获取响应对象
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        
        // 解析响应结果
        // 获取整个suggest对象
        Suggest suggest = response.getSuggest();
        // 通过指定的suggest名字，获取其对象
        CompletionSuggestion titleSuggest = suggest.getSuggestion("title_suggest");
        for (CompletionSuggestion.Entry options : titleSuggest) {
            // 获取每一个options中的test内容
            String context = options.getText().string();
            // 按需求对内容进行处理
            System.out.println("context = " + context);
        }
    }
}
```

代码与DSL、响应结果对应关系：

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230627235802008-1214986995.png" alt="image-20230627235426570" />









#### ES与MySQL数据同步

这里的同步指的是：MySQL发生变化，则elasticsearch索引库也需要跟着发生变化



数据同步一般有三种方式：同步调用方式、异步通知方式、监听MySQL的binlog方式





1. **同步调用：**

- 优点：实现简单，粗暴
- 缺点：业务耦合度高

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230628160455106-588758222.png" alt="image-20230628155716064" />



2. **异步通知：**

- 优点：低耦合，实现难度一般
- 缺点：依赖mq的可靠性



<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230628160455289-350716727.png" alt="image-20230628160432048" />



3. **监听MySQL的binlog文件：**canel是阿里巴巴的，它是将自己伪装成MySQL的slave((即：canel要基于MySQL主从实现)。canel监听MySQL的binlog文件，此文件发生改变canel就会让另外存储地也发生改变(如：MQ、ES、Redis......)

- 优点：完全解除服务间耦合
- 缺点：开启binlog增加数据库负担、实现复杂度高



<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230628160455195-1983290695.png" alt="image-20230628160321828" />



MySQL主从同步：

<img src="https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230827220155641-494759712.png" alt="image-20210821115914748" />





# Sentinel 微服务保护

Sentinel是阿里巴巴开源的一款微服务流量控制组件。官网地址：https://sentinelguard.io/zh-cn/index.html



## 雪崩问题与解决方式

> 所谓的雪崩指的是：微服务之间相互调用，调用链中某个微服务出现问题了，导致整个服务链的所有服务也跟着出问题，从而造成所有服务都不可用
>

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629232718861-1781673253.png" alt="image-20230629232716886" />





**解决方式：**

1. **超时处理**：是一种临时方针，即设置定时器，请求超过规定的时间就返回错误信息，不会无休止等待

   <img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230722001234984-1060313213.png" alt="image-20230629233450322" />

   缺点：在超时时间内，还未返回错误信息内，服务未处理完，请求激增，一样会导致后面的请求阻塞

   

2. **线程隔离**：也叫舱壁模式，即限定每个业务能使用的线程数，避免耗尽整个tomcat的资源

   <img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230629233810724-498144747.png" alt="image-20230629233809486" />

   缺点：会造成一定资源的浪费。明明服务已经不可用了，还占用固定数量的线程

   

3. **熔断降级**：

   1. **熔断：** 由“断路器”统计业务执行的异常比例，如果超出“阈值”则会熔断/暂停该业务，拦截访问该业务的一切请求，后续搞好了再开启。从而做到在流量过大时（或下游服务出现问题时），可以自动断开与下游服务的交互，并可以通过自我诊断下游系统的错误是否已经修正，或上游流量是否减少至正常水平来恢复自我恢复。熔断更像是自动化补救手段，可能发生在服务无法支撑大量请求或服务发生其他故障时，对请求进行限制处理，同时还可尝试性的进行恢复
   2. **降级：** 丢车保帅。针对非核心业务功能，核心业务超出预估峰值需要进行限流；所谓降级指的就是在预计流量峰值前提下，整体资源快不够了，忍痛将某些非核心服务先关掉，待渡过难关，再开启回来

   

4. **限流：** 也叫流量控制。指的是限制业务访问的QPS，避免服务因流量的突增而故障。是防御保护手段，从流量源头开始控制流量规避问题

   <img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230630001726890-238228668.png" alt="image-20230630001726188" />





**限流**是对服务的保护，避免因瞬间高并发流量而导致服务故障，进而避免雪崩。是一种**预防**措施

**超时处理、线程隔离、降级熔断**是在部分服务故障时，将故障控制在一定范围，避免雪崩。是一种**补救**措施





## 服务保护技术对比

在SpringCloud当中支持多种服务保护技术：

- [Netfix Hystrix](https://github.com/Netflix/Hystrix)
- [Sentinel](https://github.com/alibaba/Sentinel)
- [Resilience4J](https://github.com/resilience4j/resilience4j)



早期比较流行的是Hystrix框架(后面这吊毛不维护、不更新了)，所以目前国内实用最广泛的是阿里巴巴的Sentinel框架

|                | **Sentinel**                                   | **Hystrix**                   |
| -------------- | ---------------------------------------------- | ----------------------------- |
| 隔离策略       | 信号量隔离                                     | 线程池隔离/信号量隔离         |
| 熔断降级策略   | 基于慢调用比例或异常比例                       | 基于失败比率                  |
| 实时指标实现   | 滑动窗口                                       | 滑动窗口（基于 RxJava）       |
| 规则配置       | 支持多种数据源                                 | 支持多种数据源                |
| 扩展性         | 多个扩展点                                     | 插件的形式                    |
| 基于注解的支持 | 支持                                           | 支持                          |
| 限流           | 基于 QPS，支持基于调用关系的限流               | 有限的支持                    |
| 流量整形       | 支持慢启动、匀速排队模式                       | 不支持                        |
| 系统自适应保护 | 支持                                           | 不支持                        |
| 控制台         | 开箱即用，可配置规则、查看秒级监控、机器发现等 | 不完善                        |
| 常见框架的适配 | Servlet、Spring Cloud、Dubbo、gRPC  等         | Servlet、Spring Cloud Netflix |





## 安装sentinel

1. 下载：https://github.com/alibaba/Sentinel/releases 是一个jar包，下载了放到“非中文”目录中

2. 运行

   ```java
   java -jar sentinel-dashboard-1.8.1.jar
   ```

如果要修改Sentinel的默认端口、账户、密码，可以通过下列配置：

| **配置项**                       | **默认值** | **说明**   |
| -------------------------------- | ---------- | ---------- |
| server.port                      | 8080       | 服务端口   |
| sentinel.dashboard.auth.username | sentinel   | 默认用户名 |
| sentinel.dashboard.auth.password | sentinel   | 默认密码   |

例如，修改端口：

```sh
java -Dserver.port=8090 -jar sentinel-dashboard-1.8.1.jar
```

3. 访问。如http://localhost:8080，用户名和密码都是sentinel







## 入手sentinel

1. 依赖

   ```xml
   <!--sentinel-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId> 
       <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
   </dependency>
   ```

2. YAML配置

   ```yaml
   server:
     port: 8088
   spring:
     cloud: 
       sentinel:
         transport:
   # 		sentinel的地址
           dashboard: localhost:8080
   ```

3. 然后将服务提供者、服务消费者、网关、Feign……启动，发送请求即可在前面sentinel的ui控制台看到信息了

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230630191055598-1891678688.png" alt="image-20230630191055722" />







## 限流 / 流量控制

雪崩问题虽然有四种方案，但是限流是避免服务因突发的流量而发生故障，是对微服务雪崩问题的预防，因此先来了解这种模式，但在了解这个之前先了解一下限流算法



### 限流算法

#### 固定窗口计数器算法

1. 将时间划分为多个窗口，窗口时间跨度称为Interval
2. 每个窗口维护一个计数器，每有一次请求就将计数器 +1，限流就是设置计数器阈值
3. 如果计数器超过了限流阈值，则超出阈值的请求都被丢弃

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714000702120-26075690.png" alt="image-20230714000659139" />

但是有个缺点：时间是不固定的。如0 - 1000ms是QPS(1秒内的请求数)，这样来看没有超过阈值，可是：4500 - 5500ms也是1s啊，这是不是也是QPS啊，像下面这样就超出阈值了，服务不得干爬了

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714001041348-1019713348.png" alt="image-20230714001039504" />

#### 滑动窗口计数器算法

在固定窗口计数器算法的基础上，滑动窗口计数器算法会将一个窗口划分为n个更小的区间，如：

1. 窗口时间跨度Interval为1秒；区间数量 n = 2 ，则每个小区间时间跨度为500ms
2. 限流阈值依然为3，时间窗口（1秒）内请求超过阈值时，超出的请求被限流
3. **窗口会根据当前请求所在时间（currentTime）移动，窗口范围是从（currentTime-Interval）之后的第一个时区开始，到currentTime所在时区结束**





#### 令牌桶算法

1. 以固定的速率生成令牌，存入令牌桶中，如果令牌桶满了以后，不再生成令牌
2. 请求进入后，必须先尝试从桶中获取令牌，获取到令牌后才可以被处理，否则
3. 如果令牌桶中没有令牌，则请求等待或丢弃

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714001831603-54266991.png" alt="限流算法 - 令牌桶算法" />

也有个缺点：

1. 假如限流阈值是1000个请求
2. 假设捅中只能放1000个令牌，前一秒内没有请求，但是令牌已经生成了，放入了捅中
3. 之后下一秒来了2000个请求，可捅中前一秒生成了1000令牌，所以可以有1000个请求拿到令牌，从而放行，捅中没令牌了
4. 然后当前这一秒就要生成令牌，这样另外1000个请求也可以拿到令牌
5. 最后2000个请求都放行了，服务又干爬了





#### 漏桶算法

是对令牌桶算法做了改进：可以理解成请求在桶内排队等待

1. 将每个请求视作"水滴"放入"漏桶"进行存储
2. "漏桶"以固定速率向外"漏"出请求来执行，如果"漏桶"空了则停止"漏水”
3. 如果"漏桶"满了则多余的"水滴"会被直接丢弃

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230714002843437-1907902074.png" alt="限流算法 - 漏铜算法" />

#### 限流算法对比

因为计数器算法一般都会采用滑动窗口计数器，所以这里只对比三种算法

| **对比项**               | **滑动时间窗口**                             | **令牌桶**                                                   | **漏桶**                                       |
| ------------------------ | -------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| **能否保证流量曲线平滑** | 不能，但窗口内区间越小，流量控制越平滑       | 基本能，在请求量持续高于令牌生成速度时，流量平滑。但请求量在令牌生成速率上下波动时，无法保证曲线平滑 | 能，所有请求进入桶内，以恒定速率放行，绝对平滑 |
| **能否应对突增流量**     | 不能，徒增流量，只要高出限流阈值都会被拒绝。 | 能，桶内积累的令牌可以应对突增流量                           | 能，请求可以暂存在桶内                         |
| **流量控制精确度**       | 低，窗口区间越小，精度越高                   | 高                                                           | 高                                             |







### 簇点链路

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230630232924098-1383625366.png" alt="image-20230630232923426" />



> **簇点链路：** 就是项目内的调用链路，链路中被监控的每个接口就是一个“资源”

当请求进入微服务时，首先会访问DispatcherServlet，然后进入Controller、Service、Mapper，这样的一个调用链就叫做**簇点链路**。簇点链路中被监控的每一个接口就是一个**资源**

默认情况下sentinel会监控SpringMVC的每一个端点（Endpoint，也就是controller中的方法），因此SpringMVC的每一个端点就是调用链路中的一个资源



例如下图中的端点：/order/{orderId}

<img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230630233547524-1380692667.png" alt="image-20230630233547622" />

流控、熔断等都是针对簇点链路中的资源来设置的，因此我们可以点击对应资源后面的按钮来设置规则：

1. 流控：流量控制
2. 降级：降级熔断
3. 热点：热点参数限流
4. 授权：请求的权限控制





### 入门流控

1. 点击下图按钮

   <img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230630234126489-1698184768.png" alt="image-20230630234126929" />

2. 设置基本流控信息

   <img src="https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230630235201380-320755412.png" alt="image-20230630235201675" />

   上图的含义：限制 /order/{orderId} 这个资源的单机QPS为1，即：每秒只允许1次请求，超出的请求会被拦截并报错







### 流控模式的分类

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701001533748-1421619907.png" alt="image-20230630235600999" />

在添加限流规则时，点击高级选项，可以选择三种**流控模式**：

1. **直接模式**：一句话来说就是“对当前资源限流”。统计当前资源的请求，当其触发阈值时，对当前资源直接限流。上面这张图就是此种模式。这也是默认的模式。采用的算法就是滑动窗口算法
2. **关联模式**：一句话来说就是“高优先级触发阈值，对低优先级限流”。统计与当前资源A**“相关”**的另一个资源B，A资源触发阈值时，对B资源限流
   
   如：在一个Controller中，一个高流量的方法和一个低流量的方法都调用了这个Controller中的另一个方法，为了预防雪崩问题，就对低流量的方法进行限流设置
   
   **适用场景**：两个有竞争关系的资源，一个优先级高，一个优先级低，优先级高的触发阈值时，就对优先级低的进行限流
3. **链路模式**：一句话来说就是“对请求来源做限流”。统计从“指定链路”访问到本资源的请求，触发阈值时，对指定链路限流
   
   如：两个不同链路的请求，如需要读库和写库，这两个请求都调用了同一个服务/资源/接口，所以为了需求考虑，可以设置读库达到了阈值就进行限流





示例：

1. **关联模式：** 对谁进行限流，就点击谁的流控按钮进行设置

   <img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701010740490-994040095.png" alt="image-20230701010739230" />

   上图含义：当 /order/update 请求单机达到 每秒1000 请求量的阈值时，就会对 /order/query 进行限流，从而避免影响 /order/update 资源

   

2. **链路模式：** 请求链路访问的是哪个资源，就点击哪个资源的流控按钮进行配置

   <img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701011442949-2124782434.png" alt="image-20230701011441588" />
   
   上图含义：只有来自 /user/queryGoods 链路的请求来访问 /order/queryGoods 资源时，每秒请求量达到1000，就会对 /user/queryGoods 进行限流





> **链路模式的注意事项：**
>
> 1. 默认情况下，Service中的方法是不被Sentinel监控的，想要Service中的方法也被Sentinel监控的话，则需要我们自己通过    @SentinelResource("起个名字 或 像controllerz中请求路径写法")    注解来标记要监控的方法
>
> 2. 链路模式中，是对不同来源的两个链路做监控。但是sentinel默认会给进入SpringMVC的所有请求设置同一个root资源，进行了context整合，所以会导致链路模式失效。因此需要关闭一个context整合设置：
>
>    ```yaml
>    spring:
>      cloud:
>        sentinel:
>          web-context-unify: false # 关闭context整合
>    ```
>
>    同一个root资源指的是：
>
>    <img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701014516195-545720538.png" alt="image-20230701014514323" />







### 流控效果及其分类

> **流控效果**：指请求达到流控阈值时应该采取的措施



**分类**

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701014736593-1780186698.png" alt="image-20230701014735316" />

1. **快速失败**：达到阈值后，新的请求会被立即拒绝并抛出 FlowException异常。是默认的处理方式
2. **warm up**：预热模式，对超出阈值的请求同样是拒绝并抛出异常。但这种模式阈值会动态变化，从一个较小值逐渐增加到最大阈值
3. **排队等待**：让所有的请求按照先后次序排队执行，两个请求的间隔不能小于指定时长







#### warn up 预热模式

> **warm up**：预热模式，对超出阈值的请求同样是拒绝并抛出异常。但这种模式阈值会动态变化，从一个较小值逐渐增加到最大阈值



阈值一般是一个微服务能承担的最大QPS，但是一个服务刚刚启动时，一切资源尚未初始化（**冷启动**），如果直接将QPS跑到最大值，可能导致服务瞬间宕机



warm up也叫**预热模式**，是应对服务冷启动的一种方案

```txt
请求阈值初始值 = maxThreshold / coldFactor
```

- maxThreshold 就是设置的QPS数量。持续指定时长后，逐渐提高到maxThreshold值。
- coldFactor 预热因子，默认值是3



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701015810163-235886151.png" alt="image-20230701015808477" />







#### 排队等待

> **排队等待**：让所有的请求按照先后次序排队执行，两个请求的间隔不能小于指定时长
>
> **采用的算法**：基于漏桶算法

当请求超过QPS阈值时，快速失败和warm up 会拒绝新的请求并抛出异常

而排队等待则是让所有请求进入一个队列中，然后按照阈值允许的时间间隔依次执行。后来的请求必须等待前面执行完成，如果请求预期的等待时间超出最大时长，则会被拒绝

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701021827938-48306738.png" alt="image-20230701021826754" />

QPS = 5，那么 1/5(个/ms) = 200(个/ms)，意味着每200ms处理1个队列中的请求；timeout = 2000，意味着**预期等待时长**超过2000ms的请求会被拒绝并抛出异常

那什么叫做预期等待时长呢？

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701022552990-2084402102.png" alt="image-20230701022551052" />

如果使用队列模式做流控，所有进入的请求都要排队，以固定的200ms的间隔执行，QPS会变的很平滑

平滑的QPS曲线，对于服务器来说是更友好的







### 热点参数限流

> 之前的限流是统计访问某个资源的所有请求，判断是否超过QPS阈值
>
> 热点参数限流是**分别统计参数值相同的请求**，判断是否超过QPS阈值
>
> **采用的算法：** 令牌桶算法
>
> **注意事项**：热点参数限流对默认的SpringMVC资源无效，需要利用@SentinelResource注解标记资源，例如：
>
> <img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701121246681-1500235555.png" alt="image-20230701121244611" />



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701023409228-1581920767.png" alt="image-20230701023349080" />

但是配置时不要通过上面按钮点击配置，会有BUG，而是通过下图中的方式：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701023747870-1133268354.png" alt="image-20230701023746175" />





**所谓的参数值指的是**：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701023139319-2063828243.png" alt="image-20230701023138057" />

id参数值会有变化，热点参数限流会根据参数值分别统计QPS

当id=1的请求触发阈值被限流时，id值不为1的请求不受影响







#### 全局参数限流

就是基础设置，没有加入高级设置的情况

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701121801661-838481950.png" alt="image-20230701121800057" />

上图含义：对于来访问hot资源的请求，每1秒**相同参数值**的请求数不能超过10000









#### 热点参数限流

刚才的配置中，对查询商品这个接口的所有商品一视同仁，QPS都限定为10000

而在实际开发中，可能部分商品是热点商品，例如秒杀商品，我们希望这部分商品的QPS限制与其它商品不一样，高一些。那就需要配置热点参数限流的高级选项了

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701122406512-675478864.png" alt="image-20230701122405067" />

上图含义：对于来访问hot资源的请求，id=110时的QPS阈值为30000，id=4132443时的QPS阈值为50000，id为其他的则QPS阈值为10000





## Seatinel限流与Gateway限流的差异

Gateway则采用了基于Redis实现的令牌桶算法。而Sentinel内部所有算法都有：：

1. 默认限流模式是基于滑动时间窗口算法
2. 排队等待的限流模式则基于漏桶算法
3. 而热点参数限流则是基于令牌桶算法







## Sentinel整合Feign

Sentinel是做服务保护的，而在微服务中调来调去是常有的事，要远程调用就离不开Feign



1. **修改配置，开启sentinel功能：** 在服务“消费方”的feign配置添加如下配置内容

```yaml
feign:
  sentinel:
    enabled: true # 开启feign对sentinel的支持
```

2. **feign-client中编写失败降级逻辑：** 后面的流程就是前面玩Fengn时失败降级的流程

```java
package com.zixieqing.feign.fallback;

import com.zixieqing.feign.clients.UserClient;
import com.zixieqing.feign.pojo.User;
import feign.hystrix.FallbackFactory;
import lombok.extern.slf4j.Slf4j;

/**
 * userClient失败时的降级处理
 *
 * <p>@author       : ZiXieqing</p>
 */

@Slf4j
public class UserClientFallBackFactory implements FallbackFactory<UserClient> {
    @Override
    public UserClient create(Throwable throwable) {
        return new UserClient() {
            /**
             * 重写userClient中的方法，编写失败时的降级逻辑
             */
            @Override
            public User findById(Long id) {
                log.info("userClient的findById()在进行 id = {} 时失败", id);
                return new User();
            }
        };
    }
}
```

3. **将失败降级逻辑的类丢给Spring容器**

```java
@Bean
public UserClientFallBackFactory userClientFallBackFactory() {
    return new UserClientFallBackFactory();
}
```

4. **在相关feign-client定义处使用fallbackFactory回调函数即可**

```java
package com.zixieqing.feign.clients;


import com.zixieqing.feign.fallback.UserClientFallBackFactory;
import com.zixieqing.feign.pojo.User;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(value = "userservice",fallbackFactory = UserClientFallBackFactory.class)
public interface UserClient {

    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);
}
```

5. 调用，失败时就会进入自定义的失败逻辑中

```java
package com.zixieqing.order.service;

import com.zixieqing.feign.clients.UserClient;
import com.zixieqing.feign.pojo.User;
import com.zixieqing.order.mapper.OrderMapper;
import com.zixieqing.order.pojo.Order;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;

    @Autowired
    private UserClient userClient;

    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        // 2.用Feign远程调用
        User user = userClient.findById(order.getId());
        // 3.封装user到Order
        order.setUser(user);
        // 4.返回
        return order;
    }
}
```









## 隔离与降级

### 线程隔离

线程隔离有两种方式实现：

1. **线程池隔离**：给每个服务调用业务分配一个线程池，利用线程池本身实现隔离效果
   
   优点：
   
   - 支持主动超时：也就是调用进行逻辑处理时超过了规定时间，直接噶了，不再让其继续处理
   - 支持异步调用：线程池隔离了嘛，彼此不干扰，因此可以异步了
   
   缺点：造成资源浪费。明明被调用的服务都出问题了，还占用固定的线程池数量
   
   适用场景：低扇出。MQ中扇出交换机的那个扇出，也就是较少的请求量，扇出/广播到很多服务上



2. **信号量隔离**（Sentinel默认采用）：不创建线程池，而是计数器模式，记录业务使用的线程数量，达到信号量上限时，禁止新的请求

   优点：轻量级、无额外开销

   缺点：不支持主动超时、不支持异步调用

   适用场景：高频调用、高扇出



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701215706517-1071618767.png" alt="image-20210716123036937" />





#### 配置Sentinel的线程隔离-信号量隔离

在添加限流规则时，可以选择两种阈值类型：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701223109157-1720665438.png" alt="image-20230701223024446" />









### 熔断降级

熔断降级是解决雪崩问题的重要手段。其思路是由**断路器**统计服务调用的异常比例、慢请求比例，如果超出阈值则会**熔断**该服务。即拦截访问该服务的一切请求；而当服务恢复时，断路器会放行访问该服务的请求

断路器控制熔断和放行是通过状态机来完成的：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701224956810-101544858.png" alt="image-20230701224942874" />

断路器熔断策略有三种：慢调用、异常比例、异常数

状态机包括三个状态：

- **Closed**：关闭状态，断路器放行所有请求，并开始统计异常比例、慢请求比例。超过阈值则切换到open状态
- **Open**：打开状态，服务调用被**熔断**，访问被熔断服务的请求会被拒绝，快速失败，直接走降级逻辑。Open状态默认5秒后会进入half-open状态
- **Half-Open**：半开状态，放行一次请求，根据执行结果来判断接下来的操作。
  - 请求成功：则切换到closed状态
  - 请求失败：则切换到open状态





#### 断路器熔断策略：慢调用

> **慢调用**：业务的响应时长（RT）大于指定时长的请求认定为慢调用请求
>
> 在指定时间内，如果请求数量超过设定的最小数量，慢调用比例大于设定的阈值，则触发熔断



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701233832865-1796001964.png" alt="image-20230701233817345" />

上图含义：

1. 响应时间为500ms的即为慢调用
2. 如果1000ms内有100次请求，且慢调用比例不低于0.05(即：100*0.05=5个慢调用)，则触发熔断(暂停该服务)
3. 熔断时间达到1s进入half-open状态，然后放行一次请求测试
   1. 成功则进入Closed状态关闭断路器
   2. 失败则进入Open状态打开断路器，继续像前面一样开始统计RT=500ms，1s内有100次请求……………..







#### 断路器熔断策略：异常比例 与 异常数

1. **异常比例**：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701234156895-1334365124.png" alt="image-20230701234145913" />

上图含义：在1s内，若是请求数量不低于100个，且异常比例不低于0.08(即：100*0.08=8个有异常)，则触发熔断，熔断时长达到1s就进入half-open状态



2. **异常数：**直接敲定有多少个异常数量就触发熔断

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230701234617472-1975223605.png" alt="image-20230701234559086" />





## 授权规则

> 授权规则可以对请求方来源做判断和控制



授权规则可以对调用方的来源做控制，有白名单和黑名单两种方式：

1. 白名单：来源（origin）在白名单内的调用者允许访问
2. 黑名单：来源（origin）在黑名单内的调用者不允许访问



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702163746947-2048469173.png" alt="image-20230702163745507" />

- 资源名：就是受保护的资源，例如 /order/{orderId}

- 流控应用：是来源者的名单
  - 如果是勾选白名单，则名单中的来源被许可访问
  - 如果是勾选黑名单，则名单中的来源被禁止访问



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702163847319-229518068.png" alt="image-20230702163846680" />



我们允许请求从gateway到order-service，不允许浏览器访问order-service，那么白名单中就要填写**网关的来源名称（origin）**

但是上图中怎么区分请求是从网关来的还是浏览器来的？在微服务中的想法是所有请求只能走网关，然后由网关路由到具体的服务，直接访问服务应该阻止才对，像下面直接跳过网关去访问服务，应该不行才对

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702185116027-9908147.png" alt="image-20230702185115299" />



要做到就需要使用授权规则了：

1. 网关授权拦截：针对于别人不知道内部服务接口的情况可以拦截成功
2. 服务授权控制/流控应用控制：针对“内鬼“ 或者 别人知道了内部服务接口，我们限定只能从哪里来的请求才能访问该服务，否则直接拒绝







### 流控应用怎么控制的？

下图中的名字怎么定义？

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702184508966-522606338.png" alt="image-20230702184506257" />



需要实现 RequestOriginParser 这个接口的 parseOrigin() 来获取请求的来源从而做到

```java
public interface RequestOriginParser {
    /**
     * 从请求request对象中获取origin，获取方式自定义
     */
    String parseOrigin(HttpServletRequest request);
}
```



**示例：**

1. 在需要进行保护的服务中编写请求来源解析逻辑

```java
package com.zixieqing.order.intercepter;

import com.alibaba.csp.sentinel.adapter.spring.webmvc.callback.RequestOriginParser;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

import javax.servlet.http.HttpServletRequest;

/**
 * 拦截请求，允许从什么地方来的请求才能访问此微服务
 *
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class RequestInterceptor implements RequestOriginParser {
    @Override
    public String parseOrigin(HttpServletRequest request) {
        // 获取请求中的请求头 可自定义
        String origin = request.getHeader("origin");
        if (StringUtils.isEmpty(origin))
            origin = "black";

        return origin;
    }
}
```

2. 在网关中根据2中 parseOrigin() 的逻辑添加相应的东西

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702191131516-1120198898.png" alt="image-20230702191129751" />

3. 添加流控规则：不要在簇点链路中选择相应服务来配置授权，会有BUG

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702215010222-522341457.png" alt="image-20230702215009306" />

经过上面的操作之后，要进入服务就只能通过网关路由过来了，不是从网关过来的就无法访问服务









## 自定义异常

默认情况下，发生限流、降级、授权拦截时，都会抛出异常到调用方。异常结果都是flow limmiting（限流）。这样不够友好，无法得知是限流还是降级还是授权拦截



而如果要自定义异常时的返回结果，需要实现 BlockExceptionHandler 接口：

```java
public interface BlockExceptionHandler {
    /**
     * 处理请求被限流、降级、授权拦截时抛出的异常：BlockException
     *
     * @param e 被sentinel拦截时抛出的异常
     */
    void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception;
}
```



这里的BlockException包含多个不同的子类：

| **异常**             | **说明**           |
| -------------------- | ------------------ |
| FlowException        | 限流异常           |
| ParamFlowException   | 热点参数限流的异常 |
| DegradeException     | 降级异常           |
| AuthorityException   | 授权规则异常       |
| SystemBlockException | 系统规则异常       |





**示例：**

1. 在需要的服务中实现 BlockExceptionHandler 接口

```java
package com.zixieqing.order.exception;

import com.alibaba.csp.sentinel.adapter.spring.webmvc.callback.BlockExceptionHandler;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.alibaba.csp.sentinel.slots.block.authority.AuthorityException;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeException;
import com.alibaba.csp.sentinel.slots.block.flow.FlowException;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowException;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * 自定义sentinel的各种异常处理
 *
 * <p>@author       : ZiXieqing</p>
 */

@Component
public class SentinelExceptionHandler implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        String msg = "未知异常";
        int status = 429;

        if (e instanceof FlowException) {
            msg = "请求被限流了";
        } else if (e instanceof ParamFlowException) {
            msg = "请求被热点参数限流";
        } else if (e instanceof DegradeException) {
            msg = "请求被降级了";
        } else if (e instanceof AuthorityException) {
            msg = "没有权限访问";
            status = 401;
        }

        response.setContentType("application/json;charset=utf-8");
        response.setStatus(status);
        response.getWriter().println("{\"msg\": " + msg + ", \"status\": " + status + "}");
    }
}
```

2. 重启服务，不同异常就会出现不同结果了











## 规则持久化

在默认情况下，sentinel的所有规则都是内存存储，重启后所有规则都会丢失。在生产环境下，我们必须确保这些规则的持久化，避免丢失



规则是否能持久化，取决于规则管理模式，sentinel支持三种规则管理模式：

1. 原始模式：Sentinel的默认模式，将规则保存在内存，重启服务会丢失
2. pull模式
3. push模式



### pull模式

> pull模式：控制台将配置的规则推送到Sentinel客户端，而客户端会将配置规则保存在本地文件或数据库中。以后会定时去本地文件或数据库中查询，更新本地规则
>
> 缺点：服务之间的规则更新不及时。因为是定时去读取，在时间还未到时，可能规则发生了变化



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702220035218-1832066945.png" alt="image-20230702220034454" />



### push模式

> push模式：控制台将配置规则推送到远程配置中心(如Nacos)。Sentinel客户端监听Nacos，获取配置变更的推送消息，完成本地配置更新



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702220314078-2136869.png" alt="image-20230702220313630" />







### 使用push模式实现规则持久化

在想要进行规则持久化的服务中引入如下依赖：

```xml
<!--sentinel规则持久化到Nacos的依赖-->
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

配置此服务的YAML文件，内容如下：

```yaml
spring:
  cloud:
    sentinel:
      datasource:
        flow: # 流控规则持久化
          nacos:
            server-addr: localhost:8848 # nacos地址
            dataId: orderservice-flow-rules
            groupId: SENTINEL_GROUP
            rule-type: flow # 还可以是：degrade 降级、authority 授权、param-flow 热点参数限流
#        degrade:  # 降级规则持久化
#          nacos:
#            server-addr: localhost:8848 # nacos地址
#            dataId: orderservice-degrade-rules
#            groupId: SENTINEL_GROUP
#            rule-type: degrade
#        authority:  # 授权规则持久化
#          nacos:
#            server-addr: localhost:8848 # nacos地址
#            dataId: orderservice-authority-rules
#            groupId: SENTINEL_GROUP
#            rule-type: authority
#        param-flow: # 热电参数限流持久化
#          nacos:
#            server-addr: localhost:8848 # nacos地址
#            dataId: orderservice-param-flow-rules
#            groupId: SENTINEL_GROUP
#            rule-type: param-flow
```





#### 修改sentinel的源代码

因为阿里的sentinel默认采用的是将规则内容存到内存中的，因此需要改源码



1. 使用git克隆sentinel的源码，之后IDEA等工具打开

```shell
git clone https://github.com/alibaba/Sentinel.git
```

2. 修改nacos依赖。在sentinel-dashboard模块的pom文件中，nacos的依赖默认的scope是test，那它只能在测试时使用，所以要去除 scope 标签

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```

3. 添加nacos支持。在sentinel-dashboard的test包下，已经编写了对nacos的支持，我们需要将其拷贝到src/main/java/com/alibaba/csp/sentinel/dashboard/rule 下

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230702233651740-1927616371.png" alt="image-20230702233650568" />

4. 修改nacos地址，让其读取application.properties中的配置

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230703000723173-852016594.png" alt="image-20230703000721756" />

5. 在sentinel-dashboard的application.properties中添加nacos地址配置

```properties
nacos.addr=127.0.0.1:8848	# ip和port改为自己想要的即可
```

6. 配置nacos数据源

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230703003437293-2039227035.png" alt="image-20230703003435769" />

7. 修改前端

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230703003936123-1555282183.png" alt="image-20230703003934857" />

8. 重现编译打包Sentinel-Dashboard模块

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230703004156789-343598019.png" alt="image-20230703004155921" />

9. 重现启动sentinel即可

```shell
java -jar -Dnacos.addr=127.0.0.1:8848 sentinel-dashboard.jar
```











## 补充：Sentinel基础知识

Sentinel实现限流、隔离、降级、熔断等功能，本质要做的就是两件事情：

- 统计数据：统计某个资源的访问数据（QPS、RT等信息）
- 规则判断：判断限流规则、隔离规则、降级规则、熔断规则是否满足



这里的**资源**就是希望被Sentinel保护的业务，例如项目中定义的controller方法就是默认被Sentinel保护的资源





### ProcessorSlotChain

实现上述功能的核心骨架是一个叫做ProcessorSlotChain的类。这个类基于责任链模式来设计，将不同的功能（限流、降级、系统保护）封装为一个个的Slot，请求进入后逐个执行即可

下图在官网有：https://github.com/alibaba/Sentinel/wiki

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723143337398-1360870863.png" alt="image-20210925092845529" />



责任链中的Slot也分为两大类：

- 统计数据构建部分（statistic）
  - NodeSelectorSlot：负责构建簇点链路中的节点（DefaultNode），将这些节点形成链路树
  - ClusterBuilderSlot：负责构建某个资源的ClusterNode，ClusterNode可以保存资源的运行信息（响应时间、QPS、block 数目、线程数、异常数等）以及来源信息（origin名称）
  - StatisticSlot：负责统计实时调用数据，包括运行信息、来源信息等
- 规则判断部分（rule checking）
  - AuthoritySlot：负责授权规则（来源控制）
  - SystemSlot：负责系统保护规则
  - ParamFlowSlot：负责热点参数限流规则
  - FlowSlot：负责限流规则
  - DegradeSlot：负责降级规则







### Node

Sentinel中的簇点链路是由一个个的Node组成的，Node是一个接口，包括下面的实现：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723143500701-1355263115.png" alt="image-20210925103029924" />



所有的节点都可以记录对资源的访问统计数据，所以都是StatisticNode的子类

按照作用分为两类Node：

- DefaultNode：代表链路树中的每一个资源，一个资源出现在不同链路中时，会创建不同的DefaultNode节点。而树的入口节点叫EntranceNode，是一种特殊的DefaultNode
- ClusterNode：代表资源，一个资源不管出现在多少链路中，只会有一个ClusterNode。记录的是当前资源被访问的所有统计数据之和。



DefaultNode记录的是资源在当前链路中的访问数据，用来实现基于链路模式的限流规则。ClusterNode记录的是资源在所有链路中的访问数据，实现默认模式、关联模式的限流规则。



例如：我们在一个SpringMVC项目中，有两个业务：

- 业务1：controller中的资源`/order/query`访问了service中的资源`/goods`
- 业务2：controller中的资源`/order/save`访问了service中的资源`/goods`

创建的链路图如下：

![image-20210925104726158](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723143500741-1770222629.png)







### Entry

默认情况下，Sentinel会将controller中的方法作为被保护资源，那么问题来了，我们该如何将自己的一段代码标记为一个Sentinel的资源呢？前面是用了 @SentinelResoutce 注解来实现的，那么这个注解的原理是什么？要搞清这玩意儿，那就得先来了解Entry这个吊毛玩意儿了

Sentinel中的资源用Entry来表示。声明Entry的API示例：

```java
// 资源名可使用任意有业务语义的字符串，比如方法名、接口名或其它可唯一标识的字符串。
try (Entry entry = SphU.entry("resourceName")) {
  // 被保护的业务逻辑
  // do something here...
} catch (BlockException ex) {
  // 资源访问阻止，被限流或被降级
  // 在此处进行相应的处理操作
}
```





#### 原生方式自定义资源

1. 在需要自定义资源的服务中引入依赖

```xml
<!--sentinel-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

2. 配置Sentinel

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8089
```

3. 对自定义资源的地方进行逻辑编写

````java
public Order queryOrderById(Long orderId) {
    // 创建Entry，标记资源，资源名为resource1
    try (Entry entry = SphU.entry("resource1")) {
        // 1.查询订单，这里是假数据
        Order order = Order.build(101L, 4999L, "小米 MIX4", 1, 1L, null);
        // 2.查询用户，基于Feign的远程调用
        User user = userClient.findById(order.getUserId());
        // 3.设置
        order.setUser(user);
        // 4.返回
        return order;
    }catch (BlockException e){
        log.error("被限流或降级", e);
        return null;
    }
}
````

打开sentinel控制台，查看簇点链路：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723144105513-1678389171.png" alt="image-20210925113122759" />





#### @SentinelResoutce 注解标记资源

通过给方法添加@SentinelResource注解的形式来标记资源：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723144900547-367632476.png" alt="image-20210925141507603" />



这是怎么实现的？

Sentinel依赖中有自动装配相关的东西，spring.factories声明需要就是自动装配的配置类，内容如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723145044650-1462953243.png" alt="image-20210925115740281" />



我们来看下`SentinelAutoConfiguration`这个类：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723145132619-1405484885.png" alt="image-20210925141553785" />



可以看到，在这里声明了一个Bean，`SentinelResourceAspect`：

```java
/**
 * Aspect for methods with {@link SentinelResource} annotation.
 *
 * @author Eric Zhao
 */
@Aspect
public class SentinelResourceAspect extends AbstractSentinelAspectSupport {
	// 切点是添加了 @SentinelResource 注解的类
    @Pointcut("@annotation(com.alibaba.csp.sentinel.annotation.SentinelResource)")
    public void sentinelResourceAnnotationPointcut() {
    }
	
    // 环绕增强
    @Around("sentinelResourceAnnotationPointcut()")
    public Object invokeResourceWithSentinel(ProceedingJoinPoint pjp) throws Throwable {
        // 获取受保护的方法
        Method originMethod = resolveMethod(pjp);
		// 获取 @SentinelResource 注解
        SentinelResource annotation = originMethod.getAnnotation(SentinelResource.class);
        if (annotation == null) {
            // Should not go through here.
            throw new IllegalStateException("Wrong state for SentinelResource annotation");
        }
        // 获取注解上的资源名称
        String resourceName = getResourceName(annotation.value(), originMethod);
        EntryType entryType = annotation.entryType();
        int resourceType = annotation.resourceType();
        Entry entry = null;
        try {
            // 创建资源 Entry
            entry = SphU.entry(resourceName, resourceType, entryType, pjp.getArgs());
            // 执行受保护的方法
            Object result = pjp.proceed();
            return result;
        } catch (BlockException ex) {
            return handleBlockException(pjp, annotation, ex);
        } catch (Throwable ex) {
            Class<? extends Throwable>[] exceptionsToIgnore = annotation.exceptionsToIgnore();
            // The ignore list will be checked first.
            if (exceptionsToIgnore.length > 0 && exceptionBelongsTo(ex, exceptionsToIgnore)) {
                throw ex;
            }
            if (exceptionBelongsTo(ex, annotation.exceptionsToTrace())) {
                traceException(ex);
                return handleFallback(pjp, annotation, ex);
            }

            // No fallback function can handle the exception, so throw it out.
            throw ex;
        } finally {
            if (entry != null) {
                entry.exit(1, pjp.getArgs());
            }
        }
    }
}

```



简单来说，@SentinelResource注解就是一个标记，而Sentinel基于AOP思想，对被标记的方法做环绕增强，完成资源（`Entry`）的创建。







### Context

上一节，我们发现簇点链路中除了controller方法、service方法两个资源外，还多了一个默认的入口节点：

sentinel_spring_web_context，是一个EntranceNode类型的节点

这个节点是在初始化Context的时候由Sentinel帮我们创建的



#### 什么是Context？

1. Context 代表调用链路上下文，贯穿一次调用链路中的所有资源（ `Entry`），基于ThreadLocal
2. Context 维持着入口节点（`entranceNode`）、本次调用链路的 curNode（当前资源节点）、调用来源（`origin`）等信息
3. 后续的Slot都可以通过Context拿到DefaultNode或者ClusterNode，从而获取统计数据，完成规则判断
4. Context初始化的过程中，会创建EntranceNode，contextName就是EntranceNode的名称



对应的API如下：

```java
// 创建context，包含两个参数：context名称、 来源名称
ContextUtil.enter("contextName", "originName");
```







#### Context的初始化

Context又是在何时完成初始化的？



进入SentinelWebAutoConfiguration这个类：可以直接搜，也可以去Sentinel依赖的Spring.factories中找

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723145605631-1703952710.png" alt="image-20210925115824345" />



WebMvcConfigurer是SpringMVC自定义配置用到的类，可以配置HandlerInterceptor

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723145645099-1637729768.png" alt="image-20210925115946064" />



`SentinelWebInterceptor`的声明如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723145729062-533815876.png" alt="image-20210925120119030" />

发现继承了`AbstractSentinelInterceptor`这个类。

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723145729524-2075570147.png" alt="image-20210925120221883" />







#### AbstractSentinelInterceptor

`HandlerInterceptor`拦截器会拦截一切进入controller的方法，执行`preHandle`前置拦截方法，而Context的初始化就是在这里完成的。

我们来看看这个类的`preHandle`实现：

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {
    try {
        // 获取资源名称，一般是controller方法的 @RequestMapping 路径，例如/order/{orderId}
        String resourceName = getResourceName(request);
        if (StringUtil.isEmpty(resourceName)) {
            return true;
        }
        // 从request中获取请求来源，将来做 授权规则 判断时会用
        String origin = parseOrigin(request);
        
        // 获取 contextName，默认是sentinel_spring_web_context
        String contextName = getContextName(request);
        // 创建 Context
        ContextUtil.enter(contextName, origin);
        // 创建资源，名称就是当前请求的controller方法的映射路径
        Entry entry = SphU.entry(resourceName, ResourceTypeConstants.COMMON_WEB, EntryType.IN);
        request.setAttribute(baseWebMvcConfig.getRequestAttributeName(), entry);
        return true;
    } catch (BlockException e) {
        try {
            handleBlockException(request, response, e);
        } finally {
            ContextUtil.exit();
        }
        return false;
    }
}
```





#### ContextUtil

创建Context的方法就是` ContextUtil.enter(contextName, origin);`

进入该方法：

```java
public static Context enter(String name, String origin) {
    if (Constants.CONTEXT_DEFAULT_NAME.equals(name)) {
        throw new ContextNameDefineException(
            "The " + Constants.CONTEXT_DEFAULT_NAME + " can't be permit to defined!");
    }
    return trueEnter(name, origin);
}
```

进入`trueEnter`方法：

```java
protected static Context trueEnter(String name, String origin) {
    // 尝试获取context
    Context context = contextHolder.get();
    // 判空
    if (context == null) {
        // 如果为空，开始初始化
        Map<String, DefaultNode> localCacheNameMap = contextNameNodeMap;
        // 尝试获取入口节点
        DefaultNode node = localCacheNameMap.get(name);
        if (node == null) {
            LOCK.lock();
            try {
                node = contextNameNodeMap.get(name);
                if (node == null) {
                    // 入口节点为空，初始化入口节点 EntranceNode
                    node = new EntranceNode(new StringResourceWrapper(name, EntryType.IN), null);
                    // 添加入口节点到 ROOT
                    Constants.ROOT.addChild(node);
                    // 将入口节点放入缓存
                    Map<String, DefaultNode> newMap = new HashMap<>(contextNameNodeMap.size() + 1);
                    newMap.putAll(contextNameNodeMap);
                    newMap.put(name, node);
                    contextNameNodeMap = newMap;
                }
            } finally {
                LOCK.unlock();
            }
        }
        // 创建Context，参数为：入口节点 和 contextName
        context = new Context(node, name);
        // 设置请求来源 origin
        context.setOrigin(origin);
        // 放入ThreadLocal
        contextHolder.set(context);
    }
    // 返回
    return context;
}
```







### 综合流程

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230723175127318-95614513.png" alt="image-20230723175127601" />





























# Seata 分布式事务

Seata是 2019 年 1 月份蚂蚁金服和阿里巴巴共同开源的分布式事务解决方案。致力于提供高性能和简单易用的分布式事务服务，为用户打造一站式的分布式解决方案。

官网地址：http://seata.io/



## CAP定理和Base理论

这两个在前面弄Nacos的时候已经说过了

**CAP定理** 这是分布式事务中的一个方法论

1. **C**	即：Consistency **数据一致性**。指的是：**用户访问分布式系统中的任意节点，得到的数据必须一致**
2. **A**	即：Availability **可用性**。指的是：**用户访问集群中的任意健康节点，必须能得到响应，而不是超时或拒绝**
3. **P**	即：Partition Tolerance **分区容错性**。指的是：**由于某种原因导致系统中任意信息的丢失或失败都不能影响系统的继续独立运作**



**注：** 分区容错性是必须满足的，数据一致性( C ）和 可用性（ A ）只满足其一即可，一般的搭配是如下的**（即：取舍策略）：**

1. CP			保证数据的准确性
2. AP			保证数据的及时性





既然CAP定理都整了，那就再加一个**Base理论**吧，这个理论是对CAP中C和A这两个矛盾点的调和和选择

1. **BA**	 即：Basically Available **基本可用性**。指的是：**在发生故障的时候，可以允许损失“非核心部分”的可用性，保证系统正常运行，即保证核心部分可用**
2. **S**	   即：Soft State **软状态**。指的是：**允许系统的数据存在中间状态，只要不影响整个系统的运行就行**
3. **E**	   即：Eventual Consistency **最终一致性**。指的是：**无论以何种方式写入数据库 / 显示出来，都要保证系统最终的数据是一致的**



分布式事务最大问题就是各个子事务的数据一致性问题，由CAP定理和Base理论进行综合之后，得出的分布式事务中的两个模式：

1. AP模式 ——–> 最终一致性：各个分支事务各自执行和提交，允许出现短暂的结果不一致，采用弥补措施将数据进行同步，从而恢复数据，达到最终数据一致
2. CP模式 ——–> 强一致性：各个分支事务执行后互相等待，同时提交或回滚，达成数据的强一致性







## Seata 的架构

Seata事务管理中有三个重要的角色：

1. **TC (Transaction Coordinator) -** **事务协调者：**维护全局和分支事务的状态，协调全局事务提交或回滚
2. **TM (Transaction Manager) -** **事务管理器：**定义全局事务的范围、开始全局事务、提交或回滚全局事务
3. **RM (Resource Manager) -** **资源管理器：**管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230705233840468-213285774.png" alt="image-20230705233836478" />



Seata基于上述架构提供了四种不同的分布式事务解决方案：

1. XA模式：强一致性分阶段事务模式，牺牲了一定的可用性。无业务侵入
2. AT模式：最终一致的分阶段事务模式，也是Seata的默认模式。无业务侵入
3. TCC模式：最终一致的分阶段事务模式。有业务侵入
4. SAGA模式：长事务模式。有业务侵入



无论哪种方案，都离不开TC，也就是事务的协调者







## 部署TC服务

1. 下载Seata-Server 并 解压。链接：https://github.com/seata/seata/releases 或 http://seata.io/zh-cn/blog/download.html
2. 修改 conf/registry.conf 文件

```properties
registry {
  # TC服务的注册中心	file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  # 配置Nacos注册中心信息
  nacos {
    application = "seata-tc-server"
    serverAddr = "127.0.0.1:8848"
    group = "DEFAULT_GROUP"
    namespace = ""
    cluster = "HZ"
    username = "nacos"
    password = "nacos"
  }
}

config {
  # 配置中心：读取TC服务端的配置文件的方式，这里是从nacos配置中心读取，这样如果tc是集群，可以共享配置
  #  file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    group = "DEFAULT_GROUP"
    username = "nacos"
    password = "nacos"
    dataId = "seataServer.properties"
  }
}
```

3. 在Nacos的控制台配置管理中配置2中的 seataServer.properties，内容如下：

```properties
# 数据存储方式，db代表数据库
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true&rewriteBatchedStatements=true
store.db.user=root
store.db.password=zixieqing072413
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
# 事务、日志等配置
server.recovery.committingRetryPeriod=1000
server.recovery.asynCommittingRetryPeriod=1000
server.recovery.rollbackingRetryPeriod=1000
server.recovery.timeoutRetryPeriod=1000
server.maxCommitRetryTimeout=-1
server.maxRollbackRetryTimeout=-1
server.rollbackRetryTimeoutUnlockEnable=false
server.undo.logSaveDays=7
server.undo.logDeletePeriod=86400000
# 客户端与服务端传输方式
transport.serialization=seata
transport.compressor=none
# 关闭metrics功能，提高性能
metrics.enabled=false
metrics.registryType=compact
metrics.exporterList=prometheus
metrics.exporterPrometheusPort=9898
```

4. 创建数据库表：tc服务在管理分布式事务时，需要记录事务相关数据到数据库中(3中配置了的)

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- 分支事务表
-- ----------------------------
DROP TABLE IF EXISTS `branch_table`;
CREATE TABLE `branch_table`  (
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `transaction_id` bigint(20) NULL DEFAULT NULL,
  `resource_group_id` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `resource_id` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `branch_type` varchar(8) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `status` tinyint(4) NULL DEFAULT NULL,
  `client_id` varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `application_data` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `gmt_create` datetime(6) NULL DEFAULT NULL,
  `gmt_modified` datetime(6) NULL DEFAULT NULL,
  PRIMARY KEY (`branch_id`) USING BTREE,
  INDEX `idx_xid`(`xid`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

-- ----------------------------
-- 全局事务表
-- ----------------------------
DROP TABLE IF EXISTS `global_table`;
CREATE TABLE `global_table`  (
  `xid` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `transaction_id` bigint(20) NULL DEFAULT NULL,
  `status` tinyint(4) NOT NULL,
  `application_id` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `transaction_service_group` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `transaction_name` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `timeout` int(11) NULL DEFAULT NULL,
  `begin_time` bigint(20) NULL DEFAULT NULL,
  `application_data` varchar(2000) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `gmt_create` datetime NULL DEFAULT NULL,
  `gmt_modified` datetime NULL DEFAULT NULL,
  PRIMARY KEY (`xid`) USING BTREE,
  INDEX `idx_gmt_modified_status`(`gmt_modified`, `status`) USING BTREE,
  INDEX `idx_transaction_id`(`transaction_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;

SET FOREIGN_KEY_CHECKS = 1;
```

5. 启动seat-server

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706001014284-1512828995.png" alt="image-20230706001012905" />

6. 验证是否成功

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706001336504-412458505.png" alt="image-20230706001335257" />









## Spring Cloud集成Seata

1. 依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <exclusions>
        <!--版本较低，1.3.0，因此排除-->
        <exclusion>
            <artifactId>seata-spring-boot-starter</artifactId>
            <groupId>io.seata</groupId>
        </exclusion>
    </exclusions>
</dependency>
<!--seata starter 采用1.4.2版本-->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>${seata.version}</version>
</dependency>
```

2. 给需要注册到TC的微服务的YAML文件配置如下内容：

```yaml
seata:
  registry: # TC服务注册中心的配置，微服务根据这些信息去注册中心获取tc服务地址	参考tc服务自己的registry.conf中的配置
    type: nacos
    nacos: # tc
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: DEFAULT_GROUP
      application: seata-tc-server # tc服务在nacos中的服务名称
  tx-service-group: seata-demo # 事务组，根据这个获取tc服务的cluster名称
  service:
    vgroup-mapping: # 事务组与TC服务cluster的映射关系
      seata-demo: HZ
```

经过如上操作就集成成功了









## 分布式事务之XA模式

XA 规范 是 X/Open 组织定义的分布式事务处理（DTP，Distributed Transaction Processing）标准，XA 规范 描述了全局的TM与局部的RM之间的接口，几乎所有主流的数据库都对 XA 规范 提供了支持。实现的原理都是基于两阶段提交



1. 正常情况：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706142942006-763984510.png" alt="image-20230706142940811" />

2. 异常情况：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706143016616-1907949140.png" alt="image-20230706143016059" />

一阶段：

1. 事务协调者通知每个事务参与者执行本地事务
2. 本地事务执行完成后报告事务执行状态给事务协调者，此时事务不提交，继续持有数据库锁

二阶段：事务协调者基于一阶段的报告来判断下一步操作

1. 如果一阶段都成功，则通知所有事务参与者，提交事务
2. 如果一阶段任意一个参与者失败，则通知所有事务参与者回滚事务





## Seata之XA模式 - 强一致性

**应用场景：** 并发量不大，但数据很重要的项目

Seata对原始的XA模式做了简单的封装和改造，以适应自己的事务模型

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706143212037-1460622248.png" alt="image-20230706143211225" />



RM一阶段的工作：

1. 注册分支事务到TC
2. 执行分支业务sql但不提交
3. 报告执行状态到TC

TC二阶段的工作：TC检测各分支事务执行状态

1. 如果都成功，通知所有RM提交事务
2. 如果有失败，通知所有RM回滚事务

RM二阶段的工作：

- 接收TC指令，提交或回滚事务



XA模式的优点：

1. 事务的强一致性，满足ACID原则
2. 常用数据库都支持，实现简单，并且没有代码侵入

XA模式的缺点：

1. 因为一阶段需要锁定数据库资源，等待二阶段结束才释放，性能较差
2. 依赖关系型数据库实现事务







### Java实现Seata的XA模式

1. 修改注册到TC的微服务的YAML配置

```yaml
seata:
  data-source-proxy-mode: XA	# 开启XA模式
```

2. 给发起全局事务的入口方法添加 `@GlobalTransactional` 注解。就是要开启事务的方法，如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706144215962-1798496890.png" alt="image-20230706144212402" />

3. 重启服务即可成功实现XA模式了







## Seata之AT模式 - 最终一致性

AT模式同样是分阶段提交的事务模型，不过却弥补了XA模型中资源锁定周期过长的缺陷

**应用场景：** 高并发互联网应用，允许数据出现短时不一致



基本架构图：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706144506965-443367842.png" alt="image-20230706144505339" />



RM阶段一的工作：

1. 注册分支事务
2. 记录undo-log（数据快照）
3. 执行业务sql**并提交**
4. 报告事务状态

阶段二提交时RM的工作：删除undo-log即可

阶段二回滚时RM的工作：根据undo-log恢复数据到更新前。恢复数据之后也会把undo-log中的数据删掉



流程图如下：

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706145424746-1612481270.png" alt="image-20230706145423923" />





AT模式与XA模式的区别是什么？

- XA模式一阶段**不提交事务**，锁定资源；AT模式一阶段**直接提交**，不锁定资源。
- XA模式依赖数据库机制实现回滚；AT模式利用数据快照实现数据回滚。
- XA模式强一致；AT模式最终一致







### AT模式的脏写问题

![AT模式脏写问题](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706150656483-1306782989.gif)



解决思路就是引入了全局锁的概念。在释放DB锁之前，先拿到全局锁。避免同一时刻有另外一个事务来操作当前数据，从而来做到写隔离

- **全局锁：** 由TC记录当前正在执行数据的事务，该事务持有全局锁，具备执行权

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706145945005-1394340454.png" alt="image-20230706145943940" />



但就算引入了全局锁，也还会有BUG，因为上面两个事务都是Seata管理，若事务1是Seata管理，而事务2是非Seata管理，同时这两个事务都在修改同一条数据，那么就还会造成脏写问题

![AT模式脏写问题](https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706151012307-544058691.gif)



为了防止这个问题，Seata在保存快照时实际上会记录2份快照，一份是修改之前的快照，一份是修改之后的快照

1. 在恢复快照数据时，会将更新后的快照值和当前数据库的实际值进行比对(类似CAS过程)
   
   如果数值不匹配则说明在此期间有另外的事务修改了数据，此时直接释放全局锁，事务1记录异常，发送告警信息让人工介入
   
   如果一致则恢复数据，释放全局锁即可



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230706152038191-864543402.png" alt="AT模式脏写解决方式" />





AT模式的优点：

1. 一阶段完成直接提交事务，释放数据库资源，性能比较好
2. 利用全局锁实现读写隔离
3. 没有代码侵入，框架自动完成回滚和提交

AT模式的缺点：

1. 两阶段之间属于软状态，属于最终一致
2. 框架的快照功能会影响性能，但比XA模式要好很多







### Java实现AT模式

AT模式中的快照生成、回滚等动作都是由框架自动完成，没有任何代码侵入

只不过，AT模式需要一个表来记录全局锁、另一张表来记录数据快照undo_log。其中：

- lock_table表：需要放在“TC服务关联”的数据库中。例如表结构如下：

```sql
DROP TABLE IF EXISTS `lock_table`;
CREATE TABLE `lock_table`  (
  `row_key` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `xid` varchar(96) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `transaction_id` bigint(20) NULL DEFAULT NULL,
  `branch_id` bigint(20) NOT NULL,
  `resource_id` varchar(256) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `table_name` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `pk` varchar(36) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `gmt_create` datetime NULL DEFAULT NULL,
  `gmt_modified` datetime NULL DEFAULT NULL,
  PRIMARY KEY (`row_key`) USING BTREE,
  INDEX `idx_branch_id`(`branch_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Compact;
```

- undo_log表 ：需要放在“微服务关联”的数据库中。例如表结构如下：

```sql
DROP TABLE IF EXISTS `undo_log`;
CREATE TABLE `undo_log`  (
  `branch_id` bigint(20) NOT NULL COMMENT 'branch transaction id',
  `xid` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'global transaction id',
  `context` varchar(128) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT 'undo_log context,such as serialization',
  `rollback_info` longblob NOT NULL COMMENT 'rollback info',
  `log_status` int(11) NOT NULL COMMENT '0:normal status,1:defense status',
  `log_created` datetime(6) NOT NULL COMMENT 'create datetime',
  `log_modified` datetime(6) NOT NULL COMMENT 'modify datetime',
  UNIQUE INDEX `ux_undo_log`(`xid`, `branch_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci COMMENT = 'AT transaction mode undo table' ROW_FORMAT = Compact;
```

然后修改注册到TC中的微服务的YAML配置，最后重启服务，模式就变为AT模式了

```yaml
seata:
  data-source-proxy-mode: AT # 默认就是AT
```







## Seata之TCC模式 - 最终一致性

**应用场景：** 高并发互联网应用，允许数据出现短时不一致，可通过对账程序或补录来保证最终一致性

TCC模式与AT模式非常相似，每阶段都是独立事务，不同的是TCC通过人工编码来实现数据恢复。需要实现三个方法：

1. **Try**：资源的检测和预留
2. **Confirm**：完成资源操作业务；要求 Try 成功 Confirm 一定要能成功
3. **Cancel**：预留资源释放，可以理解为try的反向操作。



举例说明三个方法：一个扣减用户余额的业务。假设账户A原来余额是100，需要余额扣减30元

<img src="https://img2023.cnblogs.com/blog/2421736/202308/2421736-20230805212324909-129384588.png" alt="image-20230805212324076" />







### TCC模式的架构

<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230707133410708-343026245.png" alt="image-20230707133410426" />



TCC模式的每个阶段是做什么的？

1. Try：资源检查和预留
2. Confirm：业务执行和提交
3. Cancel：预留资源的释放

TCC的优点是什么？

1. 一阶段完成直接提交事务，释放数据库资源，性能好
2. 相比AT模型，无需生成快照，无需使用全局锁，性能最强
3. 不依赖数据库事务，而是依赖补偿操作，可以用于非事务型数据库(如：Redis)

TCC的缺点是什么？

1. 有代码侵入，需要人为编写try、Confirm和Cancel接口，太麻烦
2. 软状态，事务是最终一致
3. **需要考虑Confirm和Cancel的失败情况，做好幂等处理**







### 空回滚和业务悬挂

> **空补偿 / 空回滚：** 未执行try(原服务)就执行了cancel(补偿服务)。即当某分支事务的try阶段**阻塞**时，可能导致全局事务超时而触发二阶段的cancel操作。在未执行try操作时先执行了cancel操作，这时cancel不能做回滚，就是“空回滚”
>
> **因此：执行cancel操作时，应当判断try是否已经执行，如果尚未执行，则应该空回滚**
>
> 
>
> **业务悬挂：** 已经空回滚的业务，之前阻塞的try恢复了，然后继续执行try，之后就永不可能执行confirm或cancel，从而变成“业务悬挂”
>
> **因此：执行try操作时，应当判断cancel是否已经执行过了，如果已经执行，应当阻止空回滚后的try操作，避免悬挂**



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230707133832213-1995370184.png" alt="image-20230707133831809" />



### Java实现TCC模式示例

Try业务：

- 根据xid查询account_freeze ，如果已经存在则证明Cancel已经执行，拒绝执行try业务
- 记录冻结金额和事务状态到account_freeze表
- 扣减account表可用金额

Confirm业务

- 需判断此方法的幂等性问题
- 根据xid删除account_freeze表的冻结记录

Cancel业务

- 需判断此方法的幂等性问题
- 根据xid查询account_freeze，如果为null则说明try还没做，需要空回滚
- 修改account_freeze表，冻结金额为0，state为2
- 修改account表，恢复可用金额



1. 在业务管理的库中建表：是为了实现空回滚、防止业务悬挂，以及幂等性要求。所以在数据库记录冻结金额的同时，记录当前事务id和执行状态

```sql
CREATE TABLE `account_freeze_tbl` (
  `xid` varchar(128) NOT NULL COMMENT '全局事务id',
  `user_id` varchar(255) DEFAULT NULL COMMENT '用户id',
  `freeze_money` int(11) unsigned DEFAULT '0' COMMENT '冻结金额',
  `state` int(1) DEFAULT NULL COMMENT '事务状态，0:try，1:confirm，2:cancel',
  PRIMARY KEY (`xid`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 ROW_FORMAT=COMPACT
```

2. 业务接口定义try+confirm+cancel三个方法

```java
package com.zixieqing.account.service;

import io.seata.rm.tcc.api.BusinessActionContext;
import io.seata.rm.tcc.api.BusinessActionContextParameter;
import io.seata.rm.tcc.api.LocalTCC;
import io.seata.rm.tcc.api.TwoPhaseBusinessAction;
import org.springframework.stereotype.Service;

/**
 * Seata之TCC模式实现业务的account接口
 *
 * "@LocalTCC"    SpringCloud + Feign，Feign的调用基于http
 *                此注解所在的接口需要实现TCC的两阶段提交对应方法才行
 *
 * <p>@author       : ZiXieqing</p>
 */

@Service
@LocalTCC
public interface AccountTccService {
    /**
     * 扣款
     *
     * Try逻辑	资源检查和预留，同时需要判断Cancel是否已经执行，是则拒绝执行本次业务
     *
     * "@TwoPhaseBusinessAction" 中
     * 								name属性				 要与当前方法名一致，用于指定Try逻辑对应的方法
     * 								commitMethod属性值		就是confirm逻辑的方法
     * 								rollbackMethod属性值	就是cancel逻辑的方法
     *
     * "@BusinessActionContextParameter" 将指定的参数传递给confirm和cancel
     *
     * @param userId 用户id
     * @param money 要扣的钱
     */
    @TwoPhaseBusinessAction(
            name = "deduct",
            commitMethod = "confirm",
            rollbackMethod = "cancel"
    )
    void deduct(@BusinessActionContextParameter(paramName = "userId") String userId,
                @BusinessActionContextParameter(paramName = "money") int money);

    /**
     * 二阶段confirm确认方法	业务执行和提交		另外需考虑幂等性问题
     * 						 方法名可以另命名，但需保证与commitMethod一致
     *
     * @param context 上下文，可以传递try方法的参数
     * @return boolean 执行是否成功
     */
    boolean confirm(BusinessActionContext context);

    /**
     * 二阶段回滚方法	预留资源释放	另外需考虑幂等性问题	需要判断try是否已经执行，否就需要空回滚
     * 				 方法名须保证与rollbackMethod一致
     *
     * @param context 上下文，可以传递try方法的参数
     * @return boolean 执行是否成功
     */
    boolean cancel(BusinessActionContext context);
}
```

3. 实现类逻辑编写

```java
package com.zixieqing.account.service.impl;

import com.zixieqing.account.entity.AccountFreeze;
import com.zixieqing.account.mapper.AccountFreezeMapper;
import com.zixieqing.account.mapper.AccountMapper;
import com.zixieqing.account.service.AccountTccService;
import io.seata.core.context.RootContext;
import io.seata.rm.tcc.api.BusinessActionContext;
import org.springframework.beans.factory.annotation.Autowired;

/**
 * 扣款业务
 *
 * <p>@author       : ZiXieqing</p>
 */


public class AccountTccServiceImpl implements AccountTccService {
    @Autowired
    private AccountMapper accountMapper;
    @Autowired
    private AccountFreezeMapper accountFreezeMapper;

    /**
     * 扣款
     *
     * Try逻辑	资源检查和预留，同时需要判断Cancel是否已经执行，是则拒绝执行本次业务
     *
     * "@TwoPhaseBusinessAction" 中
     * 		name属性要				与当前方法名一致，用于指定Try逻辑对应的方法
     * 		commitMethod属性值		就是confirm逻辑的方法
     * 		rollbackMethod属性值	就是cancel逻辑的方法
     *
     * "@BusinessActionContextParameter" 将指定的参数传递给confirm和cancel
     *
     * @param userId 用户id
     * @param money  要扣的钱
     */
    @Override
    public void deduct(String userId, int money) {
        // 获取事务ID，RootContext 是seata中的
        String xid = RootContext.getXID();
        AccountFreeze accountFreeze = accountFreezeMapper.selectById(xid);
        // 业务悬挂处理：判断cancel是否已经执行，若执行过则free表中肯定有数据
        if (accountFreeze != null) {
            // CANCEL执行过，拒绝业务
            return:
        }
        // 进行扣款
        accountMapper.deduct(userId, money);
        // 记录本次状态
        AccountFreeze freeze = new AccountFreeze();
        freeze.setXid(xid)
            .setUserId(userId)
            .setFreezeMoney(money)
            .setState(AccountFreeze.State.TRY);
            
        accountFreezeMapper.insert(freeze);
    }

    /**
     * 二阶段confirm确认方法	业务执行和提交		另外需考虑幂等性问题
     * 方法名可以另命名，但需保证与commitMethod一致
     *
     * @param context 上下文，可以传递try方法的参数
     * @return boolean 执行是否成功
     */
    @Override
    public boolean confirm(BusinessActionContext context) {
        // 删掉freeze表中的记录即可  delete方法本身就具有幂等性
        return accountFreezeMapper.deleteById(context.getXid()) == 1;
    }

    /**
     * 二阶段回滚方法	预留资源释放	另外需考虑幂等性问题	需要判断try是否已经执行，否 就需要空回滚
     * 方法名须保证与rollbackMethod一致
     *
     * @param context 上下文，可以传递try方法的参数
     * @return boolean 执行是否成功
     */
    @Override
    public boolean cancel(BusinessActionContext context) {
        // 空回滚处理：判断try是否已经执行
        AccountFreeze freeze = accountFreezeMapper.selectById(context.getXid());
        // 若为null，则try肯定没执行
        if (freeze == null) {
            // 需要进行空回滚
            freeze = new AccountFreeze();
            freeze.setXid(context.getXid())
                    // getActionContext("userId") 的key就是@BusinessActionContextParameter(paramName = "userId")的pramName值
                    .setUserId(context.getActionContext("userId").toString())
                    .setFreezeMoney(0)
                    .setState(AccountFreeze.State.CANCEL);
            
            return accountFreezeMapper.insert(freeze) == 1;
        }

        // 幂等性处理
        if (freeze.getState() == AccountFreeze.State.CANCEL) {
            // 说明已经执行过一次cancel了，直接拒绝执行本次业务
            return true;
        }

        // 不为null，则回滚数据
        accountMapper.refund(freeze.getUserId(), freeze.getFreezeMoney());
        // 将冻结金额归0，并修改本次状态
        freeze.setFreezeMoney(0)
                .setState(AccountFreeze.State.CANCEL);
        
        return accountFreezeMapper.updateById(freeze) == 1;
    }
}
```

最后正常使用service调用使用3中的实现类即可









## Seata之Saga模式 - 最终一致性

Saga 模式是 Seata 的长事务解决方案，由蚂蚁金服主要贡献

其理论基础是Hector & Kenneth  在1987年发表的论文[Sagas](https://microservices.io/patterns/data/saga.html)

Seata官网对于Saga的指南：https://seata.io/zh-cn/docs/user/saga.html

**适用场景：** 

1. 业务流程长、业务流程多且需要保证事务最终一致性的业务系统
2. 银行业金融机构
3. 需要与第三方交互，如：调用支付宝支付接口->出库失败->调用支付宝退款接口



优点：

1. 事务参与者**可以基于事件驱动实现异步调用**，吞吐高
2. 一阶段直接提交事务，无锁，性能好
3. 不用编写TCC中的三个阶段，实现简单

缺点：

1. 软状态持续时间不确定，时效性差
2. 由于一阶段已经提交本地数据库事务，且没有进行“预留”动作，所以不能保证隔离性，同时也没有锁，所以会有脏写



Saga模式是SEATA提供的长事务解决方案。也分为两个阶段：

1. 一阶段：直接提交本地事务
2. 二阶段：成功则什么都不做；失败则通过编写补偿业务来回滚



<img src="https://img2023.cnblogs.com/blog/2421736/202307/2421736-20230708123823966-1360469057.png" alt="image-20230708123817123" />

Saga 是一种补偿协议，Saga 正向服务与补偿服务也需要业务开发者实现。在 Saga 模式下，分布式事务内有多个参与者，每一个参与者都是一个冲正补偿服务，需要用户根据业务场景实现其正向操作和逆向回滚操作。

分布式事务执行过程中，依次执行各参与者的正向操作，如果所有正向操作均执行成功，那么分布式事务提交；如果任何一个正向操作执行失败，那么分布式事务会退回去执行前面各参与者的逆向回滚操作，回滚已提交的参与者，使分布式事务回到初始状态









## Seata四种模式对比

|              | **XA**                         | **AT**                                       | **TCC**                                            | **SAGA**                                                     |
| ------------ | ------------------------------ | -------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------ |
| **一致性**   | 强一致                         | 弱一致                                       | 弱一致                                             | 最终一致                                                     |
| **隔离性**   | 完全隔离                       | 基于全局锁隔离                               | 基于资源预留隔离                                   | 无隔离                                                       |
| **代码侵入** | 无                             | 无                                           | 有，要编写三个接口                                 | 有，要编写状态机和补偿业务                                   |
| **性能**     | 差                             | 好                                           | 非常好                                             | 非常好                                                       |
| **场景**     | 对一致性、隔离性有高要求的业务 | 基于关系型数据库的大多数分布式事务场景都可以 | 对性能要求较高的事务。有非关系型数据库要参与的事务 | 业务流程长、业务流程多参与者包含其它公司或遗留系统服务，无法提供 TCC 模式要求的三个接口 |























