---
title: 编写JavaScript代码的小技巧
categories: JS
tags: 代码实战
img: https://pic1.zhimg.com/v2-6ef93db4040b0eb0b4524d06e372ad0f.jpg
---

### 生成随机颜色/随机数/Boolean值

```js
// 生成随机颜色
const generateRandomHexColor = () => Math.floor(Math.random() * 0xffffff).toString(16)
console.log(generateRandomHexColor())

// 获取随机颜色
// 日常我们经常会需要获取一个随机颜色，通过随机数即可完成
function getRandomColor(){
    return `#${Math.floor(Math.random() * 0xffffff) .toString(16)}`;
}

// 带有范围的随机数生成器
function randomNumber(max = 1, min = 0) {
  if (min >= max) {
    return max;
  }
  return Math.floor(Math.random() * (max - min) + min);
}

// 随机ID生成器
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

// 创建一个范围内的数字
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

// 随机获取一个Boolean值
// 通过随机数获取，Math.random()的区间是0-0.99，用0.5在中间百分之五十的概率
function randomBool() {
    return 0.5 - Math.random()
}

// uuid
export const uuid = () => {
    const temp_url = URL.createObjectURL(new Blob())
    const uuid = temp_url.toString()
    URL.revokeObjectURL(temp_url) //释放这个url
    return uuid.substring(uuid.lastIndexOf('/') + 1)
}
// 示例：
uuid() // a640be34-689f-4b98-be77-e3972f9bffdd

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

### 其他操作

