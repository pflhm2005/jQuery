# 	敲敲jQuery源码-2(Sizzle变量)

## 黑知识

##### compareDocumentPosition

<div id="a"><div id="b"></div></div>

a.compareDocumentPosition(b) 返回: 20

解析：a 在 b 之前(4)，a 包含 b (16)，加起来就是20

| Bits   | Number | Meaning  |
| ------ | :----- | :------- |
| 000000 | 0      | 元素一致     |
| 000001 | 1      | 节点在不同文档  |
| 000010 | 2      | 节点B在A之前  |
| 000100 | 4      | 节点A在B之前  |
| 001000 | 8      | 节点B包含节点A |
| 010000 | 16     | 节点A包含节点B |
| 100000 | 32     | 浏览器的私有使用 |

---

##### sourceIndex 

1. IE-only 
2. 表示了元素在DOM树的顺序


---

##### ownerDocument与documentElement

1. ownerDocument返回当前元素的document对象
2. documentElement为document的专属方法 返回根节点 在html中就是<html></html>


---

##### nodeType11

1. DocumentFragment接口表示文档的一部分
2. 该节点不属于文档树，parentNode === null
3. 插入文档树时会同时插入所有子节点
4. 通过document.createDocumentFragment()创建 


---

##### textContent

1. 获取内部文本

| 与innerText相比          | 与innerHTML相比      |
| --------------------- | ----------------- |
| 获取包括<style><script>内容 | 写文本应使用textContent |
| 不受样式影响 返回隐藏内容         | 该方法插入的内容不会解析成HTML |
| 修改文本内容会永久销毁内部节点       |                   |

---

##### querySelectorAll与getElementBy

|      | querySelectorAll | getElementBy             |
| ---- | ---------------- | ------------------------ |
| 兼容性  | IE8+             | 最晚添加的ClassName IE9+      |
| 接受参数 | 接受CSS选择符         | 单一参数                     |
| 参数规范 | 严格符合规范           | 可接受任何字符 如：...ById("12a") |
| 返回值  | 返回静态NodeList对象   | 返回 HTMLCollection 对象     |
| (区别) | 包含所有节点           | 只包含Element节点             |
| 速度   | 慢                | 快100+倍                   |
|      |                  |                          |



## 方法



##### addCombinator

```javascript
//关联函数 负责将关系符与原子匹配器关联 例如：div>
//matcher: 原子匹配
//combinator: 关系符
//true
function addCombinator(matcher,combinator,base){
  var dir = combinator.dir,
      //next???
      skip = combinator.next,
      key = skip || dir,
      checkNonElements = base && key === "parentNode",
      doneName = done++;
  
  //匹配到 > +
  if(combinator.first){
    return function( elem, context, xml){
      while((elem = elem[dir])){
        if(elem.nodeType === 1 || checkNonElements){
          return matcher(elem,context,xml);
        }
      }
      return false;
    }
  }
  //匹配到 ~ " "
  else{
    return function(elem,context,xml){
      var oldCache,uniqueCache,outerCache,
          newCache = [dirruns,doneName];
      //又是xml
      if(xml){
        while((elem = elem[dir])){
          if(elem.nodeType === 1 || checkNonElements){
            if(matcher(elem,context,xml)){
              return true;
            }
          }
        }
      }
      else{
        while((elem = elem[dir])){
          if(elem.nodeType === 1 || checkNonElements){
            outerCache = elem[expando] || (elem[expando] = {});
          }
          //IE<9 不写了
          //...
          if(skip && skip === elem.nodeName.toLowerCase()){
            elem = elem[dir] || elem;
          }
          else if((oldCache = uniqueCache[key]) && 
                 oldCache[0] === dirruns && oldCache[1] === doneName){
                   return (newCache[2] === oldCache[2]);  
                 }
          else{
            uniqueCache[key] = newCache;
            if((newCache[2] = matcher(elem,context,xml))){
              return true;
            }
          }
        }
      }
      return false;
    };
  }
}  
```



---

##### elementMatcher

```javascript
function elementMatcher(matchers){
  if(matchers.length > 1){
    return function(elem,context,xml){
      var i = matchers.length;
      while(i--){
        if(!matchers[i](elem,context,xml)){
          return false;
        }
      }
      return true;
    }
  }
  else{
    return matchers[0];
  }
}
```





---

##### matcherFromTokens

