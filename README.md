# docker使用笔记
把在windows上使用docker踩的坑，记下了，备忘。

# Description
这里是一个备忘也是一个提升的过程记录

# How To Use
1.  安装dockertool

    1.1. 下载dockertool 官方地址是http://get.daocloud.io/#install-toolbox ，在https://get.daocloud.io/toolbox/ 这里下载国内镜像
    
    1.2. 安装请参考官方手册https://docs.docker.com/toolbox/toolbox_install_windows/
    
    1.3. 替换国内镜像源，具体方法参考http://docs.daocloud.io/faq/what-is-daocloud-accelerator#docker-toolbox
    
    1.4. 安装好后，点击桌面上的Docker Quickstart Terminal这快捷方式，会自动创建名字为default的dockervm，并启动。你会发现在这个启动的窗口中不能复制，很不方便，你可以安装最新版的git bash替换现在已经安装的，体验要好的多。在git-bash中输入docker-machine ssh，就可以进入刚刚启动的docker-vm。docker-machine start/stop分别为启动和关闭docker-vm，更多操作，请用docker-machine --help查看。`注意：1. docker-machine ssh是以docker用户进入boot2docker，进入后可以用“sudo -i”或者“sudo su root”切换到root用户 2. /var/lib/boot2docker/bootlocal.sh 是 boot2docker 的开机启动配置脚本,有新的配置我们可以直接写在里面`
    
    1.5. 创建machine   docker-machine create --driver virtualbox my-machine
    
    1.6. 启动脚步。1.boot2dokcer的启动脚步是/var/lib/boot2docker/bootlocal.sh可以持久化。2.Ubuntu容器的启动脚步是/etc/bash.bashrc可以持久化。3.centos容器的启动脚步是/etc/bashrc可以持久化
    
