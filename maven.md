# 概述

作为Apache组织中的一个颇为成功的开源项目，maven主要服务于基于java平台的项目构建、依赖管理和项目信息管理。

maven的功能之一就是服务于构建，它能帮助我们自动或构建过程：从清理、编译、测试到生成报告，再到打包和部署。只需一个简单的命令，如mvn clean install，maven就能自动的帮我们执行一些重复的工作

maven功能：

- 构建
- 文档生成
- 报告
- 依赖
- SCMs
- 发布
- 分发
- 邮件列表

maven对比ant

- Ant起初用来构建tomcat，它是过程式的，避免不了一些重复性工作。ant没有依赖管理
- maven是声明式的，项目构建过程和过程各个阶段所需的工作都由插件来完成，而且大部分插件是现成的，所以消除了大量的重复性工作。maven有依赖管理



maven会创建默认的工程结构，如一些默认的配置如下（${basedir}表示工程目录）

| 配置项             | 默认值                        |
| ------------------ | ----------------------------- |
| source code        | ${basedir}/src/main/java      |
| resources          | ${basedir}/src/main/resources |
| Tests              | ${basedir}/src/test           |
| Complied byte code | ${basedir}/target             |
| distributable JAR  | ${basedir}/target/classes     |



## 安装

略



## maven目录

- bin：包含执行命令
- boot：maven默认类加载器
- conf：配置文件，如里面的setting.xml是全局配置文件
- lib：maven所需的java类库
- LICENCE：记录软件许可证
- NOTICE：记录maven包含的第三方软件
- README.txt：maven的简要介绍

常用命令：mvn help:system，打印出所有的Java系统属性和环境变量



## 配置相关

maven仓库的默认目录在用户目录下./m2文件夹下，如我本机是C:\Users\zwz\.m2



### 设置HTTP代理

有些公司对安全有要求，要求使用通过安全认证的代理访问因特网，即不能直接从maven官网更新资源，这时我们可以配置代理，通过代理帮我们下载资源。

检查是否能连接maven公共仓库：

```shell
ping repol.maven.org
```

用telnet检查能不能连接上代理

maven代理配置在./m2下（或者$M2_HOME/conf）的settings.xml文件中

```xml
  <proxies>
    <proxy>
      <id>optional</id>
      <active>true</active><!--是否激活-->
      <protocol>http</protocol><!--使用的代理协议-->
      <username>proxyuser</username><!--用户名、密码（代理服务可能配置了验证）-->
      <password>proxypass</password>
      <host>proxy.host.net</host><!--代理主机-->
      <port>80</port><!--代理端口-->
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts><!--不代理主机-->
    </proxy>
  </proxies>
```



## 常用实践

### 设置MAVEN_OPTS环境变量

mvn实际上执行的是java命令，也就是运行的是java程序，那么运行java命令可用的参数也应该在运行mvn命令时可用。MAVEN_OPTS环境变量常用来设置一些JVM参数，如：-Xms128m -Xmx512m

> 环境变量的设置方法参考M2_HOME



### 设置setting.xml文件

M2_HOME/conf目录下的setting.xml是maven的全局配置，.m2目录下的是用户配置，只影响当前用户。升级maven时，用户目录下的setting.xml不会被覆盖



# Maven使用入门

## 一些规范

maven项目规范目录：

- 默认java类所在目录：src/main/java
- 默认测试代码目录：src/test/java

一般包命令规范：group-id.artifactId



## 使用ArcheType生成项目骨架

maven3中命令：

```properties
mvn archetype:generate
```

maven2中最好执行，因为不指定版本会下载最新的snapshat版本，而在maven3只会下载最新的稳定版本

```properties
mvn org.apache.maven.plugins:maven-archetype-plugin:2.0-alpha-5:generate
```

> groupId:artifactId:version:goal

## 常用执行步骤

1. mvn clean compile：清理编译

2. mvn clean test：清理测试

   - 典型单元测试步骤（代码执行逻辑）：
     - 准备测试类和数据
     - 执行要测试的行为
     - 检查结果

3. mvn clean package：清理打包（默认为jar包）

4. mvn clean install：清理安装（安装到maven本地仓库或者远程仓库）

   - 使用maven生成可执行jar包：默认生成的jar包META-INF/MANIFEST.MF文件中不包含main-class信息，可借助maven-shade-plugin插件指定main-class：

   ![1575510884158](maven.assets\1575510884158.png)



eclipse中插件使用

执行多个步骤（Run As菜单提供单个步骤执行）：

![1575516749948](maven.assets\1575516749948.png)



## POM文件

常见pom文件内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.personal</groupId>
    <artifactId>java-base-demo</artifactId>
    <packaging>pom</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>java-base-demo</name>
    <modules>
        <module>lession1</module>
        <module>design-patterns-demo</module>
        <module>virtual-machine-demo</module>
        <module>common-interview</module>
        <module>concorrent-demo</module>
    </modules>

    <dependencyManagement>
        <dependencies>
            <!-- https://mvnrepository.com/artifact/net.sf.cglib/cglib-nodep-javadoc -->
            <dependency>
                <groupId>net.sf.cglib</groupId>
                <artifactId>cglib-nodep-javadoc</artifactId>
                <version>2.2</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

- `modelVersion`：当前pom模型的版本，对于maven2和maven3只能是4.0.0
- `groupId、artifactId、version`定义了一个项目基本的坐标，在maven世界中，任何的jar、pom或者war都是以这三个元素区分的
- name：项目名称，不是必须



