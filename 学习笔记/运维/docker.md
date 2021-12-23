# Docker

## Docker安装

> https://docs.docker.com/engine/install/centos/

## Docker常用命令

>  https://docs.docker.com/engine/reference/run/

### 帮助命令

```--shell
--help
```

### 镜像命令

**docker images**

```shell
-a  # 列出所有镜像
-q  # 只列出id
```

**docekr search**

**docker pull**

**docker rmi**  删除镜像

**docker history** 查看镜像构成过程

```shell
docker rmi -f     # 删除指定（多个）容器 
docker rmi -f $(docker images -aq)   # 删除全部容器
```

### 容器命令

**docker run** 

```shell
--name = ""  # 容器名字
-d			 # 后台运行
-it			 # 使用交互方式运行，进入容器
-p			 # 指定容器端口 -p 8080:8080
	-p ip:主机端口：容器端口
	-p 主机端口：容器端口
	-p 容器端口
-e			 # 配置环境（如mysql的账户密码）
-v			 # 数据卷挂载（双向绑定）
```

**docker ps** 

列出所有运行的容器

```shell
docekr ps 
-a			# 当前运行的容器+历史运行的容器
-n=?		# 显示最近创建的容器  ？是数字 
```

**exit**

``` shell
# 停止容器并退出
# Ctrl+P+Q 不停止退出
```

**docker rm**

``` shell
docker rm 容器id      # 删除指定容器，不能删除正在运行的
docker rm -f		 # 强制删除
docker rm -f $(docker ps -aq)  #删除所有
```

**容器启动停止**

```shell
docker start
docker restart
docker stop
docker kill
```

### 常用其他命令

**后台启动容器**

``` shell
docker run -d centos
# 发现centos停止了

# 坑：docker容器使用后台运行，必须要有一个前台进程，没有的话，会自动停止  
```

**查看日志**

```shell
-tf			# 显示日志
--tail	3	# 要显示日志条数
docker logs -tf --tail 10 dece123hjkasdfu
```

**查看容器中进程信息**

```shell
docker top 容器id
```

**查看镜像元数据**

```shell
docker inspect 容器id
```

**进入当前运行的容器**

```shell
docker exec -it {容器id} {bashshell}  #开启一个新的终端，可以在里面操作

# 方式二
docker attach {容器id}    #进入正在执行的终端，不会启动新的进程
```

**从容器内拷贝文件到外面**

```shell
docker cp {容器id}:{原路径} {目标路径}
```

**访问网址**

```shell
curl www.badiu.com
curl localhost:8080
```

### 小结

![image-20210916160621384](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20210916160621384.png)



### 可视化

- portainer

- Rancher（CI/CD）



**portainer：**

Docker图形化界面管理工具

```shell
docker run -d -p 8090:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

![image-20210917142834537](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20210917142834537.png)

## Linux命令

从一个服务器进另一个服务器

```shell
ssh {ip}
```

从一个服务器上传文件到另一个服务器

```shell
scp /iids/docker-compose/iids-hub-tias/iids-hub-tias.jar root@172.18.39.149:/iids/docker-compose/
```

进入另一个终端（docker或者mysql）

```shell
# Shell中通常将EOF与 << 结合使用，表示后续的输入作为子命令或子Shell的输入，直到遇到EOF为止，再返回到主调Shell。可以把EOF替换成其他东西，意思是把内容当作标准输入传给程序

<<EOF

EOF
```



## Docker镜像

####  分层理解

RootFS

镜像层（pull下来），容器层（run之后，我们的操作）

### commit镜像

```shell
docker commit 提交容器成为一个新的副本

docker commit -m="描述信息" -a="作者" 容器id 目标镜像名: [TAG]
```



## 容器数据卷

需求：MySQL数据可以存储在本地，实现持久化

容器间也能数据共享

### 使用数据卷

> 方法一：使用命令来挂载

```shell
docker run -it -v 主机目录：容器内目录	-p -e
```

**其实实现了双向绑定，同时在本地持久化了相应的数据，即容器删除后，数据依然在**

### MySQL启动命令

```shell
docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7   #mysql
```

### 具名挂载和匿名挂载

```shell
# 匿名挂载
-v 容器内路径

# 查看所有volume的情况
docker volume ls
local     4d2886271dd57af23492fc47bfdf9abcdfbe3cac0f2be83074442e12a2889467
# 这种就是匿名挂载，只在-v里写了容器内的路径，没有容器外的

