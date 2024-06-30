GitLab介绍
--------

GitLab 是一个基于 web 的 Git 仓库管理工具，提供了代码托管、版本控制、协作开发、持续集成等功能，是一个综合的 DevOps 平台。用户可以使用 GitLab 托管他们的代码仓库，并利用其丰富的功能来管理和协作开发项目。  
以下是 GitLab 的一些主要特点和功能：

1.  **代码托管：** GitLab 提供了强大的 Git 仓库管理功能，用户可以轻松地创建、克隆、推送和拉取代码，实现团队协作开发。
    
2.  **问题追踪：** 用户可以在 GitLab 中创建和管理问题、任务和缺陷报告，方便团队成员跟踪和解决项目中的各种事务。
    
3.  **持续集成/持续部署 (CI/CD)：** GitLab 提供了内置的 CI/CD 功能，支持自动化构建、测试和部署应用程序，帮助团队实现快速交付和持续集成。
    
4.  **代码审核：** GitLab 提供了代码审核功能，可以进行代码评审、审查和讨论，帮助团队改善代码质量和合作效率。
    
5.  **权限管理：** GitLab 具有灵活的权限管理机制，管理员可以根据需要设置不同用户或团队的访问权限，保护代码和项目的安全性。
    
6.  **集成插件：** GitLab 支持与其他 DevOps 工具和服务的集成，如 Jira、Slack、Kubernetes 等，帮助用户构建完整的开发和部署流程。
    
7.  **自托管选项：** 除了 GitLab 的托管服务之外，用户还可以选择在自己的服务器上部署 GitLab，实现自主控制和定制化需求。
    

GitLab 的功能丰富且易于使用，适用于个人开发者、小型团队和大型企业，帮助他们更高效地管理代码、协作开发并实现持续交付。GitLab 的开源版本和商业版本提供了不同的功能和服务，满足了不同用户的需求。  
官网地址：https://about.gitlab.com/solutions/devops-platform/

GitLab安装
--------

### 下载镜像

```bash
docker pull gitlab/gitlab-ce:latest
```



### 创建gitlab容器

```bash
# 创建容器
docker run -d  \
 -p 8888:80 -p 1024:22 -p 443:443 \
 -v /opt/gitlab/config:/etc/gitlab  \
 -v /opt/gitlab/logs:/var/log/gitlab \
 -v /opt/gitlab/data:/var/opt/gitlab \
 --restart always \
 --privileged=true \
 --name gitlab \
 gitlab/gitlab-ce
```



### 进入容器中查看GitLab启动状态

进入容器

```bash
docker exec -it gitlab bash
```

查看状态容器中GitLab的运行状态

```bash
gitlab-ctl status
```

效果如下：  

![image-20240630192612243](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630192615586-1245693854.png)




启动需要的时间较长，当看到所有`15个服务`都是run状态之后才能成功访问

查看日志

```bash
# 查看GitLab所有的logs，用于观察GitLab是否启动完成 按 Ctrl-C 退出
gitlab-ctl tail
```

> 注意：gitlab依赖的服务较多，启动所需内存官方建议为4G以上，不要立马就访问，要等两三分钟左右

等启动完毕，浏览器访问：http://192.168.100.132:8888

!

配置SSH访问端口
---------

```bash
# 进入容器
docker exec -it gitlab /bin/bash


# 编辑gitlab.rb文件
vi /etc/gitlab/gitlab.rb
# 进入文件编辑，跳转到文件末位
G

# 把下面这3个配置放到文件末尾
# 配置http协议所使用的访问地址，可以写域名，默认端口为80【容器内部端口】
external_url 'http://192.168.72.120'

# 配置ssh协议所使用的访问地址和端口 # 此端口是run时22端口映射的1024端口，即ssh链接端口
gitlab_rails['gitlab_ssh_host'] = '192.168.72.120'
gitlab_rails['gitlab_shell_ssh_port'] = 1024


##################可配置的另外内容##################################
# 时区
gitlab_rails['time_zone'] = 'Asia/Shanghai'
# 开启备份功能
gitlab_rails['manage_backup_path'] = true
# 备份文件的权限
gitlab_rails['backup_archive_permissions'] = 0644
# 保存备份 60 天
gitlab_rails['backup_keep_time'] = 5184000




# 重新配置gitlab
gitlab-ctl reconfigure
```

