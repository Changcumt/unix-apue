###  chapter03 文件IO

本长所列文件相关操作 open write create 等方法 如无特殊说明，都是在<fcntl.h> 头文件中。

#### open / openat
```c
 int open(const char *path, int flag, ... /* mode_t mode*/ );
 int openat(int fd, const char *path, int flag, ... /* mode_t mode*/ );
```
