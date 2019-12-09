# Promise/A+ 规范的实现

> 官方文档地址 https://promisesaplus.com/ 本文不涉及具体用法，只从具体实现上出发

# 第一步，实现同步版的promise

promise包含几个关键词：

- resolve
- reject
- then

其中resolve和reject的代码在正常使用的时候是看不到的，但可以猜测他们两个都应该是回调函数，传递给了用户传入的函数，而then则挂在原型上

结构如下：

```
calss Promise{
    consturctor(exector){
        function resolve(){
            
        }
        function reject(){

        }
        exector(resolve,reject)
    }
    then(){

    }
}
复制代码
```

文档中提出，promise具备三种逻辑判断状态pending、fulfilled、rejected，三者不并处，同一时间只能存在一种状态

pending向fulfilled或rejected单向流动。

## 同步版本实现

```
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
class Promise{
    constructor(exector){
        let self = this;
        self.status = PENDING;
        self.value = undefined;
        self.reason = undefined;
        let resolve = (value)=>{
            if(self.status === PENDING){
                self.status = FULFILLED
                self.value = value;
            }
        }
        let reject = (reason)=>{
            if(self.status === PENDING){
                self.status = REJECTED;
                self.reason = reason
            }
        }
        try{
            exector(resolve,reject)
        }catch(e){
            reject(e)
        }
    }
    then(onFulfilled,onRejected){
        let self = this;
        if(self.status === FULFILLED){
            onFulfilled(self.value);
        }
        if(self.status === REJECTED){
            onRejected(self.reason);
        }
    }
};
复制代码
```

现在测试一下：

```
new Promise((resolve,reject)=>{
    reject("1");
}).then((data)=>{
    console.log(data)
},(reason)=>{
    console.log(reason)
})
复制代码
```

resolve和reject都能够得到准确输出

## 存在的问题

### 异步不支持

```
new Promise((resolve,reject)=>{
    setTimeout(()=>{
        resolve(2)
    })
}).then((data)=>{
    console.log(data)
},(reason)=>{
    console.log(reason)
})
复制代码
```

现在用setTimeout包裹resolve，结果就是什么输出也没有

同步的话，resolve先执行，onFulfilled后执行，此时状态已经变成了fulfilled

异步的话，then先执行，此时status还是pending，无法进入fulfilled状态，所以onFulfilled不会执行，setTimeout之后resolve改变status，但已经找不到onFulfilled了

### then没有容错

```
new Promise((resolve,reject)=>{
    resolve(2)
}).then()
复制代码
```

像上面这种情况，如果没有给then方法传递参数，那么程序会报错

## 解决存在的问题

### 异步promise

```
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
class Promise{
    constructor(exector){
        let self = this;
        self.status = PENDING;
        self.value = undefined;
        self.reason = undefined;
        self.onResolveCallBacks = [];
        self.onRejectCallBacks = [];
        let resolve = (value)=>{
            if(self.status === PENDING){
                self.status = FULFILLED
                self.value = value;
                self.onResolveCallBacks.forEach(cb=>cb(self.value));
            }
        }
        let reject = (reason)=>{
            if(self.status === PENDING){
                self.status = REJECTED;
                self.reason = reason
                self.onRejectCallBacks.forEach(cb=>cb(self.reason));
            }
        }
        try{
            exector(resolve,reject)
        }catch(e){
            reject(e)
        }
    }
    then(onFulfilled,onRejected){
        let self = this;
        if(self.status === FULFILLED){
            onFulfilled(self.value);
        }
        if(self.status === REJECTED){
            onRejected(self.reason);
        }
        if(self.status === PENDING){
            self.onResolveCallBacks.push(onFulfilled);
            self.onRejectCallBacks.push(onRejected);
        }
    }
};
复制代码
```

因为resolve和then的执行顺序无法保证，所以要用订阅发布的方式来实现，所以创建两个数组，分别存储成功和失败的回调函数