# 具名挂载
-v 卷名:容器内路径 # 这里卷名不以/开头，否则是绝对路径了

# 所有docker容器内的卷，没有指定目录的情况下都是在  /var/lib/docker/volumes/xxxx/_data 
# 大多数情况下使用具名挂载
```

拓展：

```shell
# 通过 -v ... 容器内路径: ro rw 改变读写权限
ro	 # 只读， 只能通过宿主机来操作，容器内部是无法操作的
rw	 # 可读可写，默认

# 一旦这个设置了容器权限，容器对我们挂载出来的内容就有限定了
```



### 初始Dockerfile

Dockerfile就是构建docker镜像的构建文件

每个命令就是一层镜像



### 数据卷容器

 多个mysql同步数据

```shell
--volumes-from {容器名} # 实现容器间所有数据共享，只要一个容器在，这个文件就不会丢失

# 启动三个容器
```

### 结论

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止。

但是只要持久化到本地，本地数据不会删除



## DockerFile

### DockerFile介绍

dockerfile是用来构建docker镜像的文件，命令参数脚本。

构建步骤:

1、编写dockerfile

2、docker build构建成为一个镜像

3、docker run 运行镜像

4、docker push 发布镜像



很多官方镜像都是基础包



### Dockerfile构建过程

**基础知识：**

1、每个指令都是大写字母

2、执行从上到下顺序执行

3、# 表示注释

4、每个指令都会创建提交一个新的镜像层，并提交

dockerfile是面向开发的，发布项目，做镜像，就需要编写dockerfile

### Dockerfile的指令

```shell
FROM               # 基础镜像，从这里开始
MAINTAINER		   # 镜像作者，姓名+邮箱
RUN				   # 镜像构建时需要运行的命令
ADD				   # 添加内容，比如tomcat镜像，的压缩包
WORKDIR			   # 镜像的工作目录
VOLUME			   # 挂载目录
EXPOSE			   # 对外暴露端口
CMD				   # 指定容器启动的时候运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		   # 指定容器启动的时候运行的命令，可以追加命令
ONBUILD    		   # 当构建一个被继承 DockerFile 这个时候就会运行该指令
COPY			   # 类似ADD，文件拷贝到镜像
ENV  			   # 构建时的环境变量
```

### 实战测试

> 创建一个自己centos

```shell
# 1 编写Dockerfile的文件
[root@iids01 dockerfile]# vim dockerfile-zjr
FROM centos
MAINTAINER zjr<163@.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "---end---"
CMD /bin/bash

# 2 构建镜像
# docker build -f dockerfile文件路径 -t 镜像名:[tag] .
docker build -f dockerfile-zjr -t mycentos:0.1 .

```

> CMD 和 ENTRYPOINT 区别

```shell
CMD				   # 指定容器启动的时候运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		   # 指定容器启动的时候运行的命令，可以追加命令
```

测试cmd

```shell
FROM centos
CMD ["ls","-a"]
docker run {} -l  # 报错
# 想追加一个 -l  必须得ls -al 才能替换掉 ls -a

FROM centos
ENTRYPOINT ["ls","-a"]
docker run {} -l  # 成功打印文件详细信息，直接拼接到 ls -a后面
```

官方命名Dockerfile，build会自动加载该文件



## Docker网络

### 理解Docker0

![image-20211009154045744](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211009154045744.png)

lo: 本机回环地址

eth0：内网地址

docker0：docker0地址

> 三个网络

```shell
# docker如何处理容器网络访问的？

docker exec -it tomcat01 ip addr
ping xxxx
# linux 可以ping通docker容器内部
```

> 原理

1、我们每启动一个docker容器，docker就会给容器分配一个ip，只要装了docker，就会有一个docker0网卡

桥接模式，使用的技术evth-pair

2、再启动一个容器，发现又多了一对网卡

```shell
# 我们发现这个容器带来网卡，都是一对对的
# evth-pair就是一对的虚拟设备接口，成对出现
# 可以使用evth-pair充当一个桥梁
```



### --link

单向连接另一个容器

cat  /etc/hosts 可以看到增加了容器地址映射

> --link原理 就是在hosts中增加   ip  容器名的映射
>
> 现在已经不建议使用--link了



### 自定义网络

> 查看所有docker网络

docker network ls

#### 网络模式

bridge： 桥接 （默认，自定义也使用）

none： 不配置网络

host： 和宿主机共享网络

container： 容器网络连通（少）

#### 测试

```shell
# 我们直接启动的命令 --net bridge ，这个就是docker0
docker run -d -p --name tomcat01 tomcat
docker run -d -p --name tomcat01  --net bridge  tomcat
# docker0特点： 默认，域名不能访问 --link后可以

