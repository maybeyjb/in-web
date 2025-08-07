# 重放攻击

中间人攻击，NTML、重放攻击。

捕获，重放，爆破           kali环境已配好，3个网卡，既能和本机通讯，也能和god内网通讯

  捕获，用kali (配置时候记得先拍个快照)，一般用钓鱼 ，让对方访问（被动方式）。用得到的hash值可以重放， kali 监听webserver，sqlserver执行访问kali的命令（net use  ip(kali)会将自己的用户密码给kali），然后得到web的回弹。kali就是做的是中继重放，然后给了web，web比对后和sql密码一致，执行了kali重放过来的上线命令。31web   32sql  21 DC

kali监听，用web机进行通讯---因为web和kali的账户密码不匹配，所以没有成功，不过得到的hash可以用来比对重放：

![1733753163835](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942272.png)









 不同用户/相同用户都尝试,。web用administrator去尝试连接sql成功，反之sql也是。web用自己的webadmin用户去连接sql不行，拒绝。就是因为用户密码不同。这里就是web机重放攻击sql机

 降权 令牌窃取和进程注入。

  脚本重放攻击

 破解重放的hash ------- 重放失败 

![1733757925010](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942273.png)

   强制的。  Coercer 自动化工具。让目标主机强制的访问别的主机（前面是钓鱼式的）。然后就可以抓到目标访问的hash，拿到后可以用来利用。

##### 强制访问

让目标主机（21）强制访问被控机（32）：

![1733757243654](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942274.png)

被控机sql监听：

![1733757315850](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942275.png)

或者在kali中监听，让DC强制访问kali主机：

![1733757476371](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942276.png)



  另2个工具。先用前面的工具smbrelayx.py监听，再用下来的两个工具进行强制访问获取hash -------- 。就是让目标用户（Web机被控机）访问本机（kali），然后本机在将请求重放到3.22DC（目标机）后在执行whoami。这里需要用户密码都对应着。

![1733757857637](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942277.png)

最后还是用PetitPotam.py实现强制让目标访问kali，kali本地用py监听成功收到：

强制访问：让31访问128

![1733758304978](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942278.png)

监听到webserver的目标，wd：

![1733758369861](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942279.png)

，总结用这些方法主要还是看环境，看环境适用哪种方法。强制访问抓的是机器密码，钓鱼抓的是用户密码

 不断翻车，没有复现
