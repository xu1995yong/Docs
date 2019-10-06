## 镜像
### 获取镜像

  从 Docker 镜像仓库获取镜像的命令是 docker pull 。其命令格式为：    
	docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
	
	镜像仓库地址：格式一般是 <域名/IP>[:端口号] 。默认地址是 Docker Hub。
 	仓库名:格式<用户名>/<软件名>.对于 Docker Hub，如果不给出用户名，则默认为 library ，也就是官方镜像。

### 运行镜像

	后台运行：添加 -d 参数

### 列出镜像
​	docker image ls
​	docker image ls 仓库名

### 删除本地镜像
​	docker image rm [选项] <镜像1> [<镜像2> ...]
​	其中， <镜像> 可以是 镜像短 ID 、 镜像长 ID 、 镜像名 或者 镜像摘要 。




## 容器
容器是 Docker 又一核心概念。    
简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。    
要点：容器 = 镜像 + 可读层。并且容器的定义并没有提及是否要运行容器。 
### 启动容器
​	docker run -it 镜像id

-	当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：    

		检查本地是否存在指定的镜像，不存在就从公有仓库下载    
		利用镜像创建并启动一个容器    
		分配一个文件系统，并在只读的镜像层外面挂载一层可读写层    
		从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去    
		从地址池配置一个 ip 地址给容器    
		执行用户指定的应用程序    
		执行完毕后容器被终止   
        
### 启动容器并在后台运行
	docker run -it -d 镜像id
### 启动已终止容器
	docker container start 容器id
### 查看所有状态的容器
	docker container ls -a
### 终止容器
	docker container stop  容器id
### 进入容器
	docker exec -it 容器id bash
### 删除处于终止状态的容器
	docker container rm 容器id
### 删除运行中的容器
	docker container rm -f 容器id
### 清理所有处于终止状态的容器
	docker container prune


## Docker 中的网络功能
Docker 允许通过外部访问容器或容器互联的方式来提供网络服务。    
容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 -P 或 -p 参数来指定端口映射。    
	
	docker run -d -P training/webapp python app.py
可以通过 docker logs 命令来查看应用的日志信息。

	docker logs -f 容器id
使用 docker port 来查看当前映射的端口配置
	
	docker port 容器id


docker run -p 6379:6379 -v $PWD/data:/data  -d redis  redis-server --appendonly yes