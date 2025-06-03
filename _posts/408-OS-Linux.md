---
name: linux-command
title: Linux 基本命令
date: 2024-11-30
tags:
- linux
- docker
- nginx
- ssh
categories:
- OS

---





# Linux 概述

## Unix家族

在unix编写过程中发明出了新的编程语言: C

![image-20241007165031608](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007165031608.png)

## GNU: 自由软件运动

FreeBSD: 允许闭源

GPL: GNU General Public License 不允许闭源

MIT: 声明MIT即可

![image-20241007170423889](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007170423889.png)



## Linux 发行版

![image-20241007171216031](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007171216031.png)

## Linux vs. Windows

![image-20241007171911495](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007171911495.png)

# 文件系统

Linux用正斜杠表示路径，一切皆文件

![image-20241007174437116](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007174437116.png)

虚拟目录，只是逻辑上的关系，分区和挂载点才会决定物理存放位置的关系

- **/bin /sbin** 存放二进制命令的目录，这里是一个快捷方式，实际上是/usr/bin
- /lib /lib64 存放动态链接库，这里是一个快捷方式，实际上是/usr/lib
- /media /mnt 外部设备挂载目录
- /home /root用户文件夹
- /boot 启动相关
- /dev 设备相关 
- /run /proc 进程
- /srv 服务
- /sys 系统硬件
- **/var** 可变目录 log 日志文件
- /opt 
- /tmp 临时文件
- **/etc** 配置文件

# Vim

vi->vim 

![image-20241007182423055](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007182423055.png)

## 一般模式

**光标操作**：

- e 移动到当前词尾
- b 上一个词的词头  shift+方向左
- w 下一个词的词头 shift+方向右
- gg 当前文档的开头 shift+h 
- 3 gg 跳转到第3行
- shift+g 跳转到最后一行
- \$ 定位到行尾
- ^ 定位到行开头

**复制粘贴整行**：

- [3] yy 复制从光标开始的3行内容
- [3] p 粘贴剪贴板的内容3次
- y + w  复制当前单词
- y + $ 复制从光标到行尾(\$可以定位到行尾)
- y + ^ 复制从行开头到光标（^可以定位到行尾） 记忆：正则表达式^ $

**删除行**：

- [3] dd 删除光标开始的3行
- d + w 删除单词，（要把光标移到单词开头）
- d + $ 删除从光标到行尾
- d + ^ 删除从行开头到光标

**撤销操作**：u

**剪切字符**：

- x 剪切 光标所指字符
- shift + x  剪切光标之前的字符，类似退格

**替换**：

- r 替换 光标所指字符 
- shift + r 进入替换模式 类似insert模式 输入的字符将直接覆盖光标处的内容

**显示行号**：

- `:set nu`

## 编辑模式

`i `光标在当前字符上 （在当前字符前面插入内容）shift+i 当前行头

`a `光标在下一个字符上（在当前字符后面插入内容）shift+a 当前行尾

`o `光标移动到新建的下一行 shift+o 移动到新建的上一行

## 命令模式

**:w 保存 :q退出 :q! 强制退出**

**:wq保存退出** 

:wq!强制保存退出（只读文件）

:set nonu 取消行号

**/boot** 查找boot并高亮 n下一个 shift+n上一个

:s/old/new(/g) 把当前行的第一个old替换为new（加上/g为当前行所有）

**:%s/old/new(/g)** 每一行的第一个old换成new（加上/g为所有行）

# 网络配置

ifconfig linux中的ipconfig

ping

**traceroute**：追踪网络路径

## 网络连接模式

### 桥接

![image-20241007200741497](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007200741497.png)

路由器到PC，PC这边搭一个虚拟网桥到虚拟交换机，交换机连接虚拟机网络，好处：虚拟机跟PC地位相同，坏处：隐私安全无法保证，VM占用路由器的子网地址

VM和PC的子网掩码相同

### NAT（虚拟地址转换）

VMware建立虚拟路由，构建了一个VM专用的子网。

虚拟路由（虚拟NAT服务器和DHCP服务器）分配子网IP，子网设备（虚拟机）只有通过这个虚拟路由才能连到外网。VM和PC不在一个子网中。VM要请求外网，虚拟路由会根据端口号和子网IP分配端口并更新到映射表中，把信息传到PC网卡，重复上述步骤，实现VM对外网的请求。

虚拟路由通过PC网卡跟外界通信，PC网卡此时只是一个中介，PC本身不算在内网中，不能跟VM通信，虚拟路由不能从外部向内部发送请求，因为NAT只能将子网IP转成外部IP，这里VMware的解决方案是虚拟出一个网络适配器（VMnet8），将PC本身也接入VM子网中。

![image-20241007215205758](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007215205758.png)

![image-20241007220244669](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007220244669.png)![image-20241007220336304](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007220336304.png)

如图网关/路由器是168.111.2，VMnet8（给PC虚拟出的网卡）是168.111.1

### 仅主机

NAT把路由器换成交换机，主机和VM都连到这个交换机上，组成一个局域网，VM只能与主机通信。虚拟出一张主机的网卡，连到交换机上

### 路由器vs交换机

|                  | 路由器     | 交换机                  |
| ---------------- | ---------- | ----------------------- |
| 动态分配IP和端口 | 支持       | 不支持                  |
| NAT              | 支持       | 不支持                  |
| 转发数据         | 基于IP地址 | 基于MAC地址             |
| 适用场景         | WAN、LAN   | LAN（只能用于内部通信） |

systemd    守护进程

# 系统服务控制

## System V 

守护进程 init 第一个进程 调用init.d的脚本根据运行级别启动服务

开机 bios /boot init进程 运行级别 运行级别对应的服务

