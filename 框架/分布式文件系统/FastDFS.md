# 1、FastDFS

## 1.1、了解基础概念

### 1.1.1、什么是分布式文件系统？

- 全称：Distributed File System，即简称的DFS
- 这个东西可以是一个软件，也可以说是服务器，和tomcat差不多，即相当于软件也相当于是服务器，这个软件就是用来管理文件的
- 这个软件所管理的文件通常不是在一个服务器节点上，而是在多个服务器节点上
- 服务器节点通过网络相连构成一个庞大的文件存储服务器集群，这些服务器都用于存储文件资源，通过分布式文件系统来管理这些服务器上的文件



### 1.1.2、传统文件系统 和 分布式文件系统对比

**传统文件系统**

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531130933679-2059297028.png)

- **缺点**
  - 所有的文件都存放在一台计算机中，如果这台计算机挂彩了，那么就会导致整个服务不可用（ 文件不能上传和下载了 )
  - 如果这台计算机磁盘损坏了，那么会丢失所有的文件
  - 这台计算机的磁盘空间非常有限，很容易到达磁盘的上限，导致无法上传文件



- **回顾玩servlet时的文件上传和下载**

  **文件上传**

  - 假如前端轰HTML写法是如下的样子：

  ```html
  
  		  <div id="image">
  			  <label for="">标题图片:</label>
  			  <input type="file" id="file" name="file" >
  			  <img src="" alt="" width="100px" height="150px">
  		  </div>
  
  ```

  - JS写法如下：

  ```javascript
  
  		// 当图片发生改变时 —— 也就是用户点击file框，上传文件时
  		$("#file").on( 'change' , function () {
   
  			// 创建一个FormData空对象，就相当于是伪造了一个form表单
  			let formData = new FormData();
   
  			// 这个FromData对象就用来装文件内容
              // 文件的files属性本质是个数组
  			let files = $("#file").prop("files");
  			formData.append("upFile" , files[0] );
   
  			$.ajax( {
   
  				url: '/ajax/upload.do',
  				type: 'post',
  				data: formData,
  				dataType: 'json',
   
  				cache: false,    // 上传文件不需要缓存
  				contentType: false,      // 不需要对内容类型进行处理  因为内容是一个FormData对象
  				processData: false,       // 不需要对数据进行处理，因为上面的data是一个FormData对象
   
  				// 后台返回的格式 ：
  				// { "errno":"0" , "data":[ {"alt":"1633528500498.jpg" , "url":"/upload/2021-10-06/1633528500498.jpg"} ] }
  				success: function (info) {
  					info.data.forEach( function (data) {
   
  						// $("#image img").remove();
  						// $("#image").append( ' <img src=" '+data.url+' " alt="" width="100px" height="150px"> ' )
   
  	/*
  	 注掉的这种是：html中没有img标签时使用
  	 因为：使用下面这种方法的情景是 —— 页面本来就有一个img框（ 即：初始页面上这个file本身有一张图片 ），所以下面这种可以做到图片改变时把图片的路径换掉，也就是图片渲染（ 也是数据回填 的思想 ）
  	 但是：如果页面一开始file的位置是不应该有图片的，是后面用户选了之后才出现图片预览效果，那么：就使用注释掉的这种方法：追加
  	*/
   
  						$("#image img").attr("src" , data.url );
  					});
  				}
  			} );
   
  		})
  
  ```

  - 那么后端的low代码如下：

  ```java
  
  	import com.alibaba.fastjson.JSON;
   
  	import javax.servlet.ServletException;
  	import javax.servlet.annotation.MultipartConfig;
  	import javax.servlet.annotation.WebServlet;
  	import javax.servlet.http.HttpServlet;
  	import javax.servlet.http.HttpServletRequest;
  	import javax.servlet.http.HttpServletResponse;
  	import javax.servlet.http.Part;
  	import java.io.File;
  	import java.io.IOException;
  	import java.time.LocalDate;
  	import java.util.ArrayList;
  	import java.util.Collection;
  	import java.util.Date;
  	import java.util.HashMap;
   
   
  	// @MultipartConfig 注解就是文件注解，要获取前端的文件信息，必须加这个注解，不然做的所有事情都是无用功
  	@MultipartConfig
  	@WebServlet("/ajax/upload.do")
  	public class UploadServlet extends HttpServlet {
   
  		@Override
  		protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
   
  			/*
  			*   想要构建的是这么一个玩意儿
  			*       "errno":0 data:[ { url:"图片地址“ } , { alt:"图片说明“ } , { href:"null" } ]
  			*
  			* */
   
  			ArrayList<Object> list = new ArrayList<>();
   
  			Collection<Part> parts = req.getParts();   // 这是获取前台上传的文件
   
  			for (Part part : parts) {
   
  				// 先构建 data:[ { } , { } ]中的[ { } , { } ]
   
  				// 获取文件的全路径
                  // 但是：不同浏览器的这个全路径都不一样，所以需要截取从而自定义文件名
  				String filePath = part.getSubmittedFileName();  
  				// System.out.println(filePath);
  				// 截取文件的后缀名
  				int subFileName = filePath.lastIndexOf(".");
  				String fileSuffix = filePath.substring(subFileName);
   
  				// 自己给文件重新定义一个名字，并规定存放的地方
  				String timeStr = LocalDate.now().toString();
   
  				// 获取当前项目的一个指定文件夹名字，用来保存文件 注意：getRealPath这是获取的当前项目的全路径，即：从盘符开始的路径
  				String proPathName = this.getServletContext().getRealPath("/upload/" + timeStr );
  				File file = new File(proPathName);
  				if ( !file.exists() ){
  					file.mkdirs();
  				}
   
  				// 拼接文件后缀名并保存文件
  				long timeStamp = new Date().getTime();
  				part.write(proPathName + "/" + timeStamp + fileSuffix );
   
  				HashMap<String, String> map = new HashMap<>();
  				map.put( "url" , "/upload/" + timeStr + "/" + timeStamp + fileSuffix );
  				map.put( "alt" , timeStamp + fileSuffix );
  				map.put( "href" , null );
  				list.add(map);
  			}
   
  			// 再构建"errno":0 data:[ { url:"图片地址“ } , { alt:"图片说明“ } , { href:"null" } ]
  			HashMap<String, Object> map = new HashMap<>();
  			map.put("errno", "0");
  			map.put("data", list);
   
  			resp.getWriter().print( JSON.toJSONString(map) );
  		}
  	}
  
  ```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531132604814-61556119.png)



