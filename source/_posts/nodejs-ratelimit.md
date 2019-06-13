---
title: node-ratelimit 学习
tags: []
date: 2017-9-07 10:20
---

Web 服务器通常使用内存数据库如 redis 进行会话管理，使用速率限制算法来检查用户会话是否超过限制

在 Node.js 中我们可以考虑使用[node-ratelimiter](https://github.com/tj/node-ratelimite)配合 redis 就能很简单的实现该功能

## Tips

如果客户端在给定的时间内发起太多的请求，服务器可以响应 status code 为 429（[Too many Requestes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes#429_Too_Many_Requests)）

- （可选）将限制信息写入 Header

```
#=============================#=============================================#
# HTTP Header                 # Description                                 #
#=============================#=============================================#
| X-RateLimit-Limit           | Request limit per day / per 5 minutes       |
+-----------------------------+---------------------------------------------+
| X-RateLimit-Remaining       | The number of requests left for the time    |
|                             | window                                      |
+-----------------------------+---------------------------------------------+
| X-RateLimit-Reset           | The remaining window before the rate limit  |
|                             | resets in UTC epoch seconds                 |
+-----------------------------+---------------------------------------------+
```

- (可选) 将多久之后可调用时间写入 Header

设置 HTTP Header 的 Retry-After

---

## 看看源码

使用 node-ratelimiter 我们最主要传入一个 db(redis 实例), 和一个唯一的 id 做区分用户使用，
其余的 max，duration, key 先使用默认

### Limiter.prototype.get

`var now = microtime.now()` 拿到当前时间戳

`var start = now - duration * 1000` 开始时间

`zremrangebyscore([key, 0, start])` reset 所有 删除掉 0-start 区间的条目，也就是 duration 设定时间的数据

`zcard([key])` 如果 key 存在返回长度, 不然返回 0

`zadd([key, now, now])` score 为 now 并且 value 为 now 存入名为 key 的有序集合中

`zrange([key, 0, 0])` 获取第一条数据

`pexpire([key, duration])` 设置 key 的 TTL(存活时间)，每次有正常访问都会重设 TTL

`exec` 获取当前集合中的条数，获取第一条数据，返回如下

```javascript
{
  remaining: count < max ? max - count : 0, // 剩余次数
  reset: Math.floor((oldest + duration * 1000) / 1000000), // reset时间点为第一条加上duration
  total: max
}
```
