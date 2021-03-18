# linux 内核优化

## 概要
```shell
    1. 分析工具
            (1) ftrace, ebpf,  perf 等
```

## Page Cache 管理问题(磁盘 I/O)
```shell
    1. 概念
            (1) 由 Page Cache 问题引起的情况
                    a. 服务器的 load 飙高
                    b. 服务器的 I/O 吞吐飙高
                    c. 业务响应时延出现大的毛刺
                    d. 业务平均访问时延明显增加
                    
            (2) Page Cache 就是内核管理的内存, 属于内核态.
            
    2. 查看 Page Cache 的方法
            /proc/meminfo, free , /proc/vmstat 命令
            
            (1) cat /proc/meminfo
            
                    ...
                    Buffers: 1224 kB
                    Cached: 111472 kB
                    SwapCached: 36364 kB
                    Active: 6224232 kB
                    Inactive: 979432 kB
                    Active(anon): 6173036 kB
                    Inactive(anon): 927932 kB
                    Active(file): 51196 kB
                    Inactive(file): 51500 kB
                    ...
                    Shmem: 10000 kB
                    ...
                    SReclaimable: 43532 kB
                    
                            Page Cache                             Page Cache 
                  Buffers + Cached + SwapCached == Active(file) + Inactive(file) + Shmem + SwapCached
                  
            (2) 在 Page Cache 中,
                    Active(file) + Inactive(file) 是 File-backed page(与文件对应的内存页).
                    SwapCached: 只在 Swap 分区打开才会有, 建议在生产环境中关闭 Swap 分区, Swap 过程产生的 I/O 会
                                很容易引起性能抖动
                    Shmem: 指匿名共享映射方式分配的内存 (free 命令中 shared 这一项), 比如 tmpfs(临时文件系统)
                    
            (3) free 命令中
                     buff/cache:  是由 /proc/memninfo 中的 Buffers, Cached 和 SReclaimable 这三项组成的, 强调的是内存的可回收性, 
                                  即可以被回收的内存会统计在这一项里, 例如 SReclaimable 是可以被回收的内核内存, 包括 dentry 和
                                  inode 等
                                  
    3. 使用 page cache 目的
            (1) 标准 I/O 和内存映射, 是先把数据写入到 Page Cache, 可以减少 I/O 次数来提升读写效率
           
    4.  Page Cache 生命周期
            (1) Page Cache 产生(读写文件的时候)
                   有两种方式, Buffered I/O(标准 I/O) 和 Memory-Mapped I/O(存储映射 I/O)
                   
                   a. Buffered I/O(标准 I/O)---常用
                        write() 是将写的用户缓冲区(Userpace Page 对应的内存), 拷贝到内核缓冲区(Pagecache Page 对应的内存).
                        read() 是先从内核缓冲区拷贝到用户缓冲区, 再从用户缓冲区读数据, 即 buffer 和文件内容不存在任何映射关系
                        
                        标准 I/O 下文件的写入, 通常会影响 page cache 中的 Active(file) 和 Inactive(file) 两项(/proc/meminfo),
                        流程如下: 往用户缓冲区 buffer(Userspace Page) 写入数据, 然后 buffer 中的数据拷贝到内核缓冲区
                                (Pagecache Page), 如果内核缓冲区中还没有这个 Page, 就会发生 Page Fault(缺页)会去分配一个 Page,
                                再进行拷贝, 拷贝结束后该 Pagecache Page 是一个 Dirty Page(脏页), 需要将该 Dirty Page 中的内容
                                同步到磁盘, 同步到磁盘后, 该 Pagecache Page 变为 Clean Page 并且继续存在系统中.
                                
                                判断 Pagecache Page 是 dirty 还是 clean, 主要是看这个内存中的 page 内容与磁盘是否一致.
                                
                        查看脏页
                            > cat /proc/vmstat | egrep "dirty|writeback"
                              结果: 
                                  nr_dirty 40   // 目前脏页数量
                                  nr_writeback 2   // 正在回写同步到磁盘的脏也数量
                        
                   b. Memory-Mapped I/O(存储映射 I/O)
                        是直接将 Pagecache Page 给映射到用户地址空间, 用户直接读写 Pagecache Page 中内容. 存储映射 I/O 要比
                        标准 I/O 效率高, 少量数据拷贝的过程, 但需要管理映射关系, 管理复杂.
                        
            (2) Page Cache 释放
                    a. free 命令中 free 项代表可以真正使用的空闲内存, buff/cache 代表活跃的 page cache. 
                       没有 free 内存没有关系, 只要还有足够可回收的 Page Cache(通过直接回收和后台回收). 如果还是没有可用的 free
                       内存, 则看看是否启用 swap , 没有启动 swap 直接 dump 程序.
                       
                    b. sar 命令(系统性能)
                        > sar -B 1     // -B 显示 page 指标, 1 每秒
                        结果:
                        04:23:00 PM  pgpgin/s pgpgout/s   fault/s  majflt/s  pgfree/s pgscank/s pgscand/s pgsteal/s   %vmeff
                        04:23:01 PM      0.00      0.00     25.00      0.00     35.00      0.00      0.00      0.00    0.00
                        04:23:02 PM      0.00      0.00     29.00      0.00     28.00      0.00      0.00      0.00    0.00
                        04:23:03 PM      0.00      0.00     19.00      0.00     27.00      0.00      0.00      0.00    0.00
                        
                        
                        pgscank/s : kswapd(后台回收线程) 每秒扫描的 page 个数, 对应与 /proc/vmstat 中的 pgscan_kswapd
                        pgscand/s: Application 在内存申请过程中每秒直接扫描的 page 个数, 查看是否是用户进程申请内存进行时,进行内存回收
                                   对应与 /proc/vmstat 中的 pgscan_direct
                        pgsteal/s: 扫描的 page 中每秒被回收的个数,  对应与 /proc/vmstat 中的 pgsteal_kswapd + pgsteal_direct
                        %vmeff: pgsteal/(pgscank+pgscand), 回收效率, 越接近 100 说明系统越安全. 越接近 0 说明系统内存压力越大

                        
     
    5. 案例分析
            (1) page cache 管理不当导致 load 异常高
                    a. load 飙高的原因
                            第一种: 直接内存回收引起的 load 飙高
                            第二种: 系统中脏页积压过多引起的 load 飙高
                            第三种: 系统 NUMA 策略配置不当引起的 load 飙高
                            
                    第一种: 直接内存回收引起 load 飙高或者业务时延抖动
                    
                           直接内存回收是指在进程申请内存的过程中同步进行其他内存回收, 而这个回收过程可能会消耗很多时间, 进而导致进程
                           的都被迫等待, 造成很长时间的延迟, 以及系统的 CPU 利用率会升高, 最终引起 load 飙高.
                           
                           内存回收流程: 在开始内存回收, 先进行后台异步回收, 这不会引起进程的延迟；如果后台异步回收跟不上进行
                                       内存申请的速度, 就开始同步阻塞回收, 导致延迟, 这就是引起 load 高的原因.
                                       
                           解决方案是: 及早进行后台的内存回收来避免应用程序进行直接内存回收(同步阻塞回收), 后台的内存回收触发的时机是
                                      当内存水位低于 watermark low 时, 就会唤醒 kswapd 进行后台回收, 然后 kswapd 会一直回收到
                                       watermark high. 我们可以调整 vm.min_free_kbytes = 4194304 进行及早后台回收, 
                                       对于大于等于 128G 的系统而言, 将 min_free_kbytes 设置为 4G 比较合理.
                                       缺点是 提高了内存水位后, 应用程序可以直接使用的内存量就会减少, 这在一定程度上浪费内存
                                       
                           影响:
                                系统始终要保障有这么多的 free 内存, 压缩 Page Cache 的空间.  通过 /proc/zoneinfo 来观察
                                >  egrep "min|low|high" /proc/zoneinfo
                                
                                    ...
                                    min 7019
                                    low 8773
                                    high 10527
                                    ...
                 
                    第二种: 系统中脏页积压过多引起的 load 飙高
                                脏页积压过多会存在大量的 page cache 写回到磁盘, 存在大量的 I/O 操作.
                                可以通过 >  sar -r 1
                          07:19:04 PM kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit  kbactive   kbinact   kbdirty
                          07:19:05 PM    639460  32028608     98.04      5216  11710764  80337428    163.56  24714352   5624864     964
                          
                          kbdirty: 等待写回磁盘的内存量（以千字节为单位）
                          
                          可以进行系统脏页个数控制
                                vm.dirty_background_bytes = 0
                                vm.dirty_background_ratio = 10
                                vm.dirty_bytes = 0
                                vm.dirty_expire_centisecs = 3000
                                vm.dirty_ratio = 20
                                
                    第三种: 系统 NUMA 策略配置不当引起的 load 飙高
                                a. 例如 系统中还有一半左右的 free 内存, 但还是频频触发 direct reclaim, 导致业务抖动得比较厉害, 
                                   原因是 NUMA 策略设置为 zone_reclaim_mode
                                   
                                   推荐 vm.zone_reclaim_mode = 0
                                   
            (2) Page Cache 异常被回收导致业务性能问题
                   a. Page Cache 异常被回收的原因
                            第一种: 误操作而导致 Page Cache 被回收掉, 进而导致业务性能下降明显
                            第二种: 内核的一些机制导致业务 Page Cache 被回收, 从而引起性能下降
                            
                   第一种: 对 page cache 误操作
                            其中涉及到 inode, 是文件对应磁盘位置的映射关系, 进程会通过 inode 来找到文件的地址空间(address_space). 
                            然后结合文件偏移, 转换成 page index 来找具体的 Page. 如果该 Page 存在, 那就说明文件内容已经被读取到内存
                            如果该 Page 不存在那就说明不在内存中, 需要到磁盘中去读取.
                            
                            drop_cache 释放 inode 时, 也会导致对应的 page cache 映射丢失, 也相当于 page cache drop 掉.
                            
                                echo 1 > /proc/sys/vm/drop_caches    // 释放掉 Page Cache 中的 clean page
                                echo 2 > /proc/sys/vm/drop_caches    // 释放掉 Slab, 包括 dentry, inode 等, 把 inode 释放掉
                                　　　　　　　　　　　　　　　　　　　　　　　 也会释放 page cache
                                echo 3 > /proc/sys/vm/drop_caches    // 既释放掉 Slab, 又释放 Page Cache
                            
                            记录 drop_caches 执行次数(/proc/vmstat 文件), 可以排查业务性能问题是否由 drop_cache 造成的, 如果
                            发生问题前后, drop_caches 执行次数没有发生变化, 则跟 drop_cache 没有关系.
                                > grep drop /proc/vmstat
                                    drop_pagecache 3
                                    drop_slab 2
                                    
                   第二种: 内核机制
                            内存回收机制, 通过 Reclaimer(回收者), 这个回收者可以是 kswapd(内核线程), 也可以是用户线程, 会依次
                            扫描 pagecache page 和 slab page(slab 又包含 inode) 中有哪些可以被回收, 如果有就会尝试回收, 
                            如果没有就跳过. 在扫描 可回收 page 的过程中回收者一开始扫描的较少, 然后逐渐增加扫描比例直至全部都被扫描完. 
                            
                            观察内核机制对 slab 中的 inode 是否释放
                                > grep inodesteal /proc/vmstat
                                    pginodesteal 114341
                                    kswapd_inodesteal 1291853
                                    
                                    kswapd_inodesteal 是在 kswapd 回收的过程中, 因为回收 inode 而释放的 pagecache page 个数；
                                    pginodesteal 是 kswapd 之外其他线程在回收过程中, 回收 inode 而释放的 pagecache page 个数
                                    
                   b. page cache 被异常回收解决方案
                            方案一: 从应用代码层面优化
                                        明确地来对读写文件过程中产生的 Page Cache 区别对待, 对重要的数据, 可以通过 mlock() 来
                                        保护, 防止被回收或则被 drop；对于不重要的数据(比如日志), 可以通过 madvise() 告诉内核
                                        立即释放这些 Page Cache
                                        
                            方案二: 从 linux 系统层面优化(memory cgroup protection)
                                        将需要保护的应用程序使用 memory cgroup 来保护起来, 这样该应用
                                        程序读写文件过程中所产生的 Page Cache 就会被保护起来不被回收或者最后被回收.
                                        
                                    memory cgroup 提供以下内存水位控制线
                                        memory.max: 指 memory cgroup 内的进程最多能分配的内存, 如果不设置就默认不做内存大小的限制
                                        memory.high: 如果设置这一项, 当 memory cgroup 内进程的内存使用量超过该值后就会立即被
                                                     回收掉, 是为了尽快的回收掉不活跃的 Page Cache.
                                        memory.low(最后被回收的): 这一项是用来保护重要数据的, 当 memory cgroup 内进程的
                                        　　　　　　　　　　　　　　内存使用量低于该值后, 在内存紧张触发回收后就会先去回收不属于
                                        　　　　　　　　　　　　　　该 memory cgroup 的 Page Cache, 等到其他的 Page Cache 都被
                                        　　　　　　　　　　　　　　回收掉后再来回收这些 Page Cache.
                                        memory.min(禁止被回收): 当 memory cgroup 内进程的内存使用量低于该值后, 即使其他不在
                                                              该 memory cgroup 内的 Page Cache 都被回收完了也不会去回收这些
                                                               Page Cache, 可以理解为这是用来保护最高优先级的数据的
                                                               
                                    可以考虑将业务进程放在一个 memory cgroup 中, 然后设置 memory.{min,low} 来进行保护
                                
                            

    6. page cache 排查思路
            (1) /proc/vmstat 中的 pgscan 相关指标变化较大, 可能与 page cache 有关, pgscan 代表 Page Cache 的内存回收行为, 
                变化较大往往意味着系统内存压力紧张
                
            (2) 查看　/proc/vmstat　文件指标
                   a. Page Cache 回收问题
                        pgscan_kswapd : kswapd 后台扫描的 Page 个数
                        pgsteal_kswapd : kswapd 后台回收的 Page 个数
                        pgscan_direct : 进程直接扫描的 Page 个数
                        pgsteal_direct : 进程直接回收的 Page 个数
                        
                   b. 碎片整理问题
                        compact_stall : 直接碎片整理的次数
                        compact_fail : 直接碎片整理失败的次数, 如果变化很大, 则很难申请到连续内存, 需要调整碎片指数
                        compact_success : 直接碎片整理成功的次数
                        
                   c. 脏页回写问题
                        nr_dirty : 脏页个数
                        
                   d. drop cache 问题
                        drop_pagecache: 执行 drop_cache 来 drop page cache 的次数
                        drop_slab: 执行 drop_cache 来 drop slab 的次数
                        pginodesteal: 直接回收 inode 过程触发回收 page cache 页数
                        kswapd_inodesteal: kswapd 后台回收 inode 过程中触发的回收 page cache 页数
                        
                   e. IO
                        pgpgin : 从磁盘读文件读了多少 page 到内存
                        pgpgout: 内存中多少 page 被写回到磁盘.
                        
                   f. swap I/O
                        pswpin: 从 swap 分区读了多少 page 到内存
                        pswpout: 内存中多少 page 被交换到 swap 分区
                        
                   g. workingset
                        workingset_refault: page 被释放后短时间内再次从磁盘读入到内存
                        workingset_restore: page 被回收前又被检测为活跃 page 从而避免被回收.
                        
            (3) 观察 Page Cache 时, 可以先从简单的分析工具比如 sar 开始, 得到概况, 然后再使用更专业的工具比如 tracepoint 去做
                更细致的分析, 就能分析清楚 Page Cache 的详细行为, 以及它为什么会产生问题；
                对于偶发问题, 往往需要采集很多的信息才能抓取出来问题现场, 这种场景下最好使用 perf script 来写一些自动化分析的工具来
                提升效率；
                如果担心分析工具会对生产环境产生性能影响, 可以把信息采集下来之后进行离线分析, 或者使用 ebpf 来进行自动过滤分析, 
                注意 ebpf 需要高版本内核的支持
                
```
## 内存泄漏

