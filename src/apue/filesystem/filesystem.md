# File System
## 1. 概述
UNIX的内核使用三种数据结构表示打开的文件：

（1）process table entry
    进程表项，用于记录进程使用的文件描述符。其中包括两个重要的项目：一个就是文件的标志位，另外一个就是指向文件表项的指针。

（2）file table entry
    文件表项，内核为所有打开的文件维持一张文件表，每个文件表项包含：文件状态标志（读，写，追加写，同步和阻塞等），文件的偏移量，指向该文件的vnode表项的指针。

（3）v-node table entry
    每个打开的文件都有一个v-node结构，文件类型和对此文件进行各种操作的函数的指针。v-node还包括了i-node。i-node包含了文件的所有者、文件长度、指向文件的实际数据块在磁盘上位置的指针等等。

总的来说，三者之间的关系就如下图所示：
![内核数据结构关系](figure_3.7.png)
## 2. 文件控制
```C++
#include <sys/stat.h>
int stat(const char *restrict path, struct stat *restric buf);

int fstat(int fd, struct stat *buf);

int lstat(const char *restrict path, struct stat *restrict buf);

int fstatat(int fd, const char *restrict path, struct path *restrict buf, int flag);
```
返回值：成功：返回0；失败，返回-1。

```C++
struct stat{
    mode_t st_mode;           //文件mode，权限控制
    ino_t st_ino;             //i-node序号
    dev_t st_dev;             //设备序号
    dev_t st_rdev;            //设备序号（针对特殊类型的文件）
    nlink_t st_nlink;         //链接的数量
    uid_t st_uid;             //所有者的userid
    gid_t st_gid;             //所有者的groipid
    off_t st_size;            //文件大小
    struct timespec at_time;  //最后一次访问时间
    struct timespec st_mtime; //最后一次修改时间
    struct timespec st_ctime; //最后一次文件状态改变的时间
    blksize_t st_blksize;     //最佳IO块
    blksize_t st_blocks;      //分配的磁盘blocks。
}
```
## 3. 文件类型
UNIX中对文件的类型做了区分：
(1)普通文件(regular file)
    就是通常人们所理解的文件，里面存储了数据。在UNIX中，里面存储的是二进制还是文本文件并不重要，UNIX将他们一视同仁。但是UNIX必须识别可执行文件的格式(executable)
(2)目录文件(directory)
    目录文件，这种文件包含了其他文件的名字以及指向这些文件相关信息的指针。对一个目录内具有读权限的任何一个文件都可以读该文件内的名称，只有内核可以直接写目录文件。
(3)块特殊文件
    这种类型的文件提供对设备带缓冲的访问，每次访问以固定长度为单位进行。
(4)字符特殊文件
    这种类型的文件提供对设备不带缓冲的访问，每次访问以可变长度进行，系统中的所有设备要么是块设备，要么是字符设备。
(5)FIFO
    这种类型的文件用于进程间通信，有时也被称为是具名管道(name pipes)。
(6)套接字(Socket)
    这种类型的文件用进程间的网络通信。套接字也可以用在一个宿主机上进行进程间非网络通信。
(7)符号连接(symbolic link)
    这种类型的文件用于指向其他的文件。

## 4. 用户ID和组ID
与一个进程相关的ID有6个或者更多：

（1）实际用户ID和实际组ID

    实际用户ID和实际组ID表示当前用户是谁，这个两个ID是在登录的时候取自口令文件中的内容，通常两个字段在登录期间不可变，但是root有方法修改。
（2）有效用户ID，有效组ID和附属组ID

    这三个字段决定了用户访问文件的权限
（3）保存的设置ID和保存的设置组ID

    作为有效ID副本。
在通常情况下，有效用户ID就等于实际用户ID，有效组ID就等于实际组ID。
## 5. 文件访问权限
文件权限通过st_mode值进行设定，
