# File Systems

## 设备与设备文件

文件系统主要功能就是管理文件，而文件必须存在于某种存储设备上，因此文件系统就免不了需要和各种存储打交道。在内核中，每种设备都具有与之对应的设备驱动程序，用来处理设备的所有IO请求。倘若文件系统需要以某种形式与设备交互，最理想的一种形式就是将设备抽象成文件，文件系统几乎可以零成本支持各种设备，对文件的各种系统调用也可以拓展到设备上。在文件系统中代表一个设备的文件，被成为设备文件。根据FHS标准，设备文件会放在/dev目录下。

设备可以是实际存在的，也可以是抽象的，通常设备可以分为两种类型：（1）字符设备（2）块设备。
| | 字符设备 | 块设备 |
| - | - | - |
| IO形式 | 不定长字节流 | 定长数据块（512字节） |
| 随机读取 | 不支持 | 支持 |
| 代表设备 | 键鼠 | 硬盘 |

注意，字符设备和块设备的区别由驱动层定义。

## 引言

Linux作为一个开源操作系统，必须支持种类繁多的文件系统，如果直接将不同文件系统的接口暴露给上层应用，那么显然是个噩梦。因此必须增加一层抽象，使应用可以避免对接每一个文件系统，这个中间层就是VFS。上层应用发出的各种操作由VFS转发到下层的文件系统中。VFS支持的系统调用有：open(), read(), write(), lseek(), close(),
truncate(), stat(), mount(), umount(), mmpa(), mkdir(), link(), unlink(), symlink(), reaname().

## Directory Entry

  在应用层传入的路径会通过VFS转换成Direcoty entry对象，Directory entry对象可以缓存在内存中，所以效率非常高。

## Inode

每个单独的dentry都有一个指针指向一个inode. Inodes可以是文件文件系统对象，比如常规文件，目录，FIFOs以及其他的东西，它们要么存在于磁盘上，要么存在于内存上（准文件系统）。当需要时，光盘上的inodes会复制到内存中，对inodes的更改会写回光盘。单个inode可以由多个dentry指向（例如，硬链接执行此操作）。i节点维护的信息如下：

文件类型：类型有常规文件、目录、符号链接，以及字符设备。

文件属主：用户ID或者UID

文件属组：组ID或者GID

三个访问权限：属主（用户），属组和其他用户（属主和属组之内的其他用户）

三个事件戳：对文件的最后访问(ls -lu)，对文件的最后修改(ls -l，默认)，对文件状态的最后修改(ls -lc，状态修改是指修改inode节点信息)。

指向文件的hard link数量

文件的大小：以字节为单位

实际分配给文件的块数量，以512字节为单位，这一数字可能不会简单等同于文件大小，因为考虑文件中存在空洞的情况。

## API

### mkfs

### mkswap

### mount

```c
int mount(char const *source, char const *target, char const *fstype, unsigned long mountflags, void const *data);
```

返回值：0成功，-1失败。

### umount

```c
int umount(char const *target);
```

返回值：0成功，-1失败。

## Refs

\[1\]. <https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html>
