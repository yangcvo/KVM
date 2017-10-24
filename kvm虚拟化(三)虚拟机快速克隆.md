##  KVM虚拟机更改存储路径解决方法

# kvm虚拟化(三)虚拟机快速克隆

第一步：

在复制之前需要的操作：


有一个DHCP服务用于IP地址自动获取，获取后期VNC修改静态IP地址
开启VNC远程桌面
设置网卡为自动获取IP地址
安装必要的补丁程序
执行setupmgr和Sysprep（关机）
复制关机后的VM镜像模板文件


举例：

我的kvm镜像都是指定在同一个数据挂载目录下面：

```
 /opt/vm/
 
 #cp -av tomcat2.img tomcat3.img
镜像拷贝完成以后

复制模板配置文件为tomcat3.xml

 cd /etc/libvirt/qemu
 cp tomcat2.xml tomcat3.xml
 
 修改模板配置文件
 
 编辑 vim tomcat3.xml
 
1. 修改虚拟机的名称，如：<name>tomcat3</name>
2. 修改uuid编号 ，这个是拷贝过来的tomcat2的uuid 修改几个数字，跟uuid不冲突就行。 如：  <uuid>0712eb79-5f7b-42d0-bb95-859e3f37f972</uuid>
3. 修改镜像路径名称： <source file='/opt/vm/tomcat3.img'/> 
4. 修改mac地址，如：<mac address='52:54:00:11:12:1f'/>
5. 修改VNC 端口：这里我写5947 <graphics type='vnc' port='5947' autoport='no' listen='0.0.0.0'>
```
其他的就不需要修改了。

对于网络部分的设置如果模板的桥接已经与物理的桥接设置相同时，无需进行修改

```
#virsh list --all    //可以看到新添加的VM已经添加了
#virsh  define tomcat3.xml  定义tomcat3
#virsh start tomcat3 启动tomcat3
如果报错了。会有提示的。 然后编辑tomcat3.xml
需要重新定义。先取消定义
#virsh undefine tomcat3  取消定义。
```


然后使用VNC连接。 登陆的时候root密码当然是tomcat2的密码了。后期可以更改。
这里需要修改下静态IP。

修改了IP启动网卡。

克隆虚拟机引起的“Device eth0 does not seem to be present, delaying initialization”

找到/etc/udev/rules.d/70-persistent-net.rules 删除后重启机器，系统会自动生成一个70-persistent-net.rules文件。