compiler插件配置java编译版本

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
    <source>1.8</source>
    <target>1.8</target>
    </configuration>
</plugin>
```



# 坐标依赖

## maven坐标

坐标元素：

- groupId：当前maven项目隶属的实际项目
- artifactId：定义实际项目中的一个maven项目（模块）
- version：版本
- packaging：打包方式，可能为jar、war等，不定义默认是jar
- classifier：帮助定义构建输出的一些附属构件。附属构件与主构件对应，如xxx.jar主构件会有xxx-javadoc.jar、xxx-source.jar等附属构件。注意附属构件不是项目直接默认生成的，而是由附加的插件生成

> 注：构建名字一般类似artifactId-version-[classfier].jar这种方式



## 依赖

常见依赖内容如下：

```xml
<dependencyManagement>
    <dependencies>
        <!-- https://mvnrepository.com/artifact/net.sf.cglib/cglib-nodep-javadoc -->
        <dependency>
        <groupId>net.sf.cglib</groupId>
        <artifactId>cglib-nodep-javadoc</artifactId>
        <version>2.2</version>
        <type></type>
        <scope></scope>
        <optional></optional>
        <exclusions></exclusions>
        </dependency>
    </dependencies>
</dependencyManagement>
```

- groupId:artifactId:version

- type：依赖的类型，对应于项目坐标定义的packing，大部分情况下不用声明，其默认值为jar

- scope：依赖的范围，控制依赖与三种classpath（编译classpath、测试classpath、运行classpath）的关系

  - test：只对于测试classpath有效，即在测试代码中import Junit有效，在编译主代码或者运行项目的使用时将无法使用此依赖

  - compile：默认。编译依赖范围，对于三种classpath都有效

  - provided：已提供依赖范围。对于编译和测试classpath有效，但在运行时无效。如servlet-api，编译和测试该项目时需要该依赖，但运行时不需要，因为容器已经提供。

  - runtime：运行时依赖范围。对于测试和运行classpath有效，但在编译主代码时无效。比如JDBC驱动实现，项目主代码的编译只需要JDK提供的JDBC接口，只有在执行测试或者运行项目时才需要实现上述的具体驱动

  - system：系统依赖范围。与三种classpath的关系与provided一致，但需要指定systemPath，因为其不是依赖maven仓库解析的，而且往往与本机系统绑定，所以移植性低

    ![1575538126001](maven.assets\1575538126001.png)

  - import：导入依赖范围，不会对三种classpath产生实际影响。import在dependencyManagement元素中在有效，一般用来导入pom类型依赖，用来引入指定pom文件中的dependencyManagement依赖配置

    ![1576115211966](maven.assets\1576115211966.png)

    > 若有多个项目使用的依赖版本是一致的，则可以定义一个使用dependencyManagement专门管理依赖的POM，然后在各个项目中导入这些依赖管理配置

  | 依赖范围（scope） | 对于编译classpath有效 | 对于测试classpath有效 | 对于运行classpath有效 | 例子                            |
  | ----------------- | --------------------- | --------------------- | --------------------- | ------------------------------- |
  | compile           | Y                     | Y                     | Y                     | spring-core                     |
  | test              | -                     | Y                     | -                     | Junit                           |
  | provided          | Y                     | Y                     | -                     | servlet-api                     |
  | runtime           | -                     | Y                     | Y                     | JDBC驱动实现                    |
  | system            | Y                     | Y                     | -                     | 本地的，maven仓库之外的类库文件 |

- optional：标记依赖是否可选。表示一个项目实现了多种特性，如一个项目中实现了支持多种数据库连接（实际使用时只会依赖一种数据库）

  ![1575545598103](maven.assets\1575545598103.png)

  - 加入存在依赖关系：A->B，B->X(可选)，B->Y(可选)，实际X或者Y依赖并不会传递到A，A中需要显示声明依赖。如承上，A中需要显示声明依赖Mysql还是Postgresql

    > 按照单一职责原则，应该提供两种数据库依赖的实现，尽量少用可选依赖

- exclusions：用来排除传递性依赖，如：

  ![1575546296174](maven.assets\1575546296174.png)

  > exclusion只需指定groupId和artifactId

## 传递性依赖

传递性依赖是指加入A依赖B，B依赖C，那么A也会依赖C。A依赖B称为第一直接依赖，B依赖C称为第二直接依赖

### 依赖范围对传递性依赖的影响

| 第一直接/第二直接 | compile  | test | provided | runtime  |
| ----------------- | -------- | ---- | -------- | -------- |
| compile           | compile  | -    | -        | runtime  |
| test              | test     | -    | -        | test     |
| provided          | provided | -    | provided | provided |
| runtime           | runtime  | -    | -        | runtime  |

> 左边一列为第一直接依赖范围，第一行是第二直接依赖范围，中间是传递性依赖的范围

规律总结：

- 当第二直接依赖为compile时，传递性依赖范围与第二直接依赖一致
- 当第二直接依赖为test时，不传递依赖
- 当第二直接依赖为provided时，只传递范围为provided的依赖
- 当第二直接依赖为runtime时，除第一直接依赖为compile时，其他与第二直接依赖一致



## 依赖调解

对于冲突的依赖，maven有两种依赖调解原则：

- 路径最近者优先
  - 如存在依赖关系：A->B->C->X(1.0)，A->D->X(2.0)，由于第二条路径较短，故最终选择X(2.0)
- 路径长度一样时，第一声明优先（maven2.0.9及以后）
  - 如存在依赖关系：A->B->X(1.0)，A->C->X(2.0)，两个版本依赖路径一样长，假如X(1.0)声明在X(2.0)之前，则会选择X(1.0)



## 优化依赖

查看当前项目已解析依赖：

mvn dependency:list

查看当前项目依赖树：

mvn dependency:tree

分析当前项目依赖：

mvn dependency:analyze



# 仓库

坐标与依赖在仓库中的路径的关系为：

groupId/artifactId/version/artifactId-version.packaging



## 仓库分类

- 本地仓库

  - maven仓库的默认目录在用户目录下./m2文件夹下，如我本机是C:\Users\zwz\.m2

  可在setting.xml文件中设置本地仓库地址：

  ```xml
    <!-- localRepository
     | The path to the local repository maven will use to store artifacts.
     |
     | Default: ${user.home}/.m2/repository
    <localRepository>/path/to/local/repo</localRepository>
    -->
  ```

  > 可使用mvn install命令将对应的依赖包安装到本地

- 远程仓库

  - 中央仓库：在M2_HOME\lib中的maven-model-builder-3.6.0.jar中的`org\apache\maven\model\pom-4.0.0.xml`（此配置文件是所有maven项目都会继承的超级POM）存在如下配置：

    ```xml
      <repositories>
        <repository>
          <id>central</id>
          <name>Central Repository</name>
          <url>https://repo.maven.apache.org/maven2</url>
          <layout>default</layout>
          <snapshots>
            <enabled>false</enabled>
          </snapshots>
        </repository>
      </repositories>
    ```

    - id：唯一标识
    - snapshots：false表示不从中央仓库中下载快照版本的构件

  - 私服：自行搭建的一个maven仓库，依赖本地找不到时会到私服下载，当私服不存在对应依赖时，私服会先将远程仓库的依赖缓存到私服中，然后提供下载。

    ![1575550812314](maven.assets\1575550812314.png)

    - 使用私服的好处：
      - 节省外网带宽
    - 加速maven构建：因为maven构建时会不断检查远程仓库数据，私服在同个局域网内会快很多
      - 部署第三方构件：某些第三方构件无法从外部远程仓库获取时，如oracle的JDBC驱动，建立私服后可把这些构件部署到内部仓库中，供内部使用
      - 提高稳定性，增强控制：私服网路较稳定
      - 降低中央仓库的负荷
  
  - 其他公共库

> 当需要更改或者需要多个远程仓库时，可以在POM文件或者setting.xml文件中配置



## 远程仓库配置

可在pom文件或者setting文件中配置：

例1：

![1575892976190](maven.assets\1575892976190.png)

例2：

![1575708591693](maven.assets\1575708591693.png)

- releasse：仓库的版本下载支持配置
  - enabled=true：表示支持仓库的版本下载支持
- snapshots：仓库快照版本下载支持配置
  - enabled
  - updatePolicy：更新频率，默认daily
    - daily：每天检查一次
    - never：从不检查
    - always：每次构建都检查
    - `interval:x`：每隔x分钟检查一次
  - checksumPolicy：配置maven检查校验和文件的策略（部署时会部署对应的校验和文件，下载时也会下载）
    - warn：警告信息
    - fail：报错，构建失败
    - ignore：忽略

## 部署至远程仓库

在pom.xml文件中配置：

![1575892400514](maven.assets\1575892400514.png)

- repository：发布版本构件的仓库
  - id：仓库标识，若有验证，与server中id一致
  - name：方便查看
  - url：仓库地址
- snapshotRepository：发布快照版本的仓库

配置完执行命令：mvn clean deploy



## 快照版本

对于快照版本，每次发布时maven会在版本名字后面增加时间戳，这样每次都能更新到最新的版本，而发布版本没这效果，更新频率取决于`updatePolicy`参数，可强制更新：mvn clean install-U

- RELEASE：计算最新发版版本
- LATEST：最终版本，计算时会计算所有版本（发布版、快照版本）



## 镜像

```xml
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>
```

镜像表示从mirrorOf仓库中下载的依赖包从配置的镜像地址中下载，如上面的central表示配置为中央仓库的镜像

- id：唯一标识，若需要配置验证，则可以根据此id对应相应的server
- name：名字
- url：镜像地址
- mirrorOf：配置作为某些仓库的镜像
  - `<mirrorOf>*</mirrorOf>`：匹配所有远程仓库
  - `<mirrorOf>external:*</mirrorOf>`：匹配所有远程仓库，lasthost、使用file文件协议的除外，即除本机上的远程仓库
  - `<mirrorOf>repo1,repo2</mirrorOf>`：匹配多个远程仓库
  - `<mirrorOf>*,!repo1</mirrorOf>`：匹配除repo1外的所有远程仓库



## 仓库搜索服务

- sonatype Nexus
- Jarvana
- MVNbrowser
- MVNrepository

都代理了主流的maven公共仓库：central、JBoss、Java.net等

# setting.xml文件配置(3.6)

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->

  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->

  <!-- offline
   | Determines whether maven should attempt to connect to the network when executing a build.
   | This will have an effect on artifact downloads, artifact deployment, and others.
   |
   | Default: false
  <offline>false</offline>
  -->

  <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <!--认证信息配置：id用于认证信息与仓库配置联系（即id与仓库id一致）-->
  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     |
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->

    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

  <!-- mirrors
   | This is a list of mirrors to be used in downloading artifacts from remote repositories.
   |
   | It works like this: a POM may declare a repository to use in resolving certain artifacts.
   | However, this repository may have problems with heavy traffic at times, so people have mirrored
   | it to several places.
   |
   | That repository definition will have a unique id, so we can create a mirror reference for that
   | repository, to be used as an alternate download site. The mirror site will be the preferred
   | server for that repository.
   |-->
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  </mirrors>

  <!-- profiles
   | This is a list of profiles which can be activated in a variety of ways, and which can modify
   | the build process. Profiles provided in the settings.xml are intended to provide local machine-
   | specific paths and repository locations which allow the build to work in the local environment.
   |
   | For example, if you have an integration testing plugin - like cactus - that needs to know where
   | your Tomcat instance is installed, you can provide a variable here such that the variable is
   | dereferenced during the build process to configure the cactus plugin.
   |
   | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
   | section of this document (settings.xml) - will be discussed later. Another way essentially
   | relies on the detection of a system property, either matching a particular value for the property,
   | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
   | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
   | Finally, the list of active profiles can be specified directly from the command line.
   |
   | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
   |       repositories, plugin repositories, and free-form properties to be used as configuration
   |       variables for plugins in the POM.
   |
   |-->
  <profiles>
    <!-- profile
     | Specifies a set of introductions to the build process, to be activated using one or more of the
     | mechanisms described above. For inheritance purposes, and to activate profiles via <activatedProfiles/>
     | or the command line, profiles have to have an ID that is unique.
     |
     | An encouraged best practice for profile identification is to use a consistent naming convention
     | for profiles, such as 'env-dev', 'env-test', 'env-production', 'user-jdcasey', 'user-brett', etc.
     | This will make it more intuitive to understand what the set of introduced profiles is attempting
     | to accomplish, particularly when you only have a list of profile id's for debug.
     |
     | This profile example uses the JDK version to trigger activation, and provides a JDK-specific repo.
    <profile>
      <id>jdk-1.4</id>

      <activation>
        <jdk>1.4</jdk>
      </activation>

      <repositories>
        <repository>
          <id>jdk14</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://www.myhost.com/maven/jdk14</url>
          <layout>default</layout>
          <snapshotPolicy>always</snapshotPolicy>
        </repository>
      </repositories>
    </profile>
    -->

    <!--
     | Here is another profile, activated by the system property 'target-env' with a value of 'dev',
     | which provides a specific path to the Tomcat instance. To use this, your plugin configuration
     | might hypothetically look like:
     |
     | ...
     | <plugin>
     |   <groupId>org.myco.myplugins</groupId>
     |   <artifactId>myplugin</artifactId>
     |
     |   <configuration>
     |     <tomcatLocation>${tomcatPath}</tomcatLocation>
     |   </configuration>
     | </plugin>
     | ...
     |
     | NOTE: If you just wanted to inject this configuration whenever someone set 'target-env' to
     |       anything, you could just leave off the <value/> inside the activation-property.
     |
    <profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>

  <!-- activeProfiles
   | List of profiles that are active for all builds.
   |
  <activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
  -->
</settings>

```



