# 1、认识MQ

## 1.1、什么是MQ？

MQ全称：message queue 即 消息队列

队列里面存的就是message



## 1.2、为什么要用MQ？

### 1.2.1、流量削峰

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125238-212175822.png)


这种情况，要是访问 1020次 / s呢？这种肯定会让支付系统宕机了，因为太大了嘛，受不了，所以：流量削峰

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124992-153781379.png)


这样就让message排着队了，这样支付系统就可以承受得了了

<br>



### 1.2.2、应用解耦

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124393-25273329.png)




上面这种，只要支付系统或库存系统其中一个挂彩了，那么订单系统也要挂彩，因此：解耦呗

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124499-1077051331.png)


而采用了MQ之后，支付系统和库存系统有一个出问题，那么它的处理内存是在MQ中的，此时订单系统就不会有影响，可以正常完成，等到故障恢复了，订单系统再处理对应的事情，这就提高了系统的可用性



<br>




### 1.2.3、异步处理

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125565-460270788.png)

如上图，订单系统要调用支付系统的API，而订单系统并不知道支付系统处理对应的业务需要多久，要解决可以采用订单系统隔一定时间又去访问支付系统，看处理完没有，而使用MQ更容易解决。

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124624-633201002.png)


<br>
<br>



## 1.3、RabbitMQ的结构？

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126710-974752633.png)


图中的东西后续会慢慢见到

**Broker实体：**接收和分发消息的应用 / RabbitMQ Server / Message Broker

**而上图中RabbitMQ的四个核心就是：Producer生产者、exchange交换机、queue队列、Consumer消费者**
- Producer生产者：就是负责推送消息的程序
- Exchange交换机：接收来自生产者的消息，并且把消息放到队列中
- queue队列：就是一个数据结构，本质就是一个很大的消息缓冲区
- Consumer消费者：就是接收消息的程序

**注意：**生产者、消息中间件MQ、消费者大多时候并不是在同一台机器上的，所以：生产者有时可以是消费者；而消费者有时也可以是生产者

**Connection链接：**就是让Producer生产者、Broker实体、Consumer消费者之间建立TCP链接

**Virtual host虚拟机：**处于多租户和安全因素考虑而设计的，当多个不同的用户使用同一个 RabbitMQ server 提供的服务时，可以划分出多个 vhost，每个用户在自己的 vhost 创建 exchange／queue 等

**Channel信道：**就是发消息的通道，它是在Connection内部建立的逻辑连接

**Routes路由策略  / binding绑定：**交换机以什么样的策略将消息发布到Queue。也就是exchange交换机 和 queue队列之间的联系，即 二者之间的虚拟连接，它里面可以包含routing key 路由键


<br>
<br>

## 1.4、RabbitMQ的通讯方式

这个玩意儿在官网中有图，地址：https://www.rabbitmq.com/getstarted.html 学完之后这张图最好放在自己脑海里

另外：下面放的是七种，实质上第六种RPC需要在特定的场景和技术下才会使用，因此目前就不演示这个模式了

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125338-706979566.png)

- 1、hello word - 简单模式
- 2、work queues - 工作队列模式
- 3、publish / subscribe - 发布订阅模式
- 4、Routing - 路由模式
- 5、Topics - 主题模式
- 6、RPC模式 - 目前不用了解也行
- 7、publisher confirms - 发布确认模式


<br>
<br>

# 2、安装RabbitMQ

**以下的方式自行选择一种即可**

<br>

## 2.1、在Centos 7下安装

查看自己的Linux版本

```json
	uname -a
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123061-1873517583.png)


<br>
<br>




### 2.1.1、使用rpm红帽软件


> **准备工作**

**1、下载Erlang，**因为：RabbitMQ是Erlang语言写的，Erlang下载地址【 ps：这是官网 】：https://www.erlang.org/downloads，选择自己要的版本即可

另外：RabbitMQ和Erlang的版本对应关系链接地址 https://www.rabbitmq.com/which-erlang.html

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125399-155943226.png)

<br>

当然：上面这种是下载gz压缩包，配置挺烦的，可以直接去github中下载rpm文件，地址：https://github.com/rabbitmq/erlang-rpm/releases ， 选择自己需要的版本即可，注意一个问题：要看是基于什么Linux的版本

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125048-918139794.png)

<br>

要是github下载慢的话，都有自己的文明上网加速方式，要是没有的话，可以进入 https://github.com/fhefh2015/Fast-GitHub 下载好了然后集成到自己浏览器的扩展程序中即可，而如果进入github很慢的话，可以选择去gitee中搜索一个叫做：`dev-sidecar`的东西安装，这样以后进入github就很快了，还有另外的很多方式，不介绍了。



<br>


**2、执行`rpm -ivh  erlang文件`** 命令
- i 就是 install的意思
- vh 就是显示安装进度条
- 注意：需要保证自己的Linux中有rpm命令，没有的话，执行`yum install rpm`指令即可安装rpm

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125452-1201074316.png)

<br>


**3、安装RabbitMQ需要的依赖环境**

```json
	yum install socat -y
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123453-2086115234.png)

<br>


**4、下载RabbitMQ的rpm文件**，github地址：https://github.com/rabbitmq/rabbitmq-server/releases ， 选择自己要的版本即可

<br>


**5、安装RabbitMQ**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124650-1493924187.png)



<br>


**6、启动RabbitMQ服务**

```json
  	启动服务
  	/sbin/service rabbitmq-server start

  	停止服务
  	/sbin/service rabbitmq-server stop

  	查看启动状态
  	/sbin/service rabbitmq-server status

  	开启开机自启
  	chkconfig rabbitmq-server on

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125548-1693241806.png)

查看启动状态

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125406-570163083.png)

这表示正在启动，需要等一会儿，看到下面的样子就表示启动成功

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123032-2141196099.png)




<br>


**7、安装web管理插件**

```json
	1、停止RabbitMQ服务
	service rabbitmq-server stop   // 使用上面的命令 /sbin/service rabbitmq-server stop也行

	2、安装插件
	rabbitmq-plugins enable rabbitmq_management

	3、开启RabbitMQ服务
	service rabbitmq-server start
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125747-1769393023.png)



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124975-238289132.png)

要是访问不了，看看自己的防火墙关没关啊

```json
# 查看防火墙状态
systemctl status firewalld

# 关闭防火墙
systemctl stop firewalld

# 一劳永逸 禁用防火墙
systemctl disable firewalld

# ============================================

# 当然：上面的方式不建议用，可以用如下的方式

# 6379端口号是否开放
firewall-cmd --query-port=6379/tcp

# 开放6379端口
firewall-cmd --permanent --add-port=6379/tcp

#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload

```

同时查看自己的服务器有没有开放15672端口，不同的东西有不同的处理方式，如我的云服务器直接在服务器网址中添加规则即可，其他的方式自行百度


<br>
<br>


### 2.1.2、使用Docker安装

需要保证自己的Linux中有Docker容器，教程链接：https://www.cnblogs.com/xiegongzi/p/15621992.html

使用下面的两种方式都不需要进行web管理插件的安装和erlang的安装


<br>

**1、查看自己的docker容器中是否已有了rabbitmq这个名字的镜像**

```json
	docker images
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123483-2030539881.png)

删除镜像

```json
	docker rmi 镜像ID  // 如上例的 dockerrmi 16c 即可删除镜像
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124933-309485112.png)


<br>



**2、拉取RabbitMQ镜像 并 启动Docker容器**

```json
	docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124430-1297320280.png)


<br>



**3、查看Docker容器是否启动**

```json
	docker ps
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125573-280154900.png)


<br>


**4、再次在浏览器进行访问就可以吃鸡了，不需要再安装插件啊，刚刚上一步拉镜像和启动时已经安装好了**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124881-295576187.png)



<br>
<br>

### 2.1.3、使用Docker-compose安装

采用了第二种方式的话，记得把已经启动的Docker容器关了，以下是附加的一些Docker的基操

<br>

```json
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

# 跟参数的运行
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像ID | 镜像名称
# 如：docker run -d -p 8081:8080 --name tomcat b8
# -d：代表后台运行容器 
# -p 宿主机端口:容器端口：为了映射当前Linux的端口和容器的端口 
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
# 退出容器：exit

# 将宿主机的文件复制到容器内部的指定目录
docker cp 文件名称 容器id:容器内部路径
docker cp index.html 982:/usr/local/tomcat/webapps/ROOT


=====================================================================


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

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123678-288529908.png)



<br>


**1、创建一个文件夹**，这些我很早之前就玩过了，所以建好了的

```json
	# 创建文件夹
	mkdir 文件夹名
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180122384-507194680.png)


<br>


**2、进入文件夹，创建docker-compose.yml文件，注意：文件名必须是这个**

```json
	# 创建文件
	touch docker-compose.yml
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125691-1713496554.png)

<br>


**3、编辑docker-compose.yml文件**

```json
	# 编辑文件
	vim docker-compose.yml
```

里面编写的内容如下，编写好保存即可。注意：别用tab缩进啊，会出问题的，另外：每句的后面别有空格，严格遵循yml格式的

```yml
version: "3.1"
services:
  rabbitmq:
# 镜像
    image: rabbitmq:3.9-management
# 自启
    restart: always
# Docker容器名
    container_name: rabbitmq
# 端口号，docker容器内部端口 映射 外部端口
    ports:
      - 5672:5672
      - 15672:15672
# 数据卷映射 把容器里面的东西映射到容器外面去 容易操作，否则每次都要进入容器
    volumes:
      - ./data:/opt/install/rabbitMQ-docker/
```


<br>



**4、在docker-compose.yml所在路径执行如下命令**，注意：一定要在此文件路径中才行，因为默认是在当前文件夹下找寻docker-compose文件

```json
# 启动
docker-compose up -d
# -d 后台启动

=========================================================

# 附加内容：docker-compose的一些命令操作

# 1. 基于docker-compose.yml启动管理的容器
docker-compose up -d

# 2. 关闭并删除容器
docker-compose down

# 3. 开启|关闭|重启已经存在的由docker-compose维护的容器
docker-compose start|stop|restart

# 4. 查看由docker-compose管理的容器
docker-compose ps 

# 5. 查看日志 
docker-compose logs -f

# 有兴趣的也可以去了解docker-file自定义镜像

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123917-1425970549.png)

<br>


去浏览器访问一样的吃鸡

**上面就是RabbitMQ的基操做完了，不过默认账号是guest游客状态，很多事情还做不了呢，所以还得做一些操作**




<br>
<br>



### 2.1.4、解决不能登入web管理界面的问题

#### 2.1.4.1、使用rpm红帽软件安装的RabbitMQ

这种方式直接使用guest进行登录是不得吃的

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125765-375056583.png)

<br>


