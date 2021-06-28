---
title: Redis分布式锁
categories:
  - null
tags:
  - null
translate_title: redis-distributed-lock
date: 2021-04-26 21:10:30
---


分布式锁，就是在分布式项目中使用的锁，控制分布式系统同步访问共享资源。

### 分布式锁需要满足的特性：

1、互斥性。任何时刻，对于一条数据，保证只有一台应用可以获取分布式锁。

2、高可用性。分布式场景下小部分的服务器宕机不会影响正常运行。（提供分布式锁的服务以集群方式部署）

3、防止锁超时。客户端没有主动释放锁，服务器会在一段时间后自动释放，防止客户端宕机或者网络不可达等因素产生死锁。

4、独占性。加锁解锁必须是同一台服务器进行。（锁的持有者才能释放锁）

### 怎么获取锁？

```shell
1、SETNX，用法
SETNX key value
#SETNX是『 SET if Not eXists』(如果不存在，则 SET)的简写，设置成功就返回1，否则返回0。
```

可以看出，当把**key**为**lock**的值设置为"Java"后，再设置成别的值就会失败，看上去很简单，也好像独占了锁，但有个致命的问题，就是**key没有过期时间**，这样一来，除非手动删除key或者获取锁后设置过期时间，不然其他线程永远拿不到锁。

```shell
SETNX Key 1
EXPIRE Key Seconds
```

但是这个方案把获取锁和设置过期时间分成两步，没有原子性。（有可能获取锁成功，但是设置过期时间失败）

```shell
2、SETEX，用法
SETEX key seconds value

#值 value 关联到 key ，并将 key 的生存时间设为 seconds (以秒为单位)。如果 key 已经存在，SETEX 命令将覆写旧值。
```

```shell
3、PSETEX，用法
PSETEX key milliseconds value

#这个命令和SETEX命令相似，但它以毫秒为单位设置 key 的生存时间，而不是像SETEX命令那样，以秒为单位。
```

不过，从`Redis 2.6.12` 版本开始，SET命令可以通过参数来实现和SETNX、SETEX、PSETEX 三个命令相同的效果。

```shell
SET key value NX EX seconds 
```

### 怎么释放锁？

释放锁的命令就简单了，直接删除key就行，但我们前面说了，因为分布式锁必须由锁的持有者自己释放，所以我们必须先确**保当前释放锁的线程是持有者**，没问题了再**删除**，这样一来，就变成两个步骤了，似乎又违背了原子性了，怎么办呢？

使用`lua`脚本把两步操作拼装。

```lua
if redis.call("get",KEYS[1]) == ARGV[1]
then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

KEYS[1]是当前key的名称，ARGV[1]可以是当前线程的ID(或者其他不固定的值，能识别所属线程即可)，这样就可以防止持有过期锁的线程，或者其他线程误删现有锁的情况出现。

### 实现一个分布式锁

```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.params.SetParams;

import java.util.Collections;

public class RedisDistributedLock {
    /**
     * 获取响应锁的数据key
     */
    private String LOCK_KEY = "redis_lock";
    /**
     * 分布式锁自动过期时间5ms
     */
    private long expireTime = 5;
    /**
     * 获取锁的最大连接时长1s
     */
    private long timeOut = 1000;
    /**
     * redis命令参数，相当于nx和px的命令合集
     */
    private SetParams params = SetParams.setParams().nx().px(expireTime);
    /**
     * redis连接池
     */
    JedisPool jedisPool = new JedisPool("127.0.0.1",6379);