下面命令不用，了解即可

```bash
# 拉取/var/log/gitlab下子目录的日志
gitlab-ctl tail gitlab-rails

# 拉取某个指定的日志文件
gitlab-ctl tail nginx/gitlab_error.log

# 启动 gitlab 服务
gitlab-ctl start

# 停止 gitlab 服务
gitlab-ctl stop
```

修改账号密码
------

初始化 gitlab 中的 root 账号密码：  
1、登录容器

```bash
docker exec -it -u root gitlab /bin/bash
```

2、登录GitLab的Rails控制台

```bash
cd /opt/gitlab/bin

# Rails控制台
gitlab-rails console
```

等待一段时间后，可以在控制台中输入命令 

3、定位到root用户

```bash
# 最好手动输入	直接粘贴可能会出现莫名错误
user = User.where(id: 1).first
```

![image-20240630193803383](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630193805353-810202034.png)

![image-20240630194027419](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630194028290-2088601126.png)

4、修改root密码

```bash
# 执行下面的命令，将 xxxxx 改为自己的密码
# 注意：密码至少需要8个字符
user.password='zixieqing072413'

# 修改后，保存用密码
user.save
```

5、下面命令不用，了解即可

```bash
# 查询所有的用户
user = User.all
# 通过条件查询用户 常见的where条件有 username email state 
user = User.where(id:1).first
user = User.find_by(email: 'admin@local.host')
# 通过id查询用户
user = User.find(1)

# 查询用户某个字段的值 显示当前用户的email
user.email

# 修改密码
user.password = 'zixieqing072413'
user.password_confirmation = 'zixieqing072413'
user.save
echo 'user = User.find_by(username: "root");user.password="secret_pass!";user.password_confirmation="secret_pass!";user.save' | sudo gitlab-rails console

# 修改用户状态
user.state = 'active'
user.save
```



GitLab使用\[管理员\]
---------------

> 一般这些配置是需要公司的管理员去配置的，初级开发人员是没有这个权限的。

### 修改显示主题为中文

在主页面左上角用户头像上点击，选择`Edit profile`


选择`Preferences`菜单，滚动到下面的`Localization`组的`Language`，设置语言为`Chinese,Simplified-简体中文(98%translated)`

点击`Save changes`保存修改

修改语言为中文后，刷新网页就能看到页面全部修改为中文了



### 取消用户自动注册功能

点击`管理中心` -> `仪表盘` -> `己启用注册功能` 去掉勾选，然后拉到下面点击`保存更改`



### 创建用户

创建用户cxypa01，点击`仪表盘` -> `新建用户`

填写用户账号信息，点击`创建用户`

创建后点击`编辑`

添加用户的密码



### 创建组

`群组`可理解为企业中的`开发小组`，每个小组可能负责固定的几个项目

点击`仪表盘` -> `新建群组`

输入相关信息，点击`创建群组`

![image-20240630171752265](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180003281-48290621.png)



### 将用户添加到组中

点击`群组`，点击群组名`testGroup`，进入之前创建的组中

点击`管理权限`

点击`邀请成员`

在邀请成员对话框中输入相关信息

![image-20240630171952995](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000128-175742454.png)





### 给用户组创建项目

点击`testGroup`群组，点击右侧`创建新项目`

点击`创建空白项目`

在`创建空白项目`中输入项目名称，不要勾选`使用自述文件初始化仓库`

完成项目创建



SSH 协议
------

![image-20240630172140313](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180003172-2126420324.png)


GitLab 可以通过 HTTP 的方式可以上传和下载代码，这种方式可以用户的账号和密码。此方式在这里就不过多介绍了，下面将介绍使用 SSH 协议来操作 GitLab 上的项目。

### SSH协议概述

SSH 为 Secure Shell （安全外壳协议）的缩写，由 IETF 的网络小组（Network Working Group）所制定。SSH 是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。

### 基于密匙的安全验证

