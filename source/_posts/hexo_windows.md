---
title: Windows Server搭建私有gitblit服务器托管Hexo博客
date: 2020-04-06 02:14:14
tags: Hexo
categories: Hexo
cover: /img/hexo.jpg
---
# Windows Server搭建私有gitblit服务器托管Hexo博客

## 前言

　　网上关于Hexo博客的部署教程很多，大都是托管在Github。虽然免费，但由于Github在外网，访问速度较慢。如果博客上有许多高清图片，访问时要等好一会才能加载出来，严重影响观感。

　　由此，萌生了在私有云服务器上托管Hexo博客的想法。网上找到的教程是在Linux系统服务器上搭建的，而我的云服务器系统是Windows Server 2012 R2（工作原因必须用），能不能按照同样的思路在Windows Server上搭建呢？

　　下面，跟大家分享我在Windows Server 2012 R2上搭建Git私库，配置Niginx静态资源Web服务，成功托管Hexo博客的经验。



### 主要思路：

1.  在云服务器使用Gitblit搭建Git私库
2.  搭建Nginx Web服务器，访问静态资源
3.  配置Git钩子脚本，每次推送，自动将Hexo静态资源拷贝到Nginx默认主页目录



## 详细步骤

### 使用Gitblit搭建Git私库

#### 安装 JRE

Gitblit 需要 Java运行环境，版本至少Java 7，JRE和JDK都行，为了精简我选的JRE 8

JRE 官方下载地址（注册后才能下载）： https://www.oracle.com/java/technologies/javase-downloads.html



#### 安装Gitblit

##### 官网下载Windows版压缩包：http://gitblit.github.io/gitblit/

<img src="Gitblit.JPG" style="zoom:70%;" />



##### 解压后，修改 gitblit-1.9.0/data/defaults.properties 配置文件中的以下几项参数：

```properties
# Git总仓库目录(会自动创建，但只能在gitblit根目录下)
# 我定义在 C:\Program Files\gitblit-1.9.0\gitRepository
git.repositoriesFolder = gitRepository
# HTTP协议端口号（自定义端口号，记得在云服务器的防火墙开放自定义的端口，TCP协议即可）
server.httpPort = 12070
# HTTP协议传输数据的接口
#（默认localhost，网上大都教填公网IP，但我填了之后启动报错闪退，找到的解决办法是填空）
# 解决方法：https://blog.csdn.net/wwongcong/article/details/88567314
server.httpBindInterface = 
```



##### 以管理员模式运行Gitblit根目录下的 gitblit.cmd 开启服务

浏览器访问：http://localhost:自定义端口号/　或　http://公网IP:自定义端口号/

即可进入Gitblit使用界面

![](gitblit_cmd.JPG)



##### 将Gitblit服务器注册成系统服务，开机自启动

修改Gitblit根目录下的 installServcie.cmd 文件中的几项参数：

```properties
# 32位系统：x86 64位系统：amd64 
SET ARCH=amd64
# Gitblit根目录
SET CD=C:\Program Files\gitblit-1.9.0
# 修改StartParams为空
--StartParams="" ^
```

<img src="installService.JPG" style="zoom:70%;" />



修改完参数后，以管理员运行installService.cmd 安装服务

然后打开 运行＝> services.msc

就能在Windows服务管理策略里看到新注册的gitblit服务



<img src="gitblit_service.JPG" alt="image-20200405011845990" style="zoom:80%;" />



<img src="gitblit_service2.JPG" alt="image-20200406012825794" style="zoom:80%;" />



#### 创建Git仓库

##### 登录Gitblit管理员账号，修改密码

初始账号：admin，初始密码：admin

![](gitblit_login.JPG)



一定要修改初始密码并牢记！！！

今后通过http协议推送Git仓库需要用到该密码，相当于Github的账号密码



![](gitblit_password.JPG)



##### 创建Git仓库

仓库名自定义，无需像托管在Github上时一样规定为username.github.io

<img src="gitblit_create.JPG" alt="image-20200403212807119" style="zoom:60%;" />



##### 获取Git仓库地址

这里我选用http协议地址，是因为懒得在服务器防火墙再开放一个29418端口了

想要用ssh的，劳烦移步官网文档 http://gitblit.github.io/gitblit/setup_transport_ssh.html

<img src="gitblit_repository.JPG" alt="image-20200403212807119" style="zoom:80%;" />




#### 至此，Git私库搭建完成！



### 搭建Nginx Web服务器

#### 安装Nginx

官网下载Windows版压缩包：http://nginx.org/en/download.html

<img src="nginx_download.JPG" alt="image-20200405013330255" style="zoom:70%;" />



