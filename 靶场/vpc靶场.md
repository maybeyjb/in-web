# VPC2


  入口：tomcat后台上传后门，哥斯拉连接。

![1734701298138](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942549.png)

这种就是有域，但是没在域内或者权限不够------这里是权限（getsystem）：

![1734680689908](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942550.png)

下来就是扫网段， 扫端口，发现被控机（2.10）开放1433和3306、2.11机器开放了1433 ，所以这里猜测通过数据库控制2.11机，然后找2.10的数据库的配置文件（d ir /b /s web.config），得到账户密码。

![1734681816178](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942551.png)

找2.10的数据库的配置文件

![1734681719557](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942552.png)

 然后用数据库连接工具MDUT+Proxifier尝试连接2.11，然后就上传后门，（这里不知道有没有防火墙或者不知道是正向还是反向的），就ping 一下就行。

工具，启动jar包：

![1734681868582](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942553.png)

这里这工具启动时要注意配置文件：

![1734681974492](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942554.png)

判断防火墙，目标能通讯，但是ping不通，这里就用反向后门：

![1734682024832](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942555.png)

所以创建2.10的后门，上传（用刚才的mdut工具）让2.11主动连接：

![1734682154948](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942556.png)

上传后执行上线：

![1734682276310](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942557.png)

下来就是抓取（2.11）明文密码 ：

![1734684062780](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942558.png)

##### DC1

用获取的密码尝试打，（实战中肯定是测试每一个机器）、发现是打到了DC1（2.20）成功，域名是前面收集到的。这里DC1上部署了火绒：

![1734684907560](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942559.png)

但是被拦截：

![1734685000038](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942560.png)

横向失败，所以换手法，用套件里的工具，wmiexec.py（代理建立好，然后火绒也没有拦截）：

![1734685176696](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942561.png)

下来就是执行下载后门（后门需要免杀）命令、下载命令也需要处理：

先处理下载命令：

这里是将certutil.exe先copy成a.exe（因为他会对certutil.exe监控），然后在加一些标签最后成功绕过，成功实现下载文件。下载地址就是域内2.10web机上有web服务：

![1734688199471](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942562.png)

最后生成wy.exe，可以执行下载命令。这里就是本来是通过certutil.exe（自带的）这个来实现远程下载，但是被拦截，所以就一系列操作实现绕过。

后面免杀：

用异或、分离。火绒的免杀还是比较好做的，将后面bin文件和执行的exe（放在一起（反向）这里的dc.exe就是解密和执行生成的bin后门的）：

![1734687768601](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942563.png)

在webshell工具上先上传到2.10（2.10的权限已经通过哥斯拉拿到了），然后2.20在下载，这里的远程下载命令的wy.exe就是上面经过加签后的certutil.exe：

![1734687738624](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942564.png)

在执行后门成功上线。这下拿到DC了，后面拿2.12就容易了。

注意：实战中你的cs也得魔改，因为流量明显，会被内存查杀。这样就能持久。 			这里就发现可能存在cs，但是没有处理，因为你的动静不大，他不好判断你。

![1734688629570](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942565.png)

##### DC2:

又找到了3网段的信息：

![1734688467684](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942567.png)

这里工具没有扫出来，被拦截了。所以下来就是手工测，方法就不展示了，代理配好换工具扫，找到了3.11机器上存在通达OA的Web服务，这里用liqun梭哈，先cookie 在上shell:

![1734689125821](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942568.png)

这里上的后门是蚁剑的，所以用蚁剑连接：

![1734689237920](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942569.png)

这里2.20（3.10）有防火墙，所以上线3.11用正向，上传后门，connect：

![1734689533200](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942570.png)

然后抓取3.11密码，收集到DC2的信息、ip、域名。这里抓密码还是没有抓到明文，只有hash，所以还是采用置空的办法来打DC2：

![1734689753758](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942571.png)

这里至于为什么选这个漏洞：是先看条件满足哪些，然后才试出来的。回显正常才能使用的。

下面还是和上节课一样，先配置好域名的hosts，先配好代理，在kali 中进行利用Py工具置空DC2：

![1734690178908](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942572.png)

置空后用工具包里的py工具连接DC2（3.20）得到dc2的hash密码：

![1734690242325](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942573.png)

然后这里目标列表中没有3.20，所以先扫出来3.20，才能进行横向移动：

