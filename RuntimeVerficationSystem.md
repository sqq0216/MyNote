#   
## 问题列表　

| 序号 | 问题 | 是否解决 |　　
|:---:|:---:|:---:|   
|1|如何使用Volatility框架实现对虚拟机的内存取证|是|     
|2|如何获取目标程序的进程信息|是| 　　
|3|如何定位目标程序的指定线程|是|　　
|4|如何定位指定线程对应的虚拟机栈|是| 　　
|5|如何成功分析下一个栈帧|是|　　
|6|如何分析对应的方法参数和局部内存|是|　　
|7|如何缩短Z3求解器的验证时间|是|　　
|8|如何对乱序事件进行验证|是|　　
|9|如何实现同时解析多线程程序中的多个线程||　　
|10|如何将变量修改为可读取配置文件方式||　　　　
|11|确定脏数据的具体表现，复现及解决脏数据问题||　　
|12|将存在冗余的代码部分进行优化||　　
|13|确定模型验证过程中将字符串生成什么样的Z3表达式||

## 解决方法
### 1、如何使用Volatility框架实现对虚拟机的内存取证？　　
* ### 思路 　　
    volatility 可用来将内存语义重构，但volatility的标准用法是针对镜像文件的，libvmi 是一个虚拟机自省库，可用来访问运行虚拟机的内存，pyvmi基于libvmi，并提供python接口，可以方便的通过它与其它使用python的工具连接。将其配合 pyvmi 使用（之后将Volatility和pyvmi统称为Volatility框架）可直接获取运行虚拟机的内存信息，而不用将虚拟机信息转储成文件再进行重构，可实现对虚拟机内存的实时取证。
* ### 具体实现　　　
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
    注：其中 vmi 代表虚拟机名称，profile 代表在虚拟机内部打包生成的profile名称，x64、x86 代表虚拟机位数
  - 运行即可获取虚拟机进程信息   
### ２、如何获取目标程序的进程信息？　　
* ### 思路    
    Linux操作系统是通过进程链表（task_struct）来控制进程的。通过Volatility框架获取内核第一个进程init_task进程的逻辑地址，进而通过遍历进程链表找到所有的java程序，存在一个列表中。所有的java程序在进程信息中都被命名为java,且每个进程的ID都是变化不固定的，所以这里可以通过每次返回列表中的第一个或最后一个进程来进行分析。      
* ### 具体实现    
    该部分功能在linux_runtime.py模块中定义的allprocs(self)、calculate(self)、render_text(self, outfd, data)三个方法中实现，各个方法的功能如下:   
    + allprocs(self)：获取linux 内核第一个进程 init_task，然后遍历进程链表。  
    + calculate(self)：获取所有的java进程，存在tasks进程列表中。
    + render_text(self, outfd, data)：返回tasks列表中的最后一个进程。
### ３、如何定位目标程序的指定线程？　　
* ### 思路  
    每个线程都有其独立的一个栈空间，保存其运行状态和局部自动变量。 Serviceability Agent（sa-jdi.jar） 中已经提供好一些类、接口用于分析Java程序的堆栈信息。因此可以在 Volatility 框架中调用 Serviceability Agent 接口去获取分析堆栈信息。
* ### 具体实现  
    + 在项目中导入JDI.jar、sa-jdi.jar；  
    + 通过Volatility框架获取JVM信息——；  
    + 重写Serviceability Agent的初始化流程，将其初始化信息直接替换为Volatility框架获取的JVM信息，使其直接可以分析指定的目标程序；  
    + 使用Serviceability Agent中定义好的类、接口实现一些方法用于分析堆栈信息，并将其统一封装在 sun.tools.python.PyDump 类中供 python 部分调用；    
    + 借助JPype工具实现Volatility框架对Serviceability Agent的调用——；  
    + 在linux_runtime.py模块中调用PyDump类中的initJavaFirstFPAddress(threadName, cached)方法获取指定线程的栈底栈针，实现对指定线程的定位。
### ４、如何定位指定线程对应的虚拟机栈?　　
* ### 思路  
    前面已经实现了线程的定位，如通过self.first_fp = PyDump.initJavaFirstFPAddress("main", True)就可以获取到主线程的栈底栈针。然后根据栈底栈针就可以读取线程对应的虚拟机栈信息。  
* ### 具体实现  
    在linux_runtime.py模块中定义了readMemory方法，传入指定线程的栈底栈针和读取字节数两个参数，就可以读取指定线程的虚拟机栈信息。如：
    memory = self.readMemory(first_fp - 5000, 6000)
    实现了main线程栈帧信息的读取，这里从first_fp – 5000开始读取是为了确保读取到完整的虚拟机栈内存，读取字节数为6000。
### ５、如何成功分析下一个栈帧？　　
* ### 思路  
    每个栈帧的地址内容是上一栈帧的 fp 地址，memory[1] 是内容到地址的映射，我们在地址内容中查找当前栈帧的 fp，存在的话说明下一栈帧是存在的，而且以地址内容为key，value就是下一栈帧的 fp，然后更新frame值为下一栈帧的fp，就可以开始下一栈帧的分析了。
* ### 具体实现  
    在linux_runtime.py模块中定义了getNextFrame(self)方法来获取下一个栈帧。
### ６、如何分析对应的方法参数和局部内存？  
* ### 思路  　　
    栈帧存储了方法的局部变量表、操作数栈、动态连接和方法返回地址等信息，其中局部变量表（Local Variable Table）是一组变量值存储空间，用于存放方法参数和方法内部定义的局部变量。 在Java程序编译为Class文件时，就在方法的Code属性的max_locals数据项中确定了该方法所需要分配的局部变量表的最大容量。
    从fp - 48 地址开始存放的是方法参数、局部变量。  
* ### 具体实现  
    在linux_runtime.py模块中定义了getLocals 方法来分析方法参数、局部变量等信息。　　
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
### 11、确定脏数据的具体表现，复现及解决脏数据问题。　　
* ### 思路　　
    工程在每次分析虚拟机内存信息之前，首先通过volatility框架结合libvmi来获取虚拟机内存信息，但是libvmi获取虚拟机内存无需挂起，导致部分栈帧是上一时刻的，另一部分栈帧是当前时刻的，从而使libvmi获取到的数据不够明确清晰，总有一部分是多余的，即脏数据问题。
### 12、将存在冗余的代码部分进行优化。　　
* ### 思路　　　
    第一：内存重构工程中python 代码部分的优化，内存重构分析工程是基于Volatility框架用python开发的，所以系统性能优化要结合python编程特性，首先可以结合网上python性能优化的一些建议优化代码，其次利用一些性能分析工具定位工程中的性能瓶颈，针对性能瓶颈再做优化。   
    第二：物联网平台监控系统中c++代码部分的优化，c++程序的内存申请和释放都是由开发者控制的，所以在系统性能优化时需要研读代码，查找内存申请集中的代码块，审核代码逻辑，优化程序内存占用。　　　　　　　　
### 13、确定模型验证过程中将字符串生成什么样的Z3表达式。　　