![image-20241008174512092](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241008174512092.png)![image-20241008174709567](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241008174709567.png)

target运行服务的集合

systemd: 很多守护进程 相比于init效率更高

## systemctl

控制systemd

`status` 服务状态

`restart start stop` 开启停止重启

`enable/disable` 自启动

![image-20241007004917995](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241007004917995-1728748475007-17.png)

**systemctl**是**systemd**的主命令，用于管理系统。除此之外，还有**hostnamectl**(查看当前主机信息)、**timedatectl**(查看当前时区设置)、**loginctl**(查看当前登录的用户)等命令。

# 常用命令

Bash Bourne Again Shell 解释器

Debian dash

## 文件操作

`stat filename`: 查看inode信息

**存储文件元信息的区域就叫 inode**，译为索引节点：**i（index）+node**。 **每个文件都有一个唯一的 inode，存储文件的元信息。** 同一块不能存两个文件

`ln -s file.txt file_link` 创建软链接

硬链接：inode相同 软连接：inode不同，跨文件系统，只是路径相同

`touch a.txt` 更新文件的访问修改时间戳，**如果没有则创建** 所以就是创建文件

### 查看文件内容

wc wordcount

Print newline, word, and byte counts for each FILE,

#### **cat** (concat)(concatenate)

- **功能**：用于连接和显示文件的内容。
- **用法**：可以一次性显示整个文件内容，也可以将多个文件合并。
- **示例**：`cat filename.txt`
- **特点**：直接将文件内容输出到终端，不支持分页或滚动。

#### **more** 

- **功能**：用于分页显示文件内容。
- **用法**：按空格键查看下一页，按回车键查看下一行，按 `q` 退出。
- **示例**：`more filename.txt`
- **特点**：只能向前翻页，不能向后翻。

#### **less** 分页查看

- **功能**：也是用于分页显示文件内容，功能比 `more` 强大。
- **用法**：支持向前和向后翻页，使用方向键、空格键、回车键等进行导航，按 `q` 退出。
- **示例**：`less filename.txt`
- **特点**：可以在文件中快速查找，支持多种命令（例如 `/` 用于搜索）。

#### **tail** 实时输出

- **功能**：用于查看文件的最后几行内容。
- **用法**：默认显示最后 10 行，可以使用 `-n` 选项指定行数，也可以使用 `-f` 选项实时跟踪文件变化（如日志文件）。
- **示例**：`tail filename.txt` 或 `tail -f filename.txt`
- **特点**：常用于监控日志文件的实时输出。

#### **grep** 过滤

g/re/p（globally search a regular expression and print，以正则表达式进行全局查找以及打印

```sh
grep -i apple fruitlist.txt
# 在fruitlist.txt中寻找apple字符串 忽略大小写 返回出现该字符串的行内容
```

Windows平台下，`findstr`代替了`grep`

### 修改文件内容

`vim / nano`编辑器编辑

`awk / sed` 批量处理（流）

`echo / cat` 快速修改

- `echo "New content" > filename` 覆盖内容
- `echo "New content" >> filename` 追加内容
- `cat > filename` 自行输入内容，输完ctrl+D 

### 解压缩文件（tar）

解压：tar -xvf

压缩：tar -zcvf

比如：假如 test 目录下有三个文件分别是：`aaa.txt`、 `bbb.txt`、`ccc.txt`，如果我们要打包 `test` 目录并指定压缩后的压缩包名称为 `test.tar.gz` 可以使用命令：`tar -zcvf test.tar.gz aaa.txt bbb.txt ccc.txt` 或 `tar -zcvf test.tar.gz /test/`

## 目录

**cd**: change directory

