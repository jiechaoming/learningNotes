# JavaScript知识点

# 一.浏览器运行的原理

>**了解浏览器的运行原理，有助于理解js的EventLoop**

### 进程和线程

​	进程是CPU资源分配的最小单位，一个进程拥有多个线程，而线程是CPU调度的最小单位。可以把进程比作一个工厂，线程就是工厂中的工人，每个工厂相互独立（进程之间也可以互相通信，但代价比较大），CPU要给每个工厂都分配相应的资源，工厂中有一个至多个员工，这些员工一起协作去完成工厂的任务，员工共享工厂内的空间。而我们说的单线程就是指一个进程中只有一个线程。

### 浏览器的进程和线程

>tip：浏览器是多线程的，每打开一个tab栏，就相当于新建了一个进程，但浏览器存在优化的方式

​	**浏览器为什么是多线程**：避免单个页面崩溃影响到整个浏览器，避免第三方插件崩溃影响到整个浏览器，多进程可以充分利用多核的优势。

​	**浏览器主要的4个进程**：

- browser进程：该进程只有一个，负责管理、创建、销毁tab页（一个tab页就是一个进程），负责页面的前进和后退，负责网络资源的上传和下载，负责将从Rendered进程中获取到的Bitmap，绘制到用户界面。
- GPU进程：该进程只要一个，复制3D渲染
- 第三方插件进程：每种类型的第三方插件就对应着一个进程，仅当该插件被使用的时候才会调用。
- Rendered进程（浏览器内核）：每个tab界面，都用一个Rendered进程负责页面渲染，脚本执行，事件处理。

​	**Rendered进程中的五大线程**：

- GUI渲染线程：
  - 解析html，css，构建dom树，style树，根据dom树和style树去构建RenderObject，布局和绘制，注意这里的绘制是指生成Bitmap，真真绘制到界面上是由browser进程完成的。
  - 当触发重绘和回流的时候，触发该线程，进行重新布局
  - GUI线程和js引擎线程是互斥的，js执行的时候，GUI线程会被挂起，有关于GUI的更新（js执行过程中，对界面的改变）会被保留在一个队列中，等js引擎空闲即js执行完后，立即执行GUI线程。
- js引擎线程
  - js引擎即浏览器内核，负责执行js脚本，如著名的V8
  - js引擎一直等待任务队列中任务的到来，一个Rendered进程中只有一个js引擎线程来运行js程序。
  - 因为GUI线程和js引擎线程互斥，所以如果js执行的时间过长，会导致页面渲染阻塞，最终产生白页面
  - 不确定的理解：一个浏览器中只有一个js引擎，js引擎为每一个rendered进程创建一个js引擎线程。但js引擎同一时间只能执行一个js线程，所以js是单线程的。
- 事件触发线程
  - 该线程属于浏览器，不属于js引擎，用来控制事件循环
  - js引擎遇到setTimeout，鼠标点击，或者ajax请求等事件的时候，会将对应的事件添加到事件触发线程中，当对应的事件符合触发条件时，该线程就会把事件添加到任务队列中，等待js引擎的处理。
- 定时器触发线程
  - 定时间由浏览器计时，是因为js引擎可能执行阻塞，会影响计时的准确性
  - 通过单独的线程来计时并触发，计时结束后，添加到事件队列中，等待js引擎的执行，倘若此时js引擎是忙碌的，那么定时器结束后就不能立即执行定时器事件，所以定时器任务不一定是准时的
  - W3C规定，setTImeout的最小时间间隔是4ms
- 异步http请求线程
  - 在XMLHttpRequest连接后，通过浏览器新开一个线程去请求网络资源
  - 在检查到状态变更后，倘若有设置回调函数，就把回调函数放入事件队列中，等待js引擎的执行



# 二.JavaScript的原型和原型链

>##### 诞生的原因：原型链是JavaScript中最主要的继承方法，

### 原型，实例，构造函数的关系

​	每一个构造函数都可以通过**prototype**去访问各自原型对象，原型对象可以通过**constructor**指针指回构造函数，构造函数创建地实例中可以通过**[[proto]]**去访问原型对象。如果原型对象是另一个构造函数的实例，那么这个原型对象本身就会存在一个内部指针去指向另一个原型对象，以此类推。这样实例和原型之间就创建了一条原型链。

```javascript
function SuperType(){
    this.property = true;
}
SuperType.prototype.getSupervalue = function(){
    return this.property;
}
function subType(){
    this.subproperty = false;
}
subType.prototype = new SuperType;
subType.prototype.getSubvalue = fucntion(){
    return this.subproperty;
}
let instance = new subType();
console.log(instance.getSupervalue(),instance.property);
```

​	根据原型搜索机制，当实例对象上不存在某个属性或者方法时，会沿着原型链由下自上，查看原型对象上是否存在相应的属性或者方法，倘若找到就返回相应的值。倘若到原型链的末端还未找到就返回undefined。在上述例子中，方法getSupervalue，先查询instance，然后查询subType.prototype，最后在superType.prototype查询成功，最后返回方法getSupervalue的值。

### 默认原型

JavaScript有一个自带的构造函数Object，Object的prototype指向一个原型对象(也是我们所说的原型链的末端)，这个原型对象上有toString()，valueof()，hasOwnProperty()等方法或属性。默认情况下，所有函数的默认原型对象都是构造函数Object的一个实例，这就是自定义的对象有toString()，valueof()等方法或者变量的原因。

### 原型与继承关系

原型和实例的关系，可以通过操作符instanceof和方法isPrototype()来判断.

# 三.JavaScript中的作用域和闭包

# 四.JavaScript中this的指向

> **this提供了一种更加优雅的方式，来隐式传递一个对象的引用**

### 对this的误解

- this指向函数本身

  ```javascript
  function foo(){
    for(let i = 0 ; i < 6 ; i++){
      this.count++;
    }
  }
  foo.count = 0;
  for(let i = 0 ; i < 5 ; i++){
    foo();
  }
  console.log(foo.count);
  //此时输出的值为0，证明了函数foo中的this，并不指向函数本身
  
  ```

  ------

  解决方法：

  - 利用词法作用域

    ```javascript
    function foo(){
      for(let i = 0 ; i < 6 ; i++){
        data.count++;
      }
    }
    data = {
    	count : 0,
    }
    ```

    

  - 利用函数名称标识符

    ```javascript
    function foo(){
      for(let i = 0 ; i < 6 ; i++){
        foo.count++;
      }
    }
    foo.count = 0;
    //匿名函数没有函数名
    ```

    

  - 利用call，apply，bind来强制绑定this

    ```javascript
    function foo(){
      for(let i = 0 ; i < 6 ; i++){
        foo.count++;
      }
    }
    foo.count = 0;
    for(let i = 0 ; i < 5 ; i++){
      foo().call(foo);
    }
    
    ```

