---
title: node开发日志功能
date: 2019-07-11 11:31:46
tags:
---
### 日志需要的文件
``` javascript
log  // 存放日志的文件夹
>> access.log  // 日志记录文件
utils // 工具文件夹
>> log.js   // 日志主要功能代码
app.js
package.json
```
### 不使用第三方日志模块方式
1. 在utils/log.js文件中定义功能
``` javascript
const fs = require('fs')
const path = require('path')

 // 获取文件Strema流
function createWriteStrema(filename) {
    // 获取文件路径
    const fullFileName = path.join(__dirname, '../log', filename)
    // 根据路径获取文件Strema流
    const writeStrema = fs.createWriteStrema(fullFileName, {
        flags: 'a'  // 使用append方式向文件添加内容
    })
    retrun writeStrema
}

const accessWriteStrema = createWriteStrema('access.log')

// 写入日志
function writeLog(writeStrema, log) {
    writeStrema.write(log + '\n')  // 写入到文件中
}

function access (log) {
    writeLog(accessWriteStrema, log)
}

module.exports = {
    access
}

```

2. 在app.js中使用刚才写的日志功能
``` javascript
const { access } = require('./utils/log')

// 在文件中写上这个中间件
app.use(async (ctx, next) => {
    // 定义Log格式
    const log = `${ctx.method} -- ${ctx.url} -- ${ctx.headers['user-agent']} -- ${Date.now()}`
    // 调用写入文件方法
    access(log)
    // 跳到下一个中间件
    next()
})
```

### 使用morgan模块方式
1. 下载morgan模块
    `npm i koa-morgan`

2. app.js
``` javascript
 // 1. 引入模块
const morgan = require('koa-morgan')
const fs = require('fs')
const path = require('path')

// 2. 获取记录日志的文件
const writeStream =fs.createWriteStream(path.resolve(__dirname, './log', 'access.log'), {
    flags: 'a'
})
// 3. 注册中间件
// combined 代表使用较全的格式写到文件中，这个参数可以参照官方文档选择自己需要的格式  https://github.com/expressjs/morgan
// :remote-addr - :remote-user [:date[clf]] ":method :url HTTP/:http-version" :status :res[content-length] ":referrer" ":user-agent"
app.use(mogran('combined', {
    stream: writeStream // 写入到文件中
}))
```
这样就可以了，在每次访问的时候就会在日志中写入一条记录，如果在开发时不需要写入到文件中，可以使用环境变量来区分
```javascript
const ENV = process.env.NODE_ENV
// 如果不是正式环境就只在控制台输出 dev 格式的日志， 否则就写入 combined 格式的日志到文件中
if (ENV !== 'production' ) {
    app.use(morgan('dev'))
} else {
    app.use(mogran('combined', {
        stream: writeStream // 写入到文件中
    }))
}
```