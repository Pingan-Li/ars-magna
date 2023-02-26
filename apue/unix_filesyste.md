# UNIX Filesystem

## UNIX文件共享的情况

TODO：补充文件系统的三张表。
TODO：O_APPEND的作用需要仔细研究。

### 情况1

在同一个进程内，使用不同的fd操作同一个文件。
这种情况下，如果分别在不同的fd上写入数据，那么会发生数据的覆盖，这是因为每个fd都有自己的偏移量，多个fd之间的偏移量没有同步，每个fd在写入数据的时候都会从自己的偏移量那里开始写入。然而这种情况并不常见，在同一个进程中可以避免使用多个fd打开同一个文件。

### 情况2

多个进程共享操作同一个文件。
由于每个进程在进程表中都有一个记录表，表中就包含了该进程所有打开的文件描述符，所以每个进程的文件描述符都是独占的，然而文件描述符关联的文件系统节点却是共享的，当多个进程使用各自的fd对同一个文件进行写入时，也会发数据覆盖的情况。由于进程之间的数据是隔离的，这种情况下只能求助于操作系统。
在打开fd时，使用O_APPEND可以解决这种情况。

### 情况3

父进程和子进程fork之后的情况与情况2相同

### 情况4

父进程在调用fork之后，子进程会继承父进程打打开的fd，所以子进程和父进程的fd指向了相同的文件表。在这种情况下，子进程和父进程的写入不会相互覆盖。

## Ref

1. <https://blog.csdn.net/weixin_43916755/article/details/128858825>