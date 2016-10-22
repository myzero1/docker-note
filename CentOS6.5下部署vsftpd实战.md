CentOS6.5下部署vsftpd实战

冤枉地址 http://www.centoscn.com/CentosServer/ftp/2014/1215/4313.html

```doc

1.          实验需求：
1)     使用RPM包安装vsftpd服务
2)     实现匿名用户访问，验证仅可以访问和下载，不可以上传
3)     实现匿名用户可上传、下载、修改等完全权限（现实环境中这样的需求可能性很小）
4)     实现把登陆的用户禁锢在自己的家目录中
5)     实现限制某些用户的访问
6)     实现虚拟用户的访问
7)     实现针对不同的虚拟用户拥有不同的权限
 
2.          实验环境：
Linux服务器系统版本：Red Hat Enterprise Linux 6.5     IP：192.168.20.3
Windows客户机系统版本：Windows 7 Ultimate x64      IP：192.168.20.2
vsftpd软件版本：vsftpd-2.2.2
 
3.      实验步骤：
 
基本安装操作
 
A.   挂载系统光盘并安装vsftpd
这里我们使用rpm安装包安装vsftpd，安装包放在系统光盘中的Packages目录中，我们首先挂载系统光盘到系统的mnt目录下
[root@localhost~]# mount /dev/sr0 /mnt
wKiom1SFEhGhFd2sAAA8mm_IYQk841.jpg
到Packages目录下找到vsftpd服务的软件包并安装，安装完成。
[root@localhost~]# rpm -ivh /mnt/Packages/vsftpd-2.2.2-11.el6_4.1.x86_64.rpmwKioL1SFErCjRh9VAACwkv-cb-I757.jpg
 
B.   查看vsftpd配置文件
 [root@localhost ~]# grep -v "#"/etc/vsftpd/vsftpd.conf #过滤掉配置文件中#号的注释
anonymous_enable=YES   #已开启匿名用户的访问
local_enable=YES   #已开启本地账号的访问
write_enable=YES   #已开启写入的权限
local_umask=022    #本地用户上传文件的权限是644，文件夹是755
---------------------以下配置为服务默认，此实验中无需关心----------------------
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=YES
 
pam_service_name=vsftpd
userlist_enable=YES
    tcp_wrappers=YES
 
 
 
实现匿名用户访问，验证仅可以访问和下载，不可以上传
根据vsftpd配置文件的默认配置，当vsftpd搭建好之后什么都不做就可以被匿名用户和本地用户访问了
 
A.   效果验证：
[root@localhost~]# service vsftpd start  #启动服务
为vsftpd 启动 vsftpd：                                    [确定]
 
测试之前，必须把防火墙和selinux关掉
             [root@localhost ~]# serviceiptables stop
             iptables：将链设置为政策 ACCEPT：filter                    [确定]
             iptables：清除防火墙规则：                                 [确定]
             iptables：正在卸载模块：                                  [确定]
 
             [root@localhost ~]# setenforce 0
 
我们在客户机上用文件夹的方式访问 ftp://192.168.20.3 
wKiom1SFEjGTUqwWAAD0ghPk8jg635.jpg
现在来测试一下匿名用户的上传和下载权限
注意：我们用匿名账号ftp登陆（无密码），查看现在所在的工作目录为/，这个/并不是服务器的根目录，而是匿名用户自己的家目录，ls查看发现里面有一个pub的文件夹
wKiom1SFEkSzKu3RAADD4NnMOiE856.jpg
家目录就是服务器上/var/ftp这个目录：
wKioL1SFEumykyrhAAAXEIsPc8E244.jpg
我们验证匿名用户可不可以下载首先要在ftp这个目录下新建一个可供下载的文件[root@localhost ~]# cd/var/ftp  #切换到ftp目录下
[root@localhostftp]# echo "this is test ftp" > test.txt  #新建一个test.txt文件，内容为this is test ftp
wKiom1SFEmCBXeiqAABKnRT561I283.jpg
回到客户机上的cmd控制台
C:\Users\Administrator>f:\    //切换到f盘，我们下载的文件就会下载到f盘
wKioL1SFEv_hSK5LAAD-_XujADI013.jpg
用匿名账号ftp登陆，下载test.txt文件，打开f盘，看到已经下载的文件
wKioL1SFExLzXkpwAAGJq_3Ipk8133.jpg
现在来验证匿名用户是否可以上传文件，我们把刚才下载都客户机上的test.txt文件改名为tes.txt文件用于上传测试（避免重名）
wKiom1SFEo6z7jn6AAAxAJs9Di8275.jpg
上传时被拒绝，所以我们知道匿名用户是只可以下载不能够上传的。
那如果是系统本地账号的话可不可以上传呢？
[root@localhostftp]# useradd tom  #新建一个tom的账户
[root@localhostftp]# passwd tom  #设置tom的密码
回到客户机的cmd控制台，用tom登陆并上传tes.txt文件，上传成功。
wKiom1SFEqSBvp6rAADgNtfi9aw614.jpg
[root@localhostftp]# ls -l /home/tom  #查看上传的文件的权限为644（因为配置文件中local_umask=022）
wKiom1SFEvyjo89iAABQNJsYPmA300.jpg
注意：如果selunux没有关掉，用本地账户也是不可以登陆的，会报以下错误
wKioL1SFE6CT6ft4AAA1D-w_Fao285.jpg
 
 
实现匿名用户可上传、下载、修改等完全权限
我们要让匿名用户可以上传文件夹，需要修改配置文件：
           anon_mkdir_write_enable=YES
注意：当我们不知道如何去配置要添加哪些或要修改哪些选项时，我们可以借助man手册来寻求帮助：
           [root@localhost vsftpd]# manvsftpd.conf
在手册里面查找跟anon相关的内容，我们看到有一项是other write，他的默认值是No，如果设为yes就可以允许用户上传、创建目录并且删除、重命名等操作
 
 
接着我们到vsftpd.conf脚本中插入一行内容：anon_other_write_enable=YES
修改完毕之后匿名用户就获得了最高权限（可读写，删除，重命名）
A.   修改后的脚本
wKioL1SFE7XA-L8XAAIeR_ZU61I507.jpg
 
B.   效果验证
 
[root@localhostvsftpd]# service vsftpd reload  #重新加载配置
关闭 vsftpd：                                              [确定]
为 vsftpd 启动 vsftpd：                                    [确定]
 
回到客户端的cmd控制台，以ftp匿名账号登陆，并删除文件，提示失败。
wKiom1SFEzGjPx38AAG5RFfQ_Qw822.jpg
回到服务器查看ftp这个目录的权限，是没有写权限的，所以也就无法删除
wKiom1SFE0PA0xlxAAC8d6LlRnE834.jpg
[root@localhostvsftpd]# chmod 777 /var/ftp  #将ftp权限设为最大
wKioL1SFE-ngNDoHAABEJkDOX-0744.jpg
回到客户机，用ftp登陆，发现直接登陆就报错了，因为我们把var/ftp这个目录权限改到最大了，这样匿名用户就可以为所欲为了，为了安全，vsftpd设置了直接就不允许登陆了。
wKioL1SFE_zyaI5TAABAMWJTOkY443.jpg
 
那么如果我们想让匿名用户有最大权限应该怎么做呢？这时我们应该对ftp目录下的子目录进行操作
[root@localhostvsftpd]# chmod 755 /var/ftp  #把ftp的权限改为默认的755
[root@localhostvsftpd]# cd /var/ftp
[root@localhostftp]# mkdir anon  #在ftp目录下新建一个anon目录
[root@localhostftp]# chmod 777 anon #修改目录权限为最大
wKiom1SFE3fiPaAaAACD1Pq9vpk041.jpg
[root@localhostftp]# cp test.txt anon  #将test文件拷贝到anon目录下，用于后面的删除测试
[root@localhostanon]# ls -l #查看test文件权限为不可写就不可以删除
wKioL1SFFDTSfknLAAAzvbtGgY8775.jpg
[root@localhostanon]# chmod 666 test.txt  #将文件权限改为可写
wKiom1SFE8bQApBTAABDYburVYo676.jpg
这样，test.txt就可以被匿名用户删除了。
回到客户机的cmd控制台，用ftp登陆，切换到anon目录，删除test.txt文件，删除成功。当然匿名用户也是可以进行其他操作了这里就不一一测试了。
wKioL1SFFG2jePJPAAErtF6ZJ1E854.jpg
 
 
实现把登陆的用户禁锢在自己的家目录中
现在我们用本地账号tom登陆，默认的当前工作目录是tom自己的家目录，我们可以将他任意切换到服务器的任何目录下：
wKiom1SFE_KiDKWiAADWkBYlM9Q917.jpg
 
这样是非常不安全的，所以我们要让用户登陆之后仅仅只能在他的家目录这个范围活动不允许他任意切换到其他目录，我们需要修改配置文件使这行配置生效：chroot_local_user=YES
 
 
A.   修改后的脚本
wKioL1SFFJeihjbAAAHiV3S9A1Q252.jpg
 
 
B.   效果验证
   [root@localhostanon]# service vsftpd reload  #重新加载配置
关闭vsftpd：                                              [确定]
为vsftpd 启动 vsftpd：                                    [确定]
回到客户机的cmd控制台，用tom登陆，切换到根，发现这时候的根是他自己的家目录，并不是服务器上的根目录下了，他已经被禁锢在自己的家目录中了。
wKiom1SFFBSwFXzTAADh_P650R0728.jpg
 
实现限制某些用户的访问
我们查看vsftpd目录下的内容，发现有一个user_list文件
wKiom1SFFB_DREd0AAA34k1XzYI722.jpg
 
查看user_list文件内容，发现这个里面都是被拒绝登陆的用户，所以我们要让谁不可以登陆就把账号写进这个文件里面就可以了，这里我们测试不让tom登陆
 
wKioL1SFFMDCn39iAADdtyGCgp8507.jpg
A.   修改后的脚本
 
 
wKiom1SFFEPh1S3CAADx0o6uUTo087.jpg
 
B.   效果验证
回到客户机的cmd控制台，用tom登陆，登陆失败。
 
wKioL1SFFO3T-e3TAABJwDz_S9Q678.jpg
注意：这里tom登陆失败的原因是因为我们把tom写进了user_list这个文件当中，另外也是因为配置文件中userlist_enable=YES这行设置，如果配置成userlist_deny=YES那么就是只允许user_list中的用户登陆
 
 
实现虚拟用户的访问
如果我们只是想要新建vsftpd账号而不让他作为一个系统账号的话就可以用到虚拟账号功能了
[root@localhost vsftpd]# vim vuser  #新建一个叫做vuser的文件
在这个文件中基数行为用户名，偶数行为密码，我们新建两个用户lisa和jack
wKiom1SFFFvy-KLUAABTKVLRlbs988.jpg
wKioL1SFGFTwpmMuAACkWBwOIG8557.jpg
[root@localhost vsftpd]# db_load -T -thash -f vuser vuser.db  #将vuser转换成数据库文件 db_load是命令 -T指定转换-t 指定转换类型为hash -f指定要转换的文件vuser.db为转换后的文件名
[root@localhost vsftpd]# file vuser.db  #查看文件类型，可以看到vuser已经被转换成一个vsftpd能识别的hash数据库文件
wKioL1SFGHODJJv0AABAyc_NsTQ691.jpg
[root@localhost vsftpd]# chmod 600vuser     #为了安全，不想让其他用户可以看到这个文件里有哪些东西，修改权限为600
[root@localhost vsftpd]# chmod 600vuser.db
 
虚拟用户创建之后，要给虚拟用户映射到一个系统账号
[root@localhost vsftpd]# useradd -d/opt/vuser -s /sbin/nologin vuser  #新建一个虚拟用户的映射账号vuser，指定宿主目录为opt/vuser，并指定不允许登陆系统
 
[root@localhost vsftpd]# vim/etc/pam.d/vsftpd.vu  #为虚拟用户创建pam认证模块命名为vsftpd.vu
在认证文件中加入以下两行认证信息：
auth required pam_userdb.sodb=/etc/vsftpd/vuser 
account required pam_userdb.sodb=/etc/vsftpd/vuser
#这里的vuser其实是vuser.db，这里省略了db，否则会报错
 
之后到vsftpd.conf配置文件中插入以下三行内容：
guest_enable=YES   #开启虚拟用户访问
guest_username=vuser   #映射到系统账号vuser
pam_service_name=vsftpd.vu   #指定pam认证模块
注意：vsftpd有一个默认的pam认证模块，需要注释掉
 
A.   修改后的脚本
wKioL1SFGI-Rm07sAAIKobx0GiI605.jpg
 
B.   效果验证
[root@localhostvsftpd]# service vsftpd restart   #重启服务
关闭 vsftpd：                                              [确定]
为vsftpd 启动 vsftpd：                                    [确定]
回到客户机的cmd控制台，用lisa登陆并上传文件成功
wKiom1SFGA3TmwYIAACxKL6hYp8657.jpg
[root@localhostvsftpd]# ls -l /opt/vuser    #查看上传的文件的属主是vuser，说明lisa是映射到vuser这个系统账号了
wKiom1SFGByx9aGKAAA62fYs-nk859.jpg
接着我们把tes.txt文件改名为te.txt防止重名，以jack登陆并上传te.txt
wKioL1SFGMGDu1e4AACzejlcrX0878.jpg
查看jack上传的文件的属主依然是vuser
wKiom1SFGDzh0hqMAABlq20ZoGI390.jpg
注意：如果虚拟用户都可以登陆成功但是上传时提示权限被拒绝的话，需要去vsftpd.conf文件中使这行生效anon_upload_enable=YES
 
 
实现针对不同的虚拟用户拥有不同的权限
以上实验中lisa和jack的权限是一样的，现在我们让lisa上传的文件权限为600而jack上传的文件权限为644，我们需要开启这个单独配置文件功能
打开vsftpd.conf配置文件，加入以下一行配置告诉他去vu_dir目录下找他们的单独配置文件：
user_config_dir=/etc/vsftpd/vu_dir
 
[root@localhost vsftpd]# mkdir vu_dir   #在vsftpd目录下新建vu_dir目录
[root@localhost vsftpd]# cd vu_dir
[root@localhost vu_dir]# vi jack    #在vu_dir目录下为jack新建一个单独的配置文件（暂时不写任何内容保存退出）
wKioL1SFGN-wJ8sWAAAUOtU6tm0512.jpg
我们用man手册查看一下anon_umask（匿名上传）这个选项，发现默认权限是600，所以前面我们看到lisa和jack上传的文件权限是600
wKiom1SFGHrCglBTAAHEtFdZf8s231.jpg
如果要使jack的上传权限是644，那就把这个值设为022即可
接着我们就vi jack在文件中加入一行内容：anon_umask=022
 
A.   修改后的脚本
 
wKioL1SFGUbj-3UaAAHuSJ4S3Yg016.jpg
wKiom1SFGLXjOv87AAA19qIchCs765.jpg
 
B.   效果验证
[root@localhostvu_dir]# service vsftpd restart  #重启服务
关闭 vsftpd：                                              [确定]
为 vsftpd 启动 vsftpd：                                    [确定]
回到客户机的cmd控制台，用jack登陆并上传t.txt文件
wKioL1SFGWajzXzMAACWGzFeVGk485.jpg
服务器上查看上传的文件权限为644，实验成功。
wKiom1SFGOHQF8cNAACNf2WQll4098.jpg
 
4.          实验总结：
1)     vsftpd是一款在Linux发行版中最受推崇的FTP服务器程序。特点是小巧轻快，安全易用。
2)     vsftpd中的虚拟用户又是一个非常实用的功能，满足不同用户的不同访问特性，在配置文件中一定要注意权限的配置，以降低权限虚拟用户是默认作为匿名用户进行处理的。

```
