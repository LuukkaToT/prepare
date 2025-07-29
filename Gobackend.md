## Go

### 1. 协程池的作用？
**答案：**
- **任务复用**：避免频繁创建和销毁goroutine的开销
- **资源控制**：限制并发goroutine的数量，防止系统资源耗尽
- **性能优化**：减少调度开销，提高系统吞吐量
- **内存管理**：避免goroutine过多导致的内存溢出

### 2. 内存逃逸分析？
**答案：**
内存逃逸是指本应分配在栈上的变量被分配到堆上的现象。
**逃逸场景：**
- 指针逃逸：函数返回局部变量的指针
- 栈空间不足：变量太大，栈容纳不下
- 动态类型：interface{}类型
- 闭包引用：闭包函数引用了外部变量

**分析工具：**
```bash
go build -gcflags=-m main.go
```

### 3. Go的内存回收什么条件会触发？Go的GC能够手动触发吗？
**答案：**
**触发条件：**
- 内存分配量达到上次GC后内存的2倍时
- 距离上次GC超过2分钟时
- 手动调用runtime.GC()

**手动触发：**
```go
runtime.GC() // 可以手动触发GC
runtime.ReadMemStats() // 读取内存统计信息
```

### 4. channel的底层实现？有缓冲和无缓冲的channel区别？
**答案：**
**底层结构：**
```go
type hchan struct {
    qcount   uint           // 队列中数据个数
    dataqsiz uint           // 环形队列大小
    buf      unsafe.Pointer // 环形队列指针
    elemsize uint16         // 元素大小
    closed   uint32         // 是否关闭
    sendx    uint           // 发送索引
    recvx    uint           // 接收索引
    recvq    waitq          // 接收等待队列
    sendq    waitq          // 发送等待队列
    lock     mutex          // 互斥锁
}
```

**区别：**
- **无缓冲channel**：同步通信，发送和接收必须同时准备好
- **有缓冲channel**：异步通信，可以存储数据，当缓冲区满时才阻塞

**关闭后读取：**
- 已关闭的channel读取会返回零值和false
- 写入已关闭的channel会panic

### 5. 切片使用时需要注意什么？
**答案：**
- **扩容机制**：当容量不足时会重新分配内存
- **共享底层数组**：多个slice可能共享同一个底层数组
- **内存泄漏**：大slice的小部分引用可能导致整个底层数组无法释放
- **并发安全**：slice不是并发安全的

### 6. Go中的参数传递是值传递还是引用传递？
**答案：**
Go语言只有值传递，包括：
- **基本类型**：直接复制值
- **指针**：复制指针的值（指针指向的地址）
- **slice、map、channel**：复制的是结构体，但结构体中包含指向底层数据的指针

### 7. defer的执行顺序？
**答案：**
- **LIFO顺序**：后进先出，栈的方式执行
- **参数预计算**：defer后面的参数会立即计算
- **执行时机**：函数return之后，真正返回调用者之前

**示例：**
```go
func test() {
    defer fmt.Println("1")
    defer fmt.Println("2")
    defer fmt.Println("3")
}
// 输出：3 2 1
```

### 8. GMP模型？
**答案：**
- **G (Goroutine)**：协程，轻量级线程
- **M (Machine)**：系统线程，真正执行代码的实体
- **P (Processor)**：逻辑处理器，调度G到M上执行

**调度流程：**
1. G优先从P的本地队列获取
2. 本地队列为空时从全局队列获取
3. 全局队列为空时从其他P窃取
4. G阻塞时，M会寻找其他可执行的G

### 9. make和new的区别？
**答案：**
- **new**：分配内存并返回指针，内存值为零值
- **make**：只能用于slice、map、channel，返回初始化后的对象

```go
// new
p := new(int)    // 返回*int，值为0
s := new([]int)  // 返回*[]int，值为nil

// make
s := make([]int, 5)    // 返回[]int，长度为5
m := make(map[string]int) // 返回map[string]int
c := make(chan int)    // 返回chan int
```

