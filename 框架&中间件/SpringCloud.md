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







# Feign远程调用

## Feign与OpenFeign是什么？

Feign是`Netflix`开发的`声明式、模板化`的HTTP客户端， 在 RestTemplate 的基础上做了进一步的封装，Feign可以帮助我们更快捷、优雅地调用HTTP API。具有可插入注释支持，包括Feign注释和JAX-RS注释，通过 Feign，我们只需要声明一个接口并通过注解进行简单的配置（类似于 Dao 接口上面的 Mapper 注解一样）即可实现对 HTTP 接口的绑定；通过 Feign，我们可以像调用本地方法一样来调用远程服务，而完全感觉不到这是在进行远程调用



OpenFeign全称Spring Cloud OpenFeign，2019 年 Netflix 公司宣布 Feign 组件正式进入停更维护状态，于是 Spring 官方便推出了一个名为 OpenFeign 的组件作为 Feign 的替代方案。基于Netflix feign实现，是一个**声明式**的http客户端，整合了`Spring Cloud Ribbon`，除了支持netflix的feign注解之外，增加了对Spring MVC注释的支持，OpenFeign 的@FeignClient可以解析SpringMVC的@RequestMapping注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务

- **声明式·：** 即只需要将调研服务需要的东西声明出来，剩下就不用管了，交给feign即可



> Spring Cloud Finchley 及以上版本一般使用 OpenFeign 作为其服务调用组件。由于 OpenFeign 是在 2019 年 Feign 停更进入维护后推出的，因此大多数 2019 年及以后的新项目使用的都是 OpenFeign，而 2018 年以前的项目一般使用 Feign





## OpenFeign 常用注解

使用 OpenFegin 进行远程服务调用时，常用注解如下表。 



| 注解                | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| @FeignClient        | 该注解用于通知 OpenFeign 组件对 @RequestMapping 注解下的接口进行解析，并通过动态代理的方式产生实现类，实现负载均衡和服务调用。 |
| @EnableFeignClients | 该注解用于开启 OpenFeign 功能，当 Spring Cloud 应用启动时，OpenFeign 会扫描标有 @FeignClient 注解的接口，生成代理并注册到 Spring 容器中。 |
| @RequestMapping     | Spring MVC 注解，在 Spring MVC 中使用该注解映射请求，通过它来指定控制器（Controller）可以处理哪些 URL 请求，相当于 Servlet 中 web.xml 的配置。 |
| @GetMapping         | Spring MVC 注解，用来映射 GET 请求，它是一个组合注解，相当于 @RequestMapping(method = RequestMethod.GET) 。 |
| @PostMapping        | Spring MVC 注解，用来映射 POST 请求，它是一个组合注解，相当于 @RequestMapping(method = RequestMethod.POST) 。 |







## Feign VS OpenFeign 

下面我们就来对比下 Feign 和 OpenFeign 的异同





### 相同点

Feign 和 OpenFegin 具有以下相同点：

- Feign 和 OpenFeign 都是 Spring Cloud 下的远程调用和负载均衡组件
- Feign 和 OpenFeign 作用一样，都可以实现服务的远程调用和负载均衡
- Feign 和 OpenFeign 都对 Ribbon 进行了集成，都利用 Ribbon 维护了可用服务清单，并通过 Ribbon 实现了客户端的负载均衡
- Feign 和 OpenFeign 都是在服务消费者（客户端）定义服务绑定接口并通过注解的方式进行配置，以实现远程服务的调用



### 不同点

Feign 和 OpenFeign 具有以下不同：

- Feign 和 OpenFeign 的依赖项不同，Feign 的依赖为 spring-cloud-starter-feign，而 OpenFeign 的依赖为 spring-cloud-starter-openfeign
- Feign 和 OpenFeign 支持的注解不同，Feign 支持 Feign 注解和 JAX-RS 注解，但不支持 Spring MVC 注解；OpenFeign 除了支持 Feign 注解和 JAX-RS 注解外，还支持 Spring MVC 注解



## 入手OpenFeign

OpenFeign是Feign的增强版，使用时将依赖换一下，然后注意一下二者能支持的朱姐的区别即可



