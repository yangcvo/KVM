## KVM虚拟化 (二) help帮助文档大全


## 基本语句用法：

``` bash 
    virsh start x       启动名字为x的非活动虚拟机
    virsh create x.xml    创建虚拟机（创建后，虚拟机立     即执行，成为活动主机）
    virsh suspend x          暂停虚拟机
    virsh resume x           启动暂停的虚拟机
    virsh shutdown x         正常关闭虚拟机
    virsh destroy x          强制关闭虚拟机
    virsh undefine $         删除虚拟机取消定义   删除虚拟机先执行这个 取消定义。
    virus define  x1         虚拟机定义x1
     virsh dominfo x         显示虚拟机的基本信息
     virsh domname 2         显示id号为2的虚拟机名
     virsh domid x           显示虚拟机id号
     virsh domuuid x         显示虚拟机的uuid
    virsh domstate x         显示虚拟机的当前状态
    virsh dumpxml x          显示虚拟机的当前配置文件（可能和定义虚拟机时的配置不同，因为当虚拟机启动时，需要给虚拟机分配id号、uuid、   vnc端口号等等）
    virsh setmem x 512000    给不活动虚拟机设置内存大小
    virsh edit x             编辑配置文件（一般是在刚定义完虚拟机之后）
```
### KVM secret语句用法：

``` bash 
    virsh #   snapshot-list centos7  查看centos7快照
    Secret (help keyword 'secret'):
    secret-define                  定义或者修改 XML 中的 secret
    secret-dumpxml                 XML 中的 secret 属性
    secret-get-value               secret 值输出
    secret-list                    列出 secret
    secret-set-value               设定 secret 值
    secret-undefine                取消定义 secret
```
### KVM Snapshot 语句用法：

``` bash 
    Snapshot (help keyword 'snapshot'):
    snapshot-create                使用 XML 生成快照
    snapshot-create-as             使用一组参数生成快照
    snapshot-current               获取或者设定当前快照
    snapshot-delete                删除域快照
    snapshot-dumpxml               为域快照转储 XML
    snapshot-edit                  编辑快照 XML
    snapshot-info                  快照信息
    snapshot-list                  为域列出快照
    snapshot-parent                获取快照的上级快照名称
    snapshot-revert                将域转换为快照

```
### KVM  Storage Pool语句用法：

``` bash 
    Storage Pool (help keyword 'pool'):
    find-storage-pool-sources-as   找到潜在存储池源
    find-storage-pool-sources      发现潜在存储池源
    pool-autostart                 自动启动某个池
    pool-build                     建立池
    pool-create-as                 从一组变量中创建一个池
    pool-create                    从一个 XML 文件中创建一个池
    pool-define-as                 在一组变量中定义池
    pool-define                    define an inactive persistent storage pool or modify an existing persistent one from an XML file
    pool-delete                    删除池
    pool-destroy                   销毁（删除）池
    pool-dumpxml                   XML 中的池信息
    pool-edit                      为存储池编辑 XML 配置
    pool-info                      存储池信息
    pool-list                      列出池
    pool-name                      将池 UUID 转换为池名称
    pool-refresh                   刷新池
    pool-start                     启动一个（以前定义的）非活跃的池
    pool-undefine                  取消定义一个不活跃的池
    pool-uuid                      把一个池名称转换为池 UUID
```

### KVM Storage Volume 语句用法：

``` bash 
    Storage Volume (help keyword 'volume'):
    vol-clone                      克隆卷。
    vol-create-as                  从一组变量中创建卷
    vol-create                     从一个 XML 文件创建一个卷
    vol-create-from                生成卷，使用另一个卷作为输入。
    vol-delete                     删除卷
    vol-download                   将卷内容下载到文件中
    vol-dumpxml                    XML 中的卷信息
    vol-info                       存储卷信息
    vol-key                        为给定密钥或者路径返回卷密钥
    vol-list                       列出卷
    vol-name                       为给定密钥或者路径返回卷名
    vol-path                       为给定密钥或者路径返回卷路径
    vol-pool                       为给定密钥或者路径返回存储池
    vol-resize                     创新定义卷大小
    vol-upload                     将文件内容上传到卷中
    vol-wipe                       擦除卷
```
### KVM  Virsh itself 语句用法：

``` bash 
 Virsh itself (help keyword 'virsh'):
    cd                             更改当前目录
    connect                        连接（重新连接）到 hypervisor

    echo            echo 参数   
    exit           退出这个非交互式终端  
    help            打印帮助
    pwd             输出当前目录
    quit            退出这个非交互式终端
```

### 如何卸载kvm虚拟机  
 
``` bash 
    yum remove kvm 或者 yum remove kvm*
```
如果是源码安装，先找到目录后直接删除就可以了

``` bash 
    find / -name kvm // 找到目录找到该虚机的文件位置，删除映像文件和配置文件就可以了。
    cd **** //进入目录
    rm -rf ***
    如果是rpm安装
    rpm -e kvm 或者 rpm -e kvm*
```
版权声明：本站原创文章，于2015年6月18日，由yangc发表.
转载请注明：kvm 虚拟化实用篇（第二篇）help帮助文档大全 


