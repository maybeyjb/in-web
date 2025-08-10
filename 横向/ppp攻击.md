
# p+p+p横向攻击 

背景：windows 2012以上默认关闭了Wdigest，所以攻击者无法通过内存获取到明文密码，所以就有下面的几种方法(前面的wmi和smb服务也可以)：

pass the hash（哈希传递攻击，简称pth）

pass the ticket（票据传递攻击，简称ptt）重点

pass the key（密钥传递攻击，简称ptk）

![1733304344193](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942195.png)

### pth:   hash传递

可以看出来基本现在都是ntml的hash。

![1733151600149](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942196.png)

 LM和NTML两个hash值的种类。前面的Wmi  smb 都是hash的pth攻击。
 主要还是前面的协议和服务两个节里的横向手法，主要为pth攻击


#### NTML
NTLM 是 Windows 系统中经典的认证协议，基于挑战 - 响应机制，依赖用户密码的 NTLM Hash 完成认证。

抓取明文密码时候，以NTML的hash为准：

![1733304997753](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942197.png)

![1733305018372](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942198.png)
#### hashcat

 hash解密          cmd5在线，工具：https://hashcat.net/hashcat/   不过原理都是hash碰撞。

![1733305436114](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942199.png)

这里工具还有许多玩法，-m 密文类型           -a 破解类型         ?l 小写?s 符号?d 数字      	大写字母就是?u		eg: admin!@#45  5个字母（?l）3个符号（?s）2个数字（?d）  ?l?l?l?l?l?s?s?s?d?d   工具比较看cpu显卡的速度

参数1000就是指用NTML的hash来进行碰撞：

![1733305572632](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942200.png)

mimikatz向计算机中写内存：

hashcat -a 0 -m 1000  518b98ad4178a53695dc997aa02d455c  --show

利用hash值进行移动可以用上面的wmi   smb  dcom 几个协议进行上线

### ptt:票据攻击         

 Kerberos票据中心。

 思想：用漏洞让Kerberos给自己发一张票，用mimikate进入，（导入内存中 ，好像做了个假的身份证），在连接上。这里需要明文（不支持hash）。先获取目标的SID值，然后用ms14漏洞生成门票，然后用mimikate导入门票到目标中，然后连接上线

![1733306579595](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942201.png)

这里的攻击条件是域内用户，然后域内密码是刚才破解出来的。先将漏洞工具上传：

![1733307451179](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942202.png)

ping 一下域控（DC是前面收集到的ip）

![1733318539839](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942203.png)

生成票据文件：

![1733307660101](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942204.png)

此时查看没有票据：

![1733307707298](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942205.png)

ms14漏洞 -- 生成票据文件

![1733318582880](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942206.png)

先尝试连接DC ,不行。然后显示当前票据：

![1733318619644](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942207.png)



用mimikate导入票据：mimikatz kerberos::ptc TGT_webadmin@god.org.ccache

![1733318733005](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942208.png)

连接目标上线，这里就连接到了DC机器：

![1733318787834](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942209.png)

#### DC上线： 
下来就上传正向后门到目标机器在执行上线----思路是上传后建立一个服务（类似于定时服务一样），然后执行服务，用web机连接connect ，得到DC权限：

先查看当前目录下的后门文件 4444.exe，在copy到DC机器：

![1733319097667](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942210.png)

copy：

![1733319159007](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942211.png)



####  kekeo工具:

也是一款工具，使用hash值来换成票据。直接调用cmd。利用获取的NTLM生成新的票据尝试认证（pth（hash）的攻击转为ptt(票据)攻击）

![1733319708457](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942212.png)



####   历史漏洞

 历史遗留的票据重新认证尝试。DC登录过目标机器，或者连接过（比如说更改当前机器防火墙就需要登录域管权限）。就会有票据遗留，所以就可以抓出来这个票据（时间就几个小时内登录过）

