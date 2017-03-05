# jQuery源码6-数据缓存、队列、toggle



## 变量



#### cssExpand

```javascript
var cssExpand = ["Top","Right","Bottom","Left"];
```



---



#### defaultDisplayMap

```javascript
var defaultDisplayMap = {};
```



---





## 正则



#### pnum

```javascript
var pnum = /[+-]?(?:\d*\.|)\d+(?:[eE][+-]?\d+|)/;
```





---



#### rcssNum

```javascript
var rcssNum = /^(?:([+-])=|)([+-]?(?:\d*\.|)\d+(?:[eE][+-]?\d+|))([a-z%]*)$/i;
```



---



## 野方法





#### adjustCSS

```javascript
function adjustCSS(elem,prop,valueParts,tween){
  var adjusted,
      scale = 1,
      maxIterations = 20,
      currentValue = tween ? function(){
        return tween.cur();
      } : function(){
        return jQuery.css(elem,prop,"");
      },
      initial = currentValue(),
      unit = valueParts && valueParts[3] || (jQuery.cssNumber[prop]) ? "" : "px",
      initialInUnit = (jQuery.cssNumber[prop] || unit !== "px" && +initial) && rcssNum.exec(jQuery.css(elem,prop));

  if(initialInUnit && initialInUnit[3] !== unit){
    unit = unit || initialInUnit[3];
    valueParts = valueParts || [];
    initialInUnit = +initial || 1;

    do{
      scale = scale || ".5";
      initialInUnit = initialInUnit / scale;
      jQuery.style(elem,prop,initialInUnit + unit);
    }while(scale !== 
           (scale = currentValue() / initial) && 
           scale !== 1 && --maxIterations );
  }
  
  if(valueParts){
    initialInUnit = +initialInUnit || +initial || 0;
    
    adjusted = valueParts[1] ? 
      initialInUnit + (valueParts[1] + 1) * valueParts[2] : 
      +valueParts[2];
    if(tween){
      tween.unit = unit;
      tween.start = initialInUnit;
      tween.end = adjusted;
    }
  }
  return adjusted;
}
```



---



#### swap

```javascript
var swap = function(elem,options,callback,args){
  var ret,name,
      old = {};
  
  //将elem指定的style属性放进old对象
  for(name in options){
    old[name] = elem.style[name];
    elem.style[name] = options[name];
  }
  
  //执行回调函数callback传入参数args或者空数组
  ret = callback.apply(elem,args || []);
  
  //将elem的style属性复原
  for(name in options){
    elem.style[name] = old[name];
  }
  
  return ret;
}
```





## Data缓存函数



#### 变量



##### dataPriv

```javascript
//缓存上一个变量
var dataPriv = new Data();
```



------



##### dataUser

```javascript
var dataUser = new Data();
```





#### 正则



##### rbrace

```javascript
//匹配{}或[]
var rbrace = /^(?:\{[\w\W]*\}|\[[\w\W]*\])$/;
```



---



##### rmultiDash

```javascript
var rmultiDash = /[A-Z]/g;
```





#### Data()

```javascript
//创建唯一标记expando!
function Data(){
  this.expando = jQuery.expando + Data.uid++;
}
Data.uid = 1;
```



---



#### Data.prototype

```javascript
Data.prototype = {}
```



---



##### access

```javascript
access: function(owner,key,value){
  //未传key、value
  //或者key为string且未传value
  //获得key对应的值
  if(key === undefined || 
     ((key && typeof key === "string") && 
      value === undefined)){
    return this.get(owner,key);
  }
  //三个参数时设置缓存
  this.set(owner,key,value);
  //返回value的值是否有意义
  return value !== undefined;
}
```



---



##### cache

