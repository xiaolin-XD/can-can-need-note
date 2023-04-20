http1.0 一次请求一次响应
http1.1 keepalive，host虚拟主机，管道化请求未响应就发下一个请求，队头阻塞,range（断点续传，配合head请求方式获取总字节长度）
http2.0 帧，二进制，无法解决tcp队头阻塞，header压缩
http3.0 connectionid dh交换密钥 quic udp

tcp ngale算法，延迟确认，滑动窗口（客户端不断发送包，丢包场景1，服务端确认丢包，会在下次确认补上。丢包场景2，客户端发送丢包，一直收不到服务端确认会重发），拥塞窗口。
tcp为什么三次，用来交换序列号，验证两边网络。为什么四次需要等待2msl，其中一次丢包会如何

tsl/ssl，公钥加密私钥解密-签名。私钥加密共钥解密-加解密。中间人攻击

浏览器输入url，dns解析，建连，发送请求，响应请求，浏览器对响应解析渲染执行

进程通信
- 管道，fifo，半双工，只能单向，亲缘进程用无名管道，非亲缘进程用有名管道
- 信号 无法携带参数，可用于非血缘
- 消息队列 可用于非血缘，是内核维护的消息链表
- 共享内存 需要互斥信号量处理冲突
- socket 

用户态切内核的方式
- 系统调用 通过系统提供的中断
- 异常 内存映射时，在磁盘上，触发缺页异常，进入内核态
- 设备（cpu，硬盘）完成某些请求

cpu需要内存对其，方便一次性读取所需内容，单个cpu读取大小 x86是1k

内核内存管理，逻辑内存页，页表（进程私有），物理内存页，页替换算法（fifo，lru），swap交换区。32位单进程最大内存4G

死锁，互斥，保持占用资源，不可被其他剥夺占用的资源，循环等待无超时。解决方案：乐观锁，cas（aba），新申请的资源得不到时放弃已占用的资源，超时设置，要求任何请求都必须按顺序分配资源

动态绑定，静态绑定，虚方法表：子类会继承父类的虚方法表

类不可被继承：final、私有化构造器

泛型：伪泛型，桥方法

arraylist：初始10，扩容1.5倍
安全list：vectpor、Collections.synchronizedList、CopyOnWriteArrayList
删除元素：快速失败、安全失败
删除元素：fori迭代（删除时，list的索引也在改变，有坑），foreach迭代（ConcurrentModificationException），迭代器（双指针）

CopyOnWriteArrayList：为什么每次写都复制数组？数组元素用了volatile，但是只对数组地址修改起作用，对单个元素修改不起作用，如果直接操作数组，单个元素的修改操作对读不是立即可见

hashmap，1.7 数组+链表，1.8才加红黑树，1.7头插
concurrenthashmap，协助扩容，sync head节点，size，高低位

spi：顶层jdk接口必须由bootstrap加载器加载，bootstrap无法加载到用户配置的类，咋办？threadlocal，当加载实现，从threadlocal中拿到bootstrap加载器加载。

tomcat类加载：先直接委派 WebAppClassLoader → ExtClassLoader → Bootstrap ClassLoader。找不到再由WebAppClassLoader找，再找不到就委派给AppClassLoader。这么做是为了web应用之间的类隔离

复制算法：分两块，用其中一块，gc时copy到另外一块。不易产生碎片，内存使用率低，copy效率低
标记清除（cms）：顾名思义，产生碎片，浮动垃圾
标记整理：老年代
分代：eden，from，to，old

safepoint：线程运行到一个安全位置（对象地址不会变化）。抢占式中断，主动式中断

空间分配担保

对象引用地址修正：句柄池、直接引用（hotspot）

**Serial**：单线程，单cpu，还要暂停所有工作线程，复制算法
parnew：serial多线程版本，复制算法
parallel Scavenge：(关注吞吐量)
serial old：标记整理，配合ps，cms的兜底
parrallel old：标记整理
cms：写后屏障（），jdk8，初始标记（stw，标记跟），并发标记（和用户线程一起工作），重新标记（stw，标记前一个阶段运行时，改动地址的那些对象），并发清除（和用户线程一起工作）。并发收集，低停顿。cpu敏感，gc线程与cpu数量有关，无法处理浮动垃圾，标记清除有碎片。卡表(老年代跨代引用年轻代)
g1: jdk9，8g+，分区大小固定，三色标记（白，灰（对象被访问，属性未访问）黑）Remembered Set，期望停顿时间
zgc：jdk11。tb级别（numa架构）。ZGC中的Region区不存在分代的概念，而是大中小。染色指针（禁用压缩指针）
多标：浮动垃圾，一般放到下轮gc

