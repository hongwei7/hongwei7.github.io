# 制作自己的 LFS 项目（操作系统课设）


Linux From Scratch (LFS) is a project that provides you with step-by-step instructions for building your own customized Linux system entirely from source.

### 准备工作

环境:`parallels`虚拟机

下载LFS-LiveCD的ISO文件(6.3版本)

用虚拟机打开,选择为Linux Kernel 2.6.

`passwd`更改密码

`/etc/rc.d/init.d/sshd start` 启动ssh服务

### 分区

使用`cfdisk`创建两个分区

```bash
mkswap /dev/sda1
mke2fs -jv /dev/sda2
```

sda1作为交换分区,sda2作为ext3格式分区.

### 工作环境设置

```bash
export LFS=/mnt/lfs
mkdir -pv $LFS
mount /dev/sda2 $LFS
swapon /dev/sda1
ln -sv $LFS/tools /
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
passwd lfs


chown -v lfs $LFS/tools
chown -v lfs $LFS/sources 
su - lfs
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash 
EOF
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/tools/bin:/bin:/usr/bin
export LFS LC_ALL PATH
EOF
source ~/.bash_profile
```

新建了一个lfs用户,并赋予其编译工具链的目录和权限,并设置一些环境变量.

### 工具链编译

`cd $LFS/sources`

里面是工具链的软件包,大约需要安装26个.

逐个编译起来

- **Binutils**

- **GCC-4.1.2**

- **Linux-2.6.22.1 API Headers**

- **Glibc** (The Glibc package contains the main C library. This library provides the basic routines for allocating memory, searching directories, opening and closing files, reading and writing files, string handling, pattern matching, arithmetic, and so on.)]

### 链接编译工具链到编译过的GCC

​	过程略.继续编译其他包

- **Tcl**(This package and the next two (Expect and DejaGNU) are installed to support running the test suites for GCC and Binutils. Installing three packages for testing purposes may seem excessive, but it is very reassuring, if not essential, to know that the most important tools are working properly. Even if the test suites are not run in this chapter (they are not mandatory), these packages are required to run the test suites in Chapter 6.)
- **Expect**
- **DejaGNU**(The DejaGNU package contains a framework for testing other programs.)
- **Ncurses-5.6**
- **Bash-3.2**
- **Bzip2-1.0.4**
- **Coreutils-6.9**(The Coreutils package contains utilities for showing and setting the basic system characteristics.)
- **Diffutils-2.8.1**(The Diffutils package contains programs that show the differences between files or directories.)
- **Findutils-4.2.31**(The Findutils package contains programs to find files. These programs are provided to recursively search through a directory tree and to create, maintain, and search a database (often faster than the recursive find, but unreliable if the database has not been recently updated).)
- **Gawk-3.1.5**
- **Gettext-0.16.1**(The Gettext package contains utilities for internationalization and localization. These allow programs to be compiled with NLS (Native Language Support), enabling them to output messages in the user's native language.)
- **grep**
- **Gzip-1.3.12**
- **Make-3.81**
- **Patch-2.5.4**
- **Perl-5.8.8**
- **Sed-4.1.5**
- **Tar-1.18**
- **Texinfo-4.9**(The Texinfo package contains programs for reading, writing, and converting info pages.)
- **Util-linux-2.12r**(The Util-linux package contains miscellaneous utility programs. Among them are utilities for handling file systems, consoles, partitions, and messages.)

### 内核编译安装

`chroot`到新系统下的root,编写`fstab`和编译安装内核

**内核选项配置(sda,parallel)**

因为我是用的是 parallel 虚拟机，需要配置特殊的硬盘配置。

```bash
make menuconfig
```

> Device Drivers
>
> |---> SCSI device support
>
> ​		|---> <\*> SCSI device support
>
> ​		|---> <\*> SCSI disk support
>
> |---> <*> Serial ATA (prod) and Parallel ATA (experimental) drivers
>
> ​		|---> <\*> AHCI SATA support
>
> ​		|---> <\*> Intel ESB, ICH, PIIX3, PIIX4 PATA/SATA support (NEW)
>
> ​		|---> <\*> Intel PATA old PIIX support (NEW)
>
> File systems
>
> |---> <\*> Ext3 journalling file system support
>
> |---> [\*]  Ext3 extended attributes
>
> |---> [\*]   Ext3 POSIX Access Control Lists

### 设置Grub引导(sda2分区块,parallels环境)

```bash
cat > /etc/fstab << "EOF"
# Begin /etc/fstab
# file system mount-point type options dump fsck
#                                           order
/dev/sda2 / ext2 defaults 1 1
/dev/sda1 swap swap pri=1 0 0
proc /proc proc defaults 0 0
sysfs /sys sysfs defaults 0 0
devpts /dev/pts devpts gid=4,mode=620 0 0
shm /dev/shm tmpfs defaults 0 0
# End /etc/fstab
EOF

cat > /boot/grub/menu.lst << "EOF"
# Begin /boot/grub/menu.lst
# By default boot the first menu entry.
default 0
# Allow 30 seconds before booting the default.
timeout 5
# Use prettier colors.
color green/black light-green/black
# The first entry is for LFS.
title LFS 6.3
root (hd0,1)
kernel /boot/lfskernel-2.6.22.5 root=/dev/sda2
EOF

grub

root (hd0,1) 
setup (hd0) 
quit
```

重启顺利进入LFS系统.

### 安装openssh

返回livecd下载编译安装`wget`,接着升级`perl`到最新版本,安装`openssl`和`openssh`.

至此系统环境顺利搭建.

### 安装Xwindow,运行Qt软件

此处参考了https://blog.csdn.net/u012333520/article/details/50563462

最后成功编译`qt-4.1.0`.虽然是比较旧的版本,但是作为课设的编译工具也够用了. 最后利用 qt 完成了一个进程管理器和一个计算器。
