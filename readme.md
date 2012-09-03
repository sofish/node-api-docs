## 前言

本篇有些部分直译自 node 的 api 文档，版本为 0.8.8。与其说是翻译，不如说是个人的一篇笔记。为保证全面，没有做接口的增删，并从中加入自己的注解。希望以学习+传达的方式达到更好的记忆，并把知识传播出去。对这些文字，你可以用于任何非商业的传播而无须征求我的意见；拒绝一切商业传播，包括我自己，希望这篇文档永远都是免费的。

## 一、全局对象

### 1. global {Object} 全局命名空间

对于浏览器来说，最上层的作用域是全局作用域，这意味着在浏览器的全局作用域中 `var something` 将会定义一个全局变量，但 Node 并不是这样，其最高作用域并非全局，在一个模块中 `var something` 将只是定义了模块的一个本地变量。

### 2. console {Object}

用以打印于 stdout 和 stderr 的对象。最有用的用是 `console.log()` 来打印内容。支持运行表达式；支持多个参数，用逗号分隔。其实次是 `console.dir()` 用以打印对象下属性/方法。

### 3. require {Function} 引入模块

在 node 中，模块使用 `require('module_name')` 来引用。

### 4. require.resolve() {Function} 查看模块路径

使用 `require('module_name')` 的方式会加载一个模块，而 `require.resolve('module_name')` 是查找 `module_name` 的路径，这个路径可能是：

```js
require.resolve('./src/class')
// 返回：'/Library/WebServer/Documents/github/sofish/src/class.js'
```

### 5. require.cache {Object} 模块缓存

每个模块被 `require` 一次之后，都会缓存起来，信息存于 `require.cahce` 这个对象中。删除 key 的值可重新加载一个模块，也即进行多一次 `require`，而非从缓存中拿。

### 6. require.extensions {Array}

告诉 `require` 如何处理一个特定文件扩展名。譬如把 `.sjs` 扩展名当作 `.js` 文件来渲染可以这样做：

```js
require.extensions['.sjs'] = require.extensions['.js'];
```

写一个扩展名处理器：

```js
require.extensions['.sjs'] = function(module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  // ...
  // 处理文件内容，并通过 module.exports 层出
  module.exports = content;
}
```

### 6. __filename {String} 文件名

被执行文件的文件名。在命令行中执行则会返回错误。严格来说它并不是一个全局的对象，而是每个模块都拥有的本地对象。

__值得注意的是__：访问一个模块的时候，如果模块是文件夹，则返回模块下 `index.js`，而如果模块是单独文件，则返回其本身。返回的值都是一个绝对路径。

```bash
# 模块目录
> node ../fish/
/Library/WebServer/Documents/github/fish/index.js
```