```javascript
//懵逼一号函数
function matcherFromTokens(tokens){
  var checkContext,matcher,j,
      len = tokens.length,
      //判断tokens是否关系符开头 即ID过滤
      leadingRelative = Expr.relative[tokens[0].type],
      //如果不是关系符开头赋值为" " {dir:"parentNode"}
      implicitRelative = leadingRelative || Expr.relative[" "],
      //略
      i = leadingRelative ? 1 : 0,
      //
      matchContext = addCombinator(function(elem){
        return elem === checkContext;
      },implicitRelative,true),
      //
      matchAnyContext = addCombinator(function(elem){
        return indexOf(checkContext,elem) > -1;
      },implicitRelative,true),
      
      //最终匹配函数
      matchers = [function(elem,context,xml){
        var ret = (!leadingRelative && 
                   (xml || context !== outermostContext)) || 
            ((checkContext = context).nodeType ? 
             matchContext(elem,context,xml) : matchAnyContext(elem,context,xml));
        checkContext = null;
        return ret;
      }];
  
  for(;i < len;i++){
    //匹配到关系符
    if((matcher = Expr.relative[tokens[i].type])){
      matchers = [addCombinator(elementMatcher(matchers),matcher)];
    }
    //匹配到原子标签	
    else{
      matcher = Expr.filter[tokens[i].type].apply(null,tokens[i].matches);
      //
      if(matcher[expando]){
        j = ++i;
        for(;j < len;j++){
          if(Expr.relative[tokens[j].type]){
            break;
          }
        }
        return setMatcher(
          i > 1 && elementMatcher(matchers),
          i > 1 && toSelector(
          	tokens.slice( 0,i - 1).concat
            ({value : tokens[i - 2].type === " " ? 
              "*" : ""})).replace(rtrim,"$1"),
          matcher,
          i < j && matcherFromTokens(tokens.slice(i,j)),
          j < len && matcherFromTokens((tokens = tokens.slice(j))),
          j < len && toSelector(tokens)
        );
      }
      //把原子匹配器弹入函数数组
      matchers.push(matcher);
    }
  }
  return elementMatcher(matchers);
}
```



---



##### matcherFromGroupMatchers

```javascript
//懵逼二号函数
function matcherFromGroupMatchers(elementMatchers,setMatchers){
  var bySet = setMatchers.length > 0,
      byElement = elementMatchers.length > 0,
      superMatcher = function(seed,context,xml,results,outermost){
        var elem,j,matcher,
            matchedCount = 0,
            i = "0",
            unmatched = seed & [],
            setMatched = [],
            contextBackup = outermostContext,
            //
            elems = seed || byElement && Expr.find["TAG"]("*",outermost),
            dirrunsUnique = (dirruns += contextBackup == null ? 
                             1 : Math.random() || 0.1),
            len = elems.length; 
        if(outermost){
          outermostContext = context === document || context || outermost;
        }

        //
        for(;i !== len && (elem = elems[i]) != null;i++){
          if(byElement && elem){
            j = 0;
            if(!context && elem.ownerDocument !== document){
              setDocument(elem);
              xml = !documentIsHTML;
            }
            while((matcher = elementMatchers[j++])){
              if(matcher(elem,context || document,xml)){
                results.push(elem);
                break;
              }
            }
            if(outermost){
              dirruns = dirrunsUnique;
            }
          }

          //
          if(bySet){
            //遍历
            if((elem = !matcher && elem)){
              matchedCount--;
            }
            if(seed){
              unmatched.push(elem);
            }
          }
        }
        //
        matchedCount += i;

        //
        if(bySet && i !== matchedCount){
          j = 0;
          while((matcher = setMatchers[j++])){
            matcher(unmatched,setMatched,context,xml);
          }
          if(seed){
            if(matchedCount > 0){
              while(i--){
                if(!(unmatched[i] || setMatched[i])){
                  setMatched[i] = pop.call(results);
                }
              }
            }
            setMatched = condense(setMatched);
          }
          push.apply(results,setMatched);

          //
          if(outermost && !seed && 
             setMatched.length \> 0 && 
             (matchedCount + setMatched.length) > 1){
            Sizzle.uniqueSort(results);
          }
        }
        if(outermost){
          dirruns = dirrunsUnique;
          outermostContext = contextBackup;
        }
        return unmatched;
      };
  return bySet ? markFunction(superMatcher) : superMatcher;
}
```





## 变量



##### arr

```javascript
var arr = [];
```

##### booleans

```javascript
//该变量保存所有值为布尔的属性
var booleans = "checked|selected|async|autofocus|autoplay|controls|defer|disabled|hidden|ismap|loop|multiple|open|readonly|required|scoped";
```

##### classCache

```javascript
var classCache = createCache();
//缓存函数
var createCache = function(){
  var keys = [];
  function cache( key, value ){
    //cacheLength默认为50 可以设置
    if( keys.push(key + " ") > Expr.cacheLength ){
      delete cache[key.shift()]; 
    }
    //用key+" "来防止与本地原型属性冲突
    return (cache[key + " "] = value);
  }
  return cache;
}
```

##### compile

```javascript
var compile = Sizzle.compile = function(selector,match){
  var i=0,
      setMatchers = [],
      elementMatchers = [],
      cached = compilerCache[selector + " "];
  //缓存
  if(!cached){
    //没有解析先进行词法解析
    if(!match){
      match = tokenize(selector);
    }
    //第二种方法过滤时 match是剔除最后一个标签的对象数组
    i = match.length;
    while(i--){
      //懵逼一号函数
      cached = matcherFromTokens(match[i]);
      //判断是否有伪类
      if(cached[expando]){
        srtMatchers.push(cached);
      }
      //普通选择器压入elementMatchers
      else{
        elementMatchers.push(cached);
      }
    }
    cached =
      //懵逼二号函数
      compilerCache(selector,matchFromGroupMatchers(elementMatchers,setMatchers));
    cached.selector = selector;
  }
  return cached;
};
```

##### compilerCache

```javascript
var compilerCache = createCache();
```

##### contains

