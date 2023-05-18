---
title: JS数组操作
categories: JS
tags: 代码实战
img: https://picx.zhimg.com/v2-271693316513b5a0410ca80a2c160b5e_1440w.jpg

---


### 声明数组

```js
const fruits = new Array('Apple', 'Banana');
console.log(fruits.length);

// 通过数组字面量创建一个有2个元素的'fruits'数组.
const fruits = ['Apple', 'Banana'];
console.log(fruits.length);
```

### 数组去重

```js
// 方案一：Set + ...
function noRepeat(arr) {
  return [...new Set(arr)];
}
noRepeat([1, 2, 3, 1, 2, 3]);

// 方案二：Set + Array.from
function noRepeat(arr) {
  return Array.from(new Set(arr));
}
noRepeat([1, 2, 3, 1, 2, 3]);

// 方案三：双重遍历比对下标
function noRepeat(arr) {
  return arr.filter((v, idx) => idx == arr.lastIndexOf(v));
}
noRepeat([1, 2, 3, 1, 2, 3]);

// 方案四：单遍历 + Object特性
// Object的特性是Key不会重复。
// 这里使用values是因为可以保留类型，keys会变成字符串。
function noRepeat(arr) {
  return Object.values(
    arr.reduce((s, n) => {
      s[n] = n;
      return s;
    }, {})
  );
}
noRepeat([1, 2, 3, 1, 2, 3]);
function uniqueArr(arr){
    return [...new Set(arr)]
}

// 数组对象根据字段去重
// 参数：arr:要去重的数组 key:根据去重的字段名
export const uniqueArrayObject = (arr = [], key = 'id') => {
    if (arr.length === 0) return
    let list = []
    const map = {}
    arr.forEach((item) => {
        if (!map[item[key]]) {
            map[item[key]] = item
        }
    })
    list = Object.values(map)

    return list
}

// 示例：
const responseList = [
    { id: 1, name: '树哥' },
    { id: 2, name: '黄老爷' },
    { id: 3, name: '张麻子' },
    { id: 1, name: '黄老爷' },
    { id: 2, name: '张麻子' },
    { id: 3, name: '树哥' },
    { id: 1, name: '树哥' },
    { id: 2, name: '黄老爷' },
    { id: 3, name: '张麻子' },
]
uniqueArrayObject(responseList, 'id')
// [{ id: 1, name: '树哥' },{ id: 2, name: '黄老爷' },{ id: 3, name: '张麻子' }]

// 提取唯一值 我们可以使用Set对象和Spread运算符，创建一个剔除重复值的新数组。
var entries = [1, 2, 2, 3, 4, 5, 6, 6, 7, 7, 8, 4, 2, 1]
var unique_entries = [...new Set(entries)];
console.log(unique_entries);// [1, 2, 3, 4, 5, 6, 7, 8]
```

### 返回两个数组中相同的元素

```js
// 方案一：filter + includes
function intersection(arr1, arr2) {
  return arr2.filter((v) => arr1.includes(v));
}
intersection([1, 2, 3], [3, 4, 5, 2]);

// 方案二：同理变种用 hash
function intersection(arr1, arr2) {
  var set = new Set(arr2);
  return arr1.filter((v) => set.has(v));
}
intersection([1, 2, 3], [3, 4, 5, 2]);
```

### 对比两个数组并且返回其中不同的元素

```js
// 方案一：filter + includes
// 他原文有问题，以下方法的4,5没有返回

function diffrence(arrA, arrB) {
  return arrA.filter((v) => !arrB.includes(v));
}
diffrence([1, 2, 3], [3, 4, 5, 2]);
// 需要再操作一遍
function diffrence(arrA, arrB) {
  return arrA
    .filter((v) => !arrB.includes(v))
    .concat(arrB.filter((v) => !arrA.includes(v)));
}
diffrence([1, 2, 3], [3, 4, 5, 2]);

// 方案二：hash + 遍历
// 算是方案1的变种吧，优化了includes的性能。
```