使用 SSH 协议通信时，推荐使用基于密钥的验证方式。你必须为自己创建一对密匙，**并把公用密匙放在需要访问的服务器上。**如果你要连接到 SSH 服务器上，客户端软件就会向服务器发出请求，请求用你的密匙进行安全验证。服务器收到请求之后，先在该服务器上你的主目录下寻找你的公用密匙，然后把它和你发送过来的公用密匙进行比较。如果两个密匙一致，服务器就用公用密匙加密“质询”（challenge）并把它发送给客户端软件。客户端软件收到“质询”之后就可以用你的私人密匙解密再把它发送给服务器。

### SSH密钥生成

密钥生成的方式有很多种，常见的JDK的工具或者是GitBash等。本教程使用GitBash

在GitBash执行命令，生命公钥和私钥

```bash
ssh-keygen -t rsa
```


执行命令完成后，在window本地用户.ssh目录（C:\\Users\\用户名.ssh中），生成如下名称的公钥和私钥

![image-20240630172312196](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180002719-1402167047.png)





### SSH密钥配置

密钥生成后需要在GitLab上配置密钥本地才可以顺利访问

SSH密钥配置操作步骤：GitLab头像 ——》偏好设置 ——》 SSH密钥，将上面本地中生成的公钥添加进入即可

设置成功后， 我们就可以使用SSH的形式上传和下载代码了

在windows中使用`Git Bash`命令行测试是否能够使用ssh连接服务器，命令如下：

```bash
ssh -T Git@192.168.100.132 -p 1024
```

看到`Welcome to GitLab, @root!`说明成功了!

上传项目至GitLab\[管理员\]
------------------

使用 IDEA 中把项目上传到 GitLab 中，无需其他的插件。这种方式不仅可以向 GitLab 中上传，也可以向 Gitee 和 Github 中上传。

1.  点击 VCS，创建本地仓库`Create Git Repository`

2.  选择项目的根目录，作为 Git 本地仓库的根资源库

3.  向项目的根路径下添加 `.gitignore` 文件忽略不要Git仓库管理的文件

4.  选择要提交到本地仓库中的文件

5.  将项目上传到远程仓库 GitLab 中

选中项目的根目录，在 Git 菜单栏中选择 Push

![image-20240630172809247](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180001913-2140861313.png)

6.  点击 `Define remote` 在弹出窗口中的 URL 里填写之前的项目地址

注意：保证远程仓库是一个新的空仓库，否则提交失败

7.  点击 Push 进行提交

8.  检查 Git 中是否已经上传到远端仓库地址中

![image-20240630173106635](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180001062-309242632.png)

9.  远端仓库中上传后项目的内容

![image-20240630173159315](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180001857-1618073679.png)


以上将自己的项目上传到了 GitLab 组中的项目中，只有本组的人员才可以看到此项目中的内容

新建和上传分支\[管理员\]
--------------

### 新建分支

![image-20240630173454934](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630175959524-1231604279.png)



### 上传分支

![image-20240630173524994](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180001602-1121642217.png)



添加上传信息，例如：

![image-20240630173559074](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000770-474717447.png)





新员工到公司\[新员工\]
---------------

模拟新员工到公司，在GitLab上删除之前的SSH秘钥，退出的root用户

使用新建的用户登录GitLab\[开发人员\]

第一次登录后会提示修改密码，修改密钥即可



### SSH密钥生成与配置\[新员工\]

在`Git bash` 执行下面命令，生成公钥和私钥

```bash
ssh-keygen -t rsa
```

生成密钥后，需要在 GitLab 上配置密钥，本地才可以顺利访问 GitLab

操作方式和前面一样：GitLb头像 ——》 偏好设置 ——》 SSH密钥 ——》 粘贴生成的公钥



### 使用IDEA从GitLab拉取代码\[新员工\]

在IDEA中关闭项目，回到欢迎界面，选择`Get from VCS`

到GitLab上复制项目的地址

输入GitLab对应项目的地址，选择本地保存项目的路径，点击`Clone`


就可以看到代码拉取到本地了

如果使用HTTP协议拉取代码，第一次拉取代码会显示需要输出账号和密码，那输入GitLab的账号和密码



