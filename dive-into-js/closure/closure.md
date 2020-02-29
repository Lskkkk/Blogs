[返回目录](../../README.md)

# 闭包
闭包提供了外部环境访问函数内部的途径，函数内部的变量会始终存在于内存中。

## 显式闭包
一般的闭包形式为函数中返回一个函数，用于外部访问函数内部变量。
````
const testA = () => {
    const a = 0;
    return () => console.log(a);
};

const a = testA();
a(); // 0
````

## 隐式闭包
还有一种不那么明显的闭包，在一个函数中不直接return函数，而是为一个对象设置属性方法，这种方式也是闭包的一种；
一般的，Object.defineProperty也是这种形式。
````
const testA = (data) => {
    let _a = 3;
    defineA(data, _a);
};

const defineA = (data, _a) => {
    data.getA = () => {
        console.log(_a);
    };
    data.changeA = () => _a++;
};

const test = {
    getA: null,
    changeA: null
};
testA(test);
test.getA(); // 3
test.changeA();
test.getA(); // 4
````