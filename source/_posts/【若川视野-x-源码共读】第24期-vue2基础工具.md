---
title: 【若川视野 x 源码共读】第24期|vue2基础工具
date: 2022-04-26 10:31:02
update: 2022-04-26 10:31:02
categories: 源码阅读
---
**本文参加了由[公众号@若川视野](https://link.juejin.cn/?target=https%3A%2F%2Flxchuan12.gitee.io "https://lxchuan12.gitee.io") 发起的每周源码共读活动，[点击了解详情一起参与。](https://juejin.cn/post/7079706017579139102 "https://juejin.cn/post/7079706017579139102")**

# 前言

1. 这是跟随[@若川](https://juejin.cn/user/1415826704971918) 源码活动的第二个打卡。
2. 该期活动地址[【若川视野 x 源码共读】第24期 | vue2工具函数](https://juejin.cn/post/7079765637437849614)

# 工具函数

## 1. emptyObject

```javascript
/*!
 * Vue.js v2.6.14
 * (c) 2014-2021 Evan You
 * Released under the MIT License.
 */
/*  */
var emptyObject = Object.freeze({});
```

[**Object.freeze()**](https://lxchuan12.gitee.io/js-object-api/#object-isfrozen-obj-es5)

> `Object.freeze()` 方法可以冻结一个对象，冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象。

```javascript
var deadline = Object.freeze({date: 'yesterday'});
deadline.date = 'tomorrow';
deadline.excuse = 'lame';
deadline.date; // 'yesterday' 不可以修改已有属性的值
deadline.excuse; // undefined 不可以向这个对象中添加新的属性
Object.isSealed(deadline); // true; 
Object.isFrozen(deadline); // true 判断对象是否冻结的方法
Object.getOwnPropertyDescriptor(deadline, 'date');
// {value: "yesterday", writable: false, enumerable: true, configurable: false} (不可配置，不可写)
Object.keys(deadline); // ['date'] (可枚举)
```

## 2. isUndef 

> 判断是否是未定义

```javascript
// These helpers produce better VM code in JS engines due to their
// explicitness and function inlining.
function isUndef (v) {
  return v === undefined || v === null
}
```

## 3. isDef

> 判断是否已经定义

```javascript
// These helpers produce better VM code in JS engines due to their
// explicitness and function inlining.
function isDef (v) {
  return v !== undefined && v !== null
}
```

## 4. isTrue

```javascript
function isTrue (v) {
  return v === true
}
```

## 5. isFalse

```javascript
function isFalse (v) {
  return v === false
}
```

## 6. isPrimitive

> 判断值是否是原始值

```javascript
/**
 * Check if value is primitive.
 */
function isPrimitive (value) {
  return (
    typeof value === 'string' ||
    typeof value === 'number' ||
    // $flow-disable-line
    typeof value === 'symbol' ||
    typeof value === 'boolean'
  )
}
```

## 7. isObject

> 判断是否是对象，没有区分对象和数组

```javascript
/**
 * Quick object check - this is primarily used to tell
 * Objects from primitive values when we know the value
 * is a JSON-compliant type.
 */
function isObejct(obj) {
  return obj !== null && type of === 'object'
}
```

## 8. toRowType

> 转换成原始类型

```javascript
var _toString = Object.prototype.toString;
function toRowType(value) {
  return _toString.call(value).slice(8, -1)
}

toRowType('') // 'String'
toRowType()// 'undefined'
```

**为什么需要.slice(8, -1)**

1. _toString.call(value) 生成结果[object String] 。
2. [二、vue-shared.md](二、vue-shared.md) __toString.call(value).slice(8, -1) 去掉[object和]。

## 9. isPlainObject

> 判断是否是纯对象

```javascript
function isPlainObject (obj) {
  return _toString.call(obj) === '[object Object]'
}

isPlainObject({}) // true
isPlainObject([])
```

## 10. isRegExp

> 判断是否是正则表达式

```javascript
function isRegExp(obj) {
  return _toString.call(obj) === '[object RegExp]'
}
```

## 11. isValidArrayIndex

> 判断是否是可用的数组索引值

```javascript
function isValidArrayIndex(val) {
  var n = parseFloat(String(val));
  return n >= 0 && Math.floor(n) === n && isFinite(val);、
}
```

1. 为什么要使用parseFloat？

   使用parseFloat可解析其他进制字符串时，有可能由于计算精度问题无法得到一个确定的十进制证书。

2. 为什么使用isFinite

   > is Finite()函数判断被传入的参数值是否为一个有限数值。

   ```javascript
   isFinite(Infinity);  // false
   isFinite(NaN);       // false
   isFinite(-Infinity); // false
   
   isFinite(0);         // true
   isFinite(2e64);      // true
   
   isFinite('0');       // true
   Number.isFinite('0') // false 
   Number.isFinite(null) // false
   ```

# 12. isPromise

> 判断是否是promise

```javascript
funciton isPromise(val) {
  return (
  	isDef(val) && 
    typeof val.then === 'function' &&
    typeof val.catch === 'function'
 	)
}

isPromise(new Promise()) // true
```

# 13. toString转字符串

```javascript
let _toString = Object.prototype.toString;

function toString(val) {
	return val == null 
		? '' 
		: Array.isArray(val) || (isPlainObject(val) && val.toString === _toString)
			? JSON.stringify(val, null, 2)
			: String(val)
}
```

1. 数组或者对象并且对象的`toString` 方法是Object.prototype.toString，用JSON.stringify转换

2. JSON.stringify(val, null, 2)

   1. JSON.stringify(value, replacer, space)

      - `value` 序列化为一个JSON字符串的值
      - `replacer` 如果该参数是一个函数，则在序列化过程中，被序列化的值的每个属性都会经过该函数的转换和处理；如果该参数是一个数组，则只有包含在这个数组中的属性名才会被序列化到最终的 JSON 字符串中；如果该参数为 null 或者未提供，则对象所有的属性都会被序列化。
      - `space` 指定缩进用的空白字符串，用于美化输出（pretty-print）；如果参数是个数字，它代表有多少的空格；上限为10。该值若小于1，则意味着没有空格；如果该参数为字符串（当字符串长度超过10个字母，取其前10个字母），该字符串将被作为空格；如果该参数没有提供（或者为 null），将没有空格。

   2. 例子

      ```javascript
      function replacer(key, value) {
        if (typeof value === "string") {
          return undefined;
        }
        return value;
      }
      
      var foo = {
        foundation: "Mozilla",
        model: "box",
        week: 45,
        transport: "car",
        month: 7
      };
      var jsonString = JSON.stringify(foo, replacer);// {week: 45, month: 7}
      ```

# 14. toNumber

> 转数字

```javascript
function toNumber(val) {
  var n = parseFloat(val);
  return isNaN(n) ? val : n;
}


toNumber('a') // 'a'
toNumber('1') // 1
toNumber('1a') // 1
toNumber('a1') // 'a1'
```

# 15. makeMap生成一个map

> 传入一个以逗号分隔的字符串，生成一个map，并且返回一个函数检测key值在不在这个map中。第二个参数是小写选项。

```javascript
// 传入一个以逗号分隔的字符串，生成一个 map(键值对) ，并且返回一个函数检测 key 值在不在这个 map 中。第二个参数是小写选项。
function makeMap(str, expectsLowerCase) {
  const map = Object.create(null);
  let list = str.split(',');
  for (let i = 0; i < list.length; i++) {
    map[list[i]] = true;
  }
  return expectsLowerCase
    ? function (val) { return map[val.toLowerCase()]; }
    : function (val) { return map[val] };
}

let str = '123,456,789';
let makeMapStrFunc = makeMap(str, false);
makeMapStrFunc('123'); // true
makeMapStrFunc('145'); // undefined

```

# 16. isBuiltInTag

> 判断是否是内置的Tag

```javascript
var isBuiltInTag = makeMap('slot, component', true);

isBuiltInTag('slot') // true
isBuiltInTag('component') // true
isBuiltInTag('Slot') // true
isBuiltInTag('Component') // true
```

# 17. isReservedAttribute

> 判断是否是保留的属性

```javascript
let isReservedAttribute = makeMap('key,ref,slot,slot-scope,is');

isReservedAttribute('key') // true
isReservedAttribute('ref') // true
isReservedAttribute('slot') // true
isReservedAttribute('slot-scope') // true
isReservedAttribute('is') // true
isReservedAttribute('IS') // undefined
```

# 18. remove

> 移除数组中的某一项

```javascript
function remove(arr, item) {
  if (arr.length) {
    var index = arr.indexOf(item);
    if (index > -1) {
      return arr.splice(index, 1)
    }
  }
}
```

> `splice` 其实是一个很耗性能的方法。删除数组中的一项，其他元素都要移动位置。
>
> **引申**：[`axios InterceptorManager` 拦截器源码](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Faxios%2Faxios%2Fblob%2Fmaster%2Flib%2Fcore%2FInterceptorManager.js) 中，拦截器用数组存储的。但实际移除拦截器时，只是把拦截器置为 `null` 。而不是用`splice`移除。最后执行时为 `null` 的不执行，同样效果。`axios` 拦截器这个场景下，不得不说为性能做到了很好的考虑。**因为拦截器是用户自定义的，理论上可以有无数个，所以做性能考虑是必要的**。
>
> 看如下 `axios` 拦截器代码示例：
>
> ```javascript
> // 代码有删减
> // 声明
> this.handlers = [];
> 
> // 移除
> if (this.handlers[id]) {
> this.handlers[id] = null;
> }
> 
> // 执行
> if (h !== null) {
> fn(h);
> }
> 
> ```

# 19. hasOwn

> 检测是否是自己的属性

```javascript
var hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return hasOwnProperty.call(obj, key)
}

let hasOwnProperty = Object.prototype.hasOwnProperty;
function hasOwn(obj, key) {
  return hasOwnProperty.call(obj, key);
}

let obj = {
  a: 123
}
console.log(obj.hasOwnProperty('a'));  // true
console.log(hasOwn(obj, 'a')); // true
console.log(hasOwn(obj, 'b')); // false
```

# 20. cached

> 缓存

```javascript
function cached (fn) {
  let cache = Object.create(null);
  return (function cachedFn(str) {
    var hit = cached[str];
    return hit || (cache[str] = fn(str));
  })
}
```

# 21. camelize

> 连字符转小驼峰 如 on-click => onClick

```javascript
var camelizeRE = /-(\w)/g;
var camelize = cached(function (str) {
  return str.replace(camelizeRE, function (_, c) { return c ? c.toUpperCase() : '' });
})
```

1. \w 表示 [0-9a-zA-Z_]。表示数字、大小写字母和下划线。 

2. ```javascript
   String.prototype.replace(regexp | substr, newSubStr|function)
   ```

   我们来重点看一下第二个参数中的`function` 中的参数 

   [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace#%E6%8C%87%E5%AE%9A%E4%B8%80%E4%B8%AA%E5%87%BD%E6%95%B0%E4%BD%9C%E4%B8%BA%E5%8F%82%E6%95%B0)

   - `match`

   - `p1,p2, ...` 假如replace()方法的第一个参数是一个[`RegExp`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp) 对象，则代表第n个括号匹配的字符串。如果是用 `/(\a+)(\b+)/` 这个来匹配，`p1` 就是匹配的 `\a+`，`p2` 就是匹配的 `\b+`。

     > 因此，在这里，_ 的值是 -c,  c 的值 是 c

   - ...

# 22. capitalize

> 首字母转大写

```javascript
var capitalize = cached(function (str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
})
```

# 23. hyphenate

> 小驼峰转连字符 onClick => on-click

```javascript
var hyphenateRE = /\B([A-Z])/g;
var hyphenate = cached(function(str) {
  return str.replace(hyphenateRE, '-$1').toLowerCase();
})
```

# 24. polyfillBind

>  bind垫片 兼容了老版本浏览器不支持原生的bind函数。同时兼容写法，对参数的多少做出了判断，使用call和apply实现。

```javascript
function polyfillBind(fn, ctx) {
  function boundFn(a) {
    var l = arguments.length;
    return l
      ? l > 1
        ? fn.apply(ctx, arguments)
        : fn.call(ctx, a)
      : fn.call(ctx)
  }

  boundFn._length = fn.length;
  return boundFn;
}

function nativeBind(fn, ctx) {
  return fn.bind(ctx);
}

var bind = Function.prototype.bind
  ? nativeBind
  : polyfillBind;
```

# 25. toArray

> 把类数组转成真正的数组， 支持从指定位置开始转化，默认从0开始。

```javascript
function toArray(list, start) {
  start = start || 0;
  var i = list.length - start;
  var ret = new Array(i);
  while (i--) {
    ret[i] = list[i + start];
  }
  return ret;
}

// 例子
function fn() {
  var args = toArray(arguments); 
  console.log(args); // 1, 2, 3, 4, 5
  var args2 = toArray(arguments, 2);
  console.log(args2); // 3, 4, 5
}
fn(1, 2, 3, 4, 5)
```

# 26. extend

> 合并对象

```javascript
function extend(to, _from) {
  for (let key in _from) {
    to[key] = _from[key];
  }
  return to;
}

// 例子
const data = { name: 'lily' };
const data2 = extend(data, { age: '18', name: 'lily1' });
console.log(data); // { age: '18', name: 'lily1' }
console.log(data2); // { age: '18', name: 'lily1' }
console.log(data === data2); // true
```

# 27. toObject

> 转对象， 传入的是个数组，后面的元素会覆盖前面的元素

```javascript
function toObject(arr) {
  var res = {};
  for (let i = 0; i < arr.length; i++) {
    if (arr[i]) {
      extend(res, arr[i])
    }
  }

  return res;
}

//例子
toObject(['笑嘻嘻','哈哈呵呵']); // 哈哈呵呵
```

# 28. noop

> 空函数

```javascript
function noop(a, b, c) {}
```

# 29. no

> 一直返回false

```javascript
var no = function(a, b, c) { return false; };
```

# 30. identity

> 返回参数本身

```javascript
var identity = function(_) {  return _; };
```

# 31. genStaticKeys

> 生成静态属性

```javascript
function genStaticKeys (modules) {
  return modules.reduce(function (keys, m) {
    return keys.concat(m.staticKeys || [])
  }, []).join(',')
}
```

# 32. looseEqual

> 宽松相等

由于数组、对象等是引用类型，所以两个内容看起来相等，严格相等都是不相等。

```javascript
var a = {};
var b = {};
a === b; // false
a == b; // false
复制代码
```

所以该函数是对数组、日期、对象进行递归比对。如果内容完全相等则宽松相等。

```javascript
function looseEqual(a, b) {
  if (a === b) { return true };
  var isObjectA = isObject(a);
  var isObjectB = isObject(b);
  if (isObjectA && isObjectB) {
    try {
      var isArrayA = Array.isArray(a);
      var isArrayB = Array.isArray(b);
      if (isArrayA && isArrayB) {
        return a.length === b.length && a.every(function (e, i) {
          return looseEqual(e, b[i])
        })
      } else if (a instanceof Date && b instanceof Date) {
        return a.getTime() === b.getTime();
      } else if (!isArrayA && !isArrayB) {
        var keysA = Object.keys(a);
        var keysB = Object.keys(b);
        return keysA.length === keysB.length && keysA.every(function (key) {
          return looseEqual(a[key], b[key])
        })
      } else {
        return false;
      }
    } catch (e) {
      return false;
    }
  } else if (!isObjectA && !isObjectB) {
    return String(a) === String(b);
  } else {
    return false;
  }
}
```

# 33. looseIndexOf

> 宽松的in de x O f

```javascript
function looseIndexOf(arr, val) {
  for (var i = 0; i < arr.length; i++) {
    if (looseEqual(arr[i], val) { return i };
  }
  return -1;
}
```

# 34. once

> 利用闭包特性，存储状态，实现函数只执行一次

```javascript
function once(fn) {
  var called = false;
  return function () {
    if (!called) { 
      called = true;
      fn.apply(called, arguments);
    }
  }
}
```