### 把远程分支拉到本地\[新员工\]

刚从远程仓库把代码拉取到本地，本地只有一个master分支。需要把远程分支拉取到本地

点击`远程分支名`，再选择`Checkout`，就可以把远程分支拉取到本地



### 使用IDEA创建新分支开发\[新员工\]

假设`张三`接到了开发`订单功能`的任务，张三需要创建一个新分支`zhangsan_order`分支开发订单功能，这样不会影响其他的人开发。当订单功能开发完成后就需要合并到`dev`分支。  
张三创建一个新分支`zhangsan_order`分支开发订单功能

在`zhangsan_order`分支提交代码

张三把订单功能`zhangsan_order`分支合并到`dev`分支【要合并到哪个分支就切到哪个分支，然后合并要合并进去的分支即可】，有冲突就解决冲突，见后续内容

![image-20240630174148436](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180003190-1547742413.png)



![image-20240630174049123](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180003115-817595886.png)



推送`dev`分支到远程仓库

![image-20240630174303947](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000739-1562114989.png)



在远程仓库的`dev`分支可以看到订单功能的代码



Git开发常见问题
---------

### commit前回滚\[新员工必会\]

假设用户添加了一行代码，但是**还没有提交到本地仓库**

![image-20240630174503472](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180003210-1420612536.png)

在commit窗口中右键选择要回滚的文件，点击`Git`，再点击`Rollback`

![image-20240630174602559](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180003200-836979110.png)

回滚后，多添加的但是未提交的代码没有了



### commit后回滚\[新员工必会\]

commit后回滚有两种方式：`Undo Commmit`和`Revert Commit`

`Undo Commmit`：撤销上次提交，保留修改的内容到新的changelist中，Git log中无任何提交信息

`Revert Commit`：直接回滚到上个版本，不保留修改的内容，在Git log中会留下上次commit和本次revert的信息

#### Undo Commmit 操作

在Git log的提交日志上右键，选择`Undo Commit`

![image-20240630174816421](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000136-1183816161.png)



点击`OK` 



`Undo Commmit`操作后，在Git log中就没有了这次提交

![image-20240630174930706](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630175959228-1981899691.png)

但是代码还是存在的，需要`Rollback`回滚代码：文件右键 ——》 Git ——》 Rollback

这样代码还原了，Git log也没有了

![image-20240630175219009](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630175959334-594567750.png)



#### Revert Commmit 操作

先提交测试代码

![image-20240630175258794](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180001872-110695333.png)

在Git log提交日志上右键，选择`Revert Commit`

![image-20240630175323714](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630175959975-1959170482.png)



提交的代码撤回了，但是Git log提交日志中多了`Revert"commit后回滚测试2！提交"`

![image-20240630175408832](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000232-342811548.png)



### push后回滚\[新员工必会\]

假设用户不小心push到了服务器

在Git log提交日志中，选择要撤回的到的提交日志，右键，点击`Reset Current Branch to Here`  

![image-20240630175457445](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180002909-1117781987.png)

在弹出窗中选择soft或者hard

区别：hard彻底回滚，不保留修改的内容；soft回滚到commit前的状态，修改的内容被保留在changelist中

这里选择`Soft`

修改完成之后使用`Force Push`强制推送到远程仓库【注意：别选`push`推送，而是`Force Push`】

到远程仓库查看，就可以看到，之前的提交被覆盖掉了

如果IDEA的`Force Push`按钮不能点击，则使用Git命令强行覆盖远程分支`Git push origin 分支名 --force`

```bash
# 示例
Git push origin master --force
```

冲突解决\[新员工必会\]
-------------

两个人在同一个类的同一行添加代码并先后提交到远程仓库

![image-20240630175732319](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180004092-1766681744.png)



产生了冲突，解决冲突

![image-20240630175816689](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000426-1098639576.png)



![image-20240630175831947](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000111-794628767.png)



![image-20240630175856601](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180001963-1022179649.png)



解决冲突后的效果：  

![image-20240630175914102](https://img2023.cnblogs.com/blog/2421736/202406/2421736-20240630180000924-1384077968.png)























