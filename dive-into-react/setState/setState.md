[返回目录](../../README.md)
# setState详解
先说结论：
- setState只在合成事件和生命周期钩子函数中是“异步”的，在原生事件和延时函数中都是同步的；
- “异步”的原因是合成事件和生命周期钩子函数执行在state更新结果之前，并不是因为setState中执行是异步的；
- state批量更新机制，如果是同一个值，会导致只应用最后一个更新，如果是不同的值，会合并批量更新。

## 异步Or同步
示例代码：[github](https://github.com/Lskkkk/Demo/tree/master/set-state-demo/src)

### 合成事件、生命周期钩子函数
> 合成事件是指react封装的统一的代理的事件，如onClick、onLayout等等。
  
在合成事件和生命周期中，setState后不能立刻取到更新后的state值。看一个例子：
````
import * as React from 'react';

export default class AsyncTest extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            eventCount: 0,
            hookCount: 0
        };
    }

    componentDidMount() {
        this.setState({
            hookCount: this.state.hookCount + 1
        });
        console.log(`async hook count: initial 0, now ${this.state.hookCount}`);
    }

    render() {
        return (
            <div style={{ width: 100, height: 100, background: '#4373' }} onClick={this.onClick}>
                async click
            </div>
        );
    }

    onClick = () => {
        this.setState({
            eventCount: this.state.eventCount + 1
        });
        console.log(`async event count: initial 0, now ${this.state.eventCount}`);
    }
}

// 点击后，输出
// async hook count: initial 0, now 0
// async event count: initial 0, now 0
````

### 原生事件与延时函数
> 原生事件是指Dom层直接触发的事件，如元素的点击等等，一般会以document.querySelector('#id').addEventListener绑定；
  
在原生事件和延时函数中更新setState后能直接访问到state的值。

````
import * as React from 'react';

export default class SyncTest extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            eventCount: 0,
            timeOutCount: 0
        };
    }

    componentDidMount() {
        document.querySelector('#sync').addEventListener('click', this.onClick);
        setTimeout(() => {
            this.setState({
                timeOutCount: this.state.timeOutCount + 1
            });
            console.log(`sync timeOut count: initial 0, now ${this.state.timeOutCount}`);
        }, 0);
    }

    render() {
        return (
            <div id={'sync'} style={{ width: 100, height: 100, background: '#9383' }}>
                sync click
            </div>
        );
    }

    onClick = () => {
        this.setState({
            eventCount: this.state.eventCount + 1
        });
        console.log(`sync event count: initial 0, now ${this.state.eventCount}`);
    }
}

// 点击后输出: 
// sync timeOut count: initial 0, now 1
// sync event count: initial 0, now 1
````

## 批量更新
- 多次setState会合并，统一更新后再rerender；
- 使用受上面异步的影响，对同一值使用setState只会应用一次，要应用多次，setState时需要传入Updater Function；

````
import * as React from 'react';

export default class QueueTest extends React.Component {
    constructor(props) {
        super(props);
        this.refresh = 0;
        this.state = {
            queueCount: 0,
            anotherCount: 0,
            funcCount: 0
        };
    }

    componentDidMount() {
        // 多次对同一个值setState，每次this.state.queueCount值都是一样的，看起来只应用了一次
        this.setState({ queueCount: this.state.queueCount + 1 });
        this.setState({ queueCount: this.state.queueCount + 1 });
        this.setState({ queueCount: this.state.queueCount + 1 });

        // 使用callback对setState，state值更新完后才执行下一个，因此都能更新
        this.setState((state) => ({ funcCount: state.funcCount + 1 }));
        this.setState((state) => ({ funcCount: state.funcCount + 1 }));
        this.setState((state) => ({ funcCount: state.funcCount + 1 }));

        // 同时对不同的值setState则不受影响
        this.setState({ anotherCount: this.state.anotherCount + 1 });

    }

    render() {
        console.log(`refresh count: ${++this.refresh}`);
        console.log(JSON.stringify(this.state));
        return (
            <div id={'sync'} style={{ width: 400, height: 100, background: '#0193' }}>
                queue test
            </div>
        );
    }
}

// 输出：
// refresh count: 1
// {"queueCount":0,"anotherCount":0,"funcCount":0}

// refresh count: 2
// {"queueCount":1,"anotherCount":1,"funcCount":3}
````

## 参考
- [你真的理解setState吗？](https://juejin.im/post/5b45c57c51882519790c7441#heading-0)
- [揭密React setState](https://imweb.io/topic/5b189d04d4c96b9b1b4c4ed6)