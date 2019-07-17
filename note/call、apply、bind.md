## call

### 描述
- 允许为不同的对象分配和调用属于一个对象的函数/方法
- 提供新的**this**值给当前调用的函数/方法

### 使用语法
> fun.call(thisArg, arg1, arg2, ...)

## apply

### 使用语法
> fun.apply(thisArg, [argArray]) // [argArray]为数组或者类数组对象

## bind

### 描述
> 创建一个新的函数（**也就是返回值是一个函数，通常使用变量或者对象中键值对的方式去接收**），函数调用时，**this**指向**bind**的第一个参数，其余参数作为新函数调用时使用，新函数调用时的所有参数也将接在**bind**的其余参数后面调用
> 例如：
```js
function fn(a, b) {} // 原函数
var obj = {};
var newFn = fn.bind(obj, 'a'); // 新函数，'a'作为参数a调用
newFn('b'); // 'b'作为参数b调用
```

### 使用语法
> fun.bind(thisArg[, arg1[, arg2[, ...]]])

## 例子

### call

#### 指定函数上下文（最简单的使用，apply也是）

#### 调用父级构造函数
```js
function Property(faction, property) {
  this.faction = faction;
  this.property = property;
}

function Hero(faction, property, name) {
  Property.call(this, faction, property); // 能用call的时候不用apply
  this.name = name;
}

const morphling = new Hero('dire', 'power', 'morphling');
```

#### 类数组对象使用数组的方法(直接使用会有兼容性问题，例如nodeList直接调用数组的方法)
```js
let nodeList = document.querySelectorAll('div');
[].forEach.call(nodeList, (node, i) => { ... });
```

### apply

#### 把一个数据添加到另一个数组
> 直接使用push需要循环一个个push，使用concat需要使用另一个用一个新数组接收返回值
```js
let arr1 = [1, 2];
let arr2 = [3, 4];
[].push.apply(arr1, arr2); // arr1 变成了[1, 2, 3, 4]，但是push的返回值依然是内部的this.length
```

#### 与内置函数结合使用
```js
let arr = [8, 5, 3, 9, 6];
Math.max.apply(null, arr); // 求arr中的最大值
Math.min.apply(null, arr); // 求arr中的最小值
```
> 浏览器会有参数数量上的限制，如果arr长度超过了限制，浏览器可能会报错，也可能会忽略掉超过的部分，所以可以通过以下方法优化：
```js
// 例如求最大值
let arr = [8, 5, 3, 9, 6];
let min = Infinity
const QUANTUM = 32768;
for (let i = 0; i < arr.length; i += QUANTUM) {
  let subMin = Math.min.apply(null, arr.slice(i, i + QUANTUM)); // arr.slice(i, i + QUANTUM) 改成 arr.slice(i, Math.min(i + QUANTUM, arr.length)) 会更严谨，但slice的第二个参数大于数组长度也是会截取到数组末尾
  min = Math.min(min, subMin);
}
```

### bind

#### 指定函数上下文
> 与call类似的作用，区别在于call是在调用时改变指定this的指向。而bind是在声明的时候，例如
```js
let obj = { a: 1 };

// call
let fn1 = function() { console.log(this.a) };
fn1.call(obj);

// bind
let fn2 = function() { console.log(this.a) }.bind(obj)
fn2()
```
> 另外，如果先使用了**bind**，再使用**call**，**call**是不起作用的，例如上面的: ``fn2.call(obj2)``，fn2中的this并不会指向obj2

#### 偏函数:预设初始参数

### 实现bind
```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    var _this = this;
    var args = [].slice.call(arguments, 1);
    var fn = function() {
      args = args.concat([].slice.call(arguments))
      // 如果是使用new关键字生成的话，this的指向不是oThis
      _this.apply(this instanceof fn ? this : oThis, args);
    }
    if (this.prototype) {
      fn.prototype = this.prototype
    }
    return fn
  }
}

// 使用例子：
function fn(str, str2) {
    this.name = 'fnName';
    console.log(this.name + ' ' + str + ' ' + str2)
}
var a = { name: 'test' }
var fn2 = fn.bind(a, '初始化参数')

fn2('动态参数') // test 初始化参数 动态参数
// 这里的this就不会被bind影响
new fn2('动态参数') // fnName 初始化参数 动态参数
```

