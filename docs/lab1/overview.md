## 实验目的

1. 了解 Cache 访问速度与RAM访问速度的差距；
2. 了解什么是 FLUSH + RELOAD;
3. 掌握编译和安装内核模块的指令；
4. 学会如何处理 SIGSEGV 信号，使得程序能够继续执行；
5. 理解 CPU 乱序执行的原理；
6. 掌握如何使用户级程序读取到内核内存中数据的方法。

## 实验背景

Meltdown(熔断)是2018年1月披露的CPU特性漏洞，对应漏洞编号 [CVE-2017-5754](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5754)，利用Meltdown漏洞，低权限用户可以访问内核的内容，获取本地操作系统底层的信息。是影响全球CPU的两个漏洞之一，另外一个是Spectre（幽灵），对应两个漏洞编号分别为[CVE-2017-5753](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5753) 和 [CVE-2017-5715](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-5715)。



## 实验内容

 本次实验来自于https://seedsecuritylabs.org/Labs_20.04/System/Meltdown_Attack/，我们需要完成以下8个任务，通过这8个任务完成一次Meltdown攻击。 Task1&2 完成一个简单的侧信道攻击的示例；Task3-5 为Meltdown攻击做准备；Take6 完成乱序执行的示例；Task7 完成一次Meltdown攻击；Task8 给出一个更加实用的攻击效果。

1. 分别测试从 Cache 读取和从内存读取数据的时间长度，对比时间差
2. 完成利用缓存进行侧信道攻击的示例
3. 将秘密数据加载到内核空间
4. 从用户空间访问内核内存
5. 处理程序异常
6. 测试 CPU 的乱序执行
7. 开始基本的 Meltdown 攻击
8. 优化完成更有效的 Meltdown 攻击

## 实验环境

Step1：下载seed环境的镜像

Step2：下载本次实验需要的容器压缩包[Lab1-MeltDown_Labsetup.zip](https://gitee.com/hitsz-cslab/net-work-security/tree/master/stupkt)。也可以通过[官网链接](https://seedsecuritylabs.org/Labs_20.04/System/Meltdown_Attack/)。


Step3：将容器压缩包上传到Seed镜像环境中, 建议放在新建的文件/home/seed/MeltDown目录下，并解压。使用命令为 unzip Lab1-MeltDown_Labsetup.zip

    mkdir MeltDown
    cd MeltDownl
    unzip Lab1-MeltDown_Labsetup.zip




