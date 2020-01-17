## Observer需要实现的功能
例如vue里面的data，修改data中的数据的时候，能够触发视图更新

### 先实现最简单的值为常量的情况
例如
``` js
const data = {
  name: '魁拔',
  title: '十万火急'
}
```
定义`observe`函数：遍历data的所有属性，并对属性进行处理
``` js
function observe (data) {
  Object.keys.forEach(key => {
    defineReactive(data, key, data[key])
  })
}
```
实现`defineReactive`：利用`Object.defineProperty()`对对象属性进行重写
``` js
function defineReactive (data, key, val) {
  Object.defineProperty(data, key, {
    get () {
      // 这里不能return data[key]，会一直触发get，造成死循环
      return val
    },
    set (newVal) {
      // 同理这里使用data[key]去判断也会出现死循环
      if (val === newVal) reutrn
      val = newVal
      console.log('更新视图')
    }
  })
}
```
> 注意：这里的val，为常量的时候，可以看做是一个中间变量。为引用类型时是对data[key]的值的一个引用，此时调用val并不会执行data[key]的get方法
此时，执行以下代码就会打印“更新视图”
``` js
observe(data)
data.title = '战神崛起'
```

### 深度监听data
data下的属性值也可能是一个对象，需要做以下修改
``` js
function observe (data) {
  if (typeof data !== 'object' || data === null) return
  Object.keys(data).forEach(key => {
    defineReactive(data, key, data[key])
  })
}

function defineReactive (data, key, val) {
  observe(val)
  Object.defineProperty(data, key, {
    get () {
      return val
    },
    set (newVal) {
      if (val === newVal) reutrn
      val = newVal
      console.log('更新视图')
    }
  })
}
```
这个例子也就能触发“视图更新”
``` js
const data = {
  name: '魁拔',
  title: '十万火急',
  exp: {
    boxOffice: 100
  }
}
observe(data)
data.exp.boxOffice = 200
```

### 传给observe函数的是一个数组
需要遍历数组，把数组的每一项再传给observe
``` js
function observe (data) {
  if (typeof data !== 'object' || data === null) return
  if (Array.isArray(data)) {
    for (let i = 0; i < data.length; i += 1) {
      observe(data[i])
    }
  } else {
    Object.keys(data).forEach(key => {
      defineReactive(data, key, data[key])
    })
  }
}
```
这个例子也就能触发“视图更新”
``` js
const data = {
  name: '魁拔',
  title: '十万火急',
  version: [1, 2, { n: 100 }]
}
observe(data)
data.version[2].n = 200
```

### push,pop,shift,unshift,splice,sort,reverse也能触发视图更新
修改代码
``` js
const arrayProto = Array.prototype
// 复制一份新的原型，避免对数组的原型造成污染
const proto = Object.create(arrayProto)
// 改写数组的7个方法
;['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(method => {
  proto[method] = function (...args) {
    // Array.prototype[method]
    const result = arrayProto[method].call(this, ...args)
    console.log('更新视图')
    return result
  }
})
function observe (data) {
  if (typeof data !== 'object' || data === null) return
  if (Array.isArray(data)) {
    // 给数组设置新的原型
    // data.__proto__ = proto
    Object.setPrototypeOf(data, proto)
    for (let i = 0; i < data.length; i += 1) {
      observe(data[i])
    }
  } else {
    Object.keys(data).forEach(key => {
      defineReactive(data, key, data[key])
    })
  }
}
function defineReactive (data, key, val) {
  observe(val)
  Object.defineProperty(data, key, {
    get () {
      return val
    },
    set (newVal) {
      if (val === newVal) reutrn
      val = newVal
      console.log('更新视图')
    }
  })
}
```
这个例子也就能触发“视图更新”
``` js
const data = {
  name: '魁拔',
  title: '十万火急',
  version: [1, 2]
}
observe(data)
data.version.push(3)
```

### 数组方法中如果增加了元素也是对象的话，也需要进行监听
增加元素的方法有push、unshift、splice，所以最后的代码为：
``` js
const arrayProto = Array.prototype
const proto = Object.create(arrayProto)
// 改写数组的7个方法
;['push', 'pop', 'shift', 'unshift', 'splice', 'sort', 'reverse'].forEach(method => {
  proto[method] = function (...args) {
    // Array.prototype[method]
    const result = arrayProto[method].call(this, ...args)
    let inserted
    switch (method) {
      case 'push':
      case 'unshift':
        inserted = args
        break
      case 'splice':
        inserted = args.slice(2)
        break
    }
    observeArray(inserted)
    console.log('更新视图')
    return result
  }
})

function observe (data) {
  if (typeof data !== 'object' || data === null) return
  if (Array.isArray(data)) {
    // data.__proto__ = proto
    Object.setPrototypeOf(data, proto)
    observeArray(data)
  } else {
    Object.keys(data).forEach(key => {
      defineReactive(data, key, data[key])
    })
  }
}

function observeArray (data) {
  for (let i = 0; i < data.length; i += 1) {
    observe(data[i])
  }
}

function defineReactive (data, key, val) {
  observe(val)
  Object.defineProperty(data, key, {
    get () {
      return val
    },
    set (newVal) {
      if (val === newVal) reutrn
      val = newVal
      console.log('更新视图')
    }
  })
}
```
``` js
const data = {
  name: '魁拔',
  title: '十万火急',
  version: [1, 2]
}
observe(data)
data.version.push({ n: 100 }) // 添加对象，对象也需要监听
data.version[2].n = 200 // 更改添加对象的属性
```
