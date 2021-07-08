[toc]
# 1.基础概念

### 1.1 maven的作用

①  管理jar

- i. 增加第三方jar
- ii. jar包之间的依赖关系【自动关联下载所有依赖的jar，且不会冲突】

② 将项目拆分成若干个模块

### 1.2 maven的概念

maven是一个基于Java平台的自动化构建工具【make-ant-maven-gradle】，将原材料（java、js、css、html、图片）变成产品（可发布项目）

- 清理：删除编译的结果，为重新编译做准备
- 编译：java->class 
- 测试：针对于项目中的关键点进行测试，亦可用项目中的测试代码去测试开发代码
- 报告：将测试的结果进行显示
- 打包：将项目中包含的多个文件压缩成一个文件，用于安装或部署【java项目-jar，web项目-war】
- 安装：将打成的包放到本地仓库，供其他项目使用
- 部署：将打成的包放到服务器上准备运行
  - 将java、js、jsp等各个文件进行筛选、组装，变成一个可以直接运行的项目

> 打包方式：java工程--jar、web项目--war、父工程--pom

> 通常下载一个jar，先在本地仓库中下载；如果本地仓库中不存在，则再联网到中央仓库（或中央仓库镜像）去下载

> Eclipse中的项目，在部署时会生成一个对应的部署项目（在wtpwebapps中），区别在于：部署项目没有源码文件src（java），只有编译后的class文件和jsp文件；
eclipse中的项目复制到tomcat/webapps中不能直接运行，因为二者目录结构不一致（若要在tomcat中运行一个项目，则该项目必须严格遵守tomcat的目录结构）

- eclipse中的项目要在tomcat中运行，就需要部署： 
  - a. 通过eclipse中Add and Remove按钮进行部署
  - b. 将web项目打成一个war包，然后将该war包复制到tomcat/webapps中即可运行

### 1.3 下载配置maven

- 配置JAVA_HOME
- 配置MAVEN_HOME(或M2_HOME)【D:\download\Maven\apache-maven-3.6.3】
- 配置path【D:\download\Maven\apache-maven-3.6.3\bin】
- 验证：mvn -v
- 配置本地仓库（maven目录/conf/settings.xml）
  - 默认本地仓库
  - 修改本地仓库【<localRepository>D:/download/Maven/mvnrep</localRepository>】
- eclipse配置
  - window->preference->maven->installations\user settings

# 2. 使用maven

### 2.1 maven约定的目录结构

项目
- src
  - main【程序功能代码】
    - java【java代码】
    - resources【资源代码、配置代码】
  - test【测试代码】
    - java
    - resource
- pom.xml【项目对象模型】

```
<groupId>org.lanqiao.maven</groupId>   //域名反转.大项目名
<artifactId>HelloWorld</artifactId>   //子模块名
<version>0.0.1-SNAPHOT</version>   //版本号
```

```
assertEquals("Hello zs",result);      //断言
```



### 2.2 maven常见命令

> 运行mvn命令，必须在pom.xml目录

第一次执行命令时，因为需要下载执行该命令的基础环境，所以会从远程仓库下载环境（Maven基础组件，基础jar）
- mvn compile: 编译，只编译main目录中的java文件
- mvn test: 测试
- mvn package: 打成jar/war
- mvn install: 将开发的模块放入本地仓库，供其他模块使用【放入的位置是通过gav决定】
- mvn clean: 删除target目录（删除编译文件的目录）


### 2.3 依赖

- 依赖：
  - A中的某些类需要使用B中的某些类，则称为A依赖于B
  - 在maven项目中，如果要使用一个当时存在的jar或模块，则可以通过依赖实现【去本地仓库、远程仓库找】

##### 2.3.1 依赖有效性

> Maven在编译、测试、运行项目时，各自使用一套classpath

> 依赖的有效性：compile（默认）、test、provided

依赖有效性 | compile | test | provided
---|---|---| ---
编译（main） | 1 |  | 1
测试（test） | 1 | 1 | 1
部署（运行） | 1 |  | 


##### 2.3.2 依赖排除

```
<exclusions>
    <exclusion>
        <groupId>org.springframework</groupId>
        <artifactId>spring-beans</artifactId>
    </exclusion>
</exclusions>
```
##### 2.3.3 依赖的传递性

- A.jar-B.jar-C.jar
- 当且仅当B.jar依赖于C.jar的范围是compile

##### 2.3.3 依赖原则

> 为了防止冲突

- ① 路径最短优先原则
- ② 路径长度相同：
  - i. 在同一个pom.xml文件中有2个相同的依赖：在后面声明的依赖会覆盖前面声明的依赖（严禁-在同一pom.xml中声明版本不同的两个依赖）
  - ii. 如果是不同的pom.xml中有2个相同的依赖，则先声明的依赖会覆盖后声明的依赖


### 2.4 在eclipse中创建maven工程

- 配置maven：
  - 配置maven版本
  - 配置本地仓库

在eclipse中编写完pom.xml依赖后，需要maven-update project

### 2.5 maven生命周期

- 生命周期和构建的关系：
  - 生命周期中的顺序：a b c d e
  - 执行c命令时，实际执行的是a b c
  
- 生命周期包含的3个阶段：
  - clean lifecycle(清理):
    - pre-clean  clean  post-clean
  - default lifecycle（默认）【常用】
  - site lifecycle（站点）:
    - pre-site  site  post-site site=deploy


### 2.6 统一项目的jdk版本

- a、 build path：删除旧版本，增加新版本
- b、 右键项目-属性-Project Factors-java version 改版本（存在要改的版本）

实用maven：
-  在pom.xml中配置<profiles>

```
<profiles>
    <profile>
        <id>jdk-18</id>
        <activation>
            <activationByDefault>true</activationByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
    </profiles>
</profiles>
```

### 2.7 统一编码

```
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <!-- 统一版本 -->
    <lanqiao.World.junit.version>4.0</lanqiao.World.junit.version>
    ......
</properties>
```


# 3. 继承

A继承B，则A可以使用B所有的依赖，可以通过父工程统一管理依赖的版本

- i. 建立父工程【父工程的打包方式为pom】
- ii. 在父工程的pom.xml中编写依赖<dependencyManagement>
- iii. 子类，给当前工程继承一个父工程<parent>
  - ① 加入父工程坐标gav
  - ② 当前工程的Pom.xml到父工程的Pom.xml之间的路径<relativePath>
- iiii. 在子类中需要声明：使用父类的哪些依赖【ga】

# 4. 聚合

> Maven项目能够识别的：自身包含、本地仓库

> Maven2依赖于Maven1，则在执行时，必须先将Maven1加入到本地仓库（install）

- 以上前置工程的install操作可以交由“聚合”一次搞定【<modules><module>项目的根路径(可以不按顺序)</>】
- 在一个总工程中配置聚合【聚合的配置只能在打包方式为pom的Maven工程中】
- 配置完聚合后，只要操作总工程，则会自动操作该聚合中配置过的工程
- Maven将一个大工程拆分成若干个子工程（子模块），聚合可以将拆分的多个子工程合起来

> 注意：clean命令是删除target目录，并不说清理install存放入的本地仓库


# 5. 部署web工程

- a、打包【war】手动放到tomcat运行

- b、通过maven直接部署运行web项目
  - ① 配置cargo
  - ② maven命令：deploy

> 实际开发中，开发人员将自己的项目开发完毕后，打成war包交给实施人员去部署