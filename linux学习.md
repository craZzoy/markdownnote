# 初始Linux Shell

## 什么是Linux

linux可分为以下四部分：

- Linux内核
- GNU工具
- 图形化桌面环境
- 应用软件

linux系统结构：

![1573728294247](linux学习.assets\1573728294247.png)







# 文件链接

```properties
#查看文件的最终链接位置
readlink –f
```





# Bash Shell命令

## 处理数据文件

### 数据搜索

#### grep命令

格式：grep [option] pattern [file]

示例：

```shell
#在file1中搜索匹配three的文本
$ grep three file1
three
$ grep t file1
two
three
$

#反向搜索，即不匹配的
$ grep -v t file1
one
four
five
$

#可加上显示行号
$ grep -n t file1
2:two
3:three
$

#多少行匹配
$ grep -c t file1
2
$

#用-e指定多个模式，这里表示匹配问t和f
$ grep -e t -e f file1
two
three
four
five
$
```



结合正则表达式使用grep：

```shell
#
$ grep [tf] file1
two
three
four
five
$
```

> 正则表达式的[]表示匹配t或者f，若不使用正则，则匹配tf



#### grep的衍生命令

- egrep：支持posix拓展正则表达式
- fgrep：fgrep则是另外一个版本，支持将匹配模式指定为用换行符分隔的一列固定长度的字符串。这样就可以把这列字符串放到一个文件中，然后在fgrep命令中用其在一个大型文件中搜索字符串了 





# 防火墙（centos7）

- 查看防火墙
  - `systemctl status firewalld`
- 启动防火墙
  - `systemctl start firewalld`
- 关闭防火墙
  - `systemctl stop firewalld`



## 开放端口

### netstat

​	查看端口相关信息

选项和参数：

- -t：显示TCP端口
- -u：显示UDP端口
- -l：仅显示监听套接字
- -p：显示进程标识符和程序名称
- -n：不进行DNS轮询，显示IP

查看当前运行的tcp端口：

netstat -ntlp

查看所有80端口使用情况：

netstat -ntulp |grep 80

查看所有3306端口使用情况：

netstat -an | grep 3306



查看开放的端口：

```properties
firewall-cmd --list-port
```



```properties
#开放端口
[root@izwz9c61wsgboaq9aoxis9z redis-5.0.5]# firewall-cmd --zone=public --add-port=6379/tcp --permanent
success
[root@izwz9c61wsgboaq9aoxis9z redis-5.0.5]# firewall-cmd --zone=public --query-port=6379/tcp
no
[root@izwz9c61wsgboaq9aoxis9z redis-5.0.5]# firewall-cmd --reload
success
[root@izwz9c61wsgboaq9aoxis9z redis-5.0.5]# firewall-cmd --zone=public --query-port=6379/tcp
yes
```

# 编辑器

## Vim/Vi编辑器

### vim软件包检查

```properties
#查看别名
alias vi
#查看vim（alias vi输出为vim时）实际的位置
#注意在某些版本的linux系统中，vi被连接到一个较低权限的命令，如vim.tiny，它只提供了vim中较少的命令集合
which vim
#查看相关权限信息
ls -l /usr/bin/vim
```

在Ubuntu中安装基本版的vim包：

```properties
sudo apt-get install vim
```



### vim基础

vim的三种模式

- 普通模式

  - 移动光标命令

    - h（左）、j（上）、k（下）、l（右）
    - PgUp/PgDn：上/下翻一页
    - PgUp/PgDn：上/下翻一页
    - G：移动到缓冲区最后一行
    - num G：移动到缓冲区的第num行
    - gg：移动到缓冲区的第一行

  - 编辑数据命令

    - x：删除光标所在字符

      - 如2x是删除两个字符

    - dd：删除当前行

      - 如5dd是删除5行

    - dw：删除光标所在位置单词

    - d$：删除光标所在位置至行尾的内容

    - J：删除当前所在行行尾的换行符（用于拼接行）

    - u：撤销前一条编辑命令

    - a：在当前光标后追加内容

    - A：在当前行行位追加数据

    - r char：用char替换当前光标所在位置单个字符

    - R text：用text覆盖光标位置数据，直至按下esc键

    - ctrl+v：选中多行，按shift+i或者s进入编辑模式
    
      

- 插入模式

- 命令模式

  - q
  - wq
  - q!
  - w filename：保存到某个文件



### 复制和粘贴

- 复制、粘贴
  - y为复制命令，可以类似d命令一样使用，例如yw复制光标所在一个字符，y$表示复制到行尾
    - 例如：
      - v触发高亮选中需复制的文本，y触发复制，p粘贴数据
      - yy+p 复制当前行到下一行
- 剪切、粘贴：
  - 删除数据在缓冲区，用p命令可取回数据

### 查找和替换

#### 查找

命令模式下输入/

#### 替换（命令模式下执行）

- :s/old/new/g：一行命令替换所有old。
- :n,ms/old/new/g：替换行号n和m之间所有old。
- :%s/old/new/g：替换整个文件中的所有old。
- :%s/old/new/gc：替换整个文件中的所有old，但在每次出现时提示。 





# 编码问题

## 查看系统编码

cat /etc/sysconfig/i18n





# 发布相关

## jar命令

jar -xvf

jar -uvf



容器

jeckin



# Shell脚本编程

helloworld：

linux中shel分为：

- Bourne Shell（/usr/bin/sh或/bin/sh）
- Bourne Again Shell（/bin/bash）
- C Shell（/usr/bin/csh）
- K Shell（/usr/bin/ksh）
- Shell for Root（/sbin/sh）
- ...

helloworld实例：

```shell
#!/bin/bash
echo "Hello World !"
```

> 第一行标明使用哪种shell解析器执行

运行方式：

1. 作为可执行参数

   ```shell
   chmod +x ./test.sh  #使脚本具有执行权限
   ./test.sh  #执行脚本
   ```

2. 作为解释器参数

   ```shell
   /bin/sh test.sh
   /bin/php test.php
   ```




## Shell 变量

### 注释

- 单行注释：

  ```shell
  ##### 用户配置区 开始 #####
  ```

- 多行注释

  - 多个#

    ```shell
    #--------------------------------------------
    # 这是一个注释
    # author：菜鸟教程
    # site：www.runoob.com
    # slogan：学的不仅是技术，更是梦想！
    #--------------------------------------------
    ##### 用户配置区 开始 #####
    #
    #
    # 这里可以添加脚本描述信息
    # 
    #
    ##### 用户配置区 结束  #####
    ```

  - 用花括号括起来，将其定义为一个函数，没有地方调用这块代码就不会执行

  - 使用EOF

    ```shell
    :<<EOF
    注释内容...
    注释内容...
    注释内容...
    EOF
    ```

    或

    ```shell
    :<<'
    注释内容...
    注释内容...
    注释内容...
    '
    
    :<<!
    注释内容...
    注释内容...
    注释内容...
    !
    ```



### shell参数传递







查看缓存：
`free -m`

```bash
# 释放缓存区内存的方法

1）清理pagecache（页面缓存）

# echo 1 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=1

2）清理dentries（目录缓存）和inodes

# echo 2 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=2

3）清理pagecache、dentries和inodes

# echo 3 > /proc/sys/vm/drop_caches     或者 # sysctl -w vm.drop_caches=3

注：上面三种方式都是临时释放缓存的方法，要想永久释放缓存，需要在/etc/sysctl.conf文件中配置：vm.drop_caches=1/2/3，然后sysctl -p生效即可！
```