# LOGROTATE

## Logrotate 介绍

**logrotate**是linux自带的一个日志轮询管理工具。可以很方便的用它来进行系统或应用服务日志的压缩和轮询更新。

## Logrotate 作用

* Logrotate可以轮换、压缩和调用邮件系统将日志文件发送到指定的E-mail邮箱。
* Logrotate 可以用来把旧的日志文件删除，并创建新的日志文件，有效的降低日志存储对磁盘空间的需求，我们把这种功能叫做“转储”
* Logrotate 可以调用cron（计划任务）根据日志文件的大小，也可以根据其天数来转储

## Logrotate 原理
* 默认的logrotate是被加入cron的`/etc/cron.daily`中作为每日任务执行。
* `/etc/logrotate.conf`为其默认配置文件指定每个日志文件的默认规则。
* `etc/logrotate.d/*` 为`/etc/logrotate.conf`默认包含目录`include(关键字)`中的文件也会被logrotate读取。指明每个系统服务日志文件的特定规则。
* `/var/lib/logrotate/statue`中默认记录logrotate上次轮换日志文件的时间。

## Logrotate 配置说明

| 配置项                              | 说明                                       |
| -------------------------------- | ---------------------------------------- |
| compress/nocompress              | 通过gzip 压缩转储以后的日志/反之不需要压缩时，用nocompress    |
| copytruncate/ nocopytruncate     | 用于还在打开中的日志文件，把当前日志备份并截断/反之，备份日志文件但是不截断   |
| create mode owner group/nocreate | 转储文件，使用指定的文件模式创建新的日志文件反之，不建立新的日志文件       |
| delaycompress/nodelaycompress    | 和 compress 一起使用时，转储的日志文件到下一次转储时才压缩覆盖 delaycompress 选项，转储同时压缩。 |
| errors address                   | 专储时的错误信息发送到指定的Email 地址                   |
| ifempty/notifempty               | 即使是空文件也转储，这个是 logrotate 的缺省选项。如果是空文件的话，不转储 |
| mail address/nomail              | 把转储的日志文件发送到指定的E-mail 地址转储时不发送日志文件        |
| olddir directory/noolddir        | 转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统转储后的日志文件和当前日志文件放在同一个目录下 |
| prerotate/endscript              | 在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行         |
| postrotate/endscript             | 在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行         |
| daily                            | 指定转储周期为每天                                |
| weekly                           | 指定转储周期为每周                                |
| monthly                          | 指定转储周期为每月                                |
| rotate count                     | 指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份       |
| size size                        | 当日志文件到达指定的大小时才转储，Size 可以指定 bytes (缺省)以及KB (sizek)或者MB (sizem). |
| tabootext [+] list               | 让logrotate 不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和 ~ |

配置格式

```config
/full/path/to/file
{
option(s)
}
```

## 日志轮询实验

如果不指定rotate、周期等参数，则根据/etc/logrotate.conf主配置文件里的默认周期和rotate参数来执行轮询

    /var/log/test.log {
        missingok    # 如果没有上一次轮询的文件，则不报错，执行下一次轮询 
        sharedscripts  # 轮询前后可以执行脚本，打开该参数
        postrotate # 轮训后执行脚本
                bash /tmp/test.sh  # 实际执行脚本的命令
        endscript # 脚本结束后标志
        create # 是否要创建个同名的文件
        daily # 轮询周期
        rotate 10000 # 保留多少后轮询文件
       }
测试方法：

```shell
[root@serverb log]# logrotate -v /etc/logrotate.conf
[root@serverb log]# ls
anaconda               cron       ovirt-guest-agent  tallylog
audit                  dmesg      ppp                test.log
boot.log               dmesg.old  qemu-ga            test.log-20161125
btmp                   htpd      rhsm               tuned
chrony                 lastlog    sa                 wtmp
cloud-init.log         maillog    secure             yum.log
cloud-init-output.log  messages   spooler
```

注意：logrotate实际是以当前时间和日志的时间做对比的方式完成日志轮询

什么意思：举个例子，根据我们当前的配置，日志是一天轮询一次

```shell
[root@serverb log]# ll test.log
-rw-r--r--. 1 root root 0 Nov 23 00:00 test.log

```

由于轮训后文件和当前时间相隔超过一天，所以，现在执行logrotate 命令会出现以下结果

