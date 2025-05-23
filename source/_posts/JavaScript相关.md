---
title: JavaScript相关
categories: JS
tags: 代码实战
index_img: /assert/js.jpg
img: https://picx1.zhimg.com/v2-ff627d205bd26ab15e4bdefd5daf1dda_1440w.jpg

---

## Note

### JQuery、JS常用方法

```js
// json字符串
var str1 = '{"name":"cxh","sex":"man"}';
// json对象
var str2 = {name:"cxh",sex:"man"};
```
**obj.toJSONString()** :将JSON对象转化为JSON字符。

> 要引入https://github.com/douglascrockford/JSON-js/blob/master/json.js

**JSON.stringify()** :将一个JavaScript值(对象或者数组)转换为一个JSON字符串,用于从一个对象解析出字符串

```js
var a = {a:1,b:2}
JSON.stringify(a)  // 结果：{"a":1,"b":2}
```
当使用ajax作为参数提交的时候需要将contentType设置为"application/json",将提交请求格式设置为post后台使用@RequestBody接收

> [差点因为JSON.stringify丢了奖金..](https://mp.weixin.qq.com/s/JDah47ariZY3aerQ97RFKw)

**JSON.parse()** :用来解析JSON字符串,转换成json对象.相似于$.parseJSON()/jQuery.parseJSON()

```js
var str = '{"name":"huangxiaojian","age":"23"}'
JSON.parse(str)

// {age: "23",name: "huangxiaojian",__proto__: Object}
```
注意：单引号写在{}外，每个属性名都必须用双引号，否则会抛出异常。

**Object.assign(target1,target2,...,targetn)** ：合并对象，对象合并到第一个对象中，注意目标对象target1也会改变

**join(separator)** :用于把数组中的所有元素放入一个字符串。separator可选，指定要使用的分隔符。如果省略该参数，则使用逗号作为分隔符。

**split(separator,howmany)** :用于把一个字符串分割成字符串数组。separator必需。字符串或正则表达式，从该参数指定的地方分割。howmany可选。该参数可指定返回的数组的最大长度。如果设置了该参数，返回的子串不会多于这个参数指定的数组。如果没有设置该参数，整个字符串都会被分割，不考虑它的长度。

**splice(index,howmany,item1,.....,itemX)** :向/从数组中删除项目，然后返回被删除的项目。index必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。howmany必需。要删除的项目数量。如果设置为0，则不会删除项目。item1,...,itemX可选。向数组添加的新项目，在原位置添加。

```js
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2,0,"Lemon","Kiwi");
// fruits 输出结果： Banana,Orange,Lemon,Kiwi,Apple,Mango
```
**unshift(newelement1,newelement2,....,newelementX)** :可向数组的开头添加一个或更多元素，并返回新的长度。newelement1必需。向数组添加的第一个元素。newelement2可选。向数组添加的第二个元素。newelementX可选。可添加若干个元素

**pop()** :删除原数组最后一项，并返回删除元素的值；如果数组为空则返回undefined

**shift()** :用于把数组的第一个元素从其中删除，并返回第一个元素的值

**push(newelement1,newelement2,....,newelementX)** :可向数组的末尾添加一个或多个元素，并返回新的长度。

**instanceof Array  typeof()** :判断是否为数组

**concat()** :返回一个新数组，是将参数添加到原数组中构成的

**reverse()** :将数组反序

**sort(orderfunction)** :按指定的参数对数组进行排序

**slice(start,end)** :返回从原数组中指定开始下标到结束下标之间的项组成的新数组。start-必填；设定新数组的起始位置；如果是负数，则表示从数组尾部开始算起（-1指最后一个元素，-2指倒数第二个元素，以此类推）。end-可选；设定新数组的结束位置；如果不填写该参数，默认到数组结尾；如果是负数，则表示从数组尾部开始算起（-1指最后一个元素，-2指倒数第二个元素，以此类推）

**siblings()** :方法返回被选元素的所有同胞元素。

**prev()** :查找上一个兄弟节点，不是所有的兄弟节点

**prevAll()** :查找所有之前的兄弟节点

**next()** :方法返回被选元素的下一个同胞元素。

**nextAll()** :方法返回被选元素的所有跟随的同胞元素。

**nextUntil()** :方法返回介于两个给定参数之间的所有跟随的同胞元素

**parent(expr)** :找父元素

**parents(expr)** :找到所有祖先元素，不限于父元素

**children(expr)** :查找所有子元素，只会找到直接的孩子节点，不会返回所有子孙

**contents()** :查找下面的所有内容，包括节点和文本。

**slideToggle()** :方法通过使用滑动效果（高度变化）来切换元素的可见状态。如果被选元素是可见的，则隐藏这些元素，如果被选元素是隐藏的，则显示这些元素

**slideUp()** :方法用于向上滑动元素。

**slideDown()** :方法用于向下滑动元素

**first()** :方法返回被选元素的首个元素。

**last()** :方法返回被选元素的最后一个元素。

**filter()** :方法允许您规定一个标准。不匹配这个标准的元素会被从集合中删除，匹配的元素会被返回。

**not()** :方法返回不匹配标准的所有元素

**after()** :方法在被选元素后插入指定的内容

**clone()** :方法生成被选元素的副本，包含子节点、文本和属性

**toFixed(num)** :规定小数的位数，是0~20之间的值，包括0和20，有些实现可以支持更大的数值范围。如果省略了该参数，将用0代替。

**substring(start,end)** :索引从0开始。start必需。一个非负的整数，规定要提取的子串的第一个字符在stringObject中的位置。end可选。一个非负的整数，比要提取的子串的最后一个字符在stringObject中的位置多1。如果省略该参数，那么返回的子串会一直到字符串的结尾。

**substr(start,length)** :start必需。要抽取的子串的起始下标。必须是数值。如果是负数，那么该参数声明从字符串的尾部开始算起的位置。也就是说，-1指字符串中最后一个字符，-2指倒数第二个字符，以此类推。length可选。子串中的字符数。必须是数值。如果省略了该参数，那么返回从stringObject的开始位置到结尾的字串。

**every()** :是对数组中每一项运行给定函数，如果该函数对每一项返回true,则返回true。

**some()** :是对数组中每一项运行给定函数，如果该函数对任一项返回true,则返回true。

```js
var arr = [ 1, 2, 3, 4, 5, 6 ];
console.log( arr.some( function( item,index,array){
  console.log( 'item=' + item + ',index='+index+',array='+array );
  return item > 3;

})); // true
console.log( arr.every( function( item, index, array ){
  console.log( 'item=' + item + ',index='+index+',array='+array);
  return item > 3;

})); // false
```

**find(selector)** :获得当前元素集合中每个元素的后代，通过选择器、jQuery对象或元素来筛选。

**padStart(maxLength,fillString)** :填充字符串例:2000-1-1变成2000-01-01

**padEnd(maxLength,fillString)**

**setInterval()** :方法可按照指定的周期（以毫秒计）来调用函数或计算表达式。
setInterval()方法会不停地调用函数，直到**clearInterval()** 被调用或窗口被关闭。由setInterval()返回的ID值可用作clearInterval()方法的参数。

**setTimeout()** :方法用于在指定的毫秒数后调用函数或计算表达式。

**jQuery.extend()** :函数用于将一个或多个对象的内容合并到目标对象。

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
// 去除字符串内所有的空格：
str = str.replace(/\s*/g,"");
// 去除字符串内两头的空格：
str = str.replace(/^\s*|\s*$/g,"");
// 去除字符串内左侧的空格：
str = str.replace(/^\s*/,"");
// 去除字符串内右侧的空格：
str = str.replace(/(\s*$)/g,"");
```

**JS阻止原生态提交事件**

```js
event.preventDefault();
```

**执行函数!function(){}()**

```js
//!function(){}()写法和(function(){})()是相同的,函数意义相同，叫做立即运行的匿名函bai数(也叫立即调用函数)。
//js中可以这样创建一个匿名函数：
(function(){do something...})()
//或
(function(){do something...}())
//而匿名函数后面的小括号()是为了让匿名函数立即执行，其实就是一个函数调用。

alert((new Function("x","y","return x*y;"))(2,3));// "6"
```

### JQuery选择器

> [jQuery选择器](https://tool.oschina.net/uploads/apidocs/jquery/)

```js
// >: 选择某元素后面的第一代子元素div>p选择其父元素是<div>元素的所有<p>元素。
// ~: 选取某个元素之后的所有同级元素,.box~h2这句就是选取.box后面所有的h2,.box和h2同级,并且不需要紧邻
// 空格: 选择某元素后面的所有子元素.不一定是直接子元素.孙子元素也可以
// +: 可选择紧接在另一元素后的兄弟元素，且二者有相同父元素,元素需要紧邻. div + p选择所有紧随<div>元素之后的<p>元素。

$("#select option:selected").val();
$("#select option[value='1']").attr("selected",true);
$("p :eq(1)")  // 选择第二个<p>元素：
```

### 获取div class的值

```js
$("#divid").attr('class');// attr基本上选中html原生属性
$("#divid").prop('class');// prop自定义属性
```

### display属性

- display:block块级元素特点：
1. 每个块级元素都从新的一行开始，并且其后的元素也另起一行。（很霸道，一个块级元素独占一行）
2. 元素的高度、宽度、行高以及顶和底边距都可设置。
3. 元素宽度在不设置的情况下，是它本身父容器的100%（和父元素的宽度一致），除非设定一个宽度。
- display:inline内联元素特点：
1. 和其他元素都在一行上；
2. 元素的高度、宽度及顶部和底部边距不可设置；
3. 元素的宽度就是它包含的文字或图片的宽度，不可改变。
- display:inline-block内联块状元素（inline-block）就是同时具备内联元素、块状元素的特点。
inline-block元素特点：
1. 和其他元素都在一行上；
2. 元素的高度、宽度、行高以及顶和底边距都可设置

### aria-hidden=true

aria-hidden="true"不会导致任何事情,它只是声明作者已经隐藏了该元素.

### 三种方式脱离文档流

```css
position:absolute
position:fixed
float
```

### Map转json

```js
function MapToJson(m) {
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
var jsons=JSON.parse(MapToJson(map));
```

### if(obj)判断

if(jsObj){}过滤**undefined不能过,null不能过 ,''不能过 ,0不能过,{}能过,[]能过**。即`jsObj!=undefined && jsObj!=null && jsObj!='' && jsObj!=0`

**if(!!jsObj)与上面等价 可以隐式转换类型**

**js判断对象是否为空对象的几种方法**
1、将json对象转化为json字符串，再判断该字符串是否为"{}"
```js
var data = {};
var b = (JSON.stringify(data) == "{}");
alert(b);//true
```

2、for in循环判断

```js
var obj = {};
var b = function() {
for(var key in obj) {
	return false; 
} 
return true;

alert(b());//true
```

3、jquery的**isEmptyObject**方法

此方法是jquery将2方法(for in)进行封装，使用时需要依赖jquery
```js
var data = {};
var b = $.isEmptyObject(data);
alert(b);//true
```

4、**Object.getOwnPropertyNames()** 方法

此方法是使用Object对象的getOwnPropertyNames方法，获取到对象中的属性名，存到一个数组中，返回数组对象，我们可以通过判断数组的length来判断此对象是否为空。注意：此方法不兼容ie8，其余浏览器没有测试
```js
var data = {};
var arr = Object.getOwnPropertyNames(data);
alert(arr.length == 0);//true
```

5、使用ES6的**Object.keys()** 方法
与4方法类似，是ES6的新方法,返回值也是对象中属性名组成的数组
```js
var data = {};
var arr = Object.keys(data);
alert(arr.length == 0);//true
```

### 滚动条位置

- scrollTop:获取或设置一个元素的内容垂直滚动的像素数。//页面内容的滚动距离
- scrollHeight:一个元素内容高度的度量，包括由于溢出导致的视图中不可见内容。//滚动内容的总大小
- clientHeight:元素内部的高度(单位像素)，包含内边距，但不包括水平滚动条、边框和外边距。//可见网页内容高

### 浏览器展示页面执行顺序

$(document).ready(function) = $().ready(function) = $(function)
当DOM（文档对象模型）已经加载(页面所有的html标签（包括图片等）都加载完了，即浏览器已经响应完了，加载完了，全部展现到浏览器界面上了。)，并且页面（包括图像）已经完全呈现时，会发生ready事件。对于一个HTML文档，浏览器的解析顺序为：按照文档流，从上到下逐步解析页面的结构。JavaScript代码作为通过标签嵌入或引入的脚本，也HTML文档的组成部分。因此，JavaScript代码在装载时的执行顺序也是根据脚本标签<script\>的出现顺序来确定的。但是，浏览器加载JavaScript时有个特点，那就是载入之后立即就会执行（先编译后执行），因为JavaScript可能会影响DOM树的结构，所以浏览器在执行完后才能继续加载下面的HTML内容。也就是说，浏览器下载并执行JavaScript的过程会阻塞DOM树的继续建立。所以，引入的多个js文件，会按顺序分开执行。同样的，对于不同<script\>标签嵌入的JavaScript代码，也会分开执行。同一组<script\>标签包括的代码就是一个代码块。后引入的JavaScript文件可以调用先引入的JavaScript文件的资源，下面的代码块可以访问上面代码块的资源，反之则不行。由于JavaScript通常需要操作DOM，所以，一般把JavaScript放在</body\>前或者文档结尾处引入。若需要在<head\>中引入，可以通过修改window.onload或者document.ready事件，强制等到DOM加载完成后再执行相关函数。


### 引用

- 数字、字符串、布尔类型的为原始类型，是值引用
- 数组、对象类型为地址引用
- 值引用可以深拷贝
- 地址引用循环到原始类型方可进行深拷贝

### http状态码

- 301永久重定向
- 302临时重定向
- 400 bad request：错误请求，一般是前端提交数据的字段名称或者是字段类型和后台的实体类不一致(后台接收的数据类型不一致)，导致无法封装。或前端提交的到后台的数据应该是json字符串类型，而前端没有将对象转化为字符串类型；
- 403服务器理解请求但拒绝执行，一般是资源权限问题导致,无权限访问
- 405 get post等请求类型错误
- 406 服务器返回的数据前端无法解析,一般是返回json格式数据前端的Content-Type是text:html
- 502 bad gateway:错误的网关,上游服务器出现问题。连接超时 我们向服务器发送请求,由于服务器当前链接太多，导致服务器方面无法给于正常的响应，产生此类报错
- 503过载
- 504 gateway time-out
程序执行时间过长导致响应超时，例如程序需要执行20秒，而nginx最大响应等待时间为10秒，这样就会出现超时

> [http状态码详解](http://tool.oschina.net/commons?type=5)
> [请求头响应头等参数详解](https://mp.weixin.qq.com/s/w_FgN5tVGr1DlbqyF5g4gw)

### GET与POST区别

1. GET在浏览器回退时是无害的，而POST会再次提交请求。
2. GET产生的URL地址可以被Bookmark，而POST不可以。
3. GET请求会被浏览器主动cache，而POST不会，除非手动设置。
4. GET请求只能进行url编码，而POST支持多种编码方式。
5. GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
6. GET请求在URL中传送的参数是有长度限制的，而POST没有。
7. 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
8. GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
9. GET参数通过URL传递，POST放在Request body中。

> [听我讲完GET、POST原理，面试官给我倒了杯卡布奇诺](https://mp.weixin.qq.com/s/W68JzNIoUpm9hyXinOzkMw)

### ||和&&
只要"||"前面为false,不管"||"后面是true还是false，都返回"||"后面的值
只要"||"前面为true,不管"||"后面是true还是false，都返回"||"前面的值

只要"&&"前面是false，无论"&&"后面是true还是false，结果都将返"&&"前面的值
只要"&&"前面是true，无论"&&"后面是true还是false，结果都将返"&&"后面的值

## JS关键字


### arguments

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

### prototype、\__proto\__(两个下划线)与constructor

\__proto\__和constructor属性是对象所独有的；prototype属性是函数所独有的，因为函数也是一种对象，所以函数也拥有\__proto\__和constructor属性。\__proto\__属性的作用就是当访问一个对象的属性时，如果该对象内部不存在这个属性，那么就会去它的\__proto\__属性所指向的那个对象（父对象）里找，一直找，直到\__proto\__属性的终点null，再往上找就相当于在null上取值，会报错。通过\__proto\__属性将对象连接起来的这条链路即我们所谓的原型链。大多数情况下，\__proto\__可以理解为“构造器的原型”，即\__proto\__===constructor.prototype,但是通过Object.create()创建的对象有可能不是，Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的\__proto_\_
prototype属性的作用就是让该函数所实例化的对象们都可以找到公用的属性和方法，即**book1.\__proto\__ === Book.prototype**。
构造函数有什么缺点呢？构造函数的缺点就是会将构造函数内部的对象都复制一份：

```js
function Book(){
    this.name ='www.flydean.com';
    this.getName =function (){
        console.log('flydean');
    }
}
var book1 = new Book();
var book2  = new Book();
console.log(book1.getName  === book2.getName);//false
```
输出结果是false,说明每次new一个对象，对象中的方法也被拷贝了一份。而这并不是必须的。JavaScript的每个对象都继承另一个对象，后者称为“原型”（prototype）对象。只有null除外，它没有自己的原型对象。原型对象上的所有属性和方法，都能被派生对象共享。这就是JavaScript继承机制的基本设计。通过构造函数生成实例对象时，会自动为实例对象分配原型对象。每一个构造函数都有一个prototype属性，这个属性就是实例对象的原型对象。

```js

function Book(name){
    this.name = name;
}
Book.prototype.author ='flydean';
var book1 = new Book();
var book2 = new Book();
console.log(book1.author);//flydean
console.log(book2.author);//flydean
```
上面例子中的author属性会被Book的所有实例所继承，Book的prototype对象，就是book1和book2的原型对象。
原型对象的属性不是实例对象自身的属性。只要修改原型对象，变动就立刻会体现在所有实例对象上。由于原型本身也是对象，又有自己的原型，所以形成了一条原型链（prototype chain）。如果一层层地上溯，所有对象的原型最终都可以上溯到Object.prototype，即Object构造函数的prototype属性指向的那个对象。Object.prototype对象有没有它的原型呢？回答可以是有的，就是没有任何属性和方法的null对象，而null对象没有自己的原型。

```js
console.log(Object.getPrototypeOf(Object.prototype));//null
```
prototype对象有一个constructor属性，默认指向prototype对象所在的构造函数.
```js
function Book(name){
    this.name = name;
}
var book3 = new Book();
console.log(book3.constructor);//function Book(name){this.name = name;}
console.log(book3.constructor === Book.prototype.constructor);//true
console.log(book3.hasOwnProperty(constructor));//false
```
还是刚刚的book，book3.constructor就是function Book本身。它也等于Book.prototype.constructor。constructor属性的含义就是指向该对象的构造函数，所有函数（此时看成对象了）最终的构造函数都指向Function。constructor属性的作用，是分辨原型对象到底属于哪个构造函数。因为prototype是一个对象，所以对象可以被赋值，也就是说prototype可以被改变：

```js
function A(){}
var a = new A();
console.log(a instanceof A);//true
function B(){}
A.prototype = B.prototype;
console.log(a instanceof A);//false
```
上面的例子中，我们修改了A.prototype，最后a instanceof A值是false。为了保证不会出现这样错误匹配的问题，我们再构建prototype的时候，一定不要直接重写整个的prototype，只需要修改其中的某个属性就好:
```js
// 不要这样写
A.prototype = {
    method1:function (){}
}
// 比较好的写法
A.prototype = {
    constructor:A,
    method1:function (){}
}
// 更好的写法
A.prototype.method1 = function (){}
```
#### Object的prototype操作
**Object.getPrototypeOf**方法返回一个对象的原型。这是获取原型对象的标准方法.

```js
// 空对象的prototype是Object.prototype
console.log(Object.getPrototypeOf({}) === Object.prototype);//true
// function的prototype是Function.prototype
function f(){}
console.log(Object.getPrototypeOf(f)  === Function.prototype);//true

function F(){this.name ='flydean'}
var f1 = new F();
console.log(Object.getPrototypeOf(f1) === F.prototype);//true
var f2 = new f();
console.log(Object.getPrototypeOf(f2) === f.prototype);//true
```
**Object.setPrototypeOf**方法可以为现有对象设置原型，返回一个新对象。
Object.setPrototypeOf方法接受两个参数，第一个是现有对象，第二个是原型对象。

```js
var a = {name: 'flydean'};
var b = Object.setPrototypeOf({},a);
console.log(b.name);//flydean
```
**Object.prototype.isPrototypeOf()**
对象实例的isPrototypeOf方法，用来判断一个对象是否是另一个对象的原型.

```js
var a = {name: 'flydean'};
var b = Object.setPrototypeOf({},a);
console.log(a.isPrototypeOf(b));//true
```
**Object.prototype.proto**
proto属性（前后各两个下划线）可以改写某个对象的原型对象。还是刚才的例子，这次我们使用proto来改写对象的原型。

```js
var a = {name: 'flydean'};
var c ={};
c.__proto__ = a;
console.log(Object.getPrototypeOf(c));//{"name": "flydean",[[Prototype]]:object}

//-------
function Book(name){
    this.name = name;
}
console.log(Book.prototype)//{"author": "flydean",constructor:f Book(name)..}
var book1 = new Book();
console.log(book1.__proto__===Book.prototype);//true
```
proto属性只有浏览器才需要部署，其他环境可以没有这个属性，而且前后的两根下划线，表示它本质是一个内部属性，不应该对使用者暴露。因此，应该尽量少用这个属性，而是用Object.getPrototypeof()（读取）和Object.setPrototypeOf()（设置），进行原型对象的读写操作

综上，我们有三种获取原型对象的方法：
```js
obj.proto
obj.constructor.prototype
Object.getPrototypeOf(obj)
```

### Promise

**什么时候用?** 回调地狱时使用，当代码难以维护，常常第一个的函数的输出是第二个函数的输入这种现象时使用promise。Promise对象用于表示一个异步操作的最终完成(或失败)及其结果值
demo1

```js
let myFirstPromise = new Promise(function(resolve,reject){
//调用then()时,当异步代码执行成功时，调用resolve(...),当异步代码失败时,调用reject(...)
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

| [前端基透彻掌握Promise的使用，读这篇就够了](https://www.jianshu.com/p/fe5f173276bd) | [JavaScript Promise对象](https://www.runoob.com/w3cnote/javascript-promise-object.html) | [mozilla promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [写给Java程序员的前端Promise教程](https://mp.weixin.qq.com/s/92AHFPWjtH_y2_88mP3CYQ) |                                                              |                                                              |


### await、async

async是一个修饰符，async定义的函数会默认的返回一个Promise对象resolve的值，因此对async函数可以直接进行then操作,返回的值即为then方法的传入函数
```js
// 0. async基础用法测试
async function fun0() {
    console.log(1)
    return 1
}
fun0().then( x => { console.log(x) })  //  输出结果 1， 1，

async function funa() {
    console.log('a')
    return 'a'
}
funa().then( x => { console.log(x) })  //  输出结果a， a，

async function funo() {
    console.log({})
    return {}
}
funo().then( x => { console.log(x) })   // 输出结果 {}  {}

async function funp() {
    console.log('Promise')
    return new Promise(function(resolve, reject){
        resolve('Promise')
    })
}

funp().then( x => { console.log(x) })   // 输出promise  promise
```
await也是一个修饰符，await操作符用于等待一个Promise对象。它只能在异步函数async function中使用(await关键字只能放在async函数内部)，await关键字的作用就是返回Promise对象的处理结果，如果await后面并不是一个Promise的返回值，则会按照同步程序返回该值本身
```js
//  await关键字只能放在async函数内部，await关键字的作用就是获取Promise中返回的内容，获取的是Promise函数中resolve或者reject的值
// 如果await后面并不是一个Promise的返回值，则会按照同步程序返回值处理,为undefined
const bbb = function(){ return 'string'}

async function funAsy() {
   const a = await 1
   const b = await new Promise((resolve, reject)=>{
        setTimeout(function(){
           resolve('time')
        }, 3000)
   })
   const c = await bbb()
   console.log(a, b, c)
}
funAsy()  //  运行结果是3秒钟之后输出1，time,string,
```

那么由此看来async/await的综合用法如下

```js
// 1.定义一个或多个普通函数，函数必须返回Promise对象，如果返回其他类型的数据，将按照普通同步程序处理
function log(time) {
    return new Promise((resolve, reject)=> {
        setTimeout(function(){
           console.log(time)
           resolve()
        }, time)
    })
}
async function fun() {
    await log(5000)
    await log(10000)
    log(1000)
    console.log(1)
}
fun()

// 2.如果不使用promise的方法的话
function log2(time) {
   setTimeout(function(){
       console.log(time)
       return 1
    }, time)
}
async function fun1() {
    const a = await log2(5000)
    const b = await log2(10000)
    const c = log2(2000)
    console.log(a)
    console.log(1)
}

fun1()
// 以上运行结果为：立刻输出undefined 立刻输出1  2秒后输出2000  5秒后输出5000  10秒后输出10000

// 3.async/await的重要应用 
const asy = function(x, time) {
    return new Promise((resolve, reject) =>{
        setTimeout(()=>{
            resolve(x)
        }, time)
    })
}
const add = async function() {
    const a = await asy(3, 5000)
    console.log(a)
    const b = await asy(4, 10000)
    console.log(b)
    const c =  await asy(5, 15000)
    console.log(a,b,c)
    const d = a + b +c  
    console.log(d)
}
add();
// 5秒后输出3  又10秒后输出4 又15秒后输出5  然后立刻输出3,4,5，然后输出12
```

### js new

帮我们做了这样几件事：
1. 帮我们创建了一个空对象，作为返回对象的实例,例如：obj；
2. 将空对象原型指向构造函数的property属性
3. 将这个空对象赋值给函数内部的this
4. 执行构造函数(如果构造函数内部有return语句，而且return后面跟着一个对象，new命令会返回return语句指定的对象；否则，就会不管return语句，返回this对象。)
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
> [JavaScript：对象都是这样生成的!※(很重要的一篇文章)！](https://mp.weixin.qq.com/s/QJj9TnUKeXGj1HIX039etg)


### JS this

this总是返回一个对象，简单说，就是返回属性或方法“当前”所在的对象。
```js
var book = {
    name :'flydean',
    getName : function (){
        return '书名：'+ this.name;
    }
}
console.log(book.getName());
//书名：flydean
```
这里this的指向是可变的，我们看一个例子：
```js
var book = {
    name :'flydean',
    getName : function (){
        return '书名：'+ this.name;
    }
}
var car ={
    name :'car'
}
car.getName = book.getName;
console.log(car.getName());
//书名：car
```
当A对象的方法被赋予B对象，该方法中的this就从指向A对象变成了指向B对象
上面的例子中，我们把book中的getName方法赋值给了car对象，this对象现在就指向了car。如果某个方法位于多层对象的内部，这时this只是指向当前一层的对象，而不会继承更上面的层。

```js
var book1 = {
    name :'flydean',
    book2: {
        getName : function (){
            return '书名：'+ this.name;
        }
    }
}
console.log(book1.book2.getName());
//书名：undefined
```
上面的例子中，this是定义在对象中的函数中，如果是在函数中的函数中定义的this，代表什么呢？
```js
var book3 = {
    name :'flydean',
    book4: function(){
        console.log('book4');
        var getName = function (){
            console.log(this); //Window
        }();
    }
}
book3.book4();
```
如果在函数中的函数中使用了this，那么内层的this指向的是全局的window对象。所以我们在使用的过程中要避免多层this。由于this的指向是不确定的，所以切勿在函数中包含多层的this。如果在全局环境使用this，它指的就是顶层对象window。
数组的map和foreach方法，允许提供一个函数作为参数。这个函数内部不应该使用this。
```js
var book5 ={
    name : 'flydean',
    author : ['max','jacken'],
    f: function (){
        this.author.forEach(function (item) {
            console.log(this.name+' '+item);
        })
    }
}
book5.f();
//undefined max
//undefined jacken
```
foreach方法的回调函数中的this，其实是指向window对象，因此取不到o.v的值。原因跟上一段的多层this是一样的，就是内层的this不指向外部，而指向顶层对象。怎么解决呢？我们使用一个中间变量：
```js
var book6 ={
    name : 'flydean',
    author : ['max','jacken'],
    f: function (){
        var that = this;
        this.author.forEach(function (item) {
            console.log(that.name+' '+item);
        })
    }
}
book6.f();
//flydean max
//flydean jacken
```
或者将this当作foreach方法的第二个参数，固定它的运行环境：
```js
var book7 ={
    name : 'flydean',
    author : ['max','jacken'],
    f: function (){
        this.author.forEach(function (item) {
            console.log(this.name+' '+item);
        },this)
    }
}
book7.f();
//flydean max
//flydean jacken
```
绑定this的方法,JavaScript提供了call、apply、bind这三个方法，来切换/固定this的指向.

**call**
函数实例的call方法，可以指定函数内部this的指向（即函数执行时所在的作用域），然后在所指定的作用域中，调用该函数.

```js
var book = {};
var f = function () {
    return this;
}
f()  === this ; //true
f.call(book) === book; //true
```
上面例子中，如果直接调用f()，那么返回的就是全局的window对象。如果传入book对象，那么返回的就是book对象。call方法的参数，应该是一个对象。如果参数为空、null和undefined，则默认传入全局对象。如果call方法的参数是一个原始值，那么这个原始值会自动转成对应的包装对象，然后传入call方法。

```js
var f = function () {
    return this;
}
console.log(f.call(100));
//[Number: 100]
```
call方法还可以接受多个参数。func.call(thisValue,arg1,arg2,...);call的第一个参数就是this所要指向的那个对象，后面的参数则是函数调用时所需的参数。call一般用在调用对象的原始方法：
```js
var person =  {};
person.hasOwnProperty('getName');//false
//覆盖person的getName方法
person.getName  = function(){
    return true;
}
person.hasOwnProperty('getName');//true
Object.prototype.hasOwnProperty.call(person,'getName');//false
```
**apply**
apply方法的作用与call方法类似，也是改变this指向，然后再调用该函数。唯一的区别就是，它接收一个数组作为函数执行时的参数.
```js
func.apply(thisValue,[arg1,arg2,...])
```
**bind**
call和apply是改变this的指向，然后调用该函数，而bind方法用于将函数体内的this绑定到某个对象，然后返回一个新函数.
```js
var d = new Date();
console.log(d.getTime()); //1600755862787
var getTime= d.getTime;
console.log(getTime());//TypeError: this is not a Date object.
```
上面的例子中，getTime方法里面调用了this，如果直接把d.getTime赋值给getTime变量，那么this将会指向全局的window对象，导致运行错误。我们可以这样修改：
```js
var d = new Date();
console.log(d.getTime()); //1600755862787
var getTime2= d.getTime.bind(d);
console.log(getTime2());
```
bind比call方法和apply方法更进一步的是，除了绑定this以外，还可以绑定原函数的参数。
```js
var add = function(x,y){
    return x +this.m +  y + this.n;
}
var addObj ={
    m: 10,
    n: 10
}
var newAdd = add.bind(addObj,2);
console.log(newAdd(3));//25
```
上面的例子中，bind将两个参数的add方法，替换成了1个参数的add方法。注意：bind每次调用都会返回一个新的函数，从而导致无法取消之前的绑定。

> [理解js中this的指向](https://www.cnblogs.com/pssp/p/5216085.html)
> [连this都搞不懂，我直接pass](https://mp.weixin.qq.com/s/wSzDsLCIEuK4ES1F2P-tCQ)


### class

ES6的class可以看作只是一个语法糖，它的绝大部分功能，ES5都可以做到，新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。
```js
class Person {
    constructor(name,sex) {
        this.name=name;
        this.sex =sex;
    }
    toString(){
        return this.name + ' '+ this.sex;
    }
}
```
构造函数的prototype属性，在ES6的“类”上面继续存在。事实上，类的所有方法都定义在类的prototype属性上面。
上面的类等同于：
```js
Person.prototype = {
       constructor(name,sex) {
        this.name=name;
        this.sex =sex;
    }
    toString(){
        return this.name + ' '+ this.sex;
    } 
}
```
表达式属性名
class还支持动态的表达式属性名：
```js
let methodName = 'getName';
class Person {
    constructor(name,sex) {
        this.name=name;
        this.sex =sex;
    }

    toString(){
        return this.name + ' '+ this.sex;
    }

    [methodName](){
        return this.name;
    }
}
```
静态方法
类相当于实例的原型，所有在类中定义的方法，都会被实例继承。如果在一个方法前，加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。
```js
class Person {
    constructor(name,sex) {
        this.name=name;
        this.sex =sex;
    }
    static getSex(){
        return '男';
    }
}
console.log(Person.getSex()); //男
let  p  = new Person();
console.log(p.getSex());//TypeError: p.getSex is not a function
```
静态属性
静态属性指的是Class本身的属性，即Class.propName，而不是定义在实例对象（this）上的属性.
```js
class Person {
    constructor(name,sex) {
        this.name=name;
        this.sex =sex;
    }
}
Person.address ='address';
console.log(Person.address);
```
目前，只有这种写法可行，因为ES6明确规定，Class内部只有静态方法，没有静态属性。

class的继承
class 的继承一般使用extends关键字：
```js
class Boy extends Person{
    constructor(name,sex,address) {
        super(name,sex); //调用父类的构造函数
        this.address =address;
    }
    toString() {
        return super.toString();//调用父类的方法
    }
}
```
在子类的构造函数中，只有调用super之后，才可以使用this关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有super方法才能返回父类实例。super作为函数调用时，代表父类的构造函数。ES6要求，子类的构造函数必须执行一次super函数。super作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。上面的例子，我们在子类Boy中的toString普通方法中，调用了super.toString()，之前我们也讲了，类的所有方法都定义在类的prototype属性上面。所以super.toString就是Person中定义的toString方法。由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的。定义在父类实例上的方法或属性就是指在constructor中定义的方法或者属性。Person 类，在constructor中定义了name属性。我们看一下在Boy中的普通方法中访问会有什么问题：
```js
class Boy extends Person{
    constructor(name,sex,address) {
        super(name,sex); //调用父类的构造函数
        console.log(super.name);  //undefined
        console.log(this.name);  //hanmeimei
        this.address =address;
    }
    toString() {
        return super.toString();//调用父类的方法
    }
    getName(){
        console.log(super.name);  //undefined
        console.log(this.name);    //hanmeimei
    }
}
var b =new Boy('hanmeimei','女','北京');
b.getName();
```

### 继承

构造函数的继承。
构造函数的继承第一步是在子类的构造函数中，调用父类的构造函数,让子类实例具有父类实例的属性。然后让子类的原型指向父类的原型，这样子类就可以继承父类原型。

```js
function Person (){
    this.name = 'person';
}

function Boy(){
    Person.call(this);
    this.title = 'boy';
}

Boy.prototype= Object.create(Person.prototype);
Boy.prototype.constructor=Boy;
Boy.prototype.getTitle=function (){console.log(this.title)};

var b =new Boy();
b.getTitle();
console.log(b);
```
调用父类的构造函数是初始化实例对象的属性。子类的原型指向父类的原型是为了基础父类的原型对象的属性。
另外一种写法是Boy.prototype等于一个父类实例：
```js
Boy.prototype = new Person();
```
上面这种写法也有继承的效果，但是子类会具有父类实例的方法。有时，这可能不是我们需要的，所以不推荐使用这种写法.
JavaScript不提供多重继承功能，即不允许一个对象同时继承多个对象。但是，可以通过变通方法，实现这个功能:
```js
function Person1 (){
    this.name = 'person';
}
function Person2 (){
    this.sex = '男';
}

function Boy(){
    Person1.call(this);
    Person2.call(this);
    this.title = 'boy';
}

//继承Person1
Boy.prototype= Object.create(Person1.prototype);
//继承链加上Person2
Object.assign(Boy.prototype,Person2.prototype);

Boy.prototype.constructor=Boy;
Boy.prototype.getTitle=function (){console.log(this.title)};

var b =new Boy();
b.getTitle();
console.log(b);
//Boy { name: 'person', sex: '男', title: 'boy' }
```

### spread运算符(扩展运算符)

扩展运算符`...`是ES6中引入的一个新运算符，它可以将一个数组或者类数组对象展开（或者说“拍扁”）成一系列单独的值，或者将多个值合并成一个数组

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

## 相关文章

| [Web开发技术(mozilla)](https://developer.mozilla.org/zh-CN/docs/Web) | [万字干货！详解JavaScript执行过程](https://mp.weixin.qq.com/s/wolPlpUDizVnzh-kBKMtxg) | [20分钟全面理解JavaScript事件机制](https://mp.weixin.qq.com/s/ct4AyU--sewtOaDOlO2Oxw) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [面试官：有了for循环为什么还要forEach？](https://mp.weixin.qq.com/s/aPFCrPGBTus_Spf1QL1WWA) | [每日一题」JS中的闭包是什么？](https://zhuanlan.zhihu.com/p/22486908) | [post请求下载excel文档解决方法](https://blog.csdn.net/weixin_44001753/article/details/114266453) |
| [不懂Vue的Java猿不是一个好的前端工程师](https://mp.weixin.qq.com/s/QoS5z9Qfokpcw42zDbiE2A) | [前端面试复习笔记](https://github.com/CavsZhouyou/Front-End-Interview-Notebook) | [当对象遇上代码：JavaScript中的浪漫邂逅](https://mp.weixin.qq.com/s/Sn8r_3pVN7EZOyF4Fhg-zw) |
| [接口一异常你的前端页面就直接崩溃了？](https://mp.weixin.qq.com/s/ze3c1N_qLeg1rvK3kamFTQ) | [??=操作符，更优雅的空值处理方式](https://mp.weixin.qq.com/s/jNIpCp73BAEAS-PFM0Fh-w) | [JavaScript的12大“天坑”！90%程序员都踩过](https://mp.weixin.qq.com/s/XPNn-5e1Gi_wIyjYG4z4EA) |