# 工作组环境

 针对的是工作组和局域网下的环境，非域环境。攻击主要是针对网络攻击，与目标是什么操作系统无关

内网：域环境，工作组

中间人攻击：SSLStrip攻击、ARP欺骗攻击、SSL劫持攻击。。。。

网络环境：

![1734085220532](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942330.png)

##### arp：

  	火绒---139.139       win7 ----  139.2         360 ----139.131       kali        ping后获取到ip和Mac地址，在火绒上用科来对目标win7 发送错误的mac地址，使目标ip的mac地址改为自己已经设置的，这样目标win7  就没有网络了。这就是单向欺骗。所以一般需要将ip和mac地址绑定，这样mac地址就为静态了，攻击网络就不会发生没网的情况了。这就是骗ip我是你的网关（mac地址）

网关地址：192.168.139.2                 目标机work----139.129    目标机360：139.132

因为这里是动态的，所以就先用单向欺骗：

![1734095544195](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942331.png)

科莱-------单向欺骗：

![1734096865321](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942332.png)

目标机反应：

![1734096707202](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942333.png)

断网：

![1734096812638](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942334.png)



  双向欺骗----------既骗ip我是你的网关，又骗网关我是你的ip。----------         断网

![1734097484022](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942335.png)

用自己的mac地址伪造成360的ip

![1734098013483](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942336.png)

用别的ip伪造mac地址

![1734098129871](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942337.png)

开始：

![1734098219907](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942338.png)

  确保本机（攻击机）和目标机（手机）在同一个局域网下。科莱正常来说是抓不到手机的流量的。需要用工具来抓手机的流量------Arpspoof-----需要用管理员运行 ---------- 先过去目标网段。自动执行双向欺骗，充当一个中间人给双方发送流量，----------Arpspoof运行后，本机科莱就可以监听到手机流量了

这里搞不了，电脑内存跟不上科莱的节奏

![1734098769098](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942339.png)

管理员运行，不过就是arpspoof.exe 手机ip 就可以在科莱中监控到手机流量

不会影响目标网络通讯

##### dns欺骗：

 可能会引起目标网络不稳定，所以与arp比起来还是差点意思ettercap------kali里的ettercap，这里还需要修改一个文件，* A ip   修改解析。然后这里修改文件里还有个插件----就是dns欺骗。实现就是域名相同，但是打开的页面是自己显示出的页面。会掉完。

在内网域中用不了是因为内网域是由DC操控的，都听DC的，不会被骗

![1734099054574](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942340.png)

修改这个文件：

![1734100464710](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942341.png)

配置文件

![1734100530994](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942342.png)

后面即可开始攻击

![1734100606325](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942343.png)

![1734100621085](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942344.png)

使用插件里面的这个进行钓鱼

![1734100654647](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942345.png)
