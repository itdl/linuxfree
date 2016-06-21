# **10. Linux 计划任务**
##### 什么是计划任务？
##### 我们每个人每周都在经历很多事情，有些事情是例行性的，比如说是上课或者上班；有些事情则是突发性的，譬如逃课去看球、翘班去钓鱼等等。
##### 针对这些事情，通常情况下我们需要在手机上设置闹钟来提醒我们，这大大的方便了我们的生活。介于我们正在进行 Redhat 系统管理的培训，那么 Redhat 系统里是不是也有类似的工具，可以让我们很方便的对系统进行管理呢？
##### 接下来我们将会介绍两种工具来处理不同情况下的任务：
##### - 处理突发性任务 -- at
##### - 处理周期性任务 -- cron

### **10.1 at 工具**
##### 1. 要处理突发事务前，我们首先需要确认系统是否安装 at 工具.

```
[root@test3 ~]# rpm -qa at 
at-3.1.13-20.el7.x86_64
```

##### 如上图所示，我们系统中安装的 at 工具版本为3.1.13.
```
[root@test3 ~]# rpm -ivh at-3.1.13-20.el7.x86_64.rpm 
warning: at-3.1.13-20.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:at-3.1.13-20.el7                 ################################# [100%]
```

##### 默认情况下系统已经安装好 at 工具，针对特殊情况下没有安装 at 工具的系统，可以使用上述命令来手动安装。
##### 2. 查看 atd 服务运行状态，启动、关闭、重启 atd 服务
##### 2.1 查看 atd 服务运行状态.
```
[root@test3 ~]# systemctl status atd.service 
● atd.service - Job spooling tools
   Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2016-06-19 14:54:14 CST; 1h 19min ago
 Main PID: 1564 (atd)
   CGroup: /system.slice/atd.service
           └─1564 /usr/sbin/atd -f

Jun 19 14:54:14 localhost.localdomain systemd[1]: Started Job spooling tools.
Jun 19 14:54:14 localhost.localdomain systemd[1]: Starting Job spooling tools...
```
##### 根据上述命令的输出我们可以看到 atd service 是处于active状态，也就是我们通常说的 atd service 正在运行中。
##### 2.2 启动 atd 服务.
```
[root@test3 ~]# systemctl start atd.service    
[root@test3 ~]# systemctl status atd.service   
● atd.service - Job spooling tools
   Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2016-06-19 19:30:52 CST; 5s ago
 Main PID: 3906 (atd)
   CGroup: /system.slice/atd.service
           └─3906 /usr/sbin/atd -f

Jun 19 19:30:52 test3 systemd[1]: Started Job spooling tools.
Jun 19 19:30:52 test3 systemd[1]: Starting Job spooling tools...
```
##### 2.3 关闭 atd 服务.
```
[root@test3 ~]# systemctl status atd.service 
● atd.service - Job spooling tools
   Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Sun 2016-06-19 19:29:03 CST; 5s ago
  Process: 1554 ExecStart=/usr/sbin/atd -f $OPTS (code=exited, status=0/SUCCESS)
 Main PID: 1554 (code=exited, status=0/SUCCESS)

Jun 19 19:18:20 localhost.localdomain systemd[1]: Started Job spooling tools.
Jun 19 19:18:20 localhost.localdomain systemd[1]: Starting Job spooling tools...
Jun 19 19:28:49 test3 systemd[1]: Started Job spooling tools.
Jun 19 19:29:03 test3 systemd[1]: Stopping Job spooling tools...
Jun 19 19:29:03 test3 systemd[1]: Stopped Job spooling tools.
```
##### 2.4 重启 atd 服务.
```
[root@test3 ~]# systemctl restart atd.service       
[root@test3 ~]# systemctl status atd.service  
● atd.service - Job spooling tools
   Loaded: loaded (/usr/lib/systemd/system/atd.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2016-06-19 19:31:29 CST; 3s ago
 Main PID: 4047 (atd)
   CGroup: /system.slice/atd.service
           └─4047 /usr/sbin/atd -f

Jun 19 19:31:29 test3 systemd[1]: Started Job spooling tools.
Jun 19 19:31:29 test3 systemd[1]: Starting Job spooling tools...
```
##### 通过执行上述三条命令，我们可以很清楚的看到，在 start、stop、restart atd service 时，命令行均无输出，但是我们可以配合查看 status 参数来查看 atd service 的运行情况以及启动的相关信息，诸如 PID、服务路径等等。