2.  配置docker-vm

    2.0 执行docker-machine stop关闭docker-vm然后进行配置

    2.1 配置docker-vm与windows主机的共享目录：
    
    docker-vm创建时默认设置了一个共享目录c:/Users的映射，这个在docker-vm中应该有特殊的处理，可以在虚拟机里面看到，且是实时更新的，具体是如何处理的，现在还不太清楚。为简单我们就直接修改一下这个映射，是它映射到我们自己的目录就好了。如图![image](https://github.com/mywoogle/docker-note/blob/master/image/1.png)
    
    2.2 配置docker-vm网卡
    
    docker-vm默认添加了2个网卡，我们只需在“网卡1”中添加一条 127.0.0.1 80:80的转发就可以了。就这样修改后，就可以保证docker-vm既可以访问外网，也可以在windows上通过127.0.0.1，在浏览器上访问docker-vm的127.0.0.1。如图![image](https://github.com/mywoogle/docker-note/blob/master/image/2.png)
    
3.  容器操作

    3.1 用docker-machine start启动docker-vm，然后就可在里面操作了。
    
    3.2 创建容器
        
    创建并运行容器 docker run ubuntu：14.04,它会在docker-vm寻找ubuntu：14.04的镜像，若没有就从dockerhub上下载。
    查看容器 docker ps [-a] [-q] 不加参数查看正在运行的容器，加-a查看所有的容器，加-q查看容器id
    启动容器 docker start 容器id/名字
    添加客户端 docker attach 容器id/名字
    关闭容器 docker stop 容器id/名字
    删除容器 docker rm 容器id，coker rm $(docekr ps -a -q)删除所有的容器
    
    docker run的其它参数 --name=lampv2添加名字，--net=host容器使用docker-vm的网络设置，-p 80:80把容器的80端口映射到docker-vm的80端口，-v /c/Users/website:/var/www/html把docker-vm的/c/Users/website目录挂载到容器的/var/www/html目录进行共享，-i使用标准输入，-t启动交互式客户端。所以一般创建容器可能会用这个命令 docker run -it -v /c/Users/:/var/www/html/website --name=lampv3 --net=host woogle/lamp:v3 /bin/bash
    
4.  镜像操作

    4.1 导入镜像：cat lampv4.tar | docker import - woogle/lamp:v4,把lampv4.tar这个文件导入成，仓库名为woogle/lamp,版本我v4的镜像。导出镜像 docker export contain_id > customer.tar
    
    4.2 查看镜像： docker images
    
    4.3 commit创建镜像： 首先停止需要创建进行的容器。docker commit -m='messge' --author='woogle' 容器id woogle/lampv1:v1
   
    4.4 使用Dockerfile创建镜像：后面一点研究。
    
    4.5 删除镜像： docker rmi 镜像id
    
    5.6 这里是我导出来的一个lamp容器，放在百度云上的，http://pan.baidu.com/s/1dFNqMlF
    
    若果你要使用这个容器，请在你的工作目录下建一个website目录，并在website目录下建一个web目录存你的web代码，在website目录下建一个database目录用来存mysql数据。都穿创建好后，在docker-vm与windows的共享目录映射修改成映射到你的website目录上。然后后把这个文件导入成镜像，最后使用类似样子的命令docker run -it -v /c/Users/:/var/www/html/website --name=lampv3 --net=host woogle/lamp:v3 /bin/bash创建并启动容器。
    

# License
Feel free to use it.

# other
* 如何打开多个终端进入Docker容器 

```shell
    docker exec -it <container_id> /bin/bash
    #退出docker exec打开的终端“不会”关闭这容器，而退出attach打开的终端“会”关闭这个容器
```

* docker 容器中 crond服务启动后 无法执行
 
```shell
        docker宿主机系统版本
        # cat /etc/issue
        CentOS release 6.7 (Final)

        内核版本
        # uname -a
        Linux test 3.10.101-1.el6.elrepo.x86_64 #1 SMP Wed Mar 16 20:55:27 EDT 2016 x86_64 x86_64 x86_64 GNU/Linux

        容器也是同样的系统版本

        进入容器后安装crond服务：yum install crontabs
        启动程序：# /etc/init.d/crond start
        写入计划任务：
        # crontab -l
        */1 * * * * echo "aaaaaaaaaaaaa"  >> /tmp/test.log
        由于镜像最简化安装，所以crond程序是无日志的，此时等待几分钟时间是无法出现/tmp/test.log文件的，由此判断crond程序没有正常工作，我们需要修改文件如下：
        # cat /etc/pam.d/crond
        #session    required   pam_loginuid.so      #注释此行修改成下一行
        session    sufficient   pam_loginuid.so
        # /etc/init.d/crond restart
        Stopping crond:                                            [  OK  ]
        Starting crond:                                            [  OK  ]
        [root@BW-GL11 ~]# tailf /tmp/test.log     此时看到已经可以执行
        aaaaaaaaaaaaa
```

* 利用iptables给Docker绑定一个外网IP 

```shell
    

目标
给docker容器绑定公网ip。把公网ip（192.168.99.222）绑定到容器（171.17.0.2）上

环境
这是部署在win7上的docker toolbox上完成的操作。
物理机：win7系统（192.168.99.1）
宿主机：eth1（192.168.99.100）
容器ip：container_name(171.17.0.2)


操作
1为宿主机绑定多个ip
进入宿主机把192.168.99.222绑定到eth1:1上
ifconfig eth1:1 192.168.99.222

2通过DNAT来重写包的目的地址，将指向192.168.99.222的数据包的目的地址都改为172.17.0.2
iptables -t nat -A PREROUTING -d 192.168.99.222 -p tcp -m tcp --dport 1:65535 -j DNAT --to-destination 172.17.0.2:1-65535

3重写包的源IP地址，即在容器中收到数据包之后，将其源改为docker0的地址（172.17.42.1是典型的ip）。（要实现我们的目的，这一步不是必须的）
iptables -t nat -A POSTROUTING -d 172.17.0.2 -p tcp -m tcp --dport 1:65535 -j SNAT --to-source 172.17.42.1



注意：docker run -it -p 10.10.10.1.222:80:80  -p 10.10.10.1.221:3306:3306 --name=container_name lnmpv2:cenos6 /bin/bash，这个可以替代第二步，只是需要一个一个第绑定端口，而第二步，一次性绑定了端口1-65535.


补充

查看iptables    
iptables -n -t nat -L --line-numbers


删除规则
iptables –t nat –D PREROUTING <number>
iptables –t nat –D POSTROUTING <number>


资料
http://blog.csdn.net/shipengfei92/article/details/47089055
http://stackoverflow.com/questions/34688906/how-to-assign-static-public-ip-to-docker-container



```   
         
         
         
         
        
    
    
        
