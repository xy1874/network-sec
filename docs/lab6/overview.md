## 实验目的

1. 了解防火墙的工作原理；

2. 掌握防火墙的工作机制；

3. 熟练掌握iptable防火墙的配置。


## 实验内容

本次实验通过使用Netfilter实现一个简单的数据包过滤器，并用Linux内置的防火墙iptables搭建一个稍微复杂的防火墙。

## 实验环境

本次实验需要的主机沿用实验一搭建好的SEED实验室虚拟环境，另外我们还需要5个容器来，因为实验一已经安装好了VM虚拟主机，本次只需要部署容器即可。

###  部署容器

Step1：下载本次实验需要的容器压缩包[SEED实验室Firewall Lab](https://seedsecuritylabs.org/Labs_20.04/Networking/Firewall/)。

Step2：将容器压缩包上传到Seed镜像环境中, 建议新建一个文件夹/home/seed/Firewall，将压缩包放在/home/seed/Firewall目录下，并解压。使用命令为 unzip Labsetup.zip
   
    mkdir Firewall
    cd Firewall
    unzip Labsetup.zip

Step3：Build容器，并启动，启动后应该能够看到5个容器启动。
    
    cd Labsetup/
    dcbuild # Alias for: docker-compose build
    dcup # Alias for: docker-compose up

<center><img src="../assets/1-2.png" width = 500></center>


!!! info "说明 :sparkles:"
容器启动后如果要进入容器的shell，需要通过如下两个命令；在主机终端中输入 dockps 命令，查看刚启动的容器ID；输入命令 docksh ID的前两个字符
    
    dockps // Alias for: docker ps --format "{{.ID}} {{.Names}}"
    docksh <id> // Alias for: docker exec -it <id> /bin/bash   id的前两个字符就行