### 检查数组中某元素出现的次数

```js
// 方案一：reduce
function countOccurrences(arr, value) {
  return arr.reduce((a, v) => (v === value ? a + 1 : a + 0), 0);
}
countOccurrences([1, 2, 3, 4, 5, 1, 2, 1, 2, 3], 1);

// 方案二：filter
function countOccurrences(arr, value) {
  return arr.filter((v) => v === value).length;
}
countOccurrences([1, 2, 3, 4, 5, 1, 2, 1, 2, 3], 1);
```

### 删除数组重复项

```js
const numberArrays = [undefined,Infinity,
  12,NaN,false,5,7,null,12,false,5,undefined,89,9,
  null,Infinity,5, NaN];
const objArrays = [{ id: 1 }, { id: 4 }, { id: 1 }, { id: 5 }, { id: 4 }];
console.log(
  // prints [undefined, Infinity, 12, NaN, false, 5, 7, null, 89, 9]
  Array.from(new Set(numberArrays)),
  // prints [{id: 1}, {id: 4}, {id: 1}, {id: 5}, {id: 4}]
  // nothing changes because even though the ids repeat in some objects
  // the objects are different instances, different objects
  Array.from(new Set(objArrays))
);
const idSet = new Set();
console.log(
  // prints [{id: 1}, {id: 4}, {id: 5}] using id to track id uniqueness
  objArrays.filter((obj) => {
    const existingId = idSet.has(obj.id);
    idSet.add(obj.id);
    return !existingId;
  })
);
```

### 查找数组最大

```js
// 方案一：Math.max + ...
function arrayMax(arr) {
  return Math.max(...arr);
}
arrayMax([-1, -4, 5, 2, 0]);

// 方案二：Math.max + apply
function arrayMax(arr) {
  return Math.max.apply(Math, arr);
}
arrayMax([-1, -4, 5, 2, 0]);

// 方案三：Math.max + 遍历
function arrayMax(arr) {
  return arr.reduce((s, n) => Math.max(s, n));
}
arrayMax([-1, -4, 5, 2, 0]);

// 方案四：比较、条件运算法 + 遍历
function arrayMax(arr) {
  return arr.reduce((s, n) => (s > n ? s : n));
}
arrayMax([-1, -4, 5, 2, 0]);

// 方案五：排序
function arrayMax(arr) {
  return arr.sort((n, m) => m - n)[0];
}
arrayMax([-1, -4, 5, 2, 0]);

```
### 查找数组最小

```js
// Math.max换成Math.min
// s>n?s:n换成s<n?s:n
// (n,m)=>m-n换成(n,m)=>n-m，或者直接取最后一个元素
```

### 返回以size为长度的数组分割的原数组

```js
// 方案一：Array.from + slice
function chunk(arr, size = 1) {
  return Array.from(
    {
      length: Math.ceil(arr.length / size),
    },
    (v, i) => arr.slice(i * size, i * size + size)
  );
}
chunk([1, 2, 3, 4, 5, 6, 7, 8], 3);

// 方案二：Array.from + splice
function chunk(arr, size = 1) {
  return Array.from(
    {
      length: Math.ceil(arr.length / size),
    },
    (v, i) => arr.splice(0, size)
  );
}
chunk([1, 2, 3, 4, 5, 6, 7, 8], 3);

// 方案三：遍历 + splice
function chunk(arr, size = 1) {
  var _returnArr = [];
  while (arr.length) {
    _returnArr.push(arr.splice(0, size));
  }
  return _returnArr;
}
chunk([1, 2, 3, 4, 5, 6, 7, 8], 3);
```

### 扁平化数组

