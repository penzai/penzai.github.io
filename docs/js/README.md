#### `Object.defineProperty(o, p, descriptor)`

定义对象 o 的属性 p，并返回对象 o。

##### descriptor

- 公共描述符
  - `configurable`(false)能否配置以及被删除（具体影响见下文）
  - `enumerable`(false) 能否枚举
- 择其一
  - data descriptor
    - `value`(undefined)
    - `writable`(false) 该值是否能够被更改（只设置此值只能限制赋值运算符更改，想完全限制需要结合 configurable 属性）
  - accessor descriptor
    - `get`(undefined)
    - `set`(undefined)

##### `configurable`

配置为 false 时，即：

1. 不能删除属性 p
2. 不能重新配置属性 p，但有例外：
   1. writable 为 true 时，配置 value
   2. writable 为 true 时，配置 writable
3. 不能数据描述符与访问器描述符之间互转

> 所谓不能重新配置，并不是不能再次调用，只要这次调用的参数值与 configurable 为 false 时当前的 descriptor 的属性不冲突，依然可以调用。实际上就是一个徒劳的举动。

2.2 的来历：

> There is one exception to this rule—JavaScript allows you to change an unconfigurable property from writable to read-only, for historic reasons; the property length of arrays has always been writable and unconfigurable. Without this exception, you wouldn’t be able to freeze (see Freezing) arrays.

##### 与赋值运算符=的区别

区别：

1. 原对象 o 有属性 p。赋值运算符会被访问器属性中的 setter 和数据属性中的 writable:false 控制，就此结束。而定义是根据一定的规则重新配置原属性。
2. 原对象 o 的原型有属性 p。如果将要操作的属性 p 是来自于原型，那么赋值会被 setter 和 writable 影响，而定义不会（就算原型的 p 属性被设置为 configurable:false, writable:false 也不会）。
3. 原对象 o 无属性 p。执行新增操作，赋值运算符虽然是调用[[Put]]进行操作，但是最后两者都会殊途同归调用[[DefineOwnProperty]]，区别在于默认值的设置。前者全为 true，后者全为 false。

共同：

1. 创建新的值时都会检测对象 o 是否是非扩展对象。
   > 扩展对象，检查用 Object.isExtensible()，设置用 Object.preventExtensions()，注意与冻结的区别。

#### `Object.is`

与`===`区别：

- `+0` 与 `-0`，`===`返回 true，Object.is 返回 false
- `NaN`与`NaN`，`===`返回 false，Object.is 返回 true

#### `Object.freeze()`

- 冻结对象，只是首层属性的冻结。
- 冻结过后，修改数据并不会报错，只是会无效。
- 当继承一个冻结对象时，对冻结对象里的属性操作是无效的（正常情况下的赋值会添加一个自身属性）。

#### 对象遍历

古老的遍历方式，自然不能遍历出 symbol 属性。

- for in。遍历自身和原型链上的可枚举属性。
  > 在继承关系中，手动设置的 constructor 会被遍历出来，引擎自己设置的不会被遍历出来。因此记得配置 enumerable 为 false。

以下是新的属性遍历方式，都有一个特点，那就是不再遍历原型链上的属性，这样更符合逻辑。

其中 Object 上分门别类了各种遍历。

- Object.keys()、Object.values()、Object.entries()。自身的可枚举属性。
- Object.getOwnPropertyNames()。自身的可枚举与不可枚举属性。
- Object.getOwnPropertySymbols()。自身的 symbol 属性。

> symbol 属性的属性描述符为：不可枚举，不可配置，不可重写。

最后在内置对象 Reflect 上挂载了一个综合的方法，当然，依然不可遍历原型链的属性。

- Reflect.ownKeys()。自身的可枚举、不可枚举、symbol 属性。

其他遍历方法：

- for of，用于遍历可迭代对象。

#### 遍历顺序

js 引擎本身是散乱无序的，浏览器实现时加入了自己的顺序规则。

- 首先遍历所有数值键，按照数值升序排列。
- 其次遍历所有字符串键，按照加入时间升序排列。
- 最后遍历所有 Symbol 键，按照加入时间升序排列。

### Array

- 数组是特殊的对象。
- shift/unshift 比 pop/push 性能差很多，尤其是数组很大的时候尤为明显。
- 具有破坏的方法：sort、reverse、splice/fill/pop/push/shift/unshift。

### reduce

`arr.reduce(callback(accumulator, currentValue[, index[, array]])[, initialValue])`

注意第 2 个参数为可选参数，不传时默认为数组第 0 个元素。

### sort

默认按照升序 ascending 顺序就地（in place）排列，会将元素转换为字符串，再按照 utf-16 编码单元值排列。

不稳定的，但是 ES10 规定之后稳定。

callback 方法中第一个参数排前面就返回负数，第二个参数排前面就返回正数。也可以理解为需要传入的两个值交换位置时就需要返回一个正数，相等就需要返回 0。

### `Array.isArray()`

该 api 判断对象是否是 Array 类的实例或者是以 class 方式继承的子类实例，自行使用组合寄生方法继承的实例，不在范围之内。

```js
// Array.isArray(arr) === true
class MyArray extends Array {}

// Array.isArray(arr) === false
function MyArray() {
  Array.apply(this, arguments);
}
MyArray.prototype = Object.create(Array.prototype);
```

### 类型转换

基本类型中只有 Number、String、Boolean、Symbol 存在构造函数，
有几条规则注意：

- Number -> Boolean，0/+0/-0/NaN 为 false，其余为 true 包括+Infinity/-Infinity。
- String -> Boolean, 只有''为 false。
- Symbol -> Number/String，报错
- Object -> Number/String，拆箱转换，调用 valueOf、toString 方法，遇到基本类型就停止，最后无基本类型就报错，有了基本类型再转换位 Number/String。
  > 转成数字时顺序为 valueOf 再 toString，转成字符串时相反。注意，`n + ''`与`n + 1`都是属于转数字优先，调用顺序为 valueOf 再 toString。

