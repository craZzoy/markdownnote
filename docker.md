# 概念

docker是一个给开发者、系统管理员用容器构建、分享和运行程序的一个平台。这种用容器部署运用的技术叫做容器化。



虚拟机、LXC、docker



# 环境准备（win10）

## 安装vagrant+virtualbox

需要一个Centos环境，在其上面安装docker。为方便本地调试，采用以下方案

- vagrant(vagrant_2.2.6_x86_64)
  - 官网https://www.vagrantup.com/下载安装包，傻瓜式安装
  - 命令行执行vagrant命令，查看是否执行成功
- virtualbox(VirtualBox-6.0.12-133076-Win)
  - 官网下载https://www.virtualbox.org/，傻瓜式安装



## 安装centos7

1. 创建centos7目录，在此目录下启动cmd，运行vagrant init centos/7。此时会在当前目录下生成Vagrantfile文件，同时指定使用的镜像为centos/7

2. 通过virtualbox.box添加centos7镜像文件，并起名为centos/7

   ```properties
   D:\opensource\virtual-system\centos7>vagrant box add centos/7  D:\opensource\virtual-system\mirror-files\virtualbox.box
   D:\opensource\virtual-system\centos7>vagrant box list
   centos/7 (virtualbox, 0)
   ```

3. 启动虚拟机：进入centos7文件夹，在cmd命令行中输入vagrant up，通过virtual box可以观察到centos7创建成功

4. vagrant常用命令（在cenos7目录下执行，即对应添加的操作系统目录下）

   - vagrant halt：优雅关闭
   - vagrant up：正常启动
   - vagrant ssh：进入创建的centos
   - vagrant status：查看虚拟机状态
   - vagrant destroy：删除centos7
   - vagrant reload：重启
   - vagrant ssh-config：ssh相关配置
   - Vagrantfile：该文件可用于配置centos7，修改后需重启才能生效

### 使用xshell连接centos7

1. 修改root用户信息

   ```properties
   D:\opensource\virtual-system\centos7>vagrant ssh
   Last login: Thu Nov  7 00:36:28 2019 from 10.0.2.2
   #切换为root账户
   [vagrant@localhost ~]$ sudo -i
   #修改ssh配置:PasswordAuthentication yes
   [root@localhost ~]# vi /etc/ssh/sshd_config
   #修改密码
   [root@localhost ~]# passwd
   Changing password for user root.
   New password:
   BAD PASSWORD: The password is shorter than 8 characters
   Retype new password:
   Sorry, passwords do not match.
   New password:
   BAD PASSWORD: The password is shorter than 8 characters
   Retype new password:
   passwd: all authentication tokens updated successfully.
   #重启ssh服务
   [root@localhost ~]# systemctl restart sshd
   ```

2. 查看ssh相关信息

   ```properties
   D:\opensource\virtual-system\centos7>vagrant ssh-config
   Host default
     HostName 127.0.0.1
     User vagrant
     Port 2222
     UserKnownHostsFile /dev/null
     StrictHostKeyChecking no
     PasswordAuthentication no
     IdentityFile D:/opensource/virtual-system/centos7/.vagrant/machines/default/virtualbox/private_key
     IdentitiesOnly yes
     LogLevel FATAL
   ```



### Vagrantfile使用示例

```properties
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"
  
  #公共网络
  config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  
  config.vm.provider "virtualbox" do |vb|
	vb.memory = "4000"
	vb.name = "centos7-server01"
	vb.cpus = "2"
  end
  
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end

```



## box的打包分发

```properties
01 退出虚拟机
	vagrant halt

02 打包
	vagrant package --output first-docker-centos7.box
	
03 得到first-docker-centos7.box
	
04 将first-docker-centos7.box添加到其他的vagrant环境中
	vagrant box add first-docker-centos7 first-docker-centos7.box
	
05 得到Vagrantfile
	vagrant init first-docker-centos7

06 根据Vagrantfile启动虚拟机
	vagrant up [此时可以得到和之前一模一样的环境，但是网络要重新配置]
```



## docker安装

https://docs.docker.com/install/linux/docker-ce/centos/

