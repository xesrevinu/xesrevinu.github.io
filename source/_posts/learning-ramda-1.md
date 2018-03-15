---
title: 学习 Ramda.js (一)
tags: [Ramda.js]
date: 2018-03-15 23:32:57
---

 用一段简单的测试数据， 和我们的一些模拟的需求，来学习 Ramda.js 基础的 Api 使用方法。

```js
const group_data = {
  name: '学习交流群',
  create_time: new Date().getTime(),
  users: [
    {
      id: 1,
      name: 'Kee',
      age: 18,
      like: ['吃饭', '睡觉', '玩'],
      hot: 10, // 人气
      address: '贵阳',
      last_say_time: new Date().getTime() - 1000, // 上一秒
      last_login_time: new Date().getTime() - 5000 //最后登录时间
    },
    {
      id: 2,
      name: 'While',
      age: 20,
      like: ['睡觉', '玩'],
      hot: 8, // 人气
      address: '湖南',
      last_say_time: new Date().getTime() - 9000,
      last_login_time: new Date().getTime() - 3000
    },
    {
      id: 3,
      name: '老王',
      age: 30,
      like: [],
      hot: 99, // 人气
      address: '湖南',
      last_say_time: new Date().getTime() - 7000,
      last_login_time: new Date().getTime() - 2000
    }
  ]
}
```

获取圈子用户

```js
let users = R.prop('users', group_data)
```

获取圈子总人气，圈子人气等于圈子内所有用户人气的综合

```js
let hotTotal = R.compose(R.sum, R.map(R.prop('hot')), R.prop('users'))

let hotTotalResult = hotTotal(group_data)
```

获取圈子人气最高的用户

```js
let maxHotUser = R.compose(
  R.head,
  R.sort(R.descend(R.prop('hot'))),
  R.prop('users')
)

let maxHotUserResult = maxHotUser(group_data)
```

将圈子里用户按地区分组

```js
let addressGroup = R.compose(R.groupBy(x => x.address), R.prop('users'))

let addressGroupResult = addressGroup(group_data)
```

圈子最后在线用户

```js
let lastLoginUser = R.compose(
  R.last,
  R.sort((a, b) => a.last_login_time > b.last_login_time),
  R.prop('users')
)

let lastLoginUserResult = lastLoginUser(group_data)
```

圈子最后发言

```js
let lastSayUser = R.compose(
  R.last,
  R.sort((a, b) => a.last_say_time > b.last_say_time),
  R.prop('users')
)

let lastSayUserResult = lastSayUser(group_data)
```

获取圈子最大（最小）年龄

```js
// sort就是排序规则函数实现
let selectUserAge = R.curry((sort, group_data) =>
  R.compose(R.head, R.sort(sort(R.prop('age'))), R.prop('users'))(group_data)
)

// 最小年龄用户
let minAgeUserResult = selectUserAge(R.ascend, group_data)

// 最大年龄
let maxAgeUserResult = selectUserAge(R.descend, group_data)
```

 一些很简单的小例子， 看一下就知道是在做什么了，对我们新上手 Ramda.js 很有帮助，在以后的文章中，会一步一步学习到这个库有意思的一些地方。
