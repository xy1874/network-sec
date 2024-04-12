# 实验步骤
本次实验通过6个分解的任务模拟证书签发和认证过程，并通过模拟不同策略的中间人攻击验证PKI防御中间人攻击的安全性。首先将本地主机作为一个CA,完成签发证书的过程；接着用签发的证书去配置安全的Web服务器，通过这个配置，分析验证整个证书验证的过程；最后用不同的策略模仿中间人攻击，验证PKI是如何防御攻击策略的。

## Task0. 部署容器

Step1：下载本次实验需要的容器压缩包[SEED实验室PKI Lab](https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_PKI/)。

<center><img src="../assets/1-3.png" width = 500></center>

**Step2**：将容器压缩包上传到Seed镜像环境中，建议先新建一个文件夹PKI，让压缩包传到/home/seed/PKI路径下并解压。使用命令为 unzip Labsetup.zip

    mkdir PKI  //在/home/seed目录下，我们用seed用户登录，这个默认的路径
    cd PKI
    unzip Labsetup.zip

**Step3**：Build容器

    cd Labsetup   //在上面已解压的路径 /home/seed/PKI路径下执行该命令
    dcbuild   // docker-compose build 也可以

**Step4**：启动  

    dcup   //命令启动后如下图所示，不是挂住了，而是正常现象，可以重新打开另外的终端继续操作

<center><img src="../assets/1-4.png" width = 500></center>
<center>图1-4 容器正常启动</center>

**Step5**: 最后，在主机的/etc/hosts文件增加如下一条配置，其中10.9.0.80是容器的IP地址中。待web服务器配置完成后就可以通过主机访问了。

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


## Task1. 成为认证颁发机构（CA）

认证颁发机构（CA）是一个可信的、能够签发数字证书的实体。在签发证书之前，CA需要验证证书申请者的身份。CA的核心功能有如下两个：（1）验证Subject域；（2）对证书进行数字签名。

一些商业性的CAs被视为根类CAs，想要获得商业核证机关发出的数字证书的用户需要向这些核证机关支付费用。在实验中，我们不使用商业的CA而是将自己成为根CA，然后使用此CA为其他人（例如服务器）颁发证书。

任务1中，我们将使本地主机设置成为根CA，并为此CA生成证书。根CA的证书是自签名的，通常预加载到大多数操作系统、web浏览器和其他依赖PKI的软件中。

### Step1.部署CA。

签名时，openssl会使用一个默认的配置文件（/usr/lib/ssl/openssl.cnf）,该文件中已经配置了需要的文件夹和文件的名字，因为我们要修改这个配置文件，因此我们cp这个文件到自己的目录下，新cp的文件命名为myCA_openssl.cnf。Openssl.cnf文件部分配置内容如下,将unique_subject前面的注释去掉。
<center><img src="../assets/3-1.png" width = 500></center>
<center>图3-1 openssl.conf</center>

因此需要在/home/seed/PKI下创建一个demoCA的目录，并在该文件夹下创建三个文件夹certs、crl和newcerts和两个文件index.txt和serial。Seiral文件包含证书的序列号可以将任意数字反正文件中来初始化序列号，我们采用1000为例。具体命令如下

    cd PKI   //在 /home/seed目录下创建本次实验的文件夹 PKI
    cp /usr/lib/ssl/openssl.cnf myCA_openssl.cnf   //将openssl.cnf 拷贝一份到myCA_openssl.cnf中
    vi myCA_openssl.cnf  //查看文件的配置内容，并把unique_subject和copy_extensions前面的注释去掉

    mkdir demoCA   //根据myCA_openssl.cnf中的内容创建需要的文件夹和文件
    cd demoCA
    mkdir certs crl newcerts

    touch index.txt serial
    vi serial   //最后打开serial写入1000

设置好配置文件myCA_openssl.cnf中需要的信息之后，就可以创建和颁发证书了。具体文件路径如下：

