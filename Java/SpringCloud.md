# 前言

玩SpringCloud之前最好懂SpringBoot，别搞撑死骆驼的事。SSM封装、加入东西就变为SpringBoot，SpringBoot再封装、加入东西就变为SpringCloud



# 架构的演进

## 单体应用架构

**单体架构**：表示层、业务逻辑层和数据访问层即所有功能都在一个工程里，打成一个jar包、war包进行部署，例如：GitHub 是基于 Ruby on Rails 的单体架构，直到 2021 年，为了让超过一半的开发人员在单体代码库之外富有成效地开展工作，GitHub 以赋能为出发点开始了向微服务架构的迁移

下图服务器用Tomcat举例

![image-20230521164028933](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521164029004-1514387794.png)



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

![image-20230521164632822](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521164632922-534104745.png)



缓存能够将经常访问的页面或信息存起来，从而不让其去直接访问数据库，从而增大数据库压力，但是：这就会把压力变成单机Tomcat来承受了，因此缺点就是：此时单机的tomcat又不足以支撑起高并发的请求





## 垂直应用架构：引入Nginx

![image-20230521170148649](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521170148624-1735985478.png)



搭配N个tomcat，从而对请求"均衡处理"，如：如果Nginx可以处理10000条请求，假设一个 tomcat可以处理100个请求，那么：就需要100个tomcat从而实现每个tomcat处理100个请求(假设每个tomcat的性能都一样 )

缺点就是数据库不足以支撑压力

后面就是将数据库做读写分离

![image-20230521170535184](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521170535267-191396252.png)



后面还有数据库大表拆小表、大业务拆为小业务、复用功能抽离..............





## 面向服务架构：SOA

SOA指的是Service-OrientedArchitecture，即面向服务架构

随着业务越来越多，代码越来越多，按照业务功能将本来一整块的系统拆分为各个不同的子系统分别提供不同的服务，服务之间会彼此调用，错综复杂

![image-20230521175435052](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521175435390-508194047.png)



而SOA的思想就是基于前面拆成不同的服务之后，继续再抽离一层，搞一个和事佬，即下图的“统一接口”

![image-20230521224344141](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521224344850-262357819.png)



这样不同服务之间调用就可以通过统一接口进行调用了，如：用户服务需要调用订单服务，那么用户服务去找统一接口，然后由统一接口去调用订单服务，从而将订单服务中需要的结果通过统一接口的http+json或其他两种格式返回给用户服务，这样订单服务就是服务提供者，用户服务就是服务消费者，而统一接口就相当于是服务的注册与发现

- 注意：上面这段话很重要，和后面要玩的微服务框架SpringCloud技术栈有关

学过设计模式的话，上面这种不就类似行为型设计模式的“中介者模式”吗

上面这种若是反应不过来，那拆回单体架构就懂了

![image-20230521230448349](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521230448878-888627647.png)





## 微服务架构

微服务架构是分布式架构的具体实现方式，和Spring的IOC控制反转和DI依赖注入的关系一样，一种是理论，一种是具体实现方案

微服务架构和前面的SOA架构是孪生兄弟，即：微服务架构是在SOA架构的基础上，通过前人不断实践、不断踩坑、不断总结，添加了一些东西之后(如：链路追踪、配置管理、负债均衡............)，从而变出来的一种经过良好架构设计的**分布式架构方案**