# 生命周期和插件

maven构建项目的过程是通过生命周期来体现的，不同的生命周期执行不同的任务，而生命周期具体的行为由插件实现。可理解为这是一种模板模式的体现，生命周期就是一个模板，具体的实现交给插件，对应某个生命周期，maven有默认的插件，当然用户也可以自定义实现插件



## 三套生命周期

maven的有三套独立生命周期：clean，default，site。每个生命周期分多个阶段，后面阶段依赖前面阶段，如执行mvn clean命令时会执行两个阶段：pre-clean、clean

### clean生命周期

主要用于清理项目，包含三个阶段：

1. pre clean：清理前工作
2. clean：清理上一次构建生成文件
3. post clean：清理后工作



### default生命周期

定义了真正构建时所需执行的步骤，是所有生命周期中最核心的部分

1. validate
2. initialize
3. generate-sources
4. process-sources：处理项目注资源文件。一般来说是对src/main/resources目录的内容进行变量替换等工作后，复制到项目输出的主classpath目录中
5. generate-resources
6. process-resources
7. compile：编译项目主代码。一般是编译src/main/java目录下的java文件至项目输出的主classpath目录中
8. process-classes
9. generate-test-sources
10. process-test-sources：处理项目注资源文件。一般来说是对src/test/resources目录的内容进行变量替换等工作后，复制到项目输出的测试classpath目录中
11. generate-test-resources
12. process-test-resources
13. test-compile：编译项目测试代码。一般是编译src/test/java目录下的java文件至项目输出的测试classpath目录中
14. process-test-classes
15. test：执行单元测试框架运行测试，测试代码不会被打包或者部署
16. prepare-package
17. package：将编译好的代码打包成可发布的形式，如jar
18. pre-integration-test
19. integration-test
20. post-integration-test
21. verify
22. install：将包安装安装到本地
23. deploy：将最终的包发布到远程仓库