```shell
[root@serverb log]# logrotate -v /etc/logrotate.d/test.log 
reading config file /etc/logrotate.d/test.log

Handling 1 logs

rotating pattern: /var/log/test.log  after 1 days (10000 rotations)  # 请于一天后再去执行轮询
empty log files are rotated, old logs are removed
considering log /var/log/test.log
  log does not need rotating
not running postrotate script, since no logs were rotated
set default create context

```

如果当前时间和日志文件时间没有超过一天，且没有日志轮询文件，则执行命令会出现如下结果

这里我们将test.log的文件时间改掉

```shell
[root@serverb log]# touch -t "201611230000" test.log
[root@serverb log]# date -s "20161123"
Wed Nov 23 00:00:00 EST 2016
[root@serverb log]# logrotate -v /etc/logrotate.d/test.log 
reading config file /etc/logrotate.d/test.log

Handling 1 logs

rotating pattern: /var/log/test.log  after 1 days (10000 rotations)
empty log files are rotated, old logs are removed
considering log /var/log/test.log
  log needs rotating
rotating log /var/log/test.log, log->rotateCount is 10000
dateext suffix '-20161123'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
fscreate context set to system_u:object_r:var_log_t:s0
renaming /var/log/test.log to /var/log/test.log-20161123
creating new /var/log/test.log mode = 0644 uid = 0 gid = 0
running postrotate script
compressing log with: /bin/gzip
set default create context
[root@serverb log]# ls
anaconda               lastlog            spooler
audit                  maillog            spooler-20161127
boot.log               maillog-20161127   t
btmp                   messages           tallylog
chrony                 messages-20161127  test.log
cloud-init.log         ovirt-guest-agent  test.log-20161122.gz
cloud-init-output.log  ppp                test.log-20161123.gz
cron                   qemu-ga            tuned
cron-20161127          rhsm               wtmp
dmesg                  sa                 yum.log
dmesg.old              secure
httpd                  secure-20161127
[root@serverb log]# 

```

```shell
[root@serverb log]# rm -rf test.log-20161122.gz 
[root@serverb log]# ll test.log
-rw-r--r--. 1 root root 0 Nov 23 00:00 test.log
[root@serverb log]# touch -t "201611220000" test.log 
[root@serverb log]# date -s "20161122"
Tue Nov 22 00:00:00 EST 2016
[root@serverb log]# logrotate -v /etc/logrotate.d/test.log 
reading config file /etc/logrotate.d/test.log

Handling 1 logs

rotating pattern: /var/log/test.log  after 1 days (10000 rotations)
empty log files are rotated, old logs are removed
considering log /var/log/test.log
  log needs rotating
rotating log /var/log/test.log, log->rotateCount is 10000
dateext suffix '-20161122'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
fscreate context set to system_u:object_r:var_log_t:s0
renaming /var/log/test.log to /var/log/test.log-20161122
creating new /var/log/test.log mode = 0644 uid = 0 gid = 0
running postrotate script
compressing log with: /bin/gzip

```

```shell
[root@serverb log]# ll test.log*
-rw-r--r--. 1 root root  0 Nov 22 00:00 test.log
-rw-r--r--. 1 root root 20 Nov 22 00:00 test.log-20161122.gz
-rw-r--r--. 1 root root 20 Nov 23  2016 test.log-20161123.gz

```

可以看到现在的test.log的时间为1122日

那原则上，我们需要将该文件时间改为当前实际时间，当前时间是11月23日21:58分

```shell
[root@serverb log]# date -s "20161123 21:58"
Wed Nov 23 21:58:00 EST 2016
[root@serverb log]# rm -rf test.log-20161123.gz 
[root@serverb log]# touch -t "201611232158" "test.log"
[root@serverb log]# ll test.log
-rw-r--r--. 1 root root 0 Nov 23 21:58 test.log
[root@serverb log]# logrotate -v /etc/logrotate.d/test.log 
reading config file /etc/logrotate.d/test.log

Handling 1 logs

rotating pattern: /var/log/test.log  after 1 days (10000 rotations)
empty log files are rotated, old logs are removed
considering log /var/log/test.log
  log needs rotating
rotating log /var/log/test.log, log->rotateCount is 10000
dateext suffix '-20161123'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
fscreate context set to system_u:object_r:var_log_t:s0
renaming /var/log/test.log to /var/log/test.log-20161123
creating new /var/log/test.log mode = 0644 uid = 0 gid = 0
running postrotate script
compressing log with: /bin/gzip
set default create context
```

/var/lib/logrotate.status记录了logrotate的文件操作时间

可以修改这个文件去完成