```js
// 复制到剪切板
const copyToClipboard = (text) => navigator.clipboard && navigator.clipboard.writeText && navigator.clipboard.writeText(text)
copyToClipboard("Hello World!")

// 将下标转为中文零一二三...
// 日常可能有的列表我们需要将对应的012345转为中文的一、二、三、四、五...，在老的项目看到还有通过自己手动定义很多行这样的写法，于是写了一个这样的方法转换
export function transfromNumber(number){
  const  INDEX_MAP = ['零'，'一'.....]
  if(!number) return
  if(number === 10) return INDEX_MAP[number]
  return [...number.toString()].reduce( (pre, cur) => pre  + INDEX_MAP[cur] , '' )
}

// Boolean转换
// 一些场景下我们会将boolean值定义为场景，但是在js中非空的字符串都会被认为是true
function toBoolean(value, truthyValues = ['true']){
  const normalizedValue = String(value).toLowerCase().trim();
  return truthyValues.includes(normalizedValue);
}
toBoolean('TRUE'); // true
toBoolean('FALSE'); // false
toBoolean('YES', ['yes']); // true

// 计算两个坐标之间的距离
function distance(p1, p2){
    return Math.sqrt(Math.pow(p2.x - p1.x, 2) + Math.pow(p2.y - p1.y, 2));
}

// 获取列表最后一项
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

// 格式化JSON字符串，stringify任何内容
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

// 轮询数据
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

// 使用别名和默认值来销毁
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

// 循环任何内容
function forEach(list, callback) {
  const entries = Object.entries(list);
  let i = 0;
  const len = entries.length;

  for (; i < len; i++) {
    const res = callback(entries[i][1], entries[i][0], list);
    if (res === true) break;
  }
}

// 使函数参数为required
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

// Replace All
var example = "potato potato";
console.log(example.replace(/pot/, "tom"));// "tomato potato"
console.log(example.replace(/pot/g, "tom"));// "tomato tomato"

// 防抖
export const debounce = (() => {
    let timer = null
    return (callback, wait = 800) => {
        timer&&clearTimeout(timer)
        timer = setTimeout(callback, wait)
    }
})()
// 示例：如vue中使用
methods: {
    loadList() {
        debounce(() => {
            console.log('加载数据')
        }, 500)
    }
}

// 节流
export const throttle = (() => {
    let last = 0
    return (callback, wait = 800) => {
        let now = +new Date()
        if (now - last > wait) {
            callback()
            last = now
        }
    }
})()

// 手机号脱敏
export const hideMobile = (mobile) => {
  return mobile.replace(/^(\d{3})\d{4}(\d{4})$/, "$1****$2")
}
// 大小写转换
// 参数：str:待转换的字符串 type:1-全大写 2-全小写 3-首字母大写
export const turnCase = (str, type) => {
    switch (type) {
        case 1:
            return str.toUpperCase()
        case 2:
            return str.toLowerCase()
        case 3:
            //return str[0].toUpperCase() + str.substr(1).toLowerCase() // substr 已不推荐使用
            return str[0].toUpperCase() + str.substring(1).toLowerCase()
        default:
            return str
    }
}
// 示例：
turnCase('vue', 1) // VUE
turnCase('REACT', 2) // react
turnCase('vue', 3) // Vue

// 金额格式化
/*参数：
{number} number：要格式化的数字
{number} decimals：保留几位小数
{string} dec_point：小数点符号
{string} thousands_sep：千分位符号
*/
export const moneyFormat = (number, decimals, dec_point, thousands_sep) => {
    number = (number + '').replace(/[^0-9+-Ee.]/g, '')
    const n = !isFinite(+number) ? 0 : +number
    const prec = !isFinite(+decimals) ? 2 : Math.abs(decimals)
    const sep = typeof thousands_sep === 'undefined' ? ',' : thousands_sep
    const dec = typeof dec_point === 'undefined' ? '.' : dec_point
    let s = ''
    const toFixedFix = function(n, prec) {
        const k = Math.pow(10, prec)
        return '' + Math.ceil(n * k) / k
    }
    s = (prec ? toFixedFix(n, prec) : '' + Math.round(n)).split('.')
    const re = /(-?\d+)(\d{3})/
    while (re.test(s[0])) {
        s[0] = s[0].replace(re, '$1' + sep + '$2')
    }

    if ((s[1] || '').length < prec) {
        s[1] = s[1] || ''
        s[1] += new Array(prec - s[1].length + 1).join('0')
    }
    return s.join(dec)
}
// 示例：
moneyFormat(10000000) // 10,000,000.00
moneyFormat(10000000, 3, '.', '-') // 10-000-000.000

// 存储操作
class MyCache {
    constructor(isLocal = true) {
        this.storage = isLocal ? localStorage : sessionStorage
    }
    setItem(key, value) {
        if (typeof (value) === 'object') value = JSON.stringify(value)
        this.storage.setItem(key, value)
    }
    getItem(key) {
        try {
            return JSON.parse(this.storage.getItem(key))
        } catch (err) {
            return this.storage.getItem(key)
        }
    }
    removeItem(key) {
        this.storage.removeItem(key)
    }
    clear() {
        this.storage.clear()
    }
    key(index) {
        return this.storage.key(index)
    }
    length() {
        return this.storage.length
    }
}
const localCache = new MyCache()
const sessionCache = new MyCache(false)
export { localCache, sessionCache }
// 示例：
localCache.getItem('user')
sessionCache.setItem('name','树哥')
sessionCache.getItem('token')
localCache.clear()

// 下载文件
/*参数：
api 接口
params 请求参数
fileName 文件名
*/
const downloadFile = (api, params, fileName, type = 'get') => {
    axios({
        method: type,
        url: api,
        responseType: 'blob', 
        params: params
    }).then((res) => {
        let str = res.headers['content-disposition']
        if (!res || !str) {
            return
        }
        let suffix = ''
        // 截取文件名和文件类型
        if (str.lastIndexOf('.')) {
            fileName ? '' : fileName = decodeURI(str.substring(str.indexOf('=') + 1, str.lastIndexOf('.')))
            suffix = str.substring(str.lastIndexOf('.'), str.length)
        }
        //  如果支持微软的文件下载方式(ie10+浏览器)
        if (window.navigator.msSaveBlob) {
            try {
                const blobObject = new Blob([res.data]);
                window.navigator.msSaveBlob(blobObject, fileName + suffix);
            } catch (e) {
                console.log(e);
            }
        } else {
            //  其他浏览器
            let url = window.URL.createObjectURL(res.data)
            let link = document.createElement('a')
            link.style.display = 'none'
            link.href = url
            link.setAttribute('download', fileName + suffix)
            document.body.appendChild(link)
            link.click()
            document.body.removeChild(link)
            window.URL.revokeObjectURL(link.href);
        }
    }).catch((err) => {
        console.log(err.message);
    })
}

// 使用：
downloadFile('/api/download', {id}, '文件名')
```

