#  linux横向

linux横向移动主要为ssh、web。         这里10、30、50都是机器ip的尾缀（前面都是172.16.250）

shh协议中，登录连接目标机器主要方式有密码和密钥对（密钥对文件）登录

​        看主机是密码还是密钥登录方式。密码就是猜测爆破得到，密钥就是找配置文件或者在历史命令中找，全局搜索命令

![1734265624954](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942346.png)

![1734253580338](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942347.png)

  全局搜索、搜索含有SSH凭证文件

![1734265688504](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942348.png)

  linux靶场，先拿到web权限后提权---这里是tomcat用户权限（10机器），所以需要提权，这里是linux提权，这里选择用脏牛提权，先将脏牛漏洞里的.c上传放到目标的tmp目录，然后gcc编译目标.c文件成可执行文件，因为脏牛系列会弹shell所以这里需要先用py起一个终端在执行文件，然后在用命令让反弹的权限持续下来。

先namp扫到目标机：-----这里眼瞎了耽搁，没看清ip然后xd给了靶场的账号密码。

nmap扫太慢了。就是扫到了3个ip  分别就是图上的3个ip，这里就不看了------环境说明kali 和目标机在一个网段中

![1734267076853](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942349.png)

### 10机

目标机上存在strut2的历史漏洞：

![1734267254293](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942350.png)

拿到10的权限：

![1734267300129](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942351.png)

#### 脏牛提权
.c上传放到目标的tmp目录，然后gcc编译目标.c文件成可执行文件（当然这里目标机是有gcc环境的）

![1734267559024](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942352.png)



脏牛系列会弹shell所以这里需要先用py起一个终端在执行文件

![1734267622068](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942353.png)

用命令让反弹的权限持续下来

![1734267780901](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942354.png)

这里注意：这两个命令都要执行，刚才就只执行第一个导致掉线了。

![1734268220373](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942355.png)

### 30机：

得到root权限后查看历史命令发现用过密钥文件登录过其他主机，这里就将密钥文件下载到本地，然后利用文件登录另一台目标主机（30主机）-----------这里得到30主机后因为是通过ssh密钥连接，但是没有8080端口权限，因为50机器的3306是和30机器的8080端口是有关系的，所以这里需要先得到30主机的8080端口权限。

查看历史命令，找到了密钥对文件，所以将密钥对文件复制到本地来，因为本地也可以和30通讯

![1734268324008](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942356.png)

![1734270231653](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942357.png)

这里kali 的ssh坏了，没修好，不过执行下面两个命令后就可以连接得到30的权限：

![1734270098226](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942358.png)



后面本机ssh成功，下面有解释：

![1734273923643](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942359.png)

​30主机的8080在DMZ区，所以本地是访问不到，需要在10建立节点通讯（这里是因为30在DMZ区域，本地只能访问到22端口，访问不到8080，所以建立30节点也不行，而10访问30时可以访问到30的8080端口），
#### linux创建socks节点

![1734270464496](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942360.png)

  配置节点有个坑，创建了却访问不到，需要先将被控机路由添加进来然后在建立节点，这里就成功建立节点了，还有配置proxychaix.config文件时，不仅要改端口和版本，这里还有更改动态和静态配置--------  还是翻车，最后用的是windows的proxifier进行配置代理。不过上面的节点还是需要建立的。

建立路由：

![1734270707798](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942361.png)

![1734270701749](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942362.png)

还需要改这儿：意思就是当使用proxychains4时候，这时候的流量就是一直走代理，不是执行一次才走一次。

![1734270849776](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942363.png)

配置好，记得   run：

![1734271594450](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942364.png)

![1734271651872](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942365.png)

建立通讯：

![1734271724580](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942366.png)

### 50机---jenkins

找到jenkins路径后打开发现有credentials,这里是存放密钥的地方

![1734271813045](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942368.png)

可以看到30主机是有jenkins，这是一种监控配置文件，里面可能存一些配置性文件。这里是有数据库配置性文件，里面找到了连接50的数据库的操作记录，

![1734272015168](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942369.png)

然后30这边的网页打开发现da数据库，这里就猜测是与50目标机有关，因为上面30机尝试连接过50的mysql。

![1734272141712](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942370.png)

1h  前端更改隐藏密码功能，还需要解密，需要三个文件。

点进去db_backup然后发现有账号密码：

![1734272289584](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942371.png)

先去除密码隐藏，只需要将上面的password置空即可：

![1734272361165](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942372.png)

 这里就得用nc监听30让他主动给出来文件,让进行解密。解密后进行连接50就行

上面得到的密码就是jenkins的加密算法，需要解密3个文件进行解密：

这里刚才的没ssh连接成功30又回来了，用本地机器连接（这里代理没有断），但是报错一直Permission denied, please try again.管理员打开cmd也不行，最后才是这样，怪不得在kali中也只是给他0600 的权限，不给777，因为这里这个id_rsa文件（是刚才在10上获取连接30的密钥文件）权限设置得太开放，SSH 客户端要求私钥文件不能被其他用户访问。

所以下面这样让文件只允许一个人（本地用户）访问---（这类文件应该都一样）

<img width="1227" height="668" alt="image" src="https://github.com/user-attachments/assets/924cf0dc-1685-4a07-89d6-fe0ea8d0a66b" />


成功：

![1734273884074](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942374.png)

那么这里就用本地作为中转，这里代理好后可以和攻击机和目标机都能通讯：

![1734274219512](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942375.png)

#### 解密
下来接着上面的解密，已经得到了解密的密码，需要3个解密文件。获取（在已经控制的30机子上获取）：

![1734274452407](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942376.png)

解密需要的py环境也难配置，一直报这个Crtpto，最后需要安装pycrptodome模板，有时候报错将脚本的导入from Crypto.Cipher import AES语句扔给gpt，他能知道你需要下载的是哪个脚本：

![1734275284369](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942377.png)

解密得到的3个解密文件，得到了最后的密码---用py解密出来的文件如果是这样：     b'asdasd'   这里是密码应该是asdasd   因为py解密出来的前面的b是bit字节流的意思  ，所以这里的b')uDvra{4UL^;r?*h'   最后的密码就是：       	)uDvra{4UL^;r?*h   ：

![1734275429367](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942378.png)

用密码连接50，这里的用户就是刚才30网站得到的db_backup的用户名：

![1734275839162](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942379.png)

![1734275857031](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942380.png)

提权没有设置难度，直接就是sudo su   （这是提到最高权限的命令，而不是su root）.....：

![1734275991667](https://cdn.jsdelivr.net/gh/maybeyjb/blue-team/img/202506170942381.png)
## 总结：

3台linux机子的横向演示
10机子的提权+维权
30机子：本机是通过密钥对实现ssh横向的         而8080端口代理出网---linux---msf建立socks代理  
50机子通过解密获取到账密实现横向 


