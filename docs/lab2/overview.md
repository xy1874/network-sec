## 实验目的
&emsp;&emsp;传输层安全协议（transport layer security, TLS）是一种在两个通信应用程序之间提供安全通道的协议，该通道中的数据传输时加密的，数据的完整性也是被保护的。从先前的SSL(secure socket layer)发展而来，并逐渐q取代SSL。
&emsp;&emsp;通过本次实验的系列任务期望能够达到如下几个目的。

&emsp;&emsp;1. 了解TLS的工作原理；

&emsp;&emsp;2. 通过抓包分析，理解TLS握手过程中的各字段的含义；

&emsp;&emsp;3. 掌握TLS协议的工作过程。


## 实验内容

&emsp;&emsp;本次实验来自于https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_TLS/ ，我们只需要完成其中的前2个任务。通过任务1，我们完成TLS客户端的握手抓包分析过程和TLS的证书验证过，并理解分析TLS的客户端编程；通过任务2我们利用多种该方式完成TLS的服务器端响应客户端的过程，并理解分析TLS的服务器端编程。第三个任务作为附加题，欢迎大家完成附加题。任务列表如下：

### Task1 TLS 客户端

&emsp;&emsp;Task1.1 TLS握手

&emsp;&emsp;Task1.2 TLS协议中的CA认证

&emsp;&emsp;Task1.3 TLS认证中的校验服务器的主机名

&emsp;&emsp;Task1.4 利用TLS协议传输应用数据

### Task2 TLS 服务器端

&emsp;&emsp;Task2.1 实现一个简单的TLS服务器

&emsp;&emsp;Task2.2 利用主机浏览器测试实现是TLS服务器

&emsp;&emsp;Task2.3 测试服务器有别名的情况


## 实验环境

&emsp;&emsp;本次实验需要的主机沿用实验一搭建好的SEED实验室虚拟环境，另外我们还需要三个容器来分别模拟客户端、服务器和中间代理（用做附加题），因为实验一已经安装好了VM虚拟主机，本次只需要部署容器即可。

###  部署容器

&emsp;&emsp;Step1：下载本次实验需要的容器压缩包[TLS_Labsetup.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)。也可以通过[SEED实验室TLS Lab](https://seedsecuritylabs.org/Labs_20.04/Crypto/Crypto_TLS/)。

&emsp;&emsp;Step2：将容器压缩包上传到Seed镜像环境中, 建议放在新建的文件/home/seed/TLS目录下，并解压。使用命令为 unzip TLS_Labsetup.zip

    mkdir TLS
    cd TLS
    unzip TLS_Labsetup.zip

&emsp;&emsp;Step3：Build容器，并启动，启动后应该能够看到client，server和proxy三个容器
    
    cd Labsetup/
    dcbuild # Alias for: docker-compose build
    dcup # Alias for: docker-compose up

    dcup
    WARNING: Found orphan containers (www-10.9.0.80) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
    Starting client-10.9.0.5       ... done
    Starting server-10.9.0.43      ... done
    Starting mitm-proxy-10.9.0.143 ... done
    Attaching to mitm-proxy-10.9.0.143, server-10.9.0.43, client-10.9.0.5


!!! info "说明 :sparkles:"
&emsp;&emsp;容器启动后如果要进入容器的shell，需要通过如下两个命令；在主机终端中输入 dockps 命令，查看刚启动的容器ID；输入命令 docksh ID的前两个字符
    
    dockps // Alias for: docker ps --format "{{.ID}} {{.Names}}"
    docksh <id> // Alias for: docker exec -it <id> /bin/bash   id的前两个字符就行


