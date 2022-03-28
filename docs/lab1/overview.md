## 实验目的

&emsp;&emsp;公钥加密是当今安全通信的基础，但是当通信的一方向另一方发送公钥时，却容易遭受到中间人攻击。根本问题在于没有一个简单的方式验证公钥所有者的身份。也就是说当收到一个公钥和它的所有者信息时，无法确定该公钥确实为这个所有者所拥有。公钥基础设施（PKI）就是解决此问题的一个方案。

&emsp;&emsp;本次实验涉及到PKI、CA、Apache、HTTPS和中间人攻击这几个知识点，所通过本次实验一系列的任务期望能够到达如下几个目的。

&emsp;&emsp;1. 了解PKI的工作原理；

&emsp;&emsp;2. 掌握如何使用PKI保护网络；

&emsp;&emsp;3. 掌握PKI如何击败中间人攻击。


## 实验内容

&emsp;&emsp;本次实验来自于https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_PKI/，共需要完成如下6个小任务。通过这6个任务我们完成一个银行服务器bank32.com的部署、认证、攻击过程。

&emsp;&emsp;Task1 成为认证颁发机构（CA）

&emsp;&emsp;Task2 为web server生成签名请求

&emsp;&emsp;Task3 为web server生成签名证书

&emsp;&emsp;Task4 在网络服务器中部署公钥证书

&emsp;&emsp;Task5 抵御中间人攻击

&emsp;&emsp;Task6 用一个已经劫持到的CA发动一次中间人攻击


## 实验环境

&emsp;&emsp;本次实验需要一个服务器产生证书，另外我们还需要一个容器来模拟web服务器，分两步完成。

### 1. 搭建Seed服务器，需要在Oracle VM VirtualBox上部署Seed实验室的Ubuntu20.04。

<center><img src="../assets/0-1.png" width = 500></center>
<center>进入虚拟机页面显示结果</center>

&emsp;&emsp;下载已经做好镜像环境压缩包，通过[SEED实验室环境部署链接](https://seedsecuritylabs.org/labsetup.html)。
<center><img src="../assets/1-1.png" width = 500></center>

#### 1.1 如果解压后是vdi文件，按照下面步骤导入并启动

<center><img src="../assets/0-2.png" width = 500></center>

<center><img src="../assets/0-3.png" width = 500></center>

<center><img src="../assets/0-4.png" width = 500></center>

<center><img src="../assets/0-5.png" width = 500></center>

<center><img src="../assets/0-6.png" width = 500></center>

<center><img src="../assets/0-7.png" width = 500></center>

<center><img src="../assets/0-8.png" width = 500></center>



&emsp;&emsp;查看IP地址，使用下面的MobaXterm从主机连接虚拟机。

<center><img src="../assets/0-9.png" width = 500></center>

#### 1.1 如果解压后是vbox文件，按照下面步骤导入并启动

&emsp;&emsp;Step1：在VirtualBox上导入镜像并启动。用户名为seed，密码为dees
<center><img src="../assets/1-2.png" width = 500></center>

<center><img src="../assets/1-3.png" width = 500></center>

### 2. 安装连接虚拟机的工具MobaXterm

&emsp;&emsp;安装好虚拟机后，如果方便的从主机方便的上传下载文件呢，可以使用如下工具来实现。

&emsp;&emsp;下载本次绿色版压缩包[MobaXterm_Portable_v21.1.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)，直接解压到后，点击MobaXterm_Personal_21.1.exe文件即可使用。

&emsp;&emsp;打开后，按照下图即可连接虚拟机。IP地址为安装完虚拟机后用ip addr命令查看到的地址。

<center><img src="../assets/1-8.png" width = 500></center>

<center><img src="../assets/1-9.png" width = 500></center>

&emsp;&emsp;连接后的界面如下，就可以直接从主机往虚拟机拖动文件了。

<center><img src="../assets/1-10.png" width = 500></center>

### 3. 部署容器

&emsp;&emsp;Step1：下载本次实验需要的容器压缩包[PKI_Labsetup.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)。也可以通过[SEED实验室PKI Lab](https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_PKI/)。

&emsp;&emsp;Step2：将容器压缩包上传到Seed镜像环境中，并解压。使用命令为 unzip Labsetup.zip

&emsp;&emsp;Step3：Build容器
    &emsp;&emsp;   cd Labsetup/
    &emsp;&emsp;   docker-compose build

&emsp;&emsp;Step4：启动  命令为 dcup
<center><img src="../assets/1-4.png" width = 500></center>
<center>图1-4 容器正常启动</center>

&emsp;&emsp;Step5: 最后，在主机的/etc/hosts文件增加如下一条配置10.9.0.80       www.bank32.com ，其中10.9.0.80是容器的IP地址中。待web服务器配置完成后就可以通过主机访问了。
<center><img src="../assets/1-5.png" width = 500></center>
<center>图1-5 配置主机hosts文件</center>



!!! info "说明 :sparkles:"
&emsp;&emsp;容器启动后如果要进入容器的shell，需要通过如下两个命令；在主机终端中输入 dockps 命令，查看刚启动的容器ID；输入命令 docksh ID的前两个字符
<center><img src="../assets/1-6.png" width = 500></center>
<center>图1-6 查看正在启动的容器ID</center>
<center><img src="../assets/1-7.png" width = 500></center>
<center>图1-7 进入容器的shell</center>
         