```js
// 方案一：递归 + ...
function flatten(arr, depth = -1) {
  if (depth === -1) {
    return [].concat(
      ...arr.map((v) => (Array.isArray(v) ? this.flatten(v) : v))
    );
  }
  if (depth === 1) {
    return arr.reduce((a, v) => a.concat(v), []);
  }
  return arr.reduce(
    (a, v) => a.concat(Array.isArray(v) ? this.flatten(v, depth - 1) : v),
    []
  );
}
flatten([1, [2, [3]]]);

// 方案二：es6原生flat
function flatten(arr, depth = Infinity) {
  return arr.flat(depth);
}
flatten([1, [2, [3]]]);
```

### 从右删除n个元素
```js
// 方案一：slice
function dropRight(arr, n = 0) {
  return n < arr.length ? arr.slice(0, arr.length - n) : [];
}
dropRight([1, 2, 3, 4, 5], 2);

// 方案二: splice
function dropRight(arr, n = 0) {
  return arr.splice(0, arr.length - n);
}
dropRight([1, 2, 3, 4, 5], 2);

// 方案三: slice另一种
function dropRight(arr, n = 0) {
  return arr.slice(0, -n);
}
dropRight([1, 2, 3, 4, 5], 2);

// 方案四: 修改length
function dropRight(arr, n = 0) {
  arr.length = Math.max(arr.length - n, 0);
  return arr;
}
dropRight([1, 2, 3, 4, 5], 2);
```

### 截取第一个符合条件的元素及其以后的元素

```js
// 方案一：slice + 循环
function dropElements(arr, fn) {
  while (arr.length && !fn(arr[0])) arr = arr.slice(1);
  return arr;
}
dropElements([1, 2, 3, 4, 5, 1, 2, 3], (v) => v == 2);

// 方案二：findIndex + slice
function dropElements(arr, fn) {
  return arr.slice(Math.max(arr.findIndex(fn), 0));
}
dropElements([1, 2, 3, 4, 5, 1, 2, 3], (v) => v === 3);

// 方案三：splice + 循环
function dropElements(arr, fn) {
  while (arr.length && !fn(arr[0])) arr.splice(0, 1);
  return arr;
}
dropElements([1, 2, 3, 4, 5, 1, 2, 3], (v) => v == 2);
```

### 返回数组中下标间隔nth的元素

```js
// 方案一：filter
function everyNth(arr, nth) {
  return arr.filter((v, i) => i % nth === nth - 1);
}
everyNth([1, 2, 3, 4, 5, 6, 7, 8], 2);

// 方案二：方案一修改判断条件
function everyNth(arr, nth) {
  return arr.filter((v, i) => (i + 1) % nth === 0);
}
everyNth([1, 2, 3, 4, 5, 6, 7, 8], 2);
```

### 返回数组中第n个元素（支持负数）

```js
// 方案一：slice
function nthElement(arr, n = 0) {
  return (n >= 0 ? arr.slice(n, n + 1) : arr.slice(n))[0];
}
nthElement([1, 2, 3, 4, 5], 0);
nthElement([1, 2, 3, 4, 5], -1);

// 方案二：三目运算符
function nthElement(arr, n = 0) {
  return n >= 0 ? arr[0] : arr[arr.length + n];
}
nthElement([1, 2, 3, 4, 5], 0);
nthElement([1, 2, 3, 4, 5], -1);
```

### 返回数组头元素
```js
// 方案一：
function head(arr) {
  return arr[0];
}
head([1, 2, 3, 4]);

// 方案二：
function head(arr) {
  return arr.slice(0, 1)[0];
}
head([1, 2, 3, 4]);
```

### 返回数组末尾元素

```js
// 方案一：
function last(arr) {
  return arr[arr.length - 1];
}

// 方案二：
function last(arr) {
  return arr.slice(-1)[0];
}
last([1, 2, 3, 4, 5]);
```

### 数组乱排

