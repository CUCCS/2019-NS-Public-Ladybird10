# 实验总览

- [x] 完成以下扫描技术的编程实现
  - TCP connect scan / TCP stealth scan
  - TCP Xmas scan / TCP fin scan / TCP null scan
  - UDP scan
- [x] 上述每种扫描技术的实现测试均需要测试端口状态为：`开放`、`关闭` 和 `过滤` 状态时的程序执行结果
- [x] 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因；
- [x] 在实验报告中详细说明实验网络环境拓扑、被测试 IP 的端口状态是如何模拟的
- [ ] （可选）复刻 `nmap` 的上述扫描技术实现的命令行参数开关

# 环境搭建

* 拓扑：kali-victim-1、kali-victim-2、gateway共处内网intnet1中

* 要将该网卡打开，否则会默认连接host only网卡，拒绝内网连接。

<img src="chp5_pic\openeth0.png" alt="openeth0" style="zoom:50%;" />

* 两个受害者的网络情况

<img src="chp5_pic\victims.png" alt="victims" style="zoom:50%;" />

* 受害者之间互相访问

<img src="chp5_pic\visit_victims.png" alt="visit_victims" style="zoom:50%;" />

# 端口状态查看

80端口开放

<img src="chp5_pic\port80.png" alt="port80" style="zoom:70%;" />

验证

 <img src="chp5_pic\port80_2.png" alt="port80_2" style="zoom:50%;" />

可以有正确的返回值

可用端口列举

 <img src="chp5_pic\OpenPort.png" alt="OpenPort" style="zoom:70%;" />

# 扫描

## TCP Connection Scan

* 代码

```python
from scapy.all import *
import getopt
import sys

def scan(argv):
    opts, args = getopt.getopt(argv, "-h:")
    for opt,arg in opts:
        if opt in ("-h"):
            host=arg
    all_port=[80]

    for port in all_port:
        send=sr1(IP(dst=host)/TCP(dport=port,flags="S"),timeout=2,verbose=0)
        if (send is None):
            print ("[+] %s %d \033[91m Closed \033[0m") % (host,port)
        elif send.haslayer("TCP"):
            if send["TCP"].flags == "SA":
                send_1 = sr1(IP(dst=host) / TCP(dport=port, flags="AR"), timeout=2, verbose=0)
                print ("[+] %s %d \033[92m Open \033[0m") % (host, port)
            elif send["TCP"].flags == "RA":
                print ("[+] %s %d \033[91m Closed \033[0m") % (host,port)
if __name__=="__main__":
    scan(sys.argv[1:])
```

<img src="chp5_pic\TCP_con2.png" alt="tcpcon2"  />

* 开放

  * <img src="chp5_pic\TCP_con.png" alt="TCPCON" style="zoom:50%;" />

* 关闭

  * <img src="chp5_pic\TCP_con3.png" alt="tcpcon3" style="zoom:50%;" />

* 过滤

  * 安装ufw，设置拒绝80端口的tcp包

  <img src="chp5_pic\tcpfilcon1.png" alt="tcpfilcon1" style="zoom:70%;" />

  * 可见80端口已被关闭

  <img src="chp5_pic\tcpfilcon2.png" alt="tcpfilcon2" style="zoom:70%;" />

  * 观察cap文件

  <img src="chp5_pic\tcpfilcon3.png" alt="tcpfilcon" style="zoom:70%;" />

* 参考资料

>  客户端与服务器建立 TCP 连接要进行一次三次握手，如果进行了一次成功的三次握手，则说明端口开放 。
>
> <img src="chp5_pic\ref1.png" alt="ref1" style="zoom:50%;" />
>
> <img src="chp5_pic\ref1_2.png" alt="ref1.2" style="zoom:50%;" />

## TCP Null Scan

* 代码

```python
from scapy.all import *
import getopt
import sys

def scan(argv):
    opts, args = getopt.getopt(argv, "-h:")
    for opt,arg in opts:
        if opt in ("-h"):
            host=arg
    all_port=[80]

    for port in all_port:
        send=sr1(IP(dst=host)/TCP(dport=port,flags=""),timeout=2,verbose=0)
        if (send is None):
            print ("[+] %s  %d \033[91m Open | filtered\033[0m")%(host,port)
        elif send.haslayer("TCP"):
            if send["TCP"].flags=="RA":
                print ("[+] %s %d \033[92m Closed \033[0m")%(host, port)

if __name__=="__main__":
    scan(sys.argv[1:])
```

![Tcp_nu2](chp5_pic\TCP_NU2.png)

* 开放状态
  * <img src="chp5_pic\TCP_NU.png" alt="tcp_nu" style="zoom:50%;" />

* 关闭状态
  * <img src="chp5_pic\TCPNU3.png" alt="TCPNU3" style="zoom:50%;" />
* 过滤
  * <img src="chp5_pic\TCPNU4.png" alt="TCPNU4" style="zoom:50%;" />
  * 抓无数遍都没有ICMP包...明明80端口都用防火墙deny了
* 参考资料

>  在空扫描中，客户端发出的 TCP 数据包仅仅只会包含端口号而不会有其他任何的标识信息。如果目标端口是开放的则不会回复任何信息。 
>
> <img src="chp5_pic\ref2_1.png" alt="ref2.1" style="zoom:50%;" />
>
> 如果服务器返回了一个 RST 数据包，则说明目标端口是关闭的。 
>
> <img src="chp5_pic\ref2_2.png" alt="ref2.2" style="zoom:50%;" />
>
>  如果返回 ICMP 错误类型3且代码为1，2，3，9，10或13的数据包，则说明端口被服务器过滤了。 
>
> <img src="chp5_pic\ref2_3.png" alt="ref2.3" style="zoom:50%;" />

## UDP Scan

* 代码

  ```python
  #!/usr/bin/python
  # -*- coding:UTF-8 -*-
  import logging
  logging.getLogger("scapy.runtime").setLevel(logging.ERROR)
  from scapy.all import *
  import time
  import sys
    
  ip = "172.16.111.130"
  port = 80
   
  a = sr1(IP(dst=ip)/UDP(dport=port),timeout=5,verbose=0)
  time.sleep(1)
  if a == None:
      print (port)
  else:
      pass
  ```

* 开放
  
* <img src="chp5_pic\UDP.png" alt="UPD" style="zoom:50%;" />
  
* 关闭
  
* <img src="chp5_pic\UDP_close.png" alt="udp_close" style="zoom:50%;" />
  
* 过滤
  * <img src="chp5_pic\UDPfli.png" alt="udpfil" style="zoom:50%;" />
  * haha，明明也把68端口用防火墙关掉了，为什么没有ICMP的包呢。哈哈，我不活啦
* 参考资料

> TCP 是面向连接的协议，而UDP则是无连接的协议。
>
> 面向连接的协议会先在客户端和服务器之间建立通信信道，然后才会开始传输数据。如果客户端和服务器之间没有建立通信信道，则不会有任何产生任何通信数据。
>
> 无连接的协议则不会事先建立客户端和服务器之间的通信信道，只要客户端到服务器存在可用信道，就会假设目标是可达的然后向对方发送数据。
>
> <img src="chp5_pic\ref3_1.png" alt="ref3.1" style="zoom:50%;" />
>
> <img src="chp5_pic\ref3_2.png" alt="ref3.2" style="zoom:50%;" />
>
> <img src="chp5_pic\ref3_3.png" alt="ref3.3" style="zoom:50%;" />