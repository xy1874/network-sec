# 1.概述

https://blog.csdn.net/xxx_qz/article/details/72621025
https://blog.csdn.net/qq_51927659/article/details/122773139

跨站点脚本（XSS）是通常在 Web 应用程序中发现的一种计算机安全漏洞。此漏洞可使攻击者将恶意代码（例如JavaScript）插入受害者的Web浏览器。使用这种恶意代码，攻击者可以窃取受害者的凭据，如 Cookie 。可以通过利用XSS漏洞来绕过浏览器用于保护这些凭据的访问控制策略（即，同源策略）。这类漏洞已经被利用来制作强大的网络钓鱼攻击和浏览器攻击。

本次实验在预制的 Ubuntu VM 映像中设置了名为 Elgg 的 Web 应用程序。Elgg 是社交网络非常受欢迎的开源 Web 应用程序，它本身实施了一些对抗措施来弥补 XSS 的威胁。本次实验为了演示 XSS 攻击如何工作，在 Elgg 的安装中取消了这些对抗措施，故意使 Elgg 容易遭受 XSS 攻击。没有这些对抗措施，用户可以将任意消息（包括 JavaScript 程序）发布到 user profiles 。在这个实验中，学生们需要利用这个漏洞，对修改后的 Elgg 进行 XSS 攻击，其方式与 Samy Kamkar 在2005年通过臭名昭著的 Samy 蠕虫对 MySpace 做了类似的处理。这种攻击的最终目标是在用户之间传播 XSS 蠕虫，以便任何查看受感染用户个人资料的用户被感染，被感染的人会将你（即攻击者）添加到他/她的朋友列表中。


**实验目的**

```
1. 掌握Cross-Site Scripting attack 的基本原理
2. 应用XSS worm以及了解 self-propagation
3. 掌握Session cookies的具体使用技巧
4. 熟练使用HTTP GET and POST requests


```

# 2.实验环境

本次实验用到一个 web 应用程序，我们使用 Docker 来设置这个 web 应用程序。在实验中设置有两个 Docker ，一个用于托管 web 应用程序，另一个用于托管 web 应用程序的数据库。web 应用容器的IP地址为 10.9.0.5, web 应用程序的 URL 如下

```
http://www.seed-server.com
```

我们需要将主机名映射到 Docker 的 IP地址。请将以下条目添加到`/etc/hosts`文件。您需要使用根权限来更改此文件(使用`sudo`)。值得注意的是,由于其他一些实验可能重名如果它被映射到不同的IP地址， 旧的必须删除。

修改文件的命令可以用 vi , 打开文件后可以用i插入语句，修改完成后先按 esc 键，然后 :wq!  表示保存修改退出。  

```
sudo vi /etc/hosts
```

待添加的映射语句

```
10.9.0.5 www.seed-server.com
```

## 2.1 容器安装和容器常用命令

请从SEED网站下载 [Labsetup.zip](https://seedsecuritylabs.org/Labs_20.04/Web/Web_SQL_Injection/) 文件到你的虚拟机，解压缩，进入`Labsetup`文件夹，使用`docker-compose.yml`文件来设置实验环境。这个文件的内容和所有涉及的`Dockerfile`的详细说明可以在 [用户手册](https://github.com/seed-labs/seed-labs/blob/master/manuals/docker/SEEDManual-Container.md) 中找到，用户手册链接到本实验的网站。如果这是您第一次使用容器设置SEED实验室环境，那么阅读用户手册是非常重要的。

下面，我们将列出一些与 Docker 和 Compose 相关的常用命令。因为我们后面也将非常频繁地使用这些命令，所以我们在 .bashrc文件中为它们创建了别名。

```
//完整的docker命令
$ docker-compose build # 创建容器
$ docker-compose up # 启动容器
$ docker-compose down # 关闭容器
// 对应简写的docker命令，如下命令只能在seed环境中使用，因为该环境对命令做了映射
$ dcbuild # 创建容器
$ dcup # 启动容器
$ dcdown # 关闭容器
```

所有容器都将在后台运行。要在容器上运行命令，我们通常需要在该容器上获取shell。我们首先需要使用`“docker ps”`命令找出容器的ID，然后使用`“docker exec”`在该容器上启动一个shell。我们在`.bashrc`文件中为它们创建了别名。

```
$ dockps // docker ps 别名
$ docksh <id> //docker exec -it <id> /bin/bash 别名
// 具体使用，举例说明：
$ dockps
b1004832e275 hostA-10.9.0.5
0af4ea7a3e2e hostB-10.9.0.6
9652715c8e0a hostC-10.9.0.7
$ docksh 96
root@9652715c8e0a:/#
```

**MySQL数据库**。容器通常是一次性的，因此一旦被破坏，容器内的所有数据都丢失了。对于这个实验，我们希望将数据保存在MySQL数据库中，这样当我们关闭容器时就不会丢失我们的工作数据。为了实现这一点，我们已经在主机上挂载了 mysql_data 数据文件夹(在Labsetup中，它将在MySQL容器运行后创建)到MySQL容器中的`/var/lib/mysql`文件夹。这个文件夹是MySQL存储数据库的地方。

因此，即使容器被破坏，数据库中的数据仍然被保留。如果要从干净的数据库开始，可以删除此目录，删除命令如下。

```
 sudo rm -rf mysql_data
```

## 2.2 web应用程序

我们已经创建了一个 web 应用程序，这是一个简单的员工管理应用程序。员工可以通过这个 web 应用程序查看和更新数据库中的个人信息。这个 web 应用程序主要有两个角色:管理员是一个特权角色，可以管理每个员工的个人档案信息;员工是正常角色，可以查看或更新自己的个人信息。所有员工信息如 表1 所示

<center><img src="../assets/image-20230409151703983.png" width = 600></center>