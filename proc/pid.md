# proc
## [pid]
### 1. attr（Problem）
目录文件。记录进程的安全属性，可以读取or设置（SELinux）。只有在内核中配置了CONFIG_SECURITY参数后才能提供。
#### 1.1 current
当前进程的安全属性
#### 1.2 exec
随后的execve调用的安全属性
#### 1.3 fscreate
被进程调用open、mkdir、symlink、mknod创建出来的文件的安全属性
#### 1.4 keycreate

???

#### 1.5 prev
上次执行前的上下文，即调用此进程的上下文
#### 1.6 sockcreate
被这个进程创建的socket使用的上下文

### 2. autogroup
记录当前进程的自动进程组信息。执行setsid系统调用时创建新的进程组；当其创建子进程时，子进程也属于此进程组；当进程组中最后一个进程退出时，自动进程组随之销毁。
[index : kernel/git/torvalds/linux.git（autogroup commit信息）](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=5091faa449ee0b7d73bc296a93bca9540fc51d0a)
```
#(example)
#cat /proc/1/autogroup
/autogroup-2 nice 0
```

### 3. auxv
二进制信息。记录ELF文件的解释信息，是启动动态链接器从而执行程序的必要信息。具体参数值可在```/usr/include/elf.h```中找到对照信息以及具体含义

### 4. cgroup
记录当前进程属于哪些子系统，子系统是cgroup的资源控制系统。cgroup用来对一组进程所占用的资源做限制、统计、隔离。
以下参数的解释来自于Docker
```
#(example)
#cat /proc/1/cgroup
10:hugetlb:/user.slice/user-1000.slice/session-c2.scope //hugetlb大页使用配置控制
9:memory:/init.scope        //可以设定cgroup中任务对内存使用量的限定，并且自动生成这些任务对内存资源使用情况的报告
8:cpuset:/        //可以为cgroup中的任务分配独立的CPU（针对多处理器系统）和内存
7:pids:/init.scope
6:net_cls:/        //通过使用等级识别符标记网络数据包，从而允许Linux流量控制程序识别从具体cgroup中生成的数据包
5:freezer:/        //可以挂起或回复cgroup中的任务
4:blkio:/init.scope        //可以为块设备设定I/O限制
3:cpu,cpuacct:/init.scope        //cpu用于使用调度程序控制任务对CPU的使用；cpuacct用于自动生成cgroup中任务对CPU资源使用情况的报告
2:devices:/init.scope        //用于开启or关闭任务对设备的访问
1:name=systemd:/init.scope
```

### 5. clear_refs
此文件只能由进程的所有者进行写入操作，不能被读取。
当一个非零数写入其中时，会清除相应的进程的所有的PG_referenced和PAGE_ACCESSED标志。可以用来计算进程使用了多少内存。

### 6. cmdline
记录命令行参数
```
#(example)
#cat /proc/cmdline
/sbin/init
```

### 7. comm
记录启动此进程的可执行文件名称
```
#(example)
#cat /proc/1/comm
systemd
```

### 8. coredump_filter
指定当前进程出现coredump时要转储的内存段（另外，系统默认不产生coredump文件，因此当需要的时候要打开core文件的限制）。
文件内容是一个32位的16进制数，通过低7位来指定七个内存段的转储与否：
+ bit 0 ：anonymous private memory（匿名私有内存段）    
+ bit 1 ：anonymous shared memory（匿名共享内存段）    
+ bit 2 ：file-backed private memory（file-backed 私有内存段）    
+ bit 3 ：file-backed shared memory（file-bakced 共享内存段）    
+ bit 4 ：ELF header pages in file-backed private memory areas (it iseffective only if the bit 2 is cleared)（ELF 文件映射，只有在bit 2 复位的时候才起作用）    
+ bit 5 ：hugetlb private memory（大页面私有内存）    
+ bit 6 ：hugetlb shared memory（大页面共享内存）    

```
#(example)
#cat /proc/1/coredump_filter
00000033

#将其换算为二进制：00110011
#根据上面的bit描述来看，当出现coredump需要转储匿名私有内存段、匿名共享内存段、大页面私有内存段 以及 大页面共享内存段
```


### 9. cpuset（Problem）
confine processes to processor and memory node subsets，即基于cgroup的cpu使用限制配置
```
#(example)
#cat /proc/1/cpuset
/  
```

### 10. cwd
软链接文件，指向了进程当前的工作目录
```
#(example)
#ls -l /proc/1/cwd
lrwxrwxrwx 1 root root 0 Jan  8 05:51 cwd -> /
```

### 11. environ
记录此进程的环境变量内容
```
#(example)
#cat /proc/1/environ
TERM=linux
```

### 12. exe
软链接文件，指向了正在执行文件
```

```

### 13. fd
是一个目录。记录了进程打开的所有文件，每个文件以其文件描述符号码为名称的软链接表示，这些软链接指明了具体的文件，pipe,  device, socket等的位置
```
#(example)
#ls -l /proc/1/fd
total 0
lrwx------ 1 root root 64 Jan  8 05:47 0 -> /dev/null
lrwx------ 1 root root 64 Jan  8 05:47 1 -> /dev/null
lr-x------ 1 root root 64 Jan  8 05:53 10 -> /proc/1/mountinfo
lr-x------ 1 root root 64 Jan  8 05:53 11 -> anon_inode:inotify
lr-x------ 1 root root 64 Jan  8 05:53 12 -> /proc/swaps
lrwx------ 1 root root 64 Jan  8 05:53 13 -> socket:[9910]
..．
lrwx------ 1 root root 64 Jan  8 05:53 34 -> socket:[9938]
lrwx------ 1 root root 64 Jan  8 05:53 35 -> anon_inode:[timerfd]
lr-x------ 1 root root 64 Jan  8 05:53 36 -> /dev/autofs
lr-x------ 1 root root 64 Jan  8 05:53 37 -> pipe:[9973]
lrwx------ 1 root root 64 Jan  8 05:53 38 -> socket:[11334]
lrwx------ 1 root root 64 Jan  8 05:53 39 -> anon_inode:[timerfd]
lrwx------ 1 root root 64 Jan  8 05:53 4 -> anon_inode:[eventpoll]
...
lrwx------ 1 root root 64 Jan  8 05:53 9 -> anon_inode:[eventpoll]
```

