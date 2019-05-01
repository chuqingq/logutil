# 分析传统logrotate工作原理

https://my.oschina.net/chuqq/blog/3044695

https://www.cnblogs.com/sailrancho/p/4784763.html

logrotate实现：

https://github.com/logrotate/logrotate

缺点：

- create方案依赖业务的信号处理
- copytruncate方案可能导致丢日志数据
- 两套方案都是定时执行：无法准确按照大小进行分割

# 新方案

设计理念：

- 日志内容的格式应该有日志库实现：包括时间、级别、线程、文件、函数、行数等
- 输出后和日志库无关，需要日志轮转工具实现：日志文件按时间、大小分割

# Usage

## 使用方式1：看护子进程的输出

启动子进程，并把标准输出和标准错误保存到日志文件中。对日志内容不做任何更改。

    logutil [options] watch [command] [args]

例如：

    logutil -dir /home/chuqq/log_dir -file mylog-%Y%m%d.log -count 10 [-size 1048576 | -time 86400] watch ./count -n 10000

## 使用方式2：看护其他进程的标注输出/标准错误

启动业务进程，并把标准输出/标准错误重定向给logutil。对日志内容不做任何更改。

    ./count.sh | logutil [options] pipe

例如：

    logutil -dir /home/chuqq/log_dir -file mylog-%Y%m%d.log -count 10 [-size 1048576 | -time 86400] watch ./count.sh


## 使用方式3：监控日志文件，并轮转（和传统logrotate方案类似）

    logutil [options] rotate [file]

例如：

    logutil -dir /home/chuqq/log_dir -file mylog-%Y%m%d.log -count 10 [-size 1048576 | -time 86400] rotate /home/chuqq/log_dir/count.log

### TODO

- 支持通过信号让业务进程重新打开日志文件
- 支持多个日志文件都copytruncate后统一让业务重新打开日志文件

## 其他使用方式（考虑中，待定）

### TODO

- `logutil -p <pid> attach` attach到使用方式1或2的logutil进程上，可以实时查看正在输出的日志

# 参数说明

- `-dir <logdir>` logutil写入的日志文件的路径。不支持动态参数。
- `-log <logfile>` logutil写入的日志文件。支持动态参数，例如 mylog-%Y%m%d.log，生成 mylog-20190501.log
- `-count <logcount>` logutil写入的日志文件数量，决定了-log参数指定的文件最大数量
- `-size <maxsize>` logutil分割日志文件的最大字节数，不会超过这个大小。和`-time`二选一
- `-time <seconds>` logutil分割日志文件的时间，按分钟、小时、天对齐，单位是秒。例如86400表示每天0点分割。和`-size`二选一

