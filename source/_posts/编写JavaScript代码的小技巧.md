---
title: 编写JavaScript代码的小技巧
categories: JS
tags: 代码实战
img: https://pic1.zhimg.com/v2-6ef93db4040b0eb0b4524d06e372ad0f.jpg

---

## 12个非常实用的JavaScript函数

### 生成随机颜色

```js
const generateRandomHexColor = () => Math.floor(Math.random() * 0xffffff).toString(16)
console.log(generateRandomHexColor())
```

### 数组重排序

```js
const shuffle = (arr) => arr.sort(() => Math.random() - 0.5)
const arr = [1, 2, 3, 4, 5]
console.log(shuffle(arr))
```

### 复制到剪切板

```js
const copyToClipboard = (text) => navigator.clipboard && navigator.clipboard.writeText && navigator.clipboard.writeText(text)
copyToClipboard("Hello World!")
```

### 检测暗色主题

```js
const isDarkMode = () => window.matchMedia && window.matchMedia("(prefers-color-scheme: dark)").matches;
console.log(isDarkMode())
```

### 滚动到顶部

```js
// 将元素滚动到顶部最简单的方法是使用scrollIntoView。设置block为start可以滚动到顶部；设置behavior为smooth可以开启平滑滚动。
const scrollToTop = (element) => element.scrollIntoView({ behavior: "smooth", block: "start" });
```

### 滚动到底部

```js
// 与滚动到顶部一样，滚动到底部只需要设置block为end即可。
const scrollToBottom = (element) =>  element.scrollIntoView({ behavior: "smooth", block: "end" });
```

### 检测元素是否在屏幕中
检查元素是否在窗口中最好的方法是使用IntersectionObserver。

```js
const callback = (entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      // `entry.target` is the dom element
      console.log(`${entry.target.id} is visible`);
    }
  });
};

const options = {
  threshold: 1.0,
};
const observer = new IntersectionObserver(callback, options);
const btn = document.getElementById("btn");
const bottomBtn = document.getElementById("bottom-btn");
observer.observe(btn);
observer.observe(bottomBtn);
```

### 检测设备

```js
// 使用navigator.userAgent来检测网站运行在哪种平台设备上。
const detectDeviceType = () =>
  /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
    navigator.userAgent
  ) ? "Mobile" : "Desktop";
console.log(detectDeviceType());
```

### 隐藏元素

```js
// 我们可以将元素的style.visibility设置为hidden，隐藏元素的可见性，但元素的空间仍然会被占用。如果设置元素的style.display为none，会将元素从渲染流中删除。
const hideElement = (el, removeFromFlow = false) => {
  removeFromFlow ? (el.style.display = 'none')
  : (el.style.visibility = 'hidden')
}
```

### 从URL中获取参数

```js
// JavaScript中有一个URL对象，通过它可以非常方便的获取URL中的参数。
const getParamByUrl = (key) => {
  const url = new URL(location.href)
  return url.searchParams.get(key)
}
```

### 深拷贝对象

```js
// 深拷贝对象非常简单，先将对象转换为字符串，再转换成对象即可。
const deepCopy = obj => JSON.parse(JSON.stringify(obj))
// 除了利用JSON的API，还有更新的深拷贝对象的structuredClone API，但并不是在所有的浏览器中都支持。
structuredClone(obj)
```

### 等待函数

```js
// JavaScript提供了setTimeout函数，但是它并不返回Promise对象，所以我们没办法使用async作用在这个函数上，但是我们可以封装等待函数。
const wait = (ms) => new Promise((resolve)=> setTimeout(resolve, ms))
const asyncFn = async () => {
  await wait(1000)
  console.log('等待异步函数执行结束')
}
asyncFn()
```


## 7个你必须知道的JavaScript简写技巧

### 多个字符串检查

通常，如果我们需要检查字符串是否等于多个值中的一个，往往很快会觉得疲惫不堪。幸运的是，JavaScript有一个内置的方法来帮助你解决这个问题

```js
// 普通写法
const isVowel = (letter) => {
  if (
    letter === "a" ||
    letter === "e" ||
    letter === "i" ||
    letter === "o" ||
    letter === "u"
  ) {
    return true;
  }
  return false;
};

// 简写方法
const isVowel = (letter) =>
  ["a", "e", "i", "o", "u"].includes(letter);
```

### For-of和For-in循环

For-of和For-in循环是迭代array或object的好方法，因为无需手动跟踪object键的索引。

**For-of**
```js
const arr = [1, 2, 3, 4, 5];
// 普通写法
for (let i = 0; i < arr.length; i++) {
  const element = arr[i];
  // ...
}
// 简写方法
for (const element of arr) {
  // ...
}
```

**For-in**

```js
const obj = {
  a: 1,
  b: 2,
  c: 3,
};
// 普通写法
const keys = Object.keys(obj);
for (let i = 0; i < keys.length; i++) {
  const key = keys[i];
  const value = obj[key];
  // ...
}
// 简写方法
for (const key in obj) {
  const value = obj[key];
  // ...
}
```

### Falsey（假值）检查

如果要检查变量是null、undefined、0、false、NaN还是空string，可以使用逻辑非(!)运算符一次检查所有变量，而无需编写多个条件。这使得检查变量是否包含有效数据变得相对容易多了。

```js
// 普通写法
const isFalsey = (value) => {
  if (
    value === null ||
    value === undefined ||
    value === 0 ||
    value === false ||
    value === NaN ||
    value === ""
  ) {
    return true;
  }
  return false;
};

// 简写方法
const isFalsey = (value) => !value;
```

### 三元运算符

作为JavaScript开发人员，你一定遇到过三元运算符。这是编写简洁if-else语句的好方法。但是，也可用来编写简洁的代码，甚至将它们链接起来来检查多个条件。