    /**
     * 获取锁
     * @param id 线程的id，或者其他可识别当前线程且不重复的字段
     * @return 是否获取到锁
     */
    public boolean lock(String id) {
        Long start = System.currentTimeMillis();
        Jedis jedis = jedisPool.getResource();
        try {
            for(;;){
                // lock 为设置key的结果
                String lock = jedis.set(LOCK_KEY, id,params);
                if("OK".equals(lock)) {
                    return true;
                }
                // 目前距离获取锁开始的时间间隔
                long interval = System.currentTimeMillis() - start;
                if(interval > timeOut) {
                    return false;
                }

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        } finally {
            jedis.close();
        }
    }
    /**
     * 释放锁
     * @param id 线程的id，或者其他可识别当前线程且不重复的字段
     * @return
     */
    public boolean unlock(String id) {
        Jedis jedis = jedisPool.getResource();
        // lua脚本字符串
        String script = "if redis.call('get',KEYS[1]) == ARGV[1] " +
                "then " +
                "return redis.call('del',KEYS[1]) " +
                "else " +
                "return 0 " +
                "end";
        try {
            String result = jedis.eval(script, Collections.singletonList(LOCK_KEY), Collections.singletonList(id)).toString();
            return "1".equals(result);
        } finally {
            jedis.close();
        }
    }
}
```

测试类

```java
import redisDemo.RedisDistributedLock;

public class RedisDistributedLockTest {
    private static RedisDistributedLock demo = new RedisDistributedLock();
    private static Integer NUM = 101;

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() -> {
                String id = String.valueOf(Thread.currentThread().getId());
                boolean isLock = demo.lock(id);
                try {
                    if (isLock) {
                        NUM--;
                        System.out.println(NUM);
                    }
                } finally {
                    demo.unlock(id);
                }

            }).start();
        }
    }
}
```

### 分布式锁的缺陷

一、客户端长时间阻塞导致锁失效问题

客户端1得到了锁，因为网络问题或者GC等原因导致长时间阻塞，然后业务程序还没执行完锁就过期了，这时候客户端2也能正常拿到锁，可能会导致线程安全的问题。


客户端长时间阻塞

二、`redis`服务器时钟漂移问题

如果`redis`服务器的机器时钟发生了向前跳跃，就会导致这个key过早超时失效，比如说客户端1拿到锁后，key的过期时间是12:02分，但`redis`服务器本身的时钟比客户端快了2分钟，导致key在12:00的时候就失效了，这时候，如果客户端1还没有释放锁的话，就可能导致多个客户端同时持有同一把锁的问题。

三、单点实例安全问题

如果`redis`是单master模式的，当这台机宕机的时候，那么所有的客户端都获取不到锁了，为了提高可用性，可能就会给这个master加一个slave，但是因为`redis`的主从同步是异步进行的，可能会出现客户端1设置完锁后，master挂掉，slave提升为master，因为异步复制的特性，客户端1设置的锁丢失了，这时候客户端2设置锁也能够成功，导致客户端1和客户端2同时拥有锁。

为了解决`Redis`单点问题，`redis`的作者提出了**`RedLock`**算法。

### `RedLock`算法

该算法的实现前提在于`Redis`必须是多节点部署的，可以有效防止单点故障，具体的实现思路是这样的：

1、获取当前时间戳（`ms`）；

2、先设定key的有效时长（TTL），超出这个时间就会自动释放，然后client（客户端）尝试使用相同的key和value对所有`redis`实例进行设置，每次链接`redis`实例时设置一个比TTL短很多的超时时间，这是为了不要过长时间等待已经关闭的`redis`服务。并且试着获取下一个`redis`实例。

比如：TTL（也就是过期时间）为5s，那获取锁的超时时间就可以设置成50ms，所以如果50ms内无法获取锁，就放弃获取这个锁，从而尝试获取下个锁；

3、client通过获取所有能获取的锁后的当前时间**减去**申请锁进行第一步的时间，还有`redis`服务器的时钟**漂移误差**，然后这个时间差要小于TTL时间并且成功设置锁的实例数>= **N/2 + 1**（N为`Redis`实例的数量），也就是获取超过半数以上的`redis`实例的锁，那么加锁成功

比如TTL是5s，连接`redis`获取所有锁用了2s，然后再减去时钟漂移（假设误差是1s左右），那么锁的真正有效时长就只有2s了；

4、如果客户端由于某些原因获取锁失败，便会开始解锁所有`redis`实例。

### `RedLock`算法的隐患

好了，算法也介绍完了，从设计上看，毫无疑问，`RedLock`算法的思想主要是为了有效防止`Redis`单点故障的问题，而且在设计TTL的时候也考虑到了服务器时钟漂移的误差，让分布式锁的安全性提高了不少。

但事实真的是这样吗？反正我个人的话感觉效果一般般，

首先第一点，我们可以看到，在`RedLock`算法中，锁的有效时间会减去连接`Redis`实例的时长，如果这个过程因为网络问题导致耗时太长的话，那么最终留给锁的有效时长就会大大减少，客户端访问共享资源的时间很短，很可能程序处理的过程中锁就到期了。而且，锁的有效时间还需要减去服务器的时钟漂移，但是应该减多少合适呢，要是这个值设置不好，很容易出现问题。

然后第二点，这样的算法虽然考虑到用多节点来防止`Redis`单点故障的问题，但但如果有节点发生崩溃重启的话，还是有可能出现多个客户端同时获取锁的情况。

假设一共有5个`Redis`节点：A、B、C、D、E，客户端1和2分别加锁

1. 客户端1成功锁住了A，B，C，获取锁成功（但D和E没有锁住）。
2. 节点C的master挂了，然后锁还没同步到slave，slave升级为master后丢失了客户端1加的锁。
3. 客户端2这个时候获取锁，锁住了C，D，E，获取锁成功。

这样，客户端1和客户端2就同时拿到了锁，程序安全的隐患依然存在。除此之外，如果这些节点里面某个节点发生了时间漂移的话，也有可能导致锁的安全问题。