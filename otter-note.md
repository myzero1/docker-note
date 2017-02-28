##otter-note
#note
```pre

大平台，分布式架构，数据同步（阿里的Otter&Canal）

	低级节点全部表同步到高级节点
	高级节点部分必须的配置表同步到每个低级节点

	问题：
		缺少抽取组件  ---- 可以暂时不处理，因为下发的数据不多

		配置数据写入优先权设置 ----- 把节点id配置成省市区的id，上级可以修改，下级的数据。

		叶子节点的sqe=num的处理 ----- 定时跑脚本去更新，或者把sqe设置成某一个可以一修改就自增的默认值

		过滤解析：需要改otter的源码，有一定的难度。


		



sudo mount -t vboxsf website_test /website_test


在虚拟机里面构造几个分布式系统
	建一个有图形界面的ubuntu系统，作为分布式系统的宿主机	ok
	在宿主机上创建容器做作为分布式的各个节点并配置固定ip 	p1,c1,a1 ok

		sudo docker run -it --name=p1 -v /website_test/otter_canal:/website_test/otter_canal kunming2:v1 /bin/bash

		sudo docker start p1

		sudo pipework docker0 p1 172.17.0.231/24@172.17.0.1


		sudo docker run -it --name=c1 otter:v1 /bin/bash
		sudo docker start c1
		sudo pipework docker0 c1 172.17.0.221/24@172.17.0.1

		sudo docker run -it --name=c2 otter:v1 /bin/bash
		sudo docker start c2
		sudo pipework docker0 c2 172.17.0.222/24@172.17.0.1

		sudo docker run -it --name=a1 kunming2:v1 /bin/bash
		sudo docker start a1
		sudo pipework docker0 c1 172.17.0.211/24@172.17.0.1



		sudo docker run -it --name=a11 otter:v1 /bin/bash
		sudo docker start a11
		sudo pipework docker0 c11 172.17.0.211/24@172.17.0.1

	配置p1
		配置zeekeeper
		配置otter
		配置canel

	配置c1
		配置otter
		配置canel

	配置a1
		配置otter
		配置canel


	配置普通节点
	配置管理节点

	




other
	sudo docker stop $(sudo docker ps -a -q)

	otter中的问题
		多对一：
			同主键会被替换，只有给不同主键步长来解决。 id的起始值为数据源的id，步长暂定位100(江苏省有13个地级城市，52个县级城市)对于单个省基本够用了。
		3层以上的同步：
			单向多层可以行
			双向多层，不支持。
				拆分双向为单向，架构2个zookeeper，尝试。

		总结：
			多层双向同步解决方案：转换为多层单向来实现
				每个节点的mysql的server-id和auto_increment_offset等于manager中的node-id，以避免双向写导致的auto_crement主键冲突。具体实现，在my.cnf的[mysqld]下面加上下面代码
					character_set_server=utf8
					collation_server=utf8_bin

					auto_increment_increment = 100 #表示中节点数为100
					auto_increment_offset = [node-id]

				正确设置canal的权限
					GRANT USAGE ON *.* TO `canal`@'%';

					GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO `canal`@'%';

					GRANT SELECT, INSERT, UPDATE, DELETE, EXECUTE ON `my_test`.* TO `canal`@'%';

					flush privileges;

					show grants for 'canal';

			需要使用的端口号及定义
				2088
				9090
				2181
				3306
				1099

				8080
				80
				8090	manager的访问端口


				sudo vi /etc/network/interface
					auto eth0:0
					iface eth0:0 inet static
					address 110.25.*.*

192.168.0.242-149
192.168.135.242-149
234-241

				weave run 192.168.135.242/24 -it --name=test0 -p 192.168.0.242:2088:2088 -p 192.168.0.242:9090:9090 -p 192.168.0.242:2181:2181 -p 192.168.0.242:3306:3306 -p 192.168.0.242:8080:8080 -p 192.168.0.242:80:80 -p 192.168.0.242:1099:1099 -p 192.168.0.242:8090:8090 otter /bin/bash

				weave run 192.168.135.243/24 -it --name=test1 -p 192.168.0.243:2088:2088 -p 192.168.0.243:9090:9090 -p 192.168.0.243:2181:2181 -p 192.168.0.243:3306:3306 -p 192.168.0.243:8080:8080 -p 192.168.0.243:80:80 -p 192.168.0.243:8090:8090 otter /bin/bash

				weave run 192.168.135.244/24 -it --name=test2 -p 192.168.0.244:2088:2088 -p 192.168.0.244:9090:9090 -p 192.168.0.244:2181:2181 -p 192.168.0.244:3306:3306 -p 192.168.0.244:8080:8080 -p 192.168.0.244:80:80 -p 192.168.0.244:8090:8090 otter /bin/bash


			持久文件定义
				为每个容器，建立一个和容器名称一致的文件夹
				把对应的文件夹挂载到容器里面
					数据库文件
					web文件
					nginx配置文件
					otter配置文件










\----------------------------------------------


在channel id大于等于1000的channel下面添加pipi会报错-----通过升级软件是否能解决？

测试在一个节点上添加多个canal，是否是各种独立的----可以各种独立。下面的效果可以达到。在一对多的节点上建立多个canal，每个对应关系上用相应的canal，在一对多的多个节点中任意一个节点down了，只会影响down的这个节点。











234-241

				weave run 192.168.135.234/24 -it --name=test234 -p 192.168.0.242:2088:2088 -p 192.168.0.242:9090:9090 -p 192.168.0.242:2181:2181 -p 192.168.0.242:3306:3306 -p 192.168.0.242:8080:8080 -p 192.168.0.242:80:80 -p 192.168.0.242:1099:1099 -p 192.168.0.242:8090:8090 otter /bin/bash

				weave start 192.168.0.234/24 $(sudo docker ps -a -q -f name=test234)


				weave run 192.168.135.235/24 -it --name=test235  -p 192.168.0.235:3306:3306 -p 192.168.0.235:8080:8080 -p 192.168.0.235:80:80 -p 192.168.0.235:8090:8090 otter /bin/bash

				weave start 192.168.0.235/24 $(sudo docker ps -a -q -f name=test235)


				weave run 192.168.135.236/24 -it --name=test236  -p 192.168.0.236:3306:3306 -p 192.168.0.236:8080:8080 -p 192.168.0.236:80:80 -p 192.168.0.236:8090:8090 otter /bin/bash

				weave start 192.168.0.236/24 $(sudo docker ps -a -q -f name=test236)









----------------------

	weave run 192.168.135.240/24 -it --name=test0 -p 192.168.1.240:2088:2088 -p 192.168.1.240:9090:9090 -p 192.168.1.240:2181:2181 -p 192.168.1.240:3306:3306 -p 192.168.1.240:8080:8080 -p 192.168.1.240:80:80 -p 192.168.1.240:8090:8090 otter /bin/bash

	weave run 192.168.135.240/24 -it --name=test0 -p 192.168.1.240:2088:2088 -p 192.168.1.240:9090:9090 -p 192.168.1.240:2181:2181 -p 192.168.1.240:3306:3306 -p 192.168.1.240:8080:8080 -p 192.168.1.240:80:80 -p 192.168.1.240:8090:8090 otter /bin/bash

	weave run 192.168.135.240/24 -it --name=test0 -p 192.168.1.240:2088:2088 -p 192.168.1.240:9090:9090 -p 192.168.1.240:2181:2181 -p 192.168.1.240:3306:3306 -p 192.168.1.240:8080:8080 -p 192.168.1.240:80:80 -p 192.168.1.240:8090:8090 otter /bin/bash








				weave run 192.168.1.240/24 -it --name=wc1 ubuntu /bin/bash

				weave run 192.168.1.240/24 -it --name=test3 -p 192.168.1.243:2088:2088 -p 192.168.1.243:9090:9090 -p 192.168.1.243:2181:2181 -p 192.168.1.243:3306:3306 -p 192.168.1.243:8080:8080 -p 192.168.1.243:80:80 -p 192.168.1.243:8090:8090 otter /bin/bash


-------------family---------------
192.168.1.230---

weave run 192.168.135.231/24 -it --name=test231 -p 192.168.1.231:2088:2088 -p 192.168.1.231:9090:9090 -p 192.168.1.231:2181:2181 -p 192.168.1.231:3306:3306 -p 192.168.1.231:8080:8080 -p 192.168.1.231:80:80 -p 192.168.1.231:1099:1099 -p 192.168.1.231:8090:8090 otter /bin/bash

weave start 192.168.135.231/24 $(sudo docker ps -a -q -f name=test231)



weave run 192.168.135.232/24 -it --name=test232 -p 192.168.1.232:3306:3306 -p 192.168.1.232:8080:8080 -p 192.168.1.232:80:80 -p 192.168.1.232:8090:8090 otter /bin/bash

weave start 192.168.135.232/24 $(sudo docker ps -a -q -f name=test232)


weave run 192.168.135.233/24 -it --name=test233 -p 192.168.1.233:3306:3306 -p 192.168.1.233:8080:8080 -p 192.168.1.233:80:80 -p 192.168.1.233:8090:8090 otter /bin/bash

weave start 192.168.135.233/24 $(sudo docker ps -a -q -f name=test233)




	有用的知识
		docker容器绑定独立的公网ip
			给宿主机现有公网ip所在的网卡绑定多个公网ip，后面新建容器时就在绑定的这一些公网ip中挑选一个公网ip进行绑定
				ifconfig eth0:0 192.168.0.240 netmask 255.255.255.0 up
				ifconfig eth0:1 192.168.0.241 netmask 255.255.255.0 up
				ifconfig eth0:2 192.168.0.242 netmask 255.255.255.0 up

			在新建docker容器时，使用-p参数进行公网ip和端口的绑定
				docker run -it --name=test240 -p 192.168.0.240:80:80 -p 192.168.0.240:3306:3306 -p 192.168.0.240:2088:2088 ubuntu:last /bin/bash

		Docker使用weave实现跨主机容器连接
			使用weave进行处理(docker容器直接可以通过公网ip连接吗？)

			环境如下：
				win7+virtualbox
				两台ubuntu16.04虚拟机
				双网卡，Host-Only & NAT
				IP地址：
					Host1:192.168.59.103
					Host2:192.168.59.104

			具体操作如下:

				在192.168.59.103上的操作

					sudo wget -O /usr/bin/weave https://github.com/weaveworks/weave/releases/download/latest_release/weave

					sudo chmod a+x /usr/bin/weave

					weave launch


				在192.168.59.104上的操作

					sudo wget -O /usr/bin/weave https://github.com/weaveworks/weave/releases/download/latest_release/weave

					sudo chmod a+x /usr/bin/weave

					weave launch 192.168.59.103

					c2=$(weave run 192.168.1.2/24 -it --name=host2.1 ubuntu /bin/bash)

					docker attache $c2  (或者用docker attache host2.1)

					ifconfig

				在192.168.59.103上的操作

					weave run 192.168.1.10/24 -it --name=wc1 ubuntu /bin/bash

					docker attach wc1

					ifconfig

					ping 192.168.1.2


					









			


weave run 192.168.1.240/24 -it --name=wc1 ubuntu /bin/bash


grant all privileges on my_test.* to canal@'%' identified by 'canal';

grant all privileges on my_test.* to canal@host identified by 'canal';

grant select,insert,update,delete on my_test.* to canal@"%" identified by "canal";

 show grants for 'canal';

grant select,insert,update,delete on my_test.t_place to  'canal'@'172.17.0.221' identified by "canal";

GRANT USAGE ON *.* TO `canal`@'%';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO `canal`@'%';
GRANT SELECT, INSERT, UPDATE, DELETE, EXECUTE ON `my_test`.* TO `canal`@'%';
flush privileges;



character_set_server=utf8
collation_server=utf8_bin

auto_increment_increment = 100
auto_increment_offset = 1











	/home/woogle/otter_canel/zookeeper/data


	/usr/local/java/jdk1.7.0_25


export JAVA_HOME=/usr/local/java/jdk1.7.0_25
export JRE_HOME=/usr/local/java/jdk1.7.0_25/jre
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin: $PATH



























province 	p
The city	c
area		a


sudo docker start p1
sudo pipework docker0 c1 172.17.0.221/24@172.17.0.1

sudo docker start c1
sudo pipework docker0 c1 172.17.0.221/24@172.17.0.1

sudo docker start a1
sudo pipework docker0 c1 172.17.0.211/24@172.17.0.1
















sudo docker network  create  -d bridge --subnet=192.168.99.99/24 -o parent=enp0s9 br9
sudo docker network  create  -d bridge --subnet=192.168.56.88/24 -o parent=enp0s8 br8


sudo docker network  create  -d bridge --subnet=192.168.210.0/24 --gateway=192.168.210.1 -o parent=eth0 br1
22dda6f921c9f9d539a9141777c64a4e36e967e665dd65c29fc194d1e6e1f99a


sudo docker run -it -d --net=br1 --ip=192.168.210.3 --name=nginx2 nginx:1.10-alpine


sudo docker run -it --net=bridge --ip=192.168.99.33 --name=test33 kunming2:v1 /bin/bash







auto enp0s8
iface enp0s8 inet static
address 192.168.56.135
netmask 255.255.255.0
#gateway 192.168.56.254
#bridge_ports em1
#bridge_stp off
#idns-nameservers 8.8.8.8 192.168.56.1



auto enp0s8
iface enp0s8 inet static
address 192.168.56.135
netmask 255.255.255.0




sudo mount -t vboxsf d /data
sudo vi /etc/network/interfaces
sudo /etc/init.d/networking  restart




VBoxManage modifyhd "C:\Users\Administrator\VirtualBox VMs\ubuntu16.04 desktop\ubuntu16.04 desktop.vdi" --resize 20000




INSERT IGNORE `WebKeyWordList` SELECT
	*
FROM
	`WebKeyWordList`


insert into retl.retl_buffer(ID,TABLE_ID, FULL_NAME,TYPE,PK_DATA,GMT_CREATE,GMT_MODIFIED) (select null,0,'$cultrur_server.WebKeyWordList$','I',id,now(),now() from $cultrur_server.WebKeyWordList$); 


INSERT INTO retl.retl_buffer (
	ID,
	TABLE_ID,
	FULL_NAME,
	TYPE,
	PK_DATA,
	GMT_CREATE,
	GMT_MODIFIED
)(
	SELECT
		NULL,
		0,
		'culture_server.WebKeyWordList',
		'I',
		id,
		now(),
		now()
	FROM
		culture_server.WebKeyWordList
);


不重复插入,新加入数据

INSERT IGNORE retl.retl_buffer (
	ID,
	TABLE_ID,
	FULL_NAME,
	TYPE,
	PK_DATA,
	GMT_CREATE,
	GMT_MODIFIED
)(
	SELECT
		ID,
		0,
		'culture_server.WebKeyWordList',
		'I',
		id,
		now(),
		now()
	FROM
		culture_server.WebKeyWordList
);


替换一条数据，触发自由门全量同步表

REPLACE retl.retl_buffer (
	ID,
	TABLE_ID,
	FULL_NAME,
	TYPE,
	PK_DATA,
	GMT_CREATE,
	GMT_MODIFIED
)(
	SELECT
		ID,
		0,
		'culture_server.WebKeyWordList',
		'I',
		id,
		now(),
		now()
	FROM
		culture_server.WebKeyWordList
	LIMIT 1
);










ALARM_LOG
CfgDetail
ChatKeyWordList
ClientTask
CommitStatus
GameList
MESSAGE_LOG
NETLOG_20161026
NETONLINE_LOG
PunishList
SelectMessage
VIRTUALID_LOG
-WebKeyWordList
WebList
anti_addiction
business_hour
commit
customer
customer_blacklist
customer_record
game_record
netlogstatics
place
place_record
place_touch_record
process_list
sequence
statistic-
steward_company-
tSysArea-
user_action_log-
users-
users_limit_point-
users_role-


///////////////////
WebKeyWordList--=
CfgDetail--=
ChatKeyWordList--=
ClientTask--=
CommitStatus--
GameList--
PunishList--
SelectMessage--
WebList--
business_hour--
commit--
customer--
customer_blacklist--
customer_record--
game_record--
place--
place_record--
place_touch_record--
statistic--
steward_company--
tSysArea--
user_action_log--
users--
users_limit_point--
users_role--



netlogstatics
MESSAGE_LOG
NETLOG_20161026
NETONLINE_LOG
VIRTUALID_LOG
anti_addiction
process_list
sequence



//、、、、、数据表配置模板、、、、、、、、、、、、、

INSERT INTO DATA_MEDIA
VALUES
	(
		NULL,
		'user_action_log',
		'culture_server',
		'{"mode":"SINGLE","name":"user_action_log","namespace":"culture_server","source":{"driver":"com.mysql.jdbc.Driver","encode":"UTF8","gmtCreate":1487657657000,"gmtModified":1487657657000,"id":1,"name":"test234_p1","password":"canal","type":"MYSQL","url":"jdbc:mysql://192.168.135.234:3306","username":"canal"}}',
		1,
		'2017-02-22 16:38:49',
		'2017-02-22 16:38:49'
	),
	(
		NULL,
		'user_action_log',
		'culture_server',
		'{"mode":"SINGLE","name":"user_action_log","namespace":"culture_server","source":{"driver":"com.mysql.jdbc.Driver","encode":"UTF8","gmtCreate":1487657667000,"gmtModified":1487657667000,"id":2,"name":"test235_c1","password":"canal","type":"MYSQL","url":"jdbc:mysql://192.168.135.235:3306","username":"canal"}}',
		2,
		'2017-02-22 16:38:49',
		'2017-02-22 16:38:49'
	),
	(
		NULL,
		'user_action_log',
		'culture_server',
		'{"mode":"SINGLE","name":"user_action_log","namespace":"culture_server","source":{"driver":"com.mysql.jdbc.Driver","encode":"UTF8","gmtCreate":1487657677000,"gmtModified":1487657677000,"id":3,"name":"test236_c2","password":"canal","type":"MYSQL","url":"jdbc:mysql://192.168.135.236:3306","username":"canal"}}',
		3,
		'2017-02-22 16:38:49',
		'2017-02-22 16:38:49'
	),
	(
		NULL,
		'user_action_log',
		'culture_server',
		'{"mode":"SINGLE","name":"user_action_log","namespace":"culture_server","source":{"driver":"com.mysql.jdbc.Driver","encode":"UTF8","gmtCreate":1487657688000,"gmtModified":1487657688000,"id":4,"name":"test237_a1","password":"canal","type":"MYSQL","url":"jdbc:mysql://192.168.135.237:3306","username":"canal"}}',
		4,
		'2017-02-22 16:38:49',
		'2017-02-22 16:38:49'
	),
	(
		NULL,
		'user_action_log',
		'culture_server',
		'{"mode":"SINGLE","name":"user_action_log","namespace":"culture_server","source":{"driver":"com.mysql.jdbc.Driver","encode":"UTF8","gmtCreate":1487657699000,"gmtModified":1487657699000,"id":5,"name":"test238_a2","password":"canal","type":"MYSQL","url":"jdbc:mysql://192.168.135.238:3306","username":"canal"}}',
		5,
		'2017-02-22 16:38:49',
		'2017-02-22 16:38:49'
	)



	/////////////////////批量添加映射关系//////

	Schema Name:culture_serverTable Name:WebKeyWordListSource Name:test234_p1

culture_server,WebKeyWordList,1,2,5
culture_server,CfgDetail,1,2,5
culture_server,ChatKeyWordList,1,2,5
culture_server,ClientTask,1,2,5
culture_server,CommitStatus,1,2,5
culture_server,GameList,1,2,5
culture_server,PunishList,1,2,5
culture_server,SelectMessage,1,2,5
culture_server,WebList,1,2,5
culture_server,business_hour,1,2,5
culture_server,commit,1,2,5
culture_server,customer,1,2,5
culture_server,customer_blacklist,1,2,5
culture_server,customer_record,1,2,5
culture_server,game_record,1,2,5
culture_server,place,1,2,5
culture_server,place_record,1,2,5
culture_server,place_touch_record,1,2,5
culture_server,statistic,1,2,5
culture_server,steward_company,1,2,5
culture_server,tSysArea,1,2,5
culture_server,user_action_log,1,2,5
culture_server,users,1,2,5
culture_server,users_limit_point,1,2,5
culture_server,users_role,1,2,5



序号	数据源名字	类型	编码	URL	操作
5	test238_a2	MYSQL	UTF8	jdbc:mysql://192.168.135.238:3306	查看 | 编辑 |删除
4	test237_a1	MYSQL	UTF8	jdbc:mysql://192.168.135.237:3306	查看 | 编辑 |删除
3	test236_c2	MYSQL	UTF8	jdbc:mysql://192.168.135.236:3306	查看 | 编辑 |删除
2	test235_c1	MYSQL	UTF8	jdbc:mysql://192.168.135.235:3306	查看 | 编辑 |删除
1	test234_p1







上下
WebKeyWordList
CfgDetail
ChatKeyWordList
GameList
WebList
business_hour
customer_blacklist
place
steward_company
tSysArea
user_action_log
users
users_limit_point
users_role
PunishList
CommitStatus
SelectMessage
ClientTask


只上
customer
customer_record
game_record
statistic
place_touch_record
place_record


只下










----



commit是一个view












```