```js
// 方案一：洗牌算法
function shuffle(arr) {
  let array = arr;
  let index = array.length;

  while (index) {
    index -= 1;
    let randomInedx = Math.floor(Math.random() * index);
    let middleware = array[index];
    array[index] = array[randomInedx];
    array[randomInedx] = middleware;
  }

  return array;
}
shuffle([1, 2, 3, 4, 5]);

/**
 * 方案二：sort + random
 */
function shuffle(arr) {
  return arr.sort((n, m) => Math.random() - 0.5);
}
shuffle([1, 2, 3, 4, 5]);
```

### 伪数组转换为数组
```js
// Array.from将伪数组变成数组，就是只要有length的属性就可以转成数组
Array.from({ length: 2 });
let name = "javascript";
console.log(name.length); // 10
let arr = Array.from(name);
console.log(arr); // [ 'j', 'a', 'v', 'a', 's', 'c', 'r', 'i', 'p', 't' ]
// prototype.slice
Array.prototype.slice.call({ length: 2, 1: 1 });
// prototype.splice
Array.prototype.splice.call({ length: 2, 1: 1 }, 0);
// Array.of()将一组值转换成数组，类似于声明数组
let arr = Array.of(10);
let arr2 = Array.of("hello", "world");
console.log(arr); // [ 10 ] 
console.log(arr2); // [ 'hello', 'world' ]
```

### 数组重排序

```js
const shuffle = (arr) => arr.sort(() => Math.random() - 0.5)
const arr = [1, 2, 3, 4, 5]
console.log(shuffle(arr))
```

### 数组随机打乱顺序

通过0.5-Math.random()得到一个随机数，再通过两次sort排序打乱的更彻底,但是这个方法实际上并不够随机，如果是企业级运用，建议使用第二种洗牌算法

```js
shuffle(arr) {
      return arr.sort(() => 0.5 - Math.random()). sort(() => 0.5 - Math.random());
 },

function shuffle(arr) {
  for (let i = arr.length - 1; i > 0; i--) {
    const randomIndex = Math.floor(Math.random() * (i + 1))
    ;[arr[i], arr[randomIndex]] = [arr[randomIndex], arr[i]]
  }
  return arr
}
```
### 把数组最后一项移到第一项

```js
function (arr){
    return arr.push(arr.shift());
}
```

### 把数组的第一项放到最后一项

```js
function(arr){
  return arr.unshift(arr.pop());
}
```

### 各种数组克隆方法
数组克隆的方法其实特别多了，看看有没有你没见过的！

```js
const clone = (arr) => arr.slice(0);
const clone = (arr) => [...arr];
const clone = (arr) => Array.from(arr);
const clone = (arr) => arr.map((x) => x);
const clone = (arr) => JSON.parse(JSON.stringify(arr));
const clone = (arr) => arr.concat([]);
const clone = (arr) => structuredClone(arr);
```

### 交换数组值的位置
```js
const array = [12, 24, 48];
const swap0ldway = (arr, i, j) => {
  const arrayCopy = [...arr];
  let temp = arayCopy[i];
  arrayCopy[i] = arrayCopy[j];

  arrayCopy[j] = temp;
  return arrayCopy;
};

const swapNewWay = (arr, i, j) => {
  const arrayCopy = [...arr];
  [arrayCopy[0], arrayCopy[2]] = [arrayCopy[2], arrayCopy[0]];
  return arrayCopy;
};

console.log(swap0ldWay(array, 0, 2)); // outputs: [48, 24, 12]
console.log(swapNewWay(array, 0, 2)); // outputs: [48, 24, 12]
```

### 随机排列数组中的元素
```js
var my_list = [1, 2, 3, 4, 5, 6, 7, 8, 9];
console.log(my_list.sort(function() {
    return Math.random() - 0.5
}));// [4, 8, 2, 9, 1, 3, 6, 5, 7]
```
### 展平多维数组
```js
var entries = [1, [2, 5], [6, 7], 9];
var flat_entries = [].concat(...entries);// [1, 2, 5, 6, 7, 9]
```

