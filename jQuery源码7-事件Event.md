# jQuery源码7-事件Event



##  事件





### 正则



#### rmouseEvent

```javascript
//鼠标事件
var rmouseEvent = /^(?:mouse|pointer|contextmenu|drag|drop)|click/;
```





---



#### rkeyEvent

```javascript
//键盘事件
var rkeyEvent = /^key/;
```



---



#### rtypenamespace

```javascript
var rtypenamespace = /^([^.]*)(?:\.(.+)|)/;
```



---



### jQuery.fn.extend



```javascript
jQuery.fn.extend({});
```



---



#### on

```javascript
on: function(types,selector,data,fn){
  return on(this,types,selector,data,fn);
}
```



---



#### one

```javascript
one: function(types,selector,data,fn){
  return on(this,types,selector,data,fn,1);
}
```



---



#### off

```javascript
off: function(types,selector,fn){
  var handleObj,type;
  if(types && types.preventDefault && types.handleObj){
    //
    handleObj = types.handleObj;
    jQuery(types.delegateTarget).off(handleObj.namespace ? 
                                     handleObj.origType + "." + handleObj.namespace : 
                                     handleObj.origType,handleObj.selector,handleObj.handler);
    return this;
  }
  if(typeof types === "object"){
    for(type in types){
      this.off(type,selector,types[type]);
    }
    return this;
  }
  if(selector === false || typeof selector === "function"){
    fn = selector;
    selector = undefined;
  }
  if(fn === false){
    fn = returnFalse;
  }
  return this.each(function(){
    jQuery.event.remove(this,types,fn,selector);
  });
}
```





---



### 主方法



#### on

```javascript
//参数列表
//elem: DOM节点
//types: 事件类型
//[selector]: 过滤器
//[data]: 传递给回调函数的缓存对象
//fn: 事件触发后的回调函数
//one: 事件是否只绑定一次
function on(elem,types,selector,data,fn,one){
  var origFn,type;
  
  //多事件绑定
  if(typeof types === "object"){
    if(typeof selector !== "string"){
      data = data || selector;
      selector = undefined;
    }
    
    //遍历types对象 递归执行函数
    //参数形式为$().on({type:fn,type2:fn2})
    for(type in types){
      on(elem,type,selector,data,types[type],one);
    }
    return elem;
  }
  
  //只传2个参数时
  if(data == null && fn == null){
    //第二个参数当成fn
    fn = selector;
    data = selector = undefined;
  }
  //只传3个参数时
  else if(fn == null){
    //若selector为字符串
    if(typeof selector === "string"){
      //第三个参数当成fn
      fn = data;
      data = undefined;
    }
    //否则
    else{
      //第三个参数还是fn
      fn = data;
      //第二个参数当成data
      data = selector;
      selector = undefined;
    }
  }
  //fn传入false的话 
  if(fn === false){
    //变成一个fn(){return false}的函数
    fn = returnFalse;
  }
  //只传了一个参数时 直接返回元素本身
  else if(!fn){
    return elem;
  }
  //one一次绑定
  if(one === 1){
    //将fn赋值给临时变量
    origFn = fn;
    //解除事件绑定并执行一次事件
    fn = function(event){
      jQuery().off(event);
      return origFn.apply(this,arguments);
    };
    fn.guid = origFn.guid || (origFn.guid = jQuery.guid++);
  }
  //相当于调用add(elem,types,fn,data,selector)
  return elem.each(function(){
    jQuery.event.add(this,types,fn,data,selector);
  });
}
```



------



#### 原生参数添加

```javascript
jQuery.each({
  altKey: true,
  bubbles: true,
  cancelable: true,
  changedTouches: true,
  ctrlKey: true,
  detail: true,
  eventPhase: true,
  metaKey: true,
  pageX: true,
  pageY: true,
  shiftKey: true,
  view: true,
  "char": true,
  charCode: true,
  button: true,
  buttons: true,
  clientX: true,
  clientY: true,
  offsetX: true,
  offsetY: true,
  pointerId: true,
  pointerType: true,
  screenX: true,
  screenY: true,
  targetTouches: true,
  toElement: true,
  touches: true,
  
  which: function(event){
    var button = event.button;
    //键盘事件
    if(event.which == null && rkeyEvent.test(event.type)){
      return event.charCode != null ? event.charCode : event.keyCode;
    }
    //
    if(!event.which && button !== undefined && rmouseEvent.test(event.type)){
      if(button & 1){
        return 1;
      }
      if(button & 2){
        return 3;
      }
      if(button & 4){
        return 2;
      }
      return 0;
    }
    return event.which;
  }
},jQuery.event.addProp);
```



