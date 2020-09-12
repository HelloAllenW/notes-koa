# Koa
>  Koa是由Express原版人马打造的，基于Node.js平台的下一代web开发框架，最大的特点就是可以避免异步嵌套。

## Koa框架的安装使用

- 安装：（在项目文件夹下执行命令）
	```npm install --save koa```
- 使用：
```
	// 引入koa
	const koa = require('koa')
	const app = new koa()
	
	// 配置中间件（可以先当做路由）
	app.use(async (cxt)=>{
		cxt.body='hello koa2'
	})
	
	// 监听端口
	app.listen(3000);
```



## Koa路由

- 安装：
  ```npm install --save koa-router```
- 使用

```
// 引入路由
const router=require('koa-router')()

// 实例化路由 
var router=new Router() 

// ctx上下文context，包含了request和response等信息
router.get('/', async(ctx)=>{
	ctx.body="hello koa";
})
router.get('/news', async(ctx)=>{

	// get传值，获取url后的参数
	console.log(ctx.query)
	ctx.body="新闻page"

});

 // 启动路由
app.use(router.routes());

// 选择性配置，会根据ctx.status设置response响应头
app.use(router.allowedMethods());
```



## koa中间件

> 通俗的讲，中间件就是匹配路由之前或者匹配路由完成做的一系列的操作，我们就可以把它叫做中间件。
>
> 在express中间件是一个函数，它可以访问请求对象，相应对象，和web应用中处理请求-响应循环流程中的中间件，一般被命名为next的变量。在koa中中间件和express有点类似。
>
> 中间件的功能包括：执行任何代码、修改请求和响应对象、终结请求-响应循环、调用堆栈中的下一个中间件。
>
> 如果我的get、post回调函数中，没有next参数，那么就匹配上第一个路由，就不会往下匹配了。如果想往下匹配的话，那么需要写next()

- 应用级中间件：访问任何路由之前都会先经过此中间件
```
	app.use(async(ctx, next)=>{
		ctx.body='这是一个中间件'
		await next(); // 当前路由匹配完成之后继续向下匹配，如果不写，就不会向下继续匹配
	})
```
- 路由级中间件
```
	router.get('/news', async(ctx, next)=> {
		console.log('news')
		await next() // 匹配到news路由之后继续向下匹配
	})
```
- 错误处理的中间件
```
	app.use(async(ctx,next)=>{
		//...
		next()
		
		if (ctx.status==404) { // 根据洋葱模型，匹配完之后会回头再执行此方法
			ctx.status=404;
			ctx.body='这是一个404页面'
		}
	})
```
### 洋葱模型

> 中间件执行顺序：洋葱模型。（从外向内，又从内向外）

```
app.use(async (ctx, next) => {
    console.log('1. 这是一个中间件01')
    await next()
    
    console.log('5. 匹配路由完成之后又会返回来执行中间件')
})
app.use(async (ctx, next) => {
    console.log('2. 这是一个中间件02')
    await next()
    
    console.log('4. 匹配路由完成之后又会返回来执行中间件')
})
router.get('/news', async(ctx)=>{
    console.log('3. 匹配到了news这个路由')
})

// 上述代码的打印结果是：
// 1. 这是一个中间件01
// 2. 这是一个中间件02
// 3. 匹配到了news这个路由
// 4. 匹配路由完成之后又会返回来执行中间件
// 5. 匹配路由完成之后又会返回来执行中间件
```



## EJS模板引擎

- 安装```koa-views```和```ejs```

  ```
  npm install --save koa-views 
  
  npm install ejs --save
  ```

- 引入```koa-views```配置中间件

```
const views=require('koa-views')

// 第一个参数表示视图的地址，这样配置完后模板后缀名为html
app.use(views('views', { map:{html: 'ejs'} })) 

// 这样配置完后模板后缀名为ejs
// 或者 app.use(views('views', { map:{extension: 'ejs'} })) 
```



- 在对应路由中进行渲染

```
await ctx.render('index', {
  	// 后台要传给前端的数据写在这儿
})

// 公共的数据放在这个里面，这样的话在模板任何地方都可以使用
ctx.state={ // 放在中间件中 
	session: this.session,
	title: 'app'
}
```



## 使用koa-bodyparser获取post提交数据

```
npm install --save koa-bodyparser

引入 var bodyParser = require('koa-bodyparser')

app.use(bodyParser())

ctx.request.body
```



## koa-static 静态资源中间件
>  静态web服务，使用后可以访问static下的文件：
> http://localhost:3000/css/basic.css 首先去static目录找，如果能找到返回对应的文件，找不到next()

```
npm install koa-static --save

const static = require('koa-static')

// 配置中间件
app.use(static(__dirname+'static')) // 传入静态目录
```



## art-template 模板引擎
>  性能比EJS高，支持EJS语法，也可以用自己的类似angular数据绑定的语法

```
// 安装
npm install --save art-template
npm install --save koa-art-template

const render = require('koa-art-template')

// 配置
render(app, {
	root: path.josin(__dirname, 'view'), // 视图位置
	extname: '.art' // 后缀名
	debud: process.env.NODE_ENV !== 'production' // 是否开启调试模式
})

await ctx.render('index')
```



## Cookie的使用
```
// 设置
ctx.cookie.set(name,value,[options])

// 获取
ctx.cookie.get(name)
```



## session的使用

> 在koa中使用session时，一般采用 koa-session 模块

1. 安装与引入 ```koa-session```

```
// 安装
npm install koa-session --save

// 引入
const session = require('koa-session')

```

2. 配置中间件

```
// npm 官方文档中搜索可以找到配置方法
app.keys = ['some secret hurr'] // cookie签名
const CONFIG = {
    key: 'koa:sess', // cookie key
    maxAge: 8640000, // cookie的过期时间，默认一天
    overwrite: true, // 是否可以overwrite
    httpOnly: true, // true表示只有服务端可以获取cookie
    signed: true, // 默认签名
    rolling: false, // 在每次请求时强行设置cookie，这将重置cookie过期时间，默认false
    renew: false // 快过期时，是否重新设置
}
app.use(session(CONFIG, app))
```



2. 使用

```
// 设置值
ctx.session.username = '张三'

// 获取值
ctx.session.username
```



## MongoDB Compass Community可视化工具

MongoDB Compass 是 MongoDB 官网提供的一个集创建数据库、管理集合和文档、运行临时查询、评估和优化查询、性能图标、构建地理查询等功能为一体的MongoDB可视化管理工具。



## 封装Koa操作MongoDB数据库

目标： 基于官方的 node-mongodb-native 驱动，封装一个更小、更快、更灵活的DB模块，让我们用 nodejs 操作 MongoDB 数据库更方便，更灵活。

### 官方推荐nodejs操作MongoDB数据库方法

1、安装mongodb

```
cnpm install mongodb --save
```

2、引入 mongodb 下面的MongoClient

```
var MongoClient = require('mongodb').MongoClient
```

3、定义数据库连接的地址 以及配置数据库

```
var url = 'mongodb://localhost:27017'
var dbName = 'koa' // koa是要连接的数据库名称
```

4、nodejs 连接数据库

```
MongoClient.connect(url, function(err, client) {
    const db = client.db(dbName) // 数据库db对象
})
```

5、操作数据库

```
MongoClient.connect(url, function(err,db) {
    db.collection('user').insertOne({ 'name': '张三' }， function(err, result) {
        db.close()
    })
})
```



> 感谢： B站 《Nodejs教程_Nodejs+Koa2入门实战视频教程》
>
> 源码和笔记可前往：百度云 > 01 2019年Node.js入门视频教程（IT营大地）- 联系博主获取