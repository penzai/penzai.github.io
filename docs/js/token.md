## 词法结构

### 标识符

用于常量、变量、属性、函数和类，以及为某些循环提供标记（label）。

必须以字母、下划线（\_）或美元符号（$）开头。

> 数字不能作为第一个字符，以便 JavaScript 区分标识符和数值。

虽建议标识符只使用 ASCLL 字母和数字，但是本身支持 Unicode 字母、数字、象形文字。

### 转义序列

为了仅使用 ASCII 字符来表示 Unicode 字符。

以`\u`开头，后跟 4 位 16 进制数字，es6 中可使用`\u{1-6位16进制数字}`来更好的表示 unicode 字符。

### 分号策略

JavaScript 并非任何时候都把换行符当作分号，而只是
在不隐式添加分号就无法解析代码的情况下才这么做。更准确地讲
（除了稍后介绍的三种例外情况），JavaScript 只在下一个非空格
字符无法被解释为当前语句的一部分时才把换行符当作分号。

> 通常，如果语句以(、[、/、+或-开头，就有可能被解释为之前语句的一部分。

JavaScript 在不能把第二行解析为第一行的连续部分时，对换
行符的解释有三种例外情况。

1. 涉及 return、throw、yield、break 和 continue 语句，这些语句经常独立存在，但有时候后面也会跟一个标识符或表达式。如果这几个单词后面（任何其他标记前面）有换行符，JavaScript 就会把这个换行符解释为分号。
   > 一定不能在 return、break 或 continue 等关键字和它们后面的表达式之间加入换行符。
2. ++和--操作符。
3. 箭头=>必须跟参数列表在同一行。
