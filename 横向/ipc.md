# 域内域外

![1732855937089](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942137.png)

​      拿到权限先看是什么用户，如果为普通用户，那么就先提权到system权限（逃离DC控制，能于域内用户通讯），或者切换成域内用户，先获取密码凭证---->然后切换用户输入密码

![1732893793057](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942138.png)

![1732893767715](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942139.png)

提权插件：

![1732893944807](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942140.png)

可以执行域内命令：

![1732894006981](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942141.png)

这里因为ailiyun老报警，所以重新配置环境后用kali当作攻击机服务端------------目前环境说明：环境也配不动，不管了，现在就是本机YJB能ping 通web 机，web 出不来，就用YJB当跳板机连接web



  用工具枚举爆破用户密码--在域外爆破域内用户的账户密码

![1732945720079](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942142.png)

 minkatz  在win10及win-server2012R2以上抓取不到明文，只能抓取hash值。因为这些版本以上默认内存中关闭了注册表缓存明文密码

![1732945990353](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942143.png)

#### IPC文件共享
 域内域外用户都需要提权			

33        ipc （文件共享） 测试攻击   利用ipc文件共享与目标主机建立连接      dc:3.21             sql:3.32 	
ipc工具：用前面收集到的密码凭证去套用        建立ipc连接          user\admin 一般为域内用户，admin或者\admin一般为域外本地用户：			

![1732946217851](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942144.png)

#### IPC$利用命令

net use \\server\ipc$ "password" /user:计算机名\username # 工作组												                                  net use \\server\ipc$ "password" /user:域名\username #域内

net user \\192.168.3.32\ips$ "123456" /user:god\administrator     登录god域（这里有god域就自动识别）的administrator 用户

net use \\192.168.3.32\ipc$ "admin!@#45" /user:administrator   如果当前为域内用户执行命令，这种写法是默认为登录要域里的administrator用户（DC），如果为本地用户执行这命令时，就是登录本地机器（当前控制机器）的administrator用户，因为没有加目标计算机机器的名字。

net use \\192.168.3.32\ipc$ "admin!@#45" /user:sqlserver\administrator   如果当前为域内用户执行命令的话：就是先找有没有sqlserver域，没有的话就当作sqlserver计算机名，用户为administrator，如果有域是 sqlserver，那么就找sqlserver域下的administrator用户。如果为本地用户执行这命令时，就直接找的是sqlserve计算机的administrator用户。

总结下来就是要登录域内用户时候要加上域+用户（god\administrator）    登录本地时：本地机器名+用户（sqlserver\administrator ）

![1732961884029](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942145.png)

这里是用web机的域外（administartor用户）可以通过ipc连接到sqlserver的administrator:

![1732979651063](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942146.png)

![1732979752669](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942147.png)

控制Webserber--(后面简称web机)，能ping 通内网sqlserver机，使用正向后门，先上传到web机上---在上传copy到目标

​    sql主机----再用at 或者schtasks（两个有使用场景）计划任务执行。运行后正向后门需要web机主动connect，at使用时目标小于win2012  schtasks是目标大于Win2012

![1734321268190](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942148.png)

 反向后门，监听器为web 的内网地址3.31

后面继续得到其他域内主机的密码，最后喷射出dc密码进行登录连接

#### cs插件：

![1732946359542](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942149.png)

  工具箱  借助socks代理用py文件进行ipc建立连接

先连接sql，然后设置好代理，再用工具：

![1732947502501](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942150.png)

![1732947488408](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942151.png)

这里连接后执行whoami成功，还可以实现上线：让sql机远程下载后门（不建议）2、powershell命令实现上线

![1732948094690](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942152.png)

  关闭ipc，

 ipc策略问题    ---- 后续补充改错

## 总结
看是否开启ipc文件共享，开启的话即可使用ipc命令或者c2插件登录域内用户。