而广泛应用的方案框架之一就是 [SpringCloud](https://spring.io/projects/spring-cloud)

其中常见的组件包括：

![image-20210713204155887](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521231654377-404634780.png)



另外，SpringCloud底层是依赖于SpringBoot的，并且有版本的兼容关系，如下：

![image-20210713205003790](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521231719947-1707829950.png)



因此。现在系统架构就变成了下面这样，当然不是一定是下面这样架构设计，还得看看架构师，看领导

![image-20230521232051716](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521232052531-1243047828.png)



因此，微服务技术知识如下

![image-20230521232536647](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230521232537360-774880611.png)









# Eureka注册中心

SpringCloud中文官网：https://www.springcloud.cc/spring-cloud-greenwich.html#netflix-ribbon-starter

SpringCloud英文网：https://spring.io/projects/spring-cloud

## Eureka是什么？

Eureka是Netflix开发的服务发现框架，本身是一个基于REST的服务，主要用于定位运行在AWS域中的中间层服务，以达到负载均衡和中间层服务故障转移的目的。

SpringCloud将它集成在其子项目spring-cloud-netflix中，以实现SpringCloud的服务发现功能

偷张图更直观地了解一下：

![image-20210713220104956](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230522222600452-117614713.png)



如上图所示，服务提供方会将自己注册到EurekaServer中，这样EurekaServer就会存储各种服务信息，而服务消费方想要调用服务提供方的服务时，直接找EurekaServer拉取服务列表，然后根据特定地算法(轮询、随机......)，选择一个服务从而进行远程调用

- 服务提供方：一次业务中，被其它微服务调用的服务。（提供接口给其它微服务）
- 服务消费方：一次业务中，调用其它微服务的服务。（调用其它微服务提供的接口）

服务提供者与服务消费者的角色并不是绝对的，而是相对于业务而言

如果服务A调用了服务B，而服务B又调用了服务C，服务B的角色是什么？

- 对于A调用B的业务而言：A是服务消费者，B是服务提供者
- 对于B调用C的业务而言：B是服务消费者，C是服务提供者

因此，服务B既可以是服务提供者，也可以是服务消费者





## Eureka的自我保护机制

![image-20210713220104956](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230522223650571-1467351950.png)



这张图中EurekaServer和服务提供方有一个心跳检测机制，这是EurekaServer为了确定这些服务是否还在正常工作，所以进行的心跳检测

eureka-client启动时， 会开启一个心跳任务，向Eureka Server发送心跳，默认周期为30秒/次，如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除(默认90秒)

eureka-server维护了每个实例的最后一次心跳时间，客户端发送心跳包过来后，会更新这个心跳时间

eureka-server启动时，开启了一个定时任务，该任务每60s/次，检查每个实例的最后一次心跳时间是否超过90s，如果超过则认为过期，需要剔除



但是EurekaClient也会因为网络等原因导致没有及时向EurekaServer发送心跳，因此EurekaServer为了保证误删服务就会有一个“自我保护机制”，俗称“好死不如赖活着”

如果在短时间内EurekaServer丢失过多客户端时 (可能断网了，**低于85%的客户端节点都没有正常的心跳** )，那么Eureka Server就认为客户端与注册中心出现了网络故障，Eureka Server自动进入自我保护状态 。Eureka的这样设计更加精准地控制是网络通信延迟，而不是服务挂掉了，一旦进入自我保护模式，那么 EurekaServer就会保留这个节点的熟悉，不会删除，直到这个节点恢复正常心跳 

- 85% 这个阈值，可以通过如下配置来设置：

```yaml
eureka:
  server:
    renewal-percent-threshold: 0.85
```

这里存在一个个问题，这个85%是超过谁呢？这里有一个预期的续约数量，计算公式如下:

```txt
自我保护阀值 = 服务总数 * 每分钟续约数(60S/客户端续约间隔) * 自我保护续约百分比阀值因子
```





在自我保护模式中，EurekaServer会保留注册表中的信息，不再注销任何服务信息，当它收到正常心跳时，才会退出自我保护模式，也就是：**宁可保留错误的服务注册信息，也不会盲目注销任何可能健康的服务实例，即：好死不如赖活着** 



因此Eureka进入自我保护状态后，会出现以下几种情况:

- Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其他节点上，保证当前节点依然可用。Eureka自我保护机制，Eureka的自我保护机制可以通过如下的方式开启或关闭

```yaml
eureka:
  server:
    # 开启Eureka自我保护机制，默认为true
    enable-self-preservation: true
```

- Eureka Server不再从注册列表中移除因为长时间没有收到心跳而应该剔除的过期服务，如果在保护期内如果服务刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等！







## 使用Eureka

实现如下的逻辑：

![image-20230523105025549](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523105026659-571765101.png)



### 搭建Eureka Server

根据前面Eureka的自我保护机制中提到的结构图，现在来搭建Eureka

自行单独创建一个Maven项目，导入依赖如下：

```xml
<!--EurekaServer-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```



在yml文件中填入如下内容：

```yaml
server:
  port: 10086
spring:
  application:
    name: EUREKA-SERVER
eureka:
  instance:
    # 此机Eureka的主机名，eureka集群服务器之间的区分
    hostname: 127.0.0.1
    # 最后一次心跳后，间隔多久认定微服务不可用，默认90
    lease-expiration-duration-in-seconds: 90
  client:
    # 不向自身注册。应用为单个注册中心设置为false，代表不向注册中心注册自己，默认true
    #registerWithEureka: false
    # 不从自身拉取注册信息。单个注册中心则不拉去自身信息，默认true
    # fetchRegistry: false
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
#    server:
#      # 开启Eureka自我保护机制，默认为true
#      enable-self-preservation: true
```

- **注：**在SpringCloud中配置文件yml有两种方式，一种是 `application.yml ` 另一种是 `bootstrap.yml `，区别去这里：https://www.cnblogs.com/sharpest/p/13678443.html





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





yml配置内容如下：

```yaml
server:
  port: 8081
spring:
  application:
    name: USER-SERVICE
eureka:
  client:
    # 将服务注册到哪个eureka server
    service-url:
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

上面一节中启动类里面有些人会看到是如下的方式：

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

在eureka client启动类中，为什么有些人会加 `@EnableEurekaClient` 注解，而有些人不会加上？

要弄这个问题，首先看yml中的配置，有些是在yml中做了一个操作：

```yaml
eureka:
  client:
    service-url:
      # 向那个eureka服务端进行服务注册
      defaultZone: http://localhost:10086/eureka
    # 开启eureka client功能，默认就是true，差不多等价于启动类中加 @EnableEurekaClient 注解
    enabled: true
```

既然上面配置默认值都是true，那还有必要在启动类中加入 `@EnableEurekaClient` 注解吗？

答案是根本不用加，加了也是多此一举(前提：yml配置中没有手动地把值改为false)，具体原因看源码：答案就在Eureka client对应的自动配置类 `EurekaClientAutoConfiguration 中`

![image-20230523140656713](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523140658133-980195700.png)

上图中这一行的意思是只有当application.yaml（或者环境变量，或者系统变量）里，`eureka.client.enabled`这个属性的值为`true`才会初始化这个类（如果手动赋值为false，就不会初始化这个类了）



另外再加上另一个原因，同样在 `EurekaClientAutoConfiguration` 类中还有一个 `eurekaAutoServiceRegistration()` 方法

![image-20230523141136544](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523141137735-1222105186.png)



在这里使用 `EurekaAutoServiceRegistration类+@Bean注解` 意思就是通过 `@Bean` 注解，装配一个 EurekaAutoServiceRegistration 对象作为Spring的bean，而我们从名字就可以看出来EurekaClient的注册就是 EurekaAutoServiceRegistration 对象所进行操作的。

同时，在这个方法上，也有这么一行 `@ConditionalOnProperty(value = "spring.cloud.service-registry.auto-registration.enabled", matchIfMissing = true)` 



综上所述：我们可以看出来，EurekaClient的注册和两个配置项有关的，一个是 `eureka.client.enabled` ，另一个是 `spring.cloud.service-registry.auto-registration.enabled` ，只不过这两个配置默认都是true。这两个配置无论哪个我们手动配置成false，我们的服务都无法进行注册，测试自行做





另外还有一个原因：上图中不是提到了 `EurekaAutoServiceRegistration类+@Bean注解` 吗，那去看一下

![image-20230523142606183](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523142607917-175927233.png)



可以看到 `EurekaAutoServiceRegistration `类实现了Spring的 `SmartLifecycle `接口，这个接口的作用是帮助一个类在作为Spring的Bean的时候，由Spring帮助我们自动进行一些和生命周期有关的工作，比如在初始化或者停止的时候进行一些操作。而我们最关心的 `注册(register)` 这个动作，就是在SmartLifecycle接口的 `start()` 方法实现里完成的



而上一步讲到，`EurekaAutoServiceRegistration `类在 `EurekaClientAutoConfiguration `类里恰好被配置成Spring的Bean，所以这里的 `start()` 方法是会自动被Spring调用的，我们不需要进行任何操作





##### 总结

当我们引用了EurekaClient的依赖后，并且  `eureka.client.enabled` 和 `spring.cloud.service-registry.auto-registration.enabled` 两个开关不手动置为false，Spring就会自动帮助我们执行 `EurekaAutoServiceRegistration` 类里的 `start()` 方法，而注册的动作就是在该方法里完成的



**所以，我们的EurekaClient工程，并不需要显式的在SpringBoot的启动类上标注 `@EnableEurekaClient` 注解**





















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



yml配置如下：

```yaml
server:
  port: 8080
spring:
  application:
    name: ORDER-SERVICE
eureka:
  client:
    service-url:
      # 向那个eureka服务端进行服务拉取
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

依次启动eureka-server、user-service、order-service，然后将user-service做一下模拟集群即可，将user-service弄为模拟集群操作方式如下：

![image-20230523113542449](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523113543880-720911336.png)





![image-20230523113728396](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523113729320-191314494.png)



再将复刻的use-service2也启动即可，启动之后点一下eureka-server的端口就可以在浏览器看到服务qingk

![image-20230523114005087](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523114005904-1151512200.png)



![image-20230523114153992](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523114155475-1176148165.png)



可以自行在服务提供方和服务消费方编写逻辑，去链接数据库，然后在服务消费方调用服务提供方的业务，最后访问自己controller中定义的路径和参数即可









# Ribbon负载均衡

## Ribbon是什么？

Ribbon是Netflix发布的开源项目，`Spring Cloud Ribbon`是基于`Netflix Ribbon`实现的一套客户端`负载均衡`的框架





## Ribbon属于哪种负载均衡？

**LB负载均衡(Load Balance)是什么？**

- 简单的说就是将用户的请求平摊的分配到多个服务上，从而达到系统的HA（高可用）
- 常见的负载均衡有软件Nginx，硬件 F5等





**什么情况下需要负载均衡？**

- 现在Java非常流行微服务，也就是所谓的面向服务开发，将一个项目拆分成了多个项目，其优点有很多，其中一个优点就是：将服务拆分成一个一个微服务后，我们很容易的来针对性的进行集群部署。例如订单模块用的人比较多，我就可以将这个模块多部署几台机器，来分担单个服务器的压力

- 这时候有个问题来了，前端页面请求的时候到底请求集群当中的哪一台？既然是降低单个服务器的压力，所以肯定全部机器都要利用起来，而不是说一台用着，其他空余着。这时候就需要用负载均衡了，像这种前端页面调用后端请求的，要做负载均衡的话，常用的就是Nginx





**Ribbon和Nginx负载均衡区别**

- 当后端服务是集群的情况下，前端页面调用后端请求，要做负载均衡的话，常用的就是Nginx
- Ribbon主要是在服务端内做负载均衡，举例：订单后端服务 要调用 支付后端服务，这属于后端之间的服务调用，压根根本不经过页面，而支付后端服务是集群，这时候订单服务就需要做负载均衡来调用支付服务，记住是订单服务做负载均衡 来调用 支付服务





**负载均衡分类**

-  **集中式LB**：即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx)，由该设施负责把访问请求通过某种策略转发至服务的提供方
- **进程内LB**：将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器





