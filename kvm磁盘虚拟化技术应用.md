## kvm磁盘虚拟化技术应用
镜像创建：

qemu-img create 命令。 

-f 参数指定镜像格式，默认的话是raw格式。 

```bash
qemu-img create test 50G
生成出是默认的raw格式

指定格式：qcow2

qemu-img create test.qcow2 -f qcow2 50G
```

qcow2 格式 文件特别小，压缩比tar压缩效率高很多。在镜像传输上非常有意义。

raw是不支持快照的，qcow2是支持快照。

快照管理命令：snapshot

```bash
查看快照：使用-l 
删除快照：使用-d
还原快照：使用-a
```

镜像检查只支持：`qcow2, qed. vdi`