**mkdir**/**rmdir**: make/remove directory(not null)

**ls**: list

- -a: 显示.开头的隐藏文件 
- -l: 显示长文件信息 
- -F: 显示可执行信息

**pwd**: print work directory

**cp**: copy

**rm**: remove 

**mv**: move 可以用来改变文件名

find`/home`目录下查找以 `.txt` 结尾的文件名:`find /home -name "*.txt"` ,忽略大小写: `find /home -i name "*.txt" 

## 权限管理

change owner /change mode

`_rwxrw_r_`  普通文件

**所有者(u)**：一般为文件的创建者，谁创建了该文件，就天然的成为该文件的所有者，用 `ls ‐ahl` 命令可以看到文件的所有者 也可以使用 `chown 用户名` 文件名来修改文件的所有者 。

**文件所在组(g)**：当某个用户创建了一个文件后，这个文件的所在组就是该用户所在的组用 `ls ‐ahl`命令可以看到文件的所有组也可以使用 chgrp 组名 文件名来修改文件所在的组。

**其它组(o)**：除开文件的所有者和所在组的用户外，系统的其它用户都是文件的其它组。

### chmod

前提：你是文件所有者   **超级用户可以无视普通用户的权限 ** 普通用户可以用 `sudo chmod`

文件： `r`可用cat读  `w`修改 `x`可执行

目录：`r` 可用ls命令 `w` 可新建，删除目录下文件 `x` 可以用cd访问

#### 基本用法

- **添加权限**
  - `chmod +x filename`：给文件添加执行权限。
  - `chmod +r filename`：给文件添加读取权限。
  - `chmod +w filename`：给文件添加写入权限。

- **删除权限**
  - `chmod -x filename`：去掉文件的执行权限。
  - `chmod -r filename`：去掉文件的读取权限。
  - `chmod -w filename`：去掉文件的写入权限。

#### 设置特定用户的权限

- **针对特定用户**
  - `chmod u+x filename`：给文件的所有者（user）添加执行权限。
  - `chmod g+w filename`：给文件的所属组（group）添加写入权限。
  - `chmod o-r filename`：去掉其他用户（others）的读取权限。

#### 使用八进制数设置权限

权限可以用数字表示，常见的数字对应如下：

- `4`：读取权限（r）
- `2`：写入权限（w）
- `1`：执行权限（x）

通过加和可以设置权限：

- `chmod 7filename`：所有者有读、写、执行权限，组用户和其他用户有读、执行权限。
- `chmod 6filename`：所有者有读、写权限，组用户和其他用户有读取权限。

#### 递归修改权限

- **递归应用权限**：
  - `chmod -R 7directory`：递归地给目录及其所有子文件和子目录设置权限。

#### 其他选项

- **查看当前权限**：
  - 使用 `ls -l filename` 命令可以查看文件的当前权限。

#### 示例

假设你有一个脚本文件 `script.sh`，你希望让所有用户都能执行它：

```bash
chmod a+x script.sh
```

如果你想让所有者有读、写、执行权限，而其他用户只有读权限，可以使用：

```bash
chmod 7script.sh
```

这些是 `chmod` 命令的一些常用选项和用法，希望对你有帮助！如果你有具体的权限需求，也可以告诉我，我可以提供更详细的命令。

### 用户管理

`useradd/del/mod` `groupadd/del/mod` `passwd` 认证相关 

`su username` 切换用户 Switch user

## 查看系统状态

```sh
# ps : process status
ps -ef 
ps -aux #都是查看所有进程的运行情况

ps aux | grep redis 
pgrep redis -a #都是查看包含redis的进程运行情况

kill 5894 # 杀死pid5394
kill -9 5894 # 强制杀死 pid5894
```

```sh
# uptime : startup time 用于查看系统总共运行了多长时间、系统的平均负载等信息
# top : table of processes 用于实时查看系统的 CPU 使用率、内存使用率、进程信息等
# htop : 用户更加友好的top
# vmstat : virtual memory status 进程、内存、I/O 等系统整体运行状态
# pmap : 分析具体进程的使用情况
# free : 用于查看系统的内存使用情况，包括已用内存、可用内存、缓冲区和缓存等
# df : disk free  df -h 查看磁盘的使用情况(可读性高)
# du : disk usage 用于查看指定目录或文件的磁盘空间使用情况，可以指定不同的选项来控制输出格式和单位。
```

## 开发调试

静态分析：

- `objdump -d <program_file_name>`：==查看二进制文件信息==
- `readelf`：查看ELF格式文件详细信息
- `ldd`：查看动态链接库依赖关系
- `nm`：列出目标文件中的符号表
- `strings`：提取文件中的可打印字符串

内存使用与性能分析：

- `cat /proc/meminfo` ：==内存使用概括==
- `pmap -x pid` ==显示进程的内存映射情况，包括虚拟内存地址和大小。==
- `valgrind --tool=memcheck ./program` 分析内存占用，泄露情况
- `perf`：性能分析工具

调试器：

- `gdb`：==GNU调试器==
- `strace`：==跟踪系统调用==
- `ltrace`：跟踪库函数调用

构建工具：

**`make`** 是 Linux 和 Unix 系统上常用的 **构建自动化工具**，用于**自动化编译和管理项目中的依赖关系**

```makefile
# 定义编译器和编译选项
CC = gcc
CFLAGS = -Wall -g

# 目标文件和依赖关系
all: main

main: main.o utils.o
	$(CC) $(CFLAGS) -o main main.o utils.o

main.o: main.c
	$(CC) $(CFLAGS) -c main.c

utils.o: utils.c
	$(CC) $(CFLAGS) -c utils.c

clean:
	rm -f *.o main

# 目标 (Target)：构建的最终产物，例如main。
# 依赖 (Dependencies)：目标所依赖的文件，例如 main.o 和 utils.o。
# 命令 (Commands)：生成目标的命令，需要以 Tab 缩进（不能用空格）
```

## 挂载设备

[Linux系统连接/挂载U盘（移动硬盘）详细步骤](https://blog.csdn.net/qq_43800449/article/details/130447269) 

1.插入U盘，执行如下指令后能看到设备则说明连接成功

```shell
sudo fdisk -l #查看外接设备名称，一般为/dev/sd...，这里假设为/dev/sdc1
```

2.在/mnt下创建挂载点，进行挂载

```shell
sudo mkdir /mnt/mydev #创建挂载点
sudo mount /dev/sdc1 /mnt/mydev #将/dev/sdc1挂载到mnt中的挂载点去
sudo df -h #查看是否挂载成功
```

3.此时挂载文件夹/mnt/mydev里面就是你U盘的内容了，可以随意访问

4.取消挂载，这一步相当于windows下的弹出U盘

```shell
sudo umount /mnt/mydev #取消挂载，拔出硬盘
#sudo rmdir –p /mnt/mydev  删除挂载目录（选做）
```

## 网络

```sh
ping www.baidu.com	#ICMP
traceroute www.baidu.com # 路由路径
ifconfig / ip # 查看网络信息 interface configuration
netstat -aon | grep 3306 # network status 过滤3306字符串 （监听3306端口状态）
ss # 比netstat高效

```

[Linux用netstat查看服务及监听端口详解](https://blog.csdn.net/wade3015/article/details/90779669)

[Linux网络状态工具ss命令使用详解](https://blog.csdn.net/qq_17496365/article/details/96480424) 

## 其他

`Source`将xx应用于当前bash命令行

.bashrc .bash_profile

[linux下的source命令及~/.bashrc, ~/.bash_profile详解_source bashrc-CSDN博客](https://blog.csdn.net/weixin_44439904/article/details/109505753)

[【Linux】什么是.bashrc，以及其使用方法-CSDN博客](https://blog.csdn.net/weixin_57208584/article/details/135868555)

[Linux下source命令详解 - 水车 - 博客园 (cnblogs.com)](https://www.cnblogs.com/shuiche/p/9436126.html)

**快捷键：** 

`Ctrl`+`D` 用于结束输入或表示没有更多数据，通常用于交互式输入的结束

`Ctrl`+`C` 强行停止正在运行的命令或进程

大部分命令，当用于文件夹时加 -r

**部署软件**：

LAMP: Linux + Apache + MySQL/MariaDB + PHP

LEMP/LNMP: Linux + Nginx(Engine-X) + MySQL/MariaDB + PHP

[Apache 与 Nginx：哪一种 Web 服务器最适合您？ (linux-terminal.com)](https://cn.linux-terminal.com/?p=607)

[使用 Docker 轻松部署 LAMP 和/或 LEMP 堆栈 (linux-console.net)](https://cn.linux-console.net/?p=27779)

[如何在 CentOS 7 上安装 Nginx 1.15、MariaDB 10 和 PHP 7 (linux-console.net)](https://cn.linux-console.net/?p=1038) 



# [Shell编程](https://javaguide.cn/cs-basics/operating-system/shell-intro.html)

# 软件安装

Debian系 **apt**

RedHat系 rpm,yum

## 包管理器

### RPM

Redhat Package Manager

```sh
rpm -qi firefox # 查询firefox的安装信息 
rpm -qa # 查询所有

rpm -e firefox # 卸载firefox
--nodeps  #不检查依赖

rpm -ivh forefox_x86_64.rpm # 安装firefox，v=verbose 详细信息 h=hash 进度条
```

### YUM

Yellow dog Updater Modified

基于RPM

```sh
yum -y install firefox # 安装firefox -y表示所有yesno都填y
yum list # list
yum update 
yum check-update
yum deplist
```

#### 换源

```sh
# yum 提示 Could not retrieve mirrorlist
# 检查是否连接到网络 Ctrl+C/D停止ping
ping www.baidu.com
# ping通说明是镜像源出问题
# 备份当前yum源
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
# 下载新的 CentOS-Base.repo 到/etc/yum.repos.d/
# 网易: http://mirrors.163.com/.help/CentOS7-Base-163.repo 
# 阿里: http://mirrors.aliyun.com/repo/Centos-7.repo 
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 清除下载过的安装包, 生成缓存
yum clean all
yum makecache
```

yum 会把下载的软件包和header存储在cache中(默认路径/var/cache/yum/)，而不自动删除。如果觉得占用磁盘空间，可以使用`yum clean`指令进行清除，更精确 的用法是`yum clean headers`(清除header)，`yum clean packages`(清除下载的rpm包)，`yum clean all`(全部清除)

### apt

apt（Advanced Packaging Tool）是一个在 Debian 和 Ubuntu 中的 Shell 前端软件包管理器。

apt 命令提供了查找、安装、升级、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记。

apt 命令执行需要超级管理员权限(root)。

- 列出所有可更新的软件清单命令：**sudo apt update**

- 升级软件包：**sudo apt upgrade**

  列出可更新的软件包及版本信息：**apt list --upgradable**

  升级软件包，升级前先删除需要更新软件包：**sudo apt full-upgrade**

- 安装指定的软件命令：**sudo apt install <package_name>**

  安装多个软件包：**sudo apt install <package_1> <package_2> <package_3>**

- 更新指定的软件命令：**sudo apt update <package_name>**

- 显示软件包具体信息,例如：版本号，安装大小，依赖关系等等：**sudo apt show <package_name>**

- 删除软件包命令：**sudo apt remove <package_name>**

- 清理不再使用的依赖和库文件: **sudo apt autoremove**

- 移除软件包及配置文件: **sudo apt purge <package_name>**

- 查找软件包命令： **sudo apt search <keyword>**

- 列出所有已安装的包：**apt list --installed**

## 下载工具

### wget

webget下载工具

 `wget https://example.com/file.zip` 支持断点续传

`wget -O myfile.zip http://www.example.com/testfile.zip` 更改文件名

`wget -b http://www.example.com/testfile.zip` 后台下载

**递归下载**

- `wget -r http://www.example.com/path1/path2/`
  - `-r`：递归在下整个站点（www.example.com）资源
  - `-nd`：递归下载时不创建一层一层的目录，把所有的文件下载到当前目录；不指定该选项默认按照资源在站点位置创建相应目录
  - `-np`：递归下载时不搜索上层目录，只在当前路径path2下进行下载；不指定该选项默认搜素整个站点
  - `-A 后缀名`：指定要下载文件的后缀名，多个后缀名之间使用逗号进行分隔
  - `-R 后缀名`：排除要下载文件的后缀名，多个后缀名之间使用逗号进行分隔
  - `-L`：递归时不进入其它主机。不指定该选项的话，如果站点包含了外部站点的链接，这样可能会导致下载内容无限大

> 示例，只下载path2路径下的所有pdf和png文件，不创建额外目录全都保存在当前下载目录下:
> `wget -r -nd -np -A pdf,png http://www.example.com/path1/path2/`

### curl

**curl命令**是一个利用URL规则在命令行下工作的文件传输工具。它支持文件的上传和下载，所以是综合传输工具，但按传统，习惯称curl为下载工具。作为一款强力工具，curl支持包括HTTP、HTTPS、ftp等众多协议，还支持POST、cookies、认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。做网页处理流程和数据检索自动化，curl可以祝一臂之力。

**wget** 是一个独立的下载程序，无需额外的资源库，它也允许你下载网页中或是 FTP 目录中的任何内容, 能享受它超凡的下载速度，简单直接。
**curl** 是一个多功能工具，是libcurl这个库支持的。它可以下载网络内容，但同时它也能做更多别的事情。

从用途方面，wget倾向于网络文件下载；curl倾向于网络接口调试，相当于一个无图形界面的 PostMan 工具

### 处理复杂的web请求

**1. 自动跳转**

- `curl -L http://www.example.com`
  - `-L`：自动跳转到重定向链接(Location)

有些链接访问时会自动跳转(响应状态码为3xx)，`-L`参数会让 HTTP 请求跟随服务器的重定向。例如：访问 "http://a.com" 会重定向到 "http://b.com"，使用"-L"选项会返回 "http://b.com" 的响应内容

**2. 显示响应头信息**

- `curl -i http://www.example.com`
  - `-i`：输出包含响应头信息
  - `-I`：输出仅包含响应头信息，不包含响应内容

**3. 显示通信过程**

- `curl -v http://www.example.com`
  - `-v`：显示一次http通信的整个过程，包括端口连接和http request头信息

如果还需要查看额外的通信信息，还可以使用选项 "`--trace 输出文件`" 或者 "`--trace-ascii 输出文件`"，例如：`curl --trace-ascii output.txt http://www.example.com`，打开文件 "output.txt"可以查看结果。

**4. 指定http请求方式**

- `curl -X 请求方式 http://www.example.com/test`
  - `-X 请求方式`：指定http请求方式(GET|POST|DELETE|PUT等)。默认是"GET"

**5. 添加http请求头**

- `curl -H 'kev:value' http://www.example.com/test`
  - `-H 'kev:value'`：添加http请求头。例：`-H 'Content-Type:application/json'`

添加多个请求头，`-H` 选项重复多次即可。例如：
`curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' http://www.example.com/test`

## 编译安装

以OpenSSL 1.1.1 的编译安装为例

> yum -y install wget

1.安装构建 OpenSSL 所需的依赖项。

```sh
sudo yum -y groupinstall "Development Tools"
```

2.下载 OpenSSL 1.1.x 的源代码，其中**x**替换为所需的实际版本。

```sh
wget https://www.openssl.org/source/openssl-1.1.1t.tar.gz
```

3.提取下载的文件。

```sh
tar xvf openssl-1.1.1t.tar.gz
```

4.导航到从文件提取创建的目录。

```shell
cd openssl-1.1*/
```

5.配置 OpenSSL。您可以指定

```shell
./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl
```

```text
Operating system: x86_64-whatever-linux2
Configuring OpenSSL version 1.1.1t (0x1010114fL) for linux-x86_64
Using os-specific seed configuration
Creating configdata.pm
Creating Makefile