# 自定义网络
# --driver bridge
# --subnet 192.168.0.0/16
# --gateway 192.168.0.1
[root@iids01 etc]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
dec13f912f8aab64559e4b9b0deb7a1bea2cb550f8b19ba84ffc2b5c27d26f0b
[root@iids01 etc]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
c8d855202c39   bridge    bridge    local
7ff37835249d   host      host      local
dec13f912f8a   mynet     bridge    local
43202bae7f4b   none      null      local
```

自己创建的网络就好了，不需要--link就可以ping名字

![image-20211013110227906](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211013110227906.png)

#### 好处

redis、mysql不同的集群使用不同的网络，保证集群是安全和健康的



```shell
# -P 随机指定端口 -p 指定确定端口
[root@iids01 etc]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
[root@iids01 etc]# docker run -d -P --name tomcat-net-02 --net mynet tomcat

[root@iids01 /]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.076 ms

# 证明这俩容器不需要--link也可以通过容器名进行互联
```

![image-20211013151725971](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211013151725971.png)

#### 在容器（Linux）中安装软件工具包

```shell 
# 工具包：vim，iputils-ping。。。  
# -y 安装过程中全部yes
方法1：yum [-y] install {工具名}

方法2：apt-get install [-y] {工具名}
```



![image-20211013151121682](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211013151121682.png)

![image-20211013151136435](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211013151136435.png)

### 网络连通

![image-20211013153522879](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211013153522879.png)



```shell
# 测试打通 tomcat01 - mynet
docker network connect mynet tomcat01 
# 一个容器两个ip地址

 # 连通成功
[root@iids01 ~]# docker exec -it tomcat-net-01 ping tomcat01
PING tomcat01 (192.168.0.4) 56(84) bytes of data.
64 bytes from tomcat01.mynet (192.168.0.4): icmp_seq=1 ttl=64 time=0.077 ms

```

### 实战：部署Redis集群





## SpringBoot微服务打包Docker镜像

1、构建项目

2、打包jar项目

3、编写dockerfile

4、构建镜像

```shell
docker build -t {name} .
```

5、发布运行



# Docker Compose

Docker Compose轻松高效管理容器。定义运行多个容器

> 官方介绍三步骤

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. 



A `docker-compose.yml` looks like this:

```shell
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"  # 当前目录使用Dockerfile构建自己的镜像
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis # 使用官方的镜像
volumes:
  logvolume01: {}
```



Compose：重要概念

- 服务services。容器、应用（web、redis、mysql）
- 项目project。（博客、web、mysql）



## 安装

1、下载

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 国内镜像

```

2、授权

手动cp

 [docker-compose](..\..\..\fsdownload\docker-compose) 

```
 sudo chmod +x /usr/local/bin/docker-compose
```

3、测试

```
 docker-compose --version
```



## Getting Start

https://docs.docker.com/compose/gettingstarted/



1. 应用app.py

2. Dockerfile 应用打包为镜像
3. Docker-compose.yml文件
4. docker-compose up



## yaml规则

```yaml
# 3层

version: '' # 版本
services: # 服务
	服务1: web
volumes:
	
```

 

# Docker Swarm

## 基本命令

初始化节点

```shell
docker swarm init
```

![image-20211127223822173](C:\Users\12589\Desktop\Better\学习笔记\运维\docker.assets\image-20211127223822173.png)

加入一个节点

```shell
docker swarm join
```

获取令牌

```shell
docker swarm join-token manager
docker swarm join-token worker
```

在节点执行,加入集群

![image-20211127231242732](C:\Users\12589\Desktop\Better\学习笔记\运维\docker.assets\image-20211127231242732.png)

节点降级，由管理节点降级为工作节点：

```sh
docker node demote ${ID}
```

删除节点：

```sh
docker node rm ${ID}
```

节点退出Swarm集群，在工作节点运行命令：

```sh
docker swarm leave ${ID}
```

## Node Label 管理

