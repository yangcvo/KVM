## KVM虚拟机更改存储路径解决方法

玩kvm的2014年开始慢慢接触了，发现功能特别大，比vm好用，vm是基于Windows上面需要安装客户端登陆很麻烦,也有人说也有web管理端其实我觉得运维在kvm伸缩现在很火的openstack也是底层kvm去生成虚拟机的。



kvm 在之前的搭建环境 一开始安装指定的虚拟机镜像没有放到挂载单独的一块硬盘路径。后果会不堪设想。


kvm服务器：

    IP：192.168.1.250
    镜像存储路径：/opt/vm/
    默认的xml文件路径：/etc/libvirt/qemu
 
发现MySQL的服务器已经`挂了`。=_=

 
登陆到kvm：
 
一开始我MySQL跟MySQL主从存储路径都在`/opt/vm/mysql.img` 、`/opt/vm/mysql-2.img`

这里我看了下这两台服务器都已经挂了，因为kvm空间被根目录被吃光了，一开始就没有注意这个配置，后面我在 创建/home/vm/，分了块硬盘空间挂载到这个目录下面。

### 先不要：

把/opt/vm/mysql.img  直接mv到了/home/vm/.

### 这里第一步：

需要先停掉虚拟机。如果虚拟机已经挂了，在kvm用virsh查看下。

```
 -     mysql                          暂停
 -     mysql-2                        暂停
```

如果暂停说明已经卡死了。那么shutdown 也是无效的。

这里可以查看我写的篇：[kvm-hellp帮助篇](http://blog.yangcvo.me/2015/06/18/KVM%E8%99%9A%E6%8B%9F%E5%8C%96%E5%AE%9E%E7%94%A8%E7%AF%87-%E7%AC%AC%E4%BA%8C%E7%AF%87-help%E5%B8%AE%E5%8A%A9%E6%96%87%E6%A1%A3%E5%A4%A7%E5%85%A8/)-只对kvm入门人员帮助

    virsh undefine $1   删除虚拟机取消定义   删除虚拟机先执行这个 取消定义。
 

强制关闭虚拟机：
    
    virsh undefine mysql 
    virsh undefine mysql-2
    
### 这里第二步移动镜像存储目录：

然后： 把/opt/vm/mysql.img  直接mv到了/home/vm/.
移动指定的目录。

### 第三步需要修改xml文件

这里一开始遇到很多头大的问题：

* 第一我直接vim /etc/libvirt/qemu/mysql.xml
* 第一我直接vim /etc/libvirt/qemu/mysql-2.xml

这样修改了以后是不能生效的。而且一错在错，因为如果改完了以后直接去启动MySQL

    virsh start mysql-2
提示：`/opt/vm/ 下没有 mysql-2.img`

![](http://7xrthw.com1.z0.glb.clouddn.com/KVM3.png)
    
这里我知道是没有生效，就取消了定义mysql-2 取消定义需要删除快照才能取消定义。

这里又做错了一步：

### 如何取消定义虚拟机

![](http://7xrthw.com1.z0.glb.clouddn.com/KVM2.png)

我把自己辛辛苦苦的快照都删除了，然后取消了定义。

### 注意：

发现取消了定义以后，我的/etc/libvirt/qemu/mysql-2.xml 已经被删除了，取消定义是会删除xml文件的。所以这个时候发现自己大错特错，之前忘记了一个重要的操作。
这里我发现我MySQL已经被删除了。可是镜像文件还是在/home/vm/mysql-2.img
还好我做了主从，删除了只是其中一台。这个时候求助了之前搞虚拟化的朋友--`甜橙大神`。


### 紧急修复

这里我在他的指导帮助下搞定了。

### 修复第一步：

我这里先拷贝了一份其他虚拟机的xml文件,做相应修改.

    cp nginx.xml mysql-2.xml

然后打开文件：

name uuid img 之类的都要修改

* 先删除:uuid- uuid 可以不填-删除这行就行了

```
<uuid>876f7ff7-a0e7-f64d-fc3c-bc48fee28635</uuid>
```
* 改成自己恢复的镜像。

```
 <source file='/home/vm/mysql-2.img'/>
```

* 修改网卡信息：

```
  <mac address='52:54:00:e9:cb:c4'/>
```
这里的是拷贝过来的，所以你稍微做下修改，我这里把F1改成了c4.

* 修改VNC链接的端口

```
<graphics type='vnc' port='5950' autoport='no' listen='0.0.0.0'>
      <listen type='address' address='0.0.0.0'/>
```
这里是我拷贝过来的，所以5950我改成了我之前MySQL的5941的。

改好后保存退出。。

### 启动虚拟机-加载xml文件。

重新定义xml文件

```
virsh # define /etc/libvirt/qemu/mysql-2.xml
定义域 mysql-2（从 /etc/libvirt/qemu/mysql-2.xml）
virsh # start mysql
域 mysql 已开始
```

这里显示已经启动成功了。所以取消定义是最危险的。因为没有备份xml文件。


### VNC连接修改网卡信息。

这里需要用VNC连接到这台服务器，然后修改网卡信息。

这里查看网卡信息，没有IP，在查看if-eth0
![](http://7xrthw.com1.z0.glb.clouddn.com/KVM4.png)

需要修改mac地址：

![](http://7xrthw.com1.z0.glb.clouddn.com/kvm5.png)

改完保存退出。这个时候重启网卡。会发现报错。

![](http://7xrthw.com1.z0.glb.clouddn.com/KVM6.png)

centos6.5，查看 `/etc/udev/rules.d/70-net* `文件，确保里面`eth0` 的mac 跟xml里 的一样

![](http://7xrthw.com1.z0.glb.clouddn.com/KVM7.png)

这里修改eth0 的mac 到xml里的，删了eth1
重启就可以了。

![](http://7xrthw.com1.z0.glb.clouddn.com/KVM8.png)

现在已经恢复回来了。



### 切记

<div class="tip">
xml文件修修改改就行了，还是不要随便删了 。
</div>

### 这里的最简单解决方法：

***
修改xml 要 virsh edit，不是vi ，vi修改不生效的，所以才会报错找不到之前的镜像
直接关机虚拟机，virsh edit修改就不会出现找不到镜像问题了。

总结了经验。以后存储方法-定义和取消定义的危险性，-正确修改配置方法。

