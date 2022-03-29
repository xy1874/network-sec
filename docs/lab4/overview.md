## 实验目的

&emsp;&emsp;1. 了解xx的工作原理；

&emsp;&emsp;2. 

&emsp;&emsp;3. 


## 实验内容

&emsp;&emsp;


## 实验环境

&emsp;&emsp;

###  部署容器

&emsp;&emsp;Step1：下载本次实验需要的容器压缩包[VPN_Labsetup.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)。也可以通过[官网链接](https://seedsecuritylabs.org/Labs_20.04/Networking/VPN_Tunnel/)。

&emsp;&emsp;Step2：将容器压缩包上传到Seed镜像环境中, 建议放在新建的文件/home/seed/VPN_Tunnel目录下，并解压。使用命令为 unzip VPN_Labsetup.zip

    mkdir VPN_Tunnel
    cd VPN_Tunnel
    unzip VPN_Labsetup.zip

&emsp;&emsp;Step3：删除其中的文件docker-compose2.yml，然后再Build容器，并启动，启动后应该能够看到client，Router和另外两台计算机共4个容器
    
    cd Labsetup/
    dcbuild # Alias for: docker-compose build
    dcup # Alias for: docker-compose up
   <center><img src="../assets/1-1.png" width = 400></center>

&emsp;&emsp;如果启动中遇到如下问题，直接用docker rm 容器ID，重启即可。
    
    ERROR: for VPN_Client  Cannot create container for service VPN_Client: Conflict. The container name "/client-10.9.0.5" is already in use by container "1a3dc7dce80f2c68a937090408c98875e566242579ac2cb2a5f06ab98f3fe1f2". You have to remove (or rename) that container to be able to reuse that name.

    docker rm 1a3dc7dce80f2c68a937090408c98875e566242579ac2cb2a5f06ab98f3fe1f2

&emsp;&emsp;如果启动中遇到如下warning，直接用dcup --remove-orphans 启动即可。

    WARNING: Found orphan containers (mitm-proxy-10.9.0.143, server-10.9.0.43, www-10.9.0.80) for this project. If you removed or renamed this service in your compose file, you can run this command with the --remove-orphans flag to clean it up.
    

!!! info "说明 :sparkles:"
&emsp;&emsp;容器启动后如果要进入容器的shell，需要通过如下两个命令；在主机终端中输入 dockps 命令，查看刚启动的容器ID；输入命令 docksh ID的前两个字符
    
    dockps // Alias for: docker ps --format "{{.ID}} {{.Names}}"
    docksh <id> // Alias for: docker exec -it <id> /bin/bash   id的前两个字符就行