### 内存泄露概念
```shell
    0. 观察进程内存工具
            pmap, ps, top 等
    1. 应用程序申请内存的流程
            应用程序调用库函数(malloc(), free(), calloc()), 库函数继而调用系统函数(mmap(), munmap(), brk(), sbrk()), 其中
       库函数是在系统函数上做了内存优化, 这些内存申请和释放相关的系统调用会修改进程的地址空间(address space), 其中
       brk() 和 sbrk() 修改的是 heap(堆), 而 mmap() 和 munmap() 修改的是 Memory Mapping Region(内存映射区), 目前为止都是
       作用于虚拟地址, 虚拟地址到真实的物理地址通过页管理.
       
    2. 内存的类型
            (1) 私有匿名内存
                    进程的堆, 栈, 以及 mmap(MAP_ANON | MAP_PRIVATE) 这种方式申请的内存. 其中栈是由操作系统来进行管理的,
                应用程序无需关注它的申请和释放；堆和私有匿名映射则是由应用程序来进行管理的, 它们的申请和释放都是由应用程序来负责的, 
                所以它们是容易产生内存泄漏的.
                
            (2) 共享匿名内存
                    进程通过 mmap(MAP_ANON | MAP_SHARED) 这种方式来申请的内存, 比如 tmpfs 和 shm. 这个类型的内存也由应用程序
                进行管理的, 可能会发生内存泄漏.
                
            (3) 私有文件映射
                    进程通过 mmap(MAP_FILE | MAP_PRIVATE) 这种方式来申请的内存, 比如进程将共享库(Shared libraries)和可执行文件
                的代码段(Text Segment) 映射到自己的地址空间. 对于共享库和可执行文件的代码段的映射, 是通过操作系统来进行管理, 
                应用程序无需关注它们的申请和释放. 而应用程序直接通过 mmap(MAP_FILE | MAP_PRIVATE) 来申请的内存则是需要应用程序自己来
                进行管理
                
            (4) 共享文件映射
                    进程通过 mmap(MAP_FILE | MAP_SHARED) 这种方式来申请的内存, File Page Cache 就属于这类内存, 这部分内存也需要
                应用程序来申请和释放, 存在内存泄漏的可能性
                
    3.paging(分页)的过程
            CPU 将要请求的虚拟地址传给 MMU（Memory　Management Unit, 内存管理单元), 然后 MMU 先在高速缓存 TLB( Translation
            Lookaside Buffer, 页表缓存) 中查找转换关系, 如果找到相应的物理地址则直接访问；如果找不到则在地址转换表(Page Table)里
            查找计算, 最终进程访问的虚拟地址就对应到实际的物理地址
            
    4. 对内存规划
            通过 ulimit　来规划进程最大的虚拟地址空间, 物理地址空间, 栈空间等等
            
    5. top 命令
            (1) 使用 top, 再键入 'g', 再键入 '3'
                    VIRT 项对应与 size(statm 文件), 代表进程申请的虚拟内存大小.
                    RES 项对应与 resident(statm 文件), 映射到进程地址空间的物理内存大小.
                    SHR 项对应与 shared(statm 文件), 以 MAP_SHARED 方式映射的内存, 可能被共享.
                    CODE 项对应与 text(statm 文件), 代码段大小
                    DATA 项对应与 data(statm 文件), 数据段大小.
                    
                有时候所有进程的 RES 相加起来要比系统总的物理内存大, 这是因为 RES 中有一些内存是被一些进程给共享的
                
                如果 SHR 很高, 可能是 tmpfs/shm 之类的数据在持续增长.
                如果 VIRT 很高而 RES 很小, 可能是进程不停地在申请内存, 但是却没有对这些内存进行任何的读写操作, 即虚拟地址空间存在内存泄漏
                
    6. pmap 命令
            (1) 在了解了进程大致内存情况, 想要继续了解使用 pmap
                    > pmap -x `pidof sshd`
                        Address           Kbytes RSS  Dirty   Mode   Mapping
                        000055e798e1d000    768  652    0     r-x--   sshd
                        000055e7990dc000    16   16    16     r----   sshd
                        000055e7990e0000    4    4      4     rw---   sshd
                        000055e7990e1000    40   40     40    rw---  [ anon ]
                        ...
                        00007f189613a000    1800 1624    0    r-x--  libc-2.17.so
                        00007f18962fc000    2048 0       0    ----   libc-2.17
                        
                    每一行表示一种类型的内存
                        Mapping: 用来表示文件映射中占用内存的文件, 比如 sshd 可执行文件, 或者堆 [heap], 或者栈[stack]
                        
                        Mode: 是该内存的权限, “r-x”是可读可执行, 往往是代码段 (Text Segment)；
                              “rw-”是可读可写往往是数据段 (Data Segment)；
                              “r–” 是只读, 往往是数据段中的只读部分
                              
                        Address, Kbytes, RSS, Dirty, Address 和 Kbytes 分别表示起始地址和虚拟内存的大小,
                                                     RSS(Resident Set Size)则表示虚拟内存中已经分配的物理内存的大小, 
                                                     Dirty 则表示内存中数据未同步到磁盘的字节数
                                                     
            (2) 对进程的内存使用概况进行大致了解, 如果地址空间中[heap]太大, 那有可能是堆内存产生泄漏；
                如果进程地址空间包含太多的 vma(可以把 maps 中的每一行理解为一个 vma), 很可能是应用程序调用很多 mmap 而没有 munmap；
                如果持续观察地址空间的变化, 发现某些项在持续增长, 那很可能是那里存在问题。
                
                
```

