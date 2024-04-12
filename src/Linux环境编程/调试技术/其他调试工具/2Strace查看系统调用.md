# 1 简介

strace 是 Linux 环境下的一款程序调试工具，用来监察一个应用程序所使用的系统调用。在其最简单的形式中，它可以从开始到结束跟踪二进制的执行，并在进程的生命周期中输出一行具有系统调用名称，每个系统调用的参数和返回值的文本行。
strace 常用来跟踪进程执行时的系统调用和接收所接收的信号，在 Linux 中，进程不能直接访问硬件设备，当进程需要访问硬件设备的时候（如读取磁盘，接收网络数据的时候），必须由用户态模式切换至内核态模式，通过系统调用访问硬件设备。**strace 可以跟踪到一个进程产生的系统调用，包括参数，返回值，执行消耗的时间**。

## 1.1 命令选项

- -c 统计每一系统调用的所执行的时间,次数和出错的次数等.
- -d 输出 strace 关于标准错误的调试信息.
- -f 跟踪由 fork 调用所产生的子进程.
- -ff 如果提供-o filename,则所有进程的跟踪结果输出到相应的 filename.pid 中,pid 是各进程的进程号.
- -F 尝试跟踪 vfork 调用.在-f 时,vfork 不被跟踪.
- -h 输出简要的帮助信息.
- -i 输出系统调用的入口指针.
- -q 禁止输出关于脱离的消息.
- -r 打印出相对时间关于,,每一个系统调用.
- -t 在输出中的每一行前加上时间信息.
- -tt 在输出中的每一行前加上时间信息,微秒级.
- -ttt 微秒级输出,以秒了表示时间.
- -T 显示每一调用所耗的时间.
- -v 输出所有的系统调用.一些调用关于环境变量,状态,输入输出等调用由于使用频繁,默认不输出.
- -V 输出 strace 的版本信息.
- -x 以十六进制形式输出非标准字符串
- -xx 所有字符串以十六进制形式输出.
- -a column 设置返回值的输出位置.默认为 40.
- -e expr 指定一个表达式,用来控制如何跟踪.格式如下:
- [qualifier=][!]value1[,value2]... ：qualifier 只能是 trace,abbrev,verbose,raw,signal,read,write 其中之一.value 是用来限定的符号或数字.默认的 qualifier 是 trace.感叹号是否\* 定符号.例如:
- -eopen 等价于 -e trace=open,表示只跟踪 open 调用.而-etrace!=open 表示跟踪除了 open 以外的其他调用.有两个特殊的符号 all 和 none.
- 注意有些 shell 使用!来执行历史记录里的命令,所以要使用\.
- -e trace=set 只跟踪指定的系统 调用.例如:-e trace=open,close,rean,write 表示只跟踪这四个系统调用.默认的为 set=all.
- -e trace=file 只跟踪有关文件操作的系统调用.
- -e trace=process 只跟踪有关进程控制的系统调用.
- -e trace=network 跟踪与网络有关的所有系统调用.
- -e strace=signal 跟踪所有与系统信号有关的 系统调用
- -e trace=ipc 跟踪所有与进程通讯有关的系统调用
- -e abbrev=set 设定 strace 输出的系统调用的结果集.-v 等与 abbrev=none.默认为 abbrev=all.
- -e raw=set 将指 定的系统调用的参数以十六进制显示.
- -e signal=set 指定跟踪的系统信号.默认为 all.如 signal=!SIGIO(或者 signal=!io),表示不跟踪 SIGIO 信号.
- -e read=set 输出从指定文件中读出 的数据.例如:
- -e read=3,5
- -e write=set 输出写入到指定文件中的数据.
- -o filename 将 strace 的输出写入文件 filename
- -p pid 跟踪指定的进程 pid.
- -s strsize 指定输出的字符串的最大长度.默认为 32.文件名一直全部输出.
- -u username 以 username 的 UID 和 GID 执行被跟踪的命令

## 1.2 strace 的功能

- 它可以基于特定的系统调用或系统调用组进行过滤
- 它可以通过统计特定系统调用的使用次数，所花费的时间，以及成功和错误的数量来分析系统调用的使用。
- 它跟踪发送到进程的信号。
- 可以通过 pid 附加到任何正在运行的进程。
- 调试性能问题，查看系统调用的频率，找出耗时的程序段
- 查看程序读取的是哪些文件从而定位比如配置文件加载错误问题
- 查看程序长时间运行“假死”情况
- 当程序出现“Out of memory”时被系统发出的 SIGKILL 信息所 kill
- 另外因为 strace 拿到的是系统调用相关信息，一般也即是 IO 操作信息，这个对于排查比如 cpu 占用 100%问题是无能为力的。这个时候就可以使用 GDB 工具了。

## 1.3 运行模式

strace 有两种运行模式:

- 通过 strace 启动要跟踪的进程。用法很简单，在原本的命令前加上 strace 即可
- 在不中断进程执行的情况下, 跟踪已经在运行的进程。 这种情况，给 strace 传递个`-p pid`选项即可