**文件下载**

- 后端low代码如下

```java

	import javax.servlet.ServletException;
	import javax.servlet.ServletOutputStream;
	import javax.servlet.annotation.WebServlet;
	import javax.servlet.http.HttpServlet;
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	import java.io.FileInputStream;
	import java.io.IOException;
	import java.net.URLEncoder;
 
	@WebServlet("/downFile")
	public class downFileInClientServlet extends HttpServlet {
 
		@Override
		protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
			doPost(req, resp);
		}
 
		@Override
		protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
 
			// 1、获取让浏览器下载的文件路径
			String FileRealPath = "D:\\JavaTrainStudy\\servlet\\out\\production\\study06-httpServletResponse\\loginbg.png";
 
			// 2、告知浏览器要下载的文件名是什么？
			String fileName = FileRealPath.substring( FileRealPath.lastIndexOf("\\") + 1 );
 
			// 3、让浏览器支持文件下载
			// Content-Disposition这个就是让浏览器支持文件下载
			// URLEncoder.encode（ String s , String enc ) 是为了以防文件名是中文名，这样就设置编码格式了，让浏览器能够解析这个中文文件名
			resp.setHeader("Content-Disposition" , "attachment ; filename=" + URLEncoder.encode(fileName , "utf-8"));
 
			// 4、获取输入、输出流对象 并 把服务器中的文件输出到浏览器上
			FileInputStream fis = new FileInputStream( FileRealPath );
			ServletOutputStream os = resp.getOutputStream();
 
			// 创建缓冲区
			int len = 0 ;
			byte[] buffer = new byte[1024];
			while ( ( len = fis.read( buffer ) )  > 0 ){
				os.write( buffer , 0 , len);
			}
 
			// 5、关闭流管道
			if ( os != null ){
				os.close();
			}
			if ( fis != null ){
				fis.close();
			}
 
		}
	}
 
```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531132639716-1294505210.png)



**分布式文件系统**

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531133554497-813437547.png)

- **优点**
  - 解决了传统方式的单点故障问题
  - 若某一个节点出现故障，则还有其他的节点可以用来读取和写入文件
  - 提供数据备份从而避免磁盘损坏而导致的文件丢失
  - 提供扩容机制，无限增加文件存放的空间上限





## 1.2、认识FastDFS

> **补充：常见的分布式文件系统**

- **FastDFS**、GFS、**HDFS**、Lustre 、Ceph 、GridFS 、mogileFS、TFS





### 1.2.1、了解FastDFS

- **官网：https://github.com/happyfish100/fastdfs** 
- FastDFS是一个开源的轻量级分布式文件系统，为互联网应用量身定做，简单、灵活、高效，采用C语言开发，由阿里巴巴开发并开源
- FastDFS对文件进行管理，功能包括：文件存储、文件同步( 指的是：文件系统 和 数据备份之间的同步 )、文件上传、文件下载、文件删除等
- FastDFS解决了大容量文件存储的问题
- FastDFS特别适合以文件为载体的在线服务，如相册网站、文档网站、图片网站、视频网站等
- FastDFS充分考虑了冗余备份、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务
  - 冗余备份：指的是文件系统中存的文件 和 数据备份中存的文件完全一致的问题
    - ![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531140634652-1681210056.png)
  - 线性扩容：文件系统 和 数据备份不断增加呗( 就是上图中再加几份嘛 ），和水平扩容类似



### 1.2.2、FastDFS的组成结构

- **由两大部分构成，一个是客户端，一个是服务端**
  - **客户端：**指我们的程序，比如我们的Java程序去连接FastDFS、操作FastDFS，那我们的Java程序就是一个客户端。FastDFS提供专有API访问，目前提供了C、Java和PHP几种编程语言的API，用来访问FastDFS文件系统
  - **服务端由两个部分构成：一个是跟踪器（tracker），一个是存储节点（storage）**
    - **跟踪器 tracker：**这个玩意儿类似于Erueka / zookeeper注册中心，**起到一个调度的作用**。它是在内存中记录集群中存储节点storage的状态信息，是前端Client和后端存储节点storage的枢纽，因为相关信息全部在内存中，Tracker server的性能非常高，一个较大的集群（比如上百个group，group指的就是：文件系统 和 数据备份的组合，这二者就是一个group）中有3台就足够了
    - **存储节点 storage：用于存储文件**，包括文件和文件属性（meta data，如：文件名、文件大小、文件后缀...）都保存到存储服务器磁盘上。以及完成文件管理的所有功能：文件存储、文件同步和提供文件访问( 上传、下载、删除 )等





# 2、开始玩FastDFS

## 2.1、安装FastDFS

- **<span style = "color:blue">注：我的系统是centos 7</span>**

- **安装需要的依赖环境 gcc、libevent、libevent-devel**

```linux

yum install gcc libevent libevent-devel -y

```



- **安装公共函数库libfastcommon 和 fastDFS压缩包**
  - 自行去前面官网中进行下载，当然：官网的wiki中有在线拉取命令

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531151515193-138524580.png)



