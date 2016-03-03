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

## 内存篇
vm.swappiness = 20

// swap 空间使用权重，值越小系统使用swap的次数就越小，类似mysql数据库就可以将这个值设为0

vm.dirty_ratio = 60

// 控制一个在产生磁盘写操作的进程开始写出脏数据到内存缓冲区。缓冲区的值大小是系统内存的百分比。增大会使用更多系统内存用于磁盘写缓冲，可以提高系统的写性能。当需要持续、恒定的写入场合时，应该降低该数值

vm.dirty_background_ratio = 5

// 这个参数指定了当文件系统缓存脏页数量达到系统内存百分之多少时（如5%）就会触发pdflush/flush/kdmflush等后台回写进程运行，将一定缓存的脏页异步地刷入外存

vm.overcommit_memory

// 默认值为：0
从内核文档里得知，该参数有三个值，分别是：
0：当用户空间请求更多的的内存时，内核尝试估算出剩余可用的内存。
1：当设这个参数值为1时，内核允许超量使用内存直到用完为止，主要用于科学计算
2：当设这个参数值为2时，内核会使用一个决不过量使用内存的算法，即系统整个内存地址空间不能超过swap+50%的RAM值，50%参数的设定是在overcommit_ratio中设定。

vm.overcommit_ratio

// 默认值为：50
这个参数值只有在vm.overcommit_memory=2的情况下，这个参数才会生效。

vm.page-cluster

// 默认值: 3
这个参数用来控制VM的虚拟内存的，读取大量的page，同时发生page错误时，linux VM子系统为了避免过多的磁盘寻址。读取大量的page依赖于系统的内存。内核 一次读取page的数量等于2的page-cluster值的次方即2^page-cluster。当设的值超过2的5次方即2^5，它不会被swap所检测到。因为swap的数据page最大为2的5次方即32-page。

vm.min_free_kbytes

// 默认值 ：3519
这个参数值用来强制linux虚拟内存保留最小值的空闲。

vm.drop_caches

// 默认值 ：0
设置这个参数的值会让内核清理内存中的caches、denties、inodes，从而释放更多的内存。
有三个值可以设置，每设一个值都会引发内核释放不同的内容：

1：释放pagecache
2：释放denties、inodes
3：释放pagecache、denties、inodes
由于这是一个非破坏性操作而且脏对象不会被释放，因此应当先执行”sync“后再设置这个参数。

vm.dirty_ratio

// 默认值：40

参数意义：控制一个在产生磁盘写操作的进程开始写出脏数据到内存缓冲区。缓冲区的值大小是系统内存的百分比。增大会使用更多系统内存用于磁盘写缓冲，可以提高系统的写性能。当需要持续、恒定的写入场合时，应该降低该数值。

vm.dirty_expire_centisecs

// 默认值：2999

参数意义：用来指定内存中数据是多长时间才算脏(dirty)数据。指定的值是按100算做一秒计算。只有当超过这个值后，才会触发内核进程pdflush将dirty数据写到磁盘。

vm.dirty_background_ratio

// 默认值 ：10

参数意义：控制pdflush后台回写进程开始写出脏数据到系统内存缓冲区。缓冲区的值大小是系统内存的百分比。增大会使用更多系统内存用于磁盘写缓冲，可以提高系统的写性能。当需要持续、恒定的写入场合时，应该降低该数值。

vm.vfs_cache_pressure

// 默认值：100
参数意义：控制内核回收再利用用于缓存目录与inode对象的内存的趁势。
默认值设为100表示内核以平等的速度去考虑pagecache和swapcache的回收再利用。
减小它，会触发内核保持目录与inodes的缓存内存。
增大它，会触发内核回收再利用目录与inodes的缓存内存。

vm.panic_on_oom

// 默认值 ：0

参数意义：当超出内存时，是否开启内核崩溃特性。
当设为1时，表示当发生超出内存时，内核会panic
当设为0时，表示当发生超出内存时，内核会kill掉一些空闲进程从而不让系统内核崩溃而继续运行，通常也称它为oom_killer
因此一般用它的默认值即可

#网络篇

