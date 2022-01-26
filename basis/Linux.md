# Linux

## 内核空间&用户空间

在一个32位系统中，一个程序的虚拟空间最大可以是4GB，最直接的做法就是将内核也看作是一个程序，使它和其他程序一样也具有4GB空间，但这种做法会使系统不断地切换用户程序的页表和内核页表，以致影响计算机的效率。解决此问题之最好做法是把4GB空间分成两部分：一之用户空间，一之内核空间，这样则可保证内核空间固定不变，而当程序切换时改变的仅是程序之页表，其唯一缺点在于内核空间与用户空间均变小了。

Linux以`PAGE_OFFSET`为界将4GB的虚拟内存空间分为两部分：地址`0 to 3GB-1`这段低地址空间为用户空间，大小3GB；地址`3GB to 4GB-1`这段高地址空间为内核空间，大小1GB。

## 常用命令

- 网络
  - 查询域名对应IP地址
    - nslookup：检测网络中DNS服务器能否正确解析域名，用于查询互联网域名服务器

      ```bash
      nslookup <domain>
      ```
    - dig：用于查询DNS名称
    - host：用于执行DNS查询
    - ping：用于向网络主机发送ICMP请求应答数据包，检测某服务器到其他服务器的网络连接情况
    - fping：用于向网络主机发送ICMP请求应答数据包
  - ifconfig、ip：显示网卡挂载的信息

    ```bash
    ifconfig -a
    ```
  - telnet：检测某服务器的端口是否可以正常对外服务

    ```bash
    telnet <ip> <port>
    ```
  - nc：模拟开启TCP/IP的服务器，通常用于拦截HTTP传递的参数，帮助定位RESTful服务的问题
  - mtr：检测网络连通性问题，并可以获取某一个域名或者IP的丢包率

    ```bash
    mtr -r <domain>
    ```
  - traceroute：跟踪网络传输的详细路径，显示每一级网关的信息

    ```bash
    traceroute <domain>
    ```
  - sar：为全面监控网络、磁盘、CPU、内存等信息的轻量级工具
  - netstat、ss：用于查看网络端口的连接情况

    ```bash
    netstat -nap
    ```
  - tcpdump：可以拦截本机网卡上任何协议的通信内容，用于调试网络问题
  - iptraf：用于获取网络I/O的传输速度及其他网络状态信息
  - nmap：扫描某一服务器打开的端口

    ```bash
    nmap -v -A localhost
    ```
  - ethtool：查看网卡的配置或配置网卡
  - curl：模拟HTTP调用，常用于RESTful服务的简单测试

    ```bash
    curl -i <URL> # 打印请求响应头信息
    curl -v <URL> # 打印更多的调试信息
    curl -d 'abc=def' <URL> # 使用POST方法提交HTTP请求
    curl -I <URL> # 仅返回HTTP头
    curl -sw '%{http_code}' <URL> # 打印HTTP响应码
    ```
- 进程 
  - ps：显示系统内的进程
  - top、htop：用于查看活动进程内的CPU和内存信息，能够实时显示系统中各个进程的资源占用情况，可以按照CPU、内存的使用情况和执行时间对进程进行排序
  - pidstat：针对某一进程输出系统资源的使用情况，包括CPU、内存、I/O等

    ```bash
    pidstat -[u|r|d] -p <PID> # u=cpu, r=内存, d=磁盘I/O
    ```
  - pstack：打印进程内的调用堆栈

    ```bash
    pstack <PID>
    ```
  - strace：跟踪进程内的工作机制

    ```bash
    # 跟踪进程ID为PID的进程上与信号有关的系统调用
    strace -e trace=signal -p PID
    ```

  - lsof：查看某个进程打开的文件句柄

    ```bash
    lsof -p <PID> # 查看某个进程打开的文件句柄
    lsof -i:<PORT> # 查看某个端口的使用方式
    ```
  - ulimit：查看用户对资源使用的限制，如打开的最大文件句柄、创建的最大线程数等

    ```bash
    ulimit -a
    ```
- 内存
  - free：查看系统的内存使用情况
  - pmap：查看进程的详细的内存分配情况

    ```bash
    pmap -d <PID>
    ```
- CPU监控
  - vmstat：查看系统的CPU利用率、负载、内存等信息
  - mpstat：查看系统的CPU利用率、负载，并且按照CPU核心分别显示相关信息

    ```bash
    mpstat -P ALL
    ```
- 磁盘I/O监控
  - iostat：查看磁盘I/O的信息及传输速度

    ```bash
    iostat -x
    ```
  - swapon：查看系统交换区的使用情况

    ```bash
    swapon -s
    ```
  - df：显示磁盘挂载的信息

    ```bash
    df -h
    ```
- 摘要
  - md5sum：生成md5摘要
  - sha256：生成sha256摘要
  - base64：生成base64编码