### 类型判断

- typeof 操作符： 简单判断类型，无法区分例如数组与对象、null 与对象。
- instanceof 操作符/isPrototypeOf()： 测试实例与原型链中**出现过**的构造函数。
  > 当构造函数返回的是引用类型时，无法判断。
- Object.prototype.toString.call(obj)。这个方法返回该值的 class 类，即`[[class]]`私有属性。在早期，由于该属性是不能更改的，所以这样判断很安全。但是，es6 后该属性被`Symbol.toStringTag`所代替，变得可以被更改，如下：

```javascript
var o = {
  [Symbol.toStringTag]: "HelloKitty",
};

console.log(Object.prototype.toString.call(o)); // "[object HelloKitty]"
```

> 当变量为装箱类型时，需要再加上 typeof 判断是否是基本类型。

- obj.hasOwnProperty(key) 判断 obj 实例有没有 key 属性

## 可迭代对象

实现了**可迭代协议**的对象就是可迭代对象。即实现了`[Symbol.iterator]`接口的对象。

### 迭代器协议

可迭代协议是一个函数，该函数在准备迭代时会进行调用，用返回的迭代器，进行每一次迭代调用 next 方法获取每一次得到的值。

也就是说`[Symbol.iterator]`属性要么是一个自行返回如下迭代器的函数，要么是一个 generator 函数。

```ts
type IteratorReturn = {
  next(): {
    done: boolean;
    value?: any;
  };
};
function iterator(): IteratorReturn;
```

### 内置可迭代对象

String、Array、TypedArray、Map、Set、arguments

### 接受可迭代对象的内置 api

- new Map([iterable])
- new WeakMap([iterable])
- new Set([iterable])
- new WeakSet([iterable])
- Promise.all(iterable)
- Promise.race(iterable)
- Array.from(iterable)

### 需要可迭代对象的语法

for...of 循环、展开语法、yield \*、解构赋值

## Map

保存键值对。

- 能够记住键的原始插入顺序
- 任何值都可以作为键值
- 是 iterable 的，可以直接被迭代
- 在频繁增删键值对的场景下表现更好

## WeakMap

与 map 的区别：

- 只能使用引用类型作为 key
- key 保存的是弱引用，会直接被 gc 回收，因此不能遍历 WeakMap 的 key

在 shim 中被用于类的私有变量`#`的实现。

```javascript
class Foo {
  constructor(p) {
    this.#p = p
  },

  fn() {
    console.log(this.#p)
  }
}

// 转换后代码
var __classPrivateFieldSet = (receiver, privateMap, value) => privateMap.set(receiver, value);
var __classPrivateFieldGet = (receiver, privateMap) => privateMap.get(receiver);

var _p;
class Foo {
  constructor(p) {
    _p.set(this, void 0)
    __classPrivateFieldSet(this, _p, p)
  }

  fn() {
    console.log(__classPrivateFieldGet(this, _p))
  }
}
_p = new WeakMap();
```

> 在 ts 中，#私有变量与 private 的区别在于，private 只是 ts 内定的，其实转换后代码无变化，在继承中相同的 private 变量会被覆盖，而#私有变量跟**当前类**跟**实例**挂钩，不会被覆盖，始终是取的当前类中的变量。

## 运算符优先级

这里只列出易混顺序

- 成员访问类：
  - `.`访问
  - `[]`访问
  - `new F()`**new 运算带参数列表**
  - `F()`函数调用
  - `?.` 可选链
- `new F`**new 运算无参数列表**
- `&&`
- `||`
- 三元运算符`? :`
- 赋值类运算符`=`

## 隐式转换

### 运算符隐式转换

#### 减`-`乘`*`除`/`

尽量都转 number，不行就返回`NaN`

- 转换规则，null -> 0，true -> 1，false -> 0，undefined -> NaN，数组 -> string -> number -> NaN（一步步尝试），对象 -> NaN，'' -> 0
- NaN, Infinity 都是 number 类型

```javascript
1 - true; // 0
1 - null; // 1
1 * undefined; // NaN
2 * ["5"]; // 10
2 * ["5", "5"]; // NaN
3 / ""; // Infinity
3 / []; // Infinity
```

> Symbol 会报错，毕竟这种隐式转换是以前的，Symbol 是新出的原始类型。

#### 加`+`

优先级如下：

1. 有字符串出现，全都转成 string 拼接
2. number + 原始类型（其实也就只剩下 null/true/false/undefined），正常数学运算
3. 含有引用类型，转成 string 拼接

#### 等于符`==`

两条特例规则：

- `NaN`与任何类型比较，返回 false
- `null`，`undefined`只有互相比较时才返回 true，其余情况均返回 false

转换规则优先级 ：
`boolean` > `引用类型` > `string` > `number`

一步步按照优先级进行转换，最终都会向 number 靠齐，当然途中如果已经是同类型了，那么自然无需再进行转换，直接比较即可，其中：

- `boolean`被转换成 number
- 引用类型会使用 ToPrimitive 规则（valueOf/toString 方法）转换成原始类型，再进行优先级转换

> 对于数组，`[]`转为数字`0`，`[1]`转为数字`1`，`[1,2]`转为数字`NaN`。


## 语句

### 普通语句

- 声明类语句（var/const/let/function/class）
- 表达式语句
- 空语句
- with 语句
- debugger 语句

### 语句块

### 控制型语句

if/switch/for/while/continue/break/return/throw/try

在各种控制语句组合中，有如下情况：

