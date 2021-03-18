# 详解 AMD,CMD,ComminJS,ES6 和 UMD 模块化规范

> 开发的时候，我们经常会把某些功能封装成可复用的模块。模块封装了功能，并且对外暴露一个 API。随着 Node.js 的诞生和发展，JavaScript 可以在服务端运行，同时客户端应用也越来越流行，JavaScript 界产生了对优秀和健壮模块系统的需求。在 JavaScript 中定义模块的规范也随之产生。

## 模块方案

![avatar](https://img2020.cnblogs.com/blog/595142/202007/595142-20200708211347397-160641153.png)

## CommonJS

> CommonJS 模块可以说是当前最流行的模块定义规范。相比于 AMD，它的工作效率更高、语法更简单。一开始，CommonJS 模块是 JavaScript 服务器模块的规范。

### 基本原理

> 实现编译代码，创建一个比原先更大的代码包，其中包含了所有应用运行所需的代码。

### 语法

```javascript
const $ = require('juqery')

exprots.init = function () {}
```

### 特点

- 没有回调函数
- 同步加载，每个 require 语句会短暂阻塞代码的运行，知道模块加载完毕。不过这个加载不是通过网络加载，而是从内存或者文件系统中加载，所以这个过程很快。
- 代码简单易懂
- CommonJS 模块不适合浏览器，因为浏览器的加载机制不支持同步。需要用打包工具把所有 CommonJS 模块打包成一个 js 文件。

## AMD-为浏览器设计的模块规范化

AMD 规范参考地址：https://github.com/amdjs/amdjs-api/wiki/AMD

AMD 是 RequireJS 在推广过程中对模块定义的规范化产出。

根据 AMD 规范,我们可以使用 define 定义模块,使用 require 调用模块,语法:

```javascript
define(id?, dependencies?, factory);

/**
 * id: 定义中模块的名字;可选；如果没有提供该参数,模块的名字应该默认为模块加载器请求的指定脚本的名字；
 * dependencies：依赖的模块；
 * factory：工厂方法，返回定义模块的输出值。
 * /


```

> 总结一段话：声明模块的时候指定所有的依赖 dependencies，并且还要当做形参传到 factory 中，对于依赖的模块提前执行，依赖前置

### AMD 示例

```javascript
define('alpha', ['require', 'exports', 'beta'], function (require, exports, beta) {
	exports.verb = function () {
		return beta.verb()
		//Or:
		return require('beta').verb()
	}
})
```

```javascript
define(['alpha'], function (alpha) {
	return {
		verb: function () {
			return alpha.verb() + 2
		},
	}
})
```

```javascript
define({
	add: function (x, y) {
		return x + y
	},
})
```

```javascript
define(function (require, exports, module) {
	var a = require('a'),
		b = require('b')

	exports.action = function () {}
})
```

使用 require 函数加载模块：

```javascript
require([dependencies], function () {})
```

## CMD

> CMD 规范参考地址：https://github.com/seajs/seajs/issues/242 CMD 是 SeaJS 在推广过程中对模块定义的规范化产出。

### define Function

define 是一个全局函数，用来定义模块。

#### define define(factory)

define 接受 factory 参数，factory 可以是一个函数，也可以是一个对象或字符串。

factory 为对象、字符串时，表示模块的接口就是该对象、字符串。比如可以如下定义一个 JSON 数据模块：

```javascript
define({ foo: 'bar' })
```

也可以通过字符串定义模板模块：

```javascript
define('I am a template. My name is {{name}}.')
```

factory 为函数时，表示是模块的构造方法。执行该构造方法，可以得到模块向外提供的接口。factory 方法在执行时，默认会传入三个参数：require、exports 和 module：

```javascript
define(function (require, exports, module) {
	// 模块代码
})
```

#### define define(id?, deps?, factory)

define 也可以接受两个以上参数。字符串 id 表示模块标识，数组 deps 是模块依赖。比如：

```javascript
define('hello', ['jquery'], function (require, exports, module) {
	// 模块代码
})
```

id 和 deps 参数可以省略。省略时，可以通过构建工具自动生成。

注意：带 id 和 deps 参数的 define 用法不属于 CMD 规范，而属于 Modules/Transport 规范。

### define.cmd Object

一个空对象，可用来判定当前页面是否有 CMD 模块加载器：

```javascript
if (typeof define === 'function' && define.cmd) {
	// 有 Sea.js 等 CMD 模块加载器存在
}
```

### require Function

require 是 factory 函数的第一个参数。

#### require require(id)

require 是一个方法，接受 模块标识 作为唯一参数，用来获取其他模块提供的接口。

```javascript
define(function (require, exports) {
	// 获取模块 a 的接口
	var a = require('./a')

	// 调用模块 a 的方法
	a.doSomething()
})
```

#### require.async

```javascript
define(function (require, exports, module) {
	// 异步加载一个模块，在加载完成时，执行回调
	require.async('./b', function (b) {
		b.doSomething()
	})

	// 异步加载多个模块，在加载完成时，执行回调
	require.async(['./c', './d'], function (c, d) {
		c.doSomething()
		d.doSomething()
	})
})
```

### exports

> exports 是一个对象，用来向外提供模块接口。

```javascript
define(function (require, exports) {
	// 对外提供 foo 属性
	exports.foo = 'bar'

	// 对外提供 doSomething 方法
	exports.doSomething = function () {}
})
```

除了给 exports 对象增加成员，还可以使用 return 直接向外提供接口。

```javascript
define(function (require) {
	// 通过 return 直接提供接口
	return {
		foo: 'bar',
		doSomething: function () {},
	}
})
```

## AMD 与 CMD 的主要区别：

1. 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。不过 RequireJS 从 2.0 开始，也改成可以延迟执行（根据写法不同，处理方式不同）。CMD 推崇 as lazy as possible.
2. CMD 推崇依赖就近，AMD 推崇依赖前置。看代码：

```javascript
// CMD
define(function (require, exports, module) {
	var a = require('./a')
	a.doSomething()
	var b = require('./b')
	b.doSomething()
})
// AMD 默认推荐的是
define(['./a', './b'], function (a, b) {
	// 依赖必须一开始就写好
	a.doSomething()
	b.doSomething()
})
```

AMD 与 CMD 的其它区别可参考地址：https://www.zhihu.com/question/20351507

## CommonJS

CommonJS 是服务器端模块的规范,Node.js 采用了这个规范.Node.JS 首先采用了 js 模块化的概念. CommonJS 定义的模块分为:模块引用(require)/ 模块定义(exports)/模块标识(module)

> 所有代码都运行在模块作用域，不会污染全局作用域。

> 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后再加载，就直接读取缓存结果。要想让模块再次运行，必须清除缓存。

> 模块加载的顺序，按照其在代码中出现的顺序。

```javascript
//common.js
module.exports = function (a, b) {
	return a - b
}

let minus = require('./common.js') //文件相对路径
console.log(minus(5, 4))
// 结果： 1
```

详细可以参考[《阮一峰：CommonJS 规范》](https://javascript.ruanyifeng.com/nodejs/module.html)

## ES6

### 导出

1. 命名导出
2. 默认导出

#### 命名导出

```javascript
// 写法1
export const name = 'calculator'
export const add = function (a, b) {
	return a + b
}

// 写法2
const name = 'calculator'
const add = function (a, b) {
	return a + b
}
export { name, add }
```

在使用命名导出时，可以通过 as 关键字对变量重命名

#### 默认导出

```javascript
export default {
    name: 'calculator';,
    add: function (a,b) {
        return a + b;
    }
};
```

### 导入

```javascript
// calculator.js
const name = 'calculator'
const add = function (a, b) {
	return a + b
}
export { name, add }

// 一般导入方式
import { name, add } from './calculator.js'
add(2, 3)

// 通过as关键字对导入的变量重命名
import { name, add as calculateSum } from './calculator.js'
calculateSum(2, 3)

// 使用 import * as <myModule>可以把所有导入的变量作为属性值添加到<myModule>对象中，从而减少了对当前作用域的影响
import * as calculateObj from './calculator.js'
calculateObj.add(2, 3)
```

对于默认导出来说，import 后面直接跟变量名，并且这个名字可以自由指定，它指代了 calculator.js 中默认导出的值。从原理上可以这样理解：

```javascript
import { default as myCalculator } from './calculator.js'
```

还有两种导入方式混合起来使用的例子，如下：

```javascript
// index.js
import React, { Component } from 'react'
```

> 注：这里的 React 必须写在大括号前面，而不能顺序颠倒，否则会提示语法错误。

### 复合写法

在工程中，有时候需要把一个模块导入后立即导出，比如专门用来集合所有页面或组件的入口文件。此时可以采用复合形式的写法：

```javascript
export { name, add } from './calculator.js'
```

复合写法只支持通过命名导出的方式暴露出来的变量，默认导出则没有对应的复合形式，只能将导入和导出拆分开。

```javascript
import calculator from './calculator.js'
export default calculator
```

## UMD

UMD 并不能说是一种模块标准，不如说它是一组模块形式的集合更准确。UMD 的全称是 Universal Module Definition，也就是通用模块标准。它的目标是使一个模块能运行在各种环境下，不论是 CommonJS、AMD，还是非模块化的环境（当时 ES6 Module 还未被提出）

```javascript
;(function (root, factory) {
	if (typeof define === 'function' && define.amd) {
		// AMD
		define(['jquery'], factory)
	} else if (typeof exports === 'object') {
		// Node, CommonJS之类的
		module.exports = factory(require('jquery'))
	} else {
		// 浏览器全局变量(root 即 window)
		root.returnExports = factory(root.jQuery)
	}
})(this, function ($) {
	// 方法
	function myFunc() {}

	// 暴露公共方法
	return myFunc
})
```

详细可参考：[《可能是最详细的 UMD 模块入门指南》](https://www.jianshu.com/p/9f5a0351a532)、[《AMD , CMD, CommonJS，ES Module，UMD》](https://juejin.cn/post/6844903663404580878)