### 滚动到顶部/底部

```js
// 滚动到页面顶部
export const scrollToTop = () => {
    const height = document.documentElement.scrollTop || document.body.scrollTop;
    if (height > 0) {
        window.requestAnimationFrame(scrollToTop);
        window.scrollTo(0, height - height / 8);
    }
}
// 滚动到元素位置
export const smoothScroll = element =>{
    document.querySelector(element).scrollIntoView({
        behavior: 'smooth'
    });
};
// 示例：
smoothScroll('#target'); // 平滑滚动到ID为target 的元素

// 滚动到顶部。将元素滚动到顶部最简单的方法是使用scrollIntoView。设置block为start可以滚动到顶部；设置behavior为smooth可以开启平滑滚动。
const scrollToTop = (element) => element.scrollIntoView({ behavior: "smooth", block: "start" });

// 滚动到底部。与滚动到顶部一样，滚动到底部只需要设置block为end即可。
const scrollToBottom = (element) =>  element.scrollIntoView({ behavior: "smooth", block: "end" });

// dom节点平滑滚动到可视区域，顶部，底部。原生的scrollTo方法没有动画，类似于锚点跳转，比较生硬，可以通过这个方法会自带平滑的过度效果
function scrollTo(element) {
    element.scrollIntoView({ behavior: "smooth", block: "start" }) // 顶部
    element.scrollIntoView({ behavior: "smooth", block: "end" }) // 底部
    element.scrollIntoView({ behavior: "smooth"}) // 可视区域
}

/**
 * 获取滚动条位置⬇
 */ 
function getScrollPosition(el = window) {
  return {
    x: el.pageXOffset !== undefined ? el.pageXOffset : el.scrollLeft,
    y: el.pageYOffset !== undefined ? el.pageYOffset : el.scrollTop,
  };
}

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

// 模糊搜索
/*
 * list 原数组
 * keyWord 查询的关键词
 * attribute 数组需要检索属性
 **/
export const fuzzyQuery = (list, keyWord, attribute = 'name') => {
    const reg = new RegExp(keyWord)
    const arr = []
    for (let i = 0; i < list.length; i++) {
        if (reg.test(list[i][attribute])) {
            arr.push(list[i])
        }
    }
    return arr
}
// 示例：
const list = [
    { id: 1, name: '树哥' },
    { id: 2, name: '黄老爷' },
    { id: 3, name: '张麻子' },
    { id: 4, name: '汤师爷' },
    { id: 5, name: '胡万' },
    { id: 6, name: '花姐' },
    { id: 7, name: '小梅' }
]
fuzzyQuery(list, '树', 'name') // [{id: 1, name: '树哥'}]

// 遍历树节点
export const foreachTree = (data, callback, childrenName = 'children') => {
    for (let i = 0; i < data.length; i++) {
        callback(data[i])
        if (data[i][childrenName] && data[i][childrenName].length > 0) {
            foreachTree(data[i][childrenName], callback, childrenName)
        }
    }
}
// 示例：假设我们要从树状结构数据中查找id为9的节点
const treeData = [{
    id: 1,
    label: '一级 1',
    children: [{
        id: 4,
        label: '二级 1-1',
        children: [{
            id: 9,
            label: '三级 1-1-1'
        }, {
            id: 10,
            label: '三级 1-1-2'
        }]
    }]
}, {
    id: 2,
    label: '一级 2',
    children: [{
        id: 5,
        label: '二级 2-1'
    }, {
        id: 6,
        label: '二级 2-2'
    }]
}, {
    id: 3,
    label: '一级 3',
    children: [{
        id: 7,
        label: '二级 3-1'
    }, {
        id: 8,
        label: '二级 3-2'
    }]
}],
let result = foreachTree(data, (item) => {
          if (item.id === 9) {
              result = item
          }
      })
console.log('result', result)  // {id: 9,label: "三级 1-1-1"}   

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

### 判断操作
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

// 判断一个参数是不是函数
// 有时候我们的方法需要传入一个函数回调，但是需要检测其类型，我们可以通过Object的原型方法去检测，当然这个方法可以准确检测任何类型。
function isFunction(v){
   return ['[object Function]', '[object GeneratorFunction]', '[object AsyncFunction]', '[object Promise]'].includes(Object.prototype.toString.call(v));
}

// 判断是否是NodeJs环境
// 前端的日常开发是离不开nodeJs的，通过判断全局环境来检测是否是nodeJs环境
function isNode(){
    return typeof process !== 'undefined' && process.versions != null && process.versions.node != null;
}

// 判断手机是Andoird还是IOS
/** 
 * 1: ios
 * 2: android
 * 3: 其它
 */
export const getOSType=() => {
    let u = navigator.userAgent, app = navigator.appVersion;
    let isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1;
    let isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/);
    if (isIOS) {
        return 1;
    }
    if (isAndroid) {
        return 2;
    }
    return 3;
}
```
### 检查/检测操作
```js
// 检测元素是否在屏幕中
// 最好的方法是使用IntersectionObserver。
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

// 检测设备
// 使用navigator.userAgent来检测网站运行在哪种平台设备上。
const detectDeviceType = () =>
  /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
    navigator.userAgent
  ) ? "Mobile" : "Desktop";
console.log(detectDeviceType());

// 检测暗色主题
const isDarkMode = () => window.matchMedia && window.matchMedia("(prefers-color-scheme: dark)").matches;
console.log(isDarkMode())

// 多个字符串检查
// 通常，如果我们需要检查字符串是否等于多个值中的一个，往往很快会觉得疲惫不堪。幸运的是，JavaScript有一个内置的方法来帮助你解决这个问题
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

// 类型检查小工具
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

// 检查是否为空
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

// 检测是否为空对象
// 通过使用ES6的Reflect静态方法判断他的长度就可以判断是否是空数组了，也可以通过Object.keys()来判断
function isEmpty(obj){
    return  Reflect.ownKeys(obj).length === 0 && obj.constructor === Object;
}

// 检测两个dom节点是否覆盖重叠
// 有些场景下我们需要判断dom是否发生碰撞了或者重叠了，我们可以通过getBoundingClientRect获取到dom的x1,y1,x2,y2坐标然后进行坐标比对即可判断出来
function overlaps = (a, b) {
   return (a.x1 < b.x2 && b.x1 < a.x2) || (a.y1 < b.y2 && b.y1 < a.y2);
}

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

// Falsey（假值）检查
// 如果要检查变量是null、undefined、0、false、NaN还是空string，可以使用逻辑非(!)运算符一次检查所有变量，而无需编写多个条件。这使得检查变量是否包含有效数据变得相对容易多了。
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

### 从URL中获取参数

```js
// JavaScript中有一个URL对象，通过它可以非常方便的获取URL中的参数。
const getParamByUrl = (key) => {
  const url = new URL(location.href)
  return url.searchParams.get(key)
}

