# 实验步骤

&emsp;&emsp;本次的实验一共4个任务，基于以下的组网方式实现。

<center><img src="../assets/1-1.png" width = 500></center>

## 1. 实现一个简单的防火墙

&emsp;&emsp;本次任务使用Netfilter技术实现一个简单的数据包过滤器。数据包过滤器只能在内核中实现，因此代码需要运行的内核中，也就是我们需要修改内核。Linux提供了两个技术使得不需要重新编译就能实现数据包过滤器，分别是Netfilter和可加载内核模块（loadable kernel modules）。

### 1.1 实现一个简单的内核模块

&emsp;&emsp;Step1：分析可加载内核模块代码。

&emsp;&emsp;每个内核模块有两个入口点分别用于载入模块和协助模块。具体代码和makefile文件在解压后的路径Labsetup/Files/kernel_module下。

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

&emsp;&emsp;我们来简单分析下代码。上面的代码实现了模块加载时打印Hello World!，模块卸载时打印Bye-bye World!

    module_init()指定的函数在模块载入时被调用，用来进行初始化。
    module_exit()指定的函数在模块卸载时被调用，用来进行清理工作。
    printk()函数用于内核中打印信息，打印内容到内核日志缓存中。

&emsp;&emsp;Step2：编译并安装内核模块

    cd Labsetup/Files/kernel_module  //进入内核文件路径
    make   //编译内核模块
    sudo insmod hello.ko   //把内核模块载入内核
    lsmod | grep hello   //查看hello模块是否已载入内核
    sudo rmmod hello    //把内核模块从内核中删除    

&emsp;&emsp;Step3：执行完加载和卸载我们实现的内核模块，查看内核日志缓冲区输出的信息，可以看到包含我们实现的简单内核模块hello输出的信息。

    dmesg  //检查内核日志缓冲区信息
    .......
    [22263.877120] Hello World!
    [22292.177388] Bye-bye World!.

### 1.2 使用Netfilter搭建一个简单的防火墙

&emsp;&emsp;Linux内核通过Netfilter的钩子函数提供了强大的数据包处理及过滤框架，每个协议栈都在数据包经过的路径上定义了一系列的钩子点。开发者使用内核模块建他们定义的函数“钩”到这些钩子点上。当数据包道道每个钩子点时，协议栈会调用Netfilter框架查找是否有内核模块挂在该钩子点，如果有，这些模块的函数会被调用来分析或处理数据包，然后返回对数据包的处理结果。数据包的5种处理结果如下：

    (1)NF_ACCEPT:允许数据包通过。
    (2)NF_DROP:丢弃数据包，数据包将不会继续传输。
    (3)NF_QUEUE:使用nf_queue机制将数据包传递到用户空间处理。
    (4)NF_STOLEN:告知Netfilter框架武略则个数据包，把数据包进一步的处理工作从Netfilter转交给模块。典型用途时把分片的数据包存储起来，以便在同一个上下文中分析它们。
    (5)NF_REPEAT:请求Netfilter框架再次调用这个模块。

&emsp;&emsp;Netfilter为IPv4定义了5中钩子函数。

    (1)NF_INET_PRE_ROUTING:除了混杂模式，所有数据包都将经过这个钩子点。它上面注册的钩子函数在路由判决之前被调用。
    (2)NF_INET_LOCAL_IN:数据包要进行路由判决，决定时需要被转发还是发完本机。前一种情况下，数据包将前往转发路径；后一种情况，数据包将通过这个钩子点后，被发送到网络协议栈，并最终被主机接收。
    (3)NF_INET_FORWARD：需要被转发的数据包会到达这个钩子点。对于实现防火墙非常重要。
    (4)NF_INET_LOCAL_OUT:本机产生的数据包到达的第一个钩子点。
    (2)NF_INET_POST_ROUTING:需要被转发或者由本机产生的数据包都会经过钩子点。

#### 1.2.1 阻止UDP数据包

&emsp;&emsp;为做对比，先执行dig命令查看，发现有回复，如下图所示。

<center><img src="../assets/1-3.png" width = 500></center>

&emsp;&emsp;使用Labsetup/Files/packet_filter下的代码，阻止IP地址是8.8.8.8和端口为53的UDP数据包。运行命令如下：

    make  //编译seedFilter
    sudo insmod seedFilter.ko   //加入到内核
    lsmod | grep seedFilter    //查看是否加载成功

&emsp;&emsp;加载成功后再执行 dig @8.8.8.8 www.example.com命令，查看结果如下图所示，已经得不到任何响应了，说明防火墙设置成功。

<center><img src="../assets/1-4.png" width = 500></center>

#### 1.2.2 阻止UDP数据包










## 2. XXX

&emsp;&emsp;XXX

### 2.1 XXX

&emsp;&emsp;XXXXX