```js
// 普通写法
let info;
if (value < minValue) {
  info = "Value is too small";
} else if (value > maxValue) {
  info = "Value is too large";
} else {
  info = "Value is in range";
}

// 简写方法
const info =
  value < minValue
    ? "Value is too small"
    : value > maxValue ? "Value is too large" : "Value is in range";
```

### 函数调用

在三元运算符的帮助下，你还可以根据条件确定要调用哪个函数。

> 注：函数的call signature必须相同，否则可能会遇到错误。

```js
function f1() {
  // ...
}
function f2() {
  // ...
}
// 普通写法
if (condition) {
  f1();
} else {
  f2();
}
// 简写方法
(condition ? f1 : f2)();
```

### Switch简写

通常我们可以使用以键作为switch条件并将值作为返回值的对象来优化长switch语句。

```js
const dayNumber = new Date().getDay();
// 普通写法
let day;
switch (dayNumber) {
  case 0:
    day = "Sunday";
    break;
  case 1:
    day = "Monday";
    break;
  case 2:
    day = "Tuesday";
    break;
  case 3:
    day = "Wednesday";
    break;
  case 4:
    day = "Thursday";
    break;
  case 5:
    day = "Friday";
    break;
  case 6:
    day = "Saturday";
}
// 简写方法
const days = {
  0: "Sunday",
  1: "Monday",
  2: "Tuesday",
  3: "Wednesday",
  4: "Thursday",
  5: "Friday",
  6: "Saturday",
};
const day = days[dayNumber];
```

### 回退值

||运算符可以为变量设置回退值。

```js
// 普通写法
let name;
if (user?.name) {
  name = user.name;
} else {
  name = "Anonymous";
}
// 简写方法
const name = user?.name || "Anonymous";
```

## 30个常用JavaScript知识点总结

### 一行代码完成结构加赋值

```js
let people = { name: null, age: null };
let result = { name: '张三',  age: 16 };
({ name: people.name, age: people.age} = result);
console.log(people) // {"name":"张三","age":16}
```

### 对基础数据类型进行解构

```js
const {length : a} = '1234';
console.log(a) // 4
```

### 对数组解构快速拿到最后一项值

```js
const arr = [1, 2, 3];
const { 0: first, length, [length - 1]: last } = arr;
first; // 1
last; // 3
length; // 3
```

### 将下标转为中文零一二三...

日常可能有的列表我们需要将对应的012345转为中文的一、二、三、四、五...，在老的项目看到还有通过自己手动定义很多行这样的写法，于是写了一个这样的方法转换

```js
export function transfromNumber(number){
  const  INDEX_MAP = ['零'，'一'.....]
  if(!number) return
  if(number === 10) return INDEX_MAP[number]
  return [...number.toString()].reduce( (pre, cur) => pre  + INDEX_MAP[cur] , '' )
}
```

### 判断整数的不同方法

```js
/* 1.任何整数都会被1整除，即余数是0。利用这个规则来判断是否是整数。但是对字符串不准确 */
function isInteger(obj) {
 return obj%1 === 0
}
/* 2. 添加一个是数字的判断 */
function isInteger(obj) {
 return typeof obj === 'number' && obj%1 === 0
}
/* 3. 使用Math.round、Math.ceil、Math.floor判断整数取整后还是等于自己。利用这个特性来判断是否是整数*/
function isInteger(obj) {
 return Math.floor(obj) === obj
}
/* 4. 通过parseInt判断 某些场景不准确 */
function isInteger(obj) {
 return parseInt(obj, 10) === obj
}
/* 5. 通过位运算符*/
function isInteger(obj) {
 return (obj | 0) === obj
}
```

### 通过css检测系统的主题色从而全局修改样式

@media的属性prefers-color-scheme就可以知道当前的系统主题，当然使用前需要查查兼容性

```css
@media (prefers-color-scheme: dark) { //... } 
@media (prefers-color-scheme: light) { //... }
```
javascript也可以轻松做到

```js
window.addEventListener('theme-mode', event =>{ 
    if(event.mode == 'dark'){}
   if(event.mode == 'light'){} 
})
window.matchMedia('(prefers-color-scheme: dark)') .addEventListener('change', event => { 
    if (event.matches) {} // dark mode
})
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
### 随机获取一个Boolean值
和上个原理相同，通过随机数获取，Math.random()的区间是0-0.99，用0.5在中间百分之五十的概率

```js
function randomBool() {
    return 0.5 - Math.random()
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

### 利用set数组去重

```js
function uniqueArr(arr){
    return [...new Set(arr)]
}
```
### dom节点平滑滚动到可是区域，顶部，底部

原生的scrollTo方法没有动画，类似于锚点跳转，比较生硬，可以通过这个方法会自带平滑的过度效果

```js
function scrollTo(element) {
    element.scrollIntoView({ behavior: "smooth", block: "start" }) // 顶部
    element.scrollIntoView({ behavior: "smooth", block: "end" }) // 底部
    element.scrollIntoView({ behavior: "smooth"}) // 可视区域
}
```

### 获取随机颜色
日常我们经常会需要获取一个随机颜色，通过随机数即可完成

```js
function getRandomColor(){
    return `#${Math.floor(Math.random() * 0xffffff) .toString(16)}`;
}
```

### 检测是否为空对象

通过使用Es6的Reflect静态方法判断他的长度就可以判断是否是空数组了，也可以通过Object.keys()来判断

```js
function isEmpty(obj){
    return  Reflect.ownKeys(obj).length === 0 && obj.constructor === Object;
}
```


### Boolean转换
一些场景下我们会将boolean值定义为场景，但是在js中非空的字符串都会被认为是true

```js
function toBoolean(value, truthyValues = ['true']){
  const normalizedValue = String(value).toLowerCase().trim();
  return truthyValues.includes(normalizedValue);
}
toBoolean('TRUE'); // true
toBoolean('FALSE'); // false
toBoolean('YES', ['yes']); // true
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


### 比较两个时间大小
通过调用getTime获取时间戳比较就可以了