示例集群信息：

![img](https://img2018.cnblogs.com/i-beta/1577453/202002/1577453-20200229115659267-645516038.png)

- **添加标签**

```
docker node update --label-add role=masl manager-node
```

- **查看标签**

```
docker node inspect manager-node
```

![img](https://img2018.cnblogs.com/i-beta/1577453/202002/1577453-20200229121549971-1148361130.png)

- **删除标签**

```
docker node update --label-rm role manager-node
```



## Raft协议

双主双从:

raft协议: 保证大多数节点存活才可以用. 只要>1管理节点存活, 集群至少大于三台

所以保证集群可用,至少三台主节点,坏了一台才能正常工作.

> ## Manager节点
>
> Manager节点用来处理集群管理任务：
>
> - 维护集群状态
> - 调度service
> - 提供swarm模式[HTTP API](https://docs.docker.com/engine/api/) 端点
>
> Manager通过[Raft](https://raft.github.io/raft.pdf)的实现，来使Swarm的整体维持在一个一致的内部状态上，service则运行在这个一致的内部状态之上。为了测试的目的，我们可以使用单独manager节点。如果单独的manager节点出现问题，service依然会一直运行，但是这样就需要创建一个新的集群来恢复一致的内部状态维护。
>
> 利用Swarm模式的容错特性，推荐使Swarm的节点为奇数个，来实现高可用的要求。当采用多个manager节点时，我们就可以在不停止集群运行的情况下恢复停止运行的manager节点。
>
> - 3个manager节点最大容错允许1个manager不可用
>
> - 5个manager节点最大容错允许2个manager不可用。
>
> - ## Worker节点
>
>   Worker节点也是Docker Engine的实例，目的是用来运行Container。Worker节点不会像Manager节点那样提供集群的管理、任务调度和API。
>
>   可以创建仅有一个manager节点的Swarm，但是不能在没有manager节点的前提下，创建Worker节点。在默认情况下，manager节点同时也是一个worker节点。在一个manager节点的集群中，执行`docker service create`命令，调度器会将所有的task调度在本地Docker Engine上执行。



## Docker Service

```shell
docker run 容器启动 不具有扩缩容器
docker service 服务  具有扩缩容器, 滚动更新
```

查看服务

![image-20211208170931593](C:\Users\12589\Desktop\Better\学习笔记\运维\docker.assets\image-20211208170931593.png)

![image-20211208171146678](C:\Users\12589\Desktop\Better\学习笔记\运维\docker.assets\image-20211208171146678.png)



更新副本 (4个节点也能跑10个副本)

动态扩缩容, 实现高可用

```shell
docker service update --replicas 3 my-nginx
#一样 
docker service scale my-nginx=3
```

## 概念总结

**swarm**

集群的管理和编号。docker可以初始化一个swarm集群，其他节点可以加入。

**Node**

就是一个docker节点。多个节点就组成了一个网络集群。

**service**

任务，可以在管理节点或者工作节点来运行。

![image-20211208223426846](C:\Users\12589\Desktop\Better\学习笔记\运维\docker.assets\image-20211208223426846.png)

**Task**

容器内的命令， 细节任务！

![image-20211208223333701](C:\Users\12589\Desktop\Better\学习笔记\运维\docker.assets\image-20211208223333701.png)



# Docker Stack

docker-compose 单机部署项目

Docker Stack部署, 集群部署

```shell
# 单机
docker-compose up -d wordpress.yaml
# 集群

```



# Docker Secret

安全 配置密码 证书 



# Docker Config



# Docker安装其他

## nacos

```undefined
docker pull nacos/nacos-server
```

```undefined
docker run --name nacos-standalone -e MODE=standalone -p 8848:8848 -d nacos/nacos-server:latest
```



[初始化.sql](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Falibaba%2Fnacos%2Fblob%2Fmaster%2Fdistribution%2Fconf%2Fnacos-mysql.sql)

```undefined
docker run --name nacos-standalone-mysql -e MODE=standalone \
--link mysql57:db \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e  MYSQL_SERVICE_HOST=db \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_DB_NAME=nacosConf \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=123 \
-p 8848:8848 -d nacos/nacos-server:latest 
```

iids中nacos

```shell  
docker run --name nacos -e MODE=standalone \
--link mysql \
-e SPRING_DATASOURCE_PLATFORM=mysql \
-e  MYSQL_SERVICE_HOST=mysql \
-e MYSQL_SERVICE_PORT=3306 \
-e MYSQL_SERVICE_DB_NAME=nacos \
-e MYSQL_SERVICE_USER=root \
-e MYSQL_SERVICE_PASSWORD=root \
-p 8848:8848 -d nacos/nacos-server:1.4.2 
```



## mysql

非挂载

```shell
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```



## Nginx

# IIDS-Nginx部署

> 在线版

### 1.在指定目录下创建目录并放入需要的文件

​	1.1	以本文为例，按下图所示创建/iids/nginx目录。 

/

├── iids

│	├── nginx
│	│   ├── conf.d   文件夹
│	│   ├── html
│	│   ├── logs
│	│   ├── nginx.conf    文件

​	1.2	 将配置文件按文件中结构放入上文创建的目录中，修改其中default.conf文件中http的ip,长连接ip目前需要web前端程序修改。[iids-nginx.zip](iids-nginx.zip) 

> 注意事项：如果是拿到新的配置文件（本地部署nginx版），则对配置文件进行修改，包括配置文件中有些路径修改成绝对路径，ip修改，配置文件拆分成两个（nginx.conf与default.conf）

### 2.在docker上部署nginx

```shell
docker run -p 8080:80 --name iids-nginx -v /iids/nginx/html:/usr/share/nginx/html  -v /iids/nginx/nginx.conf:/etc/nginx/nginx.conf -v /iids/nginx/conf.d:/etc/nginx/conf.d -v /iids/nginx/logs:/var/log/nginx -d nginx
```



## IIDS-Nginx部署

> 在线版

### 1.在指定目录下创建目录并放入需要的文件

	1.1	以本文为例，按下图所示创建/iids/nginx目录。 

/

├── iids

│	├── nginx

│	│   ├── conf.d

│	│   ├── html

│	│   ├── logs

│	│   ├── nginx.conf   

	1.2	 将配置文件按文件中结构放入上文创建的目录中，修改其中default.conf文件中http的ip,长连接ip目前需要web前端程序修改。[iids-nginx.zip](iids-nginx.zip) 

> 注意事项：如果是拿到新的配置文件（本地部署nginx版），则对配置文件进行修改，包括配置文件中有些路径修改成绝对路径，ip修改，配置文件拆分成两个（nginx.conf与default.conf）

### 2.在docker上部署nginx

```shell
docker run -p 8080:80 --name iids-nginx -v /iids/nginx/html:/usr/share/nginx/html  -v /iids/nginx/nginx.conf:/etc/nginx/nginx.conf -v /iids/nginx/conf.d:/etc/nginx/conf.d -v /iids/nginx/logs:/var/log/nginx -d nginx
```



# Docker Swarm安装其他

## 搭建Redis主从集群

环境

```shell
# 准备swarm环境
docker swarm init

# 如果没有redis镜像，拉取镜像
docker pull redis

# 在所有swarm节点上创建挂载目录以及配置文件
mkdir -p iids/redis/{bin,config,data,logs}

vi /iids/redis/config/master.conf
## 自定义配置

vi /iids/redis/config/slave.conf
## 自定义配置
slaveof master 6379

```

compose文件

```shell
# 创建docker-compose.yml
vi /iids/redis/redis-compose.yml
```

```shell
version: '3.7'
services:
  master:
    image: redis
    restart: always
    hostname: redis-master
    container_name: redis-master
    command: [ "redis-server", "/usr/local/etc/redis/redis.conf"]
    volumes:
      - /iids/redis/data:/data			
      - /iids/redis/config/master.conf:/usr/local/etc/redis/redis.conf
      - /iids/redis/logs:/var/log/redis
    ports:
    - target: 6379
      published: 6379
      protocol: tcp
      mode: host
  slave:
    image: redis
    restart: always
    hostname: redis-slave
    container_name: redis-slave
    command: ["redis-server", "/usr/local/etc/redis/redis.conf"]
    ports:
    - target: 6379
      published: 6379
      protocol: tcp
      mode: host
    volumes:
      - /iids/redis/data:/data	
      - /iids/redis/config/slave.conf:/usr/local/etc/redis/redis.conf
      - /iids/redis/logs:/var/log/redis

```

启动

```shell
docker stack deploy -c redis-compose.yml redis
```



## 搭建Redis哨兵集群

