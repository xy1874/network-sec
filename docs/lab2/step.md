# 实验步骤

&emsp;&emsp;本次实验通过2个程序来看TLS是如何在程序中来保护通信安全的，并讨论TLS编程中常见的错误。一个是HTTPS客户端程序，它可以从HTTPS网络服务器获取网页；另一个是HTTPS服务器程序，它可以给浏览器返回网页。HTTPS是建立在TLS之上的应用层协议。


## 1. TLS客户端

### 1.1 TLS握手

&emsp;&emsp; 在客户端容器中执行给出的TLS握手代码，观察执行结果，回答下面的几个问题。并通过主机的wireshark抓包工具，抓包分析TLS的握手协议。

查看客户端的容器id
dockps
285ad3691dd4  server-10.9.0.43
5883d206e679  mitm-proxy-10.9.0.143
1a3dc7dce80f  client-10.9.0.5

进入容器端的shell，执行如下命令，可以根据结果查看到交换的密钥和证书等信息
docksh 1a
cd volumes/
./handshake.py www.baidu.com

进入主机，打开wireshark工具，选择我们正在使用的网卡开始抓包。然后在客户端容器中重复执行 ./handshake.py www.baidu.com  命令，可以查看分析TLS握手协议。建议大家再通过主机的浏览器访问下www.baidu.com网站，抓包分析下，可以看到握手后的应用层的协议信息。

<center><img src="../assets/2-1.png" width = 200></center>
<center>图2-1 抓包网卡选择</center>

问题1：根据执行结果，客户端和服务器端使用的加密算法有哪些？
问题2：简单分析打印的服务器端的证书
问题3：抓包分析握手协议


### 1.2 TLS协议中的CA认证

&emsp;&emsp;将代码中的证书文件路径改成 ./client-certs,再次在客户端容器运行 ./handshake.py www.baidu.com,观察出现的结果，想想为什么？将www.baidu.com需要的证书copy到 ./client-certs下，根据下面的命令生成hash并做个软链接，再次运行./handshake.py www.baidu.com，查看出现的结果。

openssl x509 -in GlobalSign_Root_CA.pem -noout -subject_hash
5ad8a5d6
ln -s GlobalSign_Root_CA.pem 5ad8a5d6.0

请同学们尝试其他的一个网站来测试对应的结果，并将过程截图保存在报告里。

!!! info "提示 :" 如果找到网站需要的证书呢？我们可以根据第一步执行结果中的信息，查看subjec中的commonName信息，再根据代码中的证书路径/etc/ssl/certs找到对应的证书，copy到client-certs目录下。
  'subject': ((('countryName', 'BE'),),
              (('organizationName', 'GlobalSign nv-sa'),),
              (('organizationalUnitName', 'Root CA'),),
              (('commonName', 'GlobalSign Root CA'),)),

### 1.3 TLS认证中的校验服务器的主机名

&emsp;&emsp;用dig命令查看www.baidu.com的服务器IP地址：
dig www.baidu.com
www.baidu.com.          0       IN      A       163.177.151.110

然后将一个假的主机名比如www.baidu.2022.com写入到客户端的/etc/hosts中
echo 163.177.151.110 wwww.baidu2022.com >> /etc/hosts

然后将代码中的主机名检测分别设置False和True的情况，执行下面的命令，查看执行的结果并分析，如果不做主机名校验会出现的问题。
./handshake.py www.baidu2022.com

### 1.4 利用TLS协议传输应用数据

&emsp;&emsp;利用hankshake.py的代码，增加下面的代码内容放在client.py文件中，尝试传送应用数据，建议删除等待press。

# Send HTTP Request to Server
request = b"GET / HTTP/1.0\r\nHost: " + hostname.encode(’utf-8’) + b"\r\n\r\n"
ssock.sendall(request)
# Read HTTP Response from Server
response = ssock.recv(2048)
while response:
 pprint.pprint(response.split(b"\r\n"))
 response = ssock.recv(2048)

 问题1：简单分析TLS编程的几个主要步骤。

## 2. TLS 服务器


### 2.1 实现一个简单的TLS服务器

&emsp;&emsp;这个任务中我们将使用实验中生成的CA证书和www.bank32.com的证书和私钥，首先将CA证书copy到客户端的client-certs目录下并进行软链接，然后将bank32服务器的证书和私钥拷贝到到server-certs目录下。

cp ca.crt ../TLS/Labsetup/volumes/client-certs/
sudo cp server.crt server.key ../TLS/Labsetup/volumes/server-certs/

在主机中，修改server.py的证书正确的路径和名称(server.crt 和server.key)，并绑定服务器的ip 10.9.0.43。
在服务器容器中，启动服务器 ./server.py,输入我们实验中设置的证书密码dees

在客户端容器中，将ca.crt做软链接，并将www.bank32.com的信息写入/etc/hosts中
openssl x509 -in ca.crt -noout -subject_hash
  dbb9c584
ln -s ca.crt dbb9c584.0
echo 10.9.0.43 www.bank32.com >> /etc/hosts
更改client.py中证书的路径为 client-certs，并将端口号改为4433和服务器保持一致。
在客户端容器中,启动客户端。
/client.py www.bank32.com

再切换到服务器端可以看到有消息接收。
问题1：请分析server.py的代码，说明下服务器程序的关键步骤。

### 2.2 利用主机浏览器测试实现的TLS服务器

&emsp;&emsp;从主机的浏览器进入到https://www.bank32.com网站，查看服务器的链接情况，如果还是在第一次的实验环境中，我们已经加入了证书，如果没有加入证书，需要在firefox浏览器加入ca.crt的证书。


### 2.3 测试服务器有别名的情况

&emsp;&emsp;因为我们在第一个PKI的实验中已经让大家添加过www.bank32.com的别名，所以证书部分我们不需要修改，只需要在客户端容器中的/etc/hosts中添加另外两个别名和对应的服务器IP就可以了。
问题1：请分别用client.py和浏览器两种方式访问服务器，并记录你观察的结果（截图）。
