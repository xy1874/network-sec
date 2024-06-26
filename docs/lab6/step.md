# 实验步骤

本次的实验一共4个小任务（每个大任务有2个小任务），基于以下的组网方式实现。

<center><img src="../assets/1-1.png" width = 500></center>

## 1. 使用Netfilter技术实现一个简单的防火墙

本次任务使用Netfilter技术实现一个简单的数据包过滤器。数据包过滤器只能在内核中实现，因此代码需要运行在内核中，也就是我们需要修改内核。Linux提供了两个技术使得不需要重新编译就能实现数据包过滤器，分别是Netfilter和可加载内核模块（loadable kernel modules）。

### 1.1 实现一个简单的内核模块

Step1：分析可加载内核模块代码。

每个内核模块有两个入口点分别用于载入模块和卸载模块。具体代码和makefile文件在解压后的路径Labsetup/Files/kernel_module下。

    #include <linux/module.h>
    #include <linux/kernel.h>

    int initialization(void)
    {
        printk(KERN_INFO "Hello World!\n");
        return 0;
    }

    void cleanup(void)
    {
        printk(KERN_INFO "Bye-bye World!.\n");
    }

    module_init(initialization);
    module_exit(cleanup);

    MODULE_LICENSE("GPL");

我们来简单分析下代码。上面的代码实现了模块加载时打印 Hello World!，模块卸载时打印 Bye-bye World!

    module_init()指定的函数在模块载入时被调用，用来进行初始化。
    module_exit()指定的函数在模块卸载时被调用，用来进行清理工作。
    printk()函数用于内核中打印信息，打印内容到内核日志缓存中。

Step2：编译并安装内核模块

    cd Labsetup/Files/kernel_module  //进入内核文件路径
    make   //编译内核模块
    sudo insmod hello.ko   //把内核模块载入内核
    lsmod | grep hello   //查看hello模块是否已载入内核
    sudo rmmod hello    //把内核模块从内核中删除    

Step3：执行完加载和卸载我们实现的内核模块，查看内核日志缓冲区输出的信息，可以看到包含我们实现的简单内核模块 hello 输出的信息。

    dmesg  //检查内核日志缓冲区信息
    .......
    [22263.877120] Hello World!
    [22292.177388] Bye-bye World!.

### 1.2 使用Netfilter搭建一个简单的防火墙

启动容器，如果在已经启动了本次防火墙实验的容器，可忽略此步骤。

    cd ../..    //进入到本次实验的Labsetup目录
    dcbuild     //build容器
    dcup     //启动容器，启动后请将本Terminal保持，其他命令再启动一个新的Terminal

#### 1.2.1 阻止UDP数据包

为做对比，先执行 dig 命令查看，发现有回复，如下图所示。

    dig @8.8.8.8 www.example.com

<center><img src="../assets/1-3.png" width = 500></center>

使用 Labsetup/Files/packet_filter 下的代码，阻止IP地址是 8.8.8.8 和端口为 53 的 UDP 数据包。运行命令如下：

    cd Labsetup/Files/packet_filter //进入防火墙代码路径
    make  //编译seedFilter
    sudo insmod seedFilter.ko   //加入到内核
    lsmod | grep seedFilter    //查看是否加载成功

加载成功后再执行 dig @8.8.8.8 www.example.com 命令，查看结果如下图所示，已经得不到任何响应了，说明防火墙设置成功。

<center><img src="../assets/1-4.png" width = 500></center>

卸载 seedFilter 模块成功后再执行 dig @8.8.8.8 www.example.com 命令，又可以收到回复了。

    sudo rmmod seedFilter

<center><img src="../assets/1-3.png" width = 500></center>

使用 dmesg 命令查看内核日志信息，可以看到注册和卸载的信息，以及防火墙阻止后丢掉的数据包,请参考下面的图片截取内核日志信息在实验报告中。

<center><img src="../assets/1-5.png" width = 500></center>

#### 1.2.2 阻止TCP和PING

参考阻止 UDP 的代码，实现阻止 ping 10.9.0.1 和 telnet 10.9.0.1 的防火墙。

Step1：首先输入 ping 10.9.0.1 和 telnet 10.9.0.1 命令，发现能够正常返回，telnet 也能登录（seed/dees），如下图所示：

<center><img src="../assets/1-6.png" width = 500></center>

<center><img src="../assets/1-7.png" width = 500></center>

Step2：将 seedFilter.c 文件 copy 一份为 task2.c , 仿造 blockUDP 函数，增加两个函数 blockICMP 和 blockTCP ，并在增加两个钩子 hook3，hook4, 分别在 registerFilter 函数和removeFilter 函数中注册和删除。

Step3：修改 Makefile ，将里面的 seedFilter 相关的内容修改为 task2 的内容。

