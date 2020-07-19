#### 发布系统
1. maven project
clean package      （target）
2.本地发布：打包后解压到tomcat root文件夹中



[root@node37 ~]# cd /usr/cr999/tomcat8.5/bin
[root@node37 bin]# ./shutdown.sh 
[root@node37 bin]# ps -ef|grep tomcat
root     21074     1  2 Dec19 ?        00:33:27 /usr/cr999/java/jdk1.8/bin/java -Djava.util.logging.config.file=/usr/cr999/tomcat8.5/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -classpath /usr/cr999/tomcat8.5/bin/bootstrap.jar:/usr/cr999/tomcat8.5/bin/tomcat-juli.jar -Dcatalina.base=/usr/cr999/tomcat8.5 -Dcatalina.home=/usr/cr999/tomcat8.5 -Djava.io.tmpdir=/usr/cr999/tomcat8.5/temp org.apache.catalina.startup.Bootstrap start
root     22066 22027  0 15:11 pts/1    00:00:00 grep --color=auto tomcat
[root@node37 bin]# 
[root@node37 bin]# 
[root@node37 bin]# kill -9 21074
