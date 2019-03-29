#   
## 问题列表　

| 序号 | 问题 | 是否解决 |　　
|:---:|:---:|:---:|   
|1|如何使用Volatility框架实现对虚拟机的内存取证|是|     
|2|如何获取目标程序的进程信息|是| 　　
|３|如何定位目标程序的指定线程|是|　　
|４|如何定位指定线程对应的虚拟机栈|是| 　　
|５|如何成功分析下一个栈帧|是|　　
|６|如何分析对应的方法参数和局部内存|是|　　
|７|如何缩短Z3求解器的验证时间|是|　　
|８|如何对乱序事件进行验证|是|　　
|９|如何实现同时解析多线程程序中的多个线程||　　
|１０|如何将变量修改为可读取配置文件方式||　　
|１１|如何实现对编译型栈帧的解析||　　
|１２|确定脏数据的具体表现，复现及解决脏数据问题||　　
|１３|将存在冗余的代码部分进行优化||　　
|１４|确定模型验证过程中将字符串生成什么样的Z3表达式||

## 解决方法
### 1、如何使用Volatility框架实现对虚拟机的内存取证？　　
* ### 思路 　　

    volatility的标准用法是针对镜像文件的，但是将 volatility 配合 pyvmi 使用，就可以直接重构运行虚拟机的内存，而不用转储成文件后再重构。
* ### 实现　　　
  - 安装pycharm，克隆volatility工程（JavaMemory——volatility-2.6）到本地
  - 下载volatility依赖的包(具体详见JavaMemory—— README.md)　　
  - 制作profile（包含system.map和module.dwarf）文件：　　  
    + 将volatility工程拷到虚拟机中
    + sudo apt-get install dwarfdump
    + sudo apt-get install build-essential
    + sudo apt-mark hold 内核版本号，这里禁止了内核的自动升级，若虚拟机内核版本与overlays/linux目录下的压缩包不匹配，则无法成功获取虚拟机内存信息（虚拟中可通过 uname -a 命令查看内核版本）
    + cd volatility/tools/linux 并 make
    + head module.dwarf
    + sudo zip volatility-2.6/volatility/plugins/overlays/linux/Ubuntu系统版本号_内核版本号.zip volatility-2.6/tools/linux/module.dwarf /boot/System.map-内核版本号
    + 将生成的zip文件拷到宿主机的 volatility-2.6/volatility/plugins/overlays/linux目录下（scp Ubuntu1604_138.zip sqq@10.108.:/home/sqq/.../Linux）  
  - volatitlity工程 Pycharm配置参数:  
    （如果虚拟机为64位）: -l vmi://ubuntu --profile=LinuxUbuntu1604_内核版本号x64 linux_pslist  
    （如果虚拟机为32位): -l vmi://ubuntu12.04_32bit--profile=LinuxUbuntu1204_23x86 linux_pslist   
    其中 vmi 代表虚拟机名称，profile 代表在虚拟机内部打包生成的profile名称，x64、x86 代表虚拟机位数
  - 运行即可获取虚拟机进程信息   
### ２、如何获取目标程序的进程信息？　　

### ３、如何定位目标程序的指定线程？　　

### ４、如何定位指定线程对应的虚拟机栈?　　

### ５、如何成功分析下一个栈帧？　　

### ６、如何分析对应的方法参数和局部内存？　　

### ７、如何缩短Z3求解器的验证时间？　　

### ８、如何对乱序事件进行验证？　　

### ９、如何实现同时解析多线程程序中的多个线程？　　
* ### 思路　　
    调用JDI.jar中的PyDump类中定义的initJavaFirstFPAddress方法获取所有线程的栈底栈帧，同单线程分析原理一样依次分析所有线程。　　
* ### 代码定位与实现　　
    线程分析的实现定位到JavaMemory工程中的
JavaMemory/volatility-2.6/volatility/plugins/linux/linux_runtime.py中。     
### １０、如何将变量修改为可读取配置文件方式？　　
* ### 思路　　
    python有一个配置模块ConfigParser，可以处理配置文件信息。　　
* ### 实现　　
    在工程中新建一个配置模块，在该模块中创建一个.ini文件,将需要配置的变量编写在里面，然后新建一个模块调用ConfigParser模块中方法读取.ini文件中的变量值。
### １１、如何实现对编译型栈帧的解析？　　

### １２、确定脏数据的具体表现，复现及解决脏数据问题。　　
* ### 思路　　
    工程在每次分析虚拟机内存信息之前，首先通过volatility框架结合libvmi来获取虚拟机内存信息，但是libvmi获取虚拟机内存无需挂起，导致部分栈帧是上一时刻的，另一部分栈帧是当前时刻的，从而使libvmi获取到的数据不够明确清晰，总有一部分是多余的，即脏数据问题。
### １３、将存在冗余的代码部分进行优化。　　
* ### 思路　　　
    第一：内存重构工程中python 代码部分的优化，内存重构分析工程是基于Volatility框架用python开发的，所以系统性能优化要结合python编程特性，首先可以结合网上python性能优化的一些建议优化代码，其次利用一些性能分析工具定位工程中的性能瓶颈，针对性能瓶颈再做优化。　　

    第二：物联网平台监控系统中c++代码部分的优化，c++程序的内存申请和释放都是由开发者控制的，所以在系统性能优化时需要研读代码，查找内存申请集中的代码块，审核代码逻辑，优化程序内存占用。　　　　　　　　
### １４、确定模型验证过程中将字符串生成什么样的Z3表达式。　　