**1、依赖:**在服务消费方添加如下依赖

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





**2、启动类假如如下注解：**在服务消费方启动类添加

```java
@EnableFeignClients     /*开启feign客户端功能*/
```





**3、创建接口，并使用 `@@org.springframework.cloud.openfeign.FeignClient` 注解：**这种方式相当于是 `DAO`

```java
/**
 * Spring Cloud 应用在启动时，OpenFeign 会扫描标有 @FeignClient 注解的接口生成代理，并注人到 Spring 容器中
 */

@FeignClient("USER-SERVICE")            /*参数为要调研的服务名，这里的服务名区分大小写*/
public interface FeignClient {
    /**
     * 支持SpringMVC的所有注解
     * @param id
     * @return
     */
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") long id);
}
```

在编写服务绑定接口时，需要注意以下 2 点：

- 在 @FeignClient 注解中，value 属性的取值为：服务提供者的服务名，即服务提供者配置文件（application.yml）中 spring.application.name 的取值。
- 接口中定义的每个方法都与 服务提供者 中 Controller 定义的服务方法对应





**4、在需要调用3中服务与方法的地方进行调用**

```java
import com.zixieqing.order.client.FeignClient;
import com.zixieqing.order.entity.Order;
import com.zixieqing.order.entity.User;
import com.zixieqing.order.mapper.OrderMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * <p>@description  : 该类功能  order服务
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
| **feign.Logger.Level** | 修改日志级别     | 包含四种不同的级别：NONE、BASIC、HEADERS、FULL<br />1、NONE：默认的，不显示任何日志<br />2、BACK：仅记录请求方法、URL、响应状态码及执行时间<br />3、HEADERS：除了BASIC中定义的信息之外，还有请求和响应的头信息<br />4、FULL：除了HEADERS中定义的信息之外，还有请求和响应的正文及元数据 |
| feign.codec.Decoder    | 响应结果的解析器 | http远程调用的结果做解析，例如解析json字符串为Java对象       |
| feign.codec.Encoder    | 请求参数编码     | 将请求参数编码，便于通过http请求发送                         |
| feign. Contract        | 支持的注解格式   | 默认是SpringMVC的注解                                        |
| feign. Retryer         | 失败重试机制     | 请求失败的重试机制，默认是没有，不过会使用Ribbon的重试       |

一般情况下，默认值就能满足我们使用，如果要自定义时，只需要创建自定义的 `@Bean` 覆盖默认Bean即可







### 配置日志增强

这个有4中配置方式，局部配置（2种=YAML+代码实现）、全局配置（2种=YAML+代码实现）





基于YAML文件修改Feign的日志级别可以针对单个服务：即局部配置

```yaml
feign:  
  client:
    config: 
      userservice: # 针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```

也可以针对所有服务：即全局配置

```yaml
feign:  
  client:
    config: 
      default: # 这里用default就是全局配置，如果是写服务名称，则是针对某个微服务的配置
        loggerLevel: FULL #  日志级别 
```





也可以基于Java代码来修改日志级别，先声明一个类，然后声明一个Logger.Level的对象：

```java
/** 
 * 注：这里可以不用加 @Configuration 注解
 * 因为要么在启动类得到 @EnableFeignClients 注解中进行声明这个配置类
 * 要么在远程服务调用的接口的 @FeignClient 注解中声明该配置
 */
public class DefaultFeignConfiguration  {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC; // 日志级别为BASIC
    }
}
```



如果要**全局生效**，将其放到启动类的 `@EnableFeignClients` 这个注解中：

```java
@EnableFeignClients(defaultConfiguration = DefaultFeignConfiguration .class) 
```



如果是**局部生效**，则把它放到对应的 `@FeignClient` 这个注解中：

```java
@FeignClient(value = "userservice", configuration = DefaultFeignConfiguration .class) 
```











### 配置客户端

Feign底层发起http请求，依赖于其它的框架。其底层客户端实现包括：

•URLConnection：默认实现，不支持连接池

•Apache HttpClient ：支持连接池

•OKHttp：支持连接池





