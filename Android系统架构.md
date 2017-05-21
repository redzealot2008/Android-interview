# Android系统架构
<!-- toc -->
---

## Google官方公布的Android系统架构

![Google官方公布的Android系统架构][1]


## Linux内核层
Android以Linux操作系统内核为基础，借助Linux内核服务实现硬件设备驱动、进程和内存管理、网络协议栈、电源管理、无线通信等核心功能。Android4.0版本之前基于Linux2.6系列内核，4.0及之后的版本使用更新的Linux3.X内核，并且两个开源项目开始有了互通。Linux3.3内核中正式包括一些Android代码，可以直接引导进入Android。Linux3.4增添了电源管理等更多功能，以增加与Android的硬件兼容性，使Android在更多设备上得到支持。直到现在最新的android6.0仍然继续延用着linux3.4.0，而linux最新的版本已经到了4.3系列，那么为什么android没有继续去更新Linux kernel的版本也是一个值得探讨的课题。 
Android内核对Linux内核进行了增强，增加了一些面向移动计算的特有功能。例如，**低内存管理器LMK**（Low Memory Keller），**匿名共享内存**（Ashmem）,以及**轻量级的进程间通信Binder机制**等。这些内核的增强使Android在继承Linux内核安全机制的同时，进一步提升了内存管理，进程间通信等方面的安全性。
下表列举了Android内核的主要驱动模块：

|驱动名称|	说明|
|---|---|
|Android电源管理（Power Management）|	针对嵌入式设备的，基于标准Linux电源管理系统的，轻量级的电源管理驱动|
|低内存管理器（Low Memory Keller）|	低内存管理器（Low Memory Keller）可以根据需要杀死进程来释放需要的内存。扩展了Linux的OOM机制，形成独特的LMK机制|
|匿名共享内存（Ashmem）|	为进程之间提供共享内存资源，同时为内核提供回收和管理内存的机制|
|日志（Android Logger）|	一个轻量级的日志设备|
|定时器（Anroid Alarm）|	提供了一个定时器用于把设备从睡眠状态唤醒|
|物理内存映射管理（Android PMEM）|	DSP及其他设备只能工作在连续的物理内存上，PMEM用于向用户空间提供连续的物理内存区域映射|
|Android定时设备（Android Timed device）|	可以执行对设备的定时控制功能|
|Yaffs2文件系统|	Android采用大容量的NAND闪存作为存储设备，使用Yaffs2作为文件系统管理大容量MTD NAND Flash；Yaffs2占用内存小，垃圾回收简洁迅速。|
|Android Paranoid网络|	对Linux内核的网络代码进行了改动，增加了网络认证机制。可在IPV4，IPV6和蓝牙中设置，由ANDROID_PARANOID_NETWORK宏来启用此特性。|

## 硬件抽象层
内核驱动和用户软件之间还存在所谓的硬件抽象层（Hardware Abstract Layer，HAL），它是对硬件设备的具体实现加以抽象。HAL没有在Android官方系统架构图中标明，下图标出了硬件抽象层在android系统中的位置：

![硬件抽象层][2]

鉴于许多硬件设备厂商不希望公开其设备驱动的源代码，如果能将android的应用框架层与linux系统内核的设备驱动隔离，使应用程序框架的开发尽量独立于具体的驱动程序，则android将减少对Linux内核的依赖。HAL由此而生，它是对Linux内核驱动程序进行的封装，将硬件抽象化，屏蔽掉了底层的实现细节。HAL规定了一套应用层对硬件层读写和配置的统一接口，本质上就是将硬件的驱动分为用户空间和内核空间两个层面；Linux内核驱动程序运行于内核空间，硬件抽象层运行于用户空间。

## 系统运行库层
官方的系统架构图中，位于Linux内核层之上的系统运行库层是应用程序框架的支撑，为Android系统中的各个组件提供服务。系统运行库层由系统类库和Android运行时构成。

### 系统类库
系统类库大部分由C/C++编写，所提供的功能通过Android应用程序框架为开发者所使用。主要的系统类库及说明如下表：

|系统类库名称|说明|
|---|---|
|Surface Manager|执行多个应用程序时，管理子系统的显示，另外也对2D和3D图形提供支持|
|Media Framework|基于PacketVideoOpenCore的多媒体库，支持多种常用的音频和视频格式的录制和回放，所支持的编码格式包括MPEG4，MP3，H264，AAC，ARM|
|SQLite|本地小型关系数据库，Android提供了一些新的SQLite数据库API，以替代传统的耗费资源的JDBC API|
|OpenGL｜ES|基于OpenGL ES 1.0 API标准实现的3D跨平台图形库|
|FreeType|	用于显示位图和矢量字体|
|WebKit|	Web浏览器的软件引擎|
|SGL|	底层的2D图形引擎|
|Libc（bionic libc）|	继承自BSD的C函数库bionic  libc，更适合基于嵌入式Linux的移动设备|
|SSL|	安全套接层，是为网络通信提供安全及数据完整性的一种安全协议|

除上表列举的主要系统类库之外，Android NDK（Native Development Kit），即Android原生库，也十分重要。NDK为开发者提供了直接使用Android系统资源，并采用C或C++语言编写程序的接口。因此，第三方应用程序可以不依赖于Dalvik虚拟机进行开发。实际上，NDK提供了一系列从C或C++生成原生代码所需要的工具，为开发者快速开发C或C++的动态库提供方便，并能自动将生成的动态库和Java应用程序一起打包成应用程序包文件，即.apk文件。 
注意，使用原生库无法访问应用框架层API，兼容性可能无法保障。而且从安全性角度考虑，Android原生库用非类型安全的程序语言C/C++编写，更容易产生安全漏洞，原生库的缺陷（bug）也可能更容易直接影响应用程序的安全性。

