###  chapter03 文件IO

本长所列文件相关操作 open write create 等方法 如无特殊说明，都是在<fcntl.h> 头文件中。

#### open / openat
```c
 int open(const char *path, int flag, ... /* mode_t mode*/ );
 int openat(int fd, const char *path, int flag, ... /* mode_t mode*/ );
```
flag 参数列表 参数常量定义在fcntl.h中

参数常量名称 | 含义
---------- | ---
O_RDONLY   | 只读打开
O_WRONLY   | 只写打开
O_RDWR     | 读写打开 
O_EXEC     | 只执行打开
O_SEARCH   | 只搜索打开（针对目录）
O_APPEND   | 每次写时候追加到文件尾端
O_CLOEXEC  | 把 FD_CLOEXEC设置为文件描述标志
O_CREAT    | 若文件不存在 则创建
O_DIRECTORY| 如果path引用的不是目录 则报错
O_EXCL     | 测试文件是否存在，如果已经存在 则会报错
O_NOCTTY   | 链接终端相关 暂时不考虑
O_NOFOLLOW | path是符号链接 暂时不考虑
O_NONBLOCK | path引用特殊文件 fifo 等 暂时不考虑
O_SYNC     | 每次等待write I/O完成后再行读
O_TRUNC    | 如果文件已经存在，而且为只写或者读写成功打开，将其长度截断为0
O_TTY_INIT | 终端有关 暂时不考虑
O_DSYNC    | 动态的判断是否需要等待，如果write不影响读取，则不需要等待
O_RSYNC    | read等待，直到写操作完成

fd 参数把open 和 openat 区分开。 如果使用的是绝对路径，open等同于openat，如果使用相对路径，fd只出了相对路径在文件系统中的开始地址。
fd有个特殊值 AT_FDCWD 指向当前工作目录。

mode 权限相关参数在chapter4 中会说明

#### create
```c
int creat (const char *path, mode_t mpde)
```
可以用open代替, creat在实际应用中的用处已不是很大

```c
open(path, O_RDWR | O_CREAT,mode)
```

#### close

```c
#include <unistd.h>

int close(int fd);
```

#### lseek 

```c
#include <unistd.h>
off_t lseek(int fd,off_t offset,int whence);
```
通常读和写操作都会从当前的文件偏移量开始，并使偏移量增加所读写的字数。 当打开一个文件时候 除了指定O_APPEND的情况，偏移量默认是0.

offet是一个类型为off_t的整数。值的意义与whence有关:
* whence 是 SEEK_SET的时候，将文件的偏移量设置为距离文件开始的offset字节
* whence 是 SEEK_CUR的时候，将文件的偏移量设置为当前值加offset个字节，offset可以为正或者负
* whence 是 SEEK_END的时候，将文件的偏移量设置为文件长度+offset


#### demo 使用方法open write lseek

注： 使用Xcode无法看到生成的文件 需要使用cc 或者 gcc命令来编译文成out文件 然后运行
```c
#include "apue.h"
#include <limits.h>
#include <fcntl.h>

char buf1[] = "abcdefgij";
char buf2[] = "ABCDEFGIJ";

int main(void) {
    // insert code here...
    int fd;
    
    if((fd = open("file.hole", O_RDWR | O_CREAT ,FILE_MODE))<0){
        printf("creat error\n");
    }
    
    if(write(fd, buf1, 10)!= 10){
          printf("buf1 write error\n");
    }
    
    // offset now is 10
    
    if(lseek(fd, 16384,SEEK_SET) == -1){
          printf("lseek error\n");
    }
    
    // offset now is 16384
    
    if(write(fd, buf2, 10)!=10){
          printf("write buf2 error\n");
    }
    // offset niw is 16394

    return 0;
}

```

#### read

```c

ssize_t read(int fd, void *buf, size_t nbytes);

```
如果read成功，则返回已经读到的字节数。如已经达到文件尾端，则返回0. 如果出错 返回-1

#### write
```c
ssize_t write(int fd,const void *buf,size_t nbytes)
```
若成功 返回已写的字节数，若出错 返回-1

#### demo
```c
#include "apue.h"
#include <limits.h>
#include <fcntl.h>

#define BUFFSIZE 4096

int main(void) {
    // insert code here...
    int n;
    char buf[BUFFSIZE];
    while((n=read(STDIN_FILENO,buf,BUFFSIZE))>0)
      if(write(STDOUT_FILENO,buf,n) !=n)
          printf('write error');

      if(n<0)
          printf('read error');
    
    return 0;
}

```