### 14. fdinfo
目录文件，已打开文件的对应信息，具体各参数如下：
+ pos    
  文件偏移量
+ flags    
  对应file结构体中的f_flags字段，表示文件的访问权限
+ mnt_id    
  文件描述符号码

```
#(example)
#ls -l /proc/1/fdinfo
total 0
-r-------- 1 root root 0 Jan  8 05:57 0
-r-------- 1 root root 0 Jan  8 05:57 1
-r-------- 1 root root 0 Jan  8 05:57 10
...
-r-------- 1 root root 0 Jan  8 05:57 9

#(example)
#cat /proc/1/fdinfo/8
pos:    0
flags:    02004002
mnt_id:    8
```

### 15. io
读写字节数目以及读写系统调用次数。
```
#(example)
#cat /proc/1/io
rchar: 50720173        //读出的总字节数，read或pread的长度总和
wchar: 2260304522        //写入的粽子节数，write或pwrite的长度总和
syscr: 52300        //read或pread的总调用次数
syscw: 552278        //write或pwrite的总调用次数
read_bytes: 109913088        //实际从磁盘中读取的字节数
write_bytes: 2237005824        //实际写入到磁盘中的字节数
cancelled_write_bytes: 6557696        //由于截断pagecache导致应该发生而没有发生的写入字节数（可能为负数）
```

### 16.limits
记录此进程的运行环境的各项限制值，包括硬限制、软限制等
```
#(example)
#cat /proc/1/limits
Limit(限制名称)          Soft Limit(软限制)   Hard Limit(硬限制)    Units(单位)     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            8388608              unlimited            bytes     
Max core file size        unlimited            unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             15716                15716                processes 
Max open files            65536                65536                files     
Max locked memory         65536                65536                bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       15716                15716                signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us 
```

### 17. map_files
目录文件。内容为以虚拟内存地址范围命名的软链接文件，指明了可执行文件和所加载的所有库文件
```
#(example)
#ls -l /proc/1/map_files
total 0
lr-------- 1 root root 64 Jan  8 06:07 5635f4a42000-5635f4b2a000 -> /usr/lib/systemd/systemd
lr-------- 1 root root 64 Jan  8 06:07 5635f4b2b000-5635f4b50000 -> /usr/lib/systemd/systemd
lr-------- 1 root root 64 Jan  8 06:07 5635f4b50000-5635f4b51000 -> /usr/lib/systemd/systemd
lr-------- 1 root root 64 Jan  8 06:07 7fa7e9798000-7fa7e979c000 -> /usr/lib/libattr.so.1.1.0
...
lr-------- 1 root root 64 Jan  8 06:07 7fa7ec947000-7fa7ec96a000 -> /usr/lib/ld-2.24.so
lr-------- 1 root root 64 Jan  8 06:07 7fa7ecb69000-7fa7ecb6a000 -> /usr/lib/ld-2.24.so
lr-------- 1 root root 64 Jan  8 06:07 7fa7ecb6a000-7fa7ecb6b000 -> /usr/lib/ld-2.24.so

```

### 18. maps
以文本形式描述的进程运行内存内存图，与当前进程有关联的所有可执行文件和库文件在虚拟内存地址中的映射区域、访问权限等参数形成的列表。
具体参数如下：	
+ address        
  进程占用的地址空间    
+ perms        
  访问权限集    
  r=read; w=write; x=execute; s=shared; p=private（copy on write）    
+ offset        
  文件偏移量    
+ dev        
  设备号（主设备号：次设备号）    
+ inode        
  文件具体的inode号。    
  0表示没有inode关联互内存区域
```
#(example)
#cat /proc/1/maps
#address                  perms  offset  dev  inode
559235185000-55923526d000 r-xp 00000000 08:01 799480                     /usr/lib/systemd/systemd
55923526e000-559235293000 r--p 000e8000 08:01 799480                     /usr/lib/systemd/systemd
559235293000-559235294000 rw-p 0010d000 08:01 799480                     /usr/lib/systemd/systemd
...
7fc1fcdeb000-7fc1fcdec000 r--p 00022000 08:01 789676                     /usr/lib/ld-2.24.so
7fc1fcdec000-7fc1fcded000 rw-p 00023000 08:01 789676                     /usr/lib/ld-2.24.so
7fc1fcded000-7fc1fcdee000 rw-p 00000000 00:00 0 
7ffc5f05c000-7ffc5f07d000 rw-p 00000000 00:00 0                          [stack]
7ffc5f148000-7ffc5f14a000 r--p 00000000 00:00 0                          [vvar]
7ffc5f14a000-7ffc5f14c000 r-xp 00000000 00:00 0                          [vdso]
ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                  [vsyscall]
```

### 19. mem（Problem）
进程虚拟内存，在进行I/O操作前需要调用lseek()移至有效偏移量
```
#(example)
#cat /proc/1/mem
Input/output error
```