```
self.onResolveCallBacks = [];
self.onRejectCallBacks = [];
复制代码
```

然后再改写resolve和reject函数，执行发布

```
let resolve = (value)=>{
    if(self.status === PENDING){
        self.status = FULFILLED
        self.value = value;
        self.onResolveCallBacks.forEach(cb=>cb(self.value));
    }
}
复制代码
```

此时进行测试：

```
new Promise((resolve,reject)=>{
    setTimeout(()=>{
        reject("失败")
    },2000)
}).then((data)=>{
    console.log(data);
},(reason)=>{
    console.log(reason);
})
复制代码
```

成功输出`失败`，到此异步的promise完成了

### then容错

总错这里不太好理解，所以后面再做

# 第二步，Promise的链式调用

promise支持链式调用，使用方法如：

```
new Promise((resolve,reject)=>{
    //resolve or reject
}).then((d)=>{
    return d
},(r)=>{
    return r
}).then((d)=>{
    return d
},(r)=>{
    return r
}).then((data)=>{
    console.log(data);
},(reason)=>{
    console.log(reason);
})
复制代码
```

上面的.then中return后再then就是链式调用

前面知道promise实例具备then方法，所以如果我们的then方法执行后，再返回一个Promise方法的话，不就可以继续进行.then了吗？

## 同步链式调用

```
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
class Promise{
    constructor(exector){
        let self = this;
        self.status = PENDING;
        self.value = undefined;
        self.reason = undefined;
        self.onResolveCallBacks = [];
        self.onRejectCallBacks = [];
        let resolve = (value)=>{
            if(self.status === PENDING){
                self.status = FULFILLED
                self.value = value;
                self.onResolveCallBacks.forEach(cb=>cb(self.value));
            }
        }
        let reject = (reason)=>{
            if(self.status === PENDING){
                self.status = REJECTED;
                self.reason = reason
                self.onRejectCallBacks.forEach(cb=>cb(self.reason));
            }
        }
        try{
            exector(resolve,reject)
        }catch(e){
            reject(e)
        }
    }
    then(onFulfilled,onRejected){
        // onFulfilled = typeof onFulfilled == 'function'?onFulfilled:value=>value;
        // onRejected = typeof onRejected == 'function'?onRejected:reason => {throw reason};
        let self = this;
        if(self.status === FULFILLED){
            return new Promise((resolve,reject)=>{
                let x = onFulfilled(self.value);
                resolve(x);
            })
        }
        if(self.status === REJECTED){
            return new Promise((resolve,reject)=>{   
            let x = onRejected(self.reason);
                reject(x);
            })
        }
        if(self.status === PENDING){
            self.onResolveCallBacks.push(onFulfilled);
            self.onRejectCallBacks.push(onRejected);
        }
    }
};
复制代码
```

这里核心就是then方法返回了一个新的promise实例:

```
return new Promise((resolve,reject)=>{
    let x = onFulfilled(self.value);
    resolve(x);
})
复制代码
```

1，pormise(p1)执行resolve，改变status为fulfilled

2，then方法执行onFulfilled

3，onFulfilled返回一个新的promise，简称p2

4，由于p1是resolve，所以p2也知行resolve

5，p2执行，改变了p2的status为fulfilled

6，p2继续执行p2的onFulfilled

至此就做到了一个同步的链式调用

## 异步链式调用

