---
title: Docker安装相关软件
Author: tzcqupt
categories: [Docker]
tags: [Docker]
comments: true
toc: true
date: 2022-02-13 00:00:00
---

# Docker相关

## 安装Docker

### Windows 安装Docker

#### Windows家庭版开启Hyper-V(可选)

~~~bash
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
~~~

#### 官网下载安装包进行安装

~~~
https://desktop.docker.com/win/stable/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&amp;utm_medium=webreferral&amp;utm_campaign=dd-smartbutton&amp;utm_location=module
~~~

#### Win10 修改docker镜像存储位置

1. 查看dockek是否安装成功

   ~~~
   docker Info
   ~~~

2. 确保docker使用的Linux Containers(使用的WSL)或者查看上述命令的回显是否如下

   ~~~
   Docker Root Dir: /var/lib/docker
   ~~~

3.  关闭`Docker Desktop`,使用下面命令查看是否停止了

   ~~~bash
   C:\Users\tzcqupt> wsl --list -v
     NAME                   STATE           VERSION
     docker-desktop         Stopped         2
     docker-desktop-data    Stopped         2
   ~~~

   >默认情况下，Docker Desktop for Window会创建如下两个发行版（distro) 
   >docker-desktop (`C:\Users\tzcqupt\AppData\Local\Docker\wsl\distro\ext4.vhdx`)
   >docker-desktop-data (`C:\Users\tzcqupt\AppData\Local\Docker\wsl\data\ext4.vhdx`)

4. 对`docker-desktop`和`docker-desktop-data`分别执行导出,取消注册,导入命令

   ~~~bash
    #导出
    wsl --export docker-desktop "D:\\soft\\develop\\docker\\images\\docker-desktop.tar"
    wsl --export docker-desktop-data "D:\\soft\\develop\\docker\\images\\docker-desktop-data.tar"
    
    #取消注册
    wsl --unregister docker-desktop
    wsl --unregister docker-desktop-data
    
    #导入
    wsl --import docker-desktop-data "D:\\soft\\develop\\docker\\images\\wsl\\data" "D:\\soft\\develop\\docker\\images\\docker-desktop-data.tar" --version 2
   
   wsl --import docker-desktop "D:\\soft\\develop\\docker\\images\\wsl\\distro" "D:\\soft\\develop\\docker\\images\\docker-desktop.tar" --version 2
   ~~~

   

### 安装Docker方式(新)

在 CentOS 7安装docker要求系统为64位、系统内核版本为 3.10 以上

`uname -r`查看内核版本

~~~bash
# 查看是否已安装docker列表
yum list installed | grep docker
# 安装Docker
yum -y install docker
# 启动Docker
systemctl start docker
# 加入开机启动
systemctl enable docker
# 查看docker服务状态
systemctl status docker
~~~



### 安装并运行Docker(旧)

1. 更新yum包

   ~~~
   sudo yum update
   ~~~

2. 安装必要的系统工具

   ~~~
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   ~~~

3. 添加阿里云镜像

   ~~~
   sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   ~~~

4. 更新yum缓存

   ~~~
   sudo yum makecache fast
   ~~~

5. 安装Docker-ce

   ~~~
   sudo yum -y install docker-ce
   ~~~

6. 查看版本是否安装成功

   ~~~
   docker -v
   ~~~

7. 启动docker后台服务

   - 启动:```sudo systemctl start docker```

   - 停止:```sudo systemctl stop docker```

   - 重启:```sudo systemctl restart docker```
   - 查看状态:```systemctl status docker```
   - 开机启动docker:```systemctl enable docker```

8. 测试运行hello-world

   ~~~
   docker run hello-world
   ~~~

### 配置镜像加速

新增/修改 ```/etc/docker/daemon.json```,加入

~~~json
{
"registry-mirrors": ["https://0xp8b743.mirror.aliyuncs.com"]
}
~~~

### 卸载Docker

1. 查询docker安装过的包

   ~~~
   yum list installed | grep docker
   ~~~

2. 删除安装包

   ~~~
   yum remove docker-ce.x86_64 docker-ce-cli.x86_64 -y
   ~~~

3. 删除镜像/容器

## 容器相关

如何使进入容器的用户角色为root

 ~~~
docker exec -u 0 -it 容器名称/id bash
 ~~~

### 查看容器

查看正在运行中的容器:```docker ps```

查看所有容器：```docker ps -a```

查看最后一次运行的容器： ```docker ps -l```

查看停止的容器： ```docker ps -f status=exited```

### 交互式方式创建容器

~~~
docker run -i -t --name=容器名称 镜像名称:标签 /bin/bash
~~~

`-i` 表示运行容器

