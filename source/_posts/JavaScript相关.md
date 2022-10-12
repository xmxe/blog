---
title: JavaScript相关
sticky: 53
categories: JS
tags: 
- 前端
- 随笔
index_img: /assert/js.jpg
img: 
---

#### JQuery、JS常用方法

json字符串  var str1 = '{ "name": "cxh", "sex": "man" }'; 
json对象   var str2 = { name: "cxh", sex: "man" };

**obj.toJSONString()**：将JSON对象转化为JSON字符。要引入https://github.com/douglascrockford/JSON-js/blob/master/json.js

**JSON.stringify()** ：将一个JavaScript值(对象或者数组)转换为一个JSON字符串  用于从一个对象解析出字符串 

```js
var a = {a:1,b:2}
JSON.stringify(a)  // 结果：{"a":1,"b":2}
```
当使用ajax作为参数提交的时候需要将contentType设置为"application/json",将提交请求格式设置为post 后台使用@RequestBody接收
[差点因为 JSON.stringify 丢了奖金..](https://mp.weixin.qq.com/s/JDah47ariZY3aerQ97RFKw)

**JSON.parse()** ：用来解析JSON字符串 转换成json对象  相似于$.parseJSON()/jQuery.parseJSON()

```js
var str = '{"name":"huangxiaojian","age":"23"}'  
JSON.parse(str) 

// {age: "23",name: "huangxiaojian",__proto__: Object}
```
注意：单引号写在{}外，每个属性名都必须用双引号，否则会抛出异常。

**Object.assign(target1,target2,...,targetn)：**合并对象，对象合并到第一个对象中，注意目标对象target1也会改变

**join(separator)** ：用于把数组中的所有元素放入一个字符串。separator可选，指定要使用的分隔符。如果省略该参数，则使用逗号作为分隔符。

**split(separator,howmany)** ：用于把一个字符串分割成字符串数组。separator  必需。字符串或正则表达式，从该参数指定的地方分割。howmany 可选。该参数可指定返回的数组的最大长度。如果设置了该参数，返回的子串不会多于这个参数指定的数组。如果没有设置该参数，整个字符串都会被分割，不考虑它的长度。

**splice(index,howmany,item1,.....,itemX)** ：向/从数组中删除项目，然后返回被删除的项目。index必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。howmany必需。要删除的项目数量。如果设置为 0，则不会删除项目。item1, ..., itemX 可选。向数组添加的新项目，在原位置添加。

```js
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2,0,"Lemon","Kiwi");
// fruits 输出结果： Banana,Orange,Lemon,Kiwi,Apple,Mango
```
**unshift(newelement1,newelement2,....,newelementX)** ：可向数组的开头添加一个或更多元素，并返回新的长度。newelement1 必需。向数组添加的第一个元素。newelement2 可选。向数组添加的第二个元素。newelementX 可选。可添加若干个元素

**pop()：**删除原数组最后一项，并返回删除元素的值；如果数组为空则返回undefined 

**shift()** ：用于把数组的第一个元素从其中删除，并返回第一个元素的值

**push(newelement1,newelement2,....,newelementX)** ：可向数组的末尾添加一个或多个元素，并返回新的长度。

**instanceof Array  typeof()：**判断是否为数组 

**concat()：**返回一个新数组，是将参数添加到原数组中构成的

**reverse()：**将数组反序

**sort(orderfunction)：**按指定的参数对数组进行排序

**slice(start,end)：**返回从原数组中指定开始下标到结束下标之间的项组成的新数组。start - 必填；设定新数组的起始位置；如果是负数，则表示从数组尾部开始算起（-1 指最后一个元素，-2 指倒数第二个元素，以此类推）。 end - 可选；设定新数组的结束位置；如果不填写该参数，默认到数组结尾；如果是负数，则表示从数组尾部开始算起（-1 指最后一个元素，-2 指倒数第二个元素，以此类推）

**siblings()：**方法返回被选元素的所有同胞元素。

**prev()：**查找上一个兄弟节点，不是所有的兄弟节点

**prevAll()：**查找所有之前的兄弟节点

**next()：**方法返回被选元素的下一个同胞元素。

**nextAll()：**方法返回被选元素的所有跟随的同胞元素。

**nextUntil()：**方法返回介于两个给定参数之间的所有跟随的同胞元素

**parent(expr)：**找父元素

**parents(expr)：**找到所有祖先元素，不限于父元素

**children(expr)：**查找所有子元素，只会找到直接的孩子节点，不会返回所有子孙

**contents()：**查找下面的所有内容，包括节点和文本。

**slideToggle()：**方法通过使用滑动效果（高度变化）来切换元素的可见状态。如果被选元素是可见的，则隐藏这些元素，如果被选元素是隐藏的，则显示这些元素

**slideUp()：**：方法用于向上滑动元素。

**slideDown()：** 方法用于向下滑动元素

**first()：** 方法返回被选元素的首个元素。

**last()：** 方法返回被选元素的最后一个元素。

**filter()：** 方法允许您规定一个标准。不匹配这个标准的元素会被从集合中删除，匹配的元素会被返回。

**not()：** 方法返回不匹配标准的所有元素

**after()：** 方法在被选元素后插入指定的内容

**clone()：** 方法生成被选元素的副本，包含子节点、文本和属性

**toFixed(num)** ：规定小数的位数，是 0 ~ 20 之间的值，包括 0 和 20，有些实现可以支持更大的数值范围。如果省略了该参数，将用 0 代替。

**substring(start,end)** ：索引从0开始。start必需。一个非负的整数，规定要提取的子串的第一个字符在 stringObject 中的位置。 end可选。一个非负的整数，比要提取的子串的最后一个字符在 stringObject 中的位置多 1。如果省略该参数，那么返回的子串会一直到字符串的结尾。

**substr(start,length)：**start 必需。要抽取的子串的起始下标。必须是数值。如果是负数，那么该参数声明从字符串的尾部开始算起的位置。也就是说，-1 指字符串中最后一个字符，-2 指倒数第二个字符，以此类推。length 可选。子串中的字符数。必须是数值。如果省略了该参数，那么返回从 stringObject 的开始位置到结尾的字串。

**every()：**是对数组中每一项运行给定函数，如果该函数对每一项返回true,则返回true。

**some()：**是对数组中每一项运行给定函数，如果该函数对任一项返回true,则返回true。

```js
var arr = [ 1, 2, 3, 4, 5, 6 ]; 
console.log( arr.some( function( item, index, array ){ 
  console.log( 'item=' + item + ',index='+index+',array='+array ); 
  return item > 3; 

})); // true
console.log( arr.every( function( item, index, array ){ 
  console.log( 'item=' + item + ',index='+index+',array='+array ); 
  return item > 3; 

})); // false
```

**find(selector)：**：获得当前元素集合中每个元素的后代，通过选择器、jQuery 对象或元素来筛选。

**padStart(maxLength, fillString)：**填充字符串 例:2000-1-1变成2000-01-01

**padEnd(maxLength, fillString)**

**setInterval()** ：方法可按照指定的周期（以毫秒计）来调用函数或计算表达式。
setInterval() 方法会不停地调用函数，直到 **clearInterval()** 被调用或窗口被关闭。由 setInterval() 返回的 ID 值可用作 clearInterval() 方法的参数。

**setTimeout()** ：方法用于在指定的毫秒数后调用函数或计算表达式。

**jQuery.extend()：** 函数用于将一个或多个对象的内容合并到目标对象。

1. 如果只为$.extend()指定了一个参数，则意味着参数target被省略。此时，target就是jQuery对象本身。通过这种方式，我们可以为全局对象jQuery添加新的函数。
2. 如果多个对象具有相同的属性，则后者会覆盖前者的属性值。
```js
jQuery.extend({
  min: function(a, b) {
	return a < b ? a : b;
  },
  max: function(a, b) {
	return a > b ? a : b;
  }
});

jQuery.min(2, 3); // 2
jQuery.max(4, 5); // 5

$.fn.extend({
  alertWhileClick: function() {
	$(this).click(function() {
		alert($(this).val());
	});
  }
});
$("#input1").alertWhileClick();
```
jquery.extend()将两个或更多对象的内容合并到第一个对象
jquery.extend(target,object1,...,objectN)

**replaceAll**

```js
var str = "dogdogdog";
var str2 = str.replace(/dog/g,"cat");
```
#### JQuery选择器
[jQuery选择器](https://tool.oschina.net/uploads/apidocs/jquery/)

```js
$("#select option:selected").val();
$("#select option[value='1']").attr("selected",true);
$("p :eq(1)")  // 选择第二个 <p> 元素：
```

#### 获取div class的值
```js
$("#divid").attr('class');// attr基本上选中html原生属性
$("#divid").prop('class');// prop自定义属性
```

#### Map转json

```js
function MapTOJson(m) {
  var str = '{';
  var i = 1;
  m.forEach(function (item, key, mapObj) {
   if(mapObj.size == i){
     str += '"'+ key+'":"'+ item + '"';
   }else{
     str += '"'+ key+'":"'+ item + '",';
   }
   i++;
  });
  str +='}';
  return str;
}
var jsons=JSON.parse(MapTOJson(map));
```

#### if判断
if (jsObj) {  } 过滤
**undefined不能过,null不能过 ,'' 不能过 ,0不能过,{}能过,[] 能过** 
即jsObj != undefined && jsObj != null && jsObj != '' && jsObj != 0

**if(!! jsObj) 与上面等价 可以隐式转换类型**


#### js判断对象是否为空对象的几种方法

1. 将json对象转化为json字符串，再判断该字符串是否为"{}"
```js
var data = {};
var b = (JSON.stringify(data) == "{}");
alert(b);//true
```
2. for in 循环判断
```js
var obj = {};
var b = function() {
for(var key in obj) { 
	return false; 
} 
return true;

alert(b());//true
```
3. jquery的**isEmptyObject**方法

此方法是jquery将2方法(for in)进行封装，使用时需要依赖jquery
```js
var data = {};
var b = $.isEmptyObject(data);
alert(b);//true
```
4. **Object.getOwnPropertyNames()** 方法

此方法是使用Object对象的getOwnPropertyNames方法，获取到对象中的属性名，存到一个数组中，返回数组对象，我们可以通过判断数组的length来判断此对象是否为空。注意：此方法不兼容ie8，其余浏览器没有测试
```js
var data = {};
var arr = Object.getOwnPropertyNames(data);
alert(arr.length == 0);//true
```
5. 使用ES6的**Object.keys()**方法
与4方法类似，是ES6的新方法, 返回值也是对象中属性名组成的数组
```js
var data = {};
var arr = Object.keys(data);
alert(arr.length == 0);//true
```

#### arguments
arguments是js中内置的一个对象数组，存放的是调用函数的参数
arguments.callee()递归函数(callee是arguments内置的一个函数)
```js
function factorial(num) {
  if(num<=1) {
   return 1;
  }else {
   return num * factorial(num-1);
  }
} 
```
相当于
```js
function factorial(num) {
  if(num<=1) {
   return 1;
  }else {
   return num * arguments.callee(num-1);
  }
} 
```
#### 滚动条位置

- scrollTop:获取或设置一个元素的内容垂直滚动的像素数。//页面内容的滚动距离
- scrollHeight:一个元素内容高度的度量，包括由于溢出导致的视图中不可见内容。 //滚动内容的总大小
- clientHeight:元素内部的高度(单位像素)，包含内边距，但不包括水平滚动条、边框和外边距。//可见网页内容高


#### prototype、__proto__与constructor

1. \__proto\__和constructor属性是对象所独有的；prototype属性是函数所独有的，因为函数也是一种对象，所以函数也拥有\__proto\__和constructor属性。

2. \__proto\__属性的作用就是当访问一个对象的属性时，如果该对象内部不存在这个属性，那么就会去它的\__proto\__属性所指向的那个对象（父对象）里找，一直找，直到\__proto\__属性的终点null，再往上找就相当于在null上取值，会报错。通过\__proto\__属性将对象连接起来的这条链路即我们所谓的原型链。大多数情况下，\__proto\__可以理解为“构造器的原型”，即\__proto\__===constructor.prototype,但是通过 Object.create()创建的对象有可能不是， Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的\__proto_\_

3. prototype属性的作用就是让该函数所实例化的对象们都可以找到公用的属性和方法，即f1.\__proto\__ === Foo.prototype。

4. constructor属性的含义就是指向该对象的构造函数，所有函数（此时看成对象了）最终的构造函数都指向Function。constructor 属性的作用，是分辨原型对象到底属于哪个构造函数。


#### Promise

什么时候用
回调地狱时使用，当代码难以维护， 常常第一个的函数的输出是第二个函数的输入这种现象时使用promise。Promise 对象用于表示一个异步操作的最终完成 (或失败)及其结果值
基本用法
demo1
````js
let myFirstPromise = new Promise(function(resolve, reject){
//调用then()时,当异步代码执行成功时，调用resolve(...), 当异步代码失败时,调用reject(...)
  if(){
   resolve(111)
  }else{
   reject(222)
  }
});
myFirstPromise.then(function(successMessage){
//successMessage的值是上面调用resolve(...)方法传入的值.上面不写resolve()的话不会执行这个函数
//successMessage参数不一定非要是字符串类型，也可以是函数
  console.log("Yay! " + successMessage);
},function(failedMessage){
  // failedMessage的值是reject(..)传过来的参数，如果没有在上面写reject()的话不会执行此函数，写上reject()才会执行此快函数
});
```
demo2：
```js
function test(fun) {
  //这里表示执行了一大堆各种代码;
  ...
  // 返回Promise对象
  return new Promise(function(resolve, reject) {
  // 这里也可以执行相关代码
	if (typeof fun== 'function') {
	   resolve(fun);
	 } else {
	   reject('TypeError: '+ fun+'不是一个函数')
	 }
  })
}
// 当fun是一个function时
test(fun).then(function(abc) {// 参数abc是上面resolve(fun)的参数
  abc();
})
test('1234').catch(function(err) {
// 参数err是上面reject('TypeError: '+ fun+'不是一个函数')的参数
  console.log(err);
})
//promise.then(onFulfilled, onRejected)等价于promise.then(onFulfilled).catch(onRejected)
```
[前端基础进阶（十五）：透彻掌握Promise的使用，读这篇就够了](https://www.jianshu.com/p/fe5f173276bd)
[JavaScript Promise 对象](https://www.runoob.com/w3cnote/javascript-promise-object.html)
[mozilla promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
[写给 Java 程序员的前端 Promise 教程](https://mp.weixin.qq.com/s/92AHFPWjtH_y2_88mP3CYQ)


#### js new

帮我们做了这样几件事： 
1. 帮我们创建了一个空对象，作为返回对象的实例 例如：obj；
2. 将空对象原型指向构造函数的property 属性
3. 将这个空对象赋值给函数内部的this
4. 执行构造函数(如果构造函数内部有 return 语句，而且 return 后面跟着一个对象，new 命令会返回 return 语句指定的对象；否则，就会不管 return 语句，返回 this 对象。)
```js
function Foo(name) { 
  this.name = name;
  return this;
}
var obj = {};
obj.__proto__ = Foo.prototype;
var foo = Foo.call(obj, 'mm'); //call会改变this的指向
console.log(foo);
// 等同于
var foo = new Foo('mm')
console.log(foo)

```
[JavaScript：对象都是这样生成的!※(很重要的一篇文章)！](https://mp.weixin.qq.com/s/QJj9TnUKeXGj1HIX039etg)


#### 浏览器展示页面执行顺序

$(document).ready(function) = $().ready(function) = $(function)

当 DOM（文档对象模型） 已经加载(页面所有的html标签（包括图片等）都加载完了，即浏览器已经响应完了，加载完了，全部展现到浏览器界面上了。)，并且页面（包括图像）已经完全呈现时，会发生 ready 事件。
对于一个HTML文档，浏览器的解析顺序为：按照文档流，从上到下逐步解析页面的结构。JavaScript代码作为通过标签嵌入或引入的脚本，也HTML文档的组成部分。因此，JavaScript代码在装载时的执行顺序也是根据脚本标签<script\>的出现顺序来确定的。
但是，浏览器加载JavaScript时有个特点，那就是载入之后立即就会执行（先编译后执行），因为JavaScript可能会影响DOM树的结构，所以浏览器在执行完后才能继续加载下面的HTML内容。也就是说，浏览器下载并执行JavaScript的过程会阻塞DOM树的继续建立。所以，引入的多个js文件，会按顺序分开执行。同样的，对于不同<script\>标签嵌入的JavaScript代码，也会分开执行。同一组<script\>标签包括的代码就是一个代码块。后引入的JavaScript文件可以调用先引入的JavaScript文件的资源，下面的代码块可以访问上面代码块的资源，反之则不行。
由于JavaScript通常需要操作DOM，所以，一般把JavaScript放在</body\>前或者文档结尾处引入。若需要在<head\>中引入，可以通过修改window.onload或者document.ready事件，强制等到DOM加载完成后再执行相关函数。


#### 引用

数字、字符串、布尔类型的为原始类型，是值引用
数组、对象类型为地址引用
值引用 可以深拷贝
地址引用 循环到原始类型方可进行深拷贝

#### http状态码
301永久重定向
302临时重定向
400 bad request ：错误请求，一般是前端提交数据的字段名称或者是字段类型和后台的实体类不一致(后台接收的数据类型不一致)，导致无法封装。或前端提交的到后台的数据应该是json字符串类型，而前端没有将对象转化为字符串类型；
403服务器理解请求但拒绝执行，一般是资源权限问题导致,无权限访问
405 get post 等请求类型错误
406 服务器返回的数据前端无法解析 一般是返回json格式数据前端的Content-Type是text:html
502 bad gateway 错误的网关 上游服务器出现问题
连接超时 我们向服务器发送请求 由于服务器当前链接太多，导致服务器方面无法给于正常的响应，产生此类报错
503过载
504 gateway time-out
程序执行时间过长导致响应超时，例如程序需要执行20秒，而nginx最大响应等待时间为10秒，这样就会出现超时

[http状态码详解](http://tool.oschina.net/commons?type=5)
[请求头响应头等参数详解](https://mp.weixin.qq.com/s/w_FgN5tVGr1DlbqyF5g4gw)

#### GET与POST区别

1. GET在浏览器回退时是无害的，而POST会再次提交请求。
2. GET产生的URL地址可以被Bookmark，而POST不可以。
3. GET请求会被浏览器主动cache，而POST不会，除非手动设置。
4. GET请求只能进行url编码，而POST支持多种编码方式。
5. GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
6. GET请求在URL中传送的参数是有长度限制的，而POST没有。
7. 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
8. GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
9. GET参数通过URL传递，POST放在Request body中。

[听我讲完GET、POST原理，面试官给我倒了杯卡布奇诺](https://mp.weixin.qq.com/s/W68JzNIoUpm9hyXinOzkMw)

#### other

[理解js中this的指向](https://www.cnblogs.com/pssp/p/5216085.html)
[万字干货！详解JavaScript执行过程](https://mp.weixin.qq.com/s/wolPlpUDizVnzh-kBKMtxg)
[面试官：有了 for 循环 为什么还要 forEach ？](https://mp.weixin.qq.com/s/aPFCrPGBTus_Spf1QL1WWA)
[每日一题」JS 中的闭包是什么？](https://zhuanlan.zhihu.com/p/22486908)
[post请求下载excel文档解决方法](https://blog.csdn.net/weixin_44001753/article/details/114266453)
[不懂Vue的Java猿不是一个好的前端工程师](https://mp.weixin.qq.com/s/QoS5z9Qfokpcw42zDbiE2A)