### 20. mountinfo
文件系统的挂载信息，具体各项参数如下：
+ (1) mount ID     
  挂载的唯一标识符
+ (2) parent ID     
  父挂载标识符
+ (3) major:minor         
  设备号（主：次）
+ (4) root    
  挂载的文件系统的根目录
+ (5) mount point    
  挂载点
+ (6) mount options    
  挂载参数
+ (7) optional fields    
  不定数量的形式为tag[:value]格式的信息
+ (8) separator
  标记optional fields项结束
+ (9) filesystem type    
  文件系统名称，以type[.subtype]的形式记录
+ (10) mount source    
  文件系统的信息
+ (11) super options    
  对于super block（超级块）的参数
```
#(example)
#cat /proc/1/mountinfo
#1 2  3   4 5     6                               7        8 9    10   11
16 20 0:4 / /proc rw,nosuid,nodev,noexec,relatime shared:5 - proc proc rw
17 20 0:16 / /sys rw,nosuid,nodev,noexec,relatime shared:6 - sysfs sys rw
18 20 0:6 / /dev rw,nosuid,relatime shared:2 - devtmpfs dev rw,size=2011756k,nr_inodes=502939,mode=755
19 20 0:17 / /run rw,nosuid,nodev,relatime shared:11 - tmpfs run rw,mode=755
20 0 8:1 / / rw,relatime shared:1 - ext4 /dev/sda1 rw,data=ordered
21 17 0:18 / /sys/kernel/security rw,nosuid,nodev,noexec,relatime shared:7 - securityfs securityfs rw
22 18 0:19 / /dev/shm rw,nosuid,nodev shared:3 - tmpfs tmpfs rw
23 18 0:20 / /dev/pts rw,nosuid,noexec,relatime shared:4 - devpts devpts rw,gid=5,mode=620,ptmxmode=000
24 17 0:21 / /sys/fs/cgroup ro,nosuid,nodev,noexec shared:8 - tmpfs tmpfs ro,mode=755
25 24 0:22 / /sys/fs/cgroup/systemd rw,nosuid,nodev,noexec,relatime shared:9 - cgroup cgroup rw,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd
26 17 0:23 / /sys/fs/pstore rw,nosuid,nodev,noexec,relatime shared:10 - pstore pstore rw
27 24 0:24 / /sys/fs/cgroup/devices rw,nosuid,nodev,noexec,relatime shared:12 - cgroup cgroup rw,devices
28 24 0:25 / /sys/fs/cgroup/cpu,cpuacct rw,nosuid,nodev,noexec,relatime shared:13 - cgroup cgroup rw,cpu,cpuacct
29 24 0:26 / /sys/fs/cgroup/blkio rw,nosuid,nodev,noexec,relatime shared:14 - cgroup cgroup rw,blkio
30 24 0:27 / /sys/fs/cgroup/freezer rw,nosuid,nodev,noexec,relatime shared:15 - cgroup cgroup rw,freezer
31 24 0:28 / /sys/fs/cgroup/net_cls rw,nosuid,nodev,noexec,relatime shared:16 - cgroup cgroup rw,net_cls
32 24 0:29 / /sys/fs/cgroup/pids rw,nosuid,nodev,noexec,relatime shared:17 - cgroup cgroup rw,pids
33 24 0:30 / /sys/fs/cgroup/cpuset rw,nosuid,nodev,noexec,relatime shared:18 - cgroup cgroup rw,cpuset
34 24 0:31 / /sys/fs/cgroup/memory rw,nosuid,nodev,noexec,relatime shared:19 - cgroup cgroup rw,memory
35 16 0:32 / /proc/sys/fs/binfmt_misc rw,relatime shared:20 - autofs systemd-1 rw,fd=28,pgrp=1,timeout=0,minproto=5,maxproto=5,direct
37 18 0:15 / /dev/mqueue rw,relatime shared:21 - mqueue mqueue rw
36 18 0:33 / /dev/hugepages rw,relatime shared:22 - hugetlbfs hugetlbfs rw
38 20 0:34 / /tmp rw,nosuid,nodev shared:23 - tmpfs tmpfs rw
39 17 0:7 / /sys/kernel/debug rw,relatime shared:24 - debugfs debugfs rw
40 17 0:35 / /sys/kernel/config rw,relatime shared:25 - configfs configfs rw
68 19 0:37 / /run/vmblock-fuse rw,nosuid,nodev,relatime shared:26 - fuse.vmware-vmblock vmware-vmblock rw,user_id=0,group_id=0,default_permissions,allow_other
70 17 0:38 / /sys/fs/fuse/connections rw,relatime shared:27 - fusectl fusectl rw
72 19 0:39 / /run/user/0 rw,nosuid,nodev,relatime shared:28 - tmpfs tmpfs rw,size=403260k,mode=700
74 20 0:40 / /mnt/hgfs rw,nosuid,nodev,relatime shared:29 - fuse.vmhgfs-fuse vmhgfs-fuse rw,user_id=0,group_id=0,allow_other
```

