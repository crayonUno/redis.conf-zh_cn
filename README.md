# Redis - redis.conf
> Redis 5.0.8 默认配置文件的翻译。个人英语水平有限，应以原文档为标准。
>
> <https://xie.infoq.cn/article/bf4d640ad438576c6e879edb8> 「InfoQ 写作平台」
>

**持续更新中...**

Redis 配置文件范例。

需要注意的是为了能顺利读取配置文件，Redis 启动时要将配置文件路径作为第一个参数：

./redis-server /path/to/redis.conf

## INCLUDES （包含）

在这配置包含一个或多个配置文件。这个配置项适用于那些对大部分 Redis 实例有标准的配置模板，但对小部分 Redis 实例有定制化需求的场景。 包括文件可以包含其他文件，所以请明智使用。

请注意 "include" 配置不会被 admin 或者 Redis 哨兵 "CONFIG REWRITE" 命令重写。由于 Redis 总是使用最后处理的行作为配置值，所以最好将 includes 配置放在该文件的最开始以此避免配置在运行的时候被重写。

相反的你想要用 includes 配置来重写配置项，那 include 应该放在最后一行会更好。

### **#include /path/to/local.conf**

### **#include /path/to/other.conf**

## MODULES（模块）

启动时（at startup）加载模块。如果 server 加载模块失败服务器会终止（abort）。

### **#loadmodule /path/to/my_module.so**

### **#loadmodule /path/to/other_moudle.so**

## NETWORK（网络）

如果没有使用 bind 进行配置，Redis 则默认监听所有 Server 上可以访问的网络接口的连接。如果配置了 bind 指向具体的值，Redis 则只监听配置的那些连接的网络接口。可以是一个 IP 或者紧接着多个 IP 地址。

示例：

**#bind 192.167.2.34 10.0.0.1**

**#bind 127.0.0.1 ::1**

警告：如果跑 Redis 的机器直接暴露在网络中，binding（指定，绑定）所有的网络接口有潜在的危险，且会让实例暴露给网络上的所有人。因此，我们取消注释了下面的 bind 指令，这会让 Redis 只监听 IPv4 的环回地址（意味着 Redis 只接受跑在和 Redis 实例一台机器上的客户端连接）。

**如果你确认你的 Redis 实例可以接受来自所有地址的请求，把下面的指令注释掉即可。**

### **bind 127.0.0.1**

保护模式是安全防护的其中一层，保护模式的存在是为了避免暴露在网络中的 Redis 实例被不当的连接滥用（Redis instances left open on the internet are accessed and exploited）。

当保护模式打开且：

1）Redis 服务没有使用 “bind” 去绑定明确的 ip 地址集合。

2）没有配置密码。

那么，Redis 服务只接受来自 IPv4 和 IPv6 的环回地址 127.0.0.1 和 ::1并且是来自 Unix 域的套接字。

保护模式默认开启。除非你确定你的 Redis 实例在没有配置连接认证或者使用 bind 命令限制特定的 ip 连接的情况下还可以被连接。不然最好保持该模式开启。

### **protected-mode yes**

通过特定端口进行连接，默认端口是 6379（IANA #815344）。如果端口配置成 0，Redis 就不会监听 TCP 套接字。

### **port 6379**

TCP listen() 积压（backlog）。

在高频请求场景下的 Redis，为了避免慢的客户端连接，你需要配置较高的 backlog。提醒事项：Linux 内核会默默的将其截断成 /proc/sys/net/core/somaxconn 的值，所以保证同时提高 somaxconn 和 tcp_max_syn_backlog 的值以求预期的效果。

### **tcp-backlog 511**

**Unix 套接字**

自己指定特定的 Unix 套接字路径来监听可能来的连接。Redis 没有为此配置默认值，如果你也没有手动去配置指定的话，那 Redis 不会监听一个 unix 套接字。

**#unixsocket /tmp/redis.sock**

**#unixsocketperm 700**

**N 秒后**（0 表示此配置无效），客户端和服务端之间是空闲的，则断开连接。

### **timeout 0**

**TCP keepalive**

如果配置了非零的值，使用 SO_KEEPALIVE 发送 TCP 的 ACKs 给那些可能断连的客户端。这很管用，原因有：

1）检测死掉的同伴链接（Detect dead peers）。

2）从中间网络设备的视角来看，连接持续保存。

