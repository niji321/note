title: linux 自动挂载硬盘
author: xg wang
tags: []
categories:
- linux

date: 2018-07-09 11:45:00
---

固态硬盘太小了，随便放放数据就满了。本着不浪费的原则，就没有新买一块SSD。所以在机械硬盘上分了500G出来，格式化为 ext4文件系统。自动挂载到linux的目录上，作为数据盘。

配置开机自动挂载

~~~shell
sudo vim /etc/fstab**
~~~

~~~python
<fs spec> <fs file> <fs vfstype> <fs mntops> <fs freq> <fs passno>
具体说明，以挂载/dev/sdb1为例:
<fs spec> :
分区定位，可以给UUID或LABEL，例如：UUID=6E9ADAC29ADA85CD或LABEL=software
<fs file> : 具体挂载点的位置，例如：/data
<fs vfstype> : 挂载磁盘类型，linux分区一般为ext4，windows分区一般为ntfs
<fs mntops> : 挂载参数，一般为defaults
<fs freq> : 磁盘检查，默认为0
<fs passno> : 磁盘检查，默认为0,不需要检查

UUID=bf164ef7-50f9-4b33-ac45-1d80ebd81eef /home/xgwang/data ext4 errors=remount-ro 0 1
~~~

修改完/etc/fstab文件后，运行sudo mount -a命令验证一下配置是否正确，配置不正确会导致系统无法启动。