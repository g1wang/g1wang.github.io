# JVM 组件
- Runtime Data Area 运行时数据区
- ClassLoader 类加载器，负责加载class文件
- Execution Engine 执行引擎，负责与操作系统交互
- Native Interface 负责调用本地接口的



## JVM_01_Runtime_Data_Area

[TOC]



### 线程共享的数据区

 - 1 方法区：存储虚拟机加载的类信息、常量、静态变量等
 - 2 堆 ：垃圾收集器管理的主要区域，GC堆

### 线程隔离数据区
 - 3 虚拟机栈：栈帧-存储局部变量、操作数栈、动态链接等
 - 4 本地方法栈：虚拟机使用到的native方法
 - 5 程序计数器：存放对象实例

### 例子：HotSpot 虚拟机对象
- 以HotSpot为例，介绍对象创建的两种分配方式
	- 1 指针碰撞 ：前提 java堆内存规整，空闲内存和已使用内存由指针分割，分配时向空闲区移动。
		java堆是否规整，由垃圾收集器是否有压缩整理功能（Serial、ParNew，带有Compact过程的收集器）
	- 2 空闲列表： 维护空闲内存列表，在创建对象是从空闲列表中选取（CMS，使用Mark-Sweep算法的收集器）。