- this指向函数的作用域

  this在任何情况下都不指向函数的词法作用域。在JavaScript内部，作用域确实和对象类似，可见的标识符都是他的属性。但作用域“对象”无法通过JavaScript代码访问，它存在于JavaScript引擎内部。

  ```javascript
  //以下是一个错误的例子
  function foo(){
  	var a = 2;
  	this.bar()
  }
  function bar(){
  	console.log(this.a);
  }
  foo();
  //倘若this指向foo函数的作用域的话，那么bar()函数中应该可以通过this.a来访问到foo()中的变量a。
  ```

  ​	这段代码试图通过this.bar()来引用bar函数，但这是错误的，应该省略前面的this.，直接使用词法引用标识符。此外还试图通过this联通foo()和bar()的词法作用域。从而让bar可以访问到foo中的变量a，但这这是不可能实现的，你不能使用this来引用一个词法作用域内部的东西。

  >在上诉错误例子中，bar函数还是可以被正常调用，因为调用函数foo的是全局对象，所以foo函数中的this也是指向全局对象的，bar函数也是定义在全局对象当中，所以this.bar是可以正常被调用的。

### this到底是什么

this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式，即this是在运行时绑定的，不是在编写时绑定的。当一个函数被调用时，会创建一个活动记录（有时也被称作执行上下文）。这个记录会包含函数在那里被调用，函数的调用方法，传入的参数等，而this就是记录的其中一个属性，一个会在函数执行过程中会被用到的属性。

#### 函数的调用位置

调用位置即函数在代码中被调用的位置，调用栈即为了到达当前执行位置所调用的所有函数，而调用位置就在当前正在执行的函数的前一个调用中。

```javascript
function baz(){
  //当前调用栈是:baz,因此当前调用位置是全局作用域
  console.log('baz');
  bar();
}
function bar(){
   //当前调用栈是:baz->bar,因此当前调用位置是baz
  console.log('bar');
  foo()
}
function foo(){
  //当前调用栈是:baz->bar->foo,因此当前调用位置是bar
  console.log('foo');
}
baz(); //<----baz的调用位置
```

### this的绑定规则

#### 默认绑定

this默认指向全局，最常见的如函数独立调用。

```javascript
var a = 2 ;
function foo(){
	console.log(this.a);
}
//这里的foo就是前面不带任何修饰的独立调用,此时的this指向全局对象
foo();   // 2
```

------

倘若当前是严格模式，那么全局对象将无法使用默认绑定，此时的this将会指向undefined

```javascript
function foo(){
	“use strict”
	console.log(this.a);
}
foo();           //Type:this is undefined
```

------

带修饰的函数调用

```javascript

var a = 2 ;
function foo(){
	console.log(this.a);
}
obj = {
	a: 3,
}
obj.foo2 = foo;
//这里的foo2前面有"obj."，这个就是函数foo2被调用时，前面的修饰。此时foo2的this就指向obj
obj.foo2();   // 3

```

------

注意：虽然this的绑定规则完全取决于调用位置，但是只有foo()运行在非严格模式下时，默认绑定才能绑定到全局对象；严格模式下与foo()的调用位置无关：

```javascript
var a = 2 ;
function foo(){
	console.log(this.a);
}
(function(){
    "use strict"
    foo();  // 2   
})()

(function(){
    "use strict"
    function foo2(){
        console.log(this.a);
    }
    foo2();  // a is not undefined;
})()

```

上述例子中，严格模式也和变量一样用于全局作用和局部作用，foo是声明在全局且foo的调用是独立调用，所以在上诉例子中，foo的调用相当于在全局中直接调用foo()，那么IIFE中的严格模式自然而然的无法影响到foo。 foo2是在IIFE中定义的，那么foo2函数也会受到在IIFE的严格模式的影响（是一个作用域的问题）。

#### 隐式绑定

this指向调用位置的上下文，或者说是否被修饰

```javascript
function foo(){
	console.log(this.a);
}
obj = {
	a: 3,
	foo:foo,
}
obj.foo();   // 3

obj2 = {
    a:2,
    obj:obj,
}
//对象属性引用链，只有最后一层会影响调用位置。   
obj2.obj.foo()  // 3      

```

注意：无论是直接在obj中声明foo函数，还是先声明foo函数，在把foo函数添加到obj对象中，这个函数严格来说都不属于obj对象，但调用位置会使用obj上下文来引用函数，因此可以说函数被调用时，obj对象拥有或者包含这个函数。

------

隐式丢失（绑定丢失）

```javascript
function foo(){
    console.log(this.a);
}
var obj = {
    a:2,
    foo:foo,
}
var bar = obj.foo;
var a = "global";
//bar是obj.foo的一个引用，但实际上，它引用的是foo函数本身，因此bar()就是独立函数调用   
bar();   //globa;
```

```javascript
function foo(){
    console.log(this.a);
}

function doFoo(fn){
	fn()
}
var obj = {
    a:2,
    foo:foo,
}
var a = "global"
doFoo(obj.foo);   //global
```

#### 显式绑定

- call/apply

  两个方法的第一个参数都是对象，当你传递的是一个原始值的来当做this的绑定对象时，这个原始值会被自动转换成它的对象形式即利用new String(xxx)，new Boolean(xxx)，new Number(xxx)等。两者的区别是call的剩余参数必须一个个列出来，apply可以接收一个参数数组。但并没有解决绑定丢失的问题

- bind

  ```javascript
  function bind(fn,obj){
  	return function(){
  		return fn.call(obj,arguments);
  	}
  }
  function foo(){
      console.log(this.a);
  }
  var obj = {
      a:2,
      foo:foo,
  }
  var bar = bind(obj.foo,obj);
  var a = "global";  
  bar();   // 2;
  
  //简易的bind函数解决了绑定丢失的问题，由于这是一个频繁的操作，所以ES提供了bind方法
  ```

#### new绑定

- JavaScript中的构造函数

  JavaScript中的构造函数，只是被new操作符调用的普通函数，它们并不会属于某个类，也不会实例化一个类

- 构造函数被调用时执行的步骤

  - 创建一个全新的对象

  - 这个对象会被执行[[原型]]连接

  - 这个新对象会绑定到函数调用的this

  - 如果函数没有返回其他对象，那么new表达式中的函数调用就会自动返回这个新对象

    ```javascript
    function foo(a){
    	this.a = a;
    }
    var bar = new foo(2);
    console.log(bar.a);  // 2
    ```

​						更多详情可以阅读下文的手写call/apply/bind

#### 绑定规则优先级

new > 硬  > 隐式 > 默认

### 规则之外的特例——箭头函数

