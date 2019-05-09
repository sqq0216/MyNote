
# 安全监控系统迭代记录

## 第一阶段（vmsystem工程的测试与调试）

### 1.每次获取到的SSDT系统调用表总是改变，导致md5值不同，总是执行策略．
　　
- 解决：打印每次获取到的初始ssdt调用表，发现列表最后有空格，将空格和获取时间删掉即可．
　　
### ２．策略执行（通过socket连接）失败

- 解决：改用ssh方式执行，模拟linux_runtime中的sshcmd方法实现．

### 遗留问题

1.策略执行通过命令实现，但是不同的进程打开和关闭的命令或方式不同，有待优化，形成统一的关闭开启方式．　　

## 第二阶段（获取本地内存信息）

### 实现方法

- 制作本地profiel文件，放在volatility相关目录下
　　
- 通过LiME工具获取本地内存镜像，具体步骤详见［LiME使用］(https://www.jianshu.com/p/5f13ee20f5b0)