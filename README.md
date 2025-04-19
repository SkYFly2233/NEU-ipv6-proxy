# 东北大学校园网免流全新教程

### 免流介绍

![image-20250419212113105](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212113105.png)

由于之前的教程已经彻底亡了（我明明用了快一年，但写出教程一个月就不行了），因此出个新的教程。

 

免流顾名思义，就是通过校园网ipv6免费的特性让所有流量都通过ipv6，这样不使用ipv4自然就不会产生费用。

 

之前的方案不能用了，主要在于国内用不了cloudflare的服务器了，因此这次的免流方案的最主要目标就是解决免费的服务器，而通过GitHub学生认证之后刚好可以获得DigitalOcean的1年200刀的学生优惠额度，就可以很爽的用这200刀买服务器了，因此本教程是部署于DigitalOcean，基于hysteria2协议，运用虚拟专用网原理实现的东北大学校园网免流教程。

 

想要百分百的白嫖DigitalOcean条件还是比较苛刻的，如果不能白嫖，还可以购买cloudcone、racknerd等运营商的服务器，他们的服务器也很划算，在黑五，圣诞，等各种节日促销中，能低至9.9刀一年的价格，并且也能有1T起步的月流量，量大也不贵，宿舍合租也很实惠。但是cloudcone、racknerd的服务器都在美国，因此延迟较DigitalOcean较高，速度较慢，但速度也同样能有400左右，用起来依旧也非常爽。

### 准备环节&前情提要

申请github学生认证、申请全币种信用卡等操作可能比较繁琐，但这些步骤都是白嫖免费的DigitalOcean vps所需要的，如果您在以上环节有卡住或者白嫖的一年DigitalOcean vps已经到期，那么可以使用购买cloudcone、racknerd的服务器实现免流，这两家的服务器在节日打折时可以低至10刀一年，相当于1刀一月，相比校园网来已经足够便宜，且流量大。如果不进行白嫖DigitalOcean，可以直接跳到第4步进行查看。

### 1. 申请github学生认证

[Github学生认证及学生包保姆级申请指南 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/578964972)

目前申请的方法教程很多，自行申请即可，不过目前申请条件越来越严格，要尽早申请。

![image-20250419212301055](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212301055.png)

### 2. 使用DigitalOcean学生认证

github的学生包有很多东西，最有用的当然是copilot，极大方便写代码。当然除此之外，还有DigitalOcean，其中就包括了1年200刀的免费额度（普通注册也有200刀，但是限时60天），比亚马逊还大度。

![image-20250419212328702](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212328702.png)

使用github登录即可

![image-20250419212335456](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212335456.png)

初次登录会提示需要绑定支付方式，有两种方法：1.如果使用信用卡，需要使用国外信用卡或者国内的全币种的信用卡，大部分人是没有的，因此比较麻烦；2.如果使用PayPal，使用会很简单，PayPal很容易注册，绑定银联银行卡即可，但是使用PayPal要求必须初始要支付5刀。

![image-20250419212343223](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212343223.png)

登录进去在在billing里就可以看到已经充值的钱（如果用PayPal的话）

![image-20250419212349452](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212349452.png)

划到最下面，就能看到github的200刀，当购买服务器的时候就会优先使用这200刀。

![image-20250419212355328](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212355328.png)

### 3. 创建vps实例

![image-20250419212409084](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212409084.png)

具体设置如下即可

国家选择美国，新加坡等都可以，实测新加坡网速最快

系统选ubuntu

类型选basic

cpu型号可以任选，但是最初我在regular上没有成功使用ipv6，premium amd、premium intel没问题

选择适当的配置，免流不吃配置因此一般选择最烂的cpu、内存即可，免流流量自选，一般1T十分充裕，但是更多也可以

设置密码

最后最重要的，**在高级选项里面开启IPV6，在高级选项里面开启IPV6，在高级选项里面开启IPV6**，没有ipv6怎么免流？

![image-20250419212516193](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212516193.png)

### 4. 设置服务器架设代理

当DigitalOcean的服务器创建好之后，控制面板大概是这样的。

![image-20250419212536281](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212536281.png)

这里可以先复制ipv6地址在cmd中用"ping -t ip地址"的方式先测一下延迟丢包率如何，延迟太高丢包太高或者根本连不上的情况建议换地区或者重开服务器。

![image-20250419212542683](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212542683.png)

没问题的话就用finalshell连服务器

![image-20250419212548673](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212548673.png)

接下来进行搭建：首先更新组件

```
apt update -y
apt install curl sudo -y
```

安装一键安装脚本

