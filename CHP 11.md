

# CHP 11

## 实验目的

- 了解蜜罐的分类和基本原理
- 了解不同类型蜜罐的适用场合
- 掌握常见蜜罐的搭建和使用

## 实验环境

- 从paralax/awesome-honeypots中选择 1 种低交互蜜罐和 1 种中等交互蜜罐进行搭建实验
- 推荐 `SSH` 蜜罐

## 实验要求

- [x] 记录蜜罐的详细搭建过程;
- [x] 使用 `nmap` 扫描搭建好的蜜罐并分析扫描结果，同时分析「 `nmap` 扫描期间」蜜罐上记录得到的信息；
- [x] 如何辨别当前目标是一个「蜜罐」？以自己搭建的蜜罐为例进行说明；
- [ ] （可选）总结常见的蜜罐识别和检测方法；
- [ ] （可选）基于 [canarytokens](https://github.com/thinkst/canarytokens) 搭建蜜信实验环境进行自由探索型实验；

## 实验过程

### SSH蜜罐

* 搭建环境

1. 修改端口号  

   ```shell
   sudo vi /etc/ssh/sshd_config 
   ```

   <img src="chp11\1.png" alt="1" style="zoom:80%;" />

2. 重启ssh

```shell
sudo service ssh restart
```

<img src="chp11\2.png" alt="2" style="zoom:80%;" />

3. 安装libssh和libjson-c-dev

<img src="chp11\3.png" alt="3" style="zoom:80%;" />

4. 下载安装蜜罐

```shell
git clone https://github.com/droberson/ssh-honeypot.git
cd ssh-honeypot
# 安装libssh-dev库
sudo apt install libssh-dev
make
ssh-keygen -t rsa -f ./ssh-honeypot.rsa
# -l mylog.log选择日志文件，-u cowrie以刚才创建的cowrie身份运行，-d作为守护进程运行
bin/ssh-honeypot -r ./ssh-honeypot.rsa -p 22 -l mylog.log -f my.pid -u cowrie -d
```

<img src="chp11\4.png" alt="4" style="zoom:80%;" />

5. 此时查看ip地址时，发现两台主机ip地址相同，为避免后续出现攻击者找不到目标主机情况，重新设置网卡为NAT

<img src="chp11\5.png" alt="5" style="zoom:80%;" />

修改后，攻击者IP为10.0.2.5，受害者IP为10.0.2.15

重启虚拟机后，开启蜜罐的一系列操作也重来一遍

<img src="chp11\6.png" alt="6" style="zoom:80%;" />

* 蜜罐实验

1. 受害者开启22端口监听

```shell
bin/ssh-honeypot -p 22
```

攻击者连接受害者，受害者实时记录下攻击者的IP。攻击者无论输入什么密码都无法登陆

<img src="chp11\7.png" alt="7" style="zoom:80%;" />

2. 攻击者进行nmap扫描,发现这种蜜罐并不能发现攻击者的nmap扫描行为

```shell
nmap -p 22.10.0.2.15
```



<img src="chp11\8.png" alt="8" style="zoom:80%;" />

### cowrie蜜罐

1. 安装并启动docker

```shell
apt-get update
apt-get install docker docker-compose
service docker start
```

<img src="chp11\9.png" alt="9" style="zoom:80%;" />

2. cowrie蜜罐环境搭建

```shell
docker pull cowrie/cowrie
docker run -p 2222:2222 cowrie/cowrie #启用蜜罐
```

<img src="chp11\10.png" alt="10" style="zoom:80%;" />

<img src="chp11\11.png" alt="11" style="zoom:80%;" />

3. 攻击者连接受害者

```shell
ssh root@10.0.2.15 -p 2222
```

<img src="chp11\12.png" alt="12" style="zoom:80%;" />

执行id，cat /etc/passwd命令，会看到所有命令都被记录下来，而且查看到了对应的结果

<img src="chp11\14.png" alt="14" style="zoom:80%;" />

<img src="chp11\13.png" alt="13" style="zoom:80%;" />

```shell
nmap -p 2222 10.0.2.15
```

扫描端口2222

<img src="chp11\15.png" alt="15" style="zoom:80%;" />

```shell
nmap -sV 10.0.2.15
```

用此命令扫描端口并不能成功

<img src="chp11\16.png" alt="16" style="zoom:80%;" />



## 实验结果

可以发现，对于SSH-honeypot来说，无论使用什么口令登陆，都会被拒绝，是出于拒绝被暴力攻击的角度来设计的。记录下来的信息并不是非常完备。连接还有时间限制，100多秒不进行操作的话连接就lost了，对于正常非蜜罐连接来说，是不会出现这种情况的。

而cowrie更像是诱导攻击者来攻击，所以无论什么口令都可以登陆，而且攻击者能够查看到一些想要看的信息。记录下来的信息相比ssh-honeypot要更完备一些，包括攻击者使用了什么指令。

