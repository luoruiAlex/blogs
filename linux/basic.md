## 硬盘
- 磁盘、扇区(sector 512B或4KB)、磁道(track)
- 接口种类
  - IDE /dev/hd[a-d]
  - SATA SCSI USB硬盘光盘 /dev/sd\[a-p]
  - VirtI/O /dev/vd\[a-p]
- MBR分区
  - Main Boot Recorder主引导区(446B)，存放引导程序和磁盘分区表(64B)
  - 最多4个分区，其中最多一个Extend(扩展分区)，其余必须为Primary(主分区)
  - 扩展分区可指向额外分区表，扩展分区必须分层逻辑分区才能使用————用于分区超过4个，比如hda1 hda2 hda4(hda5 hda6 hda7)，1~4号已被预留
- GPT分区
  - GUID partition table
  - 没有主分区、扩展分区、逻辑分区的概念了
  - 使用LBA(Logical Block Address 512B)来分区，前面34个LBA都可以用来分区，磁盘最后的34个LBA可用于备份
  - LBA0 引导程序 + GPT格式标记
  - LBA1 分布表本身的位置与大小 + 备份GPT分区位置 + CRC32校验码
  - LBA2-33 每个LBA科记录4个分区，所以默认可记录128个分区
  - 每个LBA使用64bit记录扇区位置，分区最大为1ZB
  
## 寻求帮助
- 指令 `xx --help`即可知道选项和参数
- 其他 `man xx`
- man page
  - 第一行\[xx(1)] (1)中的数字表示xx的含义
  
    | 代号 | 代表内容
    | ---- | -------
    | **1** | **shell中可操作的指令或可执行文件**
    | 2 | 系统核心可呼叫的函数与工具
    | 3 | 常用的function lib
    | 4 | 设备文件，通常在/dev下
    | **5** | **配置文件或某些文件的格式**
    | 6 | games
    | 7 | 惯例与协议
    | **8** | **系统管理员可使用的管理指令**
    | 9 | 跟kernal有关的文件
  - NAME 简短指令
  - SYNOPSIS 简短的指令下达语法
  - DESCRIPTION 完整说明
  - OPTIONS 针对SYNOPSIS中的选项的说明
  - COMMANDS 程序执行中可下达的指令
  - FILES 使用、参考或连接到的文件
  - SEE ALSO 相关说明
  - EXAMPLE
- 操作快捷键
  - 空格键
  - Page Up/Page Down/Home/End
  - /string 向下搜索 ?string向上搜索，n/N：下一个/前一个
  - q 退出
- `whatis` `apropos` 需要执行`mandb`建立whatis数据库
- `man -f xx` 搜索完整的指令/数据名称，相当于`whatis`
- `man -k xx` 搜索关键字，相当于`apropos`
- `info xx` 分页之后的说明
- `/usr/share/doc` 其他说明文件

## 关机
- `who`查看谁在线
- `netstat` -a查看
- `ps -aux`查看后台程序
- 过去，关机前多执行几次`sync`将内存数据写入硬盘
- tty1~tty7中登入系统时，可使用任意账号关机(都要输入root密码)，远程工具登录只能用root关机
- `shutdown`
  - `-k` 只发送警告讯息
  - `-r` 停掉系统服务后立即重启
  - `-h` 停掉系统服务后立即关机
  - `-c` 取消
  - `shutdown -h now`
  - `shutdown -h 20:25`
  - `shutdown -h +10`过10分钟
  - `shutdown -r +30 'The system will reboot'`
- `halt` `poweroff` `reboot` `shutdown`都是呼叫systemctl指令
  - `systemctl reboot`
  - `systemctl poweroff`
  
## 权限
- 记录
  - 账号在/etc/passwd文件中
  - 密码在/etc/shadow文件中
  - 组名在/ect/group文件中
### 文件的权限
- `ls -l`
  - 第一栏：文件的类型与权限(10个字符)
    - 1: d表示目录，-表示文件，l表示link file，b表示存储设备，c表示串口设备
    - 2-4: 文件拥有者的权限
    - 5-7: 群组账号的权限
    - 8-10: 其他群组其他账号的权限
  - 第二栏：有多少文件名连接到此节点(i-node)
  - 第三栏： 拥有者账号
  - 第四栏： 文件所属群组
  - 第五栏： 文件大小，默认单位为bytes
  - 第六栏： 文件创建时间或最近修改时间
  - 第七栏： 文件名
  - 纯文本终端显示中文：export LC_ALL=en_US.utf8
- `chgrp groupname [-R] dirname/filename` groupname必须已存在，-R表示递归修改文件的群组
- `chown username[:groupname] [-R] dirname/filename`
- `chmod [-R] r:4,w:2,x:1 dirname/filename`
- `chmod [-R] u/g/o/a +/-/= r/w/x dirname/filename`
### 文件类型
- regular file `-` 
- directory `d`
- link file `l`
- block `b`
- character `c`
- sockets `s`
- FIFO,pipe `p` 解决多个程序同时存取一个文件造成的错误