```js
function compare(a, b){
    return a.getTime() > b.getTime();
}
```


### 计算两个时间之间的月份差异

```js
function monthDiff(startDate, endDate){
    return  Math.max(0, (endDate.getFullYear() - startDate.getFullYear()) * 12 - startDate.getMonth() + endDate.getMonth());
}
```
### 一步从时间中提取年月日时分秒
时间格式化轻松解决，一步获取到年月日时分秒毫秒，由于toISOString会丢失时区，导致时间差八小时，所以在格式化之前我们加上八个小时时间即可

```js
function extract(date){
   const d = new Date(new Date(date).getTime() + 8*3600*1000);
  return new Date(d).toISOString().split(/[^0-9]/).slice(0, -1);
}
console.log(extract(new Date())) // ['2022', '09', '19', '18', '06', '11', '187']
```


### 判断一个参数是不是函数

有时候我们的方法需要传入一个函数回调，但是需要检测其类型，我们可以通过Object的原型方法去检测，当然这个方法可以准确检测任何类型。

```js
function isFunction(v){
   return ['[object Function]', '[object GeneratorFunction]', '[object AsyncFunction]', '[object Promise]'].includes(Object.prototype.toString.call(v));
}
```


### 计算两个坐标之间的距离

```js
function distance(p1, p2){
    return Math.sqrt(Math.pow(p2.x - p1.x, 2) + Math.pow(p2.y - p1.y, 2));
}
```

### 检测两个dom节点是否覆盖重叠

有些场景下我们需要判断dom是否发生碰撞了或者重叠了，我们可以通过getBoundingClientRect获取到dom的x1,y1,x2,y2坐标然后进行坐标比对即可判断出来

```js
function overlaps = (a, b) {
   return (a.x1 < b.x2 && b.x1 < a.x2) || (a.y1 < b.y2 && b.y1 < a.y2);
}
```

### 判断是否是NodeJs环境

前端的日常开发是离不开nodeJs的，通过判断全局环境来检测是否是nodeJs环境

```js
function isNode(){
    return typeof process !== 'undefined' && process.versions != null && process.versions.node != null;
}
```

### 参数求和

之前看到有通过函数柯理化形式来求和的，通过reduce一行即可

```js
function sum(...args){
    args.reduce((a, b) => a + b);
}
```

## 你应该了解的25个JS技巧

### 类型检查小工具
```js
const isOfType = (() => {
  // create a plain object with no prototype
  const type = Object.create(null);
  // check for null type
  type.null = (x) => x === null;
  // check for undefined type
  type.undefined = (x) => x === undefined;
  // check for nil type. Either null or undefined
  type.nil = (x) => type.null(x) || type.undefined(x);
  // check for strings and string literal type. e.g: 's', "s", `str`, new String()
  type.string = (x) => !type.nil(x) && (typeof x === "string" || x instanceof String);
  // check for number or number literal type. e.g: 12, 30.5, new Number()
  type.number = (x) =>
!type.nil(x) && // NaN & Infinity have typeof "number" and this excludes that
    ((!isNaN(x) && isFinite(x) && typeof x === "number") ||
      x instanceof Number);
  // check for boolean or boolean literal type. e.g: true, false, new Boolean()
  type.boolean = (x) =>
    !type.nil(x) && (typeof x === "boolean" || x instanceof Boolean);
  // check for array type
  type.array = (x) => !type.nil(x) && Array.isArray(x);
  // check for object or object literal type. e.g: {}, new Object(), Object.create(null)
  type.object = (x) => ({}.toString.call(x) === "[object Object]");
  // check for provided type instance
  type.type = (x, X) => !type.nil(x) && x instanceof X;
  // check for set type
  type.set = (x) => type.type(x, Set);
  // check for map type
  type.map = (x) => type.type(x, Map);
  // check for date type
  type.date = (x) => type.type(x, Date);

  return type;
})();
```

### 检查是否为空
```js
function isEmpty(x) {
  if (Array.isArray(x) || typeof x === "string" || x instanceof String) {
    return x.length === 0;
  }
  if (x instanceof Map || x instanceof Set) {
    return x.size === 0;
  }
  if ({}.toString.call(x) === "[object Object]") {
    return Object.keys(x).length === 0;
  }

  return false;
}
```

### 获取列表最后一项
```js
function lastItem(list) {
  if (Array.isArray(list)) {
    return list.slice(-1)[0];
  }

  if (list instanceof Set) {
    return Array.from(list).slice(-1)[0];
  }

  if (list instanceof Map) {
    return Array.from(list.values()).slice(-1)[0];
  }
}
```

### 带有范围的随机数生成器
```js
function randomNumber(max = 1, min = 0) {
  if (min >= max) {
    return max;
  }

  return Math.floor(Math.random() * (max - min) + min);
}
```

### 随机ID生成器
```js
// create unique id starting from current time in milliseconds
// incrementing it by 1 everytime requested
const uniqueId = (() => {
  const id = (function* () {
    let mil = new Date().getTime();

    while (true) yield (mil += 1);
  })();

  return () => id.next().value;
})();
// create unique incrementing id starting from provided value or zero
// good for temporary things or things that id resets
const uniqueIncrementingId = ((lastId = 0) => {
  const id = (function* () {
    let numb = lastId;

    while (true) yield (numb += 1);
  })();

  return (length = 12) => `${id.next().value}`.padStart(length, "0");
})();
// create unique id from letters and numbers
const uniqueAlphaNumericId = (() => {
  const heyStack = "0123456789abcdefghijklmnopqrstuvwxyz";
  const randomInt = () =>
    Math.floor(Math.random() * Math.floor(heyStack.length));

  return (length = 24) =>
    Array.from({ length }, () => heyStack[randomInt()]).join("");
})();
```

