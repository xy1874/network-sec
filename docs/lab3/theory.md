# 实验原理

&emsp;&emsp;IPSec(Internet Protocol security)是TCP/IP协议栈网络层和传输层之间的一套互联网安全协议合集。IPSec协议框架设计初衷是要弥补TCP/IP协议栈安全性的不足，用以保障通信双方IP层之上的协议安全。IPSec是IPv6的组成部分，也是IPv4的可选扩展协议。

## 1. IPSec的安全体系结构

&emsp;&emsp;IPSec协议可以提供数据加密、数据完整性、数据源认证和放重放几个主要功能，其安全体系结构如下图所示。

<center><img src="../assets/1-1.png" width = 500></center>
<center>图1-1 IPsec的安全体系结构</center>

&emsp;&emsp;（1）	封装安全载荷（Encapsulating Security Payload,ESP）,ESP是一个安全协议头，它采用加密和认证机制，为IP数据包提供数据源认证、数据完整性、抗重播和机密性安全服务，可以在传输模式和隧道模式下使用。

&emsp;&emsp;（2）	认证头（Authentication Header，AH），AH也是一个安全协议头，它采用认证机制，为IP数据包提供数据源认证、数据完整性、抗重播安全服务，工作于传输模式和隧道模式。

&emsp;&emsp;（3）	加密算法，加密算法米搜狐了各种能用于ESP的机密算法，IPSec种有许多加密算法可供选择，包括DES、3DES、IDES、AES等。

&emsp;&emsp;（4）	认证算法，认证算法描述了各种能用于AH和ESP的认证算法，IPSec用基于消息认证的密钥散列算法（HMAC）作为认证算法，也支持HMAC与安全散列算法（SHA）等其他算法结合使用，以提供安全强度。

&emsp;&emsp;（5）	解释域（Domain of Interpretation,DOI），为了IPSec通信两端能相互交互，通信双方应该理解AH协议和ESP协议载荷种各字段的取值，因此通信双方必须保持对通信消息相同的解释规则，即应该持有相同的解释域。IPSec给出了两个解释域：IPSec DOI和ISAKMP（Internet Security Association and Key Management Protocol，Internet 安全关联和密钥管理协议） DOI，它们各有不同的使用范围。解释域定义了协议用来确定安全服务的信息通信双方必须支持的安全策略，规定所采用的句法，命名相关安全服务信息时的方案，包括加密算法、密钥交换算法、安全策略特性和认证中心等。

&emsp;&emsp;（6）	密钥管理，IPSec的密钥管理包括密钥的确定和分配，有手工和自动两种方式。IPSec默认的自动密钥管理协议是Internet密钥交换（Internet Key Exchange，IKE）。IKE规定了自动认证IPSec对等实体、协商安全服务和产生共享密钥的标准。

&emsp;&emsp;（7）	策略，策略位于安全检查规范的最高一级，是决定系统的安全要。安全策略（Security Policy，SP）指示对IP数据包提供何种保护，并以何种方式实施保护。安全关联（Security Association，SA）是两个IPSec应用实体间的一个单向逻辑连接，决定保护什么、如何保护以及谁来保护通信数据。SA是SP的具体化及实例化。

## 2. IPSec的工作模式

&emsp;&emsp;IPSec协议可以设置成在两种模式下运行：传输模式和隧道模式。

### 2.1 传输模式

&emsp;&emsp;IPSec部署在两个终端上，提供端到端的安全保护。IPSec模块在IP包的IP头和数据载荷之间添加AH或ESP，不会修改IP头中的信息。传输模式对数据包的修改情况如下图所示。

<center><img src="../assets/1-2.png" width = 500></center>
<center>图1-2 IPsec传输模式对数据包的修改</center>

### 2.2 隧道模式

&emsp;&emsp;IPSec部署在两个安全网关上，即用于路由器、防火墙、VPN集中器等网络设备之间。发送端对原始IP报文整体加密，再在前面加入一个新的IP包头，用新的IP地址将数据分组路由到接收端。因此IP隧道模式的数据包包含两个IP头：内部IP头和外部IP头，内部头是数据包的源主机创建的,外部头是IPSec实体添加的。

<center><img src="../assets/1-3.png" width = 500></center>
<center>图1-3 IPsec隧道模式对数据包的修改</center>

## 3. 安全关联SA

&emsp;&emsp;SA是构成IPSec的基础。AH和ESP协议都使用SA来提供服务。SA是两个IPSec通信实体之间经协商建立起来的一种共同协定，它规定了通信双方使用那种IPSec协议保护数据安全、协议的操作模式（传输模式和隧道模式）、加密算法、密钥以及密钥的生存周期等等安全属性值。任何IPSec实施方案始终会构建一个SADB，由它来维护IPSec协议用来保障数据包安全的SA记录。

&emsp;&emsp;一个SA是一个发送方于接收方之间的单项关系，若需要一个对等关系，即双向安全交换，就需要建立两个SA。一个单一的SA可以向AH或ESP提供安全服务，但不能同事提供给两者。SA是由三个参数来唯一标识的：

&emsp;&emsp;（1）	安全参数索引（Security Parameters Index，SPI），SPI是分配给SA的比特串，仅在本地可用。SPI在AH头或ESP头中出现，解决了在目的主机上标识SA的问题。随每个数据包以到，需要发送一个SPI，以便将SA独一无二地标识出来。目标主机再利用这个值，对接收SADB进行检索，提取适当的SA。

&emsp;&emsp;（2）	IP目的地址，目前只允许单播地址，它是SA的终端地址

&emsp;&emsp;（3）	安全协议标志符，指出该SA是用于AH头还是ESP头。