### 内存泄露案例
```shell
    1. 长时间的内存泄漏问题最后基本都会以 OOM 结束, 如果服务器有慢速的串口设备(console), 要防止它接收太多的日志, 尤其是 OOM 产
       生的日志, 因为 OOM 的日志量很大, 打印完整个 OOM 信息 kennel 会很耗时, 进而导致阻塞申请内存的进程, 甚至会导致整个系统假死．
       
       每个申请内存失败的进程, 会触发 OOM , 同时会加全局 oom_lock 锁, 再将信息输入到对应的日志中, kill 合适的进程(一般是自己), 
       之后再解 oom_lock 锁.
       
       
       OOM killer 在杀进程的时候, 会把系统中可以被杀掉的进程扫描一遍, 根据进程占用的内存以及配置的 oom_score_adj 来计算出进程
       最终的得分, 然后把得分(oom_score) 最大的进程给杀掉, 如果得分最大的进程有多个, 那就把先扫描到的那个给杀掉.
       
       OOM killer 的作用之一, 是找到系统中不停泄漏内存的进程然后把它给杀掉, 如果没有找对, 就会误杀其他进程, 甚至是误杀更为重要的业务进程
       
       配置合适的 OOM 策略(oom_score_adj)来防止重要的业务被过早杀掉(比如将重要业务的 oom_score_adj 调小为负值), 考虑误杀其他进程,
       可以通过比较进程的 /proc/[pid]/oom_score, 来判断出进程被杀的先后顺序
       
    2. shmem 内存泄露
            (1) /proc/meminfo 中 shmem 很大, 但是没有找到对应的进程的内存, 可能是 tmpfs 这类的 shmem
            (2) 除了 tmpfs 之外, 其他一些类型的内存也不会体现在进程内存中, 比如内核消耗的内存：
                    /proc/meminfo 中的 Slab(高速缓存), KernelStack(内核栈) 和 VmallocUsed(内核通过 vmalloc 申请的内存)
```