```
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
class Promise{
    constructor(exector){
        let self = this;
        self.status = PENDING;
        self.value = undefined;
        self.reason = undefined;
        self.onResolveCallBacks = [];
        self.onRejectCallBacks = [];
        let resolve = (value)=>{
            if(self.status === PENDING){
                self.status = FULFILLED
                self.value = value;
                self.onResolveCallBacks.forEach(cb=>cb(self.value));
            }
        }
        let reject = (reason)=>{
            if(self.status === PENDING){
                self.status = REJECTED;
                self.reason = reason
                self.onRejectCallBacks.forEach(cb=>cb(self.reason));
            }
        }
        try{
            exector(resolve,reject)
        }catch(e){
            reject(e)
        }
    }
    then(onFulfilled,onRejected){
        let self = this;
        if(self.status === FULFILLED){
            return new Promise((resolve,reject)=>{
                let x = onFulfilled(self.value);
                resolve(x);
            })
        }
        if(self.status === REJECTED){
            return new Promise((resolve,reject)=>{   
            let x = onRejected(self.reason);
                reject(x);
            })
        }
        if(self.status === PENDING){
            return new Promise(function(resolve,reject){
                self.onResolveCallBacks.push(function(){
                    let x = onFulfilled(self.value);
                    resolve(x);
                });
                self.onRejectCallBacks.push(function(){
                    let x = onRejected(self.reason);
                    reject(x);
                });
            })
        }
    }
};

复制代码
```

这里的核心是当status为pending状态时，同样要`立刻`返回一个promise对象，否则没有返回值的话，第二次链式调用的then方法根本不存在，需要仔细理解这里

```
if(self.status === PENDING){
    return new Promise(function(resolve,reject){
        self.onResolveCallBacks.push(function(){
            let x = onFulfilled(self.value);
            resolve(x);
        });
        self.onRejectCallBacks.push(function(){
            let x = onRejected(self.reason);
            reject(x);
        });
    })
}
复制代码
```

# 第三步，链式调用其他的promise实现

经过前面，我们已经实现了一个链式调用的promise，但还存在一种情况，就是如果第一个onFulfilled返回一个Promise对像的话怎么处理

```
new Promise((resolve,reject)=>{
    setTimeout(()=>{
        resolve(new Promise((resolve,reject)=>{
            resolve("10000");
        }))
    },3000)
}).then((d1)=>{
    console.log(d1);
    return d1
},(r1)=>{
    console.log(r1);
    return r1;
}).then((d2)=>{
    console.log(d2);
},(r2)=>{
    console.log(r2);
})
复制代码
```

例如上面的情况，拿到的结果就出现了问题

```
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";
class Promise{
    constructor(exector){
        let self = this;
        self.status = PENDING;
        self.value = undefined;
        self.reason = undefined;
        self.onResolveCallBacks = [];
        self.onRejectCallBacks = [];
        let resolve = (value)=>{
            if(self.status === PENDING){
                self.status = FULFILLED
                self.value = value;
                self.onResolveCallBacks.forEach(cb=>cb(self.value));
            }
        }
        let reject = (reason)=>{
            if(self.status === PENDING){
                self.status = REJECTED;
                self.reason = reason
                self.onRejectCallBacks.forEach(cb=>cb(self.reason));
            }
        }
        try{
            exector(resolve,reject)
        }catch(e){
            reject(e)
        }
    }
    then(onFulfilled,onRejected){
        // onFulfilled = typeof onFulfilled == 'function'?onFulfilled:value=>value;
        // onRejected = typeof onRejected == 'function'?onRejected:reason => {throw reason};
        let self = this;
        if(self.status === FULFILLED){
            return new Promise((resolve,reject)=>{
                let x = onFulfilled(self.value);
                resolve(x);
            })
        }
        if(self.status === REJECTED){
            return new Promise((resolve,reject)=>{   
            let x = onRejected(self.reason);
                reject(x);
            })
        }
        if(self.status === PENDING){
            return new Promise(function(resolve,reject){
                self.onResolveCallBacks.push(function(){
                    let x = onFulfilled(self.value);
                    if(x instanceof Promise){
                        x.then(resolve,reject)
                    }else{
                        resolve(x);
                    }
                });
                self.onRejectCallBacks.push(function(){
                    let x = onRejected(self.reason);
                    if(x instanceof Promise){
                        x.then(resolve,reject);
                    }else{
                        reject(x);
                    }
                });
            })
        }
    }
};
复制代码
```

