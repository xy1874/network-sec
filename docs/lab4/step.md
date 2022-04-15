# 实验步骤

&emsp;&emsp;

## 1. 网络部署

&emsp;&emsp;本次实验的网络部署如下图所示。

 <center><img src="../assets/2-1.png" width = 600></center>

&emsp;&emsp;容器启动后就可以在后台继续运行了，我们开一个新的连接，查看容器信息。

    dockps
    19f26b5bdb71  client-10.9.0.5
    e019b7e018b1  server-router
    af2671fc2a0e  host-192.168.60.6
    b5378add757b  host-192.168.60.5

&emsp;&emsp;分别验证HostU(VPN客户端)、VPN服务器和HostV的连通性。进入容器命令为 docksh 容器id的前两个字符。

&emsp;&emsp;根据前面的网络部署图我们知道VPN服务器的IP地址分别为10.9.0.11和192.168.60.11，也可通过进入VPN服务器的容器中通过ip addr命令查看。

    docksh e0
    root@e019b7e018b1:/# ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
    73: eth0@if74: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:0a:09:00:0b brd ff:ff:ff:ff:ff:ff link-netnsid 0
        inet 10.9.0.11/24 brd 10.9.0.255 scope global eth0
           valid_lft forever preferred_lft forever
    75: eth1@if76: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:c0:a8:3c:0b brd ff:ff:ff:ff:ff:ff link-netnsid 0
        inet 192.168.60.11/24 brd 192.168.60.255 scope global eth1
           valid_lft forever preferred_lft forever

&emsp;&emsp;通过下面的命令测试，可以发现HostU可以与VPN服务器的通信，但是不能与主机HostV通信

    docksh 19   //进入客户端容器HostU
    ping 10.9.0.11
    ping 192.168.60.5

&emsp;&emsp;VPN服务器可以与HostU、HostV通信。
    
    docksh e0  //进入VPN服务器容器
    ping 10.9.0.5
    ping 192.168.60.5

&emsp;&emsp;在VPN服务器上运行tcpdump，并嗅探每个网络上的流量。使用下面的命令，分别通过容器HostU和HostV发送ping命令，显示你可以捕获的数据包。

    tcpdump -i eth0 -n  //捕获 10.9.0.5发送过来的ICMP数据包
    tcpdump -i eth1 -n  ////捕获 192.168.60.5发送过来的ICMP数据包

<center><img src="../assets/2-2.png" width = 600></center>

## 2. 创建和配置TUN接口

### 2.1通过tun.py脚本配置tun并启动，启动后在后台运行，不启动输入其他命令

    root@19f26b5bdb71:/volumes# ls -l
    total 4
    -rwxr-xr-x 1 seed seed 511 Dec  5  2020 tun.py
    root@19f26b5bdb71:/volumes# chmod a+x tun.py
    root@19f26b5bdb71:/volumes# ./tun.py
    Interface Name: tun0

!!! info "提示 :sparkles:"
&emsp;&emsp;执行命令前请给./tun.py通过chmod a+x 文件名  加权限。

&emsp;&emsp;从另一个终端连接启动client客户端，利用ip addr查看ip配置信息，发现多了一个接口tun0，如下图所示。

<center><img src="../assets/2-3.png" width = 600></center>

### 2.2 配置TUN接口

&emsp;&emsp;使用下面两条命令给TUN接口配置ip并启动，然后再通过ip addr命令观察TUN接口的信息如下图所示，ip地址已经配置上了。

    ip addr add 192.168.53.99/24 dev tun0
    ip link set dev tun0 up

<center><img src="../assets/2-4.png" width = 600></center>

### 2.3 从TUN接口读取信息

&emsp;&emsp;将如下两条命令和上面的配置TUN接口的两条命令一起写入tun.py

    //下面这段代码放在打印接口名字后面
    # Set up the tun interface
    os.system("ip addr add 192.168.53.99/24 dev {}".format(ifname))
    os.system("ip link set dev {} up".format(ifname))

    //下面这段代码替换原来的循环语句
    while True:
    # Get a packet from the tun interface
      packet = os.read(tun, 2048)
      if packet:
        ip = IP(packet)
        print(ip.summary())

