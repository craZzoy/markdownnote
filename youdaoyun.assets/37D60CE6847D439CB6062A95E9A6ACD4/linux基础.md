# linux基础
## Linux系统启动过程
## Linux关机
正确关机流程：sync>shutdown>reboot>halt

关机指令：shutdown (man shutdown查看文档) 

```
#这个命令告诉大家，计算机将在10分钟后关机，并且会显示在登陆用户的当前屏幕中
shutdown –h 10 ‘This server will shutdown after 10 mins’ 。
#立马关机
shutdown -h now 
#今天20:30关机
shutdown -h 20:30
#10分钟后关机
shutdown -h +10
#系统立马重启
shutdown -r now
#10分钟后重启
shutdown -r +10
#重启 = shutdown -r now
reboot
#关闭系统 = shutdown -h now
halt
```

## linux系统目录结构
- bin：binary的缩写，这个目录存放着最经常使用的命令
- boot：存放的是启动linux时使用的一些核心文件，包括一些连接文件和镜像文件
- dev：device的缩写，存放的是linux的外部设备，在linux中访问设备的方式和访问文件的方式是相同的
- etc：存放所有的系统管理所需要的配置文件和子目录
- home：用户的主目录，linux中，每一个用户都有一个自己的目录，一般该目录名是以用户的账号命名的
- lib：存放着系统的系统的最基本的动态连接共享库，其作用类似于windows中dll文件。几乎所有的应用程序都会用到这个共享库
- lost+found：这里一般是空的，当系统非法关机后，这里就存放了一些文件
- media：linux系统会自动识别一些设备，如u盘、光驱等，当识别后，linux会把识别的设备挂载到这个目录
- mnt：系统提供给目录的目的是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载到这个目录，然后进入mnt目录就可以查看光驱的内容了
- opt：给主机额外安装软件的目录。比如安装一个oracle系统就可以放到这个目录。默认是空的
- proc：这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件，比如可以通过下面的命令来屏蔽主机的ping命令，使别人无法ping你的机器：
```
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
```
- root：该目录为系统管理员，也称作超级权限者的用户主目录
- sbin：s就是super user，这里存放着是系统管理员使用的系统管理程序
- srv：该目录存放一些服务启动之后需要提取的数据。
- sys：这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。

sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。
该文件系统是内核设备树的一个直观反映。

当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。
- tmp：用来存放一些临时文件
- usr： 这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。
- bin：系统用户使用的应用程序。
- usr/sbin：超级用户使用的比较高级的管理程序和系统守护程序
- usr/src：内核源代码存放的默认位置
- var：这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。
- run：是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。

在 Linux 系统中，有几个目录是比较重要的，平时需要注意不要误删除或者随意更改内部文件。

/etc： 上边也提到了，这个是系统中的配置文件，如果你更改了该目录下的某个文件可能会导致系统不能启动。

/bin, /sbin, /usr/bin, /usr/sbin: 这是系统预设的执行文件的放置目录，比如 ls 就是在/bin/ls 目录下的。

值得提出的是，/bin, /usr/bin 是给系统用户使用的指令（除root外的通用户），而/sbin, /usr/sbin 则是给root使用的指令。

/var： 这是一个非常重要的目录，系统上跑了很多程序，那么每个程序都会有相应的日志产生，而这些日志就被记录到这个目录下，具体在/var/log 目录下，另外mail的预设放置也是在这里。


## linux文件系统
查看文件属性以及文件所属的用户和组
```
[root@izwz9c61wsgboaq9aoxis9z /]# ll
total 60
lrwxrwxrwx.    1 root root     7 Oct 15  2017 bin -> usr/bin
dr-xr-xr-x.    5 root root  4096 Nov 12  2018 boot
drwxr-xr-x    19 root root  2980 Sep 18 16:29 dev
drwxr-xr-x.   82 root root  4096 Sep 18 14:01 etc
drwxr-xr-x.    2 root root  4096 Nov  5  2016 home
lrwxrwxrwx.    1 root root     7 Oct 15  2017 lib -> usr/lib
lrwxrwxrwx.    1 root root     9 Oct 15  2017 lib64 -> usr/lib64
drwx------.    2 root root 16384 Oct 15  2017 lost+found
drwxr-xr-x.    2 root root  4096 Nov  5  2016 media
drwxr-xr-x.    2 root root  4096 Nov  5  2016 mnt
drwxr-xr-x.    2 root root  4096 Nov  5  2016 opt
dr-xr-xr-x  1358 root root     0 Sep 18 13:48 proc
dr-xr-x---.    6 root root  4096 Sep 16 20:12 root
drwxr-xr-x    22 root root   640 Sep 18 14:19 run
lrwxrwxrwx.    1 root root     8 Oct 15  2017 sbin -> usr/sbin
drwxr-xr-x.    2 root root  4096 Nov  5  2016 srv
dr-xr-xr-x    13 root root     0 Sep 18  2019 sys
drwxrwxrwt.   11 root root  4096 Sep 18 16:25 tmp
drwxr-xr-x.   14 root root  4096 Nov 12  2018 usr
drwxr-xr-x.   20 root root  4096 Nov 13  2018 var
```
其中第一个字符表示这个文件是目录、文件或者链接文件，如bin第一个字符是‘l’，则表示它是一个链接文件，链接到了usr/bin

- d：目录
- -：文件
- l：链接文档
- b：装置文件里面可共存储的接口设备（可随机存储设备）
- c：装置文件里的串行端口设备，例如键盘、鼠标（一次性读取装置）








