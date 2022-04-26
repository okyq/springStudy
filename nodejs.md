# 1. Node.js简介

1. Node.js是一个基于**ChromeV8引擎**的JavaScript运行环境（nodejs.org/zh-cn)
2. Node.js中是JavaScript后端运行环境



# 2. fs文件系统模块

- 引入fs 

```javascript
const fs = require('fs')
```

- fs 模块中所有的操作都有两种形式可供选择:同步和异步
- 同步文件系统会阻塞程序的执行，也就是除非操作完毕，否则不会向下执行代码
- 异步文件系统不会阻塞程序的执行，而是在操作完成时，通过回调函数将结果返回
- 实际开发很少用同步方式，因此只介绍异步方式

| 模式 | 说明                                     |
| ---- | ---------------------------------------- |
| r    | 读取文件，文件不存在抛异常               |
| r+   | 读写文件，文件不存在抛异常               |
| rs   | 同步模式下打开文件用于读取               |
| rs+  | 同步模式下打开文件用于读写               |
| w    | 写文件，不存在则创建，存在则覆盖原有内容 |
| wx   | 写文件，文件存在打开失败                 |
| w+   | 读写文件，不存在创建，存在截断           |
| wx+  | 读写，存在打开失败                       |
| a    | 追加，不存在创建                         |
| ax   | 追加，存在失败                           |
| a+   | 追加和读取，不存在创建                   |
| ax+  | 追加和读取，存在失败                     |

## 2.1 读取文件

语法格式：

```javascript
fs.readFile(path[, options], callback)
```

+ path : 文件路径
+ options： 配置选项，若是字符串则指定编码格式
  + encoding：编码格式
  + flag：打开方式
+ callback：回调函数
  + err：错误信息
  + data：读取到的数据，如果未指定编码格式则返回一个Buffer

```javascript
const fs = require('fs')

fs.readFile('./files/1.txt', 'utf-8', function(err, data) => {
  if(err) {
    return console.log('failed!' + err.message)
  }
  console.log('content:' + data)
})


// 复制文件内容
fs.readFile("C:/Users/笔记.mp3", function(err, data) {
	if(!err) {
		console.log(data);
		// 将data写入到文件中
		fs.writeFile("C:/Users/hello.jpg", data, function(err){
			if(!err){
				console.log("文件写入成功");
			}
		} );
	}
});
```

## 2.2 写入文件

```javascript
fs.writeFile(file, data[, options], callback)
```

- `file`：文件路径
- `data`：写入内容
- `options`：配置选项，包含 `encoding, mode, flag`；若是字符串则指定编码格式
- `callback`：回调函数

```javascript
const fs = require('fs')
fs.writeFile('./files/2.txt', 'Hello Nodejs', function (err) {
  if (err) {
    return console.log('failed!' + err.message)
  }
  console.log('success!')
})

//用绝对路径可以避免路径拼接问题

fs.writeFile('C:/Users/hello.txt', '通过 writeFile 写入的内容', { flag: 'w' }, function (err) {
  if (!err) {
    console.log('写入成功！')
  } else {
    console.log(err)
  }
})
```

**使用fs模块操作文件的时候，使用完整文件路径，不要使用相对路径**

## 2.3 路径动态拼接问题

- __dirname (双下划线)
- 在使用 fs 模块操作文件时，如果提供的操作路径是以 `./` 或 `../` 开头的相对路径时，容易出现路径动态拼接错误的问题
- 原因：代码在运行的时候，会以执行 node 命令时所处的目录，动态拼接出被操作文件的完整路径
- 解决方案：在使用 fs 模块操作文件时，直接提供完整的路径，从而防止路径动态拼接的问题
- `__dirname` 获取文件所处的绝对路径

## 2.4 其他操作

验证路径是否存在：

- `fs.exists(path, callback)`
- `fs.existsSync(path)`

获取文件信息：

- `fs.stat(path, callback)`
- `fs.stat(path)`

删除文件：

- `fs.unlink(path, callback)`
- `fs.unlinkSync(path)`

列出文件：

- `fs.readdir(path[,options], callback)`
- `fs.readdirSync(path[, options])`

截断文件：