### 21. mounts
当前进程的文件系统挂载信息
```
#(example)
#cat /proc/1/mounts
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
sys /sys sysfs rw,nosuid,nodev,noexec,relatime 0 0
dev /dev devtmpfs rw,nosuid,relatime,size=2011756k,nr_inodes=502939,mode=755 0 0
run /run tmpfs rw,nosuid,nodev,relatime,mode=755 0 0
/dev/sda1 / ext4 rw,relatime,data=ordered 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev/shm tmpfs rw,nosuid,nodev 0 0
devpts /dev/pts devpts rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
tmpfs /sys/fs/cgroup tmpfs ro,nosuid,nodev,noexec,mode=755 0 0
cgroup /sys/fs/cgroup/systemd cgroup rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd 0 0
pstore /sys/fs/pstore pstore rw,nosuid,nodev,noexec,relatime 0 0
cgroup /sys/fs/cgroup/devices cgroup rw,nosuid,nodev,noexec,relatime,devices 0 0
cgroup /sys/fs/cgroup/cpu,cpuacct cgroup rw,nosuid,nodev,noexec,relatime,cpu,cpuacct 0 0
cgroup /sys/fs/cgroup/blkio cgroup rw,nosuid,nodev,noexec,relatime,blkio 0 0
cgroup /sys/fs/cgroup/freezer cgroup rw,nosuid,nodev,noexec,relatime,freezer 0 0
cgroup /sys/fs/cgroup/net_cls cgroup rw,nosuid,nodev,noexec,relatime,net_cls 0 0
cgroup /sys/fs/cgroup/pids cgroup rw,nosuid,nodev,noexec,relatime,pids 0 0
cgroup /sys/fs/cgroup/cpuset cgroup rw,nosuid,nodev,noexec,relatime,cpuset 0 0
cgroup /sys/fs/cgroup/memory cgroup rw,nosuid,nodev,noexec,relatime,memory 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=28,pgrp=1,timeout=0,minproto=5,maxproto=5,direct 0 0
mqueue /dev/mqueue mqueue rw,relatime 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,relatime 0 0
tmpfs /tmp tmpfs rw,nosuid,nodev 0 0
debugfs /sys/kernel/debug debugfs rw,relatime 0 0
configfs /sys/kernel/config configfs rw,relatime 0 0
vmware-vmblock /run/vmblock-fuse fuse.vmware-vmblock rw,nosuid,nodev,relatime,user_id=0,group_id=0,default_permissions,allow_other 0 0
fusectl /sys/fs/fuse/connections fusectl rw,relatime 0 0
tmpfs /run/user/0 tmpfs rw,nosuid,nodev,relatime,size=403260k,mode=700 0 0
vmhgfs-fuse /mnt/hgfs fuse.vmhgfs-fuse rw,nosuid,nodev,relatime,user_id=0,group_id=0,allow_other 0 0
```

### 22. mountstats
当前进程挂载信息记录，具体参数信息如下：
+ (1) 挂载设备名    
+ (2) 挂载点    
+ (3) 文件系统种类    
+ (4) 选项和配置信息    

```
#(example)
#cat /proc/1/mountstats
#      （1）            （2）      （3）  （4）
device proc mounted on /proc with fstype proc
device sys mounted on /sys with fstype sysfs
device dev mounted on /dev with fstype devtmpfs
device run mounted on /run with fstype tmpfs
device /dev/sda1 mounted on / with fstype ext4
device securityfs mounted on /sys/kernel/security with fstype securityfs
device tmpfs mounted on /dev/shm with fstype tmpfs
device devpts mounted on /dev/pts with fstype devpts
device tmpfs mounted on /sys/fs/cgroup with fstype tmpfs
device cgroup mounted on /sys/fs/cgroup/systemd with fstype cgroup
device pstore mounted on /sys/fs/pstore with fstype pstore
device cgroup mounted on /sys/fs/cgroup/devices with fstype cgroup
device cgroup mounted on /sys/fs/cgroup/cpu,cpuacct with fstype cgroup
device cgroup mounted on /sys/fs/cgroup/blkio with fstype cgroup
device cgroup mounted on /sys/fs/cgroup/freezer with fstype cgroup
device cgroup mounted on /sys/fs/cgroup/net_cls with fstype cgroup
device cgroup mounted on /sys/fs/cgroup/pids with fstype cgroup
device cgroup mounted on /sys/fs/cgroup/cpuset with fstype cgroup
device cgroup mounted on /sys/fs/cgroup/memory with fstype cgroup
device systemd-1 mounted on /proc/sys/fs/binfmt_misc with fstype autofs
device mqueue mounted on /dev/mqueue with fstype mqueue
device hugetlbfs mounted on /dev/hugepages with fstype hugetlbfs
device tmpfs mounted on /tmp with fstype tmpfs
device debugfs mounted on /sys/kernel/debug with fstype debugfs
device configfs mounted on /sys/kernel/config with fstype configfs
device vmware-vmblock mounted on /run/vmblock-fuse with fstype fuse.vmware-vmblock
device fusectl mounted on /sys/fs/fuse/connections with fstype fusectl
device tmpfs mounted on /run/user/0 with fstype tmpfs
device vmhgfs-fuse mounted on /mnt/hgfs with fstype fuse.vmhgfs-fuse
```

### 23. net（Problem）
目录文件。进程网络相关信息

TODO：net目录下各文件

### 24. ns
目录文件，记录当前进程所属的namespace
```
#(example)
#ls -l /proc/1/ns
total 0
lrwxrwxrwx 1 root root 0 Jan  8 04:53 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 root root 0 Jan  8 04:53 ipc -> ipc:[4026531839]
lrwxrwxrwx 1 root root 0 Jan  8 04:53 mnt -> mnt:[4026531840]
lrwxrwxrwx 1 root root 0 Jan  8 04:53 net -> net:[4026531957]
lrwxrwxrwx 1 root root 0 Jan  8 04:53 pid -> pid:[4026531836]
lrwxrwxrwx 1 root root 0 Jan  8 04:53 uts -> uts:[4026531838]
```