```javascript
//检测本地方法compareDocumentPositon
//该方法是DOM3的一个方法 确定元素位置信息
//元素一致：0 不同文档：1 B在A前：2 A在B前：4
var hasCompare = rnative.test(docElem.compareDocumentPositon);

//测试a是否包含b
//两个方法有一个就行
//源码乱 重写一下
//这是源码
var contains = hasCompare || rnative.test(docElem.contains) ? 
    function(a,b){
      var adown = a.nodeType === 9 ? a.documentElement : a,
          bup = b && b.parentNode;
      return a === bup || !!(bup && bup.nodeType === 1 && (adown.contains ? adown.contains(bup) : a.compareDocumentPosition && a.compareDocumentPosition(bup) & 16 ));
    } : 
function(a,b){
  if(b){
    while(( b = b.parentNode )){
      if( b === a ){
        return true;
      } 
    }
  }
  return false;
};

//重写
if(hasCompare || rnative.test(docElem.contains)){
    //containd(a,b) a,b分别为两个节点
    var contains = function(a,b){
    //a.nodeType === 9代表a是#document
    //此时a.documentElement获得document所有子节点
    var adown = a.nodeType === 9 ? a.documentElement : a,
        //bup是b的父节点 document父节点是null
        bup = b && b.parentNode;
    //b父节点是a 直接返回true
    if(a === bup){
      return true;
    }
    //这TM好麻烦啊
    //首先bup必须是元素节点
    else if(!!(bup && bup.nodeType === 1)){
      //如果adown有contains()方法 直接调用判断包含
      if(adown.contains){
        return !!(adown.contains(bup));
      }
      //否则调用compareDocumentPosition()方法
      //跟100000与运算什么鬼...
      else{
        return !!(a.compareDocumentPosition(bup) & 16);
      }
    }
    else{
      return false;
    }
  }
  }
//contains与compareDocumentPosition方法都不存在
else{
  contains = function(a,b){
    if(b){
      //对b不停调用parentNode
      //document.parentNode === null时终止
      while(b = b.parentNode){
        if(b === a){
          return true;
        }
      }
    }
    return false;
  }
}
```

##### dirruns

```javascript
var dirruns = 0;
```

##### done

```javascript
var done = 0;
```



##### docElem

```javascript
var docElem = document.documentElement;
```

##### document

```javascript
//该变量为未定义 setDocument中进行设置
var document = node ? node.ownerDocument || node : preferredDoc;
```

##### documentIsHTML

```javascript
var documentIsHTML = !isXML(document);
```

##### expando

```javascript
var expando = "sizzle" + 1 * new Date();
```

##### Expr