```properties
01 进入centos7
	vagrant ssh
	
02 卸载之前的docker
	sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
03 安装必要的依赖
	sudo yum install -y yum-utils \
    device-mapper-persistent-data \
    lvm2
    
04 设置docker仓库  [设置阿里云镜像仓库可以先自行百度，后面课程也会有自己的docker hub讲解]	
	sudo yum-config-manager \
      --add-repo \
      https://download.docker.com/linux/centos/docker-ce.repo
      
    [访问这个地址，使用自己的阿里云账号登录，查看菜单栏左下角，发现有一个镜像加速器:https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors]

05 安装docker
	sudo yum install -y docker-ce docker-ce-cli containerd.io
	
06 启动docker
	sudo systemctl start docker
	
07 测试docker安装是否成功
	sudo docker run hello-world
```

## docker初体验

```properties
01 创建tomcat容器
	docker pull tomcat
	docker run -d --name my-tomcat -p 9090:8080 tomcat

02 创建mysql容器
	docker run -d --name my-mysql -p 3301:3306 -e MYSQL_ROOT_PASSWORD=jack123 --privileged mysql
	
03 进入到容器里面中并交互式运行
	docker exec -it containerid /bin/bash
```

docker run中参数解释：

- -d：让容器在后台运行，即以一个进程运行
- --name：给容器命名
- -p：将容器的端口映射到宿主主机的端口，9090为宿主主机端口，8080为docker中运行端口

### 常用命令

```shell
docker pull #拉去镜像
docker run #根据某个镜像运行容器
-d	#后台运行
--name #指定要创建容器的名称
-p	#将容器的端口映射到宿主主机的端口
docker exec -it #进入到某个容器
	例：docker exec -it tomcat01 bash
docker rmi #删除某个容器
docker ps #查看运行的container
docker rm #删除container
```



查看centos7系统的IP：

```properties
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83219sec preferred_lft 83219sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:60:46:64 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.105/24 brd 192.168.1.255 scope global noprefixroute dynamic eth1
       valid_lft 4019sec preferred_lft 4019sec
    inet6 fe80::a00:27ff:fe60:4664/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:bc:0d:c4:00 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:bcff:fe0d:c400/64 scope link 
       valid_lft forever preferred_lft forever
8: veth3575872@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 96:82:47:c4:d1:32 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::9482:47ff:fec4:d132/64 scope link 
       valid_lft forever preferred_lft forever
```

访问http://192.168.1.105:9090/可见tomcat运行中

## 问题

1. docker哪里拉取的镜像

   默认是在hub.docker.hub

2. docker pull tomcat拉取的是哪个版本？

   默认是最新版本，可以在后面指定版本“:”

3. 常用的简单命令

```properties
docker pull        拉取镜像到本地
docker run         根据某个镜像创建容器
docker images	查看images
docker ps 		查看运行的container
docker ps -a	查看所有container，包括停止的
```



# Image和Container

## Image探讨

![1575182591973](docker.assets\1575182591973.png)

在docker中，container运行在image上（是image的实例），而image是有一层一层的layer组成的，如运行一个jar包，则需要jdk image，而jdk image又需要centos image（一般这个centos image是体积较小的，即功能较少的）

image一般是基于一个dockerfile文件构建的，参考官方的image：https://github.com/docker-library

如mysql的dockerfile文件：

8.0版本：

```dockerfile
FROM debian:stretch-slim

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN apt-get update && apt-get install -y --no-install-recommends gnupg dirmngr && rm -rf /var/lib/apt/lists/*

# add gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& gpgconf --kill all \
	&& rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apt-get purge -y --auto-remove ca-certificates wget

RUN mkdir /docker-entrypoint-initdb.d

RUN apt-get update && apt-get install -y --no-install-recommends \
# for MYSQL_RANDOM_ROOT_PASSWORD
		pwgen \
# for mysql_ssl_rsa_setup
		openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
		perl \
	&& rm -rf /var/lib/apt/lists/*

RUN set -ex; \
# gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
	key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
	export GNUPGHOME="$(mktemp -d)"; \
	gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	gpg --batch --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
	gpgconf --kill all; \
	rm -rf "$GNUPGHOME"; \
	apt-key list > /dev/null

ENV MYSQL_MAJOR 8.0
ENV MYSQL_VERSION 8.0.18-1debian9

RUN echo "deb http://repo.mysql.com/apt/debian/ stretch mysql-${MYSQL_MAJOR}" > /etc/apt/sources.list.d/mysql.list

# the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
# also, we set debconf keys to make APT a little quieter
RUN { \
		echo mysql-community-server mysql-community-server/data-dir select ''; \
		echo mysql-community-server mysql-community-server/root-pass password ''; \
		echo mysql-community-server mysql-community-server/re-root-pass password ''; \
		echo mysql-community-server mysql-community-server/remove-test-db select false; \
	} | debconf-set-selections \
	&& apt-get update && apt-get install -y mysql-community-client="${MYSQL_VERSION}" mysql-community-server-core="${MYSQL_VERSION}" && rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld \
	&& chown -R mysql:mysql /var/lib/mysql /var/run/mysqld \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
	&& chmod 777 /var/run/mysqld

VOLUME /var/lib/mysql
# Config files
COPY config/ /etc/mysql/
COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306 33060
CMD ["mysqld"]
```