### 25. numa_maps
NUMA的内存映射，记录了进程的内存区域正在被哪一个节点使用的信息。具体参数信息如下：
+ (1) 起始地址    
+ (2) 对于当前内存区域的memory policy（用于numa架构中）    
  Memory policy：In the Linux kernel, "memory policy" determines from which node the kernel will allocate memory in a NUMA system or in an emulated NUMA system.  Linux has supported platforms with Non-Uniform Memory Access architectur.
+ (3) 不固定的信息    
  N<node>=<pages>  node使用了多少个page    
  file=<filename>  内存映射的文件名    
  heap  表示内存区域用于堆    
  stack  表示内存区域用于堆栈    
  huge  大内存区域    
  anon=<pages>  范围内的匿名page数    
  dirty=<pages>  “脏页”数    
  mapped=<pages>  已映射page数(只有在数目与anon、dirty数目不同时才会显示)    
  mapmax=<count>  最大映射数    
  swapcache=<count>  被分配给交换设备的page数    
  active=<pages>  活页数    
  writeback=<pages>  当前写入到磁盘中的page数    

```
#(example)
#cat /proc/1/numa_maps
#(1)         (2)     (3)
559235185000 default file=/usr/lib/systemd/systemd mapped=218 mapmax=3 active=142 N0=218 kernelpagesize_kB=4
55923526e000 default file=/usr/lib/systemd/systemd anon=37 dirty=37 mapmax=2 N0=37 kernelpagesize_kB=4
559235293000 default file=/usr/lib/systemd/systemd anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
559235a81000 default heap anon=148 dirty=148 mapmax=2 N0=148 kernelpagesize_kB=4
7fc1f4000000 default anon=3 dirty=3 N0=3 kernelpagesize_kB=4
7fc1f4029000 default
7fc1f8a18000 default
7fc1f8a19000 default anon=2 dirty=2 N0=2 kernelpagesize_kB=4
7fc1f9219000 default
7fc1f921a000 default anon=2 dirty=2 N0=2 kernelpagesize_kB=4
7fc1f9a1a000 default file=/usr/lib/libattr.so.1.1.0 mapped=4 mapmax=6 N0=4 kernelpagesize_kB=4
7fc1f9a1e000 default file=/usr/lib/libattr.so.1.1.0
7fc1f9c1d000 default file=/usr/lib/libattr.so.1.1.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1f9c1e000 default file=/usr/lib/libattr.so.1.1.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1f9c1f000 default file=/usr/lib/libuuid.so.1.3.0 mapped=4 mapmax=22 N0=4 kernelpagesize_kB=4
7fc1f9c23000 default file=/usr/lib/libuuid.so.1.3.0
7fc1f9e22000 default file=/usr/lib/libuuid.so.1.3.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1f9e23000 default file=/usr/lib/libuuid.so.1.3.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1f9e24000 default file=/usr/lib/libblkid.so.1.1.0 mapped=28 mapmax=22 N0=28 kernelpagesize_kB=4
7fc1f9e65000 default file=/usr/lib/libblkid.so.1.1.0
7fc1fa064000 default file=/usr/lib/libblkid.so.1.1.0 anon=4 dirty=4 mapmax=2 N0=4 kernelpagesize_kB=4
7fc1fa068000 default file=/usr/lib/libblkid.so.1.1.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa069000 default
7fc1fa06a000 default file=/usr/lib/libz.so.1.2.8 mapped=20 mapmax=28 N0=20 kernelpagesize_kB=4
7fc1fa07f000 default file=/usr/lib/libz.so.1.2.8
7fc1fa27e000 default file=/usr/lib/libz.so.1.2.8 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa27f000 default file=/usr/lib/libz.so.1.2.8 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa280000 default file=/usr/lib/libdl-2.24.so mapped=2 mapmax=34 N0=2 kernelpagesize_kB=4
7fc1fa282000 default file=/usr/lib/libdl-2.24.so
7fc1fa482000 default file=/usr/lib/libdl-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa483000 default file=/usr/lib/libdl-2.24.so anon=1 dirty=1 N0=1 kernelpagesize_kB=4
7fc1fa484000 default file=/usr/lib/libidn.so.11.6.16 mapped=4 mapmax=5 N0=4 kernelpagesize_kB=4
7fc1fa4b6000 default file=/usr/lib/libidn.so.11.6.16
7fc1fa6b6000 default file=/usr/lib/libidn.so.11.6.16 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa6b7000 default file=/usr/lib/libidn.so.11.6.16 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa6b8000 default file=/usr/lib/libacl.so.1.1.0 mapped=8 mapmax=6 N0=8 kernelpagesize_kB=4
7fc1fa6c0000 default file=/usr/lib/libacl.so.1.1.0
7fc1fa8bf000 default file=/usr/lib/libacl.so.1.1.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa8c0000 default file=/usr/lib/libacl.so.1.1.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fa8c1000 default file=/usr/lib/libgpg-error.so.0.21.0 mapped=16 mapmax=28 N0=16 kernelpagesize_kB=4
7fc1fa8d4000 default file=/usr/lib/libgpg-error.so.0.21.0
7fc1faad3000 default file=/usr/lib/libgpg-error.so.0.21.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1faad4000 default file=/usr/lib/libgpg-error.so.0.21.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1faad5000 default file=/usr/lib/libgcrypt.so.20.1.5 mapped=16 mapmax=28 N0=16 kernelpagesize_kB=4
7fc1fabdc000 default file=/usr/lib/libgcrypt.so.20.1.5
7fc1faddb000 default file=/usr/lib/libgcrypt.so.20.1.5 anon=2 dirty=2 mapmax=2 N0=2 kernelpagesize_kB=4
7fc1faddd000 default file=/usr/lib/libgcrypt.so.20.1.5 anon=7 dirty=7 mapmax=2 N0=7 kernelpagesize_kB=4
7fc1fade4000 default file=/usr/lib/liblz4.so.1.7.4 mapped=16 mapmax=27 N0=16 kernelpagesize_kB=4
7fc1fadf5000 default file=/usr/lib/liblz4.so.1.7.4
7fc1faff4000 default file=/usr/lib/liblz4.so.1.7.4 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1faff5000 default file=/usr/lib/liblz4.so.1.7.4 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1faff6000 default file=/usr/lib/liblzma.so.5.2.3 mapped=4 mapmax=29 N0=4 kernelpagesize_kB=4
7fc1fb01b000 default file=/usr/lib/liblzma.so.5.2.3
7fc1fb21a000 default file=/usr/lib/liblzma.so.5.2.3 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fb21b000 default file=/usr/lib/liblzma.so.5.2.3 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fb21c000 default file=/usr/lib/libresolv-2.24.so mapped=16 mapmax=30 N0=16 kernelpagesize_kB=4
7fc1fb230000 default file=/usr/lib/libresolv-2.24.so
7fc1fb42f000 default file=/usr/lib/libresolv-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fb430000 default file=/usr/lib/libresolv-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fb431000 default
7fc1fb433000 default file=/usr/lib/libm-2.24.so mapped=45 mapmax=30 N0=45 kernelpagesize_kB=4
7fc1fb536000 default file=/usr/lib/libm-2.24.so
7fc1fb735000 default file=/usr/lib/libm-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fb736000 default file=/usr/lib/libm-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fb737000 default file=/usr/lib/libcap.so.2.25 mapped=4 mapmax=29 N0=4 kernelpagesize_kB=4
7fc1fb73b000 default file=/usr/lib/libcap.so.2.25
7fc1fb93a000 default file=/usr/lib/libcap.so.2.25 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fb93b000 default file=/usr/lib/libc-2.24.so mapped=333 mapmax=45 N0=333 kernelpagesize_kB=4
7fc1fbad0000 default file=/usr/lib/libc-2.24.so
7fc1fbccf000 default file=/usr/lib/libc-2.24.so anon=4 dirty=4 mapmax=2 N0=4 kernelpagesize_kB=4
7fc1fbcd3000 default file=/usr/lib/libc-2.24.so anon=2 dirty=2 N0=2 kernelpagesize_kB=4
7fc1fbcd5000 default anon=3 dirty=3 N0=3 kernelpagesize_kB=4
7fc1fbcd9000 default file=/usr/lib/libpthread-2.24.so mapped=23 mapmax=39 N0=23 kernelpagesize_kB=4
7fc1fbcf1000 default file=/usr/lib/libpthread-2.24.so
7fc1fbef0000 default file=/usr/lib/libpthread-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fbef1000 default file=/usr/lib/libpthread-2.24.so anon=1 dirty=1 N0=1 kernelpagesize_kB=4
7fc1fbef2000 default anon=1 dirty=1 N0=1 kernelpagesize_kB=4
7fc1fbef6000 default file=/usr/lib/libmount.so.1.1.0 mapped=73 mapmax=21 N0=73 kernelpagesize_kB=4
7fc1fbf40000 default file=/usr/lib/libmount.so.1.1.0
7fc1fc140000 default file=/usr/lib/libmount.so.1.1.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc141000 default file=/usr/lib/libmount.so.1.1.0 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc142000 default
7fc1fc144000 default file=/usr/lib/libkmod.so.2.3.1 mapped=21 mapmax=3 N0=21 kernelpagesize_kB=4
7fc1fc159000 default file=/usr/lib/libkmod.so.2.3.1
7fc1fc358000 default file=/usr/lib/libkmod.so.2.3.1 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc359000 default file=/usr/lib/libkmod.so.2.3.1 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc35a000 default file=/usr/lib/libpam.so.0.84.2 mapped=13 mapmax=3 N0=13 kernelpagesize_kB=4
7fc1fc367000 default file=/usr/lib/libpam.so.0.84.2
7fc1fc566000 default file=/usr/lib/libpam.so.0.84.2 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc567000 default file=/usr/lib/libpam.so.0.84.2 anon=1 dirty=1 N0=1 kernelpagesize_kB=4
7fc1fc568000 default file=/usr/lib/libseccomp.so.2.3.1 mapped=44 mapmax=5 N0=44 kernelpagesize_kB=4
7fc1fc594000 default file=/usr/lib/libseccomp.so.2.3.1
7fc1fc793000 default file=/usr/lib/libseccomp.so.2.3.1 anon=21 dirty=21 mapmax=2 N0=21 kernelpagesize_kB=4
7fc1fc7a8000 default file=/usr/lib/libseccomp.so.2.3.1 anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc7a9000 default file=/usr/lib/librt-2.24.so mapped=7 mapmax=31 N0=7 kernelpagesize_kB=4
7fc1fc7b0000 default file=/usr/lib/librt-2.24.so
7fc1fc9af000 default file=/usr/lib/librt-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc9b0000 default file=/usr/lib/librt-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fc9b1000 default file=/usr/lib/systemd/libsystemd-shared-232.so mapped=347 mapmax=5 N0=347 kernelpagesize_kB=4
7fc1fcb3d000 default file=/usr/lib/systemd/libsystemd-shared-232.so anon=13 dirty=13 mapmax=2 N0=13 kernelpagesize_kB=4
7fc1fcbc7000 default file=/usr/lib/systemd/libsystemd-shared-232.so anon=1 dirty=1 N0=1 kernelpagesize_kB=4
7fc1fcbc8000 default
7fc1fcbc9000 default file=/usr/lib/ld-2.24.so mapped=35 mapmax=44 N0=35 kernelpagesize_kB=4
7fc1fcdcb000 default anon=10 dirty=10 mapmax=2 N0=10 kernelpagesize_kB=4
7fc1fcde9000 default anon=2 dirty=2 N0=2 kernelpagesize_kB=4
7fc1fcdeb000 default file=/usr/lib/ld-2.24.so anon=1 dirty=1 mapmax=2 N0=1 kernelpagesize_kB=4
7fc1fcdec000 default file=/usr/lib/ld-2.24.so anon=1 dirty=1 N0=1 kernelpagesize_kB=4
7fc1fcded000 default anon=1 dirty=1 N0=1 kernelpagesize_kB=4
7ffc5f05b000 default stack anon=6 dirty=6 mapmax=2 N0=6 kernelpagesize_kB=4
7ffc5f148000 default
7ffc5f14a000 default
```