![1734690330142](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942574.png)

这里就是用psexec横向移动，这里因为3.20没有开防火墙，所以选择的是用3.11正向连接3.20，如果开了防火墙的话，这里就得先建立一个3.11的监听器，然后让3.20反向连接3.11，域名、密码就是上面py工具得到的hash：

![1734690513328](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942575.png)

补充一下上面的会话选的是下面的3.11会话：

![1734690700800](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942576.png)

如果开了防火墙的话，走的反向的话，就先在3.11（能和3.20通讯）建立转发上线的监听器，下来就和上面一样，只需要改监听器就行：

![1734690939857](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942577.png)

不过上面正向，域内没有成功，这里重试了，换成反向，走本地通讯（域不写），成功。可能是因为置空密码后让DC2有的小问题了，然后走的是本地了。

![1734691166876](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942578.png)

成功连接3.20。

##### DC

连接后继续扫（为什么继续扫呢，一个是试，主要还是看域名，明显上面还有一级），这里扫完目标列表也就有了:

![1734691355973](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942579.png)

继续抓3.20密码，进行密码喷射，4.11没有防火墙，直接横向移动正向--成功：

![1734691705125](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942580.png)

下来用到约束委派，先上传adfind：

![1734691931931](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942581.png)

下来就是委派的知识，kekeo先获取用户票据，在利用得到的用户票据获取域控票据，导入票据到内存、连接通讯域控、利用CIFS通讯。

这里是已经抓到了4.11的hash值

这里会有一些差异：

![1734692347853](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942582.png)

![1734703312707](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942583.png)

建立通讯后就是copy后门，创建计划任务执行上线、具体成功还是看约束委派有没有限制：

![1734703427366](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942584.png)

因为这里委派是受限制的，所以是只能看文件，不能执行任务，所以换另一种方法：

![1734703872009](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942585.png)

里面找到一个文件，记录账号密码，然后让DC上线。

下来就利用找到的账户密码，直接执行横向移动。

# VPC3

#### 后面记得复现：

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

#VPC4

自己复现--与VPC3类似，稍有改动，DF变成360  ----入口多了个docker逃逸。

![1734795029542](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942624.png)

 配置环境说明，和图基本一致，就是360的机子开了个外网，这里的杀软开了后可以提升查杀能力，360的云查杀

![1734795775887](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942625.png)

##### 逃逸

docker逃逸目的主要是拿下宿主机权限能和后面网段建立通讯， -- 这里入口是shiro框架（rememberme），直接工具梭哈。

![1734843278671](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942626.png)

梭哈，植入哥斯拉的内存码：

![1734843493568](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942627.png)

哥斯拉连接：

![1734843708509](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942628.png)



这里还是很容易判断出来是Docker容器的----docker逃逸还是又许多方法的，前面回顾一下95天。   特权逃逸，将真机的磁盘挂载出来，然后就可以访问真机139.131的目录了。得到访问真机目录后下来要做的思路是：写入ssh密钥（或者计划任务），获取root权限，执行后门上线真机。

判断出来是Docker：

![1734843781439](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942629.png)

![1734843830130](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942630.png)

下来环境有问题，查看特权时返回值不对：

![1734845541302](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942631.png)

这里如果有特权capeff返回值应该为这样，然后就挂载里面的sda3磁盘，sda3磁盘的大小最像个真机磁盘。

![1734845648157](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942632.png)

好了，饶了一大圈，整整一大圈，xd也没有提醒，直接docker -compose up  -d 就启动环境打了，不给人说一下，需要启动拉好的镜像，并且需要用特权模式启动，这样才能和xd一样，回显正常。。。。真服了，还以为环境有问题：

先用docker特权启动，特权命令：

![1734847423110](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942633.png)

在执行命令就没问题了：

![1734847642080](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942634.png)

这下打开刚才创建的test目录，mkdir /test && mount /dev/sda3 /test --将sda3的磁盘放到我们创建的test目录下    将真机目录放进来了：

![1734847716691](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942635.png)

这下就是拿到真机目录了，然后我们就可以将后门上传到真机目录（test）上，或者上传密钥文件到真机里，就可以连接真机了。

这里是写入ssh密钥后成功ssh连接目标真机。

又又有坑，这里ssh密钥写入docker的真机时候发现真机下没有要写入的文件，也就是没有root/.ssh/authorized_keys，又得搞半天才弄出来：