- 这是因为guest是游客身份，不能进入，需要添加新用户

```json
	查看当前用户 / 角色有哪些
	rabbitmqctl list_users

	删除用户
	rabbitmqctl delete_user 用户名

	添加用户
	rabbitmqctl add_user 用户名 密码

	设置用户角色
	rabbitmqctl set_user_tags 用户名 administrator

	设置用户权限【 ps：guest角色就是没有这一步 】
	rabbitmqctl set_permissions -p "/" 用户名 ".*" ".*" ".*"
# 设置用户权限指令解释
				set_permissions [-p <vhostpath>] <user> <conf> <write> <read>

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124083-387562532.png)

<br>

- 现在使用admin去浏览器登录就可以了

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124011-767221155.png)



<br>
<br>





#### 2.1.4.2、使用docker 或 docker-compose安装的RabbitMQ

这两种方式直接使用guest就可以进行登录，然后在Web管理界面添加一个新用户就可以了，记得权限选成管理员

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124431-960486601.png)

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123784-1716951022.png)

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124451-1130594298.png)



<br>
<br>



# 3、开始玩RabbitMQ

创建Maven项目 并导入如下依赖

```xml
    <dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.9.0</version>
        </dependency>
    </dependencies>
```


<br>




回到前面的RabbitMQ原理图

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126710-974752633.png)



<br>
<br>

## 3.1、Hello word 简单模式

对照原理图来玩，官网中有Hello word的模式图

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124894-243944497.png)

即：一个生产者Producer、一个默认交换机Exchange、一个队列queue、一个消费者Consumer




<br>



> **生产者**

就是下图前面部分

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124392-1012867010.png)

<br>


```java
package cn.zixieqing;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Producer {

    private static final String HOST = "ip";		// 放RabbitMQ服务的服务器ip
    private static final int PORT = 5672;		// 服务器中RabbitMQ的端口号，在浏览器用的15672是通过5672映射出来的15672
    private static final String USER_NAME = "admin";
    private static final String PASSWORD = "admin";
    private static final String QUEUE_NAME = "hello word";

    public static void main(String[] args) throws IOException, TimeoutException {

        // 1、获取链接工厂
        ConnectionFactory factory = new ConnectionFactory();

        // 2、设置链接信息
        factory.setHost(HOST);
        factory.setPort(PORT);
        factory.setUsername(USER_NAME);
        factory.setPassword(PASSWORD);
        /*
            当然：这里还可以设置vhost虚拟机 - 前提是自己在web管理界面中添加了vhost
            factory.setVirtualHost();
         */

        // 3、获取链接Connection
        Connection connection = factory.newConnection();

        // 4、创建channel信道 - 它才是去和交换机 / 队列打交道的
        Channel channel = connection.createChannel();

        // 5、准备一个队列queue
        // 这里理论上是去和exchange打交道，但是：这里是hello word简单模式，所以直接使用默认的exchange即可
        /*
            下面这是参数的完整意思，源码中偷懒了，没有见名知意
            queueDeclare( queueName,isPersist,isShare,isAutoDelete,properties )
            参数1、队列名字
            参数2、是否持久化( 保存到磁盘 ），默认是在内存中的
            参数3、是否共享，即：是否只供一个消费者消费，是否让多个消费者共享这个队列中的信息
            参数4、是否自动删除，即：最后一个消费者获取信息之后，这个队列是否自动删除
            参数5、其他配置项，这涉及到后面的知识，目前选择null
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println("正在发送信息！！！");
        // 6、推送信息到队列中
        // 准备发送的信息内容
        String message = "it is hello word";
        /*
            basicPublish( exchangeName,queueName,properties,message )
            参数1、交互机名字 - 目前使用了默认的
            参数2、指定路由规则 - 目前使用队列名字
            参数3、指定传递的消息所携带的properties
            参数4、推送的具体消息 - byte类型的
         */
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());

        // 7、释放资源 - 倒着关闭即可
        if ( null != channel ) channel.close();

        if ( null != connection ) connection.close();

        System.out.println("消息发送完毕");

    }
}


```

<br>


运行之后，去浏览器管理界面进行查看

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125335-1513978601.png)





<br>



> **消费者**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124328-2016279574.png)





```java
public class Consumer {

    private static final String HOST = "ip";   // 自己的服务器ip
    private static final int PORT = 5672;
    private static final String USER_NAME = "admin";
    private static final String PASSWORD = "admin";
    private static final String QUEUE_NAME = "hello word";

    public static void main(String[] args) throws IOException, TimeoutException {

        // 1、创建链接工厂
        ConnectionFactory factory = new ConnectionFactory();

        // 2、设置链接信息
        factory.setHost(HOST);
        factory.setPort(PORT);
        factory.setUsername(USER_NAME);
        factory.setPassword(PASSWORD);

        // 3、创建链接对象
        Connection connection = factory.newConnection();

        // 4、创建信道channel
        Channel channel = connection.createChannel();

        // 5、从指定队列中获取消息
        /*
            basicConsume( queueName,isAutoAnswer,deliverCallback,cancelCallback )
            参数1、队列名
            参数2、是否自动应答,为true时，消费者接收到消息后，会立即告诉RabbitMQ
            参数3、消费者如何消费消息的回调
            参数4、消费者取消消费的回调
         */
        System.out.println("开始接收消息！！！");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("接收到了消息：" + new String(message.getBody(), StandardCharsets.UTF_8) );
        };

        CancelCallback cancelCallback = consumerTag -> System.out.println("消费者取消了消费信息行为");

        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);

        // 6、释放资源 - 但是这里不能直接关闭啊，否则：看不到接收的结果的，可以选择不关，也可以选择加一句代码System.in.read();

        // channel.close();
        // connection.close();

    }
}

```



<br>
<br>

## 3.2、work queue工作队列模式

流程图就是官网中的

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123800-1827288841.png)


- 一个生产者批量生产消息
- 一个默认交换机
- 一个队列
- 多个消费者
- 换言之：就是有大量的任务 / 密集型任务有待处理（ 生产者生产的消息 ），此时我们就将这些任务推到队列中去，然后使用多个工作线程（ 消费者 ）来进行处理，否则：一堆任务直接就跑来了，那消费者不得乱套了，因此：这种就需要让这种模式具有如下的特点：
  - 1、消息是有序排好的（ 也就是在队列中 ）
  - 2、工作线程 / 消费者不能同时接收同一个消息，换言之：生产者推送的任务必须是轮询分发的，即：工作线程1接收第一个，工作线程2接收第二个；工作线程1再接收第三个，工作线程2接收第四个


<br>


> **抽取RabbitMQ链接的工具类**

```java
package cn.zixieqing.util;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class MQUtil {

    private static final String HOST = "自己的ip";
    private static final int PORT = 5672;
    private static final String USER_NAME = "admin";
    private static final String PASSWORD = "admin";

    public static Channel getChannel(String vHost ) throws IOException, TimeoutException {

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(HOST);
        factory.setPort(PORT);
        factory.setUsername(USER_NAME);
        factory.setPassword(PASSWORD);
        if ( !vHost.isEmpty() ) factory.setVirtualHost(vHost);

        return factory.newConnection().createChannel();

    }
}

```



<br>




> **生产者**

和hello word没什么两样

<br>

```java
package cn.zixieqing.workqueue;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.util.Scanner;
import java.util.concurrent.TimeoutException;


public class WorkProducer {

    private static final String QUEUE_NAME = "work queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        // 1、声明队列
        /*
            下面这是参数的完整意思，源码中偷懒了，没有见名知意
            queueDeclare( queueName,isPersist,isShare,isAutoDelete,properties )
            参数1、队列名字
            参数2、是否持久化( 保存到磁盘 ），默认是在内存中的
            参数3、是否共享，即：是否只供一个消费者消费，是否让多个消费者共享这个队列中的信息
            参数4、是否自动删除，即：最后一个消费者获取信息之后，这个队列是否自动删除
            参数5、其他配置项，这涉及到后面的知识，目前选择null
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 2、准备消息
        System.out.println("请输入要推送的信息，按回车确认：");
        Scanner input = new Scanner(System.in);

        // 3、推送信息到队列中
        while (input.hasNext()) {
            /*
                basicPublish( exchangeName,routing key,properties,message )
                参数1、交互机名字 - 目前是使用了默认的
                参数2、指定路由规则 - 目前使用队列名字
                参数3、指定传递的消息所携带的properties
                参数4、推送的具体消息 - byte类型的
            */
            String message = input.next();
            channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
            System.out.println("消息====>" + message + "====>推送完毕！");
        }
    }
}

```

<br>


> **消费者**

消费者01

```java
package cn.zixieqing.workqueue;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class WorkConsumer {

    private static final String QUEUE_NAME = "work queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("接收到了消息====>" + new String(message.getBody(), StandardCharsets.UTF_8));
        };
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println( consumerTag + "消费者中断了接收消息====>" );
        };

        System.out.println("消费者01正在接收消息......");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);

    }
}

```

<br>

消费者02

```java
package cn.zixieqing.workqueue;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class WorkConsumer {

    private static final String QUEUE_NAME = "work queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("接收到了消息====>" + new String(message.getBody(), StandardCharsets.UTF_8));
        };
        CancelCallback cancelCallback = consumerTag -> {
            System.out.println( consumerTag + "消费者中断了接收消息====>" );
        };

        System.out.println("消费者02正在接收消息......");
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);

    }
}

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126365-1427356406.png)

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125055-466624779.png)

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125342-1740492801.png)



<br>
<br>



## 3.3、消息应答机制

**消费者在接收到消息并且处理该消息<span style="color:blue">之后</span>，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了**

目的就是为了保证数据的安全，如果没有这个机制的话，那么就会造成下面的情况

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125425-942305354.png)

消费者接收队列中的消息时，没接收完，出现异常了，然后此时MQ以为消费者已经把消息接收并处理了（ MQ并没有接收到消息有没有被消费者处理完毕 ），然后MQ就把队列 / 消息给删了，后续消费者异常恢复之后再次接收消息，就会出现：接收不到了


<br>
<br>

### 3.3.1、消息应答机制的分类

这个东西已经见过了

```java
	/*
            basicConsume( queueName,isAutoAnswer,deliverCallback,cancelCallback )
            参数1、队列名
            参数2、是否自动应答,为true时，消费者接收到消息后，会立即告诉RabbitMQ
            参数3、消费者如何消费消息的回调
            参数4、消费者取消消费的回调
    */
	channel.basicConsume(QUEUE_NAME, true, deliverCallback, cancelCallback);
```



<br>



#### 3.3.1.1、自动应答

**指的是：消息发送后立即被认为已经传送成功**

需要具备的条件：
- 1、发送的消息很多，就是高吞吐量的那种
- 2、发送的消息在传输方面是安全的

