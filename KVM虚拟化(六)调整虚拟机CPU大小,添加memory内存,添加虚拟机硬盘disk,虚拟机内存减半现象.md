## KVM虚拟化(六)调整虚拟机CPU大小,添加memory内存,添加虚拟机硬盘disk,虚拟机内存减半现象


## 调整虚拟机CPU大小

* 这个是我需要修改的虚拟机dataname1
* 操作这些都需要shutdown off

```bash
virsh # shutdown dataname1 
```

* 查看你要修改的虚拟机的配置信息。 

``` bash
[root@kvm ~]# virsh
欢迎使用 virsh，虚拟化的交互式终端。

输入：'help' 来获得命令的帮助信息
       'quit' 退出

virsh # dominfo dataname1     
Id:             29
名称：       dataname1
UUID:           0301d59b-5b19-40af-b0f9-063d646a38db
OS 类型：    hvm
状态：       running
CPU：          1     
CPU 时间：   14.2s
最大内存： 2097152 KiB
使用的内存： 2097152 KiB
持久：       是
自动启动： 禁用
管理的保存： 否
安全性模式： selinux
安全性 DOI： 0
安全性标签： system_u:system_r:svirt_t:s0:c903,c922 (enforcing)
```

* 这里很明显查看下CPU这里是1核 ，2G内存。
* 修改dataname1的cpu

这里需要编辑了域 dataname1 XML 配置。

``` bash
virsh # edit dataname1
```

修改下CPU

```bash
<vcpu placement='static'>2</vcpu>
```
大概在第六行这里改成2 就是CPU2核 ，如果需要改成4核直接改成4

然后wq!保存并退出。

提示：编辑了域 dataname1	 XML 配置。

然后在执行

``` bash
virsh # setvcpus dataname1 2 --config
```
然后查看已添加CPU核数。

```bash
virsh # dominfo dataname1
Id:             -
名称：       dataname1
UUID:           0301d59b-5b19-40af-b0f9-063d646a38db
OS 类型：    hvm
状态：       关闭
CPU：          2
最大内存： 2097152 KiB
使用的内存： 0 KiB
持久：       是
自动启动： 禁用
管理的保存： 否
安全性模式： selinux
安全性 DOI： 0
```

### 调整虚拟机内存大小

* 我们查看下dataname1 配置文件

```bash
<memory unit='KiB'>2097152</memory>
<currentMemory unit='KiB'>2097152</currentMemory>
```

可以看到，虚拟机中实际显示的为currentMemory（minRam），即为当前内存为2G。
memory unit实际为最大使用内存（maxRam）。

修改4G 2097152x2=4194304

所以我们这里的最大内存是2G,修改配置文件。

``` bash
virsh # edit dataname1
  <memory unit='KiB'>4194304</memory>
  <currentMemory unit='KiB'>4194304</currentMemory>
```
然后保存退出。

设置生效：

```bash

virsh setmaxmem dataname1 4194304 --config   

#dataname1 处于shut off状态时才能设置成功

virsh setmem dataname1 4

#数值不能超过maxmem    setmem意思是改变内存的分配

查看生效内存现在变4G内存。
virsh # dominfo dataname1
Id:             -
名称：       dataname1
UUID:           0301d59b-5b19-40af-b0f9-063d646a38db
OS 类型：    hvm
状态：       关闭
CPU：          2
最大内存： 4194304 KiB
使用的内存： 0 KiB
持久：       是
自动启动： 禁用
管理的保存： 否
安全性模式： selinux
安全性 DOI： 0
```

这里提下还有些时候KVM虚拟机内存超配后-虚拟机内存减半现象
使用kvm虚拟机时，如果配置了内存超用，会发现创建的虚拟机内存为计算方案的一半。

* 分析：

* 配置完超配系数为2以后，创建虚拟机，打开虚拟机（计算方案为2C/2G）的xml配置文件如下：

```bash
<name>i-2-32-VM</name>
<uuid>eb1a307f-ff54-4f40-aa88-d6071535cd92</uuid>
<description>dataname1</description>
<memory unit='KiB'>2097152</memory>
<currentMemory unit='KiB'>1048576</currentMemory>
```
可以看到，虚拟机中实际显示的为currentMemory（minRam），即为当前内存为1G。
但memory unit实际为最大使用内存（maxRam）。
可以看到，2G实际被定义为虚拟机的maxRam，但实际分配为minRam，即看到的减半现象。

* 解决：

1.编辑agent配置文件，添加参数“vm.memballoon.disable=true”

```bash 
[root@SJCloudKVM-5 agent]# cat /etc/cloudstack/agent/agent.properties| grep mem
vm.memballoon.disable=true
```
2.重启libvirtd和cloudstack-agent服务。
3.关闭，并重新启动虚拟机。

关于vm.memballoon.disable=true的解释：

``` bash
# vm.memballoon.disable=true
# Disable memory ballooning on vm guestsfor overcommit, by default overcommit feature enables balloon and setscurr
entMemoryto a minimum value.
```
### 调整虚拟机硬盘大小

* 有两种方法添加：
  * 第一种：是要通过修改配置文件来添加硬盘，我们首先要关闭虚拟机，否则无法正常添加。通过修改虚拟机配置文件进行添加，永久生效。
  * 第二种：通过virsh attach-disk命令添加一块硬盘到系统中，即时生效，但系统重启后新硬盘会消失。
  * 第二种方法简单：现在开始使用virsh attach-disk命令把新硬盘添加到虚拟机上。
virsh attach-disk 虚拟机名称 /vhost/新建的硬盘.img vdb 


那我们都选择第一种方法：
  
* 关闭虚拟机，然后使用virsh edit命令修改虚拟机的主配置文件。

 ``` bash
 virsh # shutdown dataname2
 
然后给他新建个硬盘:
 
  qemu-img create -f qcow2 /opt/vm/dataname2-add.img 150G

虚拟机的所有配置文件默认都存放在/etc/libvirt/qemu，如下:


-rw-------. 1 root root 3868 7月   9 18:49 dataname2.xml

```

编辑虚拟机配置文件，如下:
默认在第32行

默认配置前：

```bash
[root@kvm qemu]# virsh edit dataname2
 <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/opt/vm/dataname2.img'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </disk>
    <disk type='block' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <target dev='hda' bus='ide'/>
      <readonly/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
```
配置后修改，我们找到有关硬盘的代码：

``` bash
<disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/opt/vm/dataname2.img'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </disk>
现在我们在</disk>这之后，添加如下的代码
  <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/opt/vm/dataname2-add.img'/>
      <target dev='vdb' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
 </disk>
保存退出。
编辑了域 dataname2 XML 配置。
```

注意其中type表示硬盘的格式

file表示硬盘所在的路径

dev表示硬盘在系统中显示的硬盘名称

bus表示硬盘的接线类型，如果是windows系统一般是ide。

添加完毕后，我们来启动虚拟机看看实际的效果。

``` bash
[root@datanode2 ~]# fdisk -l

Disk /dev/vda: 107.4 GB, 107374182400 bytes
16 heads, 63 sectors/track, 208050 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000e0dd6

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *           3        1018      512000   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/vda2            1018      208051   104344576   8e  Linux LVM
Partition 2 does not end on cylinder boundary.

Disk /dev/vdb: 161.1 GB, 161061273600 bytes
16 heads, 63 sectors/track, 312076 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

```
然后就是格式化挂载上去就可以了。