- **加压公共函数库libfastcommon**

```linux

tar -zxvf libfastcommon-1.0.36.tar.gz

```

- 进入libfastcommon，执行里面的make.sh，编译公共函数

```linux

./make.sh 

# 当然：可以把命令进行合并 执行如下命名 就是编译并安装
./make.sh && ./make.sh install

```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531152212821-846136292.png)

- 安装公共函数

```linux

./make.sh install

```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531152401205-116210948.png)



- **解压缩`fastdfs-5.11.tar.gz`压缩包**

```linux

tar -zxvf fastdfs-5.11.tar.gz

```

- 进入解压之后的文件，使用`make sh`进行编译

```linux

./make.sh

# 一样的可以用组合命令 即：编译并安装
./make.sh && ./make.sh install

```

- 安装

```linux

./make.sh install

```



- **检查是否安装成功，进入如下的目录即可**

```linux

cd /usr/bin

```

- 往后找，出现这些fdfs开头的文件就表示成功（ 这些文件就是fastDFS的相关命令 )

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531153158850-1837925811.png)



- **fastDFS配置文件所在地，进入如下目录即可**
- **想要让fastDFS的配置文件生效，那么就需要放到下面的这个目录中**

```linux

cd /etc/fdfs

```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531153601182-198451142.png)



- **拷贝两个配置文件到`etc/fdfs`中，这两个配置文件在解压的fastDFS的conf中，一个叫`http.conf`，一个叫`mime.types`**

```linux
# 供nginx访问使用
cp http.conf /etc/fdfs

# 供nginx访问使用
cp mime.types /etc/fdfs

```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531155849016-1117685090.png)



## 2.2、启动FastDFS

- 这个玩意儿不可以直接启动，因为默认的配置文件中有一些关于文件目录的配置是不存在的，因此：只要直接启动就会报错



### 2.2.1、修改配置文件

- 要修改的文件就两个

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531160843547-962705415.png)

- 以防万一，因此：将上面的文件拷贝一份

```linux

/etc/fdfs

mv storage.conf.sample ./storage.conf

mv tracker.conf.sample ./tracker.conf

```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531162036373-1222382368.png)



#### 2.2.1.1、修改tracker.conf

- 在这个配置文件中有一个`base_path`配置，指向的是fastDFS作者余庆的地址，而我们自己的linux中并没有这个目录，因此：做修改

```conf

vim tracker.conf

# 搜索此配置
/base_path

# 改成的值，也可以自定义自己的目录（ 注意：需要保证这个目录必须存在，没存在那就需要创建 ）
base_path=/opt/fastdfs/tracker

```

- **注意：需要保证这个目录必须存在，没存在那就需要创建**



#### 2.2.1.2、修改storage.conf

- 需要改的内容如下

```conf

# storage存储数据目录
base_path=/opt/fastdfs/storage

# 真正存放文件的目录
store_path0=/opt/fastdfs/storage/files

# 注册当前存储节点的跟踪器地址
tracker_server=服务器ip:22122
	
```



- **注意：要是前面的那三个目录没有的话，记得创建，若指向的是已经创建好的目录，那就不用创建了**

```linux

mkdir -p /opt/fastdfs/tracker

mkdir -p /opt/fastdfs/storage

mkdir -p /opt/fastdfs/storage/files

```





### 2.2.2、开启fastDFS

- 在任意目录下，执行如下的命令即可

```linux

# 启动tracker 要想看fdfs_trackerd的命令用法，那直接输入fdfs_trackerd就可以弹出其用法了
# 如：要关闭tracker，则命令为：fdfs_trackerd /etc/fdfs/tracker.conf stop
# 开启 | 重启就是把stop改成start | restart即可
fdfs_trackerd /etc/fdfs/tracker.conf


# 启动storage 同样的，看命令用户就直接输入fdfs_storaged
fdfs_storaged /etc/fdfs/storage.conf


```



- **查看是否启动成功**

```linux

ps -ef | grep fdfs

```

- 如下图表示启动成功

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531164935428-59444575.png)

- **但是上面的启动会有坑儿，所以需要确认一把**

```linux

# 查看日志文件是否有报ERROR
cd /opt/fastdfs/storage/logs/storage.log

```

- **若是发现日志中报的是如下信息**

```json

ERROR - file: storage_ip_changed_dealer.c, line: 180, connect to tracker server 服务器ip:22122 fail, errno: 110, error info: Connection timed out

即：链接超时

```

- 这种情况一般都是22122端口没开放

```linux

# 开放22122端口
firewall-cmd --zone=public --add-port=22122/tcp --permanent

# 重启防火墙
systemctl restart firewalld.service

# 当然：要是云服务器的话，直接在web管理界面的管理中添加规则（ 开放22122端口 ) 即可

```



#### 2.2.3、查看默认创建的文件数

- **进入如下的目录**

```linux

cd /opt/fastdfs/storage/files/data

```

![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531190140636-1058734685.png)