#### 修改配置文件

修改Nginx 根目录的conf文件夹下的配置文件  `C:\Program Files\nginx-1.17.9\conf\nginx.conf`

<img src="nginx_conf.JPG" alt="image-20200406014354829" style="zoom:80%;" />



#### 启动测试Nginx

双击nginx.exe，黑色弹窗一闪而过即为启动成功，关闭需在任务管理器结束进程。

访问测试页面 `http://localhost/`

<img src="nginx_test.JPG" alt="image-20200406013403238" style="zoom:90%;" />



#### 配置开机启动

##### 下载WinSW

WinSW下载地址： https://github.com/kohsuke/winsw/releases

WinSW.NET2.exe （适用于32位系统）

WinSW.NET4.exe （适用于64位系统。我下载这个版本）



##### 配置步骤：

###### 将 `WinSW.NET4.exe` 复制到 `C:\Program Files\nginx-1.17.9\` Nginx 根目录中，重命名为 `nginxservice.exe`



###### 在与 `nginxservice.exe` 同目录中，新建一个 `nginxservice.xml` 文件（名字一定要与`nginxservice.exe` 名字前缀保持一致） 

`nginxservice.xml` 编写以下内容：

```
<service>
	<id>nginx</id>
	<name>nginx</name>
	<description>nginx</description>
	<logpath>C:\Program Files\nginx-1.17.9</logpath>
	<logmode>roll</logmode>
	<depend></depend>
	<executable>C:\Program Files\nginx-1.17.9\nginx.exe</executable>
	<stopexecutable>C:\Program Files\nginx-1.17.9\nginx.exe -s stop</stopexecutable>
</service>
```

请自行修改<logpath>、<executable>、<stopexecutable>为自己的Nginx 根目录

<img src="nginx_service.JPG" alt="image-20200406011927319" style="zoom:90%;" />



用 **管理员身份** 运行 cmd ，cd 进入 `C:\Program Files\nginx-1.17.9` Nginx 根目录下，执行 `nginxservice.exe install` 命令。

![image-20200406012420521](nginx_service2.JPG)



<img src="nginx_service3.JPG" alt="image-20200406012538546" style="zoom:80%;" />



### 配置Git钩子脚本

#### 原理：

　　Git钩子是在Git仓库中特定事件发生时自动运行的脚本。

　　这里我们需要配置一个脚本，每当我们向Git服务器推送Hexo博客静态资源文件时，脚本自动拷贝上传的的资源文件到Nginx Web服务器目录。

　　有小伙伴可能会提出，直接把Nginx Web服务器的默认主页目录指定到仓库目录不就行了吗？

​		这有一个误区，如果你翻找过服务端上的仓库目录，会发现根本找不到你所推送的文件！

　　这是因为Git是一个版本管理系统，而非文件系统。Git服务端存放的叫裸库（bare），不包含工作区（working place），推送到Git服务端的文件会以二进制的方式存储在仓库的objects文件夹中。

　　你需要在服务器上，像平时克隆仓库一样执行　git clone 仓库地址　才能获取到推送的文件。

![](git_file.JPG)

　　我们要配置的钩子脚本就是要实现以上功能，而且并不需要额外安装 Git。



#### 详细步骤：

##### 修改现成的钩子脚本

Gitblit 钩子脚本都放在 gitblit-1.9.0\data\groovy 目录下，要使用的脚本模板为localclone.groovy

拷贝一份，重命名为autoclone.groovy

修改 autoclone.groovy 中的：

```properties
# 自动在此目录执行克隆代码，我这直接指向了Nginx的默认主页路径
def rootFolder = 'C:/Program Files/nginx-1.17.9/html/'
```

##### 给仓库配置脚本

浏览器进入Gitblit 管理界面 ＝> blog版本库 ＝> 编辑＝> receive＝> post-receive

选择刚才编写的autoclone脚本，保存

<img src="git_script.JPG" alt="image-20200404025524747" style="zoom:70%;" />



## 最后

　　在用于推送博客的电脑上，修改配置文件_config.yml，添加多Git仓库推送

```
deploy:
  type: git
  repository:
    github: https://github.com/username/username.github.io.git
    gitblit: http://admin@公网IP:自定义端口/r/仓库名.git
  branch: master
```



　CMD执行hexo clean，hexo deploy命令

　因使用http协议，推送到Gitblit时会提示输入密码，密码就是登录Gitblit管理页面的admin用户密码。

　现在浏览器直接访问公网IP就能看到博客了，高清图片即时呈现，丝滑流畅~~~



<img src="myblog.JPG" alt="image-20200406020350893" style="zoom:60%;" />