- 产生“消费的”：
  - switch + break
  - for/while + break
  - for/while + continue
  - function + return
  - try + throw
- 产生报错的
  - function + break
  - function + continue
- 特殊处理的，try/catch/finally + break/continue/return
  > 这里的特殊处理指的是，try/catch 的非 normal 型完成记录并不会立即返回，而是还要加上 finally 的完成记录一起判断。
- 其余均穿透

### 带标签的语句

形如`hello: var a = 3`，冒号前面的字符串用于设定“完成记录类型 Completion Record”的[[target]]字段，主要用于以指定方式跳出多重循环。

例如`break hello`会产生带有[[target]]为 hello 的完成记录，那么只有带有 hello 的标签才能“消费”它。

## switch

- 用全等比对 switch 传入的表达式值与 case 后面的表达式值，true 就执行 case 下面的代码，false 则找寻 default 代码。
- 一旦代码开始执行（包括 case 的代码或者 default 的代码），如果没有碰到 break，不管后面的 case 条件，一律直接执行。

## `bind`方法

- bind 会产生一个**新方法**，方法名为`bound 原函数名`
- `bind`传递的参数优先级比原函数高，且后来的参数是紧跟着`bind`参数的后面，而不是对位取代
- bind 返回的函数可以通过 new 调用，这时提供的 this 的参数被忽略，指向了 new 生成的全新对象。内部模拟实现了 new 操作符

### 手写 bind

1. 返回`bound`方法
2. 通过`this instanceof bound`来决定是否模拟 new 操作符
3. 调整返回方法的`name`与参数的`length`

## Class

### 定义

类（Class）为 JavaScript 原型的语法糖。

- 原型的构造函数，这里直接使用`class`关键字声明。
- 原型的继承，之前使用手动设置 prototype，这里使用`extends`关键字。

在 react 中经常会用到=符号设置属性，转化为 es5 的过程稍有区别，=符号设置的实例属性是直接挂载在生成的实例属性上，而构造函数里的属性则是挂载在类的 prototype 对象上。类的属性设置区别不大。

```javascript
class Cat {
  constructor(name) {
    this.name = name;
  }

  fn = () => {
    console.log('from "=" setting', this);
  };

  fn2 = function () {
    console.log(this);
  };
  name = 3;

  fn() {
    console.log('from "()" setting', this);
  }

  static s = 1;
  static sfn() {
    console.log('from "()" setting', this);
  }
  static sfn = () => {
    console.log('from "=" setting', this);
  };
}
```

转化后

```javascript
function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true,
    });
  } else {
    obj[key] = value;
  }
  return obj;
}

var Cat = /*#__PURE__*/ (function () {
  "use strict";

  function Cat(name) {
    var _this = this;

    _defineProperty(this, "fn", function () {
      console.log('from "=" setting', _this);
    });

    _defineProperty(this, "fn2", function () {
      console.log(this);
    });

    _defineProperty(this, "name", 3);

    this.name = name;
  }

  var _proto = Cat.prototype;

  _proto.fn = function fn() {
    console.log('from "()" setting', this);
  };

  Cat.sfn = function sfn() {
    console.log('from "()" setting', this);
  };

  return Cat;
})();

_defineProperty(Cat, "s", 1);

_defineProperty(Cat, "sfn", function () {
  console.log('from "=" setting', Cat);
});
```

### 继承

最好的继承的特点：

- 能给父类构造函数传参
- 函数复用，即父类子类原型上的方法要能在新的对象上复用
- constructor 能正确指向

#### 原型链继承

使用 new 继承，导致了超类的实例属性变成了子类的原型属性。

```javascript
function SubType() {}
SubType.prototype = new SuperType();
```

#### 借用构造函数继承（constructor stealing）

使用 call/apply，导致了不能复用父类的方法。

```javascript
function SubType() {
  SuperType.call(this);
}
```

#### 组合继承（conbination inheritance）

结合原型链继承与借用构造函数继承。

#### 原型式继承（prototypal inheritance）

即 es5 的 Object.create()方法。需要一个基础对象，来创建另一个类似的对象。

这种继承实质上是对类继承的一种补充，属于**对象之间的继承**，而无需去创建抽象类。

```javascript
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}
```

#### 寄生式继承（parasitic inheritance）

与原型式继承一样，只是说把设置方法的过程单独提了出来，但本质上“方法”还是没有得到复用，挂在了实例上。

```javascript
function createAnother(original) {
  var clone = Object.create(original);
  clone.sayHi = function () {
    alert("hi");
  };
  return clone;
}
```

#### 寄生组合式继承

属于类之间的继承，在原型对象的继承上使用上面讨论的结果，即寄生式。在实例属性的继承上，使用“借用构造函数继承”方式。

```javascript
SubType.prototype = object(SuperType.prototype);
Object.defineProperty(SubType.prototype, "constructor", {
  value: SubType,
});
```

> 类的静态属性继承：`Object.setPrototypeOf(SubType, SuperType)`

### 原型

原型，即`[[prototype]]`。目前最新的操作原型的方式如下，在早期我们只能使用 new 一个要继承的实例来链接原型（**proto**在个浏览器中也没普及开来）。

- Object.create，根据指定对象为原型创建新的对象。
- Object.getPrototypeOf，获取原型
- Object.setPrototypeOf，设置原型

要产生一个实例 v，需要：

1. 能设置 v 的自有属性（要是能按一定逻辑设置就更好了）
2. 能设置 v 的公有属性

js 用了以下东西来操作：

1. 用一个函数 f 来设置自有属性，函数拥有逻辑，更贴切
2. 一个对象 p，包含所有类 v 实例的公有属性

为了更加紧密、规范：

1. 函数采用大写命名 F，别名构造函数
2. 函数天生拥有一个属性`prototype`，来存储对象 p

