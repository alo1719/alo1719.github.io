---
title: JavaScript学习笔记
date: 2019-09-21 21:06:56
tags: [javascript]
---

## JavaScript Wiki

[JavaScript对象参考手册](http://www.w3school.com.cn/jsref/index.asp)

## JavaScript安放位置

HTML中的脚本必须位于`<script>`与`</script>`标签之间.

脚本可被放置在HTML页面的`<body>`和`<head>`部分中.

那些老旧的实例可能会在`<script>`标签中使用`type="text/javascript"`. 现在已经不必这样做了. JavaScript是所有现代浏览器以及HTML5中的默认脚本语言. 通常的做法是**放入`<head>`部分**中, 或者**放在页面底部**. 这样就可以把它们安置到同一处位置, 不会干扰页面的内容.

### 外链脚本实例

```html
<script src="myScript.js"> </script>
```
<!-- more -->
## JavaScript Syntax

### 折行范例

```js
document.write("Hello \
World!");
```

### 按钮响应

先定义一个按钮元素

```html
<button type="submit" class="btn" id="mybutton1"> 提交 </button>
```

然后在JavaScript中写`onclick`函数

```js
var button = document.getElementById("mybutton1");
button.onclick = function() {
    //要写的东西
}
```

## 联动HTML

JavaScript 能够改变页面中的所有HTML元素/属性, CSS样式, 事件.
修改HTML内容的最简单的方法是使用`innerHTML`属性.

```js
document.getElementById(id).innerHTML = ... ;
```

js可以对`onclick` / `onmouseover`等事件作出反应.

## JavaScript Window

`window.prompt()`: 可以提示用户*输入内容*
`window.alert()`: 弹出警告框

## 常用函数

`isNaN()`: 判断是不是不是数字(NaN = Not a Number)  
`document.writeln()` / `document.write()`: 向HTML写入文本  
`Math.random()`: 从0到1随机生成一个浮点数  
`console.log()`: 向终端输出日志  
`console.info()`: 向终端输出信息  
`Number.isInteger()`: 判断是否是整数  
`String.fromCharCode()`: 从字符编码创建一个字符串

## JavaScript组成部分

- 核心(ECMAScript)——核心语言功能
- 文档对象模型(DOM)——访问和操作网页内容的方法和接口
- 浏览器对象模型(BOM)——与浏览器交互的方法和接口

## JSON

JSON(JavaScript Object Notation)是⼀种轻量级的数据交换格式. 它基于ECMAScript的⼀个⼦集.

### CoffeeScript/TypeScript

CoffeeScript是⼀套JavaScript的转译语⾔, 诞生于2009年, 它增强了JavaScript的简洁性与可读性.

TypeScript是微软开发的开源语言. 它是JavaScript的一个超集, 而且本质上向这个语言添加了可选的**静态类型**和**基于类的面向对象**编程.

## JavaScript应用

- Electron + Node.js + JavaScript 桌面应用
- Ionic + JavaScript 移动应用
- Node.js + JavaScript 网站前后台
- JavaScript + Tessl 硬件

## echarts.js

### 绘制一个简单图表

1. 为ECharts准备一个具备高宽的DOM容器

   ```html
   <div id="main" style="width: 600px;height: 400px;">
   </div>
   ```

2. 通过`echarts.init`初始化一个echarts实例并通过`setOption`方法生成一个简单的柱状图

   ```html
   <script>
        // 基于准备好的dom, 初始化echarts实例
        var myChart = echarts.init(document.getElementById('main'));

        // 指定图表的配置项和数据
        var option = {
            title: {
                text: 'ECharts 入门示例'
            },
            tooltip: {},
            legend: {
                data:['销量']
            },
            xAxis: {
                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [5, 20, 36, 10, 10, 20]
            }]
        };

        // 使用刚指定的配置项和数据显示图表
        myChart.setOption(option);
    </script>
   ```

## Node.js

Node.js是一个能够在服务器端运行JavaScript的开放源代码、跨平台JavaScript 运行环境. 在Node.js出现之前, JavaScript通常作为客户端程序设计语言使用, 以JavaScript写出的程序常在用户的浏览器上运行. Node.js的出现使JavaScript也能用于服务端编程. Node.js含有一系列内置模块, 使得程序可以脱离Apache HTTP Server或IIS, 作为独立服务器运行.

### npm

node.js的包管理程序

国内可以使用国内镜像

```js
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 词法作用域(Lexical Scoping)

JavaScript的函数是基于词法作用域, 而不是动态作用域(Dynamic Scoping).

JavaScript确实有函数作用域. 那意味着定义在函数中的参数和变量**在函数外部是不可见的**, 而在一个函数内部任何位置定义的变量, 在该函数内部任何地方都可见.

很多现代语言都推荐尽可能延迟声明变量. 而用在JavaScript上的话却会成为糟糕的建议, 因为它**缺少块级作用域**. 所以, 最好的做法是在函数体的顶部声明函数中可能用到的所有变量.

## 弱类型

JavaScript是弱类型语言.

## 原型继承

JavaScript中只有一种结构: 对象, 它并没有类这种说法. 每个实例对象都有一个私有属性指向它的构造函数的原型对象(prototype). 该原型对象也有一个自己的原型对象, 层层向上直到一个对象的原型对象为`null`.

JavaScript有一个无类型(class-free)对象系统, 对象直接从其他对象继承属性.

每个对象都连接到一个原型对象, 并且它可以从中继承属性. 所有通过对象字面量创建的对象都连接到`Object.prototype`, 它是JavaScript中的标配对象. 原型链接只有在检索值的时候才会被用到. 如果我们尝试去获取对象的某个属性值, 但该对象没有此属性名, 那么JavaScript会试着从原型对象中获取属性值. 如果那个原型对象也没有该属性, 那么再从它的原型中寻找, 依此类推, 直到该过程最后到达终点`Object.prototype`. 如果想要的属性完全不存在在原型链中, 那么结果就是`undefined`值. 这个过程称为**委托**.

原型关系是一种**动态**的关系. 如果我们添加一个新的属性到原型中, 该属性会*立即对所有基于该原型创建的对象可见*.

### hasOwnProperty方法

如果对象拥有独有的属性, 它将返回true. hasOwnProperty方法不会检查原型链.

### delete

delete运算符可以用来删除对象的属性. 如果对象包含该属性, 那么该属性就会被移除. 他不会触及原型链中任何对象.

## 注释

JavaScript支持`//`和`/* */`注释, 但是后者会出现在正则表达式字面量中, 所以只推荐使用`//`注释.

## 数字类型

JavaScript只有一个数字类型, 内部表示为**64位浮点数**(和Java中的double一样). JavaScript没有分离出整数类型, 所以1和1.0的值相同.

`NaN`是一个**数值**, 表示一个不能产生正常结果的运算结果. NaN不等于任何值, 包括*它自己*. 可以使用`isNaN(number)`检测NaN.

`Infinity`表示所有大于`1.79769313486231570e+308`的值.

## 字符串

和Java一样, Unicode是一个16位的字符集. JavaScript没有字符类型, 要表示一个字符, 只需创建只包含一个字符的字符串即可. 和Java一样, 字符串是不可变的.

## 代码域

当`var`被用在函数内部时, 它定义的是这个函数的私有变量. 不像大多数其他语言, JavaScript中的代码块不会创建新的作用域.

## if

除了`false`, `null`, `undefined`, `''`(空字符串), `0`(数字0), `NaN`(数字NaN)之外, 所有其他值都是真. 值得注意的是, `"false"`(字符串false)也被当作真.

## 简单数据类型和对象

number, string, boolean, null值和undefined值是简单数据类型, 其他所有值*都是对象*. JavaScript中的对象是可变的键控集合(keyed collections). 数组是对象, 函数是对象, 正则表达式也是对象.

对象是属性的容器, 其中每个属性都有名字和值.

## 对象字面量及其检索

一个**对象字面量**就是包围在一对花括号中的零个或多个"名/值"对. 在对象字面量中, 如果属性名是一个合法的JavaScript标识符且不是保留字, 则并不强制要求用引号括住属性名.(例如`first-name`必须用引号括住, 而`first_name`是否括住是可选的, 因为`-`在标识符中是不合法的).

检索可以采用在`[]`中括住一个字符串表达式的方式, 如果字符串表达式是一个字符串字面量, 而且是一个合法的JavaScript标识符且不是保留字, 也可以用`.`代替. 如果尝试检索一个并不存在的成员属性的值, 将返回`undefined`.

## 对象的更新

使用赋值语句, 如果属性名已经存在于对象中, 那么属性值会被替换, 如果没有那个属性名, 那么该属性会被扩充到对象中.

## 参数

当实际参数(arguments)的个数与形式参数(parameters)的个数不匹配时, 不会导致运行时错误. 如果实际参数值多了, 超出的参数值会被忽略. 如果实际参数值过少, 缺失的值会被替换为`undefined`. 对参数值不会进行类型检查: 任何类型的值都可以被传递给任何参数.

## 方法

当一个函数被保存为对象的一个属性时, 我们称它为一个方法. 当一个方法被调用时, `this`被绑定到该对象.

倘若语言设计正确, 那么当内部函数被调用时, `this`应该仍然绑定到外部函数的`this`变量. 这个设计错误的后果就是方法不能利用内部函数来帮助它工作, 因为内部函数的`this`值被绑定了错误的值, 所以不能共享该方法对对象的访问权. 解决方法如下:

```js
// 给myObject增加一个double方法(已有add方法)
myObject.double = function () {
    var that = this;

    var helper = function() {
        that.value = add(that.value, that.value);
    };
    helper();
};

// 以方法的形式调用double
myObject.double();
document.writeln(myObject.value);
```

## new关键字

如果在一个函数前面加上`new`来调用, 那么背地里将会创建一个连接到该函数的`prototype`成员的新对象, 同时`this`也会被绑定到那个新对象上.

一个函数, 如果创建的目的就是希望结合`new`的前缀来调用, 那它就被称为**构造器函数**. 按照约定, 他们保存在以大写字母命名的变量里. 不推荐使用这种形式的构造器函数.

## 闭包

```js
// 创建一个名为quo的构造函数
// 它构造出带有get_status方法和status私有属性的一个对象
var quo = function (status) {
    return {
        get_status: function () {
            return status;
        }
    };
};

// 构造一个quo实例
var myQuo = quo("amazed");
document.writeln(myQuo.get_status());
```

这个`quo`函数被设计成无须在前面加上`new`来使用, 所以没有首字母大写. 即使`quo`已经返回了, 但`get_status`方法仍然享有访问`quo`对象的`status`属性的特权. `get_status`方法并不是访问该参数的一个副本, 它访问的就是该参数本身. 这是可能的, 因为该函数可以访问它被创建时所处的上下文环境. 这被称为闭包.

避免在循环中创建函数, 它可能只会带来无谓的计算, 还会引起混淆. 可以现在循环之外创建一个辅助函数, 让这个辅助函数再返回一个绑定了当前`i`值的函数, 这样就不会导致混淆了.

## 模块

我们可以使用函数和闭包来构造模块. 模块是一个提供接口却隐藏状态与实现的函数或对象. 通过使用函数产生模块, 我们几乎可以完全**摒弃全局变量**的使用, 从而缓解这个JavaScript的**最为糟糕的特性之一**所带来的影响.

模块模式的一般形式是: 一个定义了私有变量和函数的函数; 利用闭包创建可以访问私有变量和函数的特权函数; 最后返回这个特权函数, 或者把他们保存到一个可访问到的地方.

## 级联

让方法返回`this`. 例如:

```js
getElement('myBoxDiv')
    .move(350, 150)
    .width(100)
    .height(100);
```

## JavaScript的继承

### 伪类Pseudoclassical

JavaScript不直接让对象从其他对象继承, 反而插入了一个多余的间接层: 通过构造器函数产生那个对象.

我们可以隐藏一些丑陋的细节, 通过使用method方法来定义一个inherits方法实现:

```js
Function.method('inherits', function (Parent) {
    this.prototype = new Parent();
    return this;
});
```

`inherits`和`method`方法都返回this, 这样允许我们采用级联的形式编程.

```js
var Mammal = function (name) {
    this.name = name;
    this.says = function () {
        return this.saying || '';
    }
}
var Cat = function (name) {
    this.name = name;
    this.saying = 'meow';
}
.inherits(Mammal)
.method('get_name', function () {
    return this.says() + ' ' + this.name + ' ' + this.says();
});
```

现在有了行为像"类"的构造器函数, 但是他们没有私有环境, 所有属性都是公开的. 无法访问super(父类)的方法.

更糟糕的是, 如果在调用构造器函数时忘记了在前面加上new前缀, 那么this不会被绑定到一个新对象上, 而是被绑定到全局对象上. "伪类"形式可以给不熟悉JavaScript的程序员提供便利, 但它也隐藏了该语言的真实的本质. 在基于类的语言中, 类继承是代码重用的唯一方式, 而JavaScript有着更多且更好的选择.

### 原型Prototypal

在一个纯粹的原型模式中, 我们摒弃类, 转而专注于对象. 一旦有了一个想要的对象, 就可以利用`Object.create`方法创造出更多的实例.

```js
var myMammal = {
    name: 'Herb the Mammal',
    get_name: function () {
        return this.name;
    }
};
var myCat = Object.create(myMammal);
myCat.name = 'Henrietta';
```

这是一种差异化继承(differential inheritance). 通过定制一个新的对象, 我们指明它与所基于的基本对象的区别.

### 函数化Functional

以上的继承模式的弱点就是没法保护隐私, 对象的所有属性都是可见的. 应用模块模式可以解决这个问题.

我们从构造一个生成对象的函数开始. 我们以小写字母开头来命名它, 因为它并不需要使用new前缀. 该函数包括4个步骤.

1. 创建一个新对象. 有很多的方式去构造一个对象. 它可以构造一个对象字面量, 或者它可以和new前缀连用去调用一个构造器函数, 或者它可以使用`Object.create`方法去构造一个已经存在的对象的新实例, 或者它可以调用任意一个会返回一个对象的函数.
2. 有选择地定义私有实例变量和方法. 这些就是函数中通过var语句定义的普通变量.
3. 给这个新对象扩充方法. 这些方法拥有特权去访问参数, 以及在第2步中通过var语句定义的变量.
4. 返回那个新对象.

```js
var constructor = funciton (spec, my) {
    var that, 其他的私有实例变量;
    my = my || {};
    // 把共享的变量和函数添加到my中;
    that = 一个新对象;
    // 添加给that的特权方法
    return that;
}
```

spec对象包含构造器需要构造一个新实例的所有信息. spec的内容可能会被复制到私有变量中, 或者被其他函数改变, 或者方法可以在需要的时候访问spec的信息.

my对象是一个为继承链中的构造器提供秘密共享的容器. my对象可以选择性地使用. 如果没有传入一个my对象, 那么会创建一个my对象.

接下来, 声明该对象私有的实例变量和方法. 通过简单地声明变量就可以做到. 构造器的变量和内部函数变成了该实例的私有成员. 内部函数可以访问spec、my、that, 以及其他私有变量.

接下来, 给my对象添加共享的秘密成员. 这是通过赋值语句来实现的, 例如`my.member = value;`

现在, 我们构造了一个新对象并把它赋值给that. 有很多方式可以构造一个新对象. 我们可以使用对象字面量, 可以用new运算符调用一个伪类构造器, 可以在一个原型对象上使用Object.create方法, 或者可以调用另一个函数化的构造器, 传给它一个spec对象(可能就是传递给当前构造器的同一个spec对象)和my对象. my对象允许其他的构造器分享我们放到my中的资料. 其他构造器可能也会把自己可分享的秘密成员放进my对象里, 以便我们的构造器可以利用它.

接下来, 我们扩充that, 加入组成该对象接口的特权方法. 我们可以分配一个新函数成为that的成员方法. 或者, 更安全地, 我们可以先把函数定义为私有方法, 然后再把它们分配给that:

```js
var methodical = function ( ) {
       ...
    };
    that.methodical = methodical;
```

分两步去定义methodical的好处是, 如果其他方法想要调用methodical, 它们可以直接调用methodical()而不是that.methodical(). 如果该实例被破坏或篡改, 甚至that.methodical被替换掉了, 调用methodical的方法同样会继续工作, 因为它们私有的methodical不受该实例被修改的影响.

最后, 我们返回that.

函数化模式还给我们提供了一个处理父类方法的方法. 我们会构造一个`superior`方法, 它取得一个方法名并返回调用那个方法的函数.

```js
Object.method('superior', function (name) {
    var that = this,
        method = that[name];
    return function () {
        return method.apply(that, arguments);
    };
});
```

#### 实例

```js
var mammal = function (spec) {
    var that = {};

    that.get_name = function () {
        return spec.name;
    };
    that.says = function () {
        return spec.saying || '';
    };

    return that;
};

var myMammal = mammal({ name: 'Herb' });
```

现在, name和saying属性是完全私有的. 只有通过get_name和says两个特权方法才可以访问它们.

## 数组

JavaScript没有数组一样的数据结构, 作为替代, JavaScript拥有一些类数组的对象.

### 数组字面量

一个数组字面量是在一对方括号中包围零个或多个用逗号分隔的值的表达式. 数组字面量允许出现在任何表达式可以出现的地方. 数组的第一个值将获得属性名'0', 第二个值将获得属性名'1', 依此类推.

JavaScript允许数组包含任意混合类型的值.

### 长度

和大多数其他语言不同, JavaScript数组的length是**没有上界**的. 如果你用大于或等于当前length的数字作为下标来存储一个元素, 那么length值会被增大以容纳新元素, 不会发生数组越界错误.

length属性的值是这个数组的*最大整数属性名加上1*. 它不一定等于数组里的属性的个数.

你可以直接设置length的值. 设置更大的length不会给数组分配更多的空间. 而把length设小将导致所有下标大于等于新length的属性被删除.

通过把下标指定为一个数组的当前length, 可以附加一个新元素到该数组的尾部(以下行为等价):

```js
numbers[numbers.length] = 'go';
numbers.push('go');
```

### 删除

JavaScript的数组其实就是对象, 所以可以用delete来移除元素, 但是这样会在数组中留下一个空洞.

JavaScript数组有一个splice方法, 第1个参数是数组中的一个序号, 第2个参数是要删除的元素个数. 任何额外的参数会在序号那个点的位置被插入到数组中.

### 混淆点

JavaScript本身对于数组和对象的区别是混乱的. `typeof`运算符报告数组的类型是`object`, 这没有任何意义.

### 多维数组

JavaScript没有多维数组, 但就像大多数类C语言一样, 它支持元素为数组的数组.

## 正则表达式

正则表达式起源于对形式语言(formal language)的数学研究. 在JavaScript中, 正则表达式的语法是对Perl版本的改进和发展, 它非常接近于贝尔实验室(Bell Labs)最初提出的构想. 正则表达式的书写规则出奇地复杂, 在某些位置上的字符串可能解析为运算符, 而仅在位置上稍微不同的相同字符串却可能被当做字面量. 比不易书写更糟糕的是, 这使得正则表达式不仅难以阅读, 而且修改时充满危险.

在JavaScript程序中, 正则表达式必须写在一行中.

### 正则表达式入门

#### 举例

```js
var parse_url = /^(?:([A-Za-z]+):)?(\/{0,3})([0-9.\-A-Za-z]+)(?::(\d+))?(?:\/([^?#]*))?(?:\?([^#]*))?(?:#(.*))?$/
```

`^`表示此字符串的开始.

针对`(?:([A-Za-z]+):)?`,

`(?:...)`表示一个非捕获型分组(noncapturing group). 后缀`?`表示这个分组是可选的(重复0或1次).

`(...)`表示一个捕获型分组(capturing group). 一个捕获型分组会复制它所匹配的文本, 并把其放到数组里. 每个捕获型分组都会被指定一个编号. 第一个捕获型分组的编号是1.

`[...]`表示一个字符类, `A-Za-z`这个字符类包含26个大写字母和26个小写字母. 后缀`+`表示这个字符类会被匹配一次或多次.

针对`(\/{0,3})`,

`\/`表示应该匹配`/`, 它用`\`来进行转义, 这样它就不会被错误的解释为这个正则表达式的结束符. 后缀`{0,3}`表示`/`会被匹配0次, 或者1-3次.

针对`([0-9.\-A-Za-z]+)`,

它会匹配一个主机名, 由一个或多个数字、字母, 以及.或-组成. `-`会被转义为`\-`以防止与表示范围的连字符混淆.

针对`(?::(\d+))?`,

又是一个可选的分组, 以一个`:`加上一个或多个数字而组成的序列. `\d`表示一个数字字符.

针对`(?:\/([^?#]*))?`,

另一个可选的分组, 以一个`/`开始, 之后的字符类`[^?#]`表示除了`?`和`#`之外的所有字符. `*`表示这个字符类会被匹配0次或者多次.

针对`(?:\?([^#]*))?`,

以`?`开始, 包含0个或多个非`#`字符.

针对`(?:#(.*))?`,

以`#`开始, `.`会匹配除行结束符以外的所有字符.

`$`表示这个字符串的结束.

#### 标识

i标识: `//i`表示匹配字母时忽略大小写.
g标识: 全局的, 匹配多次.
m标识: 多行, `^`和`$`能匹配行结束符.

#### 结构

正则表达式字面量被包围在一对斜杠中.

#### 分支

```js
"into".match(/in/int/)
```

会在into中匹配in, 但是不会匹配int, 因为in已被成功匹配了.

#### 转义

`\n`是换行符, `\r`是回车符, `\t`是制表符, 并且`\u`允许指定一个Unicode字符来表示一个十六进制的常量.

`\d`等同于`[0-9]`, `\D`则表示相反的`[^0-9]`. `\w`等同于[0-9A-Za-z].

`\1`指向分组1所捕获到的文本的一个引用, `\2`指向分组2的引用, 以此类推.

#### 分组

一个捕获型分组是一个被包围在圆括号中的正则表达式分支.

非捕获型分组有一个`(?:`前缀, 仅作简单的匹配, 并不会捕获所匹配的文本. 不会干扰捕获型分组的编号.

#### 量词

?等同于{0,1}, *等同于{0,}, +等同于{1,}

## 方法Methods

### Array

`array.concat(item...)`  
`array.join(separator)`  
`array.pop()`  
`array.push(item...)`  
`array.reverse()`  
`array.shift()`  
`array.slice(start, end)`  
`array.sort(comparefn)`默认比较函数把要排序的元素视为字符串.  
`array.splice(start, deleteCount, item...)`  
`array.unshift(item...)`  

`regexp.exec(string)`如果成功匹配regexp和字符串string, 会返回一个数组. 数组中下标为0的元素将包含正则表达式regexp匹配的字符串, 下标为1的元素是分组1捕获的文本, 下标为2的元素是分组2捕获的文本, 以此类推. 匹配失败会返回null.  
`regexp.test(string)`如果该regexp匹配string, 返回true, 否则返回false.  

`string.indexOf(searchString, position)`  
`string.replace(searchValue, replaceValue)`如果searchValue是一个字符串, searchValue只会在第1次出现的地方被替换. 如果searchValue是一个正则表达式并且带有g标识, 会替换所有的匹配.  
`string.slice(start, end)`没有任何理由去使用substring方法, 请使用slice替代他.  

## JavaScript的代码风格

JavaScript的弱类型和过度的容错性导致程序质量无法在编译时获得保障，所以为了弥补，我们应该按照严格的规范进行编码。

当我现在评审一门语言的特性的时候，我把注意力放在那些有时很有用但偶尔很危险的特性上。那些是最糟糕的部分，因为我们很难辨别它们是否被正确使用。那是bug的藏身之地。

## JavaScript优美的特性

函数是顶级对象: 函数是有词法作用域的闭包.  
基于原型继承的动态对象: 对象是无类别的。我们可以通过普通的赋值给任何对象增加一个新成员属性。一个对象可以从另一个对象继承成员属性。  
对象字面量和数组字面量: 这对创建新的对象和数组来说是一种非常方便的表示法。JavaScript字面量是数据交换格式JSON的灵感之源。

## JavaScript毒瘤

### 全局变量

全局变量就是在所有作用域中都可见的变量。全局变量在微型程序中可能会带来方便，但随着程序变得越来越大，它们很快变得难以管理。因为一个全局变量可以被程序的任何部分在任意时间修改，它们使得程序的行为变得极度复杂。在程序中使用全局变量降低了程序的可靠性。JavaScript的问题不仅在于它允许使用全局变量，而且在于它依赖全局变量。JavaScript没有链接器（linker），所有的编译单元都载入一个公共全局对象中。

### 作用域

在所有其他类似C语言风格的语言里，一个代码块（括在一对花括号中的一组语句）会创造一个作用域。代码块中声明的变量在其外部是不可见的。JavaScript采用了这样的块语法，却没有提供块级作用域：代码块中声明的变量在包含此代码块的函数的任何位置都是可见的。

在大多数语言中，一般来说，声明变量的最好的地方是在第一次用到它的地方。但这种做法在JavaScript里反而是一个坏习惯，因为它没有块级作用域。更好的方式是在每个函数的开头部分声明所有变量。

### 自动插入分号

### 保留字

JavaScript保留的单词大多数并没有在语言中使用.

### parseInt

parseInt如果字符串第1个字符是0, 那么该字符串会基于八进制而不是十进制来求值。在八进制中，8和9不是数字，所以parseInt("08")和parseInt("09")都产生0作为结果。这个错误会导致程序解析日期和时间时出现问题。幸运的是，parseInt可以接受一个基数作为参数，如此一来parseInt("08", 10)结果为8。我建议你总是加上这个基数参数。

### 浮点数

进制的浮点数不能正确地处理十进制的小数，因此0.1+0.2不等于0.3。这是JavaScript中最经常被报告的bug，并且它是遵循二进制浮点数算术标准而有意导致的结果。这个标准对很多应用都是适合的，但它违背了大多数你在中学所学过的关于数字的知识。幸运的是，浮点数中的整数运算是精确的，所以小数表现出来的错误可以通过指定精度来避免。

### NaN

typeof不能辨别数字和NaN, 而且NaN也不等同于它自己. 所以, `NaN === NaN`的结果是false. JavaScript提供了一个isNaN函数, 可以辨别数字与NaN.

### ===和!==

===和!==会按照期望的方式工作, 如果两个运算数类型一致且拥有相同的值，那么===返回true，!==返回false. 而==和!=只有在两个运算数类型一致时才会做出正确的判断，如果两个运算数是不同的类型，它们试图去强制转换值的类型。转换的规则复杂且难以记忆。
