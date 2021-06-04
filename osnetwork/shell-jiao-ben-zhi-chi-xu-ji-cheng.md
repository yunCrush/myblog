---
description: 这里介绍关于Jenkins持续集成，以及自动化部署。
---

# shell脚本之持续集成

　　这篇文章用来记录使用Jenkins打RPM包以及自动化部署时，需要注意的点。

## 1.配置　　

　　Jenkins项目配置页面，**Pre Steps--&gt;**Build --&gt; Root POM: pom.xml   Goals and options: clean package

**Post Steps**: Run only if build succeeds

　　添加Sonar监控时选择：Execute SonarQube Scanner,并在pom.xml文件中配置jacoco的相关配置。

Jenkins调用远程服务器：

```text
# 使用scp将rpm传送到远程服务器指定地址，再ssh连接服务器
ssh -tt root@IP << eeooff
......执行远程服务器的安装脚本，实现自动化安装
exit
eeooff
```

## 2.脚本

　　首先需要编写4个shell脚本，环境变量的配置文件：evn.conf  服务管理的service文件project-name.service、打包流程的executor.sh ,SPEC文件：project-name.spec。

　　首先看一下文件结构：我们在Jenkins\_workspace目录下创建rpmBuild：SOURCES（资源文件夹，config，lib，bin）、SPECS（\*.spec文件）、BUILD、BUILDROOT、RPMS（RPM包）、SRPMS

**executor.sh逻辑：**　　

1. 我们需要将资源文件复制到SOURCES下，spec文件移动到SPECS下。
2. 如果独立用户，需要给rpmBuild赋予属组。
3. 调用spec文件开始打包（**%\_topdir来指定spec文件位置**）

```text
cd ${rpmBuild_path}
rpmbuild --target=noarch -D "%_topdir ${rpmBuild_path}" -bb SPEC/project-name.spec
```

**project-name.spec逻辑：**

　　spec文件可以理解为，相当于在用户下创建了一个独立的临时空间。**对于某些国产机无法给脚本赋予执行权限的情况，可以在spec文件中对脚本进行赋予权限。**将project-name.service文件复制到临时空间下的lib/systemd/system下。

service文件即centos7通过systemctl star/stop/status来启动停止项目的配置文件

```text

```