### 创建一个范围内的数字
```js
function range(maxOrStart, end = null, step = null) {
  if (!end) {
    return Array.from({ length: maxOrStart }, (_, i) => i);
  }

  if (end <= maxOrStart) {
    return [];
  }

  if (step !== null) {
    return Array.from(
      { length: Math.ceil((end - maxOrStart) / step) },
      (_, i) => i * step + maxOrStart
    );
  }

  return Array.from(
    { length: Math.ceil(end - maxOrStart) },
    (_, i) => i + maxOrStart
  );
}
```

### 格式化JSON字符串，stringify任何内容
```js
const stringify = (() => {
  const replacer = (key, val) => {
    if (typeof val === "symbol") {
      return val.toString();
    }
    if (val instanceof Set) {
      return Array.from(val);
    }
    if (val instanceof Map) {
      return Array.from(val.entries());
    }
    if (typeof val === "function") {
      return val.toString();
    }
    return val;
  };

  return (obj, spaces = 0) => JSON.stringify(obj, replacer, spaces);
})();
```

### 顺序执行promise
```js
const asyncSequentializer = (() => {
  const toPromise = (x) => {
    if (x instanceof Promise) {
      // if promise just return it
      return x;
    }

    if (typeof x === "function") {
      // if function is not async this will turn its result into a promise
      // if it is async this will await for the result
      return (async () => await x())();
    }
    
    return Promise.resolve(x);
  };

  return (list) => {
    const results = [];

    return (
      list.reduce((lastPromise, currentPromise) => {
          return lastPromise.then((res) => {
            results.push(res); // collect the results
            return toPromise(currentPromise);
          });
        }, toPromise(list.shift()))
        // collect the final result and return the array of results as resolved promise
        .then((res) => Promise.resolve([...results, res]))
    );
  };
})();
```

### 轮询数据
```js
async function poll(fn, validate, interval = 2500) {
  const resolver = async (resolve, reject) => {
    try {
      // catch any error thrown by the "fn" function
      const result = await fn(); // fn does not need to be asynchronous or return promise
      // call validator to see if the data is at the state to stop the polling
      const valid = validate(result);
      if (valid === true) {
        resolve(result);
      } else if (valid === false) {
        setTimeout(resolver, interval, resolve, reject);
      } // if validator returns anything other than "true" or "false" it stops polling
    } catch (e) {
      reject(e);
    }
  };
  return new Promise(resolver);
}
```

### 等待所有promise完成
```js
const prom1 = Promise.reject(12);
const prom2 = Promise.resolve(24);
const prom3 = Promise.resolve(48);
const prom4 = Promise.resolve("error");
// completes when all promises resolve or at least one fail
// if all resolve it will return an array of results in the same order of each promise
// if fail it will return the error in catch

Promise.all([prom1, prom2, prom3, prom4])
  .then((res) => console.log("all", res))
  .catch((err) => console.log("all failed", err));

// completes with an array of objects with "status" and "value" or "reason" of each promise
// status can be "fullfilled" or "rejected"
// if fullfilled it will contain a "value" property

// if failed it will contain a "reasor property
Promise.allSettled([prom1, prom2, prom3, prom4])
  .then((res) => console.log("allSettled", res))
  .catch((err) => console.log("allSettled failed", err));

// completes with the first promise that resolves
// fails if all promises fail
Promise.any([prom1, prom2, prom3, prom4])
  .then((res) => console.log("any", res))
  .catch((err) => console.log("any failed", err));

// completes with the first promise that either resolve or fail
// whichever comes first
Promise.race([prom1, prom2, prom3, prom4])
  .then((res) => console.log("race", res))
  .catch((err) => console.log("race failed", err));
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

### 条件对象键
```js
let condition = true;
const man = {
  someProperty: "some value",
  // the parenthesis will execute the ternary that will
  // result in the object with the property you want to insert
  // or an empty object.then its content is spreaded in the wrapper object
  ...(condition === true ? { newProperty: "value" } : {}),
};
```

### 使用变量作为对象键
```js
let property = "newValidProp";
const man2 = {
  someProperty: "some value",
  // the "square bracket" notation is a valid way to acces object key
  // like object[prop] but it is used inside to assign a property as well
  // using the 'backtick' to first change it into a string

  // but it is optional
["${property}"]: "value",
};
```

### 检查对象里的键
```js
const sample = {
  prop: "value",
};
// using the "in" keyword will still consider proptotype keys
// which makes it unsafe and one of the issues with "for...in" loop
console.log("prop" in sample); // prints "true"
console.log("toString" in sample); // prints "true"
// using the "hasOwnProperty" methods is safer
console.log(sample.hasOwnProperty("prop")); // prints "true"
console.log(sample.hasOwnProperty("toString")); // prints "false"
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

### 在ArrayforEach中执行“break”和“continue”
```js
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9];
for (const number of numbers) {
  if (number % 2 === 0) {
    continue;
  }
  if (number > 5) {
    break;
  }
  console.log(number);
}
numbers.some((number) => {
  if (number % 2 === 0) {
    // continue;
  }
  if (number > 5) {
    // break;
  }
  console.log(number);
});
```

### 使用别名和默认值来销毁
```js
function demo1({ dt: data }) {
  // rename "dt" to "data"
  console.log(data); // prints {name: 'sample', id: 50}
}
function demo2({ dt: { name, id = 10 } }) {
  // deep destruct "dt" and if no "id" use 10 as default
  console.log(name, id); // prints 'sample', '10'
}
demo1({
  dt: { name: "sample", id: 50 },
});

demo2({
  dt: { name: "sample" },
});
```