所以任何一个实例 v 就可以由一个构造函数 F 生成。

生成后的 v 怎么与构造函数关联呢。

想当然的是 v 的一个属性指向构造函数 F，问题有两：

1. 该关联属性应该有所隐藏，不能算作 v 实际的东西，而且要能避免属性名冲突。
2. 假如 v 与 F 已经建立了关系，那么要使用公有属性，需要找到 F，然后调用 F.prototype.fn，不符合使用体验。

js 采用了以下设置：

1. 变量 v 拥有一个隐式原型`[[prototype]]`，各个浏览器实现为`__proto__`属性来进行访问。
2. `__proto__`直接指向公有属性 p，而不是构造函数 F，而出于又要能与构造函数 F 关联，所以把 F 存在了公有属性 p 的属性`constructor`中。

至此，一切关联都理所当然。

#### 解决 Function

构造函数 F 又是谁生成的呢？按照推论到底，是 Function 函数，那么 Function 函数又是由哪个构造函数生成的呢。

函数 Function 自己创造了自己，这条规则却不适用于 Object.prototype

- ✅`Function.__proto__.constructor === Function`
- ✅`Function.__proto__ === Function.prototype`
- ❌`Object.prototype.__proto__ === Object.prototype`

#### 解决 Object.prorotype

公有属性 p，也就是原型 prototype 推论到底，是由 Object 构造函数生成的，Object 在上述第一个问题中产生，可是 Object.prototype 又是怎么产生的。

null 不借构造函数就创造了 Object.prototype，即：

- ✅`Object.prototype.__proto__ === null`

那么自然 Object.prototype 的构造函数也没有。

![http://www.mollypages.org/tutorials/js.mp](~images/js/prototype.jpg)

### new

1. 创建空对象
2. 链接原型链
3. 执行构造函数
4. 根据步骤 3 的结果类型返回不同的值

```javascript
var _new = function () {
  var Constructor = [].shift.call(arguments);
  if (typeof Constructor !== "function") {
    throw new Error("请传入正确的构造函数");
  }
  var instance = Object.create(Constructor.prototype);
  var ret = Constructor.apply(instance, arguments);
  return ret instanceof Object ? ret : instance;
};
```

## eval
### 调用方式
eval的运行时很低效。且不安全。可使用Function代替相关逻辑。
- `eval()`，当前作用域
- `eval?.()`，全局作用域

## 异步编程

### Promise

promise 的 then，如果在执行 promise 后，该 promise 为 resolved 状态，那么 then 会被马上进入微任务队列。

#### Promise.resolve()

灵活运用转 Promise 对象的接口 Promise.resolve()。

参数处理情况如下：

- Promise 对象，直接返回。
- thenable 对象，该对象 then 方法里必须使用 resolve 或者 reject，否则 Promise.resolve(thenable 对象)一直是 pending 状态

```javascript
const obj = {
  then: () => {},
};

Promise.resolve(obj);
// 等价于
new Promise(obj.then);
```

- 其他值。

```javascript
const obj = {};
Promise.resolve(obj);
// 等价于
new Promise((resolve) => resolve(obj));
```

- 不传值。等价于 Promise(undefined)

#### 利用`runner`函数进行带逻辑的异步请求

```javascript
function runner(_gen) {
  return new Promise((resolve, reject) => {
    var gen = _gen();

    _next();
    function _next(_last_res) {
      var res = gen.next(_last_res);

      if (!res.done) {
        var obj = res.value;

        if (obj.then) {
          obj.then(
            (res) => {
              _next(res);
            },
            (err) => {
              reject(err);
            }
          );
        } else if (typeof obj == "function") {
          if (
            obj.constructor
              .toString()
              .startsWith("function GeneratorFunction()")
          ) {
            runner(obj).then((res) => _next(res), reject);
          } else {
            _next(obj());
          }
        } else {
          _next(obj);
        }
      } else {
        resolve(res.value);
      }
    }
  });
}

runner(function* () {
  let userData = yield $.ajax({ url: "getUserData", dataType: "json" });

  if (userData.type == "VIP") {
    let items = yield $.ajax({ url: "getVIPItems", dataType: "json" });
  } else {
    let items = yield $.ajax({ url: "getItems", dataType: "json" });
  }

  //生成、...
});
```

> 与`async/await`区别
>
> - 写法一样，还不需要`runner`函数
> - 还可以写成箭头函数。

### Generator 函数

- 一个状态机，内部封装了很多种状态。
- 此函数会返回一个遍历器对象

可用结构操作符`...`遍历。

```javascript
function* initializer(count, mapFunc = (i) => i) {
  for (let i = 0; i < count; i++) {
    const value = mapFunc(i, count);
    if (mapFunc.constructor.name === "GeneratorFunction") {
      yield* value;
    } else {
      yield value;
    }
  }
}

// 用于生成52张扑克牌
const cards = [
  ...initializer(13, function* (i) {
    let p = i + 1;
    if (p === 1) {
      p = "A";
    }
    if (p === 11) {
      p = "J";
    }
    if (p === 12) {
      p = "Q";
    }
    if (p === 13) {
      p = "K";
    }
    yield `♠${p}`;
    yield `♣️${p}`;
    yield `♥️${p}`;
    yield `♦️${p}`;
  }),
];
```

### async/await

使用 try/catch 捕获 await 的错误，如果不这样，那么报错后面的代码逻辑将不会执行。另外，catch 中抛出错误，代码也会停止。

```javascaript
// 注意区分
const a = yield 3; // a: undefined
const b = await 3; // b: 3
```

## `Iterator`

默认的 Iterator 接口部署在数据结构的 Symbol.iterator 属性上。换言之，只要一个数据实现了这个属性，他就可以使用 for...of 进行遍历。此属性值必须为一个**遍历器对象**（含有 next 方法的对象）。

```javascript
let flag = false;
const obj = {
  [Symbol.iterator]() {
    return {
      next() {
        if (flag) {
          return {
            value: 3,
            done: true,
          };
        } else {
          flag = true;
          return {
            value: 3,
            done: false,
          };
        }
      },
    };
  },
};
for (let k of obj) {
  console.log(k);
}
// 3
```

## 正则表达式

正则要么匹配字符，要么匹配位置。

### 提示要点

- 字符集`[]`中的特殊字符无需转义符`\`（转义也可以）。
- `\1` `\2`代表括号匹配结果，可在匹配的时候使用，例如：`/(e)n\1/`可以匹配`ene`字符。

### 位置

`\b` `\B` `^` `$` `(?=p)` 都表示位置，位置都不参与实际匹配结果。

#### `(?=p)`：

表示 p 前面的位置。反义是`(?!p)`。反向是`(?<=p)`，表示 p 后面的位置。反义是`(?<!p)`。

p 本身并不是匹配结果，它只是匹配的条件。例如你要匹配的字符是 xxx。只能使用如下几种组合方式：
- `xxx(?=p)` positive lookahead
- `xxx(?!p)` negative lookahead
- `(?<=p)xxx` positive lookbehind
- `(?<!p)xxx` negative lookbehind

### 贪婪模式

默认匹配是贪婪匹配，可以在量词`*`或者`{m,n}`后面加上`?`符号，使其惰性匹配。

### 分组

匹配下面一个字符

- 在代码里是字符串模式
- 里面有双斜杠`//`

那么规则为

```
/(?<!\\)('|"|`).*?\/\/.*?(?<!\\)\1/mg
```

命名捕获数组可在replace方法中使用。语法为：`(?<name>pattern)`

### 计算匹配结果

- `match`字符串的方法，在没 g 的情况下跟 exec 类似（结果数组为[匹配结果，括号结果1，括号结果2...]）。但是正则表达式带 g，那么就只返回匹配结果的数组，**不含括号捕获值**。
- `exec`正则表达式的方法，返回数组，只返回第一次的匹配，数组位置 0 为匹配字符，后续为括号捕获值，如果正则表达式是含 g 的全局匹配，那么可以通过再次执行 exec 方法来获取新的匹配结果，直到结果为 null。
- `test`正则表达式的方法，返回 Boolean 值。

## `ajax`

#### 原生写法

```javascript
if (window.XMLHttpRequest) {　 // Mozilla, Safari...
  xhr = new XMLHttpRequest();
} else if (window.ActiveXObject) { // IE
  try {
    xhr = new ActiveXObject('Msxml2.XMLHTTP');
  } catch (e) {
    try {
      xhr = new ActiveXObject('Microsoft.XMLHTTP');
    } catch (e) { }
  }
}
if (xhr) {
  xhr.onreadystatechange = onReadyStateChange;
  xhr.open('get', 'http://localhost:3000/user', true);
  // 设置 Content-Type 为 application/x-www-form-urlencoded
  // 以表单的形式传递数据
  xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
  xhr.send('username=admin&password=root');
}