**Ribbon负载均衡**

- Ribbon就属于进程内LB，它只是一个类库，集成于**消费方进程**







## Ribbon的流程

![image-20230523150220629](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523150221319-1099794917.png)



通过上图一定要明白一点：**Ribbon一定是用在消费方，而不是服务的提供方！**





**Ribbon在工作时分成两步（这里以Eureka为例，consul和zk同样道理）：**

- 第一步先选择 EurekaServer ,它优先选择在同一个区域内负载较少的server.
- 第二步再根据用户指定的策略(轮询、随机、响应时间加权.....)，从server取到的服务注册列表中选择一个地址







## 请求怎么从服务名地址变为真实地址的？

只要引入了注册中心(Eureka、consul、zookeeper)，那Ribbon的依赖就在注册中心里面了，证明如下：

![image-20230523150713088](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523150714289-322784310.png)







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

![image-20230523164301233](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523164303496-89314463.png)



然后进行Debug服务消费者

![image-20230523164748273](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523164749307-84514600.png)



![image-20230523164905276](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523164906453-461257891.png)



![image-20230523165133615](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523165134892-1027704730.png)



![image-20230523165332402](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523165333560-193867768.png)



![image-20230523170043376](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523170045156-1643577053.png)



![image-20230523170132894](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523170133632-1871497916.png)