```javascript
//对象
var Expr = Sizzle.selectors = {
  //默认缓存大小 可修改
  cacheLength: 50,
  //标记函数专供Sizzle使用
  createPseudo: function markFunction(fn){
    fn[ expando ] = true;
    return fn;
  },  
  match:{
      "ID": /^#((?:\\.|[\w-]|[^-\xa0])+)/,	//ID 	  #* #slide	
      "CLASS": /^\.((?:\\.|[\w-]|[^-\xa0])+)/,	//CLASS   .*  .content
      "TAG": /^((?:\\.|[\w-]|[^-\xa0])+|[*])/,	//标签	(div),(span)
      "ATTR": /^attributes/,	  //属性	   "href","src"
      "PSEUDO": /^pseudos/,		  //伪元素	  :first
      //子元素
      "CHILD": /^:(only|first|last|nth|nth-last)-(child|of-type)(?:\([\x20\t\r\n\f]*(even|odd|(([+-]|)(\d*)n|)[\x20\t\r\n\f]*(?:([+-]|)[\x20\t\r\n\f]*(\d+)|))[\x20\t\r\n\f]*\)|)/i,
      "bool": /^(?:booleans)$,i/,	  //值为布尔的属性 例如checked
      "needsContext": /^[\x20\t\r\n\f]*[>+~]|:(even|odd|eq|gt|lt|nth|first|last)(?:\([\x20\t\r\n\f]*((?:-\d)?\d*)[\x20\t\r\n\f]*\)|)(?=[^-]|$)/i	//需要内容的属性
    },
  attrHandle:{},
  find:{},
  relative:{
    ">" : {dir : "parentNode",
           first:true},
    " " : {dir : "parentNode"},
    "+" : {dir : "previousSibling",
           first:true},
    "~" : {dir : "previousSibling"}
  },
  
  preFilter:{
    "ATTR" : function(match){
      match[1] = match[1].replace(runescape,funescape);
      match[3] = (match[3] || match[4] || match[5] || "").replace(runescape,funescape);
      if(match[2] === "~="){
        match[3] = " " + match[3] + " ";
      }
      return match.slice(0, 4);
    },
    
    "CHILD" : function(match){
      //
      match[1] = match[1].toLoweCase();
      if(match[1].slice(0, 3) === "nth"){
        if(!match[3]){
          Sizzle.error(match[0];)
        }
        match[4] = +(match[4] ? match[5] + (match[6] || 1) : 
                     2 * (match[3] === "even" || match[3] === "odd") );
        match[5] = +((match[7] + match[8]) || match[3] === "odd");
      }
      else if(match[3]){
        Sizzle.error(match[0]);
      }
      return match;
    },
    
    "PSEUDO" : function(match){
      var excess,
          unquoted = !match[6] && match[2];
      if(matchExpr["CHILD"].test(match[0])){
        return null;
      }
      
      if(match[3]){
        match[2] = match[4] || match[5] || "";
      }
      //rpseudo伪类
      else if(unquoted && rpseudo.test(unquoted) && 
              (excess = tokenize(unquoted,true)) && 
              (excess = unquoted.indexOf(")",unquoted.length - excess) 
               - unquoted.length)){
        match[0] = match[0].slice(0, excess);
        match[0] = unquoted.slice(0, excess);
      }
      return match.slice(0,3);
    }
  },
  //原子匹配器 返回一个函数
  filter:{
    //传入TAG名称
    "TAG" : function(nodeNameSelector){
      var nodeName = nodeNameSelector.replace(runescape.funescape).toLowerCase();
      //通配符全部符合
      return nodeNameSelector === "*" ? function(){return true;} :
      //判断节点的nodeName是否符合传入的TAG类型
      function(elem){
        return elem.nodeName && elem.nodeName.toLowerCase() === nodeName;
      };
    },
    
    "CLASS" : function(className){
      var pattern = classCache[className + " "];
      //pattern = /^|className|$/;
      return pattern || 
        (pattern = new RegExp("(^|" + whitespace + ")" + className + "(" + whitespace + "|$)")) && classCache(className,function(elem){
        if(typeof elem.className === "string"){
          return pattern.test(elem.className);
        }
        else if(typeof elem.getAttribute !== "undefined"){
          return pattern.test(elem.getAttribute("class"));
        }
        else{
          return pattern.test("");
        }
      });
    },
    
    "ATTR" : function(name,operator,check){
      return function(elem){
        var result = Sizzle.attr(elem,name);
        if(result == null){
          return operator === "!=";
        }
        if(!operator){
          return true;
        }
        result += "";
        return operator === "=" ? result === check : 
        	operator === "!=" ? result !==check : 
        	operator === "^=" ? check && result.indexOf(check) === 0 : 
        	operator === "*=" ? check && result.indexOf(check) > -1 : 
        	operator === "$=" ? check && result.slice(-check.length) === check : 
        	operator === "~=" ? (" " + result.replace(rwhitespace," ").indexOf(check) > -1) : 
        	operator === "|=" ? result === check || 
          result.slice(0,check.length + 1) === check + "-" : false;
      };
    },
    
    "CHILD" : function(type,what,argument,first,last){
      //不以nth开头true
      var simple = type.slice(0,3) !== "nth",
          //不以last结尾true
          forward = type.slice(-4) !== "last",
          //中间是of-type
          ofType = what === "of-type";
      if(first === 1 && last === 0){
        return function(elem){
          return !!elem.parentNode;
        }
      }
      else{
        return function(elem,context,xml){
          var cache,uniqueCache,outerCache,node,nodeIndex,start,
              dir = simple !== forward ? "nextSibling" : "previousSibling",
              parent = elem.parentNode,
              name = ofType && elem.nodeName.toLowerCase(),
              useCache = !xml && !ofType,
              diff = false;
          //确保不是document对象
          if(parent){
            //:(first|last|only)-(child|of-type)
            if(simple){
              while(dir){
                node = elem;
                while((node = node[dir])){
                  if(ofType ? node.nodeName.toLowerCase() === name:
                     node.nodeType ===1){
                    return false;
                  }
                }
                start = dir = type === "only" && !start && "nextSibling";
              }
              return true;
            }
            start = [forward ? parent.firstChild : parent.lastChild];
            //nth-child(...)
            if(forward && useCache){
              node = parent;
              outerCache = node[expando] || (node[expando] ={});
              //兼容IE<9
              uniqueCache = outerCache[node.uniqueID] || 
                (outerCache[node.uniqueID] = {});
              cache = uniqueCache[type] || [];
              nodeIndex = cache[0] === dirruns && cache[1];
              diff = nodeIndex && cache[2];
              node = nodeIndex && parent.childNodes[nodeIndex];
              while((node = ++nodeIndex && node && 
                     node[dir] || (diff = nodeIndex = 0) || start.pop())){
                if(node.nodeType === 1 && ++diff && node === elem){
                  uniqueCache[type] = [dirruns,nodeIndex,diff];
                  break;
                }
              }
            }
            else{
              if(useCache){
                node = elem;
                outerCache = node[expando] || 
                  (node[expando] = {});
                //兼容IE<9
                uniqueCache = outerCache[node.uniqueID] || 
                  (outerCache[node.uniqueID] = {});
                cache = uniqueCache[type] || [];
                nodeIndex = cache[0] === dirruns && cache[1];
                diff = nodeIndex;
              }
              //xml:nth:child(...)为什么要兼容xml啊
              //:nth-last-child(...)  :nth(-last)?-of-type(...)
              if(diff === false){
                while((node = ++nodeIndex && node && 
                       node[dir] || (diff = nodeIndex = 0) || start.pop())){
                  if((ofType ? node.nodeName.toLowerCase() === name : 
                      node.nodeType === 1) && ++diff){
                    if(useCache){
                      outerCache = node[expando] || (node[expando] = {});
                      //兼容IE<9
                      uniqueCache = outerCache[node.uniqueID] || 
                        (outerCache[node.uniqueID] = {});
                      uniqueCache[type] = [dirruns,diff];
                    }
                    if(node === elem){
                      break;
                    }
                  }
                }
              }
            }
            diff -= last;
            return diff === first || (diff % first === 0 && diff/first >= 0);
          }
        };
      }--else
    },
    
    "PSEUDO" : function(pseudo,argument){
      var args,
          fn = Expr.pseudo[pseudo] || 
          Expr.setFilters[pseudo.toLowerCase()] || 
          Sizzle.error("unsupported pseudo: " + pseudo);
      //
      if(fn]expando){
        return fn(arguments);
      }
      //
      if(fn.length > 1){
        args = [pseudo,pseudo,"",argument];
        if(Expr.setFilters.hasOwnProperty(pseudo.toLowerCase())){
          return markFuntion(function(seed,matches){
            var idx,
                matched = fn(seed,argument),
                i = matched.length;
            while(i--){
              idx = indexOf(seed,matched[i]);
              seed[idx] = !(matches[idx]) = matched[i];
            }
          });
        }
        else{
          return function(elem){
            return fn(elem,0,args);
          };
        }
      }
      return fn;
    }
  },
  
  pseudos:{
    "not" : markFunction(function(selector){
      var input = [],
          results = [],
          matcher = compile(selector.replace(rtrim,"$1"));
      if(matcher[expando]){
        return markFunction(function(seed,matches,context,xml){
          var elem,
              unmatched = matcher(seed,null,xml,[]),
              i = seed.length;
          while(i--){
            if((elem = unmatched[i])){
              seed[i] = !(match[i] = elem);
            }
          }
        });
      }
      else{
        return function(elem,context,xml){
          input[0] = elem;
          matcher(input,null,xml,results);
          //不要继续持有元素...
          input[0] = null;
          return !results.pop();
        };
      }
    }),
    
    "has" : markFunction(function(selector){
      return function(elem){
        return Sizzle(selector,elem).length > 0;
      };
    }),
    
    "contains" : markFunction(function(text){
      text = text.replace(runescape,funescape);
      return function(elem){
        return (elem.textContent || 
                elem.innerText || 
                getText(elem)).indexOf(text) > -1;
      };
    }),
    //这个就不写了
    "lang" : "",
    
    //杂项
    "target" : function(elem){
      var hash = window.location && window.location.hash;
      return hash && hash.slice(1) === elem.id;
    },
    //是否是根元素
    "root" : function(elem){
      return elem === docElem;
    },
    "focus" : function(elem){
      return elem === document.activeElement && 
        (!document.hasFocus || document.hasFocus()) && 
        !!(elem.type || elem.href || ~elem.tabIndex);
    },
    //布尔属性
    "enabled" : createDisabledPseudo(false),
    "disabled" : createDisabledPseudo(true),
    "checked" : function(elem){
      //在CSS3中 checked需要同时返回checked与selected元素
      var nodeName = elem.nodeName.toLowerCase();
      return (nodeName === "input" && !!elem.checked) || 
        (nodeName === "option" && !!elem.selected);
    },
    "selected" : function(elem){
      if(elem.parentNode){
        elem.parentNode.selectedIndex;
      }
      return elem.selected === true;
    },
    //判断是否空?
    "empty" : function(elem){
      for(elem = elem.firstChild;elem;elem = elem.nextSibling){
        //1,2,3-元素,属性,文本
        if(elem.nodeType < 6){
          return false;
        }
      }
      return true;
    },
    "parent" : function(elem){
      return !Expr.pseudo["empty"](elem);
    },
    //标签类型
    //h1-h6
    "header" : function(elem){
      return rheader.test(elem.nodeName);
    },
    //input输入框
    "input" : function(elem){
      return rinputs.test(elem.nodeName);
    },
    //按钮
    "button" : function(elem){
      var name = elem.nodeName.toLowerCase();
      return name === "input" && elem.type === "button" || 
        name === "button";
    },
    //输入框且type="text"
    "text" : function(elem){
      var attr;
      return elem.nodeName.toLowerCase() === "input" && 
        elem.type === "text" && 
        ((attr = elem.getAttribute("type")) == null || 
         attr.toLowerCase() === "text");
    },
    //返回的是数组?
    "first" : createPositionalPseudo(function(){
      reutrn [0];
    }),
    "last" : createPostionalPseudo(function(matchIndexes,length){
      reutrn [length-1];
    }),
    "eq" : createPositionalPseudo(function(matchIndexed,length,argument){
      return [argument < 0 ? argument + length : argument];
    }),
    "even" : createPositionalPseudo(function(matchIndexes,lenth){
      var i = 0;
      for(;i < length;i += 2){
        matchIndexes.push(i);
      }
      return matchIndexes;
    }),
    "odd" : createPositionalPseudo(function(matchIndexes,length){
      var i = 1;
      for(;i < length;i += 2){
        matchIndexes.push(i);
      }
      return matchIndexes;
    }),
    //前面所有元素
    "lt" : createPositionalPseudo(function(matchIndexes,length,argument){
      var i = argument < 0 ? argument + length : argument;
      for(;--i >= 0;){
        matchIndexes.push(i);
      }
      return matchIndexes;
    }),
    //后面所有元素
    "gt" : createPositionalPseudo(function(matchIndexes,length,argument){
      var i = argument < 0 ? argument + length : argument;
      for(;++i < length;){
        matchIndexes.push(i);
      }
      return matchIndexes;
    })
  }
};

Expr.pseudos["nth"] = Expr.pseudos["eq"];
//创建button、input类型伪类
for(i in {radio:true,checkbox:true,file:true,password:true,image:true}){
  Expr.pseudos[i] = createInputPesudo(i);
}
for(i in {submit:true,reset:true}){
  Expr.pseudos[i] = createButtonPseudo(i);
}
//
function setFilters(){}
setFilters.prototype = Expr.filters = Expr.pseudos;
Expr.setFileters = new setFilters();
```

