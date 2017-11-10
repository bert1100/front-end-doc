# 各种module的写法

javascript的模块化编程各种写法备忘录


## CommonJS

同步加载规范，一模块一文件，倾向于服务端的模块化规范。

> 当第一次require一个模块文件时，实现原理是：执行这个模块文件，并在内存生成这样一个对象：{ id:'模块名', exports:{…}, loaded:true, … }；当再次require时，则直接从缓存中取到它。
>
> nodejs中的模块采用的是这种规范。

### 特性 features

- 同步阻塞：当模块requie到以后才能继续，这种同步加载方式并不适合浏览器端。
- 在本地磁盘运行表现佳

### 写法示例

#### 模块的定义

```javascript
// math.js
exports.add = function(a,b){
    return a + b;
}
```

#### 模块的使用

```javascript

// 从全局或者node_modules中require模块,可忽略
// require("someModules");
// 从当前目录中require自定义的模块，一般不可忽略后缀名.js
// require("./somefile.js")

var math = require("math");
math.add(2,3);  // =>5
```

#### 谁在用

- 服务器端的nodejs（当然了，你本地上跑的nodejs也算哦）
- Browserify,浏览器端的CommonJS实现，可以使用npm模块，但编译打包后的文件体积可能很大




## AMD

异步加载模块定义规范

> AMD是”Asynchronous ModuleDefinition”的缩写（Asynchronous读音：/ei'siŋkrənəs/ ），即”异步模块定义”。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。 这里异步指的是不堵塞浏览器其他任务（dom构建，css渲染等），而加载内部是同步的（加载完模块后立即执行回调）。

### 特性features

- 依赖前置：当有多个依赖时，`必须`将所有的依赖都写在define()函数第一个参数数组中。
- 异步加载：它会当require中第一个参数数组中的依赖模块加载完，才进行callback回调。
> 在这里，很多人会认为，这不是同步的表现么？！其实，这并不冲突。注意`异步`的本质概念；代码运行并没有阻塞而停止下来，仍然会执行后面的其他代码。

- 适合在浏览器环境中异步加载模块
- 可以并行加载多个模块
> 但是太难书写了，不符合通用模块化思维方式

### 写法示例

#### 模块的定义

```javascript

// 定义语法：define(id?, dependencies?, factory)
// @param id?: 模块名字，默认为模块的文件名
// @param dependencies?: 模块的依赖标识，数组字面量,可忽略。
// @param factory: 模块的工厂函数，模块初始化要执行的函数或对象，若有return的对象即为模块的输出exports.

// math.js--- 无依赖写法AMD
define(function(){
  var add = function(x,y){return x + y;}
  return { add: add};
})

// test.js--- 有依赖的写法AMD
define(['a', 'b'], function(a, b) {
  a.doSomething();
  b.doSomething();
})

```



#### 模块的使用

```javascript
// main.js
require(['math'],function(math){ // 提示：所依赖的模块必须放在文件的前面
  alert(math.add(2,3));
})
```

#### 谁在用
- RequireJS
- curl


## CMD
Common Module Definition 规范，尽量保持简单，和CommonJS保持了最大的兼容性。
seajs的CMD推崇**依赖就近，延迟执行**。可以把你的依赖写进代码的任意一行，和AMD不同。

> define(factory)中的`factory`为函数时，表示是模块的构造方法。执行该构造方法，可以得到模块向外提供的接口。factory 方法在执行时，默认会传入三个参数：require、exports 和 module.
>
> CMD标准倾向于在使用过程中提出依赖，就是不管代码写到哪突然发现需要依赖另一个模块，那就在当前代码用require引入就可以了，规范会帮你搞定预加载，你随便写就可以了。但是AMD标准让你必须提前在头部依赖参数部分写好（没有写好？ 倒回去写好咯），这就是最明显的区别。

### 特性features

- 依赖就近：写到哪儿，哪里就能require
- 延迟执行：按需异步加载后，再执行。
- 可以很容易在Node.js中运行
> 缺点： 依赖SPM打包，模块的加载逻辑偏重。也就是说不好打包。

### 写法示例

#### 模块的定义

```javascript
// CMD
define(function(require,exports,module){
  var a = require('./a');
  a.dosomething();
  var b = require('./b');
  b.dosomething();
  // 通过 exports 对外提供接口
  exports.doSomething = ...
  // 或者通过 module.exports 提供整个接口
  module.exports = ...
})
```

