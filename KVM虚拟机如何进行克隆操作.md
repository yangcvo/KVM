##KVM虚拟机如何进行克隆操作

### KVM虚拟机如何进行克隆操作


我们知道，重复的事情做多了就需要上脚本或采用工具以便节省操作时间，提升工作效率，减低繁琐操作。本人也是懒人一个，相同的事情做多了，就想法子弄成脚本或采用工具去搞。KVM虚拟机的创建同样如此，我们不可能每次创建都要手工执行一次，于是就有了KVM克隆出现。克隆操作有两种方式：

**(1) KVM本机虚拟机直接克隆。**
**(2) 通过复制配置文件与磁盘文件的虚拟机复制克隆(适用于异机的静态迁移)。**


##### 一. KVM本机虚拟机直接克隆。

1. 克隆主机。以我们之前创建的samba为例,克隆完毕以后直接启动该主机并进行配置。


```bash
[root@kvm vm]# virsh list --all
 Id    名称                         状态
----------------------------------------------------
 -     namenode                       关闭
 -     samba                          关闭
 -     tomcat_F1                      关闭
 
[root@21yunwei autostart]# virt-clone --original samba --name tomcat_F1  --file /opt/vm/tomcat_F1.img 
Cloning tomcat_F1.img                                                                                                            | 25.0 GB     01:39     
 
Clone 'tomcat_F1' created successfully.
[root@kvm vm]# virsh  list --all
 Id    Name                           State
----------------------------------------------------
 -     namenode                       关闭
 -     samba                          关闭
 -     tomcat_F1                      关闭
```

克隆完成以后服务是没有启动的，这会我们把服务启动：

```bash
[root@kvm qemu]# virsh start tomcat_F1
域 tomcat_F1 已开始
```

注意：
（1）上述命令可以简化成

     virt-clone -o  samba -n  tomcat_F1 -f/opt/vm/tomcat_F1.img 


**（2）操作之前，请先将拷贝的对象机关闭或暂停，否则会报错。
（3）建议将之前做好的第一个主机设置成模板，这样以后每次创建主机以后用vnc进去修改网络参数配置就行了。**

这里提示下 这里克隆的时候默认是随机VNC端口的，所有这里需要你克隆完成以后自己设置下端口，不然不知道端口是多少。


vim /etc/libvirt/qemu/tomcat_F1.xml

```bash
    <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='vnc' port='5921' autoport='no' listen='0.0.0.0'>
      <listen type='address' address='0.0.0.0'/>   ##vnc prot=远程端口 autoport='no' 设置no
    </graphics>
    <video>
      <model type='cirrus' vram='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </memballoon>
  </devices>
</domain>
```
![](media/14888992336268.jpg)

2. vnc登录进去修改网卡参数和配置

（1）`ifconfig -a` 查看网卡情况。注意mac地址等影响网卡因素。可以编辑网卡改成`ifconfig -a`网卡信息或/etc/udev/rule.d/70-persistent-net.rules里边的信息与要设置的网卡匹配就可以了。
（2）或者删除/etc/udev/rule.d/70-persistent-net.rules文件，这个重启系统会自动生成的。 
修改下IP地址：vim /etc/sysconfig/network-scripts/ifctg-eth0修改自己要的IP，完以后tomcat_F1 也可以上网了。如下图所示：


![](media/14889006599911.jpg)


### 二 通过复制配置文件与磁盘文件的虚拟机复制克隆(适用于异机的静态迁移)

这个操作对大多数用户用不到，平时我们的测试环境也用不到这么专业。有兴趣的朋友可以自行参考KVM静态迁移 ，这里不做操作展示了。

本文KVM克隆操作得以顺利实现，主要操作参考了如下资料，这里做下资料参考以便后边的朋友可以自行查看。
阿铭KVM克隆：https://www.apelearn.com/bbs/thread-8299-1-1.html
koumm的BLOG：http://koumm.blog.51cto.com/703525/1291793


