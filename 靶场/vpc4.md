#VPC4

与VPC3类似，稍有改动，DF变成360  ----入口多了个docker逃逸。

![1734795029542](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942624.png)

 配置环境说明，和图基本一致，就是360的机子开了个外网，这里的杀软开了后可以提升查杀能力，360的云查杀

![1734795775887](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942625.png)

##### docker逃逸

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