### 10. Go的map是并发安全的吗？如何解决？
**答案：**
map不是并发安全的，并发读写会panic。

**解决方案：**
1. **sync.RWMutex**：读写锁保护
2. **sync.Map**：官方并发安全的map
3. **channel**：通过channel序列化访问

```go
// sync.Map
var m sync.Map
m.Store("key", "value")
value, ok := m.Load("key")
```

### 11. 空结构体的用途？会分配内存吗？
**答案：**
**用途：**
- 实现集合（set）
- 作为channel的信号
- 作为方法接收者（只关心方法不关心数据）

**内存分配：**
空结构体不会分配内存，大小为0。

```go
var s struct{}
fmt.Println(unsafe.Sizeof(s)) // 输出：0

// 用作集合
set := make(map[string]struct{})
set["key"] = struct{}{}
```

## 数据库相关

### 12. MySQL索引原理？为什么使用B+树？
**答案：**
**B+树特点：**
- 所有数据存储在叶子节点
- 叶子节点通过指针连接，支持范围查询
- 非叶子节点只存储索引，提高内存利用率
- 树的高度低，减少磁盘IO

**优势：**
- 相比二叉树：更少的磁盘IO
- 相比B树：叶子节点存数据，非叶子节点存更多索引
- 相比哈希：支持范围查询

### 13. 聚簇索引和非聚簇索引的区别？
**答案：**
- **聚簇索引**：索引和数据存储在一起，叶子节点包含完整的数据行
- **非聚簇索引**：索引和数据分开存储，叶子节点存储主键值

**InnoDB特点：**
- 主键是聚簇索引
- 辅助索引通过主键值查找数据（可能需要回表）

### 14. 什么是回表？如何避免？
**答案：**
**回表**：非聚簇索引查询时，先找到主键值，再根据主键查询完整数据的过程。

**避免方法：**
- **覆盖索引**：索引包含所有查询字段
- **索引下推**：在索引层面过滤数据

### 15. 事务隔离级别？
**答案：**
1. **读未提交**：可能出现脏读、不可重复读、幻读
2. **读已提交**：避免脏读，可能出现不可重复读、幻读
3. **可重复读**：避免脏读、不可重复读，可能出现幻读（MySQL默认）
4. **串行化**：避免所有问题，但性能最差

**MySQL实现：**
- 通过MVCC（多版本并发控制）实现
- undo log保存历史版本
- read view确定可见性

### 16. 什么是幻读？MySQL如何解决？
**答案：**
**幻读**：同一事务中，多次查询返回的记录数量不一致。

**MySQL解决方案：**
- **MVCC**：快照读避免幻读
- **Next-Key Lock**：当前读时锁定索引记录和间隙

## Redis相关

### 17. Redis常用数据结构？
**答案：**
1. **String**：简单键值对，缓存、计数器
2. **Hash**：对象存储
3. **List**：消息队列、时间轴
4. **Set**：去重、交集并集
5. **ZSet**：排行榜、延时队列
6. **Bitmap**：用户签到、布隆过滤器
7. **HyperLogLog**：基数统计

### 18. Redis持久化机制？
**答案：**
1. **RDB**：快照持久化
   - 全量备份，恢复速度快
   - 可能丢失最后一次快照后的数据

2. **AOF**：命令日志持久化
   - 记录每个写命令
   - 数据安全性更高，但文件更大

3. **混合持久化**：RDB + AOF增量

### 19. Redis为什么单线程还这么快？
**答案：**
- **内存操作**：数据存储在内存中
- **高效数据结构**：针对性优化的数据结构
- **IO多路复用**：epoll等高效IO模型
- **避免上下文切换**：单线程避免锁竞争

### 20. 缓存击穿、雪崩、穿透？
**答案：**
**击穿**：热点key过期，大量请求击穿到数据库
- 解决：互斥锁、逻辑过期

**雪崩**：大量key同时过期
- 解决：过期时间加随机值、多级缓存

