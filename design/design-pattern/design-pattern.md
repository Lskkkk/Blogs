[返回目录](../../README.md)

# JavaScript设计模式
常见JavaScript设计模式如下：

- [单例模式](##单例模式)
- [策略模式](##策略模式)

## 单例模式
> 单例模式是指一个类只提供一个供全局访问的实例。

代码演示如下：  
````
// single.js
// 实例对象
class Instance {
    constructor(name) {
        this.name = name;
    }
    getName() {
        return this.name;
    };
}

// 代理实例的工具
let instance = null;
const proxySingle = (name) => {
    if (instance === null) instance = new Instance(name);
    return instance;
};

exports.proxySingle = proxySingle;
````
````
// index.js
const proxySingle = require('./single').proxySingle;

const singleA = proxySingle('A');
const singleB = proxySingle('B');

console.log(singleA === singleB);
console.log(singleB.getName() === 'A');
````

使用场景：只需要存在一个实例的对象或者组件，如模态框，全局对象等等。

## 策略模式
> 策略模式是指定义一系列算法，封装不同处理逻辑，并且他们可以相互替换。将算法的实现和算法的使用隔离开来。

### 例子1：奖金计算
年终奖等级有'S', 'A', 'B', 'C'四种，分别对应的奖金策略为：
- 'S'：三倍工资，并且额外+1w
- 'A'：两倍工资，额外+5k
- 'B'：一倍工资，+1k
- 'C'：一倍工资，-2k

根据每个员工工资乘以系数则为年终奖。

原实现：
````
(() => {
    const calculate = (level, salary) => {
        let bonus = 0;
        if (level === 'S') {
            bonus = calculateS(salary);
        } else if (level === 'A') {
            bonus = calculateA(salary);
        } else if (level === 'B') {
            bonus = calculateB(salary);
        } else { // 如果有新的等级'D'，需要破坏这里的代码
            bonus = calculateC(salary);
        }
        return bonus;
    }
    // 实际情况的计算逻辑可能比这复杂很多，因此大部分情况下会提取计算逻辑为单独函数
    const calculateS = (salary) => salary * 3 + 10000;
    const calculateA = (salary) => salary * 2 + 5000;
    const calculateB = (salary) => salary * 1 + 1000;
    const calculateC = (salary) => salary * 1 - 2000;
    console.log(calculate('S', 1000));
    console.log(calculate('A', 2000));
})();
````
使用JavaScript实现策略模式：
````
(() => {
    const strategy = {
        'S': (salary) => salary * 3 + 10000,
        'A': (salary) => salary * 2 + 5000,
        'B': (salary) => salary * 1 + 1000,
        'C': (salary) => salary * 1 - 2000
    };
    const calculate = (level, salary) => strategy[level](salary);

    console.log(calculate('S', 1000));
    console.log(calculate('A', 2000));
})();
````
这个例子中，新增年终奖等级即在strategy对象新增策略即可。

### 例子2：表单校验
再看一个表单校验的例子，需求如下：
表单有三种验证，
- 名字不为空
- 密码长度不小于6位
- 手机号为数字

对于表单1，检验名字和密码；对于表单2，检验名字、密码和手机号。

原实现：
````
(() => {
    const validate = (data1, data2) => {
        const checkName = (value) => value !== '';
        const checkPassword = (value) => value.length >= 6;
        const checkPhone = (value) => /(^1[3|5|8][0-9]{9}$)/.test(value);

        const result1 = checkName(data1.name) && checkPassword(data1.password);
        const result2 = checkName(data2.name) && checkPassword(data2.password) && checkPhone(data2.phone);
        return result1 && result2;
    }

    console.log(validate(
        { name: 'zhang', password: '123', phone: '13333333333' },
        { name: 'liu', password: '123456', phone: '13333333333' },
    ));
})();
````
使用JavaScript实现策略模式：
````
(() => {
    const validator = {
        name: (value) => value !== '',
        password: (value) => value.length >= 6,
        phone: (value) => /(^1[3|5|8][0-9]{9}$)/.test(value)
    };
    class Context {
        constructor(rules) { this.setRules(rules); }
        setRules(rules) { this.rules = rules; }
        doRules(data) { return this.rules.every(rule => validator[rule](data[rule])); }
    }

    const validate = (data1, data2) => {
        const context = new Context([]);
        context.setRules(['name', 'password']);
        const result1 = context.doRules(data1);
        
        context.setRules(['name', 'password', 'phone']);
        const result2 = context.doRules(data1);
        return result1 && result2;
    }

    console.log(validate(
        { name: 'zhang', password: '123', phone: '13333333333' },
        { name: 'liu', password: '123456', phone: '13333333333' },
    ));
})();
````
在这个例子中，
- 对于表单校验的每个子项，定义了一组校验策略；  
- 提供一个Context对象解析规则，唯一的调用这些策略；  
- 外部只能通过Context对象，传入对应的配置，去使用这些策略；
- Context对象可由函数代替。

这样的好处显而易见：
- 校验规则的增加只需要在策略对象validator中增加一条策略；
- 修改原有校验逻辑只需要修改配置即可；
- 符合开放封闭原则；
  
缺点也很明显：
- 增加了样板代码的复杂度：这个可以通过将Context对象改为函数解决，并不一定需要持有策略规则，只需要提供一个访问策略的入口；

总结：
1. 使用哪个策略，在以class为主的语言(如Java)，通常使用工厂模式和反射消除if else；在JavaScript中，可以简单的使用对象自变量解决；
2. 单独使用策略模式并不是为了消除if else，更多的是为了封装一组可替换的策略，降低耦合度；策略的增加不影响原有策略，修改原有校验逻辑只需要修改配置。因此策略模式通常和配置结合。
3. 在JavaScript这种函数为一等公民的语言中，不必要执着于用类实现策略模式，只要注意将算法的定义和算法的使用分开即可；
4. 使用场景为：需要封装一系列业务规则，例如表单校验、动画类型等等；

## 参考
- [《JavaScript设计模式与开发实践》](http://product.dangdang.com/23698327.html)
- [策略模式](https://www.cnblogs.com/jiujiduilie/p/9191629.html)