这里执行命令需要system权限，我这里环境没有配置好，导致提权不了，提权到system需要MS14-058漏洞，这款漏洞支持反向后门，但我这儿环境反向不了。所以没有演示：

![1733320448934](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942213.png)

执行mimikate导出命令后，发现多了administrator的几个票据，然后导入，连接，上线。

![1733320317007](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942214.png)

  成功与否看加密类型------RC4类型（暴力破解）

 手工监测：SPN技术获取域内通讯服务，在去连接通讯服务，连接后查看票据加密类型能否支持爆破。



![1733321268609](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942215.png)

mimikatz kerberos::ask /target:MSSQLSvc/SqlServer.god.org:1433

![1733321289154](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942216.png)

这里的web机和DC不信任了。所以报错：

下来只需要查看klist看是什么加密类型：--会话密钥类型是不是rc4,

###### 工具检测
（需要考虑免杀和通讯问题） ：

Rubeus kerberoast     检测有没有rc4加密，这里是aes加密。如果为rc4加密的话会破解出来他的hash值

![1733321644240](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942217.png)

jack机登场：破解出来为文件票据：用 tgsrepcrack.py解密手工生成的票据      为HASH密文：hashcat 。

Rubeus   							破解主机后获取为hash值。  这里的目标为0day/jack

![1733326096273](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942218.png)

Rubeus kerberoast

![1733386730489](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942219.png)

impacket-getuserspns        这个破解后为hash值。用py文件建立代理测试，后连接破解生成hash.txt，内容和上面工具的hash一样.。用socks代理在提醒一下，代理服务器为c2服务器，端口为socks端口，差点将ip写成jack机的。：

![1733387427775](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942220.png)

用py工具连接并且将能破解的票据格式保存到hash.txt：

![1733387565057](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942221.png)

###### 手工查看：

![1733387680076](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942222.png)

##### 票据破解：

先看spn服务，在找到能通讯的服务--msssql，在用mimikate产生票据，最后导出并破解：

![1733387985231](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942223.png)

用mimikate将票据拿出来然后破解，手工连接（mimikate）后会产生票据，这时候用Mimi导出来然后用py工具破解。

将获取到的票据导出来：

![1733388225497](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942224.png)

下载到本地：

![1733388313896](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942225.png)

![1733388410795](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942226.png)

用kerberoast.py 破解票据：

![1733388684919](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942227.png)

##### hash破解：

将刚才Rubeus 得到的hash或者impacket-getuserspns得到的hash.txt用hashcat进行破解：

先看票据的加密类型为rc4类型，因为后面需要hashcat -m后的参数类型需要和当前加密一致：

![1733389146622](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942228.png)

![1733389322460](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942229.png)



有明文传递，肯定用明文->没有明文就用，PTH(HASH传递)->PTT(票据攻击)3个方法->PTK(AES)

### ptk    AEC  加密

条件严格：系统安装了KB2871997补丁（打了补丁也不一定好。哈哈）且禁用了NTLM（禁用.......）的时候，和ptt你死我活的样子。

鸡肋：得有图形化UI页面，这里不适用。最后弹出来的在目标机桌面终端操作

## 总结：
pth关注前面的协议和服务的横向手法，都是利用hash进行横向移动
ptt攻击，利用票据进行横向。获取目标（DC）的SID值，用ms14漏洞生成门票，mimikate将生成的门票导入到目标（DC）中，这时就可以与目标（DC）建立连接了。下来在上传正向后门到目标（DC）建立一个服务（定时服务），然后执行服务，用web机连接connect ，实现上线目标（DC）
票据这里还牵扯到一个加密，如果为RC4类型，可以先获取其hash，在破解票据的hash值。或者mimikate导出票据直接破解票据也可以。
kekeo：将hash值转换成票据，pth（hash）的攻击转为ptt(票据)攻击

ptk就比较不太实用，因为为aes加密。
