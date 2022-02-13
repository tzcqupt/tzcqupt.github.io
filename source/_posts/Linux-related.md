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

### Ubuntu 安装

[安装Jenkins](https://www.jenkins.io/zh/doc/book/installing/#debianubuntu)

~~~
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
~~~

### 常见问题

#### 安装成功但无法访问jenkins

1. 禁用防火墙即可
   1. 关闭防火墙

   `systemctl stop firewalld.service`

   2. 禁止开机启动

   `systemctl disable firewalld.service`

2. 确保安全组已打开相应端口

#### 插件下载慢的解决办法

##### 插件下载管理

进入jenkins的插件下载管理

`http://ip:端口/jenkins/pluginManager/advanced`

修改 **Update Site**中的URL的值为https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

##### 修改default.json

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

#### Jenkins任务下载git仓库失败

Jenkins源码管理git报错：Host key verification failed

1. jenkins用户创建ssh-key和gitee通信

   ~~~
   # 切换为jenkins用户
   sudo su -s /bin/bash jenkins
   # 执行git命令
   git ls-remote -h git@ip:xxx.git HEAD
   # 下载失败则生成ssh-key  在gitee上配置
   ssh-keygen -t rsa -C "tzcqupt@jenkins.com"
   # 在终端提示中输入yes
   ~~~

2. 切换jenkins运行角色为root

   ~~~
   vim /etc/default/jenkins
   #修改jenkins以root用户运行
   JENKINS_USER=root
   JENKINS_GROUP=root
   #重启jenkins
   ~~~

   

#### 配置git时,凭证添加后选取不了

去Jenkins的全局凭证管理里面添加

#### Jenkins 执行任务时,没有权限

1. 修改jenkins以root用户运行

   ~~~
   vim /etc/default/jenkins
   #修改jenkins以root用户运行
   JENKINS_USER=root
   JENKINS_GROUP=root
   #重启jenkins
   ~~~

2. 给相关的目录(配置的maven 仓库路径)权限

   ~~~
   cd /usr/share
   #把该目录和子级目录的权限赋值给jenkins用户组
   chown -R jenkins:jenkins maven-repo
   systemctl restart jenkins.service
   ~~~

#### Jenkins 执行任务时maven下载jar包失败

通过`apt install maven`安装maven 3.6.0下载jar包失败,删除该版本,降低为3.5.4

> 需要关联软连接 /usr/bin/mvn
>
> ```ln -s /usr/local/soft/maven/apache-maven-3.5.4/bin/mvn /usr/bin/mvn```
>
> ```ll /usr/bin/mvn```

#### Jenkins离线安装插件

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

部署Java项目并运行

~~~shell
BUILD_ID=DONTKILLME
# 获取正在运行的程序pid
pid=$(ps -aux|grep $JOB_NAME | grep -v grep| gawk '{print $2}')
if [ ${#pid} != 0 ]
    then kill -9 $pid
fi
cd $WORKSPACE
mvn clean package
nohup java -jar $WORKSPACE/target/$JOB_NAME.jar -Xmx128m -Xms256m -Xss4m &
pid=$(ps -aux|grep $JOB_NAME | grep -v grep| gawk '{print $2}')
# 获取正在运行的程序的pid并判断其字符串长度，0为不存在(即构建失败)
if [ ${#pid} == 0 ]
    then
     echo "*****  BUILD FAILED  ******"
     exit 1
     else
     echo "*****  BUILD SUCCESS  *****"
fi

~~~



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

## 安装Yapi

[官网内网部署](https://yapi.baidu.com/doc/devops/index.ht)

### 软件安装

#### Ubuntu 安装`nodejs` `npm` `git` `mongodb`

~~~
#安装git
sudo apt install git
#安装mongodb(v3.6.3)
sudo apt install mongodb
sudo systemctl enable mongodb
#安装nodejs
sudo apt install nodejs
#安装npm
sudo apt install npm
#查看版本(node 版本最好在v12.x.x版本)
node -v
~~~

#### 修改安装的`nodejs`版本

~~~
#安装n工具管理node版本
npm install n -g
#安装指定版本
n 12.16.3
#卸载指定版本
n rm 0.9.4 v0.10.0
#查看已有的node版本并切换
n
~~~

#### 安装`yapi-cli`

node 版本安装`v12.22.1`版本即可,不然会有问题

```bash
npm install -g yapi-cli --registry https://registry.npm.taobao.org
yapi server
```

#### 使用`pm2`管理node服务

~~~
#安装pm2
sudo npm install -g pm2
# 后台启动
sudo pm2 start /usr/local/soft/yapi/my-yapi/vendors/server/app.js
# 重命名任务
sudo pm2 start /usr/local/soft/yapi/my-yapi/vendors/server/app.js --name yapi
# 删除任务(进程)
#sudo pm2 delete [进程名称]
sudo pm2 delete app
#让系统开机启动pm2管理的任务
sudo pm2 startip
#可选-保存修改
sudo pm2 save
~~~



### `nginx` 配置

~~~
在location /添加
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
~~~

## 安装Nginx

~~~
sudo apt install nginx
~~~

### **nginx ssl/二级域名配置**

1. dns域名解析配置

2. [申请免费ssl](https://letsencrypt.osfipin.com/)

   > 验证时,若遇到403,直接修改nginx.conf配置,第一行的`user www-data`改为`user root`

~~~
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {


	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;


	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	gzip on;

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;

        upstream yapi {
        server 127.0.0.1:3000;
	       } 

        server{
		listen 80;
		server_name tzcqupt.top;
		return 301 https://$host$request_uri;
          location / {
           	root html;
		index index.html;
		}	
	}

 	server{
                listen 443;
                ssl on;
                server_name tzcqupt.top;
                ssl_certificate /usr/local/soft/ssl/tzcqupt-top/certificate.crt;
                ssl_certificate_key /usr/local/soft/ssl/tzcqupt-top/private.key;
                ssl_session_timeout 5m;
                ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
                ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
                ssl_prefer_server_ciphers on;


        location / {
                 root html;
		 index index.html;
       		 }

	}


	server{
		listen 80;
		server_name yapi.tzcqupt.top;
		return 301 https://$host$request_uri;
	}
	server{
		listen 443;
        	ssl on;
		server_name yapi.tzcqupt.top;
		ssl_certificate /usr/local/soft/ssl/x-tzcqupt-top/certificate.crt;
		ssl_certificate_key /usr/local/soft/ssl/x-tzcqupt-top/private.key;
		ssl_session_timeout 5m;
		ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
		ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
		ssl_prefer_server_ciphers on;
	
	
	location / {
     		 proxy_pass  http://yapi;
     		 proxy_set_header Host $host;
    		 proxy_set_header  X-Real-IP        $remote_addr;
   		 proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
     		 proxy_set_header X-NginX-Proxy true;
     		 proxy_set_header Connection "upgrade";
     		 proxy_set_header Upgrade $http_upgrade;
     		 proxy_http_version 1.1;
	}
     }
}


~~~



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

## 脚本相关

### 脚本定时将github代码同步到gitee

1. 前提:gitee和github都配置了linux的ssh的密钥

2. `git clone git@gitee.com:tzcqupt/tzcqupt.git .`

   > 后面的.表示就下载在当前目录,不新建目录

3. ` git remote add github git@github.com:tzcqupt/tzcqupt.github.io.git`

4.  对origin重命名 `git remote rename origin gitee`

5.  新建同步脚本 `_sync_from_github_to_gitee.sh`

   ~~~
   #!/bin/bash
   
   cd /usr/share/nginx/html
   
   git pull github master
   
   git push gitee master
   ~~~

   > 1. 首次执行`git push`,会提示设置 push.default,执行`git config --global push.default simple`进行设置即可
   > 2. 在windows上新建的文件拷贝到linux上,需要执行 `dos2unix _sync_from_github_to_gitee.sh `,否则linux执行该命令系统会报错

6. 给文件加执行权限`chmod +x _sync_from_github_to_gitee.sh`

7. 再增加定时任务

   ```bash
   crontab -e
   1 1 */2 * * /usr/local/data/git/_sync_from_github_to_gitee.sh
   ```

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

# Nginx 代理相关

~~~bash
 #https 配置
 server {
        listen 443 ssl;  # 1.1版本后这样写
        server_name www.tzcqupt.top; #填写绑定证书的域名
        ssl_certificate /etc/nginx/1_www.tzcqupt.top_bundle.crt;  # 指定证书的位置，绝对路径
        ssl_certificate_key /etc/nginx/2_www.tzcqupt.top.key;  # 绝对路径，同上
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
        ssl_prefer_server_ciphers on;
        location / {
            root   html; #站点目录，绝对路径
            index  index.html index.htm;
        }
         location /blog/ {
           proxy_pass https://127.0.0.1/;
        }
        location /redRain/ {
        #代理到docker 里面的nginx
        proxy_pass http://127.0.0.1:801/;
        }

~~~

# WSL2 相关

Windows中安装Linux子系统Ubuntu

~~~
#列举相关的wsl服务,查看状态
wsl -l --verbose
~~~

## 将安装的`Ubuntu`迁移到非系统盘

1. 下载 [LxRunOffline]( https://github.com/DDoSolitary/LxRunOffline/releases ),并解压

2. 在解压后的目录打开PowerShell, 使用`LxRunOffline.exe list`命令查看你可以使用子系统名称 

3.  使用 `lxrunoffline move` 进行迁移 ， -n 指定你要迁移的系统名 ，-d 指定你新系统的迁移路径 

   ```bash
   .\LxRunOffline.exe move -n Ubuntu-20.04 -d D:\soft\develop\Ubuntu20
   ```

   >若出现Couldn't set the case sensitive attribute of the directory ....
   >
   >Reason: Indicates that the directory trying to be deleted is not empty.
   >
   >参考官方 [issue](https://github.com/DDoSolitary/LxRunOffline/issues/150)
   >
   >下载https://ddosolitary-builds.sourceforge.io/LxRunOffline/LxRunOffline-v3.5.0-11-gfdab71a-msvc.zip重新解压执行
   >
   >

4.  使用`LxRunOffline.exe get-dir` 查询系统目录，可见已经更改成功

   ~~~
   .\LxRunOffline.exe get-dir -n Ubuntu-20.04
   ~~~

## Ubuntu 配置

### Xshell相关工具连接登录

1. 安装ssh服务

   `sudo apt-get install openssh-server`

    `sudo apt-get install ssh `

2. 开启ssh服务` service ssh start `

3. 修改配置,允许远程连接

   ~~~
   sudo vim /etc/ssh/sshd_config
   添加配置:PasswordAuthentication yes
   ~~~

4. 重启ssh服务 `service ssh restart`

### 间接固定WSL2 的Ubuntu的ip地址

> WSL2 启动后,每次会为Ubuntu分配新的ip地址

1. 在子系统中新建文件`/etc/init.wsl`

   ~~~bash
   #ssh
   /etc/init.d/ssh start
   #network static ip
   ip addr add 192.168.50.28/24 broadcast 192.168.50.255 dev eth0 label eth0:1
   ~~~

2. 在Windows上新建bat文件,并发送快捷方式到桌面上,设置快捷方式的运行权限为管理员权限

   ~~~
   wsl -d Ubuntu-20.04 -u root /etc/init.wsl
   netsh interface ip add address "vEthernet (WSL)" 192.168.50.93 255.255.255.0
   ~~~

   >`Ubuntu-20.04` 为在Windows上使用`wsl -l`命令列举的Ubuntu名称

3. 测试,重启电脑,或者使用`wsl -t Ubuntu-20.04 `关闭Ubuntu后,再点快捷方式,使用Xshell相关工具连接.