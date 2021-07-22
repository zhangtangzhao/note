---
sort: 5
---

# Redis

## Redis 性能优化

config set slowlog-log-slower-than 10000  //10毫秒
设置慢查询的极值

redis-benchmark -h 192.168.42.111 -p 6379 -c 100 -n 10000 
     100个并发连接，10000个请求，检测服务器性能 

redis-benchmark -h 192.168.42.111 -p 6379 -q -d 100  
      测试存取大小为100字节的数据包的性能

redis-benchmark -h 192.168.42.111 -p 6379 -t set,get -n 100000 -q
      只测试 set,lpush操作的性能

redis-benchmark -h 192.168.42.111 -p 6379 -n 100000 -q script load "redis.call('set','foo','bar')"
     只测试某些数值存取的性能