---



#### 鼠标事件绑定

```javascript
jQuery.each({
  mouseenter: "mouseover",
  mouseleave: "mouseout",
  //
  pointerenter: "pointerover",
  pointerleave: "pointerout"
},function(orig,fix){
  jQuery.event.special[orig] = {
    delegateType: fix,
    bindType: fix,
    handle: function(event){
      var ret,
          target = this,
          related = event.relateTarget,
          handleObj = event.handleObj;
      
      //
      if(!related || (related !== target && !jQuery.contains(target,related))){
        event.type = handleObj.origType;
        ret = handleObj.handler.apply(this,arguments);
        event.type = fix;
      }
      return ret;
    }
  }
})
```



---



### jQuery.Event.prototype



#### 属性

```javascript
jQuery.Event.prototype = {
  constructor: jQuery.Event,
  isDefaultPrevented: returnFalse,
  isPropagationStopped: returnFalse,
  isImmediatePropagationStopped: returnFalse,
  isSimulated: false
}
```



---



#### preventDefault

```javascript
preventDefault: function(){
  var e = this.originalEvent;
  this.isDefaultPrevented = returnTrue;
  if(e && !this.isSimulated){
    e.preventDefault();
  }
}
```



---



#### stopPropagation

```javascript
stopPropagation: function(){
  var e = this.originalEvent;
  this.isPropagationStopped = returnTrue;
  if(e && !this.isSimulated){
    e.stopPropagation();
  }
}
```



---



#### stopImmediatePropagation

```javascript
stopImmediatePropagation: function(){
  var e = this.originalEvent;
  this.isImmediatePropagationStopped = returnTrue;
  if(e && !this.isSimulated){
    e.stopImmediatePropagation();
  }
  this.stopPropagation();
}
```







### jQuery方法



#### Event

```javascript
jQuery.Event = function(src,props){
  //初始化不需要new关键字
  if(!(this instanceof jQuery.Event)){
    return new jQuery.Event(src,props);
  }
  
  //事件对象
  if(src && src.type){
    this.originalEvent = src;
    this.type = src.type;
    //
    this.isDefaultPrevented = src.defaultPrevented || 
      src.defaultPrevented === undefined && 
      src.returnValue === false ? 
      returnTrue : returnFalse;
    //
    this.target = (src.target && src.target.nodeType === 3) ? 
      src.target.parentNode : src.target;
    this.currentTarget = src.currentTarget;
    this.relatedTarget = src.relatedTarget;
  }
  //事件类型
  else{
    this.type = src;
  }
  
  //
  if(props){
    jQuery.extend(this,props);
  }
  
  //
  this.timeStamp = src && timeStamp || jQuery.now();
  //
  this[jQuery.expando] = true;
}
```





#### removeEvent

```javascript
jQuery.removeEvent = function(elem,type,handle){
  //判断节点是否有该方法
  if(elem.removeEventListener){
    elem.removeEventListener(type,handle);
  }
}
```





### jQuery.event(非公共接口)



#### global对象

```javascript
jQuery.event = {global:{}};
```



----



#### special对象

```javascript
special: {};
```



##### load

```javascript
load: {
  noBubble:true
}
```



##### focus

```javascript
focus: {
  trigger: function(){
    if(this !== safeActiveElement() && this.focus){
      this.focus();
      return false;
    }
  },
  delegateType: "focusin"
}
```



##### blur

```javascript
blur: {
  trigger: function(){
    if(this === safeActiveElement() && this.blue){
      this.blur();
      return false;
    }
  },
  delegateType: "focusout"
}
```



##### click

```javascript
click: {
  trigger: function(){
    if(this.type === "checkbox" && this.click && jQuery.nodeName(this,"input")){
      this.click();
      return false;
    }
  }
}
```



##### beforeunload

```javascript
beforeunload: {
  postDispatch: function(event){
    if(event.result !== undefined && event.originalEvent){
      event.originalEvent.returnValue = event.result;
    }
  }
}
```



---



#### add