### 可选链和空值合并
```js
const obj = {
  data: {
    container: {
      name: {
        value: "sample",
      },
      int: {
        value: 0,
      },
    },
  },
};

console.log(
  // even though the "int.value" exists, it is falsy therefore fails to be printed
  obj.data.container.int.value || "no int value", // prints 'no int value'
  // the ?? make sure to fallback to the right side only if left is null or undefined
  obj.data.container.int.value ?? "no int value" // prints 0
);
console.log(
  // "wrapper" does not exist inside "data"
  obj.data.wrapper.name.value, // throws "Cannot read property 'name' of undefined"
  // this is better but can be a problem if object is deep
  (obj && obj.data && obj.data.wrapper && obj.data.wrapper.name) || "no name", // prints 'no name"
  // using optional chaining "?" is better
  obj?.data?.wrapper?.name || "no name" // prints 'no name'
);
```

### 用函数扩展类
```js
function Parent() {
  const privateProp = 12;
  const privateMethod = () => privateProp + 10;
  this.publicMethod = (x = 0) => privateMethod() + x;
  this.publicProp = 10;
}
class Child extends Parent {
  myProp = 20;
}
const child = new Child();
console.log(
  child.myProp, // prints 20
  child.publicProp, // prints 10
  child.publicMethod(40), // prints 62
  child.privateProp, // prints undefined
  child.privateMethod() // throws "child.privateMethod is not a function"
);
```

### 扩展构造函数
```js
function Employee() {
  this.profession = "Software Engineer";
  this.salary = "$150000";
}

function DeveloperFreelancer() {
  this.programmingLanguages = ["Javascript", "Python", " Swift"];
  this.avgPerHour = "$100";
}

function Engineer(name) {
  this.name = name;
  this.freelancer = {};
  Employee.apply(this);
  DeveloperFreelancer.apply(this.freelancer);
}
const john = new Engineer("John Doe");
console.log(
  john.name, // prints "John Doe"
  john.profession, // prints "Software Engineer"
  john.salary, // prints "$150000"
  john.freelancer // prints {programmingLanguages: ['Javascript', 'Python', 'Swift'], avgPerHour: '$100'}
);
```

### 循环任何内容
```js
function forEach(list, callback) {
  const entries = Object.entries(list);
  let i = 0;
  const len = entries.length;

  for (; i < len; i++) {
    const res = callback(entries[i][1], entries[i][0], list);
    if (res === true) break;
  }
}
```

### 使函数参数为required
```js
function required(argName = "param") {
  throw new Error(`"${argName}" is required`);
}
function iHaveRequiredOptions(arg1 = required("arg1"), arg2 = 10) {
  console.log(arg1, arg2);
}
iHaveRequiredOptions(); // throws "arg1" is required
iHaveRequiredOptions(12); // prints 12, 10
iHaveRequiredOptions(12, 24); // prints 12, 24
iHaveRequiredOptions(undefined, 24); // throws "arg1" is required
```

### 创建模块或单例
```js
class Service {
  name = "service";
}
const service = (function (S) {
  // do something here like preparing data that you can use to initialize service
  const service = new S();
  return () => service;
})(Service);
const element = (function (S) {
  const element = document.createElement("DIV");
  // do something here to grab somethin on the dom
  // or create elements with javasrcipt setting it all up
  // than to return it
  return () => element;
})();
```

### 深度克隆对象
```js
const deepClone = (obj) => {
  let clone = obj;
  if (obj && typeof obj === "object") {
    clone = new obj.constructor();

    Object.getOwnPropertyNames(obj).forEach(
      (prop) => (clone[prop] = deepClone(obj[prop]))
    );
  }
  return clone;
};
```

### 深度冻结对象
```js
const deepClone2 = (obj) => {
  let clone = obj;
  if (obj && typeof obj === "object") {
    clone = new obj.constructor();

    Object.getOwnPropertyNames(obj).forEach(
      (prop) => (clone[prop] = deepClone(obj[prop]))
    );
  }
  return clone;
};

```

## 9个极其强大的JavaScript技巧

### Replace All
```js
var example = "potato potato";
console.log(example.replace(/pot/, "tom"));// "tomato potato"
console.log(example.replace(/pot/g, "tom"));// "tomato tomato"
```
### 提取唯一值 我们可以使用Set对象和Spread运算符，创建一个剔除重复值的新数组。
```js
var entries = [1, 2, 2, 3, 4, 5, 6, 6, 7, 7, 8, 4, 2, 1]
var unique_entries = [...new Set(entries)];
console.log(unique_entries);// [1, 2, 3, 4, 5, 6, 7, 8]
```
### 将数字转换为字符串
```js
var converted_number = 5 + "";
console.log(converted_number);// 5
console.log(typeof converted_number);// string
```
### 将字符串转换为数字 请注意这里的用法，因为它只适用于“字符串数字”。
```js
var the_string = "123";
console.log(+the_string);// 123
the_string = "hello";
console.log(+the_string);// NaN
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
### 短路条件
```js
// available && addToCart()代替
// if (available) {
//     addToCart();
// }
```
### 动态属性名称
```js
const dynamic = 'flavour';
var item = {
    name: 'Coke',
    [dynamic]: 'Cherry'
}
console.log(item);// { name: "Coke", flavour: "Cherry" }
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

## JS数组操作

### 声明数组

```js
const fruits = new Array('Apple', 'Banana');
console.log(fruits.length);

// 通过数组字面量创建一个有2个元素的'fruits'数组.
const fruits = ['Apple', 'Banana'];
console.log(fruits.length);
```

### 


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

### 返回已size为长度的数组分割的原数组

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

## 浏览器