漏标：cms、g1、zgc。断开灰-》白，新增黑-〉白，破坏其中一个条件就可。破坏1条件：增量更新，将新增的引用记录，cms通过写屏障将新增的引用如果是白色标记为灰色，重新标记。破坏2条件，satb，保存断掉的灰-》白关系，继续扫描灰-〉白（实际真正被断，游离的对象被标记为黑，增加了浮动垃圾）。读屏障，zgc

cms为什么增量：并发标记中记录新增的引用，重新标记stw，从记录的这些黑色根重新标记，黑色还引用了其他，都需要再次标记，所以比较耗时。
g1为什么satb：标记过程中发生新增引用，不管，交给下次gc。修改的引用通过satb记录，扫描satb，主动断开的设置为黑，被断开的设置为灰，从灰开始标记，所以没有增量耗时，这会产生浮动垃圾

promotion failed，当young gc的时候，把eden和survivor里的都还存活的对象，统一移到另一个survivor区中时，发现装不下了，就需要把部分对象，放到老年代中去，结果老年代空间也不足，这种场景呢，叫做promotion failed
concurrent mode failure，在promotion failed的前提下，老年代恰好还正在full gc

计算机内存模型，cpu指令比内存快，所以有高速缓存。为了解决数据不一致。1，锁总线，开销大。2.缓存锁，有了MESI 缓存一致性协议等。
jmm。java使用共享内存（还有一种叫消息传递，就是线程直接直接通信）的方式解决线程之间交换数据的。jmm定义多线程与主存的关系。


G1，satb queue，全局satb queue


线上故障排查：

Java 内存模型（Java Memory Model，JMM）就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了 Java 程序在各种平台下对内存的访问都能保证效果一致的机制及规范。
volatile1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；

　　2）它会强制将对缓存的修改操作立即写入主存；

　　3）如果是写操作，它会导致其他CPU中对应的缓存行无效。

事物，如何解决幻读
**RR****和****RC****使用场景**
sql执行过程
锁
发库发表
redis 为什么构建自己的vm

**索引优化：**
**语句优化：**
**表结构优化：**
**配置优化：**
mysql 二阶段提交事务

head
所有初始都是intial状态，自己要去修改前置节点状态为signal，让他记得唤醒自己
condition，打包成node加入condition队列，幻醒head下一个节点，休眠自己
获取condition队列头节点，加到aqs队列末尾，唤醒aqs队列头节点

代理
jmm

缓存一致性。更新db+查询缓存
更新db，删redis。脏
延迟双删：删redis，改db，等一会，删redis。
canal：最终一致性
雪崩：缓存同时失效。redis高可用，限流降级，随机过期，用不过期
穿透：缓存不在redis和db。参数校验，bool过滤器，都不在设置redis中key-》null
击穿：高并发下redis没数据同时去db读。读db加锁

热key
大key

redis扩容：
hash，扩容要全迁
一致性hash，不均衡问题
rediscluster，每个物理节点负责16384部分slot，key的哈希%16384个slot，即使扩容，也只会影响部分

主从，部属简单，俩节点即可。优点读写分离，缺点从无法自己升级为主
从发送psync给主，首次连接的话，主启动后台线程进行rdb，然后发给从，从先写入本地盘，再加载到内存，此时主会将写命令写入缓存，从实时同步这些命令

哨兵，读多场景。选leader哨兵，raft，客观下限，主观下限，负责将从选出主
集群，写多

super extends

分布式锁
redis，释放锁为什么lua

**RedLock****算法**


**多级缓存）**

redis优化？
redis热升级

三级缓存：objectfactory，早期对象map，最终beanmap
循环以来：当A创建时，将A对象放入factory（lambda，也会返回代理对象），记录createingset（用于发现是否循环依赖），再创建B，将B对象放入factory，发现A在CS里，查找早期map，没有就对A进行预先AOP，放入早期map


kafka epoch

ioc初始化

bean作用域
**singleton**，**prototype**，request，session

springmvc cloud
cap
es


nio, DirectByteBuffer, cleaner, 引用

写db，改redis和改redis，写db。在两个写线程的情况下，会造成脏
删redis，写db。在线程删redis后，读线程读db，读线程再改redis，写线程改db，就脏。
cache aside：写db 删redis。当读不到redis，读库

es filedata