- `fs.truncate(path, len, callback)`
- `fs.truncateSync(path, len)`

建立目录：

- `fs.mkdir(path[, mode], callback)`
- `fs.mkdirSync(path[, mode])`

删除目录：

- `fs.rmdir(path, callback)`
- `fs.rmdirSync(path)`

重命名文件和目录：

- `fs.rename(oldPath, newPath, callback)`
- `fs.renameSync(oldPath, newPath)`

监视文件更改：

- `fs.watchFile(filename[, options], listener)`

# 3. path路径模块

path 模块是 Node.js 官方提供的、用来处理路径的模块。它提供了一系列的方法和属性，用来满足用户对路径的处理需求。

**模块导入**

```javascript
const path = require('path')
```

## 3.1 路径拼接 join()

```javascript
const path = require('path')
const fs = require('fs')

//   注意 ../ 会抵消前面的路径
//   ./ 会被忽略
const pathStr = path.join('/a', '/b/c', '../../', './d', 'e')
console.log(pathStr) // \a\d\e

fs.readFile(path.join(__dirname, './files/1.txt'), 'utf8', function (err, dataStr) {
  if (err) {
    return console.log(err.message)
  }
  console.log(dataStr)
})
```

**涉及到路径拼接的问题的时候，不要使用+，使用path.join方法**

## 3.2 获取路径中的文件名 basename()

使用 `path.basename()` 方法，可以获取路径中的最后一部分，常通过该方法获取路径中的文件名

```javascript
path.basename(path[, ext])
```

- path: 文件路径
- ext: 文件扩展名

```javascript
const path = require('path')

// 定义文件的存放路径
const fpath = '/a/b/c/index.html'

const fullName = path.basename(fpath)
console.log(fullName) // index.html

const nameWithoutExt = path.basename(fpath, '.html')
console.log(nameWithoutExt) // index
```

## 3.3 获取文件路径中的扩展名 extname()

```javascript
const path = require('path')

const fpath = '/a/b/c/index.html'

const fext = path.extname(fpath)
console.log(fext) // .html
```

# 4. http模块学习

http 模块是 Node.js 官方提供的、用来创建 web 服务器的模块

```javascript
const http = require('http')

// 创建 web 服务器实例
const server = http.createServer()

// 为服务器实例绑定 request 事件，监听客户端的请求
// req是客户端相关的属性
// res是服务器端相关的属性
server.on('request', function (req, res) {
  const url = req.url
  const method = req.method
  const str = `Your request url is ${url}, and request method is ${method}`
  console.log(str)

  // 设置 Content-Type 响应头，解决中文乱码的问题
  res.setHeader('Content-Type', 'text/html; charset=utf-8')
  // 向客户端响应内容，并结束服务器
  res.end(str)
})

server.listen(8080, function () {
  console.log('server running at http://127.0.0.1:8080')
})
```

根据不同url相应不同的url

```javascript
const http = require('http')
const server = http.createServer()

server.on('request', (req, res) => {
  const url = req.url
  // 设置默认的响应内容为 404 Not found
  let content = '<h1>404 Not found!</h1>'
  // 判断用户请求的是否为 / 或 /index.html 首页
  // 判断用户请求的是否为 /about.html 关于页面
  if (url === '/' || url === '/index.html') {
    content = '<h1>首页</h1>'
  } else if (url === '/about.html') {
    content = '<h1>关于页面</h1>'
  }

  res.setHeader('Content-Type', 'text/html; charset=utf-8')
  res.end(content)
})

server.listen(80, () => {
  console.log('server running at http://127.0.0.1')
})
```

# 5. 模块化

- 模块化是指解决一个复杂问题时，自顶向下逐层把系统划分为若干模块的过程，模块是可组合、分解和更换的单元。
- 模块化可提高代码的复用性和可维护性，实现按需加载。
- 模块化规范是对代码进行模块化拆分和组合时需要遵守的规则，如使用何种语法格式引用模块和向外暴露成员。

## 5.1 Node.js中的模块的分类

- 内置模块（内置模块是官方提供的，例如fs，path等）
- 自定义模块(用户创建的js文件，都是自定义模块)
- 第三方模块（由第三方开发出来的模块，使用前需要下载）

