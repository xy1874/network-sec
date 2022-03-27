# 实验步骤

## 1. 实验组网

&emsp;&emsp;要求在中间路由器R1没有设置路由转发表的前提下，深圳——哈尔滨建立VPN隧道互通。组网方式如下图所示。

<center><img src="../assets/2-1.png" width = 200></center>

&emsp;&emsp;三台路由器IP地址列表如下：

|  设备名称  |       IP地址     |
|  ------   |    ----------    |
| R0-f0/1   | 192.168.1.254/24 |
| R0-f0/0   | 100.1.1.1/24 |
| R1-f0/1   | 200.1.1.1/24 |
| R1-f0/0   | 100.1.1.2/24 |
| R2-f0/1   | 200.1.1.2/24 |
| R2-f0/0   | 172.16.1.254/24 |

&emsp;&emsp;两台路PC机IP地址列表如下：

|  计算机   | IP地址  | 默认网关  |
|  ----    | ----    | ----  |
| P0  | 192.168.1.1 | 192.168.1.254 |
| P1  | 172.16.1.1  | 172.16.1.254  |

<center><img src="../assets/2-2.png" width = 200></center>
<center><img src="../assets/2-3.png" width = 200></center>

## 2. 路由器接口IP和路由表项

### 2.1 配置R0的信息

    Would you like to enter the initial configuration dialog? [yes/no]: no
    Press RETURN to get started!
    Router>enable
    Router#configure terminal
    Router(config)#ho R0
    R0(config)#int f0/1
    R0(config-if)#ip add 192.168.1.254 255.255.255.0
    R0(config-if)#no sh
    R0(config-if)#exit
    R0(config)#int f0/0
    R0(config-if)#ip add 100.1.1.1 255.255.255.0
    R0(config-if)#no sh
    R0(config-if)#exit
    R0(config)#ip route 0.0.0.0 0.0.0.0 100.1.1.2

### 2.2 配置R1的信息
    Would you like to enter the initial configuration dialog? [yes/no]: no
    Router>enable
    Router#configure terminal
    Router(config)#ho R1
    R1(config)#int f0/0
    R1(config-if)#ip add 100.1.1.2 255.255.255.0
    R1(config-if)#no sh
    R1(config-if)#exit
    R1(config)#int f0/1
    R1(config-if)#ip add 200.1.1.1 255.255.255.0
    R1(config-if)#no sh
    R1(config-if)#eixt

### 2.3 配置R2的信息
    Would you like to enter the initial configuration dialog? [yes/no]: no
    Router>enable
    Router#configure terminal
    Router(config)#ho R2
    R2(config)#int f0/1
    R2(config-if)#ip add 200.1.1.2 255.255.255.0
    R2(config-if)#no sh
    R2(config-if)#int f0/0
    R2(config-if)#ip add 172.16.1.254 255.255.255.0
    R2(config-if)#no sh
    R2(config-if)#exit
    R2(config)#ip route 0.0.0.0 0.0.0.0 200.1.1.1

## 3. 配置IPsec信息
&emsp;&emsp;管理连接、创建连接、创建map映射表、将map表应用到外网端口，3.1-3.4解释了这四个阶段的命令，3.5和3.6是直接进行配置信息。

### 3.1 阶段1--管理连接

&emsp;&emsp;通信双方设备通过非对称加密算法 加密 对称加密算法 所使用的 对称密钥。具体命令如下，后面是命令解释说明。

    crypto isakmp policy 1          （ IKE     传输集/策略集，可以多配置几个策略集）
    encryption des/3des/aes         (加密算法类型)
    hash md5/sha                        （完整性校验算法类型）
    group 1/2/5                              （使用D-H算法的版本？）
    authentication pre-share          （预共享密钥算法，身份验证方式）
    lifetime 秒 （默认86400秒）      （可以不配置，对称密钥有效时间，如果双方不一致，取min，最好一致）
    exit
    crypto isakmp key 预共享密钥 address 对方的公网IP地址

### 3.2 阶段2--创建连接

&emsp;&emsp;通过 对称加密算法 加密 实际所要传输的私网数据。具体命令如下

    access-list 100 permit ip 192.168.1.0 0.0.0.255 172.16.0.0 0.0.255.255 （定义允许的流量列表：）
    crypto ipsec transform-set 传输模式名 esp-des/3des/aes esp/ah-md5/sha-hmac （定义加密及认证方式：）

### 3.3 阶段3--创建MAP映射表

    crypto map map名 1 ipsec-isakmp         （1是隧道1）
    match address ACL表名               （阶段2里面的ACL表名）
    set transform-set 传输模式名
    set peer 对方的公网IP

### 3.4 阶段4--将map表应用到外网端口
&emsp;&emsp; 到对应接口下映射map
    crypto map map名

### 3.5 配置R0端的IPsec信息