**********************************************************************
***                                                                ***
***   OpenSSL has been successfully configured                     ***
***                                                                ***
***   If you encounter a problem while building, please open an    ***
***   issue on GitHub <https://github.com/openssl/openssl/issues>  ***
***   and include the output from the following command:           ***
***                                                                ***
***       perl configdata.pm --dump                                ***
***                                                                ***
***   (If you are new to OpenSSL, you might want to consult the    ***
***   'Troubleshooting' section in the INSTALL file first)         ***
***                                                                ***
**********************************************************************
```

5.使用make命令构建 OpenSSL 1.1.x。

```sh
make -j $(nproc)
```

6.在 CentOS 7 / RHEL 7 上安装 OpenSSL 1.1.1

```shell
sudo make install
```

7.更新共享库缓存。

```shell
sudo ldconfig
```

8.更新系统范围的 OpenSSL 配置：

```shell
sudo tee /etc/profile.d/openssl.sh<<EOF
export PATH=/usr/local/openssl/bin:\$PATH
export LD_LIBRARY_PATH=/usr/local/openssl/lib:\$LD_LIBRARY_PATH
EOF
```

9.重新加载shell环境：

```shell
source /etc/profile.d/openssl.sh
logout
```

10.验证 CentOS 7 / RHEL 7 上是否安装了 OpenSSL 1.1.1

```shell
$ which openssl
/usr/local/openssl/bin/openssl

