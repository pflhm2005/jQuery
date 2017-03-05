# Sizzle引擎流程



## Sizzle



#### 入口函数

```javascript
//selector:css选择器 context:上下文或document results:结果 seed:初始集合
//主函数优先调用浏览器默认解析方法 否则返回select方法继续解析
function Sizzle( selector, context, results, seed){
  var m, i, elem, nid, match, groups, newSelector,
      newContext = context && context.ownerDocument,
      nodeType = context ? context.nodeType : 9;
  
  //初始化结果集
  results = results || [];
  
  //如果selector是非法字符 直接返回结果
  if( typeof selector !== "string" || 
     !selector || nodeType !== 1 && 
     nodeType !==9 && nodeType !== 11){
    return results;
  }
  
  //seed是啥参数啊...
  if( !seed ){
    if( (context ? context.ownerDocument || 
         context : preferredDoc ) !== document ){
      setDocument( context );
    }
    //上下文不设定就设置为全局
    context = context || document;
  }
  //documentIsHTML = !isXML(document)
  if( documentIsHTML ){
    //requickExpr为快速匹配简单ID/CLASS/TAG选择器的正则
    ///^(?:#([\w-]+)|(\w+)|\.([\w-]+))$/
    //1：ID 2：TAG 3：CLASS
    if( nodeType !== 11 && (match = requickExpr.exec( selector )) ){
      //匹配到#()*
      if( m = match[1] ){
        //节点类型为document对象
        if( nodeType === 9 ){
          //
          if( (elem = context.getElementById(m)) ){
            //elem是标签 m是对应标签的ID 相等说明匹配成功
            if( elem.id === m ){
              results.push( elem );
              return results;
            }
            else{
              return results;
            }
          }
        }
        //如果传入的context不是document
        else{
          if( newContext && 
             ( elem = newContext.getElementById( m )) && 
             contains( context, elem ) && elem.id === m ){
            results.push( elem );
            return results;
          }
        }
      }---/* match[1] */
        //匹配到标签div,span等等
        else if( match[2] ){
          push.apply( results, context.getElementsByTagName( selector ) );
          return results;
        }
      //匹配到.*
      else if( (m = match[3]) && support.getElementByClassName && context.getElementByClassName ){
        push.apply( results, context.getElementByClassName( m ) );
        return results;
      }
    }
    //如果支持高级查询querySelectorAll
    if( support.qsa && 
       //且缓存中没有
       !compilerCache[ selector + " " ] && 
       //rbuggQSA = /:enabled|:disabled/
       (!rbuggQSA || !rbuggyQSA.test( selector )) ){
      if( nodeType !==1 ){
        newContext = context;
        newSelector = selector;
      }
      //上下文不是document的情况 没见过
      else if( context.nodeName.toLowerCase() !== "object" ){
        if( (nid = context.getAttribute("id")) ){
          nid = nid.replace( rcessescape, fcssescape );
        }
        else{
          context.setAttribute("id", (nid = expando));
        }
        groups = tokenize( selector );
        i = groups.length;
        while( i-- ){
          //toSelector
          group[i] = "#" + nid + " " + toSelector( groups[i] );
        }
        newSelector = groups.join(",");
        //扩展context为下一个选择器?
        newContext = rsibling.test( selector ) && 
          testContext( context.parentNode ) || context;
      }
      if( newSelector ){
        try{
          //这个方法好叼啊
          push.apply( results, newContext.querySelectorAll( newSelector ));
          //返回jQuery对象
          return results;
        }
        catch( qsaError ){}
        //这个是应付上面的else
        finally{
          if( nid === expando ){
            context.removeAttribute("id");
          }
        }
      }
    }
  }--!seed
  //原生方法搞不定 进入select函数
  //正则替换(" div ") --> ("div")
  return select( selector.replace( rtrim, "$1" ), context, results, seed );
}
```



#### 过滤函数-selector



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



#####  词法解析-tokenize



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



#### 编译-compile



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



##### matcherFromTokens



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



##### addCombinator



```javascript
function addCombinator(match,combinator,base){
  var dir = combinator.dir,
      skip = combinator.next,
      key = skip || dir,
      checkNonElements = base && key === "parentNode",
      doneName = done++;
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