Step4：执行下面的命令

    cd Labsetup/Files/packet_filter //进入防火墙代码路径，本身应该就在该目录下，这个步骤可忽略
    make  //编译task2
    sudo insmod task2.ko   //加入到内核
    lsmod | grep task2    //查看是否加载成功

Step5：分别输入 ping 10.9.0.1 和 telnet 10.9.0.1 命令，发现没有响应。注意 ping 命令如果 5 秒内没有回复即可用 ctrl+c 终止命令，否则时间会很长。

<center><img src="../assets/1-14.png" width = 500></center>

<center><img src="../assets/1-9.png" width = 500></center>

<center><img src="../assets/1-8.png" width = 500></center>

Step6：卸载 task2 模块成功后再执行 ping 10.9.0.1 和 telnet 10.9.0.1 命令，恢复正常。

    sudo rmmod task2

<center><img src="../assets/1-6.png" width = 500></center>

<center><img src="../assets/1-7.png" width = 500></center>  

Step7：使用 dmesg 命令查看内核日志信息，发现有防火墙阻止后丢掉的数据包。

<center><img src="../assets/1-10.png" width = 500></center>

<center><img src="../assets/1-11.png" width = 500></center> 

请将 dmesg 的日志信息参考上面的截图粘贴到实验报告中。

## 2. 使用 iptables 配置无状态防火墙规则

关于 iptables 的使用和介绍，可参考 https://bbs.csdn.net/topics/604460907

### 2.1 保护 Router，只能 ping 通

step1:启动容器，如果在已经启动了防火墙实验的容器，可忽略此步骤。

    cd ../..    //进入到本次实验的 Labsetup 目录
    dcbuild     // build 容器
    dcup     //启动容器，启动后请将本 Terminal 保持，其他命令再启动一个新的 Terminal

step2:使用 dockps 命令查看 HostA 容器的 id，并使用 docksh 59（HostA容器的id前2个字符），并执行 ping 10.9.0.11（Router IP）和 telent 10.9.0.11（Router IP）命令，观察下是否连通。

<center><img src="../assets/1-12.png" width = 500></center>

step3:使用 dockps 命令查看 Router 容器的 id，并使用 docksh de（Router容器的id前2个字符），并执行如下规则

<center><img src="../assets/1-12.png" width = 500></center>

    iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
    iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
    iptables -P OUTPUT DROP   
    iptables -P INPUT DROP    

上面命令中，前面两行是设置Router允许icmp类型协议的应答，下面两行是其他没有设置的协议类型默认拒绝

    iptables -P OUTPUT DROP   //Set default rule for OUTPUT
    iptables -P INPUT DROP    //Set default rule for INPUT

step4:在 HostA 容器中再次执行 ping 10.9.0.11（Router IP）和 telent 10.9.0.11（Router IP）命令，观察下是否连通。将结果截图粘贴到实验报告中，并分析原因。

step5:清理掉上面配置的防火墙规则，请注意，我们在实验中每次测试完毕都要清理掉配置规则，避免影响后面的实验。

    iptables -F
    iptables -P OUTPUT ACCEPT
    iptables -P INPUT ACCEPT

### 2.2 保护内网

step1:在 HostA 容器中执行 ping 192.168.60.5（内网host1 IP）和 telent 192.168.60.5（内网host1 IP）命令，观察下是否连通。

step2:在 Router 容器中配置如下规则，使得满足以下四个条件：
   
    1、外网不能 ping 通内网
    2、外网可以 ping 通 Router
    3、内网可以 ping 通外网
    4、所以其他的内网和外网交互的数据包被阻止掉。

配置规则如下：

    iptables -A FORWARD -i eth0 -p icmp --icmp-type echo-request -j DROP
    iptables -A FORWARD -i eth1 -p icmp --icmp-type echo-request -j ACCEPT
    iptables -A FORWARD -p icmp --icmp-type echo-reply -j ACCEPT
    iptables -P FORWARD DROP

说明：请用 ip addr 查看网卡 ip 地址配置，我这里显示的 eth0 是外网，eth1 是内网。

step3:在 HostA 容器中执行 ping 192.168.60.5（内网Host1 IP）和 telent 192.168.60.5（内网Host1 IP）命令，观察下是否连通。将结果截图粘贴到实验报告中，并分析原因。

step4:在 Host1（192.168.60.5）中分别执行 ping 192.168.60.11 和 ping 10.9.0.5（HostA）观察下是否连通。将结果截图粘贴到实验报告中，并分析原因。

step5:清理掉上面配置的防火墙规则，请注意，我们在实验中每次测试完毕都要清理掉配置规则，避免影响后面的实验。

    iptables -F
    iptables -P OUTPUT ACCEPT
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT


