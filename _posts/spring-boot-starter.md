---
name: starter-customization
title: 自定义 starter
date: 2024-10-01
tags: 
- spring
- spring-boot
- 自动装配
categories: spring
---



<mark>在 Spring Boot 生态中，“Starter” 本质上就是一组依赖的“捆绑包”，它的目标是让使用方 **一行依赖** 就把启动一个完整的 Spring Boot 应用所需的所有东西都拉过来。</mark> 

<!-- more -->

# 父模块实现

父POM的作用：作为配置的样板、插件管理、依赖管理。

`parent` 定义了继承关系，子模块从父 POM 继承配置。

`modules` 定义了聚合关系，父 POM 聚合了需要统一管理的子模块。

在我们项目顶层的pom文件中，我们会看到dependencyManagement元素。通过它元素来管理jar包的版本，让子项目中引用一个依赖而不用显示的列出版本号。Maven会沿着父子层次向上走，直到找到一个拥有dependencyManagement元素的项目，然后它就会使用在这个dependencyManagement元素中指定的版本号。

Dependencies相对于dependencyManagement，所有声明在dependencies里的依赖都会自动引入，并默认被所有的子项目继承，但是父项目引入依赖没有任何用处，所以只需要用dependencyManagement声明即可。

在使用springboot时，通常工程有自己的父模块，而[不能继承spring-boot-starter-parent](https://docs.spring.io/spring-boot/docs/2.1.12.RELEASE/reference/html/using-boot-build-systems.html#using-boot-maven-without-a-parent)时，推荐按照下面的做法

`<type>pom</type>`，`<scope>import</scope>`，表示将`spring-boot-dependencies` 中`dependencyManagement`下的`dependencies`插入到当前工程的`dependencyManagement`中，所以不存在依赖传递。当没有`<scope>import</scope>`时，意思是将`spring-boot-dependencies` 的`dependencies`全部插入到当前工程的`dependencies`中，并且会依赖传递。

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <encoding>UTF-8</encoding>      
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <spring-boot.version>3.4.4</spring-boot.version>
    <rpc.version>1.0.0</rpc.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- 其他依赖 -->
    </dependencies>
</dependencyManagement>
```



```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${maven.compiler.source}</source>
                    <target>${maven.compiler.source}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <configuration>
                    <skipTests>false</skipTests>
                </configuration>
            </plugin>
            <!-- 其他插件 -->
        </plugins>
    </pluginManagement>
</build>
```

# starter 子模块实现

starter——核心功能在 core 实现，一定要有`autoconfigure`以及`configuration-processor`

- starter 依赖 core、<mark>spring-boot-starter</mark>（一行代码引入依赖的核心） `autoconfigure`其实也包括在内了
- 增强扩展性：core 的依赖全部改为 optional，然后在 starter 内再次依赖
- 名称：以spring-boot-starter结尾，易于辨识

# 使用 starter

直接在pom引入自定义 starter 即可

