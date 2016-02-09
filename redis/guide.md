

## Redis Pipelining
名词解释：RTT round trip time
采用pipeline类似与java nio 模式
请求1000次差异比较
```
without pipelining 1.185238 seconds
with pipelining 0.250783 seconds
```
*注：所有版本都支持pipeline


## Memory optimization

## EXPIRE key
设置key的有效期(秒)，到期后key的值自动为空，但是key不会丢失。可以修改key的值冰重新设置key的有效期。

```shell
redis> SET mykey "Hello"
OK
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey
(integer) 10
redis> SET mykey "Hello World"
OK
redis> TTL mykey
(integer) -1
redis> 
```

## LRU cache
可以设置redis数据内存大小，并设置到达内存大小的策略

## Transactions
MULTI, EXEC, DISCARD and WATCH are the foundation of transactions in Redis. 
redis 不支持事务回滚

MULTI begin transaction
EXEC run transaction
DISCARD abort transaction


## Mass insertion of data



