你还在用defineProperty劫持对象吗快试试Proxy吧

> 在 JavaScript 中，`Object.defineProperty()` 一直被用来监听对象属性的变化。ES6引入了新的特性 `Proxy`，它可以代理一个对象并拦截对该对象的所有访问以提供自定义行为。

```js
// defineProperty
Object.defineProperty(obj, prop, handler)
// Proxy
new Proxy(obj, handler)
```

## 常用

+ proxy

  - `get(target, key, receiver)`：在读取属性值时拦截。

  - `set(target, key, value, receiver)`：在设置属性值时拦截。

  - `has(target, key)`：判断某个属性是否存在时拦截。

  - `deleteProperty(target, key)`：在删除属性时拦截。

  - `apply(target, thisArg, args)`：在函数调用时拦截。

  - `construct(target, args, newTarget)`：在实例化对象时拦截。

  - `clear(target)`：清空 Map 或 Set 时拦截。 

  - `forEach(target, callbackFn, thisArg)`：在使用 forEach 方法遍历 Map 或 Set 时拦截。

  - `defineProperty(target, key, descriptor)`：在定义(增加)对象属性时拦截。

  -  `getOwnPropertyDescriptor(target, key)`：获取对象属性描述时拦截。 

  - `getPrototypeOf(target)`：获取对象原型时拦截。 

  - `setPrototypeOf(target, proto)`：设置对象原型时拦截。 

  - `isExtensible(target)`：判断对象是否可扩展时拦截。 

  - `preventExtensions(target)`：防止对象扩展时拦截。 

  - `ownKeys(target)`：获取对象所有自有属性的键值数组时拦截。
  - ...

+ deleteProperty

  + `get(target, key, receiver)`：在读取属性值时拦截。
  + `set(target, key, value, receiver)`：在设置属性值时拦截。
  + `apply` 用于捕获函数的调用。
  + `construct` 用于捕获类的构造函数的调用。
  + `has` 用于捕获 in 操作符。
  + `ownKeys` 用于捕获 object.keys、Object.getOwnPropertyNames、Object.getOwnPropertySymbols 的操作。
  + `getOwnPropertyDescriptor` 用于捕获 Object.getOwnPropertyDescriptor 的操作。
  + `defineProperty` 用于捕获 Object.defineProperty 的操作。
  + `deleteProperty` 用于捕获 delete 操作符。
  + ....

## this指向区别

`Object.defineProperty`中，回调函数中的`this`指向的是要被监听属性的对象本身。

```js
let obj = {name: 'Tom', age: 18}
Object.defineProperty(obj, 'name', {
  set: function(newVal) {
    console.log(`name属性值从${this.name}变成了${newVal}`)
    this._name = newVal
  },
  get: function() {
    return this._name
  }
})
obj.name = 'Jerry' // 控制台打印出：name属性值从Tom变成了Jerry
console.log(obj.name) // 控制台打印出：Jerry
```

而`Proxy`中的回调函数则是通过第一个参数获取目标对象的引用，因此回调函数中的`this`则指向`Proxy`**实例本身**，而非目标对象。

```js
let obj = {name: 'Tom', age: 18}
let proxyObj = new Proxy(obj, {
    set: function(target, key, newVal, receiver) {
        console.log(`属性${key}的值从${target[key]}变为${newVal}`)
        console.log('this[name]: '+this.name)
        target[key] = newVal
        return true
    },
    get: function(target, key, receiver) {
        return target[key]
    }
})
proxyObj.name = 'Jerry' // 控制台打印出：属性name的值从Tom变为Jerry
// this[name]: undefined
console.log(proxyObj.name) // 控制台打印出：Jerry
```

`proxy`劫持会返回一个新的对象，**不会对源对象有影响**。

## 劫持对象属性

如果我们想要监听一个对象中的某个属性的变化，可以使用 `defineProperty`。

```js
const obj = {}
let value = 1
Object.defineProperty(obj, 'a', {
  get() {
    console.log('get:', value)
    return value
  },
  set(newValue) {
    console.log('set:', newValue)
    value = newValue
  }
})

console.log(obj.a) // get: 1, 1
obj.a = 2          // set: 2
console.log(obj.a) // get: 2, 2
```

这里我们监听了 `obj` 对象的一个属性 `a`。当获取或者设置 `a` 属性值时，会触发相应的 get 和 set 函数。

使用 Proxy 可以监听整个对象：

```js
const obj = {
  a: 1
}

const proxyObj = new Proxy(obj, {
  get(target, prop) {
    console.log('get:', target[prop])
    return target[prop]
  },
  set(target, prop, value) {
    console.log('set:', value)
    target[prop] = value
    return true
  }
})

console.log(proxyObj.a) // get: 1, 1
proxyObj.a = 2          // set: 2
console.log(proxyObj.a) // get: 2, 2
```

这里我们可以看到，每次获取或者设置 `proxyObj` 的属性值时，都会触发对应的 get 或者 set 函数。

## 劫持数组

如果要监听数组的变化，可以使用 `defineProperty` 来监听数组的 push、pop、shift、unshift 等方法：

```js
const arr = []
Object.defineProperty(arr, 'push', {
  value: function(...args) {
    console.log('push:', ...args)
    Array.prototype.push.apply(this, args)
  }
})

arr.push(1, 2) // push: 1 2

console.log(arr) // [1, 2]
```

这里我们重写了 `push` 方法，每次调用 `push` 方法时都会触发拦截函数，这也是**vue2**的解决手段。

使用 Proxy 同样可以监听数组的变化：

```js
const arr = [1, 2]

const proxyArr = new Proxy(arr, {
  get(target, prop) {
    console.log('get:', target[prop])
    return target[prop]
  },
  set(target, prop, value) {
    console.log('set:', value)
    target[prop] = value
    return true
  }
})

proxyArr.push(3) // set: 3

console.log(proxyArr) // [1, 2, 3]
```

这里我们可以看到，每次调用 `push` 方法时，都会触发对应的拦截函数。