#### 模块的使用

```javascript
// 在页面中的用法： seajs.use(id, callback?)

// 加载一个模块
seajs.use('./a');

// 加载一个模块，在加载完成时，执行回调
seajs.use('./a', function(a) {
  a.doSomething();
});

// 加载多个模块，在加载完成时，执行回调
seajs.use(['./a', './b'], function(a, b) {
  a.doSomething();
  b.doSomething();
});
```

#### 谁在用
- Seajs
- coolie


## es6

es6通过`import`、`export`实现模块的输入输出。其中import命令用于输入其他模块提供的功能，export命令用于规定模块的对外接口。

> 一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。如果希望外部文件能够读取该模块的变量，就需要在这个模块内使用export关键字导出变量，注意：export导出的变量只能位于文件的顶层，否则会报错

### 特性feature

- 语法ES6（es2015）
- 模块内作用域独立
- 模块内指针特性（非拷贝）：修改内部状态，会影响到模块本身。

### 写法示例

#### 模块的定义

```javascript
// foo.js
export var foo = 'foo';

setTimeout(function() {
  foo = 'foo2';
}, 500);

// abc.js---导出变量
var a = 1;
var b = 2;
var c = 3;
export {a, b, c}


// foo.js --导出函数1
export function foo(){}

// foo-bar.js --导出函数2
function foo(){}
function bar(){}

export {foo, bar as bar2}

// class.js --导出类
export default class {} // 使用default关键字，不必使用{}

// main.js
import * as m from './foo';

console.log(m.foo); // foo
setTimeout(() => console.log(m.foo), 500); // foo2
```

#### 模块的使用

```javascript
// abc.js
var a = 1;
var b = 2;
var c = 3;
export {a, b, c}

// main.js
import {a, b, c} from './abc';
console.log(a, b, c);

// main2.js ---使用as关键字修改名字
import {a as aa, b, c};
console.log(aa, b, c)

// main3.js ---import语句可以和export语句写在一起
import {a, b, c} from './abc';
export {a, b,  c}

// 使用连写, 可读性不好，不建议
export {a, b, c} from './abc';

// main4.js ---使用default关键字
import {default as aa} from './abc';
console.log(aa);
```

#### 谁实现了它
- BaBel
> 读音： /ˈbebəl/



## UMD

Universal Module Definition 规范是一个类似于 CommonJS 和 AMD的语法糖，它似乎去解决模块定义的跨平台问题。
解决不同加载规范的规范。

> 由于CommonJS是服务器端的规范，跟AMD、CMD两个标准实际不冲突。当我们写一个文件需要兼容不同的加载规范的时候怎么办呢?



```javascript
// 以 加载 jquery 和 underscore 为例 解决加载器的兼容问题
(function (root, factory) {

    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery', 'underscore'], factory);

    } else if (typeof exports === 'object') {

        // Node, CommonJS之类的

        module.exports = factory(require('jquery'), require('underscore'));

    } else {

        // 浏览器全局变量(root 即 window)

        root.returnExports = factory(root.jQuery, root._);

    }

}(this, function ($, _) {

    // 方法

    function a(){}; // 私有方法，因为它没被返回 (见下面)

    function b(){}; // 公共方法，因为被返回了

    function c(){}; // 公共方法，因为被返回了



    // 暴露公共方法

    return {

        b: b,

        c: c

    }

}));
```

## 参考资料

[该如何理解AMD ，CMD，CommonJS规范–javascript模块化加载学习总结](http://www.tuicool.com/articles/MVrMBrI)

[AMD/CMD与前端规范](http://www.tuicool.com/articles/2ARZf2v)

[前端模块化之旅（二）：CommonJS、AMD和CMD](http://www.tuicool.com/articles/ZFvIfmz)

[研究一下javascript的模块规范（CommonJs/AMD/CMD）](http://www.tuicool.com/articles/y2uqeaM)

[Javascript模块化编程（一）：模块的写法](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html)

[Javascript模块化编程（二）：AMD规范](http://www.ruanyifeng.com/blog/2012/10/asynchronous_module_definition.html)

[Javascript模块化编程（三）：require.js的用法](http://www.ruanyifeng.com/blog/2012/11/require_js.html)

[Module](http://es6.ruanyifeng.com/#docs/module)
