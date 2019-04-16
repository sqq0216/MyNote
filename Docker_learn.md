# docker内存取证  
## 1．ubuntu14.06/16.04 安装docker: https://blog.csdn.net/yx_222/article/details/80936757    
## ２.将可运行的jar包 制作成docker image镜像：https://blog.csdn.net/summerxiachen/article/details/82595262   
* 遇到的问题：   
镜像制作完成后，启动容器（执行 sudo docker　run threadtest）时报错　　　　
    - error:could not find or load main class FuncTest　　　
* 解决：  
DockerFile书写错误，以下两条语句中的两个路径要保持一致  
    - WORKDIR /　(指定命令执行目录为／)；  
    - ADD ThreadTest.jar 　/ThreadTest.jar(将jar包复制到／目录下)　　　
### Q：运行volatility框架尝试获取虚拟机中docker的信息，但是只能获取到dockerd（docker后台的真正的进程)进程ID，但无法获取到运行中容器中的被测试java程序的信息  
### 注:容器＝镜像＋可读可写层　　
- docker create 命令为指定的镜像（image）添加了一个可读写层，构成了一个新的容器。注意，这个容器并没有运行。  
- Docker start命令为容器文件系统创建了一个进程隔离空间。注意，每一个容器只能够有一个进程隔离空间。  
- docker run = docker create + docker start  
## ３．docker的进程管理：　 　
在Docker中，每个Container都是Docker Daemon的子进程，每个Container进程缺省都具有不同的PID名空间。
* 运行容器－sudo docker --name test threadtest/v:2(下一次可直接通过sudo docker container start test启动该容器)
* 进入容器内部查看容器中的进程信息-sudo docker exec tst ps -ef　　
* 在宿主机中查看容器中的进程信息-sudo docker top test　　　
* 在宿主机端以树形式查看容器进程id结构信息－pstree -p 801（pid of congtainerd）