优点：处理效率快，很高效

<br>



#### 3.3.1.2、手动应答

就是我们自己去设定，**好处是可以批量应答并且减少网络拥堵**

调用的API如下：
```java
    	Channel.basicACK( long, boolean );		// 用于肯定确认，即：MQ已知道该消息 并且 该消息已经成功被处理了，所以MQ可以将其丢弃了

    	Channel.basicNack( long, boolena, boolean );	// 用于否定确认

    	Channel.basicReject( long, boolea );		// 用于否定确认
    		与Channel.basicNack( long, boolena, boolean )相比，少了一个参数，这个参数名字叫做：multiple
```

**multiple参数说明，它为true和false有着截然不同的意义【 ps：建议弄成false，虽然是挨个去处理，从而应答，效率慢，但是：数据安全，否则：很大可能造成数据丢失 】**

- **true 		代表批量应答MQ，channel 上未应答 / 消费者未被处理完毕的消息**

  ![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123616-1440641152.png)

- **false			只会处理队列放到channel信道中当前正在处理的消息告知MQ是否确认应答 / 消费者处理完毕了**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180128245-1173168805.png)


<br>
<br>


#### 3.3.1.3、消息重新入队原理

**指的是：如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确保不会丢失任何消息**

如下图：消息1原本是C1这个消费者来接收的，但是C1失去链接了，而C2消费者并没有断开链接，所以：最后MQ将消息重新入队queue，然后让C2来处理消息1

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125820-505991279.png)


<br>
<br>

#### 3.3.1.4、手动应答的代码演示

> **生产者**

```java
package cn.zixieqing.ACK;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;
import java.util.concurrent.TimeoutException;


public class AckProducer {

    private static final String QUEUE_NAME = "ack queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        // 声明队列
        /*
            下面这是参数的完整意思，源码中偷懒了，没有见名知意
            queueDeclare( queueName,isPersist,isShare,isAutoDelete,properties )
            参数1、队列名字
            参数2、是否持久化( 保存到磁盘 ），默认是在内存中的
            参数3、是否共享，即：是否只供一个消费者消费，是否让多个消费者共享这个队列中的信息
            参数4、是否自动删除，即：最后一个消费者获取信息之后，这个队列是否自动删除
            参数5、其他配置项，这涉及到后面的知识，目前选择null
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        System.out.println("请输入要推送的消息：");
        Scanner input = new Scanner(System.in);
        while (input.hasNext()) {
            /*
                basicPublish( exchangeName,routing key,properties,message )
                参数1、交互机名字 - 使用了默认的
                参数2、指定路由规则，使用队列名字
                参数3、指定传递的消息所携带的properties
                参数4、推送的具体消息 - byte类型的
         */
            String message = input.next();
            channel.basicPublish("",QUEUE_NAME,null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("消息====>" + message + "推送完毕");
        }
    }
}

```

<br>


> **消费者01**

```java
package cn.zixieqing.ACK;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.concurrent.TimeoutException;


public class AckConsumer {

    private static final String QUEUE_NAME = "ack queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            try {

                Thread.sleep(5*1000);

                System.out.println("接收到了消息=====>" + new String( message.getBody(), StandardCharsets.UTF_8 ));

                // 添加手动应答
                /*
                    basicAck( long, boolean )
                    参数1、消息的标识tag，这个标识就相当于是消息的ID
                    参数2、是否批量应答multiple
                 */
                channel.basicAck(message.getEnvelope().getDeliveryTag(),false);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        System.out.println("消费者01正在接收消息，需要5秒处理完");
        channel.basicConsume(QUEUE_NAME, false, deliverCallback, consumerTag -> {
            System.out.println("触发消费者取消消费消息行为的回调");
            System.out.println(Arrays.toString(consumerTag.getBytes(StandardCharsets.UTF_8)));
        });
    }
}

```

<br>


> **消费者02**

```java

package cn.zixieqing.ACK;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.concurrent.TimeoutException;


public class AckConsumer {

    private static final String QUEUE_NAME = "ack queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            try {

                Thread.sleep(10*1000);

                System.out.println("接收到了消息=====>" + new String( message.getBody(), StandardCharsets.UTF_8 ));

                // 添加手动应答
                /*
                    basicAck( long, boolean )
                    参数1、消息的标识tag，这个标识就相当于是消息的ID
                    参数2、是否批量应答multiple
                 */
                channel.basicAck(message.getEnvelope().getDeliveryTag(),false);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        };
        System.out.println("消费者02正在接收消息，需要10秒处理完");
        channel.basicConsume(QUEUE_NAME, false, deliverCallback, consumerTag -> {
            System.out.println("触发消费者取消消费消息行为的回调");
            System.out.println(Arrays.toString(consumerTag.getBytes(StandardCharsets.UTF_8)));
        });
    }
}

```



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126861-245275661.png)



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126384-1640418516.png)



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125565-589326507.png)


<br>
<br>


### 3.4、RabbitMQ的持久化 durable

### 3.4.1、队列持久化

这个玩意儿的配置吧，早就见过了，在生产者消息发送时，有一个声明队列的过程，那里面就有一个是否持久化的配置

```java
		/*
            下面这是参数的完整意思，源码中偷懒了，没有见名知意
            queueDeclare( queueName,isPersist,isShare,isAutoDelete,properties )
            参数1、队列名字
            参数2、是否持久化( 保存到磁盘 ），默认是在内存中的
            参数3、是否共享，即：是否只供一个消费者消费，是否让多个消费者共享这个队列中的信息
            参数4、是否自动删除，即：最后一个消费者获取信息之后，这个队列是否自动删除
            参数5、其他配置项，这涉及到后面的知识，目前选择null
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

```

而如果没有持久化，那么RabbitMQ服务由于其他什么原因导致挂彩的时候，那么重启之后，这个没有持久化的队列就灰飞烟灭了**【 ps：注意和里面的消息还没关系啊，不是说队列持久化了，那么消息就持久化了 】**

在这个队列持久化配置中，它的默认值就是false，所以要改成true时，需要注意一个点：**选择队列持久化，那么必须保证当前这个队列是新的，即：RabbitMQ中没有当前队列，否则：需要进到web管理界面把已有的同名队列删了，然后重新配置当前队列持久化选项为true，不然：报错**



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124510-1687574940.png)

那么：当我把持久化选项改为true，并 重新发送消息时

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180127307-698274628.png)

`inequivalent arg 'durable' for queue 'queue durable' in vhost '/': received 'true' but current is 'false'`

告知你：vhost虚拟机中已经有了这个叫做durable的队列，要接收的选项值是true，但是它当前的值是false，所以报错了呗

解决方式就是去web管理界面，把已有的durable队列删了，重新执行

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125588-1656303734.png)

再次执行就可以吃鸡了，同时去web管理界面会发现它状态变了，多了一个D标识

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124310-1720754989.png)

有了这个玩意儿之后，那么就算RabbitMQ出问题了，后续恢复之后，那么这个队列也不会丢失



<br>
<br>


### 3.4.2、消息持久化

**注意：这里说的消息持久化不是说配置之后消息就一定不会丢失，而是：把消息标记为持久化，然后RabbitMQ尽量让其持久化到磁盘**

但是：也会有意外，比如：RabbitMQ在将消息持久化到磁盘时，这是有一个时间间隔的，数据还没完全刷写到磁盘呢，RabbitMQ万一出问题了，那么消息 / 数据还是会丢失的，所以：**消息持久化配置是一个弱持久化，但是：对于简单队列模式完全足够了**，强持久化的实现方式在后续的publisher / confirm发布确认模式中


至于配置极其地简单，在前面都已经见过这个配置项，就是生产者发消息时做文章，就是下面的第三个参数，把它改为`MessageProperties.PERSISTENT_TEXT_PLAIN`即可

```java
		 /*
            basicPublish( exchangeName,routing key,properties,message )
            参数1、交互机名字 - 使用了默认的
            参数2、指定路由规则，使用队列名字
            参数3、指定传递的消息所携带的properties
            参数4、推送的具体消息 - byte类型的
         */
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());

		// 改成消息持久化
        channel.basicPublish("",QUEUE_NAME,MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes());

```

**MessageProperties类的源码如下：**

```java
public class MessageProperties {

    public static final BasicProperties MINIMAL_BASIC = new BasicProperties((String)null, (String)null, (Map)null, (Integer)null, (Integer)null, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);

    public static final BasicProperties MINIMAL_PERSISTENT_BASIC = new BasicProperties((String)null, (String)null, (Map)null, 2, (Integer)null, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);

    public static final BasicProperties BASIC = new BasicProperties("application/octet-stream", (String)null, (Map)null, 1, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);

    public static final BasicProperties PERSISTENT_BASIC = new BasicProperties("application/octet-stream", (String)null, (Map)null, 2, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);

    public static final BasicProperties TEXT_PLAIN = new BasicProperties("text/plain", (String)null, (Map)null, 1, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);

    public static final BasicProperties PERSISTENT_TEXT_PLAIN = new BasicProperties("text/plain", (String)null, (Map)null, 2, 0, (String)null, (String)null, (String)null, (String)null, (Date)null, (String)null, (String)null, (String)null, (String)null);

    public MessageProperties() {
    }
}

```

上面用到了BasicProperties类型，它的属性如下：

```java
    public static class BasicProperties extends AMQBasicProperties {
        // 消息内容的类型
        private String contentType;
        // 消息内容的编码格式
        private String contentEncoding;
        // 消息的header
        private Map<String, Object> headers;
        // 消息是否持久化，1：否，2：是
        private Integer deliveryMode;
        // 消息的优先级
        private Integer priority;
        // 关联ID
        private String correlationId;
        // :用于指定回复的队列的名称
        private String replyTo;
        // 消息的失效时间
        private String expiration;
        // 消息ID
        private String messageId;
        // 消息的发送时间
        private Date timestamp;
        // 类型
        private String type;
        // 用户ID
        private String userId;
        // 应用程序ID
        private String appId;
        // 集群ID
        private String clusterId;
   }
```



<br>
<br>



### 3.5、不公平分发 和 预取值

> **不公平分发**

- **这个东西是在消费者那一方进行设置的**
- **RabbitMQ默认是公平分发，即：轮询分发**
- 轮询分发有缺点：如前面消费者01（ 设5秒的那个 ）和 消费者02 （ 设10秒的那个 ），这种情况如果采用轮询分发，那么：01要快一点，而02要慢一点，所以01很快处理完了，然后处于空闲状态，而02还在拼命奋斗中，最后的结果就是02不停干，而01悠悠闲闲的，浪费了时间，所以：应该压榨一下01，让它不能停
- 设置方式：在消费者接收消息之前进行`channel.basicQos（ int prefetchCount ）设置`