```javascript
//创建缓存
cache: function(owner){
  //查询当前元素是否已经有缓存对象
  var value = owner[this.expando];

  //没有就创建一个
  if(!value){
    value = {};
    //有些元素不能创建缓存对象
    if(acceptData(owner)){
      if(owner.nodeType){
        owner[this.expando] = value;
      }
      //属性不可枚举时调用Object原型方法设置
      //为保证属性可删除 configurable必须为true
      else{
        Object.defineProperty(owner,this.expando,{
          value:value,
          configurable:true
        });
      }
    }
  }
  return value;
}
```



---



##### hasData

```javascript
//查询是否有缓存数据
hasData: function(owner){
  var cache = owner[this.expando];
  return cache !== undefined && !jQuery.isEmptyObject(cache);
}
```



---



##### get

```javascript
get: function(owner,key){
  //如果不传key创建缓存
  return key === undefined ? this.cache(owner) : 
  //否则返回key键的值
  owner[this.expando] && owner[this.expando][jQuery.camelCase(key)];
}
```



------



##### set

```javascript
set: function(owner,data,value){
  var prop,
      cache = this.cache(owner);
  
  //传入data与value
  if(typeof data === "string"){
    //驼峰命名
    cache[jQuery.camelCase(data)] = value;
  }
  //传入data对象
  else{
    for(prop in data){
      cache[jQuery.camelCase[prop]] = data[prop];
    }
  }
  return cache;
}
```



---



##### remove

```javascript
remove: function(owner,key){
  var i,
      cache = owner[this.expando];
  
  //没缓存对象直接返回
  if(cache === undefined){
    return 
  }
  //传进key参数时
  if(key !== undefined){
    //key为数组
    if(jQuery.isArray(key)){
      //驼峰化
      key = key.map(jQuery.camelCase);
    }
    else{
      key = jQuery.camelcase(key);
      key = key in cache ? [key] : 
      //key有空格就返回空格 否则返回空数组
      (key.match(rnothtmlwhite) || []);
    }
    i = key.length;
    while(i--){
      delete cache[key[i]];
    }
  }
  //不传key且缓存对象为空
  //删掉缓存对象
  if(key === undefined || jQuery.isEmptyObject(cache)){
    if(owner.nodeType){
      owner[this.expando] = undefined;
    }
    else{
      delete owner[this.expando];
    }
  }
}
```



#### 功能方法



##### getData

```javascript
function getData(data){
  if(data === "true"){
    return true;
  }
  if(data === "false"){
    return false;
  }
  if(data === "null"){
    return null;
  }
  //data为纯数字
  if(data === +data + ""){
    return data;
  }
  //{},[]
  if(rbrace.test(data)){
    return JSON.parse(data);
  }
  return data;
}
```



---



##### dataAttr

```javascript
//自定义属性
function dataAttr(elem,key,data){
  var name;
  //不传第三个参数且elem为DOM节点
  if(data === undefined && elem.nodeType === 1){
    //$&: 与RegExp匹配的子串  rmutiDah: = /[A-Z]/g
    //驼峰化 camelCase --> camel-case
    name = "data-" + key.replace(rmultiDash,"-$&").toLowerCase();
    //获取自定义属性值
    data = elem.getAttribute(name);
    if(typeof data === "string"){
      try{
        data = getData(data);
      }
      catch(e){}
      //缓存对象
      dataUser.set(elem,key,data);
    }
    else{
      data = undefined;
    }
  }
  return data;
}
```



#### jQuery.extend



##### hasData

```javascript
hasData: function(elem){
  return dataUser.hasData(elem) || dataPriv.hasData(elem);
}
```



---



##### data

```javascript
data: function(elem,name,data){
  return dataUser.access(elem,name,data);
}
```



---



##### removeData

```javascript
removeData: function(elem,name){
  dataUser.remove(elem,name);
}
```



---



##### _data

```javascript
_data: function(elem,name,data){
  return dataPriv.access(elem,name,data);
}
```



---



##### _removeData

```javascript
_removeData: function(elem,name){
  dataPriv.remove(elem,name);
}
```



