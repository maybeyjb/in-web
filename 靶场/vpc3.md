# VPC3



好像又可以不用复现了，VPC4和这个类似。

linux+windows------DF       这节课的所有后门都是msf生成的，自然也都是msf监听

![1734765222666](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942586.png)

  ubuntu系统关防火墙       redies攻防：从web写shell、计划任务、写公密钥进去登录

##### linux:

这里目标（1.5）存在redies未授权

![1734750363243](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942587.png)

这里的用的redies服务的方法为：写公密钥进去（没有web，计划任务权限不够）

![1734750642802](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942588.png)

就是将目标  1.5 的连接密钥换成自己的公钥：

![1734750888015](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942589.png)

写入后登录就不需要密码验证了：

![1734750985792](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942590.png)

这里拿到ssh权限后让c2上线，方便代理和操作，这里因为是linux ,所以c2选择msf、（cs上线linux需要插件，而且比较麻烦）这里的方法就是生成http后门,让1.5远程下载：

![1734751361410](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942591.png)

成功上线后还是加个路由----------和52可以通讯，代理----然后就可以实现通讯52.10、52.20、得到后在1.5主机上找历史命令发现连接过52.20、

![1734753057777](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942592.png)

![1734751692893](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942593.png)

拿到52.20权限后，发现此主机上面还有93网段（ifconfig），那么就得和93取得路由通讯，所以要先让52.20上线msf，在用路由和93网段建立通讯---------怎么让52上线呢：这里就是借助已经拿到的52.10进而拿52.20权限路由，采用正向后门。将正向后门上传到52.20：1、先上传到52.10上，在52.10开一个web服务，让52.20下载。2、上传到52.10后，利用scp复制到52.20。

![1734752649936](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942594.png)

执行正向后门2.elf后，在用msf连接，这里注意：msf已经拿到了52网段的路由了，所以可以直接和52.20通讯，这里因为端口占用，换了个后门：6001.elf。小翻车----------

![1734753258940](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942595.png)

成功上线52.20机，然后获取52.20机器上的93路由：

![1734753371290](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942596.png)

##### windows：

windows下的，93网段。--------93的入口机器上有web服务，通过泄露源码，得到数据库文件，进而拿下入口主机：

![1734753582761](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942597.png)

-------windows的web服务，利用msf建立代理，本地工具proxy配置好后，访问目标：

![1734758104625](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942598.png)

找到一个rar压缩包，在里面找到mssql配置文件:

![1734758258335](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942599.png)

mssql就还是用MDUT这个工具进行操作，msf生成93.10的反向后门，在上传到93.128上，并执行：

![1734758591041](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942600.png)

msf监听，成功上线93.128：

![1734758885514](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942601.png)

记住，上线的第一件事情就是看还有没有别的路由，添加路由。然后看是不是域环境，是就获取域内的基本信息，          net   time  /domain    ping domain，然后这里是利用spn技术进行扫发现有exchange服务，

![1734759206383](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942602.png)

windows 2012以上默认关闭了注册表Wdigest，所以攻击者无法通过内存获取到明文密码，

但是这里是利用exchange服务，所以需要明文密码，就上传mimikate先抓取hash最后在破解hash得到明文密码：

![1734759967860](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942603.png)

用hashcat爆破，成功得到密码Admin12345、这里的速度取决于gpu图形处理器，用于并行计算：

![1734760094064](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942604.png)

下来就是exchange攻击实现目标DC远程下载msf的正向后门：先配好代理路由，3网段的，然后将后门（正向）放到3.73的web目录下，还有就是配好hosts文件、这里是域名要和对应的ip地址配好：

![1734760798068](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942605.png)

这里用的是cve的exp实现然后：

![1734761173267](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942606.png)

换到有ysoserial.exe的目录下：

![1734761218527](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942607.png)

得到payload后复制到刚才的cve脚本：

![1734761447337](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942608.png)

然后成功上传到3.144上，再用msf启动监听，成功上线3.144（路由通讯，自然能收到3.144执行正向后门的通讯）。这里执行后门流程和上面一样，还是执行后给你一个poc让你得到payload：

![1734761644018](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942609.png)

执行后正向后门，msf开启监听就成功上线3.144：

![1734761716701](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942610.png)

###### win10--DF：

上传mimikatz到3.144上获取明文密码：

![1734762146876](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942611.png)

得到DC明文密码就可以登录3.74主机，但是上面有DF,登录不代表就可以控制3.74主机。这里入口就是利用DC（3.144）的账户密码进行登录3.74进而操作：

![1734762399300](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942612.png)

这里执行远程下载后门，直接被杀了，，，所有需要免杀。这里是用powershell命令，msf生成，对生成的ps1文件进行混淆和加密，将后门5174.ps1换成5174s.ps1  --- ps1后缀为powershell命令的后缀:

![1734763337810](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942613.png)

然后将本地免杀的powershell后门放到刚才有web服务的机器3.73下，在让3.74远程下载即可。下图就先将本地生成的下载到3.73web机上：

这里注意：上传的是5074s.exe，因为upload不支持上传.ps1格式的，所有上传上去后在将exe改为ps1格式即可。

![1734763433467](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942614.png)

下来才是将3.73上的5074s.exe下载到3.74机上，这里执行命令就支持下载ps1，现在msf是没有3.74的shell的，3.74的shell是在本地机上，用cve获得的：

![1734763787370](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942615.png)

下来就执行ps1，然后msf监听好（注意模板设置好，payload是powershell的），成功上线：

![1734764026407](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942616.png)

##### cs：

这里演示cs上线linux，会有一些小问题，所以还是建议用msf上线Linux：

![1734764295579](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942617.png)

然后来到cs的服务端kali，将刚才生成的命令复制执行（在cs目录下），生成的out文件为反向后门：

![1734764405454](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942618.png)

将生成的out文件上传到目标linux上，这里是1.5的linux机器，成功上线：

![1734764586562](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942619.png)

然后在让下一台linux上线，生成正向后门命令：

![1734764674974](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942620.png)

在到cs服务端kali 执行正向后门命令，生成正向后门6001.out，在将这个正向后门上传到1.5的主机上，上传后在1.5上开一个web服务，让52.20linux主机远程下载正向后门6001.out：

![1734765067293](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942621.png)

但是这里用52.20的socks代理就有问题了，不知道为什么，反正就是通讯不了93网段，那么也就是后面的windows机器通讯不了，所以不建议采用linux上线CS，这里最多就上线这两天linux，除非你目标全是linux并且都可以不用socks代理解决通讯。



![1734765211673](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942622.png)

![1734761837973](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942623.png)
