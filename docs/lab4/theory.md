# 实验原理

PKI是一种遵循标准的利用公钥理论和技术建立的提供安全服务的基础设施。公钥基础设置的目的是从技术上解决网上身份认证、电子信息的完整性和不可抵赖性等安全问题，为网络应用（如浏览器、电子邮件、电子交易）提供可高安全的服务。本次实验就是利用PKI技术为一个浏览器提供服务。

## 1. 中间人攻击

在发明公钥加密之前，加密依赖私钥。这种加密方法的挑战在于密钥交换，也就是在加密通道建立之前，如何让通信双方把密钥确定下来。公钥解决了这个问题，因为她的加密密钥时公开的，可以用明文发送。然而，经管它可以防御窃听攻击，但是还是为遭遇到中间人攻击。

中间人攻击时发生在两个设备之间的流量被截获的情况下。当一台计算机向另外一台计算机发送数据时，数据会在多个设备之间传输，例如路由器。这些设备如果被攻击，就可以被用来实施中间人攻击。如下图所示：
<center><img src="../assets/2-1.png" width = 500></center>
<center>图2-1 中间人攻击的原理</center>

中间人攻击的基本问题是通信双方无法确定这个公钥是否属于对方，如果能够提供一个机制把公钥和所有者的身份绑定在一起，那么就可以解决这个问题。公钥基础设施（PKI）就是解决此问题的一个方案。

## 2. PKI

### 2.1 PKI体系结构
PKI体系包含证书机构（Certificate Authority, CA）、注册机构（Registration Authority,RA）、策略管理、密钥（Key）与证书管理、密钥备份与恢复、模型运算等功能模块。本实验中我们主要用到CA与证书管理模块，因此PKI可以简化为下图所示：

<center><img src="../assets/2-2.png" width = 500></center>
<center>图2-2 PKI</center>

### 2.1 数字证书
数字证书是一个经证书授权中心（CA机构）数字签名的文件，包含拥有者的公钥及相关身份信息。证书有四种类型，分别是自签名证书、CA证书、本地证书和设备本地证书。本次实验中我们用到了前两种自签名证书和CA证书。

 自签名证书：它的拥有者和证书的颁发者是同一个人。用于没办法去跟CA申请证书的情况下。比如本实验中我们用于虚拟的CA并给自己颁发的证书就是自签名证书。

CA证书，是CA机构给某一申请终端颁发的证书。

## 3. Https访问的认证过程

 我们通过下图看以下Https访问web时的证书应用场景，Https服务器通过访问CA申请并获得一个证书，管理员通过这个证书来验证这个证书就可以了。
<center><img src="../assets/2-3.png" width = 500></center>
<center>图2-3 Https访问web时的认证过程</center>
