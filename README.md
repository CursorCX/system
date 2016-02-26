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
* soft nofile 65535 
* hard nofile 65535