> 更多参考官网

### site生命周期

用于建立和发布项目站点

1. pre-site
2. site：生成项目站点文档
3. post-site
4. site-depoly：将生成的项目站点发布到服务器

## 命令行与生命周期

- mvn clean：实际执行clean生命周期的pre-clean和clean阶段
- mvn test：实际执行default声明周期validate至test阶段
- mvn clean install：clean生命周期的pre-clean、clean阶段和default生命周期validate至install阶段

# Maven-Pom

pom文件中的配置

- project dependencies
- plugins
- goals
- build profiles
- project version
- developers
- mailing list 



# 插件

## 插件目标

每个插件有多个目标，每个目标具体完成不同的任务。如maven-dependency-plugin插件有多个目标：

- dependency:analyze
- dependency:tree
- dependency:list

> :前面是前缀，后面是目标

### 内置绑定

maven的生命周期的阶段是通过插件实现的，其实是通过绑定插件中目标实现：

![1576046047173](maven.assets\1576046047173.png)

![1576046132521](maven.assets\1576046132521.png)

> 其他打包类型绑定关系参考官网



### 自定义绑定

我们可以自定义将插件的某一个目标绑定到我们制定的生命周期阶段中，如我们使用maven-source-plugin插件来创建项目源码jar包。以下实现org.apache.maven.plugins:jar-no-fork（创建源码jar包）绑定到default生命周期的verify阶段，执行mvn verify命令后，对应插件目标将执行

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>2.4</version>
                <!--执行任务-->
                <executions>
                    <execution>
                        <id>attach-sources</id>
                        <!--绑定的生命周期阶段-->
                        <phase>verify</phase>
                        <!--插件目标-->
                        <goals>
                            <goal>jar-no-fork</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```



## 插件配置

用户可以指定一些插件目标的参数

### 命令行配置

```shell
#跳过测试
mvn install-Dmaven.test.skip=true
```

### POM中插件全局配置

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

### POM中插件任务配置

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>2.4</version>
    <executions>
        <execution>
            <id>attach-sources</id>
            <phase>verify</phase>
            <goals>
            <goal>jar-no-fork</goal>
            </goals>
            <configuration>
				...
            </configuration>
        </execution>
    </executions>

</plugin>
```



