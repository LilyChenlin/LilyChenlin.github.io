---
title: axios中的utils工具文件.md
date: 2022-03-09 20:43:06
update: 2022-03-09 20:43:06
categories: 源码阅读
---
# Axios/utils.js

# 前言

1. 这是跟随[@若川](https://juejin.cn/user/1415826704971918) 打卡源码活动的第一个打卡。
2. 参考[阅读axios源码，发现了这些实用的基础工具函数](https://juejin.cn/post/7042610679815241758) ，不得不说，环境准备写得很细致，所以这边留个记录。
3. 本次源码阅读的主要内容是[utils.js](https://github.com/axios/axios/blob/master/lib/utils.js).


<!--more-->
# 工具函数

> 本节内容将遍历utils.js中所有的工具函数

## isArray

```javascript
/**
 * Determine if a value is an Array
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is an Array, otherwise false
 */
function isArray(val) {
  return Array.isArray(val)
}
```

## isUndefined

```javascript
/**
 * Determine if a value is undefined 判断值是否是undefined
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if the value is undefined, otherwise false
 */
function isUndefined(val) {
  return typeof val === 'undefined';
}
```

## isBuffer

```javascript
/**
 * Determine if a value is a Buffer 判断一个值是否是Buffer类型
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a Buffer, otherwise false
 */
function isBuffer(val) {
  return val !== null &&
    		 !isUndefined(val) &&
    		 val.constructor !== null &&
    		 !isUndefined(val.constructor) &&
    		 typeof val.constructor.isBuffer === 'function' &&
    		 val.constructor.isBuffer(val); // 通过构造函数的isBuffer方法来判断
}
```

> 什么是Buffer

[参考](https://www.runoob.com/nodejs/nodejs-buffer.html)

Buffer 缓冲区

存在原因：

1. JavaScript 语言自身只有字符串数据类型，没有二进制数据类型。
2. 但在处理像TCP流或文件流时，必须使用到二进制数据。因此在 Node.js中，定义了一个 Buffer 类，该类用来创建一个专门存放二进制数据的缓存区。

## isArrayBuffer

```javascript
let toString = Object.prototype.toString;
function isArrayBuffer(val) {
  return toString.call(val) === '[object ArrayBuffer]'
}
```

> 什么是ArrayBuffer

[参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)

ArrayBuffer 通用的、固定长度的原始二进制数据缓冲区

## isFormData

```javascript
/**
 * Determine if a value is a FormData
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is an FormData, otherwise false
 */
function isFormData(val) {
  return toString.call(val) === '[object FormData]';
}
```

## isArrayBufferView

```javascript
/**
 * Determine if a value is a view on an ArrayBuffer
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a view on an ArrayBuffer, otherwise false
 */
function isArrayBufferView(val) {
  var result;
  if ((typeof ArrayBuffer !== 'undefined') && (ArrayBuffer.isView)) {
    result = ArrayBuffer.isView(val);
  } else {
    result = (val) && (val.buffer) && (isArrayBuffer(val.buffer));
  }
  return result;
}
```

1. 正常情况下, typeof ArrayBuffer === 'function'
2. Array.isView  如果参数是ArrayBuffer的视图实例则返回true

## isString

```javascript
/**
 * Determine if a value is a String
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a String, otherwise false
 */
function isString(val) {
  return typeof val === 'string';
}
```

## isNumber

```javascript
/**
 * Determine if a value is a Number
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a Number, otherwise false
 */
function isNumber(val) {
  return typeof val === 'number';
}
```

## isObject

```javascript
function isObject(val) {
  return val !== null && typeof val === 'object';
}
```

1. typeof null === 'object';

## isPlainObject

```javascript
/**
 * Determine if a value is a plain Object 判断是否是一个纯对象
 *
 * @param {Object} val The value to test
 * @return {boolean} True if value is a plain Object, otherwise false
 */
function isPlainObject(val) {
  if (toString.call(val) !== '[object Object]') {
    return false;
  }

  var prototype = Object.getPrototypeOf(val);
  return prototype === null || prototype === Object.prototype;
}
```

> Plain Object

1. 概念： 纯粹的对象

2. 怎么创建plain object，通过'{}'或'new Object'。即：

   - let obj = {}

   - let obj = new Object()

3. 对isPlainObject方法的理解

   - Object.getPrototypeOf(val) 获取val的原型链对象

   ```javascript
   let obj1 = {};
   const obj1Pro = Object.getPrototypeOf(obj1);
   console.log("obj1Pro", obj1Pro === Object.prototype); // true
   console.log("obj1", isPlainObject(obj1)); // true
   
   
   let obj2 = new Object();
   const obj2Pro = Object.getPrototypeOf(obj2);
   console.log("obj2Pro", obj2Pro === Object.prototype); // true
   console.log("obj2", isPlainObject(obj2)); // true
   
   
   let obj3 = Object.create(Object.prototype);
   const obj3Pro = Object.getPrototypeOf(obj3);
   console.log("obj3Pro", obj3Pro === Object.prototype); // true
   
   console.log("obj3", isPlainObject(obj3)); // true
   
   
   function C() { }
   let obj4 = new C();
   const obj4Pro = Object.getPrototypeOf(obj4);
   console.log("obj4Pro", obj4Pro === Object.prototype); // false
   console.log("obj4", isPlainObject(obj4)); // false
   ```

## isDate

```javascript
/**
 * Determine if a value is a Date
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a Date, otherwise false
 */
function isDate(val) {
  return toString.call(val) === '[object Date]';
}
```

## isFile

```javascript
/**
 * Determine if a value is a File
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a File, otherwise false
 */
function isFile(val) {
  return toString.call(val) === '[object File]';
}
```

## isBlob

```javascript
/**
 * Determine if a value is a Blob
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a Blob, otherwise false
 */
function isBlob(val) {
  return toString.call(val) === '[object Blob]';
}
```

## isFunction

```javascript
function isFunction(val) {
  return toString.call(val) === '[object Function]';
}
```

## isStream

```javascript
/**
 * Determine if a value is a Stream
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a Stream, otherwise false
 */
function isStream(val) {
  return isObject(val) && isFunction(val.pipe);
}
```

## isURLSearchParams

```javascript
/**
 * Determine if a value is a URLSearchParams object
 *
 * @param {Object} val The value to test
 * @returns {boolean} True if value is a URLSearchParams object, otherwise false
 */
function isURLSearchParams(val) {
  return toString.call(val) === '[object URLSearchParams]';
}
```

## trim

> 去除首尾空格

```javascript
/**
 * Trim excess whitespace off the beginning and end of a string
 *
 * @param {String} str The String to trim
 * @returns {String} The String freed of excess whitespace
 */
function trim(str) {
  return str.trim ? str.trim() : str.replace(/^\s+|\s+$/g, '');
}

```

## isStandardBrowserEnv 

> 判断是否是标准浏览器环境

```javascript
/**
 * Determine if we're running in a standard browser environment 是否运行在标准浏览器环境
 *
 * This allows axios to run in a web worker, and react-native.
 * Both environments support XMLHttpRequest, but not fully standard globals.
 *
 * web workers:
 *  typeof window -> undefined
 *  typeof document -> undefined
 *
 * react-native:
 *  navigator.product -> 'ReactNative'
 * nativescript
 *  navigator.product -> 'NativeScript' or 'NS'
 */
function isStandardBrowserEnv() {
  if (typeof navigator !== 'undefined' && 
       (navigator.product === 'ReactNative' ||                           
        navigator.product === 'NativeScript' ||
        navigator.product === 'NS')){
      return false;
 	}
  return (
    typeof window !== 'undefined' &&
    typeof document !== 'undefined'
  );
}
```

## forEach

```javascript
/**
 * Iterate over an Array or an Object invoking a function for each item.
 *
 * // fn.call(null, obj[i], i, obj);
 * If `obj` is an Array callback will be called passing
 * the value, index, and complete array for each item.
 *
 * // fn.call(null, obj[key], key, obj);
 * If 'obj' is an Object callback will be called passing
 * the value, key, and complete object for each property.
 *
 * @param {Object|Array} obj The object to iterate
 * @param {Function} fn The callback to invoke for each item
 */
function forEach(obj, fn) {
  // Don't bother if no value provided
  if (obj === null || typeof obj === 'undefined') {
    return;
  }

  // Force an array if not already something iterable
  if (typeof obj !== 'object') {
    // 强制转换为可遍历类型
    /*eslint no-param-reassign:0*/
    obj = [obj];
  }

  if (isArray(obj)) {
    // Iterate over array values
    for (var i = 0, l = obj.length; i < l; i++) {
      fn.call(null, obj[i], i, obj);
    }
  } else {
    // Iterate over object keys
    for (var key in obj) {
      if (Object.prototype.hasOwnProperty.call(obj, key)) {
        fn.call(null, obj[key], key, obj);
      }
    }
  }
}
```

1. Object.prototype.hasOwnProperty.call(obj, key) 判断obj本身是否有key这个属性
2. for...in 以任意顺序遍历一个对象的除[Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol)以外的[可枚举](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)属性，包括继承的可枚举属性。

## merge

```javascript
/**
 * Accepts varargs expecting each argument to be an object, then
 * immutably merges the properties of each object and returns result.
 *
 * When multiple objects contain the same key the later object in
 * the arguments list will take precedence.
 *
 * Example:
 *
 * ```js
 * var result = merge({foo: 123}, {foo: 456});
 * console.log(result.foo); // outputs 456
 * ```
 *
 * @param {Object} obj1 Object to merge
 * @returns {Object} Result of all merge properties
 */
function merge(/* obj1, obj2, obj3, ... */) {
  var result = {};
  function assignValue(val, key) {
    if (isPlainObject(result[key]) && isPlainObject(val)) {
      result[key] = merge(result[key], val);
    } else if (isPlainObject(val)) {
      result[key] = merge({}, val);
    } else if (isArray(val)) {
      result[key] = val.slice();
    } else {
      result[key] = val;
    }
  }

  for (var i = 0, l = arguments.length; i < l; i++) {
    forEach(arguments[i], assignValue);
  }
  return result;
}
```

>  merge

1. 是一个不定长参数函数，所以使用arguments类数组去获取传入的参数
2. 作用：合并对象，后面的对象如果key和前面的key是一样的，会进行覆盖。

（蛮好玩的，可以自己写一下）

## extend

```javascript
// bind实现
function bind(fn, thisArg) {
    return function wrap() {
        var args = new Array(arguments.length);
        for (var i = 0; i < args.length; i++) {
            args[i] = arguments[i];
        }
        return fn.apply(thisArg, args);
    };
};
/**
 * Extends object a by mutably adding to it the properties of object b.
 *
 * @param {Object} a The object to be extended
 * @param {Object} b The object to copy properties from
 * @param {Object} thisArg The object to bind function to
 * @return {Object} The resulting value of object a
 */
function extend(a, b, thisArg) {
  forEach(b, function assignValue(val, key) {
    if (thisArg && typeof val === 'function') {
      a[key] = bind(val, thisArg);
    } else {
      a[key] = val;
    }
  });
  return a;
}
```

> entend

1. 传入三个参数a, b, thisArg; a是结果对象，b是被复制的对象，thisArg是this指向

2. 使用

   ```javascript
   let a = { foo: 123 };
   let b = {
       foo: 456,
       log: function () {
           console.log('this指向', this)
       }
   };
   
   let c = {foo: 789}
   var result = extend(a, b, c);
   console.log(result); // { foo: 456, log: [Function: wrap] }
   console.log(a); // { foo: 456, log: [Function: wrap] }
   result.log(); // this指向 { foo: 789 }
   
   ```

## stripBom

> 删除UTF-8中的Bom

```javascript
/**
 * Remove byte order marker. This catches EF BB BF (the UTF-8 BOM)
 *
 * @param {string} content with BOM
 * @return {string} content value without BOM
 */
function stripBOM(content) {
  if (content.charCodeAt(0) === 0xFEFF) {
    content = content.slice(1);
  }
  return content;
}
```

> Bom ——  Byte Order Mark

1. 它是一个Unicode字符，通常出现在文本的开头，用来标识字节序。

2. UTF-8主要的优点是可以兼容ASCII，但如果使用BOM的话，这个好处就荡然无存了。

