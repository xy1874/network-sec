# 实验步骤

## 0 环境准备

请从SEED网站下载 [Labsetup.zip](https://seedsecuritylabs.org/Labs_20.04/Web/Web_SQL_Injection/) 或者指导书位置下载[Lab3-XSS-Labsetup.zip] (https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt) 文件到你的虚拟机，或者QQ群里的实验3压缩文件，解压缩，进入`Labsetup`文件夹，使用`docker-compose.yml`文件来设置实验环境。


**主机名映射**

本次实验用到一个web应用程序，我们使用Docker来设置这个web应用程序。在实验设置中有两个Docker，一个用于托管web应用程序，另一个用于托管web应用程序的数据库。web应用容器的IP地址为10.9.0.5,web应用程序的URL如下

```
http://www.seed-server.com
```

我们需要将主机名映射到 Docker 的 IP地址。请将以下条目添加到`/etc/hosts`文件。您需要使用根权限来更改此文件(使用`sudo`)。

修改文件的命令可以用 vi , 打开文件后可以用i插入语句，修改完成后先按 esc 键，然后 :wq!  表示保存修改退出。  

```
sudo vi /etc/hosts
```

如果文件中包含如下内容，可以不做修改，否则，你需要把以下命令添加到hosts文件中，保存退出

```
10.9.0.5 www.seed-server.com   // 如果打开的文件不是这个web地址，请修改
10.9.0.5 www.example32a.com
10.9.0.5 www.example32b.com
10.9.0.5 www.example32c.com
10.9.0.5 www.example60.com
10.9.0.5 www.example70.com
```

 **创建容器**

 在 /etc/docker/daemon.json 文件中添加 docker 国内镜像源（目的是速度更快），然后使用命令 dcbuild 创建容器

```
{
        "registry-mirrors": [
                "https://hub-mirror.c.163.com",
                "https://mirror.baidubce.com",
                "https://docker.mirrors.sjtug.sjtu.edu.cn"
        ]       
} 
```
创建容器

```
$ dcbuild
```

**启动容器**

容器创建成功后，才能启动容器

```
$ dcup
```

**查看容器信息**

本次实验中用到的两个容器一个是web应用程序，一个是MySQL数据库

```
$ dockps
20c3333a8cd5  www-10.9.0.5
4a101760d086  mysql-10.9.0.6
```

## 1 任务1.1：熟悉MySQL语句

本任务的目标是通过使用提供的数据库来熟悉SQL命令。web 应用程序使用的数据存储在 MySQL 数据库中，该数据库托管在 MySQL 容器上。

我们已经创建了一个名为`sqllab_users`的数据库，其中包含一个名为`credential`的表。该表存储每个员工的个人信息(例如`eid`、密码、工资、ssn等)。在这个任务中，你需要使用数据库来熟悉SQL查询。

请在 MySQL 容器上获取一个 shell (参见容器手册中的说明)。

```
$ docksh 4a   // 这里的 4a 是取上面 dockps 命令中的 mysql 容器的前两个字母
```
然后使用`mysql`客户端程序与数据库进行交互。用户名为`root`，密码为`dees`。

```
// Inside the MySQL container
# mysql -u root -pdees
```

登录后，可以创建新的数据库，也可以加载已有的数据库。由于我们已经为你创建了`sqllab_users`数据库，你只需要使用`use`命令加载这个现有的数据库。要显示`sqllab_users`数据库中有哪些表，可以使用`show tables`命令打印所选数据库的所有表。

```
mysql> use sqllab_users;
Database changed
mysql> show tables;
+------------------------+
| Tables_in_sqllab_users |
+------------------------+
| credential             |
+------------------------+
```

**任务1.1** 运行上述命令后，需要使用SQL命令打印员工Alice的所有概要信息。请提供你的结果截图。

## 1 任务1.2：sqlmap工具的使用

sqlmap是一个使用python语言开发的开源的渗透测试工具，可以用来进行自动化检测，利用 SQL 注入漏洞，获取数据库服务器的权限。它具有功能强大的检测引擎，可以针对各种不同类型数据库的进行渗透测试，包括获取数据库中存储的数据，访问操作系统文件等。

实验室环境中已经安装，如果自行安装，请用下面的命令安装

```
apt install sqlmap
```

安装好后，执行下面的命令可以帮助你了解 sqlmap 的相关命令

```
sqlmap -h  或者 sqlmap --h
```

下面是本次实验用到的sqlmap命令，通过这几条命令可以将我们搭建的web网站服务器的数据库和数据表信息找出来，并能够检测到web的注入点：

```
//检测某个url存在的漏洞，url必须得有接收参数的地方（例如：username=），否则的话就不存在漏洞。
//可以看到很多提示信息，黄色[WARNING]表示不存在注入，加粗的绿色[INFO]信息大家关注是可能存在问题的点
sqlmap -u "http://www.seed-server.com/unsafe_home.php?username=admin&Password=" 

//读取某个url用到的数据库系统中存在的数据库名称
sqlmap -u "http://www.seed-server.com/unsafe_home.php?username=admin&Password=" --dbs

//读取某个数据库中的所有表
sqlmap -u "http://www.seed-server.com/unsafe_home.php?username=admin&Password=" -D 数据库名 --tables

//将某个表中的所有信息dump下来
sqlmap -u "http://www.seed-server.com/unsafe_home.php?username=admin&Password=" -D 数据库名 -T 数据表名 --dump
```

**任务1.2** 运行上述命令后，可以将credential中的所有信息dump下来。请提供你的结果截图，并根据第一条命令找出可以注入的信息点。

## 2 任务2：SELECT 语句下的注入攻击

SQL注入基本上是一种技术，通过这种技术，攻击者可以执行他们自己的恶意SQL语句，通常称为`payload`。通过恶意SQL语句，攻击者可以从受害数据库中窃取信息;更糟糕的是，他们可能会对数据库进行更改。我们的员工管理web应用程序有SQL注入漏洞，这里模仿了开发人员经常犯的错误。

我们将使用www.seed-server.com的登录页面来完成这项任务。登录界面如图1所示。它要求用户提供用户名和密码。web应用程序基于这两段数据验证用户身份，因此只有知道密码的员工才能登录。作为攻击者，你的工作是在不知道任何员工的凭证的情况下登录到web应用程序。

<center><img src="../assets/image-20230409152607019.png" width = 600></center>

为了帮助你完成这项任务，我们将解释如何在web应用程序中实现身份验证。

不安全的PHP代码`home.php`位于`/var/www/SQL_Injection`目录下，用于执行用户身份验证。下面的代码片段显示了如何验证用户。

```php
$input_uname = $_GET[’username’];
$input_pwd = $_GET[’Password’];
$hashed_pwd = sha1($input_pwd);
...
$sql = "SELECT id, name, eid, salary, birth, ssn, address, email,
			nickname, Password
		FROM credential
		WHERE name= ’$input_uname’ and Password=’$hashed_pwd’";
$result = $conn -> query($sql);
// The following is Pseudo Code
if(id != NULL) {
	if(name==’admin’) {
		return All employees information;
	} else if (name !=NULL){
		return employee information;
	}
} else {
Authentication Fails;
}

```

上面的SQL语句从凭据表中选择员工个人信息，如id、姓名、工资、ssn等。SQL语句使用输入uname和散列pwd两个变量，其中输入uname保存用户在登录页面的用户名字段中键入的字符串，而散列pwd保存用户键入的密码的sha1散列。程序检查是否有记录与提供的用户名和密码匹配;如果匹配，则成功验证用户，并提供相应的员工信息。如果没有匹配项，则认证失败。

!!! info "提示 :sparkles:"
    注释符参考：#, --+, --空格，/*…*/

**任务2.1**:网页SQL注入攻击。你的任务是作为管理员从登录页面登录到web应用程序，这样你就可以看到所有员工的信息。我们假设你知道管理员的帐户名admin，但不知道密码。你需要决定在“用户名”和“密码”字段中键入什么才能成功进行攻击。

**任务2.2**:命令行SQL注入攻击。你的任务是重复任务2.1，但你需要在不使用网页的情况下完成。你可以使用命令行工具，例如curl，它们可以发送HTTP请求。值得一提的是，如果你想在HTTP请求中包含多个参数，你需要将URL和参数放在一对单引号之间;否则，用于分隔参数的特殊字符(如&)将被shell程序解释，从而改变命令的含义。下面的例子展示了如何发送一个`HTTP GET`请求

```
$ curl www.seed-server.com/unsafe_home.php?username=alice&Password=11
```

如果需要在`username或password中包含特殊字符`，则需要对它们进行适当的编码，否则它们可能会改变请求的含义。如果你想在这些字段中包含单引号，你应该使用%27代替;如果你想包含空白，你应该使用%20。在这个任务中，你需要在使用curl发送请求时处理HTTP编码。(url编码)

!!! info "提示 :sparkles:"
    转义符号：
	单引号号（'）:%27
	空格：20%
	井号（#）:23%

**任务2.3**:追加一条新的SQL语句。在以上两种攻击中，我们只能从数据库中窃取信息。如果我们可以在登录页面中使用相同的漏洞修改数据库就更好了。

一种想法是使用SQL注入攻击将一条SQL语句转换为两条SQL语句，第二条SQL语句是更新或删除语句。在SQL中，分号(;)用于分隔两条SQL语句。请尝试通过登录页面运行两条SQL语句,并说明是否能够获取到信息。

## 3 任务3：UPDATE 语句下的注入攻击

如果`UPDATE`语句出现SQL注入漏洞，则损害会更严重，因为攻击者可以利用该漏洞修改数据库。在我们的`Employee Management`应用程序中，有一个`Edit Profile`页面(图2)，允许员工更新他们的概要信息，包括昵称、电子邮件、地址、电话号码和密码。员工需要先登录才能进入该页面。

当员工通过Edit Profile页面更新他们的信息时，下面的SQL语句会更新。查询将被执行。在`unsafe_edit_backend.php`文件中实现的PHP代码用于更新员工的个人资料信息。PHP文件位于/`var/www/SQLInjection`目录中。

```
$hashed_pwd = sha1($input_pwd);
    $sql = "UPDATE credential SET
    nickname=’$input_nickname’,
    email=’$input_email’,
    address=’$input_address’,
    Password=’$hashed_pwd’,
    PhoneNumber=’$input_phonenumber’
    WHERE ID=$id;";
$conn->query($sql);
```

<center><img src="../assets/image-20230409153952703.png" width = 600></center>

**任务3.1**:修改自己的工资。如“编辑个人资料”页面所示，员工只能更新自己的昵称、电子邮件、地址、电话号码和密码;他们无权改变工资。假设你(Alice)是一名心怀不满的员工，你的老板Boby今年没有给你加薪。通过编辑个人资料页面中的SQL注入漏洞来增加自己的薪水。请演示一下你是如何做到的。我们假设你知道工资存储在名为salary的列中。

<center><img src="../assets/image-20230409154132011.png" width = 600></center>


**任务3.2**:修改其他人的工资。在给自己加薪之后，你决定惩罚一下你的老板。你想把他的工资减到1美元。请演示一下你是如何做到的。

**任务3.3**:修改他人密码。在修改了老板的薪水后，你仍然很不满，所以你想把老板的密码改成你知道的密码，然后你就可以登录他的账户，进行进一步的破坏。请演示一下你是如何做到的。你需要证明你可以使用新密码成功登录到老板的帐户。这里值得一提的一点是，数据库存储的是密码的哈希值，而不是明文密码字符串。你可以再次查看不安全的`unsafe_edit_backend.php`代码，以查看密码是如何存储的，它是使用SHA1这个hash函数产生密码的hash值。

!!! info "提示 :sparkles:"
    在线获取SHA1结果可以参考 http://www.ttmd5.com/hash.php?type=5 。

### 4 任务4:对策—预处理

SQL注入漏洞的根本问题是无法将代码与数据分离。在构造SQL语句时，程序(如PHP程序)知道哪一部分是数据，哪一部分是代码。不幸的是，当SQL语句被发送到数据库时，数据和代码的边界消失了;SQL解释器看到的边界可能与开发人员设置的原始边界不同。要解决这个问题，重要的是要确保服务器端代码和数据库中的边界视图是一致的。最安全的方法是使用预处理语句。

为了理解预处理语句是如何防止SQL注入的，我们需要理解当SQL服务器接收到一个查询时会发生什么。查询如何执行的高级工作流显示在图3。在编译步骤中，查询首先经过解析和规范化阶段，其中根据语法和语义检查查询。下一个阶段是编译阶段，其中关键字(例如;SELECT, FROM, UPDATE等)被转换成机器可以理解的格式。

基本上，在这个阶段，查询是解释的。在查询优化阶段，考虑执行查询不同方案的数量，从中选择最优的优化方案。所选择的方案存储在缓存中，因此无论何时进入下一个查询，都将根据缓存中的内容进行检查，如果它已经存在在缓存中，解析，编译和查询优化阶段将被跳过。然后将编译后的查询传递到实际执行阶段。

预处理语句出现在编译之后、执行步骤之前。准备好的语句将经过编译步骤，并转换为预编译的查询，其中数据占位符为空。要运行这个预编译的查询，需要提供数据，但这些数据不会经过编译步骤;相反，它们被直接插入预编译的查询，并被发送到执行引擎。因此，即使数据中有SQL代码，如果不经过编译步骤，代码将被简单地视为数据的一部分。这就是预处理语句防止SQL注入攻击的方法。

<center><img src="../assets/image-20230409154132011.png" width = 600></center>



下面是一个如何用PHP编写预处理语句的示例。我们在下面的例子中使用SELECT语句。我们将展示如何使用预处理语句重写易受SQL注入攻击的代码。

```sql
$sql = "SELECT name, local, gender
		FROM USER_TABLE
		WHERE id = $id AND password =’$pwd’ ";
$result = $conn->query($sql)
```

上述代码容易受到SQL注入攻击。可以改写为

```sql
$stmt = $conn->prepare("SELECT name, local, gender
			FROM USER_TABLE
			WHERE id = ? and password = ? ");
// Bind parameters to the query
$stmt->bind_param("is", $id, $pwd);
$stmt->execute();
$stmt->bind_result($bind_name, $bind_local, $bind_gender);
$stmt->fetch();

```

使用准备语句机制，我们将向数据库发送SQL语句的过程分为两个步骤。第一步是只发送代码部分，即没有实际数据的SQL语句。这是准备步骤。从上面的代码片段中可以看到，实际数据被问号(?)取代。在这一步之后，我们使用`bind_param()`将数据发送到数据库。数据库将在此步骤中仅将发送的所有内容视为数据，而不再将其视为代码。它将数据绑定到预处理语句的对应问号。在`bind_param()`方法中，第一个参数“is”表示参数类型:“i”表示id中的数据为整数类型，“s”表示pwd中的数据为字符串类型。

**任务4**：在这个任务中，我们将使用预处理语句机制来修复SQL注入漏洞。

为了简单起见，我们在防御文件夹中创建了一个简化的程序。我们将对这个文件夹中的文件进行更改。如果你将浏览器指向以下URL，你将看到一个类似于web应用程序的登录页面的页面。通过该页面可以查询员工信息，但需要提供正确的用户名和密码。

```
URL: http://www.seed-server.com/defense/
```

在这个页面中输入的数据将被发送到服务器程序`getinfo.php`，该程序调用一个名为`unsafe.php`的程序。这个PHP程序中的SQL查询容易受到SQL注入攻击。你的任务是使用准备好的语句修改`unsafe.php`中的SQL查询，这样程序就可以击败SQL注入攻击。在Labsetup文件夹中，`unsafe.php`程序在`image_www/Code/defense`文件夹中。你可以直接在那里修改程序。在你完成之后，你需要重建并重新启动容器，否则更改将不生效。

你还可以在容器运行时修改该文件。在运行的容器中，`unsafe.php`程序位于`/var/www/ sql_injection /defense`内部。这种方法的缺点是，为了保持docker较小，我们只在容器中安装了一个非常简单的文本编辑器`nano`。它应该足以进行简单的编辑。如果你不喜欢这个编辑器，你可以随时使用`“apt install”`在容器中安装你最喜欢的命令行编辑器。例如，对于喜欢vim的人，你可以执行以下操作:

```
# apt install -y vim
```

!!! info "提示 :sparkles:"
    有余力的同学可以完善下unsafe_home.php、unsafe_edit_frontend.php和unsafe_edit_backend.php代码文件，把相关的漏洞修复。



# 4. Guidelines

测试SQL注入字符串。在实际应用程序中，可能很难检查SQL注入攻击是否包含语法错误，因为服务器通常不会返回这类错误消息。为了进行调查，可以将SQL语句从php源代码复制到MySQL控制台。假设你有以下SQL语句，并且注入字符串是 `' or 1=1;# `  。

```
SELECT * from credential
WHERE name=’$name’ and password=’$pwd’;
```

可以用注入字符串替换$name的值，并使用MySQL控制台测试它。这种方法可以帮助你在发起真正的攻击之前构建一个没有语法错误的注入字符串。