**穿透**：查询不存在的数据
- 解决：布隆过滤器、缓存空值

## 网络相关

### 21. TCP三次握手和四次挥手？
**答案：**
**三次握手：**
1. 客户端发送SYN
2. 服务端回复SYN+ACK
3. 客户端发送ACK

**四次挥手：**
1. 主动方发送FIN
2. 被动方回复ACK
3. 被动方发送FIN
4. 主动方回复ACK

**为什么四次**：TCP全双工，需要分别关闭两个方向的连接

### 22. TIME_WAIT的作用？
**答案：**
- **确保ACK到达**：防止最后的ACK丢失
- **防止旧连接数据**：避免新连接收到旧连接的数据
- **持续时间**：2MSL（Maximum Segment Lifetime）

### 23. HTTP和HTTPS的区别？
**答案：**
- **HTTP**：明文传输，端口80
- **HTTPS**：加密传输，端口443

**HTTPS流程：**
1. 证书验证
2. 密钥协商
3. 对称加密通信

### 24. Cookie、Session和Token的区别？
**答案：**
- **Cookie**：存储在客户端，自动发送
- **Session**：存储在服务端，通过Cookie中的SessionID关联
- **Token**：无状态，包含用户信息，如JWT

## 消息队列

### 25. Kafka和RabbitMQ的区别？
**答案：**
**Kafka：**
- **高吞吐量**：适合大数据场景，单机可达百万级TPS
- **拉模式消费**：消费者主动拉取消息
- **分区并行**：支持水平扩展
- **持久化存储**：消息持久化到磁盘
- **适用场景**：日志收集、实时计算、大数据处理

**RabbitMQ：**
- **低延迟**：毫秒级延迟，功能丰富
- **推模式消费**：服务端主动推送消息
- **多种交换机**：支持直连、主题、广播等模式
- **事务支持**：支持消息事务
- **适用场景**：业务解耦、任务队列、实时通信

### 26. 如何保证消息不丢失？
**答案：**
**生产者端：**
- 开启确认机制（acks=all）
- 重试机制和幂等性保证
- 同步发送重要消息

**Broker端：**
- 数据持久化到磁盘
- 多副本机制（replication factor > 1）
- 主从切换保证高可用

**消费者端：**
- 手动确认消息（关闭自动提交）
- 业务处理成功后再提交offset
- 消费失败时重试或进入死信队列

### 27. 如何避免消息重复消费？
**答案：**
1. **业务幂等性**：设计幂等的业务逻辑
2. **唯一键约束**：数据库层面防重复
3. **去重表**：维护已处理消息的记录
4. **分布式锁**：确保同一消息只被处理一次

```go
// 示例：使用Redis实现消息去重
func processMessage(messageID string, message Message) error {
    // 尝试设置去重标记
    result, err := redisClient.SetNX(
        "processed:"+messageID, 
        "1", 
        time.Hour*24,
    ).Result()
    if err != nil {
        return err
    }
    
    if !result {
        // 消息已处理过
        return nil
    }
    
    // 处理业务逻辑
    return handleBusinessLogic(message)
}
```

## 分布式系统