#### 替换为Apache的HttpClient

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
    # 开启feign对HttpClient的支持		默认值就是true，即导入对应客户端依赖之后就开启了，但为了提高代码可读性，还是显示声明比较好
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

![image-20210714185925910](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230603221112385-452208089.png)

Debug方式启动服务消费者，可以看到这里的client底层就是Apache HttpClient：

![image-20210714190041542](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230603221111983-216336226.png)













# Gateway网关

在微服务架构中，一个系统往往由多个微服务组成，而这些服务可能部署在不同机房、不同地区、不同域名下。这种情况下，客户端（例如浏览器、手机、软件工具等）想要直接请求这些服务，就需要知道它们具体的地址信息，例如 IP 地址、端口号等。

这种客户端直接请求服务的方式存在以下问题：

- 当服务数量众多时，客户端需要维护大量的服务地址，这对于客户端来说，是非常繁琐复杂的。
- 在某些场景下可能会存在跨域请求的问题。
- 身份认证的难度大，每个微服务需要独立认证。


我们可以通过 API 网关来解决这些问题，下面就让我们来看看什么是 API 网关。

## API 网关

API 网关是一个搭建在客户端和微服务之间的服务，我们可以在 API 网关中处理一些非业务功能的逻辑，例如权限验证、监控、缓存、请求路由等

API 网关就像整个微服务系统的门面一样，是系统对外的唯一入口。有了它，客户端会先将请求发送到 API 网关，然后由 API 网关根据请求的标识信息将请求转发到微服务实例



![img](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604173637373-1731467261.png)


对于服务数量众多、复杂度较高、规模比较大的系统来说，使用 API 网关具有以下好处：

- 客户端通过 API 网关与微服务交互时，客户端只需要知道 API 网关地址即可，而不需要维护大量的服务地址，简化了客户端的开发。
- 客户端直接与 API 网关通信，能够减少客户端与各个服务的交互次数。
- 客户端与后端的服务耦合度降低。
- 节省流量，提高性能，提升用户体验。
- API 网关还提供了安全、流控、过滤、缓存、计费以及监控等 API 管理功能





常见的 API 网关实现方案主要有以下 5 种：

- Spring Cloud Gateway
- Spring Cloud Netflix Zuul
- Kong
- Nginx+Lua
- Traefik







## 认识SpringCloud Gateway

Spring Cloud Gateway 是 Spring Cloud 团队基于 Spring 5.0、Spring Boot 2.0 和 Project Reactor 等技术开发的高性能 API 网关组件

Spring Cloud Gateway 旨在提供一种简单而有效的途径来发送 API，并为它们提供横切关注点，例如：安全性，监控/指标和弹性

> Spring Cloud Gateway 是基于 WebFlux 框架实现的，而 WebFlux 框架底层则使用了高性能的 Reactor 模式通信框架 Netty





## Spring Cloud Gateway 核心概念

Spring Cloud GateWay 最主要的功能就是路由转发，而在定义转发规则时主要涉及了以下三个核心概念，如下表。



| 核心概念          | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| Route（路由）     | 网关最基本的模块。它由一个 ID、一个目标 URI、一组断言（Predicate）和一组过滤器（Filter）组成。 |
| Predicate（断言） | 路由转发的判断条件，我们可以通过 Predicate 对 HTTP 请求进行匹配，例如请求方式、请求路径、请求头、参数等，如果请求与断言匹配成功，则将请求转发到相应的服务。 |
| Filter（过滤器）  | 过滤器，我们可以使用它对请求进行拦截和修改，还可以使用它对上文的响应进行再处理。 |

> 注意：其中 Route 和 Predicate 必须同时声明。





网关的**核心功能特性**：

- 请求路由
- 权限控制
- 限流

架构图：

![image-20210714210131152](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604174001899-867105716.png)



**权限控制**：网关作为微服务入口，需要校验用户是是否有请求资格，如果没有则进行拦截。

**路由和负载均衡**：一切请求都必须先经过gateway，但网关不处理业务，而是根据某种规则，把请求转发到某个微服务，这个过程叫做路由。当然路由的目标服务有多个时，还需要做负载均衡。