### 使用length调整大小 / 清空数组
```js
// 如果我们要调整数组的大小：
var entries = [1, 2, 3, 4, 5, 6, 7];
console.log(entries.length);// 7
entries.length = 4;
console.log(entries.length);// 4
console.log(entries);// [1, 2, 3, 4]
// 如果我们要清空数组：
var entries = [1, 2, 3, 4, 5, 6, 7];
console.log(entries.length);// 7
entries.length = 0;
console.log(entries.length);// 0
console.log(entries);// []
```

### 栈方法

后进先出

#### push()

可以接收任意数量的参数，把它们逐个添加到数组末尾，并返回修改后数组的长度

```js
let arr = [1, 2, 3];
arr.push(4);
console.log(arr); // [ 1, 2, 3, 4 ]
console.log(arr.length); // 4
```

#### pop()

从数组末尾移除最后一项，减少数组的length值，然后返回移除的项

```js
let arr = [1, 2, 3];
let delVal = arr.pop();
console.log(arr); // [ 1, 2]
console.log(delVal); // 3
```

### 队列方法

先进先出

#### shift()

移除数组中的第一个项并返回该项，同时将数组长度减1

```js
let arr = [1, 2, 3];
let delVal = arr.shift();
console.log(delVal); // 1
console.log(arr); // [ 2, 3 ]
console.log(arr.length); // 2
```

#### unshift()

在数组前端添加任意个项并返回新数组的长度

```js
let arr = [1, 2, 3];
let arrLength = arr.unshift(0);
console.log(arrLength); // 4
console.log(arr); // [ 0, 1, 2, 3 ]
```

### 排序方法

#### reverse()

反转数组项的顺序

```js
let arr = [1, 2, 3];
arr.reverse();
console.log(arr); // [ 3, 2, 1 ]
```

#### sort()

从小到大排序，但它的排序方法是根据数组转换字符串后来排序的

```js
let arr = [1, 5, 10, 15];
console.log(arr.sort()); // [ 1, 10, 15, 5 ] 原因：它们比较的是转换的字符串值
// 从小到大排序
console.log(arr.sort(compare)); // [ 1, 5, 10, 15 ]
function compare(value1, value2) {
  if (value1 < value2) {
    return -1;
  } else if (value1 > value2) {
    return 1;
  } else {
    return 0;
  }
}
```

### 操作方法

#### join()

JavaScript数组中的join()方法是一个内置方法，通过连接数组的所有元素来创建并返回新字符串。join()方法将连接数组的项到字符串并返回该字符串。指定的分隔符用于分隔元素数组。默认分隔符是逗号(,)。

```js
const elements = ['Fire', 'Air', 'Water'];
console.log(elements.join());
// expected output: "Fire,Air,Water"
console.log(elements.join(''));
// expected output: "FireAirWater"
console.log(elements.join('-'));
// expected output: "Fire-Air-Water"
```

#### concat()

可以基于当前数组中的所有项创建一个新数组，不会影响原数组的值

```js
let arr = [1, 2, 3];
let newArr = arr.concat([4, 5, 6], [7, 8, 9]);
console.log(newArr); // [ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
console.log(arr); // [1, 2, 3]
const array1 = ['a', 'b', 'c'];
const array2 = ['d', 'e', 'f'];
const array3 = array1.concat(array2);
```

#### slice()

它能够基于当前数组中的一或多个项创建一个新数组

- slice()方法可以接受一或两个参数，即要返回项的起始和结束位置
- 在只有一个参数的情况下，slice()方法返回从该参数指定位置开始到当前数组末尾的所有项。
- 如果有两个参数，该方法返回起始和结束位置之间的项——但不包括结束位置的项。
- 注意，slice()方法不会影响原始数组

```js
let arr = [1, 2, 3, 4];
let newArr = arr.slice(1, 2);
console.log(newArr); // [ 2 ]
let newArr2 = arr.slice(1);
console.log(newArr2); // [ 2, 3, 4 ]
```