​	箭头函数的作用域是根据外层函数的作用域来决定的，箭头函数的this也和外层函数的this一样

```
function foo(){
	return (a)=>{
		console.log(this.a);
	}
}
var obj1 = {
	a:2
}
var obj2 = {
	a:3
}
var bar = foo.call(obj1);
bar.call(obj2);   // 2,不是2
```

------

ES5实现箭头函数的效果（利用闭包）

```
function foo(){
	var self = this;
	setTimeout(function(){
		console.log(self.a)
	},100);
}

var obj = {
	a:2,
}
foo.call(obj);   // 2,不是undefined
```



#  五.JavaScript中class

# 六.Promise

### **关键的几个问题**

1. **如何改变promise的状态**
   - resolve(value)：如果当前是pending状态，执行该方法后，就会变成resolved状态
   - reject(value)：如果当前pending状态，执行该方法后，就会变成rejected状态
2. **一个promise可以指定多个成功或者失败的回调函数，都会调用吗**
   - 当promise状态发生改变的时候，相应的回调函数都会进行改变
3. **改变promise的状态和指定回调函数，谁先谁后**
   - 都有可能，可以先指定回调在改变状态，也可以先改变状态在指定回调
   - 如果先指定回调，那么当状态发生改变后，回调函数会立刻执行。如果先改变状态，那么指定回调函数后，函数会立刻执行。
4. **promise.then()返回的新的promise的结果状态由什么决定**
   - 如果抛出了异常，返回的新的promise的状态就位rejected，reason就是抛出的错误
   - 如果返回的是一个非promise的任意值，新promise的状态就为resolved，value为返回的值
   - 如果返回的是一个新的promise，此promise的状态将决定的promise.then返回的promise的状态
5. **promise如何串联多个操作任务**
   -  promise.then()返回的是一个新的promise,所以可以通过then()的链式调用 将多个异步或者同步任务串联起来
6. **promise异常穿透**
   - 当使用promise的then()进行链式调用时，可以在最后指定.catch()即失败的回调，前面任何操作出现了异常，都会传到最后的.catch()这个失败的回调中
7. **如何中断一个promise链**
   - 返回一个pending状态的promise

```javascript
(function(window){
    /**
   * Promise 构造函数
   * executor：执行器函数（同步执行）
   * */
    function Promise(executor){
        // 注意resolve函数的调用者，this.status ='resolved';this的指向不是当前对象
        this.PromiseState = 'pending'; //给promise对象指定PromiseState属性，初始值为pending
        this.PromiseResult = null;  //给promise对象指定一个用于存放结果数据的属性
        this.callbacks = [];     //每个元素的结构：{onResolved{},onRejected(){}}  
        const self = this
        function resolve(value){
            if(self.PromiseState !== 'pending') return
            // 将状态改为resolved
            self.PromiseState ='fulfilled';
            // 保存value数据
            self.PromiseResult = value;
            // 调用成功的回调(当状态发生改变的时候，调用回调函数)
            if(self.callbacks.length > 0){
                setTimeout(()=>{
                    self.callbacks.forEach(item=>{
                        item.onResolved(value);
                    })
                })
            }
        }
        function reject(reason){
            if(self.PromiseState !== 'pending') return
            self.PromiseState ='rejected';
            self.PromiseResult = reason;
            if(self.callbacks.length > 0){
                setTimeout(()=>{
                    self.callbacks.forEach(item=>{
                        item.onRejected(reason);
                    })
                })
            }
        }

        // 模仿promise处理错误的情况
        try { 
            // 立即同步执行executor
            executor(resolve,reject);
        } catch (error) {
            reject(error)
        }

    }

    /**
   * Promise的实例对象上面有两个实例方法，then和catch
   * then：指定成功和失败的回调函数，返回的是一个新的promise对象
   * catch：指定失败的回调函数，返回的也是一个新的promise对象
   * */ 
    Promise.prototype.then = function(onResolved,onRejected){
        const self = this;
        if(typeof onRejected !== 'function'){
            onRejected = function(reason){
                throw reason;
            }
        }
        if(typeof onResolved !== 'function'){
            onResolved = function(value){
                throw value;
            }
        }
        return new Promise((resolve,reject)=>{
            function callback(type){
                try {
                    let result = type(self.PromiseResult);
                    if(result instanceof Promise){
                        result.then(v=>{
                            resolve(v)
                        },r=>{
                            reject(r);
                        })
                    }else{    //执行结果为成功(因为程序没有异常结束)
                        resolve(result);
                    }
                } catch (error) {
                    reject(error);
                }
            }
            if(this.PromiseState === 'fulfilled'){//状态为了fulfilled就是已经成功了，然后在指定回调
                setTimeout(()=>{
                    callback(onResolved);
                })
            }
            if(this.PromiseState === 'rejected'){ //同fulfilled
                //因为then的执行应该是异步的，这里用setTimeout来模拟
                setTimeout(()=>{
                    callback(onRejected);
                })
            }

            if(this.PromiseState === 'pending'){//状态为pending就是先指定了回调函数，在改变状态
                // 保存回调函数(因为可能指定多个then)
                this.callbacks.push({
                    onResolved:function(){
                        callback(onResolved);
                    },
                    onRejected:function(){
                        callback(onRejected); 
                    }
                }); 
            }
        })

    }
    Promise.prototype.catch = function(onRejected){
        return this.then(undefined,onRejected);
    }

    /**
   * Promise有两个对象方法方法，resolve和reject
   * resolve：返回的是一个成功的promise（目前准确，后面就不准确了）
   * reject：返回的是一个失败的promise（目前准确，后面就不准确了）
   * */ 
    Promise.resolve = function(value){
        return new Promise((resolve,reject)=>{
            if(value instanceof Promise){
                value.then(v=>{
                    resolve(v);
                },r=>{
                    reject(r);
                })
            }else{
                resolve(value);
            }
        })
    }
    Promise.reject = function(reason){
        return new Promise((resolve,reject)=>{
            reject(reason);
        })
    }

    /**
   * Promise有两个对象方法方法，all和race
   * all：返回一个promise，当所有promise都成功时才成功，否则就返回失败
   * race：也是返回一个Primose，其结果由第一个成功的promise决定
   * */ 
    Promise.all = function(promises){          //接收的是promise对象数组
        return new Promise((resolve,reject)=>{
            let count = 0;
            let arr = [];
            for(let i = 0 ; i<promises.length ; i++){
                promises[i].then(v=>{
                    count++;
                    arr[i] = v;
                    if(count == promises.length){
                        resolve(arr);
                    }
                },r=>{
                    reject(r);
                })
            }
        })
    }
    Promise.race = function(promises){
        return new Promise((resolve,reject)=>{
            for(let i = 0 ; i<promises.length ; i++){
                promises[i].then(v=>{
                    resolve(v);
                },r=>{
                    reject(r);
                })
            }
        })
    }
    //向外暴露Promise实例
    window.Promise = Promise
})(window)
```