5.7版本：

```dockerfile
FROM debian:stretch-slim

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN apt-get update && apt-get install -y --no-install-recommends gnupg dirmngr && rm -rf /var/lib/apt/lists/*

# add gosu for easy step-down from root
ENV GOSU_VERSION 1.7
RUN set -x \
	&& apt-get update && apt-get install -y --no-install-recommends ca-certificates wget && rm -rf /var/lib/apt/lists/* \
	&& wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
	&& wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4 \
	&& gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
	&& gpgconf --kill all \
	&& rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu \
	&& gosu nobody true \
	&& apt-get purge -y --auto-remove ca-certificates wget

RUN mkdir /docker-entrypoint-initdb.d

RUN apt-get update && apt-get install -y --no-install-recommends \
# for MYSQL_RANDOM_ROOT_PASSWORD
		pwgen \
# for mysql_ssl_rsa_setup
		openssl \
# FATAL ERROR: please install the following Perl modules before executing /usr/local/mysql/scripts/mysql_install_db:
# File::Basename
# File::Copy
# Sys::Hostname
# Data::Dumper
		perl \
	&& rm -rf /var/lib/apt/lists/*

RUN set -ex; \
# gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
	key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
	export GNUPGHOME="$(mktemp -d)"; \
	gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	gpg --batch --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
	gpgconf --kill all; \
	rm -rf "$GNUPGHOME"; \
	apt-key list > /dev/null

ENV MYSQL_MAJOR 5.7
ENV MYSQL_VERSION 5.7.28-1debian9

RUN echo "deb http://repo.mysql.com/apt/debian/ stretch mysql-${MYSQL_MAJOR}" > /etc/apt/sources.list.d/mysql.list

# the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
# also, we set debconf keys to make APT a little quieter
RUN { \
		echo mysql-community-server mysql-community-server/data-dir select ''; \
		echo mysql-community-server mysql-community-server/root-pass password ''; \
		echo mysql-community-server mysql-community-server/re-root-pass password ''; \
		echo mysql-community-server mysql-community-server/remove-test-db select false; \
	} | debconf-set-selections \
	&& apt-get update && apt-get install -y mysql-server="${MYSQL_VERSION}" && rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/lib/mysql && mkdir -p /var/lib/mysql /var/run/mysqld \
	&& chown -R mysql:mysql /var/lib/mysql /var/run/mysqld \
# ensure that /var/run/mysqld (used for socket and lock files) is writable regardless of the UID our mysqld instance ends up having at runtime
	&& chmod 777 /var/run/mysqld \
# comment out a few problematic configuration values
	&& find /etc/mysql/ -name '*.cnf' -print0 \
		| xargs -0 grep -lZE '^(bind-address|log)' \
		| xargs -rt -0 sed -Ei 's/^(bind-address|log)/#&/' \
# don't reverse lookup hostnames, they are usually another container
	&& echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/conf.d/docker.cnf

VOLUME /var/lib/mysql

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 3306 33060
CMD ["mysqld"]
```

dockerfile文件常用命令：

- FROM：指定基础镜像，如

  ```dockerfile
  FROM ubuntu:14.04
  ```

- RUN：在镜像内部执行一些命令，比如安装软件，配置环境的等，换行可使用“”

  ```dockerfile
  RUN groupadd -r mysql && useradd -r -g mysql mysql
  ```

- ENV：设置变量的值，可通过docker run --e key=value修改，后面可以直接使用${key}，如

  ```dockerfile
  ENV MYSQL_MAJOR 5.7
  ```