**限流**：当请求流量过高时，在网关中按照下流的微服务能够接受的速度来放行请求，避免服务压力过大









## Gateway 的工作流程

Spring Cloud Gateway 工作流程如下图:



![Spring Cloud Gateway 工作流程](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604174027529-1077440941.png)


Spring Cloud Gateway 工作流程说明如下：

1. 客户端将请求发送到 Spring Cloud Gateway 上。
2. Spring Cloud Gateway 通过 Gateway Handler Mapping 找到与请求相匹配的路由，将其发送给 Gateway Web Handler。
3. Gateway Web Handler 通过指定的过滤器链（Filter Chain），将请求转发到实际的服务节点中，执行业务逻辑返回响应结果。
4. 过滤器之间用虚线分开是因为过滤器可能会在转发请求之前（pre）或之后（post）执行业务逻辑。
5. 过滤器（Filter）可以在请求被转发到服务端前，对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等。
6. 过滤器可以在响应返回客户端之前，对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量监控等。
7. 响应原路返回给客户端



总而言之，客户端发送到 Spring Cloud Gateway 的请求需要通过一定的匹配条件，才能定位到真正的服务节点。在将请求转发到服务进行处理的过程前后（pre 和 post），我们还可以对请求和响应进行一些精细化控制。

Predicate 就是路由的匹配条件，而 Filter 就是对请求和响应进行精细化控制的工具。有了这两个元素，再加上目标 URI，就可以实现一个具体的路由了



当然，要是再加上前面已经玩过的东西的流程就变成下面的样子了：

![image-20210714211742956](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604175754741-755292829.png)





## Predicate 断言

Spring Cloud Gateway 通过 Predicate 断言来实现 Route 路由的匹配规则。简单点说，Predicate 是路由转发的判断条件，请求只有满足了 Predicate 的条件，才会被转发到指定的服务上进行处理。

使用 Predicate 断言需要注意以下 3 点：

- Route 路由与 Predicate 断言的对应关系为“一对多”，一个路由可以包含多个不同断言。
- 一个请求想要转发到指定的路由上，就必须同时匹配路由上的所有断言。
- 当一个请求同时满足多个路由的断言条件时，请求只会被首个成功匹配的路由转发。



![img](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230604174131390-660098036.png)







常见的 Predicate 断言如下表（假设转发的 URI 为 http://localhost:8001）



