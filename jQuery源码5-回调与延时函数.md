# jQuery源码5-Callbacks()

## 变量



##  正则

#### 环视

| 名字   | 语法       |
| ---- | -------- |
| 右肯定  | (?=...)  |
| 右否定  | (?!...)  |
| 左肯定  | (?<=...) |
| 左否定  | (?<!...) |
|      |          |



---



#### rnothtmlwhite

```javascript
//全局匹配空格
var rnothtmlwhite = /[^\x20\t\r\n\f]+/g;
```



---



## 回调函数





#### Callbacks

```javascript
//可选参数列表
//once : 执行后清空列表 
//memory : add返回上次执行结果
//unique : 不重复添加同一个回调函数
//stopOnFalse : 当一个回调返回false中断执行
jQuery.Callbacks = function(options){
  //将传进来的参数转换为对象
  options = typeof options === "string" ? 
    createOptions(options) : jQuery.extend({},options);
  
      //回调函数正在执行
  var firing,
      //最后执行的值 为memory保存
      memory,
      //循环执行的结束位置
      fired,
      //保存once参数
      locked,
      //执行队列
      list = [],
      //给memory用
      queue = [],
      //执行队列长度
      firingIndex = -1,
      
      //执行函数
      fire = function(){
        //once参数
        locked = options.once;
        
        //执行完毕和执行中设为true
        fired = firing = true;
        //每次执行将firingIndex重置-1
        //firingIndex有可能先执行memory的函数
        for(;queue.length;firingIndex = -1){
          //执行队列首元素弹出给memory记录
          //这里同时相当于queue.length--
          //如果是add方法调用 queue.length就变成0了
          memory = queue.shift();
          //如果是add调用的fire方法 firingIndex === list.length
          while(++firingIndex < list.length){
            //检测同时执行回调函数
            if(list[firingIndex].apply(memory[0],memory[1]) === false && 
               //若设置了stopOnFalse属性 
               options.stopOnFalse){
              //跳出循环memory设为false防止add方法返回值
              firingIndex = list.length;
              memory = false;
            }
          }
        }
        //如果没有传memory参数 把之前shift出来的元素删除
        if(!options.memory){
          memory = false;
        }
        
        //执行完了 执行中设为false
        firing = false;
        
        //如果传进once参数 队列清空
        if(locked){
          //"once memory"清空队列
          if(memory){
            list = [];
          }
          //"once"
          else{
            //禁止add()
            list = "";
          }
        }
      },
      
      //保存方法的对象
      self = {
        //添加执行函数
        add: function(){
          //once情况下
          if(list){
            //有memory参数且队列不在执行中
            if(memory && !firing){
              //把执行索引指向最后一个函数 即memory函数
              firingIndex = list.length - 1;
              queue.push(memory);
            }
            //闭包
            (function add(args){
              jQuery.each(args,function(_,arg){
                //外部函数add的参数为函数时
                if(jQuery.isFunction(arg)){
                  //这个是判断unique
                  if(!options.unique || !self.has(arg)){
                    list.push(arg);
                  }
                }
                //arg是类数组时
                else if(arg && arg.length && 
                        jQuery.type(arg) !== "string"){
                  //递归添加执行函数
                  add(arg);
                }
              });
            })(arguments);
            //如果有memory参数 就要返回一个值
            if(memory && !firing){
              fire();
            }
          }
          return this;
        },
        
        //队列中移除一个回调函数
        remove: function(){
          jQuery.each(arguments,function(_,arg){
            var index;
            //arg在list中 索引位置为index
            while((index = jQuery.inArray(arg,list,index)) > -1){
              //删除该arg
              list.splice(index,1);
              
              //
              if(index <= firingIndex){
                firingIndex--;
              }
            }
          });
          return this;
        },
        
        //队列中是否有该方法
        //不传参查询队列中是否有方法
        has: function(fn){
          return fn ? 
            jQuery.inArray(fn,list) > -1 : list.length > 0;
        },
        
        //清空队列
        empty: function(){
          if(list){
            list = [];
          }
          return this;
        },
        
        //停止执行并清空队列
        disable: function(){
          locked = queue = [];
          list = memory = "";
          return this;
        },
        
        //检测队列是否为空
        disabled: function(){
          return !list;
        },
        
        //锁定队列
        lock: function(){
          locked = queue = [];
          //不在执行中就清空队列和memory
          if(!memory && !firing){
            list = memory = "";
          }
          return this;
        },
        
        //查询是否被锁定
        locked: function(){
          return !!locked;
        },
        
        //
        fireWith: function(context,args){
          //没有被锁定时
          if(!locked){
            args = args || [];
            args = [context,args.slice ? args.slice() : args];
            //插入队列
            queue.push(args);
            //执行
            if(!firing){
              fire();
            }
          }
          return this;
        },
        
        //执行队列
        fire: function(){
          self.fireWith(this,arguments);
        },
        
        //是否执行完毕
        fired: function(){
          return !!fired;
        }
      };
  //返回callback对象
  return self;   
};
```