export const getSearchParams = () => {
    const searchPar = new URLSearchParams(window.location.search)
    const paramsObj = {}
    for (const [key, value] of searchPar.entries()) {
        paramsObj[key] = value
    }
    return paramsObj
}
// 示例：
// 假设目前位于 https://****com/index?id=154513&age=18;
getSearchParams(); // {id: "154513", age: "18"}

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
### 对象操作

```js
// 深拷贝对象
// 深拷贝对象非常简单，先将对象转换为字符串，再转换成对象即可。
const deepCopy = obj => JSON.parse(JSON.stringify(obj))
// 除了利用JSON的API，还有更新的深拷贝对象的structuredClone API，但并不是在所有的浏览器中都支持。
structuredClone(obj)

// 深拷贝
export const clone = parent => {
    // 判断类型
    const isType = (obj, type) => {
        if (typeof obj !== "object") return false;
        const typeString = Object.prototype.toString.call(obj);
        let flag;
        switch (type) {
            case "Array":
                flag = typeString === "[object Array]";
                break;
            case "Date":
                flag = typeString === "[object Date]";
                break;
            case "RegExp":
                flag = typeString === "[object RegExp]";
                break;
            default:
                flag = false;
        }
        return flag;
    };
    // 处理正则
    const getRegExp = re => {
        var flags = "";
        if (re.global) flags += "g";
        if (re.ignoreCase) flags += "i";
        if (re.multiline) flags += "m";
        return flags;
    };
    // 维护两个储存循环引用的数组
    const parents = [];
    const children = [];

    const _clone = parent => {
        if (parent === null) return null;
        if (typeof parent !== "object") return parent;

        let child, proto;

        if (isType(parent, "Array")) {
            // 对数组做特殊处理
            child = [];
        } else if (isType(parent, "RegExp")) {
            // 对正则对象做特殊处理
            child = new RegExp(parent.source, getRegExp(parent));
            if (parent.lastIndex) child.lastIndex = parent.lastIndex;
        } else if (isType(parent, "Date")) {
            // 对Date对象做特殊处理
            child = new Date(parent.getTime());
        } else {
            // 处理对象原型
            proto = Object.getPrototypeOf(parent);
            // 利用Object.create切断原型链
            child = Object.create(proto);
        }

        // 处理循环引用
        const index = parents.indexOf(parent);

        if (index != -1) {
            // 如果父数组存在本对象,说明之前已经被引用过,直接返回此对象
            return children[index];
        }
        parents.push(parent);
        children.push(child);

        for (let i in parent) {
            // 递归
            child[i] = _clone(parent[i]);
        }

        return child;
    };
    return _clone(parent);
};
// 此方法存在一定局限性：一些特殊情况没有处理: 例如Buffer对象、Promise、Set、Map。如果确实想要完备的深拷贝，推荐使用lodash中的cloneDeep方法。

// 条件对象键
let condition = true;
const man = {
  someProperty: "some value",
  // the parenthesis will execute the ternary that will
  // result in the object with the property you want to insert
  // or an empty object.then its content is spreaded in the wrapper object
  ...(condition === true ? { newProperty: "value" } : {}),
};

// 使用变量作为对象键
let property = "newValidProp";
const man2 = {
  someProperty: "some value",
  // the "square bracket" notation is a valid way to acces object key
  // like object[prop] but it is used inside to assign a property as well
  // using the 'backtick' to first change it into a string

  // but it is optional
["${property}"]: "value",
};

// 检查对象里的键
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

// 深度克隆对象
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

// 深度冻结对象
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
### 语法操作

```js
// 等待函数
// JavaScript提供了setTimeout函数，但是它并不返回Promise对象，所以我们没办法使用async作用在这个函数上，但是我们可以封装等待函数。
const wait = (ms) => new Promise((resolve)=> setTimeout(resolve, ms))
const asyncFn = async () => {
  await wait(1000)
  console.log('等待异步函数执行结束')
}
asyncFn()