| 断言       | 示例                                                         | 说明                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Path       | - Path=/dept/list/**                                         | 当请求路径与 /dept/list/** 匹配时，该请求才能被转发到 http://localhost:8001 上 |
| Before     | - Before=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]        | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之前的请求，才会被转发到 http://localhost:8001 上 |
| After      | - After=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]         | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之后的请求，才会被转发到 http://localhost:8001 上 |
| Between    | - Between=2021-10-20T15:18:33.226+08:00[Asia/Shanghai],2021-10-20T15:23:33.226+08:00[Asia/Shanghai] | 在 2021 年 10 月 20 日 15 时 18 分 33.226 秒 到 2021 年 10 月 20 日 15 时 23 分 33.226 秒之间的请求，才会被转发到 http://localhost:8001 服务器上 |
| Cookie     | - Cookie=name,www.cnblogs.com/xiegongzi                      | 携带 Cookie 且 Cookie 的内容为 name=www.cnblogs.com/xiegongzi 的请求，才会被转发到 http://localhost:8001 上 |
| Header     | - Header=X-Request-Id,\d+                                    | 请求头上携带属性 X-Request-Id 且属性值为整数的请求，才会被转发到 http://localhost:8001 上 |
| Method     | - Method=GET                                                 | 只有 GET 请求才会被转发到 http://localhost:8001 上           |
| Host       | -  Host=**.somehost.org,**.anotherhost.org                   | 请求必须是访问.somehost.org,.anotherhost.org这两个host（域名）才会被转发到 http://localhost:8001 上 |
| Query      | - Query=name                                                 | 请求参数必须包含指定参数(name)，才会被转发到 http://localhost:8001 上 |
| RemoteAddr | - RemoteAddr=192.168.1.1/24                                  | 请求者的ip必须是指定范围（192.168.1.1 到 192.168.1.24)       |
| Weight     | ![image-20230605120547194](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230605120548651-1280651580.png) | 权重处理weight,有两个参数：group和weight(一个整数)<br />如示例中表示：分80%的流量给weihthigh.org |

上表中这些也叫“**Predicate断言工厂**”，我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factory读取并处理，转变为路由判断的条件

例如Path=/user/**是按照路径匹配，这个规则是由

`org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory`类来

处理的





## 入手Gateway

新建一个Maven项目，pom内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.zixieqing</groupId>
        <artifactId>gateway-parent</artifactId>
        <version>1.0-SNAPSHOT</version>
    </parent>

    <artifactId>gateway</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
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
    </dependencies>
</project>
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
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
        - id: userservice
          uri: lb://userservice
          predicates:
            - Path=/user/**
```

经过如上方式，就简单搭建了Gateway网关，启动、访问 localhost:10010/user/1 或 localhost:10010/order/101 即可







## filter 过滤器

通常情况下，出于安全方面的考虑，服务端提供的服务往往都会有一定的校验逻辑，例如用户登陆状态校验、签名校验等

在微服务架构中，系统由多个微服务组成，所有这些服务都需要这些校验逻辑，此时我们就可以将这些校验逻辑写到 Spring Cloud Gateway 的 Filter 过滤器中



Filter是网关中提供的一种过滤器，可以对进入网关的请求和微服务返回的响应做处理：

![image-20210714212312871](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230607230146990-155503942.png)



pring Cloud Gateway 提供了以下两种类型的过滤器，可以对请求和响应进行精细化控制。



| 过滤器类型 | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| Pre 类型   | 这种过滤器在请求被转发到微服务之前可以对请求进行拦截和修改，例如参数校验、权限校验、流量监控、日志输出以及协议转换等操作。 |
| Post 类型  | 这种过滤器在微服务对请求做出响应后可以对响应进行拦截和再处理，例如修改响应内容或响应头、日志输出、流量监控等。 |


按照作用范围划分，Spring Cloud gateway 的 Filter 可以分为 2 类：

- GatewayFilter：应用在单个路由或者一组路由上的过滤器。
- GlobalFilter：应用在所有的路由上的过滤器。







### GatewayFilter 网关过滤器

GatewayFilter 是 Spring Cloud Gateway 网关中提供的一种应用在单个或一组路由上的过滤器。它可以对单个路由或者一组路由上传入的请求和传出响应进行拦截，并实现一些与业务无关的功能，比如登陆状态校验、签名校验、权限校验、日志输出、流量监控等



GatewayFilter 在配置文件（例如 application.yml）中的写法与 Predicate 类似，格式如下：

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
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
          filters: # gateway过滤器
            - AddRequestHeader=name, zixieqing # 添加请求头name=zixieqing
        - id: userservice
          uri: lb://userservice
          predicates:
            - Path=/user/**
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



此种路由一共有37种，它们的用法和上面的差不多，可以多个过滤器共同使用，详细去看链接：https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories



下表中列举了几种常用的网关过滤器：



| 路由过滤器             | 描述                                                         | 参数                                                         | 使用示例                                               |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ |
| AddRequestHeader       | 拦截传入的请求，并在请求上添加一个指定的请求头参数。         | name：需要添加的请求头参数的 key； value：需要添加的请求头参数的 value。 | - AddRequestHeader=my-request-header,1024              |
| AddRequestParameter    | 拦截传入的请求，并在请求上添加一个指定的请求参数。           | name：需要添加的请求参数的 key； value：需要添加的请求参数的 value。 | - AddRequestParameter=my-request-param,c.biancheng.net |
| AddResponseHeader      | 拦截响应，并在响应上添加一个指定的响应头参数。               | name：需要添加的响应头的 key； value：需要添加的响应头的 value。 | - AddResponseHeader=my-response-header,c.biancheng.net |
| PrefixPath             | 拦截传入的请求，并在请求路径增加一个指定的前缀。             | prefix：需要增加的路径前缀。                                 | - PrefixPath=/consumer                                 |
| PreserveHostHeader     | 转发请求时，保持客户端的 Host 信息不变，然后将它传递到提供具体服务的微服务中。 | 无                                                           | - PreserveHostHeader                                   |
| RemoveRequestHeader    | 移除请求头中指定的参数。                                     | name：需要移除的请求头的 key。                               | - RemoveRequestHeader=my-request-header                |
| RemoveResponseHeader   | 移除响应头中指定的参数。                                     | name：需要移除的响应头。                                     | - RemoveResponseHeader=my-response-header              |
| RemoveRequestParameter | 移除指定的请求参数。                                         | name：需要移除的请求参数。                                   | - RemoveRequestParameter=my-request-param              |
| RequestSize            | 配置请求体的大小，当请求体过大时，将会返回 413 Payload Too Large。 | maxSize：请求体的大小。                                      | - name: RequestSize   args:    maxSize: 5000000        |







### GlobalFilter 全局过滤器

全局过滤器的作用也是处理一切进入网关的请求和微服务响应



第一种方式就是像上面一样直接在YAML文件中配置

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
          # uri: http://127.0.0.1:8081 # 路由的目标地址 http就是固定地址
          uri: lb://userservice # 路由的目标地址 lb就是负载均衡，后面跟服务名称
          predicates: # 路由断言，也就是判断请求是否符合路由规则的条件
            - Path=/user/** # 这个是按照路径匹配，只要以/user/开头就符合要求
#          filters:
#            - AddRequestHeader=name, zixieqing
        - id: userservice
          uri: lb://userservice
          predicates:
            - Path=/user/**
      default-filters:
        # 全局过滤器
        - AddRequestHeader=name, zixieqing
```

此种方式缺点就是要是需要编写复杂的业务逻辑时会非常不方便，但是：**这种过滤器的优先级比下面一种要高**





第二种方式就是使用代码实现，定义方式是实现GlobalFilter接口：

```java
public interface GlobalFilter {
    /**
     *  处理当前请求，有必要的话通过{@link GatewayFilterChain}将请求交给下一个过滤器处理
     *
     * @param exchange 请求上下文，里面可以获取Request、Response等信息
     * @param chain 用来把请求委托给下一个过滤器 
     * @return {@code Mono<Void>} 返回标示当前过滤器业务结束
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
 * <p>@description  : 该类功能  自定义gateway全局路由器
 * </p>
 * <p>@author       : ZiXieqing</p>
 */