### 内核内存泄露
```shell
    1. 应用程序可以通过 malloc() 和 free() 在用户态申请和释放内存, 也可以通过 kmalloc()/kfree() 以及 vmalloc()/vfree() 在内核态
    　 申请和释放内存
    
    2. kmalloc() 内存的物理地址是连续的, 而 vmalloc() 内存的物理地址则是不连续的. 体现在 /proc/meminfo
    
            ...
            Slab: 2400284 kB
            SReclaimable: 47248 kB
            SUnreclaim: 2353036 kB
            ...
            VmallocTotal: 34359738367 kB
            VmallocUsed: 1065948 kB
            ...
            
        vmalloc 申请的内存对应于 VmallocUsed (即已使用的 Vmalloc 区大小)
        kmalloc 申请的内存对应于 Slab 这一项, 分为两部分, 其中 SReclaimable 是在内存紧张时可以被回收的内存, 而 SUnreclaim 则是
        不可以被回收只能主动释放的内存.
        
    3. 内核空间内存的生命周期是与内核一致的, 却不与内核模块一致, 在内核模块退出时, 不会自动释放掉该内核模块申请的内存, 只有在
       内核重启(即服务器重启)时才会释放掉这部分内存. 
       
    4. 
        如果 /proc/meminfo 中内核内存(比如 VmallocUsed 和 SUnreclaim)太大, 可能发生内核内存泄漏
        可以在 /proc/vmallocinfo 文件中找到对应的模块内核内存使用信息.
        
    5. kmemleak 内核检查工具
            kmemleak 通过检查内核内存的申请和释放, 来判断是否存在申请的内存不再使用也不释放的情况. 如果存在, 就认为是内核内存泄漏, 然后
            把这些泄漏的信息通过 /sys/kernel/debug/kmemleak 这个文件导出给用户分析.
            
            一般只在测试环境上使用, 因为对性能会有比较明显的影响；
            
            在生产环境中可以使用 tracepoint 或者 kprobe, 来追踪特定类型内核内存的申请和释放, 从而判断是否存在内存泄漏
            内核内存泄漏通常都是第三方驱动或者自己写的一些内核模块导致的, 在出现内核内存泄漏时, 可以优先去排查它们
```

