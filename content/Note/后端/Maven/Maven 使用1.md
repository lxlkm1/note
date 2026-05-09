
>[!note] 概述
>
>Maven 是一个java 项目管理工具，类似于node js 的npm


# 1 配置Maven

>[!Warning] JDK 25 不兼容 maven 3.5
>


1 下载最新版maven

![[Pasted image 20260509150151.png|505]]

2 解压放置D盘

![[Pasted image 20260509150313.png|510]]

3 配置环境变量

win + s 搜索环境变量

![[Pasted image 20260509150509.png]]


![[Pasted image 20260509150617.png|350]]

4 测试是否安装

![[Pasted image 20260509151837.png]]

5 配置本地仓库和网络仓库

>[!note] 简介
>
>本地仓库就是放置一些项目开发所需要的jar包的地方，cz，jar包可以给项目提供别人写好的代码。本地仓库的jar包一般从网络仓库获取

修改settings.xml 去配置本地仓库和网络仓库

![[Pasted image 20260509152251.png|520]]

配置本地仓库

![[Pasted image 20260509153357.png|526]]

配置网络仓库

>[!Warning] 配置阿里云的源不能开启VPN

>[!Warning] 即使开启梯子，使用官方的源也会有网络问题

![[Pasted image 20260509153453.png|533]]

# 2 Maven的结构

bin 目录放置一些maven 需要的指令
boot 
conf 目录就是放置maven 配置的，我们只需要关注里面的settings.xml
lib 放置maven 项目启动需要的jar包，maven本身是java开发的

![[Pasted image 20260509154130.png]]

# 3 创建一个Maven 项目

1 标准maven项目的结构(可以手动创建，也可以使用idea创建)

main 是主程序和资源存放的目录
test 是测试程序和资源存放的目录

src 外面还有一个pom.xml文件，主要是管理依赖的

![[Pasted image 20260509160049.png|520]]

标准pom 文件

项目信息
groupId 是项目创建者的编号，一般是倒过来的域名
artifactId 是项目的名字
version 是项目的版本
scoop 表示依赖在哪生效

junit-jupiter 是maven 测试主代码需要的依赖

>[!Warning] junit 4 和 5的导入方式存在差异

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"

           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0

           https://maven.apache.org/xsd/maven-4.0.0.xsd">

  

      <modelVersion>4.0.0</modelVersion>

      <groupId>com.xq</groupId>

      <artifactId>demo</artifactId>

      <version>1.0-SNAPSHOT</version>

      <dependencies>

  <dependency>

      <groupId>org.junit.jupiter</groupId>

      <artifactId>junit-jupiter</artifactId>

      <version>5.10.2</version>

      <scope>test</scope>

  </dependency>

      </dependencies>

  </project>
```


主程序

![[Pasted image 20260509190516.png|513]]

测试程序

>[!warning] Test 在 junit 的导入方式改成import org.junit.jupiter.api.Test;

![[Pasted image 20260509190546.png|516]]



# 4 使用Mevan 指令


1 编译（把java 代码编译成class， 输出targe文件夹）
```powershell

mvn compile

```
2 测试 （在编译基础上执行一遍测试代码)
```powershell

mvn test

```
3 打包 （在测试基础上把程序变成jar或者war包)
```powershell

mvn package

```
4 安装 (在打包基础上把程序安装到本地仓库)
```powershell

mvn install

```


# 5 使用骨架构建Maven 项目

1 在idea 配置maven（可选择)

![[Pasted image 20260509202636.png|697]]

2 在idea 配置maven 指令
![[Pasted image 20260509202816.png|620]]

![[Pasted image 20260509202844.png|490]]


![[Pasted image 20260509202918.png|448]]


3 构建maven项目

>[!note] 简介
>Archetype 就是Maven 项目的模板,idea 内置了许多maven 项目的模板

![[Pasted image 20260509191740.png|497]]

quick start 适合大多数maven 项目， 而webapp 适用于javaweb 应用开发

# 6 Maven 依赖管理

1 查看当前项目依赖

![[Pasted image 20260509203446.png|697]]

>[!note] 简介
>依赖项就是该项目需要的的jar包，这种叫做直接依赖，而依赖项有时也需要依赖其他的jar包，其他jar包产生的依赖叫做间接依赖，如果该项目需要两个不同版本的依赖，这两个依赖就算是产生了冲突，图中灰色的依赖项表示该依赖和其他依赖产生了冲突

2 默认的依赖冲突处理办法

情况一  直接依赖和直接依赖的冲突

后面写入的依赖覆盖前面的，以最后的依赖为标准

情况二  直接依赖和间接依赖的冲突

直接依赖直接覆盖后面的

情况三  间接依赖和间接依赖的冲突

前面的间接依赖覆盖后面的


3 手动处理冲突

最佳实践

可以在pom.xml  使用dependencyManagement限制所以依赖的版本，依赖写依赖的地方就不需要再指定版本了

![[Pasted image 20260509210452.png]]


# 7 依赖范围

>[!note] 简介
>所谓的依赖范围指的是依赖在哪个部分的代码起作用，由依赖的scope 标签定义


![[Pasted image 20260509212803.png]]

test scope 代码在项目main 文件夹下的代码无法被使用

provided scope 代码不参与代码，比如servlet-api ,因为这个依赖里面有服务器的一些配置，而javawb 项目的tomcat 里面也有服务器相关的配置，如果这个参与打包会让服务器产生冲突

runtime scope 如jdbc，操作数据库的，我们不需要去适应jdbc的代码，我们只需要根据jdbc接口写好数据库操作就好，等到带包时，就带上jdbc代码，把数据库操作给jdbc代码