### 28. 如何设计一个分布式锁？
**答案：**
**Redis实现：**
```go
// 加锁
SET key value NX EX 30

// 解锁（Lua脚本保证原子性）
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

**要点：**
- 设置过期时间防止死锁
- 使用唯一标识防止误删
- 可重入性设计
- 看门狗机制自动续期

**其他实现方案：**
1. **ZooKeeper**：临时有序节点
2. **etcd**：lease机制
3. **数据库**：唯一约束

### 29. 分布式事务解决方案？
**答案：**
**两阶段提交（2PC）：**
- 阶段1：准备阶段，协调者询问参与者是否可以提交
- 阶段2：提交阶段，根据投票结果决定提交或回滚
- 缺点：阻塞问题，单点故障

**三阶段提交（3PC）：**
- 增加CanCommit阶段，减少阻塞时间
- 引入超时机制

**TCC模式：**
- **Try**：尝试执行业务，预留资源
- **Confirm**：确认执行业务
- **Cancel**：取消执行，释放资源

**Saga模式：**
- 将长事务分解为多个本地事务
- 通过补偿机制实现最终一致性

### 30. CAP理论？
**答案：**
- **C (Consistency)**：一致性，所有节点同时看到相同数据
- **A (Availability)**：可用性，系统持续可用
- **P (Partition tolerance)**：分区容错性，网络分区时系统仍能工作

**只能同时满足其中两个：**
- **CP系统**：强一致性系统（如传统数据库）
- **AP系统**：高可用系统（如NoSQL数据库）

### 31. 微服务的优缺点？
**答案：**
**优点：**
- 独立部署和扩展
- 技术栈灵活
- 故障隔离
- 团队独立开发

**缺点：**
- 系统复杂性增加
- 服务间通信开销
- 数据一致性难以保证
- 运维复杂度提高

## 容器化技术

### 32. Docker的底层实现原理？
**答案：**
Docker基于Linux内核的以下技术：

**Namespace：**
- **PID Namespace**：进程隔离
- **Network Namespace**：网络隔离
- **Mount Namespace**：文件系统隔离
- **UTS Namespace**：主机名隔离
- **IPC Namespace**：进程间通信隔离
- **User Namespace**：用户权限隔离

**Cgroups：**
- 限制CPU使用率
- 限制内存使用量
- 限制磁盘IO
- 限制网络带宽

**Union FS：**
- 分层文件系统
- 镜像的层次结构
- 写时复制机制

### 33. Kubernetes核心组件？
**答案：**
**Master节点：**
- **API Server**：集群的统一入口，提供RESTful API
- **etcd**：分布式键值存储，保存集群状态
- **Controller Manager**：控制器管理，维护集群状态
- **Scheduler**：Pod调度器，负责Pod分配

**Node节点：**
- **kubelet**：节点代理，管理Pod生命周期
- **kube-proxy**：网络代理，负责服务发现和负载均衡
- **Container Runtime**：容器运行时（如Docker、containerd）

### 34. Pod是什么？为什么需要Pod？
**答案：**
**Pod定义：**
- K8s中最小的部署单元
- 包含一个或多个容器
- 共享网络和存储

**为什么需要Pod：**
- **共享资源**：容器间共享网络和存储卷
- **生命周期管理**：容器统一调度和管理
- **原子性**：Pod内容器要么全部成功，要么全部失败

## Java生态（部分公司会问）

### 35. JVM垃圾回收机制？
**答案：**
**分代回收：**
- **新生代**：Eden区 + 2个Survivor区，使用复制算法
- **老年代**：长期存活对象，使用标记-清除或标记-整理

**垃圾回收器：**
- **G1**：低延迟，适合大内存应用
- **CMS**：并发标记清除，已废弃
- **ZGC**：超低延迟收集器

**GC触发条件：**
- 新生代空间不足
- 老年代空间不足
- 手动调用System.gc()

### 36. Spring IOC和AOP？
**答案：**
**IOC（控制反转）：**
- 对象创建和依赖关系由容器管理
- 降低耦合度，提高可测试性
- 通过依赖注入实现

**AOP（面向切面编程）：**
- 将横切关注点（如日志、事务）从业务逻辑中分离
- 通过代理模式实现
- 常用注解：@Aspect、@Before、@After

## 设计模式

### 37. 单例模式实现？
**答案：**
**懒汉式（延迟加载）：**
```go
type singleton struct{}

var instance *singleton
var once sync.Once

func GetInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

**饿汉式（预加载）：**
```go
var instance = &singleton{}

func GetInstance() *singleton {
    return instance
}
```

### 38. 工厂模式？
**答案：**
用于创建对象而不暴露创建逻辑：
```go
type Product interface {
    Use()
}

type ConcreteProductA struct{}
func (p *ConcreteProductA) Use() { /* 实现 */ }

type Factory struct{}
func (f *Factory) CreateProduct(productType string) Product {
    switch productType {
    case "A":
        return &ConcreteProductA{}
    default:
        return nil
    }
}
```