### 内存泄露排查思路
```shell
    1. 排查出异常点(/proc/meminfo)
            
            a. Active(anon)
                    在 active anon lru 上的 page, 和 Inactive(anon) 相互转换.
                    
            b. Inactive(anon)
                    在　inactive anon lru 上的 page, 只可以被交换到 swap 分区, 不可以被回收, Active(anon) 和 Inactive(anon)
               是应用程序使用　malloc() 或则 mmap() 匿名方式申请的内存, 如果者两项过大, 需要排查应用程序内存申请方式.
               
                    排查思路:
                        1. 使用 top 命令, 找出内存开销大的进程
                        2. 找出异常进程, 使用 pmap 分析这个进程.
                        3. 如果没有任何进程的内存开销大, 排查 tmpfs ．
                        
            c. Unevictable
                    这部分内存在系统内存紧张时不能被回收, 如果持续增长会有问题.
                    构成如下:
                        1. RAM disk 或则 ramfs 消耗的内存
                        2. 以 SHM_LOCK 方式来申请 Shmem
                        3. 使用 mlock() 系类函数管理的内存
                        
            d. Mlocked
                    是 Unevictable 的一种, 最常用, 小于等于 Unevictable, 有问题可以排查 mlock() 保护的内存
                    
            e. AnonPages
                    匿名映射页, 没有对应文件的内存, 和 Inactive(anon), Active(anon) 不一样, 这个有问题可以排查对应的 malloc(),
               mmap(PROT_WRITE, MAP_ANON|MAP_PRIVATE)
               
            f. Mapped
                    这个值是应用程序使用 mmap() 申请内存, 还没有进行 unmap() 是否对应的内存. 有问题可以排查 mmap() 
                    
            g. Shmem
                    共享内存, 其中包含 tmpfs
                    排查思路如下:
                        1. 使用 top, 查找出 SHR 大的进程
                        2. 找出异常进程, 使用 pmap 分析这个进程.
                        3. 如果没有任何进程的内存开销大, 排查 tmpfs
                        
            h. Slab
                    分为可回收(SReclaimable)和不可回收(SUnreclaim), 不可回收(SUnreclaim)发送泄露, 通过 kmalloc 申请内存没有主动
               释放,会有问题.
                    排查思路如下:
                        1. 使用 slabtop 查看哪一项 slab 较大.
                        2. 排查驱动程序以 kmalloc() 方式申请的内存
                        
            i. VmallocUsed
                    通过 vmalloc 方式申请内核内存, 通过 /proc/vmallocinfo 判断那些驱动程序以 vmalloc 方式申请内存较多.
     
    2. 分析进程的内存泄露原因
            第一步: 找到对应的进程
            第二步: 使用 pidstat 命令追踪进程的内存行为
                    > pidstat -r -p 31108 1
                    
            第三步: 查看增大内存区域是哪个, /proc/PId/smaps, 看到 有一个 10240 KB(10489856) 内存异常.
            第四步: 使用 strace 命令跟着这个进程系统调用
                    > strace -t -f -p 31108 -o 31108.strace
                    
                    在 grep 这个大小(10489856)
                    > cat 31108.strace | grep 10489856
                        
        
```
## TCP 重传(网络 I/O)