@Order(-1)  // 这个注解和本类实现 Ordered 是一样的效果，都是返回一个整数
            // 这个整数表示当前过滤器的执行优先级，越小优先级越高，取值范围就是 int的范围
@Component
public class MyGlobalFilter implements GlobalFilter {
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

        // 不再执行下去，到此结束 setComplete即设置王成的意思
        return exchange.getResponse().setComplete();
    }
}
```





### 过滤器执行顺序

请求进入网关会碰到三类过滤器：当前路由的过滤器、DefaultFilter、GlobalFilter

请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链（集合）中，排序后依次执行每个过滤器：

![image-20210714214228409](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230610170715610-1341628357.png)



排序的规则是什么呢？

- 每一个过滤器都必须指定一个int类型的order值，**order值越小，优先级越高，执行顺序越靠前**。
- GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
- 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
- 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。



详细内容，可以查看源码：

`org.springframework.cloud.gateway.route.RouteDefinitionRouteLocator#getFilters()`方法是先加载defaultFilters，然后再加载某个route的filters，然后合并。



`org.springframework.cloud.gateway.handler.FilteringWebHandler#handle()`方法会加载全局过滤器，与前面的过滤器合并后根据order排序，组织过滤器链









## 网关跨域问题

跨域：域名不一致就是跨域，主要包括：

- 域名不同： www.taobao.com 和 www.taobao.org 和 www.jd.com 和 miaosha.jd.com

- 域名相同，端口不同：localhost:8080和localhost8081

跨域问题：浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题



解决方案：CORS，想学习的话可以去这里 https://www.ruanyifeng.com/blog/2016/04/cors.html





### 全局跨域

解决方式：在gateway服务的 application.yml 文件中，添加下面的配置：

