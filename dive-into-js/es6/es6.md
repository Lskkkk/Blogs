[返回目录](../../README.md)

# ES6的一些记录
并不是一篇API文档，API文档请看[ECMAScript 6 简介(阮一峰)](https://es6.ruanyifeng.com/#docs/intro)。

## Reflect
其实就是将Object上的一些方法，放到Reflect对象上，如Object.defineProperty等；
将一些方法的返回值变得更加合理，如Reflect.defineProperty失败的话会返回false；将一些命令式的语法变为函数式的语法。
````
// 老写法
'assign' in Object // true

// 新写法
Reflect.has(Object, 'assign') // true
````
Reflect的语法与Proxy上的一一对应，因此在Proxy中可以方便的调用Reflect的方法，主要有：
- Reflect.apply(target, thisArg, args)
````
// 旧写法
const youngest = Math.min.apply(Math, ages);

// 新写法
const youngest = Reflect.apply(Math.min, Math, ages);
````
- Reflect.construct(target, args)
- Reflect.get(target, name, receiver)
  
  ````Reflect.get(myObject, 'foo');````
- Reflect.set(target, name, value, receiver)
  
````
const myObject = {
    foo: 4,
    set bar(value) {
        return this.foo = value;
    },
};
const myReceiverObject = {
    foo: 0,
};

Reflect.set(myObject, 'bar', 1, myReceiverObject);
myObject.foo // 4
myReceiverObject.foo // 1
// receiver一般为对象，用于表示foo的赋值函数的this指向
````
- Reflect.defineProperty(target, name, desc)
- Reflect.deleteProperty(target, name)

````
// 旧写法
delete myObj.foo;

// 新写法
Reflect.deleteProperty(myObj, 'foo');
````
- Reflect.has(target, name)
- Reflect.ownKeys(target)
- Reflect.isExtensible(target)
- Reflect.preventExtensions(target)
- Reflect.getOwnPropertyDescriptor(target, name)
- Reflect.getPrototypeOf(target)
- Reflect.setPrototypeOf(target, prototype)


## Proxy
Proxy相当于在原对象上加了一层拦截，访问原对象的一些操作都需要经过这层拦截代理，修改一些操作的默认行为。
````
const obj = new Proxy({}, {
    get: function (target, propKey, receiver) {
        console.log(`getting ${propKey}!`);
        return Reflect.get(target, propKey, receiver);
    },
    set: function (target, propKey, value, receiver) {
        console.log(`setting ${propKey}!`);
        return Reflect.set(target, propKey, value, receiver);
    }
});

obj.count = 1
//  setting count!
++obj.count;
//  getting count!
//  setting count!
//  2
````

不只是只针对对象的set、get，proxy还可以针对函数，例如：
````
var handler = {
    get: function(target, name) {
        if (name === 'prototype') {
            return Object.prototype;
        }
        return 'Hello, ' + name;
    },
    apply: function(target, thisBinding, args) {
        return args[0];
    },
    construct: function(target, args) {
        return {value: args[1]};
    }
};

var fproxy = new Proxy(function(x, y) {
    return x + y;
}, handler);

fproxy(1, 2) // 1
new fproxy(1, 2) // {value: 2}
fproxy.prototype === Object.prototype // true
fproxy.foo === "Hello, foo" // true
````
- proxy作为函数调用，即触发apply方法；
- proxy作为构造函数使用，即触发construct方法；

一共有13种用法，更多的api请参考 https://es6.ruanyifeng.com/#docs/proxy；

# 参考
- [ECMAScript 6 简介(阮一峰)](https://es6.ruanyifeng.com/#docs/intro)