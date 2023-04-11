## 1 SeedLab2.0环境准备

如果是使用实验室的环境请忽略下面的内容，直接进入 3 连接虚拟机下载对应实验资料。

使用VirtualBox-Windows版本，其下载地址 https://download.virtualbox.org/virtualbox/6.1.38/VirtualBox-6.1.38-153438-Win.exe

使用SeedUbuntu20.04作为运行环境，其下载参考 https://seedsecuritylabs.org/labsetup.html ，由于下载较慢，推荐提前下载或者使用第三方镜像.
打开Virtualbox，安装虚拟机

- 选择“新建”，对虚拟机进行命名，完成后点击“下一步”

<center><img src="../assets/1-1.png" width = 400></center>

- 设置内存大小，可以设置为4096MB（不改也可以），完成后点击“下一步”

<center><img src="../assets/1-2.png" width = 400></center>

- **使用已有的虚拟硬盘文件**，选择目录为下载好的`SEED-Ubuntu20.04.vdi`，完成后点击“创建”

<center><img src="../assets/1-3.png" width = 400></center>

- 根据下面的截图配置双网卡，完成虚拟机网络设置

<center><img src="../assets/1-4.png" width = 400></center>

- 启动虚拟机，默认密码是`dees`

<center><img src="../assets/1-5.png" width = 400></center>

## 2 安装连接虚拟机的工具MobaXterm

安装好虚拟机后，可以使用如下工具通过ssh访问虚拟机。

下载本次绿色版压缩包[MobaXterm_Portable_v21.1.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)，直接解压到后，点击MobaXterm_Personal_21.1.exe文件即可使用。

打开后，按照下图即可连接虚拟机。IP地址为安装完虚拟机后用ip addr命令查看到的地址。

<center><img src="../assets/1-8.png" width = 500></center>

<center><img src="../assets/1-9.png" width = 500></center>

连接后的界面如下，就可以直接从主机往虚拟机拖动文件了。

<center><img src="../assets/1-10.png" width = 500></center>

## 3 下载对应实验资料

启动虚拟机，可以通过MobaXterm 和 Vscode 连接虚拟机，也可以直接从VirtualBox中进入，用户名为SEED，密码是`dees` 。登录虚拟机后，可以用以下两种方式下载对应的实验资料，具体可参考每次实验的实验环境部分。

方式1：在虚拟机直接联网，从seed官网 https://seedsecuritylabs.org/Labs_20.04/ 下载对应资料。

方式2：在PC机上下载了相关资料，再传送到虚拟机中，ssh连接工具有 MobaXterm 和 Vscode。 