### TCP 连接断开
```shell
    1. Client 调用 connect() 后, Linux 内核就开始进行三次握手.
            第一步: Client 会给 Server 发送一个 SYN 包, 但是该 SYN 包可能会在传输过程中丢失, 或者因为其他原因导致 client 没有
            　　　　收到 synack 包, 这时 Client 侧就会触发超时重传机制, 通过 "tcp_syn_retries" 配置项决定重发几次.
                
                   例如: tcp_syn_retries 为 3, 在 Client 发出 SYN 后, 过 1 秒, 还没有收到 Server 的响应, 进行第一次重传, 
                        这时等待 2 秒, 以此类推, 第二次重传 4 秒, 第三次重传 8 秒, 还是没有收到 connect() 返回 ETIMEOUT 错误.
                        
                   通常设置　net.ipv4.tcp_syn_retries 为 3.
                   
                   此时 client 处于 SYN_SENT 状态, server 处于 LISTEN 状态
                   
            第二步: 半连接, Server 每收到一个新的 SYN 包, 都会创建一个半连接, 然后把该半连接加入到半连接队列(syn queue), 队列
            　　　　长度就是 "net.ipv4.tcp_max_syn_backlog " 配置项, Server 从这个队列中取出, 并进行处理后, 才会向对应的 client
                   发送 synack.
                   
                   对于服务端, 要适当增加 net.ipv4.tcp_max_syn_backlog 值(16384)
                   
                   设置 net.ipv4.tcp_syncookies = 1 
                   这一阶段会存在 syn flood 攻击, 恶意的 client 只管发送 sync 不接受 server 的 syncack, 导致无法正确建立 TCP 连接.
                   让 Server 的半连接队列耗尽, 无法响应正常的 SYN 包.解决方案是引入 syn cookies 机制, 即在 Server 收到 SYN 包时,
                   先不去分配资源保存 Client 的信息, 而是根据这个 SYN 包计算出一个 Cookie 值, 将 Cookie 记录到 SYNACK 包中
                   发送出去. 对于正常的连接, 该 Cookies 值会随着 Client 的 ACK 报文被带回来, 然后 Server 再根据这个 Cookie 检查
                   这个 ACK 包的合法性, 如果合法, 才去创建新的 TCP 连接, 通过 SYN Cookies 防止部分 SYN Flood 攻击.
                   
                    此时 client 处于 SYN_SENT 状态, server 处于 SYN_RCVD 状态
                   
            第三步: Server 向 client 发送 syncack, Server 也会因为某种原因收不到 client ack 包, 通过 
                  "net.ipv4.tcp_synack_retries" 进行 syncack 包重传.
                  
            第四步: Client 在收到 Serve 的 SYNACK 包后, 就会发出 ACK, Server 收到该 ACK 后, 三次握手就完成, 即产生一个 
                   TCP 全连接(complete), 会被添加到全连接队列 (accept queue)中, Server 会调用 accept() 来完成 TCP 连接的建立.
                   
                   全连接队列（accept queue）的长度是由 listen(sockfd, backlog) 这个函数里的 backlog 控制的, 而该 backlog 的
                   最大值则是 "net.core.somaxconn", 建议调大(net.core.somaxconn = 16384)
                   
                   当服务器中积压的全连接个数超过"net.core.somaxconn"后, 新的全连接就会被丢弃掉. Server 在将新连接
                   丢弃时根据 "net.ipv4.tcp_abort_on_overflow", 决定是否向 client 发送 reset 包, 让 client 不需要重连, 一般
                   "net.ipv4.tcp_abort_on_overflow" 设置为 0, 不向 client 发送 reset 包, 等 server 有能力处理 accept() 时,
                   client 重新请求连接就能连上.
                   
                   此时 client 处于 ESTABLISHED 状态, server 处于 SYN_RCVD 状态
                   
            第五步: accept() 成功返回后, 一个新的 TCP 连接建立完成, CP 连接进入 ESTABLISHED 状态
            
                   此时 client 处于 ESTABLISHED 状态, server 处于 ESTABLISHED 状态
                   
    2. 连接断开
            当应用程序调用 close() 时, 会向对端发送 FIN 包, 然后会接收 ACK；
            对端也会调用 clsoe() 来发送 FIN, 然后本端也会向对端回 ACK, 这就是 TCP 的四次挥手过程
            
            FIN_WAIT_1(主动关闭方)
                主动方调用 close(), 向对端发送 FIN 包, 就进入 FIN_WAIT_1 状态.
                
            FIN_WAIT_2(主动关闭方)
                当主动方发送完 FIN 包, 并且收到对端的 ACK 包, 但是还没有收到对端的 FIN 包, TCP 进入到这个状态后, 如果本端迟迟收不到对端的
                FIN 包, 就会一直处于这个状态, 一直消耗系统资源. Linux 为了防止这种资源的开销, 设置这个状态的超时时间 
                net.ipv4.tcp_fin_timeout(默认为 60s), 超过这个时间后就会自动销毁该连接
                
            TIME_WAIT(主动关闭方)
                TIME_WAIT 状态存在的意义是：主动关闭方收到对端的 FIN 包, 最后发送的 ACK 包可能会被丢弃掉或者有延迟, 这样对端就会
                再次发送 FIN 包. 如果不维持 TIME_WAIT 这个状态, 那么再次收到对端的 FIN 包后, 本端就会回一个 Reset 包, 这会产生一些异
                常.维持 TIME_WAIT 状态一段时间, 可以保障 TCP 连接正常断开.
                
                通过 net.ipv4.tcp_max_tw_buckets 可以设置 TIME_WAIT, 可以设置小一点(net.ipv4.tcp_max_tw_buckets = 10000)
                
                Client 关闭跟 Server 的连接后, 也有可能很快再次跟 Server 之间建立一个新的连接, 而由于 TCP 端口最多只有 65536 个, 
                如果不去复用处于 TIME_WAIT 状态的连接, 就可能在快速重启应用程序时, 出现端口被占用而无法创建新连接的情况, 建议打开复用
                TIME_WAIT 的选项：
                    net.ipv4.tcp_tw_reuse = 1
                    
                建议　tcp_tw_recycle　禁用
                    net.ipv4.tcp_tw_recycle = 0
                    
            CLOSE_WAIT 
                没有调用　close() 来关闭连接原因, 建议排查下程序哪里没有调用 close() 
                
```

