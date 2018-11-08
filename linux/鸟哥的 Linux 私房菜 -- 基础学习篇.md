title: 鸟哥的 Linux 私房菜 -- 基础学习篇
author: xg wang
date: 2018-07-09 14:34:30
categories:
  - linux
---

这本书认真读的部分是:第二部分，第三部分，第四部分，第一部分的主机规划与磁盘分区。第五部分的备份和rpm包管理。

#### 第三章、主机规划与磁盘分区  

**磁碟的第一个磁区**主要记录了两个重要的资讯，分别是：
1. 主要启动记录区(Master Boot Record, MBR)：可以安装启动管理程序的地方，有446 bytes
2. 分割表(partition table)：记录整颗硬盘分割的状态，有64 bytes

MBR是很重要的，因为当系统在启动的时候会主动去读取这个区块的内容，这样系统才会知道你的程序放在哪里且该如何进行启动。 如果你要安装多重启动的系统，MBR这个区块的管理就非常非常的重要了！ ^_^

那么分割表又是啥？其实你刚刚拿到的整颗硬盘就像一根原木，你必须要在这根原木上面切割出你想要的区段， 这个区段才能够再制作成为你想要的家具！如果没有进行切割，那么原木就不能被有效的使用。 同样的道理，你必须要针对你的硬盘进行分割，这样硬盘才可以被你使用的！

**磁盘分区表(partition table)**
64 bytes:分成4部分。叫做分割。
* 其实所谓的『分割』只是针对那个64 bytes的分割表进行配置而已！
* 硬盘默认的分割表仅能写入四组分割资讯
* 这四组分割资讯我们称为主要(Primary)或延伸(Extended)分割槽
* 分割槽的最小单位为磁柱(cylinder)
* 当系统要写入磁碟时，一定会参考磁盘分区表，才能针对某个分割槽进行数据的处理
* Primary或Extended。Extended最多只有一个。

就是以前说的，主分区最多只有4个的意思。
当要求分区大于4的时候：扩展分配的目的是使用额外的磁区来记录分割资讯，扩展分配本身并不能被拿来格式化

1. 主要分割与扩展分配最多可以有四笔(硬盘的限制)
2. 扩展分配最多只能有一个(操作系统的限制)
3. 逻辑分割是由扩展分配持续切割出来的分割槽；
4. 能够被格式化后，作为数据存取的分割槽为主要分割与逻辑分割。扩展分配无法格式化；
5. 逻辑分割的数量依操作系统而不同，在Linux系统中，IDE硬盘最多有59个逻辑分割(5号到63号)， SATA硬盘则有11个逻辑分割(5号到15号)。

**启动流程与主要启动记录区(MBR)**

1. CMOS是记录各项硬件参数且嵌入在主板上面的储存器
2. BIOS：启动主动运行的韧体，会认识第一个可启动的装置；
3. MBR：第一个可启动装置的第一个磁区内的主要启动记录区块，内含启动管理程序；
4. 启动管理程序(boot loader)：一支可读取核心文件来运行的软件；
5. 核心文件：开始操作系统的功能...

boot loader 是安装在MBR上的一套软件，提供的功能有：
1. 提供菜单：使用者可以选择不同的启动项目，这也是多重启动的重要功能！
2. 加载核心文件：直接指向可启动的程序区段来开始操作系统；
3. 转交其他loader：将启动管理功能转交给其他loader负责。启动管理程序除了可以安装在MBR之外， 还可以安装在每个分割槽的启动磁区(boot sector)。这个特色才能造就『多重启动』的功能啊！


**文件系统特性**

Linux 操作系统的文件权限(rwx)与文件属性(拥有者、群组、时间参数等)。 文件系统通常会将这两部份的数据分别存放在不同的区块，权限与属性放置到 inode 中，至于实际数据则放置到 data block 区块中。 另外，还有一个超级区块 (superblock) 会记录整个文件系统的整体信息，包括 inode 与 block 的总量、使用量、剩余量等。

1. superblock：记录此 filesystem 的整体信息，包括inode/block的总量、使用量、剩余量， 以及文件系统的格式与相关信息等；
2. inode：记录文件的属性，一个文件占用一个inode，同时记录此文件的数据所在的 block 号码；
3. block：实际记录文件的内容，若文件太大时，会占用多个 block 。
4. 文件系统最前面有一个启动扇区(boot sector)，这个启动扇区可以安装启动管理程序

如果我的文件系统高达数百GB时， 那么将所有的 inode 与 block 通通放置在一起将是很不智的决定，因为 inode 与 block 的数量太庞大，不容易管理。为此之故，因此 Ext2 文件系统在格式化的时候基本上是区分为多个区块群组 (block group) 的，每个区块群组都有独立的 inode/block/superblock 系统。

1. Ext2 是索引式文件系统，基本上不太需要常常进行碎片整理的。
2. FAT 是链接式文件系统，需要经常的碎片整理一下。因为文件写入的block太过离散了。


#### 第五章、Linux 的文件权限与目录配置

