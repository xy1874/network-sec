# 1.Overview

SQL注入是一种代码注入技术，利用Web应用程序和数据库服务器之间的接口中的漏洞。当用户的输入在发送到后端数据库服务器之前未在Web应用程序中正确检查时则可能存在漏洞。

许多Web应用程序从用户那里获取输入，然后使用这些输入构造SQL查询，以便可以从数据库中获取信息。Web应用程序还使用SQL查询将信息存储在数据库中。这些是Web应用程序开发中的常见做法。当SQL查询未经仔细构造时，可能会出现SQL注入漏洞。 SQL注入是Web应用程序上最常见的攻击之一。

在这个实验中，我们创建了一个易受SQL注入攻击的Web应用程序。我们的Web应用程序包括许多Web开发人员犯的常见错误。我们的目标是找到利用SQL注入漏洞的方法，理解攻击可能造成的损害，并掌握可以帮助防御此类攻击的技术。

本次实验主要包括了以下三个主题

```
• SQL statements: SELECT and UPDATE statements
• SQL injection
• Prepared statement
```

# 2.Lab Environment

我们为这个实验开发了一个web应用程序，我们使用Docker来设置这个web应用程序。在实验设置中有两个Docker，一个用于托管web应用程序，另一个用于托管web应用程序的数据库。web应用容器的IP地址为10.9.0.5,web应用程序的URL如下

```
http://www.seed-server.com
```

我们需要将主机名映射到Docker的IP地址。请将以下条目添加到`/etc/hosts`文件。您需要使用根权限来更改此文件(使用`sudo`)。值得注意的是,由于其他一些实验可能重名如果它被映射到不同的IP地址，旧的必须删除。

```
10.9.0.5 www.seed-server.com
```

## 2.1 Container Setup and Commands

请从实验室网站下载`Labsetup.zip`文件到你的虚拟机，解压缩，进入`Labsetup`文件夹，使用`docker-compose.Yml`文件来设置实验环境。这个文件的内容和所有涉及的`Dockerfile`的详细说明可以在用户手册中找到，用户手册链接到本实验的网站。如果这是您第一次使用容器设置SEED实验室环境，那么阅读用户手册是非常重要的。

下面，我们将列出一些与Docker和Compose相关的常用命令。因为我们将非常频繁地使用这些命令，所以我们在.bashrc文件中为它们创建了别名。

```
$ docker-compose build # 创建容器
$ docker-compose up # 打开容器
$ docker-compose down # 关闭容器
// 下面的对应上面的简写
$ dcbuild # 
$ dcup # 
$ dcdown # 
```

所有容器都将在后台运行。要在容器上运行命令，我们通常需要在该容器上获取shell。我们首先需要使用`“docker ps”`命令找出容器的ID，然后使用`“docker exec”`在该容器上启动一个shell。我们在`.bashrc`文件中为它们创建了别名。

```
$ dockps // docker ps 别名
$ docksh <id> //docker exec -it <id> /bin/bash 别名
// 举例：
$ dockps
b1004832e275 hostA-10.9.0.5
0af4ea7a3e2e hostB-10.9.0.6
9652715c8e0a hostC-10.9.0.7
$ docksh 96
root@9652715c8e0a:/#
```

**MySQL数据库**。容器通常是一次性的，因此一旦被破坏，容器内的所有数据都丢失了。对于这个实验室，我们确实希望将数据保存在MySQL数据库中，这样当我们关闭容器时就不会丢失我们的工作。为了实现这一点，我们已经在主机上挂载了mysql数据文件夹(在Labsetup中，它将在mysql容器运行一次后创建)到MySQL容器中的`/var/lib/mysql`文件夹。这个文件夹是MySQL存储数据库的地方。

因此，即使容器被破坏，数据库中的数据仍然被保留。如果要从干净的数据库开始，可以删除此目录

```
 sudo rm -rf mysql_data
```