## 目录
### 目录的权限
- r: ls dir
- w: 在dir中操作文件名(增、删、改)
- x: cd dir
- 一般r与x同时存在
### FHS(Filesystem Hierarchy Standard)
- 概念 static/variable shareable/unshareable
- 定义了3层目录
  - / 与开机系统有关，所在分区越小越不容易发生问题，/etc /bin /dev /lib /sbin必须与根目录一起，否则无法使用救援模式
  - /usr 与软件安装/执行有关
  - /var 与系统运行有关
- 要求必须存在的目录
  - /bin 单人维护模式下还能被执行的指令
  - /boot 开机会用到的文件，包括Linux核心文件及开机的配置文件，比如linux kernal文件vmlinuz，/boot/grub2
  - /dev 比如/dev/null /dev/zero /dev/tty /dev/loop* /dev/sd*
  - /etc 系统的主要配置文件
  - /lib 开机时会用的函数库和/bin /sbin下的指令会调用的函数库 /lib/modules放驱动
  - /media 可移除的设备，比如/media/floppy /media/cdrom
  - /mnt 暂时挂载额外设备的目录
  - /opt 第三方软件
  - /run 开机后产生的信息
  - /sbin 开机过程中需要的的指令，包括开机、修复、还原系统需要的指令，比如fdisk,fsck,ipconfig,mkfs等
  - /srv 服务的数据目录，比如www服务
  - /tmp 正在执行的程序暂时存放文件的地方，建议开机时清除
  - /usr Unix Software Resource现在很多Linux distributions已将很多非必要的文件移出/usr，/usr被建议为即使挂载为只读，系统仍然可以正常运作
    - 必须存在：/usr/bin 部分Linux使用link将/bin链接至此，即/bin和/usr/bin完全相同
    - 必须存在：/usr/lib /lib链接至此
    - 必须存在：/usr/local 管理员在本机下载安装的软件
    - 必须存在：/usr/sbin /sbin链接至此
    - 必须存在：/usr/share 只读架构的文件，比如 /sur/share/man /usr/share/doc
    - 建议存在：/usr/games
    - 建议存在：/usr/include c/c++的header
    - 建议存在：/usr/libexec 不被一般使用者惯用的程序脚本等
    - 建议存在：/usr/lib\<qual> /lib\<qual>链接至此
    - 建议存在：/usr/src 一般原始代码建议放到这里，核心原始代码建议放到/usr/src/linux
  - /var
    - 必须存在：/var/cache 存放程序运行中产生的临时文件
    - 必须存在：/var/lib 存放程序运行时需要用到的数据文件，每个软件要有自己的目录，比如MySQL数据放到/var/lib/mysql中去
    - 必须存在：/var/lock 给设备或文件上锁，使其一次只能被一个程序访问，已挪到/run/lock
    - 必须存在：/var/log 登录文件的放置目录，比如/var/log/messages /var/log/wtmp
    - 必须存在：/var/email 放置个人邮件信箱，已被挪到/var/spool/email，这两个目录互为链接文件
    - 必须存在：/var/run 防止程序的PID，链接到/run了
    - 必须存在：/var/spool 放置队列数据，排队等待其他程序使用，使用后就会被删除
- 建议可以存在的目录
  - /home ~代表当前用户的home目录，~user代表user的home目录
  - /lib\<equal> 与/lib不同的格式的函数
  - /root
- 未定义的目录
  - /lost+found ext2/ext3/ext4文件系统格式才会产生的一个目录
  - /proc 虚拟文件系统，数据都在内存中，比如/proc/cpuinfo /proc/dma /proc/interrupts /proc/net/* /proc/ioports
  - /sys 虚拟文件系统，记载核心与系统硬件信息
  ### 目录操作
  - `pwd[ -P]` 加上-P之后显示实际的工作目录，而非链接文件本身的目录名
  - `mkdir[ -mp]` -p 表示递归建立目录，-m abc表示直接设置权限
  - `rmdir[ -p]` -p递归删除上层空目录
  - `cp src dst` -p 连同文件的属性(权限、用户、时间)一起负责 -r 递归持续复制
## 默认权限
- `umask [-S]` 结果为`0022`，表示需要去掉的权限，文件为666-022，文件夹为777-022

## 查找文件
- `which [-a] command` 查找指令的文件的路径
- `whereis [-mbus] 文件或目录名` 在特殊的目录中找文件
  - b 只找binary格式的文件
  - m 只找manul路径下的文件
  - s 只找source来源文件
  - u 不在上述三个项目中的其他特殊文件
  - l 列出whereis回去查询的主要目录