# 七.await,async

# 八.节流和防抖

### 防抖

```javascript
function decounce(fun,wait = 1000){
  let timer;
  return function(...args){
    if(timer){
      clearTimeout(timer);
    }
    timer = setTimeout(()=>{
        
      fun.apply(this,args);
    },wait)
  }
}
```

### 节流

```javascript
//方式1，初始触发的时候，不会立即执行，会等待delay后，才会执行第一次函数
function throttle(fun,delay){
  let timer
  return function(...args){
    if(timer){
      return;
    }
    timer = setTimeout(() => {
      fun.apply(this,args);
      timer = null;
    }, delay);
  }
}

//方式2，利用时间戳来计算，初始触发的时候，会立即执行一次函数。
function throttle(fun,delay){
  let pre = 0;
  return function(...args){
    let now = new Date();
    if(now -pre > delay){
      fun().apply(this,args);
      pre = now;
    }
  }
}

```



# 九.数组的深浅拷贝

### 浅拷贝

- Object.assign

- 扩展运算符

- 数组的concat和slice方法

  ```javascript
  const objArr = [{age:18},{name:'gl'},{address:'gz'}];
  const target = Object.assign([],objArr);
  const target2 = [...objArr];
  const target3 = objArr.concat();
  const target4 = objArr.slice();
  ```



### 深拷贝

```javascript
function deepCloneSimple(value){
  return JSON.parse(JSON.stringify(value));
}

function deepClone(value,map = new Map()){
  if(typeof value == 'object'){
    if(map.get(value)){
      return map.get(value);
    }
    const cloneTarget = Array.isArray(value) ? [] : {};
    map.set(value,cloneTarget);
    for(const key in value){
      cloneTarget[key] = deepCloneUpdated(value[key],map);
    }
    return cloneTarget;
  }else{
    return value;
  }
}
```



# 十.call，apply，bind

### call

```javascript
//纯ES6语法实现
Function.prototype.myCall = function(context,...args){
  if(typeof this != 'function'){
    throw new TypeError('error');
  }
  context = context ? Object(context) : globalThis;
  context.fn = this;
  let result = context.fn(...args);
  console.log(result,'ddd');
  delete context.fn;
  return result;
}


```

### 	 apply

```javascript
Function.prototype.myApply = function(context,args){
  if(typeof this != 'function' || !Array.isArray(args)){
    throw new TypeError('error');
  }
  context = context ? Object(context) : globalThis;
  context.fn = this;
  let result = context.fn(...args);
  delete context.fn;
  return result;
}
```

### bind

bind是返回一个新的函数，传参情况和call一样

```javascript
Function.prototype.myBind = function(context,...args){
  if(typeof this != 'function'){
    throw new TypeError('error');
  }
  context = context ? Object(context) : globalThis;
  const _this = this;
  return function fn(...args2){
    if(this instanceof F) {
      return new _this(...args, ...args2)
    }else{
      return _this.apply(context,...args,...args2);
    }
  }
}
```



# 十一.对象的属性

## 对象的属性描述符

> 在 ES5 之前，JavaScript 语言本身并没有提供可以直接检测属性特性的方法，比如判断属 性是否是只读。 但是从 ES5 开始，所有的属性都具备了属性描述符。

#### 属性描述符介绍

一个普通的对象属性对应的属性描述符（也被称为“数据描述符”）包含四个特性：`value`（值）`writable`（可写）、 `enumerable`（可枚举）、`configurable`（可配置）。使用`Object.getOwnPropertyDescriptor()`

可以获取到对象的属性描述。

```javascript
var myObject = { 
 a:2
};
Object.getOwnPropertyDescriptor( myObject, "a" );
// { value: 2, writable: true, enumerable: true, configurable: true }
```

在创建普通属性时属性描述符会使用默认值，我们也可以使用 `Object.defineProperty()` 来添加一个新属性或者修改一个已有属性（如果它是 configurable）并对特性进行设置，但通常我们并不会这样做。

```javascript
var myObject = {};
Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true, 
    configurable: true, 
    enumerable: true
}); 
myObject.a; // 2
```

------

#### 属性描述符详解

1. writable

   该值为true，对象的属性值才可以修改，为false则无法修改

   ```javascript
   var myObject = {};
   Object.defineProperty(myObject, "a", {
       value: 2,
       writable: false, // 不可写！
       configurable: true,
       enumerable: true
   });
   myObject.a = 3;
   myObject.a; // 2
   
   //倘若当前处于严格模式下，修改a的值会报错
   "use strict";
   .
   .
   .
   .
   myObject.a = 3; // TypeErro
   ```

   

2. configurable

   只要改值为true，就可以使用 `defineProperty()` 方法来修改属性描述符，把 configurable 修改成 false 是单向操作，无法撤销。不管是不是处于严格模式，尝试修改一个不可配置的属性描述符都会报错。

   ```javascript
   var myObject = {
       a: 2
   };
   myObject.a = 3;
   myObject.a; // 3
   
   Object.defineProperty(myObject, "a", {
       value: 4,
       writable: true,
       configurable: false, // 不可配置！
       enumerable: true
   });
   myObject.a; // 4 
   myObject.a = 5;
   myObject.a; // 5
   Object.defineProperty(myObject, "a", {
       value: 6,
       writable: true,
       configurable: true,
       enumerable: true
   }); // TypeError
   
   delete myObject.a;         //delete无效
   myObject.a; // 2
   ```

   但即便`configurable:false`，我们还是可以 把 writable 的状态由 true 改为 false，但是无法由 false 改为 true。我们也可以通过value描述符，去修改属性的值。但为false的情况下delete语句是失效的（静默失败，不会报错）。

3. Enumerable

   从名字就可以看出，这个描述符控制的是属性是否会出现在对象的属性枚举中，比如说 `for..in` 循环。如果把 enumerable 设置成 false，这个属性就不会出现在枚举中，虽然仍 然可以正常访问它。相对地，设置成 true 就会让它出现在枚举中。 用户定义的所有的普通属性默认都是 enumerable。

## 对象的[[Get]]和[[Set]]

#### [[Get]]

```javascript
var myObject = {
 a: 2
};
myObject.a; // 2
```

myObject.a是一次属性访问，但实际上是实现了`[[Get]]`操作( 有点类似函数调用[[Get]] () )，[[Get]]操作会首先在对象中查找是否有名称相同的属性， 如果找到就会返回这个属性的值。如果没有找到名称相同的属性，按照 [[Get]] 操作会去遍历原型链找到名称相同的属性并返回，若原型链上也未找到相应的属性，[[Get]] 会返回一个undefined。