## 2.2 About the Web Application

我们已经创建了一个web应用程序，这是一个简单的员工管理应用程序。员工可以通过这个web应用程序查看和更新数据库中的个人信息。这个web应用程序主要有两个角色:管理员是一个特权角色，可以管理每个员工的个人档案信息;员工是正常角色，可以查看或更新自己的个人信息。所有员工信息如表1所示

![image-20230409151357601](C:\GitHub\net-work-security\docs\lab3\assets\images\image-20230409151357601.png)

# 3. Lab Tasks

## 3.1 任务1：熟悉mysql语句

本任务的目标是通过使用提供的数据库来熟悉SQL命令。我们的web应用程序使用的数据存储在MySQL数据库中，该数据库托管在我们的MySQL容器上。

我们已经创建了一个名为`sqllab users`的数据库，其中包含一个名为`credential`的表。该表存储每个员工的个人信息(例如`eid`、密码、工资、ssn等)。在这个任务中，你需要使用数据库来熟悉SQL查询。

请在MySQL容器上获取一个shell(参见容器手册中的说明)。然后使用`mysql`客户端程序与数据库进行交互。用户名为`root`，密码为`dees`。

```
// Inside the MySQL container
# mysql -u root -pdees
```

登录后，可以创建新的数据库，也可以加载已有的数据库。由于我们已经为您创建了`sqllab users`数据库，您只需要使用`use`命令加载这个现有的数据库。要显示`sqllab users`数据库中有哪些表，可以使用`show tables`命令打印所选数据库的所有表。

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

运行上述命令后，需要使用SQL命令打印员工Alice的所有概要信息。请提供你的结果截图。

## 3.2 任务2：Select语句下的注入攻击

SQL注入基本上是一种技术，通过这种技术，攻击者可以执行他们自己的恶意SQL语句，通常称为`payload`。通过恶意SQL语句，攻击者可以从受害数据库中窃取信息;更糟糕的是，他们可能会对数据库进行更改。我们的员工管理web应用程序有SQL注入漏洞，这模仿了开发人员经常犯的错误。

我们将使用www.seed-server.com的登录页面来完成这项任务。登录界面如图1所示。它要求用户提供用户名和密码。web应用程序基于这两段数据验证用户身份，因此只有知道密码的员工才能登录。作为攻击者，你的工作是在不知道任何员工的凭证的情况下登录到web应用程序。

![image-20230409152607019](C:\GitHub\net-work-security\docs\lab3\assets\images\image-20230409152607019.png)

为了帮助您开始这项任务，我们将解释如何在web应用程序中实现身份验证。

PHP代码不安全`home.php`位于`/var/www/SQL_Injection`目录下，用于执行用户身份验证。下面的代码片段显示了如何验证用户。

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

上面的SQL语句从凭据表中选择员工个人信息，如id、姓名、工资、ssn等。SQL语句使用输入uname和散列pwd两个变量，其中输入uname保存用户在登录页面的用户名字段中键入的字符串，而散列pwd保存用户键入的密码的sha1散列.程序检查是否有记录与提供的用户名和密码匹配;如果匹配，则成功验证用户，并提供相应的员工信息。如果没有匹配项，则认证失败。

**任务2.1**:网页SQL注入攻击。您的任务是作为管理员从登录页面登录到web应用程序，这样您就可以看到所有员工的信息。我们假设您知道管理员的帐户名admin，但不知道密码。您需要决定在“用户名”和“密码”字段中键入什么才能成功进行攻击。

**任务2.2**:命令行SQL注入攻击。你的任务是重复任务2.1，但你需要在不使用网页的情况下完成。您可以使用命令行工具，例如curl，它们可以发送HTTP请求。值得一提的是，如果你想在HTTP请求中包含多个参数，你需要将URL和参数放在一对单引号之间;否则，用于分隔参数的特殊字符(如&)将被shell程序解释，从而改变命令的含义。下面的例子展示了如何发送一个`HTTP GET`请求