- 这里面有526个文件夹，而每一个文件夹里面又有526个文件夹，即256 * 256个文件夹，总的文件夹数目为6万多个

  - 这256 * 256个文件夹的作用：解决的就是如下的问题

  ![image](https://img2022.cnblogs.com/blog/2421736/202205/2421736-20220531191447653-2142567796.png)

  - 而fastDFS就是使用那256 * 256个文件夹，把文件分别放入哪些文件夹中，这样就让搜索变得方便了





#### 2.2.4、测试FastDFS

##### 2.2.4.1、测试上传文件

- 要能进行测试的话，需要修改一个配置文件，因为这个配置文件中的配置信息是作者余庆的

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601085529351-1674223002.png)

- **要修改的内容如下：**

```json

# 注意：这个目录也要保证存在，不存在就是创建 mkdir -p /opt/fastdfs/client
base_path=/opt/fastdfs/client

tracker_server=自己服务器ip:22122

```



- **搞一个用来测试上传的文件**

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601095813784-1688866298.png)

- **执行文件上传命令**

  - 可以使用如下命令看一下测试文件上传命令是怎么写的

    - ```linux
      
      fdfs_test
      
      ```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601100046307-1815112793.png)

- **提取出测试命令语法**

```linux

fdfs_test <config_file> <operation>
	operation: upload, download, getmeta, setmeta, delete and query_servers
# <> 表示必填

# 因此：在测试中，文件上传的指令为：
fdfs_test /etc/fdfs/client.conf upload /root/hello-fastdfs.txt


```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601101742207-1931098713.png)

- **注意：防火墙的问题啊，要是报：`connect to 162.14.66.60:23000 fail, errno: 113, error info: No route to host`，这就是防火墙没开放23000端口，打开就可以了**

```linux

# 开放23000端口
firewall-cmd --zone=public --add-port=23000/tcp --permanent

# 刷新防火墙
systemctl restart firewalld.service

```

- **上面成功之后有一堆信息，很重要**

```json

This is FastDFS client test program v5.11

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.

# 这个是说访问fastdfs的主页网址 - 目前还不能访问，需要后面弄
Please visit the FastDFS Home Page http://www.csource.org/ 
for more detail.

# 这是配置的client.conf中的信息
[2022-06-01 10:17:07] DEBUG - base_path=/opt/fastdfs/client, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
	server 1. group_name=, ip_addr=162.14.66.60, port=23000

group_name=group1, ip_addr=162.14.66.60, port=23000
storage_upload_by_filename
# 重要的信息在这里group_name、remote_filename
# group_name就是组名，在前面配置中见过它，就是说的文件系统 和 数据备份这二者的组合名
# remote_filename 远程文件名 这是关键中的关键，告知你：文件保存到那里去了
group_name=group1, remote_filename=M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt
source ip address: 10.0.0.16
file timestamp=2022-06-01 10:17:07
file size=641
file crc32=1141168436
# 这就是可以在浏览器访问这个文件的路径，但是：现在还无法访问，因为还没有配置nginx
example file url: http://162.14.66.60/group1/M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt
source ip address: 10.0.0.16
file timestamp=2022-06-01 10:17:07
file size=641
file crc32=1141168436
example file url: http://162.14.66.60/group1/M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt

```

- **单独说明：`remote_filename`**

```json

remote_filename=M00/00/00/

M00 指的是：/opt/fastdfs/storage/files/data			就是前面去看默认创建文件数( 256 * 256 )的位置，跟前面的配置有关啊

00/00/ 指的就是：/opt/fastdfs/storage/files/data目录下的00子目录，这里面的00目录

CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt  指的是：保存的文件名  fastdfs会重新生成文件名，以防的就是同名文件，造成附件覆盖的问题

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601103120994-99896373.png)

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601103154752-1302284835.png)

- 上图中几个文件解读

```json

# _big 就是数据备份文件
# ——m 就是meta data文件，即：文件属性文件（ 文件名、文件后缀、文件大小..... ）
-rw-r--r-- 1 root root 641 Jun  1 10:17 CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt
-rw-r--r-- 1 root root  49 Jun  1 10:17 CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt-m

# 这两个就是文件系统中的文件
-rw-r--r-- 1 root root 641 Jun  1 10:17 CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt
-rw-r--r-- 1 root root  49 Jun  1 10:17 CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt-m


# CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt 和 文件系统中的CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt存的内容是一样的

# CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt-m 和 CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt-m这两个备份文件也是相应的

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601103625604-1533859055.png)



##### 2.2.4.2、测试文件下载和删除

- 前面已经见过对应的语法了

```linux

fdfs_test <config_file> <operation>
	operation: upload, download, getmeta, setmeta, delete and query_servers
# <> 表示必填

```

- 变一下就可以了

```linux

# 变成下载的命令，然后使用此命令查看完整命令即可
fdfs_test /etc/fdfs/client.conf download


# 根据执行上面的命令，得到文件下载的语法
fdfs_test <config_file> download <group_name> <remote_filename>

# 那么想要下载刚刚上传的文件，执行如下的命令即可
fdfs_test /etc/fdfs/client.conf download group1 M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt
# 其中：group 和 remote_filename都在前面上传时见过了
# 注：这个下载是下载到当前所在目录的位置


# 同理：就可以得到文件删除的命令了
fdfs_test /etc/fdfs/client.conf delete group1 M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt

```

- 以上这些`fdfs_test`只会在测试时使用，其他地方基本上都不用的





## 2.3、安装Nginx

- **上传`fastdfs-niginx`扩展模块 并 解压 - 使用官网中wiki说明的命令拉取也行**