#### [[Put]]

与[[Get]]操作对应的就是用于存值操作[[Put]]，但并不是每一次给对象的属性赋值都会触发 [[Put]] 来设置或者创建这个属性。具体步骤如下：

1. 属性是否是访问描述符？如果是并且存在 setter 就调用 setter。 
2. 属性的数据描述符中 writable 是否是 false ？如果是，在非严格模式下静默失败，在 严格模式下抛出 TypeError 异常。 
3. 如果都不是，将该值设置为属性的值。
4. 如果对象中不存在这个属性，[[Put]] 操作会更加复杂（后续讨论）。

## Getter和Setter

getter和setter 是一个隐藏函数，分别在对象属性获取和对象属性设置的时候调用，可以用于改写默认操作，但改写只能应用于对象的单个属性，无法应用到整个属性。当你给一个属性定义 getter、setter 或者两者都有时，这个属性会被定义为“访问描述符”（和“数据描述符”相对）。对于访问描述符来说，JavaScript 会忽略它们的 value 和 writable 特性，取而代之的是关心 set 和 get（还有 configurable 和 enumerable）特性。

> ***note***：数据描述符：即name，name的属性描述符中有writable和value；如myObject.age，age就是数据描述符。当你给age属性设置了getter后，它就变成访问描述符。(个人理解，可能有误)

```javascript
var myObject = {
    get a() {
        return 2;
    }
};
Object.defineProperty(
    myObject,
    "b", 
    { 
        get: function () {
            return this.a * 2
        },
        enumerable: true
    }
);
console.log(myObject.a,myObject.b);  // 2,4

myObject.a = 3;
myObject.a; // 2
```

两种定义方式，都会在对象中创建一个不包含值的属性，对于这个属性的访问会自动调用一个隐藏函数， 它的返回值会被当作属性访问的返回值。由于我们只定义了getter，所以对a的setter操作会被自动忽略。所以通常getter和setter是成对出现的。

```javascript
var myObject = {
    // 给 a 定义一个 getter
    get a() {
        return this._a_;
    },
    // 给 a 定义一个 setter
    set a(val) {
        this._a_ = val * 2;
    }
};
myObject.a = 2;
myObject.a; // 4

//名称 _a_ 只是一种惯例，没有任何特殊的行为,和其他普通属性一样
```

# 十二.proxy

> Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

## 基本使用

`new Proxy()`表示生成一个`Proxy`实例，`target`参数表示所要拦截的目标对象，`handler`参数也是一个对象，用来定制拦截行为。Proxy 实例也可以作为其他对象的原型对象。

```javascript
//var proxy = new Proxy(target, handler);
var proxy = new Proxy({}, {
    get: function (target, propKey) {
        return 35;
    }
});

proxy.time // 35
proxy.name // 35
```



倘若handler是一个空对象，那么将每日每日任何拦截效果，访问proxy相当于访问target

```javascript
var target = {};
var handler = {};
var proxy = new Proxy(target, handler);
proxy.a = 'b';
target.a // "b"
```

## proxy的实例方法

### **get(target, propKey, receiver?)**

拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。三个参数依次是目标对象，属性名，proxy实例本身，最后一个可选。

```javascript
var person = {
  name: "张三"
};

var proxy = new Proxy(person, {
  get: function(target, propKey) {
    if (propKey in target) {
      return target[propKey];
    } else {
      throw new ReferenceError("Prop name \"" + propKey + "\" does not exist.");
    }
  }
});

proxy.name // "张三"
proxy.age // 抛出一个错误
```



利用get()实现数组读取负数的索引

```javascript
function createArray(...elements) {
  let handler = {
    get(target, propKey, receiver) {
      let index = Number(propKey);
      if (index < 0) {
        propKey = String(target.length + index);
      }
      return target[propKey];
    }
  };

  let target = [];
  target.push(...elements);
  return new Proxy(target, handler);
}

let arr = createArray('a', 'b', 'c');
const a = arr[-1] // c
```



### **set(target, propKey, value, receiver?)**

拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值且值只能为true，严格模式下，`set`代理如果没有返回`true`，就会报错。四个参数分别是目标对象、属性名、属性值， Proxy 实例本身。`set`代理应当返回一个布尔值，

```javascript
let validator = {
  set: function(obj, prop, value) {
    if (prop === 'age') {
      if (!Number.isInteger(value)) {
        throw new TypeError('The age is not an integer');
      }
      if (value > 200) {
        throw new RangeError('The age seems invalid');
      }
    }

    // 对于满足条件的 age 属性以及其他属性，直接保存
    obj[prop] = value;
    return true;
  }
};

let person = new Proxy({}, validator);

person.age = 100;

person.age // 100
person.age = 'young' // 报错
person.age = 300 // 报错
```



### **apply(target, object, args)**

拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。三个参数分别是目标对象、目标对象的上下文对象（`this`），目标对象的参数数组。

```javascript
var target = function () { return 'I am the target'; };
var handler = {
  apply: function () {
    return 'I am the proxy';
  }
};

var p = new Proxy(target, handler);
p()   //I am the proxy
```

### has()

### construct

## proxy和defineProperty的区别

Object.defineProperty



















# 十三.Reflect

# 十四.Module

> ES6 的模块自动采用严格模式，不管你有没有在模块头部加上`"use strict";`。

## 背景

ES6之前，JavaScript并没有模块化体系，但社区中定义了CommonJS和AMD两种模块化加载方案，前者用于服务器，后者用于浏览器。ES6从语言层级设计了模块功能，完全可以代替CommonJS和AMD。ES6模块可以在编译时就确定模块的依赖关系，以及输入输出的变量。但CommonJS和AMD比较在代码运行时才能确定。

```javascript
// CommonJS模块
let { stat, exists, readfile } = require('fs');
```

上述代码的本质是加载`fs`这个模块中的全部方法，然后生成一个`fs对象`，在从`fs对象`中读取需要的三个方法。这种方式叫做“运行时加载 ”，这个加载方式是无法在编译阶段做静态优化。

```javascript
// ES6模块
import { stat, exists, readFile } from 'fs';
```

ES6 模块不是对象，而是通过`export`命令显式指定输出的代码，再通过`import`命令输入。上述代码的实质是从`fs`模块加载 3 个方法，其他方法不加载。这种加载称为“编译时加载”或者静态加载，即 ES6 可以在编译时就完成模块加载，效率要比 CommonJS 模块的加载方式高。

## export命令

一个模块就是一个独立的文件，该文件内部的所有变量，外部是无法获取到。如果希望外部可以访问的模块内部的某个变量，就必须使用export关键字去导出该变量。

### **导出变量**

```javascript
// profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1958;
```

```javascript
// profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1958;

export { firstName, lastName, year };
//这种方式更加推荐
```