### 26. oom_adj
出现OOM时进程被kill的权值。范围为[-17,15]，越小意味着越不容易被kill。可以向其中写入数据进行更改
```
#(example)
#cat /proc/1/oom_adj
0
```

### 27. oom_score
出现OOM时进程被kill的分值，即出现OOM的时候哪个进程会被内核选择杀死，数值范围为[0,1000]，badness越高越容易被kill，0表示不会被选择kill，1000则表示总是被kill。
```
#(example)
#cat /proc/1/oom_score
0
```

### 28. oom_score_adj
用于调整oom_score中所记录的值。
```
#(example)
#cat /proc/1/oom_score_adj
0
```

### 29. pagemap
通过读取该文件，从而查看用户态进程每个虚拟页映射到的物理页。


### 30. personality
记录进程执行域。因为可能是较为敏感的信息，所以只有文件所有者可以读取。
```
#(example)
#cat /proc/1/personality
00000000
```

### 31. root
软链接文件，指向根目录

### 32. sched
进程调度信息，大多数字段计算方法在sched.c和sched_fair.c文件中。
```
#(example)
#cat /proc/1/sched
systemd (1, #threads: 1)
-------------------------------------------------------------------
se.exec_start                                :      10641651.970868  //此进程最近被调度到的开始执行时刻
se.vruntime                                  :           139.854055  //虚拟运行时间
se.sum_exec_runtime                          :          1162.274514  //累计运行的物理时间
se.nr_migrations                             :                  206  //需要迁移当前进程到其他cpu时累加此字段
nr_switches                                  :                 1105  //主动切换和被动切换的累计次数
nr_voluntary_switches                        :                  883  //主动切换次数（由于prev->state为不可运行状态引起的切换）
nr_involuntary_switches                      :                  222  //被动切换次数
se.load.weight                               :              1048576   //该se的load
se.avg.load_sum                              :               556434
se.avg.util_sum                              :               402117
se.avg.load_avg                              :                   11
se.avg.util_avg                              :                    8
se.avg.last_update_time                      :       10641651970868
policy                                       :                    0  //调度策略，0表示normal
prio                                         :                  120  //优先级(nice=0)
clock-delta                                  :                  200
mm->numa_scan_seq                            :                    0
numa_pages_migrated                          :                    0
numa_preferred_nid                           :                   -1
total_numa_faults                            :                    0
current_node=0, numa_group_id=0
numa_faults node=0 task_private=0 task_shared=0 group_private=0 group_shared=0
```