```
$ curl ’www.seed-server.com/unsafe_home.php?username=alice&Password=11’
```

如果需要在`username``或password中包含特殊字符`，则需要对它们进行适当的编码，否则它们可能会改变请求的含义。如果你想在这些字段中包含单引号，你应该使用%27代替;如果你想包含空白，你应该使用%20。在这个任务中，您需要在使用curl发送请求时处理HTTP编码。(url编码)

**任务2.3**:追加一条新的SQL语句。在以上两种攻击中，我们只能从数据库中窃取信息。如果我们可以在登录页面中使用相同的漏洞修改数据库就更好了。

一种想法是使用SQL注入攻击将一条SQL语句转换为两条SQL语句，第二条SQL语句是更新或删除语句。在SQL中，分号(;)用于分隔两条SQL语句。请尝试通过登录页面运行两条SQL语句。

## 3.3 任务3：Select语句下的注入攻击

如果`UPDATE`语句出现SQL注入漏洞，则损害会更严重，因为攻击者可以利用该漏洞修改数据库。在我们的`Employee Management`应用程序中，有一个`Edit Profile`页面(图2)，允许员工更新他们的概要信息，包括昵称、电子邮件、地址、电话号码和密码。员工需要先登录才能进入该页面。

当员工通过Edit Profile页面更新他们的信息时，下面的SQL语句会更新。查询将被执行。在`unsafe_edit_backend.php`文件中实现的PHP代码用于更新日期员工的个人资料信息。PHP文件位于/`var/www/SQLInjection`目录中。

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

![image-20230409153952703](C:\GitHub\net-work-security\docs\lab3\assets\images\image-20230409153952703.png)

**任务3.1**:修改自己的工资。如“编辑个人资料”页面所示，员工只能更新自己的昵称、电子邮件、地址、电话号码和密码;他们无权改变工资。假设你(Alice)是一名心怀不满的员工，你的老板Boby今年没有给你加薪。中的SQL注入漏洞来增加自己的薪水

编辑按钮页面。请演示一下你是如何做到的。我们假设您知道工资存储在名为salary的列中。

![image-20230409154132011](C:\GitHub\net-work-security\docs\lab3\assets\images\image-20230409154132011.png)

**任务3.2**:修改其他人的工资。在给自己加薪之后，你决定惩罚一下你的老板。你想把他的工资减到1美元。请演示一下你是如何做到的。

**任务3.3**:修改他人密码。在修改了老板的薪水后，你仍然很不满，所以你想把老板的密码改成你知道的密码，然后你就可以登录他的账户，进行进一步的破坏。请演示一下你是如何做到的。您需要证明您可以使用新密码成功登录到老板的帐户。这里值得一提的一点是，数据库存储的是密码的哈希值，而不是明文密码字符串。您可以再次查看不安全的`edit backend.php`代码，以查看密码是如何存储的。

### 3.4 任务4:对策—预处理

SQL注入漏洞的根本问题是无法将代码与数据分离。在构造SQL语句时，程序(如PHP程序)知道哪一部分是数据，哪一部分是代码。不幸的是，当SQL语句被发送到数据库时，边界消失了;SQL解释器看到的边界可能与开发人员设置的原始边界不同。要解决这个问题，重要的是要确保服务器端代码和数据库中的边界视图是一致的。最安全的方法是使用预处理语句。

为了理解预处理语句是如何防止SQL注入的，我们需要理解当SQL服务器接收到一个查询时会发生什么。查询如何执行的高级工作流显示在图3。在编译步骤中，查询首先经过解析和规范化阶段，其中根据语法和语义检查查询。下一个阶段是编译阶段，其中关键字(例如;SELECT, FROM, UPDATE等)被转换成机器可以理解的格式。