```java
		// 不公平分发，就是在这里接收消息之前做处理
        /* 
            basicQos( int prefetchCount )
            为0、轮询分发 也是RabbitMQ的默认值
            为1、不公平分发
         */
        channel.basicQos(1);

        channel.basicConsume("qos queue", true, deliverCallback, consumerTag -> {
            System.out.println("消费者中断了接收消息行为触发的回调");
        });

```

<br>


> **预取值**

- 指的是：多个消费者在消费消息时，让每一个消费者预计消费多少条消息

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125374-1665297514.png)

- **而要设置这种效果，和前面不公平分发的设置是一样的，只是把里面的参数改一下即可**

```java

		// 预取值，也是在这里接收消息之前做处理，和不公平分发调的是同一个API
        /* 
            basicQos( int prefetchCount )	为0、轮询分发 也是RabbitMQ的默认值;为1、不公平分发
            而当这里的数字变成其他的，如：上图中上面的那个消费者要消费20条消息，那么把下面的数字改成对应的即可
            注意点：这是要设置哪个消费者的预取值，那就是在哪个消费者代码中进行设定啊
         */
        channel.basicQos(10);		// 这样就表示这个代码所在的消费者需要消费10条消息了

        channel.basicConsume("qos queue", true, deliverCallback, consumerTag -> {
            System.out.println("消费者中断了接收消息行为触发的回调");
        });

```


<br>
<br>



## 3.6、publisher / confirms 发布确认模式

### 3.6.1、发布确认模式的原理

**这个玩意儿的目的就是为了持久化**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124206-189359078.png)

- **在上面的过程中，想要让数据持久化，那么需要具备以下的条件**
  - 1、队列持久化
  - 2、消息持久化
  - 3、发布确认

- **而所谓的发布确认指的就是：数据在刷写到磁盘时，成功了，那么MQ就回复生产者一下，数据确认刷写到磁盘了**，否则：只具备前面的二者的话，那也有可能出问题，如：数据推到了队列中，但是还没来得及刷写到磁盘呢，结果RabbitMQ宕机了，那数据也有可能会丢失,所以：现在持久化的过程就是如下的样子：

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124468-268095635.png)



<br>


> **开启发布确认**

- **在发送消息之前（ 即：调basicPublish（） 之前 ）调一个API就可以了**

```java
	channel.confirmSelect();		// 没有参数
```

<br>
<br>

### 3.6.2、发布确认的分类

#### 3.6.2.1、单个确认发布

**一句话：一手交钱一手交货**，即 生产者发布一条消息，RabbitMQ就要回复确认状态，否则不再发放消息，**因此：这种模式是同步发布确认的方式，缺点：很慢，优点：能够实时地了解到那条消息出异常 / 哪些消息都发布成功了**

```java
    public static void main(String[] args) throws InterruptedException, TimeoutException, IOException {

        // 单个确认发布
        singleConfirm();        // 单个确认发布发送这些消息花费4797ms
    }

	public static void singleConfirm() throws IOException, TimeoutException, InterruptedException {

        Channel channel = MQUtil.getChannel("");

        // 开启确认发布
        channel.confirmSelect();

        // 声明队列 并 让队列持久化
        channel.queueDeclare("singleConfirm", true, false, false, null);

        long begin = System.currentTimeMillis();

        for (int i = 1; i <= 100; i++) {

            // 发送消息 并 让消息持久化
            channel.basicPublish("","singleConfirm", MessageProperties.PERSISTENT_TEXT_PLAIN,String.valueOf(i).getBytes() );

            // 发布一个 确认一个 channel.waitForConfirms()
            if ( channel.waitForConfirms() )
                System.out.println("消息".concat( String.valueOf(i) ).concat( "发送成功") );

        }

        long end = System.currentTimeMillis();

        System.out.println("单个确认发布发送这些消息花费".concat( String.valueOf( end-begin ) ).concat("ms") );
    }

```

<br>
<br>

#### 3.6.2.2、批量确认发布

**一句话：只要结果**，是怎么一个批量管不着，只需要把一堆消息发布之后，回复一个结果即可，**这种发布也是同步的**

**优点：**效率相比单个发布要高

**缺点：**如果因为什么系统故障而导致发布消息出现问题，那么就会导致是批量发了一些消息，然后再回复的，中间有哪个消息出问题了鬼知道

```java
    public static void main(String[] args) throws InterruptedException, TimeoutException, IOException {
        // 单个确认发布
        // singleConfirm();        // 单个确认发布发送这些消息花费4797ms

        // 批量发布
        batchConfirm();         // 批量发布发送的消息共耗时：456ms

    }

    public static void batchConfirm() throws IOException, TimeoutException, InterruptedException {

        Channel channel = MQUtil.getChannel("");

        // 开启确认发布
        channel.confirmSelect();

        // 声明队列 并 让队列持久化
        channel.queueDeclare("batchConfirm", true, false, false, null);

        long begin = System.currentTimeMillis();

        for (int i = 1; i <= 100; i++) {

            // 发送消息 并 让消息持久化
            channel.basicPublish("","batchConfirm", MessageProperties.PERSISTENT_TEXT_PLAIN,String.valueOf(i).getBytes() );

            // 批量发布 并 回复批量发布的结果 - 发了10条之后再确认
            if (i % 10 == 0) {

                channel.waitForConfirms();
                System.out.println("消息" + ( i-10 ) + "====>" + i + "的消息发布成功");
            }
        }

        // 为了以防还有另外的消息未被确认，再次确认一下
        channel.waitForConfirms();

        long end = System.currentTimeMillis();

        System.out.println("批量发布发送的消息共耗时：" + (end - begin) + "ms");

    }

```

<br>
<br>

#### 3.6.2.3、异步确认发布 - 必会

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124736-1817150434.png)

- **由上图可知：所谓的异步确认发布就是：**
  - 1、生产者只管发消息就行，不用管消息有没有成功
  - 2、发布的消息是存在一个map集合中的，其key就是消息的标识tag / id，value就是消息内容
  - 3、如果消息成功发布了，那么实体broker会有一个ackCallback()回调函数来进行处理【 ps：里面的处理逻辑是需要我们进行设计的 】
  - 4、如果消息未成功发布，那么实体broker会调用一个nackCallback()回调函数来进行处理【 ps：里面的处理逻辑是需要我们进行设计的 】
  - 5、而需要异步处理，就是因为生产者只管发就行了，因此：一轮的消息肯定是很快就发布过去了，就可以做下一轮的事情了，至于上一轮的结果是怎么样的，那就需要等到两个callback回调执行完了之后给结果，而想要能够调取到两个callback回调，那么：就需要对发送的信息进行监听 / 对信道进行监听
- **而上述牵扯到一个map集合，那么这个集合需要具备如下的条件：**
  - 1、首先此集合应是一个安全且有序的，同时还支持高并发
  - 2、其次能够将序列号( key ) 和 消息( value )轻松地进行关联





> **代码实现**

```java
    public static void main(String[] args) throws InterruptedException, TimeoutException, IOException {
        // 单个确认发布
        // singleConfirm();        // 单个确认发布发送这些消息花费4797ms

        // 批量发布
        // batchConfirm();         // 批量发布发送的消息共耗时：456ms

        asyncConfirm();             // 异步发布确认耗时：10ms

    }

    // 异步发布确认
    public static void asyncConfirm() throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");
        channel.confirmSelect();
        channel.queueDeclare("async confirm", true, false, false, null);

        // 1、准备符合条件的map
        ConcurrentSkipListMap<Long, Object> messagePoolMap = new ConcurrentSkipListMap<>();

        // 3、对信道channel进行监听
        // 成功确认发布回调
        ConfirmCallback ackCallback = (messageTag, multiple) -> {
            System.out.println("确认发布了消息=====>" + messagePoolMap.headMap(messageTag) );

            // 4、把确认发布的消息删掉，减少内存开销
            // 判断是否是批量删除
            if ( multiple ){
                // 通过消息标识tag 把 确认发布的消息取出
                messagePoolMap.headMap(messageTag).clear();
                /**
                 * 上面这句代码拆分写法
                 *    ConcurrentNavigableMap<Long, Object> confirmed = messagePoolMap.headMap(messageTag);
                 *    confirmed.clear();
                 */
            }else {
                messagePoolMap.remove(messageTag);
            }
        };

        // 没成功发布确认回调
        ConfirmCallback nackCallback = (messageTag, multiple) -> {
            System.out.println("未确认的消息是：" + messagePoolMap.get(messageTag) );
        };

        // 进行channel监听 这是异步的
        /**
         * channel.addConfirmListener(ConfirmCallback var1, ConfirmCallback var2)
         * 参数1、消息成功发布的回调函数 ackCallback()
         * 参数2、消息未成功发布的回调函数 nackCallback()
         */
        channel.addConfirmListener( ackCallback,nackCallback );

        long begin = System.currentTimeMillis();

        for (int i = 1; i <= 100; i++) {

            // 2、将要发布的全部信息保存到map中去
            /*
                channel.getNextPublishSeqNo() 获取下一次将要发送的消息标识tag
             */
            messagePoolMap.put(channel.getNextPublishSeqNo(),String.valueOf(i) );
            // 生产者只管发布就行
            channel.basicPublish("","async confirm",MessageProperties.PERSISTENT_TEXT_PLAIN,String.valueOf(i).getBytes());

            System.out.println("消息=====>" + i + "发送完毕");
        }


        long end = System.currentTimeMillis();

        System.out.println("异步发布确认耗时：" + ( end-begin ) + "ms" );
    }

```


<br>
<br>




## 3.7、交换机

正如前面一开始就画的原理图，**交换机的作用就是为了接收生产者发送的消息 并 将消息发送到队列中去**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126710-974752633.png)

**注意点：前面一直玩的那些模式，虽然没有写交换机，但并不是说RabbitMQ就没用交换机【 ps：使用的是""空串，也就是使用了RabbitMQ的默认交换机 】，生产者发送的消息只能发到交换机中，从而由交换机来把消息发给队列**


<br>
<br>


### 3.7.1、交换机exchange的分类

- **直接( direct ) / routing 模式**
- **主题( topic )**
- **标题 ( heanders )** - 这个已经很少用了
- **扇出( fancut ) / 发布订阅模式**

<br>

> **临时队列**

**所谓的临时队列指的就是：自动帮我们生成队列名 并且 当生产者和队列断开之后，这个队列会被自动删除**

所以这么一说：前面玩过的一种就属于临时队列，即：将下面的第四个参数改成true即可【 ps：当然让队列名随机生成就完全匹配了 】

