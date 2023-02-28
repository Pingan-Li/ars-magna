# Virtual File System

Virutal FIle System is also known as virutal filesystem switch.

## 引言

Linux作为一个开源操作系统，必须支持种类繁多的文件系统，如果直接将不同文件系统的接口暴露给上层应用，那么显然是个噩梦。因此必须增加一层抽象，使应用可以避免对接每一个文件系统，这个中间层就是VFS。上层应用发出的各种操作由VFS转发到下层的文件系统中。VFS支持的系统调用有：open(), read(), write(), lseek(), close(),
truncate(), stat(), mount(), umount(), mmpa(), mkdir(), link(), unlink(), symlink(), reaname().

## Directory Entry

  在应用层传入的路径会通过VFS转换成Direcoty entry对象，Directory entry对象可以缓存在内存中，所以效率非常高。

## Inode

    每个单独的dentry都有一个指针指向一个inode. Inodes可以是文件文件系统对象，比如常规文件，目录，FIFOs以及其他的东西，它们要么存在于磁盘上，要么存在于内存上（准文件系统）。当需要时，光盘上的inodes会复制到内存中，对inodes的更改会写回光盘。单个inode可以由多个dentry指向（例如，硬链接执行此操作）。