// For-of和For-in循环。For-of和For-in循环是迭代array或object的好方法，因为无需手动跟踪object键的索引。
//For-of
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

// For-in
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

// 函数调用
// 在三元运算符的帮助下，你还可以根据条件确定要调用哪个函数。
// 注：函数的call signature必须相同，否则可能会遇到错误。
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

// Switch简写
// 通常我们可以使用以键作为switch条件并将值作为返回值的对象来优化长switch语句。
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

// 回退值
// ||运算符可以为变量设置回退值。
// 普通写法
let name;
if (user?.name) {
  name = user.name;
} else {
  name = "Anonymous";
}
// 简写方法
const name = user?.name || "Anonymous";

// 结构加赋值
let people = { name: null, age: null };
let result = { name: '张三',  age: 16 };
({ name: people.name, age: people.age} = result);
console.log(people) // {"name":"张三","age":16}

// 对基础数据类型进行解构
const {length : a} = '1234';
console.log(a) // 4

// 对数组解构快速拿到最后一项值
const arr = [1, 2, 3];
const { 0: first, length, [length - 1]: last } = arr;
first; // 1
last; // 3
length; // 3

// 创建模块或单例
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

// 短路条件
// available && addToCart()代替
// if (available) {
//     addToCart();
// }

// 动态属性名称
const dynamic = 'flavour';
var item = {
    name: 'Coke',
    [dynamic]: 'Cherry'
}
console.log(item);// { name: "Coke", flavour: "Cherry" }
```

### 数值操作

```js
// 判断整数的不同方法
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

