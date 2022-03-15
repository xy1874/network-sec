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

&emsp;&emsp;Step1：下载已经做好镜像环境压缩包，通过[SEED实验室环境部署链接](https://seedsecuritylabs.org/labsetup.html)。
<center><img src="../assets/1-1.png" width = 200></center>
<center>图1-1 导入镜像</center>
&emsp;&emsp;Step2：在VirtualBox上导入镜像并启动。用户名为SEED，密码为dees
<center><img src="../assets/1-2.png" width = 200></center>
<center>图1-2 导入镜像</center>

<center><img src="../assets/1-3.png" width = 200></center>
<center>图1-3 启动</center>

### 2. 部署容器
&emsp;&emsp;Step1：下载本次实验需要的容器压缩包[PKI_Labsetup.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)。也可以通过[SEED实验室PKI Lab](https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_PKI/)。
&emsp;&emsp;Step2：将容器压缩包上传到Seed镜像环境中，并解压。使用命令为 unzip Labsetup.zip
&emsp;&emsp;Step3：Build容器
    &emsp;&emsp;   cd Labsetup/
    &emsp;&emsp;   docker-compose build
&emsp;&emsp;Step3：启动  命令为 dcup
<center><img src="../assets/1-4.png" width = 200></center>
<center>图1-4 容器正常启动</center>

!!! info "说明 :sparkles:"
&emsp;&emsp;容器启动后如果要进入容器的shell，需要通过如下两个命令；在主机终端中输入 dockps 命令，查看刚启动的容器ID；输入命令 docksh ID的前两个字符
<center><img src="../assets/1-5.png" width = 200></center>
<center>图1-5 查看正在启动的容器ID</center>
<center><img src="../assets/1-6.png" width = 200></center>
<center>图1-6 进入容器的shell</center>
         