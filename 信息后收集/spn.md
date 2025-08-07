# SPN技术

## 信息收集

信息收集----前况：--------为横向移动打基础

要注意拿到权限主机当前的角色（内网域用户还是普通）和当前区域，dmz区域：防火墙太严格会影响正常业务，太简单会有风险，所以有一种中间区域就是dmz区域。   

1、先判断内网是工作组（局域网）还是域内，直接执行域内的命令。不是域内还是工作组                                           （内网：查ip，看内网有没有ip存活等特征判断是不是内网----------ipconfig有多个内网ip就是） arp -a ：发现同网段其他ip，则为内网。

![1732769111424](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942113.png)

域内命令，执行没有，则就是在工作组中，（局域网）：

![1732769201341](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942114.png)

  本机上查看dns      ipconfig /all 判断存在域-dns 有域的有dns后缀，无域的无dns后缀

![1732772984451](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942115.png)

在域内主机写ipconfig /all  有Dns后缀显示

![1732773151130](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942116.png)

net time /domain 获取主域名，其实这个就是主域的计算机名，再通过nslookup或ping命令来获取主域的IP地址。定位DC

![1732773435662](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942117.png)

3、定性被控机     

  setspn,         spn技术 获取域内成员主机和基本服务。SPN 可以解决防火墙阻止的，SPN通讯时防火墙是放行的（白名单）。还可以通过-setspn技术来获取。

![1732776076841](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942118.png)

  不建议用工具自带的扫描工具，因为需要上传流量太大。建议还是socks代理然后本地fscan扫描。tasklist /svc >> 1.txt   显示本地或远程计算机上当前正在运行的进程并列表列出每个进程所提供的服务。在1.txt上：

![1732777087751](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942119.png)

AVCheck  ，项目对杀软进行识别并快速发现杀软：

python AVCheck.py 1.txt        <有报错，文件编码问题>            后面还可以用在线杀软识别。

  HackBrowserBata工具。获取浏览器数据-----主要针对个人用机。密码收集：								https://mp.weixin.qq.com/s/SDu4rw35-Atu5fiaNhwqrA

SharpDecryptPwd：获取NavicatCrypto上的数据账户密码

![1732778527329](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942120.png)
