# UNIX IO

## 引言

UNIX系统中最基本的IO函数有5个，分别是open，read，write，lseek，和close，这些函数通常被称为无缓冲IO，这里的无缓冲指的是，每一次调用read和write，都会发起一次内核中的系统调用。以上的函数不属于ISO C标准库中的函数，但是属于Single UNI标准中的函数。

对于内核而言，所有的文件都需要用文件描述符进行引用。比如0,1,2就分别代表标准输入，标准输出，和错误输出流，但是更应该使用STDIN_FILENO，STDOUT_FILENO，STDERR_FILENO。

### 字节流模型

字节流是一种普遍使用的IO抽象模型，字节流是一种以字节为单位的数据流，它的每一个字节都有自己的编号，代表了某个字节在整个字节流中的位置，说白了就是相对于首字节的偏移量，字节流以EOF作为结束的标志。有了字节流模型，文件的IO（但不限于文件）就转换成了对字节流的操作。

## open

调用open和openat可以打开或者创建一个文件

```C++
#include <fcntl.h>

int open(char const *path, int oflag, ... /*mode_t mode*/);

//fd代表目录的fd，path表示相对路径。
int openat(int fd, char const *path, int oflag, .../*mode_t mode*/);

// 如果正常，则返回文件描述符；如果出错，则返回-1。
```

参数中的mode是可变的，所以用...代替，先说oflag。oflag用于调整打开文件的行为。
以下五个参数必选而且唯一：

O_RDONLY: 只读打开

O_WRONLY: 只写打开

O_RDWR: 读写打开

O_EXEC: 只执行打开

O_SEARCH: 只搜索打开, 目前的操作系统都未支持这个

----
下列的常量是可选的：

O_APPEND: 保证每次写入的数据都追加到末尾。

O_CLOEXEC:

O_CREAT: 如果路径指向的文件不存在，那么就创建它；只有在这个参数存在的情况下，mode参数才会起作用。

O_DIRECTORY: 限定打开的路径必须是目录。

O_EXCL: 如果制指定了O_CREAT的情况下，文件已经存在则会出错，EXCL应该是代表excluded的意思。

O_NOCITTY:

O_NOFOLLOW: 排除符号链接的情况。

O_NONBLOCK:

O_SYNC: 每次写入都会等待IO设备在实际上完成IO，

O_TRUNC: 如果此文件存在，而且为只写或读写成功打开时，将长度截断为0。

O_RSYNC:

TODO

## creat函数

creat函数用于创建文件，早先的open函数不具备创建文件的能力，所以需要creat函数

```C++
#include <fcntl.h>
int creat(char const *path, mode_t mode);
//如果创建成功，则会返回文件描述符；否则返回-1.
```

现在已经无需creat函数了，因为creat函数创建的文件都是只写的模式，这是一个明显的缺陷，可能会导致TOCTTOU(Time of Check to Time of Use)的问题。

## close

关闭文件，而且会释放这个文件上的所有记录锁。

```C++
#include <unistd.h>

int close(int fd);
// 返回值：若成功，返回0；若出错，返回—1.
```

## lseek

每个打开的文件都有一个关联的文件偏移量，用于度量从文件的开始位置的字节数。注意，读和写共用同一个偏移量，默认的情况下，除非制定O_APPEND，否则初始的偏移量都是0.

```C++
#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);
// 返回值：若成功，返回新的文件偏移量；若出错，返回-1.
```

whence有以下的三种情况：

- SEEK_SET，将文件的偏移量设置为距离文件开始offset个字节
- SEEK_CUR，将文件的偏移量设置为当前位置加上offset个字节
- SEEK_END，将文件的偏移量设置为文件长度加上offset个字节

如果文件描述符指向的是管道，FIFO和网络套接字，那么永远都会返回-1，并且将error设置为ESPIPE。

## read

read函数用于从一个文件描述符中读取数据，数据形式是字节流，单位是char，也就是说传入的buffer应当是char类型的，C语言中的数组在传参中会退化成指针，所以参数中要显式传递缓冲区的大小。

```C++
#include <unistd.h>

int read(int fd, char *buffer, size_t size);
//返回值，若成功，则返回读取到的字节数；若已经到达文件尾端(EOF)，则返回0；若出错则返回-1.
```

实际读取的字节数有可能不会填满缓冲区，有多种情况，这里就不再列举。
一种比较标准的读取方式：
while循环中会排除掉nbytes == 0的情况，也就是碰到了EOF。

```C++
  int nbytes;
  while ((nbytes = read(fd, buff, buff_size))) {
    // 排除出错的情况
    if (nbytes == -1) {
      // error handing.
    }
    // normal control flow.
  }
```

## write

write函数用于将数据写入文件。

```C++
#include <uinstd.h>

sszie_t write(int fd, void const *buf, size_t nbytes);

// 返回值：若成功，返回已经写入的字节数；若出错，返回-1
```

返回值在绝大多数情况下都与nbytes的值相同，否则很有可能出现了错误。

## dup & dup2

dup和dup2用于复制一个现有的文件描述符。

```C++
#include <unistd.h>

int dup(int fd);

int dup2(int fd, int fd2);
```

返回值：若成功，返回新的文件描述符；若失败，返回-1.
dup返回的新文件描述符一定是当前可用fd中最小的。
dup2可以用fd2参数制定新的描述符的值，如果fd2已经打开，那么现将其关闭，如果fd等于fd2，则返回fd2但是并不关闭。

复制文件描述符还有另外一种fcntl(fc, F_DUPFD, fd2);
但是dup2如果使用fcntl不再是原子操作，可能会造成同步性问题。

TODO