- LABEL：设置镜像标签

  ```dockerfile
  LABEL email="zhangsan@163.com"
  LABEL name="zhangsan"
  ```

- VOLUME：指定数据的挂载目录

  ```dockerfile
  VOLUME /var/lib/mysql
  ```

- COPY：将主机文件复制到镜像内，如果目录不存在会自动创建所需的目录。注意只是复制，不会提取和解压

  ```dockerfile
  COPY docker-entrypoint.sh usr/local/bin/
  ```

- ADD：将主机文件复制到镜像内，和COPY类似，不过ADD会对压缩文件提取和解压

  ```dockerfile
  ADD application.xml /etc/zhangsan/
  ```

- WORKDIR：指定镜像工作目录，之后的命令都是基于此命令工作，目录不存在自动创建

  ```dockerfile
  WORKDIR /usr/local
  WORKDIR tomcat
  RUN touch test.txt
  ```

  > 会在usr/local/tomcat目录下创建test.txt文件

  ```dockerfile
  WORKDIR /root
  ADD app.xml test/
  ```

  > 会在/root/test目录下新增一个app.xml文件

- CMD：容器启动时默认会执行的命令，若有多条CMD命令，最后一条生效

  ```dockerfile
  CMD ["mysqld"]
  或
  CMD mysqld
  ```

- ENTRYPOINT：与CMD类似，与CMD不同的是，docker run时，会覆盖CMD的命令，但ENTRYPOINT不会

  ```dockerfile
  ENTRYPOINT ["docker-entrypoint.sh"]
  ```

- EXPOSE：指定镜像要暴露的端口，使用镜像时，可以使用-p将端口映射为主机端口

  ```dockerfile
  EXPOSE 3306
  ```



### 通过docker创建一个springboot项目image

```dockerfile
(1)创建一个Spring Boot项目
(2)写一个controller
@RestController
public class DockerController {
@GetMapping("/dockerfile")
@ResponseBody
String dockerfile() {
return "hello docker" ;
}
}
(3)mvn clean package打成一个jar包
在target下找到"dockerfile-demo-0.0.1-SNAPSHOT.jar"
(4)在docker环境中新建一个目录"first-dockerfile"
(5)上传"dockerfile-demo-0.0.1-SNAPSHOT.jar"到该目录下，并且在此目录创建Dockerfile
(6)创建Dockerfile文件，编写内容
FROM openjdk:8
MAINTAINER itcrazy2016
LABEL name="dockerfile-demo" version="1.0" author="itcrazy2016"
COPY dockerfile-demo-0.0.1-SNAPSHOT.jar dockerfile-image.jar
CMD ["java","-jar","dockerfile-image.jar"]
(7)基于Dockerfile构建镜像(.为基于当前路径构建)
docker build -t test-docker-image .
(8)基于image创建container
docker run -d --name user01 -p 6666:8080 test-docker-image
(9)查看启动日志docker logs user01
(10)宿主机上访问curl localhost:6666/dockerfile
hello docker
(11)还可以再次启动一个
docker run -d --name user02 -p 8081:8080 test-docker-image
```



### 镜像仓库

默认image是通过官网的docker hub拉取的，我们也可以配置为从阿里云docker hub或者自己搭建的docker hub拉取

#### 官方docker hub

首先在官网注册账号：hub.docker.com  (jackoly/1510134048@qq.com)

```properties
(1)在docker机器上登录
docker login
(2)输入用户名和密码
(3)docker push itcrazy2018/test-docker-image
[注意镜像名称要和docker id一致，不然push不成功]
(4)给image重命名，并删除掉原来的
docker tag test-docker-image itcrazy2018/test-docker-image
docker rmi -f test-docker-image
(5)再次推送，刷新hub.docker.com后台，发现成功
(6)别人下载，并且运行
docker pull itcrazy2018/test-docker-image
docker run -d --name user01 -p 6661:8080 itcrazy2018/test-docker-image
```



#### 阿里云docker hub

阿里云镜像仓库：https://cr.console.aliyun.com/cn-hangzhou/instances/repositories

