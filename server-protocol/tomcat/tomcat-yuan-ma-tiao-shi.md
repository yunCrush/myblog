---
description: 源码构建篇
---

# tomcat源码构建

1. 解压压缩包，进入文件夹内创建source文件夹，conf, webapps 移动到source目录下
2. 根目录下创建pom.xml,如下

{% code title="pom.xml" overflow="wrap" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>apache-tomcat-8.5.50-src</artifactId>
    <name>Tomcat8.5</name>
    <version>8.5</version>
    <build>
        <!--指定源目录-->
        <finalName>Tomcat8.5</finalName>
        <sourceDirectory>java</sourceDirectory>
        <resources>
            <resource>
                <directory>java</directory>
            </resource>
        </resources>
        <plugins>
            <!--引入编译插件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <!--tomcat 依赖的基础包-->
    <dependencies>
        <dependency>
            <groupId>org.easymock</groupId>
            <artifactId>easymock</artifactId>
            <version>3.4</version>
        </dependency>
        <dependency>
            <groupId>ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.7.0</version>
        </dependency>
        <dependency>
            <groupId>wsdl4j</groupId>
            <artifactId>wsdl4j</artifactId>
            <version>1.6.2</version>
        </dependency>
        <dependency>
            <groupId>javax.xml</groupId>
            <artifactId>jaxrpc</artifactId>
            <version>1.1</version>
        </dependency>
        <dependency>
            <groupId>org.eclipse.jdt.core.compiler</groupId>
            <artifactId>ecj</artifactId>
            <version>4.5.1</version>
        </dependency>
        <dependency>
            <groupId>javax.xml.soap</groupId>
            <artifactId>javax.xml.soap-api</artifactId>
            <version>1.4.0</version>
        </dependency>
    </dependencies>
</project>
```
{% endcode %}

3\. 使用idea打开文件，找到Bootstrap.java main方法，run, 按照提示，进行修复。

4\. 配置配置文件：edit configuration -> VM options

{% code overflow="wrap" %}
```editorconfig
-Dcatalina.home=D:\github\apache-tomcat-8.5.50-src\source
-Dcatalina.base=D:\github\apache-tomcat-8.5.50-src\source
-Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
-Djava.util.logging.config.file=D:\github\apache-tomcat-8.5.50-src\source\conf\logging.properties
```
{% endcode %}

5.启动项目......页面访问: [http://localhost:8080/](http://localhost:8080/)

6\. 报错解决方案： 原因是Jsp引擎Jasper没有被初始化，从而无法编译JSP，我们需要在tomcat的源码ContextConfig类 的configureStart方法中增加一行代码将 Jsp 引擎初始化,webConfig()方法下面(777)。

{% code overflow="wrap" %}
```
// 初始话JSP解析引擎
jaspercontext.addServletContainerInitializer(new JasperInitializer(),null);
```
{% endcode %}

\`