![1734848943284](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942636.png)

##### 真机

哎，又有有有有有坑，给的docker真机压根就没有ssh服务，哎弄半天好了，密钥写进去就行，直接连接：

![1734849581294](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942637.png)

![1734849506084](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942638.png)

需要注意就是哥斯拉的目录和真机目录多个test

 msf上线：ssh上传正向后门.elf，.elf文件执行上线139.131，下来就建立3网段路由通讯，在建立socks代理，可以访问3.11发现存在通达OA，直接利群梭哈，用蚁剑连接3.11

![1734850409226](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942640.png)

又又又又又又又来了，踩坑。代理啥都配好了，就是连不上，防火墙问题，把来宾不要拒绝了：

![1734853293880](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942641.png)

配socks：

![1734853323556](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942642.png)

实战中就还得扫一下3网段的信息，但是这里就不扫了，直接就是3.11上有通达OA的web服务：

![1734853393545](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942643.png)

利群、蚁剑、这里注意，利群时候代理也不能掉：

![1734853705857](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942644.png)



##### windows域



 这里msf生成3.11反向后门，用蚁剑上传反向后门，让3.11反向连接3.159（本机为3.129），3.11成功上线。mimikatz得到hash密码，下来用crackmapexec喷射hash密码，这里需要用到linux代理工具。

将后门上传到3.11的web目录下：

![1734872516111](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942645.png)

报错：

解决时间最久，不过还是没有弄懂为什么成功，为什么失败，这里刚才代理都配好了，老是执行反向后门，然后msf接收不到，也不知道为啥，下来就是这个：

![1734859424214](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942646.png)

最后是成功了，重启，自己又走了一遍，执行了一次路由，问了n遍gpt，稀里糊涂的解决了。这里应该是代理问题，反正你msf能上线3网段，应该也是跟代理（就是windons那个proxy软件）有关系：

![1734859626199](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942647.png)

![1734859544005](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942648.png)

反正最后是上线3.11了--奇奇怪怪的，稀里糊涂的刚才：

![1734859677780](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942649.png)

上面总结：先拿到docker真机后建立socks代理，然后再在本地进行代理访问3.11网段，发现通达OA，直接利群加蚁剑，下来就生成一个反向连接3.129（docker真机）的后门到3.11机器上，msf监听上线。



这里linux是没有3网段的，哦--那就说明刚才msf收不到3.11的反弹和windows的proxy没关系，那就不知道了。反正这里就是msf有3网段通讯，linux需要用到proxychains来建立3网段通讯来喷射hash。

下来就测试3的网段里的机子，21-32网段所有的，信息收集到的

![1734872282298](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942650.png)

  开始喷射hash密码，比对得到的hash和目标本地存活主机进行比对，比对成功就下载后门并执行。

这里得到3.31web（360）和3.32sql机比对成功，然后上传后门和执行后门。自然就是360没有上线，3.32成功上线，

看谁有pwd3d! 就说明喷射成功：

![1734872319210](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942651.png)

3.11的有web目录---web目录下有后门6666.exe，所以让这3网段的目标机都测试上线，远程下载3.11的后门，并保存到C盘下执行，msf开启监听。相当于批量的下载上线：

![1734873457191](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942652.png)

sql机成功上线：

![1734873492479](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942653.png)

###### 免杀360

 这里免杀思路，是rust + 混淆和分离       用rust语言先将后门bin文件加密和混淆后保存为txt文件，在用参数传递执行上线。这里采用cs上线，因为360是有外网，所以可以实现：

先加密参数--将后门加密：

![1734874255277](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942654.png)

再将后门以参数形式传递：

![1734874451415](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942655.png)

上传到3.11上（通达）：

![1734874595341](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942656.png)

这里就不用这个proxychains4 crackmapexec ，因为我们需要传递参数，所以换成有交互式的工具套件，

![1734875056519](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942657.png)

![1734875162787](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942658.png)

成功上线：

![1734875228169](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942659.png)

1h 这里因为360是有外网的，所以直接通讯，上传免杀后的后门。上线是用 cs的参数上线。成功。备注：这里域环境下的360，一方面是因为在虚拟机中，一方面是域内权限问题（因为还要考虑DC的管控），导致360有些杀毒功能不能开。在这里就是说的这里的下载命令没有被拦截：

