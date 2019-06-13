---
title: Webpack and SuperMap
tags: [webpack]
date: 2018-05-04 22:48:28
---

项目中需要使用到 SuperMap 这个地图库,被狠狠的坑了一把。嗯。让我来给大家填填坑。

因为我自己的项目 Webpack 版本为 4.0+，所以这里默认大家都是 4.0+

## 项目目录

```bash
.
├── .babelrc
├── Dockerfile
├── LICENSE
├── README.md
├── jsconfig.json
├── src
│   ├── store
│   ├── util
│   ├── components
│   ├── constants
│   ├── services
│   ├── routes
│   ├── index.html
│   ├── containers
│   ├── index.jsx
│   └── libs
│       ├── Lang
│       ├── SuperMap-8.1.1-14426.js
│       ├── SuperMap_Basic-8.1.1-14426.js
│       ├── SuperMap_Cloud-8.1.1-14426.js
│       ├── SuperMap_IServer-8.1.1-14426.js
│       ├── SuperMap_OGC-8.1.1-14426.js
│       ├── SuperMap_Plot-8.1.1-14426.js
│       ├── SuperMap_Visualization-8.1.1-14426.js
│       └── theme
├── yarn.lock
├── webpack.prod.config.js
├── package.json
├── webpack.dev.config.js
└── webpack.base.config.js
```

### 1. 首先将 SuperMap 若干 js 文件添到 entry 配置中

顺序很重要，切记不要弄错。调试很久都报错，特别感谢论坛上一位朋友告诉了我这个解决方案。lib 文件可以在他们官网进行下载。

```js
entry: {
  vendor: [
    './src/libs/SuperMap-8.1.1-14426',
    './src/libs/SuperMap_Plot-8.1.1-14426',
    './src/libs/SuperMap_Basic-8.1.1-14426',
    './src/libs/SuperMap_IServer-8.1.1-14426',
    './src/libs/SuperMap_Visualization-8.1.1-14426',
    './src/libs/SuperMap_OGC-8.1.1-14426',
    './src/libs/SuperMap_Cloud-8.1.1-14426',
    // ...other
  ],
  app: [
    // ...you app entry
  ]
}
```

### 2. 修改我们的 js|jsx 配置

```js
{
  test: /\.(js|jsx)$/,
  exclude: /node_modules/,
  include: [
    path.resolve(__dirname, 'node_modules/@supermap/iclient-common'),
    path.resolve(__dirname, 'node_modules/@supermap/iclient-classic'),
    path.resolve(__dirname, 'node_modules/elasticsearch'),
    sourcePath // 我们的项目文件地址, 我这里是path.resolve('./src')
  ],
  use: ['babel-loader']
}
```

### 3. 默认你已经有 .png|.jpg|.gif 的 loader 配置了

🎉

### 4. 经过一番操作，按理来说应该是可以完美了的，但现实

在这个位置卡住了好一会，原因是有一个报错

```bash
Delete of an unqualified identifier in strict mode.
```

使用 `yarn add babel-plugin-transform-remove-strict-mode -D` 安装这个插件，并将其加入我们的 .babelrc 文件。看到这个插件名的时候，顾名思义， 我相信你已经懂了为什么需要这样做。

```js
{
  "presets": [
    "react",
    ["env"],
    "stage-0"
  ],
  "plugins": [
    "transform-remove-strict-mode" // 新增 <-
  ]
}
```

### end. 让我们验证一下

```js
import { SuperMap } from '@supermap/iclient-classic'

console.log(SuperMap) // OK
```

还有就是不要忘了将 lib 内的 Theme 还有 Lang 放到可以被访问的路径上，否则就 404 了

问题解决，接下来的时间就让我们开始 Happy Coding 吧