#### jQuery.fn.extend



##### data

```javascript
data: function(key,value){
  var i,name,data,
      //elem为调用函数的DOM节点
      elem = this[0],
      //获取节点属性
      attrs = elem && elem.attributes;
  
  //不传参数时
  if(key === undefined){
    //节点存在
    if(this.length){
      //获取该节点的缓存对象
      data = dataUser.get(elem);
      //节点类型为元素且没有hasDataAttrs属性时
      if(elem.nodeType === 1 && !dataPriv.get(elem,"hasDataAttrs")){
        i = attrs.length;
        while(i--){
          //IE-11 only
          if(attrs[i]){
            //样式名 style、id、class等
            name = attrs[i].name;
            //存在自定义属性data0-*时
            if(name.indexOf("data") === 0){
              //去掉前缀并进行驼峰处理
              name = jQuery.camelCase(name.slice(5));
              dataAttr(elem,name,data[name]);
            }
          }
        }
        //设置"hasDataAttrs"属性为true
        dataPriv.set(elem,"hasDataAttrs",true);
      }
    }
    return data;
  }
  
  //传入key为对象
  if(typeof key === "object"){
    return this.each(function(){
      dataUser.set(this,key);
    });
  }
  
  //
  return access(this,function(value){
    var data;
    if(elem && value === undefined){
      //从缓存中寻找key的值
      data = dataUser.get(elem,key);
      if(data !== undefined){
        return data;
      }
      //从h5默认自定义标签寻找key的值
      data = dataAttr(elem,key);
      if(data !== undefined){
        return data;
      }
      //这里的翻译超萌
      return;
    }
    //设置缓存
    this.each(function(){
      dataUser.set(this,key,value);
    });
    //这么多参数是想干什么???
  },null,value,arguments.length > 1,null,true);
}
```



---



##### removeData

```javascript
removeData: function(key){
  return this.each(function(){
    dataUser.remove(this,key);
  });
}
```







## toggle效果



#### 主方法



##### showHide

```javascript
function showHide(elements,show){
  var display,elem,
      values = [],
      index = 0,
      length = elements.length;
  
  //遍历数组
  for(;index < length; index++){
    elem = elements[index];
    //元素没有style属性不操作
    if(!elem.style){
      continue;
    }
    
    //保存当前display属性
    display = elem.style.display;
    //show
    if(show){
      //
      if(display === "none"){
        //先尝试从缓存中获取
        values[index] = datePriv.get(elem,"display") || null;
        //缓存中没有 格式化elem的display属性
        if(!values[index]){
          elem.style.display = "";
        }
      }
      if(elem.style.display === "" && isHiddenWithinTree(elem)){
        //设置节点原始display属性
        values[index] = getDefaultDisplay(elem);
      }
    }
    //hide
    else{
      if(display !== "none"){
        values[index] = "none";
        //在elem元素上缓存节点当前display属性
        dataPriv.set(elem,"display",display);
      }
    }
  }
  
  //设置完变化属性后遍历数组
  for(index = 0;index < length;index++){
    //没有style属性的元素value[index]为undefined
    if(values[index] != null){
      elements[index].style.display = value[index];
    }
  }
  return elements;
}
```



------



#### 辅助方法



##### getDefaultDisplay

```javascript
//获得节点默认display属性
function getDefaultDisplay(elem){
  var temp,
      doc = elem.ownerDocument,
      nodeName = elem.nodeName,
      //缓存对象
      display = defaultDisplayMap[nodeName];
	
  //有缓存直接返回
  if(display){
    return display;
  }
	
  //创建节点并用jQuery方法获取display属性
  temp = doc.body.appendChild(doc.createElement(nodeName));
  display = jQuery.css(temp,"display");
	
  //移除节点
  temp.parentNode.removeChild(temp);
	
  //什么节点的display出来就是none???
  if(display === "none"){
    display = "block";
  }
  
  //加入缓存
  defaultDisplayMap[nodeName] = display;

  return display;
}
```