### 浏览器是否支持CSS属性
```js
// -------浏览器对象 BOM-------
/**
 * 告知浏览器支持的指定css属性情况
 * @param {String} key - css属性，是属性的名字，不需要加前缀
 * @returns {String} - 支持的属性情况
 */
function validateCssKey(key) {
  const jsKey = toCamelCase(key); // 有些css属性是连字符号形成
  if (jsKey in document.documentElement.style) {
    return key;
  }
  let validKey = "";
  // 属性名为前缀在js中的形式，属性值是前缀在css中的形式
  // 经尝试，Webkit也可是首字母小写webkit
  const prefixMap = {
    Webkit: "-webkit-",
    Moz: "-moz-",
    ms: "-ms-",
    O: "-o-",
  };
  for (const jsPrefix in prefixMap) {
    const styleKey = toCamelCase(`${jsPrefix}-${jsKey}`);
    if (styleKey in document.documentElement.style) {
      validKey = prefixMap[jsPrefix] + key;
      break;
    }
  }
  return validKey;
}

/**
 * 把有连字符号的字符串转化为驼峰命名法的字符串
 */
function toCamelCase(value) {
  return value.replace(/-(\w)/g, (matched, letter) => {
    return letter.toUpperCase();
  });
}
```
### 检查浏览器是否支持某个css属性值（es6版）
```js
/**
 * 检查浏览器是否支持某个css属性值（es6版）
 * @param {String} key - 检查的属性值所属的css属性名
 * @param {String} value - 要检查的css属性值（不要带前缀）
 * @returns {String} - 返回浏览器支持的属性值
 */
function valiateCssValue(key, value) {
  const prefix = ["-o-", "-ms-", "-moz-", "-webkit-", ""];
  const prefixValue = prefix.map((item) => {
    return item + value;
  });
  const element = document.createElement("div");
  const eleStyle = element.style;
  // 应用每个前缀的情况，且最后也要应用上没有前缀的情况，看最后浏览器起效的何种情况
  // 这就是最好在prefix里的最后一个元素是''
  prefixValue.forEach((item) => {
    eleStyle[key] = item;
  });
  return eleStyle[key];
}

/**
 * 检查浏览器是否支持某个css属性值
 * @param {String} key - 检查的属性值所属的css属性名
 * @param {String} value - 要检查的css属性值（不要带前缀）
 * @returns {String} - 返回浏览器支持的属性值
 */
function valiateCssValue(key, value) {
  var prefix = ["-o-", "-ms-", "-moz-", "-webkit-", ""];
  var prefixValue = [];
  for (var i = 0; i < prefix.length; i++) {
    prefixValue.push(prefix[i] + value);
  }
  var element = document.createElement("div");
  var eleStyle = element.style;
  for (var j = 0; j < prefixValue.length; j++) {
    eleStyle[key] = prefixValue[j];
  }
  return eleStyle[key];
}

function validCss(key, value) {
  const validCss = validateCssKey(key);
  if (validCss) {
    return validCss;
  }
  return valiateCssValue(key, value);
}
```
### 返回当前网页地址
```js
/**
 * 返回当前网页地址⬇
 */ 
// 方案一：location
function currentURL() {
  return window.location.href;
}
currentURL();

// 方案二：a 标签
function currentURL() {
  var el = document.createElement("a");
  el.href = "";
  return el.href;
}
currentURL();
```
### 获取滚动条位置
```js
/**
 * 获取滚动条位置⬇
 */ 
function getScrollPosition(el = window) {
  return {
    x: el.pageXOffset !== undefined ? el.pageXOffset : el.scrollLeft,
    y: el.pageYOffset !== undefined ? el.pageYOffset : el.scrollTop,
  };
}
```
### 获取url中的参数
```js
/**
 * 获取url中的参数⬇
 */ 
// 方案一：正则 + reduce
function getURLParameters(url) {
  return url.match(/([^?=&]+)(=([^&]*))/g).reduce((a, v) => (
        (a[v.slice(0, v.indexOf("="))] = v.slice(v.indexOf("=") + 1)), a
      ),{}
    );
}
getURLParameters(location.href);

// 方案二：split + reduce
function getURLParameters(url) {
  return url
    .split("?") //取？分割
    .slice(1) //不要第一部分
    .join() //拼接
    .split("&") //&分割
    .map((v) => v.split("=")) //=分割
    .reduce((s, n) => {
      s[n[0]] = n[1];
      return s;
    }, {});
}
getURLParameters(location.href);
// getURLParameters('')

// 方案三: URLSearchParams
```
### 页面跳转，是否记录在history中
```js
/**
 * 页面跳转，是否记录在history中⬇
 */ 
// 方案一：
function redirect(url, asLink = true) {
  asLink ? (window.location.href = url) : window.location.replace(url);
}
// 方案二：
function redirect(url, asLink = true) {
  asLink ? window.location.assign(url) : window.location.replace(url);
}
```
### 滚动条回到顶部动画
```js
/**
 * 滚动条回到顶部动画⬇
 */ 
// 方案一： c - c / 8
// c没有定义
function scrollToTop() {
  const scrollTop =
    document.documentElement.scrollTop || document.body.scrollTop;
  if (scrollTop > 0) {
    window.requestAnimationFrame(scrollToTop);
    window.scrollTo(0, c - c / 8);
  } else {
    window.cancelAnimationFrame(scrollToTop);
  }
}
scrollToTop();

// 修正之后
function scrollToTop() {
  const scrollTop =
    document.documentElement.scrollTop || document.body.scrollTop;
  if (scrollTop > 0) {
    window.requestAnimationFrame(scrollToTop);
    window.scrollTo(0, scrollTop - scrollTop / 8);
  } else {
    window.cancelAnimationFrame(scrollToTop);
  }
}
scrollToTop();
```
### 复制文本
```js
/**
 * 复制文本⬇
 */ 
// 方案一：
function copy(str) {
  const el = document.createElement("textarea");
  el.value = str;
  el.setAttribute("readonly", "");
  el.style.position = "absolute";
  el.style.left = "-9999px";
  el.style.top = "-9999px";
  document.body.appendChild(el);
  const selected =
    document.getSelection().rangeCount > 0
      ? document.getSelection().getRangeAt(0)
      : false;
  el.select();
  document.execCommand("copy");
  document.body.removeChild(el);
  if (selected) {
    document.getSelection().removeAllRanges();
    document.getSelection().addRange(selected);
  }
}
// 方案二：cliboard.js
```
### 检测设备类型
```js
/**
 * 检测设备类型⬇
 */ 
// 方案一： ua
function detectDeviceType() {
  return /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
    navigator.userAgent
  )
    ? "Mobile"
    : "Desktop";
}
detectDeviceType();

// 方案二：事件属性
function detectDeviceType() {
  return "ontouchstart" in window || navigator.msMaxTouchPoints
    ? "Mobile"
    : "Desktop";
}
detectDeviceType();

```