```javascript
//参数列表
//elem: 绑定事件的DOM节点
//types: 事件类型
//handler: 事件触发后的回调函数
//data: 传递给回调函数的缓存对象
//selector: 过滤器
add: function(elem,types,handler,data,selector){
  var handleObjIn,eventHandle,tmp,
      event,t,handleObj,
      special,handler,type,namespaces,origType,
      //创建缓存函数
      elemData = dataPriv.get(elem);
  
  //没缓存不玩
  if(!elemData){
    return ;
  }
  
  //
  if(handler.handler){
    handleObjIn = handler;
    handler = handleObjIn.handler;
    selector = handleObjIn.selector;
  }
  
  //如果传入过滤器 执行过滤
  if(selector){
    jQuery.find.matchesSelector(documentElement,selector);
  }
  
  //one方法跳过来的fn才有guid
  if(!handler.guid){
    handler.guid = jQuery.guid++;
  }
  
  //缓存对象中是否有对应对象
  //elemData = {events:{},handle:fn(){}}
  if(!(events = elemData.events)){
    events = eleData.events = {};
  }
  if(!(eventHanle = elemData.handle)){
    eventHandle = elemData.handle = function(e){
      //页面未加载完成不执行事件
      return typeof jQuery !== "undefined" && jQuery.event.triggered !== e.type ? 
        //这个函数太长了吧
        jQuery.event.dispatch.apply(elem,arguments) : undefined;
    };
  }
  //事件类型字符串去空格
  //on方法支持多事件绑定 用空格分割 on("click mouseenter",fn)
  types = (types || "").match(rnothtmlwhite) || [""];
  t = types.length;
  while(t--){
    //rtypenamespace = /^([^.]*)(?:\.(.+)|)/ 
    //匹配点开头且后面有字符 cli.ck --> [cli,ck]
    tmp = rtypenamespace.exec(types[t]) || [];
    //反正type就是事件类型
    type = origType = tmp[1];
    namespace = (tmp[2] || "").split(".").sort();
    //没事件继续循环
    if(!type){
      continue;
    }
    
    //处理某些事件的特殊情况
    special = jQuery.event.special[type] || {};
    
    //...
    type = (selector ? special.delegateType : special.bindType) || type;
    
    //
    special = jQuery.event.special[type] || {};
    
    //事件属性集合
    handleObj = jQuery.extend({
      type: type,
      origType: origType,
      data: data,
      handler: handler,
      guid: hanlder.guid,
      selector: selector,
      needsContext: selector && jQuery.expr.match.needsContext.test(selector),
      namespace: namespace.join(".")
    },handleObjIn);
    
    //
    if(!(handlers = event[type])){
      handlers = events[type] = [];
      handlers.delegateCount = 0;
      
      //
      if(!special.setup || special.setup.call(elem,data,namespaces,eventHandle) === false){
        if(elem.addEventListener){
          elem.addEventListener(type,eventHandle);
        }
      }
    }
    
    //
    if(special.add){
      special.add.call(elem,handleObj);
      if(!handleObj.handler.guid){
        handleObj.handler.guid = handler.guid;
      }
    }
    
    //
    if(selector){
      handlers.splice(handlers.delegateCount++,0,handleObj);
    }
    else{
      handlers.push(handleObj);
    }
    //
    jQuery.event.global[type] = true;
  }
}
```



---



#### addProp

```javascript
addProp: function(name,hook){
  Object.defineProperty(jQuery.Event.prototype,name,{
    enumerable: true,
    configurable: true,
    get: jQuery.isFunction(hook) ? function(){
      if(this.originalEvent){
        return hook(this.originalEvent);
      }
    } : function(){
      if(this.originalEvent){
        return this.originalEvent[name];
      }
    },
    set: function(value){
      Object.defineProperty(this,name,{
        enumerable: true,
        configurable: true,
        writable:true,
        value: value
      });
    }
  });
}
```



---



#### dispatch

```javascript
dispatch: function(nativeEvent){
  //
  var event = jQuery.event.fix(nativeEvent);
  var i,j,ret,matched,handleObj,handlerQueue,
      args = new Array(arguments.length),
      handlers = (dataPriv.get(this,"events") || {})[event.type] || [],
      special = jQuery.event.special[event.type] || [];
  
  args[0] = event;
  for(i = 1;i < arguments.length;i++){
    args[i] = arguments[i];
  }
  
  event.delegateTarget = this;
  
  //
  if(special.preDispatch && special.preDispatch.call(this,event) === false){
    return;
  }
  
  //
  handlerQueue = jQuery.event.handlers.call(this,event,handlers);
  
  //
  i = 0;
  while((matched = handlerQueue[i++]) && !event.isPropagationStropped()){
    event.currentTarget = matched.elem;
    j = 0;
    while((handleObj = matched.handler[j++]) && !event.isImmediatePropagationStopped()){
      if(!event.rnamespace || event.rnamespace.test(handleObj.namespace)){
        event.handleObj = handleObj;
        evnet.data = handleObj.data;
        
        ret = ((jQuery.event.special[handleObj.origType] || {}).handle || 
               handleObj.handler).apply(matched.elem,args);
        if(ret !== undefined){
          if((event.result = ret) === false){
            event.preventDefault();
            event.stopPropagation();
          }
        }
      }
    }
  }
  
  //
  if(special.postDispatch){
    special.postDispatch.call(this,event);
  }
  
  return event.result;
}
```