- **安装nginx，要是有的话就跳过**
- **注意点：`nginx`和`fastdfs-nginx`放到`/usr/local`目录下，不然可能会出现莫名其妙的问题**

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601150551340-722514246.png)

- **记住两个目录**

```json

# nginx安装目录
/usr/local/nginx_fdfs

# fastdfs-nginx模块的src目录
/usr/local/fastdfs-nginx-module-master/src

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601141711103-711656905.png)

- **进入nginx安装目录，进行模块添加配置**

```linux

# 进入nginx安装目录
cd nginx_fdfs

# 执行模块配置 
# prefix 就是前面让记住的nginx安装目录	add-module就是fastdfs-nginx模块的src目录
./configure --prefix=/usr/local/nginx_fdfs --add-module=/usr/local/fastdfs-nginx-module-master/src


```

- **编译并安装**

```linux
# 在安装的nginx目录下载执行下述命令
make & make install

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601150912072-1052442485.png)

- **注释事项：Nginx的安装需要Linux安装相关的几个库，否则编译会出现错误，有这几个的话就不安装了**

```linux

yum install gcc openssl openssl-devel pcre pcre-devel zlib zlib-devel –y

```



### 2.3.1、修改需要的配置文件

- **将fastdfs-nginx扩展模块中的`mod_fastdfs.conf`文件复制到`/etc/fdfs`中**

```linux

cp /usr/local/fastdfs-nginx-module-master/src/mod_fastdfs.conf /etc/fdfs

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601132037277-1059067263.png)

- **修改`/etc/fdfs/mod_fastdfs.conf`**

```conf
vim mod_fastdfs.conf

# 修改内容如下：
# 这个目录要保证存在，不存在就要配置好了创建它 mkdir -p /opt/fastdfs/nginx_mod
base_path=/opt/fastdfs/nginx_mod

tracker_server=自己服务器ip:22122

# 访问地址是否带上组名
url_have_group_name = true

store_path0=/opt/fastdfs/storage/files

```

- 上面`base_path`目录要是不存在记得创建

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601132631083-1262340685.png)

- **进入`nginx_fdfs`的安装目录中，去`nginx.conf`中配置`fastdfs-nginx`的扩展模块**

```conf
# 编辑nginx.conf文件
vim /usr/local/nginx_fdfs/conf/nginx.conf

# 配置内容
location ~ /group[1-9]/M0[0-9] {	
     ngx_fastdfs_module;  
}


# 解读：ngx_fastdfs_module
# 	这个指令不是Nginx本身提供的，是扩展模块提供的，根据这个指令找到FastDFS提供的Nginx模块配置文件，然后找到Tracker，最终找到Stroager
```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601160836508-1504938975.png)

- **启动`nginx`**

```json

/usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf  -t

/usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf

# 保险起见，查看nginx是否启动成功
ps -ef | grep nginx

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601154554178-1692693500.png)



![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601154741181-1459702966.png)

- 注意：这里很容易出现启动不起来，如果下面这个进程没有启动起来

```json

nobody    3895  3894  0 15:45 ?        00:00:00 nginx: worker process

```

- 那么：就去查看日志文件

```linux

cd /usr/local/nginx_fdfs/logs


# 还有一份日志中也可能出现错误信息
cd /opt/fastdfs/nginx_mod

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601155004253-612848326.png)


![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601155127364-1427912722.png)

- **现在就可以去浏览器中查看刚刚上传的文件的**

  - 注意开放端口啊

  ```linux
  
  # 开放80端口
  firewall-cmd --zone=public --add-port=80/tcp --permanent
  
  # 重启防火墙
  systemctl restart firewalld.service
  
  ```

  - 访问前面上传文件时的url地址

```json


This is FastDFS client test program v5.11

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.

# 这个是说访问fastdfs的主页网址 - 目前还不能访问，需要后面弄
Please visit the FastDFS Home Page http://www.csource.org/ 
for more detail.

# 这是配置的client.conf中的信息
[2022-06-01 10:17:07] DEBUG - base_path=/opt/fastdfs/client, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
	server 1. group_name=, ip_addr=162.14.66.60, port=23000

group_name=group1, ip_addr=162.14.66.60, port=23000
storage_upload_by_filename
# 重要的信息在这里group_name、remote_filename
# group_name就是组名，在前面配置中见过它，就是说的文件系统 和 数据备份这二者的组合名
# remote_filename 远程文件名 这是关键中的关键，告知你：文件保存到那里去了
group_name=group1, remote_filename=M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt
source ip address: 10.0.0.16
file timestamp=2022-06-01 10:17:07
file size=641
file crc32=1141168436
# 这就是可以在浏览器访问这个文件的路径，但是：现在还无法访问，因为还没有配置nginx
example file url: http://162.14.66.60/group1/M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt
source ip address: 10.0.0.16
file timestamp=2022-06-01 10:17:07
file size=641
file crc32=1141168436
example file url: http://162.14.66.60/group1/M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590_big.txt

```

- **访问：`http://162.14.66.60/group1/M00/00/00/CgAAEGKWzCOACGE1AAACgUQE2TQ590.txt`**

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601160349162-2029736800.png)



### 2.3.2、扩展模块执行流程

- **下面这个流程很重要，涉及到后面的知识**

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220601172753939-1467167321.png)



## 2.4、Java操作FastDFS

> **依赖**

```xml

        <dependency>
            <groupId>net.oschina.zcx7878</groupId>
            <artifactId>fastdfs-client-java</artifactId>
            <version>1.27.0.0</version>
        </dependency>

```