```java
		/*
            下面这是参数的完整意思，源码中偷懒了，没有见名知意
            queueDeclare( queueName,isPersist,isShare,isAutoDelete,properties )
            参数1、队列名字
            参数2、是否持久化( 保存到磁盘 ），默认是在内存中的
            参数3、是否共享，即：是否只供一个消费者消费，是否让多个消费者共享这个队列中的信息
            参数4、是否自动删除，即：最后一个消费者获取信息之后，这个队列是否自动删除
            参数5、其他配置项，这涉及到后面的知识，目前选择null
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

```


**而如果要更简单的生成临时队列，那么调用如下的API即可**

```java
        String queueName = channel.queueDeclare().getQueue();

```

这样帮我们生成的队列效果就和`channel.queueDeclare(QUEUE_NAME, false, false, true, null);`是一样的了

<br>
<br>


### 3.7.2、fanout扇出 / 发布订阅模式

**这玩意儿吧，好比群发，一人发，很多人收到消息**，就是原理图的另一种样子，生产者发布的一个消息，可以供多个消费者进行消费

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125760-80360966.png)

实现方式就是让一个交换机binding绑定多个队列

<br>


> **生产者**

```java
package cn.zixieqing.fanout;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;
import java.util.concurrent.TimeoutException;


public class FanoutProducer {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        /**
         * 定义交换机
         * 参数1、交换机名字
         * 参数2、交换机类型
         */
        channel.exchangeDeclare("fanoutExchange", BuiltinExchangeType.FANOUT);

        System.out.println("请输入要发送的内容：");
        Scanner input = new Scanner(System.in);
        while (input.hasNext()){
            String message = input.next();
            channel.basicPublish("fanoutExchange","", null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("消息=====>" + message + "发送完毕");
        }
    }
}

```

<br>


> **消费者01**

```java
package cn.zixieqing.fanout;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class FanoutConsumer01 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        // 绑定队列
        /**
         * 参数1、队列名字
         * 参数2、交换机名字
         * 参数3、用于绑定的routing key / binding key
         */
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, "fanoutExchange", "");

        System.out.println("01消费者正在接收消息........");
        channel.basicConsume(queueName,true,(consumerTag,message)->{
            // 这里面接收到消息之后就可以用来做其他事情了，如：存到磁盘
            System.out.println("接收到了消息====>" + new String( message.getBody(), StandardCharsets.UTF_8));
        },consumerTage->{});
    }
}

```

<br>


> **消费者02**

```java
package cn.zixieqing.fanout;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class FanoutConsumer02 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        // 绑定队列
        /**
         * 参数1、队列名字
         * 参数2、交换机名字
         * 参数3、用于绑定的routing key / binding key
         */
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName, "fanoutExchange", "");

        System.out.println("02消费者正在接收消息........");
        channel.basicConsume(queueName,true,(consumerTag,message)->{
            // 这里面接收到消息之后就可以用来做其他事情了，如：存到磁盘
            System.out.println("接收到了消息====>" + new String( message.getBody(), StandardCharsets.UTF_8));
        },consumerTage->{});
    }
}

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126160-490313496.png)


<br>
<br>


### 3.7.3、direct交换机 / routing路由模式

这个玩意儿吧就是发布订阅模式，也就是fanout类型交换机的变样板，即：多了一个routing key的配置而已，**也就是说：生产者和消费者传输消息就通过routing key进行关联起来**，**因此：现在就变成了生产者想把消息发给谁就发给谁**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124074-379528733.png)


<br>



> **生产者**

```java
package cn.zixieqing.direct;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.Scanner;
import java.util.concurrent.TimeoutException;


public class DirectProducer {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        channel.exchangeDeclare("directExchange", BuiltinExchangeType.DIRECT);

        System.out.println("请输入要发送的消息：");
        Scanner input = new Scanner(System.in);

        while (input.hasNext()){
            String message = input.next();
            /**
             * 对第二个参数routing key做文章
             * 假如这里的routing key为zixieqing 那么：就意味着消费者只能是绑定了zixieqing的队列才可以进行接收这里发的消息内容
             */
            channel.basicPublish("directExchange","zixieqing",null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("消息=====>" + message + "====>发送完毕");
        }
    }
}

```

<br>


> **消费者01**

```java
package cn.zixieqing.direct;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class DirectConsumer01 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        channel.queueDeclare("direct", false, false, false, null);
        /**
         * 队列绑定
         * 参数1、队列名
         * 参数2、交换机名字
         * 参数3、routing key 这里的routing key 就需要和生产者中的一样了，这样才可以通过这个routing key去对应的队列中取消息
         */
        channel.queueBind("direct", "directExchange", "zixieqing");

        System.out.println("01消费者正在接收消息.......");
        channel.basicConsume("direct",true,(consumerTag,message)->{
            System.out.println("01消费者接收到了消息====>" + new String( message.getBody(), StandardCharsets.UTF_8));
        },consumerTag->{});
    }
}

```

<br>


上面这种，生产者的消息肯定能够被01消费者给消费，因为：他们的交换机名字、队列名字和routing key的值都是相同的

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126585-1201473819.png)

<br>


而此时再加一个消费者，让它的routing key值和生产者中的不同

```java
package cn.zixieqing.direct;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class DirectConsumer02 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        channel.queueDeclare("direct", false, false, false, null);
        /**
         * 队列绑定
         * 参数1、队列名
         * 参数2、交换机名字
         * 参数3、routing key 这里的routing key 就需要和生产者中的一样了，这样才可以通过这个routing key去对应的队列中取消息
         */
        // 搞点事情：这里的routing key的值zixieqing和生产者的不同
        channel.queueBind("direct", "directExchange", "xiegongzi");

        System.out.println("02消费者正在接收消息.......");
        channel.basicConsume("direct",true,(consumerTag,message)->{
            System.out.println("02消费者接收到了消息====>" + new String( message.getBody(), StandardCharsets.UTF_8));
        },consumerTag->{});
    }
}

```



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180128092-251022851.png)




<br>
<br>


###  3.7.4、topic交换机 / topic主题模式 - 使用最广的一个

前面玩的fanout扇出类型的交换机 / 发布订阅模式是一个生产者发布，多个消费者共享消息，和qq群类似；而direct直接交换机 / 路由模式是消费者只能消费和消费者相同routing key的消息

而上述这两种还有局限性，如：现在生产者的routing key为zi.xie.qing，而一个消费者只消费含xie的消息，一个消费者只消费含qing的消息，另一个消费者只消费第一个为zi的零个或无数个单词的消息，甚至还有一个消费者只消费最后一个单词为qing，前面有三个单词的routing key的消息呢？

这样一看，发布订阅模式和路由模式都不能解决，更别说前面玩的简单模式、工作队列模式、发布确认模式了，这些和目前的这个需求更不搭了，因此：就来了这个topic主题模式


<br>


> **topic中routing key的要求**

只要交换机类型是topic类型的，那么其routing key就不能乱写，**要求：routing key只能是一个单词列表，多个单词之间采用点隔开，如：cn.zixieqing.rabbit**
- **单词列表的长度不能超过255个字节**

<br>


- **在routing key的规则列表中有两个替换符可以用**
  - **1、`*` 代表一个单词**
  - **2、`#` 代表零活无数个单词**


<br>



假如有如下的一个绑定关系图

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125079-989961348.png)

- Q1绑定的是：中间带 orange 带 3 个单词的字符串(*.orange.*)

- Q2绑定的是：
  - 最后一个单词是 rabbit 的 3 个单词(*.*.rabbit)
  - 第一个单词是 lazy 的多个单词(lazy.#)

- **熟悉一下这种绑定关系（ 左为一些routes路由规则，右为能匹配到上图绑定关系的结果 ）**

<br>

```json
	quick.orange.rabbit 		被队列 Q1Q2 接收到
	lazy.orange.elephant 		被队列 Q1Q2 接收到
	quick.orange.fox 			被队列 Q1 接收到
	lazy.brown.fox 				被队列 Q2 接收到
	lazy.pink.rabbit 			虽然满足两个绑定，但只被队列 Q2 接收一次
	quick.brown.fox 			不满足任何绑定关系，不会被任何队列接收到，会被丢弃
	quick.orange.male.rabbit 	是四个单词，不满足任何绑定关系，会被丢弃
	lazy.orange.male.rabbit 	虽是四个单词，但匹配 Q2，因：符合lazy.#这个规则

```

- **当队列绑定关系是下列这种情况时需要引起注意**
  - **当一个队列绑定键是#，那么这个队列将接收所有数据，就有点像 fanout 了**
  - **如果队列绑定键当中没有#和*出现，那么该队列绑定类型就是 direct 了**


<br>



> **把上面的绑定关系和测试转换成代码玩一波**
>
> **生产者**

```java
package cn.zixieqing.topic;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeoutException;


public class TopicProducer {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        channel.exchangeDeclare("topicExchange", BuiltinExchangeType.TOPIC);

        /**
         * 准备大量的routing key 和 message
         */
        HashMap<String, String> routesAndMessageMap = new HashMap<>();
        routesAndMessageMap.put("quick.orange.rabbit", "被队列 Q1Q2 接收到");
        routesAndMessageMap.put("lazy.orange.elephant", "被队列 Q1Q2 接收到");
        routesAndMessageMap.put("quick.orange.fox", "被队列 Q1 接收到");
        routesAndMessageMap.put("lazy.brown.fox", "被队列 Q2 接收到");
        routesAndMessageMap.put("lazy.pink.rabbit", "虽然满足两个绑定，但只被队列 Q2 接收一次");
        routesAndMessageMap.put("quick.brown.fox", "不满足任何绑定关系，不会被任何队列接收到，会被丢弃");
        routesAndMessageMap.put("quick.orange.male.rabbit", "是四个单词，不满足任何绑定关系，会被丢弃");
        routesAndMessageMap.put("lazy.orange.male.rabbit ", "虽是四个单词，但匹配 Q2，因：符合lazy.#这个规则");

        System.out.println("生产者正在发送消息.......");
        for (Map.Entry<String, String> routesAndMessageEntry : routesAndMessageMap.entrySet()) {
            String routingKey = routesAndMessageEntry.getKey();
            String message = routesAndMessageEntry.getValue();
            channel.basicPublish("topicExchange",routingKey,null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("消息====>" + message + "===>发送完毕");
        }
    }
}

```


<br>

> **消费者01**

```java
package cn.zixieqing.topic;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class TopicConsumer01 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");
        channel.exchangeDeclare("topicExchange", BuiltinExchangeType.TOPIC);
        channel.queueDeclare("Q1", false, false, false, null);
        channel.queueBind("Q1", "topicExchange", "*.orange.*");

        System.out.println("消费者01正在接收消息......");
        channel.basicConsume("Q1",true,(consumerTage,message)->{
            System.out.println("01消费者接收到了消息====>" + new String( message.getBody(), StandardCharsets.UTF_8));
            System.out.println("此条消息的交换机名为：" + message.getEnvelope().getExchange() + "，路由键为：" + message.getEnvelope().getRoutingKey());
        },consumerTag->{});
    }
}

```

<br>

> **消费者02**

```java
package cn.zixieqing.topic;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class TopicConsumer02 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");
        channel.exchangeDeclare("topicExchange", BuiltinExchangeType.TOPIC);
        channel.queueDeclare("Q2", false, false, false, null);
        channel.queueBind("Q2", "topicExchange", "*.*.rabbit");
        channel.queueBind("Q2", "topicExchange", "lazy.#");

        System.out.println("消费者02正在接收消息......");
        channel.basicConsume("Q2",true,(consumerTage,message)->{
            System.out.println("02消费者接收到了消息====>" + new String( message.getBody(), StandardCharsets.UTF_8));
            System.out.println("此条消息的交换机名为：" + message.getEnvelope().getExchange() + "，路由键为：" + message.getEnvelope().getRoutingKey());
        },consumerTag->{});
    }
}

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180127092-606386941.png)



<br>
<br>


## 3.8、死信队列

**死信队列指的是：死了的消息，换言之就是：生产者把消息发送到交换机中，再由交换机把消息推到队列中，但由于某些原因，队列中的消息没有被正常消费，从而就让这些消息变成了死信，而专门用来放这种消息的队列就是死信队列**


<br>



- **让消息成为死信的三大因素**
  - **1、消息过期 即：TTL( time to live )过期**
  - **2、超过队列长度**
  - **3、消息被消费者绝收了**

<br>


实现下图的逻辑（ **下图成为死信的因素是只要出现一个就成为死信**  ）

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124509-1290303022.png)


<br>
<br>

### 3.8.1、消息过期 TTL

> **生产者**

```java
package cn.zixieqing.dead_letter_queue.ttl;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class TtlProducer {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");
        channel.exchangeDeclare("normal_exchange", BuiltinExchangeType.DIRECT);

        // 设置消息的失效时间
        AMQP.BasicProperties properties = new AMQP.BasicProperties()
                .builder()
                // 10s过期 expiration( String time ) 这里的单位是ms值
                .expiration(String.valueOf(10 * 1000))
                .build();
        for (int i = 1; i < 11; i++) {

            String message = "生产者发送了消息" + i;

            channel.basicPublish("normal_exchange","zhangsan",properties,message.getBytes(StandardCharsets.UTF_8));
        }
    }
}

