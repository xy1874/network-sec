# 1.概述

SQL注入（SQL injection）是一种利用Web应用程序和数据库服务器之间接口漏洞的代码注入技术。当用户的输入在被发送到后端数据库服务器之前没有在Web应用程序中得到正确检查时，就会存在这种漏洞。

许多Web应用程序从用户那里获取输入，然后使用这些输入构建SQL查询，以便从数据库中获取信息。Web应用程序还使用SQL查询在数据库中存储信息。这些都是Web应用程序开发中的常见做法。当SQL查询没有仔细构建时，就可能发生SQL注入漏洞。SQL注入是对Web应用程序最常见的攻击之一。

在这个实验中，我们创建了一个容易受到SQL注入攻击的Web应用程序。我们的Web应用程序包括许多Web开发人员所犯的常见错误。目标是找到利用SQL注入漏洞的方法，演示攻击可能造成的损害，并掌握可以帮助抵御这种类型攻击的技术。

本次实验主要包括了以下三个主题

```
• SQL语句：SELECT和UPDATE语句 
• SQL注入 
• 预处理语句
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