<center><img src="../assets/10-3.png" width = 500></center>

### Step2.生成自签名证书ca.key (私钥证书)和 ca.crt (公钥证书)。

在PKI(自己所建实验目录下)，执行如下命令，具体命令如下：

    openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -keyout ca.key -out ca.crt -subj "/CN=www.modelCA.com/O=Model CA LTD./C=US" -passout pass:dees

    -subj用来设置Subject域信息
    -passout是使用证书时需要的密码信息，因为每次要使用此CA为其他人签名证书时，都必须输入该密码。

利用下面两个命令查看ca.crt和ca.key的内容，回答下面两个问题

    openssl x509 -in ca.crt -text -noout 

    openssl rsa -in ca.key -text -noout

问题1：证书的哪部分内容表明这是证书的持有方？

问题2：证书的哪个部分表明这是自签名证书？

可参考证书的说明如下图所示：
<center><img src="../assets/3-2.png" width = 500></center>
<center>图3-2 证书说明</center>


## Task2. 为web server生成签名请求

如果银行要部署一个基于HTTPS的网络服务器（比如www.bank32.com）来保护客户与服务器之间的交互，就需要从根CA那里获取一个公钥证书。首先需要生成一个签名请求（CSR—Certificate Singing Request），CSR中包含银行的公钥与其身份细节，如机构名称、地址与域名等信息。

生成CSR的命令如下，与Task1中生成自签名的证书类似，去掉了-x509的选项，没有这个选项就是生成了CSR，有这个选项就是生成了自签名证书。命令如下，其中bank32A和bank32B是bank32网站的别名。

    openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.bank32.com/O=Bank32 Inc./C=US" -addext "subjectAltName = DNS:www.bank32.com, DNS:www.bank32A.com, DNS:www.bank32B.com" -passout pass:dees

## Task3. 为web server生成签名证书

根据task2中生成的证书请求文件server.csr用如下命令生成证书文件server.crt.

    openssl ca -config myCA_openssl.cnf -policy policy_anything  -md sha256 -days 3650 -in server.csr -out server.crt -batch -cert ca.crt -keyfile ca.key
    
    在接下来需要输入密码的地方输入dees
    Using configuration from myCA_openssl.cnf
    Enter pass phrase for ca.key:
    Check that the request matches the signature

<center><img src="../assets/6-2.png" width = 500></center>

下面的命令可以查看server.crt的内容，观察下与ca.crt有什么不同？

    openssl x509 -in server.crt -text -noout

## Task4. 在网络服务器中部署公钥证书

一旦银行收到了数字证书，它就可以在HTTPS网站中部署该证书。我们会基于Apache部署一个HTTPS web服务器。首先需要将在主机中生成的证书server.crt和私钥server.key通过volumes文件夹传递给容器（volumes这个文件夹为主机和容器共享的文件夹，主机中放入这个文件夹的文件，容器中可以直接获取到）。

**Step1** 将证书和私钥拷贝到volumes路径下，volumes路径再我们建的目录PKI路径下的Labsetup路径下

    cp server.crt server.key Labsetup/volumes    // 前面任务生成的证书在PKI路径下

<center><img src="../assets/10-1.png" width = 400></center>

**Step2** 在Labsetup路径下启动容器服务器，进入shell，dockps 命令查看容器ID，docksh id的前两个符号即可进入容器shell
    
    dockps    
    2e6d2c851fcd  www-10.9.0.80
    8e37ba198b2f  mitm-proxy-10.9.0.143
    15097d1000a3  server-10.9.0.43
    
    docksh 2e

**Step3** 在完整服务器10.9.0.90容器中将主机传递过来的证书和私钥放到/certs路径下
    
    cd volumes
    cp server.crt server.key ../certs

<center><img src="../assets/10-2.png" width = 400></center>