&emsp;&emsp;重新运行tun.py, 并从另外一个连接进入客户端查看TUN接口的情况（ip addr）,然后分别执行如下命令，查看tun.py的运行结果，分析说明下原因。

    ping 192.168.53.88
    ping 192.168.60.6

### 2.4 从TUN接口写入信息,将下面的代码加入到tun.py中，重新运行./tun.py，再从客户端ping 192.168.53.88，可以看到客户端有消息回复。

      # Send out a spoof packet using the tun interface
      if ICMP in ip:
         newip = IP(src=ip[IP].dst, dst=ip[IP].src, ihl=ip[IP].ihl)
         newip.ttl = 99
         newicmp   = ICMP(type=0, id=ip[ICMP].id, seq=ip[ICMP].seq)
         if ip.haslayer(Raw):
            data = pkt[Raw].load
            newpkt = newip/newicmp/data
         else:
            newpkt = newip/newicmp

## 3. 通过Tunnel向VPN服务器发IP数据包

### 3.1设置VPN服务器代码，根据上面已有的tun.py文件，增加和改动如下内容，形成serve.py文件

    IP_A = "0.0.0.0"
    PORT = 9090
    ......
    # Set up the tun interface
    os.system("ip addr add 192.168.53.1/24 dev {}".format(ifname))
    os.system("ip link set dev {} up".format(ifname))
    ......
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    sock.bind((IP_A, PORT))

    while True:
        data, (ip, port) = sock.recvfrom(2048)
        print("{}:{} --> {}:{}".format(ip, port, IP_A, PORT))
        pkt = IP(data)
        print(" Inside: {} --> {}".format(pkt.src, pkt.dst))


### 3.2设置HostU的客户端代码，根据上面已有的tun.py文件，增加和改动如下内容形成client.py文件

    SERVER_IP   = "10.9.0.11"
    SERVER_PORT = 9090

    # Set up routing
    os.system("ip route add 192.168.60.0/24 dev {}".format(ifname))

    # Create UDP socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    while True:
    # Get a packet from the tun interface
    packet = os.read(tun, 2048)
    if packet:
       ip = IP(packet)
       print(ip.summary())

### 3.3 通过Tunnel通道中的进行数据传输
&emsp;&emsp;分别在VPN服务器容器和HostU容器中执行./server.py和./client.py文件，在另外一个HostU容器的shell中，执行ping 192.168.60.5,分别观察三个终端中的信息，并在HostV中执行tcpdump -i eth0的命令进行监听，可以看到信息从Tunnel通道中已经传输了，HostV收到了信息也回应了，但是在HostU容器中却看不到回复信息。

<center><img src="../assets/2-5.png" width = 600></center>

提示：执行命令前请给server.py和client.py通过chmod a+x 文件名  加权限。


## 4. 配置Tunnel的双向通道

&emsp;&emsp;在任务3中我们可以看到，HostU可以成功地向HostV发送消息，但是却收不到HostV回复的消息，这个问题在本次任务中解决。

    # We assume that sock and tun file descriptors have already been created.
    while True:
    # this will block until at least one interface is ready
        ready, _, _ = select.select([sock, tun], [], [])
        for fd in ready:
            if fd is sock:
                data, (ip, port) = sock.recvfrom(2048)
                pkt = IP(data)
                print("From socket <==: {} --> {}".format(pkt.src, pkt.dst))
                ... (code needs to be added by students) ...
    
            if fd is tun:
                packet = os.read(tun, 2048)
                pkt = IP(packet)
                print("From tun ==>: {} --> {}".format(pkt.src, pkt.dst))
                ... (code needs to be added by students) ...

&emsp;&emsp;启动VPN服务器和HostU客户端的tun接口后（./tun_server_select.py和./tun_client_select.py），在从HostU客户端ping HostV，就可以得到回应了，整个Tunnel就通了。

<center><img src="../assets/2-6.png" width = 600></center>

## 5. 观察Tunnel短暂中断发生的情况

&emsp;&emsp;完成任务4，Tunnel两边就通了，现在我们从HostU客户端telnet HostV，用户名密码是seed/dees。完成登录后请暂停下VPN服务器和HostU客户端的连接，在HostU客户当已经telnet的界面输入一些命令，然后立即再通过./tun_server_select.py和./tun_client_select.py。启动后观察telnet的界面出现的内容，思考下为什么？