net.ipv4.tcp_syncookies = 1

// 当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；

net.ipv4.tcp_synack_retries = 2

// 对于远端的连接请求 SYN, 内核会发送 SYN ACK 数据报文, 以确认收到上一个SYN 连接请求包, 这是所谓三次握手机制的第二个步骤, 这里决定内核在放弃连接之前所送出的 SYN+ACK数目, 不应大于 255

net.ipv4.ip_forward = 0

// 是否开启包转发, 默认值0, 做路由时可以设置为1

net.ipv4.tcp_dsack = 1

// 该文件表示是否允许TCP发送“两个完全相同”的SACK

net.ipv4.tcp_sack = 1

//该文件表示是否启用有选择的应答（Selective Acknowledgment），这可以通过有选择地应答乱序接收到的报文来提高性能（这样可以让发送者只发送丢失的报文段）；（对于广域网通信来说）这个选项应该启用，但是这会增加对 CPU 的占用

net.ipv4.tcp_fack = 1

// 该文件表示是否打开FACK拥塞避免和快速重传功能, 当tcp_sack 设置为0时, 这个值即使设置为1也无效

net.ipv4.conf.all.accept_redirects = 0

net.ipv4.conf.default.accept_redirects = 0

// 如果主机所在的网段中有两个路由器，你将其中一个设置成了缺省网关，但是该网关在收到你的ip包时发现该ip包必须经过另外一个路由器，这时这个路由器就会给你发一个所谓的“重定向”icmp包，告诉将ip包转发到另外一个路由器。参数值为布尔值，1表示接收这类重定向icmp 信息，0表示忽略。在充当路由器的linux主机上缺省值为0，在一般的linux主机上缺省值为1。建议将其改为0以消除安全性隐患

net.ipv4.conf.all.secure_redirects = 0

// 其实所谓的“安全重定向”就是只接受来自网关的“重定向”icmp包。该参数就是用来设置“安全重定向”功能的。参数值为布尔值，1表示启用，0表示禁止，缺省值为启用。

net.ipv4.tcp_rfc1337 = 1

net.ipv4.tcp_congestion_control = cubic

net.ipv4.tcp_ecn = 2

// 该文件表示是否打开TCP的直接拥塞通告功能。

net.ipv4.tcp_fin_timeout = 10

// 修改系統默认的 TIMEOUT 时间

net.ipv4.tcp_keepalive_time = 300

// 表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为5分钟

net.ipv4.tcp_keepalive_probes = 5

// tcp 发送keepalive探测以确定该连接已经断开的次数

net.ipv4.tcp_keepalive_intvl = 15

// 探测消息发送的频率 * tcp_keepalive_probes　= 对于从开始探测以来没有响应的连接杀除时间．默认值是７５秒，也就是没有活动的连接将在１分钟以后被丢弃

net.ipv4.ip_local_port_range = 4096 65535

// 建立连接的端口范围,这里不要将最低值设的太低，否则可能会占用掉正常的端口

net.core.netdev_max_backlog = 8192

// 网卡最大队列数, 万兆网卡可以增大该值

net.ipv4.tcp_max_syn_backlog = 4096

// 表示SYN队列的长度，默认为1024，加大队列长度为4096，可以容纳更多等待连接的网络连接数。

net.core.somaxconn = 65535

// 系统可分配给单进程的最大队列数

net.ipv4.tcp_window_scaling = 1

// 该文件表示设置tcp/ip会话的滑动窗口大小是否可变。参数值为布尔值，为1时表示可变，为0时表示不可变。tcp/ip通常使用的窗口最大可达到65535 字节，对于高速网络，该值可能太小，这时候如果启用了该功能，可以使tcp/ip滑动窗口大小增大数个数量级，从而提高数据传输的能力。该值影响 tcp_rmem 大小调节     

net.ipv4.tcp_adv_win_scale = 2
net.ipv4.tcp_rmem = 8192 131072 16777216