##### getText

```javascript
//获得文本内容
var getText = function(elem){
  var node,
      ret = "",
      i = 0,
      nodeType = elem.nodeType;
  //如果没有节点类型 应该是个元素数组
  if(!nodeType){
    while((node = elem[i++])){
      //递归获得每个元素内容
      ret += getText(node);
    }
  }
  //元素、document对象、Fragment
  else if(nodeType === 1 || nodeType === 9 || nodeType === 11){
    //nodeType为9或11的时候
    if(typeof elem.textContent === "string"){
      return elem.textContent;
    }
    //nodeType为1 即document对象
    else{
      //遍历所有节点
      for(elem = elem.firstChild;elem;elem = elem.nextSibling){
        ret += getText(elem);
      }
    }
  }
  //nodeType为3是Element或Attr中实际文字 4已被弃用
  else if(nodeType === 3 || nodeType === 4){
    return elem.nodeValue;
  }
  //不返回注释
  return ret;
}
```



##### hasDuplicate



```javascript
Sizzle.uniqueSort = function( results ) {
	var elem,duplicates = [],
		j = 0,
		i = 0;
		while ( (elem = results[i++]) ) {
			if ( elem === results[ i ] ) {
				j = duplicates.push( i );
			}
		}
		while ( j-- ) {
			results.splice( duplicates[ j ], 1 );
		}
	sortInput = null;
	return results;
};
```





