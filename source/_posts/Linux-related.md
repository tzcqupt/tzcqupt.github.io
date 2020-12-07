---
title: Linux常用命令总结
Author: tzcqupt
categories: [Linux]
tags: [Linux,腾讯云]
comments: true
toc: true
date: 2020-06-02 00:00:00
---

# 腾讯云相关

## 让客户端和腾讯云一直保持连接

1. 修改`sshd_config`配置文件

   ~~~bash
   [root@VM_0_17_centos ~]# vim /etc/ssh/sshd_config
   # 找到并修改
   #服务端每隔多少秒向客户端发送一个心跳数据
   ClientAliveInterval 30
   #客户端多少次没有响应，服务器自动断开连接
   ClientAliveCountMax 86400
   
   ~~~

2. 重启`sshd`服务

   ~~~bash
   [root@VM_0_17_centos ~]# service sshd restart
   ~~~

## Xshell 登录腾讯云缓慢

~~~bash
rm -rf /var/log/btmp
touch /var/log/btmp
~~~

# 软件安装相关

## 软件下载相关

下载github的指定版本,去gitee下载,然后使用maven编译.如下载nacos

~~~
mvn -Prelease-nacos -DskipTests clean install -U
~~~

## 安装Jenkins

> 确保安装了Java

### yum安装

#### 添加yum源

~~~
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo 
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
~~~

#### 执行安装

~~~
yum install jenkins
~~~

输入y进行安装即可

### 安装rpm包

