## 用法

```
node [options] [v8 options] [script.js | -e "script"] [arguments]
```

请参阅[Command Line Options(命令行选项)](commandLineOptions.md)中有关Node.js环境下运行脚本的不同配置参数

## 实例

使用Node.js编写的[Web服务器](http.md)示例，该适应响应"Hello World"

```javascript
const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
	res.statusCode = 200;
	res.setHeader('Content-Type', 'text/plain');
	res.end('Hello World\n');
});

server.listen(port, hostname, () => {
	console.log(`Server running at http://${hostname}:${port}/`);
});
```

将代码保存在名为example.js的文件中，通过Node.js执行脚本运行服务器

```
$ node example.js
Server running at http://127.0.0.1:3000/
```

所有文档中的示例都可以在相同环境下运行