### **导出函数或者类**

```javascript
export function multiply(x, y) {
  return x * y;
};
```

```javascript
//可以使用as关键字来对导出变量进行重命名
function v1() { ... }
function v2() { ... }

export {
  v1 as streamV1,
  v2 as streamV2,
  v2 as streamLatestVersion
};
```

上面代码使用`as`关键字，重命名了函数`v1`和`v2`的对外接口。重命名后，`v2`可以用不同的名字输出两次。

------

**`export`命令规定的是对外的接口，必须与模块内部的变量建立一一对应关系。**

```javascript
// 报错
export 1;

// 报错
var m = 1;
export m;
```

以上两种方法都是错误的，因为没有提供对外的接口。第一张写法是直接输出1，第二种写法直接通过了变量m，但还是直接输出了1。正确的写法如下

```javascript
// 写法一
export var m = 1;

// 写法二
var m = 1;
export {m};

// 写法三
var n = 1;
export {n as m};

```

对于class和函数也遵守着相同的规定

```javascript
// 报错
function f() {}
export f;

// 正确
export function f() {};

// 正确
function f() {}
export {f};
```



> **note**：`export`命令可以出现在模块的任何位置，只要处于模块顶层(全局作用域)就可以。如果处于块级作用域内，就会报错，下一节的`import`命令也是如此。这是因为处于条件代码块之中，就没法做静态优化了，违背了 ES6 模块的设计初衷。

## import命令

使用`export`命令定义了模块的对外接口以后，其他 JS 文件就可以通过`import`命令加载这个模块。`import`命令具有提升效果，会提升到整个模块的头部，首先执行，这种行为的本质是`import`命令是在编译阶段执行的。

```javascript
import { firstName, year, lastName as subName  } from './profile.js';

function setName(element) {
  element.textContent = firstName + ' ' + subName;
}

```

`import`命令输入的变量都是只读的(参考`const`)，因为它的本质是输入接口。也就是说，不允许在加载模块的脚本里面，改写接口。

```javascript
import {a} from './xxx.js'

a = {}; // Syntax Error : 'a' is read-only;

a.foo = 'gz'  //合法操作
```

因为`import`命令是在编译阶段执行的，早于代码运行之前，所以所以不能使用表达式和变量等只有在运行时才能得到结果的语法结构。

```javascript
// 报错
import { 'f' + 'oo' } from 'my_module';

// 报错
let module = 'my_module';
import { foo } from module;
// 报错
if (x === 1) {
  import { foo } from 'module1';
} else {
  import { foo } from 'module2';
}
```

`import`语句会执行所加载的模块，因此可以有下面的写法，但下面代码仅仅执行了lodash模块，并不会输入任何的值

```JavaScript
import 'lodash';
```

除了指定加载某个输出值，还可以使用整体加载，即用星号（`*`）指定一个对象，所有输出值都加载在这个对象上面。

```javascript
import { area, circumference } from './circle';

console.log('圆面积：' + area(4));
console.log('圆周长：' + circumference(14));
```

```javascript
import * as circle from './circle';

console.log('圆面积：' + circle.area(4));
console.log('圆周长：' + circle.circumference(14));
```

## export default 命令 

使用`import`命令的时候，用户需要知道所要加载的变量名或函数名，否则无法加载。为了给用户提供方便，让他们不用知道具体的变量名或者函数名就能加载模块，就要用到`export default`命令，为模块指定默认输出。

```javascript
// export-default.js
export default function () {
  console.log('foo');
}

//foo函数的函数名foo，在模块外部是无效的。加载的时候，视同匿名函数加载。
export default function foo() {
  console.log('foo');
}
```

其他模块加载该模块时，`import`命令可以为该匿名函数指定任意名字。

```javascript
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```

显然，一个模块只能有一个默认输出，因此`export default`命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能唯一对应`export default`命令。

本质上`export default`就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

```javascript
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as foo } from 'modules';
// 等同于
// import foo from 'modules';
```

如果想在一条`import`语句中，同时输入默认方法和其他接口，可以写成下面这样。

```javascript
import def, { each, forEach } from 'lodash';
// def是默认导出的函数
```

对应上面代码的`export`语句如下。

```javascript
export default function (obj) {
  // ···
}

export function each(obj, iterator, context) {
  // ···
}

export { each as forEach };
```

倘若你此时是导出全部

```javascript
import * as c from 'lodash';
//那么此时你调用相应的函数为
def.forEach(obj, iterator, context)
def.each(obj, iterator, context)
def.default(obj)
```

## export 与 import 的复合写法

如果在一个模块之中，先输入后输出同一个模块，`import`语句可以与`export`语句写在一起。

```javascript
export { foo, bar } from 'my_module';

// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar };

// 其他的用法
// 接口改名
export { foo as myFoo } from 'my_module';

// 整体输出
export * from 'my_module';
```

但要注意的是，写成一行后，foo和bar实际上并没有被导入到当前模块，只是对外转发了这两个接口，所以在当前模块并不能去使用foo和bar

ES2020 之前，有一种`import`语句，没有对应的复合写法。 导出全部并重新命名的写法

```JavaScript
import * as someIdentifier from "someModule";
```

ES2020补上了这个写法。

```javascript
export * as ns from "mod";

// 等同于以下，但下面的方式并不会忽略default方法
import * as ns from "mod";
export {ns};
```



## 模块的继承

假设有一个`circleplus`模块，继承了`circle`模块。

```javascript
// circleplus.js
export * from 'circle';
export var e = 2.71828182846;
export default function(x) {
    return Math.exp(x);
}
```

上面代码中的`export *`，表示再输出`circle`模块的所有属性和方法。注意，`export *`命令会忽略`circle`模块的`default`方法。然后，上面代码又输出了自定义的`e`变量和默认方法。

## 跨模块常量

## import ()

# 十五.事件

> HTML和JavaScript的交互是通过事件实现的，事件代表了文档或者浏览器窗口中某个有意义的时刻

## 事件流

事件流描述了页面接收事件的顺序，分别有IE的事件冒泡流和网景的事件捕捉流

## 事件冒泡 

事件被定义为从最具体的元素开始触发，然后向上传播至没有那么具体的元素 

```html
<!DOCTYPE html> 
<html> 
    <head> 
     	<title>Event Bubbling Example</title> 
    </head> 
    <body> 
     	<div id="myDiv">Click Me</div> 
    </body>  
</html>

```

在点击div后，click事件的发生顺序为：`div`  ->  `body` ->  `html` ->  `document`，div元素被点击后，最先触发click事件，然后click事件沿着DOM树一路向上，在经过的每个节点以此触发，直至到达document对象

## 事件捕捉