##### hasOwn



```javascript
var hasOwn = ({}).hasOwnProperty;
```



##### indexOf



```javascript
//常规的indexOf
var indexOf =  = function( list, elem ){
        var i = 0,
            len = list.length;
        for( ; i < len; i++ ){
          if( list[i] === elem ){
            return i;
          }
        }
        return -1;
      };
```



##### isXML



```javascript
  //判断是不是XML文档 接受一个元素节点
  //!isXML就是判断是不是HTML文档了吧..
  var isXML = Sizzle.isXML = function(elem){
    //ownerDocument可以获取#document节点
    //document.ownerDocument === null
    //document.documentElement获得文档结构树
    var documentElement = elem && (elem.ownerDocument || elem).documentElement;
    return documentElement ? documentElement.nodeName !== "HTML" : false;
  };
```





##### matches



##### outermostContext



##### pop



```javascript
var pop = arr.pop;
```



##### preferredDoc



```javascript
var preferredDoc = window.document;
```



##### push



```javascript
var push = arr.push;
```



##### push_native



```javascript
var push_native = arr.push;
```



##### rbuggyMatches



##### rbuggyQSA



##### select



```javascript
var select = Sizzle.select = function(selector,context,results,seed){
  var i,tokens,token,type,find,
      //如果selector是function类型 compiled = selector
      compiled = typeof selector === "function" && selector,
      //如果没有提供seed 调用tokenize(selector)
      match = !seed && tokenize((selector = compiled.selector || selector));
  
  results = results || [];
  //selector只有一个分组(tokenize函数中的soFar没有逗号)
  if(match.length === 1){
    tokens = match[0] = match[0].slice(0);
    
    //如果传进来的解析字符串对象数组tokens第一个元素type是ID
    if(tokens.length > 2 && (token = token[0]).type === "ID" &&
       //context是document且文档是HTML
      	context.nodeType === 9 && documentIsHTML &&
       //第二个字符是关系选择符
      	Expr.relative[tokens[1].type]){
      //范围指定为该节点
      context = (Expr.find["ID"](token.match[0].replace(runescape,funescape),context) || [])[0];
      //如果ID找不到直接返回
      if(!context){
        return results;
      }
      else if(compiled){
        context=  context.parentNode;
      }
      //去除第一个ID
      selector = selector.slice(tokens.shift().value.length);
    }
    //配合上面的if缩小范围
    //ID开头且第二个为关系运算符[>+~]则i置0
    i = matchExpr["needsContext"].test(selector) ? 0 : tokens.length;
    
    //第二种缩小范围方式
    while(i--){
      //取出最后一个数组对象
      token = tokens[i];
      //Expr.relative包含4个关系运算符对象 " " + > ~
      //最后一个是符号就出错了 直接返回
      if(Expr.relative[(type = token.type)]){
        break;
      }
      //Expr.find包含三个类型函数对象 TAG ID CLASS
      //只是简单的调用对应的getElementBy...返回对应的匹配标签
      //这里find代表一个函数fn find(*){return getElementBy...(*)}
      if((find = Expr.find[type])){
        if((seed = find(
          token.matches[0].replace(runescape,funescape),
          //rsibling是关系正则 /[+~]/
          rsibling.test(token[0].type) && 
          //context是document返回false
          testContext(context.parentNode) || context
        ))){
          //去掉已经解析出来的数组对象
          tokens.splice(i,1);
          //seed长度为0  即上面find方法找不到对应标签 返回
          //seed不为0 把tokens对象中的value拼接成字符串赋值给selector
          selector = seed.length && toSelector(tokens);
          //解析完了就返回
          if(!selector){
            push.apply(results,seed);
            return results;
          }
          //跳出while
          break;
        }
      }
    }
  }
  //break到这里 开始懵逼...
  (compiled || compile(selector,match))(
  	seed,
    context,
    !documentIsHTML,
    results,
    !context || rsibling.test(selector) && 
    testContext(context.parentNode) || context
  );
  return results;
};
```