![image-20230523171129379](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523171130565-852624319.png)





![image-20230523171313688](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523171314721-632166842.png)







![image-20230523171516222](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523171517502-1987407255.png)







问题的答案已经出来了：**为什么使用服务名就可以调到服务提供方的服务，即：请求 http://userservice/user/101 怎么变成的 http://localhost:8081 ？？**

- 原因就是使用了RibbonLoadBalancerClient+loadBalancer(默认是 ZoneAwareLoadBalance 从服务列表中选取服务)+IRule(默认是 RoundRobinRule 轮询策略选择某个服务)

![image-20230523172623741](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523172627992-1458093408.png)









### 总结

SpringCloudRibbon的底层采用了一个拦截器LoadBalancerInterceptor，拦截了RestTemplate发出的请求，对地址做了修改

![image-20230523183514694](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523183515709-898697749.png)









## 负载均衡策略有哪些？

根据前面的铺垫，也知道了负载均衡策略就在 `IRule` 中，那就去看一下

![image-20230523183830372](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523183831369-932003270.png)



转换一下：

![image-20210713225653000](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523183903273-919770286.png)



`ClientConfigEnabledRoundRobinRule`：该策略较为特殊，我们一般不直接使用它。因为它本身并没有实现什么特殊的处理逻辑。一般都是可以通过继承他重写一些自己的策略，默认的choose方法就实现了线性轮询机制

