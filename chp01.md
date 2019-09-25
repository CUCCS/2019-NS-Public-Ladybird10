# 实验总览

完成以下网络连通性测试；

- [x] 靶机可以直接访问攻击者主机
- [ ] 攻击者主机无法直接访问靶机
- [x] 网关可以直接访问攻击者主机和靶机
- [x] 靶机的所有对外上下行流量必须经过网关
- [x] 所有节点均可以访问互联网



虚拟机网络拓扑

<img src="pic\1.JPG" alt="1569415073712" style="zoom:50%;" />

# 多重加载

## 实验截图

<img src="pic\chp01_multi.JPG" alt="多重加载" style="zoom:50%;" />

## 注意事项

1. 删除备份才可以设置多重加载

2. 步骤：管理->虚拟介质管理->类型改为多重加载

   ​		多重加载后原来的会被释放，重新进入设置-存储-添加虚拟硬盘 中重新加载vdi文件

# 搭建虚拟机网络拓扑

## 实验截图

### 虚拟机网络构成

<img src="pic\chp01_topol.JPG" style="zoom:50%;" />

### 靶机可以直接访问攻击者主机

<img src="pic\1569244626019.png" alt="靶机访问攻击者" style="zoom:50%;" />

### 攻击者主机无法直接访问靶机

<img src="pic\1569245835025.png" alt="攻击者ping靶机" style="zoom:50%;" />

<img src="pic\1569335007897.png" alt='攻击者ping网关' style="zoom:50%;" />

* 证明过程
  1. 攻击者主机无法ping通靶机
  2. 攻击者主机能够ping网关（实验发现并不能）
* 预想是攻击者主机能够ping到网关，但是到达不了靶机，来证明攻击者主机无法直接访问靶机。事实证明好像不太行。

### 网关可以直接访问攻击者主机和靶机

<img src="pic\1569245589537.png" alt="网关访问攻击者和靶机" style="zoom:50%;" />

* 访问XP时要关闭防火墙才可以

### 靶机的所有上下行流量必须经过网关

<img src="pic\1569247047346.png" alt="网关地址截图" style="zoom:50%;" />

* 网关地址截图

<img src="pic\1569249058222.png" alt="靶机地址截图" style="zoom:50%;" />

* 靶机地址截图

<img src="pic\1569249854997.png" alt="流量监测a" style="zoom:50%;" />

<img src="pic\1569249761873.png" alt="流量监测b" style="zoom:50%;" />

* 关键步骤
  1. 在第一次ping完淘宝后，要将arp中的记录清除掉，以便清除淘宝服务器留下的IP地址和物理地址缓存。清除后我们可以更清楚的看到靶机的广播过程。
  2. Debian无法使用ifconfig指令时，经同学提醒配置apt install net-tools。

### 验证所有节点均可访问互联网

* Gateway-Debian

<img src="pic\chp01_GateNet.JPG" alt="网关联网" style="zoom:50%;" />

* Kali-Attacker

  <img src="pic\1569162446116.png" alt="攻击者联网" style="zoom:50%;" />

  1. 该步遇到的问题

     ​	攻击者主机无法上网

  2. 解决方案

     ​	按照老师所给的方法输入代码，第一遍失败。重启Virtual Box，重装攻击者主机，重新输入代码，成功。

* Victim-Kali

<img src="pic\1569333377134.png" alt="受害kali联网" style="zoom:50%;" />

* XP

<img src="pic\1569242772274.png" alt="XP联网" style="zoom:50%;" />

​		注意事项：刚开始只有一个内网能上网，一个不可以。后						得知网关网卡的配置要严格按照一定的顺序，否则内网						无法对应。

* Victim-Debian

<img src="pic\1569333326005.png" alt="受害debian联网" style="zoom:50%;" />