## 查看插件描述

```shell
mvn help:describe-Dplugin=org.apache.maven.plugins:maven-compiler-plugin:2.4-Ddetail
#省去版本信息，让maven自动获取版本
mvn help:describe-Dplugin=org.apache.maven.plugins:maven-compiler-plugin
#使用插件前缀替换坐标
mvn help:describe-Dplugin=compiler
#加上goal参数，仅仅查看某个目标信息
mvn help:describe-Dplugin=compiler-Dgoal=compile

```



## 命令行调用插件

maven命令帮助：

```shell
C:\Users\zwz>mvn -h

usage: mvn [options] [<goal(s)>] [<phase(s)>]

Options:
```

- options：可选参数
- goal(s)：插件目标
- phase(s)：生命周期阶段

如：

```shell
mvn dependency:tree
mvn help:describe-Dplugin=compiler
```



## 插件仓库

![1576052337230](maven.assets\1576052337230.png)





# 聚合与继承

## 聚合（模块）

父子级别目录

```xml
    <modules>
        <module>lession1</module>
        <module>design-patterns-demo</module>
        <module>virtual-machine-demo</module>
        <module>common-interview</module>
        <module>concorrent-demo</module>
    </modules>
```

同级目录：

```xml
	<modules>
        <module>../lession1</module>
        <module>../design-patterns-demo</module>
        <module>../virtual-machine-demo</module>
        <module>../common-interview</module>
        <module>../concorrent-demo</module>
    </modules>
```



## 继承

值父pom的配置能够被子pom继承，常用dependencyManagent和pluginManagement

可继承的pom元素：

- groupId：项目组id，项目坐标的核心元素
- version：项目版本，项目坐标的核心元素
- description：项目的描述信息
- organization：项目的组织信息
- inceptionYear：项目的创建年份
- url：项目的URL地址
- developers：项目的贡献者信息
- distributionManagement：项目的部署信息
- issueManagement：项目的缺陷跟踪系统信息
- ciManagement：项目的持续集成系统信息
- scm：项目的版本控制系统信息
- mailingLists：项目的邮件列表信息
- properties：自定义的Maven属性
- dependencies：项目的依赖配置
- dependencyManagement：项目的依赖管理配置
- repositories：项目的仓库配置
- build：包括项目的源码目录配置、输出目录配置、测试目录配置、插件配置、插件管理配置等
- reporting：包括项目的报告输出目录配置、报告插件配置等



## 插件管理

类似于dependencyManagent，可以使用pluginManagement使插件能够被继承

父pom配置：

![1576119969675](maven.assets\1576119969675.png)

子pom配置：

![1576120008219](maven.assets\1576120008219.png)



## super pom

所有的pom都继承一个父pom，无论是否显式声明这个父pom，父pom也被称为super pom，它包含了一些可以被继承的默认配置，即配置了将会覆盖这些默认配置。。