$ openssl version
OpenSSL 1.1.1t  7 Feb 2023
```



# SSH

## SSH端口

SSH 的端口是服务器上用于监听和处理 SSH 连接请求的网络端口。端口可以理解为设备或计算机与外界通信的一个“入口”或“通道”，每个端口对应不同的服务或应用程序。

### 默认端口：22

SSH 的默认端口号是 **22**。当你在终端中使用 SSH 连接到远程服务器时，如果没有明确指定端口，SSH 客户端会自动连接到服务器的 22 端口。

例如，以下命令使用默认的 22 端口：

```bash
ssh user@192.168.1.100
```

### 自定义端口

为了增强安全性，很多服务器管理员会修改默认的 22 端口为其他非标准端口。这样可以减少一些自动化的攻击，比如扫描网络上 22 端口的 SSH 连接尝试。

如果服务器的 SSH 服务配置在非标准端口（例如 2222），你需要显式指定端口号，才能成功连接到服务器：

```bash
ssh user@192.168.1.100 -p 2222
```

###  端口的作用

每个端口都是通过一个数字标识的，范围在 0 到 65535 之间。不同的端口对应不同的服务：

- **端口 22**：通常用于 SSH 连接。
- **端口 80**：用于 HTTP 网络服务。
- **端口 443**：用于 HTTPS 网络服务。

SSH 服务监听的端口可以自定义配置，通常在服务器的 `/etc/ssh/sshd_config` 文件中进行修改。更改默认端口可以提高安全性，减少针对默认端口的恶意攻击。

### 端口的工作原理

当你尝试通过 SSH 连接到服务器时，SSH 客户端发送请求到服务器的指定端口。服务器上的 SSH 服务会在这个端口上监听并接收连接请求，一旦验证通过，就会建立安全的加密通道，允许你远程控制服务器。

简而言之，SSH 端口就是服务器上提供 SSH 服务的入口，你可以通过指定端口来连接远程服务器。

要通过 SSH 登录到远程服务器上，可以按照以下步骤进行操作：

## 使用SSH登录远程服务器

要在 CentOS 7 上使用 OpenSSH，通常分为以下几个步骤：安装、配置并启动 SSH 服务。具体步骤如下：

### 安装 OpenSSH

CentOS 7 通常已经预装了 OpenSSH。如果没有安装，可以使用以下命令来安装：

```bash
sudo yum install -y openssh-server
```

### 启动并启用 SSH 服务

安装完成后，你需要启动并设置 SSH 服务开机自启：

```bash
# 启动 SSH 服务
sudo systemctl start sshd

