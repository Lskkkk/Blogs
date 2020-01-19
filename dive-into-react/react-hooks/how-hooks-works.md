# React hooks原理
## 疑问
- hooks调用顺序是如何保证的？
- 函数组件每次运行时hooks是如何支持记忆的？

## 原理
- react hooks使用链表结构，串联所有的hooks，保证hooks的执行；
````
type Hooks = {
	memoizedState: any, // 指向当前渲染节点 Fiber
    baseState: any, // 初始化 initialState， 已经每次 dispatch 之后 newState
    baseUpdate: Update<any> | null,// 当前需要更新的 Update，每次更新完之后，会赋值上一个 update，方便 react 在渲染错误的边缘，数据回溯
    queue: UpdateQueue<any> | null,// UpdateQueue 通过
    next: Hook | null, // link 到下一个 hooks，通过 next 串联每一 hooks
}

type Effect = {
    tag: HookEffectTag, // effectTag 标记当前 hook 作用在 life-cycles 的哪一个阶段
    create: () => mixed, // 初始化 callback
    destroy: (() => mixed) | null, // 卸载 callback
    deps: Array<mixed> | null,
    next: Effect, // 同上 
}
````
- hooks的值存在于memoizedState中，每次rerender时，如果已经存在，则不会重新创建，继续使用存在的值，就保证了记忆功能；

## 实现
以useState为例，其需求如下：
- 输入参数为initialState，返回state值和setState的数组
- 后续组件rerender不会修改state值，state只能有setState修改  

所以代码实现如下：
````
import render from '../render';

let state = null;
const setStateT = (newState) => {
    state = newState;
    render(); // 模拟react的render，将整个view重新渲染一遍
};
const useStateT = (initialState) => {
    if (!state) {
        state = initialState;
    }
    return [state, setStateT];
};

export default useStateT;
````
此useStateT和官方useState用法一样，但是有个明显的缺点，即因为state是保存在同一个变量中，所以存在多个state时就会产生互相影响覆盖的情况；
  
接着修改，使用数组存储state，即可解决这个问题：
````
import render from '../render';

let state = [];
let currentIndex = 0;
const useStateT = (initialState) => {
    if (!state[currentIndex]) {
        state[currentIndex] = initialState;
    }
    const tmpIndex = currentIndex;
    const setStateT = (newState) => {
        state[tmpIndex] = newState;
        currentIndex = 0;
        render();
    };
    return [state[currentIndex++], setStateT];
};

export default useStateT;
````
除了用数组存储state之外，为了保证state的执行顺序，需要用一个全局下标来指向当前使用的hook。
  
这样就完全实现了useState。

接下来实现useEffect，其需求为：
- 输入参数为callback（必须）和一个表示依赖项的数组（非必须）；
- callback执行后若有输出函数，则其执行时机为下次render执行之前；
- 当依赖项为[]时，useEffect只执行一次(didMount)；
- 当依赖项为undefined时，useEffect每次render之后都会执行(didMount+didUpdate)；
- 当依赖项数组不为空，仅在其中的参数改变时，useEffect才会在render后执行；

代码实现如下：
````
import render from '../render';

let state = [];
let currentIndex = 0;
let watches = []; // 存储每个useEffect需要监听的数据
let lastCleanEffect = []; // 存储每个useEffect提供的clean effect的方法
let isFirstRender = true;

const useStateT = (initialState) => {
    if (!state[currentIndex]) {
        state[currentIndex] = initialState;
    }
    const tmpIndex = currentIndex;
    const setStateT = (newState) => {
        state[tmpIndex] = newState;
        currentIndex = 0;
        isFirstRender = false;
        render();
    };
    return [state[currentIndex++], setStateT];
};

const useEffectT = (callback, watch) => {
    lastCleanEffect[currentIndex] && lastCleanEffect[currentIndex]();
    if (!watches[currentIndex]) {
        watches[currentIndex] = watch;
    }
    if (watches[currentIndex] === undefined) { // didMount+didUpdate
        lastCleanEffect[currentIndex] = callback();
    } else if (watches[currentIndex].length === 0) { // didMount
        if (isFirstRender) {
            callback();
        }
    } else if (JSON.stringify(watches[currentIndex]) !== JSON.stringify(watch)) { // 这里比较方法偷点懒，其实应该是浅比较
        lastCleanEffect[currentIndex] = callback();
    }
    currentIndex++;
};

export {
    useStateT,
    useEffectT
};
````

## 参考
- [React Hooks 原理](https://github.com/brickspert/blog/issues/26)
- [React hooks: not magic, just arrays](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)