在maven3中，其位置为M2_HOME/lib/maven-model-builder-3.6.0.jar/pom-4.0.4.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!-- START SNIPPET: superpom -->
<project>
  <modelVersion>4.0.0</modelVersion>

  <!--包依赖中央仓库-->
  <repositories>
    <repository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
    </repository>
  </repositories>

  <!--插件中央仓库-->
  <pluginRepositories>
    <pluginRepository>
      <id>central</id>
      <name>Central Repository</name>
      <url>https://repo.maven.apache.org/maven2</url>
      <layout>default</layout>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <releases>
        <updatePolicy>never</updatePolicy>
      </releases>
    </pluginRepository>
  </pluginRepositories>

 
  <build>
    <!--项目约定配置，如主代码位置、测试代码位置等-->
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    <!--插件版本-->
    <pluginManagement>
      <!-- NOTE: These plugins will be removed from future versions of the super POM -->
      <!-- They are kept for the moment as they are very unlikely to conflict with lifecycle mappings (MNG-4453) -->
      <plugins>
        <plugin>
          <artifactId>maven-antrun-plugin</artifactId>
          <version>1.3</version>
        </plugin>
        <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <version>2.2-beta-5</version>
        </plugin>
        <plugin>
          <artifactId>maven-dependency-plugin</artifactId>
          <version>2.8</version>
        </plugin>
        <plugin>
          <artifactId>maven-release-plugin</artifactId>
          <version>2.5.3</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>

  <reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
  </reporting>

  <profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
      <id>release-profile</id>

      <activation>
        <property>
          <name>performRelease</name>
          <value>true</value>
        </property>
      </activation>

      <build>
        <plugins>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-source-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-sources</id>
                <goals>
                  <goal>jar-no-fork</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-javadoc-plugin</artifactId>
            <executions>
              <execution>
                <id>attach-javadocs</id>
                <goals>
                  <goal>jar</goal>
                </goals>
              </execution>
            </executions>
          </plugin>
          <plugin>
            <inherited>true</inherited>
            <artifactId>maven-deploy-plugin</artifactId>
            <configuration>
              <updateReleaseInfo>true</updateReleaseInfo>
            </configuration>
          </plugin>
        </plugins>
      </build>
    </profile>
  </profiles>

</project>
<!-- END SNIPPET: superpom -->

```

如有一个项目pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>java-base-demo</artifactId>
        <groupId>com.personal</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>concorrent-demo</artifactId>
    <version>1.0-SNAPSHOT</version>


</project>
```

执行完`mvn help:effective-pom`后输出：





## Reator

Reator（反应堆）指所有模块组成的一个构建结构，对于单模块的项目，反应堆就是该模块本身，但对于多模块，反应堆就包含了模块之间继承与依赖的关系，从而自动计算出合理的模块构建顺序

如java-base-demo中有聚合配置：

```xml
    <modules>
        <module>lession1</module>
        <module>design-patterns-demo</module>
        <module>virtual-machine-demo</module>
        <module>common-interview</module>
        <module>concorrent-demo</module>
    </modules>
```

执行`mvn clean install`，可以看到反应堆构建顺序

```properties
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Build Order:
[INFO]
[INFO] java-base-demo                                                     [pom]
[INFO] lession1                                                           [jar]
[INFO] design-patterns-demo                                               [jar]
[INFO] virtual-machine-demo                                               [jar]
[INFO] common-interview                                                   [jar]
[INFO] concorrent-demo                                                    [jar]
```

> 构建规则中，按标记顺序构建，若构建的模块有其他依赖，会先构建依赖，比如顺序为xxx-main-module，xxx-main-denp-module，xxx-main-module依赖xxx-main-denp-module时，虽然xxx-main-module先定义，但会先构建xxx-main-denp-module



### 裁剪Reator

我们可以通过命令来改变项目反应堆的构建顺序，可通过mvn -h查看命令

- -am,--also-make同时构建所列模块的依赖模块

  - 如：`mvn clean install -pl lession1 -am`

    ```properties
    [INFO] ------------------------------------------------------------------------
    [INFO] Reactor Build Order:
    [INFO]
    [INFO] java-base-demo                                                     [pom]
    [INFO] lession1                                                           [jar]
    ```

- -amd,--also-make-dependents同时构建依赖于所列模块的模块

  - 如：`mvn clean install -pl java-base-demo -amd`

- -pl,--projects <arg>构建指定的模块，模块间用逗号分隔

  - 如：`mvn clean install -pl lession1,design-patterns-demo`

    ```properties
    [INFO] ------------------------------------------------------------------------
    [INFO] Reactor Build Order:
    [INFO]
    [INFO] lession1                                                           [jar]
    [INFO] design-patterns-demo                                               [jar]
    ```

- -rf,--resume-from <arg>从指定模块中开始构建反应堆

  - 如：`mvn clean install -rf common-interview`

    ```properties
    [INFO] ------------------------------------------------------------------------
    [INFO] Reactor Build Order:
    [INFO]
    [INFO] common-interview                                                   [jar]
    [INFO] concorrent-demo                                                    [jar]
    ```



# 使用nexus创建私服

maven私服软件有：Nexus、Archiva、Artifactory





# 使用maven进行测试







# 使用hadson进行持续集成

hodson是jetkin的前身





# 使用Maven构建web应用

一个典型war包目录结构：

![1576203448541](maven.assets\1576203448541.png)

一个典型的web项目的maven目录结构：

![1576203495693](maven.assets\1576203495693.png)

## 使用jetty-maven-piugins

使用jetty-maven-piugins可将更新的代码部署到Jetty容器中，其简单配置如下：

![1576207324742](maven.assets\1576207324742.png)

这里配置了每10s扫描项目的代码是否发生变化

项目访路径：http://localhost:port/test

只有org.apache.maven.plugins和org.codehaus.mojo两个groupId下的插件才可以通过前缀执行插件目标，为了能在命令行中执行`mvn jetty:run`，需在setting.xml中配置：

![1576207627709](maven.assets\1576207627709.png)

此时可以使用mvn jetty:run命令了