// 该文件包含3个整数值，分别是：min，default，max
Min：为TCP socket预留用于接收缓冲的内存数量，即使在内存出现紧张情况下TCP socket都至少会有这么多数量的内存用于接收缓冲。
Default：为TCP socket预留用于接收缓冲的内存数量，默认情况下该值影响其它协议使用的 net.core.wmem中default的 值。该值决定了在tcp_adv_win_scale、tcp_app_win和tcp_app_win的默认值情况下，TCP 窗口大小为65535。
Max：为TCP socket预留用于接收缓冲的内存最大值。该值不会影响 net.core.wmem中max的值，今天选择参数 SO_SNDBUF则不受该值影响。

net.core.rmem_default = 131072

net.core.rmem_max = 16777216

net.ipv4.tcp_wmem = 8192 131072 16777216

net.core.wmem_default = 131072

// 该文件包含3个整数值，分别是：min，default，max
Min：为TCP socket预留用于发送缓冲的内存最小值。每个TCP socket都可以使用它。
Default：为TCP socket预留用于发送缓冲的内存数量，默认情况下该值会影响其它协议使用的net.core.wmem中default的 值，一般要低于net.core.wmem中default的值。
Max：为TCP socket预留用于发送缓冲的内存最大值。该值不会影响net.core.wmem_max，今天选择参数SO_SNDBUF则不受该值影响。默认值为128K

net.core.wmem_max = 16777216
net.core.optmem_max = 65536

net.ipv4.tcp_timestamps = 1

// 开启时间戳标签, 在公网服务器上建议关掉, 因为会导致 NAT网络用户请求失败, 同时这个值影响 tcp_tw_recycle 和 tcp_tw_reuse 

net.ipv4.tcp_tw_recycle = 1

// 表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。

net.ipv4.tcp_tw_reuse = 1

// 表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭，在公网服务上，建议将tw_reuse打开

net.ipv4.tcp_max_tw_buckets = 20000

// 表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。默 认为180000，改为20000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于 Squid，效果却不大。此项参数可以控制TIME_WAIT的最大数量，避免Squid服务器被大量的TIME_WAIT拖死. 但是在打开　tcp_tw_reuse　时，可以增加tcp_tw_buckets 数量，压测显示，在同一个服务上设置tcp_max_tw_buckets=120000, tw_reuse=1，性能比tcp_max_tw_buckets＝20000, tw_reuse=0 并发提高了一倍

# arp 篇
net.ipv4.neigh.default.gc_thresh3 = 2048

net.ipv4.neigh.default.gc_thresh2 = 1024

net.ipv4.neigh.default.gc_thresh1 = 128

net.ipv4.neigh.default.gc_interval = 120

net.ipv4.neigh.default.proxy_qlen = 96

net.ipv4.neigh.default.unres_qlen = 6

net.ipv4.route.flush = 1

net.ipv4.rt_cache_rebuild_count = -1

// 如果系统内核打印Ｎeighbour table overflow, 然后查看本机arp 缓存列表条数　arp -an ｜wc -l
如果该值已经接近或大于1024 （系统默认1024），那么可以加大arp列表值

# 链路跟踪表设置

net.ipv4.netfilter.ip_conntrack_max = 10000000

net.ipv4.netfilter.ip_conntrack_tcp_timeout_established = 360

net.ipv4.netfilter.ip_conntrack_tcp_timeout_fin_wait = 60

net.ipv4.netfilter.ip_conntrack_tcp_timeout_syn_recv = 13

net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait = 60

// 链路跟踪表作用和影响
Linux内核的ip_conntrack模块会记录每一个tcp数据包的estiablished,timewait,syn_recv等状态。并且默认的timeout是432000秒（五天时间）。系统所能记录的ip_conntrack也是有限的，如果超过了这个限度，就会出现内核级错误“ip_conntrack: table full, dropping packet”，其结果就是无法再有任何的网络连接了。

如果启用了链路跟踪表，那么需要修改它的最大跟踪数以及链路跟踪超时时间，或者干脆卸载掉　ip_conntack　模块，但是需要注意确保iptables 规则里没有应用-m　state 状态，　否则卸载失败

net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

参考连接 ( 涵盖几乎所有的内核参数) 
http://www.cyberciti.biz/files/linux-kernel/Documentation/networking/ip-sysctl.txt