```

<br>


> **实现下面**的消费者部分

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125446-2046727187.png)

<br>


> **C1消费者**

```java
package cn.zixieqing.dead_letter_queue.ttl;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeoutException;


public class TtlConsumer01 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        // 声明正常交换机、死信交换机
        channel.exchangeDeclare("normal_exchange", BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare("dead_exchange", BuiltinExchangeType.DIRECT);

        // 声明死信队列
        channel.queueDeclare("dead-queue", false, false, false, null);
        // 死信队列绑定死信交换机
        channel.queueBind("dead-queue", "dead_exchange", "lisi");

        // 声明正常队列
        /**
         * 但是：需要考虑消息过期之后，转到死信队列去，所以：用最后一个参数做文章
         */
        Map<String, Object> params = new HashMap<>();
        // 消息过期 那需要找到死信交换机 - 因此：让正常队列和死信交换机联系起来，其中key值x-dead-letter-exchange是固定的
        params.put("x-dead-letter-exchange", "dead_exchange");
        // 知道了交换机，那还需要知道routing key路由键，其中：key值x-dead-letter-routing-key也是死的
        params.put("x-dead-letter-routing-key", "lisi");
        // 经过上面的参数配置之后，只要TTL过期，那么消息会跑到上面定义的dead_exchange，然后推到dead-queue中去
        channel.queueDeclare("normal-queue", false, false, false, params);

        // 让正常队列和正常交换机进行绑定
        channel.queueBind("normal-queue", "normal_exchange", "zhangsan");

        // 消费消息
        System.out.println("消费者01正在接收消息.......");
        channel.basicConsume("normal-queue",true,(consumeTage,message)->{
            System.out.println("01消费者从正常队列中消费了消息====>" + new String( message.getBody(), StandardCharsets.UTF_8 ) );
        },consumeTage->{});
    }
}

```

<br>


启动C1，然后把C1关了（ 伪装成消费者无法消费消息 ），最后启动生产者

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124186-980115030.png)


<br>



> **现在来收个尾**
>
> **C2消费者来消费死信队列中的消息 - 就是一个正常的消费者消费，只是跑到死信队列中去找了而已**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125686-2032920427.png)



```java
package cn.zixieqing.dead_letter_queue.ttl;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class TtlConsumer02 {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");
        channel.exchangeDeclare("dead_exchange", BuiltinExchangeType.DIRECT);

        System.out.println("02消费者正在消费死信队列中的消息.......");
        channel.basicConsume("dead-queue",true,(consumeTage,message)->{
            System.out.println("02消费者接收到了死信队列中的===>" + new String(message.getBody(), StandardCharsets.UTF_8));
        },consumeTage->{});
    }
}

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125384-1747449358.png)



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124844-693665666.png)


<br>
<br>

### 3.8.2、队列超过最大长度

#### 3.8.2.1、队列超过所限制的最大个数

**意思就是：某一个队列要求只能放N个消息，但是放了N+1个消息，这就超过队列的最大个数了**

<br>


> **生产者**

- 就是一个正常的生产者发送消息而已

```java
package cn.zixieqing.dead_letter_queue.queuelength.queuenumber;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class Producer {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        channel.exchangeDeclare("messageNumber_normal_exchange", BuiltinExchangeType.DIRECT);

        for (int i = 1; i < 11; i++) {
            String message = "生产者发送了消息" + i;
            channel.basicPublish("messageNumber_normal_exchange","zi",null,
                    message.getBytes(StandardCharsets.UTF_8) );
            System.out.println("消息====>" + message + "====>发送完毕");
        }
    }
}

```


<br>



> **01消费者**

```java
package cn.zixieqing.dead_letter_queue.queuelength.queuenumber;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.concurrent.TimeoutException;


public class Consumer01 {

    /**
     * 正常交换机名称
     */
    public static final String NORMAL_EXCHANGE = "messageNumber_normal_exchange";

    /**
     * 正常队列名称
     */
    public static final String NORMAL_QUEUE = "messageNumber_queue";

    /**
     * 死信交换机名称
     */
    public static final String DEAD_EXCHANGE = "messageNumber_dead_exchange";

    /**
     * 死信队列名称
     */
    public static final String DEAD_QUEUE = "messageNumber_dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        // 声明正常交换机、死信交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 声明死信队列
        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);
        // 死信交换机和死信队列进行绑定
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "xie");

        // 声明正常队列 并 考虑达到条件时和死信交换机进行联系
        HashMap<String, Object> params = new HashMap<>();
        // 死信交换机
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        // 死信路由键
        params.put("x-dead-letter-routing-key", "xie");
        // 达到队列能接受的最大个数限制就多了如下的配置
        params.put("x-max-length", 6);
        channel.queueDeclare(NORMAL_QUEUE, false, false, false, params);
        // 正常队列和正常交换机进行绑定
        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zi");

        System.out.println("01消费者正在接收消息......");
        channel.basicConsume(NORMAL_QUEUE,true,(consumeTag,message)->{
            System.out.println("01消费者接收到了消息：" + new String( message.getBody(), StandardCharsets.UTF_8));
        },consumeTag->{});
    }
}

```

启动01消费者，然后关掉（ 模仿异常 ），最后启动生产者，那么：生产者发送了10个消息，由于01消费者这边做了配置，所以有6个消息是在正常队列中，余下的4个消息就会进入死信队列

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123678-1534794900.png)




<br>
<br>


#### 3.8.2.2、超过队列能接受消息的最大字节长度

和前面一种相比，在01消费者方做另一个配置即可

```java
	params.put("x-max-length-bytes", 255);
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180126123-1701414327.png)

<br>


> **注意：关于两种情况同时使用的问题**

- 如配置的如下两个

```java
        params.put("x-max-length", 6);
        params.put("x-max-length-bytes", 255);

```

**那么先达到哪个上限设置就执行哪个**




<br>
<br>

### 3.8.3、消息被拒收

**注意点：必须开启手动应答**

```java
	// 第二个参数改成false
	channel.basicConsume(NORMAL_QUEUE,false,(consumeTag,message)->{},consumeTag->{});
```

<br>

> **生产者**

```java
package cn.zixieqing.dead_letter_queue.reack;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.concurrent.TimeoutException;


public class Producer {

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");
        channel.exchangeDeclare("reack_normal_exchange", BuiltinExchangeType.DIRECT);

        for (int i = 1; i < 11; i++) {
            String message = "生产者发送的消息" + i;
            channel.basicPublish("reack_normal_exchange","zixieqing",null,message.getBytes(StandardCharsets.UTF_8));
            System.out.println("消息===>" + message + "===>发送完毕");
        }
    }
}

```


<br>

> **消费者**

```java
package cn.zixieqing.dead_letter_queue.reack;

import cn.zixieqing.util.MQUtil;
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.HashMap;
import java.util.concurrent.TimeoutException;


public class Consumer01 {

    public static final String NORMAL_EXCHANGE = "reack_normal_exchange";
    public static final String DEAD_EXCHANGE = "reack_dead_exchange";

    public static final String DEAD_QUEUE = "reack_dead_queue";
    public static final String NORMAL_QUEUE = "reack_normal_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = MQUtil.getChannel("");

        // 声明正常交换机、死信交换机
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        // 声明死信队列
        channel.queueDeclare(DEAD_QUEUE, false, false, false, null);
        // 死信队列绑定死信交换机
        channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "xie");

        // 声明正常队列
        HashMap<String, Object> params = new HashMap<>();
        params.put("x-dead-letter-exchange", DEAD_EXCHANGE);
        params.put("x-dead-letter-routing-key", "xie");
        channel.queueDeclare(NORMAL_QUEUE, false, false, false, params);
        channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "zixieqing");

        System.out.println("01消费者正在接收消息.....");
        // 1、注意：需要开启手动应答（ 第二个参数为false ）
        channel.basicConsume(NORMAL_QUEUE,false,(consumeTag,message)->{
            String msg = new String(message.getBody(), StandardCharsets.UTF_8);

            // 如果发送的消息为：生产者发送的消息5  则：拒收
            if ( "生产者发送的消息5".equals( msg ) ) {
                System.out.println("此消息====>" + msg + "===>是拒收的");
                // 2、做拒收处理 - 注意：第二个参数设为false，表示不再重新入正常队列的队，这样消息才可以进入死信队列
                channel.basicReject( message.getEnvelope().getDeliveryTag(),false);
            }else {
                System.out.println("01消费者接收到了消息=====>" + msg);
            }
        },consumeTag->{});
    }
}

