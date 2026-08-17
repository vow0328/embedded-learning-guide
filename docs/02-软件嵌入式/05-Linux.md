# 2.5 Linux

> "老大，我们真的要把Linux也推荐给电控的同学吗？你当初可是啃了整整一年，才勉强把系统环境摸顺，更别提后来的嵌入式应用开发了。"
>
> "正因为当年踩了太多坑，虚度了太多时间，才更希望把它写下来。要是学弟学妹们能借着这份指南避开我走过的弯路，他们所能达到的高度说不定会远超于我。"

这段对话并非夸张。在电控的进阶之路上，Linux就像是一座绕不开却又极具挑战的高山。它与前面提到的RTOS存在本质区别，甚至可以说是一个全新的知识体系。RTOS更像是"给单片机加了一个高级的任务调度器"，而Linux则是完整的通用操作系统，拥有进程管理、虚拟内存、文件系统、权限管理、网络协议栈、驱动模型等一整套庞大的机制。除了通用的进程与线程思维可以复用之外，几乎一切都要从头学起。

对于刚从单片机或RTOS转过来的人来说，Linux开发的第一个难点甚至不是技术本身，而是适应环境的阵痛。

习惯了Windows图形化操作的人，突然被扔进黑乎乎的命令行界面往往会无所适从，你需要花足够的时间去熟悉一堆基础命令。然而，当你刚刚熬过这段适应期，连一刻都没有让你喘息的机会，立刻赶到战场的是：你曾经无比熟悉的图形化开发利器（对，没错，就是Keil）在Linux下完全失效了。取而代之的是SSH、GCC、Makefile、交叉编译等一套全新的开发工具链和工作流。等你磕磕绊绊地终于搞定了环境配置，恭喜你，现在你拥有了打开新手村大门的资格，可以正式开始面对那些动辄几十个小时、且难度极高的嵌入式Linux入门教程了。

不仅仅是工具变了，整个开发的底层逻辑也变了。

在单片机里，我们习惯直接操作寄存器，或者调用HAL库去控制GPIO、I2C、SPI、UART等外设。但在Linux中，硬件绝不会轻易暴露给应用层。Linux的核心哲学是"一切皆文件"，所有的硬件设备最终都会被抽象成设备节点，应用程序必须通过文件系统接口和系统调用，才能间接地访问硬件。

这也导致了底层驱动开发的巨大差异。嵌入式Linux没有像STM32那样官方统一的设备驱动库。面对自研硬件，你可能需要自己编写或修改驱动。而且Linux驱动绝不是"配个寄存器然后读写IO"就结束了，它需要遵循一套严苛的内核开发规范：编写内核模块、注册设备节点、实现file_operations接口、通过设备树（Device Tree）描述硬件信息，最终将底层硬件能力封装成用户空间可调用的标准API。

正因为门槛高、跨度大，嵌入式Linux的学习周期通常比较长。极其不建议新手一开始就直奔内核与驱动开发，这很容易让人从入门到放弃。更合理且平滑的学习路线应该是：先熟悉Linux系统的日常使用环境，接着学习Linux应用层开发，等建立起足够的系统级认知后，再向难度更高的驱动开发和系统移植发起挑战。

## 入门：先熟悉Linux环境

在正式进入嵌入式Linux开发之前，你首先需要会用Linux。这里的"会用"不是指会点桌面图标，而是能熟练使用命令行完成文件管理、编译运行、网络连接、进程查看和问题排查。

### 常用命令（必须熟练）

| 类别 | 常用命令 |
|------|----------|
| 文件操作 | ls、cd、cp、mv、rm、mkdir、find、tree |
| 文本处理 | cat、grep、sed、awk、vim/nano |
| 权限管理 | chmod、chown、sudo |
| 进程管理 | ps、top、htop、kill、&（后台运行） |
| 网络工具 | ping、ifconfig/ip、ssh、scp、wget、curl |
| 包管理 | apt install、apt update（Ubuntu/Debian系） |
| 压缩解压 | tar、zip/unzip |

**建议**：在自己的电脑上安装一个Ubuntu虚拟机（VMware或VirtualBox），或者直接用WSL2，日常练习就在里面操作，不要只看教程。

### 推荐的开发工具链（替代Keil）

| 工具 | 用途 |
|------|------|
| VS Code + Remote-SSH | 远程连接开发板或虚拟机，等同于本地开发体验 |
| GCC / arm-none-eabi-gcc | C/C++编译器 |
| GDB / GDB Server | 调试器 |
| Make / CMake | 构建工具 |
| Minicom / Picocom | 串口终端（替代XCOM） |