---



## jQuery.extend



```javascript
jQuery.extend({...});
```



---



#### Deferred

```javascript
Deferred: function(fnc){
  var tuples = [
    //进度回调列表
    ["notify","progress",
     jQuery.Callbacks("memory"),
     jQuery.Callbacks("memory"),2],
    //成功回调列表 
    ["resolve","done",
     jQuery.Callbacks("once memory"),
     jQuery.Callbacks("once memory"),0,"resolved"],
    //失败回调列表
    ["reject","fail",
     jQuery.Callbacks("once memory"),
     jQuery.Callbacks("once memory"),1,"rejected"]
  ],
      state = "pending",
      promise = {
        state: function(){
          return state;
        },
        always: function(){
          deferred.done(arguments).fail(arguments);
          return this;
        },
        "catch": function(){
          return promise.then(null,fn);
        },
        //
        pipe: function(){
          var fns = arguments;

          return jQuery.Deferred(function(newDefer){
            jQuery.each(tuples,function(i,tuple){
              //
              var fn = jQuery.isFunction(fns[tuple[4]]) && fns[tuple[4]];

              //
              deferred[tuple[1]](function(){
                var returned = fn && fn.apply(this,arguments);
                if(returned && jQuery.isFunction(returned.promise)){
                  returned.promise()
                    .progress(newDefer.notify)
                    .done(newDefer.resolve)
                    .fail(newDefer.reject);
                }
                else{
                  newDefer[tuple[0] + "With" ](this,fn ? [returned] : arguments);
                }
              });
            });
            fns = null;
          }).promise();
        },
        then: function(onFulfilled,onRejected,onProgress){
          var maxDepth = 0;
          function resolve(depth,deferred,handler,special){
            return function(){
              var that = this,
                  args = arguments,
                  mightThrow = function(){
                    var returned,then;
                    if(depth < maxDepth){
                      return;
                    }
                    
                    //
                    returned = handler.apply(that,args);
                    
                    //
                    if(returned === deferred.promise()){
                      throw new TypeError("Thenable self-resolution");
                    }
                    
                    //
                    then = returned && 
                      (typeof returned === "object" || 
                       typeof returned === "function") && returned.then;
                    
                    //
                    if(jQuery.isFuntion(then)){
                      //
                      if(special){
                        then.call(
                          returned,
                          resolve(maxDepth,deferred,Identity,special),
                          resolve(maxDepth,deferred,Thrower,special)
                        );
                      }
                      //
                      else{
                        //
                        maxDepth++;
                        
                        then.call(returned,
                                 resolve(maxDepth,deferred,Identity,special),
                                 resolve(maxDepth,deferred,Thrower,special),
                                 resolve(maxDepth,deferred,Identity,
                                         deferred.notifuWith)
                                 );
                      }
                    }
                    //
                    else{
                      //
                      if(handler !== Identity){
                        that = undefined;
                        args = [returned];
                      }
                      (special || deferred.resolveWith)(that,args);
                    }
                  },
                  process = special ? 
                  mightThrow : function(){
                    try{
                      mightThrow();
                    }
                    catch(e){
                      if(jQuery.Deferred.exceptionHook){
                        jQuery.Deferred.exceptionHook(e,process.stackTrace);
                      }
                      if(depth + 1 >= maxDepth){
                        //
                        if(hanlder !== Thrower){
                          that = undefined;
                          args = [e];
                        }
                        deferred.rejectWith(that,args);
                      }
                    }
                  };
              if(depth){
                process();
              }
              else{
                if(jQuery.Deferred.getStackHook){
                  process.stackTrace = jQuery.Deferred.getStackHook();
                }
                window.setTimeout(process);
              }
            };
          }
          return jQuery.Deferred(function(newDefer){
            //
            tuple[0][3].add(
              resolve(0,
                      newDefer,
                      jQuery.isFunction(onProgress) ? 
                      onProgress : Identity,
                      newDefer.notifyWith)
            );
            
            //
            tuple[1][3].add(
              resolve(0,
                      newDefer,
                      jQuery.isFunction(onFulfilled) ? 
                      onFulfilled : Identity)
            );
            
            //
            tuple[2][3].add(
              resolve(0,
                      newDefer,
                      jQuery.isFunction(onRejected) ? 
                      onRejected : Thrower)
            );
          }).promise();
        },
        
        //
        promise: function(obj){
          return obj != null ? jQuery.extend(obj,promise) : promise;
        }
      },
      deferred = {};
  jQuery.each(tuples,function(i,tuple){
    var list = tuple[2],
        stateString = tuple[5];
    
    //
    promise[tuple[1]] = list.add;
    
    //
    if(stateString){
      list.add(
        function(){
          state = stateString;
        },
        tuples[3-i][2].disable,
        tuples[0][2].lock
      );
    }
    
    //
    list.add(tuple[3].fire);
    
    //
    deferred[tuple[0]] = function(){
      deferred[tuple[0] + "With"](this === deferred ? undefined : this,arguments);
      return this;
    };
    deferred[tuple[0] + "With"] = list.fireWith;
  });
  //
  promise.promise(deferred);
  
  //
  if(fnc){
    func.call(deferred,deferred);
  }
  
  //All done...然而并看不懂
  return deferred;
}
```