```



![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125915-1423590313.png)




<br>
<br>


## 3.9、延迟队列 - 重要

### 3.9.1、延迟队列概念

这个玩意儿要表达的意思其实已经见过了，就是死信队列中说的TTL消息过期，但是文字表达得换一下

**所谓的延迟队列：就是用来存放需要在指定时间内被处理的元素的队列，其内部是有序的**

**使用场景：**
- 1、支付时，订单在30分钟以内未支付则自动取消支付
- 2、退款，用户发起退款，在3天以后商家还未处理，那官方便介入其中进行处理
- ..........

**玩延迟队列需要具备的条件：**
- 1、具备死信队列知识
- 2、具备TTL知识
- 然后将这二者结合，加一些东西，上好的烹饪就做好了

<br>

**实现如下的逻辑**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124042-1924324175.png)

- P：生产者
- X：正常交换机
- Y：死信交换机
- QA、QB：正常队列
- QD：死信队列
- XA、XB：正常交换机、正常队列的routing key
- YD：死信交换机、死信队列的routing key


<br>
<br>

### 3.9.2、集成SpringBoot

#### 3.9.2.1、依赖

```xml
<dependencies>
        <!--rabbitmq的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

```

<br>
<br>

#### 3.9.2.2、yml文件配置

```yml
# RabbitMQ的配置
spring:
  rabbitmq:
    host: 自己服务器ip
    port: 5672
    username: admin
    password: admin
    # 要是有Vhost也可以进行配置

```


<br>
<br>

#### 3.9.2.4、RabbitMQ配置

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124042-1924324175.png)



```java
package cn.zixieqing.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;


@Configuration
public class MqConfig {

    /**
     * 正常交换机名称
     */
    private static final String TTL_NORMAL_EXCHANGE = "X";

    /**
     * 死信交换机名称
     */
    private static final String TTL_DEAD_LETTER_EXCHANGE = "Y";

    /**
     * 正常队列名称
     */
    private static final String TTL_NORMAL_QUEUE_A = "QA";
    private static final String TTL_NORMAL_QUEUE_B = "QB";

    /**
     * 死信队列名称
     */
    private static final String TTL_DEAD_LETTER_QUEUE_D = "QD";

    /**
     * 正常交换机 和 正常队列A的routing key
     */
    private static final String TTL_NORMAL_EXCHANGE_BIND_QUEUE_A = "XA";

    /**
     * 正常交换机 和 正常队列B的routing key
     */
    private static final String TTL_NORMAL_EXCHANGE_BIND_QUEUE_B = "XB";

    /**
     * 正常队列 和 死信交换机 及 死信交换机 与 死信队列的routing key
     */
    private static final String TTL_NORMAL_QUEUE_AND_DEAD_LETTER_EXCHANGE_AND_DEAD_LETTER_QUEUE_BIND = "YD";


    /**
     * 声明正常交换机
     */
    @Bean("xExchange")
    public DirectExchange xExchange() {
        // 直接创建是什么类型的交换机 加上 交换机名字就可以了
        return new DirectExchange(TTL_NORMAL_EXCHANGE);
    }

    /**
     * 声明死信交换机
     */
    @Bean("yExchange")
    public DirectExchange yExchange() {
        return new DirectExchange(TTL_DEAD_LETTER_EXCHANGE);
    }

    /**
     * 声明正常队列QA 并 绑定死信交互机Y
     */
    @Bean("queueA")
    public Queue queueA() {

        HashMap<String, Object> params = new HashMap<>();
        params.put("x-dead-letter-exchange", TTL_DEAD_LETTER_EXCHANGE);
        params.put("x-dead-letter-routing-key", TTL_NORMAL_QUEUE_AND_DEAD_LETTER_EXCHANGE_AND_DEAD_LETTER_QUEUE_BIND);
        params.put("x-message-ttl", 10 * 1000);

        // 构建队列 并 传入相应的参数
        return QueueBuilder.durable(TTL_NORMAL_QUEUE_A)
                .withArguments(params)
                .build();
    }

    /**
     * X正常交换机 和 QA正常队列绑定
     */
    @Bean
    public Binding xChangeBindingQueueA(@Qualifier("queueA") Queue queueA,
                                        @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueA)
                .to(xExchange)
                .with(TTL_NORMAL_EXCHANGE_BIND_QUEUE_A);
    }

    /**
     * 声明正常队列QB 并 绑定死信交换机Y
     */
    @Bean("queueB")
    public Queue queueB() {

        HashMap<String, Object> params = new HashMap<>();
        params.put("x-dead-letter-exchange", TTL_DEAD_LETTER_EXCHANGE);
        params.put("x-dead-letter-routing-key", TTL_NORMAL_QUEUE_AND_DEAD_LETTER_EXCHANGE_AND_DEAD_LETTER_QUEUE_BIND);
        params.put("x-message-ttl", 40 * 1000);

        // 构建队列 并 传入相应的参数
        return QueueBuilder.durable(TTL_NORMAL_QUEUE_B)
                .withArguments(params)
                .build();
    }

    /**
     * X正常交换机 和 QB正常队列绑定
     */
    @Bean
    public Binding xChangeBindingQueueB(@Qualifier("queueB") Queue queueB,
                                        @Qualifier("xExchange") DirectExchange xExchange) {
        return BindingBuilder.bind(queueB)
                .to(xExchange)
                .with(TTL_NORMAL_EXCHANGE_BIND_QUEUE_B);
    }

    /**
     * 声明死信队列D
     */
    @Bean("queueD")
    public Queue queueD() {
        return new Queue(TTL_DEAD_LETTER_QUEUE_D);
    }

    /**
     * 死信交换机 和 私信队列进行绑定
     */
    @Bean
    public Binding yExchangeBindingQueueD(@Qualifier("queueD") Queue queueD,
                                          @Qualifier("yExchange") DirectExchange yExchange) {
        return BindingBuilder.bind(queueD)
                .to(yExchange)
                .with(TTL_NORMAL_QUEUE_AND_DEAD_LETTER_EXCHANGE_AND_DEAD_LETTER_QUEUE_BIND);
    }

}

```


<br>
<br>

#### 3.9.2.5、生产者

> **新加一个依赖**

```xml
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.75</version>
        </dependency>

```

<br>



> **生产者伪代码**

```java
package cn.zixieqing.controller;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;


@RestController
@RequestMapping("sendMsg")
public class MqProducerController {

    /**
     * 这个玩意儿是Spring提供的
     */
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("{message}")
    public void sendMsg(@PathVariable String message) {

        System.out.println( new Date() + "：接收到了消息===>" + message);

        // 发送消息
        rabbitTemplate.convertAndSend("X","XA","这条消息是来着TTL为10s的===>" + message);

        rabbitTemplate.convertAndSend("X","XB","这条消息是来着TTL为40s的===>" + message);
    }
}

```

<br>
<br>

#### 3.9.2.6、消费者

```java
package cn.zixieqing.consumer;


import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.nio.charset.StandardCharsets;
import java.util.Date;

@Component
public class DeadLetterQueueConsumer {

    @RabbitListener(queues = "QD")
    public void receiveMsg(Message message,Channel Channel) {
        System.out.println( new Date() + "接收到了消息===>" +
            new String( message.getBody(), StandardCharsets.UTF_8));
    }
}

```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220510163932069-119177931.png)

<br>

**但是：这种延迟队列有缺点**
- 当有很多请求，而延迟时间也都不一样时，那么就要写N多的这种代码了

<br>
<br>

### 3.9.3、RabbitMQ插件实现延迟队列

**插件下载地址：**https://www.rabbitmq.com/community-plugins.html
**github地址：**https://github.com/rabbitmq/rabbitmq-delayed-message-exchange

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124784-505551154.png)

<br>



**进入如下的目录中**

```json
	cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.9.15/plugins  # 版本号改成自己的

```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124391-1273667429.png)

<br>

**把下载的插件上传进去**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125144-1335263505.png)


<br>


**启动插件**

```json
	rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125072-1211799996.png)



<br>

**重启rabbitMQ**

```json
	systemctl restart rabbitmq-server
```

然后去web管理界面看exchange，就发现交换机类型多了一个

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123694-1845650991.png)

<br>
<br>

#### 3.9.3.1、编写配置

使用这种插件的方式，那么延迟设置就是在exchange交换机这一方进行设置，和以前在queue队列中进行延迟设置不一样


**原来的延迟队列设置**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123945-2019356500.png)


<br>


**使插件之后的延迟设置**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123398-1590867729.png)




<br>


**使用插件，实现下面的逻辑图**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124338-1808837925.png)



```java
package cn.zixieqing.config;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.CustomExchange;
import org.springframework.amqp.core.Queue;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;

@Configuration
public class DelayedExchanegConfig {

    /**
     * 交换机名字
     */
    private static final String EXCHANGE_NAME = "delayed.exchange";

    /**
     * 队列名字
     */
    private static final String QUEUE_NAME = "delayed.queue";

    /**
     * 绑定键值
     */
    private static final String EXCHANGE_BINDING_QUEUE_ROUTING_KEY = "delayed.routingkey";


    /**
     * 声明交换机 - 目前这种交换机是没有的，这是插件的，因此：选择自定义交换机
     */
    @Bean
    public CustomExchange delayedExchange() {

        HashMap<String, Object> params = new HashMap<>(3);
        // 延迟类型
        params.put("x-delayed-type", "direct");

        /*
            参数1、交换机名字
            参数2、交换机类型 - 插件的那个类型
            参数3、交换机是否持久化
            参数4、交换机是否自动删除
            参数5、交换机的其他配置
         */
        return new CustomExchange(EXCHANGE_NAME, "x-delayed-message", true, false, params);
    }

    /**
     * 声明队列
     */
    @Bean
    public Queue delayedQueue() {
        return new Queue(QUEUE_NAME);
    }

    /**
     * 交换机 和 队列 进行绑定
     */
    public Binding exchangeBindingQueue(@Qualifier("delayedExchange") CustomExchange delayedExchange,
                                        @Qualifier("delayedQueue") Queue delayedQueue) {

        return BindingBuilder
                .bind(delayedQueue)
                .to(delayedExchange)
                .with(EXCHANGE_BINDING_QUEUE_ROUTING_KEY)
                // noargs()就是构建的意思 和 build()一样
                .noargs();
    }
}

```

<br>
<br>

#### 3.9.3.2、生产者