- `BestAvailableRule`：继承自ClientConfigEnabledRoundRobinRule，会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务,然后选择一个并发量最小的服务，该策略的特性是可选出最空闲的实例



`PredicateBasedRule`：继承自ClientConfigEnabledRoundRobinRule，抽象策略，需要重写方法的，然后自己来自己定义过滤规则的

- `AvailabilityFilteringRule`：继承PredicateBasedRule，先过滤掉故障实例,再选择并发较小的实例。过滤掉的故障服务器是以下两种：
	- 1、在默认情况下，这台服务器如果3次连接失败，这台服务器就会被设置为“短路”状态。短路状态将持续30秒，如果再次连接失败，短路的持续时间就会几何级地增加
	- 2、并发数过高的服务器。如果一个服务器的并发连接数过高，配置了AvailabilityFilteringRule规则的客户端也会将其忽略。并发连接数的上限，可以由客户端的`<clientName>.<clientConfigNameSpace>.ActiveConnectionsLimit` 属性进行配置
- `ZoneAvoidanceRule`：继承PredicateBasedRule，默认规则，复合判断server所在区域的性能和server的可用性选择服务器



`com.netflix.loadbalancer.RoundRobinRule`：轮询 Ribbon的默认规则

- `WeightedResponseTimeRule`：对RoundRobinRule的扩展,为每一个服务器赋予一个权重值，服务器响应时间越长，其权重值越小，这个权重值会影响服务器的选择，即：响应速度越快的实例选择权重越大,越容易被选择
- `ResponseTimeWeightedRule`：对RoundRobinRule的扩展,响应时间加权



`com.netflix.loadbalancer.RandomRule`：随机

`com.netflix.loadbalancer.StickyRule`：这个基本也没人用

`com.netflix.loadbalancer.RetryRule`：先按照RoundRobinRule的策略获取服务,如果获取服务失败则在指定时间内会进行重试,获取可用的服务

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
// 在服务消费者的启动类中加入如下注解即可 如下注解指的是：调用 USER-SERVICE 服务时 使用MySelfRule负载均衡规则
@RibbonClient(name = "USER-SERVICE",configuration=MySelfRule.class)		// 这里的MySelfRule可以弄为自定义逻辑的策略，也可以是前面提到的那些rule策略
```

这种方式可以达到只针对某服务做负载均衡策略，但是：**官方给出了明确警告** `configuration=MySelfRule.class` 自定义配置类一定不能放到@ComponentScan所扫描的当前包下以及子包下，否则我们自定义的这个配置类就会被所有的`Ribbon`客户端所共享，达不到特殊化定制的目的了（也就是一旦被扫描到，RestTemplate直接不管调用哪个服务都会用指定的算法）

> springboot项目当中的启动类使用了@SpringBootApplication注解，这个注解内部就有@ComponentScan注解，默认是扫描启动类包下所有的包，所以我们要达到定制化一定不要放在它能扫描到的地方



cloud中文官网：https://www.springcloud.cc/spring-cloud-greenwich.html#netflix-ribbon-starter



![image-20230523193844609](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523193845845-309027790.png)







**3、使用yml配置文件方式** 在服务消费方的yml配置文件中加入如下格式的内容即可

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

而饥饿加载则会在项目启动时创建，降低第一次访问的耗时，通过下面配置开启饥饿加载：

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

GitHub主页：https://github.com/alibaba/nacos

GitHub的Release下载页：https://github.com/alibaba/nacos/releases

下载好之后直接解压即可，但：别解压到有中文路径的地方

Nacos的默认端口是8848，若该端口被占用则关闭该进程 或 修改nacos中的默认端口(conf/application.properties)



启动Nacos

```txt
startup.cmd -m standalone