### 33. schedstat
调度状态信息。总共三个数据，具体解释如下：
+ 累计运行的物理时间，同```/proc/<pid>/sched```文件中的se.sum_exec_runtime/1000000    
+ 累计在就绪队列里的等待时间    
+ 主动与被动切换的累计次数，同```/proc/<pid>/sched```文件中的nr_switches    

```
#(example)
#cat /proc/1/schedstat
1162274514 174247328 1105
```

### 34. smaps
记录进程内存中所有的映射情况，类似于详细信息版本的/proc/[pid]/maps
（该文件只有在开启了内核的CONFIG_MMU选项了才会产生）

```
#(example)
#cat /proc/1/smaps
559235185000-55923526d000 r-xp 00000000 08:01 799480                     /usr/lib/systemd/systemd
Size:                928 kB            /*映射区域的总大小*/
Rss:                 872 kB            /*当前驻留于内存中的大小，即实际内存的占用量。不包括已经交换出去的代码*/
Pss:                 409 kB            /*Private Rss，映射到内存的页面中仅由进程单独使用的量*/
Shared_Clean:        860 kB            /*驻留在内存中与其他内存共享部分的“干净页面”大小，*/
Shared_Dirty:          0 kB            /*驻留在内存中与其他内存共享部分的“赃页”大小*/
Private_Clean:        12 kB            /*驻留在内存中私有的“干净页面”大小*/
Private_Dirty:         0 kB            /*驻留在内存中私有的“赃页”大小*/
Referenced:          872 kB
Anonymous:             0 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Locked:                0 kB
VmFlags: rd ex mr mw me dw 
55923526e000-559235293000 r--p 000e8000 08:01 799480                     /usr/lib/systemd/systemd
Size:                148 kB
Rss:                 148 kB
Pss:                  74 kB
Shared_Clean:          0 kB
Shared_Dirty:        148 kB
Private_Clean:         0 kB
Private_Dirty:         0 kB
Referenced:          148 kB
Anonymous:           148 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Locked:                0 kB
VmFlags: rd mr mw me dw ac 
559235293000-559235294000 rw-p 0010d000 08:01 799480                     /usr/lib/systemd/systemd
Size:                  4 kB
Rss:                   4 kB
Pss:                   2 kB
Shared_Clean:          0 kB
Shared_Dirty:          4 kB
Private_Clean:         0 kB
Private_Dirty:         0 kB
Referenced:            4 kB
Anonymous:             4 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Locked:                0 kB
VmFlags: rd wr mr mw me dw ac 
559235a81000-559235b2e000 rw-p 00000000 00:00 0                          [heap]
Size:                692 kB
Rss:                 592 kB
Pss:                 510 kB
Shared_Clean:          0 kB
Shared_Dirty:        164 kB
Private_Clean:         0 kB
Private_Dirty:       428 kB
Referenced:          592 kB
Anonymous:           592 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Locked:                0 kB
VmFlags: rd wr mr mw me ac 
...
```

