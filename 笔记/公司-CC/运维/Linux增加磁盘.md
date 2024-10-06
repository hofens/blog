

### 添加物理磁盘
增加硬盘 https://www.jianshu.com/p/8b70c15fc92f 



添加完后，fdisk -l 可查看到新添加的磁盘/dev/sdb
``` shell
fdisk -l
```

![](assets/Pasted%20image%2020240308155602.png)

### 磁盘分区
**新建分区**
输入 n 新建分区，然后一直回车，最后输入 w 保存配置；创建完成之后，可输入 p 查看；/dev/sdb 1 即为新建的分区
``` shell
fdisk /dev/sdb
n
回车
w
```

![](assets/Pasted%20image%2020240308161144.png)

**改成 Linux LVM 格式**
不是必须步骤，转成 LVM 便于后续好扩展
``` shell
fdisk /dev/sdb
t
L
8 e
w
```

![](assets/Pasted%20image%2020240308161204.png)

**不重启读取分区表**
``` shell
partprobe
```


### 添加逻辑卷

**创建物理卷（Physical Volume）**：将 /dev/sdb 1 分区设置为 LVM 物理卷。

``` shell
pvcreate /dev/sdb1
```
![](assets/Pasted%20image%2020240308161232.png)

**将物理卷添加到卷组（Volume Group）**：假设您已经有一个卷组，将新的物理卷添加到现有卷组。
`vgcreate <new_volume_group_name> /dev/sdb1`
``` shell
vgcreate vgsdb /dev/sdb1
```
![](assets/Pasted%20image%2020240308161302.png)

**创建逻辑卷（Logical Volume）**：在卷组中创建一个新的逻辑卷。
`lvcreate -l 100%FREE -n <new_logical_volume_name> <volume_group_name>`
``` shell
lvcreate -l 100%FREE -n lvsdb vgsdb
```
![](assets/Pasted%20image%2020240308161315.png)

**格式化逻辑卷**：使用适当的文件系统格式化新创建的逻辑卷（例如，ext 4 文件系统）。
`mkfs.ext4 /dev/<volume_group_name>/<new_logical_volume_name>`

``` shell
mkfs.ext4 /dev/vgsdb/lvsdb
```
![](assets/Pasted%20image%2020240308161333.png)

**挂载逻辑卷**：将新创建的逻辑卷挂载到系统中的一个目录。

`mkdir /mnt/new_volume`
`mount /dev/<volume_group_name>/<new_logical_volume_name> /mnt/new_volume`

``` shell
mkdir /mnt/sdb
mount /dev/vgsdb/lvsdb /mnt/sdb
```

**启动挂载**：将挂载信息添加到 `/etc/fstab` 文件中
`/dev/<volume_group_name>/<new_logical_volume_name> /mnt/new_volume ext4 defaults 0 0`

```shell
vim /etc/fstab
/dev/vgsdb/lvsdb /mnt/sdb ext4 defaults 0 0
```



### 添加软连接

**背景**：/gitlab-runner/cache 目录占用太多内存，希望新增的内容存到另一个磁盘上

``` shell
mv /旧目录/A /旧目录/A_old
ln -s /mnt/sdb/新目录/A /旧目录/A
```

先把原目录去除，这里是改成另一个名字；
创建一个指向新目标目录的符号链接，它就会将文件写入到新的目标目录 /mnt/sdb/A 中。

**实操**

创建软连接
```shell
mkdir -p /mnt/sdb/gitlab-runner/cache_symlink
```

重命名原存储目录（测试没问题后可删除原目录）
```shell
mv /gitlab-runner/cache /gitlab-runner/cache_back
```

软链接指向到/gitlab-runner/cache
```shell
ln -s /mnt/sdb/gitlab-runner/cache_symlink /gitlab-runner/cache
```
![](assets/Pasted%20image%2020240308162059.png)


进入 cache 实际进入的就是/mnt/sdb/gitlab-runner/cache_symlink



### 参考资料

[Hyper-V 添加虚拟硬盘]( https://www.jianshu.com/p/8b70c15fc92f )
[Linux 配置与磁盘管理](https://blog.csdn.net/weixin_51921447/article/details/130279997)