这一次关键的代码是

```
 return new Promise(function(resolve,reject){
    self.onResolveCallBacks.push(function(){
        let x = onFulfilled(self.value);
        if(x instanceof Promise){
            x.then(resolve,reject)
        }else{
            resolve(x);
        }
    });
    self.onRejectCallBacks.push(function(){
        let x = onRejected(self.reason);
        if(x instanceof Promise){
            x.then(resolve,reject);
        }else{
            reject(x);
        }
    });
})
复制代码
```

1，通过instanceof判断了x是否是promise对象

2，如果x是promise对象，称为p3

3，将p2的resolve当作p3的onFulfilled

4，当p3resolve的时候，实际之行的是p2的resolve

5，p2的resolve执行的时候，会执行第二次的then方法中添加进来的onFulfilled

## 继续完善链式调用

```
new Promise((resolve,reject)=>{
    // setTimeout(()=>{
        resolve("成功")
    // },3000)/
}).then((d1)=>{
    console.log(d1);
    return new Promise((resolve,reject)=>{
        resolve(new Promise((resolve,reject)=>{
            resolve("111")
        }));
    })
},(r1)=>{
    console.log(r1);
    return r1;
}).then((d2)=>{
    console.log(d2);
},(r2)=>{
    console.log(r2);
})
复制代码
```

例如上面这种，多层promise对象不断返回的场景

之前的写法只能满足一层需求

所以需要使用递归来实现，除此之外还要考虑边界问题，至于边界可以对照文档来逐步实现,全部实现代码如下：

