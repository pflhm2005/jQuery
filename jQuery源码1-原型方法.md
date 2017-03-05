

# 敲敲jQuery源码-1(原型方法)



#### 类数组判断：isArrayLike

```javascript
//这个方法很好骗 不过也还好
function isArrayLike(obj){
  //1.判断是否是空值
  //2.判断对象是否有length属性
  //3.赋值
  var length=!! obj && "length" in obj && obj.length;
  //如果是函数或者全局对象 返回false
  if(type==="function"||jQuery.isWindow(obj)){
    return false;
  }
  //由于&&优先级较高 添加括号理解 两个条件
  //1.类型是数组、length===0
  //2.length为数字、长度大于0、length-1也是obj的属性(若存在a[1]=="...",那么a[0]也要有意义)
  return (type==="array" || length===0) || (typeof length==="number" && length>0 && (length-1) in obj)
}
```



---



#### 自定义的变量

```javascript
var arr=[];	//空数组对象
var document=window.document,
    getProto=Object.getPrototypeOf,	//取得对象的原型对象
    slice=arr.slice,				//截取数组
    concat=arr.concat,				//拼接数组
    push=arr.push,					//弹入元素
    indexOf=arr.indexOf,			//查询字符串
    
var class2type={};	//空对象 空你妈 后面偷偷做处理 日了狗
//特殊处理函数
//第一个参数输出[Boolean,Number,...,Symbol]
//回调函数参数中 i对应索引 name对应数组元素
jQuery.each( "Boolean Number String FuncArray Date RegExp Object Error Symbol".split( " " ),function(i,name){
  //给class2type添加对应属性
  //class2type = {
  //	[object Boolean] : "boolean",
  //	[object Number] : "number",
  //	...
  //	[object Symbol] : "symbol"
  //}
  class2type["[object"+name+"]"] = name.toLowerCase();
})
var toString=class2type.toString,		//略
    hasOwn=class2type.hasOwnProperty,	//本身是否有该属性
    fnToString=hasOwn.toString,			//true或false的字符串
    ObjectFunctionString=fnToString.call(Object);	//略

//这个东西是用来做兼容 已经被弃用
var support={};

//创建script标签 内容为code
//添加到head标签再移除 仅执行一次的脚本
function DOMEval(code,doc){
  doc = doc || document;
  var script = doc.createElement('script');
  script.text = code;
  doc.head.appendChild(script).parentNode.removeChild(script);
}
```



---



#### 正则表达式


```javascript
var rtrim = /^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g,	//匹配头尾各种空格
    rmsPrefix = /^-ms-/,							//除-ms-以外的字符
	rdashAlpha = /-([a-z])/g;						//横杠与小写字母
//驼峰转换函数
var fcamelCase = function(all,letter){
  return letter.toUpperCase();
}
```



---



#### 原型方法 jQuery.prototype



---

```javascript
var jQuery.fn = jQuery.prototype = {
  //版本
  jquery : version,
  //更改constructor指向
  constructor:jQuery
}
```



---

##### end

```javascript
//返回上一个操作点 可以进行链式操作
end: function(){
  return this.prevObject||this.constructor();
}
```



---

##### eq

```javascript
//堆栈弹入第i个元素并返回 
eq: function(){
  var len = this.length,
      j = +i+(i<0?len:0);
  return this.pushStack(j>0&&j<len?[this[j]]:[]);
}
```



---

##### get

```javascript
//取出jQuery对象数组中指定项
get: function(num){
  //不传入参数返回所有的数组
  if(num==null){
    return slice.call(this);
  }
  //返回数组第num项 负数倒着取
  return num<0 ? this[num+this.length] : this[num];
}
```

---

##### pushStack

```javascript
//设置回溯属性
//堆栈弹入新的jQuery对象elems
//并设置elems对象的prevObject属性值为调用对象
//this.constructor()是一个空jQuery对象
//example:
//var div=$('div'),span=$('span'),div.pushStack(span)	
//输出为span且div.pushStack(span).prevObject属性为div 不设置prevObject默认为document.
pushStack: function(elems){
  var ret = jQuery.merge(this.constructor(),elems);
  ret.prevObject = this;
  return ret;
}
```