#### splice()

##### 删除

可以删除任意数量的项，只需指定2个参数：要删除的第一项的位置和要删除的项数。例如，splice(0,2)会删除数组中的前两项。

```js
let arr = [1, 2, 3, 4];
arr.splice(1, 2);
console.log(arr); // [ 1, 4 ]
```

##### 插入

可以向指定位置插入任意数量的项，只需提供3个参数：起始位置、0（要删除的项数）和要插入的项。如果要插入多个项，可以再传入第四、第五，以至任意多个项。例如，splice(2,0,“red”,“green”)会从当前数组的位置2开始插入字符串"red"和"green"。

```js
let arr = [1, 2, 3, 4];
arr.splice(1, 0, "java", "script");
console.log(arr); // [ 1, 'java', 'script', 2, 3, 4 ]
```

##### 替换

可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定3个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项数不必与删除的项数相等。例如，splice(2,1,“red”,“green”)会删除当前数组位置2的项，然后再从位置2开始插入字符串"red"和"green"。

```js
let arr = [1, 2, 3, 4];
arr.splice(1, 1, "java", "script");
console.log(arr); // [ 1, 'java', 'script', 3, 4 ]
```

#### arr.fill(target,start,end)

使用给定的值，填充一个数组,ps:填充完后会改变原数组

- target – 待填充的元素
- start – 开始填充的位置-索引
- end – 终止填充的位置-索引(不包括该位置)

```js
let arr = [1, 2, 3, 4];
let arr2 = [5, 6, 7, 8];
// 全部填充5
arr.fill(5);
console.log(arr); // [ 5, 5, 5, 5 ]
// 从索引为1到3填充9
arr2.fill(9, 1, 3);
console.log(arr2); // [ 5, 9, 9, 8 ]
```

#### Array.isArray(arr)

判断传入的值是否为数组

```js
let arr = [];
let obj = {};
console.log(Array.isArray(arr)); // true
console.log(Array.isArray(obj)); // false
```

在当前数组内部，将制定位置的数组复制到其他位置，会覆盖原数组项，返回当前数组
参数:

- target --必选 索引从该位置开始替换数组项
- start --可选 索引从该位置开始读取数组项，默认为0，如果为负值，则从右往左读。
- end --可选 索引到该位置停止读取的数组项，默认是Array.length,如果是负值，表示倒数

```js
let arr = [1, 2, 3, 4, 5, 6];

console.log(arr.copyWithin(2, 0)); // [1, 2, 1, 2, 3, 4]
console.log(arr.copyWithin(2, 0, 4)); // [ 1, 2, 1, 2, 1, 2 ]
```

### 位置方法

#### at()

```js
const array1 = [5, 12, 8, 130, 44];
let index = 2;
console.log(Using an index of ${index} the item returned is ${array1.at(index)});
// expected output: "Using an index of 2 the item returned is 8"
index = -2;
console.log(Using an index of ${index} item returned is ${array1.at(index)});
```

#### indexOf()和lastIndexOf()

这两个方法都接收两个参数：要查找的项和（可选的）表示查找起点位置的索引。其中，indexOf()方法从数组的开头（位置0）开始向后查找，lastIndexOf()方法则从数组的末尾开始向前查找。

```js
let arr = [1, 2, 3, 2, 1];
// 从0开始查询值为2的位置
console.log(arr.indexOf(2)); // 1
// 从索引为2开始查询值为2的位置
console.log(arr.indexOf(2, 2)); // 3
// 倒叙查询值为2的位置
console.log(arr.lastIndexOf(2)); // 3
// 倒叙查询值为2的位置
console.log(arr.lastIndexOf(2, 2)); // 1
```

#### find()

数组实例的find方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为true的成员，然后返回该成员。如果没有符合条件的成员，则返回undefined。