在 Linux，配置特定的值（单位为 秒）为周期来发送 ACKs。注意事项：需要两倍的该时间来关闭连接。不同的内核中该周期取决于内核的配置。

300 秒是一个比较合理的选择，这也是 Redis 从 3.2.1 版本开始配置的默认值。

### tcp-keepalive 300

## GENERAL

Redis 运行默认不是守护进程。需要的话将该项配置成 yes。

注意事项：该配置开启后，Redis 会默认在 /var/run/redis.pid 文件中写相关信息。

### daemonize no

如果你是以 upstart 或者 systemd 方式跑 Redis，Redis 可以与你的监督数（supervision tree）交互。具体的选项：

- supervised no    - 不进行监督树的交互。
- supervised upstart    - 通过将 Redis 置为 SIGSTOP 模式进行 upstart 信号通知。
- supervised systemd    - 通过将 READY=1 写入 $NOTIFY_SOCKET 进行 systemd 的信号通知。
- supervised auto    - 基于 UPSTART_JOB 或者 NOTIFY_SOCKET 环境变量来检测是 upstart 还是 systemd 方式。

注意：以上的 supervision 方法只通知 “处理准备就绪” 的信号。他们不会持续的响应你配置的 supervisor。

### supervised no

如果配置指定了 pid 文件，Redis 就用该配置的 pid 文件写入，退出的时候移除对应的 pid 文件。

如果 Redis 是以非守护进程模式的运行，又没有配置指定的 pid 文件，那么不会创建 pid 文件。如果 Redis 是守护进程的模式，即使没有配置指定的 pid 文件，会默认使用 “/var/run/redis.pid”文件。

最好创建一个 pid 文件（Creating a pid file is best effort）：没有创建 pid 文件不会有任何影响，Server 还是会正常运行。

### pidfile /var/run/redis_6379.pid

指定 Server 的日志级别（Specify the server**verbosity**level）。

有以下四种级别：

- debug（包含许多具体信息，开发/测试 环境下很方便）
- verbose（包含许多不常用的信息，但没有 debug 级别那么混乱）
- notice（moderately verbose，不多不少，很适合生产环境）
- warning（只记录重要或者非常的信息）

### loglevel notice

指定 log 文件名。配置成空串的话可以强制 Redis 在标准输出记录日志。注意事项：如果你使用标准输出进行日志记录且是以 守护进程 的模式运行，日志会在 /dev/null 中。

### logfile ""

想让日志记录到系统日志，设置 'syslog-enabled' 成 yes，使用 syslog 带有的其他配置选项来满足你的需求。

### #syslog-enabled no

指定 syslog 的身份。

### #syslog-ident redis

指定 syslog 工具（facility）。一定要是 USER 或者在 LOCAL0-LOCAL7 之间。

### #syslog-facility local0

设置数据库的号码。默认的数据库号是 DB 0，你在每个连接中，通过 SELECT <dbid>，选择一个 0~databases-1 的数来配置特定的数据库号。

### databases 16

Redis 会在启动的时候，如果标准输出日志是 TTY，则会在开始记录标准输出日志的时候展示一个 ASCII 字符组成的 Redis logo。也就是说，通常只在交互的会话中会展示该 logo。

### always-show-logo yes

## SNAPSHOTTING（快照）

在硬盘保存数据库：

#save <seconds> <changes>，如果 seconds 和 写操作都配置了，那么一旦达到了配置条件 Redis 会将 DB 保存到硬盘。

以本配置文件的默认配置举例，达到了以下条件会触发写磁盘：

900 秒内（15 分钟）且数据库中至少有一个 key 被改变。

300 秒内（5 分钟）且数据库中至少有10 个 key 被改变。

60 秒内 且数据库中只有一个 10000 个 key 被改变。

提醒：你可以通过注释以下所有的 save 配置行以取消该功能。

也可以通过添加一个带空串的 save 指令来让配置的 save 选择失效。比如：

save ""

save 900 1

save 300 10

### save 60 10000

在开启了 RDB 快照后，如果最近的一次 RDB 快照在后台生成失败的话，Redis 默认会拒绝所有的写请求。这么做的目的是为了让用户注意到后台持久化可能出现了问题。否则用户可能一直无法注意到问题，进而可能导致灾难级别的事情发生。

如果后台存储（bgsave）能继续顺利工作，Redis 会自动的继续处理写请求。