## Linux运维

### 39. 常用性能监控命令？
**答案：**
**CPU监控：**
- `top`：实时显示进程信息
- `htop`：增强版top
- `vmstat`：虚拟内存统计

**内存监控：**
- `free -h`：内存使用情况
- `cat /proc/meminfo`：详细内存信息

**磁盘监控：**
- `df -h`：磁盘空间使用
- `du -sh`：目录空间占用
- `iostat`：磁盘IO统计

**网络监控：**
- `netstat -tuln`：端口监听情况
- `ss -tuln`：替代netstat
- `iftop`：实时网络流量

### 40. 如何查找和处理大文件？
**答案：**
```bash
# 查找当前目录下最大的10个文件
du -ah . | sort -rh | head -10

# 查找大于100M的文件
find / -size +100M -type f 2>/dev/null

# 查看目录空间占用
du -sh /*

# 清理日志文件
find /var/log -name "*.log" -mtime +30 -delete

# 分析磁盘使用情况
df -h
lsof | grep -i deleted  # 找出已删除但仍占用空间的文件
```

## 算法与数据结构

### 41. 如何判断单链表是否有环？
**答案：**
**快慢指针法**：
```go
func hasCycle(head *ListNode) bool {
    if head == nil || head.Next == nil {
        return false
    }
    
    slow, fast := head, head.Next
    for fast != nil && fast.Next != nil {
        if slow == fast {
            return true
        }
        slow = slow.Next
        fast = fast.Next.Next
    }
    return false
}
```

### 42. 如何找到环的入口？
**答案：**
1. 快慢指针找到环内相遇点
2. 一个指针从头开始，一个从相遇点开始
3. 两指针每次都移动一步，相遇点即为环入口

### 43. LRU缓存实现？
**答案：**
使用哈希表 + 双向链表：
```go
type LRUCache struct {
    capacity int
    cache    map[int]*Node
    head     *Node
    tail     *Node
}

type Node struct {
    key   int
    value int
    prev  *Node
    next  *Node
}
```

## 编程应用题

### 44. 两个协程交替打印1-10
**问题：** 使用两个goroutine交替打印1-10
```
G1: 1
G2: 2
G1: 3
G2: 4
...
```

**答案：**
```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    ch1 := make(chan struct{}, 1)
    ch2 := make(chan struct{}, 1)
    var wg sync.WaitGroup
    
    // 启动第一个goroutine
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 1; i <= 10; i += 2 {
            <-ch1 // 等待信号
            fmt.Printf("G1: %d\n", i)
            ch2 <- struct{}{} // 发送信号给G2
        }
    }()
    
    // 启动第二个goroutine
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 2; i <= 10; i += 2 {
            <-ch2 // 等待信号
            fmt.Printf("G2: %d\n", i)
            if i < 10 {
                ch1 <- struct{}{} // 发送信号给G1
            }
        }
    }()
    
    // 启动第一个goroutine
    ch1 <- struct{}{}
    wg.Wait()
}
```

### 45. 用两个协程和两个channel分别接收1，2并打印
**问题：** 用两个协程，两个channel分别接收1，2，并打印

**答案：**
```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    var wg sync.WaitGroup
    
    // 协程1：接收ch1的数据
    wg.Add(1)
    go func() {
        defer wg.Done()
        for val := range ch1 {
            fmt.Printf("协程1接收到: %d\n", val)
        }
    }()
    
    // 协程2：接收ch2的数据
    wg.Add(1)
    go func() {
        defer wg.Done()
        for val := range ch2 {
            fmt.Printf("协程2接收到: %d\n", val)
        }
    }()
    
    // 发送数据
    go func() {
        ch1 <- 1
        close(ch1)
    }()
    
    go func() {
        ch2 <- 2
        close(ch2)
    }()
    
    wg.Wait()
}
```