---

##### toArray

```javascript
//返回一个jQuery对象组成的数组
toArray: function(){
  return slice.call(this);	
}
```





---



#### jQuery.extend方法



```javascript
//jQuery扩展函数参数配置与一些方法
var jQuery.extend ({
  //确保唯一
  expando: "jQuery" + (version + Math.random()).replace(/\D/g,""),
  //此配置不需要$().ready()
  isReady:true,  
});
```



---

##### camelCase

```javascript
//驼峰命名
//$.camelCase('show-me') --> showMe
camelCase: function(string){
  //var rmsPrefix = /^-ms-/;					//-ms-开头的文本
  //var rdashAlpha = /-([a-z])/g;				//匹配(-)与小写字母
  //关于fcamelCase函数详情见上面
  //该方法接受2个参数 第一个参数为匹配到的文本 第二个参数为()中匹配的文本
  //例如：匹配到-a 第二个参数为小写字母 将其转化为大写实现驼峰命名
  return string.replace( rmsPrefix,"ms-" ).replace( rdashAlpha,fcamelCase );
}
```

---

##### each

```javascript
//遍历类数组或对象的元素 
//第一个参数为对象 第二个参数为回调函数
//回调函数第一个参数为索引 第二个为索引对应的值
each: function(obj,callback){
  var length, i = 0;
  //类数组
  if(isArrayLike(obj)){
    length = obj.length;
    //将索引和对应的值作为参数传入回调函数callback
    for(; i < length ; i++ ){
      if( callback.call( obj[i], i, obj[i] ) === false ){
        break;
      }
    }
  }
  //对象
  else{
    for( i in obj ){
      if( callback.call( obj[i], i, obj[i]) === false ){
        break;
      }
    } 
  } 
  return obj;
}
```



---

##### error

```javascript
//抛出对应错误
error: function (msg){
  throw new Error(msg);
}
```



---

##### globalEval

```javascript
//相当于eval() 全局调用
globalEval: function(code){
  DOMEval(code);
}
```



---

##### grep

```javascript
//过滤
//第一个参数为待过滤数组 第二个为过滤函数 第三个参数选择返回条件
grep: function( elems, callback, invert){
  var callbackInverse,
      matches = [],
      i = 0,
      length = elems.length,
      callbackExpect = !invert;
  for( ; i < length; i++){
    //满足条件的元素
    callbackInverse = !!callback( elem[i], i);
    //如果第三个参数为false 返回被过滤的元素
    if( callbackInverse == callbackExpect ){
      matches.push( elems[i] );
    }
  }
  return matches;
}
```



---

##### guid

```javascript
//全局变量
guid: 1
```



---

##### inArray

```javascript
//查询数组元素
//第一个参数为要查询的 第二个为数组 第三个为开始查找的索引
inArray: function(elem,arr,i){
  return arr == null ? -1 : indexOf.call( arr, elem, i);
}
```



---

##### isEmptyObject

```javascript
//判断是否空对象
//可以用Object.defineProperty()设定enumerable:false骗过检测
isEmptyObject: function(obj){
  var name;
  //对象存在可枚举属性 返回false
  for( name in obj ){
    return false;
  }
}
```

---

##### isFunction

```javascript
isFunction: function(){
  return jQuery.type(obj) === "function";
}
```

---

##### isNumeric

```javascript
//判断是否是数字
isNumeric: function(obj){
  var type = jQuery.type(obj);
  //如果是字符串 判断是否能转换成数字
  return ( type === "number" || type === "string" ) && 
    !isNaN( obj - parseFloat(obj) );
}
```



---

##### isPlainObject

```javascript
//判断是否是纯粹对象 即通过{}或new Object()创建
isPlainObject: function(obj){
  var proto,Ctor;
  //不是对象返回false
  if( !obj || toString.call(obj) !== "[object Object]" ){
    return false;
  }
  //获取原型对象
  proto = getProto(obj);
  //如果没对象没有原型 返回true
  //例如：var obj = Object.create(null)
  if( !proto ){
    return true;
  }
  //先判断原型对象是否有constructor属性
  //并将Ctor设置为obj的构造函数
  //严格来讲 是设置为原型对象的constructor属性值
  Ctor = hasOwn.call( proro,"constructor" ) && proto.constructor;
  //如果Ctor是函数且为Object构造函数
  //fnToString.call(Ctor) 输出原型构造函数
  //ObjectFunctionString  输出Object构造函数
  return typeof Ctor === "function" && 
    fnToString.call(Ctor) === ObjectFunctionString;
}
```



