---
title: 项目管理和构建——Maven
date: 2017-03-14 22:23:52
tags: maven
description: 从简介、安装配置、eclipse搭建、Nexus详细介绍等四个方面来简单了解Maven
---
# 简介
先看一下[Apache官网](http://maven.apache.org)的解释：
>Apache Maven is a software project management and comprehension tool. Based on the concept of a project object model (POM), Maven can manage a project's build, reporting and documentation from a central piece of information.  

翻译：**Maven**是基于项目对象模型(**POM**即Project Object Model)，可以通过一小段描述信息来管理项目的构建，报告和文档的软件项目管理工具。
## 下载
下载地址：http://maven.apache.org/release-notes-all.html ，现在Maven的最新版本是Maven3.3.9
## 什么是Maven
>Maven, a Yiddish word meaning accumulator of knowledge, was originally started as an attempt to simplify the build processes in the Jakarta Turbine project. There were several projects each with their own Ant build files that were all slightly different and JARs were checked into CVS. We wanted a standard way to build the projects, a clear definition of what the project consisted of, an easy way to publish project information and a way to share JARs across several projects.    
The result is a tool that can now be used for building and managing any Java-based project. We hope that we have created something that will make the day-to-day work of Java developers easier and generally help with the comprehension of any Java-based project.  

翻译：**Maven**这个单词来自于意第绪语，意为知识的积累，最早在Jakata Turbine项目中它开始被用来试图简化构建过程。当时有很多项目，它们的Ant build文件仅有细微的差别，而JAR文件都由CVS来维护。于是Maven创始者想要更加标准的方式构建项目，该项目的清晰定义包括：*一种很方便的方式来发布项目信息，以及一种在多个项目中共享JAR的方式*。
     现在，**Maven**成为了一种被用于构建和管理任何基于Java项目的工具。**Maven**创始者希望能够更多的让Java开发人员的日常工作更加容易，帮助理解任何基于Java项目。
## Maven的目标
>Maven’s primary goal is to allow a developer to comprehend the complete state of a development effort in the shortest period of time. In order to attain this goal there are several areas of concern that Maven attempts to deal with:    
1、Making the build process easy  
2、Providing a uniform build system  
3、Providing quality project information  
4、Providing guidelines for best practices development  
5、Allowing transparent migration to new features  

翻译：**Maven**的主要目标是为了使开发人员在最短的时间内领会项目的所有状态。为了达到这一目标，**Maven**考虑一下*五个方面*的内容：
1、使得构建过程更加容易，方便编译，打包，发布
2、为每个项目提供统一的配置
3、提供优质项目信息
4、最佳开发实践
5、安装和更新第三插件透明化

目前Apache下绝大多数项目都已经采用Maven进行管理。而Maven本身还支持多种插件，可以方便更灵活的控制项目。
# 安装
安装maven超级简单，总共分四步：

1. 下载 Maven ，其实就是一个压缩包，解压一下
![install_maven1](/maven/install_maven1.jpeg)
2. 配置一下环境变量,有两个环境变量可以配置：

    * MAVEN_HOME = D:\maven\apache-maven-3.2.3
    * MAVEN_OPTS = -Xms128m -Xmx512m(可选)

3. 在path变量末尾加入“%MAVEN_HOME%\bin;”,以上M2_HOME 是必须要配置的，如果想让 Maven 跑得更快点，可以根据自己的情况来设置 MAVEN_OPTS。
![install_maven2](/maven/install_maven2.jpeg)
4. 最后，验证是否安装成功
现在我们打开 cmd，输入：
mvn -v
我想您一定会看到一些信息，如下图所示：  
![install_maven3](/maven/install_maven3.jpeg)
恭喜您**Maven**安装成功！
在使用**Maven**之前，我们必须要了解一下 Maven 到底是怎样管理 jar 包的，这就是 Maven 仓库要干的活了。
## 修改本地仓库的地址
使用**Maven**给我们带来的最直接的好处，就是统一管理jar包，那么这些jar包存放在哪里呢？它们就在您的本地仓库中，默认地址位于*C:\Users\用户名.m2*目录下（当然也可以修改这个默认地址），下面我们就修改一下这个默认地址。
* 本地仓库可以理解为“缓存”，目的是存放jar包。开发项目时项目首先会从本地仓库中获取jar 包，当无法获取指定jar包的时候，本地仓库会从 远程仓库（或中央仓库）中下载jar包，并“缓存”到本地仓库中以备将来使用。
* 远程仓库（中央仓库）是**Maven**官方提供的，可通过 http://search.maven.org/ 来访问。这样一来，本地仓库会随着项目的积累越来越大。通过下面这张图可以清晰地表达项目、本地仓库、远程仓库之间的关系。
![repository](/maven/repository.png)
既然**Maven**安装了，那么本地仓库也就有了，默认路径在我们C盘目录下，对于专业人士来说C盘很危险，下面我们修改一下默认配置。

Maven会将下载的类库（jar包）放置到本地的一个目录下，如果想重新定义这个目录的位置就需要修改Maven本地仓库的配置：
修改文件：*D:\maven\apache-maven-3.2.3\conf\setting.xml*
```html
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
              xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <!-- localRepository
       | The path to the local repository maven will use to store artifacts.
       |
       | Default: ${user.home}/.m2/repository
      <localRepository>/path/to/local/repo</localRepository>
      -->
        <localRepository>D:\maven\repository</localRepository>
</settings>
```
依据该配置，Maven就会将下载的类库保存到D:\maven\repository中。
实验一下我们刚才做的事情产生作用没有，控制台输入：
*mvn help:system*
如图所示效果： 
![repository_change](/maven/repository_change.jpeg)
如果没有任何问题，执行完该命令之后，在*D:\maven\repository*下面就会多出很多文件，这些文件就是maven从中央仓库下载到本地仓库的文件，maven已经开始为我们工作了。
# Eclipse配置maven+创建maven项目
## Eclipse配置maven
检查eclipse的maven插件是否安装成功，如图： 
![eclipse_maven](/maven/eclipse_maven.jpeg)
若没有安装maven插件，我们需要先安装maven插件。若没有安装，请自行百度😄
**配置Maven**

1. 配置maven安装目录 
依次打开Window –> Perferences –> Maven ，展开Maven的配置界面，如下图：
![config1](/maven/config1.jpeg)
然后点击Installations –> add 选择maven安装目录，这里我的Maven安装目录为**D:\maven\apache-maven-3.2.3**，选择你的Maven安装目录，并点击确定, 之后可以点击Apply,点击OK，即可完成 
![config2](/maven/config2.jpeg)
2. 在Maven的配置界面，设置**User Settings** 
Global Settings选择maven 安装目录下conf文件夹下的settings.xml，这里我的Maven安装目录为**D:\maven\apache-maven-3.2.3\conf\settings.xml**，选择你的Maven安装目录，检查Local Repository 项，如果为**D:/maven/repository**则配置成功，否则重新配置上一步。 
![config3](/maven/config3.jpeg)
恭喜你，现在我们已经配置好了eclipse，下面，我们可以创建maven项目了。
## 创建maven项目
1. 我们在Eclipse菜单栏中点击**File->New->Other->Maven**,在弹出的对话框中会看到，如下图所示：
![create1](/maven/create1.jpeg)
2. 选择Maven Project，请选中Create a simple project(skip archetype selection),之后点击Next 
![create2](/maven/create2.jpeg)
3. 填写Group id和Artifact id， Version默认，Packaging默认为jar,Name，Description选填，其他的不填 
![create3](/maven/create3.jpeg)
之后点击Finish即可，如图所示：
![complete1](/maven/complete1.jpeg)
4. 前三步就可以创建一个简单的maven项目，如果我们想创建一个Maven的web项目，把第三步的Packaging的类型改为war，之后点击Finish即可，如图所示： 
![complete2](/maven/complete2.jpeg)

恭喜你**Maven**项目也创建完成了 
# Nexus的详细介绍以及安装
## 简介
**Nexus**是Maven仓库管理器，也可以叫Maven的私服。**Nexus**是一个强大的Maven仓库管理器，它极大地简化了自己内部仓库的维护和外部仓库的访问。利用**Nexus**你可以只在一个地方就能够完全控制访问和部署在你所维护仓库中的每个Artifact。**Nexus**是一套“开箱即用”的系统不需要数据库，它使用文件系统加Lucene来组织数据。
Nexus不是Maven的核心概念，它仅仅是一种衍生出来的特殊的Maven仓库。对于Maven来说，仓库只有两种：*本地仓库和远程仓库*。
![nexus_structure](/maven/nexus_structure.jpeg)
私服是架设在局域网的一种特殊的远程仓库，目的是代理远程仓库及部署第三方构件。有了私服之后，当 Maven 需要下载构件时，直接请求私服，私服上存在则下载到本地仓库；否则，私服请求外部的远程仓库，将构件下载到私服，再提供给本地仓库下载。
![nexus](/maven/nexus.jpeg)
## 为什么使用Nexus
1. **节省外网带宽**
<br>大量对于外部仓库的重复请求会消耗带宽，利用私服代理外部仓库，可以消除对外的重复构件下载，降低带宽的压力。
2. **加速Maven构建**
<br>不停地连接请求外部仓库十分的耗时，Maven在执行构建的时候不停地检查远程仓库的数据。利用私服，Maven只检查局域网的数据，提高构建的速度。
3. **部署第三方构件**
<br>当某个构件无法从任何一个外部远程仓库获得。建立私服之后，便可以将这些构件部署到私服，供内部的Maven项目使用。
4. **提高稳定性，增强控制**
<br>Maven构建高度依赖于远程仓库，因此，当网络不稳定的时候，Maven构建也会变得不稳定，甚至无法构建。私服缓存了大量构建，即使暂时没有网络，Maven也可以正常的运行。
5. **降低中央仓库的负荷**
<br>使用私服可以避免很多对中央仓库的重复下载，降低中央仓库的压力。

## 安装Nexus
Nexus专业版是需要付费的，我们使用的开源版Nexus OSS。Nexus提供了两种安装方式，一种是内嵌Jetty的bundle，只要你有JRE就能直接运行。第二种方式是不包含容器的WAR包，你只须简单的将其发布到web容器中即可使用。
### Nexus下载
下载地址：http://www.sonatype.org/nexus/go ，下载最新版本的Nexus，我使用的是nexus-2.14.3-bundle。（即下载2.x版本，下载3.x版本文件内部结构不同于2.x）

### Bundle方式安装
1. 将nexus-2.14.3-bundle.zip解压至任意目录，如：*D:\tools\maven*。这是会得到如下两个目录：
![bundle1](/maven/bundle1.jpeg)
nexus-2.14.3-02:该目录包含了Nexus运行所需要的文件，如启动脚本、依赖jar包等。
    打开目录\nexus-2.14.3-02\bin\jsw这个目录下面你会发现有很多系统版本的nexus环境，如下图：
![bundle2](/maven/bundle2.jpeg)
我的电脑是windows的系统，我打开一个文件夹，文件夹包含是nexus的命令，如下图：
![bundle3](/maven/bundle3.jpeg)
**sonatype-work**：该目录包含Nexus生成的配置文件、日志文件、仓库文件。该目录不是必须得，Nexus会在运行的时候动态的创建，不再过多的介绍。

2.  为方便启动和退出Nexus，将bin添加到环境变量。
![bundle4](/maven/bundle4.jpeg)
3. 使用命令nexus install将nexus安装到windows的服务中。(**注意:**此处启动控制台需要使用管理员方式)
![bundle5](/maven/bundle5.jpeg)
Nexus启动成功了，然后打开浏览器，访问http://localhost:8081/nexus ，你会看到如下的页面：
![bundle6](/maven/bundle6.jpeg)
要停止Nexus，Ctrl+C即可，也可以使用stop命令。
点击右上角 Log In，使用用户名：admin ，密码：admin123 登录，可使用更多功能：
![bundle7](/maven/bundle7.png)

4. Nexus预置的仓库
点击左侧 Repositories 链接，查看 Nexus 内置的仓库：
![bundle8](/maven/bundle8.png)
**Nexus**的仓库分为这么几类：

* *hosted 宿主仓库*：主要用于部署无法从公共仓库获取的构件（如 oracle 的 JDBC 驱动）以及自己或第三方的项目构件；
* *proxy 代理仓库*：代理公共的远程仓库；
* *virtual 虚拟仓库*：用于适配 Maven 1；
* *group 仓库组*：Nexus 通过仓库组的概念统一管理多个仓库，这样我们在项目中直接请求仓库组即可请求到仓库组管理的多个仓库。

![bundle9](/maven/bundle9.png)
5. 添加代理仓库
以 Sonatype 为例，添加一个代理仓库，用于代理 Sonatype 的公共远程仓库。点击菜单 Add - Proxy Repository ：
![bundle10](/maven/bundle10.png)
填写Repository ID - sonatype；Repository Name - Sonatype Repository；
Remote Storage Location - http://repository.sonatype.org/content/groups/public/ ，save 保存：
![bundle11](/maven/bundle11.png)
将添加的 Sonatype 代理仓库加入 Public Repositories 仓库组。选中 Public Repositories，在 Configuration 选项卡中，将 Sonatype Repository 从右侧 Available Repositories 移到左侧 Ordered Group Repositories，save 保存：

![bundle12](/maven/bundle12.png)
6. 搜索构件
为了更好的使用 Nexus 的搜索，我们可以设置所有 proxy 仓库的 Download Remote Indexes 为 true，即允许下载远程仓库索引。
![bundle13](/maven/bundle13.png)
索引下载成功之后，在 Browse Index 选项卡下，可以浏览到所有已被索引的构件信息，包括坐标、格式、Maven 依赖的 xml 代码：
![bundle14](/maven/bundle14.png)
有了索引，我们就可以搜索了：

![bundle15](/maven/bundle15.png)
7. 配置Maven使用私服
私服搭建成功，我们就可以配置 Maven 使用私服，以后下载构件、部署构件，都通过私服来管理。
在 settings.xml 文件中，为所有仓库配置一个镜像仓库，镜像仓库的地址即私服的地址（这儿我们使用私服公共仓库组 Public Repositories 的地址）:
![bundle16](/maven/bundle16.png)
```html
<mirrors>
            <mirror>
                <id>central</id>
                <mirrorOf>*</mirrorOf> <!-- * 表示让所有仓库使用该镜像--> 
                <name>central-mirror</name> 
                <url>http://localhost:8081/nexus/content/groups/public/</url>
            </mirror> 
</mirrors>
    ```
### WAR方式安装
需要有一个能运行的webapp的容器，这里以Tomcat为例，加入Tomcat的安装目录位于D:\tools\apache-tomcat-6.0.18 ，首先我们将下载的nexus-webapp-1.3.0.war 重命名为nexus.war ，然后复制到D:\tools\apache-tomcat-6.0.18\webapps\nexus.war ，然后启动CMD，cd到D:\tools\apache-tomcat-6.0.18\bin\ 目录，运行startup.bat 。一切OK，现在可以打开浏览器访问http://127.0.0.1:8080/nexus ，你会得到和上图一样的界面。

备注：本人安装过程中出现无法访问现象如下图：
![war](/maven/war.jpg)
解决办法：http://www.myexception.cn/open-source/1299355.html