但是，如果你已经为你的 Redis 实例和持久化配置了合适的监控手段，且希望 Redis 在非理想情况下（比如硬盘问题，权限问题等等）仍继续提供服务，可以将此项配置为 no。

### stop-writes-on-bgsave-error yes

想要在生成 rdb 文件的时候使用 LZF 压缩 String 对象？

将该配置保持默认为 ‘yes’ 几乎不会出现意外状况。（it's almost alwats a win）

可以将该配置设置为 “no” 来节省 CPU 开销。但是那些原本可以被压缩的 key 和 value 会让数据集更大。

### rdbcompression yes

从 5.0 版本开始 RDB 文件的末尾会默认放置一个 CRC64 的校验码。

这会让文件的格式更加容易检验验证，代价是生成和加载 RDB 文件的性能会损失 10% 左右。你可以把该配置关闭以求更佳的性能。

没有开启校验码配置的 RDB 文件会将校验码设置为 0，加载该文件的程序就会跳过校验过程。

### rdbchecksum yes

配置 rdb 文件的名称。

### dbfilename dump.rdb

存储 rdb 文件的目录。

数据库会使用该配置放置 rdb 文件，文件的名字使用上面的 'dbfilename' 指定的文件名。

AOF 文件的存储位置也会使用这个配置项。

注意：配置一个目录而不是文件名。

### dir ./

## REPLICATION（复制）

主从复制。使用 replicaof 来让一个 Redis 实例复制另一个 Redis 实例。接来下是关于 Redis 复制需要了解的一些事情。