## Cookie

```js
// -------cookie-------
/**
 *
 * @param {*} key
 * @param {*} value
 * @param {*} expiredays 过期时间
 */
function setCookie(key, value, expiredays) {
  var exdate = new Date();
  exdate.setDate(exdate.getDate() + expiredays);
  document.cookie =
    key +
    "=" +
    escape(value) +
    (expiredays == null ? "" : ";expires=" + exdate.toGMTString());
}
/**
 *
 * @param {*} name cookie key
 */
function delCookie(name) {
  var exp = new Date();
  exp.setTime(exp.getTime() - 1);
  var cval = getCookie(name);
  if (cval != null) {
    document.cookie = name + "=" + cval + ";expires=" + exp.toGMTString();
  }
}
/**
 *
 * @param {*} name cookie key
 */
function getCookie(name) {
  var arr,
    reg = new RegExp("(^| )" + name + "=([^;]*)(;|$)");
  if ((arr = document.cookie.match(reg))) {
    return arr[2];
  } else {
    return null;
  }
}
// 清空
// 有时候我们想清空，但是又无法获取到所有的cookie。
// 这个时候我们可以了利用写满，然后再清空的办法。

```

## Date

```js
// ---------日期 Date------------

/**
 * 获取当前时间戳⬇
 */ 
// 方案一：精确到秒
console.log(Date.parse(new Date())) 
// 方案二：精确到毫秒
console.log(Date.now()) 
// 方案三：精确到毫秒
console.log(+new Date()) 
// 方案四：精确到毫秒
console.log(new Date().getTime()) 
// 方案五：精确到毫秒
console.log((new Date()).valueOf())

/**
 * js字符串转时间戳⬇
 * mytime是待转换时间字符串，格式：'2018-9-12 9:11:23'
 * 为了兼容IOS，需先将字符串转换为'2018/9/11 9:11:23'
 */
let mytime = '2018-9-12 9:11:23';
let dateTmp = mytime.replace(/-/g, "/");
console.log(new Date(dateTmp).getTime());
console.log(Date.parse(dateTmp));

/**
 * 时间戳转字符串
 */ 
var dateFormat = function (timestamp) {
  //先将时间戳转为Date对象，然后才能使用Date的方法
  var time = new Date(timestamp);
  var year = time.getFullYear(),
    month = time.getMonth() + 1, //月份是从0开始的
    day = time.getDate(),
    hour = time.getHours(),
    minute = time.getMinutes(),
    second = time.getSeconds();
  //add0()方法在后面定义
  return (
    year +
    "-" +
    this.add0(month) +
    "-" +
    this.add0(day) +
    "" +
    this.add0(hour) +
    ":" +
    this.add0(minute) +
    ":" +
    this.add0(second)
  );
};
var add0 = function (m) {
  return m < 10 ? "0" + m : m;
};
/**
 * 格式化字符串
 */
Date.prototype.format = function (fmt) { //author: meizz 
  var o = {
      "M+": this.getMonth() + 1, //月份 
      "d+": this.getDate(), //日 
      "h+": this.getHours(), //小时 
      "m+": this.getMinutes(), //分 
      "s+": this.getSeconds(), //秒 
      "q+": Math.floor((this.getMonth() + 3) / 3), //季度 
      "S": this.getMilliseconds() //毫秒 
  };
  if (/(y+)/.test(fmt)) fmt = fmt.replace(RegExp.$1, (this.getFullYear() + "").substr(4 - RegExp.$1.length));
  for (var k in o)
  if (new RegExp("(" + k + ")").test(fmt)) fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k]).length)));
  return fmt;
}
console.log(new Date().format('yyyy-MM-dd hh:mm:ss'));
/**
 * JavaScript Date对象(https://www.w3school.com.cn/js/jsref_obj_date.asp)
 */

```

## 文档对象DOM

### 固定滚动条
```js
/**
 * 固定滚动条
 * 功能描述：一些业务场景，如弹框出现时，需要禁止页面滚动，这是兼容安卓和iOS禁止页面滚动的解决方案
 */
let scrollTop = 0;

function preventScroll() {
  // 存储当前滚动位置
  scrollTop = window.scrollY;

  // 将可滚动区域固定定位，可滚动区域高度为0后就不能滚动了
  document.body.style["overflow-y"] = "hidden";
  document.body.style.position = "fixed";
  document.body.style.width = "100%";
  document.body.style.top = -scrollTop + "px";
  // document.body.style['overscroll-behavior'] = 'none'
}

function recoverScroll() {
  document.body.style["overflow-y"] = "auto";
  document.body.style.position = "static";
  // document.querySelector('body').style['overscroll-behavior'] = 'none'

  window.scrollTo(0, scrollTop);
}
```
### 判断当前位置是否为页面底部
```js
/**
 * 判断当前位置是否为页面底部
 * 返回值为true/false
 */
function bottomVisible() {
  return (
    document.documentElement.clientHeight + window.scrollY >=
    (document.documentElement.scrollHeight ||
      document.documentElement.clientHeight)
  );
}
```
### 判断元素是否在可视范围内
```js
/**
 * 判断元素是否在可视范围内
 * partiallyVisible为是否为完全可见
 */
function elementIsVisibleInViewport(el, partiallyVisible = false) {
  const { top, left, bottom, right } = el.getBoundingClientRect();

  return partiallyVisible
    ? ((top > 0 && top < innerHeight) ||
        (bottom > 0 && bottom < innerHeight)) &&
        ((left > 0 && left < innerWidth) || (right > 0 && right < innerWidth))
    : top >= 0 && left >= 0 && bottom <= innerHeight && right <= innerWidth;
}
```
### 获取元素css样式
```js
/**
 * 获取元素css样式
 */
function getStyle(el, ruleName) {
  return getComputedStyle(el, null).getPropertyValue(ruleName);
}
```
### 进入全屏
```js
/**
 * 进入全屏
 */
function launchFullscreen(element) {
  if (element.requestFullscreen) {
    element.requestFullscreen();
  } else if (element.mozRequestFullScreen) {
    element.mozRequestFullScreen();
  } else if (element.msRequestFullscreen) {
    element.msRequestFullscreen();
  } else if (element.webkitRequestFullscreen) {
    element.webkitRequestFullScreen();
  }
}

launchFullscreen(document.documentElement);
launchFullscreen(document.getElementById("id")); //某个元素进入全屏
```
### 退出全屏
```js
/**
 * 退出全屏
 */
function exitFullscreen() {
  if (document.exitFullscreen) {
    document.exitFullscreen();
  } else if (document.msExitFullscreen) {
    document.msExitFullscreen();
  } else if (document.mozCancelFullScreen) {
    document.mozCancelFullScreen();
  } else if (document.webkitExitFullscreen) {
    document.webkitExitFullscreen();
  }
}

exitFullscreen();
```
### 全屏事件
```js
/**
 * 全屏事件
 */
document.addEventListener("fullscreenchange", function (e) {
  if (document.fullscreenElement) {
    console.log("进入全屏");
  } else {
    console.log("退出全屏");
  }
});
```