------



##### isHiddenWithinTree

```javascript
//判断节点是否diaplay === "none"...
var isHiddenWithinTree = function(elem,el){
  //函数被$.filter()调用时 elem为第二个参数
  elem = el || elem;
  		
  //行内样式
  return elem.style.display === "none" || 
    //style样式
    //写在style标签内的样式不会被style属性获取
    elem.style.display === "" &&
    //fragElement的display会被计算为none
    //必须保证elem节点在document中
    jQuery.contains(elem.ownerDocument,elem) && 
    jQuery.css(elem,"display") === "none";
}
```



---





#### jQuery.extend



##### show

```javascript
show: function(){
  return showHide(this,true);
}
```



------



##### hide

```javascript
hide: function(){
  return showHide(this);
}
```



------



##### toggle

```javascript
toggle: function(state){
  //传进布尔值
  if(typeof state === "boolean"){
    return state ? this.show() : this.hide();
  }
  
  return this.each(function(){
    if(isHiddenWithinTree(this)){
      jQuery(this).show();
    }
    else{
      jQuery(this).hide();
    }
  });
}
```



------



## queue队列



#### jQuery.extend



##### queue

```javascript
queue: function(elem,type,data){
  var queue;
  if(elem){
    //默认	type = "fxqueue" 
    //有传参  type = type + "queue"
    type = (type || "fx") + "queue";
    //尝试获取缓存中的queue值
    queue = dataPriv.get(elem,type);
    
    //如果传了data参数
    if(data){
      //未获取到queue值或者传入data参数为数组
      if(!queue || jQuery.isArray(data)){
        //设置对应缓存对象
        queue = dataPriv.access(elem,type,jQuery.makeArray(data));
      }
      else{
        queue.push(data);
      }
    }
    //返回queue或空数组
    return queue || [];
  }
}
```



------



##### dequeue

```javascript
dequeue: function(elem,type){
  type = type || "fx";
  var queue = jQuery.queue(elem,type),
      startLength = queue.length,
      fn = queue.shift(),
      hooks = jQuery._queueHooks(elem,type),
      next = function(){
        jQuery.dequeue(elem,type);
      };
  
  //
  if(fn === "inprogress"){
    fn = queue.shift();
    startLength--;
  }
  
  if(fn){
    //
    if(type = "fx"){
      queue.shift("inprogress");
    }
    
    //
    delete hooks.stop;
    fn.call(elem,next,hooks);
  }
  
  if(!startLength && hooks){
    hooks.empty.fire();
  }
}
```



---





##### _queueHooks

```javascript
_queueHooks: function(elem,type){
  var key = type + "queueHooks";
  return dataPriv.get(elem,key) || dataPriv.access(elem,key,{
    empty: jQuery.Callbacks("once memory").add(function(){
      dataPriv.remove(elem,[type + "queue",key]);
    })
  });
}
```





#### jQuery.fn.extend





##### queue

```javascript
queue: function(type,data){
  var setter = 2;
  if(typeof type !== "string"){
    data = type;
    type = "fx";
    setter--;
  }
  
  if(arguments.length < setter){
    return jQuery.queue(this[0],type);
  }
  
  return data === "undefined" ? 
    this : 
  this.each(function(){
    var queue = jQuery.queue(this,type,data);
    //调用钩子函数做兼容
    jQuery._queueHooks(this,type);
    if(type === "fx" && queue[0] !== "inprogress"){
      jQuery.dequeue(this,type);
    }
  });
}
```



------



##### dequeue

```javascript
dequeue: function(type){
  return this.each(function(){
    jQuery.dequeue(this,type);
  });
}
```



---



##### clearQueue

```javascript
clearQueue: function(type){
  return this.queue(type || "fx",[]);
}
```