-m 		modul 模式
standalone	单机
```



密码和账号均是 nacos

![image-20230523220722633](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230523220723591-2042011918.png)







### Linux安装

Nacos是基于Java开发的，所以需要JDK支持，因此Linux中需要有JDK环境

上传Linux版的JDK

```txt
# 解压
tar -xvf jdk-8u144-linux-x64.tar.gz

# 配置环境变量
export JAVA_HOME=/usr/local/java										=JDK解压后的路径
export PATH=$PATH:$JAVA_HOME/bin

# 刷新环境变量
source /etc/profile
```





上传Linux版的Nacos

```txt
# 解压
tar -xvf nacos-server-1.4.1.tar.gz

# 进入nacos/bin目录中，输入命令启动Nacos：
sh startup.sh -m standalone

# 有8848端口冲突和windows中一样方式解决
```





## 注册服务到Nacos中

拉取Ncaos的依赖管理，父项目工程中加入如下依赖

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

**注：**不要有其他注册中心的依赖，如前面玩的Eureka，有的话先注释掉





修改客户端的yml配置文件：

```yaml
server:
  port: 8081
spring:
  application:
    name: USER-SERVICE
  cloud:
    nacos:
      discovery:
        # Nacos服务器地址，本地启动的Nacos
        server-addr: localhost:8848
#eureka:
#  client:
#    # 去哪里拉取服务列表
#    service-url:
#      defaultZone: http://localhost:10086/eureka
```



启动之后即可在前面打开的Nacos控制台看到信息了

![image-20230524172640484](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524172641663-1724265961.png)





## Nacos集群配置与负载均衡策略调整

**1、集群配置**：Nacos的服务多级存储模型和其他的不一样

![image-20230524173246752](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524173247256-1640638181.png)



就多了一个集群，不像其他的是 服务-----> 实例，好处：微服务互相访问时，应该**尽可能访问同集群实例，因为本地访问速度更快。当本集群内不可用时，才访问其它集群**





配置服务集群：想要对哪个服务配置集群则在其yml配置文件中加入即可

```yaml
server:
  port: 8081
  application:
    name: USER-SERVICE
  cloud:
    nacos:
      discovery:
        # Nacos服务器地址，本地启动的Nacos
        server-addr: localhost:8848
        # 配置集群名称，如：HZ，杭州
        cluster-name: HZ
```

测试直接将服务提供者复刻多份，共用同一集群名启动，然后再复刻修改集群名启动即可，如下面的：

![image-20230524174419882](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524174420790-807433895.png)







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

![image-20230524184809397](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524184810939-1621404096.png)









**3、加权策略** ：服务器权重值越高，越容易被选择，所以能者多劳，性能好的服务器被访问的次数应该越多

权重值一般在 [0,10000] 之间。直接去Nacos得到控制台中选择想要修改权重值的服务，点击“详情”即可修改

![image-20230524200353921](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230524200355093-646170701.png)



**注：** 当权重值为0时，代表此服务实例不会再被访问，类似于停机迭代，而直接修改权重值为0之后，就可以直接进行版本迭代，然后慢慢调整权重值进行”引流“了







## Nacos环境隔离

前面一节见到了Nacos的集群结构，但那只是较内的一层，**Nacos不止是注册中心，也可以是数据中心**

![image-20230525115608614](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525115608477-2097480414.png)



- **namespace** ：就是环境隔离，如 dev开发环境、test测试环境、prod生产环境。若没配置，则默认是public，在没有指定命名空间时都会默认从`public`这个命名空间拉取配置以及注册到该命名空间下的注册表中
- **group** ：就是在namespace的基础上，再进行分组，如 将服务相关性强的分在一个组
- **service ----> clusters -----> instances** ：就是前面说的集群，服务 ----> 集群 ------> 实例







**配置namespace：** 注意事项如下

1.  同名的命名空间只能创建一个
2. 微服务间如果没有注册到一个命名空间下，无法使用OpenFeign指定服务名负载通信（服务拉取的配置文件不同命名空间不影响）。Feign是后面要玩的

![image-20230525120134073](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525120133562-1501134563.png)



![image-20230525120229740](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525120229108-2011422258.png)





![image-20230525120255821](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525120255381-567965134.png)





**在yml配置文件中进行环境隔离配置**

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: HZ
        # 环境隔离：即当前这个服务要注册到哪个命名空间环境去
        # 值为在Nacos控制台创建命名空间时的id值，如下面的dev环境
        namespace: e7144264-0bf4-4caa-a17d-0af8e81eac3a
```