```
const PENDING =  'pending';//初始态
const FULFILLED =  'fulfilled';//初始态
const REJECTED =  'rejected';//初始态
function Promise(executor){
  let self = this;//先缓存当前promise实例
  self.status = PENDING;//设置状态
  //定义存放成功的回调的数组
  self.onResolvedCallbacks = [];
  //定义存放失败回调的数组
  self.onRejectedCallbacks = [];
  //当调用此方法的时候，如果promise状态为pending,的话可以转成成功态,如果已经是成功态或者失败态了，则什么都不做
  //2.1
  function resolve(value){ //2.1.1
    if(value!=null &&value.then&&typeof value.then == 'function'){
      return value.then(resolve,reject);
    }
    //如果是初始态，则转成成功态
    //为什么要把它用setTimeout包起来
    setTimeout(function(){
      if(self.status == PENDING){
        self.status = FULFILLED;
        self.value = value;//成功后会得到一个值，这个值不能改
        //调用所有成功的回调
        self.onResolvedCallbacks.forEach(cb=>cb(self.value));
      }
    })

  }
  function reject(reason){ //2.1.2
    setTimeout(function(){
      //如果是初始态，则转成失败态
      if(self.status == PENDING){
        self.status = REJECTED;
        self.value = reason;//失败的原因给了value
        self.onRejectedCallbacks.forEach(cb=>cb(self.value));
      }
    });

  }
  try{
    //因为此函数执行可能会异常，所以需要捕获，如果出错了，需要用错误 对象reject
    executor(resolve,reject);
  }catch(e){
    //如果这函数执行失败了，则用失败的原因reject这个promise
    reject(e);
  };
}
function resolvePromise(promise2,x,resolve,reject){
  if(promise2 === x){
    return reject(new TypeError('循环引用'));
  }
  let called = false;//promise2是否已经resolve 或reject了
  if(x instanceof Promise){
    if(x.status == PENDING){
      x.then(function(y){
        resolvePromise(promise2,y,resolve,reject);
      },reject);
    }else{
      x.then(resolve,reject);
    }
  //x是一个thenable对象或函数，只要有then方法的对象，
  }else if(x!= null &&((typeof x=='object')||(typeof x == 'function'))){
    //当我们的promise和别的promise进行交互，编写这段代码的时候尽量的考虑兼容性，允许别人瞎写
   try{
     let then = x.then;
     if(typeof then == 'function'){
       //有些promise会同时执行成功和失败的回调
       then.call(x,function(y){
         //如果promise2已经成功或失败了，则不会再处理了
          if(called)return;
          called = true;
          resolvePromise(promise2,y,resolve,reject)
       },function(err){
         if(called)return;
         called = true;
         reject(err);
       });
     }else{
       //到此的话x不是一个thenable对象，那直接把它当成值resolve promise2就可以了
       resolve(x);
     }
   }catch(e){
     if(called)return;
     called = true;
     reject(e);
   }

  }else{
    //如果X是一个普通 的值，则用x的值去resolve promise2
    resolve(x);
  }
}
//onFulfilled 是用来接收promise成功的值或者失败的原因
Promise.prototype.then = function(onFulfilled,onRejected){
  //如果成功和失败的回调没有传，则表示这个then没有任何逻辑，只会把值往后抛
  //2.2.1
  onFulfilled = typeof onFulfilled == 'function'?onFulfilled:function(value){return  value};
  onRejected = typeof onRejected == 'function'?onRejected:reason=>{throw reason};
  //如果当前promise状态已经是成功态了，onFulfilled直接取值
  let self = this;
  let promise2;
  if(self.status == FULFILLED){
    return promise2 = new Promise(function(resolve,reject){
      setTimeout(function(){
        try{
          let x =onFulfilled(self.value);
          //如果获取到了返回值x,会走解析promise的过程
          resolvePromise(promise2,x,resolve,reject);
        }catch(e){
          //如果执行成功的回调过程中出错了，用错误原因把promise2 reject
          reject(e);
        }
      })

    });
  }
  if(self.status == REJECTED){
    return promise2 = new Promise(function(resolve,reject){
      setTimeout(function(){
        try{
          let x =onRejected(self.value);
          resolvePromise(promise2,x,resolve,reject);
        }catch(e){
          reject(e);
        }
      })
    });
  }
  if(self.status == PENDING){
   return promise2 = new Promise(function(resolve,reject){
     self.onResolvedCallbacks.push(function(){
         try{
           let x =onFulfilled(self.value);
           //如果获取到了返回值x,会走解析promise的过程
           resolvePromise(promise2,x,resolve,reject);
         }catch(e){
           reject(e);
         }

     });
     self.onRejectedCallbacks.push(function(){
         try{
           let x =onRejected(self.value);
           resolvePromise(promise2,x,resolve,reject);
         }catch(e){
           reject(e);
         }
     });
   });
  }
}
module.exports = Promise;
复制代码
```

主要是通过resolvePromise方法将具体的递归，以及边界问题全部处理完毕

# 第四步，promise api实现

## Promise.all

接收一个数组，全部成功后才返回

```
Promise.all = function(arr){
    return new Promise((resolve,reject)=>{
        let resolvList=[];
        arr.forEach((item)=>{
            item.then((data)=>{
                resolvList.push(data);
                console.log(data);
                if(arr.length == resolvList.length){
                    resolve(resolvList);
                }
            },(reason)=>{
                reject(reason);
            })
        })
    })
}
复制代码
```

## Promise.race

接受一个数组，一个成功即返回

```
Promise.race = function(arr){
    return new Promise((resolve,reject)=>{
        arr.forEach((item)=>{
            item.then((data)=>{
                resolve(data);
            },(reason)=>{
                reject(reason);
            })
        })
    })
}
复制代码
```

## Promise.resolve

立刻返回一个promise对象，一般用于没有promise对象，需要将一个东西，转为promise

```
Promise.resolve = function(value){
    return new Promise(function(resolve){
      resolve(value);
    });
  }
复制代码
```

## Promise.reject

立刻返回一个promise对象，一般用于没有promise对象，需要将一个东西，转为promise

```
Promise.reject = function(reason){
  return new Promise(function(resolve,reject){
    reject(reason);
  });
}
```