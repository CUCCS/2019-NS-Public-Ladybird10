# Chp 7

## 搭建环境

搭建webgoat实验环境

* 从git仓库clone文件
* apt install docker-compose
* 执行docker-compose up -d

docker ps结果如图

<img src="chp7_pic\port.png" alt="port"  />

* 每次重启虚拟机都会遇到问题，经搜索发现是权限问题。采用sudo service docker restart重启docker即可

<img src="chp7_pic\problem1.png" alt="problem1" style="zoom:50%;" />



## WebGoat环境



### 配置WebGoat

以便用burp suit拦截请求

<img src="chp7_pic\set.png" alt="set" style="zoom:50%;" />



### 命令注入

* Numeric SQL Injection

直接GO，Burp拦截

<img src="chp7_pic\21.png" alt="21" style="zoom:50%;" />

修改101为101 or 1=1

<img src="chp7_pic\22.png" alt="22" style="zoom:80%;" />

成功

<img src="chp7_pic\23.png" alt="23" style="zoom:80%;" />



### 未验证的用户输入

* Parameter Tampering->Bypass HTML Field Restrictions

直接submit提交，可看到burp中无法拦截到disabled输入

<img src="chp7_pic\11.png" alt="11"  />

此时我们删除input disabled value中的disabled，重新提交请求。发现能够抓到disabledinput了。

<img src="chp7_pic\12.png" alt="12" style="zoom:80%;" />

任意修改值为非法输入，再forward返回给浏览器。成功。

<img src="chp7_pic\13.png" alt="13" />



### XXE注入

* XML External Entity (XXE)

输入BOS，burp拦截

<img src="chp7_pic\31.png" alt="31" style="zoom:80%;" />

<img src="chp7_pic\32.png" alt="32" style="zoom:80%;" />

<img src="chp7_pic\33.png" alt="33" style="zoom:80%;" />



### 跨站脚本攻击

* Cross Site Scripting Stage 1

<img src="chp7_pic\41.png" alt="41" style="zoom:80%;" />

<img src="chp7_pic\42.png" alt="42" style="zoom:80%;" />

<img src="chp7_pic\43.png" alt="43" style="zoom:80%;" />



### 拒绝服务攻击

* Denial of Service from Multiple Logins

<img src="chp7_pic\51.png" alt="51" style="zoom:80%;" />

<img src="chp7_pic\52.png" alt="52" style="zoom:80%;" />

在下方框中登陆jsnow，同样的方法再打开两个标签页登陆，实验即可成功

<img src="chp7_pic\53.png" alt="53" style="zoom:80%;" />



## Juice Shop环境

### 环境搭建

在juice shop目录下，执行

```bash
docker-compose up -d
docker ps
```

<img src="chp7_pic\juiceport.png" alt="juiceport" style="zoom:80%;" />



### SQL注入

依据网络教程，在登录框输入sql语句 ‘ or 1=1失败，无法成功登陆

后问同学，得知猜测管理员邮箱可登入，故输入admin@juice-sh.op'--，成功登入

<img src="chp7_pic\61.png" alt="61" style="zoom:80%;" />

### 脆弱的访问控制

<img src="chp7_pic\71.png" alt="71" style="zoom:80%;" />

<img src="chp7_pic\72.png" alt="72" style="zoom:80%;" />

<img src="chp7_pic\73.png" alt="73" style="zoom:80%;" />



### 未验证的用户输入

<img src="chp7_pic\81.png" alt="81" style="zoom:80%;" />

<img src="chp7_pic\82.png" alt="82" style="zoom:80%;" />



### XSS

* 最初尝试在搜索框输入

  ```html
  <script>alert("XSS")</script>
  ```

  失败。并不如所想的弹出对话框

  尝试<<a|ascript>alert("XSS")</script>可以回显alert（“XSS”）了，但还是不弹窗

<img src="chp7_pic\91.png" alt="91" style="zoom:80%;" />

转换其他地方，比如账户页面的修改用户名处（进入profile界面仿佛花了100年一样久=_=）

在设置用户名处输入

```html
<<a|ascript>("XSS")</script>
```

(设置用户名的时间仿佛也花了100年一样久)

<img src="chp7_pic\92.png" alt="92" style="zoom:80%;" />

<img src="chp7_pic\93.png" alt="93" style="zoom:80%;" />



### 登陆他人账户

构造万能登陆口令

<img src="chp7_pic\101.png" alt="101" style="zoom:80%;" />

<img src="chp7_pic\102.png" alt="102" style="zoom:80%;" />

可见网站对账户安全性的保护如此薄弱