```shell
mvn jetty:run
#指定端口
mvn jetty:run-Djetty.port=9999
```



## 使用cargo实现自动化部署

### 部署到本地web容器

cargo支持两种本地部署方式

- standalone：cargo会从web容器的安装目录复制一份配置到用户指定的目录，然后在此基础上部署应用，每次重新构件时这个目录会被清空

  ![1576217254034](maven.assets\1576217254034.png)

  - container
    - containerId：容器类型
    - home：容器安装目录
  - configuration
    - type：部署模式
    - home：复制容器配置到什么位置，这里为构建输出目录，即target/tomcat6x目录

- existing：用户需要指定指定现有的web容器配置目录，然后cargo会直接使用这些配置并将应用部署到其对应的位置

  ![1576217278954](maven.assets\1576217278954.png)

  > configuration中的home对应现有web容器目录，部署后可以再webapps中找到部署的maven项目

启动cargo部署：mvn cargo:start（需在setting.xml的pluginGroup中配置）



### 部署到远程web容器

![1576217497253](maven.assets\1576217497253.png)

- container
  - type：remote表示远程部署，不显式指定，maven使用默认值installed
- configuration
  - type：runtime表示既不使用独立的容器配置，也不使用现有的容器配置，而是依赖于一个现有的容器

> 更多配置参考官网的cargo文档



# 版本管理

概念

- 版本管理：项目整体版本的演变过程管理
- 版本控制：借助版本控制工具追踪代码的每一次变更



## 何为版本管理

我们知道，项目版本一般分为发布版和快照版本，在开发过程中，开发人员一般都使用快照版本，而需要对外发布时，我们显然应该提供稳定的发布版本（能够定位到一致的组件）。版本管理关心的问题之一是快照版本和发布版本之间的转换

一般版本转换流程（一般在主干中体现）：

![1576225615002](maven.assets\1576225615002.png)

开始时快照版本，向外发布时转换为发布版，然后再转换为快照版本，再在此版本上进行开发

发布版本一般需满足的条件：

- 所有自动化测试应该全部通过
- 项目中没有配置任何快照版本的依赖：由于快照版本在不同时间构建可能会产生不一样的结果
- 项目中没有配置任何快照版本的插件：同理影响每次构建的结果
- 项目所包含的代码已经全部提交到版本控制系统中：意味着版本控制系统中丢失了当前发布版本的状态

打标签（tag）：指主干中某一个版本的代码以一个标签标识，之后可根据标签获得对应版本，如svn中打标签示例：

![1576225849781](maven.assets\1576225849781.png)



## maven版本号定义约定

看个实际的例子：1.3.4-beta-2

maven中版本号定义约定为：<主版本>.<次版本>.增量版本-<里程碑版本>

- 主版本：表示项目的重大架构变更。比如maven2和maven3，spring2和spring4等，版本之间相差甚远
- 次版本：表示较大范围的功能增加和变化，及Bug修复。如Nexus1.5较1.4增加了LDAP支持，但总体架构没发生多大变化
- 增量版本：一般表示重大bug的修复。如一个项目发布1.1.0后发现重大bug，应该修复后发布一个增量版本1.1.1
- 里程碑版本：往往指某一版本的里程碑。如maven3发布了很多里程碑版本，如maven3.0-alpha1、maven3.0-alpha2、maven3.0-beta-1等，它们相比于发布版，往往不是很稳定，还需要很多测试

注意：不是每个版本号都有这四个部分，但一般都有主版本和次版本。

对于主版本、次版本、增量版本来说，比较是基于数字的，如：

​		1.5>1.4>1.3.2>1.3

而对于里程碑版本，maven则进行简单的字符串比较，如：

​		1.2-beta-3>1.2-beta-13



## 主干、标签、分支

版本控制工具中一般存在几个概念：主干、标签、分支

- 主干：项目开发代码的主体，是从项目开始知道当前都处于活动状态，它包含了项目源代码所有的变更历史。如git中master
- 分支：从主干中某个点分离出来的代码拷贝。如git中拆出dev、sit版本，上线时再合并（merge）到主干
- 标签：用来标识主干或者分支某个点的状态，以代表项目的某个稳定状态

典型项目版本变化过程：

![1576228444315](maven.assets\1576228444315.png)



## 自动化版本发布

Maven Release Plugin插件提供了自动化发布版本的功能，它主要有是三个目标：

- release:prepare：准备发布版本

  ![1576457941574](maven.assets\1576457941574.png)

- release:rollback：将版本回退至release:prepare之前，但不会删除release:prepare生成的标签，需手动删除

- release:perform：执行版本发布。签出release:prepare生成的标签中的源代码，并在基础上执行mvn deploy命令打包并部署至仓库





# 灵活的构建

## maven属性

- 内置属性

  - ${basedir}：项目根目录，即包含pom文件的目录
  - ${version}：项目版本

- POM属性：可用该属性引用POM文件中对应元素的值

  - ${project.groupId}：项目的groupId
  - ${project.artifactId}：项目的artifactId
  - ${project.version}：项目的version，与version等价
  - ${project.build.sourceDirectory}：项目主代码目录，默认src/main/java
  - ${project.build.testSourceDirectory}：项目的测试代码目录，默认src/test/java
  - ${project.build.directory}：项目构建输出目录，默认为target/
  - ${project.outputDirectory}：项目主代码编译输出目录，默认target/classes/
  - ${project.testOutputDirectory}：项目测试代码编译输出目录，默认target/test-classes
  - ${project.build.finalName}：项目打包输出文件的名称，默认${project.artifactId}-${project.version}

  > 这些属性都对应了一个pom元素，它们中一些属性默认值都是在supe pom中定义的