事件从最不具体的元素开始触发，然后向下传递至最具体的元素，上述例子中，如果使用了事件捕捉，那么触发的顺序将为：`document` ->  `html` ->  `body` ->  `div`，click 事件首先由 document 元素捕获，然后沿 DOM 树依次向下传播，直至到达 实际的目标元素。

## DOM事件流

DOM2 Events 规范规定事件流分为 3 个阶段：`事件捕获`、`到达目标`和`事件冒泡`。事件捕获最先发生， 为提前拦截事件提供了可能。然后，实际的目标元素接收到事件。最后一个阶段是冒泡，最迟要在这个 阶段响应事件。通常把到达目标处理事件看成冒泡阶段的一部分。

## 事件处理程序

事件意味着用户执行了某种操作，比如click，load，mouseover。为了响应事件而调用的函数被叫做事件监听器（或事件处理程序），事件监听器的名字以`on`开头 ，click事件的监听器是onclick，load事件的监听器是onload

### HTML的监听器

```html
<input type="button" value="Click Me" onclick="console.log('Clicked')"/>
```

```html
<script> 
    function showMessage() { 
        console.log("Hello world!"); 
    } 
</script> 

<input type="button" value="Click Me" onclick="showMessage()"/>
```



### DOM0的监听器

```javascript
let btn = document.getElementById("myBtn"); 
btn.onclick = function() { 
    console.log("Clicked"); 
}; 

btn.onclick = null; // 移除事件处理程序
```

### DOM2的监听器

DOM2 Events 为事件处理程序的赋值和移除定义了两个方法：addEventListener()和 removeEventListener()。这两个方法暴露在所有 DOM 节点上，它们接收 3 个参数：事件名、事件处理函 数和一个布尔值，true 表示在捕获阶段调用事件处理程序，false（默认值）表示在冒泡阶段调用事 件处理程序。

```JavaScript
let btn = document.getElementById("myBtn"); 
btn.addEventListener("click", () => { 
 console.log(this.id); 
}, false); 
```

使用 DOM2 方式的主要优势是可以为同一个事件添加多个监听器，多个事件处理程序以添加顺序来触发，因此前面的代码会先 打印元素 ID，然后显示消息“Hello world!”

```javascript
let btn = document.getElementById("myBtn"); 
btn.addEventListener("click", () => { 
    console.log(this.id); 
}, false); 
btn.addEventListener("click", () => { 
    console.log("Hello world!"); 
}, false); 
```

通过 addEventListener()添加的事件处理程序只能使用 removeEventListener()并传入与添 加时同样的参数来移除。这意味着使用 addEventListener()添加的匿名函数无法移除

```javascript
let btn = document.getElementById("myBtn"); 
btn.addEventListener("click", () => { 
    console.log(this.id); 
}, false); 
// 其他代码
btn.removeEventListener("click", function() { // 没有效果！
    console.log(this.id); 
}, false); 


let handler = function() { 
    console.log(this.id); 
}; 
btn.addEventListener("click", handler, false); 
// 其他代码
btn.removeEventListener("click", handler, false); // 有效果
```

大多数情况下，事件处理程序会被添加到事件流的冒泡阶段，主要原因是跨浏览器兼容性好。把事件处理程序注册到捕获阶段通常用于在事件到达其指定目标之前拦截事件。如果不需要拦截，则不要使用事件捕获。

## 事件对象

> event 对象只在事件处理程序执行期间存在，一旦执行完毕，就会被销毁。

在 DOM 中发生事件时，所有相关信息都会被收集并存储在一个名为 event 的对象中。这个对象包含了一些基本信息，比如导致事件的元素、发生的事件类型，以及可能与特定事件相关的任何其他数据。 例如，鼠标操作导致的事件会生成鼠标位置信息，而键盘操作导致的事件会生成与被按下的键有关的信 息。所有浏览器都支持这个 event 对象，尽管支持方式不同

```html
<button onclick="showMessage(event)"></button>
```

在HTML监听器中，你传递的参数必须叫做`event`，才表示你传递的是事件对象。倘若是在vue框架当中，你传递的参数要叫做`$event`，才表示事件对象。

```JavaScript
let btn = document.getElementById("myBtn"); 
btn.onclick = function(event) { 
    console.log(event.type); // "click" 
}; 
btn.addEventListener("click", (event) => { 
    console.log(event.type); // "click" 
}, false);
```

在DOM监听器中，第一个参数就表示事件对象和形参变量名无关。

## 事件类型

- 用户界面事件（UIEvent）：涉及与 BOM 交互的通用浏览器事件。
- 焦点事件（FocusEvent）：在元素获得和失去焦点时触发。
- 鼠标事件（MouseEvent）：使用鼠标在页面上执行某些操作时触发。 
- 滚轮事件（WheelEvent）：使用鼠标滚轮（或类似设备）时触发。
- 输入事件（InputEvent）：向文档中输入文本时触发。 
- 键盘事件（KeyboardEvent）：使用键盘在页面上执行某些操作时触发。 
- 合成事件（CompositionEvent）：在使用某种 IME（Input Method Editor，输入法编辑器）输入字符时触发

## 事件委托

过多事件处理程序”的解决方案是使用事件委托。事件委托利用事件冒泡，可以只使用一个事件 处理程序来管理一种类型的事件。

例如：

```html
<ul id="myLinks"> 
    <li id="goSomewhere">Go somewhere</li> 
    <li id="doSomething">Do something</li> 
    <li id="sayHi">Say hi</li> 
</ul> 
```

给三个绑定点击事件，通常做法是

```javascript
let item1 = document.getElementById("goSomewhere"); 
let item2 = document.getElementById("doSomething"); 
let item3 = document.getElementById("sayHi"); 
item1.addEventListener("click", (event) => { 
    location.href = "http:// www.wrox.com"; 
}); 
item2.addEventListener("click", (event) => { 
    document.title = "I changed the document's title"; 
}); 
item3.addEventListener("click", (event) => { 
    console.log("hi"); 
}); 
```

但这样的写法会出现大片雷同只为指定事件处理程序的代码，倘若数量比较多，那么冗余性会比较高，可读性也会下降，这个时候使用事件委托，只要给所有元素共同的祖先节点添加一个事件处理程序， 就可以解决问题。

```javascript
let list = document.getElementById("myLinks"); 
list.addEventListener("click", (event) => { 
    let target = event.target; 
    switch(target.id) { 
        case "doSomething": 
            document.title = "I changed the document's title"; 
            break; 
        case "goSomewhere": 
            location.href = "http:// www.wrox.com"; 
            break; 
        case "sayHi": 
            console.log("hi"); 
            break; 
    } 
}); 
```

## 删除事件处理程序