## 1.4 输出示例

```bash
root@ubuntu:/usr# strace cat /dev/null
execve("/bin/cat", ["cat", "/dev/null"], [/* 22 vars */]) = 0
brk(0)                                  = 0xab1000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f29379a7000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
...
brk(0) = 0xab1000
brk(0xad2000) = 0xad2000
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
open("/dev/null", O_RDONLY) = 3
fstat(3, {st_mode=S_IFCHR|0666, st_rdev=makedev(1, 3), ...}) = 0
read(3, "", 32768) = 0
close(3) = 0
close(1) = 0
close(2) = 0
exit_group(0) = ?
```

每一行都是一条系统调用，等号左边是系统调用的函数名及其参数，右边是该调用的返回值。 strace 显示这些调用的参数并返回符号形式的值。strace 从内核接收信息，而且不需要以任何特殊的方式来构建内核。

# 2 使用示例

看下面代码，由于没有权限会导致 fopen 失败：

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
    FILE* fp;
    fp = fopen("/etc/shadow", "r");//尝试打开没有权限的文件
    if (fp == NULL)
    {
        printf("Error\n");
        return EXIT_FAILURE;
    }
    return EXIT_SUCCESS;
}
```

编译后尝试使用 strace 查看系统调用的具体错误：

```bash
barret@Barret-PC:~$ strace -i ./a.out
[00007fa5276b416b] execve("./a.out", ["./a.out"], 0x7ffeaeb94758 /* 26 vars */) = 0
[00007f8e60ef3eab] brk(NULL)            = 0x5651c6a18000
[00007f8e60ef2b55] arch_prctl(0x3001 /* ARCH_??? */, 0x7ffc75152290) = -1 EINVAL (Invalid argument)
[00007f8e60ef4d5b] access("/etc/ld.so.preload", R_OK) = -1 ENOENT (No such file or directory)
[00007f8e60ef4ec8] openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
[00007f8e60ef4c99] fstat(3, {st_mode=S_IFREG|0644, st_size=30808, ...}) = 0
[00007f8e60ef50e6] mmap(NULL, 30808, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f8e60ece000
[00007f8e60ef4d8b] close(3)             = 0
[00007f8e60ef4ec8] openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
[00007f8e60ef4f88] read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\360q\2\0\0\0\0\0"..., 832) = 832
[00007f8e60ef4fbe] pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
[00007f8e60ef4fbe] pread64(3, "\4\0\0\0\20\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0", 32, 848) = 32
[00007f8e60ef4fbe] pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0cBR\340\305\370\2609W\242\345)q\235A\1"..., 68, 880) = 68
[00007f8e60ef4c99] fstat(3, {st_mode=S_IFREG|0755, st_size=2029224, ...}) = 0
[00007f8e60ef50e6] mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f8e60ecc000
[00007f8e60ef4fbe] pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
[00007f8e60ef4fbe] pread64(3, "\4\0\0\0\20\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0", 32, 848) = 32
[00007f8e60ef4fbe] pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0cBR\340\305\370\2609W\242\345)q\235A\1"..., 68, 880) = 68
[00007f8e60ef50e6] mmap(NULL, 2036952, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f8e60cda000
[00007f8e60ef519b] mprotect(0x7f8e60cff000, 1847296, PROT_NONE) = 0
[00007f8e60ef50e6] mmap(0x7f8e60cff000, 1540096, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x25000) = 0x7f8e60cff000
[00007f8e60ef50e6] mmap(0x7f8e60e77000, 303104, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x19d000) = 0x7f8e60e77000
[00007f8e60ef50e6] mmap(0x7f8e60ec2000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1e7000) = 0x7f8e60ec2000
[00007f8e60ef50e6] mmap(0x7f8e60ec8000, 13528, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f8e60ec8000
[00007f8e60ef4d8b] close(3)             = 0
[00007f8e60ed7cc0] arch_prctl(ARCH_SET_FS, 0x7f8e60ecd540) = 0
[00007f8e60ef519b] mprotect(0x7f8e60ec2000, 12288, PROT_READ) = 0
[00007f8e60ef519b] mprotect(0x5651c4c25000, 4096, PROT_READ) = 0
[00007f8e60ef519b] mprotect(0x7f8e60f03000, 4096, PROT_READ) = 0
[00007f8e60ef516b] munmap(0x7f8e60ece000, 30808) = 0
[00007f8e60df125b] brk(NULL)            = 0x5651c6a18000
[00007f8e60df125b] brk(0x5651c6a39000)  = 0x5651c6a39000
======================这里显示系统调用没有权限
[00007f8e60dead1b] openat(AT_FDCWD, "/etc/shadow", O_RDONLY) = -1 EACCES (Permission denied)
[00007f8e60dea4f9] fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0x1), ...}) = 0
[00007f8e60deb057] write(1, "Error\n", 6Error===========这里对于printf，打印Error
) = 6
[00007f8e60dc0136] exit_group(1)        = ?
[????????????????] +++ exited with 1 +++
```