```
wget -N --no-check-certificate https://raw.githubusercontent.com/flame1ce/hysteria2-install/main/hysteria2-install-main/hy2/hysteria.sh && bash hysteria.sh
```

选1安装

![image-20250419212703120](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212703120.png)

选1自签证书

![image-20250419212725751](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212725751.png)

填写端口号

![image-20250419212733414](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212733414.png)

默认就行

![image-20250419212740023](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212740023.png)

密码，随便填，忘了也没事，足够长就行

![image-20250419212747281](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212747281.png)

混淆的网址，默认也行，或者就bing

![image-20250419212755706](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212755706.png)

后就配置完了，会生成配置文件

![image-20250419212802657](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212802657.png)

这就是我们的配置信息了，如果你忘记了配置信息，可以用

```
nano /root/hy/hy-client.yaml
```

的指令来重新查看。

重新打开就像这样：

![image-20250419212842701](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212842701.png)

### 5.配置代理文件

接下来就需要小猫咪的软件了，可以在这里下载。

[Releases · clash-verge-rev/clash-verge-rev](https://github.com/Clash-Verge-rev/clash-verge-rev/releases)

虽然服务器直接给我们生成了yaml文件，但是并不能在clash中直接用，还需要配置一些信息。

下面是一个通用的配置文件，根据下面的提示来更改一下。



新建一个txt文本文件，后缀改成yaml

![image-20250419212933378](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419212933378.png)

打开将下面的信息复制到文本里面，并且更改server，port，password三行。

```
proxies:
  - name: Hysteria2
    type: hysteria2
    server: 这里填服务器的ipv6地址，填ipv4的话还怎么免流？
    port: 这里填设置成的端口号，上面用的11451
    password: 这里填密码，也就是上面hy-client.yaml中的的auth的一行内容
    sni: www.bing.com
    skip-cert-verify: true
    hopinterval: 30
    fast-open: true
    udp: true
    recv-window-conn: 33554432
    recv-window: 16777216

proxy-groups:
  - name: PROXY
    type: select
    proxies:
      - Hysteria2

rules:
  - IP-CIDR6,::/0,DIRECT,no-resolve
  - MATCH,PROXY
```

如果你想要屏蔽掉校园内网的话，可以自己修改规则，这样可以更方便的使用校园网，东北大学的校园网规则如下：

```
rules:
  - IP-CIDR6,::/0,DIRECT,no-resolve
  # 新增的直连IP规则（IPv4）
  - IP-CIDR,202.118.0.0/19,DIRECT
  - IP-CIDR,202.199.0.0/20,DIRECT
  - IP-CIDR,210.30.192.0/20,DIRECT
  - IP-CIDR,219.216.64.0/18,DIRECT
  - IP-CIDR,58.154.160.0/19,DIRECT
  - IP-CIDR,58.154.192.0/18,DIRECT
  - IP-CIDR,118.202.0.0/19,DIRECT
  - IP-CIDR,118.202.32.0/20,DIRECT
  - IP-CIDR,172.16.0.0/12,DIRECT
  - IP-CIDR,100.64.0.0/10,DIRECT
  - IP-CIDR,192.168.1.1/24,DIRECT
  # 兜底规则（必须放在最后）
  - MATCH,PROXY
```

更改完应该是一下的样子，其中这三行填的是你的服务器的信息

![image-20250419213020702](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419213020702.png)

ipv6地址在哪找呢？在DigitalOcean的面板里面就有

![image-20250419213207630](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419213207630.png)

###  6.设置代理软件

这里以clash verge为例。

首先配置一下clash verge的基本设置，按需配置即可。

![image-20250419213307416](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419213307416.png)

在订阅里面直接 拖入刚刚写好的yaml文件，就可以在这里面看到我们订阅了

![image-20250419213317090](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419213317090.png)

之后，在代理里面就能看到我们的服务器，上文我们在配置yaml的时候只用了一个最简单的规则，只有一个服务器，因此在这里面也只有一个，我们可以在这里面测一下延迟

![image-20250419213326386](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419213326386.png)

最后，使用时只需要在右下角的图标处右键快速设置即可，这里我们使用规则模式，规则就是我们写的除了ipv6和校园网内网都走代理的规则，需要进行免流时就打开系统代理即可。

另外，默认情况下只有浏览器等少数软件可以免流，**如果需要电脑的所有软件都进行免流，选择TUN模式即可**，这样就连命令行都走的免流，妈妈再也不用担心我下载python包超时啦。

![image-20250419213346963](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20250419213346963.png)

现在你已经学会校园网免流了，赶紧去试试吧。