# 设置开机自动启动
sudo systemctl enable sshd
```

### 配置 SSH（可选）

如果需要自定义配置 SSH，可以修改配置文件 `/etc/ssh/sshd_config`。你可以使用任何文本编辑器来编辑文件，例如：

```bash
sudo vi /etc/ssh/sshd_config
```

常见的配置项包括：

- **更改默认端口**：
  找到 `#Port 22`，取消注释并设置成你想要的端口号，例如：

  ```bash
  Port 2222
  ```

- **禁止 root 账户直接登录**：
  找到 `#PermitRootLogin yes`，取消注释并改为 `no` 以提高安全性：

  ```bash
  PermitRootLogin no
  ```

完成后，保存并退出。

### 确定远程服务器的 IP 地址和用户名

- **IP 地址**：你需要知道远程服务器的公网 IP 或内网 IP。
- **用户名**：你要知道登录时使用的用户名。通常是服务器的账户名，例如 `root`、`admin` 或其他已创建的普通用户。

### 使用 SSH 命令连接服务器

在本地终端中使用 `ssh` 命令连接远程服务器，命令格式如下：

```bash
ssh username@server_ip
```

- `username`：远程服务器的登录用户名。
- `server_ip`：远程服务器的 IP 地址。

例如，如果你的远程服务器 IP 是 `192.168.1.100`，用户名是 `user123`，你可以输入以下命令：

```bash
ssh user123@192.168.1.100
```

### 端口号非默认时指定端口

如果 SSH 服务使用了非默认端口（默认是 22），你需要通过 `-p` 参数指定端口号。例如，如果远程服务器的 SSH 端口是 2222，你可以这样连接：

```bash
ssh user123@192.168.1.100 -p 2222
```

### 输入密码

第一次连接时，系统会提示你是否信任服务器的主机密钥，输入 `yes`，然后系统会要求你输入服务器用户的密码。输入密码并回车后，如果验证成功，你将进入远程服务器。

### 使用 SSH 密钥登录（可选）

如果你不想每次输入密码，可以配置 SSH 密钥对（公钥和私钥）进行无密码登录。步骤如下：

#### 生成密钥对

在本地电脑上生成 SSH 密钥对：

```bash
ssh-keygen -t rsa -b 4096
```

按回车后会提示你设置密钥保存路径和密码。默认保存在 `~/.ssh/id_rsa`。设置密码是可选的。

#### 将公钥复制到远程服务器

使用 `ssh-copy-id` 命令将你的公钥复制到远程服务器：

```bash
ssh-copy-id user123@192.168.1.100
```

或者，如果服务器使用了不同端口：

```bash
ssh-copy-id -p 2222 user123@192.168.1.100
```

#### 通过密钥连接

公钥成功上传后，下一次你再使用 SSH 登录时，就无需输入密码了：

```bash
ssh user123@192.168.1.100
```

通过以上步骤，你就可以通过 SSH 成功登录远程服务器了。如果有问题，可以检查防火墙、SSH 服务是否正常运行等配置。

**47.97.75.20(公)**



# Docker

## 数据挂载 双向绑定

**数据卷挂载**

**docker volume** 

nginx 容器内部挂载

docker run -v html:/usr/share/nginx/html

容器和外界完全隔绝无法直接访问，内部的bash也不完整不能使用vi编辑器，所以可以把容器内部html目录挂载到宿主机上，挂载之后就有了

**本地目录挂载**

本地任意目录

docker run -v 本地目录：容器内部目录，必须以/ ./开头

## 自定义镜像-dockerfile

 Java镜像：Linux运行环境，JRE, 环境变量，拷贝jar，编写运行的shell脚本

![image-20241010090300035](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010090300035-1728747968346-1.png)

ubuntu镜像中必要的部分，分层使得轮子的复用性大大提高BaseImage，也能提高下载的效率，减少网络资源和硬盘空间的占用

![image-20241010090221898](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010090221898-1728747968346-2.png)

![image-20241010090640183](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010090640183-1728747968346-3.png)

![image-20241010091128239](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010091128239-1728747968347-7.png)

## 容器间网络互连

![image-20241010095017952](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010095017952-1728747968346-6.png)

Bridged network

新创建网桥，将指定容器连接到网桥

创建容器时指定network，即可连到网桥

同一个自定义网桥的容器之间可以用容器名直接通信	 

![image-20241010095906439](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010095906439-1728747968346-4.png)



## 项目部署

Springboot + Nginx + Vue 

1.springboot内部封装了servlet容器 tomcat 能够处理对动态资源的请求（后端集成）

- 管理Servlet程序的生命周期
- 将URL映射到指定的Servlet进行处理
- 与Servlet程序合作处理HTTP请求——根据HTTP请求生成HttpServletResponse对象并传递给Servlet进行处理，将Servlet中的HttpServletResponse对象生成的内容返回给浏览器