### 46. goroutine+channel依次输出小猫小狗100次
**问题：** 使用goroutine和channel依次输出小猫小狗100次

**答案：**
```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    catCh := make(chan struct{}, 1)
    dogCh := make(chan struct{}, 1)
    var wg sync.WaitGroup
    
    // 小猫协程
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 0; i < 100; i++ {
            <-catCh
            fmt.Printf("小猫%d\n", i+1)
            dogCh <- struct{}{}
        }
    }()
    
    // 小狗协程
    wg.Add(1)
    go func() {
        defer wg.Done()
        for i := 0; i < 100; i++ {
            <-dogCh
            fmt.Printf("小狗%d\n", i+1)
            if i < 99 {
                catCh <- struct{}{}
            }
        }
    }()
    
    // 启动
    catCh <- struct{}{}
    wg.Wait()
}
```

### 47. 控制并发数量：100个任务最多10个并发执行
**问题：** 有100个并发任务，需要控制最多只有10个同时执行

**答案：**
```go
package main

import (
    "fmt"
    "sync"
    "time"
)

func main() {
    // 方法1：使用带缓冲的channel作为信号量
    semaphore := make(chan struct{}, 10) // 最多10个并发
    var wg sync.WaitGroup
    
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(taskID int) {
            defer wg.Done()
            
            // 获取信号量
            semaphore <- struct{}{}
            defer func() { <-semaphore }() // 释放信号量
            
            // 执行任务
            fmt.Printf("执行任务 %d\n", taskID)
            time.Sleep(100 * time.Millisecond) // 模拟任务执行
        }(i)
    }
    
    wg.Wait()
    fmt.Println("所有任务完成")
}

// 方法2：使用worker pool模式
func workerPoolExample() {
    taskCh := make(chan int, 100)
    var wg sync.WaitGroup
    
    // 启动10个worker
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()
            for taskID := range taskCh {
                fmt.Printf("Worker %d 执行任务 %d\n", workerID, taskID)
                time.Sleep(100 * time.Millisecond)
            }
        }(i)
    }
    
    // 发送100个任务
    for i := 0; i < 100; i++ {
        taskCh <- i
    }
    close(taskCh)
    
    wg.Wait()
}
```

### 48. 快速排序实现
**问题：** 用Go实现快速排序

**答案：**
```go
package main

import "fmt"

func quickSort(arr []int, low, high int) {
    if low < high {
        // 分区操作，返回基准元素的位置
        pi := partition(arr, low, high)
        
        // 分别对基准元素左右的子数组进行排序
        quickSort(arr, low, pi-1)
        quickSort(arr, pi+1, high)
    }
}

func partition(arr []int, low, high int) int {
    // 选择最后一个元素作为基准
    pivot := arr[high]
    i := low - 1 // 小于基准元素的最后一个位置
    
    for j := low; j < high; j++ {
        // 如果当前元素小于等于基准元素
        if arr[j] <= pivot {
            i++
            arr[i], arr[j] = arr[j], arr[i] // 交换
        }
    }
    
    // 将基准元素放到正确位置
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1
}

func main() {
    arr := []int{64, 34, 25, 12, 22, 11, 90}
    fmt.Println("排序前:", arr)
    
    quickSort(arr, 0, len(arr)-1)
    fmt.Println("排序后:", arr)
}
```

### 49. 反转链表
**问题：** 反转单向链表