## 应用层开发

嵌入式Linux建议从应用层开发开始学。应用层开发运行在用户空间，不需要直接接触内核源码，学习曲线相对平滑，也更容易做出实际效果。很多嵌入式Linux岗位本质上就是应用层开发，比如设备控制程序、通信程序、数据采集服务、网关程序、上位机后台服务等。

### ① Linux下C语言编程基础

首先要熟悉GCC编译流程：

```bash
gcc hello.c -o hello          # 直接编译
gcc -E hello.c -o hello.i    # 只预处理
gcc -S hello.c -o hello.s    # 编译到汇编
gcc -c hello.c -o hello.o    # 编译到目标文件
```

需要理解预处理、编译、汇编、链接每一步在做什么。相比Keil一键编译，Linux下的编译流程更透明，但也更要求你理解工程结构。

Makefile是基本功。项目稍微复杂一点，就不可能每次手动敲gcc命令。Makefile用来管理源文件、头文件、依赖关系和编译规则，是Linux C/C++开发绕不开的内容。后续项目变大后，可以进一步学习CMake。

GDB是Linux下最常用的调试工具。常用命令包括break、run、next、step、print、continue、backtrace。虽然它没有Keil图形化调试那么直观，但功能非常强大。实际嵌入式开发中，还经常配合gdbserver做远程调试。

### ② Linux系统编程

这是应用层开发的核心，围绕"一切皆文件"的设计哲学展开：

| 模块 | 主要内容 |
|------|----------|
| 文件IO | open、close、read、write、lseek |
| 标准IO | fopen、fread、fwrite、fprintf、fclose |
| 多进程 | fork、exec、wait，孤儿进程与僵尸进程 |
| 多线程 | pthread_create、pthread_join、线程同步 |
| 线程同步 | mutex、condition variable、semaphore |
| 进程间通信 | 管道、消息队列、共享内存、信号、Socket |
| 网络编程 | TCP/UDP Socket，bind、listen、accept、connect、send、recv |
| IO多路复用 | select、poll、epoll |

和FreeRTOS对比理解会更容易：

| FreeRTOS | Linux |
|----------|-------|
| 任务 | 线程/进程 |
| 队列 | 消息队列/管道/Socket |
| 互斥量 | pthread_mutex |
| 信号量 | POSIX semaphore |
| 事件标志组 | condition variable / eventfd |
| 任务通知 | signal / eventfd / pipe |

需要注意，这只是帮助理解的类比，并不是完全等价。Linux有虚拟内存、用户态/内核态隔离、进程地址空间等机制，这些是RTOS中通常没有的。

### ③ Shell脚本与服务管理

嵌入式Linux开发中，Shell脚本非常常用。比如开机初始化、日志清理、网络配置、程序守护、自动升级，都可能用Shell脚本完成。

如果系统使用systemd，还需要了解service文件的基本写法，用于设置程序开机自启动、异常重启、运行依赖等。很多产品级Linux设备，应用程序最终都是以systemd服务的形式运行。

### ④ 交叉编译

交叉编译指的是在x86电脑上编译出能在ARM开发板上运行的程序。因为开发板性能有限，不适合直接在板子上编译大型项目，所以通常在PC上完成编译，再把可执行文件传到开发板运行。

示例：

```bash
# 安装32位ARM Linux交叉编译工具链
sudo apt install gcc-arm-linux-gnueabihf

# 交叉编译
arm-linux-gnueabihf-gcc hello.c -o hello_arm

# 传输到开发板
scp hello_arm root@192.168.1.100:/home/root/

# 登录开发板运行
ssh root@192.168.1.100
./hello_arm
```

实际项目中还需要注意动态库、头文件、sysroot、交叉编译链版本匹配等问题。初学阶段先能完成"交叉编译 → 传到开发板 → 运行"这一套流程即可。

## 驱动层开发

驱动开发工作在Linux内核空间，负责把具体硬件封装成标准接口供应用层调用。它比应用层难很多，因为驱动一旦写错，轻则模块加载失败，重则系统崩溃、内核panic。

建议在应用层开发比较熟练之后，再开始学习驱动。

### ① Linux内核模块机制

Linux驱动通常以内核模块的形式存在，编译后生成.ko文件，可以动态加载和卸载，不需要每次都重新编译整个内核。

常用命令：

```bash
insmod mydriver.ko     # 加载驱动
rmmod mydriver         # 卸载驱动
lsmod                  # 查看已加载模块
dmesg                  # 查看内核日志
modinfo mydriver.ko    # 查看模块信息
```

