---
title: 学习 Ramda.js (二)
tags: [Ramda.js]
date: 2018-08-14 21:49:45
---

在工作中遇到一个任务需要在所有矢量图层中将中国34个省级行政区域矢量图层找出来，这个小任务用 Ramda.js 来做当然最合适不过了。

测试数据

```js
const testData = {
  '22:10:5': {
    _layers: {
      2100: {
        layerName: 'WorldElements_R@China',
        type: 'REGION', // TEXT、 LINE
        properties: {
          attributes: {},
          id: 1,
          searchValues: ''
        }
      },
      2101: {} // 同上结构
    },
    options: {},
    _size: {} // 省略其他key
  },
  '22:10:4': {
    _layers: {
      2200: {
        layerName: 'Province_R@China#1',
        type: 'REGION',
        properties: {
          attributes: {},
          id: 33,
          searchValues: '新疆维吾尔自治区'
        }
      },
      2201: {} // 同上结构
    },
    options: {},
    _size: {} // 省略其他key
  }
}
```

期望结果

```
[{ id: 33, serachValues: '新疆维吾尔自治区' }] // 34个省级行政区域
```

代码实现

```js
const test = R.compose(
  R.map(
    R.compose(
      R.pick(['id', 'searchValues']),
      R.prop('properties')
    )
  ),
  R.uniqBy(R.path(['properties', 'id'])),
  R.chain(
    R.compose(
      R.filter(
        R.where({
          layerName: R.equals('Province_R@China#1'),
          type: R.equals('REGION')
        })
      ),
      R.values
    )
  ),
  R.pluck(['_layers']),
  R.values
)
```

好像完全能看懂的样子，一眼就知道它在干嘛（也许吧

# 拆开看一下

> R.values、R.pluck 了解一下

现在我们得到了如下数据

```json
[
  {
    2100: {
      layerName: "WorldElements_R@China",
      type: "REGION" // TEXT、 LINE
      properties: {
        attributes: {},
        id: 1,
        searchValues: ""
      }
    },
    2101: {} // 同上结构
  },
  {
    2200: {
      layerName: "Province_R@China#1",
      type: "REGION",
      properties: {
        attributes: {},
        id: 33,
        searchValues: "新疆维吾尔自治区"
      }
    },
    2201: {} // 同上结构
  }
]
```

看到这肯定想到的是再次 values 不就可以拿到想要的对象了吗，再试一下

```js
R.map(R.values)
```

现在我们的数据变成这样了

```json
[
  [
     {
      layerName: "WorldElements_R@China",
      type: "REGION" // TEXT、 LINE
      properties: {
        attributes: {},
        id: 1,
        searchValues: ""
      }
    },
    {}
  ],
  [
    {
      layerName: "Province_R@China#1",
      type: "REGION",
      properties: {
        attributes: {},
        id: 33,
        searchValues: "新疆维吾尔自治区"
      }
    },
    {}
  ]
]
```

首先分析数据发现只有 layerName 等于 Province_R@China#1 并且 type 等于 REGION 时才会是我们想要的行政区域数据，其实我也不确定因为这个条件是我猜的但我们可以试一下。


```js
R.map(
  R.filter(
    R.where({
      layerName: R.equals('Province_R@China#1'),
      type: R.equals('REGION')
    })
  )
)
```

不错不错，已经成功按照我们的条件过滤了

```json
[
  [],
  [
    {
      "layerName": "Province_R@China#1",
      "type": "REGION",
      "properties": {
        "attributes": {},
        "id": 33,
        "searchValues": "新疆维吾尔自治区"
      }
    }
  ]
]
```

数据变成了嵌套的数组这样似乎不太符合我们想要的，安排一下，使用 flatten 将嵌套数组以深度优先合并成一个新数组

```json
[
  {
    "layerName": "Province_R@China#1",
    "type": "REGION",
    "properties": {
      "attributes": {},
      "id": 33,
      "searchValues": "新疆维吾尔自治区"
    }
  }
]
```

去重

```js
R.uniqBy(R.path(['properties', 'id'])),
```

获取我们所需字段

```js
R.map(
  R.compose(
    R.pick(['id', 'searchValues']),
    R.pick(['properties'])
  )
)
```

观察一下上面几步是否可以被合并到一个 compose 方法里面

```js
// 这里的 R.map 可以被替换成 R.chain
R.map(
  R.compose(
    R.filter(
      R.where({
        layerName: R.equals('Province_R@China#1'),
        type: R.equals('REGION')
      })
    ),
    R.values
  )
)
```

我们将 map 方法替换成 chain 免去我们手动 flatten，考虑到可能会有重复数据我们还需要区调用去重方法

完整代码实现见 [demo 官方 repl 地址][demo-url]

