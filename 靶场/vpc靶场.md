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


