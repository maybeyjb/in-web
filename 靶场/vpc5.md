
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

##### 绕卡巴：

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