```js
let arr = [1, 2, 3, 4, 5, 6];
let index = arr.find(val => val === 3);
let index2 = arr.find(val => val === 100);
console.log(index); // 3
console.log(index2); // undefined
const array1 = [5, 12, 8, 130, 44];
const found = array1.find(element => element > 10);
console.log(found);
```

#### findIndex()

和数组实例的findIndex方法的用法与find方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回-1。

```js
let arr = [1, 2, 3, 4, 5, 6];
let index = arr.findIndex(val => val === 3);
let index2 = arr.findIndex(val => val === 100);

console.log(index); // 2
console.log(index2); // -1
```

#### includes()

方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的includes方法类似。

```js
let arr = [1, 2, 3, 4, 5, 6];

console.log(arr.includes(3)); // true
console.log(arr.includes(100)); // false
```

### 迭代方法

#### every()

对数组中的每一项运行给定函数，如果该函数对每一项都返回true，则返回true。

```js
let arr = [1, 2, 3, 4, 5, 6];
// 是否所有的值都大于3
let isTrue = arr.every(value => value > 3);
console.log(isTrue); // false;
const isBelowThreshold = (currentValue) => currentValue < 40;
const array1 = [1, 30, 39, 29, 10, 13];
console.log(array1.every(isBelowThreshold));
```

#### filter()

对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。

```js
let arr = [1, 2, 3, 4, 5, 6];
// 取数组中大于3的值重新组成新数组
let newArr = arr.filter(value => value > 3);
console.log(newArr); // [ 4, 5, 6 ]
const words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];
const result = words.filter(word => word.length > 6);
console.log(result);
```

#### forEach()

对数组中的每一项运行给定函数。这个方法没有返回值。

```js
let arr = [1, 2, 3, 4, 5, 6];
// 迭代数组的每一项
arr.forEach((item, index) => {
  console.log(item); // 1, 2, 3, 4, 5, 6
})
arr.forEach(element => console.log(element));
```

#### map()

对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。

```js
let arr = [1, 2, 3, 4, 5, 6];
// 迭代数组每个值加上100返回新数组
let newArr = arr.map(val => val + 100);
console.log(newArr); // [ 101, 102, 103, 104, 105, 106 ]
let modifiedArr = arr.map(function(element){
    return element *3;
});
```

#### some()

对数组中的每一项运行给定函数，如果该函数对任一项返回true，则返回true。

```js
let arr = [1, 2, 3, 4, 5, 6];
// 迭代数组的每一项，只要有一项符合条件就返回true
let isTrue = arr.some(val => val >= 5);
let isTrue2 = arr.some(val => val > 6);
console.log(isTrue); // true
console.log(isTrue2); // false
const array = [1, 2, 3, 4, 5];
// checks whether an element is even
const even = (element) => element % 2 === 0;
console.log(array.some(even));
// expected output: true
```

#### reduce()和reduceRight()

这两个方法都会迭代数组的所有项，然后构建一个最终返回的值。其中，reduce()方法从数组的第一项开始，逐个遍历到最后。而reduceRight()则从数组的最后一项开始，向前遍历到第一项。

```js
let arr = [1, 2, 3, 4];
// 从左到右累加结果
let result = arr.reduce((val1, val2) => {
  return val1 + val2;
});
console.log(result); // 10
```

#### entries()，keys()和values()

ES6提供三个新的方法——entries()，keys()和values()——用于遍历数组。它们都返回一个遍历器对象，可以用for…of循环进行遍历，唯一的区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

```js
let arr = [1, 2, 3];
// entries()是对键值对的遍历
for (let val of arr.entries()) {
  console.log(val);
  /**
   [ 0, 1 ]
   [ 1, 2 ]
   [ 2, 3 ]
   */
}
// keys()是对键名的遍历
for (let val of arr.keys()) {
  console.log(val); // 0 1 2
}
// values()是对键值的遍历
for (let val of arr.values()) {
  console.log(val); // 1 2 3
}
```