最简单的内核模块框架：

```c
#include <linux/module.h>
#include <linux/init.h>

static int __init mydriver_init(void)
{
    printk(KERN_INFO "Driver loaded!\n");
    return 0;
}

static void __exit mydriver_exit(void)
{
    printk(KERN_INFO "Driver unloaded!\n");
}

module_init(mydriver_init);
module_exit(mydriver_exit);

MODULE_LICENSE("GPL");
```

### ② 字符设备驱动框架

字符设备是Linux中最常见的驱动类型之一。LED、按键、传感器、简单自定义设备，都可以先从字符设备驱动开始学。

核心是实现file_operations结构体：

```c
static struct file_operations mydev_fops = {
    .owner          = THIS_MODULE,
    .open           = mydev_open,
    .release        = mydev_release,
    .read           = mydev_read,
    .write          = mydev_write,
    .unlocked_ioctl = mydev_ioctl,
};
```

用户空间通过：

```c
fd = open("/dev/mydev", O_RDWR);
read(fd, buf, len);
write(fd, buf, len);
close(fd);
```

就会对应调用内核中的open、read、write、release等函数。这就是"一切皆文件"在驱动层的体现。

驱动开发中还必须掌握两个重要函数：

```c
copy_to_user();
copy_from_user();
```

内核空间不能直接访问用户空间指针，需要通过这两个函数安全地在内核态和用户态之间传递数据。

### ③ 设备树（Device Tree）

现代Linux驱动通过设备树（.dts文件）描述硬件信息，将硬件配置与驱动代码分离：

```c
/* 在设备树中描述一个I2C传感器 */
&i2c1 {
    mpu6050@68 {
        compatible = "invensense,mpu6050";
        reg = <0x68>;
        interrupt-parent = <&gpio1>;
        interrupts = <2 IRQ_TYPE_EDGE_FALLING>;
    };
};
```

设备树中描述了设备挂在哪条总线上、地址是多少、中断接在哪个GPIO、compatible字符串是什么。驱动通过compatible字符串和设备树节点匹配，匹配成功后调用probe函数完成初始化。

设备树是嵌入式Linux驱动开发中非常关键的一环，很多驱动问题本质上不是代码错了，而是设备树写错了。

### ④ 常见驱动开发

| 驱动类型 | 关键知识点 |
|----------|------------|
| GPIO驱动 | gpiod接口、方向配置、电平读写 |
| I2C设备驱动 | i2c_driver、i2c_client、i2c_transfer |
| SPI设备驱动 | spi_driver、spi_device、spi_sync |
| UART驱动 | tty框架、termios配置 |
| 中断驱动 | request_irq、上半部、下半部 |
| PWM驱动 | pwm_apply_state、占空比与周期配置 |
| RTC驱动 | rtc_class_ops、时间读写 |
| Input驱动 | 按键、触摸屏、输入事件上报 |
| IIO驱动 | ADC、IMU、传感器数据采集 |
| Regmap | 寄存器读写抽象，常用于I2C/SPI芯片 |

早期教程中常见gpio_request、gpio_direction_output这类接口，现在新内核更推荐使用gpiod描述符接口。学习时可以先理解旧接口，再逐步过渡到新接口。

### ⑤ Platform总线驱动模型

Platform总线是嵌入式Linux驱动开发中非常核心的模型。它的思想是"设备和驱动分离"：

| 组成 | 作用 |
|------|------|
| Platform Device | 描述硬件资源，如寄存器地址、中断号 |
| Platform Driver | 实现驱动逻辑 |
| Platform Bus | 负责匹配Device和Driver |
| probe函数 | 匹配成功后执行初始化 |

在设备树驱动模型中，设备信息通常由.dts描述，驱动通过compatible字符串进行匹配。理解了Platform驱动模型，再看I2C、SPI、GPIO、PWM这些驱动框架，思路会清晰很多。

## 调试与排错

Linux开发中，调试工具非常重要。很多时候问题不是代码本身，而是权限、路径、设备节点、驱动没加载、设备树没生效、库文件找不到。

常用调试工具：

| 工具 | 用途 |
|------|------|
| dmesg | 查看内核日志 |
| printk | 驱动调试输出 |
| strace | 跟踪应用程序系统调用 |
| ldd | 查看程序依赖的动态库 |
| gdb/gdbserver | 应用程序远程调试 |
| top/htop | 查看CPU和内存占用 |
| ip/ifconfig | 查看网络状态 |
| ls /dev | 查看设备节点 |
| cat /proc/interrupts | 查看中断触发情况 |
| cat /proc/meminfo | 查看内存信息 |
| cat /sys/class/... | 查看内核导出的设备信息 |

