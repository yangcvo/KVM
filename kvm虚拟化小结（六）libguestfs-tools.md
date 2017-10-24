## kvm虚拟化小结（六）libguestfs-tools


`libguestfs` 是一组 Linux 下的 C 语言的 API ，用来访问虚拟机的磁盘映像文件。其项目主页是http://libguestfs.org/ ，该工具包内包含的工具有

```bash
virt-cat、
virt-df、
virt-ls、
virt-copy-in、
virt-copy-out、
virt-edit、guestfs、
guestmount、
virt-list-filesystems、
virt-list-partitions
```
等工具，具体用法也可以参看官网。该工具可以在不启动`KVM guest`主机的情况下，直接查看`guest`主机内的文内容，也可以直接向img镜像中写入文件和复制文件到外面的物理机，当然其也可以像mount一样，支持挂载操作。


#### 一、libguestfs-tools的安装

由于在rpm源里直接有该包，所以可以直接通过yum进行安装：

```bash
#yum -y install libguestfs-tools
```

#### 二、linux下的使用

```bash
[root@kvm vm]# virt-df vpn.img
文件系统                         1K-blocks 已用空间 可用空间 使用百分比%
vpn.img:/dev/sda1                       487652     127677     330279   27%
vpn.img:/dev/sda2                     22043164    1614892   19285488    8%

[root@kvm vm]# virt-ls  vpn.img /
.autofsck
.pki
bin
boot
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
sbin
selinux
srv
sys
tmp
usr
var
```



查看：http://www.361way.com/kvm-libguestfs-tools/3175.html