```properties
(1)登录到阿里云docker仓库
sudo docker login --username=itcrazy2016@163.com registry.cnhangzhou.aliyuncs.com
(2)输入密码
(3)创建命名空间，比如itcrazy2016
(4)给image打tag
sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/itcrazy2016/testdocker-image:v1.0
(5)推送镜像到docker阿里云仓库
sudo docker push registry.cn-hangzhou.aliyuncs.com/itcrazy2016/test-dockerimage:v1.0
(6)别人下载，并且运行
docker pull registry.cn-hangzhou.aliyuncs.com/itcrazy2016/test-dockerimage:v1.0
docker run -d --name user01 -p 6661:8080 registry.cnhangzhou.aliyuncs.com/itcrazy2016/test-docker-image:v1.0
```



#### 自行搭建docker hub

```properties
(1)访问github上的harbor项目
咕泡学院 只为更好地你2.1.5 Image常见操作
2.2 深入探讨Container
既然container是由image运行起来的，那么是否可以理解为container和image有某种关系？
https://github.com/goharbor/harbor
(2)下载版本，比如1.7.1
https://github.com/goharbor/harbor/releases
(3)找一台安装了docker-compose[这个后面的课程会讲解]，上传并解压
tar -zxvf xxx.tar.gz
(4)进入到harbor目录
修改harbor.cfg文件，主要是ip地址的修改成当前机器的ip地址
同时也可以看到Harbor的密码，默认是Harbor12345
(5)安装harbor，需要一些时间
sh install.sh
(6)浏览器访问，比如39.100.39.63，输入用户名和密码即可
```



### image常用操作

```shell
(1)查看本地image列表
	docker images
	docker image ls

(2)获取远端镜像
	docker pull
	
(3)删除镜像（若有正在使用的镜像，或者有关联，则先处理完）
	docker image rm imageid
	docker rmi -f imageid
	docker rmi -f $(docker image ls)  #删除全部
	
(4)运行镜像
	docker run imageid
	
(5)发布镜像
	docker push	#发布到远端仓库
```



## Container探讨

![1575185564918](docker.assets\1575185564918.png)

container是根据image创建出来的，它是运行在image layer上的一层，而且在Container中，文件系统是可读可写的，而Container以下的layer是只读的



### Container到Image

从前面我们得知，image可以由一个dockerfile构建而来。而Container是由Image生成的，我们能不能根据Container得到一个Image，让别人共享？答案是可以的



> 不建议使用Container得到Image，因为这样Image的生成过程是透明的，而通过dockerfile构建的有详细的步骤，排查问题较容易



### Container资源限制

若不对Container使用的资源限制，它就会无限制的使用物理机的资源，这显然不合适。我们可以对其使用的资源进行限制

查看资源情况：docker stats

#### 内存限制

启动container增加参数 --memory limit，如：

```shell
docker run -d --memory 100M --name tomcat1 tomcat
```

查看docker stats：

![1575186551433](docker.assets\1575186551433.png)



#### CPU限制

启动时增加参数--cpu-shares 权重，如

```shell
docker run -d --cpu-shares 10 --name tomcat2 tomcat
```



#### 图形监控

我们可以通过第三方实现的图像监控界面来监控docker中应用所使用的资源，如scope（其实就是一个container）

https://github.com/weaveworks/scope

Getting start：

```shell
sudo curl -L git.io/scope -o /usr/local/bin/scope
sudo chmod a+x /usr/local/bin/scope
scope launch 39.100.39.63
```

访问4040宿主主机4040端口：

![image-20200526211905553](docker.assets\image-20200526211905553.png)

```shell
# 停止scope
scope stop
# 同时监控两台机器，在两台机器中分别执行如下命令
scope launch ip1 ip2
```





## Container常用操作

```shell
(1)根据镜像创建容器
	docker run -d --name -p 9090:8080 my-tomcat tomcat
	
(2)查看运行中的container
	docker ps
	
(3)查看所有的container（包括退出的）
	docker ps -a
	
(4)删除container
	docker rm containerid
	docker rm -f $(docker ps -a) #删除所有

(5)进入到一个container中
	docker exec -it container bash
	
(6)根据container生成image
	docker commit containerid imageid
	
(7)查看某个container日志
	docker logs containerid
	
(8)查看容器资源使用情况
	docker stats
	
(9)查看容器详细信息
	docker inspect containerid
	
(10)停止/启动容器
	docker stop/start containerid
```



## 底层技术支持

Container是一种轻量级的虚拟化技术，不容模拟硬件创建虚拟机

Docker是基于Linux Kernel的Namespace、CGroups、UnionFileSystem等技术封装成的一种自定义容器格式，从而提供一套虚拟环境