## Nacos临时与非临时实例

**1、Eureka和Nacos的不同：**不同在下图字体加粗的部分，加粗是Nacos具备而Eureka不具备的

![image-20230525141447350](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525141447179-1876206647.png)



**临时实例：** 由服务提供者主动给Nacos发送心跳情况，在规定时间内要是没有发送，则Nacos认为此服务挂了，就会从服务列表中踢掉（非亲生儿子）

**非临时实例：**由Nacos主动来询问服务是否还健康、活着(会让服务器压力大)，若非临时实例挂了，Naocs并不会将其踢掉（亲儿子）

**push：**若是Nacos检测到有服务提供者挂了，就会主动给消费者发送服务变更的消息，然后服务消费者更新自己的服务缓存列表。这一步就会让服务列表更新很及时

- 此方式是Nacos具备的，Eureka不具备，Eureka只有pull操作，因此Eureka的缺点就是服务更新可能会不及时(在30s内，服务提供者变动了，个别挂了，而消费者中的服务缓存列表还是旧的，只能等到30s到了才去重新pull)

**Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式；Eureka采用AP方式**





**补充：CAP定理** 这是分布式中的一个方法论

- C	即：数据一致性

- A	即：可用性

- P	即：分区容错性



**解读：**

​	**数据一致性：一句话来形容，就是：无论以何种方式写入 / 显示数据，最终的结果都要一样**，反例：可能由于网络 / 机器故障，从而导致开始存到缓存中的数据（副本） 和 后面最新的数据在“故障”的情况下导致最终写入数据库的数据 / 显示出来的数据不一致，反过来：保证这两者一致就是：数据一致性

​	**可用性：还是一句话来形容，就是：在可接受的时间范围内，只要最终的数据能够成功写入 / 显示出来就行**，反例：一个页面显示的数据，原本最高限度是2秒之内能够显示出来就OK的，但是：整了10多秒还没显示出来，这就不能称之为可用了，就是个“垃圾”

​	**分区容错性：照样用一句话来形容，就是：由于某种原因，某个服务挂了，整个系统照样运行**