---



#### when

```javascript
when: function(singleValue){
  //
  var remaining = arguments.length,
      //
      i = remaining,
      resolveContexts = Array(i),
      resolveValues = slice.call(arguments),
      //
      master = jQuery.Deferred(),
      //
      updateFunc = function(i){
        return function(value){
          resolveContexts[i] = this;
          resolveValues[i] = arguments.length > 1 ? slice.call(arguments) : value;
          if(!(--remaining)){
            master.resolveWith(resolveContexts,resolveValues);
          }
        };
      };
  if(remaining <= 1){
    adoptValue(singleValue,master.done(updateFunc(i)).resolve,master.reject);
    if(master.state() === "pending" || 
       jQuery.isFunction(resolveValues[i] && resolveValues[i].then)){
      return master.then();
    }
  }
  
  while(i--){
    adoptValue(resolveValues[i],updateFunc(i),master.reject);
  }
  return master.promise();
}
```



---







## jQuery.fn.extend





---



#### promise

```javascript
promise: function(type,obj){
  var tmp,
      count = 1,
      defer = jQuery.Deferred(),
      elements = this,
      i = this.length,
      resolve = function(){
        if(!(--count)){
          defer.resolveWith(elements,[elements]);
        }
      };
  if(typeof type !== "string"){
    obj = type;
    type = "undefined";
  }
  type = type || "fx";
  
  while(i--){
    tmp = dataPriv.get(elements[i],type + "dequeueHooks");
    if(tmp && tmp.empty){
      count++;
      tmp.empty.add(resolve);
    }
  }
  resolve();
  return defer.promise(obj);
}
```



---





## 辅助方法





#### adoptValue

```javascript
function adoptValue(value,resolve,rejuect){
  var method;
  
  try{
    //
    if(value && jQuery.isFunction((method = value.promise))){
      method.call(value).done(resolve).fail(reject);
    }
    //
    else if(value && jQuery.isFunction((method = value.then))){
      method.call(value,resolve,reject);
    }
    //
    else{
      resolve.call(undefined,value);
    }
  }
  catch(value){
    reject.call(undefined,value);
  }
}
```



---



#### createOptions

```javascript
//缓存函数
function createOptions(options){
  var object = {};
  jQuery.each(options.match(rnothtmlwhite) || [],function(_,flag){
    object[flag] = true;
  });
  return object;
}
```



---



#### Identity

```javascript
function Identity(v){
  return v;
}
```



---



#### Thrower

```javascript
function Thrower(ex){
  throw ex;
}
```



---



#### etc

```javascript

```



