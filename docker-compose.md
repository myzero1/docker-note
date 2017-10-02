# docker-compose的使用


```

docker-compose的使用
	1创建machine
		docker-machine create --virtualbox-disk-size "<the-size>" --driver virtualbox <machine-name>
		如：docker-machine create --virtualbox-disk-size "2048"  --driver virtualbox test

		注意：配置docker加速   https://www.daocloud.io/mirror#accelerator-doc
			docker-machine ssh test

			sudo sed -i "s|EXTRA_ARGS='|EXTRA_ARGS='--registry-mirror=http://6addb9c6.m.daocloud.io |g" /var/lib/boot2docker/profile

			exit

			docker-machine restart test

	2运行machine
		docker-machine start <machine-name>
		如：docker-machine start test

	3获取machine配置
		docker-machine env <machine-name>
		如：docker-machine env test

	4设置machine参数
		 eval $("D:\Program Files\Docker Toolbox\docker-machine.exe" env <machine-name>)
		 如： eval $("D:\Program Files\Docker Toolbox\docker-machine.exe" env test1)

	5进入docker-compose目录(docker-compose.yml文件所在的目录)
		如：cd /c/machine/docker-compose-env/docker-lnmp-3

	6运行docker-compose
		docker-compose build
		docker-compose up -d
		docker-compose down

	*注意
		上面的操作都是真对在git bash中的操作,若在Docker Quickstart Terminal中可以可以只执行5,6

		挂载的目录必须放到C:\users\username下面，因为docker只共享了这个目录。参考https://github.com/docker/compose/issues/2548

		把配置中的镜像换成daocloud.io的会快很多。镜像地址http://hub.daocloud.io/
		
	
```