```yaml
spring:
  cloud:
    gateway:
      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题。CORS跨域浏览器会问服务器可不可以跨域，而这种请求是options，网关默认会拦截这种请求
        corsConfigurations:
          '[/**]':	# 拦截哪些请求，此处为拦截所有请求，即凡是进入网关的请求都拦截
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

“route”配置允许将 CORS 作为元数据直接应用于路由，例如下面的配置：

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



**2、更新本地镜像源为阿里镜像源**

```shell
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
```



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
docker -v
```
出现docker版本号也表示成功



**8、配置镜像加速**

docker官方镜像仓库网速较差，我们需要设置国内镜像服务：

参考阿里云的镜像加速文档：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors







## 镜像名称

首先来看下镜像的名称组成：

- 镜名称一般分两部分组成：[repository]:[tag]。
- 在没有指定tag时，默认是latest，代表最新版本的镜像

如图：

![image-20210731155141362](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230612184039214-132749515.png)

这里的mysql就是repository，5.7就是tag，合一起就是镜像名称，代表5.7版本的MySQL镜像。



## Docker命令

Docker仓库地址(即dockerHub)：https://hub.docker.com

![](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230612185012119-380491969.png)







常见的镜像操作命令如图：

![image-20210731155649535](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230612184112569-603279439.png)



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

指的是：仓库名、标签都是 `<none>` ，即俗称dangling image

出现的原因：在构建镜像或删除镜像时出现了某些错误，从而导致仓库名和标签都是 `<none>`



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

![image-20230613112920030](https://img2023.cnblogs.com/blog/2421736/202306/2421736-20230613112921076-1538350151.png)



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
| RUN        | 容器“运行时”需要执行的命令<br />RUN是在进行docker build时执行 | 在进行docker build时会安装一些命令或插件，亦或输出一句话用来提示进行到哪一步了/当前这一步是否成功了 | 有两种格式：<br />1、shell格式：RUN <命令行命令>  如：RUN echo “Successfully built xxxx” 或者是 RUN yum -y imstall vim<br />这种等价于在终端中执行shell命令<br /><br />2、exec格式：RUN {“可执行文件”,”参数1”,”参数2”} 如：RUN {“./startup.cmd”,”-m”,”standalone”} 等价于 startup.cmd -m standalone |
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

   1. ```shell
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

   2. **注**：Dockerfile文件和jar包最好在同一目录

4. 构建成docker镜像

   1. ```shell
      # docker build -t 仓库名字(REPOSITORY):标签(TAG)
      docker build -t zixq_docker:0.1 .
      # 最后有一个	点.	表示：当前目录，jar包和Dockerfile不都在当前目录吗
      ```

5. 运行镜像

   1. ```shell
      docker run -d -p 8888:8888 镜像ID
      
      # 注意防火墙的问题，端口是否开放或防火墙是否关闭，否则关闭/开放，然后重启docker，重现运行镜像.........
      ```

6. 浏览器访问

   1. ```shell
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
> 3. 注释最好像下面这样写在上面，不要像在IDEA中写在行尾，这样容易导致空格(偶尔会莫名其妙启动不起来，把注释位置改为上面又可以了)



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
- web：一个基于`docker build`临时构建的镜像容器，映射端口时8090







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



**2、配置Docker信任地址**：Docker私服采用的是http协议，默认不被Docker信任，所以需要做一个配

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

1、重新tag本地镜像，名称前缀为私有仓库的地址：192.168.xxx.yyy:8080/

 ```shell
# docker tag 仓库名(REPOSITORY):标签(TAG) YourIp:ui中配置的port/新仓库名:标签
docker tag nginx:latest 192.168.xxx.yyy:8080/nginx:1.0
 ```



2、推送镜像

```shell
docker push 192.168.xxx.yyy:8080/nginx:1.0 
```



3、 拉取镜像

```shell
docker pull 192.168.xxx.yyy:8080/nginx:1.0 
```





# RabbitMQ 消息队列

这里是一部分，全系列的知识去这里：https://www.cnblogs.com/xiegongzi/p/16242291.html

































