1. [下载`.rpm`文件](https://mirror.tuna.tsinghua.edu.cn/jenkins/redhat-stable/)

2. 安装`.rpm`文件

   ~~~
   rpm -ivh jenkins-2.235.5-1.1.noarch.rpm
   ~~~

3. 查看安装位置 `rpm -qc jenkins`

   ~~~bash
   [root@localhost soft]# rpm -qc jenkins
   /etc/init.d/jenkins
   /etc/logrotate.d/jenkins
   /etc/sysconfig/jenkins
   ~~~

4. 端口修改`vi /etc/sysconfig/jenkins`

   ~~~
   JENKINS_PORT="8081"
   ~~~

5. 启动`service jenkins start`

   > 报错需要修改文件`/etc/init.d/jenkins`
   >
   > 将`/usr/bin/java`改为自己java的地址
   >
   > 自己Java地址的查看方式为`which java`

### WAR包安装

1. [下载WAR包](https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/2.251/jenkins.war)

2. 将WAR包放在tomcat的webapps下面
3. 启动tomcat
4. tomcat启动后,会生成一个.jenkins文件夹在root目录下`/root/.jenkins/`
5. 浏览器上输入IP地址(http://ip:端口/jenkins)即可访问(http://www.tzcqupt.top/jenkins/)

6. 复制生成的密码`/root/.jenkins/initialAdminPassword`

7. 进入页面后,安装建议的插件

8. 如果插件安装失败,通过https://mirrors.tuna.tsinghua.edu.cn/jenkins下载插件.

   插件管理中的高级进行上传.

### 安装成功但无法访问jenkins

禁用防火墙即可

1. 关闭防火墙

   `systemctl stop firewalld.service`

2. 禁止开机启动

   `systemctl disable firewalld.service`

### 插件下载慢的解决办法

#### 插件下载管理

进入jenkins的插件下载管理

`http://ip:端口/jenkins/pluginManager/advanced`

修改 **Update Site**中的URL的值为https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

#### 修改default.json

进入`/root/.jenkins/updates`中编辑default.json文件

替换所有插件的下载url:

~~~
1. default.json里面是updates.jenkins-ci.org
:1,$s/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g
2. default.json面是updates.jenkins.io
:1,$s/https:\/\/updates.jenkins.io\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g

~~~

替换连接测试url

~~~
:1,$s/http:\/\/www.google.com/https:\/\/www.baidu.com/g
~~~

保存并退出

重启jenkins `http://ip:端口/jenkins/restart`

### Jenkins离线安装插件

Jenkins报413错误

修改nginx的配置文件

1. 查找nginx的配置文件位置

   `find / -name nginx.conf`

2. 修改nginx的配置文件

   添加`client_max_body_size 10M;`

   ~~~
   server {
       listen       80;
       server_name  www.abc.com;
    
       add_header Access-Control-Allow-Origin *;
       add_header Access-Control-Allow-Credentials true;
       add_header Access-Control-Allow-Headers *;
       add_header Access-Control-Allow-Methods GET,POST,OPTIONS;
        
       client_max_body_size 10M;
    
       location / {
          ....
       }
   }
   ~~~

3. 重启nginx

   `nginx -s reload`

### 配置gitee

[gitee插件文档](https://gitee.com/help/articles/4193)

> 使用离线安装的方式,前往[清华大学开源软件](https://mirror.tuna.tsinghua.edu.cn/)下载插件

### 卸载Jenkins

~~~
service jenkins stop
yum clean all
yum -y remove jenkins
rm -rf /var/cache/jenkins
rm -rf /var/lib/jenkins/
rm -rf /etc/init.d/jenkins
rm -rf /etc/logrotate.d/jenkins
rm -rf /etc/sysconfig/jenkins
~~~

## 安装Maven

1. 下载软件包

2. 安装maven软件包

   ~~~
   tar -xf apache-maven-3.5.4-bin.tar.gz 
   mv apache-maven-3.5.4 /usr/local/maven
   # 与jenkins联合使用时，jenkins会到/usr/bin/下找mvn命令，如果没有回报错
   ln -s /usr/local/maven/bin/mvn  /usr/bin/mvn　　　　
   ll /usr/local/maven/
   ll /usr/bin/mvn
   ~~~

3. 环境变量修改 `/etc/profile/`

   ~~~
   export MAVEN_HOME=/usr/local/maven
   export PATH=$MAVEN_HOME/bin:$PATH
   #使环境变量生效
   source /etc/profile
   ~~~

4. 查看安装的mvn版本号

   ~~~
   which mvn
   mvn -version
   ~~~



## 其他问题

### Nginx日志查看

`/var/log/nginx`

# Linux常用相关操作

## 查看Linux服务其内存使用情况

1. `free` 命令

   > 1. 命令默认单位使kb 使用 `free -m`和`free -g`命令查看`MB`和`GB`,`free -h`自动选择合适的单位进行展示
   > 2. Mem: 表示物理内存统计
   > 3. Swap: 表示硬盘上交换分区的使用情况,shared表示共享内存,

2. top 命令

   查看系统的实时负载,包括进程,CPU负载,内存使用等

## 查看端口情况

~~~bash
netstat -ntlp
~~~

## Ubuntu 相关

### 安装Ubuntu

### 配置网关和镜像地址

~~~
subnet:192.168.66.0/24
Address:192.168.66.166
Gateway:192.168.66.2
Name Server:114.114.114.114
#镜像地址
http://mirrors.aliyun.com/ubuntu/
~~~

### Xshell 登录

确保能访问外网, 给root用户设置密码`sudo passwd`

安装`wget`:`apt-get install wget`

安装`SSH`:`apt-get install ssh`

开启远程访问SSH权限:

~~~bash
vim /etc/ssh/sshd_config
#将PermitRootLogin without-password修改为： 
PermitRootLogin yes 
#重启SSH
/etc/init.d/ssh restart
~~~

> 查看电脑的VMnet配置,设置为dhcp

# Docker相关

## 安装Docker

### Windows 安装Docker

Windows家庭版开启Hyper-V

~~~bash
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
~~~

官网下载安装包进行安装

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

修改 ```/etc/docker/daemon.json```,加入

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
  --detach \
  --privileged=true \
  --name jenkins \
  -p 8082:8080 \
  -p 50000:50000 \
  -v /usr/local/soft/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkinsci/blueocean
~~~

后续配置参考 **安装Jenkins章节**

Git/Java/Maven 安装在Docker里的Jenkins和主机交互的文件夹里面`/usr/local/soft/jenkins`

> yum方式安装Git到指定目录
>
> ~~~shell
> # --installroot 指定目录
> # --nogpgcheck 不进行公钥检查
> yum -c /etc/yum.conf --installroot=/usr/local/soft/jenkins/share/soft/git --releasever=/ --nogpgcheck  install git
> ~~~
>

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

# Nginx RMPT服务器

## 安装Nginx

### 安装所需的开发包

#### Ubuntu系统

~~~
sudo apt-get install gcc
sudo apt-get install libpcre3 libpcre3-dev
sudo apt-get install openssl libssl-dev
sudo apt-get install zlib1g.dev
sudo apt-get install zlib1g
sudo apt-get install unzip
~~~

#### CentOS系统

~~~
yum install gcc
yum install pcre pcre-devel pcre-static pcre-tools
yum install openssl openssl-static openssl-devel
yum install wget unzip
~~~



### 下载nginx和nginx-rtmp-module

~~~
wget http://nginx.org/download/nginx-1.14.2.tar.gz
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
tar -zxvf nginx-1.14.2.tar.gz
unzip master.zip
~~~

### 编译和安装nginx

~~~
cd nginx-1.14.2
./configure --with-http_ssl_module --with-http_secure_link_module --add-module=../nginx-rtmp-module-master
make
sudo make install
~~~

## 安装ffmpeg

官网安装 具体参照http://ffmpeg.org/download.html#build-linux 

~~~bash
#CentOS安装
yum localinstall --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm 
yum install ffmpeg
#Ubuntu安装
apt-get install ffmpeg
~~~

## 配置Nginx

修改`/usr/local/nginx/conf/nginx.conf`配置文件中`pid`的配置为`/var/run/nginx.pid`。

```
pid        /var/run/nginx.pid;
```

创建nginx日志目录

```
mkdir -p /var/log/nginx/
```

创建SystemD自启动文件`/lib/systemd/system/nginx.service`为如下:

```bash
[Unit]
Description=The nginx HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx t
ExecStart=/usr/local/nginx/sbin/nginx -c/usr/local/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=false

[Install]
WantedBy=multi-user.target
```

nginx启动命令

```bash
systemctl enable nginx.service  # 设置开机自启动
systemctl start nginx.service   # 设置启动nginx
systemctl stop nginx.service    # 停止nginx
```

## RTMP配置

~~~
rtmp_auto_push on;
rtmp {
    server {
        listen 1935;
        notify_method get;
        access_log  /var/log/nginx/access.log;
        chunk_size 4096;
        application live {
            # 开启直播模式
            live on;
            # 允许从任何源push流
            allow publish all;
            # 允许从任何地方来播放流
            allow play all;
            # 20秒内没有push，就断开链接。
            drop_idle_publisher 20s;
        }
    }
}
~~~

## Nginx 配置

### Nginx 配置校验

~~~
/usr/local/nginx/sbin/nginx -t
systemctl reload nginx
~~~

### 添加hls配置

1. 配置rtmp流产生hls文件

   ~~~bash
   mkdir -p /usr/local/nginx/hls/
   #创建存放hls文件的目录, 确保nobody用户可以读写该目录。
   chown nobody:root /usr/local/nginx/hls/
   ~~~

   

2. 配置nginx来访问hls文件

   ~~~nginx
   rtmp_auto_push on;
   rtmp {
       server {
           listen 1935;
           notify_method get;
           access_log  /var/log/nginx/access.log;
           chunk_size 4096;
           application live {
               # 开启直播模式
               live on;
               # 允许从任何源push流
               allow publish all;
               # 允许从任何地方来播放流
               allow play all;
               # 20秒内没有push，就断开链接。
               drop_idle_publisher 20s;
               # 开启HLS
               hls on; # Enable HTTP Live Streaming
               # Pointing this to an SSD is better as this involves lots of IO
               # 设置hls存放目录
               hls_path /usr/local/nginx/hls/;
           }
       }
   }
   ~~~

3. 设置nginx来访问hls文件

   ~~~nginx
   location /hls {
       types {
           application/vnd.apple.mpegurl m3u8;
           video/mp2t ts;
       }
       root /usr/local/nginx/;
       add_header Cache-Control no-cache; # Prevent caching of HLS fragments
       add_header Access-Control-Allow-Origin *; # Allow web player to access our playlist
   }
   ~~~

   

## ffmpeg相关操作

~~~bash
#推送
ffmpeg -re -i demo.mp4 -c copy -f flv 'rtmp://192.168.0.199/live/demo'
#播放
ffplay 'rtmp://192.168.0.199/live/xiaozhupeiqi'
~~~

