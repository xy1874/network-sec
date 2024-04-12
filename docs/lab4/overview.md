## 实验目的

公钥加密是当今安全通信的基础，但是当通信的一方向另一方发送公钥时，却容易遭受到中间人攻击。根本问题在于没有一个简单的方式验证公钥所有者的身份。也就是说当收到一个公钥和它的所有者信息时，无法确定该公钥确实为这个所有者所拥有。公钥基础设施（PKI）就是解决此问题的一个方案。

本次实验涉及到PKI、CA、Apache、HTTPS和中间人攻击这几个知识点，所通过本次实验一系列的任务期望能够到达如下几个目的。

1. 了解PKI的工作原理；

2. 掌握如何使用PKI保护网络；

3. 掌握PKI如何击败中间人攻击。


## 实验内容

本次实验来自于[PKI lab](https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_PKI/)，共需要完成如下6个小任务。通过这6个任务我们完成一个银行服务器bank32.com的部署、认证、攻击过程。

Task1 成为认证颁发机构（CA）

Task2 为web server生成签名请求

Task3 为web server生成签名证书

Task4 在网络服务器中部署公钥证书

Task5 抵御中间人攻击

Task6 用一个已经劫持到的CA发动一次中间人攻击


## 实验环境

本次实验需要一个服务器产生证书，另外我们还需要一个容器来模拟web服务器，分两步完成。

### 部署容器

Step1：下载本次实验需要的容器压缩包[SEED实验室PKI Lab](https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_PKI/)。

<center><img src="../assets/1-3.png" width = 500></center>

Step2：将容器压缩包上传到Seed镜像环境中，建议先新建一个文件夹PKI，让压缩包传到/home/seed/PKI路径下并解压。使用命令为 unzip Labsetup.zip

    mkdir PKI  //在/home/seed目录下，我们用seed用户登录，这个默认的路径
    cd PKI
    unzip Labsetup.zip

Step3：Build容器

    cd Labsetup   //在上面已解压的路径 /home/seed/PKI路径下执行该命令
    dcbuild   // docker-compose build 也可以

Step4：启动  

    dcup   //命令启动后如下图所示，不是挂住了，而是正常现象，可以重新打开另外的终端继续操作

<center><img src="../assets/1-4.png" width = 500></center>
<center>图1-4 容器正常启动</center>

Step5: 最后，在主机的/etc/hosts文件增加如下一条配置，其中10.9.0.80是容器的IP地址中。待web服务器配置完成后就可以通过主机访问了。

    10.9.0.80       www.bank32.com 

<center><img src="../assets/1-5.png" width = 500></center>
<center>图1-5 配置主机hosts文件</center>



!!! info "说明 :sparkles:"
容器启动后如果要进入容器的shell，需要通过如下两个命令；在主机终端中输入 dockps 命令，查看刚启动的容器ID；输入命令 docksh ID的前两个字符

    dockps    //查看启动容器的ID
    docksh 4c //进入开头是4c的容器
    exit      //退出容器
    
<center><img src="../assets/1-6.png" width = 500></center>
<center>图1-6 查看正在启动的容器ID</center>
<center><img src="../assets/1-7.png" width = 500></center>
<center>图1-7 进入容器的shell</center>
         