ls -la 说明：
1. 当为[ d ]则是目录，例如上表档名为『.gconf』的那一行；
2. 当为[ - ]则是文件，例如上表档名为『install.log』那一行；
3. 若是[ l ]则表示为连结档(link file)；
4. 若是[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；
5. 若是[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。
6. [p] FIFO 也是一种特殊的文件类型,他主要的目的在解决多个程序同时存取一个文件所造成的错误问题。 FIFO是 first-in-first-out 的缩写。
7. 区块(block)设备档 :就是一些储存数据, 以提供系统随机存取的接口设备,举例来说,硬盘与软盘等就是啦! 你可以随机的在硬盘的不同区块读写,这种装置就是成组设备啰!你可以自行查一下/dev/sda 看看, 会发现第一个属性为[ b ]喔!
8. 字符(character)设备文件:亦即是一些串行端口的接口设备, 例如键盘、鼠标等等!这些设备的特色就是『一次性读取』的,不能够截断输出。 举例来说,你不可能让鼠标『跳到』另一个画面,而是『连续性滑动』到另一个地方啊!第一个属性为 [ c ]

第二栏是：多少文档名连结到此节点。

第三栏是：权限 rwxrwxtwx

1. chgrp :改变文件所属群组
2. chown :改变文件拥有者
3. chmod :改变文件的权限, SUID, SGID, SBIT 等等的特性
4. cp: 会复制所给的权限

**Filesystem Hierarchy Standard (FHS)**

1. (**root**, 根目录):与开机系统有关;
2. **/usr** (unix software resource):与软件安装/执行有关;
3. **/var** (variable):与系统运作过程有关。
4. **/run** 早期的 FHS 规定系统开机后所产生的各项信息应该要放置到 /var/run 目录下,新版的 FHS 则规范到/run 底下。 由于 /run 可以使用内存来仿真,因此效能上会好很多!
5. **/lost+found** 这个目录是使用标准的 ext2/ext3/ext4 文件系统格式才会产生的一个目录,目的在于当文件系统发生错误时, 将一些遗失的片段放置到这个目录下。不过如果使用的是 xfs 文件系统的话,就不会存在个目录了
6. **/tmp** 这是让一般用户或者是正在执行的程序暂时放置文件的地方。 这个目录是任何人都能够存取的,所以你需要定期的清理一下。当然,重要数据不可放置在此目录啊! 因为 FHS 甚至建议在开机时,应该要将/tmp 下的数据都删除唷!
7. **/proc** 这个目录本身是一个『虚拟文件系统(virtual filesystem)』喔!他放置的数据都是在内存当中, 例如系统核心、行程信息(process)、周边装置的状态及网络状态等等。因为这个目录下的数据都是在内存当中, 所以本身不占任何硬盘空间啊!比较重要的文件例如:/proc/cpuinfo, /proc/dma, /proc/interrupts,/proc/ioports, /proc/net/* 等等。

usr 子目录介绍：

1. **/usr/bin/ **所有一般用户能够使用的指令都放在这里!
2. **/usr/lib/  **
3. ** /usr/local/** 系统管理员在本机自行安装自己下载的软件(非 distribution 默认提供者),建议安装到此目录,
4. **/usr/sbin/** 非系统正常运作所需要的系统指令。最常见的就是某些网络服务器软件的服务指令(daemon)啰!
5. **/usr/share/** 主要放置只读架构的数据文件,当然也包括共享文件。
6. ** /usr/games/**
7. **/usr/include/**
8. **/usr/libexec/** 某些不被一般使用者惯用的执行档或脚本(script)等等,都会放置在此目录中。例如大部分的 X 窗口底下的操作指令, 很多都是放在此目录下的。
9. **/usr/src/**

var 子目录介绍：/var 就是在系统运作后才会渐渐占用硬盘容量的目录。 因为/var 目录主要针对常态性变动的文件,包括快取(cache)、登录档(log file)以及某些软件运作所产生的文件, 包括程序文件(lock file, run file),或者例如 MySQL 数据库的文件等等。

1. **/var/cache/** 应用程序本身运作过程中会产生的一些暂存档;
2. **/var/lib/**  程序本身执行的过程中,需要使用到的数据文件放置的目录。在此目录下各自的软件应该要有各自的目录。
3. **/var/lock/**
4. **/var/log**
5. **/var/mail/**放置个人电子邮件信箱的目录,不过这个目录也被放置到/var/spool/mail/目录中! 通常这两个目录是互为链接文件啦!
6. **/var/run/**某些程序或者是服务启动后,会将他们的 PID 放置在这个目录下喔!至于 PID 的意义我们会在后续章节提到的。 与 /run 相同,这个目录链接到 /run 去了!
7. **/var/spool/** 这个目录通常放置一些队列数据,所谓的『队列』就是排队等待其他程序使用的数据啦!这些数据被使用后通常都会被删除。


linux 删除文件的权限是：
1. 第一种情况：考虑文件所属的目录，只要用户对文件所属的目录有wx权限，就能进入目录，删掉你的文件（不管你的文件是什么权限）
2. 第二种情况：用户对文件所属目录没有wx权限，这时候需要用户对你的文件有w权限就能删除了

#### 第六章、Linux 文件与目录管理

1. cd : Change Directory
2. pwd : Print Working Directory
3. mkdir -m 711 test2 创建带权限的文件
4. mkdir -p test1/test2/test3/test4 自动建立多级目录
5. ls -a -A -d 只有目录 -f 直接列出结果而不排序 -h 文件容量以比较易读的方式 -i inode信息 -l -n -r 反向输出 -R 递归输出 -S 以文件大小排序 -t 以时间排序 因为这个命令很多很多年了，所以参数是真的多啊。

**复制、删除与移动:**

1. cp [-adfilprsu] 来源文件(source) 目标文件(destination).这个指令是非常重要的,不同身份者执行这个指令会有不同的结果产生。
    1. -d :若来源文件为链接文件的属性(link file),则复制链接文件属性而非文件本身;
    2. -i :文件存在时候，询问？
    3. -l: 进行硬式连结(hard link)的连结档建立,而非复制文件本身;
    4. -s: 复制成为符号链接文件 (symbolic link),亦即『快捷方式』文件;
    5. **-a:** 相当于 -dr --preserve=all 的意思,至于 dr 请参考下列说明;(常用)
    6. -u: destination 比 source 旧才更新 destination,或 destination 不存在的情况下才复制。 update
    7. **-p**: 连同文件的属性(权限、用户、时间)一起复制过去,而非使用默认属性(备份常用);

    因此当我们在进行备份的时候,某些需要特别注意的特殊权限文件, 例如密码文件 (/etc/shadow) 以及一些配置文件,就不能直接以 cp 来复制,而必须要加上 -a 或者是 -p 等等可以完整复制文件权限的选项才行!

2. rm
    1. -f 就是 force 的意思,忽略不存在的文件,不会出现警告讯息;
    2. -i 互动模式,在删除前会询问使用者是否动作
    3. -r 递归删除啊!最常用在目录的删除了!
3. mv
    1. -f
    2. -i
    3. -u: 若目标文件已经存在,且 source 比较新,才会更新 (update)

4. cat 由第一行开始显示文件内容


5. tac : concatenate and print files in reverse


6. nl : number lines of files


7. more : 一页一页的显示文件内容


8. less : man 这个指令就是呼叫 less 来显示说明文件的内容的.很丰富的搜索功能


9.  head : 只看头几行
    1.  head [-n number] 文件
    2.  head -n -100 /etc/man_db.conf 最后100行不打印


10. tail : 只看尾巴几行
    1.  tail -n +100 /etc/man_db.conf 100行之后
    2.  tail -n 20 /etc/man_db.conf


11. od : 以二进制的方式读取文件内容!


12. touch : Update the access and modification times of each FILE to the current time.
    1. -a : 仅修订 access time;
    2. -c : 仅修改文件的时间,若该文件不存在则不建立新文件;
    3. -d : 后面可以接欲修订的日期而不用目前的日期,也可以使用 --date="日期或时间"
    4. -m : 仅修改 mtime ;
    5. -t : 后面可以接欲修订的时间而不用目前的时间,格式为[YYYYMMDDhhmm]


13. chattr : 配置文件案隐藏属性
    1. A : 当设定了 A 这个属性时,若你有存取此文件(或目录)时,他的访问时间 atime 将不会被修改,可避免 I/O 较慢的机器过度的存取磁盘。(目前建议使用文件系统挂载参数处理这个项目)
    2. S : 一般文件是异步写入磁盘的(原理请参考前一章 sync 的说明),如果加上 S 这个属性时,当你进行任何文件的修改,该更动会『同步』写入磁盘中。
    3. a : 当设定 a 之后,这个文件将只能增加数据,而不能删除也不能修改数据,只有 root 才能设定这属性
    4. c : 这个属性设定之后,将会自动的将此文件『压缩』,在读取的时候将会自动解压缩,但是在储存的时候,将会先进行压缩后再储存(看来对于大文件似乎蛮有用的!)
    5. d : 当 dump 程序被执行的时候,设定 d 属性将可使该文件(或目录)不会被 dump 备份
    6. i : 这个 i 可就很厉害了!他可以让一个文件『不能被删除、改名、设定连结也无法写入或新增数据!』对于系统安全性有相当大的帮助!只有 root 能设定此属性
    7. s : 当文件设定了 s 属性时,如果这个文件被删除,他将会被完全的移除出这个硬盘空间,所以如果误删了,完全无法救回来了喔!
    8. u : 与 s 相反的,当使用 u 来配置文件案时,如果该文件被删除了,则数据内容其实还存在磁盘中,可以使用来救援该文件喔!


14.  umask : set file mode creation mask 。文件目录默认权限
    1.  umask -s : u=rwx,g=rwx,o=rx
    2.  umask : 002 被拿掉的权限
    3.  umaks 002 : 设定同组的人可以修改文件


15. **文件特殊权限: SUID, SGID, SBIT**
    1. **SUID** : /usr/bin/passwd -rwsr-xr-x 代表当用户执行此一 binary 程序时,在执行过程中用户会暂时具有程序拥有者的权限
        1. 权限仅对二进制程序(binary program)有效;
        2. 执行者对于该程序需要具有 x 的可执行权限;
        3. 本权限仅在执行该程序的过程中有效 (run-time);
        4. 执行者将具有该程序拥有者 (owner) 的权限。
    2. 目录具有 **SGID** 的特殊权限时,代表用户在这个目录底下新建的文件之群组都会与该目录的组名相同。
        1. SGID 对二进制程序有用;
        2. 程序执行者对于该程序来说,需具备 x 的权限;
        3. 执行者在执行的过程中将会获得该程序群组的支持!
    3. 目录具有 **SBIT** 的特殊权限时,代表在该目录下用户建立的文件只有自己与 root 能够删除!当甲这个用户于 A 目录是具有群组或其他人的身份,并且拥有该目录 w 的权限, 这表示『甲用户对该目录内任何人建立的目录或文件均可进行 "删除/更名/搬移" 等动作。』 不过,如果将 A 目录加上了 SBIT 的权限项目时, 则甲只能够针对自己建立的文件或目录进行删除/更名/移动等动作,而无法删除他人的文件.


1.  file : 观察文件类型


**指令与文件的搜寻**
1. which 
    1. -a : 将所有由 PATH 目录中可以找到的指令均列出,而不止第一个被找到的指令名称
2. whereis :  只找系统中某些特定目录底下的文件
    1. -l : 可以列出 whereis 会去查询的几个主要目录而已
    2. -b : 只找 binary 格式的文件
    3. -m : 只找在说明文件 manual 路径下的文件
    4. -s : 只找 source 来源文件
    5. -u : 搜寻不在上述三个项目当中的其他特殊文件
3. locate : 根据/var/lib/mlocate 内的数据库记载,找出用户输入的关键词文件名。
    1. -i : 忽略大小写的差异;
    2. -c : 不输出档名,仅计算找到的文件数量
    3. -l : 仅输出几行的意思,例如输出五行则是 -l 5
    4. -S : 输出 locate 所使用的数据库文件的相关信息,包括该数据库纪录的文件/目录数量等
    5. -r : 后面可接正规表示法的显示方式


4. **find** ： 全盘扫描，一般来说不要用这个命令。但是确实很有用啊

    **时间相关参数**
    1. -mtime : File was last accessed n*24 hours ago.
        1. n : n 为数字,意义为在 n 天之前的『一天之内』被更动过内容的文件;
        2. -mtime + n :列出在 n 天之前(不含 n 天本身)被更动过内容的文件档名;
        3. -mtime - n :列出在 n 天之内(含 n 天本身)被更动过内容的文件档名。
        4. -newer file :file 为一个存在的文件,列出比 file 还要新的文件档名
    2. -ctime : File's  status was last changed more recently than file was modified.
    3. -mtime : File's  data was last modified n*24 hours ago.

    **用户相关参数**
    1.  -uid n : n 为数字,这个数字是用户的账号 ID,亦即 UID ,这个 UID 是记录在/etc/passwd 里面与账号名称对应的数字。这方面我们会在第四篇介绍。
    2. -gid n :n 为数字,这个数字是组名的 ID,亦即 GID,这个 GID 记录在/etc/group,相关的介绍我们会第四篇说明~
    3. -user name : name 为使用者账号名称喔!例如 dmtsai
    4. -group name : name 为组名喔,例如 users ;
    5. -nouser : 寻找文件的拥有者不存在 /etc/passwd 的人!
    6.  -nogroup : 寻找文件的拥有群组不存在于 /etc/group 的文件!当你自行安装软件时,很可能该软件的属性当中并没有文件拥有者,这是可能的!在这个时候,就可以使用 -nouser 与 -nogroup 搜寻。


    **文件权限相关参数**
    1. -name filename:搜寻文件名为 filename 的文件;
    2. -size [+-]SIZE:搜寻比 SIZE 还要大(+)或小(-)的文件。这个 SIZE 的规格有:
    3. c: 代表 byte, k: 代表 1024bytes。所以,要找比 50KB还要大的文件,就是『 -size +50k 』
    4. -type TYPE:搜寻文件的类型为 TYPE 的,类型主要有:一般正规文件 (f), 装置文件 (b, c),目录 (d), 连结档 (l), socket (s), 及 FIFO (p) 等属性。
    5. -perm mode:搜寻文件权限『刚好等于』 mode 的文件,这个 mode 为类似 chmod的属性值,举例来说, -rwsr-xr-x 的属性为 4755 !
    6. -perm -mode :搜寻文件权限『必须要全部囊括 mode 的权限』的文件.

    **额外可进行的动作**
    1. exec command : command 为其他指令,-exec 后面可再接额外的指令来处理搜寻到的结果。
        1. find /usr/bin /usr/sbin -perm /7000 -exec ls -l {} \; {}是find找到的内容 \;结束命令
    2. -print: 将结果打印到屏幕上,这个动作是预设动作!
    3. find /etc -size +50k -a -size -60k -exec ls -l {} \; -a and
    4. find /etc -size +50k -a ! -user root -type f -exec ls -l {} \; ! 不是
    5. find /etc -size +1500k -o -size 0 -o or


#### 第七章、Linux 磁盘与文件系统管理

##### 7.1.1 磁盘组成与分区的复习
1. /dev/sd[a-p][1-128]:为实体磁盘的磁盘文件名;\
2. /dev/vd[a-d][1-128]:为虚拟磁盘的磁盘文件名

传统的磁盘与文件系统之应用中,一个分区槽就是只能够被格式化成为一个文件系统,所以我们可以说一个 filesystem 就是一个 partition。但是由于新技术的利用,例如我们常听到的 LVM 与软件磁盘阵列(software raid), 这些技术可以将一个分区槽格式化为多个文件系统(例如 LVM),也能够将多个分区槽合成一个文件系统(LVM, RAID)! 所以说,目前我们在格式化时已经不再说成针对 partition来格式化了, **通常我们可以称呼一个可被挂载的数据为一个文件系统而不是一个分区槽喔!**

Ext2 文件系统在格式化的时候基本上是区分为多个区块群组 (block group) 的,每个区块群组都有独立的 inode/block/superblock 系统。
    1. Block 大小 1KB 2KB 4KB
    2. 最大单一文件限制 16GB 256GB 2TB
    3. 最大文件系统总容量 2TB 8TB 16TB

**inode:**
1. 每个 inode 大小均固定为 128 bytes (新的 ext4 与 xfs 可设定到 256 bytes);
2. 每个文件都仅会占用一个 inode 而已;因此文件系统能够建立的文件数量与 inode 的数量有关;
3. 系统读取文件时需要先找到 inode,并分析 inode 所记录的权限与用户是否符合,若符合才能够开始实际读取 block 的内容。

**Superblock (超级区块)**:
1. block 与 inode 的总量;
2. 未使用与已使用的 inode / block 数量;
3. block 与 inode 的大小 (block 为 1, 2, 4K,inode 为 128bytes 或 256bytes);
4. filesystem 的挂载时间、最近一次写入数据的时间、最近一次检验磁盘 (fsck) 的时间等文件系统的相关信息;
5. 一个 valid bit 数值,若此文件系统已被挂载,则 valid bit 为 0 ,若未被挂载,则 valid bit 为 1 。

dumpe2fs:查看超级快的内容。

**block bitmap (区块对照表)** ：从 block bitmap 当中可以知道哪些 block 是空的
**inode bitmap** ： 则是记录使用与未使用的 inode 号码

**inode 与 目录树的关系** :

 由上面的结果我们知道目录并不只会占用一个 block 而已,也就是说: 在目录底下的文件数如果太多而导致一个 block 无法容纳的下所有的档名与 inode 对照表时, Linux 会给予该目录多一个 block来继续记录相关的数据; **inode 本身并不记录文件名,文件名的记录是在目录的 block 当中**

**读取 /etc/passwd 这个文件** : 读取三次inode，三次block。

**如何新建文件：**

1. 先确定用户对于欲新增文件的目录是否具有 w 与 x 的权限,若有的话才能新增;
2. 根据 inode bitmap 找到没有使用的 inode 号码,并将新文件的权限/属性写入;
3. 根据 block bitmap 找到没有使用中的 block 号码,并将实际的数据写入 block 中,且更新 inode 的 block
指向数据;
4. 将刚刚写入的 inode 与 block 数据同步更新 inode bitmap 与 block bitmap,并更新 superblock 的内容。

**Linux VFS :**

整个 Linux 的系统都是透过一个名为 Virtual Filesystem Switch 的核心功能去读取 filesystem 的。

**现代文件系统：**

1. Ext 文件系统家族对于文件格式化的处理方面,采用的是预先规划出所有的 inode/block/meta data 等数据,未来系统可以直接取用, 不需要再进行动态配置的作法。你的 TB 以上等级的传统 ext 家族文件系统在格式化的时候,光是系
统要预先分配 inode 与 block 就消耗你好多好多时间。
2. centos 文件系统已经由预设的 Ext4 变成了 xfs 这一个较适合高容量磁盘与巨型文件效能较佳的文件系统了。

##### 7.2 文件系统的简单操作

1. df 评估文件系统的磁盘使用量
    1. -a :列出所有的文件系统,包括系统特有的 /proc 等文件系统;
    2. -k :以 KBytes 的容量显示各文件系统;
    3. -m :以 MBytes 的容量显示各文件系统;
    4. -h :以人们较易阅读的 GBytes, MBytes, KBytes 等格式自行显示;
    5. -H :以 M=1000K 取代 M=1024K 的进位方式;
    6. -T :连同该 partition 的 filesystem 名称 (例如 xfs) 也列出;
    7. -i :不用磁盘容量,而以 inode 的数量来显示
2. du 
    1. -a :列出所有的文件与目录容量,因为默认仅统计目录底下的文件量而已。
    2. -h :以人们较易读的容量格式 (G/M) 显示;
    3. -s :列出总量而已,而不列出每个各别的目录占用容量;
    4. -S :不包括子目录下的总计,与 -s 有点差别。
    5. -k :以 KBytes 列出容量显示;
    6. -m :以 MBytes 列出容量显示;
3. ln : 如果不加任何参数的话,那么就是 Hard Link
    1. -H 不能跨 Filesystem. 不能 link 目录。
    2. -s Symbolic link 所建立的文件为一个独立的新的文件,所以会占用掉 inode 与 block

##### 磁盘的分区、格式化、检验与挂载

对我来说不是重点，略过。

#### 第八章、文件与文件系统的压缩,打包与备份

1. *.Z compress 程序压缩的文件;
2. *.zip zip 程序压缩的文件;
3. *.gz gzip 程序压缩的文件;
4. *.bz2 bzip2 程序压缩的文件;
5. *.xz xz 程序压缩的文件;
6. *.tar tar 程序打包的数据,并没有压缩过;
7. *.tar.gz tar 程序打包的文件,其中并且经过 gzip 的压缩
8. *.tar.bz2 tar 程序打包的文件,其中并且经过 bzip2 的压缩
9. *.tar.xz tar 程序打包的文件,其中并且经过 xz 的压缩


gzip, bzip, xz

tar : 
    1. -c :建立打包文件,可搭配 -v 来察看过程中被打包的档名(filename)
    2. -t :察看打包文件的内容含有哪些档名,重点在察看『档名』就是了;
    3. -x :解打包或解压缩的功能,可以搭配 -C (大写) 在特定目录解开特别留意的是, -c, -t, -x 不可同时出现在一串指令列中。
    4. -z :透过 gzip的支持进行压缩/解压缩:此时档名最好为 *.tar.gz
    5. -j :透过 bzip2 的支持进行压缩/解压缩:此时档名最好为 *.tar.bz2
    6. -J :透过 xz 的支持进行压缩/解压缩:此时档名最好为 *.tar.xz 特别留意, -z, -j, -J 不可以同时出现在一串指令列中
    7. -v:在压缩/解压缩的过程中,将正在处理的文件名显示出来!
    8. -f filename:-f 后面要立刻接要被处理的档名!建议 -f 单独写一个选项啰!(比较不会忘记)
    9. -C 目录:这个选项用在解压缩,若要在特定目录解压缩,可以使用这个选项。d
    10. -p 的选项,重点在于『保留原本文件的权限与属性』之意。

tar 的使用例子：
    1. time tar -zpcv -f /root/etc.tar.gz /etc
    2. time tar -Jpcv -f /root/etc.tar.xz /etc
    3. tar -jtv -f /root/etc.tar.bz2 查看


**系统备份略过。**

#### 第九章、vim 程序编辑器

**vi 的使用：**
    1. 一般指令模式 (normal mode)
    2. 编辑模式 (insert mode)
    3. 指令列命令模式 (command-line mode)： 这三个按键都能使: / ?  :

**常用的快捷键：**

1. h,j,k,l num+h,j,k,l
2. ctrl + f, ctrl + b ,ctrl + d, ctrl + u
3. + - n<space> 0(home) end
4. gg G nG n<Enter>
5. H M L
6. 0表示第一行，$表示最后一行

**搜寻与取代**
1. /word
2. ?word
3. n 向下搜索
4. N 向上搜索
5. :n1,n2s/word1/word2/g    golbal
6. :1,$s/word1/word2/g      golbal
7. :1,$s/word1/word2/gc     golbal confirm


**删除、复制与贴上**
1. x X nx 对字符的删除还是不太会
2. dd ndd d1G dG d$ d0
3. yy nyy y1G yG y0 y$
4. p P
5. J 将光标所在列与下一列的数据结合成同一列
6. c 重复删除多个数据,例如向下删除 10 列,[ 10cj ] jklh
7. u 复原
8. ctrl+r 重做一次

**进入插入或取代的编辑模式**
1. i I 行首插入
2. a A A为从光标所在列的最后一个字符处开始插入
3. o O 在目前光标所在的下一列处插入新的一列
4. r R Replace mode

**指令列模式的储存、离开等指令**
1. :w
2. :w!
3. :q
4. :q!
5. :wq!
6. ZZ 文件没有更动,则不储存离开,若文件已经被更动过,则储存后离开!
7. :w filename
8. :r filename 在编辑的数据中,读入另一个文件的数据。
9. :n1,n2 w filename
10. :! ls /home

**区块选择(Visual Block)**
1. v 字符选择,会将光标经过的地方反白选择!
2. V 列选择,会将光标经过的列反白选择!
3. [Ctrl]+v 区块选择,可以用长方形的方式选择资料
1. y 将反白的地方复制起来
2. d 将反白的地方删除掉
3. p 将刚刚复制的区块,在游标所在处贴上!

**多文件编辑**
1. :n 编辑下一个文件
2. :N 编辑上一个文件
3. :files 列出目前这个 vim 的开启的所有文件

**多窗口功能**
1. :sp [filename]
2. [ctrl]+w+ j
3. [ctrl]+w+ k
4. [ctrl]+w+ q

#### 第十章、认识与学习 BASH

**shell 快捷键：**
1. type : 查询指令是否为 Bash shell 的内建命令
2. \ : 反斜杠连接命令
3. [ctrl]+u/[ctrl]+k
4. [ctrl]+a/[ctrl]+e
5. ctrl+p ctrl+n

**shell 变量：**
1. shell变量名称只能是英文字母与数字,但是开头字符不能是数字
2. 变量与变量内容以一个等号『=』来连结
3. 等号两边不能直接接空格符
4. **使用反单引号『`指令`』或 『$(指令)』**
5. 使用:"$变量名称" 或 ${变量} 累加内容.
6. 变量需要在其他子程序执行,则需要以 export 来使变量变成环境变量
7. 取消变量的方法为使用 unset
8. 在变量的设定当中,单引号与双引号的用途有何不同?
    1. 单引号与双引号的最大不同在于双引号仍然可以保有变量的内容,但单引号内仅能是一般字符 ,而不会有特殊符号

**环境变量：**
1. env : 所有的环境变量,环境变量可以被子程序所引用
2. set : 观察所有变量 (含环境变量与自定义变量)

**变量键盘读取、数组与宣告: read, array, declare**:
1. read [-pt] variable
    1. -p :后面可以接提示字符!
    2. -t :后面可以接等待的『秒数!』这个比较有趣~不会一直等待使用者啦!
2. declare [-aixr] variable
    1. -a :将后面名为 variable 的变量定义成为数组 (array) 类型
    2. -i :将后面名为 variable 的变量定义成为整数数字 (integer) 类型
    3. -x :用法与 export 一样,就是将后面的 variable 变成环境变量;
    4. -r :将变量设定成为 readonly 类型,该变量不可被更改内容,也不能 unset
    5. 变量类型默认为『字符串』,所以若不指定变量类型,则 1+2 为一个『字符串』而不是『计算式』 所以上述第一个执行的结果才会出现那个情况的
    6. bash 环境中的数值运算,预设最多仅能到达整数形态,所以 1/3 结果是 0;

**ulimit [-SHacdfltu] [配额]**: **可以输出，也可以配置**:
1. -H :hard limit ,严格的设定,必定不能超过这个设定的数值
2. -S :soft limit ,警告的设定,可以超过这个设定值,但是若超过则有警告讯息。在设定上,通常 soft 会比 hard 小,举例来说,soft 可设定为 80 而 hard设定为 100,那么你可以使用到 90 (因为没有超过 100),但介于 80~100 之间时,系统会有警告讯息通知你!
3. -a :后面不接任何选项与参数,可列出所有的限制额度;
4. -c :当某些程序发生错误时,系统可能会将该程序在内存中的信息写成文件(除错用),这种文件就被称为核心文件(core file)。此为限制每个核心文件的最大容量。
5. -f :此 shell 可以建立的最大文件容量(一般可能设定为 2GB)单位为 Kbytes
6. -d :程序可使用的最大断裂内存(segment)容量;
7. -l :可用于锁定 (lock) 的内存量
8. -t :可使用的最大 CPU 时间 (单位为秒)
9. -u :单一用户可以使用的最大程序(process)数量。

**变量内容的删除、取代与替换 (Optional)**:

**变量内容的删除与取代**

1. ${变量#关键词} 若变量内容从头开始的数据符合『关键词』,则将符合的最短数据删除
2. ${变量##关键词} 若变量内容从头开始的数据符合『关键词』,则将符合的最长数据删除
3. ${变量%关键词} 若变量内容从尾向前的数据符合『关键词』,则将符合的最短数据删除
4. ${变量%%关键词} 若变量内容从尾向前的数据符合『关键词』,则将符合的最长数据删除
5. ${变量/旧字符串/新字符串} 若变量内容符合『旧字符串』则『第一个旧字符串会被新字符串取代』
6. ${变量//旧字符串/新字符串} 若变量内容符合『旧字符串』则『全部的旧字符串会被新字符串取代』

**变量的测试与内容替换**

这个用法是真的奇怪！将来用到在说。

str 没有设定    str 为空字符串  str 已设定非为空字符串
= 会改变str的值
1. var=${str-expr}
2. var=${str:-expr}
3. var=${str+expr}
4. var=${str:+expr}
5. var=${str=expr}
6. var=${str:=expr}
7. var=${str?expr}
8. var=${str:?expr}

**路径与指令搜寻顺序**:

1. 以相对/绝对路径执行指令,例如『 /bin/ls 』或『 ./ls 』;
2. 由 alias 找到该指令来执行;
3. 由 bash 内建的 (builtin) 指令来执行;
4. 透过 $PATH 这个变量的顺序搜寻到的第一个指令来执行。

**bash 的进站与欢迎讯息:**

1. /etc/issue
2. /etc/motd

**bash 的环境配置文件**:

1. /etc/profile (login shell 才会读) 调用流程
    1. /etc/profile.d/*.sh
    2. ~/.bash_profile
    3. ~/.bashrc
    4. /etc/bashrc

1. Ctrl + C 终止目前的命令
2. Ctrl + D 输入结束 (EOF),例如邮件结束的时候;
3. Ctrl + M 就是 Enter 啦!
4. Ctrl + S 暂停屏幕的输出
5. Ctrl + Q 恢复屏幕的输出Ctrl + U 在提示字符下,将整列命令删除
6. Ctrl + Z 『暂停』目前的命令


set [-uvCHhmBx]:

    1. -u :预设不启用。若启用后,当使用未设定变量时,会显示错误讯息;
    2. -v :预设不启用。若启用后,在讯息被输出前,会先显示讯息的原始内容;
    3. -x :预设不启用。若启用后,在指令被执行前,会显示指令内容(前面有 ++ 符号)
    4. -h :预设启用。与历史命令有关;
    5. -H :预设启用。与历史命令有关;-m :预设启用。与工作管理有关;
    6. -B :预设启用。与刮号 [] 的作用有关;
    7. -C :预设不启用。若使用 > 等,则若文件存在时,该文件不会被覆盖。

**通配符与特殊符号**:

**通配符号：**
1. *    代表『 0 个到无穷多个』任意字符
2. ?    代表『一定有一个』任意字符
3. []   同样代表『一定有一个在括号内』的字符(非任意字符)。例如 [abcd] 代表『一定有一个字符, 可能是 a, b, c, d 这四个任何一个』
4. [-]  若有减号在中括号内时,代表『在编码顺序内的所有字符』。例如 [0-9] 代表 0 到 9 之间的所有数字, 因为数字的语系编码是连续的!
5. [^]  若中括号内的第一个字符为指数符号 (^) ,那表示『反向选择』,例如 [^abc] 代表 一定有一个字符,只 要是非 a, b, c 的其他字符就接受的意思。

**特殊符号**：

1. \# 批注符号:这个最常被使用在 script 当中,视为说明!在后的数据均不执行
2. \ 跳脱符号:将『特殊字符或通配符』还原成一般字符
3. | 管线 (pipe):分隔两个管线命令的界定(后两节介绍);
4. ; 连续指令下达分隔符:连续性命令的界定 (注意!与管线命令并不相同)
5. ~ 用户的家目录
6. $ 取用变数前导符:亦即是变量之前需要加的变量取代值
**7. & 工作控制 (job control):将指令变成背景下工作**
8. ! 逻辑运算意义上的『非』 not 的意思!
9. / 目录符号:路径分隔的符号
**10. >, >> 数据流重导向:输出导向,分别是『取代』与『累加』**
11. <, << 数据流重导向:输入导向 (这两个留待下节介绍)
12. ' ' 单引号,不具有变量置换的功能 ($ 变为纯文本)
13. " " 具有变量置换的功能! ($ 可保留相关功能)
**14. ` ` 两个『 ` 』中间为可以先执行的指令,亦可使用 $( )**
**15. ( ) 在中间为子 shell 的起始与结束**
**16. { } 在中间为命令区块的组合!**

**数据流重导向**:

1. 标准输入 (stdin) :代码为 0 ,使用 < 或 << ;
2. 标准输出 (stdout):代码为 1 ,使用 > 或 >> ;
3. 标准错误输出(stderr):代码为 2 ,使用 2> 或 2>> ;
4. find /home -name .bashrc > list **2>&1** 正确错误输出到同一文件

**命令执行的判断依据: ; , &&, ||**:

1. cmd1 && cmd2 
    1. 若 cmd1 执行完毕且正确执行($?=0),则开始执行 cmd2。
    2. 若 cmd1 执行完毕且为错误 ($?≠0),则 cmd2 不执行。
2. cmd1 || cmd2 
    1. 若 cmd1 执行完毕且正确执行($?=0),则 cmd2 不执行。
    2. 若 cmd1 执行完毕且为错误 ($?≠0),则开始执行 cmd2。 
3. command1 && command2 || command3  如果真要使用判断,那么这个 && 与 || 的顺序就不能搞错

**管线命令 (pipe)**:

每个管线后面接的第一个数据必定是『指令』喔!而且这个指令必须要能够接受 standard input 的数据才行.例如 less, more, head, tail 等都是可以接受 standard input 的管线命令啦。至于例如 ls, cp, mv 等就不是管线命令了!

**撷取命令: cut, grep**:

1. cut : 管道指令
    1. -d :后面接分隔字符。与 -f 一起使用;
    2. -f :依据 -d 的分隔字符将一段讯息分区成为数段,用 -f 取出第几段的意思;
    3. -c :以字符 (characters) 的单位取出固定字符区间;
    4. last | cut -d ' ' -f 1
    5. export | cut -c 12-20
2. grep : [-acinv] [--color=auto] '搜寻字符串' filename
    1. -a :将 binary 文件以 text 文件的方式搜寻数据
    2. -c :计算找到 '搜寻字符串' 的次数
    3. -i :忽略大小写的不同,所以大小写视为相同
    4. -n :顺便输出行号
    5. -v :反向选择,亦即显示出没有 '搜寻字符串' 内容的那一行!
    6. --color=auto :可以将找到的关键词部分加上颜色的显示喔!
3. sort [-fbMnrtuk] [file or stdin]
    1. -f :忽略大小写的差异,例如 A 与 a 视为编码相同;
    2. -b :忽略最前面的空格符部分;
    3. -M :以月份的名字来排序,例如 JAN, DEC 等等的排序方法;
    4. -n :使用『纯数字』进行排序(默认是以文字型态来排序的);
    5. -r :反向排序;
    6. -u :就是 uniq ,相同的数据中,仅出现一行代表;
    7. -t :分隔符,预设是用 [tab] 键来分隔;
    8. -k :以那个区间 (field) 来进行排序的意思
    9. cat /etc/passwd | sort -t ':' -k 3
4. uniq : 
    1. -i :忽略大小写字符的不同;
    2. -c :进行计数
5. wc : 
    1. -l :仅列出行;
    2. -w :仅列出多少字(英文单字);
    3. -m :多少字符;

**双向重导向: tee** : tee 会同时将数据流分送到文件去与屏幕 (screen);

1. -a : 以累加 (append) 的方式,将数据加入 file 当中!
2. ls -l /home | tee ~/homefile | more


**字符转换命令: tr, col, join, paste, expand**:

1. tr 可以用来删除一段讯息当中的文字,或者是进行文字讯息的替换!
    1. -d : 删除讯息当中的 SET1 这个字符串;
    2. -s : 取代掉重复的字符!
    3. last | tr '[a-z]' '[A-Z]'
    4. cat /etc/passwd | tr -d ':'


2. col :
    1. -x : 将 tab 键转换成对等的空格键


3. join : 
    1. -t : join 默认以空格符分隔数据,并且比对『第一个字段』的数据,如果两个文件相同,则将两笔数据联成一行,且第一个字段放在第一个!
    2. -i : 忽略大小写的差异;
    3. -1 : 这个是数字的 1 ,代表『第一个文件要用那个字段来分析』的意思;
    4. -2 : 代表『第二个文件要用那个字段来分析』的意思。
    5. join -t ':' /etc/passwd /etc/shadow | head -n 3
    6. join -t ':' -1 4 /etc/passwd -2 3 /etc/group | head -n 3   **GID**


4. paste : 将两行贴在一起,且中间以 [tab] 键隔开
    1. -d : 后面可以接分隔字符。预设是以 [tab] 来分隔的!
    2. - : 如果 file 部分写成 - ,表示来自 standard input 的资料的意思。


5. expand : 按键转成空格键啦
    1.  t : 后面可以接数字。一般来说,一个 tab 按键可以用 8 个空格键取代。我们也可以自行定义一个 [tab] 按键代表多少个字符呢!


6. split : 大文件依据文件大小或行数来分区
    1. -b : 后面可接欲分区成的文件大小,可加单位,例如 b, k, m 等;
    2. -l : 以行数来进行分区。
    3. ls -al / | split -l 10 - lsroot 啦!一般来说,如果需要 stdout/stdin 时,但又没有文件,有的只是 - 时,那么那个** - 就会被当成 stdin 或 stdout **


7. 参数代换: xargs [-0epn] **很多指令其实并不支持管线命令,因此我们可以透过 xargs 来提供该指令引用 standard input 之用!**
    1. -0 : 如果输入的 stdin 含有特殊字符,例如 `, \, 空格键等等字符时,这个 -0 参数可以将他还原成一般字符。这个参数可以用于特殊状态喔!
    2. -e : 这个是 EOF (end of file) 的意思。后面可以接一个字符串,当 xargs 分析到这个字符串时,就会停止继续工作!
    3. -p : 在执行每个指令的 argument 时,都会询问使用者的意思;
    4. -n : 后面接次数,每次 command 指令执行时,要使用几个参数的意思。当 xargs 后面没有接任何的指令时,默认是以 echo 来进行输出喔!
    5. cut -d ':' -f 1 /etc/passwd | head -n 3 | xargs -p -n 1 id

8. **关于减号 - 的用途**:

在管线命令当中,常常会使用到前一个指令的 stdout 作为这次的stdin , 某些指令需要用到文件名 (例如 tar) 来进行处理时,该 stdin 与 stdout 可以利用减号 "-"来替代

    1. tar -cvf - /home | tar -xvf - -C /tmp/homeback

#### 第十一章、正规表示法与文件格式化处理

**基础正规表示法**:

1. [:alnum:] 代表英文大小写字符及数字,亦即 0-9, A-Z, a-z
2. [:alpha:] 代表任何英文大小写字符,亦即 A-Z, a-z
3. [:blank:] 代表空格键与 [Tab] 按键两者
4. [:cntrl:] 代表键盘上面的控制按键,亦即包括 CR, LF, Tab, Del.. 等等
5. [:digit:] 代表数字而已,亦即 0-9
6. [:graph:] 除了空格符 (空格键与 [Tab] 按键) 外的其他所有按键
7. [:lower:] 代表小写字符,亦即 a-z
8. [:print:] 代表任何可以被打印出来的字符[:punct:] 代表标点符号 (punctuation symbol),亦即:" ' ? ! ; : # $...
9. [:upper:] 代表大写字符,亦即 A-Z
10. [:space:] 任何会产生空白的字符,包括空格键, [Tab], CR 等等
11. [:xdigit:] 代表 16 进位的数字类型,因此包括: 0-9, A-F, a-f 的数字与字符


**grep 的一些进阶选项**:
1. grep [-A] [-B] [--color=auto] '搜寻字符串' filename
    1. -A :后面可加数字,为 after 的意思,除了列出该行外,后续的 n 行也列出来;
    2. -B :后面可加数字,为 befer 的意思,除了列出该行外,前面的 n 行也列出来;
    3. --color=auto 可将正确的那个撷取数据列出颜色

    **例子：**
    4. grep -n '[^[:lower:]]oo' regular_express.txt [a-z]当然就是 [[:lower:]]
    5. grep -n '^[[:lower:]]' regular_express.txt
    6. ^ **符号,在字符集合符号(括号[])之内与之外是不同的! 在 [] 内代表『反向选择』,在 [] 之外则代表定位在行首的意义!**
    7. grep -n '\.$' regular_express.txt 搜索句尾有.
    8. grep -n '^$' regular_express.txt 搜索空行.
    9. grep -v '^$' /etc/rsyslog.conf | grep -v '^#'  不要开头是 # 的那行

    **特殊字符：**
    10. . (小数点):代表『一定有一个任意字符』的意思;
    11. * (星星号):代表『重复前一个字符, 0 到无穷多次』的意思,为组合形态
    12. 




#### 第十二章、学习 Shell Scripts





























#### 第十三章、Linux 账号管理与 ACL 权限设定


**ssh 系统处理流程：**
1. 先找寻 /etc/passwd 里面是否有你输入的账号?如果没有则跳出,如果有的话则将该账号对应的 UID 与
GID (在 /etc/group 中) 读出来,另外,该账号的家目录与 shell 设定也一并读出;
2. 再来则是核对密码表啦!这时 Linux 会进入 /etc/shadow 里面找出对应的账号与 UID,然后核对一下你刚
刚输入的密码与里头的密码是否相符?
3. 如果一切都 OK 的话,就进入 Shell 控管的阶段啰!


/etc/passwd
/etc/shadow
/etc/group
/etc/gshadow

**有效群组(effective group)与初始群组(initial group)**:

groups:查看有效群组

**什么是 ACL 与如何支持启动 ACL**：

ACL 可以针对单一使用者,单一文件或目录来进行r,w,x 的权限规范,对于需要特殊权限的使用状况非常有帮助。

1. getfacl
2. setfacl

**使用者身份切换**:

1. su
2. sudo
3. visudo

**PAM 模块简介**:

PAM 可以说是一套应用程序编程接口 (Application Programming Interface, API),他提供了一连串的验证机制,只要使用者将验证阶段的需求告知 PAM 后, PAM 就能够回报使用者验证的结果 (成
功或失败)。

**Linux 主机上的用户讯息传递**:

1. w
2. who


#### 第十五章、例行性工作排程(crontab)

**Linux 系统常见的例行性任务有:**

1. 进行登录档的轮替 (log rotate)
2. 登录文件分析 logwatch 的任务
3. 建立 locate 的数据库
4. man page 查询数据库的建立
5. RPM 软件登录文件的建立
6. 移除暂存档
7. 与网络服务有关的分析行为

1. at : 单一工作排程的进行就使用 at
    1. -m :当 at 的工作完成后,即使没有输出讯息,亦以 email 通知使用者该工作已完成。
    2. -l :at -l 相当于 atq,列出目前系统上面的所有该用户的 at 排程;
    3. -d :at -d 相当于 atrm ,可以取消一个在 at 排程中的工作;
    4. -v :可以使用较明显的时间格式栏出 at 排程中的任务栏表;
    5. -c :可以列出后面接的该项工作的实际指令内容。
    6. echo "Hello" > /dev/tty1
    7. 根据时间，根据CPU负载来设置使用量。
2. atq
3. atobm
4. crontab 循环执行的例行性工作排程则是由 cron (crond) 这个系统服务来控制的。

#### 第十六章、进程管理与 SELinux 初探

**bash 的 job control 必须要注意到的限制是:**

1. 这些工作所触发的进程必须来自于你 shell 的子进程(只管理自己的 bash);
2. 前景:你可以控制与下达指令的这个环境称为前景的工作 (foreground);
3. 背景:可以自行运作的工作,你无法使用 [ctrl]+c 终止他,可使用 bg/fg 呼叫该工作;
4. 背景中『执行』的进程不能等待 terminal/shell 的输入(input)

1. &
2. 将『目前』的工作丢到背景中『暂停』: [ctrl]-z
3. jobs [-lrs] 观察目前的背景工作状态
    1. -l : 除了列出 job number 与指令串之外,同时列出 PID 的号码;
    2. -r : 仅列出正在背景 run 的工作;
    3. -s : 仅列出正在背景当中暂停 (stop) 的工作。
4. fg : %jobnumber
    1. fg - 取出 - 号的工作
5. bg : 让工作在背景下的状态变成运作
6. kill : -signal %jobnumber 
    1. -1 :重新读取一次参数的配置文件 (类似 reload);
    2. -2 :代表与由键盘输入 [ctrl]-c 同样的动作;
    3. -9 :立刻强制删除一个工作;
    4. -15:以正常的进程方式终止一项工作。与 -9 是不一样的。
7. nohup

**进程管理**:

1. ps
    1. ps aux <==观察系统所有的进程数据
    2. ps -lA <==也是能够观察所有系统的数据
    3. ps axjf <==连同部分进程树状态
    4. -A :所有的 process 均显示出来,与 -e 具有同样的效用;
    5. -a :不与 terminal 有关的所有 process ;
    6. -u :有效使用者 (effective user) 相关的 process ;
    7. -x : 通常与 a 这个参数一起使用,可列出较完整信息。输出格式规划:
    8. -l :较长、较详细的将该 PID 的的信息列出;
    9. -j :工作的格式 (jobs format)
    10. -f :做一个更为完整的输出。
    11. ps -aux
2.  top:
    1. -d : 后面可以接秒数,就是整个进程画面更新的秒数。预设是 5 秒;
    1. -b : 以批次的方式执行 top ,还有更多的参数可以使用喔!通常会搭配数据流重导向来将批次的结果输出成为文件。
    2. -n : 与 -b 搭配,意义是,需要进行几次 top 的输出结果。
    3. -p : 指定某些个 PID 来进行观察监测而已。
    4. ? :显示在 top 当中可以输入的按键指令;
    5. P :以 CPU 的使用资源排序显示;
    6. M :以 Memory 的使用资源排序显示;
    7. N :以 PID 来排序喔!
    8. T :由该 Process 使用的 CPU 时间累积 (TIME+) 排序。
    9. k :给予某个 PID 一个讯号(signal)
    10. r :给予某个 PID 重新制订一个 nice 值。
    11. q :离开 top 软件的按键。
3. pstree : 
    1. -A : 各进程树之间的连接以 ASCII 字符来连接;
    2. -U : 各进程树之间的连接以万国码的字符来连接。在某些终端接口下可能会有错误;
    3. -p : 并同时列出每个 process 的 PID;
    4. -u : 并同时列出每个 process 的所属账号名称。d
4. signal
    1. 1 SIGHUP 启动被终止的进程,可让该 PID 重新读取自己的配置文件,类似重新启动
    2. 2 SIGINT 相当于用键盘输入 [ctrl]-c 来中断一个进程的进行
    3. 9 SIGKILL 代表强制中断一个进程的进行,如果该进程进行到一半, 那么尚未完成的部分可能会有『半产品』产生,类似 vim 会有 .filename.swp 保留下来。
    4. 15 SIGTERM 以正常的结束进程来终止该进程。由于是正常的终止, 所以后续的动作会将他完成。不过,如果该进程已经发生问题,就是无法使用正常的方法终止时, 输入这个 signal 也是没有用的。
    5. 19 SIGSTOP 相当于用键盘输入 [ctrl]-z 来暂停一个进程的进行
    6. kill -SIGHUP $(ps aux | grep 'rsyslogd' | grep -v 'grep'| awk '{print $2}')
5. killall -signal 指令名称
    1. -1 -9
6. PRI 值越低代表越优先的意思。不过这个 PRI 值是由核心动态调整的, 用户无法直接调整 PRI值的。PRI(new) = PRI(old) + nice。 nice 值可调整的范围为 -20 ~ 19 ;
    1.  nice [-n 数字] command
    2.  renice [number] PID

**系统资源的观察**:

1. free :观察内存使用情况
    1. -b:直接输入 free 时,显示的单位是 Kbytes,我们可以使用 b(bytes), m(Mbytes), k(Kbytes), 及 g(Gbytes) 来显示单位喔!也可以直接让系统自己指定单位 (-h)
    2. -t :在输出的最终结果,显示物理内存与 swap 的总量。
    3. -s :可以让系统每几秒钟输出一次,不间断的一直输出的意思!对于系统观察挺有效!
    4. -c :与 -s 同时处理~让 free 列出几次的意思~
2. uname [-asrmpi]
    1. -a :所有系统相关的信息,包括底下的数据都会被列出来;
    2. -s :系统核心名称
    3. -r :核心的版本
    4. -m :本系统的硬件名称,例如 i686 或 x86_64 等;
    5. -p :CPU 的类型,与 -m 类似,只是显示的是 CPU 的类型!
    6. -i :硬件的平台 (ix86)
3. uptime:观察系统启动时间与工作负载
4. netstat -[atunlp]
    1. -a : 将目前系统上所有的联机、监听、Socket 数据都列出来
    2. -t : 列出 tcp 网络封包的数据
    3. -u : 列出 udp 网络封包的数据
    4. -n : 不以进程的服务名称,以埠号 (port number) 来显示;
    5. -l : 列出目前正在网络监听 (listen) 的服务;
    6. -p : 列出该网络服务的进程 PID
5. dmesg :分析核心产生的讯息
6. vmstat :侦测系统资源变化
    1. -a :使用 inactive/active(活跃与否) 取代 buffer/cache 的内存输出信息;
    2. -f :开机到目前为止,系统复制 (fork) 的进程数;
    3. -s :将一些事件 (开机至目前为止) 导致的内存变化情况列表说明;
    4. -S :后面可以接单位,让显示的数据有单位。例如 K/M 取代 bytes 的容量;
    5. -d :列出磁盘的读写总量统计表
    6. -p :后面列出分区槽,可显示该分区槽的读写总量统计表
7. 特殊文件与进程

    SUID 权限
    1. SUID 权限仅对二进制程序(binary program)有效;
    2. 执行者对于该程序需要具有 x 的可执行权限;
    3. 本权限仅在执行该程序的过程中有效 (run-time);
    4. 执行者将具有该程序拥有者 (owner) 的权限。

    /proc/* 主机上面的各个进程的 PID 都是以目录的型态存在于 /proc 当中
    1. /proc/cmdline 加载 kernel 时所下达的相关指令与参数!查阅此文件,可了解指令是如何启动的!
    2. /proc/cpuinfo 本机的 CPU 的相关信息,包含频率、类型与运算功能等
    3. /proc/devices 这个文件记录了系统各个主要装置的主要装置代号,与 mknod 有关呢!
    4. /proc/filesystems 目前系统已经加载的文件系统啰!
    5. /proc/interrupts 目前系统上面的 IRQ 分配状态。
    6. /proc/ioports 目前系统上面各个装置所配置的 I/O 地址。
    7. /proc/kcore 这个就是内存的大小啦!好大对吧!但是不要读他啦!
    8. /proc/loadavg 还记得 top 以及 uptime 吧?没错!上头的三个平均数值就是记录在此!
    9. /proc/meminfo 使用 free 列出的内存信息,嘿嘿!在这里也能够查阅到!
    10. /proc/modules 目前我们的 Linux 已经加载的模块列表,也可以想成是驱动程序啦!
    11. /proc/mounts 系统已经挂载的数据,就是用 mount 这个指令呼叫出来的数据啦!
    12. /proc/swaps 到底系统挂加载的内存在哪里?呵呵!使用掉的 partition 就记录在此啦!
    13. /proc/partitions 使用 fdisk -l 会出现目前所有的 partition 吧?在这个文件当中也有纪录!
    14. /proc/uptime 就是用 uptime 的时候,会出现的信息啦!
    15. /proc/version 核心的版本,就是用 uname -a 显示的内容啦!
    16. /proc/bus/* 一些总线的装置,还有 USB 的装置也记录在此喔!

8. fuser : 藉由文件(或文件系统)找出正在使用该文件的进程
    1. -u :除了进程的 PID 之外,同时列出该进程的拥有者;
    2. -m :后面接的那个档名会主动的上提到该文件系统的最顶层,对 umount 不成功很有效!
    3. -v :可以列出每个文件与进程还有指令的完整相关性!
    4. -k :找出使用该文件/目录的 PID ,并试图以 SIGKILL 这个讯号给予该 PID;
    5. -i :必须与 -k 配合,在删除 PID 之前会先询问使用者意愿!
    6. -signal:例如 -1 -15 等等,若不加的话,预设是 SIGKILL (-9) 啰!

9. lsof : 列出被进程所开启的文件档名
    1. -a :多项数据需要『同时成立』才显示出结果时!
    2. -U :仅列出 Unix like 系统的 socket 文件类型;
    3. -u :后面接 username,列出该使用者相关进程所开启的文件;
    4. +d :后面接目录,亦即找出某个目录底下已经被开启的文件!

10. pidof : 找出某支正在执行的程序的 PID
    1. -s :仅列出一个 PID 而不列出所有的 PID
    2. -x :同时列出该 program name 可能的 PPID 那个进程的 PID

**SELinux 初探**: Security Enhanced Linux

略过



#### 第十七章、认识系统服务

系统为了某些功能必须要提供一些服务 (不论是系统本身还是网络方面),这个服务就称为 service 。 但是 service 的提供总是需要程序的运作吧!否则如何执行呢?所以达成这个 service的程序我们就称呼他为 daemon 啰!  守护进程。

**早期的init 的管理机制有几个特色如下:**

1. 服务的启动、关闭与观察等方式:
    1. o 启动:/etc/init.d/daemon start
    2. o 关闭:/etc/init.d/daemon stop
    3. o 重新启动:/etc/init.d/daemon restart
    4. o 状态观察:/etc/init.d/daemon status

2. 独立启动模式 (stand alone):服务独立启动,该服务直接常驻于内存中,提供本机或用户的服务行
为,反应速度快。
3. 总管程序 (super daemon):由特殊的 xinetd 或 inetd 这两个总管程序提供 socket 对应或 port 对应的管理。当没有用户要求某 socket 或 port 时, 所需要的服务是不会被启动的。若有用户要求时,xinetd 总管才会去唤醒相对应的服务程序。当该要求结束时,这个服务也会被结束掉~ 因为透过
xinetd 所总管,因此这个家伙就被称为 super daemon。好处是可以透过 super daemon 来进行服务的时程、联机需求等的控制,缺点是唤醒服务需要一点时间的延迟。
4. init 在管理员自己手动处理这些服务时,是没有办法协助相依服务的唤醒的!


**systemd 使用的 unit 分类**:

1. 平行处理所有服务,加速开机流程
2. 一经要求就响应的 on-demand 启动方式:systemd 全部就是仅有一只 systemd 服务搭配 systemctl 指令来处理,无须其他额外的指令来支持。
3. 服务相依性的自我检查
4. 依 daemon 功能分类,service, socket, target, path, snapshot,timer 等多种不同的类型(type), 方便管理员的分类与记忆。
5. 将多个 daemons 集合成为一个群组:systemd 亦将许多的功能集合成为一个所谓的 target 项目,这个项目主要在设计操作环境的建置, 所以是集合了许多的 daemons,亦即是执行某个 target 就是执行好多个 daemon 的意思.系统服务、数据监听与交换的插槽档服务 (socket)、储存系统状态的快照类型、提供不同类似执行等级分类的操作环境 (target) 等。
6.  /etc/systemd/system/ 配置目录
7.  几种常见的systemd服务：
    1.  .service
    2.  .socket
    3.  .target
    4.  .mount
    5.  .automount
    6.  .path
    7.  .timer
8. systemctl : 管理单一服务 (service unit) 的启动/开机启动与观察状态
    1. start :立刻启动后面接的 unit
    2. stop :立刻关闭后面接的 unit
    3. restart :立刻关闭后启动后面接的 unit,亦即执行 stop 再 start 的意思
    4. reload :不关闭后面接的 unit 的情况下,重载配置文件,让设定生效
    5. enable :设定下次开机时,后面接的 unit 会被启动
    6. disable :设定下次开机时,后面接的 unit 不会被启动
    7. status :目前后面接的这个 unit 的状态,会列出有没有正在执行、开机预设执行否、登录等信息等!
    8. is-active :目前有没有正在运作中
    9. is-enable :开机时有没有预设要启用这个 unit
9. 进程状态:
    1. active (running)
    2. active (exited)
    3. active (waiting)
    4. inactive
    5. enabled:这个开机自动启动
    6. disabled:
    7. static:daemon 不可以自己启动 (enable 不可),不过可能会被其他的 enabled 的服务来唤醒 (相依属性的服务)
    8.  mask:这个 daemon 无论如何都无法被启动!因为已经被强制注销
10. cat /etc/services 服务与端口号
11. netstat -tlunp 查看网络端口

很有意思的东西，现在不做深究。只是简单的了解。



#### 第十八章、认识与分析日志文件

这章现在完全没有了了解，略过。

#### 第十九章、开机流程、模块管理与 Loader

1. 加载 BIOS 的硬件信息与进行自我测试,并依据设定取得第一个可开机的装置;
2. 读取并执行第一个开机装置内 MBR 的 boot Loader (亦即是 grub2, spfdisk 等程序);
3. 依据 boot loader 的设定加载 Kernel ,Kernel 会开始侦测硬件与加载驱动程序;
4. 在硬件驱动成功后,Kernel 会主动呼叫 systemd 程序,并以 default.target 流程开机;
    1. systemd 执行 sysinit.target 初始化系统及 basic.target 准备操作系统;
    2. systemd 启动 multi-user.target 下的本机与服务器服务;
    3. systemd 执行 multi-user.target 下的 /etc/rc.d/rc.local 文件;
    4. systemd 执行 multi-user.target 下的 getty.target 及登入服务;
    5. systemd 执行 graphical 需要的服务


开机管理程序就被称为 Boot Loader 了。这个 Boot Loader 程序安装在哪里呢?就在开机装置的第一个扇区 (sector) 内,也就是我们一直谈到的 MBR (Master Boot Record, 主要启动记录区)。其实 BIOS 是透过硬件的INT 13 中断功能来读取 MBR.

其实每个文件系统 (filesystem,或者是 partition) 都会保留一块启动扇区 (boot sector) 提供操作系统安装 boot loader , 而通常操作系统默认都会安装一份 loader 到他根目录所在的文件系统的 boot sector 上。而在 Linux 系统安装时,你可以选择将 boot loader 安装到 MBR 去,也可以选择不安装。 如果选择安装到 MBR 的话,那理论上你在 MBR 与 boot sector 都会保有一份boot loader 程序的。 至于 Windows 安装时,他预设会主动的将 MBR 与 boot sector 都装上一份boot loader!

1. 选单一:MBR(grub2) --> kernel file --> booting
2. 选单二:MBR(grub2) --> boot sector(Windows loader) --> Windows kernel --> booting
3. 选单三:MBR(grub2) --> boot sector(grub2) --> kernel file --> booting

#### 第二十章、基础系统设定与备份策略

#### 第二十一章、软件安装:原始码与 Tarball

**makefile 的基本语法与变量**:

1. 变量与变量内容以『=』隔开,同时两边可以具有空格;
2. 变量左边不可以有 <tab> ,例如上面范例的第一行 LIBS 左边不可以是 <tab>;
3. 变量与变量内容在『=』两边不能具有『:』;
4. 在习惯上,变数最好是以『大写字母』为主;
5. 运用变量时,以 ${变量} 或 $(变量) 使用;
6. 在该 shell 的环境变量是可以被套用的,例如提到的 CFLAGS 这个变数!
7. 在指令列模式也可以给予变量。

#### 第二十二章、软件安装 RPM, SRPM 与 YUM

如何将动态函式库加载高速缓存当中呢?

如果我们将常用到的动态函式库先加载内存当中 (快取, cache),如此一来,当软件要取用动态函式库时,就不需要从头由硬盘里面读出啰! 这样不就可以增进动态函式库的读取速度?没错,是这样的!这个时候就需要 ldconfig 与 /etc/ld.so.conf 的协助了。

1. 首先,我们必须要在 /etc/ld.so.conf 里面写下『 想要读入高速缓存当中的动态函式库所在的目录』,注意喔, 是目录而不是文件;
2. 接下来则是利用 ldconfig 这个执行档将 /etc/ld.so.conf 的资料读入快取当中;

#### 网络与打印机，硬件检测

1. 系统
    1. uname -a # 查看内核/操作系统/CPU信息
    2. head -n 1 /etc/issue # 查看操作系统版本
    3. cat /proc/cpuinfo # 查看CPU信息
    4. hostname # 查看计算机名
    5. lspci -tv # 列出所有PCI设备
    6. lsusb -tv # 列出所有USB设备
    7. lsmod # 列出加载的内核模块
    8. env # 查看环境变量
2. 资源
    1. free -m # 查看内存使用量和交换区使用量
    2. df -h # 查看各分区使用情况
    3. du -sh <目录名> # 查看指定目录的大小
    4. grep MemTotal /proc/meminfo # 查看内存总量
    5. grep MemFree /proc/meminfo # 查看空闲内存量
    6. uptime # 查看系统运行时间、用户数、负载
    7. cat /proc/loadavg # 查看系统负载
3. 磁盘和分区
    1. mount | column -t # 查看挂接的分区状态
    2. fdisk -l # 查看所有分区
    3. swapon -s # 查看所有交换分区
    4. hdparm -i /dev/hda # 查看磁盘参数（仅适用于IDE设备）
    5. dmesg | grep IDE # 查看启动时IDE设备检测状况
4. 网络
    1. ifconfig # 查看所有网络接口的属性
    2. iptables -L # 查看防火墙设置
    3. route -n # 查看路由表
    4. netstat -lntp # 查看所有监听端口
    5. netstat -antp # 查看所有已经建立的连接
    6.  netstat -s # 查看网络统计信息
5. 进程
    1. ps -ef # 查看所有进程
    2. top # 实时显示进程状态
6. 用户
    1. w # 查看活动用户
    2. id <用户名> # 查看指定用户信息
    3. last # 查看用户登录日志
    4. cut -d: -f1 /etc/passwd # 查看系统所有用户
    5. cut -d: -f1 /etc/group # 查看系统所有组
    6. crontab -l # 查看当前用户的计划任务
7. 服务
    1. chkconfig --list # 列出所有系统服务
    2. chkconfig --list | grep on # 列出所有启动的系统服务
8. 程序
    1. rpm -qa # 查看所有安装的软件包
9. 其他常用命令整理如下：
    1. 查看主板的序列号：dmidecode | grep -i 'serial number'
    2. 用硬件检测程序kuduz探测新硬件：service kudzu start （ or restart）
    3.  查看CPU信息：cat /proc/cpuinfo [dmesg | grep -i 'cpu'][dmidecode -t processor]
    4.  查看内存信息：cat /proc/meminfo [free -m][vmstat]
    5.  查看板卡信息：cat /proc/pci
    6.  查看显卡/声卡信息：lspci |grep -i 'VGA'[dmesg | grep -i 'VGA']
    7.  查看网卡信息：dmesg | grep -i 'eth'[cat /etc/sysconfig/hwconf | grep -i eth][lspci | grep -i 'eth']
    8.  查看PCI信息：lspci （相比cat /proc/pci更直观）
    9.  查看USB设备：cat /proc/bus/usb/devices
    10. 查看键盘和鼠标：cat /proc/bus/input/devices
    11. 查看系统硬盘信息和使用情况：fdisk & disk – l & df
    12. 查看各设备的中断请求（IRQ）：cat /proc/interrupts
    13. 查看系统体系结构：uname -a
    14. 查看及启动系统的32位或64位内核模式：isalist –v [isainfo –v][isainfo –b]
    15. 查看硬件信息，包括bios、cpu、内存等信息：dmidecode
    16. 测定当前的显示器刷新频率：/usr/sbin/ffbconfig –rev \?
    17. 查看系统配置：/usr/platform/sun4u/sbin/prtdiag –v
    18. 查看当前系统中已经应用的补丁：showrev –p
    19. 显示当前的运行级别：who –rH
    20. 查看当前的bind版本信息：nslookup –class=chaos –q=txt version.bind
    21. 查看硬件信息：dmesg | more
    22. 显示外设信息， 如usb，网卡等信息：lspci
    23. 查看当前处理器的类型和速度（主频）：psrinfo -v
    24. 打印当前的OBP版本号：prtconf -v
    25. 查看硬盘物理信息（vendor， RPM， Capacity）：iostat –E
    26. 查看磁盘的几何参数和分区信息：prtvtoc /dev/rdsk/c0t0d0s 
    27. 显示已经使用和未使用的i-node数目：isalist –v
10. 对于“/proc”中文件可使用文件查看命令浏览其内容，文件中包含系统特定信息：
    1. 主机CPU信息：Cpuinfo 
    2. 主机DMA通道信息：Dma 
    3. 文件系统信息：Filesystems 
    4. 主机中断信息：Interrupts 
    5. 主机I/O端口号信息：Ioprots 
    6. 主机内存信息：Meninfo 
    7. Linux内存版本信息：Version