**答案：**
```go
package main

import "fmt"

type ListNode struct {
    Val  int
    Next *ListNode
}

// 迭代方法
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    curr := head
    
    for curr != nil {
        next := curr.Next  // 保存下一个节点
        curr.Next = prev   // 反转指针
        prev = curr        // prev向前移动
        curr = next        // curr向前移动
    }
    
    return prev // prev是新的头节点
}

// 递归方法
func reverseListRecursive(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    
    // 递归反转后面的链表
    newHead := reverseListRecursive(head.Next)
    
    // 反转当前节点
    head.Next.Next = head
    head.Next = nil
    
    return newHead
}

// 打印链表
func printList(head *ListNode) {
    for head != nil {
        fmt.Printf("%d -> ", head.Val)
        head = head.Next
    }
    fmt.Println("nil")
}

func main() {
    // 创建链表: 1 -> 2 -> 3 -> 4 -> 5
    head := &ListNode{Val: 1}
    head.Next = &ListNode{Val: 2}
    head.Next.Next = &ListNode{Val: 3}
    head.Next.Next.Next = &ListNode{Val: 4}
    head.Next.Next.Next.Next = &ListNode{Val: 5}
    
    fmt.Println("原链表:")
    printList(head)
    
    // 反转链表
    reversed := reverseList(head)
    fmt.Println("反转后:")
    printList(reversed)
}
```

### 50. 完整的LRU缓存实现
**问题：** 实现一个完整的LRU缓存

**答案：**
```go
package main

import "fmt"

type Node struct {
    key, value int
    prev, next *Node
}

type LRUCache struct {
    capacity int
    cache    map[int]*Node
    head     *Node // 虚拟头节点
    tail     *Node // 虚拟尾节点
}

func Constructor(capacity int) LRUCache {
    lru := LRUCache{
        capacity: capacity,
        cache:    make(map[int]*Node),
        head:     &Node{},
        tail:     &Node{},
    }
    
    // 初始化双向链表
    lru.head.next = lru.tail
    lru.tail.prev = lru.head
    
    return lru
}

func (lru *LRUCache) Get(key int) int {
    if node, exists := lru.cache[key]; exists {
        // 移动到头部
        lru.moveToHead(node)
        return node.value
    }
    return -1
}

func (lru *LRUCache) Put(key int, value int) {
    if node, exists := lru.cache[key]; exists {
        // 更新值并移动到头部
        node.value = value
        lru.moveToHead(node)
    } else {
        newNode := &Node{key: key, value: value}
        
        if len(lru.cache) >= lru.capacity {
            // 删除尾部节点
            tail := lru.removeTail()
            delete(lru.cache, tail.key)
        }
        
        // 添加到头部
        lru.cache[key] = newNode
        lru.addToHead(newNode)
    }
}

func (lru *LRUCache) addToHead(node *Node) {
    node.prev = lru.head
    node.next = lru.head.next
    
    lru.head.next.prev = node
    lru.head.next = node
}

func (lru *LRUCache) removeNode(node *Node) {
    node.prev.next = node.next
    node.next.prev = node.prev
}

func (lru *LRUCache) moveToHead(node *Node) {
    lru.removeNode(node)
    lru.addToHead(node)
}

func (lru *LRUCache) removeTail() *Node {
    lastNode := lru.tail.prev
    lru.removeNode(lastNode)
    return lastNode
}

func main() {
    lru := Constructor(2)
    
    lru.Put(1, 1)
    lru.Put(2, 2)
    fmt.Println(lru.Get(1)) // 返回 1
    
    lru.Put(3, 3)           // 该操作会使得关键字 2 作废
    fmt.Println(lru.Get(2)) // 返回 -1 (未找到)
    
    lru.Put(4, 4)           // 该操作会使得关键字 1 作废
    fmt.Println(lru.Get(1)) // 返回 -1 (未找到)
    fmt.Println(lru.Get(3)) // 返回 3
    fmt.Println(lru.Get(4)) // 返回 4
}
```

---

这份完整的面试题文档现在包含了**50个**核心问题，涵盖了Go后端开发面试的所有重要技术领域：

- **Go语言基础**（11题）
- **数据库相关**（5题）
- **Redis相关**（4题）
- **网络相关**（4题）
- **消息队列**（3题）
- **分布式系统**（4题）
- **容器化技术**（3题）
- **Java生态**（2题）
- **设计模式**（2题）
- **Linux运维**（2题）
- **算法与数据结构**（3题）
- **编程应用题**（7题）

每个问题都提供了详细的标准答案和代码实现，确保涵盖面试中的核心知识点！ 