[demo-url]: http://ramda.cn/repl/?v=0.25.0#?%0Aconst%20testData%20%3D%20%7B%0A%20%20%2222%3A10%3A5%22%3A%20%7B%0A%20%20%20%20_layers%3A%20%7B%0A%20%20%20%20%20%202100%3A%20%7B%0A%20%20%20%20%20%20%20%20layerName%3A%20%22WorldElements_R%40China%22%2C%0A%20%20%20%20%20%20%20%20type%3A%20%22REGION%22%2C%20%2F%2F%20TEXT%E3%80%81%20LINE%0A%20%20%20%20%20%20%20%20properties%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20attributes%3A%20%7B%7D%2C%0A%20%20%20%20%20%20%20%20%20%20id%3A%201%2C%0A%20%20%20%20%20%20%20%20%20%20searchValues%3A%20%22%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%202101%3A%20%7B%0A%20%20%20%20%20%20%20%20layerName%3A%20%22Province_R%40China%231%22%2C%0A%20%20%20%20%20%20%20%20type%3A%20%22REGION%22%2C%0A%20%20%20%20%20%20%20%20properties%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20attributes%3A%20%7B%7D%2C%0A%20%20%20%20%20%20%20%20%20%20id%3A%202%2C%0A%20%20%20%20%20%20%20%20%20%20searchValues%3A%20%22%E8%B4%B5%E5%B7%9E%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%202102%3A%20%7B%7D%20%2F%2F%20%E5%90%8C%E4%B8%8A%E7%BB%93%E6%9E%84%0A%20%20%20%20%7D%2C%0A%20%20%20%20options%3A%20%7B%7D%2C%0A%20%20%20%20_size%3A%20%7B%7D%2C%20%2F%2F%20%E7%9C%81%E7%95%A5%E5%85%B6%E4%BB%96key%0A%20%20%7D%2C%0A%20%20%2222%3A10%3A4%22%3A%20%7B%0A%20%20%20%20_layers%3A%20%7B%0A%20%20%20%20%20%202200%3A%20%7B%0A%20%20%20%20%20%20%20%20layerName%3A%20%22Province_R%40China%231%22%2C%0A%20%20%20%20%20%20%20%20type%3A%20%22REGION%22%2C%0A%20%20%20%20%20%20%20%20properties%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20attributes%3A%20%7B%7D%2C%0A%20%20%20%20%20%20%20%20%20%20id%3A%2033%2C%0A%20%20%20%20%20%20%20%20%20%20searchValues%3A%20%22%E6%96%B0%E7%96%86%E7%BB%B4%E5%90%BE%E5%B0%94%E8%87%AA%E6%B2%BB%E5%8C%BA%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%202201%3A%20%7B%0A%20%20%20%20%20%20%20%20layerName%3A%20%22Province_R%40China%231%22%2C%0A%20%20%20%20%20%20%20%20type%3A%20%22REGION%22%2C%0A%20%20%20%20%20%20%20%20properties%3A%20%7B%0A%20%20%20%20%20%20%20%20%20%20attributes%3A%20%7B%7D%2C%0A%20%20%20%20%20%20%20%20%20%20id%3A%2030%2C%0A%20%20%20%20%20%20%20%20%20%20searchValues%3A%20%22%E5%8C%97%E4%BA%AC%22%0A%20%20%20%20%20%20%20%20%7D%0A%20%20%20%20%20%20%7D%2C%0A%20%20%20%20%20%202202%3A%20%7B%7D%20%2F%2F%20%E5%90%8C%E4%B8%8A%E7%BB%93%E6%9E%84%0A%20%20%20%20%7D%2C%0A%20%20%20%20options%3A%20%7B%7D%2C%0A%20%20_size%3A%20%7B%7D%2C%20%2F%2F%20%E7%9C%81%E7%95%A5%E5%85%B6%E4%BB%96key%0A%20%20%7D%2C%0A%7D%3B%0A%0Aconst%20test1%20%3D%20R.compose%28%0A%20%20R.map%28%0A%20%20%20%20R.compose%28%0A%20%20%20%20%20%20R.pick%28%5B%27id%27%2C%20%27searchValues%27%5D%29%2C%0A%20%20%20%20%20%20R.prop%28%27properties%27%29%0A%20%20%20%20%29%0A%20%20%29%2C%0A%20%20R.uniqBy%28R.path%28%5B%27properties%27%2C%20%27id%27%5D%29%29%2C%0A%20%20R.chain%28%0A%20%20%20%20R.compose%28%0A%20%20%20%20%20%20R.filter%28%0A%20%20%20%20%20%20%20%20R.where%28%7B%0A%20%20%20%20%20%20%20%20%20%20layerName%3A%20R.equals%28%27Province_R%40China%231%27%29%2C%0A%20%20%20%20%20%20%20%20%20%20type%3A%20R.equals%28%27REGION%27%29%0A%20%20%20%20%20%20%20%20%7D%29%0A%20%20%20%20%20%20%29%2C%0A%20%20%20%20%20%20R.values%0A%20%20%20%20%29%0A%20%20%29%2C%0A%20%20R.pluck%28%5B%27_layers%27%5D%29%2C%0A%20%20R.values%0A%29%0A%0Atest1%28testData%29