- 文本处理 
  - grep：文本内容（关键字）查找

      ```bash
      # --color: 查找后着色
      grep -5 'pattern' INPUT_FILE # 打印匹配行的前后5行
      grep -C 5 'pattern' INPUT_FILE # 打印匹配行的前后5行
      grep -A 5 'pattern' INPUT_FILE # 打印匹配行的后5行
      grep -B 5 'pattern' INPUT_FILE # 打印匹配行的前5行
      ```
  - find：通过文件名查找文件所在位置，支持模糊匹配
      ```bash
      find . -name FILE_NAME
      ```
  - sed：文本编辑与替换
  - tr：字符替换
  - cut：选取，分析一段数据并取出想要的部分
  - dos2unix、unix2dos：转换Windows和UNIX的换行符
  - awk：文本分析工具，支持脚本处理
- 数据处理与统计
  - sort：排序
  - uniq：去重或分组统计，与sort配合使用
  - wc：统计字数和行数等
- 文件传输
  - scp：实现从本地到远程或从远程到本地的双向传输
- 文件解压缩
  - tar：创建或解压tar格式的包
  - zip、unzip：压缩成zip格式的压缩包或解压
  - rar、unrar：压缩成rar格式的压缩包或解压
- 其他
  - uptime：查看操作系统启动的时间、登录的用户、系统的负载等

## 服务

***TODO***

## 用户

***TODO***

## 内核

***TODO***

## Shell

### 管道

管道使用`|`连接多个命令，`|`唤作管道符，管道（pipe）格式：

```bash
command1 | command2 [| commandN...]
```

`|`左边的命令向标准输出写入，右边的命令向标准输出读取，此二命令可形成管道。

_注意：`command1`必须有正确输出而`command2`必须可以处理`command1`的输出结果，且`command2`只能处理`command1`的正确输出结果，不能处理其错误信息。_

使用了管道的命令有如下特点：

- 命令的语法紧凑且使用简单；
- 通过使用管道，将多个命令串联在一起完成复杂任务；
- 从管道输出的标准错误会混合到一起。

### 重定向

Linux的输入输出有3类：

- 标准输入（standard input, stdin），代码0
- 标准输出（standard output， stdout），代码1
- 标准错误（standard error, stderr），代码2

重定向符：

- `>`：输出重定向，将数据输出到文件，当文件不存在时自动建立文件，文件存在时会覆盖前一次的内容
- `<`：输入重定向，从文件读取
- `>>`：输出重定向，表示追加到文件中，不覆盖，当前输出内容会追加到文件尾部
- `<<`：表示当前标准输入来自命令行的一对分隔号的中间内容

## 信号

信号就是一个通知（事件通知），用来通知某个进程发生了某一件事。信号是异步发生的，又称“软件中断”。

信号的产生：

- 某个进程发送给另一个进程或者发送给自己
- 由内核发送给某个进程
  - 通过在键盘上输入一些命令动作
  - 内存访问异常

### 信号处理相关动作

- 执行系统默认动作
- 忽略此信号
- 捕捉该信号

_注：信号SIGKILL与SIGSTOP是特权信号，不要试图捕捉此二信号，对此二信号的处理程序代码无效。_

### kill命令

```bash
kill -n PID # n=信号编号，PID=进程ID
```

常用信号列举如下：***TODO***

|信号编号|信号名称|信号含义|操作系统默认动作|
|---|---|---|---|
|1|SIGNUP|||
|2|SIGINT|
|3|SIGQUIT||终端退出|
|4|
|5|
|6|
|7|
|8|
|9|SIGKILL||终止|
|10|
|11|
|12|
|13|
|14|
|15|
|16|
|17|
|18|SIGCOUT|
|19|SIGSTOP|
|20|SIGTSTP|
|21|
|22|
|23|
|24|

### 信号编程

信号相关的定义与函数包含在`signal.h`中，使用时需要引入头文件`signal.h`：

```c
#include <signal.h>
// c++: #include <csignal>
```

相关函数：

***TODO***

#### 可重入函数

***TODO***

#### 信号集（信号屏蔽字）

信号集可以表示为：0000000000,0000000000,0000000000...

如果收到某个信号就把信号集上的对应位置1。信号集在Linux中表示为`sigset_t`：

```c
typedef struct {
  unsigned long sig[2]; // 两个无符号32位
} sigset_t;
```

相关函数：

- `sigemptyset()`
- `sigfillset()`
- `sigaddset()`
- `sigdelset()`
- `sigprocmask()`
- `sigismember()`

## 会话

***TODO***

## 进程

关闭终端后如何让进程不退出：

- 拦截SIGHUP信号：***TODO***
- setsid命令：启动一个进程，而且能够使启动的进程在一个新的session中
- 后台执行
  - `&`：***TODO***
  - nohup：***TODO***

### 子进程

***TODO***

#### fork函数

***TODO***

### 守护进程

***TODO***