![image-20230409154132011](C:\GitHub\net-work-security\docs\lab3\assets\images\image-20230409154132011.png)

基本上，在这个阶段，查询是解释的。在查询优化阶段，考虑执行查询的不同计划的数量，从中选择最优的优化计划。所选择的计划存储在缓存中，因此无论何时进入下一个查询，都将根据缓存中的内容进行检查如果它已经存在在缓存中，解析，编译和查询优化阶段将被跳过。然后将编译后的查询传递到实际执行阶段。

预处理语句出现在编译之后、执行步骤之前。准备好的语句将经过编译步骤，并转换为预编译的查询，其中数据占位符为空。要运行这个预编译的查询，需要提供数据，但这些数据不会经过编译步骤;相反，它们被直接插入预编译的查询，并被发送到执行引擎。因此，即使数据中有SQL代码，如果不经过编译步骤，代码将被简单地视为数据的一部分。这就是预处理语句防止SQL注入攻击的方法。

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

使用准备语句机制，我们将向数据库发送SQL语句的过程分为两个步骤。第一步是只发送代码部分，即没有实际数据的SQL语句。这是准备步骤。从上面的代码片段中可以看到，实际数据被问号(?)取代。在这一步之后，我们使用`bind_param()`将数据发送到数据库。数据库将在此步骤中仅将发送的所有内容视为数据，而不再将其视为代码。它将数据绑定到预处理语句的对应问号。在`bind_param()`方法中，第一个参数“is”表示参数类型:“i”表示id中的数据为整数类型，“s”表示pwd中的数据为字符串类型.

**任务**：在这个任务中，我们将使用预处理语句机制来修复SQL注入漏洞。

为了简单起见，我们在防御文件夹中创建了一个简化的程序。我们将对这个文件夹中的文件进行更改。如果您将浏览器指向以下URL，您将看到一个类似于web应用程序的登录页面的页面。通过该页面可以查询员工信息，但需要提供正确的用户名和密码。

```
URL: http://www.seed-server.com/defense/
```

在这个页面中输入的数据将被发送到服务器程序`getinfo.php`，该程序调用一个名为`unsafe.php`的程序。这个PHP程序中的SQL查询容易受到SQL注入攻击。您的任务是使用准备好的语句修改`unsafe.php`中的SQL查询，这样程序就可以击败SQL注入攻击。在实验室安装文件夹中，`unsafe.php`程序在`image_www/Code/ defense`文件夹中。你可以直接在那里修改程序。在你完成之后，你需要重建并重新启动容器，否则更改将不生效。

您还可以在容器运行时修改该文件。在运行的容器中，`unsafe.php`程序位于`/var/www/ sql_injection /defense`内部。这种方法的缺点是，为了保持docker较小，我们只在容器中安装了一个非常简单的文本编辑器`nano`。它应该足以进行简单的编辑。如果你不喜欢这个编辑器，你可以随时使用`“apt install”`在容器中安装您最喜欢的命令行编辑器。例如，对于喜欢vim的人，您可以执行以下操作:

```
# apt install -y vim
```

# 4. Guidelines

测试SQL注入字符串。在实际应用程序中，可能很难检查SQL注入攻击是否包含语法错误，因为服务器通常不会返回这类错误消息。为了进行调查，可以将SQL语句从php源代码复制到MySQL控制台。假设您有以下SQL语句，并且注入字符串是'或1=1;#

```
SELECT * from credential
WHERE name=’$name’ and password=’$pwd’;
```

您可以用注入字符串替换$name的值，并使用MySQL控制台测试它。这种方法可以帮助您在发起真正的攻击之前构建一个没有语法错误的注入字符串。

# 5. Submission

您需要提交一份详细的实验报告，并附带截图，以描述您所做的工作和所观察到的情况。你还需要解释那些有趣或令人惊讶的观察结果。

请同时列出重要的代码片段并给出解释。没有任何解释的简单代码将不会获得分数。