- 这个是可以从阿里仓库拉取的依赖，但是严格来说这不是作者的，但是不妨碍使用
- 真正的依赖没有放到中央仓库中去，因此并不能通过maven拉取，而是需要去官网https://github.com/happyfish100/fastdfs-client-java/tags中下载源码，然后解压，进入解压目录，使用DOS窗口，执行`mvn clean install`命令，打成j本地ar包，然后就可以在maven中使用了，最后打出来的jar包是在`org.csource`目录下，所以正规依赖应该是下面这个

```xml

        <dependency>
            <groupId>org.csource</groupId>
            <artifactId>fastdfs-client-java</artifactId>
            <version>1.27-RELEASE</version>
        </dependency>

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220602090958365-92162004.png)







### 2.4.1、文件上传

- **在`resources`目录下创建`fastdfs.conf`文件，并编写如下内容：**

```conf

tracker_server=服务器ip:22122

```

- **编写文件上传代码**

```java
package com.zixieqing;

import org.csource.common.MyException;
import org.csource.fastdfs.*;

import java.io.IOException;

/**
 * @author : ZiXieQing
 * @version : V1.0.0
 * @className : UploadFile
 * @description ： 该类功能 FastDFS文件上传
 * @packageName : com.zixieqing
 */

public class UploadFile {