把事件处理程序指定给元素后，在浏览器代码和负责页面交互的 JavaScript 代码之间就建立了联系。 这种联系建立得越多，页面性能就越差。除了通过事件委托来限制这种连接之外，还应该及时删除不用 的事件处理程序。

但存在一个误区，常见的处理方式是使用 innerHTML 整体替换页面的某一部分。这时候，被 innerHTML 删除的元素上如果有事件处理程序，就不会被垃圾收集程序正常清 理。通过真正的事件方法 removeChild()或 replaceChild()删除节点。

```html
<div id="myDiv"> 
    <input type="button" value="Click Me" id="myBtn"> 
</div> 

<script type="text/javascript"> 
    let btn = document.getElementById("myBtn"); 
    btn.onclick = function() { 
        // 执行操作
        btn.onclick = null; // 删除事件处理程序
        document.getElementById("myDiv").innerHTML = "Processing..."; 
    }; 
</script>
```

## 模拟事件

> 在JavaScript中的作用很小，暂时不用理会

# 十六.错误处理

## 错误类型

ECMA-262 定义了以下 8八种错误类型。

常见的有4种。

- RangeError  

  在数值越界时抛出，如`let items1 = new Array(-20);` 

- ReferenceError 

  在找不到对象时抛出，如`x没有定义`，`let obj = x`，

- SyntaxError 

  在语法错误时抛出，如 `let 11a = 0`，`let a = “我用来中文的双引号”`，

- TypeError 

  主要发生在变量不是预期类型，或者访问不存在的方法时。如`let test = 'I am a variate'  test()`

不常见的4种

- Error 

  该错误类型是基类型，其他错误类型继承该类型，浏览器很少会抛出 Error 类型的错误，该类型主要用于开发者抛出自定义错误。

- InternalError

  该类型的错误会在底层 JavaScript 引擎抛出异常时由浏览器抛出。例如，递归过多导 致了栈溢出。这个类型并不是代码中通常要处理的错误，如果真发生了这种错误，很可能代码哪里弄错 了或者有危险了。

- EvalError

  使用 eval()函数发生异常时抛出，但官方并不推荐开发者使用eval()函数

- URIError

  只会在使用 encodeURI()或 decodeURI()但传入了格式错误的 URI 时发生。这个错误恐怕是 JavaScript 中难得一见的错误了，因为上面这两个函数非常稳健

# 十七.发布订阅模式

> 发布-订阅模式，看似陌生，其实不然。如 Node.js EventEmitter 中的 on 和 emit 方法；Vue 中的 `$on` 和 `$emit` 方法。其本质都是发布订阅模式

## 定义

发布-订阅模式其实是一种对象间一对多的依赖关系，当一个对象的状态发送改变时，所有依赖于它的对象都将得到状态改变的通知。

订阅者（Subscriber）把自己想订阅的事件注册（Subscribe）到调度中心（Event Channel），当发布者（Publisher）发布该事件（Publish Event）到调度中心，也就是该事件触发时，由调度中心统一调度（Fire Event）订阅者注册到调度中心的处理代码。

## 如何实现发布订阅

- 创建一个对象
- 在该对象上创建一个缓存列表（调度中心）
- on 方法用来把函数 fn 都加到缓存列表中（订阅者注册事件到调度中心）
- emit 方法取到 arguments 里第一个当做 event，根据 event 值去执行对应缓存列表中的函数（发布者发布事件到调度中心，调度中心处理代码）
- off 方法可以根据 event 值取消订阅（取消订阅）
- once 方法只监听一次，调用完毕后删除缓存函数（订阅一次）

## 代码实现(原型版)

```javascript
const eventEmitter = {};
eventEmitter.list = {};
eventEmitter.on = function (event, fn) {
    if (this.list[event]) {
        this.list[event].push(fn)
    } else {
        this.list[event] = [];
        this.list[event].push(fn);
    }
}

eventEmitter.emit = function (event, ...content) {
    const fn = this.list[event];
    if (!fn || fn.length == 0) {
        return false;
    } else {
        fn.forEach(element => {
            element.apply(this, content)
        });
    }
}


eventEmitter.off = function (event, fn) {
    const fns = this.list[event];
    if (!fns || fns.length === 0) {
        return false;
    }
    if (!fn) {
        fns.length = 0;
    } else {
        /**
         * once订阅注册的订阅事件名均为on，
         * 考虑以下场景：还未发布时订阅者需要取消订阅某个once事件，
         * 由于once注册事件函数名均为on，off中cb是无法判断的，
         * 因此需要添加on.fn用来标识区分，所以才有off中判断条件为cb||cb.fn
         */
        for (let i = 0; i < fns.length; i++) {
            const cb = fns[i];
            if (cb === fn || cb.fn === fn) {
                fns.splice(i, 1);
            }
        }
    }

}

eventEmitter.once = function (event, fn) {
    const _this = this;

    function on(...content) {
        fn.apply(this, content);
        _this.off(event, on)
    }
    on.fn = fn;
    this.on(event, on);
}

//测试
function user1 (content) {
    console.log('用户1订阅了:', content);
}

function user2 (content) {
    console.log('用户2订阅了:', content);
}

function user3 (content) {
    console.log('用户3订阅了:', content);
}

function user4 (content) {
    console.log('用户4订阅了:', content);
}

// 订阅
eventEmitter.on('article1', user1);
eventEmitter.on('article1', user2);
eventEmitter.on('article1', user3);

// 取消user2方法的订阅
eventEmitter.off('article1', user2);

eventEmitter.once('article2', user4)

// 发布
eventEmitter.emit('article1', 'Javascript 发布-订阅模式');
eventEmitter.emit('article1', 'Javascript 发布-订阅模式');
eventEmitter.emit('article2', 'Javascript 观察者模式');
eventEmitter.emit('article2', 'Javascript 观察者模式');
```

## 代码实现(class版)

对没有通过apply绑定this

```javascript
class eventEmitter {
    constructor() {
        this.list = {};
    }
    on(event, fn) {
        if (!this.list[event]) {
            this.list[event] = [];
        }
        this.list[event].push(fn);
    }
    emit(event, ...content) {
        const fn = this.list[event];
        if (!fn || fn.length === 0) {
            return false
        }
        fn.forEach((f) => {
            f(content)
        })
    }
    off(event, fn) {
        const fns = this.list[event];
        if (fns.length === 0 || !fns) {
            return false
        }
        if (!fn) {
            fns.length = 0;
        } else {
            for (let i = 0; i < fns.length; i++) {
                const cb = fns[i];
                if (cb == fn || cb.fn == fn) {
                    fns.splice(i, 1)
                    break;
                }
            }
        }

    }
    once(event, fn) {
        const _this = this;
        function on(...content) {
            fn(this, content);
            _this.off(event, on)
        }
        on.fn = fn;
        this.on(event, on)
    }
}
```