// onreadystatechange 方法
function onReadyStateChange() {
  // 该函数会被调用四次
  console.log(xhr.readyState);
  if (xhr.readyState === 4) {
    // everything is good, the response is received
    if (xhr.status === 200) {
      console.log(JSON.parse(xhr.responseText);
    } else {
      console.log('There was a problem with the request.');
    }
  } else {
    // still not ready
    console.log('still not ready...');
  }
}
```

#### `fetch`

基于 Promise 设计，基础用法如下。

```javascript
fetch(url, {
  method: 'post',
  body: JSON.stringify({
    a: 1,
    b: 2
  }),
  // body: 'a=1&b=2',
  headers: {
    'Content-Type': 'application/json'
    // 'Content-Type': 'application/x-www-form-urlencoded;charset=utf-8'
  },
  // 是否带cookie
  // same-origin: 同源带 include: 始终带 omit: 不带
  credentials: 'omit' //默认
})
  .then(
    response => {
      // 在这里进行中间件操作，封装
      if (response.ok) {
        return response.json()
      } else {
        // throw new Error('fetch done. but something is wrong!')
        return Promise.reject({
          status: response.status,
          statusText: response.statusText
        })
      }
    },
    error => {
      // 请求本身没成功，比如url地址错误
      console.log('fetch is error', error) })
    }
  )
  .then(data => {
    // 处理最终结果
  })
  .catch(error => {
    // 中间件捕获的错误
  })
```

 有几个坑

- `cookie` 传递
- `404`、`503` 等错误不被认为是网络错误，是不会抛错的。在中间件里用 `reponse.ok` 与 `response.status` 来确定处理

### 连等赋值

一道”面试“题

```javascript
var a = { n: 1 };
a.x = a = { n: 2 };
console.log(a.x); // ???
```

- 连等赋值会提前保存对象的引用，复制时右边取新的，左边取提前保存的。
- A=B=C 的调用过程其实是 B=C，再执行 A=B，注意 C 值只调用一次（可用 getter 进行测试）

所以上述答案为`undefined`

## Array

### Array.prototype.map

内部原理：

- 提前预知并确定循环长度
- 执行时
  - 元素为 empty，不进行方法调用，并在返回值中设为 empty
  - 元素不为 empty，获取此刻原数组的元素值，执行方法调用

> 使用 map 方法处理数组时，数组元素的范围是在 callback 方法第一次调用之前就已经确定了。在 map 方法执行的过程中：原数组中新增加的元素将不会被 callback 访问到；若已经存在的元素被改变或删除了，则它们的传递到 callback 的值是 map 方法遍历到它们的那一时刻的值；而被删除的元素将不会被访问到。 —— MDN https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map

> 边循环边更改原数组时，有两个特点。一是循环长度只减不增，二是循环值为更改后原数组按 index 取值。

### Array.prototype.reduce

传入函数的 a，b 值并不是数组的每一项，准确的讲，分别为累加器与当前项，有 2 种情况：

- 设定了初始值，第 1 次循环，累加器值为初始值
- 未设定初始值，第 1 次循环，累加器值为第 1 项，且从数组第 2 项开始循环

综上，应避免以下错误：

```javascript
// ❌ 报错
const arr = []; //一定要确定数组不为空
arr.reduce((a, b) => a + b);