**Step4** 容器中 /etc/apache2/sites-available 文件夹存放所有可用站点信息，我们需要修改本次实验 www.bank32.com 网站用到的配置文件 bank32_apache_ssl.conf 的内容。修改可以有两种方式，一种是直接在容器中下载vi编辑器。命令如下：

    apt-get update
    apt-get install vim

另一种是通过共享目录volumes传递文件进行修改，主机volumes路径为Lapsetup路径下，容器中在目录下。

    cp /etc/apache2/sites-available/bank32_apache_ssl.conf /volumes  //在容器中执行这条命令，就可以把配置文件放到共享文件夹volumes下

修改文件中的配置如下bank32_apache_ssl.conf：
<center><img src="../assets/3-3.png" width = 300></center>
<center>图3-3 配置bank32网站信息1</center>

**Step5** 在容器中重启apache

    a2enmod ssl    // 使能SSL模式 
    a2ensite bank32_apache_ssl   //使能文件中的配置信息
    service apache2 restart //重启服务器，如果报错，根据提示修改配置文件即可

**Step6** 进入主机，打开浏览器输入https://www.bank32.com的网址，不加证书的情况是报错的，加入开始生成的证书ca.crt,没加证书前，提醒你可能有风险，需要添加证书。

<center><img src="../assets/4-1.png" width = 500></center>
<center>图4-1 提示风险</center>
<center><img src="../assets/4-2.png" width = 500></center>
<center>图4-2 添加证书1</center>
<center><img src="../assets/4-3.png" width = 500></center>
<center>图4-3 添加证书2</center>
<center><img src="../assets/4-4.png" width = 500></center>
<center>图4-4 选择CA证书</center>
<center><img src="../assets/4-5.png" width = 500></center>
<center>图4-5 能够正常访问</center>

请将能够正确访问www.bank32.com的截图放到实验报告中。

## Task5. 抵御中间人攻击

根据task4中配置服务器的方式，配置一个我们学校的网页，同学们可以尝试任何其他的网页。

**Step1** 打开学校网站首页，按ctrl+s 保存文件的内容为index.html

**Step2** 在容器的/var/www目录下创建一个和bank32类似的目录hitsz，并将index.html通过volumes文件夹传到容器的/var/www/hitsz目录下，并拷贝bank32目录下内容一样的文件index_red.html。

**Step3** 将bank32_apache_ssl.conf 文件 copy 一份为 hitsz-ssl.conf ，内容修改为 hitsz 相关的网页信息，证书信息用 www.bank32.com 服务器的。

**Step4** 使能SSL和hitsz-ssl的配置，并重启apache。

    a2enmod ssl    // 使能SSL模式 
    a2ensite hitsz-ssl   //使能文件中的配置信息，如果是第一次加载会提示 service apache2 reload，执行该命令即可
    service apache2 restart //重启服务器，如果报错，根据提示修改配置文件即可

**Step5** 在主机的 /etc/hosts 中添加一条信息 10.9.0.80 www.hitsz.edu.cn，这里的主机是指虚拟机，不是自己的电脑，主机是相对于容器的叫法。使用下面的命令添加一条数据。

    sudo vi /etc/hosts

**Step6** 打开https://www.hitsz.edu.cn，发现报错，报证书有误的信息。

<center><img src="../assets/5-1.png" width = 500></center>
<center>图5-1 提示有风险</center>
<center><img src="../assets/5-2.png" width = 500></center>
<center>图5-2 查看详细信息，证书有问题</center>

## Task6. 用一个已经劫持到的CA发动一次中间人攻击

假设hitsz也是使用我们的根CA证书，而且这个证书的私钥已经被我们劫持了，使用task2和task3来生成hitsz的证书和私钥进行攻击。完成后的结果如下图所示，攻击成功，可以把用户引入到我们设置的虚假网站中。

<center><img src="../assets/6-1.png" width = 500></center>
<center>图6-1 可以访问我们虚假构建的hitsz的网站了</center>



