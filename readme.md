# Usage

## 使用方式1：看护子进程的输出

启动子进程，并把标准输出和标准错误保存到日志文件中。对日志内容不做任何更改。

    logutil [options] watch [command] [args]

例如：

    logutil -log /home/chuqq/log_dir/log-{index}.log -count 10 [-size 1048576 | -time 86400] watch ./count.sh

## 使用方式2：监控日志文件，并轮转

    logutil [options] rotate [file]

例如：

    logutil -log /home/chuqq/log_dir/log-{index}.log -count 10 [-size 1048576 | -time 86400] rotate /home/chuqq/log_dir/count.log

# 参数说明

- `-log <logfile>` logutil写入的日志文件，其中{index}表示第几个文件，取值为1～count，由近及远。
- `-count <logcount>` logutil写入的日志文件数量，决定了-log参数中{index}的最大值
- `-size <maxsize>` logutil分割日志文件的最大字节数，不会超过这个大小。和`-time`二选一
- `-time <seconds>` logutil分割日志文件的时间，按分钟、小时、天对齐，单位是秒。例如86400表示每天0点分割。