![图片](https://uploader.shimo.im/f/kDlcVKaxitCjA9FA.png!thumbnail)

1）Redis 复制时异步进行的，但是可以通过配置让 Redis 主节点拒绝写请求：配置会给定一个值，主节点至少需要和大于该值的从节点个数成功连接。

2）如果 Redis 从节点和主节点意外断连了很少的一段时间，从节点可以向主节点进行**增量复制**。你可以根据你的需要配置复制的备份日志文件大小（在下一部分可以看到相关的配置）

3）复制会自动进行且不需要人为介入（intervention）。在网络划分后复制会自动与主节点重连且同步数据。

### #replicaof <masterip> <masterport>

如果主节点配置了密码（使用了 "requirepass" 配置项），从节点需要进行密码认证才能进行复制同步的过程，否则主节点会直接拒绝从节点的复制请求。

### #masterauth <master-password>

当复制过程与主节点失去连接，或者当复制正在进行时，复制可以有两种行为模式：

1）如果 replica-serve-stale-data 设置为 'yes'（默认设置），从节点仍可以处理客户端请求，但该从节点的数据很可能和主节点不同步，从节点的数据也可能是空数据集，如果这是与主节点进行的第一次同步。

2）如果 replica-serve-stale-data 设置成 'no'，从节点会对除了 INFO，replicaOF，AUTH，PING，SHUTDOWN，REPLCONF，ROLE，CONFIG，SUBSCRIBE，UNSUBSCRIBE，PSUBSCRIBE，PUNSUBSCRIBE，PUBLISH，PUBSUB，COMMAND， POST，HOST： and LATENCY 这些命令之外的请求均返回 "SYNC with master in process"。

### replica-serve-stale-data yes

可以配置从节点是否可以处理写请求。针对从节点开启写权限来存储时效低的（ephemeral）数据可能是一种有效的方式（因为写入到从节点的数据很可能随着重新同步而被删除），但是开启该配置也会导致一些问题。

从 Redis 2.6 开始从节点默认是仅可读的。

提示：可读的从节点一般不会暴露给网络中不信任的客户端。这仅是针对不正确使用实例的一层保护。从节点默认仍会响应管理层级的命令，比如 CONFIG，DEBUG 等等。在一定程度上可以使用 'rename-command' 避免那些 管理/危险 的命令，提高安全性（To a limited extent you can improve security of read only replicas using 'rename-command' to shadow all the administrative / dangerous commands）。

### replica-read-only yes

同步复制策略：硬盘或者套接字。

------

警告：不使用硬盘的复制策略目前还在实验阶段

------

新建立连接和重连的副本不会根据数据情况进行恢复传输，只会进行全量复制。主节点会传输在从节点之间传输 RDB 文件。传输行为有两种方式：

1）硬盘备份：Redis 主节点创建一个子进程来向硬盘写 RDB 文件。之后由父进程持续的文件传给副本。

2）不使用硬盘：Redis 主节点建立一个进程直接向副本的网络套接字写 RDB 文件，不涉及到硬盘。

对于方式 1，在生成 RDB 文件时，多个副本会进行入队并在当前子进程完成 RDB 文件时立即为副本进行 RDB 传输。

对于方式 2，一旦传输开始，新来的副本传输请求会入队且只在当前的传输断开后才建立新的传输连接。

如果使用方式 2，主节点会等待一段时间，根据具体的配置，等待是为了可以在开始传输前可以有期望的副本同步请求到达，这样可以使用并行传输提高效率。

对于配置是比较慢的硬盘，而网络很快（带宽大）的情况下，使用方式 2 进行副本同步会更适合。

### repl-diskless-sync no

如果 diskless sync 是开启的话，就需要配置一个延迟的秒数，这样可以服务更多通过 socket 传输 RDB 文件的副本。

这个配置很主要，因为一旦传输开始，就不能为新来的副本传输服务，只能入队等待下一次 RDB 传输，所以该配置一个延迟的值就是为了让更多的副本请求到达。

延迟配置的单位是秒，默认是 5 秒。不想要该延迟的话可以配置为 0 秒，传输就会立即开始。

### repl-diskless-sync-delay 5

副本会根据配置好的时间间隔（interval）想主节点发送 PING 命令。可以通过 repl_ping_replica_period 配置修改时间间隔。默认为 10 秒。

### #repl-ping-replica-period 10

下面的配置会将副本进行超时处理，为了：

1）在副本的角度，在同步过程中批量进行 I/O 传输。

2）从副本s的角度，主节点超时了。

3）从主节点的角度，副本超时了。

需要重视的一点是确保该选项的配置比 repl-ping-replica-period 配置的值更高，否则每次主从之间的网络比较拥挤时就容易被判定为超时。

### #repl-timeout 60

同步过后在副本套接字上关闭 TCP_NODELAY？

如果你选择了 'yes' ，Redis 会使用很小的 TCP 包，占用很低的带宽来想副本发送数据。但是这么做到达副本的数据会有一些延迟，使用默认的配置值且是 Linux 内核该延迟最多可能 40 毫秒。

如果你选择 'no'，副本的数据延迟会更低但是占用的带宽会更多一些。

我们默认会为了低延迟进行优化，但是在比较拥挤网络情况下或者是主节点和副本之间的网络情况比较复杂，比如中间有很多路由跳转的情况下，把选项设置为 'yes' 应该会比较适合。

### repl-disable-tcp-nodelay no

配置副本的缓冲区（backlog）大小。该缓冲区用来在副本断开连接后暂存副本数据。这样做的因为但副本重新连接后，不一定要重新进行全量复制，很多时候增量复制同步（仅同步断连期间副本可能丢失的数据）完全足够了。

配置的缓冲区越大，副本可以承受的断连时间可以更长。

至少有一个副本连接时缓冲区才会进行分配。

### #repl-backlog-size 1mb

主节点如果一段时间没有副本连接，上面提到的缓冲区会被释放。你可以通过配置一个指定的时间来释放缓冲区，如果主节点在这个时间内还没有与新的副本建立连接。

需要注意的是副本不会因为超时释放缓冲区，因为副本可能会被晋升（promot）为主节点，需要保持对其他副本进行增量复制的能力：因此他们总是积累缓冲区。

配置为 0 意味着不释放缓冲区。

### #repl-backlog-ttl 3600

副本的优先级是一个整型树字，可以由 Redis 的 INFO 命令显示。优先级的作用在于当主节点无法提供服务后，Redis 哨兵会使用到优先级进行选举副本，晋升为主节点。

值越低，代表该副本晋升成为主节点的优先级越高，比如说有三个副本，优先级的值分别为 10，100，25，Redis 哨兵会选择最低的那个，即优先级配置为10的那个。

但是，一个特殊的配置值 '0'，意味着该副本不可能充当主节点的角色，故优先级配置为 0 的副本永远不会被 Redis 哨兵选择晋升。

默认的优先级配置时 100.

### replica-priority 100

主节点可以根据目前连接的延迟小于 M 秒的副本数量，选择是否拒绝写请求。

数量 N 的副本需要是 "online" 的状态。

延迟的秒数（The lag（落后） in seconds） M ，计算方式是根据上一次副本发送 ping 命令到主节点的时间计算。通常每秒都会发送 ping 命令。

这个选项不保证 N 个副本会接受写请求，但是如果没有足够的副本可用，则会限制那些丢失写请求的暴露窗口至特定的秒数（This option does not GUARANTEE that N replicas will accept the write, but will limit the window of exposure for lost writes in case not enough replicas are available, to the specified number of seconds.）

比如要求至少有三个延迟小等于 10 秒的副本，你可以这么配置：

### #min-replicas-to-write 3

### #min-replicas-max-lag 10

配置设置为 0 会关闭该功能。

默认的 min-replicas-to-write 被设置为 0（功能关闭），min-replicas-max-lag 设置为 10.

主节点应该有多种方式来列举出依附与它的副本的信息（ip 和 port）。比如 "INFO replication" 就可以提供这些信息，它也会被其他的功能使用，比如 Redis 哨兵就会使用该命令列举副本实例。还有一种方式是在主节点运行 "ROLE" 命令来获取这些信息。

副本获取监听的 IP 和 地址分别通过以下的方式：

- IP：IP 地址在副本和主节点建立的 socket 连接中自动被检测到。
- Port：端口信息会在副本进行复制的 TCP 握手中交流传递，端口也是副本用来监听连接的一部分。

然而，如果使用了端口转发或者 NAT（Network Address Translation），实际连接到副本很可能通过的是不同的 IP 和 端口对。下面的两个配置选项用来让副本上报特定的 IP 和 端口 集合给它连接的主节点，之后主节点使用 "INFO" 或者 "ROLE" 命令都可以输出这些上报的值。

如果你只想上报 ip 或 端口其中一个，就没有必要两个都使用。

### #replica-announce-ip 5.5.5.5

### #replica-announce-port 1234

## SECURITY（安全）

要求客户端先使用命令 AUTH <PASSWORD> 进行认证，才能处理其他命令。 在一个可不信的环境，也就是说你不想所有知道该主机的客户端都可以与之建立连接的情况下很有用。

该配置为了向后的兼容器应该保持被注释不使用，因为大多数的使用者不需要认证（e.g. 他们只是在自己的机器上跑实例）

警告：因为 Redis 的响应速率很快，所以恶意攻击者可能在每秒中发送 150k 数据量的密码尝试解密。这意味着你设置的密码强度要足够大，否则很容易被破解。

### #requirepass foobared

命名的重命名。

可以在共享的环境中重命名那些比较危险的命令。比如把 CONFIG 命令重命名成一个不好猜的名字，这样内部的功能还可以使用，且可以避免大部分的客户端使用。

例如：rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52

甚至可以将命名重命名成一个空串，使其失效。

### #rename-command CONFIG ""

请注意修改命令名称的行为会记录在 AOF 文件中或传输到副本可能会导致意外情况。

## 

## CLIENTS（客户端）

设置可以同时连接客户端的最大数量。默认该项设置为 10000 个客户端，但是如果 Redis server 不能配置过程文件来限制最大的同时连接数，那么实际的最大连接数会变成当前文件配置的数组再减去 32（因为 Redis 内部需要维护一部分文件描述符）

一旦达到该限制数 Redis 会拒绝所有的新连接并返回错误信息 'max number of clients reached'。

### #maxclients 10000

## MEMEORY MANAGEMENT（内存管理）

设置限定的最大内存使用。

但内存使用达到限制 Redis 会根据配置的淘汰策略（见 maxmemory-policy）移除键值对。

如果根据淘汰策略，Redis 不能移除键值对，Redis 会拒绝那些申请更大内存的命令，比如 SET，LPUSH 等等，但是仍可以处理读请求，比如 GET 等。

该选项对那些使用 Redis 进行 LRU，LFU 缓存系统或者硬性限制内存很友好（使用 'noeviction' 策略）。

警告：如果你为实例配置了 maxmemory，且该实例配置了子节点，那么已使用内存的大小就需要加上为副本配置的输出缓冲区的大小。这样因为 网络问题/重新同步 不会一直触发键的淘汰行为。相反的，副本缓冲区中充满了对键的删除或淘汰的情况可能触发更多 key 被淘汰，以此类推直到库完全被清空。

> WARNING: If you have replicas attached to an instance with maxmemory on, the size of the output buffers needed to feed the replicas are subtracted from the used memory count, so that network problems / resyncs will not trigger a loop where keys are evicted, and in turn the output buffer of replicas is full with DELs of keys evicted triggering the deletion of more keys, and so forth until the database is completely emptied.

简单说就是，如果你为实例配置了副本，那么建议你设置一个较低的 maxmemory 值，这样系统中就有更多的内存空间留给 副本缓冲区（如果淘汰策略是 'noeviction' 那上面说的就没有必要）。

### #maxmemory  <bytes>

MAXMEMORY POLICY：在内存使用达到 maxmemory 后，Redis 如何选择 键值对 进行淘汰。有以下几种：

- volatile-lru，使用 LRU 算法，在设置了过期时间的 key 中选择。
- allkeys-lru，使用 LRU 算法，在所有的 key 中选择。
- volatile-lfu，使用 LFU 算法，在设置了过期时间 key 中选择。
- allkeys-lfu，使用 LFU 算法，在所有的 key 中选择。
- volatile-random，在设置了过期时间的 key 中随机选择。
- allkeys-random，在所有 key 中随机选择。
- volatile-ttl，在设置了过期时间的 key 中，选择过期时间最近的 key。
- noeviction，不淘汰 key ，对任何写操作（使用额外内存）返回错误。

LRU 代表最近最少未使用。

LFU 代码最近最不常使用。

LRU，LFU 和 volatile-ttl 均由近似的随机算法实现。

提示：不管采用了以上的哪种策略，对于新的写请求，如果没有合适的 key 可以淘汰，Redis 均会响应一个 error。

比如如下的写命令：

set setnx setex append incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd

sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby getset mset msetnx exec sort。

默认策略是：

### #maxmemory-policy noeviction

LRU，LFU 以及最小 TTL 的实现都不是精确的而是比较粗略的近似算法（为了节省内存），为了速度或者精确度，你可以进行相应的配置。默认 Redis 会检查 5 个 key，在其中选择最近最少使用的，你也可以直接在下面的配置项中配置 Redis 选择的样本数量。

默认配置的值是 5，已经可以有一个很完美的结果。10 的话可能会让选择策略更像真正意义上的 LRU 算法，但是需要更多 CPU 资源。3 的话会更快，但是不够精确。

### #maxmemory-samples 5

从 Redis 5.0 之后，副本默认会忽略为其配置的 maxmemory 选项（除非因为故障转移（failover）或者选择将其晋升为主节点）。也就是说 key 的淘汰只会由主节点执行，副本对应的是主节点发送对应的删除命令给副本作为 key 的淘汰方式。

这个行为模式保证了主副节点的一致性（这通常也是你需要的），但是如果你的副本是可写的或者你想要你的副本有不同的内存配置，而且你也很确认到达副本的写操作能保证幂等性（idempotenet），那你可以修改这个默认值（但是最好保证你理解了这么做的原因）。

提示：因为副本默认没有 maxmemory 和淘汰策略，副本实际的内存占用可能比 maxmemeory 配置的值大（可能因为副本缓冲区，或者某些数据结构占用了额外的内存等等原因）。所以确保对副本有合适的监控手段，保证在主节点达到配置的 maxmemory 设置之前，副本有足够的内存保证不会出现真正的 out-of-memory 条件。

### #replica-ignore-maxmemory yes

## LAZY FREEING（懒释放）

Redis 有两个可以删除 key 的原语（primitive）。其中一种是调用 DEL ，阻塞地删除对象。也就是说 Redis Server 需要通过同步的方式确认回收了所有和刚才删除的 key 相关的内存后，才能处理接下来的命令。如果要删除的 key 很小，执行 DEL 命令的时间也很短，和其他时间复杂度为 O(1) 或 O(log_N) 的命令差不多。但是，如果要删除的 key 涉及到一个存储着百万级别元素的集合，Redis server 就可能因此阻塞一段时间（甚至到秒的级别）。

由于同步的处理方式可能带来的问题，Redis 提供了非阻塞的删除原语比如 UNLINK 以及异步的选项比如 FLUSHALL 和 FLUSHDB 命名，为的就是在后台回收内存。这些命名会在固定时间执行（in constant time）。另外的线程会在后台以尽可能快的速度释放这些对象。

DEL，UNLINK 和带有 ASYNC 选项的 FLUSHALL 和 FLUSHDB 命名都可以由用户控制。这取决于应用层面是否理解且合适的使用相应的命令来达到目的。但是还是有一些情况要注意，Redis 有时会因为其他操作的副作用导致触发 key 的删除或者刷新整个数据库。特别是在用户调用了对象删除的以下场景：

1. 在淘汰策略下，因为配置了 maxmemory 和 maxmemory policy，为了在不超过配置的内存限制下腾出空间给新来的数据。
2. 因为过期时间的配置：当一个 key 配置了 expire 时间且时间到了，那它必须从内存中移除。
3. 命名在已经存在的 key 上进行数据的存储操作的副作用。比如 RENAME 命名在替换的时候需要删除原本的 key 的内容。类似的带有 STORE 选项的 SUNIONSTORE 或者 SORT 命名可能会删除已存在的 key。SET 命令本身为了用新的值替换，会将要操作的 key 的旧值先删除掉。
4. 在 REPLICATION 期间，当副本执行了全量同步复制，副本的整个数据库会被清空，然后加载传输来的 RDB 文件。

上面的场景在默认情况下都是以阻塞的方式删除对象，比如调用 DEL 的时候。你在本配置项中为每个场景进行配置，这样就可以像 UNLINK 被调用时以非阻塞的方式释放内存。

### lazyfree-lazy-eviction no

### lazyfree-lazy-expire no

### lazyfree-lazy-server-del no

### lazyfree-lazy-flush no

## APPEND ONLY MODE（附加模式）

Redis 默认使用异步方式转储文件到硬盘。这种模式在很多应用场景下都很适用，但是在 Redis 处理出现问题或者设备断电的意外期间可能丢失相应的写操作（取决于 save 配置的时间点）。

AOF 文件是 Redis 提供的另外一种提供更好的持久性的持久化模式。例如如果使用默认的数据传输策略（根据之后提供的配置）Redis 在发生意外情况下比如设备断电，或者 Redis 本身的进程出现了一些问题的情况下（操作系统正常运行），Redis 可以仅仅丢失 1 秒钟的写操作。

AOF 和 RDB 的持久化策略可以同时启用。如果打开了 AOF，Redis 启动时会加载 AOF，因为 AOF 的持久化表现更好。

点击[http://redis.io/topics/persistence](http://redis.io/topics/persistence)获取更多相关的信息。

### appendonly on

AOF 的文件名（默认："appendonly.aof"）

### appendfilename "appendonly.aof"

函数 fsync() 会告诉操作系统立即把数据写到磁盘上而不是等输出缓冲区有更多的数据时才进行。有些 OS 会马上把数据刷到硬盘，有些 OS 只保证尽快进行刷盘操作。

Redis 支持三种模式：

no：不 fsync，让操作系统来决定什么时候进行刷盘。最不会影响 Server 响应。

always：每写入 aof 文件就进行 fsync。影响 Server 响应，但是数据更安全。

everysec：每秒进行 fsync。最稳健的形式。

默认的模式是 everysec，在响应速度和数据安全方面最稳妥的选择。以上三种模式的选择都取决你对应用的理解，选择 no ，让 OS 选择写入时机，这样有更好的性能表现（但是如果你的业务可以忍受一些数据的丢失，其实你可以考虑使用默认的持久化策略 - RDB）。又或者使用 always，可以会让响应变慢一些但是数据的安全性会更高。

更多的相关知识戳下面的文章链接：

[http://antirez.com/post/redis-persistence-demystified.html](http://antirez.com/post/redis-persistence-demystified.html)

如果你不确定选哪种的话，那就用 "everysec" 吧。

### #appendfsync always

### appendfsync everysec

### #appendfsync no

当 AOF fsync 策略是 always 或者  everysec，会启动一个后台进程（后台进行保存或者 AOF 文件的后台重写），该进程会在磁盘上频繁的 I/O，在一些 Linux 配置下 Redis 的 fsync() 调用可能会阻塞太久。需要注意的是目前还没有相应的优化策略，极端情况下在不同线程进行的  fsync 可能阻塞同步的 write(2) 调用。

为了减缓上面提到的问题，可以在主线程调用 BGSAVE 或者 BGREWRITEAOF 命名避免 fsync() 在主线程上调用。

这意味着但其他的子节点在保存的时候，Redis 的持久化就和 "appendfsync none" 策略一样。这意味着在实际中的最糟糕的场景下（在默认的 Linux 配置下）有可能丢失超过 30s 时间粒度的 log。

如果你的应用不能忍受延迟问题，将下面的选项配置为 "yes"。否则保持为 "no"，这样才持久化的角度上是最安全的选择。

### no-appendfsync-on-rewrite no

































