# 1.简介

1.docker是什么? 

docker是一个**开源**的**应用容器引擎**，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的linux机器上，也可以实现**虚拟化**

容器是完全使用**沙箱机制**，相互之间不会有任何接口（类似IPhone）。几乎没有性能开销，可以很容易的在机器和数据中心里运行。

不依赖任何语言、框架、系统

开发语言：GO



2.docker解决了什么? 

开发和运维之间以及其他任何环境之间的问题,能够一次成型,到处运行



3.概念:

仓库(repositry)：存放镜像的地方，分为公开仓库和私有仓库

镜像(mirrors) ：其实就是模板，跟我们常见的ISO镜像类似，是一个样板

容器(Container)：使用镜像常见的应用或系统







# 2.安装

1.安装在内核3.10+以上才行

2.windows和linux上均可安装

​	无论是工作还是学习,均不考虑windows上安装

​	centos6.x系列内核不够3.10+,可以升级6的内核,然后安装(升级过程及其痛苦)

​	centos7.x内核够3.10+,所以直接用7即可

3.预备知识

​	安装vm和centos7

​	请吃透理论再完成实践

4.安装

1.自动

```shell
#1.执行脚本
curl -sSL https://get.daocloud.io/docker | sh
#2.安装(yum默认走的是centos的源,如果卡,建议换国内源)
yum install docker-ce -y
#3.启动
systemctl start docker
#4.测试
docker -v
```



2.手动(略)





# 3.使用

## 3.1 基本

```shell
docker version #仅仅打印版本内容
docker info    #有客户端和服务器端的各种信息展示
docker --help  #其实直接docker即可
```

## 3.2 镜像命令

```shell
#1.查看镜像
docker images 
[附录]
参数：
-a：列出本地所有镜像
-q：只显示镜像id
--digests：显示镜像摘要
--no-trunc：显示完整的镜像信息



#2.搜索(docker hub上)是否有要的镜像
docker search 要查找的名

[附录]
stars表示受欢迎程度,类似于收藏数
受欢迎数大于 1000 的镜像:docker search -f stars=1000 tomcat


#3.拉镜像
docker pull 镜像名

[附录]
镜像名默认是最新的(latest)
如果要拉其他版本,采用"镜像名:版本号"的方式

#4.删除镜像
docker rmi -f 镜像名

[附录]
如果删多个就空格隔开(和linux的rm -rf一样)
```

## 3.3 容器命令

### **3.3.1 新建容器**

```shell
#1.语法：docker run [option] IMAGE [command] [args]
option:
--name="容器新名字": 为容器指定一个名称
-d: 后台运行容器，并返回容器ID，也即启动守护式容器
-i：以交互模式运行容器，通常与 -t 同时使用
-t：为容器重新分配一个伪输入终端，通常与 -i 同时使用
-P: 随机端口映射
-p: 指定端口映射，有以下四种格式
      ip:hostPort:containerPort
      ip::containerPort
      hostPort:containerPort
      containerPort
      
#2.案例
#2.1 300e315adb2f是镜像id,可通过docker images来获得
#也可以 docker run -it centos(镜像名)
docker run -it 300e315adb2f

#2.2 centos_study为新镜像的名字 centos为原镜像名
#为什么要取名?  方便知道自定义的那个镜像
docker run -it --name centos_study centos

#2.3 -d 后台运行 取名tomcat_test 分配本机8080对应容器8080端口 镜像采用tomcat8.5版本
docker run -d --name tomcat_test -p 8080:8080 tomcat:8.5
```

### 3.3.2 列出当前正在运行的容器

```shell
#单纯的ps只展示正在运行的容器
docker ps
#工作中常用 docker ps -a来查看所有的容器
```

### 3.3.3 删除'已停止'容器

```shell
docker rm 容器名/容器id

```

### 3.3.4 进入正在运行的容器

通过:docker ps可以查看哪些正在跑的容器

```shell

#1.运用多
docker exec -it 容器id /bin/bash
#2.运用少
docker attach 容器ID

#两种方式的区别
attach 直接进入容器启动命令的终端，不会启动新的进程
exec 在容器中打开新的终端，并且可以启动新的进程
```

### 3.3.5 退出容器

```shell
exit 		#容器停止退出(不多)
ctrl+q+p    #容器不停止退出(多)  用于从容器内切换到外部
```

### 3.3.6 容器启动/重启/停止/强行停止

```shell
#前提:已存在容器
#如何知道已存在哪些容器:docker ps -a
docker start/restart/stop 容器名/id
#强行停止
docker kill 容器名/id
```

### 3.3.7 从容器内拷贝文件到主机上

```shell
docker cp 容器ID:容器内的路径 主机目录
```

### 3.3.8 备份

```shell
docker commit -a='作者' -m='镜像描述'  容器ID  新的镜像名/名称:版本
```



## 3.4 案例

1.安装tomcat

```shell
#1.拉取镜像：
docker pull tomcat:8.5

#2.创建tomcat容器：
docker run -d --name tomcat_test -p 8080:8080 tomcat:8.5

#打开浏览器输入http://linux的ip:8080,就可以.但如访问出现404, 是因为webapps文件夹下内容为空，内容都在webapps.dist 目录下,所以进入容器删除掉webapps,重命名webapps.dist为webapps即可

#3.进入容器：
docker exec -it tomcat_test /bin/bash

#4.删除webapps目录:
rm -rf webapps

#5.重命名目录：
mv webapps.dist/ webapps

#6.再次启动浏览器，输入：http://linux的ip:8080，即可

#[附]可做可不做
#可备份一下，使其成为基础的tomcat，以后直接引用就行：docker commit -a='caichang' -m='tomcat基础镜像' c006dd416916 basic_tomcat:1.0


```

2.安装mysql

```shell
#1.拉取mysql镜像
docker pull mysql:5.7

#2.新建容器
docker run --name=basic_mysql -itd -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root mysql:5.7

```





3.安装nginx