##### setDocument

```javascript
//设置document文档?
//据查 这个玩意根据当前document设置document先关变量 执行一次
var setDocument = Sizzle.setDocument = function(node){
  var hasCompare, subWindow,
      //preferredDoc = window.document
      //node.ownerDocument === #document 就是html页面
      doc = node ? node.ownerDocument || node : preferredDoc;

  //如果已经设置直接返回
  if( doc === document || doc.nodeType !== 9 || !doc.documentElement ){
    return document;
  }

  //更新全局变量
  document = doc;
  docElem = document.documentElement;
  documentIsHTML = !isXML(document);

  //IE9-11
  if(preferredDoc !== document && (subWindow = document.defaultView) && subWindow.top !== subWindow){
    //IE11
    if(subWindow.addEventListener){
      subWindow.addEventListener("unload",unloadHandler,false);
    }
    //IE9-10
    else if(subWindow.attachEvent){
      subWindow.attachEvent("onunload",unloadHandler);
    }
  }
  //IE<8  attributes跟properties有卵区别...
  support.attribute = assert(function(el){
    el.className = "i";
    return !el.getAttribute("className");
  });
  //检测getElementsByTagName("*")是否只返回元素
  //据查 IE6-8混杂注释节点
  support.getElementsByTagName = assert(function(el){
    el.appendChild(document.createComment(""));
    return !el.getElementByTagName("*").length;
  });
  //IE<9
  //测试docuemnt.getElementsByClassName是否本地方法
  support.getElementsByClassName = rnative.test(document.getElementsByClassName);
  //IE<10
  support.getById = assert(function(el){
    docElem.appendChild(el).id = expando
    return !document.getElementsByName || !document.getElementsByName(expando).length;
  });
  //ID过滤寻找
  if(support.getById){
    Expr.filter["ID"] = function(id){
      var attrId = id.replace(runescape, funescape);
      return function(elem){
        return elem.getAttribute("id") === attrId;
      };
    };
    Expr.find["ID"] = function(id,context){
      if(typeof context.getElementById !== "undefined" && documentIsHTML){
        var elem = context.getElementById(id);
        return elem ? [elem] : [];
      }
    };
  }
  //兼容IE6-7 getElementById不可靠 使用getAttributeNode方法 不写了
  else{

  }
  //find标签
  Expr.find("TAG") = 
    support.getElementsByTagName ? 
    //如果getElementsByTagName方法返回正常
    function(tag,context){
    if(typeof context.getElementsByTagName !== "undefined"){
      return context.getElementsByTagName(tag);
    }
    else if(support.qsa){
      return context.querySelectorAll(Tag);
    }
  } : 
  //否则针对tag穿入通配符进行过滤
  function(tag,context){
    var elem,
        tmp = [],
        i = 0,
        results = context.getElementsByTagName(tag);
    if(tag === "*"){
      //遍历 只需要节点类型1的元素
      while((elem = results[i++])){
        if(elem.nodeType === 1){
          tmp.push(elem);
        }
      }
      return tmp;
    }
    return results;
  };
  //find类
  Expr.find["CLASS"] = support.getElementsByClassName && function(className,context){
    if(typeof context.getElementsByClassName !== "undefined" && documentIsHTML){
      return context.getElementsByClassName(className);
    }
  };
  return document;
};
```

##### slice

```javascript
var slice = arr.slice;
```

##### sortInput

##### sortOrder

