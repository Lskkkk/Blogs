[返回目录](../../README.md)

# MobX响应式原理
本文旨在探究MobX响应式原理中依赖收集与自动执行的部分，实现的代码尽量遵循其原理但不保证使用其源码；

## Demo
一个典型的MobX使用如下：
````
import { observable, autorun } from 'mobx';

const counter = observable(0);
autorun(() => {
  console.log('autorun', counter.get());
});

counter.set(1);

// 运行结果：
// autorun 0
// autorun 1
````
那么如何保证counter更新之后autorun会自动执行呢？

## 依赖收集详解
与Vue中双向绑定类似，在MobX中做到自动执行，也需要进行依赖收集然后触发订阅者的流程，那么看看MobX中是如何做的。

简化版的流程如下：
````
var globalID = 0
function observable(obj) {
    var oID = ++globalID
    return new Proxy(obj, {
        get: function (target, key, receiver) {
            collect.startCollect(oID + '' +key)
            return Reflect.get(target, key, receiver)
        },
        set: function (target, key, value, receiver) {
            Reflect.set(target, key, value, receiver)
            collection[oID + '' + key] && collection[oID + '' + key].forEach(c => {
                c()
            });
        }
    })
}

function autorun(handler) {
    collect.begin(handler)
    handler()
    collect.end()
}

const collection = {}
const collect = {
    begin: function(handler) {
        collection.handler = handler
        collection.now = true
    },
    startCollect: function(oIDKey) {
        if (collection.now) {
            if (collection[oIDKey]) {
                collection[oIDKey].push(collection.handler)
            } else {
                collection[oIDKey] = [collection.handler]
            }
        }
    },
    end: function() {
        collection.now = false
    }
}
````

可以看到，还是主要分为3个部分：
1. 在observable中，通过代理去代理对象的get、set方法，同时在get中去收集依赖，set中去通知订阅者去执行；
2. autorun部分，在初始化时会立即执行一遍，这个过程中对于使用到的observable都会触发其get方法，从而完成依赖的绑定；
3. collect部门，主要维持一个全局的依赖关系，将observable和autorun联系起来；
   
依赖的收集并不是一成不变的，它会随着运行时动态变化，看下面的例子：
````
import { observable, autorun } from 'mobx';

const counter = observable(0);
const foo = observable(0);
const bar = observable(0);
autorun(() => {
    if (counter.get() === 0) {
        console.log('foo', foo.get());
    } else {
        console.log('bar', bar.get());
    }
});

bar.set(10);    // 不触发 autorun
counter.set(1); // 触发 autorun
foo.set(100);   // 不触发 autorun
bar.set(100);   // 触发 autorun

// 结果
// foo 0
// bar 10
// bar 100
````
事实上，每一次autorun的执行，都会动态的改变observable当前的依赖，这点需要特别注意；

# 参考
- [MobX介绍](https://cn.mobx.js.org/refguide/observer-component.html)
- [MobX 原理](https://github.com/sorrycc/blog/issues/3)
- [简单讲讲mobx的observable和autoRun](https://segmentfault.com/a/1190000015049571)