```java
package cn.zixieqing.controller;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Date;


@RestController
@RequestMapping("sendMsg")
public class DelatedQueueController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @GetMapping("/{message}/{ttl}")
    public void getMesg(@PathVariable String message, @PathVariable int ttl) {

        System.out.println(new Date() + "接收到了消息===>" + message + "===>失效时间为：" + ttl);

        // 发送消息
        rabbitTemplate.convertAndSend("delayed.exchange", "delayed.routingkey", data->{
            // 设置失效时间
            data.getMessageProperties().setDelay(10 * 1000);
            return data;
        });
    }
}

```


<br>
<br>


#### 3.9.3.3、消费者

```java
package cn.zixieqing.consumer;

import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.nio.charset.StandardCharsets;
import java.util.Date;

@Component
public class DelayedQueueConsumer {

    @RabbitListener(queues = "delayed.queue")
    public void receiveMessage(Message message) {
        System.out.println("消费者正在消费消息......");
        String msg = new String(message.getBody(), StandardCharsets.UTF_8);
        System.out.println(new Date() + "消费了消息===>" + message);
    }
}

```

<br>

**发送两次消息，然后把传的TTL弄成不一样的，那么：TTL值小的消息就会先被消费，然后到了指定时间之后，TTL长的消息再消费**

<br>
<br>



## 3.10、发布确认 - 续

### 3.10.1、ConfirmCallback() 和 ReturnCallback()

正常的流程应该是下面的样子

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180127800-177545409.png)

<br>

**但是：如果交换机出问题了呢，总之就是交换机没有接收到生产者发布的消息( 如：发消息时，交换机名字搞错了 )，那消息就直接丢了吗？**

**同理：要是队列出问题了呢，总之也就是交换机没有成功地把消息推到队列中（ 如：routing key搞错了 ），咋办？**

而要解决这种问题，就需要使用标题中使用的两个回调，从而：让架构模式变成如下的样子

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124324-2114257789.png)

<br>
<br>



> **ConfirmCallback() 和 ReturnCallback()的配置**


**在yml文件中添加如下内容**

```yml
spring:
  rabbitmq:
    # 发布确认类型
    publisher-confirm-type: correlated
    # 队列未收到消息时，触发returnCallback回调
    publisher-returns: true
```

<br>
<br>


**编写ConfirmCallback 和 returnCallback回调接口（ 伪代码 ）  - 注意点：这两个接口是RabbitTemplate的内部类（ 故而：就有大文章 )**

```java
@Component
public class PublisherConfirmAndReturnConfig implements RabbitTemplate.ConfirmCallback ,RabbitTemplate.ReturnCallback {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
    	初始化方法
    	目的：因为ConfirmCallback 和 ReturnCallback这两个接口是RabbitTemplate的内部类
    	因此：想要让当前编写的PublisherConfirmAndReturnConfig能够访问到这两个接口
    	那么：就需要把当前类PublisherConfirmAndReturnConfig的confirmCallback 和 returnCallback注入到RabbitTemplate中去（ init的作用 ）
    */
    @PostConstruct
    public void init(){
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnCallback(this);
    }

    /**
    	参数1、发送消息的ID - correlationData.getID()  和 消息的相关信息
    	参数2、是否成功发送消息给exchange  true成功；false失败
    	参数3、失败原因
    */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if(ack){
            System.out.println("消息已经送达到Exchange");
        }else{
            System.out.println("消息没有送达到Exchange");
        }
    }

    /**
    	参数1、消息 new String(message.getBody())
    	参数2、消息退回的状态码
    	参数3、消息退回的原因
    	参数4、交换机名字
    	参数5、路由键
    */
    @Override
    public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
        System.out.println("消息没有送达到Queue");
    }
}

```

<br>

**生产者调用的方法是：rabbitTemplate.convertAndSend(String exchange, String routingKey, Object message, CorrelationData correlationData)**

多了一个CorrelationData 参数，这个参数携带的就是消息相关信息

<br>
<br>


## 3.11、备份交换机

这个玩意儿也是为了解决前面发布确认中队列出问题的方案

**注意：这种方式优先级比前面的 ReturnCallback回退策略要高（ 演示：跳过 - 可以采用将这二者都配置好，然后进行测试，结果是备份交换机的方式会优先执行，而前面的回退策略的方式并不会执行 ）**

<br>

**采用备份交换机时的架构图**

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123328-97108755.png)



<br>

> **上图架构的伪代码配置编写**

```java
package cn.zixieqing.config;

import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AlternateExchangeConfig {

    /**
     * 正常交换机名字
     */
    private static final String NORMAL_EXCHANGE_NAME = "normal_exchange";

    /**
     * 正常队列
     */
    private static final String NORMAL_QUEUE_NAME = "normal_queue";

    /**
     * 备份交换机名字
     */
    private static final String ALTERNATE_EXCHANGE_NAME = "alternate_exchange";

    /**
     * 备份队列名字
     */
    private static final String ALTERNATE_QUEUE_NAME = "alternate_queue";

    /**
     * 用于警告的队列名字
     */
    private static final String WARNING_QUEUE_NAME = "warning_queue";

    /**
     * 声明正常交换机 但是：需要做一件事情 - 消息没投递到正常队列时，需要让其走备份交换机
     */
    @Bean
    public DirectExchange confirmExchange() {

        return ExchangeBuilder
                .directExchange(NORMAL_EXCHANGE_NAME)
                .durable(true)
                // 绑定备份交换机
                .withArgument("alternate-exchange", ALTERNATE_EXCHANGE_NAME)
                .build();
    }

    /**
     * 声明确认队列
     */
    @Bean
    public Queue confirmQueue() {
        return new Queue(NORMAL_QUEUE_NAME);
    }

    /**
     * 确认交换机（ 正常交换机 ） 和 确认队列进行绑定
     */
    @Bean
    public Binding confirmExchangeBindingConfirmQueue(@Qualifier("confirmExchange") DirectExchange confirmExchange,
                                                      @Qualifier("confirmQueue") Queue confirmQueue) {
        return BindingBuilder
                .bind(confirmQueue)
                .to(confirmExchange)
                .with("routingkey");
    }

    /**
     * 声明备份交换机
     */
    @Bean
    public FanoutExchange alternateExchange() {
        return new FanoutExchange(ALTERNATE_EXCHANGE_NAME);
    }

    /**
     * 声明备份队列
     */
    @Bean
    public Queue alternateQueue() {
        return QueueBuilder
                .durable(ALTERNATE_QUEUE_NAME)
                .build();
    }

    /**
     * 声明警告队列
     */
    @Bean
    public Queue warningQueue() {
        return new Queue(WARNING_QUEUE_NAME);
    }

    /**
     * 备份交换机 和 备份队列进行绑定
     */
    @Bean
    public Binding alternateExchangeBindingAlternateQueue(@Qualifier("alternateQueue") Queue alternateQueue,
                                                          @Qualifier("alternateExchange") FanoutExchange alternateExchange) {
        return BindingBuilder
                .bind(alternateQueue)
                .to(alternateExchange);
    }

    /**
     * 备份交换机 和 警告队列进行绑定
     */
    @Bean
    public Binding alternateExchangeBindingWarningQueue(@Qualifier("warningQueue") Queue warningQueue,
                                                        @Qualifier("alternateExchange") FanoutExchange alternateExchange) {
        return BindingBuilder
                .bind(warningQueue)
                .to(alternateExchange);
    }
}

```

<br>


后续的操作就是差不多的，生产者发送消息，消费者消费消息，然后里面再做一些业务的细节处理就可以了

<br>
<br>


## 3.12、优先级队列

**这就是为了让MQ队列中的某个 / 某些消息能够优先被消费**

**使用场景：搞内幕，让某个人 / 某些人一定能够抢到什么商品**

<br>

**想要实现优先级队列，需要满足如下条件：**

- 1、队列本身设置优先级( 在声明队列时进行参数配置 )

```java
          /**
           * 基础型配置
           */
          Map<String, Object> params = new HashMap();
          params.put("x-max-priority", 10);  // 默认区间：(0, 255) 但是若用这个区间，则会浪费CPU和内存消耗，因此：改为(0, 10)即可
          channel.queueDeclare("hello", true, false, false, params);


         /**
          * SpringBoot中的配置
          */
          @Bean
          public Queue alternateQueue() {
              HashMap<String, Object> params = new HashMap<>();
              params.put("x-max-priority", 10);
              return QueueBuilder
                      .durable(ALTERNATE_QUEUE_NAME).withArguments(params)
                      .build();
          }

```


2、让消息有优先级

```java
         /**
          * 基础型配置 - 生产者调用basicPublisher()时配置的消息properties
          */
          AMQP.BasicProperties properties = new AMQP.BasicProperties()
              .builder()
              .priority(5)
              .build();

         /**
          * SpringBoot中的配置
          */
          // 发送消息
          rabbitTemplate.convertAndSend("normal.exchange", "normal.routingkey", data->{
              // 消息设置优先级 - 注意：这个数值不能比前面队列设置的那个优先级数值大，即：这里的消息优先级范围就是前面队列中设置的(0, 10)
              data.getMessageProperties().setPriority(5);
              return data;
          });

```

<br>


**注意点：设置了优先级之后，需要做到如下条件：**

- **需要让消息全部都发到队列之后，才可以进行消费，原因：消息进入了队列，是会重新根据优先级大小进行排队，从而让优先级数值越大越在前面**

  ![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180125364-426919581.png)


<br>
<br>


## 3.13、惰性队列

**这玩意儿指的就是让消息存放在磁盘中**


<br>


正常情况下是如下的样子

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180124756-1300698884.png)

<br>

但是：如果此时发送的消息是成千上万条，并且消费者出故障了( 下线、宕机、维护从而关闭 )，那么这些成千上万的消息就会堆积在MQ中，怎么办？就需要像下面这么搞

![image](https://img2023.cnblogs.com/blog/2421736/202302/2421736-20230203180123605-1403100043.png)


<br>


> **设置惰性队列的配置**

```java

   /**
    * 基础型配置
    */
    Map<String, Object> params = new HashMap();
    params.put("x-queue-mode", "lazy");
    channel.queueDeclare("hello", true, false, false, params);


   /**
    * SpringBoot中的配置
    */
    @Bean
    public Queue alternateQueue() {
	
        HashMap<String, Object> params = new HashMap<>();
        params.put("x-queue-mode", "lazy");
        return QueueBuilder
                .durable(ALQUEUE_NAME).withArguments(params)
                .build();
    }

```

**经过如上配置之后，那么内存中记录的就是指向磁盘的引用地址，而真实的数据是在磁盘中，下一次消费者恢复之后，就可以从磁盘中读取出来，然后再发给消费者（ 缺点：得先读取，然后发送，这性能很慢，但是：处理场景就是消费者挂彩了，不再消费消息时存储数据的情景 ）**

<br>
<br>