**注：** 分区容错性是必须满足的，数据一致性( C ）和 可用性（ A ）只满足其一即可，当然：要是不考虑网络之类的原因的话，C和A也是可以同时满足的，可是一般的搭配是如下的**（即：取舍策略）：**

- CP			保证数据的准确性

- AP			保证数据的及时性







既然CAP定理都整了，那就再加一个**Base理论**吧

- BA	 即：基本可用性

- S	   即：软状态

- E	   即：最终一致性



**解读：**

- BA 	基本可用性，在发生故障的时候，可以允许损失“部分”的可用性，保证系统正常运行。如：搜索数据时，正常情况耗时0.5秒即可有数据显示，但是：发生故障了，可以允许有极短时间的延时，如：延时1 ~ 2秒以内
- S		软状态，允许系统的数据存在中间状态，只要不影响整个系统的运行就行。如：架构的演进中说过的数据库的分库演进，在读数据库 和 写数据库之间的同步操作时，可以允许这个同步操作有一定的延时
- E		最终一致性，无论以何种方式写入数据库 / 显示出来，都要保证系统最终的数据是一致的









**2、配置临时实例与非临时实例：**在需要的一方的yml配置文件中配置如下开关即可

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        cluster-name: HZ
        # 默认为true，即临时实例
        ephemeral: false
```

改完之后可以在Nacos控制台看到服务是否为临时实例

![image-20230525142657931](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525142657387-321996528.png)



要想去追源码的话，可以找 `com.alibaba.cloud.nacos.ribbon.NacosRule`

![image-20230525143010210](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525143010235-489457706.png)



追的过程中会看到一些其他信息，目前混个脸熟：

```json
{
  "hosts": [
    {
      "ip": "xxx.xxx.xxx.xxx",
      "port": 8082,
      "valid": true,
      "healthy": true,		// 是否健康
      "marked": false,
      "instanceId": "192.168.1.117#8082#HZ#DEFAULT_GROUP@@USER-SERVICE",
      "metadata": {
        "preserved.register.source": "SPRING_CLOUD"
      },
      "enabled": true,
      "weight": 1.0,		// 权重值
      "clusterName": "HZ",		// 此服务所在的集群名
      "serviceName": "DEFAULT_GROUP@@USER-SERVICE",
      "ephemeral": true		// 是否为临时实例
    },
  ],
  "dom": "DEFAULT_GROUP@@USER-SERVICE",
  "name": "DEFAULT_GROUP@@USER-SERVICE",
  "cacheMillis": 10000,
  "lastRefTime": 1684938061774,
  "checksum": "220811216e5bec9c265e009dd8494daa",
  "useSpecifiedURL": false,
  "clusters": "",
  "env": "",
  "metadata": {
    
  }
}
```









## Nacos统一配置管理

**统一配置管理：** 将容易发生改变的配置单独弄出来，然后在后续需要变更时，直接去统一配置管理处进行更改，这样凡是依赖于这些配置的服务就可以统一被更新，而不用挨个服务更改配置，同时更改配置之后不用重启服务，直接实现热更新

![image-20230525194143607](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525194142989-983680569.png)



Nacos和SpringCloud原生的config不一样，Nacos是将注册中心+config结合在一起了，而SpringCloud原生的是Eureka+config













**1、设置Nacos配置管理**

![image-20230525200157809](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525200157228-518484269.png)



![image-20230525205325049](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525205325093-460315993.png)



![image-20230525200924326](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525200923433-1263014836.png)



以上便是在Nacos中设置了统一配置。但是：项目/服务想要得到这些配置，那就得获取到这些配置，怎么办？

在前面说过SpringCloud中有两种yml的配置方式，一种是 `application.yml` ，一种是 `bootstrap.yml` ，这里就需要借助后者了，它是引导文件，优先级比前者高，会优先被加载，这样就可以先使用它加载到Nacos中的配置文件，然后再读取 `application.yml` ，从而完成Spring的那一套注册实例的事情

![image-20230525201257171](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525201257229-1149037588.png)



**2、在需要读取Nacos统一配置的服务中引入如下依赖：**

```xml
<!--nacos配置管理依赖-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```





**3、resources下新建 bootstrap.yml，`bootstrap.yml` 中的配置内容如下**

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
      discovery:
        # nacos服务器地址，需要知道去哪里拉取配置信息
        server-addr: localhost:8848
      config:
        # 文件后缀，对应在nacos中进行配置管理的data id的后缀名
        file-extension: yaml
```



![image-20230525205430252](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525205429454-1265194565.png)



经过上面的操作之后，以前需要单独在 `application.yml` 改的事情就不需要了，`bootstrap.yml` 配置的东西会去拉取nacos中的配置





**4、设置热更新：** 假如业务代码中有需要用到nacos中的服务，那nacos中的配置改变之后，不需要重启服务，自动更新。一共有两种方式

- 1. **`@RefreshScope+@Value`注解：** 在@Value注入的变量**所在类上**添加注解@RefreshScope

![image-20230525205534523](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525205534211-29938183.png)



- 2. `@ConfigurationProperties` 注解

![image-20230525210116200](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525210115994-1555810348.png)



然后在需要的地方直接注入对象即可

![image-20230525210204143](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525210203864-778691054.png)









## Nacos多环境共享配置

有时会遇到这样的情况：生产环境、开发环境、测试环境有些配置时相同的，这种应该不需要在每个环境中都配置，因此需要让这些相同的配置单独弄出来，然后实行共享

在前面一节中已经说到了一种Nacos的配置文件格式 即 `服务名-环境.后缀`，除了这种还有一种格式 即 `服务名.后缀`

因此：想要让环境配置共享，那么直接在Nacos控制台的配置中再加一个以` 服务名.后缀名` 格式命名的配置即可，如下：

![image-20230525214926182](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525214929647-1348957116.png)

其他的都不用动，要只是针对于项目中的yml，如 `appilication.yml`，那前面已经说了，会先读取Nacos中配置，然后和 `application.yml` 进行合并

但是：若项目本地的yml中、服务名.后缀、服务名-环境.后缀 中有相同的属性/配置时，优先级不一样，如下：

![image-20230525215737066](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525215738196-645671337.png)









## Nacos集群部署

windows和Linux都是一样的思路，集群部署的逻辑如下：

![image-20210409211355037](https://img2023.cnblogs.com/blog/2421736/202305/2421736-20230525230904184-1376989748.png)



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







# OpenFeign远程调用















