`-t` 表示容器启动后会进入其命令行,可以直接 -it ==>容器创建就能登录进去,分配一个伪终端 

exit退出后,容器后就停止了

### 守护式方式创建容器

~~~
docker run -di --name=容器名称 镜像名称:标签
~~~

`-d` 表示会创建守护式容器在后台,创建后不会自动登录

> 容器名称要唯一

### 登录守护式容器

~~~
docker exec -it 容器名称(或者容器id) /bin/bash
~~~

> 执行exit退出后，容器仍在运行

### 停止容器方式

#### 停止容器

~~~
docker stop 容器名称(或者容器id)
~~~

#### 停用全部运行中的容器

~~~
docker stop $(docker ps -q)
~~~

#### 删除全部容器

~~~
docker rm $(docker ps -aq)
~~~

#### 停用并删除所有容器

~~~
docker stop $(docker ps -q) & docker rm $(docker ps -aq)
~~~

### 启动容器

~~~
docker start 容器名称(或者容器id)
~~~

### 文件拷贝

~~~
docker cp 需要拷贝的文件或目录 容器名称:容器目录
docker cp 容器名称:容器目录 需要拷贝的文件或目录
~~~

### 目录挂载

~~~
docker run -di -v 宿主机目录:容器目录 --name=容器名称 镜像名称:标签
~~~

> 相当于创建了这两个目录的映射,任何一方所做的修改都会同步给另外一方

### 查看容器ip地址

~~~
docker inspect 容器名称(容器id)
~~~

如只查看centos容器的ip地址

~~~
docker inspect --format='{{.NetworkSettings.IPAddress}}' centos
~~~

## 镜像相关命令

1. 查看镜像
   
    ~~~
    docker images
    ~~~

2. 搜索镜像
   
    ~~~
    docker search xxx
    ~~~

3. 拉取镜像

   ~~~
   docker pull 镜像名称
   ~~~

4. 删除镜像

   ~~~
   docker rmi 镜像id
   ~~~

5. 删除所有镜像

    ~~~
    docker rmi 'docker images -q'
    ~~~


## Docker里面安装软件

### MySQL 

1. 下载容器

   ~~~
   docker pull centos/mysql-57-centos7
   ~~~

2. 创建容器

   ~~~
   docker run -di --name=mysql --restart=always -p 33306:3306 -e MYSQL_ROOT_PASSWORD=root123 centos/mysql-57-centos7
   ~~~

   > `-p` 表示端口映射
   >
   > `--restart=always` , 当 Docker 重启时,容器自动启动。
   >
   > `-e MYSQL_ROOT_PASSWORD`表示MySQL的root用户的密码

3. 远程登录 `192.168.0.108:33306 root/root123`

### Nginx 

1. 下载容器

   ~~~
   docker pull nginx
   ~~~

2. 创建容器

   ~~~
   docker run -di --name=nginx --restart=always -p 80:80 nginx
   ~~~