&emsp;&emsp;进入配置模式
    Would you like to enter the initial configuration dialog? [yes/no]: no
    R0>enable
    R0#configure terminal

&emsp;&emsp;进入配置模式后，可以手动输入如下命令，或者直接将如下代码paste到R0的配置页面。
    crypto isakmp policy 1
    encryption aes
    group 2
    hash sha
    authentication pre-share
    exit
    crypto isakmp key dees address 200.1.1.2
    acc 100 permit ip 192.168.1.0 0.0.0.255 172.16.1.0 0.0.0.255
    crypto ipsec transform-set getran esp-aes esp-sha-hmac
    crypto map gemap 1 ipsec-isakmp
    set peer 200.1.1.2
    match address 100
    set transform-set getran
    exit
    int f0/0
    crypto map gemap

&emsp;&emsp;可以通过命令查看当前连接未建立
<center><img src="../assets/2-4.png" width = 200></center>

### 3.6 配置另一端R2的IPsec信息

&emsp;&emsp;配置另一端
    R2>enable
    R2#configure terminal

&emsp;&emsp;进入配置模式后，可以手动输入如下命令，或者直接将如下代码paste到R1的配置页面。
    crypto isakmp policy 2
    encryption aes
    group 2
    hash sha
    authentication pre-share
    exit
    crypto isakmp key dees address 100.1.1.1 
    acc 102 permit ip 172.16.1.0 0.0.0.255 192.168.1.0 0.0.0.255
    crypto ipsec transform-set ge2tran esp-aes esp-sha-hmac  
    crypto map ge2map 1 ipsec-isakmp
    set peer 100.1.1.1
    match address 102
    set transform-set ge2tran
    exit 
    int f0/1
    crypto map ge2map

&emsp;&emsp;可以通过命令查看当前连接建立完成
    do sh crypto isakmp sa
<center><img src="../assets/2-5.png" width = 200></center>

&emsp;&emsp;查看R0的状态
<center><img src="../assets/2-5.png" width = 200></center>

    R0(config)#do sh crypto ipsec sa

    interface: FastEthernet0/0
    Crypto map tag: gemap, local addr 100.1.1.1
    
    protected vrf: (none)
    local  ident (addr/mask/prot/port): (192.168.1.0/255.255.255.0/0/0)
    remote  ident (addr/mask/prot/port): (172.16.1.0/255.255.255.0/0/0)
    current_peer 200.1.1.2 port 500
    PERMIT, flags={origin_is_acl,}
    #pkts encaps: 2, #pkts encrypt: 2, #pkts digest: 0
    #pkts decaps: 3, #pkts decrypt: 3, #pkts verify: 0
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 100.1.1.1, remote crypto endpt.:200.1.1.2
     path mtu 1500, ip mtu 1500, ip mtu idb FastEthernet0/0
     current outbound spi: 0x7B636B63(2070113123)

     inbound esp sas:
      spi: 0x537E6D10(1400794384)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2004, flow_id: FPGA:1, crypto map: gemap
        sa timing: remaining key lifetime (k/sec): (4525504/3086)
        IV size: 16 bytes
        replay detection support: N
        Status: ACTIVE

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x7B636B63(2070113123)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2005, flow_id: FPGA:1, crypto map: gemap
        sa timing: remaining key lifetime (k/sec): (4525504/3086)
        IV size: 16 bytes
        replay detection support: N
        Status: ACTIVE

     outbound ah sas:

     outbound pcp sas:

     local crypto endpt.: 100.1.1.1, remote crypto endpt.:100.1.1.1
     path mtu 1500, ip mtu 1500, ip mtu idb FastEthernet0/0
     current outbound spi: 0x0(0)

     inbound esp sas:

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:

     outbound ah sas:

     outbound pcp sas:

## 4. 在PC端发送ICMP信息，抓包分析

### 2.1 从PC1 ping PC0，如图所示，时间响应较慢但是能够互通。
<center><img src="../assets/2-7.png" width = 200></center>

### 2.1 抓包分析

&emsp;&emsp;在Cisco Packet Tracer软件菜单栏View -> Simulation Mode，在工作区的右边出现一个“Simulation Panel”，此时就进入了仿真模式。
单击Show All/None，Event List Filters-Visible Events窗口栏中变成None。然后，再点击Edit Filters。在弹出的窗口里，选择IPv4选项卡，选择ICMP协议，本实验只需观察这个协议即可。

&emsp;&emsp;配置完成后点击play键，然后再从PC1 ping PC0，观察抓包结果。
<center><img src="../assets/2-8.png" width = 200></center>

&emsp;&emsp;分别观察发送的路径过程包，可以看到路由器R0,R1和R2之前的的数据包中是由IPSec包头的，其他的是没有的，想一想为什么？
<center><img src="../assets/2-9.png" width = 200></center>




