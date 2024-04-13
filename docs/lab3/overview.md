# 1.概述

跨站脚本（XSS）是一种常见的Web应用程序漏洞。这种漏洞使得攻击者能够将恶意代码（例如JavaScript程序）注入到受害者的Web浏览器中。利用这些恶意代码，攻击者可以窃取受害者的凭证，例如会话cookie。浏览器用于保护这些凭证的访问控制策略（即同源策略）可以被利用XSS漏洞绕过。跨站脚本攻击利用的也是数据信息和代码没有做好隔离的漏洞，主要有三种方法：反射型 XSS 、 存储型 XSS 和 DOM-based 型 XSS。本次实验主要用到存储型 XSS。

为了展示攻击者如何利用XSS漏洞进行攻击，我们在预构建的Ubuntu虚拟机映像中设置了一个名为Elgg的Web应用程序。Elgg是一个非常受欢迎的开源社交网络Web应用程序，它已经实施了一系列反制措施来应对XSS威胁。为了演示XSS攻击的工作原理，我们在安装过程中有意取消了这些反制措施，使Elgg容易受到XSS攻击。在没有这些反制措施的情况下，用户可以发布任何消息，包括JavaScript程序，到用户个人资料中。

## 1.1 反射型 XSS 攻击

从用户接收输入的信息，执行一些操作后返回给用户时携带用户信息这种行为称之为反射行为。

攻击者可以在用户的输入信息中混入 JavaScript 代码，这样当输入内容被反射回浏览器的时候， JavaScript 的代码就被注入到该网页当中了。

!!! Warning "注意 :sparkles:"
    攻击代码的输入必须是从目标用户的计算机发出的。

## 1.2 存储型 XSS 攻击

攻击者把数据发给目标服务器网站，网站对数据进行持久性存储，之后，如果网站将存储数据发送给其他用户，那么就在攻击者和其他用户之间创建了一条通道。这一类型的攻击常常存在于社交网站的主页，用户评论等。

## 1.3 DOM-base XSS 攻击

DOM 型 XSS 攻击是一种利用 DOM 基于 HTML 解析过程中的安全漏洞进行的跨站攻击。攻击完全基于客户端的机制，攻击者通过篡改网页中的 DOM 元素和属性，注入恶意代码进而达到攻击目的。

## 1.4 XSS 造成的危害

**1、污染网页** 比如将新闻篡改成假新闻。
**2、欺骗请求** 比如偷发添加好友的请求。
**3、窃取信息** 包括网页中的cookie、显示的个人信息等


**实验目的**

```
1. 掌握 Cross-Site Scripting attack 的基本原理
2. 应用 XSS worm 以及了解 self-propagation
3. 掌握 Session cookies 的具体使用技巧
4. 熟练使用 HTTP GET and POST requests
5. 熟悉 CSP 安全政策防止XSS攻击


```

# 2.实验环境

本次实验在预制的 Ubuntu VM 映像中设置了名为 Elgg 的 Web 应用程序。Elgg 是社交网络非常受欢迎的开源 Web 应用程序，它本身实施了一些对抗措施来弥补 XSS 的威胁。本次实验为了演示 XSS 攻击如何工作，在 Elgg 的安装中取消了这些对抗措施，故意使 Elgg 容易遭受 XSS 攻击。没有这些对抗措施，用户可以将任意消息（包括 JavaScript 程序）发布到 user profiles 。在这个实验中，学生需要利用这个漏洞对修改后的 Elgg 发动 XSS 攻击，方式与 2005 年萨米·卡姆卡尔（Samy Kamkar）对 MySpace 发起的臭名昭著的 Samy 蠕虫攻击相似。这次攻击的最终目标是在用户之间传播一个 XSS 蠕虫，使得任何查看被感染用户个人资料的人都会被感染，而被感染的人则会将你（即攻击者）添加为联系人。



## 2.1 容器安装和容器常用命令

请从SEED网站下载 [Labsetup.zip](https://seedsecuritylabs.org/Labs_20.04/Web/Web_XSS_Elgg/) 文件到你的虚拟机，解压缩，进入`Labsetup`文件夹，使用`docker-compose.yml`文件来设置实验环境。这个文件的内容和所有涉及的`Dockerfile`的详细说明可以在 [用户手册](https://github.com/seed-labs/seed-labs/blob/master/manuals/docker/SEEDManual-Container.md) 中找到，用户手册链接到本实验的网站。如果这是您第一次使用容器设置SEED实验室环境，那么阅读用户手册是非常重要的。

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

我们已经创建了一个 web 应用程序，这是一个简单的社交网站。用户可以通过它修改个人信息、添加好友、发博客等。这个 web 应用程序有两类角色:管理员是一个特权角色，可以管理每个人信息;用户是正常角色，可以查看或更新自己的个人信息、添加好友、浏览其他人的主页、发博客等。如下图所示：

<center><img src="../assets/2.png" width = 600></center>

本次实验用到的用户信息,如下表所示：
