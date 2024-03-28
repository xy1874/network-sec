# DVWA 概述

DVWA全称为Damn Vulnerable Web Application，意为存在糟糕漏洞的web应用。它是yikuan 开源的渗透测试漏洞练习平台，其中内含 SQL 注入、XSS、文件上传、文件包含、CSRF和暴力破解等各个难度的测试环境。

搭建后本环境不仅可以扩展学习 SQL 注入的内容，在下次实验的 XSS 攻击实验基础上也可在该环境中扩展练习。

## docker 搭建 DVWA 靶场环境

这里提供的搭建步骤是在 Ubuntu 20.04 的SEED环境中搭建，本虚拟环境中已经按照了docker，如果您想自行搭建，请确保已经安装了docker。

**下载docker镜像** 首先检查下docker是否安装，然后再拉取 DVWA 的docker镜像，完成后如下图所示。

```
docker --version  //查看是否安装了docker
docker pull vulnerables/web-dvwa  //下载 DVWA 的 docker镜像
```

<center><img src="../assets/extra-1.png" width = 400></center>


**启动容器**

容器启动时做端口映射，这里将 DVWA 容器的端口直接隐身虚拟机的 80 端口，启动后如下图所示。

```
docker run --rm -it -p 80:80 vulnerables/web-dvwa
```

<center><img src="../assets/extra-2.png" width = 600></center>

查看容器信息

```
docker ps
```
<center><img src="../assets/extra-3.png" width = 300></center>

**登录DVWA**

查看虚拟机ip

```
ip addr
```

然后根据查到的ip地址，http://192.168.56.101/login.php 登录DVWA，因为我们端口直接映射的是80端口，所以不需在IP地址后加端口号，如果是其他端口号则需要带上端口号访问。

<center><img src="../assets/extra-4.png" width = 300></center>

用用户名密码（admin/password）登录后重置数据库

<center><img src="../assets/extra-8.png" width = 600></center>

再次登录后就可以看到 DVWA 各种漏洞攻击的页面了。

<center><img src="../assets/extra-5.png" width = 600></center>

可以根据需要调整攻击的级别，初学者选择low，后续可以级别逐步提升。

<center><img src="../assets/extra-6.png" width = 600></center>

级别更改提交后，对应的漏洞注入点也会相应变化。

<center><img src="../assets/extra-7.png" width = 600></center>

## DWVA 简单说明

**漏洞的类别**

**暴力破解**：由于服务器端没有做限制，导致攻击者可以通过暴力的手段破解所需信息，DVWA 中用来测试密码暴力破解工具，展示弱口令的不安全性。
**命令执行**：应用程序有时需要调用一些执行系统命令的函数，比如 system、exec、 shell_exec等函数可以执行系统命令，如果攻击者能够控制这些函数中的参数，就可以将恶意的系统命令拼接到正常命令中，从而造成命令执行攻击，比如 DVWA 中提供的 ping 功能，就可以在后面添加其他命令执行 。
**伪造跨站请求**：简称 CSRF ，是一种对网站的恶意利用，攻击者利用目标用户的身份，以目标用户的名义执行某些非法操作。比如 DVWA 中可以更改应用的管理员密码。
**文件包含**：允许“攻击者”将远程/本地文件包含到Web应用程序中。
**文件上传**：如果服务器代码未对客户端上传的用户文件进行严格的验证和过滤，就容易造成可以上传任意文件的情况，攻击者将恶意文件（比如可执行的恶意脚本）绕过检查上传到 web 服务器。
**SQL注入**：启动一个“攻击者”将SQL语句注入到HTTP表单的输入框中，DVWA包括基于盲（盲注）和错误的SQL注入。
**跨站脚本（XSS）**：是一种针对网站应用程序的安全漏洞攻击技术，是代码注入的一种。它允许恶意用户将代码注入网页，其他用户在浏览网页时就会收到影响。包含反射型、存储型和 DOM 型 XSS 。



**漏洞的安全等级**

漏洞的安全等级分为低级、中级和高级。每个级别都会改变 DVWA 中所有漏洞的利用难度，初始加载时默认所有漏洞安全等级为低级，可以根据需要更改漏洞等级，等级越高漏洞越不容易攻破。

低级 - 此级别是完全易受攻击的，并且没有做任何安全防护。它的作用是展示不良编程实践是如何导致 Web 应用程序漏洞的，并用作教授或学习漏洞基本利用技术的平台。
中级 - 此级别主要是为用户提供糟糕的编程实践示例。开发者对程序的安全性进行了一定的保护但是仍有可被利用的漏洞，这为改进用户漏洞利用技巧提供了挑战。
高级 - 此级别旨在为用户提供良好编码实践的示例。这个级别应该保护所有漏洞的安全，它用于将易受攻击的源代码与安全源代码进行比较。




