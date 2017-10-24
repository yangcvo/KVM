# KVM
KVM虚拟化入门

概述

KVM 是最近非常火热的技术，是云计算的基础，个人在工作上接触比较多，的确是非常好的一个东西。今天的主要目的就是在自己的家用电脑上将KVM跑起来。

#### 安装过程

1 ： 确认你的电脑支持硬件辅助虚拟化

理论上来将，虚拟化可以大致分为全虚拟化与半虚拟化两种类型，全虚拟化即纯的软件虚拟化，可以不依赖硬件，完全在OS内部（大部分是用户空间）实现，由OS来模拟所有的关键指令，这类虚拟化技术通用性好，但是性能通常很差，尤其是通信性能或者某些类型的计算性能，不能做大大规模的部署；半虚拟化也就是硬件辅助虚拟化，通常需要CPU的支持才能实现，其性能通常较高，可以接近RAW OS的性能，适合在云计算的环境中大量部署，不过其对CPU有要求，好在现在几乎所有主流的X86系列CPU都提供了对虚拟化的支持，所以，一般来说，这都不是事儿！

KVM是一种硬件辅助虚拟化的虚拟机实现技术，其核心是kvm.ko，运行与linux内核中，负责处理guest os的特权指令与硬件辅助系统的交互。KVM一开始就将性能作为一个非常关键的因素加以考虑，所以它它压根儿就没有考虑全虚拟化的解决方案，也就是说，要跑KVM，你的CPU必须支持硬件虚拟化。

ps：系统是centos 7.1以上的kvm 安装虚拟机Windows 驱动不支持。 centos 6.5支持，本人亲自测试验证过。


#### 入门学习文档：

* [kvm磁盘虚拟化技术应用 (一)](https://github.com/yangcvo/KVM/blob/master/kvm%E7%A3%81%E7%9B%98%E8%99%9A%E6%8B%9F%E5%8C%96%E6%8A%80%E6%9C%AF%E5%BA%94%E7%94%A8.md)
* [kvm管理平台webvirtmgr的部署](https://github.com/yangcvo/KVM/blob/master/kvm%E7%AE%A1%E7%90%86%E5%B9%B3%E5%8F%B0webvirtmgr%E7%9A%84%E9%83%A8%E7%BD%B2.md)
* [kvm虚拟化(三)虚拟机快速克隆](https://github.com/yangcvo/KVM/blob/master/kvm%E8%99%9A%E6%8B%9F%E5%8C%96(%E4%B8%89)%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%BF%AB%E9%80%9F%E5%85%8B%E9%9A%86.md)
* [KVM虚拟化 (二) help帮助文档大全](https://github.com/yangcvo/KVM/blob/master/KVM%E8%99%9A%E6%8B%9F%E5%8C%96%20(%E4%BA%8C)%20help%E5%B8%AE%E5%8A%A9%E6%96%87%E6%A1%A3%E5%A4%A7%E5%85%A8.md)
* [KVM虚拟机如何进行克隆操作](https://github.com/yangcvo/KVM/blob/master/KVM%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E5%85%8B%E9%9A%86%E6%93%8D%E4%BD%9C.md)
* [KVM虚拟化(六)调整虚拟机CPU大小,添加memory内存,添加虚拟机硬盘disk,虚拟机内存减半现象](https://github.com/yangcvo/KVM/blob/master/KVM%E8%99%9A%E6%8B%9F%E5%8C%96(%E5%85%AD)%E8%B0%83%E6%95%B4%E8%99%9A%E6%8B%9F%E6%9C%BACPU%E5%A4%A7%E5%B0%8F%2C%E6%B7%BB%E5%8A%A0memory%E5%86%85%E5%AD%98%2C%E6%B7%BB%E5%8A%A0%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%A1%AC%E7%9B%98disk%2C%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%86%85%E5%AD%98%E5%87%8F%E5%8D%8A%E7%8E%B0%E8%B1%A1.md)
* [kvm虚拟化小结（六）libguestfs-tools](https://github.com/yangcvo/KVM/blob/master/kvm%E8%99%9A%E6%8B%9F%E5%8C%96%E5%B0%8F%E7%BB%93%EF%BC%88%E5%85%AD%EF%BC%89libguestfs-tools.md)
* [KVM虚拟机更改存储路径解决方法](https://github.com/yangcvo/KVM/blob/master/KVM虚拟机更改存储路径解决方法.md)


@后期陆续更新关注：个人博客：blog.yancy.cc

注意！！！ 转载或者复制本文内容标注原文地址。谢谢 