    public static void main(String[] args) {

        TrackerServer trackerServer = null;
        StorageServer storageServer = null;
        try {
            // 1、初始化配置文件
            ClientGlobal.init("fastdfs.conf");

            // 2、创建tracker客户端
            TrackerClient trackerClient = new TrackerClient();
            // 3、获取trackerServer
            trackerServer = trackerClient.getConnection();
            // 4、获取storageServer
            storageServer = trackerClient.getStoreStorage(trackerServer);

            // 5、创建storage客户端 - 这个对象就是用来上传文件、下载文件、删除文件的
            StorageClient storageClient = new StorageClient(trackerServer, storageServer);

            // 6、上传文件
            /*
                这里有两个API需要了解
                    String[] upload_file(byte[] file_buff, int offset, int length, String file_ext_name, NameValuePair[] meta_list)
                    这个API常用来web中上传文件的
                        参数1 file_buff、文件字节
                        offset、length、从文件的那个位置开始上传，截止位置
                        参数4 file_ext_name、文件后缀
                        参数5 meta_list、文件的属性文件

                    String[] upload_file(String local_filename, String file_ext_name, NameValuePair[] meta_list)
                    这个API是上传本地文件的
                        参数1 local_filename、本地文件的绝对路径
                        参数2 file_ext_name、文件后缀名
                        参数3 meta_list、文件的属性文件，linux1中的哪个meta data，一般都不传
                        
                上述这两个API，注意返回值，这个String[] 很重要，就是涉及到linux中的那个组名group 和 远程文件名remote_filename
             */
            String[] result = storageClient.upload_file("C:\\Users\\ZiXieQing\\Desktop\\图库\\19.jpg", "jpg", null);
            
            // 7、验证一下
            for (String data : result) {
                System.out.println("data = " + data);
            }

        } catch (IOException e) {
            throw new RuntimeException(e);
        } catch (MyException e) {
            throw new RuntimeException(e);
        } finally {
            // 8、释放资源
            if (storageServer != null) {
                try {
                    storageServer.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
            if (trackerServer != null) {
                try {
                    trackerServer.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220602131009377-273499948.png)



- **浏览器访问**

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220602131158122-421046807.png)



### 2.4.2、文件下载

- 把前面的文件上传代码改一下即可，换成另一个API而已

```java
package com.zixieqing;

import org.csource.common.MyException;
import org.csource.fastdfs.*;

import java.io.IOException;

/**
 * @author : ZiXieQing
 * @version : V1.0.0
 * @className : DownloadFile
 * @description ： 该类功能 fastDFS文件下载
 * @packageName : com.zixieqing
 */

public class DownloadFile {

    public static void main(String[] args) {

        TrackerServer trackerServer = null;
        StorageServer storageServer = null;
        try {
            // 1、初始化配置文件
            ClientGlobal.init("fastdfs.conf");

            // 2、获取tracker客户端
            TrackerClient trackerClient = new TrackerClient();
            // 3、获取trackerServer
            trackerServer = trackerClient.getConnection();
            // 4、获取storageServer
            storageServer = trackerClient.getStoreStorage(trackerServer);

            // 5、创建storage客户端
            StorageClient storageClient = new StorageClient(trackerServer, storageServer);

            // 6、下载文件
            /*
                这里需要知道两个API
                     byte[] download_file(String group_name, String remote_filename)
                     这个API常用于web操作

                    int download_file(String group_name, String remote_filename, String local_filename)
                    这个API是把文件下载到本地磁盘中
                    这个API的返回值结果很重要
             */
            String group = "group1";
            String remoteFileName = "M00/00/00/CgAAEGKYRg-AAIrWAAD8cA4U6dY771.jpg";
            // 存入本地磁盘路径+存入磁盘的文件名
            String localFileName = "d:/靓妹.jpg";
            // 只有返回值是0才表示下载成功，否则只要是其他数字都是下载失败( 其他数字有可能是组名错了，远程文件名错了........
            int result = storageClient.download_file(group, remoteFileName, localFileName);

            // 7、验证
            System.out.println("result = " + result);
        } catch (IOException e) {
            throw new RuntimeException(e);
        } catch (MyException e) {
            throw new RuntimeException(e);
        } finally {
            // 8、释放资源
            if (storageServer != null) {
                try {
                    storageServer.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
            if (trackerServer != null) {
                try {
                    trackerServer.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }

    }
}

```

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220602183220360-869930375.png)

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220602183346231-396977430.png)





### 2.4.3、文件删除

```java

package com.zixieqing;

import org.csource.common.MyException;
import org.csource.fastdfs.*;

import java.io.IOException;

/**
 * @author : ZiXieQing
 * @version : V1.0.0
 * @className : DeleteFile
 * @description ： 该类功能 FastDFS删除文件
 * @packageName : com.zixieqing
 */

public class DeleteFile {

    public static void main(String[] args) {

        TrackerServer trackerServer = null;
        StorageServer storageServer = null;
        try {
            // 1、初始化配置文件
            ClientGlobal.init("fastdfs.conf");

            // 2、获取tracker客户端
            TrackerClient trackerClient = new TrackerClient();
            // 3、获取trackerServer
            trackerServer = trackerClient.getConnection();
            // 4、获取storageServer
            storageServer = trackerClient.getStoreStorage(trackerServer);
            // 5、获取storage客户端
            StorageClient storageClient = new StorageClient(trackerServer, storageServer);

            // 6、执行文件删除
            /*
                int delete_file(String group_name, String remote_filename)
                参数1 group_name、组名
                参数2 remote_filename、远程文件名
             */
            // 一样的，返回值是0就表示成功，其他都是删除失败
            int result = storageClient.delete_file("group", "M00/00/00/CgAAEGKYRg-AAIrWAAD8cA4U6dY771.jpg");

            // 7、验证一下
            System.out.println("result = " + result);

        } catch (IOException e) {
            throw new RuntimeException(e);
        } catch (MyException e) {
            throw new RuntimeException(e);
        }finally {
            // 8、释放资源
            if (storageServer != null) {
                try {
                    storageServer.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
            if (trackerServer != null) {
                try {
                    trackerServer.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}

```







# 3、FastDFS集群



> **示例的架构图**

![image](https://img2022.cnblogs.com/blog/2421736/202206/2421736-20220606100814159-1421740184.png)



```txt

FastDFS分布式文件系统集群环境搭建-操作步骤手册

搭建一个FastDFS分布式文件系统集群，推荐至少部署6个服务器节点；
================================搭建FastDFS的集群==============================
第一步：安装6个迷你版的Linux，迷你版Linux没有图形界面，占用磁盘及资源小，企业里面使用的Linux都是没有图形界面的Linux；

第二步：由于迷你版Linux缺少一些常用的工具库，操作起来不方便，推荐安装如下的工具库：
1、安装lrzsz， yum install lrzsz -y
2、安装wget, yum install wget -y
4、安装vim， yum install vim -y
5、安装unzip，yum install unzip -y
6、安装ifconfig，yum install net-tools -y

yum install lrzsz wget vim unzip net-tools -y

7、安装nginx及fastdfs需要的库依赖：
   yum install gcc perl openssl openssl-devel pcre pcre-devel zlib zlib-devel libevent libevent-devel -y

第三步 安装fastdfs 
   1、 上传fastdfs的安装包和libfastcommon的安装包
   2、 解压libfastcommon 安装libfastcommon
   3、 解压fastdfs 安装fastdfs
   4、 拷贝fastdfs目录中的http.conf和mime.types到/etc/fdfs 目录中
注：6台机器全部执行这些操作


第四步：部署两个tracker server服务器，需要做的工作:

    修改两个tracker服务器的配置文件：
    tracker.conf: 修改一个地方：
    base_path=/opt/fastdfs/tracker   #设置tracker的数据文件和日志目录（需预先创建）
    启动tracker服务器 fdfs_trackerd /etc/fdfs/tracker.conf


第五步 修改两个组中的4台storage中storage.conf文件
    第一组group1的第一个storage server（修改storage.conf配置文件）：
    group_name=group1   #组名，根据实际情况修改，值为 group1 或 group2
    base_path=/opt/fastdfs/storage   #设置storage的日志目录（需预先创建）
    store_path0=/opt/fastdfs/storage/files    #存储路径
    tracker_server=192.168.171.135:22122  #tracker服务器的IP地址以及端口号
    tracker_server=192.168.171.136:22122

    第二组group2的第一个storage server（修改storage.conf配置文件）：
    group_name=group2   #组名，根据实际情况修改，值为 group1 或 group2
    base_path=/opt/fastdfs/storage   #设置storage的日志目录（需预先创建）
    store_path0=/opt/fastdfs/storage/files    #存储路径
    tracker_server=192.168.171.135:22122  #tracker服务器的IP地址以及端口号
    tracker_server=192.168.171.136:22122
   
    启动storage服务器
    使用之前的Java代码测试FastDFS的6台机器是否可以上传文件
注意：FastDFS默认是带有负载均衡策略的可以在tracker的2台机器中修改tracker.conf文件
    store_lookup=1

    0 随机存放策略
    1 指定组
    2 选择磁盘空间的优先存放 默认值

    修改后重启服务
    fdfs_trackerd /etc/fdfs/tracker.conf restart












======================使用Nginx进行负载均衡==============================


第六步 安装 nginx ，使用nginx 对fastdfs 进行负载均衡 
    上传 nginx-1.12.2.tar.gz以及 nginx的fastdfs扩展模块安装包fastdfs-nginx-module-master.zip
    添加nginx的安装依赖
       yum install gcc openssl openssl-devel pcre pcre-devel zlib zlib-devel -y
    解压nginx
       tar -zxvf  nginx-1.12.2.tar.gz
    解压fastdfs扩展模块
       unzip fastdfs-nginx-module-master.zip
    配置nginx的安装信息
       2台tracker服务器的配置信息（不需要fastdfs模块）
         ./configure --prefix=/usr/local/nginx_fdfs
       4台storage服务器其的配置信息（需要使用fastdfs模块）
         ./configure --prefix=/usr/local/nginx_fdfs --add-module=/root/fastdfs-nginx-module-master/src
    编译并安装nginx
       ./make
       ./make install

    4台storage的服务器需要拷贝mod_fastdfs文件
    将/root/fastdfs-nginx-module-master/src目录下的mod_fastdfs.conf文件拷贝到 /etc/fdfs/目录下，这样才能正常启动Nginx；



第七步 配置tracker 的两台机器的nginx
    进入安装目录
    cd /usr/local/nginx_fdfs

    添加一个location 对请求进行拦截( 在nginx.conf的server{}前面加上下面的内容即可 )，配置一个正则规则 拦截fastdfs的文件路径， 并将请求转发到其余的4台storage服务器(修改 conf目录下nginx.conf 文件)
    #nginx拦截请求路径：
    location ~ /group[1-9]/M0[0-9] {   
        proxy_pass http://fastdfs_group_server; 
    }



    添加一个upstream 执行服务的IP为 另外的4台stroage 的地址
    #部署配置nginx负载均衡:
    upstream fastdfs_group_server {  
        server 192.168.171.137:80;  
        server 192.168.171.138:80;
        server 192.168.171.139:80;  
        server 192.168.171.140:80;  
    }


第八步 配置另外4台storage的nginx添加http访问的请求路径拦截
    进入安装目录
    cd /usr/local/nginx_fdfs
    添加一个location 对请求进行拦截，配置一个正则规则 拦截fastdfs的文件路径，使用fastdfs的nginx模块转发请求(修改 conf目录下nginx.conf 文件)
    #nginx拦截请求路径：
    location ~ /group[1-9]/M0[0-9] {   
        ngx_fastdfs_module;
    }



第九步 分别修改4台storage服务器的mod_fasfdfs.conf文件（/etc/fdfs/mod_fastdfs.conf）
    #修改基本路径，并在指定路径创建对应文件夹
    base_path=/opt/fastdfs/nginx_mod #保存日志目录
    #指定两台tracker服务器的ip和端口
    tracker_server=192.168.171.135:22122  #tracker服务器的IP地址以及端口号
    tracker_server=192.168.171.136:22122
    #指定storage服务器的端口号
    storage_server_port=23000 #通常情况不需要修改
    #指定当前的storage服务器所属的组名 （当前案例03和04为group1 05和06为group2）
    group_name=group1  #当前服务器的group名
    #指定url路径中是否包含组名 （当前案例url包含组名）
    url_have_group_name=true     #文件url中是否有group名
    store_path_count=1           #存储路径个数，需要和store_path个数匹配（一般不用改）
    store_path0=/opt/fastdfs/storage/files    #存储路径
    #指定组个数，根据实际配置决定，（当前案例拥有2个组group1和group2）
    group_count = 2                   #设置组的个数



    在末尾增加2个组的具体信息：
    [group1]
    group_name=group1
    storage_server_port=23000
    store_path_count=1
    store_path0=/opt/fastdfs/storage/files

    [group2]
    group_name=group2
    storage_server_port=23000
    store_path_count=1
    store_path0=/opt/fastdfs/storage/files

    第一个组的第二个storage按照相同的步骤操作；

    另外一个组的两个storage也按照相同的步骤操作；

    #测试nginx的配置文件是否正确（测试全部6台服务器）
       /usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf -t
    #启动nginx服务器(全部6台服务器)
      /usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf




测试：使用浏览器分别访问 6台 服务器中的fastdfs文件


第十步：部署前端用户访问入口服务器，即访问192.168.230.128上的Nginx，该Nginx负载均衡到后端2个tracker server；
    配置nginx.conf文件
    location ~ /group[1-9]/M0[0-9] {   
        proxy_pass http://fastdfs_group_server; 
    }



    添加一个upstream 执行服务的IP为 2台tracker 的地址( 在nginx.conf的server{}前面加上下面的内容即可 )
    #部署配置nginx负载均衡:
    upstream fastdfs_group_server {  
        server 192.168.171.135:80;  
        server 192.168.171.136:80; 
    }

测试：使用浏览器访问128（唯一入口的nginx服务器）服务器中的fastdfs文件
注意：由于之前128的nginx中可能拥有静态资源拦截会导致访问不到文件，这时可以注释或删除这些静态资源拦截




==============================补充资料============================================
最后，为了让服务能正常连接tracker，请关闭所有机器的防火墙：
systemctl status firewalld   查看防火墙状态
systemctl disable firewalld  禁用开机启动防火墙
systemctl stop firewalld     停止防火墙
systemctl  restart  network  重启网络
systemctl  start network     启动网络
systemctl  stop  network     停止网络

可能安装的linux（无图形的）没有开启网卡服务，可以修改/etc/sysconfig/network-scripts 下的网卡配置文件设置 ONBOOT=yse 
表示开机启动网卡，然后启动网络服务即可

Keepalived当主nginx出现故障后会自动切换到备用nginx服务器的一款软件 通常由运维人员进行使用


```























