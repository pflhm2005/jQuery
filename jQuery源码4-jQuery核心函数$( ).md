



# jQuery源码4-jQuery对象初始化



## jQuery引用Sizzle方法

```javascript
jQuery.find = Sizzle;
jQuery.expr = Sizzle.selectors;

//不赞成?
jQuery.expr[":"] = jQuery.expr.pseudos;
jQuery.uniqueSort = jQuery.unique = Sizzle.uniqueSort;
jQuery.text = Sizzle.getText;
jQuery.isXMLDoc = Sizzle.isXML;
jQuery.contains = Sizzle.contains;
jQuery.escapeSelector = Sizzle.escape;
```



---



## $(' ')



#### main

```javascript
//初始化一个jQuery对象
var rootjQuery,
    //快速匹配<tag>...,#id
    rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]+))$/,
    init = jQuery.fn.init = function( selector, context, root ){
      var match,elem;
      
      //当selector为 "" null undefined false 时
      //直接返回空jQuery对象
      if( !selector){
        return this;
      }
      
      //doucment
      root = root || rootjQuery;
      
      //选择器为字符串
      if(typeof selector === "string"){
        //如果字符串构成类似于 <\w+>
        if(selector[0] === "<" &&
          selector[selector.length-1] === ">" && 
          selector.length >= 3){
            match = [ null, selector, null ];
          }
        //否则调用正则快速匹配
        //match = [匹配表达式,<tag>,id]
        else{
          match = rquickExpr.exec( selector );
        }
        
        //匹配成功
        if(match && (match[1] || !context)){
          //匹配到tag标签
          if(match[1]){
            context = context instanceof jQuery ? context[0] : context;
            //将字符串解析成DOM节点集合合并到当前对象
            //例如: $("<div></div><span></span>")
            //match[1] = "<div></div><span></span>"
            //解析后会创建div和span两个节点
            //this合并节点
            jQuery.merge(this,jQuery.parseHTML(
              match[1],	//"<div></div>"
              //document
              context && context.nodeType ? 
              context.ownerDocument || context : document,
              true
            ));
            
            //正则:/^<([a-z][^\/\0>:\x20\t\r\n\f]*)[\x20\t\r\n\f]*\/?>(?:<\/\1>|)$/i
            //判断是不是一个单独的标签 
            //例如:"<div></div>"
            if(rsingleTag.test(match[1]) && jQuery.isPlainObject(context)){
              for(match in context){
                if(jQuery.isFunction( this[match] )){
                  this[match]( context[match] );
                }
                else{
                  this.attr( match, context[match] );
                }
              }
            }
            //返回合并后的对象
            return this;
          }
          //匹配ID
          else{
            elem = document.getElementById( match[2] );
            if(elem){
              this[0] = elem;
              this.length = 1;
            }
            return this;
          }
        }
        //匹配:$(expr,$(...)) 调用$(...).find(expr)
        else if( !context || context.jQuery){
          return ( context || root ).find( selector );
        }
        //匹配:$(expr,context) 调用$(context).find(expr)
        else{
          return this.constructor(context).find(selector);
        }
      }
      //DOM节点 包装成伪数组返回
      else if( selector.nodeType ){
        this[0] = selector;
        this.length = 1;
        return this;
      }
      //函数 执行函数
      else if(jQuery.isFunction(selector)){
        return root.ready !== undefined ? 
          root.ready( selector ) : selector( jQuery );
      }
      return jQuery.makeArray( selector, this );
    }

init.prototype = jQuery.fn;
//初始化核心引用...
rootjQuery = jQuery( document );
```







#### 变量 







##### wrapMap

```javascript
var wrapMap = {
  //IE<=9
  option: [1,"<select mutiple = 'multiple'>","</select>"],
  thead: [1,"<table>","</table>"],
  col: [2,"<table><colgroup>","</colgroup></table>"],
  tr: [2,"<table><tbody>","</tbody></table>"],
  td: [3,"<table><tbody><tr>","</tr></tbody></table>"],
  
  _default: [0,"",""]
};
```



---



##### dataPriv

```javascript
var dataPriv = new Data();
```



---



##### dataUser

```javascript
var dataUser = new Data();
```



## 正则



---

#### 环视

| 名字   | 语法       |
| ---- | -------- |
| 右肯定  | (?=...)  |
| 右否定  | (?!...)  |
| 左肯定  | (?<=...) |
| 左否定  | (?<!...) |
|      |          |

#### replace替换文本



