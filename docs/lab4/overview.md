## 实验目的

&emsp;&emsp;1. 了解TLS/SSL VPN的工作原理；

&emsp;&emsp;2. 掌握VPN Tunnel的配置和使用方法。

## 实验内容

&emsp;&emsp; 本次实验来自于https://seedsecuritylabs.org/Labs_20.04/Networking/VPN_Tunnel/ ，我们只需要完成其中的前6个任务。通过这6个任务。第7-9个任务作为附加题，欢迎大家完成附加题。任务列表如下：

### Task1 部署网络，并测试连通性

### Task2 创建并配置TUN接口

&emsp;&emsp;Task2.1 创建TUN接口

&emsp;&emsp;Task2.2 激活TUN接口

&emsp;&emsp;Task2.3 从TUN接口读取数据

&emsp;&emsp;Task2.4 向TUN接口写入数据

### Task3 通过Tunnel向服务器（VPN）发IP数据包

&emsp;&emsp;3.1 设置服务器（VPN）

&emsp;&emsp;3.2 设置VPN客户端

&emsp;&emsp;3.3 通过Tunnel通道中的进行数据传输

### Task4 配置Tunnel的双向通道

### Task5 观察Tunnel短暂中断发生的情况

## 实验环境

&emsp;&emsp;本次实验需要的主机沿用实验一搭建好的SEED实验室虚拟环境，另外我们还需要三个容器来分别模拟主机V（客户端VPN）、服务器（VPN）和处于私有网络的主机V，因为实验一已经安装好了VM虚拟主机，本次只需要部署容器即可。

###  部署容器

&emsp;&emsp;Step1：下载本次实验需要的容器压缩包[VPN_Labsetup.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)。也可以通过[官网链接](https://seedsecuritylabs.org/Labs_20.04/Networking/VPN_Tunnel/)。

&emsp;&emsp;Step2：将容器压缩包上传到Seed镜像环境中, 建议放在新建的文件/home/seed/VPN_Tunnel目录下，并解压。使用命令为 unzip VPN_Labsetup.zip

    mkdir VPN_Tunnel
    cd VPN_Tunnel
    unzip VPN_Labsetup.zip

&emsp;&emsp;Step3：Build容器，并启动，启动后应该能够看到client，服务器（VPN）和另外两台计算机共4个容器。其中的文件docker-compose2.yml用于task8，可以先不关心。
    
    cd Labsetup/
    dcbuild # Alias for: docker-compose build
    dcup # Alias for: docker-compose up
   <center><img src="../assets/1-1.png" width = 600></center>

&emsp;&emsp;如果启动中遇到如下问题，直接用docker rm 容器ID，重启即可。
    
    ERROR: for VPN_Client  Cannot create container for service VPN_Client: Conflict. The container name "/client-10.9.0.5" is already in use by container "1a3dc7dce80f2c68a937090408c98875e566242579ac2cb2a5f06ab98f3fe1f2". You have to remove (or rename) that container to be able to reuse that name.

    docker rm 1a3dc7dce80f2c68a937090408c98875e566242579ac2cb2a5f06ab98f3fe1f2

&emsp;&emsp;如果启动中遇到如下warning，直接用dcup --remove-orphans 启动即可。

    WARNING: Found orphan containers (mitm-proxy-10.9.0.143, server-10.9.0.43, www-10.9.0.80) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
    

!!! info "说明 :sparkles:"
&emsp;&emsp;容器启动后如果要进入容器的shell，需要通过如下两个命令；在主机终端中输入 dockps 命令，查看刚启动的容器ID；输入命令 docksh ID的前两个字符
    
    dockps // Alias for: docker ps --format "{{.ID}} {{.Names}}"
    docksh <id> // Alias for: docker exec -it <id> /bin/bash   id的前两个字符就行