// 参数求和
// 之前看到有通过函数柯理化形式来求和的，通过reduce一行即可
function sum(...args){
    args.reduce((a, b) => a + b);
}

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

// 将数字转换为字符串
var converted_number = 5 + "";
console.log(converted_number);// 5
console.log(typeof converted_number);// string

// 将字符串转换为数字 请注意这里的用法，因为它只适用于“字符串数字”。
var the_string = "123";
console.log(+the_string);// 123
the_string = "hello";
console.log(+the_string);// NaN
```

### 时间操作

关于时间操作，没必要自己再写一大串代码了，强烈推荐使用day.js
Day.js是一个仅2kb大小的轻量级JavaScript时间日期处理库，下载、解析和执行的JavaScript更少，为代码留下更多的时间。

```js
// 比较两个时间大小,通过调用getTime获取时间戳比较就可以了
function compare(a, b){
    return a.getTime() > b.getTime();
}

// 计算两个时间之间的月份差异
function monthDiff(startDate, endDate){
    return  Math.max(0, (endDate.getFullYear() - startDate.getFullYear()) * 12 - startDate.getMonth() + endDate.getMonth());
}

// 一步从时间中提取年月日时分秒.时间格式化轻松解决，一步获取到年月日时分秒毫秒，由于toISOString会丢失时区，导致时间差八小时，所以在格式化之前我们加上八个小时时间即可
function extract(date){
   const d = new Date(new Date(date).getTime() + 8*3600*1000);
  return new Date(d).toISOString().split(/[^0-9]/).slice(0, -1);
}
console.log(extract(new Date())) // ['2022', '09', '19', '18', '06', '11', '187']

// Date
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

### Promise

```js
// 顺序执行promise
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

// 等待所有promise完成
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

### 全屏操作
```js
// 开启全屏
export const launchFullscreen = (element) => {
    if (element.requestFullscreen) {
        element.requestFullscreen()
    } else if (element.mozRequestFullScreen) {
        element.mozRequestFullScreen()
    } else if (element.msRequestFullscreen) {
        element.msRequestFullscreen()
    } else if (element.webkitRequestFullscreen) {
        element.webkitRequestFullScreen()
    }
}
// 关闭全屏
export const exitFullscreen = () => {
    if (document.exitFullscreen) {
        document.exitFullscreen()
    } else if (document.msExitFullscreen) {
        document.msExitFullscreen()
    } else if (document.mozCancelFullScreen) {
        document.mozCancelFullScreen()
    } else if (document.webkitExitFullscreen) {
        document.webkitExitFullscreen()
    }
}
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

### Cookie

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

### CSS操作
```js
// 获取元素css样式
function getStyle(el, ruleName) {
  return getComputedStyle(el, null).getPropertyValue(ruleName);
}

// 隐藏元素
// 我们可以将元素的style.visibility设置为hidden，隐藏元素的可见性，但元素的空间仍然会被占用。如果设置元素的style.display为none，会将元素从渲染流中删除。
const hideElement = (el, removeFromFlow = false) => {
  removeFromFlow ? (el.style.display = 'none')
  : (el.style.visibility = 'hidden')
}

// 通过css检测系统的主题色从而全局修改样式
// @media的属性prefers-color-scheme就可以知道当前的系统主题，当然使用前需要查查兼容性
// ```css
// @media (prefers-color-scheme: dark) { //... } 
// @media (prefers-color-scheme: light) { //... }
// ```
// javascript也可以轻松做到

window.addEventListener('theme-mode', event =>{ 
    if(event.mode == 'dark'){}
   if(event.mode == 'light'){} 
})
window.matchMedia('(prefers-color-scheme: dark)') .addEventListener('change', event => { 
    if (event.matches) {} // dark mode
})


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

// 检查浏览器是否支持某个css属性值（es6版）
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

### HTML实用技巧
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

### 文章
[非常有用的48个JavaScript代码片段](https://mp.weixin.qq.com/s/uzbelq2TA0Er2QXwdP3wrg)