---

##### isWindow

```javascript
//判断是否是window对象
//因为window对象的window属性指向自身 即window === window.window
isWindow: function(obj){
  return obj != null && obj === obj.window;
}
```

---

##### nodeName

```javascript
//判断节点名是否是name
nodeName: function(elem,name){
  return elem.nodeName && elem.nodeName.toLowerCase() === name.toLowerCase();
}
```

---

##### makeArray

```javascript
//拼接返回新数组[results,arr]
makeArray: function(arr,results){
  //第二个参数给ret
  var ret = results || [];
  if( arr != null ){
    //Object()方法包装arr对象
    //例如：arr="jimmy" 调用Object(arr)
    //包装后为一个String对象 String={0:"j",...,4:"y"}
    if( isArrayLike( Object(arr) )){
      //如果是字符串 包装成数组合并到ret
      jQuery.merge(ret,typeof arr === "string" ? [arr] : arr);
    }
    //如果不是类数组 强行推入ret
    else{
      push.call( ret, arr);
    }
  }
  return ret;
}
```

---

##### map

```javascript
//遍历并调用回调函数处理每一个元素
//返回一个处理后元素的数组
//基本类似于each 多了一个value值与ret数组
map: function( elems, callback, arg){
  var length, value,
      i = 0,
      ret = [];
  if( isArrayLike( elems )){
    length = elems.length;
    for( ; i<length; i++){
      value = callback( elems[i], i, arg);
      if(value != null){
        ret.push( value );
      }
    }
  }
  else{
    for( i in elems ){
      value = callback( elems[i], i, arg );
      if(value != null){
        ret.push( value );
      }
    }
  }
  return concat.apply([],ret);
}
```

---

##### merge

```javascript
//传入2个数组 第二个数组拼接到第一个
merge: function(first,second){
  var len = +second.length,
      j = 0,
      i = first.length;
  for(;j<len;j++){
    first[i++] = second[j];
  }
  first.length = i;
  return first;
}
```



---

##### noop

```javascript
//空函数
noop: function(){}
```



---

##### now

```javascript
now: Date.now
```



---

##### proxy

```javascript
//硬绑定?显式绑定?
proxy: function( fn, context ){
  var tmp, args, proxy;
  //第二个参数为字符串时 反转两个参数
  if( typeof context === "string" ){
    tmp = fn[ context ];
    context = fn;
    fn = tmp;
  }
  //判断fn能否调用call方法(即是否是函数)
  if( !jQuery.isFunction(fn)){
    return undefined;
  }
  //第二个开始后面所有的参数
  args = slice.call( arguments,2 );
  //执行后将fn的this指向强制绑定到context
  proxy = function(){
    return fn.apply( context || this, args.concat( slice.call(arguments)));
  }
  //计数?
  proxy.guid = fn.guid = fn.guid || jQuery.guid++;
  return proxy;
}
```

---

##### support

```javascript
//jQuery.support内部已弃用 某些依赖的框架可能需要 所以写一下
support: support;
```



---

##### trim

```javascript
//去除首尾空格
//rtrim = /^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g	匹配头尾所有空格  
trim: function(text){
  return text == null ? "" : (text + "").replace(rtrim,"");
}
```

---

##### type

```javascript
//类型判断
type: function(obj){
  //如果是null或undefined 返回这两个的字符串
  if( obj == null ){
    return obj + "";
  }
  //1.先判断是否是基本数据类型 如果是直接返回typeof obj
  //2.否则返回{}.toString.call(obj) 或 "object"
  //然而什么东西不吃{}.toString.call()?
  //这个class2type做了特殊处理 不是空对象 对应方法见顶部
  return typeof obj === "object" || 
    typeof obj === "function" ? 
    class2type[ toString.call(obj) ] || "object": typeof obj;
}
```

---

