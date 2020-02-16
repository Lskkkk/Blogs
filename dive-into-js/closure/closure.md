[返回目录](../../README.md)

# 闭包

## 隐形闭包
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
    a: 1,
    getA: null,
    changeA: null
};
testA(test);
test.getA(); // 3
test.changeA();
test.getA(); // 4
````