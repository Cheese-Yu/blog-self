# JavaScript浮点数精度问题
为什么 `0.1 + 0.2 = 0.30000000000000004` ，`1 - 0.9 = 0.09999999999999998`，`0.57 * 100 === 56.99999999999999` ？

在 JavaScript 中所有数值都以 `IEEE-754` 标准的`64 bit`双精度浮点数进行存储的。  
IEEE-754 标准下双精度浮点数由三部分组成, 分别如下:

> - sign(符号): 占 1 bit, 0 表示正, 1 表示负;
> - exponent(指数): 占 11 bit, 表示范围;
> - mantissa(尾数): 占 52 bit, 表示精度, 多出的末尾如果是 1 需要进位;

因此，JavaScript 中的最大精度数为 `Math.pow(2, 53) - 1`（64 位浮点的后 52 位+有效数字第一位的 1）  
十进制转化为二进制时会有精度问题，`0.1`转成二进制表示为`0.0001100110011001100(1100循环)`，因浮点数小数位的限制而截断的二进制数。

那么`0.57 * 100 === 56.99999999999999` 而 `0.57 * 1000 === 570`？计算机的乘法实际上是累加计算, 并不是我们想的按位相乘。

```javascript
// 伪代码
(0.57) * 100
= (0.57) * (64 + 32 + 4)
= (0.57二进制) * (2^6 + 2^5 + 2^2)
= 0.57二进制 * 2^6 + 0.57二进制 * 2^5 + 0.57 * 2^2
```

### 解决方案

我们可以把需要计算的数字升级（乘以 10 的 n 次幂）成计算机能够精确识别的整数，等计算完成后再进行降级（除以 10 的 n 次幂）。

```javascript
const floatCalc = (function () {
  function toInt(num) {
    return Number(num.toString().replace(".", ""));
  }

  function digistLen(num) {
    return (num.toString().split(".")[1] || "").length;
  }

  /**
   * 判断相等
   */
  function equal(num1, num2) {
    return Math.abs(num1 - num2) < Math.pow(2, -52);
  }

  /**
   * 加法
   */
  function add(num1, num2) {
    const num1Digists = digistLen(num1);
    const num2Digists = digistLen(num2);
    const baseNum = Math.pow(10, Math.max(num1Digists, num2Digists));
    return (num1 * baseNum + num2 * baseNum) / baseNum;
  }

  /**
   * 减法
   */
  function subtract(num1, num2) {
    const num1Digists = digistLen(num1);
    const num2Digists = digistLen(num2);
    const baseNum = Math.pow(10, Math.max(num1Digists, num2Digists));
    return (num1 * baseNum - num2 * baseNum) / baseNum;
  }

  /**
   * 乘法
   */
  function multiply(num1, num2) {
    const num1Digists = digistLen(num1);
    const num2Digists = digistLen(num2);
    return (
      (toInt(num1) * toInt(num2)) / Math.pow(10, num1Digists + num2Digists)
    );
  }

  /**
   * 除法
   */
  function divide(num1, num2) {
    const num1Digists = digistLen(num1);
    const num2Digists = digistLen(num2);
    return multiply(
      toInt(num1) / toInt(num2),
      Math.pow(10, num2Digists - num1Digists)
    );
  }

  return {
    equal,
    add,
    subtract,
    multiply,
    divide,
  };
})();
```
  
<br />  
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/qca1wb