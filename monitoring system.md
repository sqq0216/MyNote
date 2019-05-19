
# 安全监控系统迭代记录

## 第一阶段（vmsystem工程的测试与调试）

### 1.每次获取到的SSDT系统调用表总是改变，导致md5值不同，总是执行策略．
　　
- 解决：打印每次获取到的初始ssdt调用表，发现列表最后有空格，将空格和获取时间删掉即可．
　　
### ２．策略执行（通过socket连接）失败

- 解决：改用ssh方式执行，模拟linux_runtime中的sshcmd方法实现．

## 第二阶段（获取本地内存信息）

### 实现方法

- 制作本地profiel文件，放在volatility相关目录下
　　
- 通过LiME工具获取本地内存镜像，具体步骤详见［LiME使用］(https://www.jianshu.com/p/5f13ee20f5b0) 

- 重新制作ubuntu.lime文件的步骤：　　

  - rm ~/images／ubuntu.lime
  - cd ~/Cprojects/LIME/LiME/src
  - make clean
  - rm lime-4.15.0-20-generic.ko
  - make
  - cd /
  - sudo insmod /home/sqq/Cprojects/LIME/LiME/src/lime-4.15.0-20-generic.ko "path=/home/sqq/ubuntu3.lime format=lime"

### 遇到的问题：

1. insmod error:can't insert moduel lime-4.15.0-20-generic.ko

- 解决:make clean 之后重新make制作镜像

2. ImportError: cannot import name:...

- 原因分析：可能出现再模块导入的顺序问题上， 比如：在A文件头执行到语句 from B import XXX ，程序马上就会转到B文件中去，从头到尾顺序寻找B文件中的XXX函数，而A文件就暂停执行，直到把XXX函数复制到内存中，但B文件中的文件头可能也有导入， 如果B文件头中又导入了A文件中的函数，由于XXX函数还没有被复制。所以于A文件因为暂停执行而无法导入，就会出现上面的错误了。

### 遗留问题

1. 策略执行通过命令实现，但是不同的进程打开和关闭的命令或方式不同，有待优化，形成统一的关闭开启方式.

2. 监控本地计算机无恢复计算机的功能

3. 展示界面中本地与虚拟机信息未明确分布