### TCP 数据收发包
```shell
    1. TCP 数据包发送
            (1) TCP 发送缓冲区的大小受 "net.ipv4.tcp_wmem" 影响, 针对单个 TCP 连接的, 单位是 Byte
                                        min　　default　　　max
                    net.ipv4.tcp_wmem = 8192 　65536 　　16777216
                    
                TCP 发送缓冲区的大小会在 min 和 max 之间动态调整, 初始化大小为 default, 有其他约束, net.ipv4.tcp_wmem 的 max
                值要小于等于  net.core.wmem_max
                
            (2) 应用程序可以通过 setsockopt() 里的 SO_SNDBUF 来设置固定的缓冲区大小, 这样 tcp_wmem 就会失效, 而且这个缓冲区大小
                设置的是固定值, 内核也不会对它进行动态调整, 当然　SO_SNDBUF　大小也不能超过 net.core.wmem_max
                
            (3) net.ipv4.tcp_mem 代表所有 TCP 连接消耗的总内存, 单位是 Page(页数), 即 4K, 当所有 TCP 连接消耗的内存总和达到 max 后,
                导致法再往外发包
                    net.ipv4.tcp_mem = 8388608 12582912 16777216
                    
                通过事件方式检查到是否已经满了
                    第一步: 打开 tracepiont（需要 4.16+ 内核版本）
                            > echo 1 > /sys/kernel/debug/tracing/events/sock/sock_exceed_buf_limit/enable
                            
                    第二步: 看是否有该事件发生
                            >  cat /sys/kernel/debug/tracing/trace_pipe
                            
            (3) IP 层
                    关注 net.ipv4.ip_local_port_range 可用端口范围, 不能太小.
                        net.ipv4.ip_local_port_range = 1024 65535
                        
                    TCP/IP 数据流的流控
                        > ip -s -s link ls dev eth0
                            …
                            TX: bytes packets errors dropped carrier collsns
                            3263284 25060 0 0 0 0
                            
                        如果 dropped 不为 0, 则可能是 txqueuelen 太小, 需要调大.
                        > ifconfig eth0 txqueuelen 2000
                        
    2. TCP 数据包接收
            (1) TCP 数据包到达网卡后, 就会触发中断(IRQ) 告诉 CPU 读取这个数据包, 但是在高性能网络场景下, 数据包的数量会非常大,
               如果每来一个数据包都要产生一个中断, 那 CPU 的处理效率就会很低, 所以就产生 NAPI(New API)这种机制让 CPU 一次性地去轮询
               (poll)多个数据包, 以批量处理的方式来提升效率, 降低网卡中断带来的性能开销.
               
               设置 "net.core.netdev_budget " 决定一次可以 poll 多少个数据包.
                 net.core.netdev_budget = 600
                 
            (2) TCP 接收缓冲区大小设置
            
                    net.ipv4.tcp_rmem = 8192 87380 16777216
                    
                TCP 接收缓冲区大小也是在 min 和 max 之间动态调整, 但需要 net.ipv4.tcp_moderate_rcvbuf(值为 1 自动调整)去设置,
                TCP 接收缓冲区会直接影响 TCP 拥塞控制, 进而影响到对端的发包, 所以使用该控制选项可以更加灵活地控制对端的发包行为
                
                只有在 net.ipv4.tcp_moderate_rcvbuf 为 1, 并且应用程序没有通过 SO_RCVBUF 来配置缓冲区大小的情况下, TCP 接收缓冲区
                才会动态调节.
                
            (3) SO_RCVBUF 设置的值最大也不能超过 net.core.rmem_max
                    
```