// ❌ 结果为NaN
const arr = [{ price: 1 }, { price: 2 }];
arr.reduce((a, b) => a.price + b.price, 0);
```

### empty

使用`delete arr[i]`，原数组的下标并不会改变，而是在原来的下边出，把值设为 empty，此值有一定的特殊性。

```javascript
const arr = [1, 2, 3, 4, 5];
arr[1] = undefined;
delete arr[3];
console.log("changed : ", arr);

// for (不跳过任意)
for (let i = 0; i < arr.length; i++) {
  console.log(`for ${arr[i]}`);
}

// map（跳过empty，返回结果包含empty）
const mapedArr = arr.map((v) => {
  console.log(`map ${v}`);
  return "hello";
});
console.log(`mapedArr ${mapedArr}`, mapedArr);
// filter（跳过empty，返回结果不包含empty）
arr.forEach((v) => console.log(`forEach ${v}`));
const filteredArr = arr.filter((v) => {
  console.log(`filter ${v}`);
  return true;
});
console.log(`filteredArr ${filteredArr}`, filteredArr);
```

### async/await

注意 for 循环（循环之间会等待）与 forEach（不会等待）调用的不同。

### 稀疏与密集数组

稀疏数组即非连续性数组，里面有空元素，这在使用数组方法（map,forEach 等）时不会遍历该元素。

快速创建密集数组：

```javascript
//
const arr = Array.apply(null, Array(3));
// [undefined, undefined, undefined]

// 扩展应用
const arr = Array.apply(null, Array(5));

