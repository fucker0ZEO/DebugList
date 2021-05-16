# pstack 分析Nacos 失败 |Debug日志

## 现象 

在Ubuntu下安装pstack后，使用 `sudo pstack nacos进程的PID`命令跟踪堆栈报错

![image-20210516150103192](pstack%20%E5%88%86%E6%9E%90Nacos%20%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210516150103192.png)



## 解决问题

centOS则没有这种问题，我以`Ubuntu pstack`为关键词搜索，没有找到相关的安装教程，但是找到了

[pstack 命令打印不出堆栈 无法正常使用-问答-阿里云开发者社区-阿里云 (aliyun.com)](https://developer.aliyun.com/ask/87815)

他和我遇见的情况类似。

> 操作不允许，没有那个进程
> 没有权限，要么这个进程做了保护。

于是我加上`sudo`,结果在process处是？？？？,并不是数字。

后面又找到了

[aliyun ubuntu pstack无法使用的问题解决 crawl: Input/output error_chenycbbc0101的博客-CSDN博客](https://blog.csdn.net/chenycbbc0101/article/details/78531886)

他提示pstack可能存在乱码

在Linux下，一切皆文件，于是使用 `nano /usr/bin/pstak` 打开pstack文件

![image-20210516151143997](pstack%20%E5%88%86%E6%9E%90Nacos%20%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210516151143997.png)

果然乱码，于是删除pstack文件，并且新建一个pstack文件，copy对应脚本内容并赋权限

```shell
#删除pstack
sudo rm -rf pstack
#新建pstack文件并copy
sudo nano pstack
#copy脚本放在下一个代码框，太长了影响观感

#赋权限
chmod -R 777 pstack
#再次查询nacos进程
sudo pstack nacos的进程PID
```



结果

![image-20210516152404466](pstack%20%E5%88%86%E6%9E%90Nacos%20%E5%A4%B1%E8%B4%A5%20Debug%E6%97%A5%E5%BF%97.assets/image-20210516152404466.png)



用nano写入的pstack中的内容

```shell
#!/bin/sh

if test $# -ne 1; then
    echo "Usage: `basename $0 .sh` <process-id>" 1>&2
    exit 1
fi

if test ! -r /proc/$1; then
    echo "Process $1 not found." 1>&2
    exit 1
fi

# GDB doesn't allow "thread apply all bt" when the process isn't
# threaded; need to peek at the process to determine if that or the
# simpler "bt" should be used.

backtrace="bt"
if test -d /proc/$1/task ; then
    # Newer kernel; has a task/ directory.
    if test `/bin/ls /proc/$1/task | /usr/bin/wc -l` -gt 1 2>/dev/null ; then
	backtrace="thread apply all bt"
    fi
elif test -f /proc/$1/maps ; then
    # Older kernel; go by it loading libpthread.
    if /bin/grep -e libpthread /proc/$1/maps > /dev/null 2>&1 ; then
	backtrace="thread apply all bt"
    fi
fi

GDB=${GDB:-/usr/bin/gdb}

if $GDB -nx --quiet --batch --readnever > /dev/null 2>&1; then
    readnever=--readnever
else
    readnever=
fi

# Run GDB, strip out unwanted noise.
$GDB --quiet $readnever -nx /proc/$1/exe $1 <<EOF 2>&1 | 
set width 0
set height 0
set pagination no
$backtrace
EOF
/bin/sed -n \
    -e 's/^\((gdb) \)*//' \
    -e '/^#/p' \
    -e '/^Thread/p'
```