2.nginx能够进行反向代理，代理后端服务器处理静态资源的请求（例如图片、视频、CSS、JavaScript文件等,也就是前端的），负载均衡：当业务压力增大时，可能一个Tomcat的实例不足以处理，那么这时可以启动多个Tomcat实例进行水平扩展，而Nginx的负载均衡功能可以把请求通过算法分发到各个不同的实例进行处理

![image-20241010154836707](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010154836707-1728747968346-5.png)

[tomcat 与 nginx，apache的区别是什么？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/32212996)



HTTP server：只关心http协议层面的传输和访问控制，如实将服务器上的文件通过http协议传输给客户端。

Application server: 应用容器（servlet=server applet）tomcat需要提供JSP/servlet运行的类库，会集成一部分httpserver功能，但是不如HTTP server强大

## dockerCompose

docker-compose.yml

![image-20241010165557595](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241010165557595-1728747968347-8.png)



# MySQL on WSL

## CentOS 7 安装到 WSL2 

1. 下载CentOS: [Release CentOS 7.9-2211 · mishamosher/CentOS-WSL (github.com)](https://github.com/mishamosher/CentOS-WSL/releases/tag/7.9-2211)

2. 下载完成解压得到 CentOS7.exe rootfs.tar.gz 两个文件

3. 安装完成后在相同目录下生成 ext4.vhdx 文件

4. ```powershell
   # 在此处打开PowerShell
   wsl -l -v
   # 成功返回如下
   NAME    STATE   VERSION
   CentOS7 Stopped 2
   ```

5. ```powershell
   # 迁移默认存储位置
   # 确保为stop状态
   wsl -l -v
   # 导出CentOS文件到tar，文件夹需提前创建
   wsl --export CentOS7 D:/CentOSWSL/centos7.tar
   # 注销系统
   wsl --unregister CentOS7
   # 确定已注销
   wsl -l -v
   # 执行导入(如果失败可再次尝试执行)
   wsl --import CentOS7  D:/CentOSWSL/ D:/CentOSWSL/centos7.tar
   ```

## 安装缺乏的依赖项

有些部分会提示缺乏dependencies

```sh
rpm -ivh mysql-community-devel-8.0.17-1.el7.x86_64.rpm --force --nodeps
# 直接强制安装，无视依赖
```

```sh
yum install libaio
yum install openssl-devel
```

##  [解决执行systemctl命令报错：Failed to get D-Bus connection ](https://www.cnblogs.com/fengjq/p/17616874.html) 

```sh
echo -e "[boot]\nsystemd=true" | sudo tee -a /etc/wsl.conf
# powershell
wsl.exe --shutdown
# 验证
systemctl list-unit-files --type=service
```

## MySQL

```sh
systemctl start mysqld
systemctl restart mysqld
systemctl stop mysqld

mysql -u root -p
```

### 查看随机生成密码

```sh
vi /var/log/mysqld.log
```

### 更改密码

```sql
alter user 'root'@'localhost' identified by 'ace123456';
```

### 更改密码策略

**默认策略：**

- **至少包含 1 位大小写**
- **至少包含 1 位数字**
- **包含 1 个特殊符号**
- **必须 8 位及以上**

**连接到MySQL服务器：**

```bash
mysql -u root -p
```

输入 `root` 用户的密码进行登录。

**2. 执行以下命令来查看当前的密码策略：**

```bash
SHOW VARIABLES LIKE 'validate_password%';
```

**3. 根据需求修改以下变量：**

- `validate_password.policy`：密码策略，默认值为`MEDIUM`。可以设置为`LOW`、`MEDIUM`、`STRONG`或者自定义。例如，可以将其设置为`LOW`以降低密码复杂性要求。

  ```sql
  SET GLOBAL validate_password.policy = LOW;
  ```

  不同策略的要求：
  `0/LOW`：只验证长度；
  `1/MEDIUM`：验证长度、数字、大小写、特殊字符；默认值。
  `2/STRONG`：验证长度、数字、大小写、特殊字符、字典文件；

- `validate_password.length`：密码最小长度，默认值为`8`。可以根据需要修改最小密码长度。

  ```sql
  SET GLOBAL validate_password.length = 6;
  ```

- `validate_password.number_count`：密码中的数字要求，默认值为`1`。可以增加或减少数字的要求。

  ```sql
  SET GLOBAL validate_password.number_count = 1;
  ```

- `validate_password.special_char_count`：密码中特殊字符的要求，默认值为`1`。可以增加或减少特殊字符的要求。

  ```sql
  SET GLOBAL validate_password.special_char_count = 1;
  ```

- `validate_password.mixed_case_count`：密码中大写字母和小写字母的要求，默认值为`1`。可以增加或减少大写字母和小写字母的要求。

  ```sql
  SET GLOBAL validate_password.mixed_case_count = 1;
  ```

**4. 修改配置文件以使修改的密码策略永久生效。**

打开`MySQL`的配置文件（通常是 `mysqld.cnf` 或 `my.cnf`），添加下面的内容到文件中：

```bash
validate_password.policy=LOW
validate_password.length=6
validate_password.number_count=1
validate_password.special_char_count=1
validate_password.mixed_case_count=1
```

**5. 重启 MySQL 服务以应用更改：**

```bash
sudo systemctl restart mysql
```

完成上述步骤后，就已经修改了 `MySQL 8.0` 的密码策略。可以需求调整密码策略的参数，并确保设置合适的密码策略以提高数据库的安全性。

### 配置文件路径

通过 which 命令可以查看 mysql 安装路径，如下所示。

```bash
$ which mysql 
/usr/bin/mysql
12
```

如果服务器没有安装 mysql 命令，可以使用绝对路径下的 mysql 命令，查看配置文件在哪。如果 Linux 服务器已配置好 mysql 命令，也可以直接使用 mysql 命令查看。具体语句如下所示。

```bash
$ /usr/bin/mysql --verbose --help | grep -A 1 'Default options'
$ mysql --verbose --help | grep -A 1 'Default options'
12
```

2条命令分别执行完毕后的结果均显示如下：

```bash
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf
12
```

结果显示：服务器首先读取的是 /etc/my.cnf 文件，如果该文件不存在则继续读 /etc/mysql/my.cnf 文件，如若文件还不存在便会去读 /usr/etc/my.cnf 文件，若文件仍不存在则继续读 ~/.my.cnf文件。

通过sudo tee写入

```bash
echo -e "[mysqld]\nvalidate_password.policy=LOW\nvalidate_password.length=6\nvalidate_password.number_count=1\nvalidate_password.special_char_count=1\nvalidate_password.mixed_case_count=1" | sudo tee /etc/my.cnf
```

### 添加可供远程访问的用户

```sql
create user 'root'@'%' identified with mysql_native_password by 'ace123456';
grant all on *.* to 'root'@'%';
```

## 查看WSL的IP地址

```shell
# Powershell
wsl -d CentOS7 hostname -I
# 输出
$ wsl: 检测到 localhost 代理配置，但未镜像到 WSL。NAT 模式下的 WSL 不支持 localhost 代理。
172.19.111.46
```

## 查看服务状态

```sh
sudo systemctl status mysqld
chkconfig mysqld on
重载systemctl配置，设置mysqld服务开机自启动。

systemctl daemon-reload
systemctl enable mysqld
systemctl start mysqld
```

```sh
[root@LAPTOP-BCC46KV0 init.d]# systemctl show mysqld -p After
After=basic.target network.target syslog.target systemd-journald.socket system.slice

systemctl status mysqld
journalctl -xe

systemctl show mysqld -p After
systemctl show mysqld -p Before
netstat -tulnp | grep :3306

cat /etc/passwd

ps -eo pid,user,comm | grep mysqld

top -u mysql
```

```sh
# 写入windows环境变量+开启systemd			
echo "[boot]\nsystemd=true\n" >> /etc/wsl.conf
echo "[interop]\nappendWindowsPath=false\n" >> /etc/wsl.conf
echo '### Windows ###
export PATH="$PATH:/mnt/c/Users/Lenovo/AppData/Local/Microsoft/WindowsApps"
export PATH="$PATH:/mnt/c/Program Files/Docker/Docker/resources/bin"
export PATH="$PATH:/mnt/c/Windows"' >> /etc/profile
```

```sh
#启动时间
uptime 
who -b

vim /etc/wsl.conf

[network]
hostname = node01 #主机名称
generateHosts = false #自动生成hosts
```



# Nginx

## nginx.conf

```yaml
worker_processes  1;  # 指定 Nginx 启动的工作进程数量

events {
    worker_connections  1024;  # 每个工作进程允许的最大连接数
}

http {
    include       mime.types;  # 包含 MIME 类型配置文件
    default_type  application/json;  # 默认的 Content-Type

    sendfile        off;  # 禁用高效文件传输

    keepalive_timeout  65;  # 保持活动连接的超时时间

    server {
        listen       8080;  # Nginx 监听的端口
        server_name  localhost;  # 虚拟主机名称

        location / {
            root   html/hmdp;  # 静态文件根目录
            index  index.html index.htm;  # 默认索引文件
        }

        error_page   500 502 503 504  /50x.html;  # 自定义错误页面
        location = /50x.html {
            root   html;  # 错误页面的具体位置
        }

        location /api {  
            default_type  application/json;  # 设置默认 Content-Type

            keepalive_timeout   30s;  # 对于 /api 的连接保持活动超时时间
            keepalive_requests  1000;  # 允许的最大请求数

            proxy_http_version 1.1;  # 设置代理请求使用的 HTTP 版本

            rewrite /api(/.*) $1 break;  # 重写请求的 URI

            proxy_pass_request_headers on;  # 转发请求头

            proxy_next_upstream error timeout;  # 在错误或超时情况下尝试下一个后端服务器

            #proxy_pass http://127.0.0.1:8081;  # 将请求转发到后端服务
            proxy_pass http://backend;  # 可以使用 upstream 定义的 backend 服务器
        }
    }
#将请求转发
    upstream backend {
        server 127.0.0.1:8081 max_fails=5 fail_timeout=10s weight=1;  # 后端服务器配置
        server 127.0.0.1:8082 max_fails=5 fail_timeout=10s weight=1;  # 另一个后端服务器的注释
    }  
}
```

==虚拟主机（反向代理）Reverse Proxy==：`server` 中的`listen 8080` `server_name localhost` 分别表示虚拟主机的端口和域名，客户端发送请求就可以把虚拟主机当做服务器。

==请求转发（负载均衡）Request Forwarding & Load Balancing==：`location /api` 表示nginx接管`/api`路径下的请求，`rewrite`表示会将请求的URL重写，因为在后端并不存在接管/api的Controller，`proxy_pass`表示请求转发的目标，这里转发到了backend，对应2个服务器地址，默认采用轮询(polling)机制

<img src="https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/image-20241104184639596.png" alt="image-20241104184639596" style="zoom: 50%;" />

如图，前端所有AJAX请求的URL都有`/api`前缀，前端向nginx发送请求，URL为http://localhost:8080/api/shop/1，经过Nginx端口转发到http://127.0.0.1/shop/1下，正好就是后端的Controller所在的URL

==静态文件服务 HTTP Server==： `location /` 表示nginx将接管 `/` 路径下的请求    ` root `表示静态文件的具体位置