## number

```js
// ----数字Number-----

/**
 * 数字千分位分割
 * @param {*} num 
 */
function commafy(num) {
  return num.toString().indexOf(".") !== -1
    ? num.toLocaleString()
    : num.toString().replace(/(\d)(?=(?:\d{3})+$)/g, "$1,");
}
commafy(1000)

/**
 * 生成随机数
 * @param {*} min 
 * @param {*} max 
 */

function randomNum(min, max) {
  switch (arguments.length) {
    case 1:
      return parseInt(Math.random() * min + 1, 10);
    case 2:
      return parseInt(Math.random() * (max - min + 1) + min, 10);
    default:
      return 0;
  }
}
randomNum(1,10)
```

## spread运算符

```js
// 插入数组：
// 看看如下代码，不使用扩展语法：
var mid = [3, 4];
var arr = [1, 2, mid, 5, 6];
console.log(arr);  // [1, 2, [3, 4] , 5, 6]
// 上面这段代码将得到一个嵌套数组的数组。大部分情况，我们希望一个array（mid）展开后再插入到另一个array（arr）中。
// 使用spread操作符我们可以这样：
var mid = [3, 4];
var arr = [1, 2, ...mid, 5, 6];
console.log(arr); // [1，2，3，4，5，6]

// 展开数组作为参数
// 当一个函数接收多个参数，比如Math.max，当我们有一个数组需要找到你了的最大值，我们可以使用如下代码进行调用。
var arr = [2, 4, 8, 6, 0];
function max(arr) {
  return Math.max.apply(null, arr);
}
console.log(max(arr)); // 8

// 如果这时候使用spread运算符会变得非常方便。
var arr = [2, 4, 8, 6, 0];
var max = Math.max(...arr);
console.log(max); // 8

// 复制数组
// 用数组给新数组赋值只是获取到数组引用，并没有达到深复制的效果。
var arr = ['a', 'b', 'c'];
var arr2 = arr;
arr2.push('d');
console.log(arr);// ["a", "b", "c", "d"]

// 有多重方法可以实现深复制，但是使用spread操作符是最简洁的一种实现：
var arr = ['a', 'b', 'c'];
var arr2 = [...arr];
arr2.push('d');
console.log(arr);  // ["a", "b", "c"]

// 展开String
// 如果想将字符串转为字符数组，如果不实用展开操作：
var str = "hello";
"hello".split('') // ["h", "e", "l", "l", "o"]

// 使用展开操作符可以这样写：
var str = "hello";
var chars = [...str]; // ["h", "e", "l", "l", "o"]

// 展开Object
// 我们还可以对对象进行展开，如果有两个对象，有不同的key-value,我们需要将这两个对象合并起来(需要使用Object rest spread transform)，我们可以这样：

let Obj1 = {
 key1: 'value1'
}

let Obj2 = {
 key2: 'value3'
}

let concatValue = {
 ...Obj1,
 ...Obj2
}
console.log(concatValue) // {key1: 'value1', key2: 'value2'}
```

## 8个你可能不知道的HTML实用技巧
```html
<!-- 使用capture属性打开设备摄像头,user用于前置摄像头,environment用于后置摄像头-->
<input type="file" capture="user" accept="image/*">
<!-- 网站自动刷新,实现每10秒刷新一次网站 -->
<head>
    <meta http-equiv="refresh" content="10">
</head>
<!-- 激活拼写检查,使用spellcheck属性并将其设置为true以激活拼写检查。使用lang属性指定待检查的语言。-->
<input type="text" spellcheck="true" lang="en">
<!-- 指定要上传的文件类型,使用accept属性在input标签中指定允许用户上传的文件类型。-->
<input type="file" accept=".jpeg,.png">
<!-- 阻止浏览器翻译,将translate属性设置为no会阻止浏览器翻译该内容 -->
<p translate="no">Brand name</p>
<!-- 在input标签中输入多个项目,适用于文件和电子邮件。如果是电子邮件，则可以用逗号分隔 -->
<input type="file" multiple>
<!-- 为视频创建海报（缩略图）使用poster属性，我们可以在视频加载时，或者在用户点击播放按钮之前，显示指定的缩略图。如果不指定图片，则默认使用视频的第一帧作为缩略图 -->
<video poster="picture.png"></video>
<!-- 点击链接自动下载 -->
<a href="image.png" download>

```