### 35. stack
记录进程的内核堆栈的调试信息
```
#(example)
#cat /proc/1/stack
[<ffffffff81252f2b>] ep_poll+0x27b/0x350
[<ffffffff8125443e>] SyS_epoll_wait+0xce/0xf0
[<ffffffff815f8032>] entry_SYSCALL_64_fastpath+0x1a/0xa4
[<ffffffffffffffff>] 0xffffffffffffffff
```

### 36. stat
当前进程的所有状态信息。各项参数解释请参照[PROC系列之---/proc/pid/stat](http://blog.csdn.net/zjl_1026_2001/article/details/2294067) 
```
#(example)
#cat /proc/1/stat
1 (systemd) S 0 1 1 0 -1 4194560 2837 215477 66 637 8 107 1641 3161 20 0 1 0 5 138170368 1607 18446744073709551615 94086444371968 94086445321800 140721902830496 140721902828608 140471127129555 0 671173123 4096 1260 1 0 0 17 1 0 0 113 0 0 94086445329472 94086445478188 94086453792768 140721902833623 140721902833634 140721902833634 140721902833645 0
```

### 37. statm
记录内存使用方面的信息，具体各项参数信息如下：
+ size (same as VmSize in /proc/[pid]/status）    
  程序内存总大小
+ resident (same as VmRSS in /proc/[pid]/status)    
  实际使用物理内存大小
+ share (from shared mappings)    
  共享页面数量
+ text (code)    
  代码段大小
+ lib (library)    
  库文件大小
+ data data + stack    
  数据段与堆栈段大小总和
+ dt (dirty pages)    
  “脏页”数量
```
#(example)
#cat /proc/1/statm
#size resident share text lib data dt
33733   1607   1302  232  0   4403  0
```

### 38. status
进程的各种信息（进程id、凭证、内存使用量、信号等等）
```
#(example)
#cat /proc/1/status
Name:    systemd        /*产生此进程的命令*/
Umask:    0000
State:    S (sleeping)        /*进程的当前状态*/
Tgid:    1        /*进程组id号*/
Ngid:    0
Pid:    1        /*进程id号*/
PPid:    0
TracerPid:    0
Uid:    0    0    0    0
Gid:    0    0    0    0
FDSize:    64        /*目前已分配文件描述符个数*/
Groups:
NStgid:    1
NSpid:    1
NSpgid:    1
NSsid:    1
VmPeak:      200468 kB        /*虚拟内存峰值*/
VmSize:      134932 kB        /*虚拟内存大小*/
VmLck:           0 kB        /*已锁虚拟内存大小*/
VmPin:           0 kB
VmHWM:        6428 kB        /*实际使用物理内存峰值（即进程的“高水位”）*/
VmRSS:        6428 kB        /*实际使用物理内存大小*/
RssAnon:        1220 kB
RssFile:        5208 kB
RssShmem:           0 kB
VmData:       17476 kB        /*数据段大小*/
VmStk:         136 kB        /*堆栈段大小*/
VmExe:         928 kB        /*文本段大小*/
VmLib:        7176 kB        /*共享库大小*/
VmPTE:         124 kB        /*页表项大小*/
VmPMD:          12 kB
VmSwap:           0 kB
HugetlbPages:           0 kB
Threads:    1        /*进程中所含线程数量*/
SigQ:    0/15716
SigPnd:    0000000000000000
ShdPnd:    0000000000000000
SigBlk:    7be3c0fe28014a03
SigIgn:    0000000000001000
SigCgt:    00000001800004ec
CapInh:    0000000000000000
CapPrm:    0000003fffffffff
CapEff:    0000003fffffffff
CapBnd:    0000003fffffffff
CapAmb:    0000000000000000
Seccomp:    0
Cpus_allowed:    ffffffff,ffffffff,ffffffff,ffffffff
Cpus_allowed_list:    0-127
Mems_allowed:    00000000,00000001
Mems_allowed_list:    0
voluntary_ctxt_switches:    885
nonvoluntary_ctxt_switches:    222
```

### 39. syscall
用于记录系统调用，时间顺序为从文件末尾至文件开头。
```
#(example)
#cat /proc/1/syscall
232 0x4 0x7ffc5f07bc50 0x36 0xffffffff 0x431bde82d7b634db 0xc60 0x7ffc5f07bc40 0x7fc1fba23dd3
```
其中```0x7fc1fba23dd3```这种类型的数表示内存地址，而对于一些类似于```232```这样的数可能表示数组索引（文件描述符），也可能表示系统调用号。


### 40. task
目录文件。进程包含的线程，其内容为各个目录文件，以线程的ID命名，代表各个线程

### 41. timerslack_ns
当前进程的定时器延迟时间，单位为纳秒（nanoseconds）
```
#(example)
#cat /proc/1/timerslack_ns
50000
```

### 42. wchan
如果当前进程处于睡眠状态，记录引起调度的调用函数
```
#(example)
#cat /proc/1/wchan
ep_poll
```