```javascript
var sortOrder = function(a,b){
  if( a === b ){
          hasDuplicate = true;
        }
        return 0;
}
//有毛用啊这个函数
if(hasCompare){
  var sortOrder = function(a,b){
    if( a === b ){
      hasDuplicate = true;
      return 0;
    }

    var compare = !a.compareDocumentPosition - !b.compareDocumentPosition;
    if(compare){
      return compare;
    }
    //如果两个属于同一个document 计算对应的位置?
    if((a.ownerDocument || a) === (b.ownerDocument || b)){
      compare = a.compareDocumentPosition(b);
    }
    else{
      compare = 1;
    }
    //直接在这里给出函数定义 判断是否支持compareDocumentPosition方法
    support.sortDetached = assert(function(el){
      //1对应的二进制为1 只有当第一位为1时才返回1 否则返回0
      //这里的el就是fieldset 
      //相同标签调用compareDocumentPosition会得到37
      //37 & 1 === 1   !!1 == true
      return el.compareDocumentPosition(document.createElement("fieldset")) & 1;
    });
    if(compare & 1 || (!support.sortDetached && b.compareDocumentPosition(a) === compare)){
      //preferredDoc = window.document
      //若a属于当前document返回-1
      if( a === document || a.ownerDocument === preferredDoc && contains(preferredDoc,a)){
        return -1;
      }
      //若b返回1
      if( b === document || b.ownerDocument === preferredDoc && contains(preferredDoc,b)){
        return 1;
      }
      //sortInput
      if(sortInput){
        return indexOf(sortInput,a)-indexOf(sortInput,b)
      }
      else{
        return 0;
      }
    }
    //与运算4
    if(compare & 4){
      return -1;
    }
    else{
      return 1;
    }
  }
  }
else{
  sortOrder = function(a,b){
    //节点相同直接返回0
    if(a === b){
      hasDuplicate = true;
      return 0;
    }
    var cur,
        i = 0,
        aup = a.parentNode,
        bup = b.parentNode,
        ap = [a],
        bp = [b];
    //无父元素代表是document或者未插入的节点
    if(!aup || !bup){
      if(a === document){
        return -1;
      }
      else if(b === document){
        return 1;
      }
      else{
        return aup ? -1 : bup ? 1 : sortInput ? (indexOf(sortInput,a) - indexOf(sortInput,b)) : 0;
      }
    }
    //父节点相同判断是否兄弟元素
    else if(aup === bup){
      return siblingCheck(a,b);
    }
    //
    cur = a;
    while((cur = cur.parentNode)){
      //头部插入 数组形式[...,a.parent,a]
      ap.unshift(cur);
    }
    cur = b;
    while((cur = cur.parentNode)){
      bp.unshift(cur);
    }
    //从document开始向下遍历找不同
    //结果为第i个子节点
    while(ap[i] === bp[i]){
      i++;
    }
    //存在i说明有不同 如果是兄弟元素返回true
    //
    return i ? siblingCheck(ap[i],bp[i]) : ap[i] === preferredDoc ? -1 : bp[i] === preferredDoc ? 1 : 0;
  };
```

##### support

```javascript 
var support;	//测试兼容的类
```

##### tokenCache

```javascript
//选择器缓存
var tokenCache = createCache();
```

##### tokenize

```javascript
var tokenize = Sizzle.tokenize = function(selector,parseOnly){
  var matched,match,tokens,type,
      soFar,groups,preFilters,
      cached = tokenCache[selector + " "];
  //有缓存直接返回
  if(cached){
    return parseOnly ? 0 : cached.slice(0);
  }
  
  soFar = selector;
  groups = [];
  preFilters = Expr.preFilter;
  //如果soFar还有内容 
  while(soFar){
    //rcomma = /^ , */
    //!matched检测是否第一次运行或者还没有等待解析的字符串
    //在这里soFar是解析后的字符串
    //举例soFar = " ,span" 则match = " ,"
    if(!matched || (match = rcomma.exec(soFar))){
      if(match){
        //匹配后删除match匹配的内容 soFar="span"
        soFar = soFar.slice(match[0].length) || soFar;
      }
      //第一次运行时tokens = [] 然后将tokens弹入groups
      groups.push((tokens) = []);
    }
    matched = false;
    
    //关系匹配符 >+~空格
    //rcombinators = /^[\x20\t\r\n\f]*([>+~]|[\x20\t\r\n\f])[\x20\t\r\n\f]*/
    //举例:rcombinators.exec(" >div"))会匹配到[" >",">"]
    if((match = rcombinators.exec(soFar))){
      //此表达式执行后matched = " >" match = [">"] 
      matched = match.shift();
      //tokens = [{value:" >",type:">"}]
      tokens.push({
        value : matched,
        type : match[0].replace(rtrim," ");
      });
      //获取到关系符后面的字符串 soFar = "div"
      soFar = soFar.slice(matched.length);
    }
    
    //type = ["TAG","CLASS","ATTR","CHILD","PSEUDO"]
    for(type in Expr.filter){
      //matchExpr对象包含ID,CLASS等选择器的正则
      if((match = matchExpr[type].exec(soFar)) &&
         //是否存在type类型的预过滤函数
         //preFilters对象只对ATTR,CHILD,PSEUDO进行过滤
         (!preFilters[type] || 
          //进行预过滤并重新赋值
          //预过滤举例 soFar = :nth-of-type(n)
          //1. 正则匹配后 match = [":nth-of-type(n)","nth","of-type","n","(0)","undefined(n)","undefined","","1"]
          //2. 过滤会对第4,5个数组元素进行设置 0,(n)
          //3. 如果是nth-child(odd/even) 第4,5会设置为2,1(0)
          (match = preFilters[type](match)))){
        //如果是无需过滤的 比如CLASS:(".div") TAG("div")
        //那么CLASS: match = [".div","div"] TAG: match = ["div","div"]
        matched = match.shift();
        tokens.push({
          value : matched,	//matched就是原匹配字符串
          type : type,		//type是匹配类型CLASS,TAG等
          matches : match	//match数组
        });
        soFar = soFar.slice(matched.length);
      }
    }
    //如果匹配完了 就退出
    if(!matched){
      break;
    } 
  }
  //soFar应该被解析完了 加入缓存
  return parseOnly ? soFar.length : 
  soFar ? Sizzle:error(selector) : 
  tokenCache(selector,groups).slice(0);
}
```



