arr.map(Function.prototype.call.bind(Number));
arr.map(Function.prototype.call, Number);
// 等同于下面
arr.map((v, i, array) => {
  return Number.call(v, i, array);
});
// [1, 2, 3, 4, 5]
// ？？？为什么这样不行，arr.map(Number.call)
```

## 数据转换为规则

### ToPrimitive

转换逻辑：
如果已经是原始类型了，那就不需要转换了
如果需要转字符串类型就调用 x.toString()，转换为基础类型的话就返回转换的值。不是字符串类型的话就先调用 valueOf，结果不是基础类型的话再调用 toString
调用 x.valueOf()，如果转换为基础类型，就返回转换的值
如果都没有返回原始类型，就会报错

#### string

调用 toString 方法：

- 原始值，强转 string 后的原始值
- 非原始值，调用 valueOf 方法
  - 原始值，强转 string 后的原始值
  - 非原始值，抛出异常`TypeError: Cannot convert object to primitive value`

> **在 + 运算符时，表现为先使用 valueOf()**。Date 对象直接调用 toString 完成转换，其他对象通过 valueOf 转化，如果转换不成功则调用 toString。

#### number

调用 valueOf 方法：

- 原始值，return 强转 number 后的原始值
- 非原始值，调用 toString 方法 - 原始值，return 强转 number 后的原始值 - 非原始值，抛出异常`TypeError: Cannot convert object to primitive value`
  > 强转 number 规则，null -> 0，undefined -> NaN，true -> 1，false -> 0，失败 -> NaN

## 递归

纯粹的函数式编程语言没有循环操作命令，所有的循环都用递归实现。

递归就是函数自己调用自己。

### 尾调用

函数最后一步调用一个函数，只调用函数，不做其它运算。

### 尾递归

函数最后一步调用自身。

例如：一个阶乘函数，n 够大时发生栈溢出。

```javascript
function factorial(n) {
  if (n === 1) return 1;
  return n * factorial(n - 1);
}
```

改写成尾递归，由于每次执行完后不需要保持当前作用域的变量，所以可销毁，永远只有一层（V8 引擎未实现）。

```javascript
function factorial(n, total) {
  if (n === 1) return total;
  return factorial(n - 1, n * total);
}
```

对于不支持自动优化的，改写为迭代模式。

```javascript
const factorial = (v) => {
  let initialValue = v;
  let result = 1;
  while (initialValue > 0) {
    result = initialValue * result;
    initialValue--;
  }
  return result;
};
```

## 内存

内存的生命周期：

1. 分配内存
2. 使用内存（读写）
3. 释放内存

在低级语言中（c），1/3 步骤是需要手动操作的。而在 js 中，这变成了自动。

### 栈（stack）、堆（heap）

堆栈是操作系统给应用程序分配的两种不同的内存空间方案。stack 是 os 分配的**固定**的一片连续内存，线程在这个范围内顺序读写。heap 也是 os 分配的一片内存，但是数据却分布的存储在各处。

进行这样的方案，是因为程序通常会有一些存在时间极短，极小的数据，那么此类数据可以保存在 stack 中，在 js 里代表执行上下文变量、一些基础数据类型值。而一些大小不固定，复杂的数据会保存在 heap 中，在 js 里代表的是引用类型数据。

stack 会因为嵌套调用太多或者数据量过大造成 stackoverflow（堆栈溢出），heap 会产生更多的内存碎片。

通常一个线程会有自己独立的 stack，而 heap 可能会进行共享。

由于 stack 通常是即用即销毁，因此 os 顺便进行了内存管理，而 heap 较复杂，需要手动管理。那么垃圾回收主要就是回收堆内存的数据。

> Chrome Devtool 里的 Memory 选项卡就用于对堆内存的监控。

### 垃圾回收 GC（Garbage Collection）

- 引用计数（reference counting）。缺点：无法解决循环引用。
- 标记-清除（mark and sweep algorithm）。从根对象出发，依次搜索其子节点，再递归，所有能达到的节点即为活动节点，其余节点即可清除。

堆中分为新生代和老生代两个区域。

#### 新生代

存放临时对象，由副垃圾回收器 Scavenge 管理。标记加复制的方式整理内存。

#### 老生代

存放持久对象，由主垃圾回收器管理。在新生代多次回收过后依然保留的对象会转移到老生代区域。

### 常见内存泄漏

- 全局变量。不声明变量或者全局作用域函数中调用 this，不小心把变量挂载到全局对象上，此变量一直被全局对象引用，只有关闭窗口或者重新刷新页面才被释放。
- console.log。传入的对象会被一直引用，不会被 GC 回收。
- 闭包。首先闭包本身带作用域，所以占用内存会比一般的多。然后在使用时，很多不明显的变量会被缓存，例如:

```javascript
function f() {
  var str = Array(10000).join("#");
  var foo = {
    name: "foo",
  };
  const unused = () => {
    var message = "it is only a test message";
    str = "unused: " + str;
  };
  function getData() {
    return "data";
  }
  return getData;
}
var list = [];
document.querySelector("#btn").addEventListener(
  "click",
  function () {
    list.push(f());
  },
  false
);
// str变量虽然没使用，但一直被缓存。
```

解释如下：在相同作用域内创建的多个内部函数对象是共享同一个变量对象 AO。如果创建的内部函数没有被其他对象引用，不管内部函数是否引用外部函数的变量和函数，在外部函数执行完，对应变量对象便会被销毁。反之，如果内部函数中存在有对外部函数变量或函数的访问（可以不是被引用的内部函数），并且存在某个或多个内部函数被其他对象引用，那么就会形成闭包，外部函数的变量对象就会存在于闭包函数的作用域链中。这样确保了闭包函数有权访问外部函数的所有变量和函数。

个人理解：内部函数没有被引用，那么所有变量执行完后就销毁。如果有一个内部函数被其他地方引用，那么大家（所有内部函数）引用的东西都得存在闭包里面，就算你没有被其他地方引用也得存下来。

另一个闭包泄露例子：

```javascript
var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var unused = function () {
    if (originalThing) console.log("hi");
  };
  theThing = {
    longStr: new Array(1000000).join("*"),
    someMethod: function () {
      console.log(someMessage);
    },
  };
};
setInterval(replaceThing, 1000);
```

- DOM。dom 赋值给变量，dom 销毁后，变量依然保持引用，造成 dom 没有被真正销毁。
- timers。一般是 setInterval 用完没有被及时 clear 以及 setTimeout 嵌套没有被正确终止。定时器运行会消耗一定的内存，另外里面引用的变量也不会被回收。
- EventListener。一般是某个重复事件中绑定其他 dom 事件，当事件重复执行时，造成了重复绑定，导致异常。
  > dom.addEventListener('click', fn, false)这种方式多次执行绑定不会增加新事件。

## 严格模式

包括但不限于以下特点：

- （很重要的一个原因）全局对象为 undefined。这样就不会因为未声明变量而挂载到全局对象上，从而造成内存泄漏，因为一旦挂载了，那么只有刷新页面或者关闭 tab 才会被释放。
- arguments 不可用。据阮一峰文章解释，这样就可以优化尾递归，因为非严格模式拥有 arguments.callee，需指向本身，造成了作用域栈不能被优化。（测试未通过）
- call/apply方法，第一个参数如果传null/undefined，this指针会指向宿主对象。在严格模式下，指向真实传递的null或者undefined。

## JSON.stringify

JSON.stringify(val, replacer, space)，后面 2 个参数都只对对象有效。

### replacer

输出策略。

- Array。只转换数组内的 key 值。
- Function。传给你 key 和 value，然后自己做逻辑操作，返回想要的 value，如果返回为 undefined，则相当于排除掉这个 key 的输出。
  > 注意，第一次的传入此函数的 key 和 value 分别为空字符`''`和对象本身`obj`。后面才开始是每个 key。

### space

只针对对象有作用，用于美化对象的输出。无论是空格还是字符，个数上限都是 10。

- Number。0 就是默认，比 0 小无效。每一行前面加相应数目的空格。
- String。每一行前面加相应的字符。

## 模块化

### AMD

代表 require.js。遵循 AMD（asynchronous module definition）规范。注重依赖提前确定。过程为依赖提前下载 -> 然后乱序执行 -> 等待所有模块执行完毕 -> 执行回调函数 -> 输出值的一份引用。

### CommonJS、CJS

代表 node。nodejs 的自有模块系统。一个文件即一个模块即一个模块对象，使用`module.exports = 值`或者`exports = 值`导出。打包后，输出到 module 中的 exports 对象里

node 中的模块缓存对象：

```javascript
{
  '模块的绝对路径': Module {
    id: '.',
    path: '',
    exports: {},
    parent: null,
    filename: '',
    loaded: false,
    children: [ [Module], [Module] ],
    paths: '查找搜索路径'
  },
  }
}
```

#### CMD

代表 sea.js。之所以放这里，是以为淘宝出品，类似于造轮子，类似于 cjs。注重按需加载依赖。依赖提前下载，然后运行到使用 require 关键词的地方再执行，输出值的一份引用

### EsModule、esm

浏览器端的模块规范。使用`export default 值`或者`export 定义式`导出，后者使用定义式是因为打包工具需要把变量命挂在到 module.exports 的 getter 上（这样能获取到实时的值），而把前者放到 module.exports.default 里。

JavaScript 规范声明任何没有 export 或者顶层 await 的 JavaScript 文件都应该被认为是一个脚本，而非一个模块。

### UMD

通用模块定义。其实就是检查了各个模块支持情况，按照优先级来导出，AMD -> CJS -> 浏览器全局变量，因此在多个地方可以放心使用，因此“通用”。

### 区别

commonjs 是运行时再加载，而 esModule 在初期就已经分析出依赖关系（因此可以做 tree-shaking），预留好了要 export 的对象的内存，在具体执行时再进行填值。但是打包工具是把 esModule 打包成 commonjs 的模块。

### pkg 的 module、main 字段

通常一个包最稳的是提供一个用 es5 代码编写的 commonjs 规范文件，放在 main 字段，但是 esModule 规范有个好处可以做 tree-shaking，因此增加了一个 module 字段来使用。

### 浏览器支持情况

部分浏览器已原生支持 module，需要设置`type="module"`。规范使用`.mjs`来代表模块文件，与普通 js 文件以区分，不过`.js`兼容性更好。

## 文件

### Blob 、 File 、 Data URL 、 ArrayBuffer

- Data URL，即以`data:`协议开头的 URL，故名思议，就是地址本身代表了内容。目前遇见的有：

  - base64 编码内容后的的小文件，其地址为：`data:${MIME类型，例如: text/html,image/jpeg,text/plain等};base64,${编码后内容}`
  - 标签形式的内容，比如一张 svg 图片。其地址为：`data:image/svg+xml,${转义后的svg标签}`

- Blob，表示一个只读原始的数据的类文件对象。
- File，继承自 Blob，并提供了一些额外的元数据，例如：name、lastModified 等。
- ArrayBuffer，ArrayBuffer 对象用来表示通用的、固定长度的原始二进制数据缓冲区。我们可以通过 new ArrayBuffer(length)来获得一片连续的内存空间，它不能直接读写，但可根据需要将其传递到 TypedArray 视图或 DataView 对象来解释原始缓冲区。实际上视图只是给你提供了一个某种类型的读写接口，让你可以操作 ArrayBuffer 里的数据。TypedArray 需指定一个数组类型来保证数组成员都是同一个数据类型，而 DataView 数组成员可以是不同的数据类型。
- TypedArray，类型化数组是若干个类数组视图的统称，一个类数组视图对象描述了一个底层的二进制数据缓冲区（binay data buffer）。
  - Int8Array：8 位有符号整数，长度 1 个字节。
  - Uint8Array：8 位无符号整数，长度 1 个字节。
  - Uint8ClampedArray：8 位无符号整数，长度 1 个字节，溢出处理不同。
  - Int16Array：16 位有符号整数，长度 2 个字节。
  - Uint16Array：16 位无符号整数，长度 2 个字节。
  - Int32Array：32 位有符号整数，长度 4 个字节。
  - Uint32Array：32 位无符号整数，长度 4 个字节。
  - Float32Array：32 位浮点数，长度 4 个字节。
  - Float64Array：64 位浮点数，长度 8 个字节。

### 互相转换

#### Base64 -> Blob

先利用 atob 函数还原 base64 数据区域的内容，得到一个字符串。然后依次遍历字符串，利用 charCodeAt 提取每一个字符的 Unicode 码并放在 Uint8Array 中，最后直接使用 `new Blob([u8arr])`构造文件对象。

```js
function dataUrl2blob(dataUrl) {
  const base64String = /base64,(.*)/.exec(dataUrl)[1];
  const u8arr = new Uint8Array(base64String.length);
  window
    .atob(base64String)
    .split("")
    .forEach((s, index) => {
      u8arr[index] = s.charCodeAt(0);
    });
  return new Blob([u8arr], { type: "image/png" });
}
```

### Blob/File -> Base64

使用 FileReader 的 readAsDataURL 接口

```js
function file2base64(file) {
  return new Promise((resolve) => {
    const fs = new FileReader();
    fs.onload = function () {
      resolve(this.result);
    };
    fs.readAsDataURL(file);
  });
}
```

### Blob -> ArrayBuffer

利用 FileReader 的`readAsArrayBuffer()`读取，在 onload 事件中的 result 即为结果。

### ArrayBuffer -> Blob / File

直接`new Blob([u8arr], { type: 'text/html' })`

> ！注意 uint8Array 外层的方括号。

### ArrayBuffer -> 字符串

可以使用 TextDecoder 的 decode 实例方法把类型化数组转成字符串。

### 字符串 -> ArrayBuffer
如果字符串全英文，可以直接使用`charCodeAt`转，含有中文的话，需要中间加一层`encodeURIComponent`。

### blob -> url

URL.createObjectURL()该方法创建一个 DOMString，表示指定的 File 或 Blob 对象，这个 URL 的生命周期和 document 绑定。

## Reflect

Reflect 是一个内置对象，它提供了拦截和操作对象的 api，类似于 Math。

## ES6+各版本新特性

- ES2016。Array.prorotype.includes()
- ES2017。async/await、Object.values()/entries()
- ES2018。Promise.finally()
- ES2019。Array.prototype.flat()
- ES2020。可选链接、空值操作??、按需导入、BigInt、globalThis、Promise.allSettled

## Proxy

### handler.has()

针对以下情况的代理方法：

- in 查询`foo in obj`
- with 查询

```js
with (obj) {
  // 这里的取值会触发obj的handler.has()代理
  // true: 代表在obj上进行取值
  // false： 代表会去上级作用域取值
}
```

- Reflect.has()（与 in 操作符功能一致）
