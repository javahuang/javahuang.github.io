# redis

## 基础

## 分布式锁

### 基于单 redis 节点的分布式锁

单节点无法保证服务的高可用

```bash
# my_random_value 随机串，保证所有客户端获取锁的请求唯一
# NX resource_name 不存在才可以 SET 成功
# PX 30000 30S 过期时间
SET resource_name my_random_value NX PX 30000
```

使用 lua 脚本释放锁，保证 `get`、`判断`、`del` 三个操作的原子性，redis 内部能保证 lua 脚本操作的原子性

```lua
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

failover：Redis 节点挂载一个 slave 节点，master 节点挂了，slave 切换成 master 节点（**failover**）。
主从复制是异步的，可能会导致锁失效，具体过程如下。

1. client A 从 master 获取到锁
2. master 宕机，key 还没来得及复制到 slave
3. slave 升级成 master
4. client B 从新的 master 获取到同一个资源的锁

### 使用 Redlock 实现分布式锁

获取锁算法：

1. 获取当前时间。所以 redlock 对系统时钟强依赖
2. 向 N 个 redis 节点执行加锁操作，步骤同单节点分布式锁
3. 计算获取锁时间，当前时间-第一步时间。从 >= N/2+1 个节点获得锁算加锁成功
4. 重新计算锁有效期，所得有效时间-第2 部获取锁时间
5. 如果锁获取失败，向所有节点发起释放锁操作（操作同单节点 lua 脚本）

redlock 存在的问题

1. 强依赖系统时钟
2. 依然存在如果客户端长期阻塞导致锁过期，会导致访问共享资源不安全。可以使用 fencing tokcn 的方式解决，即每次获取锁之后，同时返回的还有一个递增的数字，然后访问共享资源带着这个数字，共享资源来检查这个数字是不是有效。（Redlock提供不了这样一种机制，但是如果共享资源都能提供这样一种机制，是不是都不需要虎刺所）

锁分成两种：

1. 为了效率（effciency）,允许锁偶尔失效，也不会产生其他不良后果。可以考虑 redis 单节点锁，redlock 实现过重
2. 为了正确性（correctness），任何时候不允许锁失效的情况。可以考虑类似 zookeeper 的方案，或者支持事务的数据库


## 语言实现

## 参考

[基于Redis的分布式锁到底安全吗（上）？](http://zhangtielei.com/posts/blog-redlock-reasoning.html)