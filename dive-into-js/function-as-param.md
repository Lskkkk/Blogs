# 函数作为参数时的作用域问题
先看一段代码：
````
let a1 = 0;
const func1 = () => {
    const output1 = () => {
        console.log(a1);
    };
    a1++;
    return output1;
};

let a2 = 0;
const func2 = () => {
    const b = a2;
    const output2 = () => {
        console.log(b);
    };
    a2++;
    return output2;
};

func1()(); // 1
func2()(); // 0
````
其实这个问题很简单，但是一开始遇到的时候的确有点困惑。  
> 为什么func2返回的是++之前的值？
  
理解的关键就在于作用域： 
> 函数执行时使用的变量，要去创建这个函数的作用域取值，而非“调用”这个函数的作用域取值。  
  
output1在执行时，会从func1函数内的局部作用域寻找a1的值，若找不到，则会向外层作用域寻找，因此返回的值是++之后的值；  
output2同理，因为b在创建时的值为0，因此返回的也就是0，不受a2的影响；  

那么如果是对象呢？
````
let a3 = {abc: 0};
const func3 = () => {
    const b = a3;
    const output3 = () => {
        console.log(b);
    };
    a3.abc++;
    return output3;
};

func3()(); // {abc: 1}
````

答案很明显。