| 字符          | 替换文本                      |
| ----------- | ------------------------- |
| \$1 \$2 ... | 与RegExp中第1，2...个子表达式匹配的文本 |
| \$&         | 与RegExp匹配的子串              |
| \$`         | 位于匹配子串左侧的文本               |
| \$'         | 位于匹配子串右侧的文本               |
| \$\$        | 直接量符号                     |
|             |                           |



#### rhtml

```javascript
var rhtml = /<|&#?\w+;/;
```



---



#### risSimple

```javascript
var risSimple = /^.[^:#\[\.,]*$/;
```



---



#### rneedsContext

```javascript
//等于:/^[\x20\t\r\n\f]*[>+~]|:(even|odd|eq|gt|lt|nth|first|last)(?:\([\x20\t\r\n\f]*((?:-\d)?\d*)[\x20\t\r\n\f]*\)|)(?=[^-]|$)/i
var rneedsContext = jQuery.expr.match.needsContext;
```



---



#### rparentsprev

```javascript
//?:代表不捕获括号内容
var rparentsprev = /^(?:parents|prev(?:Until|All))/
```



---



#### rsingleTag

```javascript
//快速匹配标签
var rsingleTag = /^<([a-z][^\/\0>:\x20\t\r\n\f]*)[\x20\t\r\n\f]*\/?>(?:<\/\1>|)$/i;
```



---



#### rscriptType

```javascript
var rscriptType = /^$|\/(?:java|ecma)script/i;
```



---



#### rtagName

```javascript
var rtagName = /<([a-z][^\/\0>\x20\t\r\n\f]+)/i;
```



---



## 功能方法





#### buildFragment

```javascript
//elems = ["<div></div><span></span>"]
//context = document 
//script = false
function buildFragment(elems,context,scripts,selection,ignored){
  var elem,tmp,tag,wrap,contains,j,
      fragment = context.createDocumentFragment(),
      nodes = [],
      i = 0,
      l = elems.length;
  
  for(;i < 1;i++){
    elem = elems[i];
    if(elem || elem === 0){
      //如果传进来的是对象
      if(jQuery.type(elem) === "object"){
        //elem是节点就包装成数组
        jQuery.merge(nodes,elem.nodeType ? [elem] : elem);
      }
      //rhtml = /<|&#?\w+;/
      //把非html转换成文本内容
      else if(!rhtml.test(elem)){
        nodes.push(context.createTextnode(elem));
      }
      //把HTML转换成DOM节点
      else{
        //tmp是临时父元素
        tmp = tmp || fragment.appendChild(context.createElement("div"));
        //rtagName = /<([a-z][^\/\0>\x20\t\r\n\f]+)/i
        tag = (rtagName.exec(elem) || ["",""])[1].toLowerCase();
        //wrapMap属性只有option thead col tr td
        wrap = wrapMap[tag] || wrapMap._default;
        //htmlPrefilter:fn(html){return html.replace(rxhtmlTag,"<$1><$2>")}
        //rxhtmlTag = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([a-z][^\/\0>\x20\t\r\n\f]*)[^>]*)\/>/gi  匹配除了自闭合标签以外的标签
        tmp.innerHTML = wrap[1] + jQuery.htmlPrefilter(elem) + wrap[2];
        //解包装?
        j = wrap[0];
        while(j--){
          tmp = tmp.lastChild;
        }
        //将tmp子节点合并到nodes数组中
        jQuery.merge(nodes,tmp.childNodes);
        //获取临时父元素div
        tmp = fragment.firstChild;
        //清空
        tmp.textContent = "";
      }
    }
  }
  //把临时fragment的内容也清空
  fragment.textContent = "";
  i = 0;
  while((elem = nodes[i++])){
    if(selection && jQuery.inArray(elem,selection) > -1){
      if(ignored)}{
        ignored.push(elem);
      }
      continue;
    }
    //document是否有elem标签
    contains = jQuery.contains(elem.ownerDocument,elem);
    tmp = getAll(fragment.appendChild(elem),"script");
    
    if(contains){
      setGlobalEval(tmp);
    }
    
    if(scripts){
      j = 0;
      while((elem = tmp[j++])){
        if(rscriptType.test(elem.type || "")){
          scripts.push(elem);
        }
      }
    }
  }
  return fragment;
}
```



---



#### dir

```javascript
var dir = function(elem,dir,until){
  var match = [],
      truncate = until !== undefined;
  while((elem = elem[dir]) && elem.nodeType !== 9){
    if(elem.nodeType === 1){
      if(truncate && jQuery(elem).is(until)){
        break;
      }
      matched.push(elem);
    }
  }
  return matched;
};
```



---



#### getAll

```javascript
function getAll(context,tag){
  //兼容IE<=9-11
  var ret;
  //支持 getElementsByTagName
  //对指定tag执行get 未指定使用通配符
  if(typeof context.getElementsByTagName !== "undefined"){
    ret = context.getElementsByTagName(tag || "*");
  }
  else if(typeof context.querySelectorAll !== "undefined"){
    ret = context.querySelectorAll(tag || "*");
  }
  //找不到返回空数组
  else{
    ret = [];
  }
  //返回要包括自身
  if(tag === undefined || tag && jQuery.nodeName(context,tag)){
    return jQuery.merge([context],ret);
  }
  return ret;
}
```



---



#### setGlobalEval

```javascript
var dataPriv = new Data();
function setGlobalEval(elems,refElements){
  var i = 0,
      l = elems.length;
  for(;i < l;i++){
    dataPriv.set(
      elems[i],
      "globalEval",
      !refElements || dataPriv.get(refElements[i],"globalEval")
    );
  }
}
```



---



#### sibling

```javascript
function sibling(cur,dir){
  //dir为节点方法
  while((cur = cur[dir]) && cur.nodeType !== 1){}
  return cur;
}
```





---



#### siblings

```javascript
//获取兄弟节点(不包括elem)
var siblings = function(n,elem){
  var match = [];
  for(;n;n = n.nextSibling){
    if(n.nodeType === 1 && n !== elem){
      matched.push(n);
    }
  }
  return matched;
};
```



---



#### winnow

```javascript
function winnow(elements,qualifier,not){
  
  //函数
  if(jQuery.isFuntion(qualifier)){
    return jQuery.grep(elements,function(elem,i){
      return !!qualifier.call(elem,i,elem) !== not;
    });
  }
  
  //简单元素
  if(qualifier.nodeType){
    return jQuery.grep(elements,function(elem){
      return (elem === qualifier) !== not;
    });
  }
  
  //类数组
  if(type qualifier !== "string"){
    return jQuery.grep(elements,function(elem){
      return (indexOf.call(qualifier,elem) > -1) !== not;
    });
  }
  
  //简单选择器 
  if(risSimple.test(qualifier)){
    return jQuery.filter(qualifier,elements,not);
  }
  
  //复杂选择器
  qualifier = jQuery.filter(qualifier,elements);
  return jQuery.grep(elements,function(elem){
    return (indexOf.call(qualifier,elem) > -1) !== not && elem.nodeType === 1;
  });
}
```



---



## jQuery方法

#### jQuery.filter

```javascript
jQuery.filter = function(expr,elems,not){
  var elem = elems[0];
  if(not){
    expr = ":not(" + expr + ")";
  }
  if(elems.length === 1 && elem.nodeType === 1){
    return jQuery.find.matchesSelector(elem,expr) ? 
      [elem] : [];
  }
  return jQuery.find.matches(expr,jQuery.grep(elems,function(elem){
    return elem.nodeType === 1;
  }));
};
```



---

#### jQuery.parseHTML

```javascript
//将字符串解析成DOM节点集合
jQuery.parseHTML = function( data, context, keepScripts){
  //参数纠正
  if( typeof data !== "string" ){
    return [];
  }
  if( typeof context === "boolean"){
    keepScripts = context;
    context = false;
  }
  var base,parsed,scripts;
  
  if(!context){
    //针对Safari 8的兼容
    //该浏览器通过document.implementation.createHTMLDocument创建标签
    if(support.createHTMLDocument){
      
    }
    else{
      context = document;
    }
  }
  
  //匹配简单标签
  parsed = rsingleTag.exec(data);
  scripts = !keepScripts && [];
  //简单的就包装下直接返回
  if(parsed){
    return [context.createElement(parsed[1])];
  }
  //其他情况使用fragment节点
  parsed = buildFragment([data],context,scripts);
  if(scripts && scripts.length){
    jQuery(scripts).remove();
  }
  //合并数组
  return jQuery.merge([],parsed.childNodes);
}
```



## jQuery.extend()主方法



```javascript
//合并对象 接受2-n个参数 (布尔值,合并对象1,...)
jQuery.extend = jQuery.fn.extend = function(){
  var options,name,src,copy,copyIsArray,clone,
    target = arguments[0]||{},	//第一个参数给target
    i = 1,
    length = arguments.length,
    deep = false;
  
  //如果第一个参数是布尔值 处理深拷贝环境
  if( typeof target==="boolean" ){
    //第一个参数给deep变量
    deep = target;
    //第二个参数给target
    target = arguments[i]||{};
    i++;	//2
  }
  
  //若target不是对象或者函数 设置为空对象
  if( typeof target !== "object" && !jQuery.isFunction(target) ){
    target={};
  }
  
  //如果只提供一个对象参数 扩展jQuery
  if( i === length ){
    //target设置为空jQuery对象
    target = this;
    i--;	//1
  }
  
  for( ; i<length; i++ ){
    if(( options = arguments[i] ) != null ){
      for( name in options ){
        src = target[name];	//合并后的对象
        copy = options[name];	//被拷贝的对象
        //防止无限循环
        if( target === copy ){
          continue;
        }
        //
        if( deep && copy && ( jQuery.isPlainObject(copy) || ( copyIsArray = jQuery.isArray(copy)))){
          if(copyIsArray){
            copyIsArray=false;
            clone = src && jQuery.isArray(src) ? src : [];
          }
          else{
            clone = src && jQuery.isPlainObject(src) ? src : {};
          }
          //递归实现深拷贝
          target[name]=jQuery.extend(deep,clone,copy);
        }
        //去除undefined值
        //浅拷贝
        else if( copy !== undefined )[
          target[name]=copy;
        ]
      }
    }
  }
    return target;
};
```







----



## jQuery.fn.extend





#### add

```javascript
//<div></div><span></span>:$('span').add('div') - [div,span]
add: function(selector,context){
  return this.pushStack(
    //去重排序
    jQuery.uniqueSort(
      //this.get()获取所有元素
      jQuery.merge(this.get(),jQuery(selector,context))));
}
```



---



#### addBack

```javascript
addBack: function(selector){
  return this.add(selector == null ?
                 //对this与this.prevObject调用add
                 this.prevObject : 
                  //只返回满足$(selector)的元素
                  this.prevObject.filter(selector));
}
```



---



#### closest

```javascript
closest: function(selectors,context){
  var cur,
      i = 0,
      l = this.length,
      matched = [],
      //selectors不是string则targets = $(selectors)
      targets = typeof selectors !== "string" && jQuery(selectors);
  
  //rneedContext:/^[\x20\t\r\n\f]*[>+~]|:(even|odd|eq|gt|lt|nth|first|last)(?:\([\x20\t\r\n\f]*((?:-\d)?\d*)[\x20\t\r\n\f]*\)|)(?=[^-]|$)/i
  //不存在>+~或者...进入分支
  if(!rneedContext.test(selectors)){
    for(;i < l;i++){
      //向上取父元素
      for(cur = this[i];cur && cur !== context;cur = cur.parentNode){
        //跳过文档碎片节点
        if(cur.nodeType < 11 &&
           //targets与cur同级
           (targets ? targets.index(cur) > -1 :
            //否则调用Sizzles
            cur.nodeType === 1 && jQuery.find.matchesSelector(cur,selectors))){
          matched.push(cur);
          break;
        }
      }
    }
  }
  return this.pushStack(matched.length > 1 ? jQuery.uniqueSort(matched) : matched);
}
```



---



#### filter

```javascript
filter: function(selector){
  return this.pushStack( winnow( this, selector || [], false) );
}
```



---



#### find

```javascript
find: function(selector){
  var i,ret,
      //this指向调用find方法的对象
      len = this.length,
      self = this;
  //传入的选择器不是字符串
  if(typeof selector !== "string"){
    //调用 $(selector) 包装后进行过滤
    return this.pushStack( jQuery( selector).filter(funtion(){
      for(i = 0;i < len;i++){
        if(jQuery.contains(self[i],this)){
          return true;
        }
      }
    }));
  }
}
```



---



#### has

```javascript
//
has: function(target){
  //$(this).find(target)
  var targers = jQuery(target,this),
      l = target.length;
  return this.filter(function(){
    var i = 0;
    for(;i < l;i++){
      if(jQuery.contains(this,target[i])){
        return true;
      }
    }
  });
}
```



---

#### not

```javascript
not: function(selector){
  return this.pushStack( winnow( this, selector || [], true) );
}
```



---



#### index

```javascript
//索引
index: function(elem){
  //如果没有传参
  //返回该标签在父元素中的索引位置
  if(!elem){
    //是否存在且有父元素
    return (this[0] && this[0].parentNode) ?
      //返回所有前面元素的长度
      this.first().prevAll().length : -1;
  }
  
  //返回元素在$(elem)中的索引位置
  if(typeof elem === "string"){
    return indexOf.call(jQuery(elem),this[0]);
  }
  
  //返回this[0]在$(elem)中的索引位置
  return indexOf.call(this,elem.jquery ? elem[0] : elem);
}
```



---



#### is

```javascript
is: function(selector){
  if(selector === "string" && rneedsContext.test(selector)){
    return !!winnow( this, jQuery(selector), false).length;
  }
  else{
    return !!winnoe( this, selector || [], false).length;
  }
}
```



---