3. 访问 [http://192.168.0.108](http://192.168.0.108)

4. 进入容器中查看nginx的静态页面存放目录

   ~~~
   docker exec -it mynginx /bin/bash
   ~~~

   执行```cat /etc/nginx/nginx.conf ```

   得到存放的目录为 ```/usr/share/nginx/html```

5. 将静态页面拷贝到html目录下

   ~~~
   docker cp html mynginx:/usr/share/nginx
   ~~~

### Redis

1. 下载容器

   ~~~
   docker pull redis
   ~~~

2. 创建容器

   ~~~
   docker run -di --name=redis --restart=always -p 6379:6379 redis
   ~~~


### GitLab

[参考博客](https://www.cnblogs.com/reasonzzy/p/11145026.html)

1. 下载镜像

   ~~~shell
   docker pull gitlab/gitlab-ce
   ~~~

2. 提前创建好gitlab的数据和配置等文件夹

3. 运行gitlab镜像

   ~~~shell
   docker run \
   --detach \
   --publish 2222:22 \
   --publish 8081:80 \
   --publish 8443:443 \
   --hostname 192.168.0.108 \
   -v /usr/local/soft/gitlab/config:/etc/gitlab \
   -v /usr/local/soft/gitlab/logs:/var/log/gitlab \
   -v /usr/local/soft/gitlab/data:/var/opt/gitla \
   -v /etc/localtime:/etc/localtime:ro \
   --name gitlab \
   --restart always \
   --privileged=true gitlab/gitlab-ce:latest
   ~~~

4. 修改配置文件

   ~~~shell
   vi /usr/local/soft/gitlab/config/gitlab.rb
   #将external_url 改为自己的ip地址
   external_url 'http://192.168.0.108'
   vi /usr/local/soft/gitlab/data/gitlab-rails/etc/gitlab.yml
   #搜索关键字 Web server settings,修改host和port
   ~~~

5. 删除容器,重新启动

   ~~~
   docker stop gitlab
   docker rm gitlab
   # 重启docker
   systemctl restart docker
   # 重新创建容器,执行[3]
   ~~~

6. 容器启动后,通过`docker inspect 容器id` 查看容器分配的ip地址

   ~~~
   "Networks": {
                   "bridge": {
                       "IPAMConfig": null,
                       "Links": null,
                       "Aliases": null,
                       "NetworkID": "db7db0f4b7b71c07540977e987b3a9b62f97328d22c493eb06db85d274ea82bb",
                       "EndpointID": "ebbb6ab495a561fb10e4edd610e593b6977014e16a21dc7aa9448dff0acfa037",
                       "Gateway": "172.17.0.1",
                       "IPAddress": "172.17.0.5",
                       "IPPrefixLen": 16,
                       "IPv6Gateway": "",
                       "GlobalIPv6Address": "",
                       "GlobalIPv6PrefixLen": 0,
                       "MacAddress": "02:42:ac:11:00:05"
                   }
               }
   # 执行命令查看是否连接成功
   curl 172.17.0.5:80
   ~~~

7. [登录gitlab](http://192.168.0.108:8081/)

   > 设置超级管理员的密码,账号为root 密码为tzcqupt@gitlab
   >
   > 配置ssh  (id_rsa.pub文件内容放在gitlab服务器上)

8. **下载在gitlab上创建的项目**

   ~~~shell
   # 修改配置文件,重启服务
   [root@tzcqupt config]# cat gitlab.rb | grep ssh_port
    gitlab_rails['gitlab_shell_ssh_port'] = 2222
   ~~~

### Jenkins

[官方文档](https://www.jenkins.io/zh/doc/book/installing/)

~~~
docker run \
  -d \
  -u root \
  --restart always \
  --privileged=true \
  --name jenkins \
  -p 8081:8080 \
  -p 50000:50000 \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /usr/local/soft/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/local/soft/maven/apache-maven-3.5.4:/usr/local/maven \
  jenkinsci/blueocean
~~~

后续配置参考 **安装Jenkins章节**

Git/Java/Maven 

docker安装的jenkins容器已经包含了Git和Java,Maven通过-v 挂载的方式

> yum方式安装Git到指定目录
>
> ~~~shell
> # --installroot 指定目录
> # --nogpgcheck 不进行公钥检查
> yum -c /etc/yum.conf --installroot=/usr/local/soft/jenkins/share/soft/git --releasever=/ --nogpgcheck  install git
> ~~~
>

### PostgreSQL

1. 安装PostgreSQL

   ~~~
   docker run --name postgres -d -p 5432:5432 --restart=always -e POSTGRES_PASSWORD=postgres postgres:13.5
   ~~~

2. 进入容器

   ~~~
   docker exec -it postgres psql -U postgres -d postgres
   ~~~

3. 查看数据库所有表

   ~~~
   select * from pg_tables;
   ~~~

### NIFI

1. 安装nifi

   ~~~
   docker run -d --restart=always -p 8080:8080 -e NIFI_WEB_HTTP_PORT='8080' --name nifi apache/nifi:1.14.0
   ~~~

   > 默认为https,需要这样修改使用http

2. 访问nifi

   ~~~
   https://localhost:8083/nifi
   ~~~

   

# Docker环境搭建终极篇

## 前提准备

1. Linux下安装Docker
2. Docker安装centos7
3. 建立各自的目录和Dockerfile,在对应的目录下执行docker的构建命令`docker build`
4. 将需要的`tar.gz`文件放在对应的目录下,避免网络问题下载失败导致构建失败.

## `Dockerfile`

### `centos7`

~~~dockerfile
FROM centos:7
MAINTAINER tzcqupt
RUN yum -y update
RUN yum install -y passwd openssh-server openssh-clients initscripts net-tools
RUN yum install -y python2-setuptools
RUN ssh-keygen
RUN easy_install supervisor
RUN echo 'root:root123' | chpasswd
RUN /usr/sbin/sshd-keygen

EXPOSE 22
CMD /usr/sbin/sshd -D

~~~

`docker build -t base/centos7-ssh . `

### `centos7-jdk8u265`

~~~dockerfile
#
# JDK-8U265
#
FROM base/centos7-ssh
MAINTAINER tzcqupt
RUN yum clean all
RUN yum -y update
# Install libs
RUN yum install deltarpm rpm make wget tar unzip \
         gcc gcc-c++ -y

RUN mkdir -p /home/work/apps/
RUN mkdir -p /usr/local/soft/java/
WORKDIR /home/work/apps/
#RUN wget https://mirror.tuna.tsinghua.edu.cn/AdoptOpenJDK/8/jdk/x64/linux/OpenJDK8U-jdk_x64_linux_hotspot_8u265b01.tar.gz
ADD OpenJDK8U-jdk_x64_linux_hotspot_8u265b01.tar.gz /usr/local/soft/java/
WORKDIR /usr/local/soft/java/
# RUN tar -zxvf OpenJDK8U-jdk_x64_linux_hotspot_8u265b01.tar.gz
# RUN rm OpenJDK8U-jdk_x64_linux_hotspot_8u265b01.tar.gz
# RUN mv -f jdk8u265-b01/ /usr/local/soft/java/jdk8u265-b01
ENV JAVA_HOME=/usr/local/soft/java/jdk8u265-b01
ENV PATH=$JAVA_HOME/bin:$PATH
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

# open ports
EXPOSE 22
~~~

`docker build -t base/centos7-jdk8u265 .  `

### `centos7-tomcat7`

~~~dockerfile
#
# Tomcat7
#

FROM base/centos7-jdk8u265
MAINTAINER tzcqupt

RUN yum clean all
RUN yum -y update
# Install libs
WORKDIR /home/work/apps/
#RUN wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-7/v7.0.85/bin/apache-tomcat-7.0.85.tar.gz
ADD apache-tomcat-7.0.85.tar.gz /usr/local/soft/
WORKDIR /usr/local/soft/
#RUN tar -zxvf apache-tomcat-7.0.85.tar.gz
#RUN rm apache-tomcat-7.0.85.tar.gz

EXPOSE 8080
COPY supervisord.conf /etc/supervisord.conf
CMD ["/usr/bin/supervisord"]
~~~

`docker build -t base/centos7-tomcat7 . `

### `centos7-jenkins`

~~~dockerfile
# Jenkins 2.235.5
FROM base/centos7-tomcat7
MAINTAINER tzcqupt
RUN yum install -y git
RUN yum install -y dejavu-sans-fonts
RUN yum install -y fontconfig
WORKDIR /usr/local/soft/apache-tomcat-7.0.85/webapps/ROOT/
RUN rm -fr ./*
RUN wget http://mirrors.tuna.tsinghua.edu.cn/jenkins/war-stable/2.235.5/jenkins.war
RUN unzip jenkins.war
RUN rm -fr docs
RUN rm -fr examples
RUN rm -fr manager
RUN rm -fr host-manager
EXPOSE 8079
ENTRYPOINT /usr/local/soft/apache-tomcat-7.0.85/bin/startup.sh && tail -F /usr/local/soft/apache-tomcat-7.0.85/logs/catalina.out
~~~

`docker build -t centos7-jenkins . `

~~~shell
docker run -di --name=jenkins --restart=always -p 8079:8080 -p 50000:50000 -v $PWD/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock centos7-jenkins
~~~

## Docker可视化容器

Windows 安装了Docker,自带Docker UI,不需要安装.

### 构建Shipyard容器 

~~~
docker pull shipyard/shipyard 
docker pull swarm 
docker pull shipyard/docker-proxy 
docker pull alpine 
docker pull microbox/etcd 
docker pull rethinkdb
~~~

 ~~~
docker run -ti -d --restart=always --name shipyard-rethinkdb rethinkdb
 ~~~

~~~
docker run -ti -d -p 4001:4001 -p 7001:7001 --restart=always --name shipyard-discovery  microbox/etcd:latest -name discovery 
~~~

~~~
docker run -ti -d -p 2375:2375 --hostname=$HOSTNAME --restart=always --name shipyard-proxy -v /var/run/docker.sock:/var/run/docker.sock -e PORT=2375 shipyard/docker-proxy:latest
~~~

~~~
docker run -ti -d --restart=always --name shipyard-swarm-manager swarm:latest manage --host tcp://0.0.0.0:3375 etcd://121.37.172.25:4001
~~~

~~~
docker run -ti -d --restart=always --name shipyard-swarm-agent swarm:latest join --addr 121.37.172.25:2375 etcd://121.37.172.25:4001 
~~~

~~~
docker run -ti -d --restart=always --name shipyard-controller --link shipyard-rethinkdb:rethinkdb --link shipyard-swarm-manager:swarm -p 8081:8080 shipyard/shipyard:latest server -d tcp://swarm:3375 
~~~

用户名/密码 admin/shipyard

## 手动安装

### 创建自己的网络(可选)

~~~shell
docker network create --subnet=192.168.0.0/24 basenet
~~~

### 安装Jenkins

~~~shell
docker run -u root --rm -d -p 8079:8080 -p 50000:50000 -v $PWD/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean
~~~

> `$PWD`代表当前目录

