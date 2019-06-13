---
title: learning-clojure
tags: [clojure, ring]
date: 2016-12-6 16:01
---

clojure ring 学习

ring 中间件

```clojure
(def app
  (-> routes
       middlewares1
       middlewares2))
```

常见编写 middleware 的方式

```clojure
(defn middleware1 [handler]
  (fn (request)
    (let [response (handler request)
      response)))
```

`->` 是一个语法糖 让我们免去这种嵌套过深的写法

我们的 middleware 接受一个 handler 参数，并返回接受一个参数为 request 的函数

首先我们先通过 macroexpand 函数将表达式展开吧（是这个展开吧）

```clojure
(macroexpand '(-> routes
                  middlewares1
                  middlewares2))
```

在 repl 中执行这段代码可以看到 `->` 语法糖将我们的这段代码变成了

`(middlewares2 (middlewares1 routes))`

嗯 这就是->宏干的那些悄悄事

那么 ring 的中间机制是怎么让这些一个一个串起来执行的呢？

handler 实际上就是下一个中间件函数对于 middlewares1 来说这里的 handler 就是 routes，对于 middleware 来说 handler 就是 `(middlewares1 routes)` 的返回值

`->` 执行完这一段代码得到最后的 `(fn [request] ())` 函数并返回，这个函数最后返回一个 response

得益于我们的 `->` 宏，每个中间件函数的 handler 实则为我们的下一个中间件函数，我们在当前中间件函数里面执行 handler 就行了

middleware2 接受的 handler 的为 `(middlewares1 routes)`
`(middleware2 (middlewares1 routes))` 执行后得到匿名函数 `(fn [request] (handler request))` 执行下一个 handler 并将 request 传入

middleware1 接受的 handler 为 routes
(middlware1 routes) 执行后得到匿名函数 (fn [request][handler request]) 继续执行下一个 handler

最后一说一下 request, 定义好我们的 app，应用并不能启动，需要通过 http server 来启动, 这里使用的是 http-kit

```
(httpkit-run/server app {:port 1024})
```

知道 request 怎么来的了吗 app 等于最后一个中间件返回的（fn [request] (handler request)

httpkit-run/server 大致执行过程

```
RingHandler -> HttpServer -> this.handler.handler(request, new RespCallback(key, this))
```

拿到 callback 以后 this.cb.run 然后 tryWrite
大致就是这样了,因为 java 我也不懂

一个修改请求头的例子

```clojure
(def app
  (-> routes
        log-request
        update-request))

(defn log-request [handler]
  (fn [request]
    (println (:test request)  ; Hello
    (let [res (handler request)]
      res)))

(defn update-request [handler]
  (fn [request]
    (let [req (assoc request :test "Hello")
          res (handler req)]
      res)))
```

ok，我们尝试先用 `macroexpand` 将所有的中间件理一理
`(update-request (log-request routes))`
首先执行 update-request

```clojure
(fn [request]
    (let [req (assoc request :test "Hello")
          res (handler req)]
      res))
```

我们添加 value 为 "Hello" 的 key 为 `:test` 到 request 中,并用 let 绑定到 req, 并将下一个 handler 执行传入修改过后的 request，返回 response
log-request 函数接受到修改以后的 request 打印 "Hello"，接着执行一下个 handler routes