### TCP 拥塞控制
```shell
    1. 流程
            第一阶段: 慢启动
                    TCP 连接建立后, 发送方进入慢速启动阶段, 然后逐渐地增大发包数量(TCP Segments), 这个阶段每经过一个 
                    RTT(round-trip time), 发包数量就会翻倍. 初始发送数据包的数量是由 init_cwnd(初始拥塞窗口)来决定的, 值为 10, 
                    如果初始拥塞窗口设置得过大, 可能会导致 TCP 重传．
                    
                    当拥塞窗口(cwnd) 增大到一个阈值(ssthresh, 慢启动阈值)后, TCP 拥塞控制就进入下一个阶段：
                    拥塞避免(Congestion Avoidance)
                    
            第二阶段: 拥塞避免
                    在这个阶段 cwnd 不再成倍增加, 而是一个 RTT 增加 1, 缓慢地增加 cwnd, 以防止网络出现拥塞, 这种情况下容易出现丢包,
                    乱序, 例如发送端一次性发送 4 个 TCP segments, 但是第 2 个 segment 在传输过程中丢失了, 接收方就接收不到该
                    segment. 第 3 个 TCP segment 和第 4 个 TCP segment 正常被接收到, 此时 3 和 4 就属于乱序报文, 会被加入到
                    接收端的 ofo queue(乱序队列)里, 此时接收方会 ack 丢掉的那一包, 发送方收到异常 ack 包后, 进行下一个阶段:
                    快速重传
                    
            第三阶段: 快速重传和快速恢复
                     判断丢包的依据是收到 3 个相同的 ack. 在网络很糟糕的情况下, 还会进行超时重传, 即发送出去一个数据包, 超过一段
                     时间(RTO)都收不到它的 ack, 就出现网络拥塞, 需要将 cwnd 恢复为初始值, 再次从慢启动开始调整 cwnd 的大小.
                     
    2. 接收方: rwnd(接收窗口)
            接收方在收到数据包后, 会给发送方回一个 ack, 然后把自己的 rwnd 大小写入到 TCP 头部的 win 这个字段, 发送方就能根据这个字段
            知道接收方的 rwnd. 发送方在发送下一个 TCP segment 时, 会先对比发送方的 cwnd 和接收方的 rwnd, 得出这二者之间的较小值, 
            然后控制发送的 TCP segment 个数不能超过这个较小值
```

### TCP 问题排查
```shell
    1. 延迟问题
            (1) tcpdump 适用于 TCP/IP 层的分析, 不适用应用协议
            (2) 使用 systemtap 工具进行网络排查. 如果使用的是 Redhat 或者 CentOS, 可以考虑使用 systemtap；
                如果是 Ubuntu, 可以考虑使用 lttng
            (3) tcprstat 适用于 MySQL 的排查
            
    2. TCP 重传
            (1) TCP 重传率通过 /proc/net/snmp 指标计算.
                    ActiveOpens: 主动打开的 TCP 连接数量
                    PassiveOpens: 被动打开的 TCP 连接数量
                    InSegs: 收到的 TCP 报文数量
                    OutSegs: 发出的 TCP 报文数量
                    EstabResets: TCP 连接处于 Established 时发生的 Reset
                    AttemptFails: 连接失败的数量
                    CurrEstab: 当前状态为 Established 的 Tcp 连接数
                    RetransSegs: 重传的报文数量
                    
                 重传率 retrans = (RetransSegs - last RetransSegs) / (OutSegs - last OutSegs) * 100
                 
            (2) 发送端在发送一个 TCP 数据包后, 会把该数据包放在发送端的发送队列里, 也叫重传队列. 此时, OutSegs 会相应地加 1, 
                队列长度也为 1. 如果收到接收端对这个数据包的 ACK, 该数据包就会在发送队列中被删掉, 然后队列长度变为 0；
                如果收不到这个数据包的 ACK, 就会触发重传机制, 有超时重传和快速重传, 超时重传是说发送端在发送数据包的时候, 会启动
                一个超时重传定时器（RTO）, 如果超过这个时间, 发送端还没有收到 ACK, 就会重传该数据包, 然后 OutSegs 加 1, 
                同时 RetransSegs 也会加 1
                
            (3) 分析 tcp 重传
                    第一步: 保存数据包
                            > tcpdump -s 0 -i eth0 -w tcpdumpfile
                            
                    第二步: 过滤 tcp 重传包
                            > tshark -r tcpdumpfile -R tcp.analysis.retransmission
                            
                 
                 使用 tcpretrans 工具判断 tcp 重传.
                 Tracepoint 是一个更加轻量级也更加方便的追踪 TCP 重传的工具, 但是需要你的内核 版本为 4.16+；
```
## 内核态 CPU 利用率高(CPU)

### CPU 概念
```shell
    1. CPU 架构
            一个实体 CPU 通常会有两个逻辑线程, 即 Core 0 和 Core 1, 每个 Core 都有自己的 L1 Cache, L1 Cache 又分为
            dCache 和 iCache, L1 Cache 只有 Core 本身可以看到, 其他的 Core 看不到. 同一个物理实体 CPU 中的两个 Core 会
            共享 L2 Cache, 其他的实体 CPU 是看不到这个 L2 Cache 的, 所有的实体 CPU 会共享 L3 Cache
            
    2. top 命令 cpu 指标
            top 命令 cpu 指标通过解析 /proc/stat 文件
                us: cpu 执行用户代码消耗的时间, 比例越高, 说明 cpu 利用率越好.
                sy: 执行内核代码消耗的 cpu 时间, 越低越好.
                ni: 显示调整该线程占用 cpu 的时间, 增大这个线程的 nice 值, 则希望少占用 cpu 的时间.
                id：空闲时间
                wa: cpu 阻塞在 I/O 的时间. 例如等待磁盘 I/O, 高说明 I/O 有问题
                hi: cpu 消耗在硬中断的时间, 一般会很低
                si: cpu 消耗在软中断的时间, 一般是网络收发包的软中断, 写文件落盘产生的软中断等, 在网络吞吐量较高的系统, 这个值会偏高.
                
            idle 是 CPU 无事可做, 而 wait 则是 CPU 想做事却做不了
            
    
```