### 运行时
Android运行时包含核心库和Dalvik虚拟机两部分。

- 核心库：核心库提供了Java5 SE API的多数功能，并提供Android的核心API，如android.os，android.net，android.media等。
- Dalvik虚拟机：Dalvik虚拟机是基于apache的java虚拟机，并被改进以适应低内存，低处理器速度的移动设备环境。Dalvik虚拟机依赖于Linux内核，实现进程隔离与线程调试管理，安全和异常管理，垃圾回收等重要功能。<br/>

本质而言，Dalvik虚拟机并非传统意义上的java虚拟机（JVM）。Dalvik虚拟机不仅不按照Java虚拟机的规范来实现，而且两者不兼容。

Dalvik和标准Java虚拟机有以下主要区别：

- Dalvik基于寄存器，而JVM基于栈。一般认为，基于寄存器的实现虽然更多依赖于具体的CPU结构，硬件通用性稍差，但其使用等长指令，在效率速度上较传统JVM更有优势。
- Dalvik经过优化，允许在有限的内存中同时高效地运行多个虚拟机的实例，并且每一个Dalvik应用作为一个独立的Linux进程执行，都拥有一个独立的Dalvik虚拟机实例。Android这种基于Linux的进程“沙箱”机制，是整个安全设计的基础之一。
- Dalvik虚拟机从DEX（Dalvik Executable）格式的文件中读取指令与数据，进行解释运行。DEX文件由传统的，编译产生的CLASS文件，经dx工具软件处理后生成。
- Dalvik的DEX文件还可以进一步优化，提高运行性能。通常，OEM的应用程序可以在系统编译后，直接生成优化文件（.ODEX）； 第三方的应用程序则可在运行时在缓存中优化与保存，优化后的格式为DEY（.dey文件）。<br/>

这部分内容，即从android4.4开始就出现了ART（android runtime），但是这个ART并不是指这一节的主题，而是一种用来代替Dalvik的新型运行环境。当然在4.4的正式环境中用的还是Dalvik，真正开始用ART取代Dalvik是从android5.0开始的。

另外这一节中有提到NDK,相信对于开发者而言SDK和NDK都是必要要接触和了解的东西，那么先从下图来看看sdk和ndk的关系。 

![sdk and ndk][3]

很显然地，NDK可以通过Native code跨过使用Dalvik runtime，直接调用到Android内核资源，而SDK则需要在Dalvik runtime环境下才能调用到内核资源。然而两者并不是各司其职，各不相关。Android提供了JNI(Java native interface)使两者可以进行相互调用和通信。

## 应用程序框架层
应用程序框架层提供开发Android应用程序所需的一系列类库，使开发人员可以进行快速的应用程序开发，方便重用组件，也可以通过继承实现个性化的扩展。具体包括的模块如表：

|应用程序框架层类库名称|	功能|
|---|---|
|活动管理器（Activity Mananger）|	管理各个应用程序生命周期并提供常用的导航回退功能，为所有程序的窗口提供交互的接口|
|窗口管理器（Window Manager）|	对所有开启的窗口程序进行管理|
|内容提供器（Content Provider）|	提供一个应用程序访问另一个应用程序数据的功能，或者实现应用程序之间的数据共享|
|视图系统（View System）|	创建应用程序的基本组件，包括列表（lists），网格（grids），文本框（text boxes），按钮（buttons），还有可嵌入的web浏览器。|
|通知管理器（Notification Manager）|	使应用程序可以在状态栏中显示自定义的客户提示信息|
|包管理器（Package Manager）|	对应用程序进行管理，提供的功能诸如安装应用程序，卸载应用程序，查询相关权限信息等。|
|资源管理器（Resource Manager）|	提供各种非代码资源供应用程序使用，如本地化字符串，图片，音频等|
|位置管理器（Location Manager）|	提供位置服务|
|电话管理器（Telephony Manager）|	管理所有的移动设备功能|
|XMPP服务|	是Google在线即时交流软件中一个通用的进程，提供后台推送服务|

## 应用层
Android平台的应用层上包括各类与用户直接交互的应用程序，或由java语言编写的运行于后台的服务程序。例如，智能手机上实现的常见基本功能程序，诸如SMS短信，电话拨号，图片浏览器，日历，游戏，地图，web浏览器等程序，以及开发人员开发的其他应用程序。

  [1]: http://onmer39jj.bkt.clouddn.com/image/1000px-Android-System-Architecture.svg.png
  [2]: http://onmer39jj.bkt.clouddn.com/image/53df3df1325b5.jpg
  [3]: http://onmer39jj.bkt.clouddn.com/image/20160107235017221.jpg

## 参考
- [Google工程师多图详解Android系统架构](http://mobile.51cto.com/android-235496.htm)
- [Android基础之Android系统架构](http://my.oschina.net/fhd/blog/349830)
- [Android对Linux内核的增强](http://tech.it168.com/a2011/0805/1228/000001228471.shtml)
- [Android Runtime](http://www.tuicool.com/articles/EN7nuiN)