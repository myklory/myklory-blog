title: Redis基本用法
date: 2016-08-10 15:13:24
tags: [Redis]
---

Redis是一个开源，先进的key-value存储，并用于构建高性能，可扩展的Web应用程序的完美解决方案。
Redis从它的许多竞争继承来的三个主要特点：
-   Redis数据库完全在内存中，使用磁盘仅用于持久性。
-   相比许多键值数据存储，Redis拥有一套较为丰富的数据类型。
-   Redis可以将数据复制到任意数量的从服务器。

**Redis 优势**

-   异常快速：Redis的速度非常快，每秒能执行约11万集合，每秒约81000+条记录。
-   支持丰富的数据类型：Redis支持最大多数开发人员已经知道像列表，集合，有序集合，散列数据类型。这使得它非常容易解决各种各样的问题，因为我们知道哪些问题是可以处理通过它的数据类型更好。
-   操作都是原子性：所有Redis操作是原子的，这保证了如果两个客户端同时访问的Redis服务器将获得更新后的值。
-   多功能实用工具：Redis是一个多实用的工具，可以在多个用例如缓存，消息，队列使用(Redis原生支持发布/订阅)，任何短暂的数据，应用程序，如Web应用程序会话，网页命中计数等。
<!--more-->

### **Redis基本用法**
Redis是一个键--值(key-value)数据库， 我们可以用命令**SET**来存储一个键值对。
```redis
SET server:name "fido"
```
Redis会使用key "server:name" 存储value "fido"。
我们可以用命令**GET**来取得key "server:name" 的value。
```redis
GET server:name
// 返回"fido"
```
这就是Redis的最基本操作。
#### **DEL，SETNX， INCR**
使用**DEL**命令可以删除已有的键值对。
**SETNX**是(SET if no exists)，如果键不存在则创建键值对。
**INCR**是将键的值加1的操作，这个操作是原子的。如果键不存在的话，这个操作会把键的值设为1.
```redis
SET connections 10
INCR connections => 11
DEL connections
GET connections => nil
INCR connections => 1
```
#### **EXPIRE, TTL**
**EXPIRE**命令可以设置键值对存在的时间，在时间到期之后Redis会把键值对删除，这时使用**GET**查询会返回nil
**TTL**命令会返回键值对存在的时间，返回正整数是指将存在的时间，返回**-1**表示会一直存在，返回**-2**表示这个键已经被删除，不再存在。如果对这个键重新使用**SET**命令，TTL的时间会重置为**-1**。
```redis
SET resource:lock "Redis Demo"
TTL resource:lock => -1 //会一直存在
EXPIRE resource:lock 120 //120秒后不再存在
//经过几秒
TTL resource:lock => 99 //还会存在99秒
//经过99秒
TTL resource:lock => -2 //键已经不存在
GET resource:lock => nil
```
#### **RPUSH, LPUSH, LLEN, LRANGE, LPOP, RPOP**
Redis还可以存储有序的List。
**RPUSH** 将值放进list的尾部。
**LPUSH** 将值放进list的头部。
**LRAMGE** 取List中的值，第一个参数为开始的序号，第二个参数为结束的序号，如果第二个参数为**-1**表示一直取到List尾部。
**LLEN** 返回列表的长度，如果列表不存在返回0，如果key不是一个列表返回错误。
**LPOP/RPOP** 返回并且删除List中第一个（LPOP）或最后一个(RPOP)元素，如果key不存在或列表为空则返回nil。这是一个原子操作。
```redis
RPUSH friends "Alice" //使用RPUSH命令存储List ['Alice']
RPUSH friends "Bob" //['Alice', 'Bob']
LPUSH friends "SAM" //['SAM', 'Alice', 'Bob']
LRANGE friends 1 -1 //['Alice', 'Bob']
LLEN friends => 3
LPOP friends => 'SAM'
RPOP friends => 'Bob' //['Alice']
```
#### **SADD,SREM,SISMEMBER,SMEMBERS,SUNTION**
Redis也支持存储无序的set。
**SADD** 将值存进set。由于set是无序的，就不存在头部还是尾部。
**SREM** 将值从set中删除。返回**1**表示删除成功，0表示元素不在set中。
**SISMEMBER** 测试值是否在set中，**1**表示在set中，**0**表示不在set中。
**SMEMBERS** 返回set中的所有元素。
**SUNION** 将所有set中的所有元素。
```redis
SADD superpowers "flight"
SADD superpowers "x-ray vision"
SADD superpowers "reflexes" // ('reflexes', 'flight', 'x-ray vision')
SMEMEBER superpowers => ('reflexes', 'flight', 'x-ray vision')
SREM superpowers "reflexes" => 1
SISMEMBER superpowers "flight" => 1
SISMEMBER superpowers "reflexes" => 0
SADD birdpowers "pecking"
SADD birdpowers "flight"
SUNION superpowers birdpowers => ('pecking', 'x-ray vision', 'flight')
```
#### **ZADD**
使用**ZADD**可以添加有序的set

#### **HSET, HGET, HGETALL, HMSET, HDEL, HINCRBY**
使用这几个命令可以创建HashMap。
**HSET** 为key增加一个属性。
**HGET** 返回key的一个属性。
**HGETALL** 返回key的所有属性。
**HMSET** 设置key的多个属性。
**HDEL** 删除一个属性。
**HINCRBY** 一个属性的值自增1，参见**INCR**命令。
```redis
HSET user:100 name "John Smith"
HSET user:100 email "john@example.com"
HSET user:100 password "s3cret"
HGETALL user:100
HMSET user:1001 name "Mary Jones" password "hidden" email "mjones@example.com"
HGET user:1001 name => "Mary Jones"
```