## 5.2 加载模块

使用require() 方法加载模块

```javascript
const fs = require('fs')
```

使用require方法加载其他模块的时候，会执行被加载模块中的代码

## 5.3 模块作用域

- 和函数作用域类似，在自定义模块中定义的变量、方法等成员，只能在当前模块内被访问，这种模块级别的访问限制，叫做模块作用域
- 防止全局变量污染

## 5.4 model对象

- 自定义模块中都有一个 `module` 对象，存储了和当前模块有关的信息

- 在自定义模块中，可以使用 `module.exports` 对象，将模块内的成员共享出去，供外界使用。导入自定义模块时，得到的就是 `module.exports` 指向的对象。

- 默认情况下，`exports` 和 `module.exports` 指向同一个对象。最终共享的结果，以 **`module.exports`** 指向的对象为准。

  ![](https://cdn.jsdelivr.net/gh/yqimg/img/20220425154937.png)

+ 第三个，因为是一个对象，所以会有两个属性

## 5.5 CommonJS模块化规范

- 每个模块内部，`module` 变量代表当前模块
- `module` 变量是一个对象，`module.exports` 是对外的接口
- 加载某个模块即加载该模块的 `module.exports` 属性

# 6 npm与包

+ 包是基于内置模块封装出来的，提供了更高级，更方便的api，极大提高了开发效率

+ 从哪里下载？`npmjs.com`， 

## 6.1 怎么下载包

+ 下载`registry.npmjs.com`
+ 包管理工具：Node Package Manage
+ npm i moment 指定版本 npm i moment@2.24.0
+ 接收： const moment = require('moment')



## 6.2 快速创建package.json

npm init -y

+ 只能在英文的目录下运行
+ npm install命令安装的时候，npm会自动记录package.json

+ dependencies节点会记录装了哪些包



## 6.3 一次性安装所有包

+ npm install命令会读取package.json中的dependencies并安装



## 6.4 卸载包

+ npm uninstall moment 卸载之后会自动从packages.json中自动移除

## 6.5 devDependencies节点

如果某些包只在开发阶段会用到，在项目上线之后不会用到，则建议把这些包记录在devDependencies

+ `npm i 包名 -D` 安装指定的包，并记录到devDependencies

## 6.6 切换镜像源

+ 切换镜像源`npm config set registry=https://registry.npm.taobao.org`

+ 也可以使用 nrm 工具 快捷切换和查看镜像源

## 6.7 模块的加载机制

+ **模块优先从缓存中加载**，模块第一次加载后会被缓存，即多次调用 `require()` 不会导致模块的代码被执行多次，提高模块加载效率。

+ **内置模块加载优先级最高**

+ **自定义模块加载**

  + 加载自定义模块时，路径要以 `./` 或 `../` 开头，否则会作为内置模块或第三方模块加载。

    导入自定义模块时，若省略文件扩展名，则 Node.js 会按顺序尝试加载文件：

    + 按确切的文件名加载
    + 补全 `.js` 扩展名加载
    + 补全 `.json` 扩展名加载
    + 补全 `.node` 扩展名加载
    + 报错

+ **第三方模块加载**

  + 如果传递给require() 的模块标识符不是一个内置模块，也没有以 **`./`** or **`../`**开头，则Node.js会从当前模块的**父目录**开始，尝试从**`/node_modules`**文件夹中加载第三方模块
  + 如果没有找到对应的第三方模块，则移动到再**上一层父目录**中，进行加载，直到**文件系统的根目录**。

  例如，假设在 `C:\Users\bruce\project\foo.js` 文件里调用了 `require('tools')`，则 Node.js 会按以下顺序查找：

  - `C:\Users\bruce\project\node_modules\tools`
  - `C:\Users\bruce\node_modules\tools`
  - `C:\Users\node_modules\tools`
  - `C:\node_modules\tools`

+ **目录作为模块加载**

  + 在被加载的目录下查找 `package.json` 的文件，并寻找 `main` 属性，作为 `require()` 加载的入口
  + 如果没有 `package.json` 文件，或者 `main` 入口不存在或无法解析，则 Node.js 将会试图加载目录下的 `index.js` 文件。
  + 若失败则报错