- 自定义属性

  ```xml
  <project>
      ...
      <properties>
      	<my.prop>hello</my.prop>
      </properties>
      ...
  </project>
  ```

  - setting属性：类似POM属性，不过它是引用setting.xml文件中值。如${setting.localRepository}执行用户本地仓库地址

- Java系统属性：例如${user.home}指向了用户目录。mvn help:system查看所有java系统属性

- 环境变量属性：所有环境变量都可以以env.开头，如${env.JAVA_HOME}。mvn help:system查看所有环境变量



## 资源过滤

假如项目中有配置如下：

![1576463664025](maven.assets\1576463664025.png)

使用maven属性配置开发环境的值：

![1576463727526](maven.assets\1576463727526.png)

注意，maven属性只有在POM文件中才能解析，也就是${db.username}放到POM中会变成dev，但如果放到src/main/resources目录下的文件中，构建时它还是${db.username}。

maven-resources-plugin除了将项目主资源文件复制到主代码编译输出目录中、将测试资源文件复制到测试代码输出目录中外，还能用于解析资源文件中的maven属性，即开启**资源过滤**

原super pom文件中有如下定义：

```xml
<build>
	...
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
      </testResource>
    </testResources>
    ...
</build>
```

增加资源过滤：

```xml
<build>
	...
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
    </resources>
    <testResources>
      <testResource>
        <directory>${project.basedir}/src/test/resources</directory>
        <filtering>true</filtering>
      </testResource>
      
    </testResources>
    ...
</build>
```

也可配置多个目录：

```xml
<build>
	...
    <resources>
      <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>true</filtering>
      </resource>
      <resource>
        <directory>${project.basedir}/src/main/sql</directory>
        <filtering>false</filtering>
      </resource>
      
    </resources>
    ...
</build>
```

执行mvn clean install-Pdev，输出目录中的配置就是已经被解析的值了



## maven profile

![1576564883864](maven.assets\1576564883864.png)

### 激活profile

1. 命令行激活

   ```shell
   #激活了两个profile
   mvn clean install -Pdev-x,dev-y
   ```

2. setting文件显式激活

   ```xml
   <settings>
       ...
       <activeProfiles>
           <activeProfile>alwaysActiveProfile</activeProfile>
           <activeProfile>anotherAlwaysActiveProfile</activeProfile>
       </activeProfiles>
       ...
   </settings>
   ```

3. 系统属性激活

   比如存在以下配置：

   ![1576565337746](maven.assets\1576565337746.png)

   然后在命令行中激活：

   ```shell
   mvn clean install -Dtest=x
   ```

4. 操作系统环境激活

   ![1576565480390](maven.assets\1576565480390.png)
   family值包括Windows、UNIX、Mac等，其他几项可通过环境系统属性获得：os.name、os.arch、os.version

5. 文件存在与否激活：maven可根据项目中某个文件是否存在来决定是否激活profile：

   ![1576566707949](maven.assets\1576566707949.png)

6. 默认激活

   ![1576566793042](maven.assets\1576566793042.png)

   不过需注意的是，如果POM中有任何一个profile通过以上任意一个方式激活了，所有的默认激活配置都会失效。可通过命令查看当前激活的profile：

   ```shell
   mvn help:active-profiles
   mvn help:all-profiles
   ```



### profile种类

根据具体需要，可以在以下位置中声明profile：

- pom.xml：只对当前项目有效
- 用户settings.xml：当前用户下的maven项目有效
- 全局settings.xml：本机所有maven项目有效
- profiles.xml：在项目根目录下使用一个额外的profiles.xml文件来声明profile，maven3中已经废弃这种方式

POM中profile可使用的元素：

![1576568213863](maven.assets\1576568213863.png)

POM外部的profile（如settings.xml）可使用的元素：

![1576568304233](maven.assets\1576568304233.png)

POM外部的profile支持元素为什么这么少？

假如某个用户setting.xml中profile中存在依赖配置，那么这样其他用户并不能把这个依赖更新下来



## Web资源过滤

web项目中存在以下资源文件目录：

- src/main/resources/：经打包后位于应用程序的classpath中，即会位于war包的WEB-INF/classes目录下，maven-resources-plugin插件处理
- src/main/webapp/：web资源文件目录，经打包后位于war包的根目录。例如一个css目录src/main/webapp/css，打包后可在war包中的css目录中找到对应文件，maven-war-plugin插件处理

前面使用maven-resources-plugin配置了普通资源文件的过滤，同理可通过maven-war-plugin插件配置web资源文件过滤，比如：构建项目时为不同用户构建不一样的资源文件（例如客户logo不同、css主体不同）

![1576570392428](maven.assets\1576570392428.png)

![1576570407490](maven.assets\1576570407490.png)

通过mvn clean install -Pclient-a告诉web资源文件使用logo图片a.jpg，css主题使用red



## 在profile中激活集成测试

![1576571310252](maven.assets\1576571310252.png)

profile中配置执行TestNG测试组

![1576571368181](maven.assets\1576571368181.png)

也就是默认是执行单元测试的，可以通过激活profile full来执行集成测试



# 生成项目站点





# 编写Maven插件



# archetype