---



#### fix

```javascript
fix: function(originalEvent){
  return originalEvent[jQuery.expando] ? 
    originalEvent : new jQuery.Event(originalEvent);
}
```



---



#### handlers

```javascript
handlers: function(event,handlers){
  var  i,handleObj,sel,matchedHandlers,matchedSelectors,
      handlerQueue = [],
      delegateCount = handlers.delegateCount,
      cur = event.target;
  
  //
  if(delegateCount && cur.nodeType && 
     !(event.type === "click" && event.button >= 1)){
    for(;cur !== this;cur = cur.parentNode || this){
      //
      if(cur.nodeType === 1 && !(event.type === "click" && cur.disabled === true)){
        matchedHandlers = [];
        matchedSelectors = {};
        for(i = 0;i < delegateCount;i++){
          handleObj = handler[i];
          //
          sel = handleObj.selector + " ";
          if(matchedSelector[sel] === undefined){
            matchedSelector[sel] = handleObj.needsContext ? 
              jQuery(sel,this).index(cur) > -1 : 
            jQuery.find(sel,this,null,[cur]).length;
          }
          if(matchedSelectors[sel]){
            matchedHandlers.push(handleObj);
          }
        }
        if(matchedHandlers.length){
          handlerQueue.push({elem:cur,handlers:matchedHandlers});
        }
      }
    }
  }
  
  //
  cur = this;
  if(delegateCount < handlers.length){
    handlerQueue.push({elem:cur,handlers:handlers.slice(delegateCount)});
  }
  return handlerQueue;
}
```



---



#### remove

```javascript
remove: function(elem,types,handler,selector,mappedTypes){
  var j,origCount,tmp,
      events,t,handleObj,
      special,handlers,type,namespaces,origType,
      elemData = dataPriv.hasData(elem) && dataPriv.get(elem);
  
  if(!elemData || !(events = elemData.events)){
    return;
  }
  
  //
  types = (types || "").match(rnothtmlwhite) || [""];
  t = types.length;
  while(t--){
    tmp = rtypenamespace.exec(types[t]) || [];
    type = origType = tmp[1];
    namespace = (tmp[2] || "").split(".").sort();
    
    if(!type){
      for(type in events){
        jQuery.event.remove(elem,type + types[t],handler,selector,true);
      }
      continue;
    }
    
    special = jQuery.event.special[type] || {};
    type = (selector ? special.delegateType : special.bindType) || type;
    handlers = events[type] || [];
    tmp = tmp[2] && new RegExp( "(^|\\.)" + namespaces.join( "\\.(?:.*\\.|)" ) + "(\\.|$)" );
    
    //
    origCount = j = handlers.length;
    while(j--){
      handleObj = handlers[j];
      if((mappedTypes || origType === handleObj.origType) && 
         (!handler || handler.guid === handleObj.guid) && 
         (!tmp || tmp.test(handleObj.namespace)) && 
         (!selector || selector === handleObj.selector || selector === "**" && handleObj.selector)){
        handlers.splice(j,1);
        
        if(handleObj.selector){
          handlers.delegateCount--;
        }
        if(special.remove){
          special.remove.call(elem,handleObj);
        }
      }
    }
    
    //
    if(origCount && !handlers.length){
      if(!special.teardown || special.teardown.call(elem,namespaces,elemData.handle) === false){
        jQuery.removeEvent(elem,type,elemData.handle);
      }
      delete events[type];
    }
  }
  
  //
  if(jQuery.isEmptyObject(events)){
    dataPriv.remove(elem,"handle events");
  }
}
```







### 辅助方法



#### returnTrue

```javascript
function returnTrue(){
  return true;
}
```



------



#### returnFalse

```javascript
function returnFalse(){
  return false;
}
```



#### safeActiveElement

```javascript
//兼容IE<=9
function safeActiveElement(){
  try{
    return document.activeElement;
  }
  catch(err){}
}
```

