![1734875326392](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942660.png)

  下来就是利用历史票据拿DC---3.21，因为DC连接过sql机。利用sqlserve机，刚才sql已经上线到了msf上，这里要利用票据，所以要先上传Mimikatz，发现有历史票据，导入票据。连接DC，这里要先切换到用户，然后在copy后门到DC，创建计划任务，在进行运行，上线。

![1734875690040](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942661.png)

用mimikatz导出历史票据，在导入：

![1734876971150](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942662.png)

成功通讯DC（3.21）：

![1734877069532](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942663.png)

与DC建立通讯后就老套路，copy......创建定时，执行定时

![1734877335127](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942664.png)

# VPC5

入口为若依系统3.31：

![1734881654518](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942665.png)

  这里为shiro框架，但是拿不到key，无key，所以shiro框架无法利用。是通过heaphump泄露来打进去的，利用工具获取到heaphump里的key成功利用shiro进入。

![image-20241223183041273](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942666.png)

![image-20241223183018512](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942667.png)

JDumpSpider-1.1-SNAPSHOT-full.jar  将刚才得到的heapdump文件放到同一个目录下：

![image-20241223183237276](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942668.png)

![image-20241223183637515](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942669.png)

  移交哥斯拉（内存马），上线cs，这里有360，所以哥斯拉上传后门后执行不了命令（因为这里是用的参数分离后门免杀），但是刚才的shiro可以执行（shiro是内存马，牛一点），所以上传免杀后门后用shiro工具来执行上线。

![image-20241223183848011](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942670.png)

这里哥斯拉的流量太明显，所以执行不了命令，360监控了。所以就换成内存马进行

![image-20241223184541879](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942671.png)

用这儿上线：

![image-20241223184814784](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942672.png)

  成功上线--------这里由于环境问题，本地是用web投递直接上线web机的

##### DC：

 这里DC直接通过横向手段拿下---还是强调一下：这里上线DC(3.21)，是通过3.31来获取的，所以是用转发上线3.31的监听器，在让3.21上线到CS。

ok这里注意：上线3.21DC时，转发上线是web机的3.31网址（我用的是他另一个网卡139.130所以失败）：

![image-20241223201459212](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942673.png)

下来还有是这里目标列表刚开始没有目标，你需要用端口扫描，（我用的是网络探测，这探测是能探到，但是放不进去目标列表里）：

![image-20241223201702727](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942674.png)

![image-20241223201818110](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942675.png)

最后就是监听器，选转发上线在一个网段那个：

![image-20241223201944509](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942676.png)



后面都是跟卡巴打交道。

##### 卡巴：

  横向移动：利用横向移动协议或者上传后门上线 。 下来是用套件里的工具进行测试，（代理），wmiexec.py   还是被阻止。  用smb测试，还有decomexec。---将几种横向手法都试一试-----179day：

![image-20241223202511365](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942677.png)

![image-20241223203842873](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942678.png)

smb---------xd当时的视频是可以弹回来的，我这里没有：

![image-20241223204250195](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942679.png)

![image-20241223204615755](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942680.png)

 再用ipc命令,成功和卡巴建立通讯，那就copy、创建计划、执行计划，  利用ipc成功上传。

![image-20241223205033124](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942681.png)

![image-20241223205141080](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942682.png)

![image-20241223205543684](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942683.png)

copy试一试：

![image-20241223210121161](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942684.png)

能copy上去，那就执行任务并且传参上线，但是明显，这里参数太长，导致不行：

![image-20241223210717537](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942685.png)

接下来思路就很简单，想办法绕过执行参数，因为这里已经验证，参数是可以绕过的：

![image-20241223210903665](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942686.png)

反正下来就是给后门做免杀：

换成smb协议执行，执行不了下载上传命令curtutil ，可以执行普通命令。下来 就是尝试绕过下载命令。就是让他执行远程下载命令-------先试试copy成别的exe，在加个混淆下载命令，成功下载后门执行文件，下来就是执行参数传递上线，但是这里的参数太长了，不适合执行。所以将参数放到txt中， ---- 将新生成的后门和文本传上去。

 经过许多尝试，还是参数形式有可能，但是传递不了，所以只能就直接写到exe中。还是叽。

下来是建议采用远程加载txt的shellcode。就是将shellcode的txt放到网站目录下，然后调用。