- Namespace：用来做隔离的，如pid（进程）、net（网络）、mnt（挂载点）等
- CGroups：Controller Groups用来做资源限制，比如内存和CPU等
- Union file System：用来做image和Container分层





虚拟化技术和容器化技术

阿里云镜像加速器

19.03.4

桥接模式







contanier layer可读可写





# Docker网络配置

常规的网络模型分为7层模型和4层模型：

![image-20200531182527494](docker.assets\image-20200531182527494.png)

在网络中，各个物理机器想要通信是通过网卡进行的。



## Linux中的网卡

网卡信息查看命令：

```shell
#简略信息
[root@localhost ~]# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 08:00:27:60:46:64 brd ff:ff:ff:ff:ff:ff
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default 
    link/ether 02:42:65:29:88:67 brd ff:ff:ff:ff:ff:ff
#以文件夹的形式呈现
[root@localhost ~]# ls /sys/class/net
docker0  eth0  eth1  lo
#详细信息
[root@localhost ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85293sec preferred_lft 85293sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:60:46:64 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.104/24 brd 192.168.1.255 scope global noprefixroute dynamic eth1
       valid_lft 6093sec preferred_lft 6093sec
    inet6 fe80::a00:27ff:fe60:4664/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:65:29:88:67 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

- 前面是网卡名，lo是本地通讯回环地址，eth0等是与外部通讯的网卡
- 状态：UP/DOWN/UNKNOW等
  - BROADCAST：网卡有广播地址，可以发送广播包
  - MULTICAST：网卡可以发送多播包
  - LOWER_UP：L1是启动的，即网卡是插着的
- link/ether：MAC地址
- inet：ipv4地址
- inet6：ipv6地址



### 配置文件

linux中的网卡其实对应的就是配置文件：

```shell
[root@localhost ~]# cd /etc/sysconfig/network-scripts/
[root@localhost network-scripts]# ls
ifcfg-eth0  ifdown       ifdown-ippp  ifdown-post    ifdown-sit       ifdown-tunnel  ifup-bnep  ifup-ipv6  ifup-plusb  ifup-routes  ifup-TeamPort  init.ipv6-global
ifcfg-eth1  ifdown-bnep  ifdown-ipv6  ifdown-ppp     ifdown-Team      ifup           ifup-eth   ifup-isdn  ifup-post   ifup-sit     ifup-tunnel    network-functions
ifcfg-lo    ifdown-eth   ifdown-isdn  ifdown-routes  ifdown-TeamPort  ifup-aliases   ifup-ippp  ifup-plip  ifup-ppp    ifup-Team    ifup-wireless  network-functions-ipv6
[root@localhost network-scripts]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE="eth0"
BOOTPROTO="dhcp"
ONBOOT="yes"
TYPE="Ethernet"
PERSISTENT_DHCLIENT="yes"
```



### 给网卡添加IP地址

```shell
[root@localhost /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84585sec preferred_lft 84585sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:60:46:64 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.104/24 brd 192.168.1.255 scope global noprefixroute dynamic eth1
       valid_lft 5385sec preferred_lft 5385sec
    inet6 fe80::a00:27ff:fe60:4664/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:65:29:88:67 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
#添加IP地址
[root@localhost /]# ip addr add 192.168.0.101/24 dev eth0
[root@localhost /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84547sec preferred_lft 84547sec
    inet 192.168.0.101/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:60:46:64 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.104/24 brd 192.168.1.255 scope global noprefixroute dynamic eth1
       valid_lft 5347sec preferred_lft 5347sec
    inet6 fe80::a00:27ff:fe60:4664/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:65:29:88:67 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
#删除IP地址
[root@localhost /]# ip addr delete 192.168.0.101/24 dev eth0
[root@localhost /]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84472sec preferred_lft 84472sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:60:46:64 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.104/24 brd 192.168.1.255 scope global noprefixroute dynamic eth1
       valid_lft 5272sec preferred_lft 5272sec
    inet6 fe80::a00:27ff:fe60:4664/64 scope link 
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:65:29:88:67 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```



### 网卡的启动和关闭

- 重启网卡：service network restart/systemctl restart network
- 启动和关闭
  - ifup eth0/ip link set eth0 up
  - ifdown eth0/ip link set eth0 down



## Network Namespace





bridge类型中笔记有缺失