> 目前初翻阶段。会在全部翻译完做一遍校对。
> 如有看到错误的同学，请多多帮手提出来。或者等1周左右后再看，理论上应该会翻译完。如果我太懒没更新，可以在 [twitter](http://twitter.com/sofish) 、[新浪微博](http://weibo.com/isofish) 催更。

## 前言

本篇有些部分直译自 node 的 api 文档，版本为 0.8.8。与其说是翻译，不如说是个人的一篇笔记。为保证全面，没有做接口的增删，并从中加入自己的注解。希望以学习+传达的方式达到更好的记忆，并把知识传播出去。对这些文字，你可以用于任何非商业的传播而无须征求我的意见；拒绝一切商业传播，包括我自己，希望这篇文档永远都是免费的。

## 一、全局对象

### 1. global {Object} 全局命名空间

对于浏览器来说，最上层的作用域是全局作用域，这意味着在浏览器的全局作用域中 `var something` 将会定义一个全局变量，但 Node 并不是这样，其最高作用域并非全局，在一个模块中 `var something` 将只是定义了模块的一个本地变量。

### 2. console {Object}

用以打印于 stdout 和 stderr 的对象。最有用的用是 `console.log()` 来打印内容。支持运行表达式；支持多个参数，用逗号分隔。其次是 `console.dir()` 用以打印对象下属性/方法。

### 3. require {Function} 引入模块

在 node 中，模块使用 `require('module_name')` 来引用。

### 4. require.resolve() {Function} 查看模块路径

使用 `require('module_name')` 的方式会加载一个模块，而 `require.resolve('module_name')` 是查找 `module_name` 的路径，返回一个绝对路径：

```js
require.resolve('./src/class')
// /Library/WebServer/Documents/github/sofish/src/class.js
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
  // 处理文件内容，并通过 module.exports 导出
  module.exports = content;
}
```

### 7. __filename {String} 文件名

被执行文件的文件名。在命令行中执行则会返回错误。严格来说它并不是一个全局的对象，而是每个模块都拥有的本地对象。

__值得注意的是__：访问一个模块的时候，如果模块是文件夹，则返回模块下 `index.js`；而如果模块是单独文件，则返回其本身。返回的值是绝对路径。

### 8. __dirname {String} 路径名

与 `__dirname` 同理，返回的是一个绝对路径。

### 9. module {Object} 模块

当前模块的一个引用。比较特殊的是，`module.exports` 等价于 `exports` 对象。更确切来说 `module` 也是一个只存在于每个模块中的本地对象，而非严格意义上的全局对象。

### 10. exports {Object}

它可共用于当前模块的所有实例，使得模块可以通过 `require()` 来加载。如 #9 所述，它们是等价的。

### 11. setTimeout(callback, milliseconds) {Function}

延迟至少 milliseconds 执行一个 callback。实际的延迟时间由外部因素，如操作系统的计时间隔和系统负载决定。1秒 < milliseconds < 2,147,483,648秒，如果超出这个范围，则会以 1 秒来处理，也就是说一个 Timer 最多能跨越 24.8 天。

返回一个非对外的对象：

```js
{ _idleTimeout: 123,
  _idlePrev: 
   { _idleNext: [Circular],
     _idlePrev: [Circular],
     ontimeout: [Function] },
  _idleNext: 
   { _idleNext: [Circular],
     _idlePrev: [Circular],
     ontimeout: [Function] },
  _onTimeout: [Function],
  _idleStart: Mon Sep 03 2012 22:26:55 GMT+0800 (CST) }
```

### 12. clearTimeout() {Function}

停止执行一个彼时操作，`setTimeout(callback, milliseconds)` 的 callback 将不会执行。

### 13. setInterval(callback, milliseconds) {Function}

在 milliseconds 这个间隔内循环执行 callback。实际的时间同 `setTimeout`，由外部因素决定。间隔的范围也是：1秒 < milliseconds < 2,147,483,648秒，如果超出这个范围，则会以 1 秒来处理。

### 14. clearInterval() {Function}

停止一个循环的执行。

### 15. process {Object}

全局对象还包含进行一个进程对象，在后面会详细讲解。

## 二、console {Object}

如大多数浏览器中的 console 对象一样，其输出会发送到 stdout 或 stderr 中。

### 1. console.log([data], [...])

可以接受多个参数，如有字符替换，则默认后续以逗号分隔的参数会替换成相应的字符，如：

```js
console.log('打印出数字: %d', '1234', '\n没有替换', '\n%s: %s', '第一个\%s', '第二个\%s');
// 打印出数字: 1234 
// 没有替换 
// %s: %s 第一个%s 第二个%s
```

默认支持 3 种类型的字符替换操作，%s, %d, %j。分别代表字符串、数字和JSON。间个 % 并不会替换。

与 `console.log()` 还有几个兄弟：

- console.info 
- console.error 但输出到 stderr
- console.warn 同上
- console.dir 在全局对象中已经介绍过，输出对象下的属性/方法
- console.trace(label) 打印一个名为 label 的堆栈信息到 stderr

### 2. console.time(label) / console.timeEnd(lable)

计算程序运行时间的利器，计算 time 和 timeEnd 之间代码的运算时间，以 label 名来确定一个区间的内容，假设我们创建一个 label 名为 _sofish_ 的计时器，如：

```js
// 这有点类似于 html 的 <div id="sofish"> ... </div>
// 会计算名字为 _sofish_ 的这个区间内 for 循环的运行时间
 console.time('sofish');
 for(var i = 0; i < 1000; i++){
   console.log('计算这 for 循环的运算时间');
 };
 console.timeEnd('sofish');
 // 在我的机器返回: sofish: 26ms
```

### 3. console.assert(expression, [message])

与 `assert.ok()` 一样，当 `expression` 这个传入的表达式返回 false 的话，抛出一个 AssertionError，`message` 为自定义信息。

```js
console.assert(1===0, '出错了')
// AssertionError: 出错了
//    at Object.exports.assert (console.js:75:23)
//    ...

// 如果不指定 message，则返回表达式计算结果
console.assert(1==0)
// AssertionError: false == true
//    at Object.exports.assert (console.js:75:23)
//    ...
```

可能和你常用的测试用例的写法一样。当然，你完全可以用它来做测试用例，或者代替 `throw new Error('hello')`。

## 三、计时器

### 1. setTimeout(callback, delay, [arg], [...])

如上面提到的，`callback` 将会在 `delay` 后执行，执行结果返回一个 `timeoutId` 以便可以使用 `clearTimeout(timeoutId)` 来阻止执行这个计时器。而从第三个参数起，传入参数将作为 `callback` 的参数。

### 2. setInterval(callback, delay, [arg], [...])

参数同 `setTimeout`，功能上 `setTimeout` 只执行一次，而 `setInterval` 则会循环执行。返回一个 `intervalId`，可以使用 `clearInterval(intervalId)` 来停下循环。

## 四、模块

Node 自带一个简单的模块加载系统，文件即模板，它可以通过 `require()` 来加载，比如在 foo.js 中加载一个同目录下的 circle.js，我们可以这样做：

foo.js 中的内容：
```js
var circle = require('./circle.js');
console.log( 'The area of a circle of radius 4 is ' + circle.area(4));
```

circle.js 中的内容：
```js
var PI = Math.PI;

exports.area = function (r) {
  return PI * r * r;
};

exports.circumference = function (r) {
  return 2 * PI * r;
};
```

由于在 Node 中模块中定义的变量仅为模块私有而非全局变量，因此当需要在引用了当前模块的模块内使用，需要用 `exports` 来导出。如上述 circle.js 中导出了 `area()` 和 `circumference()` 两个方法，将可以在 foo.js 中使用。

### 1. 循环引用

当使用 `require()` 循环引用时，一个模块返回的时候可能仍未被执行。考虑一下正面的情况：

a.js:

```js
console.log('a starting');
exports.done = false;
var b = require('./b.js');
console.log('in a, b.done = %j', b.done);
exports.done = true;
console.log('a done');
```

b.js:

```js
console.log('b starting');
exports.done = false;
var a = require('./a.js');
console.log('in b, a.done = %j', a.done);
exports.done = true;
console.log('b done');
```

main.js

```js
console.log('main starting');
var a = require('./a.js');
var b = require('./b.js');
console.log('in main, a.done=%j, b.done=%j', a.done, b.done);
````

当 main.js 加载 a.js，a.js 尝试加载 b.js。在这个时间，b.js 也尝试加载 a.js。为了防止死循环，一个不完全的 a.js 模块副本将会作为在 b.js 这个模块中所调用 a.js 的返回值，b.js 由于得到返回而完成这个加载的过程，而它在 a.js 中被调用，而将本身 `exports` 返回给 a.js。

这时，main.js 也完成了对两个模块的加载，它最终将会返回正面的数据：

```ruby
$ node main.js
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done=true, b.done=true
```

也就是说，你可能需要注意一下不要循环调用以获得更高的性能，一旦你不小心这样搞了，Node 也会帮你阻止这个死循环。

### 2. 内置模块

Node 自身编译了一批内置模块。这些模块也将在这个文档中被详细介绍。它们定义于 Node 源文件的 `lib/` 目录下。

值得注意的是，内置模块总是优先被加载，就拿 `require('http')` 来说，将会返回 HTTP 模块，即使当前目录存在一个 `http.js` 或者 `http/index.js`。 




















