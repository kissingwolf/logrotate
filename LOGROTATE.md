## LOGROTATE##

日志轮询

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

## 不要修改当前时间##

/var/lib/logrotate.status记录了logrotate的文件操作时间

可以修改这个文件去完成



