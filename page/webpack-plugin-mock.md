# webpack-plugin-mock
一款mockjs的webpack插件，配置简单、易用; Mock 编写灵活

## 安装

```bash
# npm
npm install webpack-plugin-mock -D

# yarn
yarn add webpack-plugin-mock -D
```

## 使用

需要在当前目录下有 mock 文件夹

```bash
├── mock
|  ├── books/
|  |  ├── books/
|  |  |   └── queryBooksByPage
|  |  |   |   └──  data.json
|  ├── example/
|  |  ├── json.js
|  |  └── x-www-from-urlencoded.js
|  └── user/
|     └── list.json
└── package.json
```

## webpack 配置
```bash
 new WebpackPluginMock({
    apiBasePath: './mock',
    watch: true,
    pretty: true,
    port: 8090
  })

  port  Mock 服务的启动端口
  config  Mock 服务的启动配置
  apiBasePath  Mock 服务 API 目录
  pretty  是否对 JSON Response 美化
  watch   是否监控 API 目录文件变化
```

## Mock 编写

### 用法一：定义 JSON（默认支持 mockjs 语法）

```json
{
  "code": 0,
  "msg": "",
  "data": {
    "dataList|10" :[
      {
        "id": "@id",
        "name": "@cname",
        "phone": "@pick([13913998972, 19941558406])",
        "deptName": "前端开发部"
      }
    ],
    "totalCount": 20,
    "totalPageCount": 2
  }
}
```

文件路径：`xxx/apis/demo/qux.json`，会自动创建路由为 `/demo/qux` 的接口，支持 GET/POST/JSONP 等方法。

兼容旧版 mock 服务，如果 JSON 文件路径为 `xxx/apis/demo/foo/data.json`，则创建的路由为 `/demo/foo`。

### 用法二：定义 CommonJS 模块（默认支持 mockjs 语法），要求模块默认导出的方法调用后返回对象

```js
module.exports = () => ({
  foo: 'bar',
  list: [1, 2, 3, 4, 5]
});
```

文件路径：`xxx/apis/demo/foo.js`，会自动创建路由为 `/demo/foo` 的接口，支持 GET/POST/JSONP 等方法。

### 用法三：定义 Koa 中间件，要求模块默认导出的方法调用后返回的对象中包含 `middleware` 函数

```js
module.exports = ({ mock }) => ({
  async middleware(ctx, next) {
    ctx.body = {
      foo: 'bar',
      list: [1, 2, 3, 4, 5]
    };
  }
});
```

文件路径：`xxx/apis/demo/foo.js`，会自动创建路由为 `/demo/foo` 的接口，支持 GET/POST 等方法。


### 用法四：定义 Koa 中间件、请求方法和路径（忽略文件路径），允许同时定义多个

```js
module.exports = ({ mock }) => ({
  method: 'get',
  path: '/demo/bar',
  async middleware(ctx, next) {
    ctx.body = {
      foo: 'bar',
      list: [1, 2, 3, 4, 5]
    };
  }
});
```

创建仅支持 GET 方法，路由为 `/demo/bar` 的接口。

```js
module.exports = ({ mock }) => ([
  {
    method: 'get',
    path: '/example/multiple/foo',
    async middleware(ctx, next) {
      ctx.body = {
        foo: 'foo',
        list: [1, 2, 3, 4, 5]
      };
    }
  },
  {
    method: 'get',
    path: '/example/multiple/bar',
    async middleware(ctx, next) {
      ctx.body = {
        foo: 'bar',
        list: [1, 2, 3, 4, 5]
      };
    }
  },
  {
    method: 'get',
    path: '/example/multiple/jsonp',
    async middleware(ctx, next) {
      ctx.jsonp = {
        foo: 'bar',
        list: [1, 2, 3, 4, 5]
      };
    }
  }
]);
```

在一个模块中同时定义多个接口路由（**每个子路由都必须定义 `path`**）。

### 用法五：完全自定义 Koa 路由（不能返回对象，否则和用法二冲突）

```js
module.exports = ({ router, mock }) => {
  router.get('/demo/baz', (ctx, next) => {
    ctx.body = 'Hello World!';
  });
};
```