##### 3.  at 工具的使用
##### 3.1 at 命令常用参数
```
[root@test3 ~]# at [-mldV] TIME
[root@test3 ~]# at -c jobid
选项与参数：
-m      发信通知用户计划任务完成，即使没有输出。
-l      at -l 相当于 atq ，列出目前系统中所有该用户的 at 任务。
-d      at -d 相当于 atrm ，删除一个尚未执行的 at 任务。
-V      显示当前 at 的版本。
-c      根据 jobid 列出实际执行的指令内容。
TIME    定义什么时间段执行具体的 at 任务，常见格式有：
        HH:MM                                               sample> 04:00
            在今日 HH:MM 时刻执行 at 任务，若当前时间大于定义时间，则转为明日的 HH:MM 时刻执行。
        HH:MM YYYY-MM-DD                                    sample> 04:00 2016-03-17
            在某年某月的某一天的某时刻执行 at 任务。
        HH:MM[am|pm] [Month] [Date]                         sample> 04pm March 17
            在某年某月某日的某时刻执行 at 任务。
        HH:MM[am|pm] + number [minutes|hours|days|weeks]    sample> now + 5 minutes
            在某个时间点后的多少分/时/日/周执行 at 任务。
```
##### 3.2 使用 at 工具
```
[root@test3 ~]# at now + 2 minutes
at> df -h > test   
at> <EOT>
job 2 at Sun Jun 19 20:19:00 2016
[root@test3 ~]# ll /var/spool/at/
total 4
-rwx------. 1 root   root   2785 Jun 19 20:17 a000020174e8c3
drwx------. 2 daemon daemon    6 Jun 19 19:57 spool
[root@test3 ~]# atq 
2       Sun Jun 19 20:19:00 2016 a root
```
##### 以上命令创建了一个2分钟后执行的 at 任务，任务 id 为2，任务内容时，显示当前磁盘用量并写入到当前目录下的 test 文件中；第二条命令是查看该任务在 at 工具中实际存放的位置；第三条命令则是查看 at 任务列表中未执行的命令，并指明该任务的属主。
```
[root@test3 ~]# at now + 2 minutes
at> df -h > test
at> <EOT>
job 4 at Sun Jun 19 20:22:00 2016
[root@test3 ~]# atq 
4       Sun Jun 19 20:22:00 2016 a root
[root@test3 ~]# atrm 4
[root@test3 ~]# atq
```
##### 第一条我们创建了跟 id 为2相同的 at 计划任务，此时新建任务的 id 为4；接着我们使用 atrm 命令成功删除 at 任务列表里的该条任务。
##### **备注：由于 at 命令的的特殊性，我们在输入的时候是无法正常删除之前写的命令内容的，这会带来很大的不便，但是我们可以使用 ctrl + delte 来删除写错的部分；<EOT> 为 ctrl + d 的输出。** 

##### 4. bacth 扩展
##### batch 命令被用来在系统平均负载达到 0.8 以下时执行一次性的任务，用法与at一样。
```
[root@test3 ~]# batch 
at> date && free -m<EOT>
job 9 at Tue Jun 21 21:21:00 2016
[root@test3 ~]# tail -n5 /var/spool/mail/root
Tue Jun 21 21:22:19 CST 2016
              total        used        free      shared  buff/cache   available
Mem:            977         591          74           6         312         154
Swap:          1023           1        1022
```
##### 上边的命令中我们新建了一个立即执行的计划任务，即在成功输出当前时间后显示系统的内存使用情况。由于我们是 root 用户，所以我们在系统发给 root 的 mail 列表里可以成功检索到我们的执行结果。
##### 5. at 以及 batch 工具的安全增强
##### 总所周知， Red Hat 是多用户、多任务的操作系统，介于自动化工具会带来很大危险，这时候我们需要对某些用户的权限做出限制，以防出现误删系统文件导致系统崩溃。当然，有些时候我们可以恢复回来，但是这也给我们增加了额外的工作量，作为一个 Linux SA，我们需要更聪明的工作。
##### 这时 /etc/at.allow 与 /etc/at.deny 这两个 at 工具使用权限控制文件的出现给我们带来了福音。
##### /etc/at.allow ： 系统用户在创建 at 任务时， at 工具会首先去检索这个文件，只有该文件中存在的用户，则可以使用 at 工具（root用户除外）；若是不存在该文件，则所有除了 root 的用户均不可以使用 at 工具。
##### /etc/at.deny ： 系统用户在创建 at 任务时，在不存在 /etc/at.allow 文件的情况下，系统会检索 /etc/at.deny 文件，所有在文件上用户均不可以使用 at 工具，以及所有不在该文件内的用户均可使用 at 工具。
### **10.2 crontab 工具**

# **11. SSH**

### **11.1 sshd**

### **11.2 无密码登录**