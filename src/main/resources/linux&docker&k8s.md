/bin ：最经常使用的命令

/home : 普通用户的主目录

/root：超级管理员根目录

/etc : 配置文件

###### vim:

yyp:复制一行

dd:删除行

d:删除一行

reboot:重启电脑

useradd xxx

passwd xxx：重设密码

userdel xxx

id xxx:查看用户信息

su - root:从权限高的用户切换到权限低的用户不需要输入密码，反之需要当需要返回到原来用户时，输入指令exit如果su- 没有带用户名，则默认切换为root用户

whoami：查看当前用户

> " >" a.txt = touch a.txt
>
> ls /home > a.txt:将home目录下的内容覆盖到a.txt,且改变a.txt的创建时间为当前。top:显示正在进行的进程，三秒刷新一次

whereis = type = which

docker pull xxx:从仓库拉取镜像

docker images:查看所有镜像

docker rmi -f 镜像id：删除镜像

docker run - it xxx /bin/bash : 以交互模式进入容器 (-it)

exit:退出容器

ctrl + p + q:退出容器但是容器不结束

docker run -d xxx:以后台模式运行容器





docker ps -aq:查看所有容器docker

-a:查看所有容器，包括没有运行的， -q：只查看容器id标识

docker exec -it  容器id:进入到运行的容器内部

docker stop  容器id：停止容器

docker start 容器id： 开始容器

docker run 镜像id：运行容器

docker rm -f 容器id：删除容器

>  -i保证我们的输入有效,即使在没有detach的情况下也能运行.
>
>  -t表示将分配给我们一个伪终端.我们将在伪终端输入我们的内容.

docker stop ContainnerId:

#### **docker-compose**:

创建一个docker-compose.yml文件来专门管理docker内各个容器的参数信息，启动docker时就能做到一键启动多个容器。

docker -build：构建镜像，创建dockerbuild文件：

> Dockerfile文件中常用的内容：
>
> from: 指定当前自定义镜像依赖的环境
> copy: 将相对路径下的内容复制到自定义镜像中
> workdir: 声明镜像的默认工作目录
> cmd: 需要执行的命令(在workdir下执行的，cmd可以写多个，只以最后一个为准)
>
> 举个例子，自定义一个tomcat镜像，并且将ssm.war部署到tomcat中
>
> from daocloud.io/library/tomcat:8.5.15-jre8
>
> copy ssm.war /usr/local/tomcat/webapps

docker volume：数据卷，用于映射寄主机中地址与容器中的地址。

### k8s

> k8s是一个编排容器的工具，其实也是管理应用的全生命周期的一个工具，从创建应用，应用的部署，应用提供服务，扩容缩容应用，应用更新，都非常的方便。而且可以做到故障自愈，例如一个服务器挂了，可以自动将这个服务器上的服务调度到另一个主机上进行运行，无需进行人工干涉。

一个k8s集群由一个master(用来调度，控制集群的资源，持有元数据)和多个node(运行容器的节点)组成，每个node持有多个pod，pod是k8s的集群调度的最小单元，一个pod可以持有一个容器，也能持有多个容器。

一个pod运行多个容器，相当于一个VM运行多个进程，pod的存在让几个紧密连接的几个容器之间共享资源，例如ip地址，共享存储等信息。

###### 发布应用：

​	发布应用就是对外提供服务，为什么运行了服务，还不能提供服务？因为在集群当中，每个pod的ip资源只有同一个集群中才能访问，每个pod也有唯一的ip地址，当有多个pod提供相同的服务的时候，就需要有负载均衡的能力。

一个服务(Service)的概念：

服务主要是用来提供外界访问的接口，服务可以关联一组pod，这些pod的ip地址各不相同，而service相当于一个负载均衡的vip。

springboot项目一般是以jar包得形式跑在Linux服务器上，但是在k8s中， 运行起来的不是jar，而是image，因此需要把jar包打成image。

```sql
CREATE DEFINER=`root`@`%` PROCEDURE `alter_view_counts`()
BEGIN
	#声明结束标识
	DECLARE end_flag int DEFAULT 0;
	
	DECLARE albumId bigint;
	
	#声明游标 album_curosr
	DECLARE album_curosr CURSOR FOR SELECT album_id FROM album;
	
	#设置终止标志
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET end_flag=1;
	
	#打开游标
	OPEN album_curosr;
	
	#遍历游标
	REPEAT
		#获取当前游标指针记录，取出值赋给自定义的变量
		FETCH album_curosr INTO albumId;
			#利用取到的值进行数据库的操作
			UPDATE album SET album.views_count= (SELECT SUM(light_chat.views_count) FROM `light_chat` WHERE light_chat.album_id = albumId) WHERE album.album_id = albumId;
	# 根据 end_flag 判断是否结束
	UNTIL end_flag END REPEAT;
	
	#关闭游标
	close album_curosr;
 
END
```

```sql
-- 创建存储过程之前需判断该存储过程是否已存在，若存在则删除
DROP PROCEDURE IF EXISTS init_reportUrl; 
-- 创建存储过程
CREATE PROCEDURE init_reportUrl()
BEGIN
	-- 定义变量
	DECLARE s int DEFAULT 0;
	DECLARE report_id varchar(255);
	DECLARE report_url varchar(256);
	-- 定义游标，并将sql结果集赋值到游标中
	DECLARE report CURSOR FOR select reportId,reportUrl from patrolReportHistory;
	-- 声明当游标遍历完后将标志变量置成某个值
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET s=1;
	-- 打开游标
	open report;
		-- 将游标中的值赋值给变量，注意：变量名不要和返回的列名同名，变量顺序要和sql结果列的顺序一致
		fetch report into report_id,report_url;
		-- 当s不等于1，也就是未遍历完时，会一直循环
		while s<>1 do
			-- 执行业务逻辑
			update patrolreporthistory set reportUrl = CONCAT('patrolReport.html?monitorId=',substring(report_url,15,1),'&reportId=',report_id) where reportId=report_id;
			-- 将游标中的值再赋值给变量，供下次循环使用
			fetch report into report_id,report_url;
		-- 当s等于1时表明遍历以完成，退出循环
		end while;
	-- 关闭游标
	close report;
END;
```