其中strace非常实用。比如一个程序打不开设备文件，你不用猜，直接strace一跑，就能看到open哪个路径失败、失败原因是什么。

## 系统移植

系统移植是嵌入式Linux中更底层的方向，通常由BSP工程师负责。对于应用层和普通驱动开发来说，一般直接使用芯片厂商提供的SDK和镜像即可，不需要从零移植。但了解启动流程有助于排查系统级问题。

| 内容 | 说明 |
|------|------|
| U-Boot移植 | Bootloader，上电后负责初始化硬件并加载内核 |
| Linux内核裁剪 | 通过make menuconfig配置内核功能 |
| 设备树适配 | 根据硬件原理图修改.dts |
| 根文件系统制作 | 使用Buildroot或Yocto构建rootfs |
| 驱动模块编译 | 根据内核版本和源码树编译.ko |
| 启动参数配置 | bootargs、console、rootfs挂载参数 |

系统移植不建议初学者一开始就深入，否则很容易被U-Boot、内核配置、设备树、文件系统、交叉编译链同时劝退。先会用，再会改，最后再考虑从零搭建。

## 学习建议

这里需要先强调一个非常重要的认识：嵌入式Linux开发的重点，前期在Linux，而不在"嵌入式"。换句话说，在还没有真正接触到硬件之前，你要学的很多内容，和普通的Linux开发并没有本质区别。命令行、文件系统、进程线程、网络通信、Makefile、GDB、交叉编译，这些都属于Linux开发的通用基础，不会因为前面加了"嵌入式"三个字就自动变简单。

这也是很多单片机或RTOS背景同学最容易踩的坑：总以为"嵌入式Linux"也应该像学STM32一样，一上来就接板子、点灯、串口打印、驱动外设。但实际情况恰恰相反。Linux的学习路线和前面所有内容都可以说是天差地别，在嵌入式Linux的前期学习阶段，甚至可以完全没有硬件实物参与，只靠一台电脑、一个Ubuntu虚拟机或者WSL环境，就足够把最关键的基础部分学起来了。

也就是说，前面那套"学习第一步先点亮一颗LED"的嵌入式思维，在嵌入式Linux这里并不适用。对于Linux学习来说，点亮一颗LED往往已经不是起点任务，而是学到中期、开始接触驱动和设备树之后才会真正去做的事情。

作者自己当初学Linux的时候，就吃过这个亏。因为一直没有理清学习路线，同时又有点眼高手低、外加头铁，一上来就想直接折腾硬件、点亮LED，结果发现整个过程远比想象中复杂：不会命令行、不会交叉编译、不会内核模块、不会设备树、看不懂驱动框架，最后被劝退了不止一次。直到后来花了很长时间，才慢慢把嵌入式Linux的学习顺序理清楚。

所以这里非常想强调一句：一定一定要按路线走。

Linux不是不能学，而是如果一开始学习顺序错了，就很容易在高难度内容面前晕头转向，最后不是学不会，而是先被复杂度劝退了。

推荐教学视频：韦东山的【韦东山手把手教你嵌入式Linux快速入门到精通】

## 小节总结

嵌入式Linux是一个独立且庞大的知识体系，它和RTOS的差别不只是API不同，而是整个开发模式都不同。RTOS更关注任务调度和资源同步，嵌入式Linux则涉及应用层、驱动层、内核、文件系统、网络协议栈和系统启动流程。

对于电控方向来说，不一定每个人都要成为Linux内核专家，但至少应该具备基本的Linux使用能力和应用层开发能力。尤其是现在很多机器人、智能设备、边缘计算设备都会使用Linux主控，单片机负责实时控制，Linux负责通信、感知、决策、数据处理和上位机交互。不会Linux，很多复杂系统就很难真正参与进去。

学习Linux的过程会比较痛苦，尤其是从Keil、CubeMX这种图形化工作流切换到命令行、交叉编译、Makefile、SSH、GDB的时候，很容易怀疑自己。但熬过最开始的适应期之后，你会发现Linux的开发方式非常强大，也更接近真实工程。

嵌入式Linux不是几周就能学会的内容，需要长期学习、分阶段推进。先从应用层入门，再逐步进入驱动层和系统层，不要一开始就试图吃完整个体系。选择了电控这条路，Linux迟早是要面对的。毕竟，电控不能没有Linux，就像西方不能没有耶路撒冷。

---

上一篇：[RTOS](04-RTOS.md) | 下一篇：[软件仿真](06-软件仿真.md)
