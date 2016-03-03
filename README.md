# system
## Linux 内核参数调优, 以下参数是结合生产环境给出的, 具有一定的参考价值, 当然还有很多没有涉及到参数, 希望大家多多提意见, 这里已内存24G, CPU为24核的机器作为参考

## Kernel 篇
kernel.panic = 10
// 当内核出现致命错误时, system 将会在10s后reboot

kernel.panic_on_oops = 1

kernel.sysrq=0
// 不允许使用 Magic SysRq key, 该键可以允许一些直接对内核的操作, 所以禁掉

kernel.ctrl-alt-del=1
// 当值>0,system 马上reboot,当值=0,系统会发送给 init进程信号,进行平滑重启 

kernel.core_pattern=/dev/null
// 内核错误重定向路径

kernel.shmmax = 12661555200
// 单个段允许使用的大小, 根据实际场景设定

kernel.shmall = 3091200
// 全部允许使用的共享内存大小, 单位为 4K, 实际大小为: 4K*3091200 = 12 G, 该参数应大于 kernel.shmmax 

kernel.msgmnb = 65536
// 控制消息队列的最大值

kernel.msgmax = 65536
// 控制消息的最大值, 以字节为单位

fs.file-max = 819200
// 系统所有进程一共可以打开的文件数量,同时还需要设置当前shell以及由它启动的进程的资源限制。这个需要用ulimit来设置, 系统默认的ulimit对文件打开数量的限制是1024，修改/etc/security/limits.conf并加入以下配置，永久生效
<<<<<<< Updated upstream
* soft nofile 65535 
* hard nofile 65535

## 内存篇
vm.swappiness = 20

;swap 空间使用权重，值越小系统使用swap的次数就越小，类似mysql数据库就可以将这个值设为0

vm.dirty_ratio = 60

;控制一个在产生磁盘写操作的进程开始写出脏数据到内存缓冲区。缓冲区的值大小是系统内存的百分比。增大会使用更多系统内存用于磁盘写缓冲，可以提高系统的写性能。当需要持续、恒定的写入场合时，应该降低该数值

vm.dirty_background_ratio = 5

;这个参数指定了当文件系统缓存脏页数量达到系统内存百分之多少时（如5%）就会触发pdflush/flush/kdmflush等后台回写进程运行，将一定缓存的脏页异步地刷入外存

vm.overcommit_memory

;默认值为：0
;从内核文档里得知，该参数有三个值，分别是：
;0：当用户空间请求更多的的内存时，内核尝试估算出剩余可用的内存。
;1：当设这个参数值为1时，内核允许超量使用内存直到用完为止，主要用于科学计算
;2：当设这个参数值为2时，内核会使用一个决不过量使用内存的算法，即系统整个内存地址空间不能超过swap+50%的RAM值，50%参数的设定是在overcommit_ratio中设定。

vm.overcommit_ratio

;默认值为：50
;这个参数值只有在vm.overcommit_memory=2的情况下，这个参数才会生效。
* soft nofile 100000
* hard nofile 100000
