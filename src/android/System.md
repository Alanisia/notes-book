# Android系统

Android系统架构：

1. Java应用程序：各种APP，用户交互
2. Java应用程序框架：Android四大组件、Intent和Binder
3. C/C++本地库和Android运行时环境
4. Linux内核与驱动

## 启动机制

Android系统启动加载完内核后，第一个执行的进程是init进程，init进程所做的工作：

1. 设备初始化
2. 读取init.rc文件
3. 启动系统中的重要外部程序zygote

zygote进程是Android所有进程的孵化器进程，启动之后，

1. 初始化Dalvik虚拟机
2. 启动system_server并进入zygote模式
3. 通过socket等候命令

Android应用程序执行时，

1. system_server进程通过socket方式发送命令给zygote
2. 收到命令后fork一个虚拟机实例来执行应用程序的入口函数
3. 应用程序启动

zygote提供3种用于创建进程的方法：

- `fork()`：创建一个zygote进程（这种方法实际上不会被调用）
- `forkAndSpecialize()`：创建一个非zygote进程
- `forkSystemServer()`：创建一个系统服务进程

非zygote进程不能fork出其他进程，而系统服务进程在终止后其子进程亦须终止。

## Dalvik虚拟机

Dalvik虚拟机运行Dalvik字节码。Dalvik字节码由Java字节码转换而来，并被打包到一个dex（Dalvik Executable）可执行文件中，Dalvik虚拟机通过解释dex文件来执行字节码。Android SDK中的dx工具负责将Java字节码转化为Dalvik字节码。

### Java虚拟机与Dalvik的区别

_Dalvik虚拟机与Java虚拟机不兼容。_

Java虚拟机基于栈架构，Dalvik虚拟机基于寄存器架构。

***TODO***

### dex文件结构

整体结构：

1. dex header：dex文件头，指定了dex文件的一些属性，记录了其余部分数据结构在dex文件中的物理偏移
2. `string_ids`、`type_ids`、`proto_ids`、`field_ids`、`method_ids`、`class_def`：索引结构区
3. `data`：数据区
4. `link_data`：静态链接数据区

#### 数据类型

|类型|含义|
|---|---|
|u1|uint8_t，1字节无符号数|
|u2|uint16_t，2字节无符号数|
|u4|uint32_t，4字节无符号数|
|u8|uint64_t，8字节无符号数|
|SLEB128|有符号LEB128，可变长度1~5字节|
|ULEB128|无符号LEB128，可变长度1~5字节|
|ULEB128pl|无符号LEB128值加1，可变长度1~5字节|

**LEB128数据类型**

```
2字节LEB128值按位图表
+------------------------------------+----------------------------------------+
|            First Byte              |              Second Byte               |
+-+----+----+----+----+----+----+----+-+-----+-----+-----+-----+----+----+----+
|1|bit6|bit5|bit4|bit3|bit2|bit1|bit0|0|bit13|bit12|bit11|bit10|bit9|bit8|bit7|
+-+----+----+----+----+----+----+----+-+-----+-----+-----+-----+----+----+----+
```

## 安全机制

Android系统架构的每一层都有它的安全机制：

1. Android应用层：接入权限
2. 应用程序框架层：数字证书
3. 核心库与Dalvik虚拟机层：沙箱机制
4. Linux内核层：Linux文件权限

### Root

在Android设备中获得超级用户权限的过程称为root，超级用户权限亦称root权限。

提权方法：

- 找一个已有root权限的进程完成整个系统的root

  通过系统漏洞提升权限到root，思路：init进程启动的服务进程，如adbd、rild、mtpd、vold等都有root权限，寻找其漏洞。

- 通过系统外的某些方法植入

  通过Recovery刷机方式刷入root权限。

  > **Recovery**
  >
  > 系统的修复文件夹，存放着修复文件。

## APK

每个软件在最终发布时会打包成一个APK（Android Package）文件，将APK文件传送到Android设备中运行即可进行安装。

### 组成

APK实际上就是一个zip压缩包，对其进行解压后，文件如下：

- assets文件夹：资源目录、声音、字体、网页
- lib文件夹：so库存放位置
- META-INF文件夹：存放工程的一些属性文件，如Manifest.MF
- res文件夹：资源目录，即应用中使用到的资源目录，已编译的无法直接阅读
- AndroidManifest.xml：Android工程的基础配置属性文件
- classes.dex：Dalvik VM可执行文件
- resources.arsc：res目录下的资源的一个索引文件

### 生成步骤

1. 打包资源文件，生成R.java
2. 处理aidl文件
3. 编译工程源代码
4. 转换所有class文件生成classes.dex
5. 打包生成APK文件
6. 对APK进行签名
7. 对签名后的APK文件进行对齐处理

### Android程序安装流程

安装方式：

- 系统程序安装：开机时安装，通过开机启动的PackageManagerService服务完成，此服务在启动时会扫描系统程序目录`/system/App`并重新安装所有程序
- 通过Android市场安装：网络安装
- ADB工具安装：使用Android SDK提供的ADB来安装
- 手机自带安装：通过APK文件安装，有安装界面，安装界面实为PackageInstaller的PackageInstallerActivity

安装过程：

***TODO***